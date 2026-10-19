# PLAT067 — Eliminate NORMAL/DEBUG physical execution profiles: code and API research

**Record type:** immutable source-backed architecture/implementation evidence. **Decision status:** CANDIDATE B RECOMMENDED, **NOT OWNER-RATIFIED**. **Date:** 2026-10-19.
**Product baseline:** [guillermomolina/protos@27cb31d20e517d8377e58963ea595e5ec16bfd15](https://github.com/guillermomolina/protos/commit/27cb31d20e517d8377e58963ea595e5ec16bfd15).
**GitHub:** [PLAT067/#933](https://github.com/guillermomolina/protos/issues/933), [D208/#934](https://github.com/guillermomolina/protos/issues/934), [I116/#935](https://github.com/guillermomolina/protos/issues/935).
**Reference runtime:** GraalVM / Truffle 25.4.4.1.1 (same target as Protos baseline); code citations below use exact Git commit hashes where available. **Validation:** read-only source inspection and comparative reasoning; no source edits, compilation, tests, benchmark, profile run, native run or debugger invocation were performed by the coordinating agent. No performance improvement is established.

## 1. Request and result

The owner requests removing NORMAL/DEBUG as a guest execution/compilation distinction and converging toward ordinary Truffle Bytecode DSL tooling. The proposed architecture has exactly **one** proof-optimized semantic bytecode stream, shared by all guest Contexts independent of any DAP host service. Instrumentation, source and tags materialize with Truffle's lazy facilities, not a second semantics compiler. The bytecode generator was already unified in I111; do not reintroduce a second `@GenerateBytecode`.

The existing NORMAL optimizer is the **base for all Contexts**, not discarded in favor of DEBUG. The resulting debugger can observe real surviving execution but cannot step into an optimized-away CallTarget. The trade-off is explicit, not hidden. All observable Standard Library changes must pass D208.

## 2. Current-code physical inventory and exact edits

| Owner / source (under `src/main/java/com/guillermomolina/protos/`) | Baseline role | Proposed edit (I116, only after design approval) |
|---|---|---|
| `execution/host/embedding/ProtosExecutionMode.java` | Two modes NORMAL/DEBUG | Remove class after all consumers migrate |
| `execution/host/embedding/ProtosLanguage.java` | Experimental `@Option(name="Debug")`; parse() passes Context mode | One `sourceCompiler.compileBytecode(source, language)`; remove option if exact D208 selection authorizes |
| `execution/host/embedding/ProtosLanguageContext.java` | Immutable mode from `protos.Debug` and Context-owned plan cache | Remove mode; retain language/Context identity, plan cache, real projection guarantees |
| `execution/host/embedding/ProtosPolyglotRuntimeHost.java` | Mode-labeled Engine options; `openDebug` also starts DAP service | Remove mode Engine option; keep exact DAP options, endpoint, readiness and host lifecycle, deprecate/clarify public `isDebugMode()` only as approved |
| `execution/host/embedding/ProtosStandaloneHostedSession.java` | Test helper opens NORMAL/DEBUG hosts | Convert to ordinary vs DAP/instrumented test scenarios |
| `execution/host/embedding/ProtosSourceCompiler.java` | `compileBytecode(Source,Language,Mode)` | `compileBytecode(Source,Language)`; preserve compiler/binding owner |
| `execution/modules/ProtosModuleRuntime.java` | Passes mode to module compilation | Use common compiler; preserve module ownership/source |
| `execution/bytecode/lowering/CanonicalToBytecodeLowerer.java` | Mode-controlled physical emission and root topology | Delete profile field and all mode checks; retain exact independent optimizer admission proofs and deterministic builder reparse |
| `execution/bytecode/activation/ProtosBytecodeClosureExecutionPlan.java` | Profile property and rebuilding parameter | Remove profile from constructor/state/rebuild |
| `execution/bytecode/activation/ProtosClosureExecutionPlan.java` | Exposes bytecode profile for projections/tests | Remove profile; retain canonical/source/language authority |
| `execution/bytecode/activation/ProtosClosureCallPreparation.java` | Plan admission checks Context mode match | Drop only mode-mismatch checks; **retain** foreign/language/Context/projection checks |
| `execution/bytecode/activation/ProtosBytecodeTagTreeNodeExports.java` | NodeLibrary scope projection using B5 and frame/native locals | Preserve exact real activation projection and lazy materialization; remove stale profile descriptions; never fabricate missing frames |
| `execution/bytecode/activation/ProtosBytecodeRootNode.java` | Already **one** `@GenerateBytecode`, tag instrumentation enabled, selective manual RootTag | Keep single generator and current flags, auxiliary Task/C-prime root silence |
| `execution/host/tooling/ProtosDiagnosticTraceCapture.java` | Lazy RootTag + source reparse; BytecodeLocation updates | Preserve error-only lazy operation and stable semantic root identity |
| `execution/facilities/ProtosRuntimeMetadataFacility.java` | Returns mode as public metadata | Consume D208 normative contract *after* approval |
| `protos/lib/runtime.protos`, `docs/guide/library/runtime.md`, `spec/semantics/MODULES.md` §73, `spec/PROTOS_SPEC_CHANGELOG.md` | Normative/docs currently declare exactly 5 slots | Consume ratified D208, not a PLAT-only assumption |
| `cli/ProtosCli.java` and DAP test launcher | Real host DAP protocol | Preserve CLI debug command, `PROTOS_DEBUG_READY`, endpoint and security |

### All eight mode-dependent lowering gates

Baseline file: [CanonicalToBytecodeLowerer.java@27cb31d](https://github.com/guillermomolina/protos/blob/27cb31d20e517d8377e58963ea595e5ec16bfd15/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java). The **line numbers are evidence, not patch offsets**; always inspect current HEAD when implementing.

| Approximate baseline line | Selection | Non-mode admission facts retained |
|---|---|---|
| 1250 | I101 constant-result direct root body | parameterless, scopeless, proven literal/Boolean/Number |
| 1533 | I101 scalar local create/read/fold | owned frame-native locals, bounded known constants, no effects |
| 2138 | I104 one-arg identity entry | exact parameter binding, lexical identity, no defaults/rest |
| 3239 | I104 required-prefix fast binding | verified physical ordinals, ownership, no persistent authority |
| 3947 | I112 module root prolog authority | terminal call shape, no side effects invalidating proof |
| 3993 | I112 terminal Closure invocation | direct operands, no spread, no inline state, true terminality |
| 8812 | I095/I107 canonical Boolean control | published standard Core, exact selector/receiver, arity, no conflicting inline activation |
| 10506 | I096 Closure literal/root-binding inlining | bounded args, proven non-dynamic selection, no disallowed rest/default/capture/effects |

Other `profile` argument passing around root emission/plan construction must also be removed. **Never blindly strip whole branches**: preserve all non-profile guards and fallbacks, and run independent physical Builder oracles.

## 3. Physical incompatibility discovered by direct tests

Current [I101 profile-specialized test@27cb31d](https://github.com/guillermomolina/protos/blob/27cb31d20e517d8377e58963ea595e5ec16bfd15/src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI101ProfileSpecializedLoweringTest.java) asserts:

- `() => { 1 }` and `() => { true.ifTrue() { 1 } }` produce the same minimal handwritten Builder oracle in NORMAL and **two physical roots** (module + enclosing Closure).
- In DEBUG the latter produces **three** physical roots (a callback root remains), ordinary send operations including `PrepareSendOne`, and callback instrumentation.
- `ensureComplete` and reparse must retain root-group order, CallTargets and semantic instruction signatures.

When Boolean folding omits a callback root, no future `@Instrumentation`, source materialization or debugger attach can resurrect its executable CallTarget without changing the semantic instruction/root topology. **Therefore three claims cannot be made simultaneously:** exact former NORMAL minimum; same physical bytecode for DAP and ordinary Contexts; former DEBUG callback stepping into optimized-away work. Candidate B retains the first two and provides truthful debugging at surviving executable locations only. Explicit owner approval of this trade-off is required.

## 4. Truffle 25.4 API/behavior and relevant peers

- Truffle Bytecode DSL supports `BytecodeConfig.DEFAULT`, `WITH_SOURCE`, `COMPLETE`, generated `newConfigBuilder().addTag(...)` and `BytecodeRootNodes.update(...)`/source requests; builder replay must be **deterministic and idempotent**. Source/Tag/`@Instrumentation` metadata is not eagerly materialized by default; upgrades are **monotonic** (cannot remove added instructions), and update may invalidate optimized code / shift BCIs. `BytecodeLocation.update()` must translate locations after reparse. The retained parser may keep AST/binding references alive. See [Truffle Bytecode DSL UserGuide](https://github.com/oracle/graal/blob/c40ade41d719578019e26b44a24c0c1c28f70ef1/truffle/docs/bytecode_dsl/UserGuide.md) and [InstrumentationTutorial](https://github.com/oracle/graal/blob/c40ade41d719578019e26b44a24c0c1c28f70ef1/truffle/src/com.oracle.truffle.api.bytecode.test/src/com/oracle/truffle/api/bytecode/test/examples/InstrumentationTutorial.java).
- `@Instrumentation` has **transparent stack effects**: a zero-child/zero-result operation or a one-child/one-result wrapper. It cannot replace ordinary conditional branches, semantic calls, argument binding, continuation or arbitrary frame creation. Do not introduce an `@Instrumentation` operation solely to carry a mode flag.
- [GraalPy PBytecodeDSLRootNode](https://github.com/oracle/graalpython/blob/7e7182b1036d56f8e89ecc9c7a86cee292180aa6/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java) declares one generator and optional `@Instrumentation` (e.g. `TraceOrProfileCall` / `TraceLine` near lines 1029–1051); [RootNodeCompiler](https://github.com/oracle/graalpython/blob/7e7182b1036d56f8e89ecc9c7a86cee292180aa6/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) emits trace points as builder instructions within the same parser.
- [SimpleLanguage SLBytecodeRootNode](https://github.com/oracle/graal/blob/c40ade41d719578019e26b44a24c0c1c28f70ef1/truffle/src/com.oracle.truffle.sl/src/com/oracle/truffle/sl/bytecode/SLBytecodeRootNode.java) uses one generator with `enableTagInstrumentation=true`, explicit root tagging where appropriate.
- GraalJS has [on-demand AST tag materialization](https://github.com/oracle/graaljs/blob/bc62bb7e2006cdc8ea8868d0b8d9605b946fa25d/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/control/IfNode.java); **AST technique is not a drop-in Bytecode DSL replacement**.
- Sulong/LLVM [instrumentable statement wrapper](https://github.com/oracle/graal/blob/c40ade41d719578019e26b44a24c0c1c28f70ef1/sulong/projects/com.oracle.truffle.llvm.runtime/src/com/oracle/truffle/llvm/runtime/nodes/api/LLVMStatementNode.java), [Espresso debug architecture](https://github.com/oracle/graal/blob/c40ade41d719578019e26b44a24c0c1c28f70ef1/espresso/docs/how-espresso-works.md), Apple [Pkl VmLanguage](https://github.com/apple/pkl/blob/main/pkl-core/src/main/java/org/pkl/core/runtime/VmLanguage.java) and TruffleRuby are useful comparison surfaces for instrumentation, not evidence of equivalent `@GenerateBytecode` lowering.
- **Do not conflate `BytecodeDescriptor.update` language-wide reach with per-Context options.** Test cross-Context ownership, source reparse and instrumented host isolation.

## 5. Decision packet and GITHUB021 invariant delta

| Candidate | Proposal | Main flaw | Alignment 1–5 (reasoned only) |
|---|---|---|---:|
| A | Keep two guest modes and their code streams | Violates explicit one-stream objective | 1 |
| **B (recommended)** | Always minimal/proven semantic stream + on-demand Truffle tooling | Optimized-out callbacks cannot be individually stepped into | **5** |
| C | One permanently DEBUG-shaped semantic stream | Real stepping, but abandons I101 minimal bytecode and PAYG | 2 |
| D | Rebuild/unfold semantic bytecode upon debugger attachment | Changes semantic stream/root topology; invalidates identity/reparse assumptions | 1 |
| E | Synthetic debugger callback/frames for eliminated work | Guest events never executed; invented scopes, correctness risk | 1 |

Retain PLAT058's one canonical lowerer, proof-based minimum emission, handwritten Builder oracle, deterministic reparse and Context isolation. **Proposed narrow supersession:** remove PLAT058's immutable per-Context NORMAL/DEBUG physical **emission profiles** and PERF032's fixed guest execution mode. Preserve PLAT065/I111 single generator, selective RootTag, PLAT044 lazy Error trace, PLAT060 B5 ABI, D013 ordinary call-slot semantics, and other unrelated architecture invariants. I101's optimized physical stream becomes the one default semantic stream. Proposed GITHUB021 supersession is **not a ratified decision** until owner approves exact B.

## 6. Separate D208 semantic/public contract

Current [MODULES.md §73@27cb31d](https://github.com/guillermomolina/protos/blob/27cb31d20e517d8377e58963ea595e5ec16bfd15/spec/semantics/MODULES.md) promises exactly **five** frozen own String slots in `std:runtime.info()`, including `executionMode = NORMAL|DEBUG`. This contract cannot be silently deleted or set to fake constant NORMAL.

[D208/#934](https://github.com/guillermomolina/protos/issues/934) proposes option A: remove `executionMode` and return exactly **four** own String slots (other four retained with full freshness/immutability/authority guarantees); retire language `protos.Debug` option while retaining a truthful **host DAP** service and its CLI endpoint/readiness. Alternatives (constant compatibility field, new debugger field, status quo) and trade-offs are on D208. Do not edit normative `spec/` or change the public API before explicit D208 choice and publication.

## 7. Implementation plan and acceptance (I116, BLOCKED)

**I116-A: one full coherent physical-unification publication.** Remove enum/profile propagation and Context mode selection from host/compiler/modules/Closure planning. Retain every former NORMAL optimizer path independently of tooling. Change all dependent tests/fixtures, combine any D208-approved public spec/metadata changes in one coherent patch, and preserve DAP Engine construction. Keep independent I101 oracles. One integrated human `make test` after all substantive edits; no version change until green.

**I116-B: targeted debugger/tag/source hardening.** Compare default semantic instruction signatures and root topology against source-only and instrumented reparses; preserve index update, source sections and real Activation scopes. Where a proven-fold path omits statement/source tags needed for an observable **surviving** expression, place passive builder tag/source boundaries without adding a semantic operation or fake child root. Verify DAP after target preparation and first execution, true lexical capture, ReturnHome, exception/ensure, Task/C-prime, and Native Image where applicable. If A already satisfies all B acceptance, do not produce redundant changes.

Test owners: `ProtosI101ProfileSpecializedLoweringTest`, `ProtosPerf032ExecutionModeTest`, `ProtosRuntimeMetadataEmbeddingTest`, `ProtosI101BLiteralOneTest`, `ProtosI101BArithmeticBuilderOracleTest`, `ProtosI101BLocalBindingBuilderOracleTest`, `ProtosI101CIndependentBuilderReferenceTest`, `ProtosI101CComparisonBuilderOracleTest`, `ProtosPerf006B5BSourceInstrumentationLocationTest`, `ProtosPerf006B5ABytecodeDebuggerScopeTest`, `ProtosI026EDebuggerIntegrationTest`, `ProtosLm009DPublicDebugCliTest`, `ProtosI107ContinuationEmissionTest`, `ProtosI112TerminalCallBudgetTest`, `ProtosI112TailContinuationTest`, `ProtosI115ARuntimeAdmissionTest`, `ProtosI115BDeferredCallerTest` plus lexical/Task/Actor/IO and native smoke as affected.

**Hard STOP:** no unratified D208 spec edit, no dual semantic streams, no second Bytecode interpreter, no extra required success-path context materialization, no phantom RootTag/frame, no reduction of I101 oracle without specific approval, no incorrectly rebased BytecodeLocation, no loss of cancellation/ReturnHome or C-prime suspension semantics.

## 8. Agent/human executor boundaries

In `guillermomolina/protos` the agent **edits** and the human executes builds/tests/Git and product publication. Follow `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `src/AGENTS.md`, `spec/AGENTS.md`. No subagents without human request. No `git fetch/pull/reset/checkout`; local product repo is already on current HEAD. Use `grep`, not `rg`; Python patch scripts, not unified diffs; no `set`, `pipefail` or `exit` in command snippets. After tests are green, modify `pom.xml` and `CHANGELOG.md` only immediately before human add/commit/push; **no tests afterward**. The project-docs evidence publication is separately authorized by owner and does not authorize direct product repo Git writes. Exact product revision precedes revision-coupled closure records.

### Pending approvals, not inferred

- PLAT067 **B**: one optimized semantic stream, native tools on demand; precise optimized-away stepping limit; monotonic reparse; DAP host not guest mode.
- D208 **A** (or owner's actual selection): change public 5-slot runtime schema and remove language `protos.Debug` option.
- Only after exact approval and durable ratification can I116 transition from blocked to implementation-ready. This research document is **evidence**, not the ratified `docs/project/decisions/platform/PLAT067_*.md` authority.

**Status at research publication:** `OWNER_APPROVAL=NOT_YET`; `PRODUCT_EDIT=NO`; `PRODUCT_TESTS=NOT_RUN`; `RECOMMENDATION=PLAT067-B`; `D208_PUBLIC_OPTION=A_RECOMMENDED_NOT_SELECTED`; `I116=BLOCKED`.
