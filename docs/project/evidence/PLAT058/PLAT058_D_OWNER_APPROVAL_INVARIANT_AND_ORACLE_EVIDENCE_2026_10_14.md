# PLAT058-D — Owner approval, invariant preservation, prior art, and Builder-oracle falsification

**Date:** 2026-10-14.
**Decision:** [PLAT058-D ratification](../../decisions/platform/PLAT058_PROFILE_SPECIALIZED_MINIMAL_LOWERING_AND_BUILDER_ORACLES.md).
**Issue:** [PLAT058/#891](https://github.com/guillermomolina/protos/issues/891); **implementation:** [I101/#892](https://github.com/guillermomolina/protos/issues/892).
**Observed product source:** [`guillermomolina/protos@1b4cc76f4fdfb4eae7ac14d156a10b34cdabc5c0`](https://github.com/guillermomolina/protos/commit/1b4cc76f4fdfb4eae7ac14d156a10b34cdabc5c0), read-only source reference.
**Verification boundary:** Repo files/API and upstream implementations were inspected read-only. No builds/tests/benchmarks, no product changes. The primary handwritten Builder test was **provided by the project owner in conversation and reported PASS**; it was not found as a committed `ProtosI095AggressiveConstantFoldTest` in current product search and was not executed by the coordinating agent.

## Owner authorization provenance and GITHUB021 delta check

The active project owner explicitly said **"Aprobada"** after receiving the exact PLAT058-D architecture and its strengthened acceptance gates. The immediately preceding exchange made two constraints explicit:

1. **NORMAL is allowed to have a smaller, distinct physical execution representation.** The `protos.Debug` flag exists to distinguish this behavior. Do not elevate historically possible NORMAL breakpoint suspension into a required late-attach debugger contract; DEBUG, including attaching a debugger late **inside an already-DEBUG Context**, must retain accurate observability.
2. **Graal is NOT the compiler optimizer responsible for fixing avoidable Protos lowering.** `f(Protos)` must produce a minimum proved Builder representation `B` **before** `g(Truffle/Graal)` operates. Lowerer optimizations need handwritten independent Builder oracles and full physical instruction equivalence, not outcome-only, substring instruction assertions or a favorable compiled graph.

The user provided an explicit Java test source `ProtosI095AggressiveConstantFoldTest` showing the expectation:

- Handwritten `ProtosSemanticBytecodeRootNodeGen.create` directly emits a literal-one root, with `LoadConstant(1L)` inside `MaterializeReturnCarrier`, `Return`, source span and standard tags.
- Real Protos source `() => { 1 }` and `() => { true.ifTrue() { 1 } }` are compiled separately through `ProtosLanguageContext.current().parsePublic(source).call(module)`; the real Closure `bytecodeActivationRootForTesting()` is inspected.
- `root.getRootNodes().ensureComplete()`; `getInstructionsAsList()`; compare *entire* `Instruction.getName()` and `CONSTANT` arguments, including a stable numeric value signature. No filtering or discarded fallback instructions.
- Owner reports the test passes for a **prototype patch**. The provided test compares root instruction streams; it does **not** directly compare every root/group, root identity, continuation metadata, DEBUG behavior or guest return/exception semantics.

The associated prototype lowerer patch added `normalModeAtLowering`, initialized from `ProtosLanguageContext.currentIfEnteredForRuntime()`, and `foldNormalLiteralBooleanResult` replacing a one-expression `CanonicalSequence` with the selected callback literal, plus `foldedLiteralControl || provesNonSuspendingBody(...)`. These are **demonstration implementation details**, not ratified permanent mechanisms. The user rejected `normalModeAtLowering` as a hack. The ratified design preserves the **observed structural reduction** while replacing the implicit Context access and suspension-proof bypass with explicit immutable profile and sound reusable proof.

**GITHUB021 invariant reconciliation (candidate D vs earlier owner corrections):**

| Recorded owner constraint | D treatment | Result |
|---|---|---|
| NORMAL/DEBUG is intentionally an immutable execution-mode selector, not just a debugger on/off flag | Mode-specific emission fixed at Context/compiler ownership | PASS |
| NORMAL need not support debugger scopes nor a late conversion to DEBUG | No such requirement; DEBUG preserves late attachment within DEBUG | PASS |
| Graal is *not* credited with eliminating unnecessary Protos nodes | Expected Builder executable structure created by Protos; graph after Graal only secondary | PASS |
| Builder-oracle physical equality is required for claimed optimization | Independent handwritten oracle + complete instructions and constants; group topology separately | PASS |
| No individual method-by-method test-only tricks | Shared proof classes; fallback when not proven; no test-name folding | PASS |
| D013/overrides/Closure identity, D179 presence, selected arguments, ReturnHome, Tasks/DEBUG fidelity | Ratification explicitly preserves normative contracts and existing PLAT057 boundary | PASS |
| Earlier discussion suggested changing canonical sequence to literal and OR'ing suspension proof | Owner accepted the resulting physical bytecode, *not* the constructor-time mode lookup or bypass | Superseded by explicit proof-backed emission |

**New consequences surfaced:** Profiling or instrumentation *within a fixed DEBUG profile* may add metadata via reparse, but cannot switch the semantic mode; separate physical NORMAL and DEBUG representations are intentionally allowed. Claims that fully eliminated literal callback roots require **additional group topology tests**; only the activation root instruction stream was compared in the owner's reported test. These do not contradict approved invariants.

## Protos source-level findings (current referenced baseline)

- [`ProtosLanguage.java`](https://github.com/guillermomolina/protos/blob/1b4cc76f4fdfb4eae7ac14d156a10b34cdabc5c0/src/main/java/com/guillermomolina/protos/execution/ProtosLanguage.java) declares experimental `protos.Debug` OptionKey, default false, creates `ProtosLanguageContext`; `parse` currently passes **no profile** to `ProtosSourceCompiler`.
- [`ProtosLanguageContext.java`](https://github.com/guillermomolina/protos/blob/1b4cc76f4fdfb4eae7ac14d156a10b34cdabc5c0/src/main/java/com/guillermomolina/protos/execution/ProtosLanguageContext.java) freezes `executionMode` from options on Context construction. Context-local `sharedBytecodeExecutionPlans` rebuilds executable projections from definitions/templates in destination Context via `rebuildBytecodeForLanguage`.
- [`ProtosSourceCompiler.java`](https://github.com/guillermomolina/protos/blob/1b4cc76f4fdfb4eae7ac14d156a10b34cdabc5c0/src/main/java/com/guillermomolina/protos/execution/ProtosSourceCompiler.java) calls `new CanonicalToBytecodeLowerer(language, source)`; [`ProtosModuleRuntime.java`](https://github.com/guillermomolina/protos/blob/1b4cc76f4fdfb4eae7ac14d156a10b34cdabc5c0/src/main/java/com/guillermomolina/protos/execution/ProtosModuleRuntime.java) also calls `compileBytecode`. Closure plans additionally construct independent lowerers through [`ProtosBytecodeClosureExecutionPlan.java`](https://github.com/guillermomolina/protos/blob/1b4cc76f4fdfb4eae7ac14d156a10b34cdabc5c0/src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeClosureExecutionPlan.java).
- [`CanonicalToBytecodeLowerer.java`](https://github.com/guillermomolina/protos/blob/1b4cc76f4fdfb4eae7ac14d156a10b34cdabc5c0/src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java) emits `IsNormalExecutionMode` conditional and ordinary fallback for some admitted literal canonical Boolean sends; its `normalBooleanControlSite` consults `ProtosObjectValue.rootObject().isFrozen()`. A retained parser must **not** re-decide a different shape after root publication; proof snapshot must be stable.
- [`ProtosSemanticBytecodeRootNode.java`](https://github.com/guillermomolina/protos/blob/1b4cc76f4fdfb4eae7ac14d156a10b34cdabc5c0/src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java) uses `@GenerateBytecode(enableTagInstrumentation=true,enableRootTagging=true,...)`; generation support for tags is **not** evidence of mandatory runtime costs for disabled tags.
- The [`ProtosI096InlineClosureLiteralCallTest`](https://github.com/guillermomolina/protos/blob/1b4cc76f4fdfb4eae7ac14d156a10b34cdabc5c0/src/test/java/com/guillermomolina/protos/execution/ProtosI096InlineClosureLiteralCallTest.java) includes current NORMAL/DEBUG topology and reparse order checks, but many existing proofs inspect **instruction names and regions**, not independent Builder equality. [`ProtosPerf013SliceASharedLexicalRootGroupingTest`](https://github.com/guillermomolina/protos/blob/1b4cc76f4fdfb4eae7ac14d156a10b34cdabc5c0/src/test/java/com/guillermomolina/protos/execution/ProtosPerf013SliceASharedLexicalRootGroupingTest.java) establishes same-group closure/arity-root invariants.
- [`ProtosCoreBootstrap.publishStandardRoot`](https://github.com/guillermomolina/protos/blob/1b4cc76f4fdfb4eae7ac14d156a10b34cdabc5c0/src/main/java/com/guillermomolina/protos/execution/ProtosCoreBootstrap.java) explicitly freezes and validates standard Object publication, including frozen canonical Boolean behavior and `Object.call`. Do not infer this publication state before it is actually established.

## Authoritative upstream implementation survey

1. [GraalJS `JSAddNode.create`](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/binary/JSAddNode.java) returns an immediate constant node for safe constant integer addition, or creates a one-constant specialized node when applicable. The broad JS semantics are not directly transferrable to Protos's overridable ordinary numeric sends.
2. [GraalJS `IfNode.create`](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/control/IfNode.java) converts `!a` to `a` with swapped branches during node construction; no appeal to Graal JIT is necessary for that AST transformation.
3. [GraalJS `JSLeftShiftConstantNode`](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/binary/JSLeftShiftConstantNode.java) retains compact constant operand during normal AST construction; `materializeInstrumentableNodes(BinaryOperationTag)` reconstructs a generic tagged binary node to expose instrumentation. [`JSAddConstantRightNumberNode`](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/binary/JSAddConstantRightNumberNode.java) uses a similar pattern.
4. [GraalPy `RootNodeCompiler`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) emits tracing hooks. [`PBytecodeDSLRootNode`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java) declares those as `@Instrumentation`, enabled with a tracing `BytecodeConfig` through `BYTECODE.update`; disabled hooks do not produce executable instrumentation instructions.
5. [TruffleRuby `LiteralMethodDefinitionNode`](https://github.com/truffleruby/truffleruby/blob/master/src/main/java/org/truffleruby/language/methods/LiteralMethodDefinitionNode.java) forces method CallTarget materialization on instrument request; [SimpleLanguage `SLBytecodeParser`](https://github.com/oracle/graal/blob/master/truffle/src/com.oracle.truffle.sl/src/com/oracle/truffle/sl/parser/SLBytecodeParser.java) uses the same parser with `BytecodeConfig.WITH_SOURCE` or `COMPLETE`.
6. [Truffle `BytecodeParser` javadoc](https://github.com/oracle/graal/blob/master/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/BytecodeParser.java) requires repeatable/idempotent parsing; [`BytecodeRootNodes`](https://github.com/oracle/graal/blob/master/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/BytecodeRootNodes.java) documents expensive all-group reparsing/invalidation on instrumentation; [`Instrumentation`](https://github.com/oracle/graal/blob/master/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/Instrumentation.java) requires transparent stack effects. The [Bytecode DSL guide](https://github.com/oracle/graal/blob/master/truffle/docs/bytecode_dsl/UserGuide.md) confirms instrumentation is absent until activated and reparses add, rather than remove, metadata/instructions.

The evidence strongly supports early minimal AST/DSL generation and optional instrumentation. It does **not** prove numeric equivalence of raw GraalJS/GraalPy/Protos graphs; counters and implementation models differ. Espresso/Sulong/Pkl/debugger implementations were considered as secondary tooling precedents, not as more directly comparable proof of fixed-profile Bytecode DSL lowering.

## GITHUB010 comparative candidate matrix — 12 common axes

Scoring uses **1 (poor) to 5 (strong)** for the PLAT058 problem. **A** current runtime NORMAL switch plus Graal cleanup; **B** same lowerer with only optional Truffle instrumentation; **C** duplicate fully independent NORMAL and DEBUG lowerers; **D** single proof-driven lowerer with immutable physical profile and Builder oracles; **E** dynamic NORMAL→DEBUG re-lowering on attachment. H/M/L = confidence, not approval.

| Axis | A | B | C | D (selected) | E |
|---|---|---|---|---|---|
| 1 Correctness | 4 H | 4 H | 2 M | 4 M | 2 L |
| 2 Protos alignment | 2 H | 3 H | 2 M | 5 M | 1 H |
| 3 Present need/pay-as-you-grow | 2 H | 3 H | 2 H | 5 M | 1 H |
| 4 Incremental growth | 2 M | 3 M | 2 M | 5 M | 2 L |
| 5 Future-option resilience | 3 M | 3 M | 2 M | 4 M | 3 L |
| 6 Scalability | 3 M | 4 M | 2 M | 4 M | 2 L |
| 7 Conceptual simplicity | 4 H | 4 H | 1 H | 3 M | 1 H |
| 8 Portability | 4 M | 4 H | 3 M | 4 M | 2 L |
| 9 Runtime/resource cost | 2 H | 3 H | 4 L | 5 M | 2 L |
| 10 Failure/operability | 3 M | 4 M | 2 M | 4 M | 2 L |
| 11 Deferral/migration reversibility | 2 M | 3 M | 2 M | 4 M | 1 L |
| 12 Evidence maturity/implementation risk | 4 H | 5 H | 3 M | 3 M | 2 L |

**Per-axis justification:**

1. **Correctness:** A/B reuse proven execution; C risks two semantic interpreters drifting; D reuses proof and fallback but must validate compilation/reparse invariants; E risks active-frame/continuation mismatch.
2. **Alignment:** A depends on downstream cleanup, B solves only tracing, C duplicates rules, D places physical minimality in Protos while respecting ordinary messages; E invents unwanted late-debug conversion.
3. **Present cost:** A creates unnecessary structures, B removes pure probes but not sends, C creates maintenance duplicates, D has only proof/profile costs at compilation, E adds mode-transition machinery.
4. **Growth:** A/B cannot generalize physical folding, C forks every extension, D admits additional shapes through proof/Builder tests, E requires on-stack migration infrastructure.
5. **Future options:** A/B leave code-gen debt, C locks two emission models, D reuses shared canonical/proof and has clear fallback; E ties model to invasive runtime conversion.
6. **Scalability:** A/B dynamic overhead persists; C doubles implementation/caches; D bounds proof expansion and per-Context state; E multiplies transition/invalidation interactions.
7. **Simplicity:** A/B easiest short term, C and E largest conceptual surface, D adds an explicit carrier and emission decisions but avoids parallel runtime.
8. **Portability:** A/B/D preserve Protos semantics across future backends; C demands duplication and E assumes Truffle OSR/interpreter details.
9. **Runtime cost:** A/B pay for dynamic paths; C might produce compact roots but unproven operational overhead; D physically omits proved unnecessary instructions; E retains instrumentation/transition costs.
10. **Failures:** B framework-managed; A ordinary; D explicit fallback and deterministic reparse provide isolation; C drift and E frame migration fail less predictably.
11. **Migration:** Deferring D retains repeated structural debt; D can add proofs incrementally, retaining existing fallback; C/E require fundamental rewrites if assumptions break.
12. **Evidence:** Existing A/B and upstream `@Instrumentation` are directly observable; D has owner-reported prototype proof but requires product regression/Builder tests; C/E have no local acceptance.

**Non-compensating red flags:** C/E are overengineered for a fixed Context mode, and A/B underengineered against the owner's *present* bytecode-minimality requirement. D must stay small: if it grows a full intermediate representation, automatic observer rematerializer or profile-guided universal inliner, stop and seek separate approval.

## Falsification cases and delivery gates

| Case | Must prove / rejection signal |
|---|---|
| Owner literal example in NORMAL | Complete signature identical to independent Builder literal oracle; no `IsNormalExecutionMode`/send/Closure call/fallback in admitted root |
| Group topology | Necessary real Closure/arity roots preserved; eliminated roots not counted as eliminated merely because main root is compact |
| Source reparse / `ensureComplete()` | Same physical semantic path, root count/order and identity, irrespective of frozen Object state changes |
| DEBUG late attach within DEBUG | Exact scopes/RootTags/source/stack and callback execution |
| NORMAL + DEBUG simultaneous | Distinct context-owned effective profiles and projections; no cross-context reuse of wrong root |
| Module / cross-context Closure rebuild | Profile of target Context; no source-profile authority leak |
| Mutable selector/`call` | Ordinary selected behavior or exact sound guard/fallback, no false constant fold |
| Captures/aliases/D179 | Presence, removal/recreation, same Context identity, write pre-RHS destination and escape exact |
| Error/NLR/suspension/ensure | No missing failure or duplicated side effect, task continuation only where required |
| Dynamic guarded specialization | Independent Builder expected guard/fallback; full signature matches, miss path correct |
| Native/JIT comparisons | Secondary evidence only, not a substitute for Builder result |

**Smallest sufficient present solution:** a typed Context-fixed profile, stable proof-backed emission decisions and a reusable Builder-oracle test utility; first supported physical fold and negative cases. Do **not** build full generalized optimizer architecture, tool-agnostic virtualized debugger or runtime mode migrator today.

**If omitted:** fixes continue to leak runtime-mode branches/dispatch into physical bytecode, and every subsequent optimized selector would need independent workarounds. **Escape path:** new proof families can be added to D later without altering the profile boundary or observable language model. **Strongest argument against D:** an overbuilt emission-plan IR could impose high compilation/startup/memory and maintenance costs; constrain the first implementation to local immutable decisions proven necessary and evaluate expansion against observed tests.

## Decision / publication boundary

Approval is **architecture only**. The product still requires I101 code/test patches and **human-executed** validation/publication. The conversation-reported PASS of the illustrative test is evidence of feasibility, not a released implementation; benchmark graph/steady-state claims are **not verified**. Owner authorization allows agent-direct commit/push only for this bounded project-docs record, never product changes.
