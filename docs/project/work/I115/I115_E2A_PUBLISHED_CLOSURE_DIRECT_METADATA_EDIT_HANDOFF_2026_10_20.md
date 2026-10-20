# I115-E2-A — published Closure direct-metadata edit-only handoff

Date: 2026-10-20
Status: **owner-requested implementation handoff prepared; no production edit, test result, or Tier-1 comparison has been reported**.
Repository to edit: `guillermomolina/protos`.
Source baseline inspected: public `main` at `e2200088d42d0d8be90626a79b373ddad64978f5`. The human's local HEAD, including any E1 work, is authoritative; do not checkout/reset.
Related: [I115/#931](https://github.com/guillermomolina/protos/issues/931) (historically closed for C/D), [PLAT066-C/#930](https://github.com/guillermomolina/protos/issues/930), [PERF042/#871](https://github.com/guillermomolina/protos/issues/871).
Source evidence: [I115-E2-A source comparison](../../evidence/I115/I115_E2A_SOURCE_COMPARISON_AND_EDIT_HANDOFF_2026_10_20.md).

## Agent execution boundary

This is a **file-edit-only** implementation. Do not execute any shell, Maven, Python, formatter, graph or Git commands; do not browse the web or other repositories; do not inspect unrelated code. Edit exactly three files below, preserving all unrelated/concurrent modifications and license headers. No version/changelog changes, no generated files, no `@Operation` addition, no tests executed by agent, no issue mutation.

## 1. Edit ProtosActivation.java

File: `src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java`.

Keep existing public `Optional<ProtosReturnHome> returnHome()`, `Optional<ProtosObjectValue> methodHome()` and `Optional<ProtosPrelude> prelude()` untouched. Keep existing `preludeOrNullForRuntime()` untouched.

Immediately after `returnHome()`, add:

```java
/** Nullable backend metadata accessor, equivalent to returnHome().orElse(null). */
public ProtosReturnHome returnHomeOrNullForRuntime() {
    return returnHome;
}
```

Immediately after `methodHome()`, add:

```java
/** Nullable backend metadata accessor, equivalent to methodHome().orElse(null). */
public ProtosObjectValue methodHomeOrNullForRuntime() {
    return methodHome;
}
```

No other changes in this file: no caching, allocation, annotation, synchronization, constructor or Activation-state modification.

## 2. Edit ProtosBytecodeRootNode.java

File: `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java`.

Only in `MaterializeCurrentClosure.published`, replace the existing `MaterializeClosure.perform(...)` delegation with the following body, preserving its `@Specialization(guards = "hasPublishedActivation(frame)")` and signature:

```java
ProtosActivation activation =
        ProtosFrameArguments.publishedActivation(frame.getArguments());

return new ProtosClosureValue(
        definition,
        activation.lexicalEnvironmentForClosureCapture(),
        activation.receiver(),
        activation.methodHomeOrNullForRuntime(),
        activation.returnHomeOrNullForRuntime(),
        activation.preludeOrNullForRuntime(),
        executionPlanCell.plan());
```

The published activation and exact existing environment must be used; the constructed Closure is fresh. Do not change `MaterializeClosure.perform`, `MaterializeCurrentClosure.compact`, `MaterializeCurrentClosure.rich`, its guards, `hasPublishedActivation`, `EnterRoot`, `InvokeClosureOneTail`, or other operations. Do not change the lowerer, its `useCompactClosureCapture()` admission, or the B5/B5+ header. Do not invent additional helper/factory layers, `@TruffleBoundary`, or `@InliningCutoff`.

## 3. Edit ProtosPerf025LazyLexicalCaptureTest.java

File: `src/test/java/com/guillermomolina/protos/execution/compiler/captures/ProtosPerf025LazyLexicalCaptureTest.java`.

Retain existing tests and helpers. Add the import:
```java
import com.guillermomolina.protos.runtime.control.ProtosReturnHome;
```

Add the following two `@Test` methods within the class:

```java
@Test
void publishedModuleClosureCreationPreservesCaptureAndMetadata() throws Exception {
    withCore(module -> {
        ProtosClosureValue first =
                parsedClosure("() => 1", "i115-e2a-published-first.protos", module);
        ProtosClosureValue second =
                parsedClosure("() => 2", "i115-e2a-published-second.protos", module);

        assertNotSame(first, second);

        ProtosLexicalEnvironment expectedEnvironment =
                module.lexicalEnvironmentForClosureCapture();

        assertSame(expectedEnvironment, first.capturedLexicalEnvironmentForRuntime());
        assertSame(expectedEnvironment, second.capturedLexicalEnvironmentForRuntime());
        assertSame(module.context(), first.capturedLexicalEnvironmentForRuntime().context());
        assertSame(module.receiver(), first.capturedReceiver());
        assertEquals(module.methodHome(), first.methodHome());
        assertEquals(module.returnHome(), first.returnHome());
        assertEquals(module.prelude(), first.prelude());
        assertEquals("1", integerValue(invokeDirect(first, module).result()));
        assertEquals("2", integerValue(invokeDirect(second, module).result()));
    });
}

@Test
void nullableClosureMetadataAccessorsPreserveExactValues() throws Exception {
    withCore(module -> {
        assertNull(module.methodHomeOrNullForRuntime());
        assertNull(module.returnHomeOrNullForRuntime());
        assertSame(module.prelude().orElseThrow(), module.preludeOrNullForRuntime());

        ProtosObjectValue methodHome =
                new ProtosObjectValue(ProtosObjectValue.rootObject());
        ProtosActivation withMethodHome =
                ProtosActivation.withMethodHome(
                        module.context(), List.of(), module.receiver(), methodHome);
        assertSame(methodHome, withMethodHome.methodHomeOrNullForRuntime());
        assertSame(
                withMethodHome.methodHome().orElseThrow(),
                withMethodHome.methodHomeOrNullForRuntime());

        ProtosReturnHome returnHome = new ProtosReturnHome();
        ProtosActivation withReturnHome =
                ProtosActivation.withReturnHome(
                        module.context(), List.of(), module.receiver(), returnHome);
        assertSame(returnHome, withReturnHome.returnHomeOrNullForRuntime());
        assertSame(
                withReturnHome.returnHome().orElseThrow(),
                withReturnHome.returnHomeOrNullForRuntime());
    });
}
```

The test file already has `withCore`, `parsedClosure`, `invokeDirect`, `integerValue`, `List`, `ProtosLexicalEnvironment` and the required JUnit assertions. Do not create a new test fixture or broaden the test surface.

## Semantic constraints

Fresh Closure object identity and the exact `ProtosLexicalEnvironment` instance, capture by reference, same real Context identity, D179 membership/mutation, receiver and methodHome, ReturnHome including nonlocal return, Prelude, execution-plan identity, Task/suspension, D013 and the one-interpreter invariant are unchanged. `ProtosClosureValue` constructor must remain the sole constructor; no new authority or premature Context materialization. Do not alter an unrelated operation to shift graph nodes.

## Human-only validation (not to be executed by the agent)

Human performs targeted tests for the two new methods, impacted capture semantics, PERF042 real bare identity and I112 11+5 FULL census; **no `make test` or full test suite in this incremental diagnostic slice**. Capture both `ProtosPerf042IdentityBuilderOracleTest#captureCompleteRunRootAtRequestedTier` and `#captureIdentityRootAtRequestedTier` at Tier 1 `After TruffleTier`. Compare whole module/identity graph totals, `MaterializeCurrentClosure` Java-origin attribution and `Optional.ofNullable`/allocations; do not call attribution a marginal cost or assert a 31-node reduction.

The old pre-E1 graph checkpoint was module **645** and isolated identity **263** (sum **908**) versus GraalPy Native **469+110**, *not necessarily current E2-A baseline*. Use measured post-E1 counts if available; require exact current HEAD SHA. No metadata edits, commit, push or issue-state transition is part of this editor task.

## Agent final report

Return the exact three paths edited, `I115_E2A_PRODUCTION_FILES=2`, `I115_E2A_TEST_FILES=1`, `I115_E2A_NEW_OPERATIONS=0`, `I115_E2A_TESTS_EXECUTED=NO`, `I115_E2A_GRAPHS_CAPTURED=NO`, and only concrete edit blockers.
