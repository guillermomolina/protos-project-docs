# PERF010-A — stable-identity Phase 2 causal result

Status: RETAINED CAUSAL RESULT — CURRENT CANDIDATE CLOSED AS NOT THE BIG COST

This durable, non-normative record interprets the retained PERF010-A Phase 2
control/intervention Evidence Unit. It answers only whether the already-proven
stable-specialization identity pathology explains a large runtime cost relevant
to the dominant PERF010-A gap.

## Evidence identity

```text
BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks

HARNESS_REVISION=
  952cfd1290023a04108854163da30ba48b18b3c6

BENCHMARK_EVIDENCE_REVISION=
  fb938c1f3c4d299b125b9aa08c854ef494f2e920

EVIDENCE_ROOT=
  results/perf010a-phase2/

CONTROL_PROTOS_REVISION=
  2b3a88389da7228caed231a90b14091cf2841115

INTERVENTION_PROTOS_REVISION=
  3e8e6b565c95eb5098c2168d241536ba13ad19e9

CONTROL_VARIANT=baseline
INTERVENTION_VARIANT=baseline
PATCHES_APPLIED=none

PREVIOUS_PROJECT_RECORD_REVISION=
  caeb7a6b26376c7f2c86c24eae25523e561f5f65
```

The intervention revision contains the prepared Context-owned target
specialization plus the stable executable-identity cache key whose compiler
lifecycle was previously shown to remove the exact line-29 caller-helper cache
churn and permanent bailout.

## Evidence validity

```text
PERF010A_PHASE2_REFERENCE=PASS
PERF010A_PHASE2_EVIDENCE_STATUS=RETAINED
PERF010A_PHASE2_FULL_MATRIX_EVIDENCE_UNIT=YES

WARMUP=120
STEADY=100
OPERATION_COUNT=10000
BLOCK_ORDER=A,B,A,B

BOTH_PRODUCT_VARIANTS_BASELINE=YES
PATCHES_APPLIED=NO
```

The retained artifact contains the full four-workload matrix, raw samples,
per-timed-unit stationarity, logs, image identity, and derived summary.

The causal effect is the harness-defined paired-control effect:

```text
canonical improvement =
  control canonical - intervention canonical

control movement =
  control workload-control - intervention workload-control

paired-control effect =
  canonical improvement - control movement
```

Positive values mean that the intervention is faster after subtracting movement
measured by the workload-control pair.

## Primary causal workloads

### micro/method-call

Block-order sequence is A, B, A, B.

```text
PAIRED_CONTROL_EFFECT_BLOCKS_PERCENT=
  +6.1311
  +4.9668
  +21.8355
  +9.8978

PAIRED_CONTROL_EFFECT_MEDIAN=+8.0144%
PAIRED_CONTROL_EFFECT_MAD=2.4655%
SIGN_CONSISTENCY=POSITIVE_4_OF_4
ORDER_EFFECT=NOT_DETECTED
```

Relevant steady-unit stationarity remains imperfect but percentage-scale.
Across the canonical timed units for this workload, first-quarter to
last-quarter movement ranges from -13.2755% to +6.3729%; the largest absolute
canonical movement is 13.2755%. The workload-control timed units remain within
the same ordinary percentage/tens-of-percent regime.

### runtime/monomorphic-dispatch

Block-order sequence is A, B, A, B.

```text
PAIRED_CONTROL_EFFECT_BLOCKS_PERCENT=
  +9.4429
  +9.9241
  +12.5171
  +21.5429

PAIRED_CONTROL_EFFECT_MEDIAN=+11.2206%
PAIRED_CONTROL_EFFECT_MAD=1.5371%
SIGN_CONSISTENCY=POSITIVE_4_OF_4
ORDER_EFFECT=NOT_DETECTED
```

Canonical steady-unit first-quarter to last-quarter movement ranges from
-8.4695% to +1.4828%; the largest absolute canonical movement is 8.4695%.
Again, this is percentage-scale movement rather than a multiplicative effect.

## Negative coverage

The two non-primary workloads remain close to zero in median paired-control
effect and show order effects:

| workload | block effects (%) | median | MAD | order effect |
|---|---|---:|---:|---|
| micro/slot-read | +0.2533, -1.8730, +7.3428, -3.5386 | -0.8099% | 1.8959% | DETECTED |
| micro/closure-call | +1.5413, -0.1684, +4.4621, -3.6938 | +0.6864% | 2.3153% | DETECTED |

These workloads are negative coverage only. They are not used to rescue or
amplify the primary signal.

## Materiality classification

Phase 1 established direct-workload measurement movement on the order of
20.7861% for `micro/method-call` and 18.3575% for
`runtime/monomorphic-dispatch`. That system is not suitable for precise
small-effect attribution, but it is sufficient to discriminate a large
multiplicative or order-of-magnitude effect.

The Phase 2 primary medians are +8.0144% and +11.2206%. All eight primary block
effects are positive and neither primary workload reports an order effect, so a
small/moderate causal runtime benefit is consistent with the retained evidence.
Its exact percentage should not be overinterpreted because it is below the
Phase 1 small-effect discrimination floor.

The decision required by this slice does not depend on estimating that small
effect precisely. The intervention does not produce an approximately 2x
improvement, much less an order-of-magnitude improvement.

```text
SLICE_TYPE=INVESTIGATION_EVIDENCE

PHASE2_EVIDENCE_VALID=YES
PHASE2_EVIDENCE_STATUS=RETAINED

PRIMARY_EFFECT_SCALE=PERCENT_SCALE
CAUSAL_RUNTIME_EFFECT=ESTABLISHED_SMALL_OR_MODERATE

PATHOLOGY=REAL
CURRENT_CANDIDATE_BIG_COST=NO
PATHOLOGY_DISPOSITION=REAL_PATHOLOGY_BUT_NOT_THE_BIG_COST

ORDER_OF_MAGNITUDE_RELEVANT=NO

CURRENT_CANDIDATE_DOMINANT_GAP_CAUSE=NO
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED

ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
CURRENT_EXTERNAL_BASELINE_REQUIRED=NO

MORE_MECHANISM_INVESTIGATION_BEFORE_DECISION=NO
```

The independently established pathology therefore remains a legitimate
implementation-quality finding, but this causal measurement closes it as the
current candidate for the dominant PERF010-A runtime gap.

No further warmup, PIC-identity, compiler-lifecycle, permanent-bailout,
source-identity, JFR, IGV, `continueAt`, RootTag, or stationarity refinement is
required for this candidate.

## Consequence for the dominant-cost search

The earlier cross-runtime root-cause record ranked the architecture-level
candidate classes as:

```text
TIER_A=generic composed-call preparation and insufficient early call-site specialization
TIER_B=String-keyed heap activation/lexical lookup and per-call activation/context materialization
TIER_C=local Optional/List.copyOf/map-probe/continueAt-local microcosts
```

The stable-identity/prepared-target intervention was the strongest concrete
Tier-A candidate and is now causally closed as not the big cost.

The next PERF010-A work should therefore return to the strongest remaining
architecture-level candidate, Tier B. It should not return to deferred Tier-C
micro-ablations merely because they are easier to measure.

The next candidate must itself be tested with a bounded causal intervention
capable of producing either a large multiplicative improvement or the explicit
closure:

```text
REAL_OR_PLAUSIBLE_MECHANISM_BUT_NOT_THE_BIG_COST
```

If a semantics-preserving Tier-B intervention cannot yet be expressed safely,
only the minimum implementation needed to make that causal comparison
executable is justified; no separate methodology program should precede it.

## Reconciled live state

```text
PERF010A_STABLE_IDENTITY_PATHOLOGY=REAL
PERF010A_STABLE_IDENTITY_BIG_COST=NO

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO

NEXT_ACTION=
  STOP_CURRENT_CANDIDATE_AND_RETURN_TO_DOMINANT_COST_SEARCH

NEXT_CANDIDATE_CLASS=
  TIER_B_ACTIVATION_CONTEXT_REPRESENTATION
```
