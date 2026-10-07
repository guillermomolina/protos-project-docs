# CLI008-C1 — occurrence-carried guest stack diagnostics closure

## Purpose

This record preserves publication and closure evidence for
`CLI008-C / guillermomolina/protos#416` after implementation of the bounded
PLAT049 Candidate C terminal guest diagnostic trace.

It is non-normative project evidence. The governing authorities remain:

- D063 / `guillermomolina/protos#314` — Candidate B + S3;
- PLAT049 / `guillermomolina/protos#790` — ratified Candidate C;
- the current Protos product repository.

## Published product revision

~~~text
PROTOS_REVISION=3a195f71c0834093e01f0ff8e77497ddb53dcef6
COMMIT=CLI008-C1: occurrence-carried bounded guest diagnostic trace (PLAT049 C)
VERSION=0.3.192-SNAPSHOT
PUBLISHED=YES
MAINTAINER_REPORTED_ALL_LOCAL_TESTS=PASS
~~~

The maintainer reports that all local tests passed after publication. This record
does not infer a particular command transcript or hosted-CI result that was not
provided.

The product commit changes 17 paths, including:

~~~text
src/main/java/com/guillermomolina/protos/runtime/ProtosDiagnosticTrace.java
src/main/java/com/guillermomolina/protos/runtime/ProtosSignalException.java
src/main/java/com/guillermomolina/protos/runtime/ProtosTask.java

src/main/java/com/guillermomolina/protos/execution/ProtosDiagnosticTraceCapture.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeControlTransferException.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeTaskExecution.java
src/main/java/com/guillermomolina/protos/execution/ProtosExecutionOutcome.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java

src/main/java/com/guillermomolina/protos/cli/ProtosGuestStackRenderer.java
src/main/java/com/guillermomolina/protos/cli/ProtosCli.java

src/test/java/com/guillermomolina/protos/execution/ProtosCli008C1DiagnosticTraceTest.java
src/test/java/com/guillermomolina/protos/cli/ProtosCli008C1GuestStackCliTest.java
~~~

The same commit updates `pom.xml` and `CHANGELOG.md` to
`0.3.192-SNAPSHOT`.

## Materialized architecture

The implementation follows PLAT049 Candidate C rather than introducing a second
caller-stack authority.

~~~text
escaping Error occurrence
    -> failure-only Truffle stack / recorded failure position
    -> semantic root projection
    -> continuation-root normalization
    -> PLAT044 nested RootTag augmentation
    -> bounded inert ProtosDiagnosticTrace
    -> Task terminal occurrence
    -> ProtosExecutionOutcome
    -> CLI presentation
~~~

`ProtosDiagnosticTrace` is an immutable record whose frames retain only plain
diagnostic facts:

~~~text
optional label
source name
optional display path
line
column
~~~

Its retained-frame limit is exactly:

~~~text
MAX_FRAMES=64
ORDER=INNERMOST_FIRST
TRUNCATION=KEEP_INNERMOST_64_PLUS_BOOLEAN_FLAG
~~~

The DTO retains no live Truffle frame, root, CallTarget, Bytecode node/location,
Source, Context or Protos Activation graph.

## Error occurrence versus Error identity

C1 keeps the D063 distinction intact.

The Error object remains unchanged and exact. The terminal diagnostic trace is
metadata beside that Error on the failure occurrence.

The implementation and focal tests cover the same Error instance being signalled
from distinct sites with distinct traces, while verifying that the Error's
guest-visible slots are unchanged.

~~~text
ERROR_IDENTITY=PRESERVED
ERROR_GUEST_VISIBLE_MUTATION=NO
TRACE_OWNER=FAILURE_OCCURRENCE
~~~

## Failure-time capture

For `ProtosSignalException`, the runtime captures the trace before reducing the
terminal occurrence to only its Error value.

Direct Task failure from bridged control transfer can carry the bytecode origin
needed to project the genuine failing Task position without inventing a
`ProtosSignalException`.

The trace is committed with the Task's first failure and therefore survives
structured child drain without retaining the live exception/frame graph.

## PLAT044 inline callbacks

The implementation explicitly preserves the PLAT044 B-prime semantic/physical
split.

Eligible inline callbacks do not gain a fabricated physical frame. Instead the
failure-only projector reparses source/tag information as needed and augments a
physical semantic frame with active nested semantic `RootTag` regions.

Focal coverage verifies:

~~~text
PLAT044_INLINE_CALLBACK_SEMANTIC_FRAME=YES
PLAT044_INLINE_CALLBACK_DUPLICATED=NO
PHYSICAL_CALLBACK_FALLBACK_FRAME=YES
~~~

## Continuations and Future boundary

Focal coverage verifies semantic order through suspension/resume and composed
continuations without exposing generated continuation helper frames.

Failed Future observation preserves the existing authority:

~~~text
FUTURE_STORES_ERROR=YES
FUTURE_STORES_PRODUCER_TRACE=NO
FUTURE_VALUE_SIGNAL=CONSUMER_LOCAL_OCCURRENCE
PRODUCER_FRAMES_IN_CONSUMER_TRACE=NO
~~~

No cross-Actor or cross-Process causal stack concatenation is introduced.

## CLI presentation

C1 introduces `ProtosGuestStackRenderer`, separate from ordinary program output
and from value inspection.

The focal CLI test verifies D063 S3 presentation on:

~~~text
-e
direct file
REPL
~~~

It also verifies that ordinary `print("before")` output is unchanged and that
Java/Truffle/Bytecode/Continuation/Task implementation names do not leak into
the public trace.

The workspace FAILED-outcome translation now routes through the same
`reportUncaughtError(ProtosExecutionOutcome, ...)` path and can therefore render
the occurrence trace carried by a failed workspace execution.

## CLI008-C closure

The closure condition is satisfied at the published revision:

~~~text
TRUTHFUL_GUEST_ONLY_STACK=YES
ORDINARY_NESTED_CALLS=YES
PLAT044_INLINE_CALLBACK=YES
SAME_TASK_SUSPEND_RESUME=YES
FAILED_FUTURE_CONSUMER_LOCAL=YES
TRACE_BOUNDED=YES
HOST_IMPLEMENTATION_FRAMES_EXCLUDED=YES
SUCCESS_PATH_CALLER_PROVENANCE=NO
RETAINED_LIVE_FRAME_GRAPH=NO
UNRESOLVED_PLATFORM_DECISION=NO
MAINTAINER_REPORTED_ALL_LOCAL_TESTS=PASS

CLI008_C_STATUS=CLOSED
~~~

No new Dxxx/PLATxxx decision is required for CLI008-C.

## Routed residual: CLI008-D

CLI008's parent contract already reserved a final mechanical consistency slice:

~~~text
CLI008-D — final REPL/file/-e/workspace/tool diagnostic consistency reconciliation
~~~

That slice is now justified by current HEAD rather than speculation.

At C1 HEAD, ordinary `-e`, direct-file and workspace FAILED outcomes can render
the inert trace directly, and REPL renders a terminal
`ProtosSignalException.terminalDiagnosticTrace()`.

However the bundled-tool path still executes:

~~~text
executeStandaloneRootTask(session.executeModuleSource(source))
~~~

and `executeStandaloneRootTask` converts a FAILED outcome into a fresh
`new ProtosSignalException(outcome.error())`. The bundled-tool catch then prints
only:

~~~text
<DiagnosticName> tool error: <rendered Error>
~~~

The occurrence-carried trace has therefore been discarded before that
presentation boundary.

There are also fallback direct-file/eval `ProtosSignalException` catches that
print only the Error summary. CLI008-D should reconcile all genuine presenting
boundaries so an already-available terminal occurrence trace is not lost, while
preserving each surface's historical error prefix and exit-code policy.

This is mechanical consumption of D063 + PLAT049 + C1. It does not select new
stack semantics.

~~~text
NEXT_SLICE=CLI008-D
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
NEW_DESIGN_DECISION_REQUIRED=NO
~~~
