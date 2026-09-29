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

## State

```text
V1_ISOLATED_FIRST_BOUNDARY_STRATEGY=INSUFFICIENT
V2_COMBINED_ATTEMPT=REJECTED
V3_EXACT_HANDOFF_MECHANISM=LOCALLY_EFFECTIVE
V3_END_TO_END_RECOVERY=INSUFFICIENT
DELAY_RELOCATION_OBSERVED=YES
PAIR_COVERAGE=COMPLETE
PUBLIC_TEST_TOOL_RESULT=1263_PASS_0_FAIL
PRODUCT_PUBLICATION=NONE
PERF019_STATUS=IN_PROGRESS
NEXT_ACTION=T1_TO_T6_END_TO_END_CAUSAL_MAP
```
