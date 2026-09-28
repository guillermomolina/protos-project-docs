# PERF017 — Work-conserving scheduler falsification checkpoint

Date: 2026-09-28

## Scope

This checkpoint corrects the causal interpretation published in
`PERF017_RUNTIME_SCALING_CAUSAL_CHECKPOINT.md` for PERF017 /
guillermomolina/protos#729.

The JFR/timing measurements in that earlier checkpoint remain retained evidence.
This record supersedes only the claim that the measured public
`bin/protos test --jobs 16` path was using the fixed-wave
`Runner.runBoundedWithLoader(...)` scheduler and that a whole-wave
`Future.all(...).value()` barrier therefore caused 63.92% of the observed
excess carrier span.

No product source was changed by this falsification checkpoint.

## Evidence identity

```text
WORK_ITEM=PERF017/#729

MEASURED_PROTOS_REVISION=e65509d38c8df65246e0b5d0cc241986541b49b9
MEASURED_PROTOS_VERSION=0.3.106-SNAPSHOT

CURRENT_INSPECTED_HEAD=43f883b9f244c635311c9e2a049b1b3cf65bae93

BASELINE_MAIN_PROTOS_BLOB=88270bc2e53fa7c38a48aa901ea4bbe1135dca9f
CURRENT_MAIN_PROTOS_BLOB=88270bc2e53fa7c38a48aa901ea4bbe1135dca9f

BASELINE_LOGICAL_CASE_RUNNER_BLOB=39e961087f573c70a2afec12ef17703ed6cd60cd
CURRENT_LOGICAL_CASE_RUNNER_BLOB=39e961087f573c70a2afec12ef17703ed6cd60cd
```

The public scheduling files are byte-identical between the measured revision and
the current inspected HEAD.

## Public production path at the measured revision

At the exact measured revision, the public Test Tool path is:

```text
bin/protos test --jobs N
    -> protos/tools/test/Main.protos
    -> LogicalCaseRunner.run(..., jobs, ...)
```

`Main.protos` does not route the public logical-Case run through
`Runner.runBoundedWithLoader(...)`.

`LogicalCaseRunner.run(...)` builds one flattened `workItems` sequence and one
shared `nextIndex`, then starts up to `maxInFlight` asynchronous pull lanes.
Each lane repeatedly:

```text
claim nextIndex
load the Case source
call executorAsync(...).value()
store completion at results[original logical index]
repeat while work remains
```

A lane that completes one Case can therefore claim and admit the next Case while
other lanes still have unrelated Cases in flight.

The `Future.all(...lanes)` in this runner is terminal aggregation over the lanes.
It is not an inter-wave admission barrier.

## Existing deterministic regression evidence

The measured baseline already contains:

`src/test/java/com/guillermomolina/protos/execution/ProtosTestToolLogicalCaseRunnerTest.java`

with the focused test:

`completedLanePullsNextCaseWithoutWaitingForUnrelatedRunningCase`.

With `maxInFlight=2`, that test deterministically establishes:

```text
first admitted
second admitted
first completes
third admitted while second is still pending
```

The same test also verifies that completion order does not replace logical result
order.

The same Java test class additionally contains:

`jobsCapacityIsGlobalAcrossSourceProjections`

which establishes that a freed global lane can admit work from the next source
projection without waiting for an unrelated still-running Case.

Owner-executed validation reported on 2026-09-28:

```text
COMMAND=mvn -Dtest=ProtosTestToolLogicalCaseRunnerTest test
RESULT=PASS
```

## Why the prior source attribution was wrong

`protos/tools/test/Runner.protos` does contain older bounded helpers with fixed
waves:

```text
runBoundedSimpleWithLoader(...)
runBoundedWithLoader(...)
```

Those helpers prepare a group, await `Future.all(...pending).value()`, project
the group, and then admit another group.

The prior runtime checkpoint inspected this real code but incorrectly treated it
as the scheduler used by the measured public logical-Case Test Tool path.

The code shape exists; the production-path attribution does not.

This distinction is decisive because PERF017 measured:

```text
COMMAND=bin/protos test --jobs 16
```

The requested fixed-wave implementation slice would therefore have modified a
non-owning/legacy scheduler path and would not have been a valid causal repair
for the measured public workload.

## Measurement evidence retained

The earlier runtime checkpoint remains authoritative for the measurements
themselves, including:

```text
TEST_CASES=1263
RESULT=1263 passed, 0 failed

WALL_SECONDS=106.44
USER_SECONDS=731.28
SYSTEM_SECONDS=13.61
PROCESS_CPU=699%
EFFECTIVE_PARALLELISM≈6.998

JFR_WALL_SECONDS=106.29
JFR_USER_SECONDS=703.32
JFR_SYSTEM_SECONDS=13.68
JFR_PROCESS_CPU=674%
JFR_EFFECTIVE_PARALLELISM≈6.745

PEAK_LIVE_CARRIERS=16
AVERAGE_LIVE_CARRIERS=3.883
AVERAGE_LIVE_CAPACITY=24.27%

LIVE_0_SECONDS=17.698
LIVE_1_SECONDS=26.741
LIVE_16_SECONDS=5.666

TRANSITIONS_INTO_LIVE_0=522
TRANSITIONS_INTO_LIVE_16=49
```

Those observations still require explanation.

## Reclassification of the scheduling model

The lifetime arithmetic from the earlier checkpoint remains a useful
counterfactual scheduling model:

```text
ACTUAL_CARRIER_SPAN_SECONDS=82.891407
IDEAL_WORK_CONSERVING_16_SECONDS=20.903276
IDEAL_FIXED_WAVES_16_WITH_INSTANT_ADMISSION_SECONDS=60.525044
```

However, the derived:

```text
FIXED_WAVE_BARRIER_NO_BACKFILL_PENALTY_SECONDS=39.621768
FIXED_WAVE_BARRIER_SHARE_OF_EXCESS=63.92%
```

must no longer be interpreted as attribution to the production scheduler.
It describes the difference between counterfactual schedule models applied to
the observed Case lifetimes; it does not prove that the measured production run
used the fixed-wave model.

Likewise, the earlier residual:

```text
SERIAL_ADMISSION_AND_OTHER_GAPS_PENALTY_SECONDS=22.366363
```

must not be treated as a complete causal decomposition because that decomposition
assumed the incorrect fixed-wave production model.

## Corrected causal state

```text
CURRENT_SCALING_CEILING=CONFIRMED
CURRENT_HEAD_25_4_REPRODUCES=YES
HOST_CPU_QUOTA_CAUSAL=NO

PUBLIC_LOGICAL_CASE_SCHEDULER_WORK_CONSERVING=YES
IMMEDIATE_BACKFILL=CONFIRMED
GLOBAL_JOBS_BOUND=CONFIRMED
PLAN_ORDER_INDEPENDENT_OF_COMPLETION_ORDER=CONFIRMED

FIXED_WAVE_HELPERS_EXIST=YES
FIXED_WAVE_HELPERS_OWN_MEASURED_PUBLIC_PATH=NO
FIXED_WAVE_PRODUCTION_CAUSE=FALSIFIED

ROOT_CAUSE_PROVEN=NO
PRIMARY_SHARE_63_92_PERCENT_CAUSAL_ATTRIBUTION=WITHDRAWN

PERF017_STATUS=IN_PROGRESS
```

This restores the bounded static conclusion that the production
`LogicalCaseRunner` is work-conserving and leaves the low sustained carrier
occupancy unexplained.

## Next causal discriminator

The next investigation should trace the exact interval between one exact Case
carrier becoming terminal and the next replacement carrier actually starting.

The production handoff to discriminate is conceptually:

```text
exact Case host work finishes
    -> caller-domain completion is enqueued/rematerialized
    -> execution Future becomes terminal
    -> suspended LogicalCaseRunner lane resumes from executorAsync(...).value()
    -> lifecycle/completion observers run
    -> results[index] is stored
    -> lane claims nextIndex
    -> sourceLoader(next Case)
    -> LogicalCaseDispatch / execution-requirement dispatch
    -> suite-native logical-Case execution preparation
    -> async exact execution submission
    -> PlatformThreadPerTaskSubmission.submit(...)
    -> replacement protos-test-exact-N carrier starts
```

The investigation should determine which portions are serial on
`protos-cli-guest`, which are per-Case preparation, which can overlap across
lanes, and which existing retained JFR events/samples can distinguish them.

Do not reintroduce the fixed-wave hypothesis unless current production-path
evidence changes.

The known per-Case Core bootstrap/temporary-Engine cost remains independently
owned by PERF018 / guillermomolina/protos#731 and is not reclassified here as the
PERF017 root cause.

## Specification effect

```text
NORMATIVE_SPECIFICATION_CHANGE=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
PUBLIC_TEST_TOOL_POLICY_CHANGE=NO
PRODUCT_FILES_CHANGED=NONE
```
