# I082-E — synchronous foreign callback bridge closure

Date: 2026-10-07

## Work identity

~~~text
WORK_ITEM=I082
SLICE=I082-E
PROTOS_ISSUE=guillermomolina/protos#830
DECISION_AUTHORITY=D189/guillermomolina/protos#820
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative implementation evidence.

## Published product revision

~~~text
STARTING_PROTOS_REVISION=ddb2a30626f4ec23159f4a79f6d097061b09dcb0
ENDING_PROTOS_REVISION=ca650e230a72f12685d2dcdb3a927e1b5fba0a33
ENDING_VERSION=0.3.278-SNAPSHOT
SPECIFICATION_REVISION=0.1.447
COMMIT_SUBJECT=I082-E: add synchronous foreign callback bridge
~~~

The exact published commit changes:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeTaskExecution.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignArgument.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignCallback.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignCallbackScope.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignOperation.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProjectedOperations.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignValueAdmission.java
src/main/java/com/guillermomolina/protos/execution/ProtosInvocation.java
src/main/java/com/guillermomolina/protos/runtime/ProtosDynamicControlState.java
src/main/java/com/guillermomolina/protos/runtime/ProtosSignalException.java
src/main/java/com/guillermomolina/protos/runtime/ProtosTask.java
src/test/java/com/guillermomolina/protos/execution/ProtosForeignCallbackTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosForeignValueProjectionTest.java
~~~

No normative specification file changed.

## Implemented D189 callback bridge

I082-E implements the ratified dynamic synchronous callback baseline.

~~~text
CALLBACK_OWNER_ACTOR=ORIGINATING_ACTOR
CALLBACK_OWNER_TASK=SAME_CURRENT_TASK_WHEN_TASK_BACKED
NEW_TASK_ON_CALLBACK=NO
NEW_FUTURE_ON_CALLBACK=NO
NEW_ACTOR_TURN_ON_CALLBACK=NO
NEW_STRUCTURED_SCOPE_ON_CALLBACK=NO

CALLBACK_ARGUMENT_ADMISSION=D188
CALLBACK_RESULT_PROJECTION=LOSSLESS_MINIMAL
RETAINED_ASYNC_CALLBACK=NO
~~~

The generic projected foreign call boundary admits a Protos Closure argument as
an operation-scoped `ProtosForeignCallback`. Other ordinary Protos objects,
collections and foreign-module facades do not become generic callback exports.

The callback capability denotes the exact passed Closure and invocation remains
ordinary Protos invocation. No second guest callback identity or callable
institution is introduced.

## Dynamic lifetime and re-entry

Each entered foreign operation that actually receives a Closure owns an
independent callback scope.

The scope becomes live only at provider entry and expires on every operation
exit path. Expiry explicitly drops retained Closure, caller activation, session
and carried-outcome references.

~~~text
SEQUENTIAL_REENTRY=SUPPORTED
RECURSIVE_REENTRY=SUPPORTED
NESTED_FOREIGN_CALLBACK_REENTRY=SUPPORTED
CONCURRENT_REENTRY=REJECTED_BEFORE_GUEST_ENTRY
FOREIGN_CREATED_THREAD_ENTRY=REJECTED_BEFORE_GUEST_ENTRY
POST_RETURN_CALLBACK=REJECTED
CLOSED_SESSION_GENERATION=REJECTED
SESSION_REBIND=NO
RESURRECTION=NO
~~~

No Process-wide callback lock or callback registry is introduced.

## Task suspension and cancellation

The implementation uses the existing Task suspension commit boundary rather than
a callback-local approximation.

~~~text
SUSPENSION_COMMIT_GATE=ProtosTask.beginSuspensionCapture
ACTUAL_SUSPENSION_ACROSS_FOREIGN_EXTENT=REJECTED_BEFORE_COMMIT
NEW_CALLBACK_CANCELLATION_CHECKPOINT=NO
~~~

Task-backed callback execution continues synchronously in the same Task.
Non-Task-backed synchronous execution does not manufacture a Task.

I082-E adds bounded scalar callback-nesting state to `ProtosTask`; callback
lifetime objects are allocated only when a Closure argument is actually passed.

## Error and control round-trip

A Protos Error or other control outcome that leaves a callback is represented
across the provider call by an implementation-private carrier bound to the exact
originating operation.

Only that exact carrier leaving that same operation resumes the original Protos
outcome. A stale carrier, wrapped outcome, replacement failure, or otherwise
foreign-produced failure follows D188 and becomes a fresh `ForeignError`.

Ordinary Closure non-local-return and `InvalidReturn` behavior remain governed
by the existing callable/control semantics.

## Authority and provider boundary

The provider sees only the narrow callback capability, not the Closure,
Activation, Task, Actor or Process runtime objects.

~~~text
PRODUCTION_PROVIDER_ADDED=NO
STD_INTEROP_PUBLIC_API_ADDED=NO
PLAT052_AUTHORITY_EXPANSION=NO
HOSTACCESS_BROADENED=NO
POLYGLOTACCESS_BROADENED=NO
~~~

The implementation continues to use the plain-Java test provider for conformance
proof.

## Validation provenance

The maintainer reported after publication:

> el git diff check esta limpio.
>
> Todos los tests han pasado en local

Recorded exactly as:

~~~text
GIT_DIFF_CHECK=PASS_MAINTAINER_REPORTED
LOCAL_TESTS=PASS_MAINTAINER_REPORTED
~~~

No test count, exact command list, duration, or CI result is inferred beyond
that report.

The exact published GitHub revision was independently re-read. Its product
version is `0.3.278-SNAPSHOT`, and the normative specification remains
`0.1.447`.

## Closure and next routing

I082-E is complete.

The next planned slice is I082-F, which implements the separate PLAT052
authority-profile enforcement gate and negative proof. It remains distinct from
I082-G, because G introduces the first production host-Java provider and
end-to-end `java:` baseline.

~~~text
I082_E_STATUS=COMPLETED
I082_STATUS=IN_PROGRESS

NEXT_SLICE=I082-F
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
NEXT_SLICE_GOAL=PLAT052_AUTHORITY_PROFILE_ENFORCEMENT_AND_NEGATIVE_PROOF

I082_G_RELEASE=NO
~~~

## AI-assistance disclosure

This closure evidence was materially prepared with AI assistance from ChatGPT
using the exact published product commit, ratified D189 authority, current
PLAT052/PLAT053 project records, and maintainer-reported validation. No
independent human review is claimed.
