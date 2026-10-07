# PERF025-F1 — direct-caller hosted-session execution

Status: **PUBLISHED / VALIDATED BY MAINTAINER / PERF025 REMAINS OPEN**

This durable, non-normative record retains the published PERF025-F1 product
slice for `guillermomolina/protos#758`, consuming ratified PLAT046 Candidate B.

## Identity

```text
DATE=2026-10-02
WORK_ITEM=PERF025-F1
OWNER=PERF025/#758
DECISION_OWNER=PLAT046/#778

BASE_REVISION=ff618f0dd8ef680f7884c9145552f77912f040a2
BASE_VERSION=0.3.145-SNAPSHOT

PROTOS_REVISION=59fcb8552bf294be4e5c8ffdbe7786cf93194490
PROTOS_VERSION=0.3.146-SNAPSHOT
COMMIT_SUBJECT=PERF025-F1: direct-caller hosted-session execution

PLAT046_PROJECT_RECORD_REVISION=
  9ede8bdecfa1a60cf72a7c89644b2b4f8e461d92
SELECTED_CANDIDATE=B
```

The exact published commit is the direct child of the ratification baseline.

## Ratified architecture consumed

F1 implements the PLAT046 Candidate B minimum path:

```text
ORDINARY_PHYSICAL_CARRIER=CALLER_THREAD
DEDICATED_SECOND_GUEST_EXECUTION_THREAD_FOR_MINIMAL_PATH=NO

SESSION_SERIALIZATION=EXPLICIT_LOCAL_GATE
MULTI_CALLER_SAFETY=REQUIRED
CALL_VS_CLOSE_EXCLUSION=REQUIRED

PROCESS_CONTEXT_ENTRY=PRESERVED
ROOT_TASK_AND_ACTOR_SEMANTICS=PRESERVED

CHILD_ACTOR_CARRIERS=
  EXISTING_LAZY_RUNTIMEHOST_OWNED_SUBSTRATE

PREBUILT_STRONG_STACK_MECHANISM=NO
DEEP_RECURSION_10000_REQUIREMENT=RETAINED

SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
```

## Exact product delta

Published changed paths:

```text
M CHANGELOG.md
M pom.xml
D src/main/java/com/guillermomolina/protos/execution/ProtosGuestCarrier.java
M src/main/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedExecution.java
M src/main/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedSession.java
D src/test/java/com/guillermomolina/protos/execution/ProtosGuestCarrierTest.java
A src/test/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedSessionGateTest.java
M src/test/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedSessionLifecycleTest.java
```

The reusable session now owns:

```text
SESSION_GATE=java.util.concurrent.locks.ReentrantLock
PRIVATE_SESSION_GUEST_THREAD=NO
PRIVATE_SESSION_QUEUE=NO
PRIVATE_SESSION_PARK_UNPARK_HANDOFF=NO
```

Ordinary hosted session guest work runs on the thread invoking the public
session operation. The path still reaches:

```text
ProtosProcessRuntime.callInExecutionHostForRuntime(...)
  -> Process execution host
  -> Process Polyglot Context entry
  -> ProtosRootTaskExecution
```

so F1 changes physical placement rather than bypassing Process, Context,
RootTask or RootActor semantics.

## Session behavior

The published implementation serializes these session operations through the
same local gate:

```text
invokeTopLevel(name)
prepareTopLevel(name)
PreparedTopLevel.invoke()
close teardown after close cutover
```

The close path first publishes its idempotent cutover through
`closeStarted.compareAndSet(false, true)`, then acquires the local gate. This
preserves the intended boundary:

```text
AT_MOST_ONE_SESSION_GUEST_OPERATION_AT_A_TIME=YES
MULTI_CALLER_SAFETY=YES
CALL_VS_CLOSE_EXCLUSION=YES
NO_NEW_GUEST_OPERATION_AFTER_CLOSE_CUTOVER=YES
ALREADY_STARTED_OPERATION_MAY_FINISH=YES
```

The shutdown sequence remains:

```text
request Process termination
await Process termination
await execution-host / Context terminal disposition
close RuntimeHost
close module resolver
retain first cleanup failure
suppress later cleanup failures
```

## Carrier retirement

Repository publication proves:

```text
PROTOS_GUEST_CARRIER=RETIRED_FROM_PRODUCT
PROTOS_GUEST_CARRIER_TEST=RETIRED
```

The shared `ProtosStandaloneHostedExecution.GUEST_CALL_STACK_SIZE_BYTES`
constant remains because other execution surfaces still own the explicit stack
budget. F1 does not redesign those surfaces.

## Focused regression ownership

F1 adds
`ProtosStandaloneHostedSessionGateTest`, which covers the Candidate B
session-local gate, including caller-thread execution, multi-caller
serialization and call-versus-close exclusion.

The existing hosted-session lifecycle test remains and continues to check clean
Process/Context/RuntimeHost terminal disposition after close.

The old carrier-specific lifecycle assertion/test hook was removed with the
retired carrier abstraction.

## Versioning

F1 follows the repository implementation publication policy:

```text
IMPLEMENTATION_VERSION=0.3.146-SNAPSHOT
ROOT_CHANGELOG_ENTRY=YES
ARCHIVED_CHANGELOG_MODIFIED=NO
```

## Validation status

The maintainer reported F1 pushed and tested successfully.

This project record therefore retains:

```text
MAINTAINER_REPORTED_VALIDATION=PASS
PUBLICATION=PASS
PROTOS_REVISION=59fcb8552bf294be4e5c8ffdbe7786cf93194490
```

No independent rerun is claimed by this coordination record.

## Documentation drift observed after publication

One stale Javadoc sentence remains in the published
`ProtosStandaloneHostedExecution.executeFile(...)` method documentation. It
still states that the call runs guest work on a dedicated carrier thread,
although `executeFile()` is implemented through
`ProtosStandaloneHostedSession` and F1 now executes that path on the caller
thread.

Classification:

```text
F1_FUNCTIONAL_RESULT=PASS
STALE_JAVADOC=YES
SEMANTIC_CONTRADICTION=NO
PRODUCT_BEHAVIOR_CONTRADICTION=NO
DOCUMENTATION_FOLLOWUP_REQUIRED=YES
```

This drift does not invalidate the published F1 behavior or validation, but it
must not be copied forward as the current execution contract.

## PERF025 consequence

F1 satisfies the product-side consequence of PLAT046 Candidate B for the
reusable standalone hosted-session path.

PERF025 is intentionally **not closed** by this record.

The next evidence obligation is post-F1 measurement using the existing prepared
primitive radar. The benchmark repository currently has no F1-specific
measurement harness/retained result; its latest retained carrier A/B remains
PERF025-E3B.

```text
PERF025_F1_STATUS=COMPLETE
PERF025_STATUS=OPEN

NEW_BENCHMARK_METHODOLOGY_REQUIRED=NO
EXISTING_PREPARED_PRIMITIVE_RADAR=REUSE
POST_F1_REMEASUREMENT_REQUIRED=YES

BENCHMARK_CHANGE_IN_F1=NO
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
BUG008_REOPENED=NO
```

The post-F1 measurement must compare exact published revisions and keep
historical PERF025-D3/E3B evidence immutable.

## Boundaries preserved

F1 does not redesign:

- CLI guest-carrier policy;
- Test Tool per-Case carriers;
- Actor scheduling;
- child-Actor RuntimeHost carrier ownership;
- RootTask/RootActor semantics;
- deep-recursion representation;
- strong-stack fallback/lane/pool policy;
- Native Image PLAT045 policy;
- benchmark methodology;
- Protos language or Standard Library semantics.
