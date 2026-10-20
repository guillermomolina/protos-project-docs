# I115-E2-A — source-backed published Closure creation research and owner edit handoff

Date: 2026-10-20
Status: **research and exact file-edit instructions published**, **no E2-A implementation/validation claim**.
Genuine formal owner: historical [I115/#931](https://github.com/guillermomolina/protos/issues/931), currently closed following I115-C/D; physical-gap follow-up [PLAT066-C/#930](https://github.com/guillermomolina/protos/issues/930), parent [PLAT066/#924](https://github.com/guillermomolina/protos/issues/924), measurement [PERF042/#871](https://github.com/guillermomolina/protos/issues/871).
Verified public product HEAD at research publication: [`e2200088d42d0d8be90626a79b373ddad64978f5`](https://github.com/guillermomolina/protos/commit/e2200088d42d0d8be90626a79b373ddad64978f5). **Not a claim about an uninspected human local checkout.**
Edit-only implementation handoff: [I115_E2A_PUBLISHED_CLOSURE_DIRECT_METADATA_EDIT_HANDOFF_2026_10_20.md](../../work/I115/I115_E2A_PUBLISHED_CLOSURE_DIRECT_METADATA_EDIT_HANDOFF_2026_10_20.md).
Prior independent graph evidence: [PERF042_I115_E1_TWO_ROOT_TIER1_IR_COMPARISON_2026_10_20.md](../PERF042/PERF042_I115_E1_TWO_ROOT_TIER1_IR_COMPARISON_2026_10_20.md).

## 1. Precise source mechanism

In `ProtosBytecodeRootNode.MaterializeCurrentClosure`, the `published` specialization is guarded by `hasPublishedActivation(frame)` and currently calls `MaterializeClosure.perform(ProtosFrameArguments.publishedActivation(frame.getArguments()), definition, executionPlanCell)`.

`MaterializeClosure.perform` creates a fresh `ProtosClosureValue` from these existing fields of `ProtosActivation`:

```java
activation.lexicalEnvironmentForClosureCapture()
activation.receiver()
activation.methodHome().orElse(null)
activation.returnHome().orElse(null)
activation.prelude().orElse(null)
executionPlanCell.plan()
```

The last three metadata reads use `Optional` public API wrappers even though the constructor already accepts nullable metadata. `ProtosActivation.preludeOrNullForRuntime()` already exists; `methodHomeOrNullForRuntime` and `returnHomeOrNullForRuntime` do not exist at current public HEAD. `ProtosClosureValue` already takes exactly these fields and preserves a shared `ProtosLexicalEnvironment` instance. No new representation is required.

The top-level module has an already-published activation: `ProtosPrelude.newModuleActivation()` supplies a real `moduleContext`. The existing `CanonicalToBytecodeLowerer.useCompactClosureCapture()` requires `currentActivationLocal == null && currentRootFrameLocals.isEmpty()`. The `identity` module declares a local binding, so the compact specialization is not its admitted case; `published`, not `rich`, is the source-observed route. This is an origin/source observation and does not prove exact runtime branch frequencies.

## 2. Proposed edit — isolated E2-A

Change exactly two production files:
1. `src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java`: add pure nullable getters for `methodHome` and `returnHome`, retaining the Optional methods and existing `preludeOrNullForRuntime()`.
2. `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java`: only replace `MaterializeCurrentClosure.published` body, obtaining the exact published activation once and directly calling existing `new ProtosClosureValue(definition, activation.lexicalEnvironmentForClosureCapture(), activation.receiver(), activation.methodHomeOrNullForRuntime(), activation.returnHomeOrNullForRuntime(), activation.preludeOrNullForRuntime(), executionPlanCell.plan())`. Keep all other specializations and `MaterializeClosure.perform` untouched.

Change one test file:
3. `src/test/java/com/guillermomolina/protos/execution/compiler/captures/ProtosPerf025LazyLexicalCaptureTest.java`: add an already-published module Closure regression checking fresh identity/shared lexical-environment reference/exact receiver, metadata and callable result; add a pure nullable-metadata getter regression covering null and non-null ReturnHome/methodHome, using existing helpers.

No `@Operation` or instruction changes, no lowerer changes, no `ProtosFrameArguments`/B5 modifications, no new builder/plan factory/authority. Preserve D013, D179, Context/capture identity, non-local ReturnHome, Task and debugger behavior.

The previous estimate of **31 Optional-related source-attributed nodes** in `MaterializeCurrentClosure` is attribution, **not** a guaranteed 31-node marginal reduction. The total **122** source-attributed nodes is also not an independently removable budget. Other `MaterializeClosure.perform` callers still use Optionals by design.

## 3. Cross-language source comparison — architecture only

- [GraalPy `PBytecodeDSLRootNode.MakeFunction`](https://github.com/oracle/graalpython/blob/graal-25.4.4/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java): `functionSingleContext` and `functionMultiContext` ultimately call `createFunction`, using direct code-unit, globals and closure-cell data via `PFactory.createFunction`; no sequence of nullable `Optional` metadata adapters for the published parent Activation.
- [GraalJS `JSFunctionExpressionNode`](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/function/JSFunctionExpressionNode.java): `AutonomousFunctionExpressionNode` creates a function without parent frame; `ClosureFunctionExpressionNode` preserves/materializes its captured frame as needed. It does **not** license Protos to erase genuine Context capture.
- [TruffleRuby `BlockDefinitionNode`](https://github.com/truffleruby/truffleruby/blob/master/src/main/java/org/truffleruby/language/methods/BlockDefinitionNode.java): creates a RubyProc with a `frame.materialize()` declaration frame and explicit metadata; no suggestion to remove that required storage in Protos's E2-A.

These systems have different semantics. The reusable pattern is **direct construction from already-known runtime values**, not copying each language's closure storage layout. This is a source analogy, not a matched cross-runtime node count or measured performance result.

## 4. Semantic authority and validation

- `spec/semantics/CALLABLES.md`: Closure capture of genuine lexical execution contexts is **by reference**, not a copied value. Fresh Closure identity must be preserved.
- `spec/semantics/EXECUTION_AND_CONTROL.md`: `Context` object identity, lexical-parent graph, D179 PRESENT/ABSENT including PRESENT(null), late creation/removal/rebinding must remain observationally identical.
- `ProtosActivation.lexicalEnvironmentForClosureCapture()` caches one lexical environment per activation (except object-construction passthrough). Do not replace or eagerly construct it.
- `ProtosPerf025LazyLexicalCaptureTest` already exercises reference identity, Context observations and D179; add two narrow E2-A tests.
- `ProtosI112TerminalCallBudgetTest.normalIdentityMeetsTheTerminalFullInstructionTarget` locks the **11 + 5 = 16** FULL DSL instruction shape.
- `ProtosPerf042IdentityBuilderOracleTest` supplies both real root Tier-1 BGV capture methods.

The owner explicitly requested **editing-only Claude prompt**, followed by **human-only focal tests and Tier-1 node comparison**, **no `make test`** for the incremental slice. Human local checkout may include I115-E1; keep those concurrent edits. Do not modify `pom.xml` or `CHANGELOG.md`, do not publish Protos source, and do not change Issue state in this documentation task.

## 5. Physical checkpoints, not acceptance

Historical two-root `After TruffleTier` Protos JVM totals **645 module**, **263 isolated identity**, descriptive sum **908**; GraalPy Native reference **469+110=579**. E1 or local source changes may make these old baselines inapplicable to E2-A. The source attribution **122** (`MaterializeCurrentClosure` in module) and **31** (`Optional` paths) does not justify a savings percentage.

**No E2-A files have been edited by this coordination task, no focal tests have been reported, no E2-A Tier-1 capture has been reported, and no post-E2-A node reduction can be asserted.** Report acceptance only after exact human HEAD SHA + focal tests + 11/5 FULL census + paired Tier-1 BGV/module/Closure counts and per-operation source-ancestry classification.

## 6. Follow-on gate

E2-B (possible specialization of `ProtosActivation.lexicalEnvironmentForClosureCapture()`) is **not approved as an edit in E2-A**. Retain its per-activation cache and Context identity. Reassess only against actual E2-A whole-root BGV and source attribution after E1, without shifting costs to another DSL operation.
