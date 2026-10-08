# Platform Architecture Decisions

This registry records durable, non-normative architecture decisions that depend
on a concrete implementation platform, host runtime, VM, operating system,
native backend, or equivalent execution substrate.

`PLATxxx` is deliberately distinct from implementation-independent decision
records and from normative semantic authority:

- `Dxxx` is an implementation-independent decision identifier family whose
  documentation role is classified separately by primary domain. Language/
  specification decisions live under `decisions/language/`; tooling/package-
  system decisions live under `decisions/tooling/`. Observable Protos semantics
  remain authoritative only in the applicable ratified material under `spec/`;
- `PLATxxx` owns durable platform/runtime architecture choices that materially
  constrain an implementation while remaining semantically invisible to Protos;
- `Ixxx`, `CLIxxx`, `TOOLxxx`, `PERFxxx`, `DISTxxx`, and similar work families
  implement, validate, measure, distribute, or otherwise consume those decisions.

A platform-dependent choice MUST NOT be placed in `PLATxxx` merely to avoid the
normative design process. If the choice changes observable Protos semantics,
identity, authority, ordering, isolation, failure behavior, portability
promises, or another language contract, the applicable `Dxxx`/specification
process remains authoritative first.

## Status vocabulary

- `OPEN` — a platform/runtime architecture question is identified but unresolved.
- `PROPOSED` — researched recommendation exists but project-owner approval is
  still pending.
- `RATIFIED` — explicitly approved durable platform/runtime architecture.
- `SUPERSEDED` — replaced by a later explicitly approved PLAT decision while
  retaining historical rationale.

## Decisions

| Item | Decision | Status | Approval | Primary consumers |
|---|---|---|---|---|
| PLAT001 | Truffle runtime hosting topology | RATIFIED | Explicit project-owner approval, 2026-09-08; 2026-09-09 A+ executable-layer amendment explicitly approved | I026-A4 and later Truffle-hosted runtime/tooling work |
| PLAT002 | Network capability represented-value architecture | RATIFIED | Explicit project-owner approval, 2026-09-09 after cross-language/runtime/capability/scalability review | I028-B and later Network/TCP runtime implementation |
| PLAT003 | JVM TCP live-resource and duplex-I/O architecture | RATIFIED | Explicit project-owner approval, 2026-09-09 after cross-language/runtime/full-duplex/future-scalability review | I028-C/D/E |
| PLAT004 | Truffle SourceSection ownership and materialization | RATIFIED | Explicit project-owner approval, 2026-09-09 after Truffle guidance + SimpleLanguage/TruffleRuby/GraalJS/FastR/Sulong/Apple Pkl scalability review | I026-B/C/F/G |
| PLAT005 | Truffle instrumentation coverage and StandardTags architecture | RATIFIED | Explicit project-owner approval, 2026-09-09 after Truffle + SimpleLanguage/TruffleRuby/GraalJS/Apple Pkl/GraalPy/Sulong/Espresso scalability review | I026-C/E/F/G |
| PLAT006 | JVM TCP host I/O operation engine architecture | RATIFIED | Explicit project-owner approval, 2026-09-09 after cross-language/runtime/readiness/completion/cancellation/scalability review | I028-E/F |
| PLAT007 | JVM NIO IPv6-only TCP listener enforcement | RATIFIED | Explicit project-owner approval, 2026-09-09 after mainstream-runtime, Truffle/GraalVM, Apple Pkl, future-backend and scalability review | I028-E/F |
| PLAT008 | Truffle replay-site identity across wrappers, rewrites and continuations | RATIFIED | Explicit project-owner approval, 2026-09-09 after Truffle + Apple Pkl/TruffleRuby/GraalJS/FastR/Sulong/Espresso/GraalPy Bytecode DSL replay/wrapper/future-scalability review | I026-C/F and future replay backend work |
| PLAT009 | Host-neutral first-effect attempt gate for asynchronous ByteWritable output | RATIFIED | Explicit project-owner approval, 2026-09-09 after ByteWritable/NIO race analysis plus cross-runtime, Truffle/GraalVM, Apple Pkl and future-scalability review | I028-E3 and future async output backends |
| PLAT010 | Bounded reusable platform carriers for normal Actor guest execution | RATIFIED | Explicit project-owner approval, 2026-09-09 after exhaustive BEAM/Go/Tokio/Akka/Orleans/Swift/Pony/Kotlin/GHC/OCaml/Loom and Truffle scalability review triggered by PERF001-F #239 | Actor runtime scheduler, #239, PERF001-F |
| PLAT011 | RuntimeHost-owned shared Actor carrier substrate across local Processes | RATIFIED | Explicit project-owner approval, 2026-09-09 after cross-runtime many-Process, 64/128+ core, NUMA/affinity, blocking-lane and distributed-evolution review | Actor runtime hosting, #239, PERF001-F and future host capacity governance |
| PLAT012 | Verified external package custody and source-resolution architecture | RATIFIED | Explicit project-owner approval, 2026-09-09 after exhaustive loader/runtime/package-store, CAS/distributed, lifetime and scalability review | TOOL001-F2E4/F2E5 and future immutable external package resource loading |
| PLAT013 | Truffle debugger/interop value projection architecture | RATIFIED | Explicit project-owner approval, 2026-09-09 after exhaustive Truffle-language (including Apple Pkl), Bytecode-DSL, identity, large-graph and scalability review | I026-D/E/F/G |
| PLAT014 | Truffle cooperative suspension continuation/compilation boundary | RATIFIED | Explicit project-owner approval, 2026-09-10 after deep Truffle-language/runtime review and complete A1/A2a/A2b/A2c/A2d feasibility evidence | PERF006, I026 continuation/backend work and future JVM/Truffle Task/Future execution |
| PLAT015 | Truffle debugger scope projection topology | RATIFIED | Explicit project-owner approval, 2026-09-10 after deep Truffle implementation audit including Apple Pkl, GraalPy Bytecode DSL, SimpleLanguage, TruffleRuby, GraalJS, FastR, Sulong and Espresso; future/scalability/Protos-fit review | I026-E/F/G |
| PLAT016 | Bytecode Closure default-parameter execution topology | RATIFIED | Explicit project-owner approval, 2026-09-10 after exhaustive maintained-Truffle implementation review including Apple Pkl plus future/scalability/Protos-fit scoring | PERF006-B2C3B and later Bytecode Closure activation/default execution |\n
| PLAT017 | Selected-standard `Object.call` Bytecode intrinsic after ordinary D013 lookup | RATIFIED | Explicit project-owner approval, 2026-09-10 after exhaustive Truffle implementation audit covering all 16 principal + 21 experimental catalogue entries, with deep Bytecode DSL/SimpleLanguage, TruffleSqueak, Espresso, TruffleRuby, Sulong, GraalJS, GraalPy, FastR, GraalWasm, Enso, SOMns and Pkl review plus future/scalability/Protos-fit analysis | PERF006-B2D3B and later D013-conformant Bytecode call preparation |
| PLAT018 | GraalVM DAP debug-invocation ownership, OS-ephemeral loopback endpoint and readiness boundary | RATIFIED | Explicit project-owner approval, 2026-09-10 after extended Truffle-language/GraalVM DAP lifecycle, port-0, future/scalability and Protos-fit review | LM009-D/E and future debugger launchers/distributions |
| PLAT019 | Native semantic suspension bridge into Bytecode C-prime continuations | RATIFIED | Explicit project-owner approval, 2026-09-10 after exhaustive audit of all 16 principal + 21 experimental/historical Truffle catalogue entries including Apple Pkl, with future/scalability/Protos-fit scoring selecting B-prime | PERF006-B3 and future JVM/Truffle suspension-capable native boundaries |
| PLAT020 | Context-bound Truffle file Source materialization | RATIFIED | Explicit project-owner approval, 2026-09-11 after exhaustive audit of all 16 principal + 21 experimental/historical Truffle catalogue entries including Apple Pkl, with future/scalability/Protos-fit scoring selecting A′ | LM009-E / D065 source-presentation correction and future hosted physical Source materialization |
| PLAT021 | Bytecode C-prime dynamic control/unwind representation | RATIFIED | Explicit project-owner approval, 2026-09-11 after exhaustive current Truffle-catalogue audit including Apple Pkl plus future/scalability/Protos-philosophy scoring selecting Candidate F | PERF006-B4 and later JVM/Truffle C-prime structured control/unwind |
| PLAT022 | Context source-readability authority for physical debugger paths | RATIFIED | Explicit project-owner approval, 2026-09-11 after exhaustive audit of all 16 principal + 21 experimental/historical Truffle catalogue entries, Graal LSP/Polyglot filesystem machinery, Apple Pkl, DAP/VS Code behavior and future/scalability/Protos-philosophy scoring selecting Candidate D′ | LM009-E D065/PLAT020 physical-source correction and future Truffle-hosted physical Source tooling readability |
| PLAT023 | Async exact-execution carrier topology for Test Tool | RATIFIED | Explicit project-owner approval, 2026-09-11 after exhaustive review of all 16 principal Truffle implementations including Apple Pkl plus future/scalability/Protos-philosophy scoring selecting Candidate A′ | TOOL002-H2B3 and future same-runtime async exact-execution host transport |
| PLAT024 | Static language-service hosting and protocol boundary | RATIFIED | Explicit project-owner approval, 2026-09-11 after exhaustive audit of all 16 principal + 21 experimental/historical Truffle catalogue entries including Apple Pkl, plus future-endurance/scalability/Protos-philosophy scoring selecting Candidate A′ | LM009-F and later LM009-G/H static editor intelligence |
| PLAT025 | Bytecode module-initialization composition boundary | RATIFIED | Explicit project-owner approval, 2026-09-11 after exhaustive audit of all 16 principal + 21 experimental/historical Truffle catalogue entries including Apple Pkl, plus outside-runtime evidence and future/scalability/Protos-philosophy scoring selecting Candidate A′ | PERF006-B6A5 and later C-prime production module initialization |
| PLAT026 | Bytecode production root-tag compatibility boundary | RATIFIED | Explicit project-owner approval, 2026-09-11 after expanded Truffle root/tooling audit including GraalPy Bytecode DSL, SimpleLanguage, TruffleRuby, GraalJS, Espresso, Sulong/LLVM, GraalWasm, FastR, TruffleSqueak, Enso and Apple Pkl, plus GraalVM DAP/Bytecode-DSL root-prolog evidence and future/scalability/Protos-philosophy scoring selecting Candidate C′+ | PERF006-B6A6 and later Bytecode production debugger/tooling compatibility |
| PLAT027 | Task-owned terminal lifecycle composition around C-prime guest execution | RATIFIED | Explicit project-owner approval, 2026-09-11 after exhaustive review of all 16 principal + 21 experimental/historical Truffle catalogue entries including Apple Pkl, plus Rust/Tokio, Swift concurrency, BEAM, Java CompletionStage and Go; future/scalability/Protos-philosophy scoring selected Candidate A′ | PERF006-B6A6A and later Task-owned host lifecycle finalization around C-prime guest execution |
| PLAT028 | C-prime-owned suspendible callback orchestration and native-leaf boundary | RATIFIED | Explicit project-owner approval, 2026-09-11 after exhaustive audit of all 16 principal Truffle implementations, relevant experimental/historical implementations including Apple Pkl comparison, plus non-Truffle async runtimes and future/scalability/Protos-philosophy scoring selecting Candidate C | PERF006-B production callback cutover and later JVM/Truffle standard/runtime operations that invoke suspendible ordinary guest callbacks |
| PLAT029 | Deferred C-prime ownership for non-task-backed asynchronous I/O operations | RATIFIED | Explicit project-owner approval, 2026-09-12 after exhaustive current/historical Truffle audit including Apple Pkl, GraalJS, GraalPy, SOMns, TruffleSqueak, Espresso, TruffleRuby and PorcE/Orc plus non-Truffle async-runtime review; future/scalability/Protos-philosophy scoring selected Candidate B′ | PERF006-B deferred TextReader/TextWriter/buffered-I/O callback migration and future non-task-backed asynchronous operation C-prime custody |
| PLAT030 | C-prime ownership for asynchronous I/O lifecycle release/close | RATIFIED | Explicit project-owner approval, 2026-09-12 after exhaustive current/historical Truffle audit including Apple Pkl, GraalJS, SOMns, GraalPy, grCUDA, Yona, TruffleSqueak, Espresso, TruffleRuby and PorcE/Orc plus secondary non-Truffle evidence; future/scalability/Protos-philosophy scoring selected Candidate A′ | PERF006-B TextWriter.close and later asynchronous lifecycle-release/owning-close C-prime migration |
| PLAT031 | Buffered I/O operation identity and C-prime ownership integration | RATIFIED | Explicit project-owner approval, 2026-09-12 after exhaustive current/historical Truffle audit including Apple Pkl, GraalJS, GraalPy, SOMns, Yona, TruffleSqueak, Espresso, TruffleRuby and PorcE/Orc; future/scalability/Protos-philosophy scoring selected Candidate A′ | PERF006-B buffered lifecycle/operation convergence, BufferedReader.read and BufferedWriter.flush PLAT029 C-prime migration; buffered release is released by D112/A′ and remains to be implemented under PLAT030 |
| PLAT032 | Non-canonical standard-wrapper C-prime orchestration boundary | RATIFIED | Explicit project-owner approval, 2026-09-13 after focused Truffle audit including Apple Pkl, TruffleSqueak, TruffleRuby, GraalPy, GraalJS, SOMns, Espresso, Sulong/GraalWasm and historical concurrent Truffle systems; future/scalability/Protos-philosophy scoring selected Candidate A | PERF006-B6B-F2 replay retirement and future copied/aliased/non-canonical standard wrappers that compose suspendible guest work |
| PLAT033 | Optimizing Truffle runtime dependency and packaging authority | RATIFIED | Explicit project-owner approval, 2026-09-13 after exhaustive current/historical Truffle packaging audit including GraalJS, GraalPy, GraalWasm, Espresso, Sulong, Enso, TruffleRuby, TruffleSqueak, TRegex, SimpleLanguage, FastR, grCUDA, SOMns, TruffleSOM, Yona and Apple Pkl; future/scalability/Protos-philosophy scoring selected Candidate A′ | PERF006-C optimizer runtime closure across Maven/Surefire, checkout execution, portable distribution and CI |
| PLAT034 | Truffle generic-tool tag compatibility envelope | RATIFIED | Explicit project-owner approval, 2026-09-13 after exhaustive principal Truffle implementation audit including Apple Pkl plus generic Graal DAP/LSP tag-request evidence; future/scalability/Protos-philosophy scoring selected Candidate C′ | PERF006-C3 and later production Bytecode debugger/LSP generic-tool compatibility |
| PLAT035 | Single execution backend and legacy Truffle AST retirement | RATIFIED | Explicit project-owner approval, 2026-09-17 after comparative decision packet and invariant/delta review selecting Candidate C | I067 and later single-backend execution work |
| PLAT036 | Bytecode DSL lexical-state representation boundary | RATIFIED | Explicit project-owner approval, 2026-09-24 selecting Candidate D and superseding prior E1 recommendation | I068 frame-backed lexical-state implementation |
| PLAT037 | Lazy execution-context physical materialization | RATIFIED | Explicit project-owner approval, 2026-09-25 selecting Candidate D — defer / retain eager physical context pending evidence | PERF010-A / PERF011 evidence before any lazy implementation |
| PLAT038 | Native Image bootstrap, runtime and release boundary | RATIFIED | Explicit project-owner approval, 2026-09-25 selecting Candidate B — dual-runtime build with native consumption and JVM development authority | I069 Native Image bootstrap and validation; later native DIST work after implementation evidence |
| PLAT039 | Truffle runtime-compilation boundary for Native Image | RATIFIED | Explicit project-owner approval, 2026-09-25 selecting Candidate C — evidence-gated PE-visible guest kernel with narrow host gateways | I069 Native Image runtime-boundary implementation; PERF010 causal representation/PE measurement |
| PLAT040 | Truffle hot-path dispatch and invocation architecture | RATIFIED | Explicit project-owner approval, 2026-09-25 selecting Candidate F′ — guarded hot call with frame-argument invocation and conditional guest-state materialization after expanded cross-Truffle evidence including Apple Pkl, GraalPy, GraalJS, TruffleRuby, TruffleSqueak, Espresso, Sulong, FastR, Enso and GraalWasm | Dedicated Ixxx implementation work allocated after ratification; PERF010-A / PERF011 causal performance and representation evidence |
| PLAT041 | Bytecode RootTag and materialized-local grouping boundary | RATIFIED | Explicit project-owner approval, 2026-10-01 selecting Candidate C′ — semantic-only source-root universe + inline Object-construction body + compact untagged infrastructure interpreter after PLAT026/PERF013 conflict analysis | PERF025-C / #758 slices C1a-C1c and later BUG008 carrier-retirement validation |
| PLAT042 | Structured-dispatch interpreter ownership boundary | RATIFIED | Explicit project-owner approval, 2026-10-01 selecting Candidate B′ — tagged semantic source interpreter plus one untagged structured-dispatch/C-prime owner with a structured-only helper CallTarget boundary | PERF025-C1c / #758 semantic RootTag cutover; subsequent evidence-gated BUG008 carrier-retirement validation |
| PLAT043 | Standard Boolean-control interpreter ownership boundary | RATIFIED | Explicit project-owner approval, 2026-10-01 selecting Candidate B — complete PreparedBooleanCall orchestration in the tagged semantic Bytecode interpreter, preserving PLAT042 for every other structured family | PERF025-C2B / #758 Boolean-owner cutover; unchanged-workload stack evidence before any BUG008 carrier change |
| PLAT044 | Semantic Closure activation versus physical RootTag boundary | RATIFIED | Explicit project-owner approval, 2026-10-02 selecting Candidate B′ — guarded Smalltalk-style open-coding for eligible immediate literal standard-control callbacks, including the explicit generic-Truffle stack-frame delta | PERF026-B / #767 first Boolean/common-mechanism implementation; later PERF026-C/D reuse after evidence |
| PLAT045 | Native Image guest-JIT capability boundary under upstream Bytecode DSL limitation | RATIFIED | Explicit project-owner approval, 2026-10-02 selecting Candidate B — Native Image supported with interpreter-only fallback while guest JIT is unavailable due to oracle/graal#14579; optimizing Native guest JIT restored only after objective upstream/revalidation gates pass | BUG013/#749 fallback-runtime cutover; TEST006/#755 Native admission update; DIST009/#743 after implementation validation |
| PLAT046 | Ordinary hosted single-Actor caller execution boundary | RATIFIED + OWNER AMENDMENT | Candidate B ratified 2026-10-02; explicit project-owner amendment 2026-10-07 extends the same pay-as-you-grow/lower-layer rule inward: canonical ordinary host/foreign callable entry uses the standard Truffle executable-value boundary and does not eagerly manufacture RootTask/Task/Actor/Process machinery merely for unused capability | PERF033/#832 canonical pay-as-you-grow callable interop; PERF032/#831 causal evidence; PERF025/#758 historical caller-thread cutover |
| PLAT047 | Portable Java slow-test admission architecture | RATIFIED + OWNER AMENDMENT | Candidate H ratified 2026-10-03; explicit project-owner amendment 2026-10-04 approves the deployed enforcement boundary `local=authoritative fail-closed`, `CI=advisory`, plus the exact TEST008-B calibration constants 0.5 / 4 / 1.5 / 2.5 / 1.35 / 25 / 180 | TEST008-B/#788 and TEST008/#761 closed at `e7b2ae2c`; PERF031/#787 remains independent and non-blocking |
| PLAT048 | Public-run exact external materialization authority boundary | RATIFIED | Explicit project-owner approval, 2026-10-04 selecting Candidate B′ — requirements-first Package Tool exact requirements plus public-run-bootstrap-owned exact materialization provider | TOOL001-F2E5/#93 public-run external execution |
| PLAT050 | Canonical formatter source/trivia authority and tooling bridge | RATIFIED | Explicit project-owner approval, 2026-10-04 selecting Candidate F — on-demand hybrid source-layout view + exact bundled Protos formatter policy over a tool-neutral host source mechanism | LM011-B/#670 canonical formatter implementation and later CLI/LSP/editor integration |
| PLAT054 | Standard Polyglot Context.eval and language-bindings embedding | RATIFIED — SPEC 0.1.450 PUBLISHED; I086 RUNTIME PENDING | Explicit owner approval 2026-10-08 of PLAT054-1's seven-point exact contract while preserving all eight earlier directions; Candidate A lazy Process/Context + standard Truffle scope; normative publication `guillermomolina/protos@ab1f2196` | I086/#840: 3E1 norm published, next 3E2 Future, 3E3 FS after B011 resolution, 3E4 Network B012 READY; prior A-D published; PERF032/#831 graph/latency parity remains separately unverified |

See `docs/project/decisions/platform/PLAT001_TRUFFLE_RUNTIME_HOSTING.md` for the selected topology,
its non-semantic boundary, alternatives, scaling rationale, invariants, and
explicitly deferred choices.

See `docs/project/decisions/platform/PLAT002_NETWORK_CAPABILITY_REPRESENTATION.md` for the selected
Network represented-capability boundary, rejected alternatives, scalability rationale and
backend choices deliberately deferred to later I028 slices.

See `docs/project/decisions/platform/PLAT003_TCP_LIVE_RESOURCE_ARCHITECTURE.md` for the selected JVM TCP
ordinary-resource representation, host-neutral acquisition custody and independent duplex-I/O
progress architecture; D052 remains authoritative for every observable TCP object-topology rule.

See `docs/project/decisions/platform/PLAT004_TRUFFLE_SOURCE_SECTION_OWNERSHIP.md` for the selected root-owned exact Source + node-local compact range + on-demand SourceSection projection, its cross-language rationale, scaling invariants and deliberately deferred instrumentation/tooling choices.

See `docs/project/decisions/platform/PLAT005_TRUFFLE_INSTRUMENTATION_ARCHITECTURE.md` for the selected layered semantic-minimum instrumentation architecture: common source-node mechanism, explicit Statement/Call baseline tags, replay-aware wrapper placement, deferred root/expression/value/yield surfaces, and scaling rationale.

See `docs/project/decisions/platform/PLAT006_TCP_HOST_IO_OPERATION_ENGINE.md` for the selected host-neutral I/O operation-engine boundary, initial bounded JDK NIO readiness backend, cross-runtime/scalability rationale, future completion/native-backend compatibility constraints, and deliberately deferred tuning choices.

See `docs/project/decisions/platform/PLAT007_IPV6_ONLY_TCP_LISTENER_ENFORCEMENT.md` for the public-JDK composite IPv6-only listener baseline, authority/scope and all-or-nothing port invariants, cross-runtime/Truffle rationale, known high-address-cardinality cost and preserved native O(1)-socket future path.

See `docs/project/decisions/platform/PLAT008_TRUFFLE_REPLAY_SITE_IDENTITY.md` for the selected logical replay-site identity contract, zero-allocation delegate-backed AST representation, wrapper transparency, live-replacement preservation rule and Bytecode-DSL-compatible future representation boundary.

See `docs/project/decisions/platform/PLAT009_BYTEWRITABLE_FIRST_EFFECT_GATE.md` for the host-neutral transient first-effect-attempt gate preserving existing ByteWritable cancellation/commitment semantics across readiness and future completion/native/WASI/brokered backends.

See `docs/project/decisions/platform/PLAT010_ACTOR_PLATFORM_CARRIERS.md` for the selected bounded reusable platform-carrier primitive for normal Actor guest execution, the rejection of virtual-thread carriers as the default CPU/guest substrate, and the required repeated 1/2/4/8 production-path evidence before PERF001-F #239 may close.

See `docs/project/decisions/platform/PLAT011_RUNTIMEHOST_CARRIER_SUBSTRATE.md` for the selected RuntimeHost-owned bounded carrier-capacity topology across multiple local Processes, its O(runtime CPU capacity) physical-thread scaling target, separate blocking/offload lane boundary, and preserved future work-stealing/NUMA/resource-governance evolution path.


See `docs/project/decisions/platform/PLAT012_VERIFIED_EXTERNAL_PACKAGE_CUSTODY_SOURCE_RESOLUTION.md` for the run-owned exact immutable package-resource scope, lazy host-neutral verified-resource reader, canonical no-path external ModuleKey boundary and future CAS/distributed backing evolution.

See `docs/project/decisions/platform/PLAT013_TRUFFLE_DEBUGGER_INTEROP_VALUE_PROJECTION.md` for the selected semantic-value-native read-only interop baseline, local-slot-only object projection, synthetic scope/view adapter boundary, identity/side-effect constraints and future Bytecode DSL / non-JVM evolution path.

See `docs/project/decisions/platform/PLAT014_TRUFFLE_COOPERATIVE_CONTINUATIONS.md` for the selected C-prime stackful continuation composition over Truffle Bytecode DSL, its suspension-only cost boundary, semantic-preservation constraints, cross-runtime evidence, scalability invariants and deliberately deferred implementation choices.

See `docs/project/decisions/platform/PLAT015_TRUFFLE_DEBUGGER_SCOPE_PROJECTION.md` for the selected activation-native debugger-scope projection, absence of an artificial language top scope or named receiver, one-runtime-lookup authority, suspension-bounded lifetime and AST/Bytecode-DSL-independent tooling boundary.\nSee `docs/project/decisions/platform/PLAT016_BYTECODE_CLOSURE_DEFAULT_EXECUTION_TOPOLOGY.md` for the selected single Bytecode Closure activation-root topology, default-expression execution and continuation constraints, cross-language evidence, scalability rationale and semantically invisible internal-outlining freedom.\n

See `docs/project/decisions/platform/PLAT017_SELECTED_STANDARD_OBJECT_CALL_INTRINSIC.md` for the selected post-lookup canonical-standard `Object.call` Bytecode intrinsic, its D013/PLAT014/tooling equivalence gates, exhaustive Truffle precedent, scalability constraints and the exact PERF006-B2D3B implementation boundary released by ratification.

See `docs/project/decisions/platform/PLAT018_DAP_DEBUG_SESSION_HOSTING.md` for the selected RuntimeHost/Engine-owned one-shot GraalVM DAP topology, OS-ephemeral loopback endpoint, Protos-owned readiness boundary, launch/wait-attached lifecycle constraints, scaling rationale, distribution consequence and deliberately deferred public tooling choices.


See `docs/project/decisions/platform/PLAT019_NATIVE_SEMANTIC_SUSPENSION_BRIDGE.md` for the selected B-prime suspension-capable native provenance boundary, explicit resumability requirement, interpreter-owned C-prime capture, two-phase Task continuation publication, exhaustive Truffle/Apple-Pkl review, scalability constraints and deliberately unspecified private native-to-interpreter transport.


See `docs/project/decisions/platform/PLAT020_CONTEXT_BOUND_TRUFFLE_FILE_SOURCE_MATERIALIZATION.md` for the selected A′ split between immutable backend-neutral source facts and owning-Context physical Truffle Source materialization, exhaustive Truffle/Apple-Pkl review, D065 path/content invariants, multi-Context scalability constraints and deliberately deferred Java representation/cache choices.

See `docs/project/decisions/platform/PLAT021_BYTECODE_DYNAMIC_CONTROL_UNWIND.md` for the selected Candidate F Bytecode-local structured-control ownership, separate normal/suspension/unwind lanes, narrow private EH bridge, exhaustive Truffle/Apple-Pkl review, scaling constraints and PERF006-B4 implementation boundaries.

See `docs/project/decisions/platform/PLAT022_CONTEXT_SOURCE_READABILITY_AUTHORITY.md` for the selected Candidate D′ Context-local admitted-source read-only authority, exhaustive Truffle/Apple-Pkl/tooling review, D065/PLAT020 composition, debugger-transparency and least-authority invariants, multi-Context scaling constraints and deliberately deferred public filesystem semantics.

See `docs/project/decisions/platform/PLAT023_ASYNC_EXACT_EXECUTION_CARRIER_TOPOLOGY.md` for the selected Candidate A′ fresh-platform-Thread-per-admitted-execution topology, exhaustive Truffle/Apple-Pkl carrier audit, D055/D069 composition, cancellation/custody invariants, scaling analysis and preserved virtual-thread/OS-worker/remote evolution paths.

See `docs/project/decisions/platform/PLAT024_STATIC_LANGUAGE_SERVICE_HOSTING_PROTOCOL_BOUNDARY.md` for the selected Candidate A′ dedicated toolchain-matched static LSP process, stdio baseline, thin editor client, protocol-neutral analysis core, exhaustive Truffle/Apple-Pkl survey, scalability/future stress analysis, and preserved self-hosted/native migration path.

See `docs/project/decisions/platform/PLAT025_BYTECODE_MODULE_INITIALIZATION_COMPOSITION_BOUNDARY.md` for the selected Candidate A′ exact-standard post-lookup `import` intrinsic, backend-private module-lifecycle carrier, nested C-prime child-root composition, exhaustive Truffle/Apple-Pkl survey, scaling/future constraints and PERF006-B6A5 implementation invariants.

See `docs/project/decisions/platform/PLAT026_BYTECODE_PRODUCTION_ROOT_TAG_COMPATIBILITY_BOUNDARY.md` for the selected Candidate C′+ semantic-root/helper-root split, automatic `RootTag` only on truthful top-level/module/Closure activation roots, deferred `RootBodyTag`, expanded Truffle/Apple-Pkl/DAP evidence, scaling/future constraints and PERF006-B6A6 implementation invariants.
See `docs/project/decisions/platform/PLAT027_TASK_TERMINAL_LIFECYCLE_BOUNDARY.md` for the selected Candidate A′ Task-owned one-shot terminal lifecycle boundary, C-prime sole-continuation constraint, exhaustive Truffle/non-Truffle evidence, scaling and cancellation invariants, and the bounded PERF006-B6A6A implementation authority released by ratification.

See `docs/project/decisions/platform/PLAT028_C_PRIME_CALLBACK_ORCHESTRATION_NATIVE_LEAF_BOUNDARY.md` for the selected Candidate C single-continuation architecture: callback-owning post-guest control stays C-prime-resumable, Java/native remains guest-non-reentrant leaf mechanics, PLAT019 native suspension and PLAT027 terminal finalization remain valid, and no generic host continuation stack, blocking carrier, child-Task bridge, replay island or host-stack capture is selected.

See `docs/project/decisions/platform/PLAT032_NONCANONICAL_STANDARD_WRAPPER_C_PRIME_ORCHESTRATION.md` for the selected ordinary-wrapper identity preservation rule, behavior-specific C-prime orchestration boundary, PLAT017/019/028 composition, rejected generic host-continuation ABI, scalability invariants and replay-retirement consequence.

See `docs/project/decisions/platform/PLAT034_TRUFFLE_GENERIC_TOOL_TAG_COMPATIBILITY_ENVELOPE.md` for the selected Candidate C′ semantic-minimum generic-tool tag envelope: `ExpressionTag` reuses exactly the PLAT005 statement boundary, `AlwaysHalt` is provided with zero guest locations, variable/root-body/try tags remain deferred, and any later undeclared generic-tool tag requirement must return through explicit governance.


See docs/project/decisions/platform/PLAT035_SINGLE_EXECUTION_BACKEND_LEGACY_AST_RETIREMENT.md for the selected single executable backend and bounded legacy-AST retirement boundary.

See `docs/project/decisions/platform/PLAT040_TRUFFLE_HOT_PATH_INVOCATION_ARCHITECTURE.md` for the selected guarded stable-send, compact frame-argument invocation ABI, conditional guest-state materialization, optional-state separation, PLAT037 evidence-gate result and exact PLAT036/PLAT039 composition. Final comparative evidence is retained under `docs/project/evidence/PLAT040/PLAT040_CROSS_TRUFFLE_HOT_CALL_DECISION_EVIDENCE.md`.

See docs/project/decisions/platform/PLAT036_BYTECODE_DSL_LEXICAL_STATE_REPRESENTATION_BOUNDARY.md for the selected frame-backed semantic-context adapter and one-authority lexical-state architecture.

See docs/project/decisions/platform/PLAT037_LAZY_EXECUTION_CONTEXT_PHYSICAL_MATERIALIZATION.md for the ratified defer/keep-eager physical context boundary and the evidence gate before any later lazy materialization.

See docs/project/decisions/platform/PLAT038_NATIVE_IMAGE_BOOTSTRAP_RUNTIME_RELEASE_BOUNDARY.md for the selected dual-runtime JVM/native architecture, operational stage0/stage1 model, native guest-JIT requirement, and strict build/native/dist/release separation.

See `docs/project/decisions/platform/PLAT039_TRUFFLE_RUNTIME_COMPILATION_BOUNDARY.md` for the ratified evidence-gated PE-visible guest-kernel architecture, narrow host/cold boundary rule, mixed-gateway SPLIT rule, and I069 implementation authority. See `docs/project/evidence/PLAT039/PLAT039_TRUFFLE_RUNTIME_COMPILATION_BOUNDARY_DECISION_EVIDENCE.md` for the exhaustive decision evidence, exact GraalVM 25.3.4.1 source identity, gateway inventory, falsification and GITHUB010 scoring. The earlier `docs/project/evidence/PLAT039/PLAT039_TRUFFLE_RUNTIME_COMPILATION_BOUNDARY_PRELIMINARY_EVIDENCE.md` remains historical preliminary evidence.

See `docs/project/decisions/platform/PLAT041_BYTECODE_ROOT_TAG_MATERIALIZED_LOCAL_GROUPING_BOUNDARY.md` for the ratified Candidate C′ topology: semantic-only source roots with automatic `RootTag`, inline resumable Object-construction bodies, preserved PERF013 same-generation materialized-local access, and a compact untagged infrastructure interpreter. Decision evidence is retained under `docs/project/evidence/PLAT041/PLAT041_ROOT_TAG_MATERIALIZED_LOCAL_DECISION_EVIDENCE.md`.

See `docs/project/decisions/platform/PLAT042_STRUCTURED_DISPATCH_INTERPRETER_OWNERSHIP_BOUNDARY.md` for the ratified Candidate B′ amendment to PLAT041 C1c: the tagged semantic source interpreter owns ordinary source execution, while one untagged structured-dispatch/C-prime interpreter owns resumable structured protocols behind a structured-only helper CallTarget. Investigation evidence is retained under `docs/project/evidence/PLAT042/PLAT042_STRUCTURED_DISPATCH_OWNERSHIP_INVESTIGATION.md`.

See `docs/project/decisions/platform/PLAT043_STANDARD_BOOLEAN_CONTROL_INTERPRETER_OWNERSHIP_BOUNDARY.md` for the ratified Candidate B narrow amendment to PLAT042: the finite prepared standard Boolean-control capability executes in the tagged semantic Bytecode interpreter after ordinary selection, while every other structured prepared family remains owned by the untagged structured/C-prime interpreter. Investigation evidence is retained under `docs/project/evidence/PLAT043/PLAT043_STANDARD_BOOLEAN_CONTROL_OWNERSHIP_INVESTIGATION.md`.
See `docs/project/decisions/platform/PLAT044_SEMANTIC_CLOSURE_PHYSICAL_ROOT_BOUNDARY.md` for the selected guarded standard-control callback open-coding boundary, the explicit PLAT026 RootTag/frame delta, exact ordinary Closure fallback, and PERF026-B/C/D release sequence.
See `docs/project/decisions/platform/PLAT045_NATIVE_IMAGE_GUEST_JIT_CAPABILITY_BOUNDARY.md` for the ratified temporary Native capability matrix: real Native Image AOT host support with interpreter-only Protos guest execution while `oracle/graal#14579` blocks generated Bytecode DSL Native Tier-2, plus the objective gate that restores optimizing Native guest JIT.

See `docs/project/decisions/platform/PLAT046_ORDINARY_HOSTED_SINGLE_ACTOR_CALLER_EXECUTION_BOUNDARY.md` for the ratified direct-caller ordinary hosted execution model, explicit reusable-session serialization/lifecycle boundary, orthogonal retained deep-recursion requirement, rejected universal carrier alternatives and PERF025 implementation/remeasurement handoff.

See `docs/project/decisions/platform/PLAT047_PORTABLE_JAVA_SLOW_TEST_ADMISSION_ARCHITECTURE.md` for Candidate H plus the 2026-10-04 owner amendment: the machine-control/normalization/confirmation/global-signal architecture is retained, local TEST008 admission is authoritative fail-closed, hosted CI admission is advisory, and the exact deployed calibration constants are owner-approved. Final implementation/closure evidence is retained under `docs/project/evidence/TEST008/TEST008_B_FINAL_CLOSURE.md`.

PLAT046 is OPEN and has no selected decision record. Intake/trigger evidence is retained under `docs/project/evidence/PLAT046/PLAT046_INTAKE_AND_TRIGGER_EVIDENCE.md`; live decision authority remains `guillermomolina/protos#778` until explicit project-owner ratification.

See `docs/project/decisions/platform/PLAT048_PUBLIC_RUN_EXTERNAL_MATERIALIZATION_AUTHORITY_BOUNDARY.md`
for the ratified Candidate B′ boundary: bundled Package Tool derives inert exact
external requirements, while the public-run host owns a run-scoped exact
materialization provider that maps only complete locked external identities to
already-present local roots before unchanged F2E2 verification. Decision evidence
is retained under
`docs/project/evidence/PLAT048/PLAT048_PUBLIC_RUN_EXTERNAL_MATERIALIZATION_DECISION_EVIDENCE.md`.
The decision releases TOOL001-F2E5/#93 implementation without selecting fetch,
store writes, GC, ambient lookup, a public store layout or a new CLI/config
surface.

See `docs/project/decisions/platform/PLAT049_GUEST_DIAGNOSTIC_STACK_CAPTURE_AUTHORITY.md`
for the ratified Candidate C guest diagnostic-stack capture authority: terminal
Error occurrences pay for a failure-only Truffle physical stack/current-frame
acquisition, generated continuation roots are normalized to semantic
Bytecode/source locations, PLAT044 inline callbacks are recovered from active
nested semantic RootTags, and only a bounded immutable Protos diagnostic trace is
retained. No success-path caller provenance, live frame graph, Future producer
stack, Actor/Process causal concatenation or guest-visible Error mutation is
introduced. Ratification evidence is retained under
`docs/project/evidence/PLAT049/PLAT049_CANDIDATE_C_RATIFICATION_EVIDENCE.md`;
CLI008-C/#416 is released for the bounded CLI008-C1 implementation.


See `docs/project/decisions/platform/PLAT050_CANONICAL_FORMATTER_SOURCE_TRIVIA_AUTHORITY.md`
for the ratified Candidate F formatter architecture: exact immutable source,
existing TokenOccurrence/SourceSpan and Surface AST remain the base; formatter-only
trivia, parser source facts and comment attachment are built on demand; no
lossless CST is required for the D183 baseline; higher-level fixed-style policy
is an exact bundled Protos formatter over a tool-neutral host source mechanism.
Ratification releases LM011-B1/#670 without changing Protos semantics, PLAT024,
or the deferred range/check/recovery/refactoring surface. Evidence is retained
under
`docs/project/evidence/PLAT050/PLAT050_CANDIDATE_F_RATIFICATION_EVIDENCE.md`.


See `docs/project/decisions/platform/PLAT051_STANDARD_LIBRARY_SEMANTIC_VALUE_TRANSFER_REMATERIALIZATION.md`
for the ratified Candidate C Standard Library semantic-value transfer boundary, including the
A2 implementation amendment that separates source-side validated inert transfer records from
destination-domain materialization before guest observation. The completed first production
consumer is `std:regex/Regex`: Pattern transfers source + canonical flags and recompiles in the
destination; Match transfers capture-result data and rebuilds without rematching. Generic transfer
retains Closure/capability isolation, exact bootstrap family authority and pay-as-you-grow behavior.
Final implementation evidence is retained under
`docs/project/evidence/PLAT051/PLAT051_B_IMPLEMENTATION_VALIDATION.md`.

See `docs/project/decisions/platform/PLAT052_FOREIGN_AUTHORITY_SANDBOX_ENFORCEMENT.md`
for the ratified Candidate B′ foreign-authority boundary: foreign execution starts
with zero ambient host authority; Protos capabilities may be mapped only through
scope-preserving enforcement that does not amplify authority to unrelated foreign
code; providers/libraries that cannot preserve that contract in-process require
explicit trusted-host policy, stronger provider-specific isolation, or fail
closed. Import and dependency presence never grant authority, and concrete
provider/Context/process topology remains owned by PLAT053/#822. Ratification
evidence is retained under
`docs/project/evidence/PLAT052/PLAT052_OWNER_APPROVAL_AND_RATIFICATION.md`.

See `docs/project/decisions/platform/PLAT053_FOREIGN_PROVIDER_RUNTIME_LIFECYCLE_ARCHITECTURE.md`
for the ratified Candidate B′ foreign-provider topology: each Protos Process
lazily owns provider compartments, while mutable foreign application/runtime
state is isolated behind Actor-local logical provider sessions and each provider
selects the smallest correct physical Context/realm/interpreter/classloader/
isolate/service topology. The existing Protos Process Context is not the
universal foreign Context; provider caches never replace the Actor-local Protos
module cache; PLAT052 authority profiles and D189 callback rules remain hard
gates; unused providers create no foreign runtime resources. Ratification
evidence is retained under
`docs/project/evidence/PLAT053/PLAT053_OWNER_APPROVAL_AND_RATIFICATION.md`.

See `docs/project/decisions/platform/PLAT054_STANDARD_POLYGLOT_EMBEDDING_CONTRACT.md`
for the ratified standard Polyglot embedding architecture (lazy one Process per Context,
ordinary module eval and last-completed module bindings, exact Closure extraction,
fatal RootActor turn policy and PAY AS YOU GROW). The approved comparison and
normative publication gate are recorded under
`docs/project/evidence/PLAT054/PLAT054_1_OWNER_APPROVAL_NORMATIVE_RECONCILIATION.md`.
The PLAT054-3E1 five-choice host suspension/Filesystem/Network addendum was **expressly approved by the owner on 2026-10-08** and durably published in [PLAT054-3E1 approval evidence](../evidence/PLAT054/PLAT054_3E1_HOST_SUSPENSION_AND_AUTHORITY_OWNER_APPROVAL.md). This is a non-normative approval record; the extension to `spec/0.1.449` is **not yet published**. B011/B012 remain BLOCKED pending normative publication, not pending approval; I086 is still open, and I087 has separate scope.
The approved normative specification was published in `guillermomolina/protos@ab1f2196e6736d4df01e83f685a3fc8aa3f606ac` as revision `0.1.449`; evidence is in `docs/project/evidence/PLAT054/PLAT054_2_NORMATIVE_PUBLICATION.md`. The Java runtime contract and benchmark parity remain pending.

The PLAT054-3E1 host entry and authority design addendum is now **normatively published as spec 0.1.450** at [`guillermomolina/protos@3bb1278d91ee5cea98031462be2a5c4dd3c89019`](https://github.com/guillermomolina/protos/commit/3bb1278d91ee5cea98031462be2a5c4dd3c89019). [Publication evidence](../evidence/I086/I086_PLAT054_3E1_NORMATIVE_HOST_ENTRY_AND_AUTHORITY_PUBLICATION.md). I086 next PLAT054-3E2 implementation: pending `Future.value()` from host `Value.execute()`; B012 READY for Network backend, B011 BLOCKED only for unrepresentable-base bootstrap failure versus slot absence. Reported green tests are human-reported; Native/benchmark parity unverified.
