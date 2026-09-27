# BUG010 — Test Tool progress streaming implementation evidence

Date: 2026-09-27

## Scope

This record retains the published implementation evidence for
BUG010 / guillermomolina/protos#728.

BUG010 tracks the maintained Test Tool defect where normal D120 progress was
documented as live Tool stderr output but was not observable until the logical
test invocation was effectively complete.

This record is non-normative. Live lifecycle state remains in GitHub Issue #728.

## Evidence identity

```text
WORK_ITEM=BUG010/#728

PRODUCT_REPOSITORY=guillermomolina/protos
ALLOCATION_BASELINE_REVISION=cc76159a1bc9e2e1d46363f1c9047aa1034474df
PRODUCT_REVISION=eab6a367c16dea0136e1aabb13a7da681c4839b0
PRODUCT_VERSION=0.3.104-SNAPSHOT

COMMIT_MESSAGE=BUG010: make Test Tool progress observable during logical Case execution
```

## Root cause

The defect was not caused primarily by terminal or `PrintStream` buffering.

Before the fix, `protos/tools/test/Main.protos` awaited:

```text
LogicalCaseRunner.run(...).value()
```

until every logical Case had completed. Only afterward did it create each
suite's D120 `Progress` state, group the already-completed logical results by
suite, and replay them through `Progress.observer(...)`.

Consequently even the initial:

```text
[phase] 0/N
```

could not be emitted while logical Case execution was still in progress.

The existing logical runner already had the required temporal boundary:
individual Case execution became terminal inside each lane before the final
`Future.all(...)` completed, while each completion was retained at its original
logical index for the final result array.

The observed batching therefore came from deferred progress generation, not from
an already-generated progress stream being held by the CLI stderr backend.

## Published delta

The product commit changes exactly these seven files:

```text
CHANGELOG.md
pom.xml
protos/tools/test/LogicalCaseRunner.protos
protos/tools/test/Main.protos
src/test/java/com/guillermomolina/protos/execution/ProtosTestToolBug010ProgressTemporalVisibilityTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosTestToolFileSelectionMainAdoptionTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosTestToolTool009ALifecycleReporterTest.java
```

No specification file is changed.

The implementation version moves from `0.3.103-SNAPSHOT` to
`0.3.104-SNAPSHOT`.

## Implementation result

`LogicalCaseRunner.run(...)` gains one optional trailing internal
`completionObserver`.

When supplied, the observer receives:

```text
(index, completion)
```

at the point an individual logical Case reaches the runner's terminal boundary.
The `index` is the same stable logical index used for the returned
`results[index]` slot.

`Main.protos` now creates all suite D120 progress states and
`Progress.observer(...)` instances before logical scheduling begins. Its
`logicalCompletionObserver` uses the stable logical index to recover the
Case metadata and owning suite, classifies only that completion for presentation,
and immediately drives the corresponding suite progress observer.

The final result projection remains separate:

```text
logicalCompletions
  -> LogicalCaseResult.projectAll(...)
  -> logicalResults
  -> final exit classification
```

Thus physical completion timing can drive live progress without becoming the
TestPlan result-order contract.

Per-phase terminal summaries are still emitted through
`Progress.finishPhase(...)`, and invocation summary ownership remains in
`Progress.finishInvocation(...)`.

The existing Test Tool progress writer remains the Tool-owned stderr writer:

```text
TextWriter(process.stderr(), process.stderrEncoding())
```

Guest per-Case stdout/stderr capture is not changed by the product delta.

## Temporal regression

The new
`ProtosTestToolBug010ProgressTemporalVisibilityTest` uses deterministic
manually-resolved Futures rather than timing sleeps.

Its controlled sequence establishes:

1. `Progress.begin(...)` reports `[phase] 0/3` while the runner Future is
   still pending.
2. Only the first logical Case is resolved.
3. The reporting sink then contains `[phase] 1/3`.
4. The overall runner Future is still pending and later Cases remain incomplete.
5. Remaining Cases are resolved and the returned completion array remains in
   stable logical order.

This directly guards the BUG010 temporal property: normal progress becomes
observable before invocation completion while result ordering remains logical,
not physical-completion ordered.

## Follow-up assertion repair

Before publication, two existing source-structure tests required anchor updates
because BUG010 intentionally moved progress construction before scheduling.

A reported failure was traced to an assertion anchor searching for the literal
substring `LogicalCaseRunner.run(`, which also appeared in explanatory comments.
The final test uses `logicalCompletions:` as the unique scheduling anchor
immediately preceding the production scheduling call.

The maintainer reported a Python re-simulation of all ten ordering anchors and
the exact-count/text assertions from:

```text
ProtosTestToolFileSelectionMainAdoptionTest
ProtosTestToolTool009ALifecycleReporterTest
```

against the final source, with all simulated assertions passing.

## Validation state

Reported evidence before publication:

```text
GIT_DIFF_CHECK=PASS
SOURCE_ASSERTION_RESIMULATION=PASS
PRODUCT_PUBLICATION=PASS
PRODUCT_REVISION=eab6a367c16dea0136e1aabb13a7da681c4839b0
```

At the time this durable record was written, no actual final focal JUnit run or
integrated `make test` result for the exact published BUG010 candidate had been
reported in the active interaction, and GitHub exposed no associated CI status
checks or workflow runs for the commit.

Therefore this record deliberately does not claim the Issue's required FULL
validation gate has passed.

```text
FOCAL_EXECUTABLE_VALIDATION=NOT_RECORDED
FULL_VALIDATION=NOT_RECORDED
BUG010_CLOSURE=PENDING
```

The owning Issue requires the final top-level executable closure gate before it
can be closed.

## Specification and compatibility impact

```text
SPECIFICATION_CHANGED=NO
NEW_PUBLIC_REPORTING_SCHEMA=NO
GUEST_SEMANTICS_CHANGED=NO
TESTPLAN_RESULT_ORDER_CHANGED=NO
GUEST_STDOUT_STDERR_PRIVACY_CHANGED=NO
PERF014_CAUSATION_CLAIMED=NO
```

The change repairs already-documented Test Tool progress visibility and does not
introduce a new language or Tool reporting contract.

## Checkpoint state

```text
ROOT_CAUSE=ESTABLISHED
PRODUCT_IMPLEMENTATION=PASS
PRODUCT_PUBLICATION=PASS
TEMPORAL_REGRESSION=ADDED
STABLE_LOGICAL_RESULT_ORDER=PRESERVED
TEST_TOOL_STDERR_AUTHORITY=PRESERVED
GUEST_STREAM_PRIVACY=PRESERVED

FULL_VALIDATION=PENDING
BUG010_STATUS=REVIEW
BUG010_CLOSURE=PENDING
```
