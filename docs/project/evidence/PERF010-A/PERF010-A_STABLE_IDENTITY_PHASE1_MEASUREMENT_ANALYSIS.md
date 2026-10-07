# PERF010-A — stable-identity Phase 1 measurement-gate analysis

Status: retained investigation result. This record interprets the published Phase 1 measurement-system-admission Evidence Unit and determines whether the current harness is discriminating enough to proceed to the clean causal control/intervention comparison.

## Evidence identity

```text
BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks

HARNESS_REVISION=
  43cc0ba7b70b624cf30d30d2be2bd27144223b5d

BENCHMARK_EVIDENCE_REVISION=
  8886ba285b75954eb182342ef1a51dab52c54787

EVIDENCE_ROOT=
  results/perf010a-stable-identity-phase1/

PHASE1_PROTOS_REVISION=
  4c4aa95a5852119bd280ceb40483871d5d2cbb82

PREVIOUS_PUBLICATION_RECORD_REVISION=
  c9cd7ab396d00e71fed3f90e1366df6ffcdc827c
```

The published Phase 1 experiment is baseline versus a no-op image over the same Protos runtime path. Therefore every baseline/no-op difference in this record is measurement-system movement, not a product runtime effect.

## Classification

```text
SLICE_TYPE=INVESTIGATION

WARMUP_120_STABILITY_HYPOTHESIS=WEAKENED

MEASUREMENT_GATE=OPEN

DIRECT_WORKLOAD_DISCRIMINATION=
  COARSE_BUT_SUFFICIENT_FOR_LARGE_EFFECT_DETECTION
  NOT_SUITABLE_FOR_PRECISE_SMALL_EFFECT_ATTRIBUTION

INDIRECT_WORKLOAD_DISCRIMINATION=
  NOISY_BUT_USABLE_AS_NEGATIVE_COVERAGE

ORDER_EFFECT_INTERPRETATION=
  MATERIAL_ORDER_EFFECT_PRESENT_IN_BOTH_DIRECT_WORKLOADS
  COUNTERBALANCING_REQUIRED
  DOES_NOT_PREVENT_ORDER_OF_MAGNITUDE_DISCRIMINATION

STATIONARITY_INTERPRETATION=
  STEADY_WINDOWS_NOT_FULLY_STATIONARY
  DIRECT_CANONICAL_UNITS_SHOW_OCCASIONAL_LARGE_WITHIN_UNIT_DRIFT
  DRIFT_IS_TENS_OF_PERCENT_NOT_ORDERS_OF_MAGNITUDE

BIG_GAP_QUESTION=CAN_TEST_CURRENT_CANDIDATE

PATHOLOGY_CONFIRMED=YES
COMPILER_EFFECT_CONFIRMED=YES

CAUSAL_RUNTIME_SPEEDUP=NOT_MEASURED
DOMINANT_GAP_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED

PHASE2_ACTION=PROCEED_TO_CAUSAL_MEASUREMENT

CURRENT_EXTERNAL_BASELINE_REQUIRED=NOT_YET_APPLICABLE
```

## Per-workload discrimination evidence

Historical floors are from the retained warmup=20 no-op discrimination reference. New floors and no-op summaries are from the retained warmup=120 Phase 1 Evidence Unit.

| workload | historical floor | new floor | floor movement | median no-op effect | MAD | order effect | stationarity summary | class |
|---|---:|---:|---:|---:|---:|---|---|---|
| micro/slot-read | 38.9049% | 18.0369% | -20.8680 pp | -3.6378% | 4.3087% | NOT_DETECTED | canonical median absolute Q4-vs-Q1 movement 7.02%; max 39.57% | indirect |
| micro/closure-call | 34.8335% | 22.0052% | -12.8283 pp | +4.9642% | 8.4149% | NOT_DETECTED | canonical median absolute Q4-vs-Q1 movement 3.37%; max 8.82% | indirect |
| micro/method-call | 10.8286% | 20.7861% | +9.9575 pp | +2.3226% | 6.8021% | DETECTED | canonical median absolute Q4-vs-Q1 movement 4.73%; max 16.63% | direct |
| runtime/monomorphic-dispatch | 2.0901% | 18.3575% | +16.2674 pp | +1.9121% | 8.7433% | DETECTED | canonical median absolute Q4-vs-Q1 movement 3.39%; max 23.79% | direct |

The warmup=120 hypothesis is therefore weakened, not supported. The two direct workloads that matter for the intervention became less discriminating by the retained floor metric rather than more discriminating.

## Direct-workload block behavior

The direct paired-control values are:

```text
micro/method-call
  A  -0.9462%
  B  +5.5915%
  A -20.7861%
  B +12.6580%

runtime/monomorphic-dispatch
  A  -3.3951%
  B +18.3575%
  A -10.2675%
  B  +7.2192%
```

Both direct workloads show the same directional order pattern: A-order blocks are negative and B-order blocks are positive. The retained classifier therefore correctly reports an order effect for both. Phase 2 must preserve counterbalancing and must not collapse the experiment into one fixed control/intervention order.

Stationarity also remains imperfect. In `micro/method-call`, canonical timed units include first-quarter versus last-quarter movements of approximately -16.63%, -13.04%, and -10.72%. In `runtime/monomorphic-dispatch`, canonical units include approximately +23.79%, +11.41%, and -9.35%.

This is material drift for small-effect attribution, but it remains movement on the scale of tens of percent rather than factors or orders of magnitude.

## Why the measurement gate is open

The current harness is not sufficiently discriminating to make a precise attribution claim for a small effect near the observed no-op envelope. A causal result of only a few percent, or an effect comparable to the approximately 18-21% direct-workload floors, would require cautious classification and could not support a dominant-gap claim.

The PERF010-A gate, however, is not asking whether a 5-15% effect can be estimated precisely. It asks whether the stable-identity intervention could account for a large fraction of the orders-of-magnitude performance gap under investigation.

Phase 1 establishes that the measurement system can move by tens of percent in the relevant direct workloads. It does not show spontaneous movement by factors of 2x, 10x, or tens of times. The current system is therefore coarse but still capable of distinguishing a genuinely large causal effect from its measured no-op movement.

Accordingly:

```text
CURRENT_MEASUREMENT_SYSTEM_CANNOT_PRECISELY_QUANTIFY_SMALL_EFFECTS=YES
CURRENT_MEASUREMENT_SYSTEM_CAN_DISCRIMINATE_GENUINELY_LARGE_EFFECT=YES
```

No further default warmup-tuning chain is justified before the causal comparison.

## Causal boundary

The established compiler result remains:

```text
SEMANTIC_CALL_SITE=MONOMORPHIC
FAST_SPECIALIZATION_CACHE_CHURN=REMOVED
GENERIC_REPLACING_SPECIALIZATION_ACTIVATED=NO
CALLER_HELPER_PERMANENT_BAILOUT=REMOVED
COMPILER_LIFECYCLE_STABLE=YES
```

Those facts establish a real compiler pathology and a successful compiler-state intervention. They do not establish runtime materiality.

Phase 1 itself contains no causal control/intervention product comparison, so this record does not infer:

```text
CAUSAL_SPEEDUP
ATTRIBUTABLE_FRACTION
PERCENT_OF_EXTERNAL_GAP_EXPLAINED
```

The current state remains:

```text
PATHOLOGY_CONFIRMED=YES
COMPILER_EFFECT_CONFIRMED=YES
RUNTIME_SPEEDUP_CONFIRMED=NO
DOMINANT_GAP_CAUSE_CONFIRMED=NO
```

## Authorized Phase 2 experiment

The measurement gate now makes the clean two-product-revision comparator actionable:

```text
CONTROL_PROTOS_REVISION=
  2b3a88389da7228caed231a90b14091cf2841115

INTERVENTION_PROTOS_REVISION=
  3e8e6b565c95eb5098c2168d241536ba13ad19e9

CONTROL_VARIANT=baseline
INTERVENTION_VARIANT=baseline

NO_PATCHES
NO_GUARDED_DIAGNOSTICS
NO_COMPILER_TRACE
NO_JFR

WARMUP=120
STEADY=100
OPERATION_COUNT=10000

FRESH_CONTAINER_JVM_PER_TIMED_UNIT=YES
COUNTERBALANCED_CONTROL_INTERVENTION_ORDER=YES

PRIMARY=
  micro/method-call
  runtime/monomorphic-dispatch

NEGATIVE_COVERAGE=
  micro/slot-read
  micro/closure-call
```

The paired-control calculation remains:

```text
canonical improvement =
  control canonical - intervention canonical

control movement =
  control workload-control - intervention workload-control

paired-control effect =
  canonical improvement - control movement
```

The subsequent causal analysis must distinguish `REAL BUT SMALL`, `MATERIAL`, and `ORDER-OF-MAGNITUDE RELEVANT` from the observed scale without inventing a new arbitrary threshold. A few percentage points must not be promoted to a dominant-cause claim.

## Next slice

The previously retained harness-readiness record established that the current public causal runner cannot express the required two-product-revision baseline-versus-baseline counterbalanced comparison.

Because `MEASUREMENT_GATE=OPEN`, that deferred gap is now actionable.

```text
NEXT_SLICE_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos-benchmarks
NEXT_SCOPE=PHASE2_CLEAN_TWO_REVISION_COMPARATOR_ONLY
```

The implementation slice should add the smallest retained Phase 2 path needed to build and compare the exact control and intervention revisions above as clean baseline images, preserve canonical/control pairing, use fresh container/JVM timed units, counterbalance order, retain raw samples and exact image/revision identity, and expose a reference execution path suitable for the subsequent human-executed Evidence Unit.

It must not modify `guillermomolina/protos`, reinterpret the compiler-lifecycle evidence, calculate an external-gap fraction, or declare the dominant cause before Phase 2 measurements exist.
