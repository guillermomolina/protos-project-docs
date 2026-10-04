# BUG015 — ready-before-suspend lost-wakeup diagnosis

## Scope

This snapshot records the diagnosis that allocated
`guillermomolina/protos#794` (**BUG015 — ProtosParallelRuntime ownedFuture can
lose ready-before-suspend wakeup**).

It is non-normative defect evidence. It does not define Protos semantics,
change the Future/P contract, or select durable runtime architecture.

## Evidence identities

```text
FORMAL_WORK_ITEM=BUG015
GITHUB_ISSUE=guillermomolina/protos#794
FAMILY=BUG
ISSUE_STATUS_AT_RECORDING=IN_PROGRESS
ISSUE_PRIORITY_AT_RECORDING=P1

CURRENT_PUBLISHED_PROTOS_HEAD=d7b8cbdc85ef9474395cf6eec4e70d2488e72916
CURRENT_HEAD_SUBJECT=PERF030-M: boundary lifecycle frame range scans
RACE_REVERIFIED_AT_CURRENT_HEAD=YES

REPRODUCTION_CHECKOUT_SHA=NOT_RECORDED_IN_TERMINAL_CAPTURE
OBSERVABLE_SEMANTIC_CHANGE=NO
SPEC_CHANGE=NO
FIX_PUBLISHED=NO
```

The maintainer's captured terminal transcript did not include a `git rev-parse`
for the exact local checkout that produced the stall. This record therefore
does not fabricate a product revision for the reproduction. Separately, the
defective source pattern was re-read and confirmed unchanged at the exact
published Protos revision above.

## Controlled reproduction

A focused invocation used only the seven Cases in:

```text
protos/tests/conformance/library/collections/array-parallel-sort.protos
```

with Test Tool concurrency:

```text
bin/protos test --jobs 8 \
  --file protos/tests/conformance/library/collections/array-parallel-sort.protos
```

A first controlled series produced 13 consecutive passes and then stalled on
run 14 in:

```text
array parallel sort is stable
```

A later capture loop produced 20 consecutive passes and then reproduced on run
21:

```text
[main] 6/7
[stalled] no Case completed for 30s; 1 Case in flight
[stalled] - protos/corpus/conformance:library/collections/array-parallel-sort.protos::array parallel sort orders without mutating the source
```

The changing stalled Case rules out a deterministic assertion-specific failure.
The same file also passes when isolated under `--jobs 1`, and focused
`array-parallel-map.protos` and `array-parallel-reduce.protos` runs passed
under `--jobs 1`.

## Thread-dump evidence

The run-21 stall was captured before terminating the JVM.

The remaining Test Tool Case carrier was:

```text
"protos-test-exact-1"
java.lang.Thread.State: WAITING
  at java.lang.Object.wait(...)
  at com.guillermomolina.protos.runtime.ProtosActorExecutionDomain.dispatchUntilTerminal(...)
  at com.guillermomolina.protos.execution.ProtosRootTaskExecution.runRootTask(...)
  at com.guillermomolina.protos.execution.ProtosTestLogicalCaseAttemptBridge.executeWithinProcess(...)
```

The Test Tool's root guest carrier was likewise waiting in
`ProtosActorExecutionDomain.dispatchUntilTerminal(...)`.

No live `protos-p-*` carrier appeared in the relevant dump at the stall. The
evidence therefore does not support a simple "all P carriers are occupied"
explanation. The remaining Actor domain had no runnable work capable of
terminalizing the root Task.

## Root cause

At current published HEAD,
`src/main/java/com/guillermomolina/protos/execution/ProtosParallelRuntime.java`
contains this `ownedFuture()` producer pattern:

```java
if (!completion.isReady()) {
    current.suspend(completion);
    return;
}
Outcome o = completion.outcome();
```

`ProtosTask.suspend(WaitDependency)` deliberately returns `false` when the
dependency becomes ready before suspension commits. In that case the Task
remains `RUNNING`.

The losing interleaving is:

```text
producer Task                         P completion
-------------                         ------------
completion.isReady() -> false

                                      outcome becomes ready
                                      task.resume(completion)
                                      -> false because Task is RUNNING

current.suspend(completion)
-> false because dependency is ready
-> Task remains RUNNING

producer continuation returns anyway
```

The Task is then neither terminal, suspended, nor runnable/enqueued. The Actor
dispatcher can reach `wait()` with no future event capable of making that Task
progress.

This is a lost-wakeup / ready-before-suspend correctness defect.

## Existing correct contract in the repository

At the same exact Protos revision,
`ProtosStandardFutureProtocol` already documents and handles the identical
ready-before-suspend race:

```java
if (current.suspend(dep)) return;
source.removeObserver(dep);
if (current.state() != ProtosTask.State.RUNNING) return;
```

Its comment explicitly states that a dependency may terminalize after the
pending check but before `suspend()`, and that `suspend()` returns false while
leaving the Task `RUNNING`.

BUG015 therefore does not need a new Future semantic rule. The repair should
make `ProtosParallelRuntime.ownedFuture()` obey the already-existing Task
suspension contract while preserving cancellation behavior.

## Classification and repair boundary

```text
DEFECT_CLASS=CORRECTNESS
BUG_FAMILY=BUG015
NEW_DXXX_REQUIRED=NO
NEW_PLATXXX_REQUIRED=NO
TEST_SERIALIZATION_IS_FIX=NO
ROOT_CAUSE_OWNER=ProtosParallelRuntime.ownedFuture
```

Serializing or excluding the parallel Array tests would only reduce the
probability of triggering the race and would leave the runtime defect
reachable by ordinary concurrent P/Future use.

The bounded repair should:

1. return from the producer continuation only when `suspend(completion)`
   actually suspends;
2. after a false return, continue only while the Task is still `RUNNING`, so a
   cancellation-winning path is not mistaken for dependency readiness;
3. consume the now-ready outcome and terminalize the Future/producer normally;
4. add a deterministic Java regression test that forces the
   ready-before-suspend interleaving instead of depending on probabilistic
   stress; and
5. re-run focused P/Future tests plus the concurrent
   `array-parallel-sort.protos --jobs 8` stress after the patch.

If implementation exposes an unresolved semantic or durable architecture
choice, BUG015 must stop at that boundary and route the question through the
normal Dxxx/PLATxxx process.

## Current state

```text
BUG015_ALLOCATED=YES
BUG015_ISSUE=794
ROOT_CAUSE_IDENTIFIED=YES
DETERMINISTIC_REGRESSION_TEST_ADDED=NO
PRODUCT_FIX_PUBLISHED=NO
BUG015_CLOSE_READY=NO

NEXT_SLICE=BUG015-A
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
```

AI assistance: this evidence record was drafted with ChatGPT from the
maintainer-provided focused stress output and JVM thread dump, the live BUG015
Issue, and exact source inspection of published Protos revision
`d7b8cbdc85ef9474395cf6eec4e70d2488e72916`.
