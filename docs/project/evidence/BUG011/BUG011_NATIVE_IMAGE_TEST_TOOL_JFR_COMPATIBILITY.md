# BUG011 — Native Image Test Tool JFR compatibility closure

Status: PUBLISHED
Owning Issue: `guillermomolina/protos#738` (`BUG011`)
Related work: `guillermomolina/protos#733` (`DIST006`)

## Publication identity

```text
PROTOS_REVISION=754de7a2a2d73dd4b39109bb842522ed9cc8153a
PROTOS_PARENT_REVISION=d18822e968a1ee6986d832731054b9b90b332b1d
PROTOS_VERSION=0.3.109-SNAPSHOT
COMMIT_MESSAGE=BUG011: fix Native Image Test Tool context teardown
GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1
JDK_VERSION=25.0.4.1.1
```

BUG011 was discovered immediately after the DIST006-C1 Native Image closure when
the supported public Test Tool path was exercised through the Stage1 native
executable.

## Escaped regression

At product revision
`d18822e968a1ee6986d832731054b9b90b332b1d`, the authoritative URI selection
passed through the ordinary JVM launcher:

```text
bin/protos test --file protos/tests/library/uri/parse.protos
[uri] 0/4
[uri] 1/4
[uri] 2/4
[uri] 3/4
[uri] 4/4 passed
4 passed, 0 failed
EXIT=0
```

The same supported selection through the Native Image executable stopped after
the initial progress line and surfaced:

```text
IllegalStateException:
Polyglot runtime host cannot close while Process Contexts are active
EXIT=70
```

DIST006-C1's historical Native Image regression gate had covered version/help,
ordinary guest execution and forced guest JIT/Tier-2 behavior, but had not
executed the bundled Test Tool. That historical evidence remains unchanged and
continues to describe only the surfaces it actually validated.

## Causal diagnosis

Initial investigation treated the runtime-host close failure as a possible
Process/Context teardown ordering defect. Adding a passive termination wait
changed the failure into a hang. Diagnostic state at that point showed:

```text
BEFORE_REQUEST process=RUNNING actor=INITIALIZING liveTasks=3 runnable=1
AFTER_REQUEST  process=TERMINATING actor=TERMINATING liveTasks=3 runnable=2
```

That experiment established that the passive wait was not a valid repair and was
removed.

The primary failure became visible after temporarily preventing the later
runtime-host close exception from masking it:

```text
java.lang.InternalError: Flight Recorder is not supported on this VM
at jdk.jfr.EventType.getEventType(...)
at com.guillermomolina.protos.execution.ProtosTestToolPerf017Admission.<clinit>(...)
```

The Test Tool's PERF017 diagnostic instrumentation eagerly initialized its JFR
`EventType`. The Stage1 Native Image runtime does not provide Flight Recorder,
so the optional diagnostic facility raised an `InternalError` during Test Tool
execution. Session teardown then encountered still-active Process Contexts from
that interrupted execution and produced the originally visible
`RuntimeHost.close()` failure.

Therefore the runtime-host close guard was secondary evidence, not the causal
defect.

## Published repair

Product revision
`754de7a2a2d73dd4b39109bb842522ed9cc8153a` changes the PERF017 admission
instrumentation so JFR capability is tested before requesting the event type.

When Flight Recorder is unavailable, PERF017 diagnostic emission is disabled.
On ordinary JVMs where JFR is available, the existing event and correlation
behavior remains intact.

The repair deliberately does not weaken or suppress
`ProtosPolyglotRuntimeHost.close()`'s active-Process-Context invariant.

The Native Image regression gate now also executes the canonical supported Test
Tool selection:

```text
protos/tests/library/uri/parse.protos
```

and requires all of the following:

```text
NATIVE_TEST_TOOL_STATUS=0
NATIVE_TEST_TOOL_OUTPUT_OK=1
NATIVE_TEST_TOOL_CONTEXT_TEARDOWN_FAILURES=0
NATIVE_TEST_TOOL_SMOKE=PASS
```

## Validation evidence

The owner reported the focused JVM regression set PASS after the final repair:

```text
ProtosTestToolPerf017AdmissionTest=PASS
ProtosCliPolyglotRoutingArchitectureTest=PASS
ProtosCliTest=PASS
```

A rebuilt Stage1 Native Image then executed the authoritative focal selection to
completion:

```text
[uri] 0/4
[uri] 1/4
[uri] 2/4
[uri] 3/4
[uri] 4/4 passed
4 passed, 0 failed
NATIVE_TEST_TOOL_STATUS=0
```

The ordinary JVM control remained green:

```text
[uri] 4/4 passed
4 passed, 0 failed
JVM_TEST_TOOL_STATUS=0
```

The complete canonical Native Image gate reported:

```text
NATIVE_VERSION_STATUS=0
NATIVE_HELP_STATUS=0
NATIVE_GUEST_SMOKE_STATUS=0
NATIVE_TEST_TOOL_STATUS=0
NATIVE_TEST_TOOL_OUTPUT_OK=1
NATIVE_TEST_TOOL_CONTEXT_TEARDOWN_FAILURES=0
NATIVE_FORCED_JIT_STATUS=0
OPT_DONE=2
OPT_FAILED=0
FRAME_WITHOUT_BOXING_FAILURES=0
COMPILATION_FAILURES=0
HELPER_BYTECODE_ROOT_TIER2=1
SEMANTIC_BYTECODE_ROOT_TIER2=1
NATIVE_VERSION_SMOKE=PASS
NATIVE_HELP_SMOKE=PASS
NATIVE_GUEST_SMOKE=PASS
NATIVE_TEST_TOOL_SMOKE=PASS
NATIVE_FORCED_GUEST_JIT=PASS
HELPER_BYTECODE_ROOT_TIER2=PASS
SEMANTIC_BYTECODE_ROOT_TIER2=PASS
FRAME_WITHOUT_BOXING_REGRESSION=PASS
NATIVE_REGRESSION_SUITE=PASS
```

After the implementation version advanced to `0.3.109-SNAPSHOT` and the
changelog was updated, the owner executed the required integrated repository
gate once on the final candidate:

```text
make test=PASS
FULL_REQUIRED_VALIDATION=PASS
```

## Acceptance closure

```text
BUG011_STATUS=COMPLETED
PROTOS_REVISION=754de7a2a2d73dd4b39109bb842522ed9cc8153a
JVM_URI_FOCAL=PASS
NATIVE_URI_FOCAL=PASS
NATIVE_URI_CASES=4_OF_4
NATIVE_TEST_TOOL_EXIT=0
ACTIVE_CONTEXT_CLOSE_GUARD_FAILURES=0
RUNTIME_HOST_CLOSE_INVARIANT_WEAKENED=NO
NATIVE_FORCED_GUEST_JIT=PASS
HELPER_BYTECODE_ROOT_TIER2=PASS
SEMANTIC_BYTECODE_ROOT_TIER2=PASS
NATIVE_TEST_TOOL_REGRESSION_GATE=PASS
FOCUSED_REGRESSION_VALIDATION=PASS
FULL_REQUIRED_VALIDATION=PASS
PROTOS_LANGUAGE_SEMANTIC_CHANGE=NO
PERFORMANCE_OPTIMIZATION_MIXED_IN=NO
HISTORICAL_DIST006_C1_EVIDENCE_REWRITTEN=NO
```

BUG011 therefore closes the independently routed Native Image Test Tool gap
without changing Protos language semantics, Test Tool result semantics,
Process/Task semantics, guest runtime-compilation guarantees, or the strict
runtime-host lifecycle invariant.
