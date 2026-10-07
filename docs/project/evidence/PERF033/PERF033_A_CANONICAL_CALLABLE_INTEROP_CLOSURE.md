# PERF033-A — canonical pay-as-you-grow callable interop closure

Date: 2026-10-07

## Work identity

~~~text
WORK_ITEM=PERF033
SLICE=PERF033-A
ISSUE=guillermomolina/protos#832
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

STARTING_RECONCILED_HEAD=32f61e4781270e0e5aca29aa1809a53a17f5bb02
ENDING_PROTOS_REVISION=c03370abca4592b95d35ccba5c4a185b955bd4bb
ENDING_VERSION=0.3.277-SNAPSHOT

COMMIT_SUBJECT=PERF033-A: add canonical pay-as-you-grow callable interop
~~~

This record is durable non-normative implementation/performance evidence.

No normative specification file changed.

## Published product delta

The exact published commit changes:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosHostExecutableClosure.java
src/main/java/com/guillermomolina/protos/execution/ProtosLanguageContext.java
src/main/java/com/guillermomolina/protos/execution/ProtosPolyglotExecutionContext.java
src/main/java/com/guillermomolina/protos/execution/ProtosPolyglotProcessContext.java
src/main/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedSession.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf033CanonicalCallableInteropTest.java
~~~

## Canonical callable boundary implemented

PERF033-A adds one context-bound executable presentation of an exact source-backed
Protos Closure.

~~~text
PreparedTopLevel.executable()
  -> retained org.graalvm.polyglot.Value
  -> Value.execute()
  -> framework host-to-guest boundary
  -> InteropLibrary.execute(ProtosHostExecutableClosure)
  -> compact ordinary Closure preparation
  -> cached DirectCallNode
  -> guest Bytecode target
  -> result
~~~

`ProtosHostExecutableClosure` is interop machinery, not a Protos semantic value.
It keeps the exact `ProtosClosureValue`, caller provenance, owning
`ProtosLanguageContext`, and the Context-owned `RootCallTarget`.

The zero-argument direct specialization caches the target and a
`DirectCallNode`; a megamorphic fallback uses the existing indirect call path.

## Pay-as-you-grow properties

For the canonical `Value.execute()` path exercised by the literal workload:

~~~text
ROOT_TASK_CREATED=NO
PROTOS_TASK_CREATED=NO
ACTOR_LIVE_TASK_REGISTRATION=NO
TASK_DISPATCH=NO
TASK_TERMINAL_PUBLICATION=NO

RICH_CALLEE_ACTIVATION_MATERIALIZED=NO
COMPACT_CLOSURE_FRAME_ABI=YES

PROTOS_EXPLICIT_CONTEXT_ENTER_LEAVE_PER_VALUE_CALL=NO
PROTOS_ROOT_TASK_EXECUTION_ON_VALUE_PATH=NO
PROTOS_EXECUTION_OUTCOME_ON_VALUE_PATH=NO

CANONICAL_VALUE_SESSION_GATE=NO
CANONICAL_VALUE_PER_CALL_CONCURRENCY_MARKER=NO
~~~

The adapter validates that framework entry reached the exact owning
`ProtosLanguageContext`; it does not perform an additional Protos-owned
`Context.enter()/leave()`.

The prior per-entry entered-Context ThreadLocal dependency was replaced with the
Context-owned binding required by the host wrapper, so framework-owned entry can
reach standard-stream/source-admission/runtime facilities without restoring a
manual host envelope.

## Stronger legacy embedding contract retained separately

`PreparedTopLevel.invoke()` remains unchanged in this slice.

It still owns the stronger existing contract:

~~~text
SESSION_GATE=YES
MULTI_CALLER_SERIALIZATION=YES
CALL_VS_CLOSE_EXCLUSION=YES
RETURN_TYPE=ProtosExecutionOutcome
CURRENT_INNER_ROUTE=RootTask
~~~

That stronger path does not define the canonical `Value.execute()` floor.

No claim is made in PERF033-A that `PreparedTopLevel.invoke()` itself has become
pay-as-you-grow.

## Focused executable evidence in the published commit

`ProtosPerf033CanonicalCallableInteropTest` pins at least:

- the prepared canonical value is executable and retained once;
- repeated `Value.execute()` returns the literal result and leaves the
  RootActor domain with zero live Tasks;
- compact execution retains the exact Closure in frame argument zero and has no
  compact Task provenance;
- a synchronous stdout observation sees the interop adapter but not
  `ProtosRootTaskExecution`, `ProtosTask`,
  `ProtosStandaloneHostedSession`, or
  `ProtosPolyglotExecutionContext` on the execution stack;
- top-level slot reassignment does not change an already prepared callable;
- non-local-return/control behavior remains covered;
- unsupported arguments are rejected through the interop arity boundary;
- closing the session invalidates the retained Value;
- the adapter has no private `com.oracle.truffle.polyglot` dependency.

## Maintainer-reported validation

After publishing the product commit, the maintainer reported:

~~~text
GIT_DIFF_CHECK=PASS_MAINTAINER_REPORTED
LOCAL_TESTS=PASS_MAINTAINER_REPORTED
~~~

No unreported suite count, command, timing, profiler result or benchmark result
is inferred.

A separately observed reduction in broad Protos test wall time was treated only
as an encouraging signal, not admitted PERF033 performance evidence.

## Remaining PERF033 work

PERF033-A establishes the structural canonical path. PERF033 remains open.

The next gate is bounded post-A measurement of the exact canonical
`primitive-return-literal` surface. It must establish whether the old fixed
owners actually disappeared from the sampled hot path and quantify the new
steady-state call cost against the already-retained peer reference.

The measurement must not reopen the generic Truffle optimization funnel unless
the new profile reveals a concrete residual owner.

~~~text
PERF033_A_STATUS=COMPLETED
PERF033_STATUS=OPEN

NEXT_SLICE=PERF033-B
NEXT_SLICE_GOAL=POST_A_CANONICAL_PROFILE_AND_SAME_WORKLOAD_TIMING

OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

## AI-assistance disclosure

This closure evidence was materially prepared with AI assistance from ChatGPT
using the exact published product commit, current PERF033/PLAT046 architecture
records, the published focal test, and maintainer-reported local validation.
