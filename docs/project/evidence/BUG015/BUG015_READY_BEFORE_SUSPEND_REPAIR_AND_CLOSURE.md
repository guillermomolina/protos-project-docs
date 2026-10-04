# BUG015 — ready-before-suspend lost-wakeup repair and closure evidence

## Scope

This record preserves the published repair and closure evidence for
`guillermomolina/protos#794` (**BUG015 — ProtosParallelRuntime ownedFuture can
lose ready-before-suspend wakeup**).

It follows the earlier diagnosis snapshot:

```text
DIAGNOSIS_RECORD=
  docs/project/evidence/BUG015/BUG015_READY_BEFORE_SUSPEND_RACE_DIAGNOSIS.md
DIAGNOSIS_PROJECT_RECORD_REVISION=
  5431ba7fcfd9933c11837bf548d3716c20159ecd
```

This record is non-normative. It records an implementation correctness repair;
it does not redefine Future, Task, P, or Standard Library semantics.

## Published product identity

```text
FORMAL_WORK_ITEM=BUG015
GITHUB_ISSUE=guillermomolina/protos#794

PROTOS_REVISION=89c459881b9523b9cffb4f51c37d7363f163d3ae
COMMIT_SUBJECT=BUG015: fix parallel ready-before-suspend lost wakeup
IMPLEMENTATION_VERSION=0.3.194-SNAPSHOT

OBSERVABLE_LANGUAGE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
NEW_DXXX_REQUIRED=NO
NEW_PLATXXX_REQUIRED=NO
```

The product commit is published on `guillermomolina/protos/main`.

## Defect repaired

The original producer continuation in
`ProtosParallelRuntime.ownedFuture()` could execute this interleaving:

```text
producer Task                         P completion
-------------                         ------------
completion.isReady() -> false

                                      completion becomes ready
                                      task.resume(completion)
                                      -> false because Task is still RUNNING

current.suspend(completion)
-> false because dependency is already ready
-> Task remains RUNNING

producer continuation returns anyway
```

That abandoned the producer Task in `RUNNING` state while it was no longer
executing, suspended, queued, or terminal. The owning Actor domain could then
block in `dispatchUntilTerminal()` forever.

The intermittent symptom was observed in focused concurrent execution of:

```text
protos/tests/conformance/library/collections/array-parallel-sort.protos
```

under `--jobs 8`.

## Published repair

The product commit changes the producer wait path so that:

1. the continuation returns only when `current.suspend(completion)` actually
   commits a suspension;
2. if `suspend()` returns false because the dependency became ready first, the
   still-`RUNNING` producer continues immediately and consumes the outcome;
3. if `suspend()` returns false but the Task is no longer `RUNNING` because
   cancellation won, the continuation returns without consuming the P outcome.

The production path therefore matches the already-established
ready-before-suspend contract used by `ProtosStandardFutureProtocol`.

The published source logic is equivalent to:

```java
if (!completion.isReady()) {
    if (afterPendingCheck != null) afterPendingCheck.accept(current);
    if (current.suspend(completion)) return;
    if (current.state() != ProtosTask.State.RUNNING) return;
}
Outcome o = completion.outcome();
```

The `afterPendingCheck` hook is used only by the package-private deterministic
test seam. Ordinary production callers pass no hook.

## Deterministic regression protection

The repair adds two focused Java regressions in
`ProtosParallelExecutionTest`.

### Ready wins before suspension

`readyBeforeSuspendCompletionIsConsumedWithoutLostWakeup` forces the
Completion to become ready after the producer has observed it pending but before
`suspend()`.

It verifies that:

- the producer is still `RUNNING` at the forced race point;
- the result Future resolves to the expected value;
- the producer reaches `COMPLETED`; and
- the Actor domain has no leaked live Task.

This converts the formerly probabilistic failure window into deterministic
coverage.

### Cancellation wins the same window

`readyBeforeSuspendCancellationWinsWithoutConsumingOutcome` requests producer
cancellation in the same controlled window before resolving the Completion.

It verifies that:

- the result Future becomes `CANCELLED`;
- no resolved value is consumed;
- the producer reaches `CANCELLED`; and
- the Actor domain has no leaked live Task.

This protects the state check that distinguishes a ready-before-suspend outcome
from a cancellation-winning false `suspend()` return.

## Changed product paths

The exact published BUG015 commit changes:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosParallelRuntime.java
src/test/java/com/guillermomolina/protos/execution/ProtosParallelExecutionTest.java
```

No specification file changed.

The Maven implementation version advances from `0.3.193-SNAPSHOT` to
`0.3.194-SNAPSHOT`, and the root changelog records the BUG015 repair and its
deterministic regression coverage.

## Validation

The maintainer reports that **all required local tests passed** for the
published BUG015 candidate.

This evidence preserves that report without fabricating command-level counts
that were not included in the publication handoff.

At closure reconciliation time, GitHub exposes no workflow run or combined
status for exact product revision
`89c459881b9523b9cffb4f51c37d7363f163d3ae`. No remote-CI PASS is therefore
claimed for BUG015.

The closure evidence is:

```text
MAINTAINER_REPORTED_REQUIRED_LOCAL_VALIDATION=PASS
REMOTE_WORKFLOW_RUN_OBSERVED=NO
REMOTE_COMBINED_STATUS_OBSERVED=NO
```

## I058 relationship

BUG015 was exposed by the source-backed `std:collections/Array.parallelSort`
tests introduced by I058, but the root cause was in the retained general
`Closure.parallel` / Future runtime substrate.

The repair therefore:

- does not serialize or exclude the Array parallel tests;
- does not change `parallelSort` topology;
- does not reintroduce privileged Core Array algorithms; and
- does not add a privileged P helper.

I058 remains responsible for its own remaining benchmark/closure evidence, but
BUG015 no longer blocks its retained P/Future correctness gate.

## Closure state

```text
ROOT_CAUSE_IDENTIFIED=YES
PRODUCT_FIX_PUBLISHED=YES
DETERMINISTIC_REGRESSION_TEST_ADDED=YES
CANCELLATION_WIN_PATH_COVERED=YES
MAINTAINER_REPORTED_REQUIRED_LOCAL_VALIDATION=PASS
SPECIFICATION_CHANGED=NO

DURABLE_RECORD_DECISION=REQUIRED
REQUIRED_DURABLE_PUBLICATION=PASS_AFTER_THIS_RECORD_IS_PUBLISHED
BUG015_CLOSE_READY=YES
```

BUG015 can close after the exact published project-record revision containing
this file is re-read and linked from the Issue closure comment.

AI assistance: this durable closure record was drafted with ChatGPT from the
published BUG015 product commit, the earlier BUG015 diagnosis record, live
GitHub Issue state, and the maintainer's report that all required local tests
passed.
