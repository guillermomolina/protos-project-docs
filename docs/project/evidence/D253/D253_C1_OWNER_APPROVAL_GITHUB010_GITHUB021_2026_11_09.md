# D253-C1 — Exhaustive research, adversarial assessment and exact owner-approval evidence

**Evidence date:** 2026-11-09. **Status:** owner-approved design research; *not* an implementation, product normative publication or test result. This snapshot records the comparison and GITHUB010/GITHUB021 decision gate for [D253 #1101](https://github.com/guillermomolina/protos/issues/1101). The maintained ratification record is [D253-C1](../../decisions/language/D253_EXPLICIT_CORE_WALL_CLOCK_NANOSECONDS_C1_RATIFIED_2026_11_09.md).

- Independent implementation allocated as [I163 #1102](https://github.com/guillermomolina/protos/issues/1102), with `family:I`, `status:ready`, native D253 parent **not yet established**. Issue creation is not normative product publication or validation.
- Exact inspected `PROTOS_REVISION=9c92dbc0cfdfbfe5cb2751339ea27297c8c22bcd` (`0.3.458-SNAPSHOT`); [commit](https://github.com/guillermomolina/protos/commit/9c92dbc0cfdfbfe5cb2751339ea27297c8c22bcd).
- Exact inspected RealWorld revision `d242b73e6bd327739147bfc8ce85a789b4384533`; [commit](https://github.com/guillermomolina/protos-realworld-example-app/commit/d242b73e6bd327739147bfc8ce85a789b4384533).
- Owner approval in active conversation, **2026-11-09**: **“apruebo D253-C1”**, explicitly followed by the request to update `guillermomolina/protos` Issues, publish evidence in `guillermomolina/protos-project-docs`, and create an I-family implementation item if needed. The approved label is the **second, revised C1 packet**, not the earlier first-pass B1 proposal.
- **No new runtime test, JUnit, CLI smoke, Native Image, human-reported green gate, product spec commit or RealWorld authentication success is claimed.**

## 1. GITHUB010 research scope and normative current state

Repository governance materially inspected: `AGENTS.md`; `AGENTS.work/DESIGN.md` (GITHUB010, GITHUB021, approval and incremental-balance gates); `AGENTS.work/COORDINATION.md` (GITHUB006/009/015/020, formal allocation); `AGENTS.work/REFERENCE.md` (Protos design philosophy, DOC002); `AGENTS.work/IMPLEMENTATION.md`; `spec/AGENTS.md`; `src/AGENTS.md`; `protos/AGENTS.md`; and [Core bootstrap architecture](https://github.com/guillermomolina/protos-project-docs/blob/main/docs/project/architecture/CORE_BOOTSTRAP_ARCHITECTURE.md).

Normative product documents and material sections examined:
- `spec/auth/JWT.md`: D246-C synchronous `issue` and `verify` require **explicit** genuine `std:datetime/instant` `now`; no ambient time, entropy, key authority or HTTP session.
- `spec/io/PROCESS_IO.md`: initial RootActor private local capability slots, optional independent Entropy/Resolver/Network/Filesystem, direct/message-service equality and embedding.
- `spec/io/IO_CORE.md`: ordinary synchronous operations are permitted when no wait is required; Future-returning I/O is not universal, and established Error categories apply.
- `spec/io/ENTROPY.md`: authentic host-granted independent authority, private bridge, Actor/P nontransferability; **not** evidence that every other capability needs Entropy's Future/close lifecycle.
- `spec/semantics/MODULES.md`: independent Actor-local MODULE instances; Core bootstrap precedes ordinary `std:` import; private initial-module slots are not public MODULE exports.
- `spec/concurrency/ACTORS.md`, `FUTURES_AND_TASKS.md`, `PARALLEL_EXECUTION.md`: no implicit Actor capability inheritance or P effect authority; explicit Future suspension and Actor termination rules.
- `spec/semantics/VALUES_AND_COLLECTIONS.md`: exact Integer domain and closed value-identity families; identity is not implied by immutability; Actor transfer is determined by the domain.
- `spec/io/HTTP.md` §11: no portable deadlines/timeouts/Future-race contract currently makes direct hostile Internet exposure safe.
- `protos/lib/datetime/instant/module.protos`, `docs/guide/library/datetime.md`: exact nanosecond `Instant` with no clock; `protos/lib/logging/logger/module.protos` accepts caller-provided zero-argument Instant time source.

Implementation and tests materially examined: `src/main/java/com/guillermomolina/protos/cli/ProtosCli.java`; `execution/facilities/ProtosEntropyHostFacility.java`; `runtime/io/entropy/ProtosEntropyCapabilityValue.java`; `execution/host/resources/ProtosEmbeddedProcess.java`; `execution/host/embedding/ProtosStandaloneHostedExecution.java`; `runtime/ProtosPrelude.java`; `runtime/ProtosActorValueTransfer.java`; `protos/tests/library/auth/jwt/identity-and-actor-boundaries.protos`; `protos/lib/core/README.md`; `protos/lib/core/prelude.protos`; `protos/lib/logging/logger/module.protos`; RealWorld `src/main.protos`, `src/app/Api.protos`, `src/app/ContentStore.protos` and `README.md`. The [implementation-blocker ledger](https://github.com/guillermomolina/protos-project-docs/blob/main/docs/project/registries/IMPLEMENTATION_BLOCKERS.md) has no dedicated D253 blocker entry as of inspection; D253/I153 track the dependency.

**Observed gap:** no public Clock authority or current-time Core protocol at the exact product revision. The RootActor cannot use authenticated temporal JWT flows without an explicit reusable host clock; `std:logging` has the same optional time-source need. A fabricated current time, HTTP Date header, request-supplied clock or app-only Java/FFI bridge would fail the intended security/provisioning boundary.

## 2. Primary cross-system evidence and semantic contribution

| System / family | Primary reference | Model / lesson |
| --- | --- | --- |
| Java 25 / host substrate | [InstantSource](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/time/InstantSource.html), [Clock](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/time/Clock.html) | Explicit injectable time source and test substitution; no general trust/precision guarantee |
| Python / dynamic | [time](https://docs.python.org/3/library/time.html), [datetime](https://docs.python.org/3/library/datetime.html) | Separate `time_ns` from `monotonic_ns`; interval origin is not Unix epoch |
| Rust / systems | [SystemTime](https://doc.rust-lang.org/std/time/struct.SystemTime.html), [Instant](https://doc.rust-lang.org/std/time/struct.Instant.html) | Civil time may jump; monotonic counter uses opaque origin |
| Go / systems | [time.Time](https://pkg.go.dev/time#Time) | One object may contain dual readings, but comparison and serialization semantics become complicated |
| Erlang / BEAM | [System time](https://www.erlang.org/doc/apps/erts/time_correction.html), [erlang:monotonic_time](https://www.erlang.org/doc/apps/erts/erlang.html#monotonic_time/0) | Time warp, correction, suspension and actor-oriented concurrency do not make wall clocks monotonic |
| WASI / component capabilities | [WASI clocks](https://github.com/WebAssembly/wasi-clocks) | Host interfaces separate system/wall clock from monotonic time; implementation need not be OS-specific |
| OCaml Eio / explicit capabilities | [Eio time reference](https://ocaml-multicore.github.io/eio/eio/Eio/Time/index.html) | Pass a clock explicitly rather than acquiring it from every consumer |
| GraalPy / Truffle | [TimeModuleBuiltins.java](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/modules/TimeModuleBuiltins.java) | `time_ns` uses `Instant.now()`, `monotonic` uses `System.nanoTime()`; `long` nanosecond conversion is not a Protos semantic limit |
| TruffleRuby / Truffle | [process.rb](https://github.com/truffleruby/truffleruby/blob/master/src/main/ruby/truffleruby/core/process.rb), [ProcessNodes.java](https://github.com/truffleruby/truffleruby/blob/master/src/main/java/org/truffleruby/core/ProcessNodes.java) | CLOCK_REALTIME and CLOCK_MONOTONIC select distinct host primitives; native details are not public Protos semantics |
| GraalJS / Truffle | [JSRealm.java](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/runtime/JSRealm.java), [DateFunctionBuiltins.java](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/builtins/DateFunctionBuiltins.java) | `Date.now` routes through a realm wall clock; monotonic and wall clocks are distinct; JS numeric/date bounds should not leak |
| GraalVM Native Image / host runtime | [PosixPlatformTimeUtils.java](https://github.com/oracle/graal/blob/master/substratevm/src/com.oracle.svm.core.posix/src/com/oracle/svm/core/posix/PosixPlatformTimeUtils.java) | Java Time reaches real-time OS primitives without a guest-visible Linux FFM dependency |

Other Truffle architectures considered: Espresso adheres to Java time APIs and is not an independent Protos capability-policy source; Sulong exposes LLVM/OS time primitives but no reusable source-backed Protos authority; SimpleLanguage is an instructional interpreter and has no production Clock contract; Apple Pkl is predominantly a configuration/evaluation environment and has no comparably relevant transferable Clock capability. Thus none warrants adding an ambient clock or OS-specific semantics.

**Source-level falsification:** GraalPy and TruffleRuby examples perform `epochSeconds * 1_000_000_000L + nanos` in a Java `long`. This is valid only in its bounded domain; a future epoch past the signed-64 nanosecond interval is a concrete counterexample. Protos's BigInteger-capable Integer contract must not inherit that accidental host bound. Negative epoch conversion also must use the correct floor/normalized second and fraction convention.

## 3. Meaningful candidates

| ID | Architecture | Decision |
| --- | --- | --- |
| A | Ambient `Instant.now()` / global `Clock.now()` | Disqualified: authority leaks through imports and every module |
| B | Explicit Core `clock.now() -> std:datetime/instant` | Credible, ergonomic; puts an actor-local Standard Library value into the Core host-protocol boundary |
| **C** | **Explicit Core `clock.epochNanoseconds() -> Integer`** | **Owner-approved C1**: minimal synchronous authority and composable exact Core data |
| D | C plus public `std:datetime/clock.now(clock) -> Instant` facade | Credible hybrid; no need to add facade to satisfy JWT today |
| E | C with direct Actor-delegable/proxy grants from inception | Credible scale alternative, speculative authority/lifetime/identity cost today |
| F | Explicit `clock.epochNanoseconds() -> Future<Integer>` | Credible remote/blocking-source alternative, unnecessary Future on ordinary local reading |
| G | Complete clock/timer/deadline/timeout service now | Disqualified for unjustified public concurrency and monotonic model complexity |
| H | Defer and keep no standard time authority | Rejected: leaves independently reported RealWorld/JWT blocker, encourages private workarounds |

## 4. GITHUB010 twelve-axis scorecard

Scores 1..5 (higher is better); H/M/L denote HIGH/MEDIUM/LOW confidence. Each cell includes its reason; hard authority and unnecessary-surface flags are noncompensating. A, G, H were eliminated above for specific hard/practical reasons and are not surviving candidates.

| Required dimension | B | **C** | D | E | F |
| --- | --- | --- | --- | --- | --- |
| Correctness/invariants | 4/M std/Core coupling | **5/H exact Core domain** | 5/H preserves boundary | 4/M delegation risk | 5/H explicit authority |
| Protos alignment | 4/M layered coupling | **5/H smallest mechanism** | 4/H extra API now | 3/M stronger institution | 3/H mandatory Future |
| Present proportionality | 4/H useful | **5/H one narrow read** | 4/H convenience extra | 2/H unnecessary transfer | 2/H unnecessary async |
| Incremental growth | 3/M coupled type | **5/H facade additive** | 5/H facade present | 4/M can add later | 4/M remote future |
| Future-option resilience | 3/M module coupling | **5/H separate clocks** | 5/H separate clocks | 4/M ties authority graph | 4/M locks return shape |
| Scalability | 4/M actor-local | **4/M broker option** | 4/M same as C | 5/M direct Workers | 3/M Future coordination |
| Conceptual simplicity | 5/H one result | **5/H exact one primitive** | 3/H two surfaces | 2/H proxies/revocation | 3/H Future states |
| Portability | 3/M std identity | **5/H Integer/Core** | 5/H source wrapper | 4/M rematerialization | 3/M backend scheduling |
| Runtime/resource cost | 4/M imports | **5/M one local read** | 4/M wrapper allocation | 3/M cross-Actor proxy | 2/M Future allocation |
| Failure/operability | 4/M explicit | **4/M simple failures** | 4/M same backend | 3/M revoke races | 3/M cancellation |
| Deferral/reversibility | 3/M type coupling | **5/H facade later** | 5/H compatibility | 3/M long-term delegation | 3/M async compatibility |
| Evidence maturity/risk | 4/M precedent | **4/M proven substrate** | 4/M simple composition | 3/M more runtime risk | 4/M existing Future |

**Noncompensating gates:** A violates the explicit-authority principle; G overengineers unneeded timers, monotonic origins, cancellation and cross-platform semantics. E's direct transfer might eliminate broker hops at large scale but must pay for authorization, identity, transport, ownership, closure, cross-Process placement and race semantics now. H underengineers an observed blocker. C's main objection is exposing units to consumers instead of returning `Instant`; the operation's explicit name and the existing constructor preserve clarity with bounded cost.

## 5. GITHUB010 mandatory incremental-design and future stress questions

- **Smallest sufficient today?** A host-provisioned Core Clock with one synchronous exact Integer wall-clock observation, independent bootstrap grant, no implicit capabilities, ordinary Actor/P prohibition, and independent JVM/Native host implementations. Additional feature requests have no demonstrated acceptance dependency.
- **Who pays for unused capability?** Under C, no app without authorization performs time reads or allocates Future/clock workers; a granted-but-unused Clock can retain minimal host state. D adds public facade and imports to the optional consumer. E creates cross-Actor grant semantics even when only the RootActor calls Clock. F creates Futures per read.
- **Grow incrementally?** Add `std:datetime/clock` facade later without changing the Core `Integer` operation; a separate monotonic clock can return an opaque monotonic domain; remote providers can supply a separately named Future operation if needed; explicit Actor delegation can be a later decision.
- **Concrete deferral cost?** Deferring the facade requires only adding a module and optionally migrating application call sites. Deferring Actor grant transfer requires an additional transfer/ownership protocol and runtime support; it does not force modification of C1's authorized sampling or JWT. Deferring Clock entirely forces RealWorld/app-private time logic and a future migration, which is not acceptable.
- **Future regret and escape path?** A large service needing every Worker to sample time directly may suffer RootActor broker overhead. Escape: separately approve a capability transfer/attenuation contract with destination-local proxy and correct revocation; retain C1's exact `Integer` result. A distributed app needing certified time needs an independently trusted/time-attestation service; do not rename a civil clock as a monotonic guarantee.
- **Scale and failure stress:** 1 Actor and 1000 Actors remain independent except when voluntarily brokering; simultaneous Contexts have separate grants; no globally synchronized Protos clock object; Process shutdown revokes authority; host rollback, VM suspend, NTP changes, clock failure, stale data, UTC mistakes and multi-node drift remain visible as specified. No scheduled polling, implicit reentry or an always-running clock worker.
- **Migration beyond Truffle?** A non-Java implementation can provide the same seconds/fraction host primitive and exact Integer semantics; no Java object, OS syscall, Truffle node class or Clock identity across contexts is public.

## 6. Negative and counterexample acceptance matrix — proposed, NOT EXECUTED

| Test | Expected invariant |
| --- | --- |
| RootActor with `--allow-clock`; without grant; independent Contexts | Slot present only under independent grant; no privilege inference |
| `--allow-clock` plus/no Network, Entropy or Filesystem | Grants orthogonal; no ambient permission path |
| Import `std:datetime/instant`, JWT, logging or Clock prototype | No new authority granted through import |
| Obtain an ordinary child/lookalike of `Clock` | Cannot use the private host effect |
| Valid 0-argument `epochNanoseconds()` | Immediate genuine semantic Integer; no Future/Task |
| Surplus/missing args, invalid receiver, host read failure | Ordinary controlled Error, no unauthorized host observation |
| Terminated Process/Context, host-retired grant | No usable old Clock authority; no guest resurrection |
| Actor send/request/spawn graphs containing real Clock | Atomic `NonTransferableValue`, no partial delivery |
| Parallel capture/input/output graph containing Clock | `NonParallelValue`; ordinary sampled Integer may cross |
| Samples equal; decrease after host rollback; NTP jumps | Values observed without monotonic clamp |
| Epoch -1 ns, epoch zero, before 1970, far beyond signed-64 ns | Exact arithmetic; normalized fraction; no Float/overflow |
| Provider only has coarse resolution | No fabricated finer accuracy |
| Synthetic host provider returns fixed/stepped/backward readings | Deterministic tests verify exact results, not guest grant fabrication |
| JVM, rebuilt Native CLI, installed Native distribution | Same semantics, independent actual samples and backends |
| JWT exp equality; iat/nbf future; skew 0..300 | Existing D246-C exact nanosecond comparisons unchanged |
| Host clock 30 minutes backward after JWT expiry | No false rollback-resistance claim |
| Logging source supplied explicitly from authorized Clock | Opt-in timestamps; logging import stays authority-free |
| HTTP slow-client deadline | **Not solved** by wall clock alone; keep HTTP security warning |

Potential portability trap: Native Image must not capture a fixed test `Instant` at image build time and later serve it as "now"; sample at invocation time from the installed runtime provider. Host provider failures cannot silently become `0` or a stale timestamp.

## 7. GITHUB021 delta/invariant reconciliation

| Invariant / approval history | Candidate C1 evaluation |
| --- | --- |
| D243-C authentic Entropy grant independent from other host rights | PASS: Clock independent; no redefinition of Entropy Future/close |
| D246-C JWT caller supplies actual `Instant`; import is authority-free | PASS: no JWT API or import behavior changes |
| D251-A1 explicit Future suspension in Actor bootstrap, no implicit Task | PASS: Clock is synchronous and cannot sneak in a wait |
| D242/MODULES bootstrap-private state and imports | PASS: `clock` slot only initial private module |
| Existing `std:datetime/instant` exact value, no ambient now | PASS: construction remains explicit |
| Previous D253 recommendation B1 (not owner-approved) | DELTA DISCLOSED: B1 `clock.now() -> Instant` replaced by C1 `clock.epochNanoseconds() -> Integer`; this is a refinement, not a reopened prior owner invariant |
| Owner's explicit 2026-11-09 `apruebo D253-C1` | EXACT MATCH: no implicit delegation, timer, `close()`, future or convenience facade |

**DECISION_APPROVAL_PROVENANCE=PASS; DECISION_INVARIANT_CONSISTENCY=PASS.** No earlier D253 owner-approved invariant was found that requires reopening. Every materially added feature remains expressly excluded. Approval of C1 does **not** automatically approve any later API names, host-specific backend constraints, or unsurveyed implementation semantic changes.

## 8. Publication and non-claims

The ratified design belongs to `guillermomolina/protos-project-docs:docs/project/decisions/language/` and this immutable research record to `docs/project/evidence/D253/`. Neither file defines normative Protos behavior. An independent I-family issue must publish the spec change and actual implementation in `guillermomolina/protos` with tests, CLI/embedding docs/examples and human validation. Re-check product HEAD when implementing; this evidence freezes only the inspected revision.

**NO PRODUCT COMMIT, NO CLOCK TEST RESULT, NO NATIVE CLOCK PARITY, NO REALWORLD AUTHENTICATION ACCEPTANCE and NO EXTRA DESIGN APPROVAL are claimed by this snapshot.**
