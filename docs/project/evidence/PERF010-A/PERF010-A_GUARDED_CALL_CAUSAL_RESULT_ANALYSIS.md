# PERF010-A / PERF011 — Guarded-call causal result analysis

Status: retained evidence analysis; no production optimization selected.

This durable, non-normative record reconciles the retained PERF010-A guarded-call timing result with the PERF011 compiler-visibility question. It records analysis only: no Protos source change, new ablation, benchmark run, compiler diagnostic, or semantic decision is introduced here.

## Evidence identity

```text
PROTOS_REVISION=2b3a88389da7228caed231a90b14091cf2841115
BENCHMARK_EVIDENCE_REVISION=1611f9b19005a13723cc9347addcbb74e926290d
HARNESS_REVISION=3275c108aa9b04d35a67fbe9be1c13e6fe94c58a
DIAGNOSTIC_PATCH=docker/protos-perf010a/guarded-call.patch
DIAGNOSTIC_PATCH_SHA256=a1845b3125abd80631e9d201c09359a43b4fdb4088a74b242fb0484e0c38f83c
RESULT_DIRECTORY=results/perf010a-guarded-call/
RUNTIME=GraalVM Community 25.3.4.1 / JDK 25.0.4.1 / Truffle 25.3.4.1
```

Prior durable records consumed by this reconciliation:

- `docs/project/evidence/PERF010-A/PERF010-A_GUARDED_MONOMORPHIC_CALL_PATH_ARCHITECTURE.md`
- `docs/project/evidence/PERF011/PERF011_RUNTIME_REPRESENTATION_FIT_AUDIT.md`

## Result

The diagnostic intervention is structurally and semantically valid, and its two workloads that directly exercise the patched `PrepareSendArguments` operation become slower.

The timing result does **not** establish that Graal already removes the generic machinery in the baseline. That proposition remains a compiler-visibility question.

```text
PERF010A_GUARDED_CALL_RESULT=VALID
DIRECT_WORKLOAD_TIMING=NEGATIVE
DIRECT_EFFECT_CONSISTENCY=CONSISTENT_NEGATIVE
MEASUREMENT_PRECISION=CONSISTENT_BUT_NOT_PRECISE
GUARDED_PATCH_CAUSAL_PURITY=PARTIAL
OUTCOME_C=NOT_ESTABLISHED
TIER_A_CAUSAL_BENEFIT=NEGATIVE_FOR_THIS_INTERVENTION
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PERF011_RUNTIME_FIT=INCONCLUSIVE
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

## Direct versus indirect workloads

For this exact guarded `PrepareSendArguments` intervention:

```text
micro/method-call=DIRECT
runtime/monomorphic-dispatch=DIRECT
micro/slot-read=INDIRECT
micro/closure-call=INDIRECT
```

The indirect workloads are retained as measurement/control context and are not causal evidence for the guarded-send patch.

## Paired-control reconciliation

Sign convention below: positive means faster guarded canonical execution; negative means slower guarded canonical execution.

| Workload | Class | Raw canonical effect | Control movement | Paired-control effect |
| --- | --- | ---: | ---: | ---: |
| `micro/slot-read` | INDIRECT | +1.1254505 ms (+2.17%) | +3.656064 ms | -2.5306135 ms (-4.89%) |
| `micro/closure-call` | INDIRECT | +1.0143955 ms (+1.52%) | -3.345737 ms | +4.3601325 ms (+6.54%) |
| `micro/method-call` | DIRECT | -5.091943 ms (-9.31%) | -0.863472 ms | -4.228471 ms (-7.73%) |
| `runtime/monomorphic-dispatch` | DIRECT | -4.086800 ms (-7.71%) | +0.767044 ms | -4.853844 ms (-9.16%) |

For `micro/method-call`, paired-control correction reduces but preserves the negative effect. For `runtime/monomorphic-dispatch`, correction strengthens the negative effect. Both direct workloads therefore remain negative after correction.

## Dispersion

The retained direct-workload MAD is of the same general order as the measured effect:

```text
micro/method-call:
  baseline MAD/median ~= 8.2%
  guarded  MAD/median ~= 11.4%

runtime/monomorphic-dispatch:
  baseline MAD/median ~= 7.9%
  guarded  MAD/median ~= 8.2%
```

The p95/min/max ranges are broad. The evidence therefore supports a consistent direction across the two direct workloads, but not a highly precise effect-size estimate. No samples are discarded and no unsupported confidence interval is inferred.

## Patch differential

The baseline hit performs authoritative D013 lookup, generic composed-call preparation, fresh activation construction, implementation/category classification, structured-dispatch classification, and ordinary invocation.

The guarded hit preserves authoritative lookup semantics and fresh activation/control behavior while bypassing `finishPreparingComposedCallByImplementation` classifiers and the structured-dispatch predicate universe when its specialization matches. It introduces selector/cache identity guards, an authoritative D013 lookup used by the match guard, and the guarded specialization itself.

The two `@Cached` expressions that initialize cached Closure and `methodHome` are specialization-cache initialization work; they must not be counted as two additional authoritative lookups on every steady-state specialization hit. The dynamic match guard remains per-hit work. Consequently the retained source comment/README wording that describes three independent lookup helper call sites must not be interpreted as three steady-state D013 lookups per invocation.

This makes the intervention partially discriminating rather than a zero-cost subtraction of the generic machinery:

```text
GUARDED_PATCH_CAUSAL_PURITY=PARTIAL
PER_INVOCATION_GUARD_COST=PARTIALLY_ESTABLISHED
```

## Why Outcome C is not established

The negative timing admits at least two materially different explanations.

H1: Graal partial evaluation already removes most or all baseline generic classification / structured-dispatch machinery. The guarded bypass then removes little compiled work while adding guard cost.

H2: the generic machinery survives in baseline and the guarded specialization removes it, but the added guard/specialization cost exceeds the removed work.

Mixed cases remain possible. Timing alone cannot distinguish these cases.

Therefore:

```text
BASELINE_GENERIC_CLASSIFICATION_COMPILER_VISIBLE=NOT_ESTABLISHED
GUARDED_GENERIC_CLASSIFICATION_COMPILER_VISIBLE=NOT_ESTABLISHED
BASELINE_TARGET_PE_CONSTANT=NOT_ESTABLISHED
GUARDED_TARGET_PE_CONSTANT=NOT_ESTABLISHED
OUTCOME_C=NOT_ESTABLISHED
```

The guarded timing result is negative for this intervention; it does not falsify the broader Tier-A architectural hypothesis by itself.

## Exact next discriminator

Do not implement another causal ablation.

Reuse the exact existing BASELINE and GUARDED images from this evidence unit and capture bounded Graal/Truffle compiler diagnostics for only:

```text
micro/method-call
runtime/monomorphic-dispatch
```

No Protos source change, runtime-representation change, new fast path, workload-algorithm change, or new causal variable is part of this step.

The diagnostic must determine, in both images:

1. whether generic implementation classifiers survive optimized compilation;
2. whether structured-dispatch predicates survive;
3. whether the effective ordinary invocation target is PE-constant/direct;
4. which relevant call/inlining decisions differ.

The repository already contains the pinned Graal 25.3.4.1 `docker/igv-analyzer/` and `scripts/igv_analyzer.sh` infrastructure. The implementation of this diagnostic must first verify the exact Graal/Truffle diagnostic option names supported by the pinned 25.3.4.1 runtime rather than assuming historical option names.

```text
NEXT_DISCRIMINATING_STEP=
bounded BASELINE-vs-GUARDED compiler-graph diagnostic on the existing
guarded-call experiment, with no new causal intervention
```

## Reconciled live state

```text
PERF010A_GUARDED_CALL_RESULT=VALID
DIRECT_WORKLOAD_TIMING=NEGATIVE
PAIRED_CONTROL_RESULT=NEGATIVE_IN_BOTH_DIRECT_WORKLOADS
DIRECT_EFFECT_CONSISTENCY=CONSISTENT_NEGATIVE
MEASUREMENT_PRECISION=CONSISTENT_BUT_NOT_PRECISE
GUARDED_PATCH_CAUSAL_PURITY=PARTIAL
BASELINE_GENERIC_CLASSIFICATION_COMPILER_VISIBLE=NOT_ESTABLISHED
GUARDED_GENERIC_CLASSIFICATION_COMPILER_VISIBLE=NOT_ESTABLISHED
BASELINE_TARGET_PE_CONSTANT=NOT_ESTABLISHED
GUARDED_TARGET_PE_CONSTANT=NOT_ESTABLISHED
OUTCOME_C=NOT_ESTABLISHED
TIER_A_CAUSAL_BENEFIT=NEGATIVE_FOR_THIS_INTERVENTION
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PERF011_RUNTIME_FIT=INCONCLUSIVE
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
NEXT_DISCRIMINATING_STEP=bounded BASELINE-vs-GUARDED compiler-graph diagnostic
```

PERF010-A / #691 remains open and owns the next bounded discriminator. PERF011 / #693 consumes the same compiler-visible result and does not create a separate causal experiment. PERF010 / #680 remains gated on attribution and production-selection evidence.
