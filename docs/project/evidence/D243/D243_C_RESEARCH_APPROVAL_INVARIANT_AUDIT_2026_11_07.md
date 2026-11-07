# D243 — comparative evidence, C approval, GITHUB021 audit and falsification plan

**2026-11-07; decision evidence, not product implementation/test evidence.**

- [Decision D243/#1069](https://github.com/guillermomolina/protos/issues/1069); [implementation I157/#1090](https://github.com/guillermomolina/protos/issues/1090); [ratified non-normative contract](../../decisions/language/D243_SECURE_ENTROPY_EXPLICIT_CAPABILITY_C_RATIFIED.md).
- **PROTOS_REVISION=08c27760eb3f85105277e5b1b1b159abbca67ebe**, inspected source on `guillermomolina/protos` (Java 25, version 0.3.453-SNAPSHOT). No I157 sources, tests or native image were produced by D243 research.
- **OWNER_APPROVAL=EXPLICIT_C**: 2026-11-07 active conversation following detailed D243 A–G research, assistant explicitly recommended **C**, explained C against D; owner replied **«ok apruebo tu elección de C»** and instructed issue and evidence publication. Do not attribute this to a GitHub-authored owner comment. Approval is only C, not D244–D246, broad new permissions, transferable capabilities or HSM/provider extensions.

## 1. Exact Protos source and normative authority observations

| Inspected source at PROTOS_REVISION | Relevant fact and constraint |
| --- | --- |
| `AGENTS.md`, `AGENTS.work/{DESIGN,COORDINATION,REFERENCE}.md`, `spec/AGENTS.md`, `protos/AGENTS.md` | Owner approval required for semantics; GITHUB010 comparative breadth, GITHUB021 preservation, DOC002 role-first durable records; no authority from implementation convenience |
| `spec/io/PROCESS_IO.md` §§22,26,28 | Optional root bootstrap-local resource slots; `process` is not ambient permission; non-transferable live resources never auto-proxy; Process termination revokes grants |
| `spec/io/IO_CORE.md` §§3,4,8 | Future-returning potentially waiting I/O; `InvalidIOArgument`, `IOLifecycleError`, `IOError`; commitment/cancellation/close and idempotent fresh Future followers |
| `spec/concurrency/FUTURES_AND_TASKS.md` | Ordinary Future state and cancellation, no hidden Actor turn suspension; `.value()` can suspend only where already authorized |
| `spec/concurrency/ACTORS.md` §§8,16,24L | Ordinary Actor graph copy, `NonTransferableValue` for live nontransferable grants; ActorRef ordinary value; no auto proxy |
| `spec/concurrency/PARALLEL_EXECUTION.md` §71 | `NonParallelValue` when capability graph lacks a P contract; no ambient P effects |
| `spec/semantics/{MODULES,VALUES_AND_COLLECTIONS,ERRORS}.md` | Imports do not mint host authority; Bytes mutable and fresh result identity; portable Error categories |
| `spec/io/{FILESYSTEM,NETWORK,TLS,HTTP}.md` | Capability separation; TLS/HTTP do not grant randomness; HTTP Host Worker `initialization` uses ordinary Actor transfer |
| `ProtosStandaloneProcessBootstrap.java`, `ProtosProcessRuntime.java`, `ProtosActorBootstrap.java` | Explicit root grant plumbing, Process owner, bootstrap-local slots and termination cutover already present |
| `ProtosResolverCapabilityValue.java`, `ProtosResolverLookupFlow.java`, `ProtosHostResolverBackend.java` | Direct reusable implementation pattern: per-grant host backend, non-Task I/O operation, bounded lazy worker executor, Actor-owned Future result publication and host-side retirement |
| `ProtosActorValueTransfer.java`, `ProtosParallelRuntime.java`, `ProtosBytesValue.java` | Nontransferable grants already distinguished from transferable fresh Bytes; host byte array must be projected to canonical octet Integers |
| `ProtosCoreBootstrap.java`, `ProtosPrelude.java`, `protos/lib/core/network/Resolver.protos` | Source-backed protocol and private lexical bridge; Core prototype alone is authority-free; no public provider registry needed |
| `ProtosCli.java`, `ProtosStandaloneHostedExecution.java`, `ProtosPolyglotRuntimeHost.java`, `ProtosEmbeddedProcess.java` | Standalone Network opt-in is separate from embedding Env permissions; I157 needs independent entropy opt-in; do not infer from allowIO/allowNativeAccess |
| `protos/lib/crypto/sha256/module.protos`, `docs/design/STANDARD_LIBRARY_IDEAS.md` | Digest is deterministic and not host entropy; no existing secure generator |

## 2. Comparable systems and actual mechanisms

| System / authoritative reference | Evidence and Protos implication |
| --- | --- |
| [GraalPy `PosixModuleBuiltins.URandomNode`](https://github.com/oracle/graalpython/blob/main/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/modules/PosixModuleBuiltins.java) | `os.urandom` fills `byte[]` using `PythonContext.getSecureRandom().nextBytes` behind `@TruffleBoundary`; reuse host CSPRNG, but **not** Python ambient import authority or blocking Actor semantics |
| [GraalJS `CryptoBuiltins`](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/builtins/CryptoBuiltins.java) | Uses JCA `SecureRandom`; tries native nonblocking algorithm on supporting systems and falls back to secure provider, not to `Math.random`; authority differs from Protos |
| [GraalJS/Node crypto random](https://github.com/oracle/graaljs/blob/master/graal-nodejs/lib/internal/crypto/random.js) | Sync `RandomBytesJob` vs asynchronous crypto job; host work must stay off guest Actor turn for possibly blocking operation |
| [TruffleRuby Java embedding](https://github.com/truffleruby/truffleruby/blob/master/doc/user/polyglot.md) | Polyglot Context controls host permissions; no evidence for special Truffle guest-wide entropy grant; no semantics imported from Ruby |
| [OpenJDK `SecureRandom`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/security/SecureRandom.html) | Cryptographic `nextBytes`; provider selection and initial seeding are host/JCA matters; no fixed seed, no `getInstanceStrong` blind blocking on an Actor turn |
| [GraalVM Native Image security](https://www.graalvm.org/jdk25.5/security-guide/native-image/) and [JCA registration](https://www.graalvm.org/jdk25/reference-manual/native-image/dynamic-features/JCASecurityServices/) | Build-time class initialization can freeze random seeds; providers may require reflection registration and run-time initialization; actual image must be built/tested |
| [Node `crypto.randomBytes`](https://nodejs.org/api/crypto.html#cryptorandombytessize-callback) | Async worker execution and request buffering; illustrates scheduling and provider-delay hazards |
| [Go `crypto/rand`](https://pkg.go.dev/crypto/rand) | OS-backed secure generator can be a small library surface, without user-visible provider registry |
| [Rust `getrandom`](https://docs.rs/getrandom/latest/getrandom/) | Fallible host randomness and target-dependent backends; do not substitute deterministic fallback |
| [Erlang `crypto:strong_rand_bytes`](https://www.erlang.org/doc/apps/crypto/crypto.html#strong_rand_bytes/1) | Secure PRNG can fail, including low entropy; distinguish secure generation from raw entropy-bit consumption |
| [WebCrypto `getRandomValues`](https://developer.mozilla.org/en-US/docs/Web/API/Crypto/getRandomValues) | 65536-byte per-request bound and explicit type constraints are sensible as a portable abuse bound; does **not** imply deterministic uniqueness or entropy per output byte |
| [Linux `getrandom(2)`](https://man7.org/linux/man-pages/man2/getrandom.2.html) | Early-boot blocking, partial reads/EINTR for larger requests; host adapter must return full bytes or fail, never leak partial buffer |
| WASI capability model | External authority provisioned by host handle, not by source import; matches Protos independent grant principle |

No Oracle Graal/Truffle implementation is adopted wholesale: GraalPy/GraalJS/TruffleRuby differ from Protos in ambient API, Context grants, Actor model and cancellation. Pkl/Truffle interop and LLVM/Sulong do not supply a more directly applicable Protos Actor-safe entropy protocol; their lack of an analogous solution is not positive design evidence.

## 3. Complete candidate decision

- **A ambient global** — rejects no-ambient-authority; makes imported code privileged.
- **B Process accessor** — host authority conflated with the existing Process authority; future permission separation is harder.
- **C explicit bootstrap-local nontransferable `entropy`** — **selected**: least-authority, small semantic surface, secure provider hidden, ActorRef broker is ordinary composition.
- **D new delegable secure generator/attenuated capability** — credible for high-throughput Workers, but costs transferable authority/close/revocation rules; deferred pending an observed scaling need.
- **E public provider registry** — unnecessary policy and compatibility surface for the present use.
- **F pure Protos cryptographic PRNG without seed grant** — cannot securely bootstrap its seed, although deterministic generators may be separate future libraries.
- **G defer** — no secure source for required salts, sessions or keys, forcing app-private security workarounds.

### GITHUB010 comparative scoring

1=poor, 5=strong. Each value is qualitative with confidence **H/M/L**; row explanations state the chief trade-off and confidence rationale. Column candidates A/B/C/D/E/F/G.

| Criterion | A | B | **C** | D | E | F | G | Justification and confidence |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | --- |
| Correctness/invariants | 1H | 3M | **5M** | 5M | 4L | 1H | 1H | A leaks authority; C preserves it; D can too but must prove transfer; F lacks seed |
| Protos-first composition | 1H | 3M | **5M** | 4M | 2M | 4M | 1H | C uses ordinary resource/ActorRef; E adds new provider institution |
| Present-need sufficiency | 2H | 3M | **5M** | 3M | 2M | 1H | 1H | C solves secure bytes without forcing speculative delegation |
| Incremental growth | 2M | 3M | **5M** | 4M | 4M | 2M | 2H | C can acquire D later through a separate design without changing bytes result |
| Future-option resilience | 2M | 3M | **5M** | 5M | 5L | 2L | 2H | C keeps provider private and external delegate decision open |
| Scalability | 3L | 3M | **3M** | 5M | 4L | 2L | 1H | C broker adds hop and may bottleneck; D best if independently authorized Workers required |
| Conceptual simplicity | 4M | 3M | **5M** | 2M | 1H | 3M | 5H | G costs nothing but does not solve need; C pays only for one resource |
| Portability | 3M | 4M | **4M** | 3L | 2L | 2L | 5H | JCA/Native require image proof; F seed source remains unresolved |
| Runtime/resource cost | 4M | 4M | **4M** | 3M | 2M | 3L | 5H | Lazy C backend makes unused program cheap; Actor broker pays only if used |
| Failure/operability | 2M | 3M | **5M** | 4M | 3L | 2L | 1H | C maps invalid/lifecycle/backend failures to existing Future I/O taxonomy |
| Deferral/reversibility | 1H | 3M | **5M** | 4M | 2M | 2M | 2H | Ambient APIs are sticky; C allows future separate delegation |
| Evidence maturity/risk | 4M | 4M | **4M** | 3L | 2L | 2L | 4H | JCA and host grant/IO flow are proven patterns; new worker delegation is not |

**Pay for what you need:** authorized sessions pay for an explicit grant and lazy provider, not global workers or provider catalogs. **Grow as you need:** a new separately owner-approved delegable capability can coexist with C. **Strongest argument against C:** an ActorRef broker may serially bottleneck Worker token generation, with additional messages/copies; D is the escape if real throughput data demands it.

## 4. GITHUB021 owner invariant and delta audit

| Governing approved/normative constraint | C impact | Verdict |
| --- | --- | --- |
| Only explicit host permission confers sensitive resources; import and Process do not | `entropy` only in granted initial RootActor module, distinct CLI/embedding opt-in | PASS |
| Root bootstrap locals are not prelude globals; absent means slot absent | No global secure random facility or default `null` entropy | PASS |
| Actor message copy and non-transferable live resources never auto-proxy | Capability rejected; generated `Bytes` ordinary value copy | PASS |
| P has no host effects without a separate contract | Capability denied under `NonParallelValue`, Bytes may cross | PASS |
| IO_CORE Future/cancellation/close remains authoritative | Async bytes Future and exactly one commit at full-byte publication; no hidden bootstrap suspend | PASS |
| Host/provider trust is explicit and Native portability cannot be assumed from source | JVM + rebuilt Native acceptance required; never seed at image build time | PASS |
| Worker `initialization` is ordinary transfer, not Actor.spawn special grant | ActorRef broker, no spawn-only delegation | PASS |
| D219 avoid generic services without a second customer | No registry/policy DSL/platform service abstraction | PASS |
| D244–D246 independent design checkpoints | No MAC/password/JWT semantics implied by D243 | PASS |

**New semantic consequence compared with status quo:** an independently grantable and revocable secure random-byte I/O capability and independent CLI flag; this is exactly candidate C as presented and explicitly approved, not inferred from the research. No recorded conflicting owner-approved D243 invariant was found. **OWNER_APPROVAL_PROVENANCE=PASS; DECISION_INVARIANT_CONSISTENCY=PASS** for C. A future D delegable grant would require fresh approval and revision-bound evidence.

## 5. Adversarial acceptance / falsification list for I157

T01: no authorization => no bootstrap `entropy`; T02: import/Core object possession/Process/other grants never confer it; T03: count 0 => fresh open empty Bytes and no RNG call; T04: 1,16,32,65536 => exact octets; T05: negative/non-Integer/fractional/65537 => fresh failed Future InvalidIOArgument; T06: no mutable byte-buffer aliases; T07: no Java/guest deterministic fallback; T08: unavailable/provider errors => IOError without partial success; T09: precommit cancellation suppresses result; T10: close vs worker result race linearizable at commitment; T11: idempotent close with fresh follower Future identities; T12: after-close bytes => IOLifecycleError; T13: Process/host close retires only its own grant; T14: Actor send/request/reply/spawn carrying grant => NonTransferableValue; T15: P capture => NonParallelValue; T16: completed Bytes cross Actor normally, preserving value identity/copy rules; T17: RootActor-owned entropy broker with Worker initialization ActorRef works; T18: independent JVM/Native executable grant/denial paths, repeated Native launches with no frozen RNG seed; independent Context grants cannot be amplified.

These are proposed tests and falsifiers; **NONE WAS EXECUTED** in the research/ratification activity. Tests cannot prove cryptographic quality alone; they can falsify specific implementation errors. Actual host provider correctness and OS entropy remain external trust assumptions.

## 6. Boundary and coordination state

- **DESIGN_SELECTED=C; D243 is independent of I157 implementation completion**.
- `guillermomolina/protos-project-docs` stores non-normative rationale/evidence; `guillermomolina/protos/spec/` remains normative. I157 owns a later product spec changelog revision and production acceptance.
- `I157/#1090` must have its **native sub-issue relation to D243/#1069** verified per GITHUB006/GITHUB015; textual `Parent: #1069` is not enough. If the API client cannot create it, report the pending relation without claiming a fully reconciled hierarchy.
