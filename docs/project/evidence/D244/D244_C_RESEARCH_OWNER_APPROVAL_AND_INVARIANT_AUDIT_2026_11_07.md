# D244 — comparative evidence, exact C approval, GITHUB021 check and acceptance plan

**Recorded: 2026-11-07. Evidence for a design decision only, not implementation, a JVM security audit, a native-image run or a cryptographic certification.**

- [D244 #1070](https://github.com/guillermomolina/protos/issues/1070); [exact ratified C contract](../../decisions/language/D244_HMAC_SHA256_PRIVATE_BACKEND_C_RATIFIED_2026_11_07.md).
- **PROTOS_REVISION=08c27760eb3f85105277e5b1b1b159abbca67ebe**, Java 25, 0.3.453-SNAPSHOT.
- **OWNER_APPROVAL=EXPLICIT_D244_C**: immediately after the full D244 investigation and a direct question whether to approve `D244-C` **including** its `4096`-byte key and `16 MiB` message limits, the owner answered **“apruebo D244-C”** and requested issue coordination, durable evidence, and a future concrete file-editing Claude prompt. This is user approval, **not** a fabricated GitHub approval comment or the agent's unilateral choice.
- **Reported baseline checks (human)**: `git diff --check` clean; `make test` and `make check` PASS locally before the new D244 implementation. No assumption of a clean diff for future changes or proof that HMAC/Native Image passes.

## 1. Sources materially inspected and constraints

| Source | Concrete evidence |
| --- | --- |
| `AGENTS.md`, `AGENTS.work/DESIGN.md` | Explicit owner selection, GITHUB010 exhaustive comparison and twelve scores, GITHUB021 invariant preservation, no speculative feature work |
| `AGENTS.work/COORDINATION.md`, `AGENTS.work/REFERENCE.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/LIBRARY.md`, `protos/AGENTS.md`, `spec/AGENTS.md` | GitHub-native issue coordination; correct I identifier/family, role-first durable paths, Protos-first source, actual normative owner, Test Tool inclusion, tests-before-version-bump |
| `spec/semantics/VALUES_AND_COLLECTIONS.md` (Bytes), `spec/semantics/MODULES.md`, `spec/semantics/ERRORS.md` | Core Bytes identity is **mutable and not content-value identity**; `==` is not tag equality. Bytes production is fresh/open and module instances are Actor-local; ordinary Error is the portable failure path |
| `spec/concurrency/ACTORS.md` §16/§24L, `spec/concurrency/PARALLEL_EXECUTION.md` §71 | Actor/P transfer follows complete graph rules; no capability auto-proxy or implicit shared mutable state |
| `spec/io/IO_CORE.md`, `spec/io/PROCESS_IO.md`, `spec/concurrency/FUTURES_AND_TASKS.md` | HMAC pure deterministic computation should not silently acquire I/O, Process authority, entropy, Future, Task or suspension |
| `protos/lib/crypto/sha256/module.protos`, `protos/tests/library/crypto/sha256/{vectors-and-boundaries,ownership-and-domain}.protos` | Existing LIB006 SHA-256 is unkeyed, pure Protos, no constant-time/secret guarantee. Inputs remain unchanged and fresh result identity already tested |
| `src/main/java/com/guillermomolina/protos/runtime/io/ProtosBytesValue.java` | `octetSnapshot()` yields host array of **actual** Bytes contents; avoid semantic tampering through a custom `size`/`at` slot |
| `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosCoreBootstrap.java` | `lazyStandardModuleMembers` supplies exact-module private initializer facilities, with `std:tls/client` as established precedent |
| `src/main/java/com/guillermomolina/protos/execution/facilities/ProtosTlsBackendFacility.java`, `protos/lib/tls/client/module.protos` | JCA/backend facts can stay private; module can capture private facility lexically and remove private bootstrap slot |
| `protos/tools/test/RepositoryCorpusPlans.protos`, `protos/tools/test/RepositorySuite.protos`, `src/main/java/com/guillermomolina/protos/cli/ProtosTestCorpusRegistry.java` | New tests need explicit suite/corpus registration; merely creating .protos fixtures does not schedule them |
| `docs/design/STANDARD_LIBRARY_IDEAS.md`; `guillermomolina/protos-project-docs/docs/project/work/LIB006/LIB006_HASHING_DESIGN.md` | LIB006's deliberately narrow scope excluded HMAC, entropy, password hashing, zeroization and host-secret bridges |
| Ratified [D243-C](../../decisions/language/D243_SECURE_ENTROPY_EXPLICIT_CAPABILITY_C_RATIFIED.md) | Explicit root-local nontransferable entropy grant; HMAC must not mint entropy nor infer any permission from `process` or imports |
| D219 #973 and D220 #975 | Do not build a universal runtime cryptographic framework; existing TLS private-facility architecture is a precedent, not crypto-module authority |

## 2. Primary standards, real runtime implementations, third-party systems

| Prior art / reference | Observed architecture and lesson |
| --- | --- |
| [RFC 2104](https://www.rfc-editor.org/rfc/rfc2104), [RFC 4231](https://www.rfc-editor.org/rfc/rfc4231) | Standard 64-byte inner/outer pads, normalization of keys >64 bytes, full 32-byte HMAC-SHA256 and fixed published vectors. Bytes/encoding and truncation are application-policy boundaries |
| [OpenJDK Java 25 `Mac`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/javax/crypto/Mac.html), [`MessageDigest.isEqual`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/security/MessageDigest.html), [`SecretKeySpec`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/javax/crypto/spec/SecretKeySpec.html) | Provider-backed JCA HmacSHA256 and comparison independent of content; SecretKeySpec snapshots a copy and rejects empty key: a one-zero-octet equivalent is necessary to preserve RFC empty-key semantics |
| [GraalPy HashlibModuleBuiltins.java](https://github.com/oracle/graalpython/blob/main/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/modules/hashlib/HashlibModuleBuiltins.java), [DigestObject.java](https://github.com/oracle/graalpython/blob/main/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/modules/hashlib/DigestObject.java) | Actual `@TruffleBoundary` / `Mac.getInstance` / `SecretKeySpec` code and the explicit empty-key workaround; guest API is not a Java object exposure |
| [CPython `hmac`](https://docs.python.org/3/library/hmac.html) | Pure-standard algorithm contract with optimized backend and a separate `compare_digest`; a caller can still accidentally use `==` |
| [Go `crypto/hmac`](https://pkg.go.dev/crypto/hmac) | `New`/Write/Sum and `hmac.Equal`; incremental state adds lifecycle surface, avoid until needed |
| [RustCrypto HMAC](https://docs.rs/hmac/latest/hmac/) | `verify_slice` integrates tag comparison, reducing unsafe caller behavior; no automatic key storage policy |
| [Node crypto / GraalJS Node crypto.js](https://nodejs.org/api/crypto.html), [GraalJS implementation](https://github.com/oracle/graaljs/blob/master/graal-nodejs/lib/crypto.js) | `createHmac`, `timingSafeEqual` and optional key objects; comparison primitives do not make surrounding code timing-safe |
| [TruffleRuby OpenSSL HMAC](https://github.com/oracle/truffleruby/blob/master/src/main/c/openssl/ossl_hmac.c) | Ruby-facing API using OpenSSL; mature native provider model at cost of native-runtime dependency |
| [.NET FixedTimeEquals](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.cryptographicoperations.fixedtimeequals) | Constant-time relative to content for same-length arrays, but not to array length |
| [GraalVM Native Image JCA security services](https://www.graalvm.org/reference-manual/native-image/JCASecurityServices/) | Reachability/provider registration may affect whether HmacSHA256 is actually available in a rebuilt executable |

These include **pure composition**, **guest library + native/provider primitive**, and **opaque managed key/provider** approaches. Apple Pkl, Truffle SimpleLanguage, Sulong and Espresso were considered, but do not establish an independently useful public HMAC/key-custody contract beyond the JVM/Truffle/backend families already studied.

## 3. Compared candidates and decision

- **A: pure Protos HMAC via LIB006 SHA-256.** Strong portability, but many key-sensitive guest operations and no credible timing assurance; costly on large messages.
- **B: pure-Protos HMAC with native tag comparator.** Fixes one timing risk, not the secret-bearing HMAC computation/representation, and duplicates security mechanisms.
- **C: ordinary Protos `digest` and `verify`, exact lazy private JCA MAC+comparison bridge.** Chosen: small API, no new privileges, mature implementation, Protos-first semantics, replaceable host boundary.
- **D: opaque keys, revocable key handles, provider/HSM registry.** Valuable if non-exportable keys become a current requirement; premature identity, transfer, authority and lifecycle model today.
- **No change** cannot satisfy the current reusable authentic MAC need; exposing `javax.crypto.Mac` publicly is not a portable Protos library design.

Twelve GITHUB010 dimensions: each score **1 (weak) to 5 (strong)** with confidence **H=high, M=medium, L=low**. Justification appears in every entry; totals are not dispositive.

| Dimension | A pure Protos | B mixed comparison | C private JCA | D opaque keys |
| --- | --- | --- | --- | --- |
| 1 Correctness/invariants | 3M timing uncertain | 3M incomplete boundary | **5M** end-to-end delegated | 4M unsettled custody |
| 2 Protos alignment | 5H pure source | 4H composable source | **4H** ordinary module | 2H privileged entities |
| 3 Present proportionality | 4M reuses SHA | 3M dual mechanisms | **5M** two operations | 1H overengineered |
| 4 Incremental growth | 4M can extend | 3M two evolving paths | **5M** private replaceability | 3M state model locks in |
| 5 Future option resilience | 4M host-free | 4M mostly host-free | **4M** supports different backends | 4M key-provider future |
| 6 Scalability | 2M many guest rounds | 2M same hashing cost | **4M** provider optimization | 4L potential, unproven |
| 7 Conceptual simplicity | 4H one source layer | 2M split security | **5M** minimal contract | 1H many rules |
| 8 Portability | 5H language-based | 3M host comparator | **4M** portable API | 2M backend coupling |
| 9 Runtime/resource cost | 1M pure guest cost | 2M same rounds | **4M** per-call native | 3L provider dependent |
| 10 Failure/operability | 2M hard timing audit | 3M compare improved | **4M** fail closed | 2M complex lifetime |
| 11 Reversibility/deferral | 2M likely rewrite | 2M future migration | **4M** stable API | 2M upfront migration |
| 12 Evidence maturity | 3M algorithm proven | 3M partial precedent | **5M** mature Java/GraalPy | 4M mature elsewhere |

**Noncompensating gates:** A and B cannot acquire a security claim merely by mathematical correctness; D cannot compensate premature semantic surface with speculative future HSM scale. C would be **disqualified** if a coherent JVM+Native byte-exact contract, provider availability, non-early-exit verification or Actor-local facility containment cannot be demonstrated. Approval does not override these engineering acceptance gates.

**Strongest objection to C:** JVM/provider/JIT behavior does not prove universal constant-time computation or physical erasure. The exact approved security claim is narrower and honest: complete tag comparison without content-dependent early exit, private non-retaining per-call state, no key exposure by public facility, and no claimed zeroization. A real current non-exportable key requirement would reopen design D.

## 4. Counterexamples and falsification matrix

| Input/failure | Required observable result |
| --- | --- |
| RFC 4231 test cases 1–4, 6–7; empty key; zero length message; key >64 bytes | Exact interoperable 32-byte tag; empty key never silently rejected because JCA's SecretKeySpec cannot represent it |
| Binary octets `00`, `ff`, non-UTF8, embedded NUL | No encoding or String interpretation |
| Two matching digests / repeated call | Equal octets, distinct fresh open `Bytes` objects |
| Mutation of key/message/result after call | Does not change earlier digest or mutate other inputs/results; snapshots not retained |
| Standard Bytes with adversarial slots `size`, `at`, `each` | Direct real-octet host extraction, no attacker methods invoked |
| Tag wrong at first/middle/last octet | `false` with no early-exit comparison of equal-length tags |
| Tag size 0/1/16/31/33 | `false` when tag is standard Bytes; never accept truncated tags |
| Wrong key/message type or foreign/counterfeit Bytes | Synchronous ordinary `Error`; no partial output |
| 4096/4097 key; 16777216/16777217 message | Exact admission limits, no unsafe host truncation |
| Missing `Mac` provider, initialization exception, resource pressure | Synchronous `Error`, no insecure fallback, no provider diagnostics leaking secret bytes |
| Call without entropy/Process/Network permission | Works on supplied bytes; does not mint missing authority |
| Actor/P transfer, many Actors, simultaneous Contexts | Standard Bytes transfer; no hidden provider, key or mutable Mac leaks |
| Native Image rebuilt and launched | Same RFC vectors, error domains, module-private boundary and provider availability |
| Short-circuit/timing checks | Code review confirms full equal-length comparison; timing measurements alone are not cryptographic proof |

## 5. Future scenarios, anti-overengineering and migration

- **Pay for what you need:** No HMAC import, no crypto engine allocation. Import alone publishes a small ordinary module and private callable backend without building MAC state; each operation pays for one provider instance and transient byte snapshots.
- **Grow as you need:** A new streaming protocol can be additive and separately define ownership/suspension/cancellation. A distinct sealed HSM/non-exportable-key API can be additive; do not change `Bytes` into a magic key value.
- **Cost of deferral:** Scaling above 16 MiB requires new incremental stateful API and tests, **not** a breaking revision of one-shot results. HSM requires new authority/custody semantics and application migration for those users; no current evidence justifies paying that cost for all programs.
- **Multicore/Actors:** No process-shared Mac; per-call private instances. Application key Bytes can be explicitly copied through ordinary value transfer, or kept with a broker Actor if stronger custody policy is desired.
- **Distributed/multinode:** No implicit key distribution, key loading, rotation or replay policy. Protocol layers arrange these before invoking HMAC.
- **Alternative runtime:** A new private implementation may replace JCA while keeping byte-exact public semantics.
- **Future regret/escape:** Very large streaming workloads or non-exportable key custody would outgrow C. Introduce an additive protocol/opaque-key API instead of silently weakening C.

## 6. GITHUB021 approved-invariant / delta check

| Existing explicit invariant | C consistency |
| --- | --- |
| LIB006 SHA-256 is unkeyed, pure and without secret-security claim | **PASS** — separate module; existing digest unchanged |
| D243-C entropy is a separately provisioned, root-only nontransferable grant | **PASS** — HMAC uses only supplied Bytes |
| D219-B no speculative shared runtime-facility framework | **PASS** — narrow exact-module bridge |
| D220 TLS authority/host material kept within owning TLS scope | **PASS** — no changes to TLS grants/semantics |
| Core Bytes identity/mutability/open results | **PASS** — fresh open 32-byte digest, inputs remain independent |
| Actor/P isolation/module ownership | **PASS** — per-Actor lazy facility, per-call state, no hidden transfer |
| Owner-approved D244-C, 4096 and 16MiB bounds, whole-tag verify, error on internal failure | **PASS** — ratification record retains every approved observable consequence |

**DECISION_INVARIANT_CONSISTENCY=PASS** for the exact C contract; no owner-approved invariant has been silently reopened. Source language changes await the separately tracked implementation and normative publication. Later newly discovered semantic consequences require their own owner approval.

## 7. Publication/validation scope

- **Decision selected and durable research recorded**, not implemented.
- The user's stated passing local `git diff --check`, `make test`, `make check` is pre-implementation evidence only.
- No new Protos source, test, Java benchmark or Native Image was produced or executed in this decision-record activity.
- Required future acceptance: explicit Test Tool corpus/suite registration; RFC 4231 and negative Protos tests; Java focused integration; multi-Context/Actor isolation; built Native Image execution; human-run repository gates; source/test revision identified exactly.
- **PROTOS_REVISION** above is the actual inspected product baseline. **PROJECT_RECORD_REVISION** must be taken from the successfully published GitHub commit, not predicted in this document.
