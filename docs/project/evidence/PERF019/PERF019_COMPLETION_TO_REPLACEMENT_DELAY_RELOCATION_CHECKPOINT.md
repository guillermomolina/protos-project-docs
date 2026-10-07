# PERF019 — Completion-to-replacement delay relocation checkpoint

Date: 2026-09-29

## Identity

WORK_ITEM=PERF019/#736
PARENT=PERF017/#729

PROTOS_BASE_REVISION=96f066c3689359afc0f03d6760de171bdfe2311f
PROTOS_BASE_VERSION=0.3.112-SNAPSHOT
EXPERIMENTAL_WORKTREE=DIRTY
PRODUCT_PUBLICATION=NONE

JFR_EVENT=protos.TestToolPerf017Admission
COMMAND=bin/protos test --jobs 16

This record retains an unpublished experimental checkpoint. The v3 source delta
was measured in the maintainer working tree and is not a published Protos
revision. It must not be cited as a released or merged implementation.

## Baseline

The retained PERF017 quantitative baseline was:

```text
LOGICAL_CASES=1263
LANES=16
REPLACEMENT_PAIRS=1247
PAIR_COVERAGE=COMPLETE

T2_T1_MEAN_MS=293.677
T3_T2_MEAN_MS=0.027
T4_T3_MEAN_MS=267.349
T5_T4_MEAN_MS=306.316
T6_T5_MEAN_MS=0.407
T7_T6_MEAN_MS=0.639
T8_T7_MEAN_MS=0.213

CALLER_SERIALIZATION_MEAN_MS=561.026
CALLER_SERIALIZATION_SHARE=64.58%
TOTAL_REPLACEMENT_GAP_MEAN_MS=868.628
```

PERF019 targeted the two caller-Actor waits T2-T1 and T4-T3.

## Experimental sequence

### v1 — remove the completion Task queue boundary

The first experiment materially reduced the first wait:

```text
T2_T1_MEAN_MS=32.093
T4_T3_MEAN_MS=362.795
TOTAL_REPLACEMENT_GAP_MEAN_MS=712.130
```

The first interval improved, but delay increased in the later Future-to-lane
continuation interval. This experiment did not remove the complete serialized
replacement path.

### v2 — attempted combined queue-path adjustment

The second experiment did not improve the path:

```text
T2_T1_MEAN_MS=218.963
T4_T3_MEAN_MS=292.473
TOTAL_REPLACEMENT_GAP_MEAN_MS=885.761
```

It was worse than the retained baseline total and was rejected.

### v3 — exact targeted continuation handoff

The third experiment introduced a private targeted runtime-completion path and
exact Future-waiter handoff. The focal implementation validation reported:

```text
STATIC_COMPILE=PASS
FOCAL_AFFECTED_JAVA_REGRESSIONS=PASS
```

The canonical Test Tool measurement completed successfully:

```text
LOGICAL_CASES=1263
PASSED=1263
FAILED=0

WALL_SECONDS=116.17
USER_SECONDS=789.84
SYSTEM_SECONDS=14.34
PROCESS_CPU=692%
MAX_RSS_KIB=8734144
```

JFR correlation was complete:

```text
JFR_EVENTS=10104

HOST_CASE_DONE_COUNT=1263
COMPLETION_TASK_BEGIN_COUNT=1263
FUTURE_TERMINAL_COUNT=1263
LIFECYCLE_TERMINAL_COUNT=1263
LIFECYCLE_STARTED_COUNT=1263
SUBMIT_ENTER_COUNT=1263
SUBMIT_RETURN_COUNT=1263
CARRIER_RUN_BEGIN_COUNT=1263

COMPLETE_CORRELATED_OPERATIONS=1263
LANES=16
FULL_T1_T8_OPERATIONS=1263
REPLACEMENT_PAIRS=1247
EXPECTED_REPLACEMENT_PAIRS=1247
PAIR_COVERAGE=COMPLETE
```

Mean decomposition:

```text
T2_T1_MEAN_MS=262.338
T3_T2_MEAN_MS=0.050
T4_T3_MEAN_MS=0.908
T5_T4_MEAN_MS=574.449
T6_T5_MEAN_MS=0.406
T7_T6_MEAN_MS=0.711
T8_T7_MEAN_MS=0.149

CALLER_SERIALIZATION_MEAN_MS=263.246
CALLER_SERIALIZATION_SHARE=31.38%
TOTAL_REPLACEMENT_GAP_MEAN_MS=839.012
```

Selected distributions:

```text
T2-T1  p50=63.986 ms   p95=1159.756 ms   max=3790.298 ms
T4-T3  p50=0.544 ms    p95=3.083 ms      max=17.200 ms
T5-T4  p50=81.167 ms   p95=4678.888 ms   max=7957.276 ms
TOTAL  p50=276.627 ms  p95=4794.778 ms   max=7996.914 ms
```

## Causal interpretation

The v3 exact handoff is strong evidence that the second targeted queue boundary
is removable as an implementation detail:

```text
T4_T3_BASELINE_MS=267.349
T4_T3_V3_MS=0.908
T4_T3_REDUCTION_APPROX=99.66%
```

However, the end-to-end replacement path did not receive the corresponding
gain:

```text
T5_T4_BASELINE_MS=306.316
T5_T4_V3_MS=574.449

TOTAL_BASELINE_MS=868.628
TOTAL_V3_MS=839.012
```

The removed T4-T3 delay substantially reappeared after lifecycle terminal and
before the next lifecycle start. Meanwhile T2-T1 remained large:

```text
T2_T1_BASELINE_MS=293.677
T2_T1_V3_MS=262.338
```

Together with v1, this falsifies the current optimization strategy of treating
T2-T1 and T4-T3 as independent bottlenecks and repairing one queue boundary at a
time. The experiments repeatedly move waiting time to another point in the same
completion-to-replacement pipeline.

This checkpoint does **not** establish that every interval is governed by one
physical queue or lock. It establishes that isolated queue-boundary reductions
have not recovered end-to-end admission latency and that the next investigation
must reason about the full T1-to-T6 path.

## Next discriminator

PERF019 should return temporarily to investigation before another implementation
attempt.

Map the exact path:

```text
T1 HOST_CASE_DONE
  -> T2 COMPLETION_TASK_BEGIN
  -> T3 FUTURE_TERMINAL
  -> T4 LIFECYCLE_TERMINAL
  -> T5 next LIFECYCLE_STARTED on the same lane
  -> T6 SUBMIT_ENTER
```

Classify every operation and transition as one of:

```text
GUEST_REQUIRED
ACTOR_SERIALIZATION_REQUIRED
HOST_ONLY
ACCIDENTAL_QUEUE_BOUNDARY
```

The investigation must answer which portions actually require caller-Actor
ownership or a later guest segment, which portions are Test Tool host
coordination, and whether completion plus replacement admission can cross a
smaller Actor-owned boundary without changing observable Actor/Task/Future or
Test Tool semantics.

Do not create another queue-priority or handoff experiment until this complete
T1-to-T6 causal map identifies the end-to-end serialization boundary.

## Source-load ablation — causal result

A controlled ablation then removed only the per-Case execution-time source
re-read from the Actor-local replacement path. It temporarily reused the
discovery-time source text and therefore deliberately violated the normal D153
fresh rematerialization boundary. The edit was measurement-only, was restored
immediately after the run, and is not a candidate implementation.

The workload remained complete:

```text
JFR_EVENTS=10104
HOST_CASE_DONE_COUNT=1263
COMPLETION_TASK_BEGIN_COUNT=1263
FUTURE_TERMINAL_COUNT=1263
LIFECYCLE_TERMINAL_COUNT=1263
LIFECYCLE_STARTED_COUNT=1263
SUBMIT_ENTER_COUNT=1263
SUBMIT_RETURN_COUNT=1263
CARRIER_RUN_BEGIN_COUNT=1263

COMPLETE_CORRELATED_OPERATIONS=1263
LANES=16
REPLACEMENT_PAIRS=1247
PAIR_COVERAGE=COMPLETE
```

Host-level result:

```text
WALL_SECONDS=77.44
USER_SECONDS=859.05
SYSTEM_SECONDS=14.11
PROCESS_CPU=1127%
MAX_RSS_KIB=12159152
```

Mean replacement decomposition:

```text
T2_T1_MEAN_MS=65.536
T3_T2_MEAN_MS=0.057
T4_T3_MEAN_MS=8.164
T5_T4_MEAN_MS=60.558
T6_T5_MEAN_MS=1.514
T7_T6_MEAN_MS=1.687
T8_T7_MEAN_MS=0.296

CALLER_SERIALIZATION_MEAN_MS=73.700
TOTAL_REPLACEMENT_GAP_MEAN_MS=137.812
```

Compared with the retained baseline, the ablation reduces the mean total
replacement gap from 868.628 ms to 137.812 ms and reduces wall time from
approximately 116 s to 77.44 s while raising effective process CPU utilization
from about 692% to 1127%.

The important causal observation is not merely the T5-T4 reduction. Removing the
Actor-local execution-time source read also collapses T2-T1 from 262.338 ms in
v3 to 65.536 ms. This is consistent with reducing aggregate demand on one
shared serialized Actor execution domain: once repeated source-I/O
suspension/resumption visits are removed, queueing pressure falls throughout the
same domain rather than merely moving to another interval.

```text
SOURCE_LOAD_ABLATION_WORKLOAD_COMPLETE=YES
SOURCE_LOAD_ABLATION_D153_CONFORMING=NO
SOURCE_LOAD_ABLATION_PRODUCT_CANDIDATE=NO
ACTOR_LOCAL_SOURCE_RELOAD_CAUSALLY_MATERIAL=YES
END_TO_END_QUEUE_PRESSURE_COLLAPSED=YES
CPU_PARALLELISM_RECOVERED_MATERIALLY=YES
```

The ablation therefore identifies the next implementation boundary:

- preserve D153 fresh per-Case source reconstruction and fresh semantic Process;
- do not cache discovery source as product behavior;
- move the fresh source acquisition out of the serialized caller-Actor lane
  path, or otherwise perform it without repeated caller-Actor suspension/resume
  visits;
- retain the existing work-conserving logical scheduler and jobs bound;
- re-measure with the same PERF017 JFR phases to prove that the conforming
  implementation preserves the ablation's scaling direction.

## Source-load ablation without v3 — interaction result

A second controlled ablation repeated the same discovery-source reuse experiment
with all v3 targeted continuation/handoff source changes temporarily removed and
the tracked implementation materialized from repository HEAD.

The temporary Maven-built launcher initially failed before Test Tool startup
because the experimental JAR lacked the already-versioned
`unicode17.bin` resource. The resource was restored into that temporary JAR
from repository HEAD without changing source, and the experiment was rerun.

The complete workload then passed:

```text
LOGICAL_CASES=1263
PASSED=1263
FAILED=0

WALL_SECONDS=115.54
USER_SECONDS=800.29
SYSTEM_SECONDS=14.63
PROCESS_CPU=705%
MAX_RSS_KIB=10704908
```

JFR correlation was complete:

```text
JFR_EVENTS=10104
COMPLETE_CORRELATED_OPERATIONS=1263
LANES=16
REPLACEMENT_PAIRS=1247
PAIR_COVERAGE=COMPLETE
```

Mean replacement decomposition:

```text
T2_T1_MEAN_MS=321.012
T3_T2_MEAN_MS=0.028
T4_T3_MEAN_MS=290.655
T5_T4_MEAN_MS=253.867
T6_T5_MEAN_MS=0.413
T7_T6_MEAN_MS=0.756
T8_T7_MEAN_MS=0.158

CALLER_SERIALIZATION_MEAN_MS=611.667
TOTAL_REPLACEMENT_GAP_MEAN_MS=866.889
```

This falsifies the hypothesis that moving/removing only the Actor-local source
reload is sufficient.

The four retained variants now show an interaction:

| Variant | T2-T1 ms | T4-T3 ms | T5-T4 ms | T1-T8 ms | wall s | CPU |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| retained baseline | 293.677 | 267.349 | 306.316 | 868.628 | ~116 | ~7 cores |
| v3 only | 262.338 | 0.908 | 574.449 | 839.012 | 116.17 | 692% |
| source ablation only | 321.012 | 290.655 | 253.867 | 866.889 | 115.54 | 705% |
| v3 + source ablation | 65.536 | 8.164 | 60.558 | 137.812 | 77.44 | 1127% |

Neither intervention alone materially recovers end-to-end throughput. Together
they materially reduce all major serialized replacement intervals and recover
parallel CPU utilization.

The supported causal interpretation is therefore:

```text
TARGETED_CONTINUATION_HANDOFF_NEEDED=YES
ACTOR_LOCAL_SOURCE_RELOAD_REMOVAL_NEEDED=YES
EITHER_CHANGE_ALONE_SUFFICIENT=NO
COMBINED_CHANGE_RECOVERS_SCALING=YES
```

This is consistent with a shared caller-Actor execution domain carrying more
than one independent serialized demand. Removing only one demand leaves the
remaining demand sufficient to saturate the same serial resource, so waiting
moves between intervals rather than disappearing.

Maintainer observation during the canonical runs additionally reported that CPU
utilization remained poor through most of the run but rose to full-host usage
during approximately the final ten seconds. This observation is not itself the
quantitative admission proof, but it is consistent with the causal model: once
there are no further logical Cases to admit, the serial replacement control
plane stops feeding new work and the already-running physical Case carriers can
consume the available CPUs without further admission pressure.

The conforming implementation target is now two-part:

1. retain the v3 exact targeted continuation/handoff behavior that removes the
   Future-terminal-to-lane resumption queue boundary while preserving Actor,
   Task and Future semantics;
2. preserve a fresh D153 source read for every Case, but perform that source
   acquisition on the host execution carrier after authorized physical source
   resolution rather than through `Runner.readSource(...).value()` in the
   caller Actor lane.

The discovery-source cache used by the ablations remains measurement-only and
must not enter production.

```text
PERF019_CAUSAL_INVESTIGATION_COMPLETE=YES
PERF019_IMPLEMENTATION_TARGET=
    TARGETED_HANDOFF_PLUS_HOST_CARRIER_FRESH_SOURCE_READ
D153_FRESH_PER_CASE_SOURCE=REQUIRED
DISCOVERY_SOURCE_CACHE=REJECTED_FOR_PRODUCT
SCHEDULER_REPLACEMENT_REQUIRED=NO
JOBS_SEMANTICS_CHANGE_REQUIRED=NO
```

## Published conforming implementation and closure evidence

The conforming two-part implementation was published in Protos as:

```text
PROTOS_REVISION=c7be048bab19409a139b1a1f6ad74d23f17b78d7
PROTOS_VERSION=0.3.115-SNAPSHOT
COMMIT=PERF019: recover Test Tool parallel admission scaling
```

The product implementation combines the two mechanisms identified by the
controlled experiments:

1. exact runtime-only targeted Future/lane continuation handoff for the waiting
   lane Task, while preserving unrelated Actor-local FIFO ordering and
   cancellation behavior;
2. fresh per-Case UTF-8 source acquisition on the authorized host execution
   carrier immediately before D153 fresh-Process declaration rematerialization,
   instead of transporting source text through the caller Actor.

The discovery-source cache used by the earlier ablation was not retained in the
product.

The canonical conforming measurement completed successfully:

```text
COMMAND=bin/protos test --jobs 16
LOGICAL_CASES=1263
PASSED=1263
FAILED=0

WALL_SECONDS=70.84
USER_SECONDS=763.34
SYSTEM_SECONDS=13.30
PROCESS_CPU=1096%
MAX_RSS_KIB=11303928
```

PERF017 JFR correlation remained complete:

```text
JFR_EVENTS=10104
COMPLETE_CORRELATED_OPERATIONS=1263
LANES=16
REPLACEMENT_PAIRS=1247
PAIR_COVERAGE=COMPLETE

T2_T1_MEAN_MS=53.878
T3_T2_MEAN_MS=0.069
T4_T3_MEAN_MS=6.914
T5_T4_MEAN_MS=40.380
T6_T5_MEAN_MS=1.367
T7_T6_MEAN_MS=1.812
T8_T7_MEAN_MS=0.222

CALLER_SERIALIZATION_MEAN_MS=60.792
TOTAL_REPLACEMENT_GAP_MEAN_MS=104.643
```

Relative to the retained PERF017 baseline, the final conforming candidate
reduces the mean replacement gap from 868.628 ms to 104.643 ms and the selected
caller-serialization intervals from 561.026 ms to 60.792 ms. Unlike the v1/v2/v3
isolated experiments, the large waits collapse together instead of relocating
to another interval.

The source-load result also reconciles the earlier baseline-only classification
`ADMISSION_DELAY_DOMINATED_BY_SOURCE_LOAD=NO`. That statement meant the
original aggregate T5-T4 interval could not be attributed to source loading
from the baseline trace alone. The later controlled interaction experiment
establishes a stronger causal result: Actor-local source acquisition is an
independent material serialized demand, but removing it alone is insufficient;
it must be reduced together with the Future/lane handoff demand.

Validation retained for the published delta:

```text
PRODUCTION_COMPILE=PASS
TEST_COMPILE=PASS
PERF019_FOCAL_JUNIT=PASS
D153_HOST_CARRIER_FRESHNESS_REGRESSION=PASS
H2B1_PHYSICAL_COMPLETION_ORDER_FOCAL=PASS
FULL_REPOSITORY_MAKE_TEST=PASS
GIT_DIFF_CHECK=PASS
```

The H2B1 scheduling fixture required only synchronization of its host-side
physical-completion observation: targeted handoff can finish the logical Future
before the submission wrapper's `finally` records physical completion. The
test still requires the exact physical completion order `[1, 0, 2]`; no
scheduler or result-order contract was weakened.

```text
CALLER_SERIALIZATION_PATH_REDUCED=PASS
T2_T1_MATERIAL_REDUCTION=PASS
T4_T3_MATERIAL_REDUCTION=PASS
PAIR_COVERAGE=COMPLETE
PUBLIC_LOGICAL_CASE_SCHEDULER_WORK_CONSERVING=YES
GLOBAL_JOBS_BOUND=PASS
RESULT_ORDER=PASS
FRESH_PROCESS_ISOLATION=PASS
D153_FRESH_PER_CASE_SOURCE=PASS
ACTOR_TASK_FUTURE_SEMANTICS=PASS
PUBLIC_TEST_TOOL_OUTPUT_CHANGE=NO
PERF018_SCOPE_MIXED=NO
BEFORE_AFTER_JFR_EVIDENCE=RETAINED
FULL_REQUIRED_VALIDATION=PASS
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
```

## State

```text
V1_ISOLATED_FIRST_BOUNDARY_STRATEGY=INSUFFICIENT
V2_COMBINED_ATTEMPT=REJECTED
V3_EXACT_HANDOFF_MECHANISM=LOCALLY_EFFECTIVE
V3_END_TO_END_RECOVERY=INSUFFICIENT
DELAY_RELOCATION_OBSERVED_IN_ISOLATED_EXPERIMENTS=YES
FINAL_CONFORMING_IMPLEMENTATION=PUBLISHED
PROTOS_REVISION=c7be048bab19409a139b1a1f6ad74d23f17b78d7
PROTOS_VERSION=0.3.115-SNAPSHOT
PAIR_COVERAGE=COMPLETE
PUBLIC_TEST_TOOL_RESULT=1263_PASS_0_FAIL
PERF019_STATUS=CLOSED_COMPLETE
NEXT_ACTION=NONE
```
