# PERF010-A / PERF011 — Guarded-call compiler-visibility discrimination

Status: retained diagnostic analysis; no production optimization selected.

This durable, non-normative record captures the bounded BASELINE-versus-GUARDED Graal BGV inspection requested after the guarded-call timing result. It records diagnostic evidence and its limitations only. No Protos source change, new ablation, workload change, semantic decision, or production optimization is introduced here.

## Evidence identity

```text
PROTOS_REVISION=2b3a88389da7228caed231a90b14091cf2841115
BENCHMARK_EVIDENCE_REVISION=1611f9b19005a13723cc9347addcbb74e926290d
HARNESS_REVISION=3275c108aa9b04d35a67fbe9be1c13e6fe94c58a
DIAGNOSTIC_PATCH=docker/protos-perf010a/guarded-call.patch
DIAGNOSTIC_PATCH_SHA256=a1845b3125abd80631e9d201c09359a43b4fdb4088a74b242fb0484e0c38f83c
RUNTIME=GraalVM Community 25.3.4.1 / JDK 25.0.4.1 / Truffle 25.3.4.1
ANALYZER_IMAGE=protos-benchmarks/igv-analyzer:graal-25.3.4.1
BASELINE_IMAGE=protos-benchmarks-perf010a-ablationguarded-call-baseline:2b3a88389da7
GUARDED_IMAGE=protos-benchmarks-perf010a-ablationguarded-call-ablation:2b3a88389da7
```

Prior durable record:

`docs/project/evidence/PERF010-A/PERF010-A_GUARDED_CALL_CAUSAL_RESULT_ANALYSIS.md`

The BGV files and JSON exports used for this inspection were local diagnostic artifacts under `/tmp/perf011-probe/`; they were not part of `BENCHMARK_EVIDENCE_REVISION`. The extracted inventory and conclusions are retained here, but the raw graph artifacts are not independently reproducible from the benchmark repository alone.

## Question

The prior timing result left two main explanations unresolved:

- H1: baseline partial evaluation already removes most or all generic call-preparation / structured-dispatch machinery, so the guarded intervention adds cost without removing meaningful compiled work;
- H2: the generic machinery survives in baseline and the guarded specialization removes it, but the guard/specialization cost exceeds the removed work.

The requested discriminator was to compare optimized BASELINE and GUARDED compiler graphs for the direct workloads and determine whether `PrepareSendArguments`, generic classification, structured dispatch, target visibility, and relevant inlining differ.

## BGV inventory result

The captured BASELINE and GUARDED sets have matching compilation topology: semantic Bytecode roots appear in Tier1/Tier2 pairs, and one `ProtosBytecodeRootNodeGen` helper root appears in Tier1 and Tier2 on each side.

The only captured `ProtosBytecodeRootNodeGen` roots contain these Protos Bytecode operations:

```text
BindClosureParameter
CheckClosureArgumentUpperBound
LoadClosureArgument
Lookup
```

For BASELINE:

```text
Tier1: nodes=2440 edges=6781
Tier2: nodes=2379 edges=6413
```

For GUARDED:

```text
Tier1: nodes=2440 edges=6781
Tier2: nodes=2379 edges=6413
```

Across every captured BGV on both sides, the scan found zero occurrences of the guarded-send discriminator markers:

```text
PrepareSendArguments=0
finishPreparingComposedCallByImplementation=0
prepareImmediateMethodCall=0
performGuardedOrdinaryComposedSend=0
guardedOrdinaryComposedSendMatches=0
```

The helper-root JSON contains repeated source-position references to `ProtosValueLookup.lookup`; the observed textual count (`289`) is not a dynamic execution count and must not be interpreted as one.

## Direct Tier2 graph comparison

The BASELINE and GUARDED `ProtosBytecodeRootNodeGen` Tier2 JSON exports were compared structurally.

```text
After PE Tier:
  BASELINE=2379 nodes / 6413 edges
  GUARDED =2379 nodes / 6413 edges

After TruffleTier:
  BASELINE=2232 nodes / 5937 edges
  GUARDED =2232 nodes / 5937 edges

After mid tier:
  BASELINE=3584 nodes / 7942 edges
  GUARDED =3584 nodes / 7942 edges

After low tier:
  BASELINE=9028 nodes / 16356 edges
  GUARDED =9041 nodes / 16369 edges
```

After normalizing ephemeral lambda identities, generated-Java line offsets, and native-address values, the compared graphs are node-for-node and edge-for-edge equivalent from `After PE Tier` through `After mid tier`. The low-tier GUARDED graph contains thirteen additional `HotSpotCompressionNode` / `Uncompress` nodes and thirteen additional edges. Those backend/lowering differences do not expose the guarded Protos send specialization and are not evidence that the guarded call path survived compilation.

Therefore the selected helper-root pair cannot discriminate H1 from H2.

## Source-to-graph interpretation

The pinned `micro/method-call` workload executes a caller closure 10,000 times whose body contains the composed send:

```text
sink = receiver.identity(42)
```

The selected `identity` method body itself only returns its argument.

The four operations visible in the only compiled `ProtosBytecodeRootNodeGen` helper root are consistent with the `identity` callee activation: argument load, argument-bound check, parameter bind, and lookup of the bound value. They are not consistent with the caller helper that must execute `PrepareSendArguments` for `receiver.identity(42)`.

The compiler capture therefore shows the callee helper root, but does not show the caller helper root containing the 10,000 composed sends, either as its own `ProtosBytecodeRootNodeGen` compilation or inlined into another captured graph.

This is a stronger result than the original file-size comparison, but it is not proof that the caller executes interpreted for the full steady-state workload. The capture establishes absence from the collected optimized graphs, not the exact reason for that absence.

## Reconciled result

```text
COMPILER_GRAPH_CAPTURE=VALID
CAPTURED_BASELINE_GUARDED_TOPOLOGY=COMPARABLE
SELECTED_HELPER_ROOT_BASELINE_VS_GUARDED=MATERIALLY_EQUIVALENT
GUARDED_SEND_PRESENT_IN_CAPTURED_GRAPHS=NO
PREPARE_SEND_ARGUMENTS_PRESENT_IN_CAPTURED_GRAPHS=NO
CALLER_HELPER_WITH_10000_SENDS_VISIBLE_AS_COMPILED_OR_INLINED=NO

BASELINE_GENERIC_CLASSIFICATION_COMPILER_VISIBLE=INCONCLUSIVE
GUARDED_GENERIC_CLASSIFICATION_COMPILER_VISIBLE=INCONCLUSIVE
BASELINE_STRUCTURED_DISPATCH_COMPILER_VISIBLE=INCONCLUSIVE
GUARDED_STRUCTURED_DISPATCH_COMPILER_VISIBLE=INCONCLUSIVE
BASELINE_TARGET_PE_CONSTANT=INCONCLUSIVE
GUARDED_TARGET_PE_CONSTANT=INCONCLUSIVE
H1_VS_H2=INCONCLUSIVE

NEW_STRONGEST_COMPILER_VISIBILITY_CANDIDATE=caller/send helper compilation boundary
ORDER_OF_MAGNITUDE_RELEVANCE=PLAUSIBLE_NOT_ESTABLISHED
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

The guarded-call local specialization remains a negative production candidate from timing. This graph inspection does not rehabilitate it.

The new compiler-visibility candidate is materially broader: if the hot caller containing the repeated send is not optimized while the tiny callee is, the dominant cost could lie before the compiled callee boundary. That has the right architectural scale to remain relevant to the large cross-runtime gap, but the current evidence does not yet establish runtime materiality or causality.

## PERF011 relationship

The result is directly relevant to PERF011's compiler-visibility scope, but does not yet establish a material runtime-representation mismatch.

```text
PERF011_RUNTIME_FIT=INCONCLUSIVE
PERF011_STATUS=PAUSED
PERF011_REACTIVATION_TRIGGER=NOT_YET_ESTABLISHED
```

PERF011 should remain paused until PERF010-A establishes that the caller/helper compilation boundary is a material common-path cost plausibly caused by runtime representation or compiler visibility, or until a concrete optimization decision requires resolving that question. The graph comparison by itself does not satisfy the retained reactivation contract.

## Single next discriminator

Do not implement another guarded-call ablation and do not return to `continueAt` or warmup tuning.

Map the captured semantic roots and helper targets to the exact guest closures, then determine why the caller closure containing `PrepareSendArguments` does not appear as a compiled/inlined helper root while the `identity` callee does.

The bounded questions are:

```text
CALLER_SEMANTIC_ROOT_IDENTIFIED=YES | NO
CALLER_HELPER_TARGET_PE_CONSTANT=YES | NO | INCONCLUSIVE
CALLER_HELPER_DIRECT_TRUFFLE_CALL=YES | NO | INCONCLUSIVE
CALLER_HELPER_INLINED=YES | NO | INCONCLUSIVE
CALLER_HELPER_COMPILATION_REQUESTED=YES | NO | INCONCLUSIVE
CALLER_HELPER_COMPILATION_REJECTED=YES | NO | INCONCLUSIVE
REJECTION_OR_BOUNDARY_REASON=... | INCONCLUSIVE
MATERIAL_COMMON_PATH_COST_PLAUSIBLE=YES | NO | INCONCLUSIVE
```

Only after that mapping should PERF010-A decide whether a new causal experiment is warranted.
