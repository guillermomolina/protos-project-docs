# PERF010-A guarded-call causal experiment reconciliation

## Evidence identity

```text
PROTOS_REVISION=2b3a88389da7228caed231a90b14091cf2841115
BENCHMARK_HARNESS_REVISION=3275c108aa9b04d35a67fbe9be1c13e6fe94c58a
BENCHMARK_EVIDENCE_REVISION=1611f9b19005a13723cc9347addcbb74e926290d
BENCHMARK_EVIDENCE_PATH=results/perf010a-guarded-call/
```

This record reconciles the completed PERF010-A guarded constant-selector local-method monomorphic composed-send diagnostic experiment with PERF011's runtime-representation-fit question.

The diagnostic patch was retained only in `guillermomolina/protos-benchmarks`; it was never published to `guillermomolina/protos`.

## Experiment validity

```text
PERF010A_GUARDED_CALL_EXPERIMENT=VALID
FAST_PATH_SEMANTICS=PASS
D013_LOOKUP_PRESERVED=YES
GENERIC_PREPARATION_BYPASSED=YES
STRUCTURED_DISPATCH_BYPASSED=YES
GENERIC_FALLBACK_PRESERVED=YES
```

The experiment preserves authoritative D013 lookup. The guarded arm is taken only when fresh lookup still resolves the same Closure and methodHome identities and the Closure is non-native. It preserves fresh activation and control-state machinery, while bypassing the generic implementation classifier and structured-dispatch ladder on a guarded hit. Guard misses retain the exact generic fallback.

The benchmark evidence structurally confirms exact experiment scope across all four workloads.

## Paired timing result

| workload | relation to patched call site | baseline median | guarded median | guarded change |
|---|---|---:|---:|---:|
| `micro/method-call` | direct | 54.677 ms | 59.769 ms | 9.3% slower |
| `runtime/monomorphic-dispatch` | direct | 53.005 ms | 57.092 ms | 7.7% slower |
| `micro/slot-read` | indirect | 51.800 ms | 50.675 ms | 2.2% faster |
| `micro/closure-call` | indirect | 66.625 ms | 65.610 ms | 1.5% faster |

The two workloads that exercise `PrepareSendArguments.performGuardedOrdinaryComposedSend` both move in the negative direction. The two workloads that do not traverse the patched call site move only about 1.5–2.2% in the positive direction.

The retained MAD is of the same general order as the direct-workload effect, so the evidence does not support a precise numeric causal attribution. The consistent negative direction on both direct workloads, absent from the indirect workloads, is nevertheless sufficient to reject this guarded bypass as a demonstrated positive Tier-A optimization candidate.

This does **not** establish that the generic classifier/structured-dispatch machinery has zero cost, nor does it attribute the 7.7–9.3% slowdown directly to the original generic machinery. The intervention adds guards and changes specialization structure; compiler behavior remains unresolved.

## Compiler-visibility result

BGV dumps were generated for baseline and guarded images and contain the relevant `ProtosBytecodeRootNodeGen` tier1/tier2 compilations. However, no retained structural extraction/comparison of the graph content was completed.

File existence and near-equal dump sizes are not evidence of graph equivalence.

Therefore:

```text
COMPILER_TARGET_VISIBILITY_BASELINE=INCONCLUSIVE
COMPILER_TARGET_VISIBILITY_GUARDED=INCONCLUSIVE
DIRECT_INVOCATION_BASELINE=INCONCLUSIVE
DIRECT_INVOCATION_GUARDED=INCONCLUSIVE
GENERIC_CLASSIFICATION_VISIBLE_BASELINE=INCONCLUSIVE
GENERIC_CLASSIFICATION_VISIBLE_GUARDED=INCONCLUSIVE
STRUCTURED_DISPATCH_VISIBLE_BASELINE=INCONCLUSIVE
STRUCTURED_DISPATCH_VISIBLE_GUARDED=INCONCLUSIVE
COMPILER_VISIBILITY_CHANGED=INCONCLUSIVE
```

Do not reinterpret these fields as `NO`. The graph content was not actually compared.

## PERF010-A reconciliation

```text
PAIRED_PERFORMANCE_RESULT=NEGATIVE
TIER_A_CAUSAL_CONTRIBUTION=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

The guarded-call candidate remains valid as a causal experiment, but it does not establish a positive recoverable contribution and must not be selected as the first production optimization.

The next PERF010-A step should reconcile the remaining common-path candidates against all retained ablation evidence and select the next single bounded causal target. It should not assume that the guarded-call result proves the original generic classification machinery negligible.

## PERF011 reconciliation

```text
PERF011_RUNTIME_FIT=INCONCLUSIVE
MATERIAL_RUNTIME_FIT_MISMATCH=NOT_ESTABLISHED
CALL_SITE_REPRESENTATION_MATERIALITY=NOT_ESTABLISHED
COMPILER_GRAPH_COMPARISON=PENDING_IF_LATER_JUSTIFIED
NEW_DECISION_REQUIRED=NO
```

The source-level representation-fit candidate remains architecturally plausible, but this guarded intervention did not produce a timing benefit and the compiler-graph comparison was not completed.

PERF011 therefore remains open and inconclusive. It does not block continued PERF010-A attribution. A future compiler-graph comparison may explain why this guarded specialization failed to improve timing, but it is not required merely to decide that this particular intervention is not a demonstrated production optimization.

## Final state

```text
PERF010A_GUARDED_CALL_EXPERIMENT=VALID
PAIRED_PERFORMANCE_RESULT=NEGATIVE
TIER_A_CAUSAL_CONTRIBUTION=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
PERF011_RUNTIME_FIT=INCONCLUSIVE
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```
