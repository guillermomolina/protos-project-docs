# PERF010-A — Caller / helper compilation-boundary investigation

Status: retained investigation result; no production optimization selected.

This durable, non-normative record captures the bounded follow-up to the guarded-call compiler-visibility discrimination. It records the source/runtime topology that can be established from the pinned Protos revision, the remaining compiler-lifecycle ambiguity, and the single next diagnostic required to discriminate it.

## Evidence identity

```text
PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=2b3a88389da7228caed231a90b14091cf2841115

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
BENCHMARK_EVIDENCE_REVISION=1611f9b19005a13723cc9347addcbb74e926290d
HARNESS_REVISION=3275c108aa9b04d35a67fbe9be1c13e6fe94c58a

PRIOR_PROJECT_RECORD_REVISION=8bbdbcf0b21fd31fb2bd311d8da0756cb39a658e
PRIOR_PROJECT_RECORD=docs/project/evidence/PERF010-A/PERF010-A_GUARDED_CALL_COMPILER_VISIBILITY_DISCRIMINATION.md

RUNTIME=GraalVM Community 25.3.4.1 / JDK 25.0.4.1 / Truffle 25.3.4.1
```

The prior retained compiler capture established that the only visible compiled `ProtosBytecodeRootNodeGen` helper matches the tiny `identity` callee, while the caller helper containing the repeated composed send is absent from the collected optimized graphs.

This investigation does not reinterpret that absence as proof that the caller helper executes interpreted.

## Question

Determine why the hot caller closure containing:

```protos
sink = receiver.identity(42)
```

does not appear as a compiled or inlined `ProtosBytecodeRootNode` helper in the retained optimized graphs while the small callee does, and determine whether the boundary remains plausibly large enough to matter to PERF010-A.

## Source-established topology

At the pinned revision, each source Closure activation owns two distinct roots/call targets:

```text
guest Closure activation
    |
    v
ProtosSemanticBytecodeRootNode
    |
    | InvokeSemanticHelper
    | helperTarget.call(activation)
    v
ProtosBytecodeRootNode helper
    |
    v
canonical guest operations
```

The relationship is construction-bound, not inferred from graph ordering:

- `ProtosBytecodeClosureExecutionPlan` lowers the `CanonicalClosure` body to a `ProtosBytecodeRootNode activationRoot`;
- it then creates a separate semantic wrapper with `definition.body().span()`;
- the wrapper receives `activationRoot.getCallTarget()` as its exact `helperTarget`;
- the resulting semantic `RootCallTarget` is the Closure activation target exposed for composition.

Relevant pinned source:

```text
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeClosureExecutionPlan.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
```

Therefore:

```text
CALLER_SEMANTIC_ROOT_IDENTIFIED=YES
CALLER_HELPER_TARGET_IDENTIFIED=YES
```

## Semantic/helper call boundary

`ProtosSemanticBytecodeRootNode.wrap(...)` emits the exact helper target into the generated semantic bytecode with `emitLoadConstant(helperTarget)`.

`InvokeSemanticHelper` then invokes that value through:

```java
helperTarget.call(activation)
```

The semantic wrapper does not explicitly construct a `DirectCallNode` for this helper seam.

By contrast, ordinary composed Closure entry in `ProtosBytecodeRootNode.EnterClosureCall` has a target-cached specialization using `DirectCallNode`, with an `IndirectCallNode` fallback.

This establishes a concrete architectural distinction, but does not by itself establish what partial evaluation sees at the semantic/helper call site.

A source-level constant or final object is not promoted here to compiler PE constancy without graph evidence.

Therefore:

```text
CALLER_HELPER_TARGET_PE_CONSTANT=INCONCLUSIVE
CALLER_HELPER_DIRECT_TRUFFLE_CALL=INCONCLUSIVE
```

## Caller work owned by the helper

The caller helper is not a tiny wrapper around the callee.

At the pinned revision, `CanonicalToBytecodeLowerer` lowers the repeated ordinary composed send through `PrepareSendArguments` and then `EnterClosureCall`.

The generic ordinary-send route performs the current authoritative lookup/preparation path, creates the fresh invocation activation and resolves the composed call target before entering the selected Closure activation.

Therefore, if the caller helper really remains outside the optimized region, the potentially stranded repeated work includes caller-side send preparation, lookup, activation preparation and dispatch work on every iteration.

The prior BGV evidence found no `PrepareSendArguments` in any captured graph.

## What the retained compiler evidence proves

The prior record established:

```text
COMPILER_GRAPH_CAPTURE=VALID
CAPTURED_BASELINE_GUARDED_TOPOLOGY=COMPARABLE

GUARDED_SEND_PRESENT_IN_CAPTURED_GRAPHS=NO
PREPARE_SEND_ARGUMENTS_PRESENT_IN_CAPTURED_GRAPHS=NO

CALLER_HELPER_WITH_10000_SENDS_VISIBLE_AS_COMPILED_OR_INLINED=NO
```

The visible compiled helper contains:

```text
BindClosureParameter
CheckClosureArgumentUpperBound
LoadClosureArgument
Lookup
```

which is consistent with the small `identity(value) => { value }` callee and not with the caller containing the repeated composed send.

That evidence does **not** establish any of these alternatives:

```text
caller helper never requested for compilation
caller helper compiled separately but omitted from the collected BGV set
caller helper considered for inlining and rejected
caller helper inlined through a constant target in a graph not selected previously
caller helper remains behind an opaque semantic/helper call-target boundary
```

Absence from the retained BGV set is therefore not promoted to absence of a compilation request.

## Compiler-lifecycle result

The retained raw local diagnostic artifacts named by the previous investigation are not part of the durable benchmark revision. This investigation did not regenerate compiler dumps merely for convenience.

No retained evidence currently establishes the caller helper's compilation request/completion/rejection lifecycle or the exact inlining decision at its semantic wrapper call site.

Therefore:

```text
CALLER_HELPER_INLINED=INCONCLUSIVE

CALLER_HELPER_COMPILATION_REQUESTED=INCONCLUSIVE
CALLER_HELPER_COMPILATION_COMPLETED=INCONCLUSIVE
CALLER_HELPER_COMPILATION_REJECTED=INCONCLUSIVE
REJECTION_OR_BOUNDARY_REASON=INCONCLUSIVE
```

## PERF010-A interpretation

The candidate remains architecturally large enough to investigate because, if the caller helper is not optimized, the repeated composed-send machinery can remain outside the optimized region while the tiny callee is compiled.

That topology would have qualitatively different scale from the already-tested few-percent local mechanisms.

However, the actual execution/compiler topology is not yet established.

```text
REPEATED_SEND_WORK_OUTSIDE_OPTIMIZED_REGION=INCONCLUSIVE

MATERIAL_COMMON_PATH_COST_PLAUSIBLE=YES
ORDER_OF_MAGNITUDE_RELEVANCE=PLAUSIBLE
```

`PLAUSIBLE` is deliberately not `ESTABLISHED`.

No attributable percentage is claimed.

## PERF011 relationship

PERF011 remains paused.

This result identifies a concrete compiler-visibility boundary candidate, but does not yet establish that it is a material runtime-representation/compiler-visibility mismatch.

```text
PERF011_REACTIVATION_TRIGGER=NOT_SATISFIED
PERF011_STATUS=PAUSED
```

## Reconciled result

```text
PERF010A_CALLER_COMPILATION_BOUNDARY=NOT_ESTABLISHED

CALLER_SEMANTIC_ROOT_IDENTIFIED=YES
CALLER_HELPER_TARGET_IDENTIFIED=YES

CALLER_HELPER_TARGET_PE_CONSTANT=INCONCLUSIVE
CALLER_HELPER_DIRECT_TRUFFLE_CALL=INCONCLUSIVE

CALLER_HELPER_INLINED=INCONCLUSIVE

CALLER_HELPER_COMPILATION_REQUESTED=INCONCLUSIVE
CALLER_HELPER_COMPILATION_COMPLETED=INCONCLUSIVE
CALLER_HELPER_COMPILATION_REJECTED=INCONCLUSIVE
REJECTION_OR_BOUNDARY_REASON=INCONCLUSIVE

REPEATED_SEND_WORK_OUTSIDE_OPTIMIZED_REGION=INCONCLUSIVE

MATERIAL_COMMON_PATH_COST_PLAUSIBLE=YES
ORDER_OF_MAGNITUDE_RELEVANCE=PLAUSIBLE

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED

PERF011_REACTIVATION_TRIGGER=NOT_SATISFIED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

## Stop condition

The investigation stops under **current diagnostics insufficient**.

Source inspection has reached its discriminating limit. Further source auditing cannot determine whether the caller helper was submitted, compiled, rejected or inlined.

## Exactly one next step

Run exactly one **diagnostic-only BASELINE execution of `micro/method-call`** at the same pinned Protos revision and unchanged workload, enabling compiler queuing/completion and inlining diagnostics sufficient to identify the caller semantic root by its exact source span and follow its exact helper CallTarget.

The diagnostic must answer only:

```text
Was the caller helper submitted for compilation?

If yes:
  was it compiled, invalidated, rejected or excluded?

At the caller semantic-root InvokeSemanticHelper site:
  was the exact helper target compiler-visible and inlined,
  compiled separately,
  or left behind a call-target boundary?
```

Constraints:

- same pinned BASELINE product revision;
- same exact `micro/method-call` workload;
- no source patch;
- no guarded-call image;
- no timing comparison;
- no new causal intervention;
- no workload algorithm change;
- no warmup investigation;
- no `continueAt` optimization;
- no production implementation.

Only after this compiler-lifecycle discriminator is available should PERF010-A decide whether a new causal experiment is warranted.
