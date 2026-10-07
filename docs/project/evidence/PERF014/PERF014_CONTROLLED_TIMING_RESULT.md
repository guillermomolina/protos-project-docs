# PERF014 — Direct Closure-call controlled timing result

Date: 2026-09-27

## Scope

This record closes the causal timing gate for PERF014 / guillermomolina/protos#725.

PERF014 structurally stabilized direct Closure-call selection and execution, first with an
exact receiver-identity tier and finally with a definition-keyed second tier. The remaining
question was whether that structurally correct change reduced the established
Closure-call-minus-workload-control cost by the predeclared **clearly multiplicative** factor.

This record classifies the retained benchmark evidence. It does not change Protos semantics
and does not authorize PERF015.

## Exact evidence identity

```text
WORK_ITEM=PERF014/#725
PARENT=PERF010-B/#722

PRODUCT_REPOSITORY=guillermomolina/protos

CONTROL_PRODUCT_REVISION=0af8960363a557dad1b87968cf8a632e4716ee8a
CONTROL_PRODUCT_VERSION=0.3.102-SNAPSHOT

INTERVENTION_PRODUCT_REVISION=bcf9eda164d840b0a0b4201753fe5289347afa8a
INTERVENTION_PRODUCT_VERSION=0.3.105-SNAPSHOT

INTERVENING_UNRELATED_BUG010_REVISION=eab6a367c16dea0136e1aabb13a7da681c4839b0
INTERVENING_UNRELATED_BUG010_VERSION=0.3.104-SNAPSHOT

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
BENCHMARK_HARNESS_REVISION=92f5dbcf7d54ac048763f85946c973a25fbde241
BENCHMARK_EVIDENCE_REVISION=2c722043c9638adf837ca152f682b63b2853b3aa

RESULT_NAMESPACE=results/perf014-direct-closure-call/
```

The benchmark Evidence Unit is retained at the exact evidence revision above. The harness
revision and evidence revision are intentionally distinct: the harness was committed first,
then the retained result artifacts were published separately.

## Measurement contract

```text
CAUSAL_TWO_REVISION_COMPARATOR=YES
CONTROL_VARIANT=baseline
INTERVENTION_VARIANT=baseline
PATCHES_APPLIED=NONE

CPU_AFFINITY=0
NETWORK=disabled

OPERATION_COUNT=10000
WARMUP_ITERATIONS=120
STEADY_ITERATIONS=100
BLOCK_ORDER=A,B,A,B

FULL_FOUR_WORKLOAD_MATRIX=YES
WORKLOAD_SOURCE_IDENTITY=PASS
CORRECTNESS=PASS
EVIDENCE_STATUS=RETAINED
```

The workload source SHA-256 values match exactly between control and intervention for every
retained workload. The intervening BUG010 revision is recorded honestly in the intervention
lineage and does not relax the exact product revision/version or workload-source identity
checks.

## Primary causal formula

The retained comparator uses the already-established paired-control formula:

```text
canonical_improvement =
    control_canonical_median_ns
    - intervention_canonical_median_ns

control_movement =
    control_workload_control_median_ns
    - intervention_workload_control_median_ns

paired_control_effect =
    canonical_improvement
    - control_movement
```

A positive paired-control effect means the intervention is faster after subtracting the
workload-control's own measured movement between product revisions.

## Per-workload result

```text
micro/slot-read
  median paired-control effect = -1.5693%
  MAD                          =  2.5923%
  order effect                 = NOT_DETECTED

micro/closure-call
  median paired-control effect = +6.0407%
  MAD                          =  3.8416%
  min                          = +1.3438%
  max                          = +13.7850%
  order effect                 = DETECTED

micro/method-call
  median paired-control effect = +0.8368%
  MAD                          =  3.9933%
  order effect                 = DETECTED

runtime/monomorphic-dispatch
  median paired-control effect = +2.8522%
  MAD                          =  3.3233%
  order effect                 = NOT_DETECTED
```

The primary PERF014 workload is `micro/closure-call`. All four retained block-level
paired-control effects for that workload are positive:

```text
A0 = +3.0544%
B1 = +13.7850%
A2 = +1.3438%
B3 = +9.0271%
```

That supports a positive partial effect, but the A/B split is also why the retained harness
flags an order effect.

## PERF014-specific guest-call increment

The retained Evidence Unit also measures the guest Closure-call increment directly.

Aggregate medians:

```text
CONTROL_GUEST_CALL_INCREMENT_MEDIAN_NS=
31476780.75

INTERVENTION_GUEST_CALL_INCREMENT_MEDIAN_NS=
24077258.50

GUEST_CALL_INCREMENT_RATIO_OF_MEDIANS=
1.3073241187322053
```

Per-block increments and ratios:

```text
control increment ns:
[29497397.0, 33528350.0, 33456164.5, 29381204.5]

intervention increment ns:
[26724193.0, 20965636.5, 32221696.5, 21430324.0]

per-block reduction ns:
[2773204.0, 12562713.5, 1234468.0, 7950880.5]

per-block ratio:
[1.103771290680321,
 1.5992049657066219,
 1.0383117009372862,
 1.3710107462677652]

MEDIAN_PER_BLOCK_GUEST_CALL_INCREMENT_REDUCTION_NS=
5362042.25
```

The reduction value above is the median of the four per-block reductions, as defined by the
retained Evidence Unit. The `1.3073x` ratio is the ratio of the aggregate control and
intervention increment medians; these are separate descriptive summaries and are not treated
as algebraically interchangeable.

## Stationarity / order caveats

The Evidence Unit retained stationarity diagnostics for every timed unit.

```text
STATIONARITY_MAX_ABS_LAST_VS_FIRST_QUARTER_PERCENT=24.5978

ORDER_EFFECT:
  micro/slot-read                 NOT_DETECTED
  micro/closure-call              DETECTED
  micro/method-call               DETECTED
  runtime/monomorphic-dispatch    NOT_DETECTED
```

These diagnostics do not invalidate the retained run, but they constrain interpretation.
In particular, the closure-call result must not be presented as a precise single-number
speedup detached from its per-block spread and detected order effect.

## Predeclared prediction

PERF014 declared before timing:

```text
closure-call - workload-control guest-call increment
    -> must fall by a clearly multiplicative factor
```

The harness deliberately left that phrase unclassified and hardcoded no arbitrary
`2x`/`5x`/`10x` threshold.

The retained result instead shows:

```text
paired-control closure-call median = +6.0407%
guest-call increment ratio         = 1.3073x

per-block guest-call ratios:
1.1038x
1.5992x
1.0383x
1.3710x
```

This is a positive partial improvement, not a clearly multiplicative collapse of the
Closure-call tax.

## Final PERF014 classification

```text
DIRECT_CLOSURE_STRUCTURAL_RESULT=PASS
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
FINAL_REQUIRED_VALIDATION=PASS

PERF014_PARTIAL_POSITIVE_IMPROVEMENT=OBSERVED
PERF014_CLEARLY_MULTIPLICATIVE=NO
PERF014_TIMING_PREDICTION=FALSIFIED

PERF014_TIMING_RESULT=STRUCTURAL_SUCCESS_TIMING_PREDICTION_FALSIFIED
```

The result is still a valid successful experiment: PERF014 moved the implementation toward
the intended stable Truffle call architecture and measurably reduced part of the direct
Closure-call cost. What failed is the causal hypothesis that this mechanism accounted for a
clearly multiplicative share of the remaining large guest-call overhead.

## Routing consequence

PERF014's own closure contract explicitly requires a timing falsification to route back
through PERF010-B rather than silently stacking the next represented-selection optimization.

Therefore:

```text
PERF014_STATUS=COMPLETED
PERF015_AUTHORIZED=NO

NEXT_ROUTE=PERF010-B/#722_CAUSAL_REEVALUATION
PERF015/#726=BLOCKED
PERF016/#727=BLOCKED
```

PERF010-B must decide the next causal boundary from the accumulated evidence before
PERF015 is activated. The fact that PERF015 remains independently desirable for
representation-aware Boolean selection does not override the campaign's predeclared stop
discipline.
