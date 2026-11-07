# D245-C — Comparative research, owner approval and GITHUB021 invariant audit

**Date:** 2026-11-07. **Research/approval evidence, not normative specification, runtime audit, benchmark, or passed tests.**

- Owning decision: [D245 #1071](https://github.com/guillermomolina/protos/issues/1071).
- Ratification: [D245-C](../../decisions/language/D245_ARGON2ID_PASSWORD_STORAGE_C_RATIFIED_2026_11_07.md).
- Subsequent implementation allocation: [I160 #1093](https://github.com/guillermomolina/protos/issues/1093).
- Exact inspected product HEAD: [`08c27760eb3f85105277e5b1b1b159abbca67ebe`](https://github.com/guillermomolina/protos/commit/08c27760eb3f85105277e5b1b1b159abbca67ebe) (`0.3.453-SNAPSHOT`, Java 25); no claim about uncommitted local work or later product state.
- Owner approval: active D245 conversation on 2026-11-07; after a full D245 design packet comparing A (pure Protos), B (JCA/PBKDF2), C (narrow Argon2id private backend), D (general crypto platform) and E (defer), exact public contract, 64 MiB/3/4 profile, NFC/UTF8 policy, PHC versioned records, security and failure semantics were presented, owner responded **“apruebo D245-C”** and requested Issues/evidence publication and an I-family implementation prompt. This is the approval provenance, not an invented GitHub owner comment.

## 1. Governing Protos evidence (read-only)

Read or substantively inspected: `AGENTS.md`; `AGENTS.work/DESIGN.md` (GITHUB010/GITHUB021, mandatory comparison and evidence); `AGENTS.work/REFERENCE.md` (small universe, authority, orthogonality, pay as needed); `AGENTS.work/COORDINATION.md` (issue/status/native hierarchy); `protos/AGENTS.md` (idiomatic source and existing-library reuse); `AGENTS.work/IMPLEMENTATION.md` (test/validation/bump and blocker contract); `spec/semantics/VALUES_AND_COLLECTIONS.md` (Unicode scalar String exact identity and mutable Bytes with independent octet snapshots); `spec/semantics/MODULES.md` (MODULE privacy, Actor-local imports); `spec/semantics/ERRORS.md` (ordinary library errors versus Core taxonomy); `spec/io/TEXT_IO.md` (strict UTF-8 Encoding); `spec/io/IO_CORE.md` (operation commitment); `spec/io/PROCESS_IO.md` (initial RootActor provisioned capabilities); `spec/concurrency/FUTURES_AND_TASKS.md`, `ACTORS.md` (`24D non-preemptive Actor-local CPU, `24J host call offload), `PARALLEL_EXECUTION.md`; `protos/lib/crypto/sha256/module.protos` and its vector tests; `docs/design/STANDARD_LIBRARY_IDEAS.md` (LIB006's explicit HMAC/entropy/password exclusions); `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosCoreBootstrap.java` (exact lazy MODULE facility injection); `src/main/java/com/guillermomolina/protos/execution/facilities/ProtosTlsBackendFacility.java` / `ProtosHttpAsyncFacility.java` (private host bridge), `src/main/java/com/guillermomolina/protos/runtime/io/ProtosBytesValue.java`, `ProtosIoOperation.java`, `ProtosFilesystemOpenFlow.java`, `ProtosActorExecutionDomain.java` and `ProtosFutureValue.java` (real octet snapshots and Actor-targeted host completion); `pom.xml`; `src/main/java/com/guillermomolina/protos/spi/foreign/ProtosForeignProviderPlugin.java` (host-selected foreign providers). Test Tool explicit corpus sources are `protos/tools/test/RepositoryCorpusPlans.protos`, `RepositorySuite.protos` and host registry `ProtosTestCorpusRegistry.java` / `ProtosCli.java`; implementation must inspect current HEAD contents before editing.

Invariants derived:
1. Generic unkeyed SHA-256 is deterministic and is **not** password hashing.
2. Neither import nor the Standard Library mints host entropy; D243-C explicitly places a nontransferable `entropy` capability in the initial RootActor and returns Future<Bytes>; completed Bytes remain ordinary transferable values.
3. Protos String identity is unnormalized Unicode scalar sequence; NFC is **only** a password-protocol input policy, never Core String mutation.
4. Semantically authentic standard Bytes and exact octets matter; guest-shadowable selectors are not an admission/security boundary.
5. CPU-only guest computation without explicit suspension can monopolize Actor turns. Merely offloading a synchronous host call physically does not create Actor-local reentrancy; the password operation must expose a Future.
6. No Core Secret/Key/Provider family, implicit foreign authority, new Process permission or full erasure promise is present or authorized.
7. Future settlement, cancellation and Actor termination must not run guest work on background host threads.

## 2. External comparison and sources

The investigation compared more than five real systems across four architecture groups, emphasizing byte mapping, memory cost, verification and operational failure rather than API name similarity.

| System / approach | Concrete lesson | Limitation for Protos |
| --- | --- | --- |
| [RFC 9106](https://www.rfc-editor.org/rfc/rfc9106) (CFRG Argon2id) | `m=2^16 KiB, t=3, p=4` as the memory-constrained second recommended profile; `16B` salt, `32B` tag; `5.3` vector | Parameter recommendation is not a Protos JVM/Native benchmark |
| [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) | Prefer Argon2id, distinguish scrypt/PBKDF2 and defense-in-depth pepper; min 19 MiB/2/1 | Minimum not equivalent to published 64MiB profile |
| [NIST SP 800-63B-4](https://pages.nist.gov/800-63-4/sp800-63b/authenticators/) | Unicode NFC before hashing; account-level breach blocklist and verification policy separate | Account requirements do not belong inside a generic KDF module |
| [libsodium password hashing](https://doc.libsodium.org/password_hashing/default_phf) | Small `pwhash_str`/verify/needs-rehash API and self-describing stored record | C library and allocator not implicitly available on Native Image |
| [Python argon2-cffi](https://argon2-cffi.readthedocs.io/en/stable/api.html) | High-level hash/verify/`check_needs_rehash` and parameter-bound policy | Python/native backend, String handling and concurrency are not Protos semantics |
| [RustCrypto password-hash](https://docs.rs/password-hash/latest/password_hash/) and [argon2](https://docs.rs/argon2/latest/argon2/) | Typed PHC encoding, explicit salt/randomness acquisition; algorithm independent | Cannot import Rust traits/key handling or ambient random API into Protos |
| [Go x/crypto/argon2](https://pkg.go.dev/golang.org/x/crypto/argon2) | Pure direct Argon2id derivation over bytes and explicit salt/params | Low-level KDF alone is not a complete verifier/record API |
| [Java JCA PBKDF2](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/javax/crypto/SecretKeyFactory.html) | Mature host crypto infrastructure for PBKDF2; possible regulated future profile | Not memory-hard and Java char/password conversion requires byte-exact audit |
| [Bouncy Castle Argon2BytesGenerator](https://github.com/bcgit/bc-java/blob/main/core/src/main/java/org/bouncycastle/crypto/generators/Argon2BytesGenerator.java) and [Argon2Parameters](https://github.com/bcgit/bc-java/blob/main/core/src/main/java/org/bouncycastle/crypto/params/Argon2Parameters.java) | Direct, pure-Java low-level Argon2id with input byte[] and v13 parameters; realistic first backend | Trust third-party code/dependency, memory bounds, zeroization and Native testing |
| [Spring Security DelegatingPasswordEncoder](https://docs.spring.io/spring-security/reference/features/authentication/password-storage.html) | Algorithm identifiers and transparent migration | Whole encoder registry is unnecessarily broad today |
| [GraalVM Native Image JCA services](https://www.graalvm.org/jdk25/reference-manual/native-image/dynamic-features/JCASecurityServices/) | Reflection/reachability and host security-provider initialization differ from JVM | Native cannot be assumed correct from successful Java compile |
| [TruffleRuby](https://www.graalvm.org/jdk25/reference-manual/ruby/), [GraalPy](https://www.graalvm.org/jdk25/reference-manual/python/) and [GraalJS](https://www.graalvm.org/jdk25/reference-manual/js/) | Guest language runtime is not automatically a safe cryptographic primitive; narrow host/native code can be justified for memory-hard algorithms | Do not infer unrestricted foreign/interop authority from Truffle compatibility |

No evidence here establishes a complete platform-wide survey of every native provider or a measured comparison of each Truffle runtime; production backend choice remains subject to explicit I160 tests. The survey demonstrates the substantive option classes and constraints.

## 3. Complete candidate set and explicit rejection

- **A — entirely Protos Argon2id/scrypt:** maximum source-language visibility, but a complex memory-hard implementation in guest loops pays high maintenance/safety/CPU cost and can monopolize an Actor unless parallel/offloaded; not justified when a narrow deterministic host primitive suffices.
- **B — minimal JCA PBKDF2-HMAC-SHA256:** operationally easy in Java and viable future FIPS-oriented profile, but lacks Argon2id's memory-hard property and would create concrete stored-record migration debt. Rejected for initial generation.
- **C — Protos-defined password protocol with narrow private Argon2id backend:** **owner-approved**; combines explicit entropy, stable records, predictable error/Future semantics and host off-turn memory-hard execution.
- **D — general security-provider registry/HSM/pepper/FIPS/key policy now:** too much public machinery; security complexity and new authority/transfer rules have no present requirement.
- **E — defer password library / each application embeds its own implementation:** violates reuse policy and leaves RealWorld's security problem unsolved.
- **scrypt** credible memory-hard fallback, but no distinct present advantage justifying public multi-algorithm generation. **bcrypt** accepted only in separately designed legacy migration. **SHA256/HMAC** disqualified for password storage.

## 4. GITHUB010 12-criterion scoring

Scores 1–5, larger is better. `H/M/L` denotes confidence, not observed test status. Each cell contains its rationale; scores do not compensate for fatal security or overengineering flags.

| Criterion | A Protos | B PBKDF2 | C Argon2id | D registry |
| --- | --- | --- | --- | --- |
| Correctness/invariants | 3/L complex crypto | 4/H mature | 5/M profile bounded | 4/M greater surface |
| Protos alignment | 4/M language-owned | 4/H narrow bridge | 5/M ordinary module | 2/H extra institution |
| Pay for what needed | 2/M large build | 5/H minimal | 4/M narrow dependency | 1/H premature platform |
| Incremental growth | 3/M rewrite likely | 3/M rehash migration | 5/M records versioned | 5/M configurable |
| Future options | 4/M own code | 3/M one KDF | 5/M private backend swap | 5/M many options |
| Scalability | 2/L guest-heavy | 4/M host CPU | 4/M capacity limited | 3/L complex routing |
| Conceptual simplicity | 3/M small API | 5/H small bridge | 4/M four methods | 1/H many rules |
| Portability/freedom | 4/L VM-independent source | 4/M JVM JCA | 4/L backend selectable | 3/L cross-platform |
| Runtime/resource cost | 1/L interpreter cost | 4/M no RAM-hard | 3/M 64MiB intentional | 2/L registry overhead |
| Failure/operability | 2/L crypto complexity | 4/M established | 4/M explicit admission | 3/M more failure paths |
| Reversibility/deferral | 3/M replace code | 3/M stored-record migration | 5/M versioned records | 4/M config debt |
| Evidence/feasibility | 2/L new design | 5/H mature JCA | 4/M published precedents | 3/M unproven integration |

**Non-compensating flags:** A risks ineffective memory-hard security and expensive guest CPU; B is underengineered for initial memory-hard protection; D is overengineered with speculative authority and configuration; E fails the actual security/reuse requirement. C incurs a deliberate third-party/private backend and CPU-memory budget and must be validated before publication; it cannot be selected solely because its numbers score highest.

**Smallest sufficient design:** one ordinary module, four operations, one current Argon2id profile, explicit salt authority, one private backend and bounded asynchronous work. No Core extension, new provider registry, new Source kind, global random generator, or general Secret object.

**Cost of omission:** without versioned metadata the first persisted hashes cannot be reliably migrated when normalization/parameters change; without admission limits attacker-chosen records or login bursts can exhaust host memory; without explicit Future boundary workers become semantically blocked; without explicit entropy authority each app invents a random substitute.

**Grow-as-needed:** new algorithms or FIPS come as new approved versioned profiles, not a v1 provider registry; HSM/pepper can introduce a separately approved credential-record envelope; new backend can preserve output bytes without changing module API. Adding these now would expand error, authority, transfer and testing surface without evidence of an actual user.

## 5. Adversarial / counterexample test matrix

1. A record says `m=2147483647`, `t=100000000` or forged `p`: reject before allocation or CPU work; do not call the provider even if it would reject later.
2. PHC has duplicate keys, alternate order, padded/noncanonical Base64, extra delimiters, unknown version, unrecognized algorithm or overflow: fail as invalid/unsupported, not `false`.
3. Two distinct passwords differing only by Unicode NFC representation: same input octets for the same salt; distinct normalized values must not collapse unintentionally. A U+0000 scalar is data, not terminator.
4. Input String and Bytes with shadowed operations: no arbitrary guest callbacks during backend admission. Mutating salt after starting `hashWithSalt` cannot alter the snapshot.
5. Cancellation wins before result publication: no record/tag leaks, no guest callback from stale worker or after Actor termination; release memory/queue permits. Cancellation after committed publication cannot revoke its result.
6. 16 concurrent 64MiB verifications imply about **1 GiB** Argon2 memory (plus overhead). Bounded resource budgets must reject/pause safely instead of running unbounded in a thread-per-request model.
7. Entropy grant absent, unavailable, revoked, from another Context or attempted Actor transfer: no mint by importing module, no fallback salt; RootActor broker transfers only completed salt Bytes.
8. A valid record with wrong password (first/middle/last different tag byte) -> false; malformed/unsupported record -> error; provider unavailable -> error; no content-dependent early-exit in comparison.
9. Process/Actor termination while host work computes: no use-after-termination guest entry, no retained Future registry leak; correct targeted settlement.
10. Reboot or upgrade: supported previous record still verifies; `needsRehash` detects obsolete recognized profile; application conditional write does not overwrite concurrent password rotation.
11. Native Image AOT missing provider/dependency or unexpected reflection registration: fail securely with a detectable error; never silently switch to another KDF.
12. External independent Argon2id implementations must agree on PHC core octets and tag when given *identical NFC→UTF8 bytes*, salt and params. The RFC 9106 `5.3 vector includes secret/additional inputs not exposed by this public API, so test the primitive independently or use additional independently verified empty-secret/empty-associated-data vectors for the public API.

## 6. Future-regret scenarios and escape

- **High load/many workers:** one 64MiB operation * N is unbounded without global host budget. Private admission pool and bounded queue preserve same ordinary Protos API; no public quota DSL now.
- **Tiny Native target / FIPS-only policy:** fixed Argon2id may be unsuitable. Do not mutate v1; require a further explicit profile/backend policy decision, adding PBKDF2 only if actually necessary.
- **Many Processes/Contexts/Nodes:** per-Actor Future/host completion and no shared guest mutable state. Cross-process provider scheduling may use host-private admission without a global Protos value.
- **Future credential pepper/HSM:** requires separate key custody, backend and rotation policies. Existing self-describing record envelope can be extended by new version. Not covered by D245 approval.
- **New Truffle/other runtime:** preserve portable algorithm/bytes/record contract and replace private implementation; never encode Java object identity or Native addresses in records.
- **Large input or DoS:** keep 64KiB post-NFC limit and pre-normalization guard; policy for excessive authentication input is reject/error, never truncate or silently prehash with SHA256.
- **Application stores passwords incorrectly:** no API can guarantee protected database storage, blocklists, multi-factor authentication, account rate limiting or secure logging; remain application concerns.

Strongest challenge to C: Argon2id at 64MiB and a Java backend may be unacceptable or unavailable in some actual JVM/Native configurations, whereas PBKDF2 via JCA is operationally easier. Escape: measure and validate Bouncy Castle 1.86/alternative native provider; if the approved profile is unsupportable, surface the new exact conflict for owner decision rather than downgrade or invent a compatibility mode.

## 7. GITHUB021 approval / invariant audit

| Preexisting applicable invariant | Preservation by C |
| --- | --- |
| D243-C: only explicitly host-granted RootActor `entropy`, no import-created authority, grant nontransferable; completed Bytes transferable | **PASS**: `hash` consumes caller-supplied entropy; `hashWithSalt` accepts brokered Bytes; no grant delegation |
| LIB006: pure `std:crypto/sha256` is unkeyed, not password storage; HMAC/entropy/host crypto excluded from LIB006 | **PASS**: new independent password library, no SHA256/HMAC fallback |
| D244-C: HMAC is deterministic synchronous Bytes→tag, does not mint randomness or create password hash policy | **PASS**: password library independent; HMAC I159 not an implicit prerequisite |
| Protos String identity: exact scalar sequence with no implicit NFC | **PASS**: NFC only for password protocol input; ordinary String untouched |
| Actor `24D/`24J`: no hidden scheduling/preemption; synchronous offload does not free the semantic turn | **PASS**: explicit Future and off-turn backend, targeted owning-domain publication |
| Core error taxonomy minimal; ordinary Standard Library modules private/local | **PASS**: no new Core error; PasswordError is ordinary module export; bridge private |
| User's actual D245-C packet: fixed 64MiB/3/4 profile, 16B salt, 32B result, versioned PHC+NFC, 64KiB limit, no insecure fallback | **PASS**: unchanged in ratification |
| User's actual approval text: `apruebo D245-C` | **PASS**: ratifies only C; no unrelated design, semantic downgrade or assumed tests |

**DECISION_INVARIANT_CONSISTENCY=PASS for approved scope.** No earlier D245-specific owner invariant was found that conflicts with the presented C. Exact record envelope spelling, versioned grammar, host/admission implementation and test harness are to be made precise by the conforming normative/implementation publication; an implementation must not use this freedom to alter any listed approved observable contract.

**Execution status:** GitHub issue and durable decision publication do **not** imply I160 code, package addition, native-image success, benchmark, provider security validation, focal JUnit PASS, `make test`, `make check` or `git diff --check` occurred. All remain to be evidenced by the human-executor workflow.
