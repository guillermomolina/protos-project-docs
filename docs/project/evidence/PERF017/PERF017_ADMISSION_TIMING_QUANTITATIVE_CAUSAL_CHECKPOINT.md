# PERF017 — Admission timing quantitative causal checkpoint

Date: 2026-09-28

## Identity

WORK_ITEM=PERF017/#729

PROTOS_REVISION=7aaaec6923265c99723ce5bca064e5b3ab52b8c4

JFR_EVENT=protos.TestToolPerf017Admission

COMMAND=bin/protos test --jobs 16

LOGICAL_CASES=1263
LANES=16
JFR_EVENTS=10104
REPLACEMENT_PAIRS=1247
PAIR_COVERAGE=COMPLETE

## Quantitative result

T2-T1 mean 293.677 ms.
T4-T3 mean 267.349 ms.
Combined caller-domain serialization mean 561.026 ms, 64.58% of the mean replacement gap.
T5-T4 mean 306.316 ms.
T6-T5 mean 0.407 ms.
T7-T6 mean 0.639 ms.
T8-T7 mean 0.213 ms.
Total mean replacement gap 868.628 ms.

ADMISSION_DELAY_DOMINATED_BY_CALLER_QUEUE=YES
ADMISSION_DELAY_DOMINATED_BY_SOURCE_LOAD=NO
THREAD_START_LATENCY_MATERIAL=NO
CALLER_ACTOR_SERIALIZATION_QUANTITATIVELY_MATERIAL=YES

FOLLOW_UP=PERF019/#736
PERF017_STATUS=IN_PROGRESS


## Final scaling reconciliation

PERF019/#736 published the conforming recovery implementation at:

```text
PROTOS_REVISION=c7be048bab19409a139b1a1f6ad74d23f17b78d7
PROTOS_VERSION=0.3.115-SNAPSHOT
COMMAND=bin/protos test --jobs 16
```

The final workload remained identical in logical size and correlation shape:

```text
LOGICAL_CASES=1263
PASSED=1263
FAILED=0
LANES=16
JFR_EVENTS=10104
REPLACEMENT_PAIRS=1247
PAIR_COVERAGE=COMPLETE
```

Measured recovery:

```text
WALL_SECONDS=70.84
USER_SECONDS=763.34
SYSTEM_SECONDS=13.30
PROCESS_CPU=1096%

T2_T1_MEAN_MS=53.878
T4_T3_MEAN_MS=6.914
T5_T4_MEAN_MS=40.380
TOTAL_REPLACEMENT_GAP_MEAN_MS=104.643
CALLER_SERIALIZATION_MEAN_MS=60.792
```

The final causal mechanism is the interaction of two independent serialized
demands on the caller Actor path:

- Future completion / waiting-lane continuation required an exact targeted
  handoff to avoid a redundant Actor queue turn;
- fresh per-Case source acquisition required by D153 was being performed through
  caller-Actor filesystem/Future suspension-resume visits and was moved to the
  already-authorized host execution carrier.

Controlled experiments proved that either change alone merely relocates or
preserves enough queue pressure to leave end-to-end throughput near the old
ceiling. The conforming combination collapses T2-T1, T4-T3 and T5-T4 together,
recovers substantially higher host CPU use, and reduces wall time without
changing the work-conserving logical scheduler, the global `--jobs` bound,
fresh Process isolation, deterministic result order, Actor/Task/Future
semantics, or public Test Tool output.

The historical requirement to identify an exact old
`KNOWN_HIGH_SCALING_REVISION` was superseded by stronger direct causal
evidence on the current execution pipeline: controlled mechanism ablations plus
a conforming recovery implementation reproduce the reported high-CPU behavior
without requiring an inferred historical regression endpoint.

```text
PROTOS_OWNED_SCALING_CAUSE=ESTABLISHED
CAUSAL_MECHANISM=
    TARGETED_HANDOFF_PLUS_HOST_CARRIER_FRESH_SOURCE_READ
SCALING_RECOVERY_IMPLEMENTED=YES
SCALING_RECOVERY_MEASURED=YES
FULL_REPOSITORY_VALIDATION=PASS
PERF019_STATUS=CLOSED_COMPLETE
PERF017_STATUS=CLOSED_COMPLETE
```
