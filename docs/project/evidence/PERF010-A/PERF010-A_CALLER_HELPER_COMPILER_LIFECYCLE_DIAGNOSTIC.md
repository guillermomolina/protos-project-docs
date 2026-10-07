# PERF010-A — caller/helper compiler-lifecycle diagnostic checkpoint

Status: retained diagnostic checkpoint; compiler lifecycle remains unestablished because the required pinned runtime execution was not available in the investigating environment.

## Evidence identity

```text
PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=2b3a88389da7228caed231a90b14091cf2841115

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
BENCHMARK_EVIDENCE_REVISION=1611f9b19005a13723cc9347addcbb74e926290d
HARNESS_REVISION=3275c108aa9b04d35a67fbe9be1c13e6fe94c58a

PRIOR_PROJECT_RECORD_REVISION=512cc4195e8df80073a95e0a9b3c90e6c7993737
PRIOR_PROJECT_RECORD=docs/project/evidence/PERF010-A/PERF010-A_CALLER_HELPER_COMPILATION_BOUNDARY_INVESTIGATION.md

RUNTIME=GraalVM Community 25.3.4.1 / JDK 25.0.4.1 / Truffle 25.3.4.1
```

This checkpoint owns only the exact caller-root identity and the remaining compiler-lifecycle observation. It does not replace the required runtime trace with source-level inference.

## Requested discriminator

The requested investigation was exactly one unchanged BASELINE execution of `micro/method-call`, with compiler queue/completion/failure and inlining diagnostics sufficient to identify the caller semantic root and follow its exact helper CallTarget.

No runtime patch, guarded-call ablation, timing comparison, threshold change, warmup tuning, `continueAt` investigation, version change, or production optimization was authorized.

## Exact caller semantic source identity

At the pinned revision, the caller Closure is:

```protos
() => { sink = receiver.identity(42) }
```

`Canonicalizer.lowerClosure(...)` uses the body `SurfaceSequence.span()` for the `CanonicalSequence`. For a braced closure, `ProtosParser.parseClosureBody(...)` constructs that body span from the first expression start to the final expression end.

Therefore the caller semantic root can be discriminated by the exact source section:

```text
SOURCE=method-call.protos
START_OFFSET=1226
END_OFFSET=1254
LENGTH=28
LINE=29
COLUMN_ONE_BASED=23
TEXT=sink = receiver.identity(42)
```

This identity is derived from the pinned source, not compilation order.

## Exact static semantic/helper pairing

At the same revision, `ProtosBytecodeClosureExecutionPlan` constructs the helper activation root and passes:

```text
activationRoot.getCallTarget()
```

as the exact `helperTarget` to:

```text
ProtosSemanticBytecodeRootNode.wrap(...)
```

The semantic wrapper emits that target as a Bytecode constant and `InvokeSemanticHelper` executes it through:

```java
helperTarget.call(activation)
```

This establishes the construction-time one-to-one pairing needed to correlate the runtime trace. It does **not** establish PE constancy, direct-call recognition, independent compilation, or inlining.

## Diagnostic option availability

The GraalVM JDK 25 Truffle documentation exposes the read-only diagnostic mechanisms needed for the requested execution:

```text
engine.TraceCompilation
engine.TraceCompilationDetails
compiler.TraceInlining
compiler.TraceInliningDetails
engine.CompilationFailureAction=Print
```

The documented compilation trace includes target IDs and source sections; the detailed trace reports queue/start/completion activity; the inlining trace reports guest call-target inlining decisions.

No threshold, tiering, compilation-mode, inlining-budget, or optimization-policy modification is required to obtain the requested observation.

## Execution limitation

The requested runtime observation was not performed in the investigating environment.

The environment did not provide the pinned GraalVM/JDK 25.0.4.1 runtime or a container engine capable of executing the retained harness image locally, and the available GitHub coordination interface did not expose an arbitrary workflow-dispatch mechanism for this diagnostic.

The investigation therefore stops under the explicit "one remaining diagnostic ambiguity" condition.

This limitation must not be reinterpreted as evidence that either root was not compiled.

## Reconciled result

```text
PERF010A_CALLER_COMPILATION_BOUNDARY=NOT_ESTABLISHED

CALLER_SEMANTIC_ROOT_RUNTIME_IDENTIFIED=NO
CALLER_SEMANTIC_ROOT_IDENTITY=
    SOURCE_IDENTITY_ESTABLISHED:
    method-call.protos:[1226,1254)
    line=29
    column=23
    text="sink = receiver.identity(42)"
    RUNTIME_CALL_TARGET_IDENTITY=INCONCLUSIVE

CALLER_HELPER_TARGET_RUNTIME_IDENTIFIED=NO
CALLER_HELPER_TARGET_IDENTITY=
    STATIC_PAIRING_ESTABLISHED:
    activationRoot.getCallTarget()
    passed as helperTarget to semantic wrapper
    RUNTIME_CALL_TARGET_IDENTITY=INCONCLUSIVE

CALLER_SEMANTIC_COMPILATION_REQUESTED=INCONCLUSIVE
CALLER_SEMANTIC_COMPILATION_COMPLETED=INCONCLUSIVE
CALLER_SEMANTIC_COMPILATION_REJECTED=INCONCLUSIVE
CALLER_SEMANTIC_INVALIDATED=INCONCLUSIVE

CALLER_HELPER_COMPILATION_REQUESTED=INCONCLUSIVE
CALLER_HELPER_COMPILATION_COMPLETED=INCONCLUSIVE
CALLER_HELPER_COMPILATION_REJECTED=INCONCLUSIVE
CALLER_HELPER_INVALIDATED=INCONCLUSIVE

CALLER_HELPER_TARGET_PE_CONSTANT=INCONCLUSIVE
CALLER_HELPER_DIRECT_TRUFFLE_CALL=INCONCLUSIVE
CALLER_HELPER_INLINED=INCONCLUSIVE

REJECTION_OR_BOUNDARY_REASON=INCONCLUSIVE

CALLER_COMPILER_TOPOLOGY=H

REPEATED_SEND_WORK_OUTSIDE_OPTIMIZED_REGION=INCONCLUSIVE

MATERIAL_COMMON_PATH_COST_PLAUSIBLE=YES
ORDER_OF_MAGNITUDE_RELEVANCE=PLAUSIBLE

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED

PERF011_REACTIVATION_TRIGGER=NOT_SATISFIED

PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

## Interpretation

The caller helper still owns the repeated ordinary composed-send path, including `PrepareSendArguments` and prepared Closure entry. Therefore, **if** a runtime trace establishes that this helper remains outside optimized code, the candidate still has qualitatively larger scale than the previously measured few-percent local mechanisms.

That conditional remains only a hypothesis. No lifecycle state is inferred from source shape or from the previous BGV capture.

The existing PERF011 reactivation trigger remains unsatisfied because this checkpoint does not establish a material common-path cost caused by runtime representation or compiler visibility.

## Exactly one remaining observation

Run exactly one diagnostic-only BASELINE `micro/method-call` execution at the pinned product revision and unchanged workload, with the diagnostics above, then correlate the semantic target whose source section is:

```text
method-call.protos:[1226,1254)
```

to its exact helper target.

That single trace must determine:

```text
semantic target queued / started / completed / failed / invalidated?
helper target queued / started / completed / failed / invalidated?
helper target compiler-visible at InvokeSemanticHelper?
helper inlined, separately compiled, or left behind a call-target boundary?
```

Only after that observation should PERF010-A decide whether another causal experiment is warranted.
