# PERF010-A — stable-identity Phase 1 measurement evidence published

Status: publication checkpoint only. The PERF010-A Phase 1 measurement-system-admission Evidence Unit has been executed, retained, and published in `guillermomolina/protos-benchmarks`. This record does not classify the measurement result.

## Evidence identity

```text
BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks

HARNESS_REVISION=
  43cc0ba7b70b624cf30d30d2be2bd27144223b5d

EVIDENCE_PUBLICATION_REVISION=
  8886ba285b75954eb182342ef1a51dab52c54787

EVIDENCE_ROOT=
  results/perf010a-stable-identity-phase1/

PROTOS_REVISION=
  4c4aa95a5852119bd280ceb40483871d5d2cbb82
```

The published evidence root contains:

```text
README.md
raw.json
stationarity.tsv
phase1-summary.tsv
SHA256SUMS
logs/
```

The full timed-unit stdout/stderr log set is retained under `logs/`.

## Executed measurement contract

The retained Evidence Unit records:

```text
PERF010A_PHASE1_REFERENCE=PASS
PERF010A_PHASE1_EVIDENCE_STATUS=RETAINED
PERF010A_PHASE1_FULL_MATRIX_EVIDENCE_UNIT=YES

WARMUP=120
STEADY=100
OPERATION_COUNT=10000
BLOCK_ORDER=A,B,A,B

WORKLOADS=
  micro/slot-read
  micro/closure-call
  micro/method-call
  runtime/monomorphic-dispatch
```

The experiment is a measurement-system admission run, not a causal product comparison. The baseline/no-op images execute the same Protos runtime path; the no-op runtime-path equivalence is retained in the published evidence.

Actual built-image identities are retained in `raw.json` and summarized in the benchmark evidence README.

## Analysis boundary

This checkpoint records execution and publication only.

The following classifications have **not** yet been performed:

```text
WARMUP_120_STABILITY_HYPOTHESIS=PENDING_ANALYSIS
MEASUREMENT_GATE=PENDING_ANALYSIS
```

The published benchmark README intentionally leaves those fields pending for the next investigation slice.

This record therefore does not conclude whether warmup=120 improved or worsened measurement stability, does not open or close the measurement gate, and does not authorize Phase 2.

```text
PHASE2_EXECUTED=NO
CAUSAL_CONTROL_INTERVENTION_COMPARISON=NOT_RUN
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
```

## Next action

The next slice is investigation/evidence analysis against the retained Evidence Unit at benchmark revision:

```text
8886ba285b75954eb182342ef1a51dab52c54787
```

That analysis must classify, from the retained raw data and stationarity output:

```text
WARMUP_120_STABILITY_HYPOTHESIS=
  SUPPORTED | WEAKENED | INCONCLUSIVE

MEASUREMENT_GATE=
  OPEN | CLOSED | INCONCLUSIVE
```

No Phase 2 measurement should be inferred from this publication checkpoint alone.
