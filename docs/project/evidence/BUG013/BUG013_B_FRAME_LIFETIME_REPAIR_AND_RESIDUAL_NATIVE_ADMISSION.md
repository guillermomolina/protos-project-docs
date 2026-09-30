# BUG013-B — Frame lifetime repair and residual Native admission

Status: **PUBLISHED REPAIR; BUG013 REMAINS OPEN**

This durable, non-normative record retains the BUG013-B implementation and
validation evidence for `guillermomolina/protos#749`.

It does not define Protos language or Standard Library semantics.

## Exact product identity

```text
WORK_ITEM=BUG013-B
TYPE=IMPLEMENTATION
GITHUB_ISSUE=guillermomolina/protos#749
BLOCKED_RELEASE=guillermomolina/protos#743
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=2ce1f231afacb111639f1b8294f6e4d9a1aaff8a
COMMIT=BUG013: materialize lexical frame before retention
VERSION=0.3.126-SNAPSHOT
GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1
```

## Published repair

BUG013-A established a Protos-owned Truffle `VirtualFrame` lifetime violation:
the raw frame escaped into `ProtosFrameLexicalBindingAuthority` before being
materialized.

The published BUG013-B repair changes the escape boundary to:

```text
VirtualFrame
  -> frame.materialize()
  -> MaterializedFrame
  -> ProtosFrameLexicalBindingAuthority retention
```

The authority now retains a final `MaterializedFrame`; deferred
`prepareForContextObservation()` is no longer the first materialization point.
The PERF013 captured-materialized-local seam reuses that already materialized
frame directly.

The I075-D invariant is preserved: the authority retains the stable declaring
`BytecodeRootNode` and resolves the current `BytecodeNode` for local access.
No Protos-visible semantics were changed.

## Focal and integrated validation

The owner executed the bounded focal Maven set after the substantive source/test
change and reported:

```text
Tests run: 42, Failures: 0, Errors: 0, Skipped: 0
FOCAL_VALIDATION=PASS
```

After finalization metadata, the owner reported:

```text
make test=PASS
```

Therefore the Java/integrated repository validation is green for the published
candidate.

## Native validation result

The Native build itself reached:

```text
BUILD SUCCESS
```

but the maintained Native regression gate did **not** pass.

Reported maintained gate evidence:

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
OPT_DONE=1
OPT_FAILED=0
FRAME_WITHOUT_BOXING_FAILURES=0
COMPILATION_FAILURES=0
HELPER_BYTECODE_ROOT_TIER2=0
SEMANTIC_BYTECODE_ROOT_TIER2=1

HELPER_BYTECODE_ROOT_TIER2=FAIL
make: *** [Makefile:31: test] Error 1
```

The available compilation trace contained a successful Tier-2 compilation for:

```text
ProtosSemanticBytecodeRootNodeGen
```

and no successful Tier-2 compilation for:

```text
ProtosBytecodeRootNodeGen
```

## What BUG013-B established

The original diagnosed lifetime failure is repaired at the source/API-contract
boundary and the Native run no longer reports its former diagnostic:

```text
FRAME_WITHOUT_BOXING_FAILURES=0
OPT_FAILED=0
COMPILATION_FAILURES=0
```

Therefore BUG013-B provides positive evidence that the known raw-`VirtualFrame`
escape repair removed the observed `FrameWithoutBoxing` failure mode.

However, BUG013 cannot close because the maintained release gate requires both
generated root families to reach Tier 2:

```text
HELPER_BYTECODE_ROOT_TIER2>=1
SEMANTIC_BYTECODE_ROOT_TIER2>=1
NATIVE_FORCED_GUEST_JIT=PASS
```

The published candidate satisfies only the semantic-root half of that admission.

## Maintained harness observation

At both the pre-BUG013 revision
`4b354722906055b969c67bf02a91a6d9ccd5764e` and the I075 publication
`898eb8b2bafe4be99a33032ab0cf6436ce6f3e72`, the maintained
`build/native/test-native.sh` forced-JIT workload already used:

```text
-e '1'
```

while still requiring successful Tier-2 evidence for both
`ProtosSemanticBytecodeRootNodeGen` and `ProtosBytecodeRootNodeGen`.

Accordingly, the residual failure must not be attributed to the BUG013-B source
repair merely because it was observed after that repair. The next step is to
establish whether the maintained workload is insufficient to admit the helper
root, whether current helper-root reachability changed elsewhere, or whether a
second Native runtime-compilation defect remains.

## Closure and release status

```text
BUG013_B_SOURCE_REPAIR=PUBLISHED
BUG013_B_FOCAL_VALIDATION=PASS
BUG013_B_INTEGRATED_VALIDATION=PASS

FRAME_WITHOUT_BOXING_REGRESSION=PASS_FOR_OBSERVED_FAILURE_MODE
HELPER_BYTECODE_ROOT_TIER2=FAIL
NATIVE_FORCED_GUEST_JIT=FAIL
BUG013_CLOSURE=NOT_AUTHORIZED
DIST009_UNBLOCKED=NO
```

Do not weaken or remove the helper-root Tier-2 requirement to manufacture a
release PASS.

## Next bounded slice

```text
NEXT_SLICE=BUG013-C
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_SLICE_TITLE=resolve residual helper-root Native Tier-2 admission
NEXT_REPOSITORY=guillermomolina/protos
ARCHITECTURE_DECISION_REQUIRED=NO
UPSTREAM_COORDINATION_REQUIRED=NOT_ESTABLISHED
```

BUG013-C should first determine, from current Protos HEAD and the existing
maintained Native harness, why `ProtosBytecodeRootNodeGen` does not produce
Tier-2 `opt done` evidence while the semantic root does and while all explicit
compilation-failure counters remain zero.

Until that bounded diagnosis is complete, BUG013 remains open and DIST009/#743
remains blocked.
