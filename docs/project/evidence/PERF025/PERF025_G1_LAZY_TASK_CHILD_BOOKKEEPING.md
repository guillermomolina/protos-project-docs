# PERF025-G1 — lazy Task child bookkeeping

Status: **PUBLISHED / VALIDATED BY MAINTAINER / PERF025 REMAINS OPEN**

This durable, non-normative record retains the published PERF025-G1 product
slice for `guillermomolina/protos#758`. G1 is the first bounded implementation
slice from the retained RootTask / Task / Actor bookkeeping investigation after
PERF025-F1 removed the universal hosted-session guest carrier.

## Identity

```text
DATE=2026-10-02
WORK_ITEM=PERF025-G1
OWNER=PERF025/#758

BASE_REVISION=59fcb8552bf294be4e5c8ffdbe7786cf93194490
BASE_VERSION=0.3.146-SNAPSHOT

PROTOS_REVISION=c8e0e0d59d5541007d733e8a0b53a3cf123e5818
PROTOS_VERSION=0.3.147-SNAPSHOT
COMMIT_SUBJECT=PERF025-G1: lazy Task child bookkeeping
```

The published commit is the direct child of PERF025-F1.

## Scope

G1 removes two unconditional per-Task allocations from the structured-child
bookkeeping path without changing Task, Future, Actor, cancellation, suspension,
or structured-concurrency semantics.

Before G1, every `ProtosTask` eagerly owned:

```java
private final Set<ProtosTask> children = new LinkedHashSet<>();
private final WaitDependency childDrain = new WaitDependency() {};
```

That physical cost was paid even by a childless Task, including the ordinary
fresh RootTask used by a hosted invocation.

G1 changes the representation to pay only when structured child work is
actually used.

## Published implementation

### Lazy child set

The published `ProtosTask` now stores:

```java
private Set<ProtosTask> children;
```

with the implementation invariant:

```text
children == null
    <=> no mutable structured-child collection has been materialized
```

The first `addChild(...)` materializes a `LinkedHashSet`.

Empty snapshots use `Set.of()`; they do not materialize the mutable child set
merely to represent the empty state.

The affected paths consistently use the lazy representation for:

```text
children()
addChild(...)
removeChild(...)
normal completion child drain
failure child cancellation/drain
cancellation observation
continuation-cancellation unwind
post-cleanup child drain
```

### Shared child-drain sentinel

The previous anonymous `WaitDependency` allocation per Task is replaced by:

```java
private static final WaitDependency CHILD_DRAIN = new WaitDependency() {};
```

The sentinel is backend-private and stateless. Identity checks remain scoped to
each Task's own `waitDependency` / `resumedDependency` state, so sharing the
sentinel does not merge Task lifecycle state.

The focused regression suite explicitly covers two distinct parents using the
same shared sentinel and proves that draining one parent's child does not wake
the unrelated parent.

## Exact product delta

Published changed paths:

```text
M CHANGELOG.md
M pom.xml
M src/main/java/com/guillermomolina/protos/runtime/ProtosTask.java
A src/test/java/com/guillermomolina/protos/runtime/ProtosPerf025G1LazyTaskChildrenTest.java
```

Version publication:

```text
IMPLEMENTATION_VERSION=0.3.147-SNAPSHOT
ROOT_CHANGELOG_ENTRY=YES
SPECIFICATION_CHANGE=NO
```

No Actor-domain registry, RootTask creation, scheduler, hosted-session, or
benchmark code changed in G1.

## Focused regression coverage

`ProtosPerf025G1LazyTaskChildrenTest` adds five focused cases:

1. a childless Task completes without materializing the child collection;
2. the first child materializes ownership and terminal child removal still
   empties the parent;
3. normal completion still waits for live children, including isolation between
   two parents sharing the static drain sentinel;
4. failure still cancels and drains children before terminal failure;
5. cancellation still requests child cancellation and drains before the parent
   reaches terminal cancellation.

This coverage directly protects the semantics that the physical compaction
could otherwise disturb.

## Preserved boundaries

G1 deliberately does not change:

```text
ROOT_TASK_EXISTS=YES
ROOT_TASK_REGISTRATION=UNCHANGED
ACTOR_DOMAIN_LIVE_TASKS=UNCHANGED
TASK_PARENT_CHILD_SEMANTICS=UNCHANGED
TASK_CANCELLATION_SEMANTICS=UNCHANGED
TASK_SUSPENSION_SEMANTICS=UNCHANGED
FUTURE_SEMANTICS=UNCHANGED
ACTOR_LIFECYCLE=UNCHANGED
HOSTED_SESSION_EXECUTION=UNCHANGED_FROM_F1
SCHEDULER=UNCHANGED
SPECIFICATION=UNCHANGED
```

In particular, this slice does **not** make hosted invocation taskless and does
not remove `liveTasks`. Those are independent residual RootTask-bookkeeping
surfaces.

## Validation status

The maintainer reported:

```text
PERF025-G1: lazy Task child bookkeeping pushed and tested
```

This record therefore retains:

```text
MAINTAINER_REPORTED_VALIDATION=PASS
PUBLICATION=PASS
PROTOS_REVISION=c8e0e0d59d5541007d733e8a0b53a3cf123e5818
```

No independent build or test rerun is claimed by this durable coordination
record.

## PERF025 consequence

G1 removes objectively unnecessary eager structured-child representation from
every Task while preserving the logical RootTask/Task model.

The retained RootTask investigation remains broader than G1. Current residual
product-side candidates remain separate implementation questions, including:

```text
fresh-direct Task state-machine bookkeeping
terminal root/outcome snapshot bookkeeping
liveTasks registration/removal representation
generic terminal-path checks known absent on a root Task
monitor/synchronization boundaries
```

No performance magnitude is claimed here. G1 was intentionally accepted as a
representation cleanup without requiring benchmark evidence.

```text
PERF025_G1_STATUS=COMPLETE
PERF025_STATUS=OPEN

BENCHMARK_REQUIRED_FOR_G1=NO
BENCHMARK_RESULT_CLAIMED=NO
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
```

The earlier coordination note that named PERF025-F2A benchmark work as the next
slice is superseded as scheduling direction by the maintainer's explicit choice
to continue the RootTask bookkeeping implementation line directly. Historical
measurement evidence remains immutable; no benchmark repository change is part
of G1.
