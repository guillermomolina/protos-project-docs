# PERF010-A — Measurement discrimination reference evidence

Status: VALID NO-OP DISCRIMINATION EXPERIMENT — ABLATION 5 MEASUREMENT GATE CLOSED

This durable, non-normative record captures the PERF010-A measurement-discrimination
reference run performed after the remaining-common-path investigation established a
plausible next causal candidate but found the existing paired-control design
insufficiently stable for another small local ablation.

This is not a causal ablation. The compared Protos runtime variants are
source-equivalent; movement below is measurement movement/drift, not a Protos
runtime performance effect.

## Identity and retention state

```text
PROTOS_REVISION=4c4aa95a5852119bd280ceb40483871d5d2cbb82
BENCHMARK_HARNESS_REVISION=57cb307baad3346758d34aa3da1eda01997a4208
BENCHMARK_REFERENCE_RETENTION_REVISION=25f5b52ce0feb074db1529c939b7d5aec1b5aa30
```

The benchmark evidence is retained under
`results/perf010a-discrimination/` at the exact benchmark retention revision
above. The no-op image uses the same ablation build path with a zero-byte patch.

## Structural and run result

```text
PERF010A_DISCRIMINATION_REFERENCE=PASS
PERF010A_MEASUREMENT_DISCRIMINATION=ESTABLISHED
NO_OP_EXPERIMENT=VALID
VARIANT_ORDER_COUNTERBALANCED=YES
CANONICAL_CONTROL_PAIRING_PRESERVED=YES
PERF010A_PROTOS_REPOSITORY_MODIFICATION=NONE
```

Runtime-path equivalence was checked before timing: the baseline and no-op
variants had matching SHA-256 values for the relevant runtime sources and the
no-op runtime-path equivalence gate passed.

The reference used four deterministic counterbalanced blocks with order
`A, B, A, B`, N=10,000, warmup=20, and steady=100. Block-level canonical and
control results were retained rather than collapsed before discrimination
analysis.

## Per-workload no-op discrimination envelope

The pre-specified descriptive discrimination floor is the maximum absolute
no-op paired-control movement for each workload.

| workload | samples | min % | max % | median % | MAD % | floor % | order effect | per-workload gate |
|---|---:|---:|---:|---:|---:|---:|---|---|
| micro/slot-read | 4 | -38.9049 | +11.7933 | -6.4728 | 11.9613 | 38.9049 | NOT_DETECTED | CLOSED |
| micro/closure-call | 4 | -34.8335 | +8.2006 | -6.3426 | 11.6344 | 34.8335 | DETECTED | CLOSED |
| micro/method-call | 4 | -10.8286 | -3.2091 | -5.2771 | 2.0532 | 10.8286 | DETECTED | CLOSED |
| runtime/monomorphic-dispatch | 4 | +0.9862 | +2.0901 | +1.1407 | 0.1078 | 2.0901 | NOT_DETECTED | OPEN |

The no-op envelope is highly workload-dependent. In particular, the first three
workloads show floors from about 10.8% to 38.9%, while
`runtime/monomorphic-dispatch` is substantially tighter at about 2.09%.
Order effects were detected for `micro/closure-call` and
`micro/method-call`.

These values are descriptive discrimination evidence, not confidence intervals
and not Protos speedups, slowdowns, regressions, or optimization effects.

## Relationship to retained causal ablations

Against this no-op discrimination floor, the retained effects classify as:

```text
ABLATION_1_EFFECT_VS_FLOOR=MIXED
ABLATION_3_EFFECT_VS_FLOOR=MIXED
ABLATION_4_EFFECT_VS_FLOOR=MIXED
```

This does not retroactively invalidate the structural validity or historical
conclusions of those experiments. It establishes that their local effect scale
can overlap the measurement movement of this harness and therefore constrains
how confidently another small local effect can be discriminated.

## Ablation 5 gate and next methodological step

```text
ABLATION_5_MEASUREMENT_GATE=CLOSED
```

The already-established next causal candidate — the invocation-time second
`List.copyOf` of closure `capturedLexicalContexts` — was not measured here.
No Ablation 5 source patch is authorized by this evidence.

The minimum next methodological change established by the retained run is:

```text
process/container lifecycle isolation between blocks,
plus more counterbalanced blocks (extend block_order beyond AABB)
```

The purpose is to determine whether the detected order effect remains stable
after lifecycle carry-over is isolated and to reduce/characterize the no-op
discrimination floor before trusting another paired-control comparison for a
small local runtime component.

## Reconciled state

```text
PERF010A_MEASUREMENT_DISCRIMINATION=ESTABLISHED
NO_OP_EXPERIMENT=VALID
ABLATION_5_MEASUREMENT_GATE=CLOSED

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

PERF010-A therefore remains open. The next bounded step is methodological
measurement stabilization, not Ablation 5 and not a production optimization.
