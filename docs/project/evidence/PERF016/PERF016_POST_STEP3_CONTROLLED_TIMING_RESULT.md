# PERF016 — Post-Step-3 25.4 controlled timing result

Date: 2026-09-30

## Scope

This record retains the final controlled timing checkpoint owned by PERF016/#727
after the structurally complete PERF015 canonical-Boolean represented-selection
optimization and PERF016 semantic-Integer represented-selection optimization.

It is non-normative performance evidence. It does not change Protos semantics or
authorize a further runtime architecture step.

## Exact identities

```text
WORK_ITEM=PERF016/#727
PARENT=PERF010-B/#722

CONTROL_REVISION=2e3f56fae3a500d3e4193e3345d8a82c35e4590e
CONTROL_VERSION=0.3.117-SNAPSHOT

INTERVENTION_REVISION=696b0f9797ebc8ced80009fb583027513852f55c
INTERVENTION_VERSION=0.3.119-SNAPSHOT

HARNESS_REPOSITORY=guillermomolina/protos-benchmarks
HARNESS_REVISION=ac59110d23cb4724e4aa438a2a5781aaf1b31a77

TOOLCHAIN=25.4.4.1.1
JDK=25.0.4.1.1
JVMCI=25.4-b23
CPU_AFFINITY=0
COUNTERBALANCE=A,B,A,B
```

The control-to-intervention product lineage contains exactly PERF015 and PERF016
and no unrelated product commit.

The harness revision is published on `guillermomolina/protos-benchmarks`. At
the time this project record is published, the generated
`results/perf016-post-step3/` reference evidence has not itself been published
to the benchmark repository. The numeric result below is retained here from the
maintainer-executed reference run and its PASS summary. Do not invent a
benchmark-evidence Git revision for that local result.

## Admission and correctness

The maintainer-executed reference run reported:

```text
PERF016_POST_STEP3_REFERENCE=PASS
PERF016_POST_STEP3_EVIDENCE_STATUS=RETAINED
PERF016_POST_STEP3_OUTPUT=results/perf016-post-step3

WORKLOAD_SOURCE_IDENTITY=PASS
CORRECTNESS=PASS
```

The reference used the exact published harness revision and the canonical
GraalVM/Graal/Truffle 25.4.4.1.1 environment.

## Retained timing summary

Positive direct effect means CONTROL minus INTERVENTION, so positive values
indicate the post-Step-3 intervention was faster for that view.

| workload | shared-driver/control effect | canonical effect | paired-control residual |
|---|---:|---:|---:|
| micro/slot-read | +5.3537% | -6.8125% | -10.1862% |
| micro/closure-call | +1.2583% | -6.0053% | -7.2338% |
| micro/method-call | +3.3384% | +3.7213% | -0.6383% |
| runtime/monomorphic-dispatch | +0.1797% | +0.4835% | -3.3498% |

Order-effect report:

```text
micro/closure-call:
  canonical_effect=NOT_DETECTED
  control_variant_effect=NOT_DETECTED
  paired_control_effect=NOT_DETECTED

micro/method-call:
  canonical_effect=NOT_DETECTED
  control_variant_effect=DETECTED
  paired_control_effect=NOT_DETECTED

micro/slot-read:
  canonical_effect=DETECTED
  control_variant_effect=NOT_DETECTED
  paired_control_effect=DETECTED

runtime/monomorphic-dispatch:
  canonical_effect=NOT_DETECTED
  control_variant_effect=DETECTED
  paired_control_effect=NOT_DETECTED
```

Stationarity summary:

```text
STATIONARITY_MAX_ABS_LAST_VS_FIRST_QUARTER_PERCENT=22.7265
```

## Interpretation

The structural work remains valid:

```text
BOOLEAN_REPRESENTED_SELECTION=COMPLETE
INTEGER_REPRESENTED_SELECTION=COMPLETE
```

The timing result does not establish a material, consistent post-Step-3
improvement.

The shared-driver/control variants show small positive direct movement ranging
from +0.1797% to +5.3537%, but that movement is not accompanied by a consistent
canonical-workload improvement. Two canonical workloads regress by about 6–7%,
while the other two improve by less than 4%.

Most importantly for attribution, the paired-control residual is negative for all
four workloads. The run also reports order effects in multiple views and a
maximum absolute first-quarter/last-quarter steady-state difference of 22.7265%.

This evidence therefore supports the campaign's existing stop gate:

```text
SHARED_DRIVER_TIMING_EFFECT=SMALL_MIXED_POSITIVE
COMMON_WORKLOAD_TIMING_EFFECTS=MIXED_NO_CONSISTENT_IMPROVEMENT
ORDER_EFFECT=PRESENT

STEP_3_TIMING_CLASS=ESSENTIALLY_UNCHANGED
```

`ESSENTIALLY_UNCHANGED` does not mean every measured percentage is zero. It
means the combined PERF015 + PERF016 intervention did not move the targeted
common workload/driver system into a materially better timing class with
consistent attribution.

No whole-language performance claim is made.

## PERF010-B routing

PERF010-B explicitly required a stop and re-evaluation if structurally complete
Boolean + Integer selection left the shared control/driver in the same cost
class.

That condition is now satisfied.

```text
POST_STEP3_CONTROLLED_TIMING_CHECKPOINT=RETAINED
STEP_3_TIMING_CLASS=ESSENTIALLY_UNCHANGED

STEP4_AUTHORIZED=NO
AUTOMATIC_STEP4_ALLOCATION=NO

PERF016_STATUS=CLOSED_COMPLETE
PERF010_B_NEXT_ROUTING=STOP_CAMPAIGN_AND_REEVALUATE
```

PERF015/PERF016 are not reverted: they remain semantics-preserving structural
optimizations whose predicted material timing effect was not established.

## Measurement-workflow observation

The reference also exposed two independent workflow costs that are relevant to
future performance engineering but are not PERF016 runtime findings:

1. a new dedicated comparator required substantial harness implementation for a
   comparison that conceptually should be parameterized by product endpoints and
   workload policy; and
2. the retained reference executes 64 timed units serially on one logical CPU,
   making the optimize -> measure -> decide loop operationally expensive.

Those workflow problems are routed separately and must not be used to reinterpret
this PERF016 timing result.
