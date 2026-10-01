# PERF025-A — direct-first RootTask dispatch implementation evidence

Date: 2026-10-01

## Identity

```text
WORK_ITEM=PERF025/#758
SLICE=PERF025-A
TRIGGERED_BY=PERF024/#756
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=d0045353d834257b5fb80581846b32aebd43c7e6
PROTOS_VERSION=0.3.130-SNAPSHOT
COMMIT=PERF025-A: direct-dispatch fresh RootTask entry
SEMANTIC_CHANGE=NO
```

This record retains the published implementation evidence for the first
product-side optimization promoted from PERF024. It is non-normative project
evidence; the Protos specifications remain the semantic authority.

## Trigger

PERF024 identified avoidable repeated-invocation machinery in the JVM embedding
path. One concrete component was the first segment of every fresh RootTask:

```text
create Task
-> register Task
-> enqueue Task
-> scheduler wakeup
-> dequeue same Task
-> begin dispatch
-> run first continuation
```

The runtime already supported direct dispatch for a fresh Actor-owned Task, so
PERF025-A targeted only that initial queue round-trip.

## Published implementation

Revision
`d0045353d834257b5fb80581846b32aebd43c7e6` introduces:

```java
ProtosActorExecutionDomain.runFreshRootTaskDirectly(...)
```

`ProtosRootTaskExecution.runRootTask(...)` now starts a fresh root Task through
that boundary instead of `createTask(...)`.

The new path:

1. constructs the fresh parentless Task;
2. performs the same registration and TERMINATED-Actor rejection used by
   ordinary Task creation;
3. records cancellation before first dispatch when the Actor is already
   TERMINATING;
4. enters the first segment with `beginDirectDispatch()`;
5. runs the first continuation directly; and
6. leaves any later suspension/resumption/re-runnable work on the existing
   ordinary Actor queue.

The existing `createTask(...)` contract remains queue-based for its other
callers.

## Preserved behavior

The published change explicitly preserves:

```text
Task identity and live-task accounting
Actor ownership
TERMINATED rejection
TERMINATING pre-start cancellation
first-execution cancellation observation
suspension and resumption
ordinary queue re-entry after suspension
terminal Task bookkeeping
RootTask ProtosExecutionOutcome mapping
```

No observable Protos semantic change is claimed.

## Regression coverage

`ProtosActorExecutionDomainTest` adds focused coverage for:

- direct first-segment execution with no initial runnable entry or scheduler
  wakeup;
- cancellation reaching terminal state;
- suspension followed by ordinary queued resumption;
- a fresh root Task started on an already TERMINATING Actor being cancelled
  before its ordinary continuation executes; and
- rejection of a fresh root Task on a TERMINATED Actor.

## Changed product surface

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosRootTaskExecution.java
src/main/java/com/guillermomolina/protos/runtime/ProtosActorExecutionDomain.java
src/test/java/com/guillermomolina/protos/runtime/ProtosActorExecutionDomainTest.java
```

The product version advances from `0.3.129-SNAPSHOT` to
`0.3.130-SNAPSHOT`.

Existing APL-1.0 notices are retained on modified Protos-owned source files.

## Validation evidence

The maintainer reported that the requested tests, including the integrated
suite, were all green before publication.

This documentation publication does not independently rerun the product test
suite.

```text
MAINTAINER_REPORTED_VALIDATION=PASS
PUBLICATION=PASS
PROTOS_REVISION=d0045353d834257b5fb80581846b32aebd43c7e6
```

## Result and routing

PERF025-A removes one demonstrated item of avoidable repeated-call runtime
machinery. It does not claim that the remaining embedding path is equivalent in
cost to GraalJS/GraalPy.

PERF025 remains open. The next planned slice is:

```text
PERF025-B — prepared reusable top-level callable
```

That slice owns elimination of per-invocation top-level slot lookup for callers
that explicitly retain a prepared callable, while preserving the existing
dynamic `invokeTopLevel(name)` behavior.

PERF025-C remains the later BUG008 carrier-tax / nested-CallTarget follow-up.

## References

- `guillermomolina/protos#758` — PERF025
- `guillermomolina/protos#756` — PERF024
- Protos revision `d0045353d834257b5fb80581846b32aebd43c7e6`
- historical BUG008 / `guillermomolina/protos#681` remains closed
