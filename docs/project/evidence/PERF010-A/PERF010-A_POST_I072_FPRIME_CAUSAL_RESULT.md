# PERF010-A — Post-I072 Candidate F′ causal result

Status: RETAINED CAUSAL RESULT — F′ CLOSED AS NOT THE BIG COST

This durable, non-normative record interprets the retained PERF010-A post-I072
control/intervention Evidence Unit. It answers only whether the complete
PLAT040 Candidate F′ implementation materially explains the dominant common
runtime gap.

## Evidence identity

```text
PERF_ITEM=PERF010-A
MEASUREMENT_SLICE=PERF010A_POST_I072_FPRIME_CAUSAL_COMPARATOR

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
HARNESS_REVISION=113949a1aebc0eb769a16b35368a8738f38102a4
BENCHMARK_EVIDENCE_REVISION=06aa4f4af2476997b9eaf5c88c5df6ace02b8d40
EVIDENCE_ROOT=results/perf010a-post-i072-fprime

CONTROL_PROTOS_REVISION=2d8f04a8a01ff8639e98e03fba9a176170d54936
CONTROL_PROTOS_VERSION=0.3.89-SNAPSHOT

INTERVENTION_PROTOS_REVISION=cf9b39b25dc9a3c4cd1c538749c3a363760ae45b
INTERVENTION_PROTOS_VERSION=0.3.96-SNAPSHOT

I072_FINAL_PROJECT_RECORD_REVISION=ce57c54d7c9a7135b5d787844a9a5888df38b01c

CONTROL_VARIANT=baseline
INTERVENTION_VARIANT=baseline
PATCHES_APPLIED=none
```

The benchmark evidence revision retains the full raw run, ordered logs,
stationarity table, checksums, image identities, source identities, and derived
summary.

## Measurement contract

```text
PERF010A_POST_I072_FPRIME_COMPARATOR=PASS
PERF010A_POST_I072_FPRIME_EVIDENCE_STATUS=RETAINED
PERF010A_POST_I072_FPRIME_FULL_MATRIX_EVIDENCE_UNIT=YES

WARMUP=120
STEADY=100
OPERATION_COUNT=10000
BLOCK_ORDER=A,B,A,B

BOTH_PRODUCT_VARIANTS_BASELINE=YES
PATCHES_APPLIED=NO
WORKLOAD_SOURCE_IDENTITY=PASS
```

The retained comparator uses the existing paired-control formula:

```text
canonical improvement =
  control canonical - intervention canonical

control movement =
  control workload-control - intervention workload-control

paired-control effect =
  canonical improvement - control movement
```

Positive values mean that the I072/F′ intervention is faster after subtracting
the movement observed in the corresponding workload-control pair.

## Retained per-workload result

| workload | role | samples | min % | max % | median % | MAD % | order effect |
|---|---|---:|---:|---:|---:|---:|---|
| `micro/slot-read` | negative coverage | 4 | -0.2892 | +31.5646 | +8.6025 | 5.7131 | DETECTED |
| `micro/closure-call` | primary | 4 | -3.7263 | +0.5080 | -0.9754 | 0.9588 | DETECTED |
| `micro/method-call` | primary | 4 | -4.3103 | +1.1318 | -2.3899 | 1.4878 | NOT_DETECTED |
| `runtime/monomorphic-dispatch` | primary | 4 | -6.6638 | +0.7208 | -4.4496 | 1.7341 | DETECTED |

The three primary call workloads therefore show no retained material positive
effect from the completed F′ implementation. Their paired-control medians are
approximately neutral to slightly negative, and every individual primary block
remains within ordinary single-digit percentage movement.

The positive `slot-read` median is negative-coverage evidence, not a primary
I072 call-path result, and it also reports an order effect.

## Interpretation boundary

The benchmark harness deliberately retains:

```text
I072_FPRIME_CAUSAL_EFFECT=NOT_CLASSIFIED
I072_FPRIME_BIG_COST=NOT_CLASSIFIED
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
NEXT_CAUSAL_BOUNDARY=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
```

This project-record interpretation does not rewrite those harness fields.

The order effects in `closure-call` and `monomorphic-dispatch` prevent a
precise small-effect classification, and the exact direction of a low
single-digit effect should not be overinterpreted.

They do not, however, hide a large multiplicative improvement. Across all
primary blocks the observed paired-control movement remains ordinary
single-digit percentage scale. The completed F′ intervention therefore does
not produce an approximately 2x improvement, much less the order-of-magnitude
movement required to explain the historical dominant gap.

Accordingly, the dominant-cost candidate is closed as follows:

```text
EVIDENCE_VALID=YES

HARNESS_I072_FPRIME_CAUSAL_EFFECT=NOT_CLASSIFIED
HARNESS_I072_FPRIME_BIG_COST=NOT_CLASSIFIED

PROJECT_FPRIME_BIG_COST=NO
CURRENT_CANDIDATE_DOMINANT_GAP_CAUSE=NO
ORDER_OF_MAGNITUDE_RELEVANT=NO

SMALL_EFFECT_DIRECTION=UNRESOLVED_DUE_ORDER_EFFECT

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

This result does not invalidate PLAT040 or I072 as an architectural cleanup.
It establishes that the completed F′ representation/call-path correction is not
the dominant runtime cost that PERF010-A is searching for.

## Consequence

PERF010-A must return to dominant-cost discovery rather than continue refining
or reimplementing F′ merely because the expected speedup did not appear.

The next causal boundary is not selected by this evidence record:

```text
NEXT_ACTION=RETURN_TO_DOMINANT_COST_SEARCH
NEXT_CAUSAL_BOUNDARY=NOT_YET_SELECTED
```

A subsequent investigation should identify a cost that remains present after
I072 across the primary common workloads and then test that mechanism with the
same bounded causal discipline.

No new production optimization is authorized by this record.
