# BUG018-B — retained-frame safe-read mechanism selection

Date: 2026-10-05

## Work identity

~~~text
WORK_ITEM=BUG018
INVESTIGATION_SLICE=BUG018-B
PROTOS_ISSUE=guillermomolina/protos#801
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative investigation evidence. It does not replace
the live GitHub Issue state or the normative Protos specification.

## Exact inspected product state

~~~text
PROTOS_HEAD=3c9738f5835cc8ed2e43d50fb8edcc7eb956ddc9
PROTOS_HEAD_SUBJECT=TEST009-K: cut remaining compiler expansion debt
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
BUG018_STATE=OPEN
BUG018_STATUS=READY
BUG018_ROOT_CAUSE_VERDICT=PROVEN
BUG018_B_MECHANISM_VERDICT=PROVEN
SEMANTIC_CHANGE_REQUIRED=NO
PLATFORM_DECISION_REQUIRED=NO
~~~

The product HEAD is one commit beyond the BUG018-A inspection point. That delta
does not contain a BUG018 repair and the unsafe retained-frame reads remain
present. No BUG018 implementation commit is published at evidence-capture time.

## Selected repair boundary

BUG018 has two distinct retained-frame read surfaces and the smallest correct
repair is deliberately dual:

~~~text
CAPTURED_PROVEN_READ=
  select the semantic owner frame and PRESENT state in Protos
  -> perform the value load with built-in LoadLocalMaterialized(ownerLocal)

RUNTIME_AUTHORITY_READ=
  retain stable BytecodeLocation + public BytecodeLocal.getLocalOffset metadata
  -> BytecodeLocation.update()
  -> current BytecodeNode + translated BCI
  -> BytecodeNode.getLocalValue(currentBci, retainedFrame, localOffset)
     inside the existing @TruffleBoundary authority seam

WRITE_PATH=UNCHANGED
~~~

This does not alter lexical lookup, capture semantics, ownership, late
membership, removal fallback, P/Future semantics, boxing elimination, or the
cached/uncached architecture.

## Why MaterializedLocalAccessor.getObject is unsafe here

BUG018-A proved that a retained frame can be physically PRESENT while the
current cached BytecodeNode still carries ILLEGAL local-kind metadata learned
from another activation.

In Truffle Bytecode DSL 25.4.4.1.1,
`MaterializedLocalAccessor.getObject` resolves the declaring root's current
BytecodeNode and then calls `getLocalValueInternal(frame, localOffset,
localIndex)`. The cached implementation selects the physical read from cached
local-kind metadata. Cached ILLEGAL therefore raises `FrameSlotTypeException`
even after `isCleared` on the same retained frame established PRESENT.

Putting only a `@TruffleBoundary` around that accessor call does not change this
mechanism: `getLocalValueInternal` itself still trusts cached metadata.

## Why built-in LoadLocalMaterialized is safe for the proven captured fast path

The exact 25.4.4.1.1 generator was inspected.

A materialized load encodes the builder-time local identity through generated
immediates including the declaring-root identity and local/frame identity, while
the `MaterializedFrame` is the dynamic operand. This matches Protos's existing
lowering topology: `capturedOwnerBytecodeLocal` already returns the exact owner
`BytecodeLocal` only when owner and child share the same `BytecodeRootNodes`
group.

The generated load path has its own quickening and boxing-elimination handling.
When cached metadata and the physical retained frame disagree, the generated
load reads/tolerates the physical value and can generalize/quick-en the
instruction rather than exposing the public-accessor failure mode.

The repair must keep semantic selection separate from value loading:

~~~text
ownerFrame = selectCapturedOwnerFrameAndPresence(...)

if ownerFrame exists:
    return built-in LoadLocalMaterialized(ownerLocal, ownerFrame)

return ordinary captured lexical fallback
~~~

The loaded value itself must not be used as the "no owner" sentinel. Presence is
decided first so PRESENT(null) remains distinct from ABSENT.

## Liveness and scoping proof

`GenerateBytecode.enableBlockScoping` is true by default in the effective
Truffle version.

That does not make the lexical authority locals block-local. In
`CanonicalToBytecodeLowerer.emitRootBody`, Protos opens the root and then
creates every statically declared lexical `BytecodeLocal` directly under that
root before body blocks are emitted:

~~~text
beginRoot()
createLocal(...) for every declared lexical name
install/select authority
emit body
endRoot()
~~~

Those authority locals are therefore ROOT-scoped. They are not the temporary
block locals that Truffle may reuse across nested block lifetimes.

No change to Protos's scoping model is required.

## Why BytecodeNode.getLocalValue is safe for retained authority reads

The exact generated cached `BytecodeNode.getLocalValue` implementation was
inspected.

In interpreter/Java execution it chooses the local kind from the physical frame
tag. In compiled execution it may use cached local-kind metadata. The
`ProtosFrameLexicalBindingAuthority` read seams that need this repair are
already intentionally isolated behind `@TruffleBoundary`; the safe replacement
therefore executes the physical-frame-tag branch rather than partial-evaluating
a stale cached tag.

The API requires a BCI and a logical local offset. The public
`BytecodeLocal.getLocalOffset()` is exactly the offset intended for
`BytecodeNode.getLocalValue(...)`.

A raw stored BCI is not sufficient. Truffle explicitly states that a BCI is
only valid for the `BytecodeNode` that produced it because the bytecode node
can be replaced across tier/configuration changes. The correct retained
location metadata is `BytecodeLocation`.

`BytecodeLocation.update()` moves the location to the root's newest
`BytecodeNode` and translates the BCI. The authority can therefore keep the
retained frame by reference while always pairing it with the current node and a
BCI translated for that node.

## Rejected alternatives

### Retained LocalRangeAccessor.getObject

Rejected for retained-frame reads. It also reaches
`getLocalValueInternal` and therefore has the same cached-tag sensitivity.
Its ordinary contract is for the current frame, not an arbitrary escaped
retained frame.

### BytecodeFrame

Rejected.

A non-copied `BytecodeFrame` can observe updates, but Truffle documents that
with block scoping it is only valid until interpreter execution continues. A
copied `BytecodeFrame` remains valid but stops observing future writes and
would break capture-by-reference semantics.

### Raw physical frame indices / internal offsets

Rejected. No internal `USER_LOCALS_START_INDEX`, reflection, generated
implementation detail, ordinal-as-offset assumption, or raw frame-slot
arithmetic is required.

### Catching FrameSlotTypeException

Rejected. The bug is an invalid read mechanism, not missing-binding semantics.

### Disabling boxing elimination / cached or uncached tiers / P / capture fast paths

Rejected. The safe public mechanisms are available without removing those
architectural features.

## Write-side result

~~~text
WRITES_AFFECTED_BY_SAME_BUG=NO
WRITE_FIX_REQUIRED=NO
~~~

The generated cached set path reconciles the cached local-kind metadata when a
write is incompatible with the current cached tag: it invalidates, updates the
cached tag, and repeats the physical write.

BUG018 is therefore a retained read-side defect. Existing
`setObject`/`clear` paths do not need to be replaced for this slice.

## Upstream classification

The failure is also a valid future upstream report for
`MaterializedLocalAccessor.getObject`.

Its public contract accepts a current BytecodeNode and a materialized frame
containing the local. Protos satisfies those conditions, yet
`isCleared(currentNode, retainedFrame)` can report PRESENT while
`getObject(currentNode, retainedFrame)` throws because the cached node's
local-kind metadata came from another activation.

The Protos repair need not wait for upstream action.

~~~text
UPSTREAM_TRUFFLE_BUG=YES
UPSTREAM_ACTION=FUTURE_SEPARATE_REPORT
UPSTREAM_BLOCKS_PROTOS_FIX=NO
~~~

## Deterministic regressions required by BUG018-C

Two regressions are required because the two read surfaces are independent.

### Captured-materialized regression

~~~text
CLASS=ProtosBug018CapturedMaterializedReadAfterOwnerTierTransitionTest
TEST=escapedCapturedReadOfPresentLocalSurvivesOwnerTransitionInAnotherActivation
~~~

Shape:

1. A1 enters the owner root while UNCACHED.
2. A1 establishes local `x` and returns an escaped closure capturing it.
3. The A1 owner frame remains retained.
4. The harness forces the next owner-root entry to transition to CACHED.
5. A2 enters the same owner root and triggers that transition.
6. A2 calls the escaped A1 closure before A2 writes its own `x`.
7. The closure must return A1's exact value.

Pre-fix expected failure: `FrameSlotTypeException`.
Post-fix result: exact A1 captured value.

### Runtime-authority regression

~~~text
CLASS=ProtosBug018FrameLexicalAuthorityReadAfterOwnerTierTransitionTest
TEST=retainedAuthorityReadOfPresentLocalSurvivesOwnerTransitionInAnotherActivation
~~~

Shape:

1. A1 runs the same root UNCACHED, installs its frame lexical authority and
   establishes `x`.
2. A1 returns while its authority retains the materialized frame.
3. A2 enters the same root and deterministically transitions it to CACHED
   without establishing `x`.
4. Read `x` through A1's retained authority.

Pre-fix expected failure: `FrameSlotTypeException`.
Post-fix result: exact A1 value.

The existing I075-D regression already proves that the project can force and
assert an UNCACHED -> CACHED transition deterministically; BUG018-C should reuse
that style rather than timing, sleeps, P races, or probabilistic loops.

## Minimal implementation surface

Expected product files include:

~~~text
src/main/java/com/guillermomolina/protos/execution/
  CanonicalToBytecodeLowerer.java
  ProtosBytecodeRootNode.java
  ProtosSemanticBytecodeRootNode.java
  ProtosFrameLexicalBindingAuthority.java
  ProtosFrameLexicalLayout.java
~~~

`ProtosInlineCallbackFrameBindings.java` is in scope only if the existing
captured-owner selection helper must be adjusted to preserve the same selected
frame/value-load split for inline callbacks.

Tests that assert the exact captured-read instruction identity may also need
mechanical updates, especially the PERF013/I068 captured-materialized lowering
regressions.

## Maintainer validation status

The maintainer reports the investigation checkout is clean and all local tests
pass:

~~~text
GIT_DIFF_CHECK=PASS
LOCAL_FULL_VALIDATION=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
BUG018_REPAIR_IMPLEMENTED=NO
~~~

Because BUG018-B is investigation-only, this validation establishes a clean
green baseline; it is not evidence that BUG018 itself is repaired.

## Slice result

~~~text
BUG018_B_MECHANISM_VERDICT=PROVEN
BUILTIN_LOAD_LOCAL_MATERIALIZED_ANALYZED=YES
BUILTIN_MATERIALIZED_SAFE_FOR_DIVERGED_TAGS=YES
BUILTIN_MATERIALIZED_LOWERING_FEASIBLE=YES
BYTECODE_GET_LOCAL_VALUE_ANALYZED=YES
GET_LOCAL_VALUE_SAFE_FOR_AUTHORITY=YES_WITH_BYTECODELOCATION_UPDATE
AUTHORITY_LEXICAL_LOCALS=ROOT_SCOPED
WRITE_FIX_REQUIRED=NO
UPSTREAM_TRUFFLE_BUG=YES
MINIMAL_FIX_BOUNDARY_IDENTIFIED=YES
SEMANTIC_CHANGE_REQUIRED=NO
D_OR_PLAT_DECISION_REQUIRED=NO
BUG018_READY_FOR_IMPLEMENTATION=YES
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE=BUG018-C
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
~~~
