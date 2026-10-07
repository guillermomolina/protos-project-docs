# PERF025-G2 — RootTask fixed-cost bookkeeping compaction

## Status

Published product implementation evidence for PERF025 / `guillermomolina/protos#758`.

This record is non-normative. It retains the exact product publication identity,
the bounded RootTask/Task/Actor fixed-cost bookkeeping compaction, the explicitly
preserved first-execution cancellation boundary, and maintainer-reported
validation. It does not claim a measured performance magnitude.

## Exact product publication

```text
PROTOS_REVISION=32603c68229dca33e3fa2f49950162678bdc012d
PROTOS_PARENT_REVISION=52e43ff074aafd163240241762029159e8a043bc
PROTOS_VERSION=0.3.165-SNAPSHOT
COMMIT_SUBJECT=PERF025: compact RootTask fixed-cost bookkeeping
OWNING_ISSUE=guillermomolina/protos#758
SLICE=PERF025-G2
BASE_IS_EXACT_PARENT=YES
COMMITS=1
FILES_CHANGED=5
ADDITIONS=145
DELETIONS=11
```

The product publication is exactly one commit ahead of the preceding
`0.3.164-SNAPSHOT` node-aware guest Context publication.

## Bounded objective

The post-F1 pay-as-you-grow audit retained a separate RootTask/Task/Actor
fixed-cost line because that cost is paid once per hosted top-level invocation
rather than once per recursive guest call.

This slice reduces only proven bookkeeping overhead while retaining the physical
Task architecture and all observable Task/Actor/Future semantics.

The accepted changes are:

```text
LIVE_TASK_REGISTRY=IDENTITY_BACKED_SET
STRUCTURED_PARENT=FINAL_AFTER_CONSTRUCTION
INTERNAL_PARENT_LOOKUP=RAW_NULLABLE_REFERENCE
TERMINAL_STATE_SELF_REVALIDATION=REMOVED
FIRST_EXECUTION_PATH=UNCHANGED
TASKLESS_ROOT_ARCHITECTURE=NO
TASK_POOLING=NO
NOTIFY_ALL_REMOVAL=NO
```

## Identity-backed live Task registry

`ProtosActorExecutionDomain.liveTasks` previously used:

```java
new LinkedHashSet<>()
```

The published implementation uses:

```java
Collections.newSetFromMap(new IdentityHashMap<>())
```

This preserves exact Task identity membership and enumeration for Actor
termination while avoiding linked-set entry nodes and avoiding any incidental
dependence on insertion order.

The registry remains enumerable because Actor TERMINATING handling must still
snapshot all live Tasks and request their cancellation.

```text
LIVE_TASK_ENUMERATION=PRESERVED
ACTOR_TERMINATION_CANCELLATION=PRESERVED
TASK_IDENTITY_MEMBERSHIP=PRESERVED
ITERATION_ORDER_SEMANTIC=NO
```

## Immutable structured parent

A Task parent is established only at construction and is never reassigned.
The published field is therefore final:

```java
private final ProtosTask parent;
```

The public `parent()` projection remains available and still returns
`Optional<ProtosTask>`.

Internal runtime terminal bookkeeping now uses the package-local raw projection:

```java
ProtosTask parentForRuntime()
```

and removes the child from its parent without constructing an `Optional` or
reacquiring the child Task monitor merely to rediscover immutable provenance.

```text
PUBLIC_PARENT_API=PRESERVED
STRUCTURED_CHILD_REMOVAL=PRESERVED
PARENT_PROVENANCE_IMMUTABLE=YES
INTERNAL_OPTIONAL_ALLOCATION_REMOVED=YES
```

## Terminal publication revalidation

`publishTerminal(...)` is private and all of its callers establish the exact
terminal state under the Task monitor before entering the publication boundary.

The previous implementation reacquired the same monitor solely to check that
the just-established state was terminal and matched the supplied terminal enum.

That self-revalidation was removed. The externally relevant publication
ordering remains:

```text
1. owner.terminal(this)
2. associated Future terminalization when present
3. TerminalLifecycle callback when present
```

The slice does not remove associated-Future behavior, lifecycle callbacks,
parent-child release, Actor termination completion, or scheduler notification.

```text
TERMINAL_PUBLICATION_ORDER=PRESERVED
FUTURE_TERMINALIZATION=PRESERVED
TERMINAL_LIFECYCLE=PRESERVED
STRUCTURED_CHILD_RELEASE=PRESERVED
DOMAIN_NOTIFY_ALL=PRESERVED
```

## Rejected fresh-root specialization

An initial implementation candidate attempted to replace the existing fresh-root
entry sequence:

```text
beginDirectDispatch()
runContinuation()
```

with a specialized direct first-segment helper.

Review before compilation identified that the specialized helper would move the
point at which `continuationStarted` becomes true relative to the existing
Actor-termination/pre-start-cancellation race. The candidate was withdrawn
before product validation.

The published implementation deliberately retains:

```java
task.beginDirectDispatch();
task.runContinuation();
```

and the new regression test freezes that choice.

This is an explicit semantic boundary: PERF025-G2 does not trade the established
first-execution cancellation behavior for a smaller root-entry path.

```text
FRESH_ROOT_SPECIALIZATION=REJECTED
PRE_START_CANCELLATION_BOUNDARY=PRESERVED
ACTOR_TERMINATION_RACE_BOUNDARY=PRESERVED
FIRST_EXECUTION_SEMANTICS_CHANGE=NO
```

## Focused structural regression

The publication adds:

```text
src/test/java/com/guillermomolina/protos/runtime/
ProtosPerf025G2RootTaskBookkeepingTest.java
```

The regression freezes:

- final structured-parent provenance;
- the internal allocation-free parent lookup;
- identity-backed live Task membership;
- removal of the old LinkedHashSet live-task registry;
- retention of the existing fresh-root
  `beginDirectDispatch() + runContinuation()` path; and
- removal of the redundant terminal-state self-revalidation while retaining
  owner/Future publication.

The new source carries the repository APL Part 5 notice.

## Compilation and affected validation

The maintainer first ran:

```text
mvn -q -DskipTests test-compile
```

and reported successful completion.

The focused/affected test set then covered:

```text
ProtosPerf025G2RootTaskBookkeepingTest
ProtosActorExecutionDomainTest
ProtosPerf006B6A6A2TaskTerminalLifecycleTest
ProtosActorTerminationTest
ProtosStandardFutureProtocolTest
```

Reported result:

```text
Tests run: 45
Failures: 0
Errors: 0
Skipped: 0
FOCAL_VALIDATION=PASS
```

This set specifically retains evidence for direct RootTask behavior,
suspension/re-entry, Task terminal lifecycle ordering, Actor termination, and
Future cancellation/terminal behavior.

## Concurrent-main reconciliation

The substantive slice was initially prepared while product HEAD was:

```text
e584c2345a2327eb0952c198905436e0e2205cf2
0.3.163-SNAPSHOT
PERF025: preserve String Unicode proof metadata
```

During finalization, `main` advanced to:

```text
52e43ff074aafd163240241762029159e8a043bc
0.3.164-SNAPSHOT
PERF025: make hot guest Context lookup node-aware
```

The intervening publication did not overlap the two runtime Java paths or the
new focal test owned by this slice. The maintainer fetched and reconciled to the
new exact parent before assigning publication version `0.3.165-SNAPSHOT`.

```text
CONCURRENT_MAIN_MOVEMENT=1_COMMIT
OVERLAP_WITH_SUBSTANTIVE_PATHS=NO
RECONCILIATION=PASS
FINAL_PARENT=52e43ff074aafd163240241762029159e8a043bc
```

## Final integrated validation

The final candidate contained exactly:

```text
M CHANGELOG.md
M pom.xml
M src/main/java/com/guillermomolina/protos/runtime/ProtosActorExecutionDomain.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosTask.java
A src/test/java/com/guillermomolina/protos/runtime/ProtosPerf025G2RootTaskBookkeepingTest.java
```

`git diff --check` reported no errors.

The maintainer then ran the canonical integrated gate:

```text
make test
```

and reported:

```text
1284 passed
0 failed
FULL_VALIDATION=PASS
```

No product files changed as a side effect of validation.

## Product publication verification

The maintainer committed and pushed:

```text
COMMIT=32603c68229dca33e3fa2f49950162678bdc012d
COMMIT_SUBJECT=PERF025: compact RootTask fixed-cost bookkeeping
PUSH=52e43ff0..32603c68 HEAD -> main
WORKTREE_BEFORE_PUSH=CLEAN
PUBLICATION_PUSH=PASS
```

GitHub comparison against the exact parent confirms one commit with five files:

```text
CHANGELOG.md                                                    +21  -0
pom.xml                                                          +1  -1
ProtosActorExecutionDomain.java                                 +13  -2
ProtosTask.java                                                  +11  -8
ProtosPerf025G2RootTaskBookkeepingTest.java                     +99  -0

TOTAL_ADDITIONS=145
TOTAL_DELETIONS=11
```

## Performance-claim boundary

No benchmark, timing comparison, JFR capture, IGV inspection, allocation
profile, or exact-revision performance measurement was performed for this
slice.

The durable claim is structural:

```text
LIVE_TASK_IDENTITY_REGISTRY=YES
IMMUTABLE_PARENT_PROVENANCE=YES
INTERNAL_PARENT_OPTIONAL_REMOVED=YES
REDUNDANT_TERMINAL_RELOCK_REMOVED=YES

PHYSICAL_TASK_ARCHITECTURE=PRESERVED
FIRST_EXECUTION_PATH=PRESERVED
ACTOR_TERMINATION_SEMANTICS=PRESERVED
STRUCTURED_CONCURRENCY=PRESERVED
FUTURE_SEMANTICS=PRESERVED
TERMINAL_LIFECYCLE=PRESERVED

OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PLATFORM_DECISION_CHANGE=NO
PERFORMANCE_EFFECT_MEASURED=NO
BENCHMARK_RESULT_CLAIMED=NO
```

## PERF025 status

This bounded RootTask/Task/Actor fixed-cost bookkeeping slice is complete.
PERF025 itself remains open.

The separate residual non-benchmark investigation sequence recorded after
`0.3.164-SNAPSHOT` excluded this line while it was under independent
investigation; this publication supplies the durable implementation evidence for
that excluded line.

```text
PERF025_G2_ROOT_TASK_FIXED_COST_BOOKKEEPING=COMPLETE
PERF025_STATUS=OPEN
FINAL_BENCHMARK_EXECUTED=NO
```

## Materially inspected publication evidence

The coordinating publication review inspected:

- `guillermomolina/protos@32603c68229dca33e3fa2f49950162678bdc012d`;
- its exact parent `52e43ff074aafd163240241762029159e8a043bc`;
- the exact five-file parent-to-product comparison;
- the published `0.3.165-SNAPSHOT` product identity;
- `ProtosActorExecutionDomain`;
- `ProtosTask`;
- `ProtosPerf025G2RootTaskBookkeepingTest`;
- PERF025 / `guillermomolina/protos#758`;
- the post-`0.3.164` remaining-work assessment; and
- maintainer-reported compile, focused and integrated validation outcomes.

This record is evidence only and does not replace live GitHub Issue coordination.
