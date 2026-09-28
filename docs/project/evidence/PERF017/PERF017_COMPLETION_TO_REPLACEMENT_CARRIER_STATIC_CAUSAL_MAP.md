# PERF017 — Completion-to-replacement-carrier static causal map

Date: 2026-09-28

## Scope

This checkpoint continues PERF017 / guillermomolina/protos#729 after
`PERF017_WORK_CONSERVING_SCHEDULER_FALSIFICATION_CHECKPOINT.md`.

It statically traces the current public `bin/protos test --jobs N` path from
one logical Case host execution completing until the same pull lane can submit
its replacement exact-execution carrier.

This is a read-only investigation result. No Protos product source, scheduler,
Test Tool policy, or normative specification is changed by this record.

## Evidence identity

```text
WORK_ITEM=PERF017/#729

PROTOS_REVISION=4a6df50e8577ebb0f3fc352df13dcbab368f955a
PROJECT_DOCS_PARENT_REVISION=6da3174c07565829fb73414d4a871f30e0a65bc2

MAIN_PROTOS_BLOB=88270bc2e53fa7c38a48aa901ea4bbe1135dca9f
LOGICAL_CASE_RUNNER_BLOB=39e961087f573c70a2afec12ef17703ed6cd60cd
```

The public scheduler remains the work-conserving `LogicalCaseRunner`; the
fixed-wave production attribution remains falsified.

## Static result

```text
LOGICAL_SCHEDULER_WORK_CONSERVING=YES
IMMEDIATE_BACKFILL_POLICY=YES

LANE_TASK_OWNERSHIP=
    each runLane.future() creates a distinct ProtosTask,
    but all lanes share the same caller Actor execution domain

ACTOR_LOCAL_CONTINUATION_CONCURRENCY=
    SERIAL between explicit suspension/terminal boundaries

STATIC_CAUSAL_MECHANISM_ESTABLISHED=YES
```

The mechanism is not a fixed-wave barrier. It is the convergence of completion
and refill work through one caller Actor execution domain before each replacement
host submission.

This establishes a real Protos-owned serialization boundary, but it does not yet
establish that this boundary quantitatively explains most or all of the retained
scaling loss. Runtime decomposition is still required.

## Public execution path

```text
bin/protos test --jobs N
  -> ProtosCli / bundled Test Tool
  -> protos/tools/test/Main.protos
  -> LogicalCaseRunner.run(..., jobs, ...)
  -> runLane.future() × min(jobs, logicalCases)
  -> sourceLoader(entry)
  -> LogicalCaseDispatch.logicalCaseExecutorAsync(...)
  -> binding.logicalCaseExecutionAsync(...)
     |
     + ordinary / actor / group / package
     |   -> ProtosTestLogicalCaseExecutionFacility
     |
     + process-snapshot
         -> ProtosProcessSnapshotLogicalCaseExecutionFacility
  -> Operation.submit()
  -> PlatformThreadPerTaskSubmission.submit(...)
  -> Thread.start()
  -> protos-test-exact-N
```

All current logical-Case execution requirements ultimately use the same
`PlatformThreadPerTaskSubmission`: one fresh platform Thread per accepted exact
execution, without another host executor queue or jobs cap.

## Completion-to-replacement causal map

The old exact carrier does not need to reach `ThreadEnd` before caller-domain
completion processing can begin.

The ordinary/Actor/Group/Package host completion path is:

```text
protos-test-exact-N
  -> ProtosTestLogicalCaseExecutionFacility.Operation.runHost()
  -> bridge.execute(request) returns/fails
  -> enqueueCallerCompletion(...)
       -> caller.executionDomain().createTask(...)
  -> finishHostAfterRun()
  -> carrier returns

caller Actor completion Task
  -> executeHostActionForRuntime(...)
  -> rematerialize(...)
  -> future.resolve/fail(...)
  -> ProtosFutureValue.transition(...)
  -> Waiter.ready()
  -> waiting lane ProtosTask.resume()
  -> caller Actor runnable queue
```

Process Snapshot follows the same caller-Future/completion-Task/submission shape.

Completion rematerialization and the lane continuation are therefore two
different caller-Actor turns:

```text
host completion
  -> queue completion Task
  -> caller dispatches completion Task
  -> Future terminal transition
  -> lane becomes runnable
  -> completion Task ends
  -> caller later dispatches lane Task
  -> lane continues after executorAsync(...).value()
```

Multiple exact carriers may finish concurrently and enqueue completion Tasks
faster than the caller Actor services them. Servicing those Tasks remains
serialized at the Protos Actor execution-domain level.

## Actor-local serialization

The N `LogicalCaseRunner` lanes are distinct `ProtosTask` objects, created by
ordinary `closure.future()`, but they all remain in the Test Tool caller Actor.

The normative Actor rule and implementation agree:

- Actor-local execution is cooperatively non-preemptive;
- one running Actor-local Task continues until an explicit suspension,
  completion, failure, or equivalent terminal boundary;
- another Task in the same Actor cannot simultaneously execute Protos code
  against that Actor state;
- `ProtosActorExecutionDomain.dispatchOne()` dispatches one cooperative segment;
- `ProtosActorScheduler` prevents two concurrent segments for the same Actor
  incarnation.

Thus the scheduler policy can be work-conserving while the machinery needed to
refill freed slots still passes through serialized caller-Actor turns.

## Work between Future terminalization and replacement submission

After the previous exact-execution Future becomes terminal, the awakened lane
must still perform or pass through, in order:

```text
resume from executorAsync(...).value()
  -> lifecycle "terminal" observer
  -> BUG010 completion/classification observer
  -> possible progress output
  -> results[index] store
  -> claim/increment shared nextIndex
  -> sourceLoader(next Case)
  -> lifecycle "started" observer
  -> LogicalCaseDispatch
  -> execution-requirement binding lookup
  -> logicalCaseExecutionAsync(...)
  -> Java request validation/preparation
  -> Operation.submit()
  -> PlatformThreadPerTaskSubmission.submit(...)
  -> Thread.start()
```

The non-suspending portions of this path execute serially in the caller Actor.

## Explicit pre-submission suspension points

The current path contains real suspension boundaries before a replacement
carrier is submitted.

### Completion/progress path

`Main.protos` defines:

```text
emitProgress: (line) => {
    progressWriter.writeLine(line).value()
    null
}
```

The BUG010 completion observer can therefore suspend on progress output after a
Case completes but before that lane reaches the next submission.

### Source loading

`Runner.readSource` performs:

```text
filesystem.open(...).value()

loop:
    issue up to 16 × file.read(65536)
    Future.all(read futures...).value()
    synchronously accumulate returned bytes
until EOF

file.close().value()
Encoding.UTF8.decode(bytes)
```

Source contents are re-read for the next logical Case before the lifecycle
`"started"` event and before `logicalCaseExecutionAsync(...)`.

Different lanes can overlap source I/O only while a lane is explicitly
suspended. The Protos-side byte accumulation, decode, result bookkeeping and
other non-suspending work remain serialized by the caller Actor.

## Additional synchronous pre-submit filesystem validation

For ordinary/Actor/Group/Package execution,
`ProtosTestLogicalCaseExecutionFacility.execute()` calls
`ProtosTestToolFileSelectionFacility.resolveAuthorizedSource(...)` before
`Operation.submit()`.

That authority check performs synchronous host filesystem metadata work,
including regular-file/readability checks and `Path.toRealPath()`.

Therefore the path contains both:

1. asynchronous source-content loading from the caller lane; and
2. synchronous physical-source authority validation before the exact carrier is
   created.

## Caller thread ownership

The bundled Test Tool root task is synchronously driven by
`ProtosRootTaskExecution` using:

```text
domain.createTask(...)
domain.dispatchUntilTerminal(rootTask, () -> false)
```

`ProtosCli.run()` executes the command on the platform thread named
`protos-cli-guest`.

Consequently the root Test Tool Actor's lane Tasks and caller completion Tasks
are normally dispatched on `protos-cli-guest`. This is consistent with the
retained JFR samples showing substantial guest execution on that thread.

Child Actors use the independent RuntimeHost-owned
`protos-actor-carrier-*` substrate. That cross-Actor pool does not make two
Tasks inside the Test Tool caller Actor execute simultaneously.

## PERF018 placement

For the ordinary suite-native logical Case route:

```text
Operation.submit()
  -> PlatformThreadPerTaskSubmission.submit(...)
  -> replacement carrier starts
  -> Operation.runHost()
  -> ProtosTestLogicalCaseAttemptBridge.execute(...)
  -> new ProtosCoreBootstrap().bootstrap(...)
  -> new ProtosProcessRuntime(...)
  -> runtimeHost.hostProcess(...)
  -> Case declaration/selection/execution
```

Therefore:

```text
PERF018_WORK_POSITION=AFTER_REPLACEMENT_CARRIER_START
PERF018_IS_PERF017_PRE_START_CAUSE=NO
```

PERF018/#731 remains valid and independent, but its per-Case Core
bootstrap/temporary-Engine work is structurally downstream of replacement
carrier start and cannot explain the pre-start admission interval itself.

## Interpretation of retained runtime evidence

Retained PERF017 evidence includes approximately:

```text
carrier duration p50=46.459 ms
carrier start gap p50=46.170 ms
average live carriers=3.883 / 16
time at live=0=17.698 s
time at live=1=26.741 s
transitions into live=0=522
```

The static mechanism above is compatible with that shape:

1. exact carriers execute independently;
2. their completions converge on caller-domain completion Tasks;
3. those completion Tasks are serviced serially;
4. each awakened lane requires its own later caller turn;
5. refill can suspend on progress and source I/O;
6. synchronous refill work before the next submit cannot execute on all 16 lanes
   simultaneously inside the same Actor;
7. only after that path reaches `Operation.submit()` is the replacement
   platform Thread constructed and started.

This is also compatible with the absence of a dominant Java monitor: the
serialization here is primarily semantic/cooperative Actor serialization rather
than necessarily monitor contention.

The retained runtime evidence does not yet quantify the relative contribution
of:

```text
completion-Task queue delay
completion rematerialization
lane runnable/dispatch delay
completion/progress bookkeeping
source-loading latency
host-side source-authority validation
Thread.start call-path latency
post-start OS/JVM scheduling latency
```

## Corrected causal state

```text
PUBLIC_LOGICAL_CASE_SCHEDULER_WORK_CONSERVING=YES
IMMEDIATE_BACKFILL=CONFIRMED
FIXED_WAVE_PRODUCTION_CAUSE=FALSIFIED

STATIC_CAUSAL_MECHANISM_ESTABLISHED=YES

MECHANISM=
    completion rematerialization and logical-lane refill continuations
    are serialized through the single caller Actor execution domain before
    each replacement host submission; source-loading and conditional progress
    waits add explicit suspension points in that refill path

QUANTITATIVE_DOMINANCE_PROVEN=NO
PERF017_STATUS=IN_PROGRESS
```

## Smallest next empirical slice

The next slice should instrument the current product path without changing
scheduler policy or observable semantics.

For Case A followed by replacement Case B on the same pull lane, capture:

```text
T1 = A_HOST_CASE_DONE
T2 = A_COMPLETION_TASK_BEGIN
T3 = A_FUTURE_TERMINAL
T4 = A_LIFECYCLE_TERMINAL
T5 = B_LIFECYCLE_STARTED
T6 = B_SUBMIT_ENTER
T7 = B_SUBMIT_RETURN
T8 = B_CARRIER_THREAD_START
```

Then derive:

```text
T2-T1 = completion-Task queue/service latency
T3-T2 = rematerialization + Future terminalization
T4-T3 = lane runnable/dispatch latency
T5-T4 = completion observer + progress + result store + nextIndex + sourceLoader
T6-T5 = dispatch + Java preparation + source-authority validation
T7-T6 = thread construction + Thread.start call path
T8-T7 = post-start JVM/OS scheduling latency
```

The implementation must correlate A to B by the caller lane `ProtosTask`
identity, because the same lane Task waits on A's execution Future and later
issues B's logical execution request.

Existing JFR can provide exact-carrier ThreadStart/ThreadEnd and thread names.
The retained JFR cannot provide completion-Task begin, exact Future
terminalization, lane identity, or submit-enter. Narrow new diagnostic/JFR
instrumentation is therefore required in `guillermomolina/protos`.

The slice must be instrumentation-only:

- no scheduling-policy change;
- no new semantic wait;
- no additional Future;
- no public Test Tool output;
- no new language/runtime capability;
- no PERF018 redesign;
- no attempt to optimize before measuring this decomposition.

## Next work classification

```text
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
NEXT_SLICE_OWNER=PERF017/#729
NEW_FORMAL_ISSUE_REQUIRED=NO

NEXT_SLICE_BOUNDARY=
    diagnostic timing/correlation instrumentation from host Case completion
    through same-lane replacement submit/start, followed by a controlled
    jobs=16 measurement on the existing workload
```

This is a bounded diagnostic implementation slice owned directly by PERF017; it
does not require allocating another formal Issue merely for the instrumentation.

## Specification effect

```text
NORMATIVE_SPECIFICATION_CHANGE=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
PUBLIC_TEST_TOOL_POLICY_CHANGE=NO
PRODUCT_FILES_CHANGED=NONE
```
