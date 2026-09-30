# BUG012 — Native Image DAP `stackTrace` divergence checkpoint

## Scope

This record retains the diagnostic checkpoint that promoted the Native-backed
DIST006-B2 debugger failure into independently tracked BUG012 / #742.

It is non-normative project evidence. It does not define Protos language,
debugger-protocol, Process, Task, or runtime-host semantics, and it does not yet
select an implementation fix or assert that the root cause is upstream Graal.

## Exact published identity

```text
BUG012_ISSUE=742
DIST006_B_ISSUE=734
RELEASE=v0.3.116
RELEASE_ID=399355583
CANDIDATE_SOURCE_REVISION=0336ae20216bf2eec17854bea0f6435e4e1e9b19
GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1
JDK_VERSION=25.0.4.1.1
JVMCI=25.4-b23

NATIVE_ASSET=protos-0.3.116-native-linux-x86_64.zip
NATIVE_SHA256=61fd90b39a43c574900b3c61d4fe2e65e100166ced66e495281b336f934883e8

JVM_ASSET=protos-0.3.116-posix-jvm.zip
JVM_SHA256=ac5848ce3e4f371e7424ca5628899cc53a4d0be23d09381ca48db68218cd2add
```

The Native and JVM artifacts belong to the same published candidate source
revision. The JVM comparison used GraalVM Community Edition 25.4.4.1.1 / JDK
25.0.4.1.1, matching the release runtime contract.

## Initial editor-level symptom

DIST006-B2 real Debug acceptance in `guillermomolina/protos-vscode-extension`
proved that VS Code could:

- find and activate the installed Protos extension;
- configure the exact locked Protos runtime;
- install a source breakpoint;
- start the Protos debug session; and
- observe a genuine DAP `stopped` event with `reason:"breakpoint"`.

The session then emitted `terminated` before the first requested stack frame was
returned. An event-synchronized acceptance driver still failed with no usable
stack frame, scopes, locals, step, or continue operation.

This was sufficient to stop changing the editor driver and isolate the runtime
path directly.

## Direct DAP reproduction without VS Code

A direct TCP DAP probe was run against the published Native artifact. It used a
physical source file containing:

```protos
x: 41
print(x)
print("done")
```

The probe completed the following sequence:

```text
initialize
launch
setBreakpoints(line=1)
setExceptionBreakpoints([])
configurationDone
breakpoint changed -> verified=true
stopped -> reason=breakpoint, threadId=1
stackTrace(threadId=1)
```

Native result:

```text
NATIVE_BREAKPOINT=PASS
NATIVE_STACKTRACE=FAIL_TERMINATED_BEFORE_RESPONSE
NATIVE_PROCESS_EXIT=1
Runtime error: canonical initial module execution failed
protos debug: Polyglot runtime host cannot close while Process Contexts are active
[Graal DAP] Error: null
```

The decisive protocol ordering was:

```text
adapter -> client: stopped(reason=breakpoint)
client  -> adapter: stackTrace(threadId=1)
adapter -> client: terminated
```

No `stackTrace` response was returned before termination.

## Exact JVM control

The same direct probe was then run through the published portable JVM artifact
using exact GraalVM Community Edition 25.4.4.1.1.

Result:

```text
JVM_BREAKPOINT=PASS
JVM_STACKTRACE_FRAMES=1
JVM_STACKTRACE=PASS
JVM_TOP_FRAME={"line":1,"column":1,"id":1,"source":{"path":"/tmp/protos-dap-ab.protos","name":"protos-dap-ab.protos"}}
```

The diagnostic harness deliberately terminated the JVM process after the
successful stack response. That later process termination is not a debugger
failure.

This A/B establishes the bounded divergence:

```text
SAME_PROTOS_RELEASE_CANDIDATE=YES
SAME_GRAAL_TRUFFLE_LINE=25.4.4.1.1
DIRECT_DAP_WITHOUT_VSCODE=YES
NATIVE_BREAKPOINT=PASS
NATIVE_STACKTRACE=FAIL
JVM_BREAKPOINT=PASS
JVM_STACKTRACE=PASS
```

## Additional diagnostics

The Native probe was repeated with:

```text
-XX:MissingRegistrationReportingMode=Exit
-Dpolyglot.log.dap.level=FINEST
```

Observed facts:

- no `MissingReflectionRegistrationError`, `MissingJNIRegistrationError`,
  `MissingResourceRegistrationError`, or other `Missing*RegistrationError` was
  reported;
- the DAP trace logged receipt of the `stackTrace` request and immediately sent
  `terminated`;
- Graal DAP subsequently reported only `[Graal DAP] Error: null`, so the primary
  exception was not exposed by that logging path; and
- the later `Polyglot runtime host cannot close while Process Contexts are
  active` diagnostic is treated as secondary evidence, not as the established
  root cause. BUG011 already demonstrated that the same close invariant can
  surface after an earlier Native-only failure and must not be weakened merely
  to hide the primary defect.

## Validation gap exposed

The retained Native distribution admission proved DAP startup/TCP behavior, but
it did not require a usable suspended-session path through:

```text
breakpoint
  -> stopped
  -> stackTrace
  -> scopes/variables
  -> next/continue
```

The published Native artifact therefore passed a shallower debugger gate than
DIST006-B2 real editor-debug acceptance requires.

This does not invalidate the earlier evidence for the surfaces those gates
actually exercised.

## Classification and routing

The evidence is sufficient to classify the observed behavior as a confirmed,
independently trackable defect in maintained functionality. It is tracked as:

```text
BUG012 / #742
TITLE=Native Image DAP terminates on stackTrace after breakpoint
STATUS=IN_PROGRESS
DISCOVERED_BY=DIST006-B2
```

The failure reproduces without VS Code, so
`guillermomolina/protos-vscode-extension` is not the owning repair surface.
DIST006-B / #734 is blocked for its Native-backed real Debug acceptance while
BUG012 remains unresolved.

At this checkpoint the available GitHub connector actions did not expose native
sub-issue or issue-dependency mutation. The required live coordination relations
therefore remain explicit pending postconditions rather than being represented
by misleading textual substitutes:

```text
REQUIRED_NATIVE_PARENT=#742 -> #734
REQUIRED_NATIVE_DEPENDENCY=#734 blocked by #742
NATIVE_RELATION_RECONCILIATION=PENDING_TOOL_CAPABILITY
```

## What is not established

This checkpoint does **not** establish any of the following:

- that the defect is caused by missing Native Image reachability metadata;
- that the defect is necessarily in Oracle/Graal rather than Protos Native Image
  configuration or integration;
- that `ProtosPolyglotRuntimeHost.close()` is incorrect;
- that Protos language, debugger public protocol, Process, Task, or Actor
  semantics should change; or
- that DIST006-B1's JVM/core debugger evidence was invalid.

## Next investigation boundary

The next BUG012 slice is investigation only. It must use current repository HEAD
and public upstream/source evidence to identify the primary Native-only failure
triggered while Graal DAP services `stackTrace`.

The investigation should determine one of:

```text
OWNER=PROTOS_NATIVE_INTEGRATION
  CAUSE=<evidence-backed mechanism>
  NEXT_SLICE=<smallest implementation repair>

OWNER=UPSTREAM_GRAAL
  CAUSE=<evidence-backed upstream mechanism>
  MINIMAL_REPRODUCER=<bounded reproducer>
  NEXT_SLICE=<UPSTREAM coordination owner>

OWNER=INCONCLUSIVE
  NEXT_DISCRIMINATOR=<specific evidence needed>
```

No implementation should be started until that ownership boundary is established.
