# BUG018-C — tier-transition-safe retained-frame reads implementation

Date: 2026-10-05

## Work identity

~~~text
WORK_ITEM=BUG018
IMPLEMENTATION_SLICE=BUG018-C
PROTOS_ISSUE=guillermomolina/protos#801
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative implementation evidence. It does not
replace the live GitHub Issue state or the normative Protos specification.

## Exact published product state

~~~text
PROTOS_REVISION=234314b1791e5dc1c05ee2341ae46a56f8670512
PROTOS_REVISION_SUBJECT=BUG018-C: tier-transition-safe retained-frame reads
PROTOS_PARENT_REVISION=812b3f0b29dcba59562e3d03930c163db8529410
IMPLEMENTATION_VERSION=0.3.215-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
SPECIFICATION_CHANGED=NO
SEMANTIC_CHANGE=NO
D_OR_PLAT_DECISION_REQUIRED=NO
~~~

## Implemented mechanism

The commit implements the dual mechanism selected and proven by
[`BUG018-B`](BUG018_B_RETAINED_FRAME_SAFE_READ_MECHANISM_SELECTION.md). The
root cause is the one proven by
[`BUG018-A`](BUG018_A_CAPTURED_MATERIALIZED_FRAME_TAG_DIVERGENCE_INVESTIGATION.md):
a retained frame is physically PRESENT, but another activation of the same root
published a cached `BytecodeNode` whose local-kind metadata for that local is
still `ILLEGAL`.

### Compile-time-proven captured-materialized reads

~~~text
SelectCapturedMaterializedOwnerFrame[accessor](activation, name, depth)
    -> owner MaterializedFrame | null      (presence via accessor.isCleared; value never read)
store to block-local temporary
Conditional(
    IsCapturedOwnerFrameSelected(temp),
    LoadLocalMaterialized(ownerLocal, temp),   (built-in Bytecode DSL operation)
    ReadCapturedFallback(activation, name))    (unchanged generic captured lookup)
~~~

- The owner frame is still selected dynamically. `lexicalDepth`,
  `currentContextHasLocalSlotForRuntime`, `capturedOwnerWithoutNearerBinding`
  (D179 late nearer-binding retargeting), the owner authority, and the
  `isCleared` PRESENT check are unchanged.
- The selected value never encodes presence. `PRESENT(null) != ABSENT`.
- Admission is unchanged. `capturedOwnerBytecodeLocal` stays same-group only,
  and `LoadLocalMaterialized` applies the same builder scope validation
  (`validateMaterializedLocalScope`) as the former accessor operand.
- The inline-callback form uses `SelectInlineCapturedMaterializedOwnerFrame` and
  `ReadInlineCapturedFallback`. It selects without materializing the callback
  activation whenever the existing direct path admits it, which preserves the
  PLAT044/PERF025 pay-as-you-grow model.
- `ReadCapturedMaterializedLocal`, `ReadInlineCapturedMaterializedLocal`,
  `readCapturedMaterializedBindingOrNull` and
  `ProtosInlineCallbackFrameBindings.readCapturedMaterialized` are removed. No
  copy of `MaterializedLocalAccessor.getObject` remains on these read paths.

### `ProtosFrameLexicalBindingAuthority` retained reads

- The authority retains a `BytecodeLocation` of its declaring root, taken from
  a real location of that root:
  - `InstallFrameLexicalAuthority` and the frame-native transition operations
    (`BindClosureFrameParameter`, `BindClosureFrameRest`,
    `CreateCurrentFrameLocal`) bind `$bytecodeIndex`;
  - tooling uses the tag-tree node's enter bytecode index.
- Each PRESENT value read goes through one helper behind `@TruffleBoundary`:

  ~~~text
  current = retainedLocation.update()
  current.getBytecodeNode().getLocalValue(
      current.getBytecodeIndex(), retainedFrame, layout.localOffsetAt(ordinal))
  ~~~

  The helper is used by `readFrameBackedBindingAt`, `readBinding`,
  `bindingsSnapshot`, `appendBindingsToSlow` and `removeBinding` (the previous
  value). Presence still comes from the separate `isCleared` check.
- `ProtosFrameLexicalLayout` keeps public `BytecodeLocal.getLocalOffset()`
  integers only, never parse-local `BytecodeLocal` objects. Only a **root**
  lowering of the scope binds them (`bindRootLocalOffsets`), from the same
  root-scoped local array that forms the `LocalRangeAccessor`. Every later root
  lowering, including a Bytecode parser replay, must reproduce them exactly.
  The parser replay check keeps validating names (`requireSameNames`).
- An inline-callback lowering of the same lexical scope contributes no offsets.
  Its locals are block-scoped and their offsets depend on the enclosing
  context, and it never installs a `ProtosFrameLexicalBindingAuthority` (it
  uses a map-backed durable authority). This refinement was found during
  focal validation: binding inline offsets into the shared per-scope layout
  falsely tripped the replay check in
  `ProtosPerf025CallbackConsumerSpecializationTest`.

### Unchanged

~~~text
WRITE_PATH_CHANGED=NO        (only a bytecodeIndex parameter threaded through createCurrentFrameBinding)
BYTECODE_FRAME_USED=NO
CATCH_FRAMESLOTTYPEEXCEPTION=NO
ENABLE_BLOCK_SCOPING_CHANGED=NO
BOXING_ELIMINATION/UNCACHED/CACHED/INLINE_CALLBACKS/P/MATERIALIZED_CAPTURES=UNCHANGED
P_OR_FUTURE_SEMANTICS_CHANGED=NO
~~~

`UPSTREAM_TRUFFLE_BUG=YES` (`MaterializedLocalAccessor.getObject`) remains
documented by BUG018-B. It was not reported upstream in this slice and does not
block the Protos repair.

## Regressions

- `ProtosBug018CapturedMaterializedReadAfterOwnerTierTransitionTest` has three
  cases: an object value, `PRESENT(null)`, and an integer value. Activation A1
  of the owner root runs UNCACHED, establishes `x`, and lets `r1` escape.
  Activation A2 of the same root enters CACHED (`setUncachedThreshold(1)`) and
  invokes `r1` before establishing its own `x`. The test asserts the exact A1
  value and the tiers. Structural assertions (selector plus `load.local.mat`)
  run after the behavioral ones.
- `ProtosBug018FrameLexicalAuthorityReadAfterOwnerTierTransitionTest`: A1
  installs the authority and establishes `x` and `y` = `PRESENT(null)` while
  UNCACHED. A2 of the same root enters CACHED and establishes nothing. A1's
  authority is then read through its context. The test covers `readBinding`,
  snapshot order and values, removal (previous value), ABSENT, and recreation.
- Both regressions are deterministic. They use no sleep, timing races, retries,
  `taskset` or P concurrency.
- Structural assertions in PERF013-B1 and I068 Slices 5 and 7 now name
  `SelectCapturedMaterializedOwnerFrame` plus `load.local.mat`. Layout
  construction in the PERF012, I075-D, D179, PERF028A, BUG013 and PERF025
  Compact tests follows the new layout API. PERF012 adds bind/replay/inline
  layout cases.
- The local-range PE guard baselines
  (`tools/java_local_range_pe_guard_baseline.json` and
  `tools/java_local_range_pe_reachability_baseline.json`) are updated for the
  renamed signatures (added `int bytecodeIndex`) and the retired authority
  `getObject` sinks. The candidate diff contained no new risk classification.

## Validation

~~~text
COMPILE=PASS                              (mvn -q compile)
LOCAL_RANGE_PE_GUARD=FAIL->BASELINE_UPDATED (re-run result not separately reported)
FOCAL_SET=PASS                            (98 tests, 13 classes)
LOCAL_FULL_VALIDATION=PASS                (make test)
GIT_DIFF_CHECK=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
EXACT_SHA_CI_RUN=37325362553 (#2161)      IN_PROGRESS at record time
~~~

Validation ran on the substantive candidate before the version and changelog
metadata were added, which followed repository policy. The metadata-only
finalization was not re-tested.

## Remaining BUG018 acceptance

BUG018/#801 acceptance also requires that focused P `parallel-array-map`
execution no longer produce the exception. That workload validation is not
part of BUG018-C and has not yet been performed on this revision.

~~~text
BUG018_C_IMPLEMENTED=YES
CAPTURED_READ_USES_LOAD_LOCAL_MATERIALIZED=YES
INLINE_CAPTURED_READ_FIXED=YES
AUTHORITY_RETAINED_GETOBJECT_READS_REMOVED=YES
AUTHORITY_READ_USES_BYTECODE_LOCATION_UPDATE=YES
AUTHORITY_READ_USES_PUBLIC_LOCAL_OFFSET=YES
REGRESSION_CAPTURED_ADDED=YES
REGRESSION_AUTHORITY_ADDED=YES
P_WORKLOAD_REVALIDATION=PENDING
BUG018_STATE=OPEN
NEXT_SLICE=BUG018-D
NEXT_SLICE_TYPE=VALIDATION (executes commands in guillermomolina/protos; no product change expected)
~~~
