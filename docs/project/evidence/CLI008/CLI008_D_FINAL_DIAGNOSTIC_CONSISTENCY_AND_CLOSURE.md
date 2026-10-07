# CLI008-D — final diagnostic consistency and CLI008 closure

## Purpose

This record preserves publication and closure evidence for the final CLI008
diagnostic consistency slice.

Parent work item:

`CLI008 / guillermomolina/protos#312`

Preceding durable owners:

- D063 / `guillermomolina/protos#314` — Candidate B + S3 diagnostic contract;
- CLI008-B / `guillermomolina/protos#415` — value inspection / pretty rendering;
- CLI008-C / `guillermomolina/protos#416` — uncaught guest stack diagnostics;
- PLAT049 / `guillermomolina/protos#790` — Candidate C stack-capture authority.

This is non-normative implementation/closure evidence.

## Published product revision

~~~text
PROTOS_REVISION=a5b00b7c1bb72bacd6ef36bc714a3c8e0972dac4
COMMIT=CLI008-D: reconcile diagnostic presentation surfaces
VERSION=0.3.195-SNAPSHOT
PUBLISHED=YES
MAINTAINER_REPORTED_ALL_LOCAL_TESTS=PASS
~~~

The maintainer reports that all local tests passed after publication. No exact
test command or hosted-CI result is inferred beyond that report.

The product commit changes:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/cli/ProtosCli.java
src/test/java/com/guillermomolina/protos/cli/ProtosCli008DDiagnosticReconciliationTest.java
src/test/java/com/guillermomolina/protos/cli/ProtosCliPolyglotRoutingArchitectureTest.java
~~~

## Residual closed by CLI008-D

CLI008-C1 had already established a truthful occurrence-carried
`ProtosDiagnosticTrace`, but one presenting path still discarded it.

The bundled Tool launcher previously performed:

~~~text
session.executeModuleSource(source)
    -> ProtosExecutionOutcome FAILED(Error, trace)
    -> executeStandaloneRootTask(...)
    -> new ProtosSignalException(outcome.error())
    -> Tool catch prints Error only
~~~

The fresh exception preserved Error identity but had no terminal occurrence
trace. That made bundled Tool error presentation inconsistent with the already
reconciled REPL, `-e`, direct-file and workspace paths.

CLI008-D removes that presentation-only loss.

## Final presentation routing

The bundled Tool launcher now routes the exact terminal
`ProtosExecutionOutcome` through a bounded outcome translator before any
trace-losing exception reconstruction.

For a FAILED Tool outcome:

~~~text
exact Error
+
optional existing ProtosDiagnosticTrace
    ->
existing "<Tool> tool error: " summary prefix
    ->
ProtosGuestStackRenderer
    ->
exit 1
~~~

COMPLETED and CANCELLED retain their previous translation.

In particular, Test Tool COMPLETED results still flow through
`testToolExitCodeForRuntime`; CLI008-D does not reinterpret TestRunOutcome or
test-case semantics.

## Shared summary-plus-trace presentation

The CLI now centralizes the mechanical combination:

~~~text
surface-specific summary prefix
+
diagnosticInspector.render(exact Error)
+
optional existing ProtosDiagnosticTrace
~~~

The same mechanism is consumed by:

~~~text
REPL
-e
direct physical file
workspace/package run
bundled Tools
~~~

Fallback `ProtosSignalException` catches use the already-attached
`terminalDiagnosticTrace()` when present.

The CLI does not call `ProtosDiagnosticTraceCapture` and does not recapture a
stack.

An absent trace remains a valid state and prints only the historical Error
summary.

## Surface policies preserved

CLI008-D deliberately preserves existing presentation and exit boundaries.

~~~text
ERROR_SUMMARY_PREFIXES=PRESERVED
BUNDLED_TOOL_PREFIXES=PRESERVED
PROGRAM_STDOUT=UNCHANGED
EXIT_CODE_POLICIES=PRESERVED

TEST_TOOL_COMPLETED_CLASSIFICATION=PRESERVED
TEST_TOOL_SEMANTICS_CHANGED=NO
PACKAGE_TOOL_SEMANTICS_CHANGED=NO
PRINT_SEMANTICS_CHANGED=NO
SERIALIZATION_CHANGED=NO
~~~

Host/runtime and syntax errors remain distinct from guest Error diagnostics.

No guest stack is fabricated for ParseError, IOException, host RuntimeException,
Tool provisioning failure or cancellation.

## D063 and PLAT049 preservation

CLI008-D consumes already-produced diagnostic metadata only.

It does not change:

~~~text
ERROR_IDENTITY
ERROR_GUEST_VISIBLE_STATE
TRACE_BOUND=64
FAILURE_TIME_CAPTURE_AUTHORITY
PLAT044_INLINE_FRAME_POLICY
CONTINUATION_NORMALIZATION
FAILED_FUTURE_CONSUMER_LOCAL_POLICY
ACTOR_PROCESS_CAUSAL_STACK_POLICY
SUCCESS_PATH_CALLER_PROVENANCE
~~~

Therefore:

~~~text
D063_DELTA=NONE
PLAT049_DELTA=NONE
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

## Regression evidence encoded in the product revision

`ProtosCli008DDiagnosticReconciliationTest` freezes:

- workspace FAILED outcome renders its trace exactly once;
- Package and Test bundled Tool FAILED outcomes retain Tool-specific prefixes,
  render the existing trace exactly once and return exit 1;
- FAILED Tool outcome without a trace prints summary only;
- ordinary bundled Tool COMPLETED translation remains intact;
- Test Tool COMPLETED still uses `testToolExitCodeForRuntime`;
- CANCELLED translation remains the historical standalone-root cancellation
  failure consumed by the Tool runtime-error path;
- a fallback `ProtosSignalException` with attached terminal trace renders it;
- a fallback signal without trace prints only the summary.

`ProtosCliPolyglotRoutingArchitectureTest` is updated to freeze that production
bundled Tool routing uses the new outcome translator rather than the
trace-losing `executeStandaloneRootTask(session.executeModuleSource(source))`
composition.

The previously published CLI008-C1 tests remain the regression authority for
`-e`, direct-file and REPL guest-stack presentation.

## CLI008 complete history

The final work decomposition is:

~~~text
CLI008-A = presentation-contract audit / D063 routing
CLI008-B = value inspection and pretty rendering
CLI008-C = uncaught guest stack diagnostics
PLAT049 = failure-time guest stack capture authority
CLI008-D = final public-surface diagnostic consistency reconciliation
~~~

All planned implementation slices are now complete.

The explicitly deferred D063 possibilities remain deferred by design and are not
unfinished CLI008 work:

~~~text
exact :inspect command syntax
user-customizable guest inspect protocol
source snippets/carets
colors
persistent Error payload/message conventions
async-origin retention mechanism
~~~

They require separate future justification/authority if ever pursued.

## Closure

~~~text
CLI008_B_STATUS=CLOSED
CLI008_C_STATUS=CLOSED
PLAT049_STATUS=RATIFIED_CLOSED
CLI008_D_STATUS=COMPLETE

REPL_ERROR_TRACE=CONSISTENT
EVAL_ERROR_TRACE=CONSISTENT
DIRECT_FILE_ERROR_TRACE=CONSISTENT
WORKSPACE_ERROR_TRACE=CONSISTENT
BUNDLED_TOOL_ERROR_TRACE=CONSISTENT

AVAILABLE_TERMINAL_TRACE_SILENTLY_DROPPED=NO
TRACE_DUPLICATED=NO
TRACE_RECAPTURED_IN_CLI=NO

MAINTAINER_REPORTED_ALL_LOCAL_TESTS=PASS
CLI008_STATUS=CLOSED
NEXT_CLI008_SLICE=NONE
~~~

No CLI008-E is justified or allocated.
