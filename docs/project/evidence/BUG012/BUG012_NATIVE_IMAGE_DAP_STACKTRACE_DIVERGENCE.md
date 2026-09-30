# BUG012 — Native Image DAP `stackTrace` divergence checkpoint

## Scope

This record retains the diagnostic checkpoint that promoted the Native-backed
DIST006-B2 debugger failure into independently tracked BUG012 / #742.

It is non-normative project evidence. It does not define Protos language,
debugger-protocol, Process, Task, or runtime-host semantics. It retains the
diagnostic progression, final ownership decision, published repair, and
validation identity for BUG012.

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

Post-publication live-state verification shows that GitHub reconciled the native
parent relation successfully:

```text
NATIVE_PARENT=#742 -> #734
NATIVE_PARENT_RECONCILIATION=PASS
```

The available GitHub connector actions still do not expose issue-dependency
mutation. The remaining required live dependency is therefore an explicit
pending postcondition rather than a misleading textual substitute:

```text
REQUIRED_NATIVE_DEPENDENCY=#734 blocked by #742
NATIVE_DEPENDENCY_RECONCILIATION=PENDING_TOOL_CAPABILITY
```

## Static causal narrowing after the initial A/B

Subsequent source-level investigation narrowed the failing boundary without
changing either product artifact.

Graal 25.4 DAP processes `stackTrace` by enqueueing work into the suspended
thread. `ThreadsHandler.SuspendedThreadInfo.runExecutables()` invokes the queued
function directly and catches only `InterruptedException`; an unchecked
exception from the stack-trace task therefore escapes on the guest execution
thread. `DebugProtocolServerImpl.stackTrace(...)` first calls
`StackFramesHandler.getStackTrace(info)`, and that call must complete before the
DAP response future is completed.

This matches the observed Protos failure shape. The unchecked failure escapes
the DAP callback into guest execution, and
`ProtosCanonicalInitialModuleExecution.execute(...)` catches a
`RuntimeException` and preserves it as the cause of:

```text
IOException: canonical initial module execution failed
```

The direct-file CLI path currently prints only that outer IOException message
for an internal runtime failure, so the exact primary `RuntimeException` remains
present as `IOException.getCause()` but is hidden from the published Native
probe output.

The first Native-specific machinery introduced by DAP `stackTrace` is Truffle
stack-frame reconstruction. `SuspendedEvent.getStackFrames()` lazily walks
caller frames through `Truffle.getRuntime().iterateFrames(...)`; under Native
Image this resolves to SVM's `SubstrateStackIntrospection`. The walk reconstructs
Truffle call targets/call nodes from inspected-frame locals and may therefore
read physical-frame deoptimization metadata even before DAP requests scopes or
variables.

No Protos-specific root-instance or instrumentable-call-node hook was found on
this path. Protos uses the standard Bytecode DSL roots/call targets, and its
Native build adds no option that disables SVM frame information. The existing
Protos `NodeLibrary` bridge owns debugger scope projection, but DAP
`stackTrace` fails before `scopes`/variables are requested, so that projection
is not part of the current failing boundary.

The upstream SVM implementation was materially changed immediately before the
25.4 release:

```text
043cb6751a2e3f6416662f479697eda38599501f  Add SVM inspected frame support
                                                    2026-06-22
d8803e2460f73c291babb950eecb967888db756d  Fix materialization of already deoptimized frames
                                                    2026-07-15
818330b321cabe96d8f9845cc67847306d762906  [GR-74398] Add SVM primitive frame accessors and stack trace coverage
                                                    2026-08-21
```

The first of those commits added a SVM test, but that test exercises a synthetic
`ValueInfo[]` helper rather than a Native Truffle stack walk or DAP session. The
GR-74398 commit substantially changes `SubstrateInspectedFrame` but does not add
an end-to-end Native DAP `stackTrace` test. Oracle/Graal does have DAP tests that
exercise `stackTrace` (for example `SimpleLanguageDAPTest` and
`ITLDAPTest`), but repository search did not identify a corresponding
SubstrateVM/Native Image DAP test.

GraalVM documentation has historically exposed Native Image DAP inclusion
through `--tool:dap`, so a Truffle-language DAP session inside a Native Image is
not being treated here as a JVM-only unsupported usage. The current evidence
therefore makes the SVM inspected-frame path the strongest causal candidate,
but it still does not prove upstream ownership: a Protos-specific Native build
interaction could in principle trigger a valid SVM failure condition.

The next discriminator is now precise: expose the preserved
`IOException.getCause()` for one diagnostic Native `protos debug` run and record
its exact type/message/stack before selecting the repair owner.

## Primary cause identified

The diagnostic Native rebuild exposed the previously preserved
`IOException.getCause()` and established the primary failure:

```text
com.oracle.truffle.api.debug.DebugException
Caused by: java.lang.ExceptionInInitializerError
Caused by: java.lang.IllegalStateException:
    Receiver class com.guillermomolina.protos.execution.ProtosBytecodeTagTreeNodeExports
    is already registered.
    at com.oracle.truffle.api.library.LibraryFactory$ResolvedDispatch.register(...)
    at com.oracle.truffle.api.library.LibraryExport.register(...)
    at com.guillermomolina.protos.execution.ProtosBytecodeTagTreeNodeExportsGen.<clinit>(...)
```

The failure occurs while Graal's debugger constructs the top
`DebugStackFrame` and dispatches `NodeLibrary.hasRootInstance`. It therefore
precedes DAP caller-frame stack walking, `scopes`, variables, and Protos
debugger-scope member projection. The prior SVM stack-introspection hypothesis is
falsified as the primary cause for this BUG012 reproducer.

Truffle's 25.4 annotation processor generates the outer
`ProtosBytecodeTagTreeNodeExportsGen` class with a static initializer that
executes `LibraryExport.register(...)`. Native Image's
`TruffleBaseFeature` independently resolves explicit-receiver
`@ExportLibrary` owners during analysis through
`LibraryFactory$ResolvedDispatch.lookup(receiverClass)`, populating the
Truffle library receiver registration in the image.

Protos' Native initialization generator currently forces:

- all generated nested `*Gen$*LibraryExports*.class` classes; and
- the source owner `ProtosBytecodeTagTreeNodeExports`

to build-time initialization, but it does not include the generated outer
`ProtosBytecodeTagTreeNodeExportsGen` class that owns the registration
`<clinit>`.

The resulting split lifecycle is the defect: the receiver registration already
exists in the Native image, while the generated outer registration class is
still runtime-initialized. The first debugger `NodeLibrary` uncached dispatch
then initializes `ProtosBytecodeTagTreeNodeExportsGen` at runtime and executes
`LibraryExport.register(...)` a second time, which fails closed with
`Receiver class ... is already registered`.

Classification:

```text
PRIMARY_CAUSE=PROTOS_NATIVE_TRUFFLE_LIBRARY_CLASS_INITIALIZATION_SPLIT
OWNER=PROTOS_NATIVE_INTEGRATION
UPSTREAM_GRAAL_PRIMARY_DEFECT=NO
VSCODE_PRIMARY_DEFECT=NO
SVM_STACK_INTROSPECTION_PRIMARY_DEFECT=NO
```

The repair boundary was kept narrow: build-time initialization of the special
external-receiver export now covers both the source export owner and the
generated outer registration owner coherently. This preserves DIST006-C1's
hosted-parsing visibility requirement and I069 guest runtime compilation while
preventing duplicate runtime registration.

## Published repair and validation

The repair was published in `guillermomolina/protos` as:

```text
PROTOS_REVISION=417e44a8c60eaba1db7859d78bbb1e61c5dace64
PROTOS_VERSION=0.3.121-SNAPSHOT
COMMIT=BUG012: repair Native DAP stack traces
GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1
JDK_VERSION=25.0.4.1.1
```

The Native initialization generator now includes both:

```text
com.guillermomolina.protos.execution.ProtosBytecodeTagTreeNodeExports
com.guillermomolina.protos.execution.ProtosBytecodeTagTreeNodeExportsGen
```

The first owner remains build-time initialized so
`ProtosBytecodeTagTreeNodeExports.hasScope%%D(...)` is visible during hosted
Bytecode parsing for runtime compilation. The generated outer owner is also
build-time initialized so its generated `LibraryExport.register(...)`
initializer cannot first run at debugger runtime and attempt a duplicate receiver
registration.

A failed intermediate candidate that replaced the source owner with only the
generated outer owner was rejected because Native Image then failed compilation
with:

```text
The following method is reachable during compilation, but was not seen during
Bytecode parsing:
com.guillermomolina.protos.execution.ProtosBytecodeTagTreeNodeExports.hasScope%%D(TagTreeNode, Frame)
```

Keeping both owners is therefore required by the retained evidence.

The final rebuilt Native Image passed the maintained Native regression gate:

```text
NATIVE_VERSION_STATUS=0
NATIVE_HELP_STATUS=0
NATIVE_GUEST_SMOKE_STATUS=0
NATIVE_TEST_TOOL_STATUS=0
NATIVE_TEST_TOOL_OUTPUT_OK=1
NATIVE_TEST_TOOL_CONTEXT_TEARDOWN_FAILURES=0

NATIVE_DAP_BREAKPOINT=PASS
NATIVE_DAP_STACKTRACE_FRAMES=1
NATIVE_DAP_STACKTRACE=PASS
NATIVE_DAP_CONTINUE=PASS
NATIVE_DAP_PROCESS_STATUS=0
NATIVE_DAP_REGRESSION=PASS
NATIVE_DAP_STACKTRACE_STATUS=0

NATIVE_FORCED_JIT_STATUS=0
OPT_DONE=2
OPT_FAILED=0
FRAME_WITHOUT_BOXING_FAILURES=0
COMPILATION_FAILURES=0
HELPER_BYTECODE_ROOT_TIER2=1
SEMANTIC_BYTECODE_ROOT_TIER2=1

NATIVE_DAP_STACKTRACE_REGRESSION=PASS
NATIVE_FORCED_GUEST_JIT=PASS
HELPER_BYTECODE_ROOT_TIER2=PASS
SEMANTIC_BYTECODE_ROOT_TIER2=PASS
FRAME_WITHOUT_BOXING_REGRESSION=PASS
NATIVE_REGRESSION_SUITE=PASS
```

The repository-integrated `make test` gate also passed after the repair and
after rebasing onto the then-current product baseline.

No fallback compilation was enabled. No shutdown invariant was weakened. No
Protos language, Standard Library, Process, Task, Actor, debugger protocol, or
performance semantics were changed.

## Final classification

```text
BUG012_STATUS=CLOSED
PRIMARY_CAUSE=PROTOS_NATIVE_TRUFFLE_LIBRARY_CLASS_INITIALIZATION_SPLIT
OWNER=PROTOS_NATIVE_INTEGRATION
UPSTREAM_GRAAL_PRIMARY_DEFECT=NO
VSCODE_PRIMARY_DEFECT=NO
SVM_STACK_INTROSPECTION_PRIMARY_DEFECT=NO
REPAIR_REVISION=417e44a8c60eaba1db7859d78bbb1e61c5dace64
NATIVE_REGRESSION=PASS
INTEGRATED_TESTS=PASS
SEMANTIC_SCOPE_EXPANSION=NO
PERFORMANCE_SCOPE_EXPANSION=NO
SHUTDOWN_INVARIANT_SUPPRESSION=NO
```

BUG012 itself no longer blocks DIST006-B / #734. The repaired product revision
is not yet present in a public Native release asset, so the artifact prerequisite
is routed to DIST009 / #743.

```text
NEXT_OWNER=DIST009/#743
NEXT_SLICE=DIST009-A
NEXT_SLICE_TYPE=INVESTIGATION
PURPOSE=publish a JVM_PLUS_NATIVE prerelease containing BUG012 before B2 retry
```

After DIST009 publishes and verifies an exact Native artifact containing repair
revision `417e44a8c60eaba1db7859d78bbb1e61c5dace64`, DIST006-B2 may resume real VS
Code Run/Debug acceptance in `guillermomolina/protos-vscode-extension`.
