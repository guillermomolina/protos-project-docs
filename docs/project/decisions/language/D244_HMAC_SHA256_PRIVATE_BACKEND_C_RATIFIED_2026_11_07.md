# D244 — Reusable HMAC-SHA256 with a narrow private backend (Candidate C)

**Status: RATIFIED DESIGN — exact owner-approved D244-C, 2026-11-07.** This is a durable **non-normative** decision record. It authorizes subsequent implementation; it does not itself change `spec/` or assert that HMAC is implemented.

- Design issue: [D244 #1070](https://github.com/guillermomolina/protos/issues/1070).
- Originating application: [I153 RealWorld #1068](https://github.com/guillermomolina/protos/issues/1068), but this design is a reusable Standard Library facility.
- Exact inspected product revision: [`08c27760eb3f85105277e5b1b1b159abbca67ebe`](https://github.com/guillermomolina/protos/commit/08c27760eb3f85105277e5b1b1b159abbca67ebe), 0.3.453-SNAPSHOT, Java 25, before any D244 implementation.
- Comparative research, adversarial matrix, GITHUB021 check and approval evidence: [D244 evidence](../../evidence/D244/D244_C_RESEARCH_OWNER_APPROVAL_AND_INVARIANT_AUDIT_2026_11_07.md).
- Owner approval provenance: active conversation 2026-11-07, immediately after the complete A/B/C/D decision packet, exact API, key/message limits, native/JCA boundary and explicit question. The owner replied: **“apruebo D244-C”** and authorized Issue/evidence coordination and repository-record publication. This is explicit approval of the presented C contract, not of any excluded D key-handle or future algorithm registry.
- Separate decisions: [D243-C secure entropy #1069](https://github.com/guillermomolina/protos/issues/1069), [D245 password hashing #1071](https://github.com/guillermomolina/protos/issues/1071), [D246 JWT #1072](https://github.com/guillermomolina/protos/issues/1072).

## 1. Selected public semantics

Provide an ordinary Actor-local Standard Library MODULE at **`std:crypto/hmac_sha256`**, with exactly these two initial exported operations:

```protos
HMAC: import("std:crypto/hmac_sha256")
tag: HMAC.digest(keyBytes, messageBytes)
valid: HMAC.verify(keyBytes, messageBytes, receivedTag)
```

- `digest(Bytes key, Bytes message) -> Bytes`: synchronous, deterministic, complete RFC 2104 HMAC-SHA256 over **exact octets**, returning a **fresh, open, independent standard Bytes object containing exactly 32 octets**. Every successful invocation returns a new identity even for equal inputs.
- `verify(Bytes key, Bytes message, Bytes tag) -> Boolean`: synchronous, deterministic, recomputes HMAC-SHA256 and verifies its **complete** 32-octet tag. Returns true exactly for authentic matching tags. A well-typed tag of any other length returns false; an equal-length wrong tag returns false. The comparison of equal-length tags must inspect all 32 octets without secret/content-dependent early exit. No truncated-tag acceptance.
- All three arguments must be actual **standard Bytes** values. No String conversion, implicit UTF-8, array coercion, delegation-only counterfeit, user-defined getter dispatch, foreign Bytes coercion, or ambient key lookup. Invalid type/domain/oversized operands synchronously signal a fresh ordinary **Error**, without releasing partial results.
- `key.size()` is bounded to **0..4096 octets**. A zero-length key computes the RFC-defined HMAC, but is **not** acceptable as an application authentication secret; applications need an adequately unpredictable key. A long key (above SHA-256's 64-byte block size) is hashed and normalized by the RFC algorithm.
- `message.size()` is bounded to **0..16,777,216 octets (16 MiB)**. This is a portable admission bound, not an HMAC algorithm limit or a memory/latency/real-time guarantee. The returned tag length is exactly 32.
- Receiver/argument evaluation follows ordinary Protos evaluation order. On successful ordinary dispatch, snapshot all validated octets before cryptographic work. No user callback, suspension point, reentrancy or implicit task/future is introduced by either operation. The inputs are never mutated, retained after completion, or aliased to the returned result.
- Provider unavailability, cryptographic initialization failure or unexpected resource/runtime failure are **errors**, not an unauthenticated `false` or insecure fallback. Do not reveal Java provider exception messages or secret material. Type and length errors likewise fail through `Error`; `verify` only returns false for well-typed, in-bounds nonmatching tags.
- No generic constant-time guarantee is made for all of HMAC computation, argument validation, size-dependent work, allocations, JIT/VM behavior or external application orchestration. The required guarantee is **no content-dependent early exit in complete equal-length tag comparison**.

## 2. Protos-first implementation boundary

Keep **public behavior, module identity, validation policy, readable documentation, algorithm selection and composition** in Protos. Place the unavoidable key-sensitive cryptographic calculation and comparison behind an exact-module, private, lazy, Actor-local runtime facility; initially implement it using **Java 25 `javax.crypto.Mac.getInstance("HmacSHA256")`**, `SecretKeySpec` and a content-independent 32-octet comparison (`MessageDigest.isEqual` when appropriate).

The facility is injected only for `std:crypto/hmac_sha256` as a private initializer binding by the established `ProtosCoreBootstrap.lazyStandardModuleMembers` mechanism. The module lexically captures the facility and removes its private initializer slot, as `std:tls/client` does. Neither the public module object nor the Prelude, `Process`, imported consumers, the host API or any other standard module obtains this capability by ordinary import. No public `Mac`, `SecretKey`, `KeyObject`, Java interop target, extra Core value family, provider registry, permission or authority-bearing bootstrap slot is added.

The bridge must recognize actual `ProtosBytesValue` and snapshot its **real octets** directly through `octetSnapshot()`, not by invoking guest-shadowable `size`/`at` messages. No mutable shared or cached `Mac` object; create one per invocation. Handle JCA's rejection of an empty `SecretKeySpec` key by using the RFC-equivalent one-zero-octet normalized key as GraalPy does, **only** for the empty key case. All working buffers under direct backend control should be wiped where possible and released in `finally`. This is memory exposure reduction, **not** a promise of zeroization, deterministic erasure of JVM/provider copies, sealed memory, HSM custody, or confidentiality against a compromised process.

No input key or provider MAC context can cross an Actor, Context, Process or isolated-P boundary through hidden runtime state. Ordinary completed Bytes follow existing Actor/P value transfer. Applications choose where to hold/copy/share their own key Bytes; D243-C entropy grants do **not** transfer automatically. Imported HMAC is authority-free: it receives the caller's bytes and draws **no entropy**, time, network, filesystem or Process permissions.

## 3. Separation from existing and adjacent work

- Preserve pure, deterministic unkeyed `std:crypto/sha256` from LIB006 unchanged, including its explicit exclusion of secret-bearing cryptographic guarantees. Do **not** reimplement HMAC as `SHA256(key || message)`.
- Respect [D243-C](./D243_SECURE_ENTROPY_EXPLICIT_CAPABILITY_C_RATIFIED.md): key generation/salt acquisition requires the separate, explicitly provisioned `entropy` capability. HMAC does not mint keys.
- D245 owns password hashing/salt/credential policies and **must not** substitute plain SHA-256 or HMAC for a password hashing scheme.
- D246 owns JWS/JWT serialization, header/claim validation, key selection and token replay/expiry policies. D244 signs **bytes**, not JSON, HTTP headers, claims or tokens.
- No automatic domain separation, key rotation, prevention of message replay, authenticated encryption, MAC truncation, streaming, cloned incremental states, alternative MAC algorithms, opaque non-exportable key handles, HSM, system-wide crypto-provider registry or exposed `timingSafeEqual` primitive is approved.

## 4. Portability, resource and failure guarantees

The first implementation is a synchronous local CPU operation with an admission bound, **not** a waiting I/O or Future-producing operation. It must not silently block on external permission/entropy acquisition or create Task work. JCA provider discovery/construction may fail, in which case the operation signals Error. No insecure pure-Protos fallback is permitted.

The same Protos-visible contract must be verified on JVM and an **actually rebuilt installed Native Image**. JCA provider reachability in Native Image is a build-time concern; enable only necessary `Mac` services if the existing image build does not already preserve them. Do not add external dependencies or all-provider/all-security-service registration speculatively.

Secret-dependent comparisons must be reviewed at the code level; benchmark noise or one statistical timing experiment does not prove constant-time operation. No cross-Actor sharing, state retention after completion, provider-algorithm switching, or ambient capability increase is permitted.

## 5. Explicit deferrals and escape paths

- If messages above 16 MiB become necessary, add an **incremental/streaming** HMAC protocol with a separately approved lifetime, cancellation/close and state-ownership contract. Do not weaken today's bounded one-shot behavior silently.
- If applications require non-exportable keys, add a **distinct explicitly designed key-handle API** with custody/authority/transfer/revocation contracts. Existing `Bytes` API remains stable; no fake security guarantee is retroactively claimed.
- If the runtime moves away from JCA, substitute an implementation behind this private boundary after interoperability and security tests. Public Protos API and accepted RFC byte mapping remain unchanged.
- If any of the above introduces newly observable semantics, route it through an independent design decision; do not hide it inside an I-slice.

## 6. Consequences and implementation authorization

This approval authorizes a subsequent **I-family implementation work item** for one integrated Protos-first slice: normative contract publication in `spec/` with changelog, private Java JCA bridge, private module injection, ordinary Protos module, complete conformance/Java/Native tests and Test Tool corpus registration, developer-facing documentation, and exact implementation version/changelog after human validation.

The owner reported `git diff --check`, `make test` and `make check` passing locally **before this HMAC implementation**. These are a user-reported baseline, not evidence that the future module, timing contract or Native Image passes.

**D244's durable design record is not normative:** implementation must publish new/updated canonical `spec/` ownership and obey full target validation before claiming its behavior is standardized and implemented.
