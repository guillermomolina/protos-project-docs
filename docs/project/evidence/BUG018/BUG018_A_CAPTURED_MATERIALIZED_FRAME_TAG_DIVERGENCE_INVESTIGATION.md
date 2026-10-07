# BUG018-A — captured-materialized frame/local-tag divergence investigation

Date: 2026-10-05

## Work identity

~~~text
WORK_ITEM=BUG018
INVESTIGATION_SLICE=BUG018-A
PROTOS_ISSUE=guillermomolina/protos#801
RELATED_I058=guillermomolina/protos#661
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative investigation evidence. It does not replace
the live GitHub Issue state or the normative Protos specification.

## Exact inspected product state

~~~text
PROTOS_HEAD=04189acc0021ba3514e9937113efdb98b176b93e
PROTOS_HEAD_SUBJECT=I066-B: move canonical IP families to std:network
BUG018_STATE=OPEN
BUG018_STATUS=READY
BUG018_ROOT_CAUSE_VERDICT=PROVEN
SEMANTIC_CHANGE_REQUIRED=NO
PLATFORM_DECISION_REQUIRED=NO
~~~

No BUG018 repair had been published at evidence-capture time.

## Root cause

BUG018 is not an accessor/root/frame ownership mismatch.

The captured-materialized lowering creates the
`MaterializedLocalAccessor` from the owner root's own `BytecodeLocal` only when
owner and child are in the same `BytecodeRootNodes` group. The retained
`MaterializedFrame` comes from that same owner activation's
`ProtosFrameLexicalBindingAuthority`.

Truffle's `MaterializedLocalAccessor` stores `rootIndex`, `localOffset`, and
`localIndex`. On every access it uses the supplied current BytecodeNode only to
reach the shared root group, then resolves the current BytecodeNode of
`rootIndex`. Therefore the accessor, retained frame, and resolved declaring
root all identify the same owner root.

The defect is local-kind/tag incoherence across an uncached-to-cached tier
transition with multiple live/materialized frames of the same root:

~~~text
uncached activation A
  -> local written in retained/materialized frame F_A
  -> physical frame slot is PRESENT

another activation B of the same root
  -> triggers uncached -> cached transition
  -> cached BytecodeNode is published with localTags initialized ILLEGAL
  -> transition reconciles cached tags from B's currently executing frame only

retained F_A
  -> remains physically PRESENT
  -> cached node can still record ILLEGAL for that local
~~~

The resulting read is inconsistent in exactly this way:

~~~text
MaterializedLocalAccessor.isCleared(current-node, F_A)
  -> inspects physical frame tag
  -> false (PRESENT)

MaterializedLocalAccessor.getObject(current-node, F_A)
  -> resolves current cached declaring node
  -> cached local-kind metadata says ILLEGAL
  -> FrameSlotTypeException
~~~

This is BUG018 class E:

~~~text
E. local-kind/tag metadata became incoherent with the retained frame
~~~

Classes A, B, C and D are rejected:

- A: the current caller BytecodeNode need not be the owner root; this is allowed
  by the Truffle API and is used only to reach the shared root group.
- B: the accessor does not retain or dereference a stale declaring node; it
  resolves the declaring root's current BytecodeNode.
- C: the retained frame belongs to the owner activation.
- D: lowering admits the materialized fast path only when the owner
  `BytecodeLocal` is in the same physical root group.

## Truffle mechanism

The Protos generated interpreters enable both:

~~~text
enableUncachedInterpreter=true
boxingEliminationTypes={int.class}
~~~

In Truffle Bytecode DSL 25.4.4.1.1:

1. cached nodes own a `localTags_` array;
2. a new cached node initializes that array to `FrameSlotKind.Illegal.tag`;
3. the transition path populates cached tags for locals already stored in the
   currently transitioning frame;
4. `MaterializedLocalAccessor.getObject` resolves the declaring root's current
   BytecodeNode and calls its `getLocalValueInternal`;
5. the cached implementation selects the physical read using cached local-kind
   metadata and throws `FrameSlotTypeException` for cached `ILLEGAL`;
6. `MaterializedLocalAccessor.isCleared` instead checks the materialized
   frame's physical slot tag.

The two observations therefore can disagree for a retained frame that did not
participate in the tier transition.

The Bytecode DSL's built-in `LoadLocal` / `LoadLocalMaterialized` paths have
their own transition/slow-path handling and do not expose this exact public
accessor failure mode.

## Why P exposed the defect

The canonical `parallel-array-map` workload executes the same source-backed
worker root concurrently on P carriers.

One activation can publish the cached node while another activation still owns
a frame populated by uncached execution. An inline callback on the latter then
performs the captured-materialized read after publication of the cached node but
before that retained frame has any mechanism to reconcile its state with the
cached node's local-kind metadata.

This explains the historical combination:

~~~text
caller execution = uncached interpreter
access = MaterializedLocalAccessor
resolved declaring node = cached
frame = retained owner MaterializedFrame
host exception = FrameSlotTypeException
~~~

P creates the narrow concurrency window, but P is not necessary for a
deterministic regression.

## Existing-test gap

Existing captured-materialized tests cover ordinary capture, mutation by
reference, escaped capture, multi-depth capture, PRESENT(null), removal/fallback,
recreation, late nearer binding creation, owner/frame materialization, and
inline-callback semantics.

The missing dimension is:

~~~text
owner local already PRESENT in frame A
+
same owner root transitions UNCACHED -> CACHED through activation/frame B
+
frame A remains retained/alive
+
frame A is read through the new cached declaring node before a write reconciles it
~~~

The existing I075-D current-node regression is related but does not close this
gap: its binding is still ABSENT when the transition occurs and is created
afterward through the current-node authority.

## Deterministic regression design

~~~text
PROPOSED_TEST_CLASS=
ProtosBug018CapturedMaterializedReadAfterOwnerTierTransitionTest

PROPOSED_TEST_NAME=
escapedCapturedReadOfPresentLocalSurvivesOwnerTransitionInAnotherActivation
~~~

Shape:

1. Create a source root `make` that can return a closure capturing a local
   `x`.
2. First invocation A1 runs uncached, establishes `x`, and returns an escaped
   closure `r1` that reads `x`; A1's owner frame is retained.
3. Force the owner root's uncached threshold to zero.
4. Invoke `make` again as A2 so entry transitions the owner root to cached.
5. In A2, call `r1` before A2 establishes its own `x`.
6. Assert the root is cached and `r1` returns A1's exact captured value.

Expected pre-fix result:

~~~text
com.oracle.truffle.api.frame.FrameSlotTypeException
~~~

Expected post-fix result: the exact A1 captured object.

The reproduction is single-threaded and transition-driven; it requires no
sleep, race timing, benchmark loop, taskset affinity, or probabilistic retries.

## Repair boundary status

The causal boundary is closed, but the concrete read mechanism is not yet
selected.

The repair must make a physically PRESENT owner-frame local readable regardless
of stale/unpopulated cached local-kind metadata for another frame of the same
owner root. It must preserve:

- capture by reference;
- late lexical membership;
- PRESENT(null) != ABSENT;
- removal fallback;
- escaped closures;
- multi-depth capture;
- inline callback behavior;
- P/Future semantics;
- boxing elimination;
- cached Bytecode execution.

The same risk applies to `LocalRangeAccessor.getObject` reads made by
`ProtosFrameLexicalBindingAuthority`: using the declaring root's current node
is necessary but is not sufficient when the cached node's metadata was learned
from a different live frame.

## Candidate mechanisms for BUG018-B

The next investigation must choose the smallest safe read mechanism, in this
order:

1. Determine whether the built-in Bytecode DSL `LoadLocalMaterialized`
   mechanism can be used after Protos dynamically selects the lexical owner
   frame while retaining the compile-time-proven owner `BytecodeLocal`.
   This is the preferred candidate because the DSL already owns transition and
   boxing-elimination behavior for materialized local loads.
2. If that shape cannot express Protos's dynamic owner selection, evaluate the
   public `BytecodeNode.getLocalValue` family behind the existing
   `@TruffleBoundary` authority seams. Prove that it cannot turn PRESENT into
   ABSENT/null when cached metadata is ILLEGAL.
3. If neither public API can implement the invariant safely, identify the
   smallest upstream Truffle API/runtime issue and prepare separate upstream
   routing; do not hide the invalid read by catching
   `FrameSlotTypeException`.

No language-semantic or platform decision is currently required.

## Maintainer validation status

The maintainer reports all current local tests PASS at the inspected product
state.

~~~text
LOCAL_FULL_VALIDATION=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
BUG018_REPAIR_IMPLEMENTED=NO
~~~

This validation confirms the current baseline remains otherwise green; it does
not close BUG018 because the deterministic regression and repair do not exist
yet.

## Slice result

~~~text
BUG018_ROOT_CAUSE_VERDICT=PROVEN
ACCESSOR_OWNER_IDENTIFIED=YES
FRAME_OWNER_IDENTIFIED=YES
BYTECODE_NODE_ORIGIN_IDENTIFIED=YES
ACCESSOR_FRAME_ROOT_MATCH=YES
ACCESSOR_NODE_ROOT_MATCH=YES
CACHED_UNCACHED_ROLE=PROVEN
FRAMESLOTTYPEEXCEPTION_MECHANISM=PROVEN
EXISTING_TEST_GAP_IDENTIFIED=YES
DETERMINISTIC_REGRESSION_DESIGNED=YES
MINIMAL_FIX_BOUNDARY_IDENTIFIED=NO
SEMANTIC_CHANGE_REQUIRED=NO
D_OR_PLAT_DECISION_REQUIRED=NO
BUG018_READY_FOR_IMPLEMENTATION=NO
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_SLICE=BUG018-B
~~~
