# PERF010-A stable-identity Phase 2 causal comparator published

## Purpose

Record publication of the clean two-product-revision Phase 2 comparator required by the
PERF010-A Phase 1 measurement-gate result.

This checkpoint records harness readiness only. It does **not** retain Phase 2 causal timing
evidence and does not classify the runtime materiality of the stable-identity intervention.

## Exact harness identity

```text
BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
BENCHMARK_REVISION=952cfd1290023a04108854163da30ba48b18b3c6
COMMIT_MESSAGE=perf010a: add stable-identity Phase 2 causal comparator

CHANGED_FILES=
  config/perf010a-phase2.json
  runner/perf010a.py
```

The comparator is authorized by the preceding Phase 1 analysis:

```text
PROJECT_RECORD_REVISION=ed99717e5c31a607889f0cc03d24d736eab479aa
PHASE2_ACTION=PROCEED_TO_CAUSAL_MEASUREMENT
MEASUREMENT_GATE=OPEN
```

## Causal comparison contract

The published harness compares these exact Protos revisions:

```text
CONTROL_PROTOS_REVISION=
  2b3a88389da7228caed231a90b14091cf2841115

INTERVENTION_PROTOS_REVISION=
  3e8e6b565c95eb5098c2168d241536ba13ad19e9

CONTROL_VARIANT=baseline
INTERVENTION_VARIANT=baseline
PATCHES_APPLIED=none
```

The product revision is the intended Protos-code difference between the two images.

The retained Phase 2 Evidence Unit is defined as:

```text
WARMUP=120
STEADY=100
OPERATION_COUNT=10000
BLOCK_ORDER=A,B,A,B

PRIMARY_CAUSAL_WORKLOADS=
  micro/method-call
  runtime/monomorphic-dispatch

NEGATIVE_COVERAGE_WORKLOADS=
  micro/slot-read
  micro/closure-call
```

The harness preserves canonical/workload-control pairing, uses a fresh container/JVM per timed
unit, disables network access, records actual image identities, retains raw timed samples and
stationarity data, and fails closed on revision/variant/patch-identity mismatches.

## Validation reported before publication

The maintainer reported the Phase 2 smoke gate PASS before publishing the harness.

A bounded full-scale reference-validation run was also reported PASS for one primary workload and
one block:

```text
PERF010A_PHASE2_REFERENCE=PASS
PERF010A_PHASE2_EVIDENCE_STATUS=VALIDATION_ONLY_NOT_RETAINED
PERF010A_PHASE2_FULL_MATRIX_EVIDENCE_UNIT=NO
PERF010A_PHASE2_WARMUP=120
PERF010A_PHASE2_STEADY=100

WORKLOAD=micro/method-call
PAIRED_CONTROL_EFFECT_MEDIAN=19.2894
ORDER_EFFECT=INCONCLUSIVE

PERF010A_PROTOS_REPOSITORY_MODIFICATION=NONE
```

The `19.2894` value is **not** Phase 2 evidence. The run is explicitly
`VALIDATION_ONLY_NOT_RETAINED`, covers only one workload/block, and exists only to prove that the
published comparator executes the intended causal calculation path.

Therefore:

```text
CAUSAL_RUNTIME_SPEEDUP=NOT_MEASURED
DOMINANT_GAP_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
```

## Next slice

The next slice is investigation/evidence execution, not implementation.

Run the full retained Phase 2 Evidence Unit from the clean exact benchmark revision
`952cfd1290023a04108854163da30ba48b18b3c6`, with no Phase 2 matrix overrides, producing
`results/perf010a-phase2`.

That evidence must directly answer the materiality question for the current candidate. The next
analysis must be allowed to close this candidate as **not the large PERF010-A cost** if the
causal effect is only comparable to ordinary measurement movement / tens-of-percent scale rather
than a large multiplicative improvement.

No further PIC-churn, compiler-lifecycle, warmup, source-identity, JFR, or adjacent mechanism
investigation is authorized before this causal timing result.
