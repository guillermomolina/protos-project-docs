# TEST006-D — PLAT045 extracted Native distribution admission routing

Status: **READY IMPLEMENTATION ROUTING**

Date: 2026-10-02

## Trigger

BUG013-F was published and human-reported validation passed on:

~~~text
PROTOS_REVISION=7c16cec611c3cf5e504d9271c6964a32656ba32f
PROTOS_VERSION=0.3.137-SNAPSHOT

FOCAL_PLAT045_JVM_POLICY_TEST=PASS
make -C build/native test=PASS
make test=PASS
~~~

That closes the direct Native executable gate under PLAT045, but a post-validation
release-path audit found a second maintained Native admission surface:

~~~text
dist/prepare_release.py
    -> phase native-complete-admission
    -> python3 dist/validate_native.py --archive <native-zip>
~~~

The exact current `dist/validate_native.py` still encodes the superseded
PLAT038 forced guest Tier-2 contract.

## Exact stale release-distribution gate

At product revision:

~~~text
7c16cec611c3cf5e504d9271c6964a32656ba32f
~~~

`dist/validate_native.py` defines `validate_forced_guest_jit(...)` and executes
the extracted Native payload with:

~~~text
-Dpolyglot.engine.AllowExperimentalOptions=true
-Dpolyglot.engine.BackgroundCompilation=false
-Dpolyglot.engine.CompileImmediately=true
-Dpolyglot.engine.TraceCompilation=true
-Dpolyglot.engine.CompilationFailureAction=Print
-e 1
~~~

It then requires all of:

~~~text
OPT_FAILED=0
FRAME_WITHOUT_BOXING_FAILURES=0
COMPILATION_FAILURES=0
HELPER_BYTECODE_ROOT_TIER2>=1
SEMANTIC_BYTECODE_ROOT_TIER2>=1
~~~

and reports the stale success markers:

~~~text
NATIVE_DIST_FORCED_GUEST_JIT_CHECK=PASS
NATIVE_DIST_HELPER_BYTECODE_ROOT_TIER2_CHECK=PASS
NATIVE_DIST_SEMANTIC_BYTECODE_ROOT_TIER2_CHECK=PASS
~~~

This conflicts with ratified PLAT045 Candidate B while `oracle/graal#14579`
remains unresolved.

## Why prior validation did not catch it

The reported PASS sequence exercised:

~~~text
ProtosPerf006C1OptimizingRuntimeClosureTest
make -C build/native test
make test
~~~

The maintained direct Native gate `build/native/test-native.sh` was correctly
updated by BUG013-F.

However, `make test` does not build/extract a Native public-prerelease ZIP and
run `dist/validate_native.py` against it. Therefore all reported tests can pass
while DIST009-B1 would still fail at its extracted-distribution
`native-complete-admission` phase.

This is not a contradiction in the human-reported results; it is an uncovered
release-validation surface.

## Ownership

~~~text
BUG013=#749 PRODUCT_RUNTIME_BLOCKER=CLOSED
PLAT045=#772 RATIFIED
TEST006=#755 RELEASE_ADMISSION_OWNER=REOPEN
DIST009=#743 BLOCKED_BY_TEST006_D
~~~

BUG013 remains closed because the product/runtime fallback implementation is
validated.

TEST006 is reopened because its scope owns maintained Native release admission,
and one release-distribution admission surface still asserts the superseded
contract.

## Required TEST006-D implementation

Repository:

~~~text
guillermomolina/protos
~~~

Update the extracted Native distribution admission so it validates the same
PLAT045 capability matrix as the direct Native gate:

~~~text
NATIVE_IMAGE_SUPPORT=YES
NATIVE_RUNTIME=INTERPRETER_ONLY_FALLBACK
NATIVE_GUEST_JIT=UNAVAILABLE_UPSTREAM_ORACLE_GRAAL_14579

FALLBACK_RUNTIME_MARKER_REQUIRED=YES
OPT_DONE=0
OPT_FAILED=0
FRAME_WITHOUT_BOXING_FAILURES=0
COMPILATION_FAILURES=0
~~~

Preserve all existing extracted-distribution coverage unrelated to forced Tier-2,
including launcher, external-CWD/source execution, Test Tool, REPL, LSP, DAP,
archive immutability and complete admission.

The extracted distribution validator must fail closed if:

~~~text
fallback runtime marker is absent
guest compilation unexpectedly succeeds
opt failed is observed
FrameWithoutBoxing materialization is observed
compilation/internal failure is observed
~~~

It must not claim helper/semantic Tier-2 success while fallback mode is selected.

## Scope boundary

TEST006-D must not:

- modify Protos language or Standard Library semantics;
- change the Native build architecture again;
- change JVM optimizing-runtime policy;
- patch/fork Graal;
- upgrade GraalVM/Truffle;
- publish or prepare a real release candidate;
- create a Git tag or GitHub Release;
- weaken DAP/Test Tool/LSP/REPL/extracted-distribution checks;
- restore guest JIT before the PLAT045 objective re-enable gate passes.

## Validation direction

First prove the distribution-validator policy structurally/focally, then exercise
the real development Native distribution archive.

At minimum the implementation must provide focused automated evidence that:

~~~text
OLD_FORCED_TIER2_RELEASE_GATE=ABSENT
PLAT045_DISTRIBUTION_FALLBACK_GATE=PRESENT
FALLBACK_MARKER_REQUIRED=YES
UNEXPECTED_GUEST_COMPILATION_FAILS_CLOSED=YES
OPT_FAILURES_FAIL_CLOSED=YES
FRAME_FAILURES_FAIL_CLOSED=YES
COMPILATION_FAILURES_FAIL_CLOSED=YES

DAP_GATE=PRESERVED
TEST_TOOL_GATE=PRESERVED
REPL_GATE=PRESERVED
LSP_GATE=PRESERVED
~~~

After TEST006-D is published and exact-revision validation passes:

~~~text
TEST006_READY_TO_CLOSE=YES
DIST009_CAN_RESUME=YES
NEXT_SLICE=DIST009-B1
~~~

## Current routing

~~~text
NEXT_SLICE=TEST006-D
TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

BUG013_STATUS=CLOSED
TEST006_STATUS=READY
DIST009_STATUS=BLOCKED
~~~
