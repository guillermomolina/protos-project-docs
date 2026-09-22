# PERF010-A — Ablation 4 reference evidence

Status: VALID CAUSAL ABLATION — COMMON CONTRIBUTION NOT ESTABLISHED

This durable, non-normative record captures the PERF010-A Ablation 4 reference
result for the duplicate `ProtosClosureValue.nativeBody()` Optional projection
inside `ProtosBytecodeRootNode.finishPreparingComposedCall`.

## Identity and retention state

```text
PROTOS_REVISION=4c4aa95a5852119bd280ceb40483871d5d2cbb82
BENCHMARK_IMPLEMENTATION_REVISION=9cb377cec8f25cf6ed3db459d69dfa85c21f3f88
BENCHMARK_REFERENCE_RETENTION_REVISION=b829ba00a22dd06e06aaf91faf3411c8e65f4511
```

The benchmark reference evidence is published under the exact retention revision
above with commit message
`PERF010-A: retain fourth causal ablation evidence (#691)`.

## Structural and run result

```text
ABLATION_4_BASELINE_HAS_TWO_PROJECTIONS=PASS
ABLATION_4_ABLATION_HAS_ONE_PROJECTION=PASS
ABLATION_4_OTHER_CALL_SITES_UNCHANGED=PASS
ABLATION_4_CLOSURE_VALUE_UNCHANGED=PASS
ABLATION_4_PATCH_SCOPE_MATCH=PASS

PERF010A_REFERENCE=PASS
PERF010A_WORKLOADS=4
PERF010A_EVIDENCE_UNITS=32
PERF010A_ABLATION_4=VALID
PERF010A_PROTOS_REPOSITORY_MODIFICATION=NONE
```

One supplementary structural sample for
`micro/closure-call / ablation / canonical` reported
`helper_continue_at_present=False`; this does not invalidate Ablation 4 because
its causal structural contract is source-derived and the exact-scope checks
above passed. No cost conclusion is derived from that sampled marker.

## Retained medians

| workload | baseline canonical ns | baseline control ns | ablation canonical ns | ablation control ns |
|---|---:|---:|---:|---:|
| micro/slot-read | 48,057,331.0 | 47,759,029.0 | 50,589,520.5 | 44,850,309.0 |
| micro/closure-call | 68,856,395.5 | 46,314,467.0 | 66,070,455.0 | 48,336,809.5 |
| micro/method-call | 60,225,027.5 | 45,519,117.0 | 67,064,194.0 | 44,412,907.0 |
| runtime/monomorphic-dispatch | 55,869,245.5 | 46,588,695.0 | 60,807,671.0 | 48,435,563.0 |

## Paired-control reconciliation

```text
canonical improvement = baseline canonical - ablation canonical
control movement       = baseline control - ablation control
paired-control effect  = canonical improvement - control movement
paired-control fraction = paired-control effect / baseline canonical
```

| workload | canonical improvement ms | control movement ms | paired-control effect ms | fraction of baseline |
|---|---:|---:|---:|---:|
| micro/slot-read | -2.5322 | +2.9087 | -5.4409 | -11.32% |
| micro/closure-call | +2.7859 | -2.0223 | +4.8083 | +6.98% |
| micro/method-call | -6.8392 | +1.1062 | -7.9454 | -13.19% |
| runtime/monomorphic-dispatch | -4.9384 | -1.8469 | -3.0916 | -5.53% |

The signs and magnitudes are strongly non-uniform. The exact-scope diagnostic is
valid, but these retained medians do not establish a positive common-path
contribution or dominance for the duplicate projection. Raw canonical movement
alone is not interpreted causally.

The descriptive paired-control calculation is not a confidence interval and is
not an attributable fraction of cross-language excess cost.

## Reconciled state

```text
PERF010A_ABLATION_4=VALID
ABLATION_4_SCOPE_MATCH=PASS
ABLATION_4_CAUSAL_COMPONENT_MEASURED=YES
ABLATION_4_COMMON_CONTRIBUTION=NOT_ESTABLISHED
ABLATION_4_DOMINANCE=NOT_ESTABLISHED

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

Ablation 4 therefore does not justify selecting the duplicate
`nativeBody()` projection as the production optimization target.

## Next step

Do not infer a fifth ablation merely from sequence. Reconcile this result with
the existing Ablation 1 and corrected Ablation 3 paired-control evidence, then
return to investigation of the remaining common execution path. A future
candidate still has to pass the static causal-ablation admission gate before any
new benchmark is implemented.
