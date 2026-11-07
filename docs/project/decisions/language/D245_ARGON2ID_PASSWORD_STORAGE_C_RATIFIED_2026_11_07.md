# D245 — Argon2id password storage and verification as a Protos-first Standard Library module (Candidate C)

**Status: RATIFIED DESIGN — exact owner-approved D245-C, 2026-11-07.** This is a durable **non-normative** decision record; it does not alter the specification, prove implementation, or claim JVM/Native cryptographic conformance.

- Design issue: [D245 #1071](https://github.com/guillermomolina/protos/issues/1071).
- Implementation: [I160 #1093](https://github.com/guillermomolina/protos/issues/1093), **allocated, not implemented** at this decision publication.
- Motivating app: [I153 RealWorld #1068](https://github.com/guillermomolina/protos/issues/1068); no account, email, HTTP or JWT semantics enter the library.
- **Approval:** in the active D245 conversation on 2026-11-07, after the comparative D245 A–E proposal and the complete C contract (API, Argon2id parameters, NFC, PHC versioning, explicit entropy, Future/Actor behavior, negative tests, and limitations), the project owner explicitly stated **“apruebo D245-C”**, then authorized Issue updates and direct commit/push publication of bounded records in this repository. No other alternatives or unpresented policies are approved.
- Inspected baseline: [guillermomolina/protos@`08c27760eb3f85105277e5b1b1b159abbca67ebe`](https://github.com/guillermomolina/protos/commit/08c27760eb3f85105277e5b1b1b159abbca67ebe) (Java 25, `0.3.453-SNAPSHOT`); this is the published HEAD inspected, not a claim about a maintainer's uncommitted checkout.
- Comparative research, constraints, risk scenarios, scores, external citations and invariant check: [D245 evidence](../../evidence/D245/D245_C_RESEARCH_APPROVAL_INVARIANT_AUDIT_2026_11_07.md).
- Reused decision: [D243-C explicit secure entropy](./D243_SECURE_ENTROPY_EXPLICIT_CAPABILITY_C_RATIFIED.md), with implementing [I157 #1090](https://github.com/guillermomolina/protos/issues/1090). HMAC D244-C/I159 and JWT D246 are independently scoped.

## 1. Selected contract and authority boundary

Provide one ordinary Standard Library MODULE at **`std:crypto/password`**, with four initial operations:

```protos
Password: import("std:crypto/password")
recordFuture: Password.hash(passwordString, entropy)
recordFuture: Password.hashWithSalt(passwordString, saltBytes)
validFuture: Password.verify(passwordString, storedRecordString)
outdated: Password.needsRehash(storedRecordString)
```

- `hash(String, entropy) -> Future<String>` is the safe default and obtains exactly 16 salt octets from the **explicitly supplied authentic D243-C capability**. Importing this module confers no entropy, Process, network, filesystem, clock, FFI or provider-access authority.
- `hashWithSalt(String, Bytes) -> Future<String>` is the explicit broker/advanced path. It validates genuine standard Bytes containing **exactly 16 octets**, captures an independent exact-octet snapshot at invocation, and never mutates/retains an alias to the supplied Bytes. It cannot prove that the caller chose unpredictable salt; caller responsibility is explicit.
- `verify(String, String) -> Future<Boolean>` returns `true` only for a correct password and a supported valid record, `false` only for an incorrect password and such a record. Invalid, unsupported, tampered or backend-failed inputs fail as errors, never as a successful `false` or insecure fallback. Compare the complete 32-octet tag without content-dependent early exit; no universal constant-time execution guarantee is claimed.
- `needsRehash(String) -> Boolean` performs cheap, bounded, local record validation/inspection and reports whether a valid recognized profile differs from the current creation profile; malformed or unsupported records signal error. Applications decide whether and when to replace a successfully verified record, protecting against concurrent password changes.

The public operations are ordinary Protos module-object members, not new Core/prelude syntax, global classes, key objects, host imports, a crypto-provider registry, or capabilities minted by the library.

## 2. Creation algorithm and persistent representation

New records use **Argon2id v=19/0x13**, with exactly **m=65,536 KiB (64 MiB)**, **t=3 passes**, **p=4 lanes**, **16-octet salt** and **32-octet tag** (RFC 9106 `7.4 second recommended parameter set). This is the fixed initial portable profile, not a runtime-chosen weaker fallback. Deployment must benchmark memory and latency; inability to support the profile fails closed rather than silently lowering work.

Input policy: require a semantic Protos String; normalize to **Unicode NFC**, then encode **strict UTF-8** (no implicit trimming, case folding, conversion, locale interpretation, null truncation or alteration of ordinary String identity). Bound accepted input to **64 KiB of post-NFC UTF-8 bytes**, with a reasonable early bound on work for pathological pre-normalization input. The canonical profile metadata must unambiguously identify `protos-password-v1/nfc` and distinguish future text policies.

Persist a **versioned Protos password record with a strictly parsed PHC-format Argon2id core**; the core spells its algorithm/version/parameters/salt/tag and uses canonical unpadded Base64. For example, the PHC core shape is:

```text
$argon2id$v=19$m=65536,t=3,p=4$<unpadded-base64-16-octet-salt>$<unpadded-base64-32-octet-tag>
```

The persistent outer profile envelope distinguishes NFC from raw UTF-8, so the PHC core is independently interoperable only after matching the selected password-to-octets preprocessing. The exact envelope spelling and strict grammar must be made unambiguous in the normative implementation change, rather than allowing provider-specific/ad hoc formats. Do not use an unversioned PHC-only record that silently loses the NFC policy.

Only explicit, recognized bounded profiles may be verified. Parsers reject unexpected fields, duplicate fields, noncanonical encoding, leading zeros/overflowed numbers, excessive lengths, wrong algorithm, manipulated work factors and unrecognized profiles **before** expensive computation or memory allocation. Existing supported records remain verifiable after later profile upgrades; registration of new profiles is a separate specification evolution and does not automatically authorize arbitrary algorithms.

The contract does **not** include bcrypt, scrypt, PBKDF2 or raw SHA-256/HMAC for new creation, plaintext fallback, legacy SHA-256/MD5 silent imports, or a configurable algorithm selection registry.

## 3. Futures, Actor ownership and resource admission

Argon2id computation is expensive CPU/memory work and MUST NOT occupy a Protos Actor's uninterrupted guest-execution segment while pending. `hash`, `hashWithSalt` and `verify` provide explicit operation Futures, perform bounded host-side work off-turn, then settle the completed result in the originating Actor domain through existing targeted completion/cancellation machinery. A worker must never invoke Protos guest callbacks or construct Actor-owned objects directly.

- Failure after successful ordinary operation dispatch follows the method's Future result channel. Ordinary language lookup/arity errors before dispatch retain ordinary Core error semantics.
- Full result publication is the **only** semantic commitment boundary; no partial record or tag becomes visible. A successful precommit cancellation publishes nothing. Late host work that cannot be physically preempted must be retired under host custody and must not re-enter a terminated Actor.
- Actor termination, provider failure and cancellation must not strand retained Future registrations or leaked queue/memory permits.
- Bound concurrent memory-hard operations and queue admission privately. One derivation requires about 64 MiB for Argon2 alone; 16 simultaneous derivations imply about 1 GiB, excluding overhead. Neither unlimited workers nor unlimited queued password Strings are acceptable.
- Failure categories use a single ordinary exported **Standard Library** `PasswordError` prototype, with an own stable `kind` discriminator (`invalidArgument`, `invalidRecord`, `unsupportedRecord`, `unavailable`, `capacity`). No additional Core error ancestry, Java stack/exception text, or leaked secret diagnostics.
- Reduce exposure of temporary private byte arrays and intermediate tag material with best-effort clearing; no complete JVM/GC/provider memory erasure guarantee exists. The public String is immutable and ordinary Bytes are mutable; neither is a new secure-memory value family.

D243-C retains the live `entropy` grant in the initial RootActor. Because that grant is nontransferable to other Actors and P, a RootActor/owner Actor may expose an ActorRef broker that returns completed 16-byte salt **Bytes** to HTTP workers, which call `hashWithSalt`. No grant transfer, shadow authority, hidden entropy generation, or automatic Host worker delegation is permitted.

## 4. Private host backend and portability

Algorithm selection, public API, profile, authority, validation, lifecycle and error semantics are Protos-owned. The numerical Argon2id primitive may be implemented behind an **exact-module, lazy, private** host bridge, initially with a vetted Java implementation (a Bouncy Castle `Argon2BytesGenerator` is the first candidate) if validated against independent vectors and an actually rebuilt JVM/Native Image. A private FFM/libargon2 backend is an escape path, not a public import or permission.

The bridge must snapshot actual Protos values without relying on guest-overridable `size`/`at` getters for security admission. Do not expose the bridge in module exports, Prelude, Process, unrelated importers, foreign provider registry, or public key classes. Never fall back automatically to SHA-256/HMAC, unbounded Protos-language CPU loops, PBKDF2 or fake success when a provider is unavailable.

No JCA/FIPS validation, Native Image availability, security audit or performance benchmark is claimed by this ratification. A conforming implementation needs real JVM and rebuilt installed Native testing, bounded memory concurrency, and verified interoperability.

## 5. Failure scenarios, migration and explicit exclusions

- A stolen database contains individually salted memory-hard verifiers, not plaintext or quick deterministic hashes. Online attack limits, generic authentication errors, account enumeration defense, compromised-password blocklists, registration and authorization remain **application** policies.
- The library cannot guarantee complete constant-time control flow across Unicode normalization, record validation, VM/JIT, admission and provider internals. It guarantees an equal-length full-tag comparison without secret-content-dependent early exit.
- Existing supported profiles must remain verifiable; `needsRehash` reports whether a new profile should be created after successful authentication. The caller must conditionally replace the stored record, avoiding lost updates under concurrent password changes.
- FIPS-only deployments may later add an explicit PBKDF2 provider/profile after a distinct normative decision; FIPS cannot be claimed merely because JCA exists.
- Future pepper/HSM, Secret/Key types, configurable work-factor policy, general provider registry, alternative KDFs, or delegable entropy capabilities are deferred. Their later addition requires explicit security/authority and compatibility contracts.
- Future runtime/platform changes may replace the private backend while maintaining the same output bytes and accepted record semantics.

**Strongest objection:** 64 MiB Argon2id and a Java-only backend may be expensive or fail on selected Native targets; a PBKDF2/JCA-only solution would be easier. The chosen response is to validate and fail closed, retain a private-backend escape path and require an explicit new decision if the actual target cannot meet the published profile, rather than silently weakening password protection.

## 6. Publication and acceptance

The design is **ratified independently** of its implementation. [I160 #1093](https://github.com/guillermomolina/protos/issues/1093) owns specification changes, production code, tests, Native acceptance, user-facing docs, publication metadata and completion evidence. The normative owner(s) must be selected and updated under `spec/` with `spec/PROTOS_SPEC_CHANGELOG.md`.

No RealWorld credentials, magic demo hash, hardcoded salt or fake test results were produced by this decision. Formal closure of D245 does not imply I160 or I157 is implemented or closed.
