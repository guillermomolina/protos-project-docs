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

### Current state

The investigation has now produced and published a bounded first implementation
at exact product revision
`ffc351dca7363bcded452dd4d19d30e787cac391`
(`0.3.166-SNAPSHOT`, `PERF025: specialize stable current lexical reads`).

For statically `Resolved` bindings owned by the current genuine lexical root,
the lowered frame layout now carries a one-way Truffle `Assumption` per
root/name. While valid, ordinary current-scope reads avoid
`LocalAccessor.isCleared(...)`; the first successful static
`PRESENT -> ABSENT` removal invalidates that exact token before clearing the
local, and legal recreation does not renew it.

The compact unmaterialized current-root read is narrower still and reads the
proven local directly because that activation's guest Context has not yet become
observable.

The published implementation also keeps the exact layout/token identity stable
across Bytecode DSL retained-parser reparses, preventing an escaped pre-reparse
authority from invalidating an obsolete token while reparsed instructions trust
a fresh one.

Durable implementation evidence:
`PERF025_D179_A_STABLE_CURRENT_LEXICAL_READS.md`.

### What remains in this line

This slice deliberately does not specialize captured lexical membership.

Captured reads must still preserve:

- owner binding removal/recreation;
- nearer lexical binding introduction/removal;
- late nearer-binding retargeting;
- capture-by-reference semantics; and
- exact generic fallback.

A later bounded slice may use owner-presence continuity and/or a separate
"no nearer binding introduced" assumption if implementation evidence still
justifies it. Current/captured write selection and post-RHS validation were also
left unchanged; post-RHS selected-destination validation remains semantically
required.

```text
RESIDUAL_LINE_2=D179_LEXICAL_MEMBERSHIP_STABILITY_ASSUMPTION
STATUS=PARTIALLY_IMPLEMENTED
CURRENT_RESOLVED_READ_SLICE=COMPLETE
CURRENT_RESOLVED_READ_PRODUCT_REVISION=ffc351dca7363bcded452dd4d19d30e787cac391
CAPTURED_MEMBERSHIP_SPECIALIZATION=REMAINING_IF_JUSTIFIED
WRITE_SPECIALIZATION=NOT_PART_OF_D179_A
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
STATUS=PARTIALLY_IMPLEMENTED
ARRAY_FAMILY_SLICE=COMPLETE
ARRAY_PRODUCT_REVISION=f48f67a94553b8929bb3830005850d55ccc53cf3
OTHER_COLLECTION_FAMILIES=REMAINING_AS_APPLICABLE
SEMANTIC_SNAPSHOT_WEAKENING=FORBIDDEN
```

### Array-family publication

The first family-specific implementation is now published at exact product
revision `f48f67a94553b8929bb3830005850d55ccc53cf3`
(`0.3.166-SNAPSHOT`, `PERF025: make Array snapshots generation-backed`).

`ProtosArrayValue.indexedSnapshot()` now publishes a read-only view of the
current indexed generation without eagerly copying the complete element-reference
sequence. The first later indexed replacement detaches the live Array to a fresh
generation, so already-published shallow snapshots keep their exact references
and order.

This closes the Array sub-slice only. It does not establish that Map,
IdentityMap, Bytes, or any other snapshot-bearing family should use the same
physical representation without its own bounded validation.

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
STATUS=BOUNDED_H3A_IMPLEMENTED
PRODUCT_REVISION=7d507be8940528b045aa96426c7433ba040dfc83
GENERAL_SHAPE_REQUIRED_FOR_PERF025=NO
ARCHITECTURAL_PROMOTION_REQUIRED=NO
```

### Bounded implementation result — PERF025-H3A

The line-4 investigation selected a narrower implementation than a general
Shape migration.

At exact product revision
`7d507be8940528b045aa96426c7433ba040dfc83`,
distinct exact ordinary sibling receivers with the same exact direct parent can
share an inherited selector selection. Lookup dependency registration begins at
the parent, while each actual receiver dynamically guards that it still lacks
the selector locally.

The existing exact-receiver PIC remains for own-local and other admitted cases,
and generic lookup remains authoritative outside the bounded specializations.

Parent/ancestor mutation of the selected selector invalidates through the
existing selector-specific D013 Assumption. A local shadow on one sibling makes
that sibling fail the local-absence guard without invalidating the shared
parent-chain selection for other siblings. Removing the shadow makes the shared
descriptor eligible again if its parent-chain Assumption is still valid.

Closure extraction still calls
`materializeMemberRead(actualReceiver, selected)`, preserving fresh extraction
identity, the actual sibling as captured receiver, and the selected owner as
method home.

No general Protos Shape, Graal `DynamicObject`, ordinal own-slot storage, or
new platform decision was introduced.

Durable implementation evidence:
`PERF025_H3A_SHARED_INHERITED_MEMBER_LOOKUP.md`.

## Recommended order before final benchmark

Excluding the separate RootTask/Task/Actor line:

```text
1. INLINE_CALLBACK_COMPACT_FRAME_NATIVE_EXECUTION
2. D179_LEXICAL_MEMBERSHIP_STABILITY_ASSUMPTION
   - current Resolved reads: COMPLETE at ffc351dca7363bcded452dd4d19d30e787cac391
   - captured membership specialization: continue only if separately justified
3. COLLECTION_SNAPSHOT_PHYSICAL_REPRESENTATION
   - Array family: COMPLETE at f48f67a94553b8929bb3830005850d55ccc53cf3
   - other families: continue only where separately justified
4. FINAL_BENCHMARK
```

PERF025-H3A closes the bounded shared-inherited member-lookup line at
`7d507be8940528b045aa96426c7433ba040dfc83`; a general Shape migration is not
required by PERF025 on this evidence.

Line 1 remains a candidate for costs that may repeat inside
recursive/control-heavy guest execution. Line 2 is partially advanced: current
Resolved reads are complete, while captured-membership specialization remains a
bounded follow-up only if separately justified. Line 3 remains family-dependent
after the completed Array slice.

This order is a work sequence, not a measured performance ranking.

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
RESIDUAL_LINE_2_CURRENT_RESOLVED_READ_SLICE=COMPLETE
RESIDUAL_LINE_2_CAPTURED_MEMBERSHIP=REMAINING_IF_JUSTIFIED
RESIDUAL_LINE_3_ARRAY_SLICE=COMPLETE
RESIDUAL_LINE_4_H3A=COMPLETE
GENERAL_SHAPE_PROMOTION_REQUIRED=NO
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

## Current-head revalidation after PERF025-G2

While this record was being published, product `main` advanced by one commit:

```text
CURRENT_PRODUCT_REVISION=32603c6896262e1a731714986d92da81c0d56aca
CURRENT_PRODUCT_VERSION=0.3.165-SNAPSHOT
CURRENT_PRODUCT_SUBJECT=PERF025: compact RootTask fixed-cost bookkeeping
PREVIOUS_ASSESSMENT_BASE=52e43ff074aafd163240241762029159e8a043bc
COMMITS_ADVANCED=1
```

The G2 delta is confined to:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/runtime/ProtosActorExecutionDomain.java
src/main/java/com/guillermomolina/protos/runtime/ProtosTask.java
src/test/java/com/guillermomolina/protos/runtime/ProtosPerf025G2RootTaskBookkeepingTest.java
```

It does not touch the inline-callback lowering/activation, lexical membership,
collection snapshot, or ordinary member-read surfaces that own residual lines
1–4.

Therefore the four-line residual sequence remains applicable at
`32603c6896262e1a731714986d92da81c0d56aca`.

```text
ROOT_TASK_LINE_ADVANCED=YES
RESIDUAL_LINES_1_TO_4_SOURCE_OVERLAP=NO
RESIDUAL_SEQUENCE_REVALIDATED_AT_0_3_165=YES
```

This record is planning/evidence only and does not replace live GitHub Issue
coordination.


## Current-head revalidation after Array snapshot and PERF025-H3A

Product `main` subsequently advanced through:

```text
ARRAY_SNAPSHOT_REVISION=f48f67a94553b8929bb3830005850d55ccc53cf3
ARRAY_SNAPSHOT_VERSION=0.3.166-SNAPSHOT
ARRAY_SNAPSHOT_SUBJECT=PERF025: make Array snapshots generation-backed

H3A_REVISION=7d507be8940528b045aa96426c7433ba040dfc83
H3A_PARENT=f48f67a94553b8929bb3830005850d55ccc53cf3
H3A_VERSION=0.3.166-SNAPSHOT
H3A_SUBJECT=PERF025: share inherited member lookup across siblings
```

The H3A publication changes only the two Bytecode roots, ordinary-object/value
lookup support, and focused tests. It does not overlap the Array generation
representation introduced by its exact parent.

Owner-executed H3A validation reported Maven compile PASS, 28 focused tests with
0 failures / 0 errors / 1 skipped, full `make test` PASS, and
`git diff --check` PASS. No exact full-suite test count was supplied for that
final run and none is inferred here.

```text
RESIDUAL_LINE_3_ARRAY_ADVANCED=YES
RESIDUAL_LINE_3_ALL_FAMILIES_CLOSED=NO
RESIDUAL_LINE_4_H3A_COMPLETE=YES
GENERAL_SHAPE_MIGRATION_REQUIRED=NO
CURRENT_PRODUCT_REVISION=7d507be8940528b045aa96426c7433ba040dfc83
CURRENT_PRODUCT_VERSION=0.3.166-SNAPSHOT
FINAL_BENCHMARK_PENDING=YES
```


## Current-head revalidation after PERF025-D179-A

Product `main` subsequently advanced by one exact commit:

```text
D179_A_REVISION=ffc351dca7363bcded452dd4d19d30e787cac391
D179_A_PARENT=7d507be8940528b045aa96426c7433ba040dfc83
D179_A_VERSION=0.3.166-SNAPSHOT
D179_A_SUBJECT=PERF025: specialize stable current lexical reads
```

The publication specializes only current-scope statically `Resolved` lexical
reads. It introduces one-way root/name membership assumptions, invalidates the
exact token before static removal clears the local, and preserves token identity
across Bytecode DSL reparses by reusing the exact frame layout for a retained
`CanonicalLexicalScope`.

The publication deliberately leaves captured paths, write selection, creation,
`Candidate`, and `Dynamic` behavior unchanged.

Owner-executed validation reported:

```text
MAVEN_COMPILE=PASS
FOCAL_TESTS_AFTER_REPARSE_FIX=4_PASSED_0_FAILED
RELATED_D179_LEXICAL_TESTS=79_PASSED_0_FAILED
FINAL_MAKE_TEST=PASS
FINAL_PROTOS_TESTS=1284_PASSED_0_FAILED
FINAL_GIT_DIFF_CHECK=PASS
```

No benchmark, timing comparison, JFR, IGV, or allocation measurement was
performed for this slice.

```text
RESIDUAL_LINE_2_CURRENT_RESOLVED_READS_ADVANCED=YES
RESIDUAL_LINE_2_CURRENT_RESOLVED_READS_COMPLETE=YES
RESIDUAL_LINE_2_CAPTURED_MEMBERSHIP_CLOSED=NO
D179_SEMANTICS_CHANGED=NO
CURRENT_PRODUCT_REVISION=ffc351dca7363bcded452dd4d19d30e787cac391
CURRENT_PRODUCT_VERSION=0.3.166-SNAPSHOT
FINAL_BENCHMARK_PENDING=YES
```


## Current reconciliation after frame-native callbacks and IdentityMap snapshots

Product `main` subsequently advanced through:

```text
INLINE_CALLBACK_FRAME_NATIVE_REVISION=4fa64c3821e9e6566fd9a1f0b38fdfcba875fd78
INLINE_CALLBACK_FRAME_NATIVE_VERSION=0.3.166-SNAPSHOT
INLINE_CALLBACK_FRAME_NATIVE_SUBJECT=PERF025: frame-native inline callback bindings

IDENTITYMAP_SNAPSHOT_REVISION=85bd050e8706c02205db4ede8824681a1180d075
IDENTITYMAP_SNAPSHOT_PARENT=4fa64c3821e9e6566fd9a1f0b38fdfcba875fd78
IDENTITYMAP_SNAPSHOT_VERSION=0.3.167-SNAPSHOT
IDENTITYMAP_SNAPSHOT_SUBJECT=PERF025: make IdentityMap snapshots generation-backed
```

The inline-callback publication completes static frame-local authority for
admitted PLAT044 B-prime callbacks while deliberately leaving semantic callback
`ProtosActivation` eager. The next callback slice is therefore the bounded
lazy semantic callback Activation work. A further Activation-consumer
specialization remains conditional and must be reconsidered only after that
slice; it is not independently justified in advance.

The IdentityMap publication removes unconditional O(n) snapshot publication
copies with a generation-backed copy-on-write representation and removes the
redundant second structured-`each` copy. Exact semantic identity, recorded
identity hash buckets, insertion order, snapshot stability, state rules, Actor
transfer and isolated-P transfer are preserved.

The bounded follow-up investigations for the other residual lines have now also
been reconciled:

```text
RESIDUAL_LINE_1_INLINE_CALLBACK:
  STATIC_FRAME_LOCAL_AUTHORITY=COMPLETE
  LAZY_SEMANTIC_ACTIVATION=NEXT
  RESIDUAL_ACTIVATION_CONSUMER_SPECIALIZATION=OPTIONAL_AFTER_LAZY_ACTIVATION_IF_JUSTIFIED

RESIDUAL_LINE_2_D179:
  CURRENT_RESOLVED_READ_MEMBERSHIP=COMPLETE
  CAPTURED_MEMBERSHIP_SPECIALIZATION=NOT_JUSTIFIED
  FURTHER_PRODUCT_SLICE=NO

RESIDUAL_LINE_3_COLLECTION_SNAPSHOTS:
  ARRAY=COMPLETE
  MAP_ADDITIONAL_SNAPSHOT_REPRESENTATION=NOT_JUSTIFIED
  IDENTITYMAP=COMPLETE
  BYTES_ADDITIONAL_SNAPSHOT_REPRESENTATION=NOT_JUSTIFIED
  FURTHER_PRODUCT_SLICE_OUTSIDE_CALLBACK_LINE=NO

RESIDUAL_LINE_4_SHARED_SHAPE:
  H3A=COMPLETE
  GENERAL_SHAPE_REQUIRED_FOR_PERF025=NO

ROOT_TASK_TASK_ACTOR_LINE=COMPLETE
FINAL_BENCHMARK_PENDING=YES
PERF025_STATUS=OPEN
```

Accordingly, at this checkpoint the only remaining PERF025 technical sequence is:

```text
1. LAZY_SEMANTIC_INLINE_CALLBACK_ACTIVATION
2. OPTIONAL_RESIDUAL_ACTIVATION_CONSUMER_SPECIALIZATION
   - execute only if post-slice evidence justifies it
3. FINAL_BENCHMARK
```

No additional Map, Bytes, D179, Shape, Actor/Task or collection-snapshot
implementation slice remains before the final benchmark.

Durable family-specific evidence for the IdentityMap publication is:
`PERF025_IDENTITYMAP_GENERATION_BACKED_SNAPSHOTS.md`.
