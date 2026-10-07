# PERF017 — Runtime scaling causal checkpoint

Date: 2026-09-28

## Scope

This record retains the runtime/JFR causal checkpoint for PERF017 /
`guillermomolina/protos#729` after the GraalVM / Truffle 25.4 baseline was
adopted.

It supersedes only the earlier static checkpoint statement that the current
logical Case scheduler was work-conserving. The earlier record remains retained
as historical evidence of what was known before runtime measurement.

This checkpoint does not close PERF017. It establishes the Protos-owned runtime
mechanism to be falsified by the next implementation slice.

## Evidence identity

```text
WORK_ITEM=PERF017/#729
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=e65509d38c8df65246e0b5d0cc241986541b49b9
PRODUCT_VERSION=0.3.106-SNAPSHOT

GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1
JAVA_VERSION=25.0.4.1.1
VISIBLE_CPUS=16
CPUSET=0-15
CGROUP_CPU_QUOTA=unlimited

COMMAND=bin/protos test --jobs 16
TEST_CASES=1263
TEST_RESULT=1263 passed, 0 failed
```

The host/container evidence therefore does not support an external CPU quota,
cpuset restriction, or eight-core visibility limit as the explanation.

## Reproduced scaling ceiling

The first owner-executed baseline run at the exact product revision above
reported:

```text
USER_SECONDS=731.28
SYSTEM_SECONDS=13.61
WALL_SECONDS=106.44
PROCESS_CPU=699%
EFFECTIVE_PARALLELISM=(731.28+13.61)/106.44=6.998
AVAILABLE_CPU_UTILIZATION=43.7%
MAX_RSS_KIB=10205624
```

A second run with JFR profile recording retained the same qualitative ceiling:

```text
USER_SECONDS=703.32
SYSTEM_SECONDS=13.68
WALL_SECONDS=106.29
PROCESS_CPU=674%
EFFECTIVE_PARALLELISM=6.745
AVAILABLE_CPU_UTILIZATION=42.2%
MAX_RSS_KIB=10340704
```

The scaling ceiling is therefore reproducible on the selected 25.4 baseline and
is not an artifact of JFR itself.

## JFR discrimination

The 106-second JFR contained, among other events:

```text
jdk.ExecutionSample=21469
jdk.ThreadCPULoad=1562
jdk.ThreadStart=1386
jdk.ThreadEnd=1375
jdk.JavaMonitorWait=1187
jdk.ThreadPark=177
jdk.JavaMonitorEnter=3

jdk.graal.compiler.truffle.Deoptimization=19031
jdk.graal.compiler.truffle.Compilation=507
jdk.graal.compiler.truffle.CompilationFailure=202

jdk.GarbageCollection=146
jdk.YoungGarbageCollection=127
jdk.OldGarbageCollection=19
```

The initially suspicious Truffle deoptimization activity was not global:

```text
TOTAL_TRUFFLE_DEOPTIMIZATIONS=19031
protos-test-exact-258=19028
protos-cli-guest=3

DEOPT_REASON_UNKNOWN=18789
DEOPT_REASON_UNCOMMON_TRAP=242
```

The compilation-failure classes were:

```text
171  PermanentBailoutException: Too deep inlining, probably caused by recursive inlining
31   BailoutException: Code installation failed: code is too large
```

These are real optimization-path findings, but their concentration does not
explain the repository-wide CPU-scaling ceiling.

The low counts of `JavaMonitorEnter` and `ThreadPark` likewise do not support a
simple global monitor/park bottleneck as the dominant cause.

## Carrier lifecycle evidence

JFR paired all 1263 Test Tool exact-execution platform Threads:

```text
PAIRED_EXACT_CARRIERS=1263

CARRIER_DURATION_MIN_MS=26.670
CARRIER_DURATION_P50_MS=46.459
CARRIER_DURATION_P90_MS=359.446
CARRIER_DURATION_P99_MS=3893.919
CARRIER_DURATION_MAX_MS=8567.330

START_GAP_MIN_MS=3.847
START_GAP_P50_MS=46.170
START_GAP_P90_MS=126.475
START_GAP_P99_MS=345.488
START_GAP_MAX_MS=1625.032
```

The near equality of median Case lifetime and median carrier start gap is
material: short Cases often finish at approximately the same rate at which the
single runner path admits the next execution.

Time-weighted carrier occupancy was:

```text
PEAK_LIVE_CARRIERS=16
TIMELINE_SECONDS=82.891
AVERAGE_LIVE_CARRIERS=3.883
AVERAGE_LIVE_CAPACITY=24.27%

LIVE_0_SECONDS=17.698
LIVE_1_SECONDS=26.741
LIVE_16_SECONDS=5.666

TRANSITIONS_INTO_LIVE_0=522
TRANSITIONS_INTO_LIVE_16=49
```

Thus the previously retained fact that `jobs=16` can reach a peak of 16 live
Case carriers was correct but insufficient. The scheduler spends most of the
carrier timeline far below that peak.

## Source mechanism at the inspected revision

At `e65509d38c8df65246e0b5d0cc241986541b49b9`,
`protos/tools/test/Runner.protos` implements bounded Case scheduling in fixed
waves:

```text
while cases remain:
    prepare/admit up to maxInFlight Cases
    observations = Future.all(...pending).value()
    project the completed wave
    only then admit the next wave
```

This is bounded, but it is not globally work-conserving. A completed Case does
not cause immediate backfill while another Case in the same wave is still
running.

The PLAT023 host submission boundary in
`ProtosTestToolAsyncExecutionScope.PlatformThreadPerTaskSubmission` creates a
fresh platform Thread and starts it immediately for each already-admitted
execution. It has no second pool or queue. Therefore measured gaps between
successive `protos-test-exact-N` starts occur upstream of the host submission
carrier itself.

Execution sampling also showed substantial work on the serial
`protos-cli-guest` coordinator:

```text
PROTOS_CLI_GUEST_EXECUTION_SAMPLES=8259
EXACT_CARRIER_EXECUTION_SAMPLES=13188

protos-cli-guest top frame:
  5697 / 8259 = 68.98%
  ProtosBytecodeRootNodeGen$CachedBytecodeNode.continueAt
```

The coordinator also showed bytecode lowering/building and runner execution
frames. The evidence establishes a serial admission/preparation component but
does not attribute all of that component to one individual helper such as
`readSource`.

## Quantitative causal attribution

Using the 1263 exact measured Case carrier lifetimes from the JFR, three
schedules were compared without changing individual Case durations:

```text
ACTUAL_CARRIER_SPAN_SECONDS=82.891407
SUM_CASE_LIFETIME_SECONDS=321.882277
WORK_LOWER_BOUND_16_SECONDS=20.117642

IDEAL_WORK_CONSERVING_16_SECONDS=20.903276
IDEAL_FIXED_WAVES_16_WITH_INSTANT_ADMISSION_SECONDS=60.525044
```

This separates the observed excess over the work-conserving schedule:

```text
TOTAL_EXCESS_SECONDS=61.988131

FIXED_WAVE_BARRIER_NO_BACKFILL_PENALTY_SECONDS=39.621768
FIXED_WAVE_BARRIER_SHARE_OF_EXCESS=63.92%

SERIAL_ADMISSION_AND_OTHER_GAPS_PENALTY_SECONDS=22.366363
SERIAL_ADMISSION_AND_OTHER_GAPS_SHARE_OF_EXCESS=36.08%

FIXED_WAVES_IF_INSTANT_ADMISSION_SPEEDUP=1.370x
WORK_CONSERVING_IF_INSTANT_ADMISSION_SPEEDUP=3.965x
```

The fixed-wave effect is amplified by heterogeneous Case durations. Examples
from the same observed run include:

```text
wave 16: avg=171.990 ms, max=2042.693 ms, tail amplification=11.88x
wave 62: avg=742.710 ms, max=8567.330 ms, tail amplification=11.54x
wave 63: avg=361.368 ms, max=3151.110 ms, tail amplification=8.72x
```

In such a wave, already-finished slots remain unused until the slowest Case
finishes and `Future.all(...pending).value()` releases the whole wave.

## Causal conclusion

```text
CURRENT_SCALING_CEILING=CONFIRMED
CURRENT_HEAD_25_4_REPRODUCES=YES
HOST_CPU_QUOTA_CAUSAL=NO

ROOT_CAUSE_PROVEN=YES

PRIMARY_CAUSE=FIXED_WAVE_BARRIER_NO_BACKFILL
PRIMARY_ATTRIBUTED_EXCESS_SECONDS=39.621768
PRIMARY_SHARE_OF_EXCESS=63.92%

SECONDARY_CAUSE=SERIAL_ADMISSION_PREPARATION_AND_GAPS
SECONDARY_ATTRIBUTED_EXCESS_SECONDS=22.366363
SECONDARY_SHARE_OF_EXCESS=36.08%

PEAK_CARRIER_ADMISSION_16=TRUE
SUSTAINED_CARRIER_CONCURRENCY_16=FALSE
AVERAGE_LIVE_CARRIERS=3.883
AVERAGE_LIVE_CAPACITY=24.27%

GLOBAL_MONITOR_OR_PARK_BOTTLENECK_SUPPORTED=NO
TRUFFLE_DEOPTIMIZATION_GLOBAL_CAUSE=NO
GC_GLOBAL_CAUSE=NOT_SUPPORTED_BY_CURRENT_EVIDENCE
```

The earlier static checkpoint statement
`CURRENT_LOGICAL_RUNNER_WORK_CONSERVING=YES` is superseded by this runtime and
source-backed result.

## Next implementation boundary

The next bounded PERF017 implementation slice should change only the primary
mechanism first:

```text
fixed waves + whole-wave Future.all barrier
    ->
bounded work-conserving scheduling with immediate backfill
```

Required invariants:

- never exceed `maxInFlight` / public `--jobs N`;
- preserve fresh semantic Process isolation per logical Case;
- preserve Case expectation and observation semantics;
- preserve deterministic TestPlan result order independently of completion order;
- preserve case-private stdout/stderr and deterministic teardown;
- do not weaken Context/Process/Actor/Task semantics;
- do not mix the secondary serial-admission optimization into the same causal
  implementation slice.

The implementation is successful only if validation shows both semantic
correctness and the predicted scheduler-shape change: substantially higher
time-weighted live-carrier occupancy, disappearance of fixed-wave tail collapse,
and a material wall/CPU-scaling improvement on the same `--jobs 16` workload.

The remaining serial-admission/preparation component should then be remeasured
and addressed separately only if it remains material.

## Specification effect

```text
NORMATIVE_SPECIFICATION_CHANGE=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
PUBLIC_TEST_TOOL_POLICY_CHANGE=NO
```

If implementation discovers that immediate backfill would require a public Test
Tool policy or semantic change, the slice must stop and route that decision
through the normal design/platform authority rather than silently changing the
contract.
