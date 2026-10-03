# PERF025 — Remaining non-benchmark work after 0.3.164

## Status

Durable non-normative planning/evidence record for PERF025 /
`guillermomolina/protos#758`.

This record answers one bounded question: after the published
`0.3.164-SNAPSHOT` node-aware Context slice, what PERF025 investigation or
implementation work still remains **before the final benchmark**, excluding the
separate RootTask/Task/Actor fixed-cost line currently being investigated
independently.

No benchmark magnitude is claimed here.

## Exact assessment base

```text
ASSESSED_PRODUCT_REVISION=52e43ff074aafd163240241762029159e8a043bc
ASSESSED_PRODUCT_VERSION=0.3.164-SNAPSHOT
ASSESSED_PRODUCT_SUBJECT=PERF025: make hot guest Context lookup node-aware
OWNING_ISSUE=guillermomolina/protos#758
SOURCE_AUDIT=docs/project/evidence/PERF025/PERF025_POST_F1_PAY_AS_YOU_GROW_RUNTIME_COST_AUDIT.md
```

The assessment cross-checks the original post-F1 pay-as-you-grow audit against
the product publications that followed it.

## Excluded from this residual list

The following are not counted as remaining non-benchmark work in this record:

- final benchmark / exact-revision performance comparison;
- RootTask/Task/Actor fixed per-`run()` bookkeeping, because that line is being
  investigated separately and does not multiply per recursive guest call;
- unconditional frame materialization for ordinary locals;
- universal Closure capture;
- rich direct source-Closure invocation;
- immediate compact-callee re-expansion;
- statically known Closure parameter establishment through generic named slots;
- observable/unobservable ReturnHome virtualization;
- guarded canonical Integer direct execution;
- small-Integer representation;
- eager frame lexical overflow/order structures;
- eager ordinary-object slot storage;
- the first exact-receiver/constant-selector member-read PIC;
- Map/IdentityMap exact recorded-hash indexing;
- Bytes reservation synchronization pay-as-you-grow;
- String Unicode proof/scalar-count preservation; and
- hot guest Polyglot Context lookup / repeated Context rediscovery.

Those lines have published implementation evidence or were moved to another
formal work item where appropriate.

## Remaining line 1 — Inline callback compact/frame-native execution

### Why it remains

PERF025 `0.3.159-SNAPSHOT` made PLAT044 B-prime literal callback
**preparation** lean: an admitted callback does not construct the rich ordinary
prepared call before admission.

However, at product revision
`52e43ff074aafd163240241762029159e8a043bc`, entering an admitted inline
callback still obtains a fresh semantic activation through:

```text
LoadInlineCallbackActivation
    -> PreparedInlineLiteralCall.activation()
    -> ProtosFrameArguments.activation(compactTargetArguments)
    -> ProtosActivation
```

The lowerer also deliberately disables the enclosing root's lexical analysis
inside `emitInlineLiteralCallback(...)`:

```text
currentRootAnalysis = null
currentRootTopScope = null
currentRootFrameLocals = Map.of()
currentActivationLocal = callbackActivation
```

Therefore the callback body does not reuse the already-proven frame-native /
direct-local lowering machinery and instead executes through the general
activation authority.

This combines residual audit findings 8 and 9.

### Investigation target

Determine whether an admitted inline literal callback can:

```text
keep parameters/locals frame-native in the containing semantic root
and
defer ProtosActivation materialization until some semantic observation
actually requires it
```

while preserving:

- one fresh semantic callback activation when it becomes observable;
- callback `context`;
- lexical capture by reference;
- parameter/default/rest semantics;
- non-local return / InvalidReturn;
- Error/ensure behavior;
- suspension/resumption;
- debugger scope projection and custom RootTag behavior;
- Task/dynamic-control provenance; and
- ordinary physical fallback for every non-admitted shape.

```text
RESIDUAL_LINE_1=INLINE_CALLBACK_COMPACT_FRAME_NATIVE_EXECUTION
STATUS=INVESTIGATION_REQUIRED
MULTIPLICATIVE_GUEST_COST=POSSIBLE
```

## Remaining line 2 — D179 lexical-membership fast-path assumption

### Why it remains

PERF028-A specialized statically resolved current-scope lexical writes, and the
frame-native PERF025 work specialized several current-scope binding operations.
Those changes deliberately preserve D179 dynamic membership semantics.

Current resolved reads still use presence machinery such as
`LocalAccessor.isCleared(...)`, and captured reads must still preserve nearer
binding retargeting, removal and recreation semantics.

The original audit explicitly left open whether ordinary unobserved lexical
contexts could avoid those checks behind an invalidatable stability assumption.

### Investigation target

Investigate a representation such as:

```text
LEXICAL_MEMBERSHIP_STABLE assumption

valid:
    proven current/captured access may take a narrower direct path

invalidated by the first operation that can make dynamic membership observable:
    remove
    dynamic creation
    relevant nearer-binding establishment
    reflection/context mutation
    other D179 membership-changing operations
```

The investigation must not weaken D179 C0/C3 semantics and must account for all
ways membership stability can become observable before proposing an
implementation.

```text
RESIDUAL_LINE_2=D179_LEXICAL_MEMBERSHIP_STABILITY_ASSUMPTION
STATUS=INVESTIGATION_REQUIRED
D179_SEMANTICS_MUST_REMAIN_EXACT=YES
```

## Remaining line 3 — Snapshot semantics without unconditional eager copy

### Why it remains

The Map/IdentityMap physical-index slice deliberately removed whole-collection
copies from keyed lookup while preserving genuine snapshot semantics for
iteration/matching/copy surfaces.

Several collection operations still implement a semantic snapshot as an eager
physical copy, including families of Array/Map/IdentityMap/Bytes iteration and
related stable-snapshot paths.

The language requires stable observable snapshot semantics. It does not
necessarily require an O(n) copy at snapshot creation when no later mutation
forces physical separation.

### Investigation target

Investigate one collection family at a time for a representation such as:

```text
immutable generation
versioned backing
copy-on-write backing
snapshot view + detach on first mutation
persistent storage
```

The chosen form must preserve exactly:

- snapshot membership/order/content at the semantic boundary;
- concurrent/isolated execution rules;
- mutation visibility rules;
- callback suspension/resumption;
- transfer/copy boundaries; and
- existing public protocol behavior.

Do not attempt Array + Map + IdentityMap + Bytes in one implementation slice
unless investigation proves one shared minimal mechanism is actually correct.

```text
RESIDUAL_LINE_3=COLLECTION_SNAPSHOT_PHYSICAL_REPRESENTATION
STATUS=INVESTIGATION_REQUIRED
SEMANTIC_SNAPSHOT_WEAKENING=FORBIDDEN
```

## Remaining line 4 — Shared-shape member lookup beyond exact-receiver PIC

### Why it remains

The `0.3.160-SNAPSHOT` object-model slice already implemented:

```text
lazy ordinary slot storage
+
three-entry exact-receiver / constant-selector ReadMember PIC
+
selector-specific invalidation
```

That closes the narrow member-read cost identified by the audit for repeated
reads of the same receiver identity.

The broader audit candidate remains: distinct ordinary objects with equivalent
physical slot/delegation structure do not currently share a Shape/property-style
lookup specialization.

### Investigation target

Determine whether Protos can safely cache member resolution by a shared physical
structure/version token rather than exact receiver identity, e.g.:

```text
shape-or-layout-token + selector
    -> local/delegated resolution path
```

while preserving:

- ordinary dynamic `:` creation;
- `=` assignment;
- local removal/recreation;
- delegation and shadowing;
- selector-specific invalidation;
- Closure extraction/rebinding freshness;
- representative method home;
- lookup Error behavior; and
- generic fallback.

The investigation must first determine whether a Protos-internal
shape/version-token scheme is sufficient. It must not introduce Graal
`DynamicObject`/`Shape` merely by analogy. If the correct solution requires a
durable runtime-architecture choice, promote that decision instead of silently
embedding it in PERF025.

```text
RESIDUAL_LINE_4=SHARED_SHAPE_MEMBER_LOOKUP
STATUS=INVESTIGATION_REQUIRED
ARCHITECTURAL_PROMOTION_POSSIBLE=YES
```

## Recommended order before final benchmark

Excluding the separate RootTask/Task/Actor line:

```text
1. INLINE_CALLBACK_COMPACT_FRAME_NATIVE_EXECUTION
2. D179_LEXICAL_MEMBERSHIP_STABILITY_ASSUMPTION
3. COLLECTION_SNAPSHOT_PHYSICAL_REPRESENTATION
4. SHARED_SHAPE_MEMBER_LOOKUP
5. FINAL_BENCHMARK
```

Lines 1 and 2 are the strongest remaining candidates for costs that may repeat
inside recursive/control-heavy guest execution. Line 3 is a real remaining
pay-as-you-grow representation cost but is workload-family dependent. Line 4 is
the broadest and may legitimately terminate in a separate architecture work
item rather than an implementation inside PERF025.

This order is an investigation sequence, not a measured performance ranking.

## Issue-granularity conclusion

At this checkpoint these remain bounded slices of PERF025. No new formal Issue
is allocated merely to mirror the four numbered lines.

If investigation later establishes an independent design gate, independent
blockage/dependency, or larger multi-publication architecture scope, the
corresponding line should then be promoted according to repository Issue-slice
governance.

```text
NEW_FORMAL_ISSUES_ALLOCATED=NO
PERF025_STATUS=OPEN
FINAL_BENCHMARK_PENDING=YES
PERFORMANCE_EFFECT_MEASURED=NO
BENCHMARK_RESULT_CLAIMED=NO
```

## Materially inspected evidence

This assessment inspected:

- `guillermomolina/protos@52e43ff074aafd163240241762029159e8a043bc`;
- PERF025 / `guillermomolina/protos#758`;
- the post-F1 pay-as-you-grow runtime-cost audit;
- current `CanonicalToBytecodeLowerer.emitInlineLiteralCallback(...)`;
- current `PreparedInlineLiteralCall.activation()`;
- current resolved lexical-access machinery and PERF028-A evidence;
- the `0.3.160-SNAPSHOT` object-model/member-read publication;
- Map/IdentityMap physical-index evidence; and
- current collection snapshot surfaces.

This record is planning/evidence only and does not replace live GitHub Issue
coordination.
