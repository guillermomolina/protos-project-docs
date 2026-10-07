# PERF025 — Object-model pay-as-you-grow representation

## Status

Published product implementation evidence for PERF025 / `guillermomolina/protos#758`.

This record is non-normative. It retains the exact product publication identity,
the object/member and frame-lexical pay-as-you-grow representation changes,
validation provenance, and the deliberate boundary that keeps a future Graal
`DynamicObject`/`Shape` migration outside this bounded implementation slice.

## Exact product publication

```text
PROTOS_REVISION=16432b0baf2fe9bd8e56acbc07bbef7ca0acf2cb
PROTOS_PARENT_REVISION=56f615628efb9c78fee06cbdba6ff21a952539f9
PROTOS_VERSION=0.3.160-SNAPSHOT
COMMIT_SUBJECT=PERF025: make object model generality pay as used
OWNING_ISSUE=guillermomolina/protos#758
BASE_IS_EXACT_PARENT=YES
```

The product publication is exactly one commit ahead of the preceding
`0.3.159-SNAPSHOT` PLAT044 lean-inline-callback preparation publication.

## Audit findings consumed

This slice directly consumes findings from:

```text
docs/project/evidence/PERF025/PERF025_POST_F1_PAY_AS_YOU_GROW_RUNTIME_COST_AUDIT.md
```

The bounded mapping is:

```text
AUDIT_FINDING_4_DIRECT_CLOSURE_CONTEXT_REPLACEMENT=MATERIALIZED
AUDIT_FINDING_11_EAGER_FRAME_DYNAMIC_ORDER_STRUCTURES=MATERIALIZED
AUDIT_FINDING_12_EAGER_OBJECT_SLOT_STORAGE=MATERIALIZED
AUDIT_FINDING_13_GENERAL_MEMBER_READ_BOUNDARY=PARTIALLY_MATERIALIZED
```

Finding 13 is only partially materialized: this publication removes the
unnecessary member-materialization `TruffleBoundary` and adds an ordinary
selector-stable PIC using the already-existing D013 lookup-invalidation
authority. It deliberately does **not** introduce Graal
`DynamicObject`/`Shape`, shared property Shapes, a new slot-key model, or an
object-model architecture migration.

## Empty ordinary-object storage

Before this slice, every ordinary `ProtosObjectValue` installed a fresh
`ProtosMapBackedLexicalBindingAuthority`, and that authority eagerly owned the
mutable slot map/order representation even when the object never acquired a
local slot.

The published representation introduces one shared storage-free empty lexical
authority:

```text
EMPTY_LEXICAL_BINDINGS
```

An ordinary object remains on that shared authority until the first local-slot
write requires mutable storage. The write path then promotes the object to a
`ProtosMapBackedLexicalBindingAuthority`.

The map-backed authority itself now treats:

```text
bindings == null
```

as its empty physical state. Its `LinkedHashMap` is allocated only on first
write and released again when removal empties the authority.

The same ordered map is sufficient for both value storage and ordinary
establishment order. The former separate order list is no longer maintained.

```text
EMPTY_OBJECT_PER_INSTANCE_SLOT_MAP=NO
MAP_STORAGE_BEFORE_FIRST_SLOT=NO
SEPARATE_MAP_BINDING_ORDER_STRUCTURE=NO
BACKING_MAP_RELEASED_WHEN_EMPTY=YES
ORDINARY_SLOT_SEMANTICS_CHANGED=NO
```

## Definitive execution-context authority

The post-F1 audit recorded a direct Closure/context path that could construct a
map-backed execution-context authority and then immediately replace it with the
real deferred/frame-backed authority.

The publication adds a runtime construction path that attaches the definitive
lexical authority when the execution Context is first materialized.

The common deferred/frame-backed shape is therefore:

```text
deferred lexical authority
  -> execution Context materialization
  -> Context constructed with that same authority
```

rather than:

```text
construct provisional map-backed authority
  -> construct Context
  -> migrate/replace with frame/deferred authority
```

Existing deferred-authority migration still handles the cases where authority
kind actually changes before observation; the change here removes the
unnecessary provisional representation when the definitive authority is already
known.

```text
PROVISIONAL_CONTEXT_MAP_AUTHORITY_ON_OBSERVATION=NO
DEFINITIVE_DEFERRED_AUTHORITY_ATTACHED_DIRECTLY=YES
CONTEXT_IDENTITY_SEMANTICS_CHANGED=NO
```

## Frame lexical bookkeeping

Before this slice, every `ProtosFrameLexicalBindingAuthority` eagerly
constructed:

```text
LinkedHashMap<String,Object> dynamicOverflow
LinkedHashSet<String> establishmentOrder
```

The published representation keeps both optional structures absent on the
ordinary static path.

Static monotonic establishment is represented by the frame-local presence state
plus one compact ordinal frontier. The full establishment-order set is
materialized only when the current history can no longer be reconstructed from
the static layout, such as non-monotonic remove/recreate history. Dynamic
bindings allocate `dynamicOverflow` only when a dynamic binding is actually
created.

Closure frame-layout declaration ordering is also aligned with unavoidable
runtime establishment order:

```text
Closure parameters
  -> body declarations
```

Parameter lexical identity still exists from Closure entry and parameter
presence is still established sequentially at each binding point. The ordering
change is backend layout order only; declaration-set membership remains
semantically order-independent.

```text
EAGER_DYNAMIC_OVERFLOW=NO
EAGER_ESTABLISHMENT_ORDER_SET=NO
STATIC_MONOTONIC_HISTORY_COMPACT=YES
GENERAL_HISTORY_MATERIALIZED_ON_DEMAND=YES
DYNAMIC_OVERFLOW_MATERIALIZED_ON_DEMAND=YES
PARAMETER_LEFT_TO_RIGHT_PRESENCE=PRESERVED
D179_REMOVE_RECREATE_SEMANTICS=PRESERVED
```

## Empty-method extraction fast path

A stored Closure selected as an ordinary member must still produce one fresh
receiver-bound Closure semantic identity for every successful extraction.

The publication therefore does **not** cache a bound Closure. It only avoids
allocating/copying local-slot lists in `bindMethod` when the stored Closure has
no local bindings.

```text
FRESH_BOUND_CLOSURE_PER_EXTRACTION=PRESERVED
EMPTY_BIND_METHOD_LOCAL_COPY=NO
CAPTURED_RECEIVER=PRESERVED
METHOD_HOME=PRESERVED
```

## Ordinary ReadMember PIC

Both generated Bytecode interpreter operation families now declare the same
bounded ordinary-member specialization.

The guarded entry is keyed by:

```text
exact receiver identity
constant/equal selector
ProtosValueLookup.GuardedLookup
selector-specific stability Assumption
PIC limit = 3
```

The cache retains the lookup selection, not the observable extracted value.
Every hit still calls `materializeMemberRead(receiver, selected)`.

That distinction is required for Closure-valued members:

```text
stored Closure selection cached
  -> member read hit
  -> bindMethod(receiver, home)
  -> fresh bound Closure

next hit
  -> same stable selection
  -> bindMethod(receiver, home)
  -> another fresh bound Closure
```

Same-selector mutation invalidates through the existing selector-specific D013
lookup dependency system. Unrelated-selector mutation retains the assumption.
Unsupported/represented forms and sites beyond the bounded PIC retain the exact
generic `ReadMember` fallback.

The member materialization helper is now PE-visible rather than forced across a
`TruffleBoundary`.

```text
READ_MEMBER_PIC_ENTRIES=3
LOOKUP_SELECTION_CACHED=YES
EXTRACTED_CLOSURE_CACHED=NO
FRESH_CLOSURE_EXTRACTION=PRESERVED
SAME_SELECTOR_INVALIDATION=PRESERVED
UNRELATED_SELECTOR_INVALIDATION=NO
GENERIC_FALLBACK=PRESERVED
MEMBER_MATERIALIZATION_TRUFFLE_BOUNDARY=NO
```

## Deliberate Shape boundary

The post-F1 audit also identified a broader mature-Truffle-object-model
opportunity. This publication intentionally stops before that boundary.

Not introduced:

```text
com.oracle.truffle.api.object.DynamicObject
Shape
shared cross-object property layout
canonical SlotKey redesign
delegation/object-model semantic redesign
```

The current PIC is an implementation-local specialization over existing
ordinary lookup authority. A future Shape investigation must separately address
dynamic create/assign/remove, delegation/shadowing, String-key semantics,
receiver-bound Closure extraction identity, invalidation, Context subclasses,
multi-Context ownership, and Native Image/runtime-compilation fit.

```text
SHAPE_MIGRATION_IMPLEMENTED=NO
SHAPE_RESEARCH_STILL_SEPARATE=YES
NEW_ARCHITECTURE_DECISION_SELECTED=NO
```

## Structural regression evidence

New test:

```text
ProtosPerf025ObjectModelPayAsYouGrowTest
```

It freezes:

- shared storage-free authority identity across empty ordinary objects;
- lazy promotion on first slot creation;
- lazy map allocation and release after last removal;
- ordered-map create/assign/remove/recreate behavior;
- direct execution-Context construction with its definitive authority; and
- fresh empty bound-Closure identity without local-slot storage.

Expanded:

```text
ProtosPerf025FrameMaterializationSliceTest
ProtosPerf025LazyLexicalCaptureTest
```

These additions freeze:

- compact static frame bookkeeping before general history is needed;
- lazy dynamic-overflow and establishment-order materialization;
- remove/recreate establishment history;
- guarded member-read invalidation;
- fresh Closure-valued member extraction; and
- bounded-PIC generic fallback.

Existing focal authority also includes:

```text
ProtosObjectValueTest
ProtosLexicalBindingAuthoritySeamTest
ProtosGuardedLookupTest
ProtosPerf025H1IndexedParameterEstablishmentTest
ProtosPerf012FrameLexicalLayoutTest
```

## Implementation-course evidence

The first complete focal run exposed one structural-test failure:

```text
TESTS_RUN=64
FAILURES=1
ERRORS=0
SKIPPED=1
FAILED_ASSERTION=monotonic static establishment stays in compact mode
OBSERVED_ESTABLISHMENT_ORDER=[value, other]
```

The failure was not a guest-semantic regression. Inspection established that the
compile-time scope discovered body declarations before declaring Closure
parameters, so the physical frame layout could be `[other, value]` while
runtime establishment necessarily occurred as `[value, other]`.

The bounded repair declared parameter identities before body declaration
discovery. This does not change scope membership or sequential parameter
presence; it only aligns backend frame-layout insertion order with the already
required parameter-before-body establishment order.

The frame/layout focal set then reported PASS, including:

```text
FRAME_AUTHORITY_PAY_AS_YOU_GROW=PASS
STATIC_BINDING_PRESENCE=NO
DUPLICATE_CREATION_ERROR=PASS
PRESENT_NULL_DISTINCT_FROM_ABSENT=PASS
D179_C0_REMOVE_RECREATE=PASS
ESTABLISHMENT_ORDER_SEMANTICS=PASS
CONTEXT_REFLECTION=PASS
CAPTURE_BY_REFERENCE=PASS
FRAME_NATIVE_PARAMETER_PATH_PRESERVED=YES
REST_SEMANTICS=PASS
OPEN_CLOSED_FROZEN_RULES=PASS
GENERIC_DYNAMIC_PARAMETER_FALLBACK_PRESERVED=YES
LEFT_TO_RIGHT_PARAMETER_BINDING=PASS
DEFAULT_SEMANTICS=PASS
STATIC_PARAMETER_USES_INDEXED_AUTHORITY_ESTABLISHMENT=YES
GENERIC_NAME_LOOKUP_FOR_STATIC_PARAMETER=NO
```

The complete focal regression then reported PASS, including:

```text
FRAME_AUTHORITY_PAY_AS_YOU_GROW=PASS
DEBUGGER_SCOPE_PROJECTION=PASS
MULTIPLE_CLOSURES_SHARE_ENVIRONMENT_IDENTITY=PASS
BIND_METHOD_CAPTURE_PRESERVED=PASS
PARALLEL_PROJECTION_BOUNDARY_PRESERVED=PASS
TRIVIAL_CLOSURE_DOES_NOT_FORCE_OUTER_CONTEXT=PASS
OBJECT_BODY_LEXICAL_BOUNDARY=PASS
GUARDED_MEMBER_READ_INVALIDATION=PASS
GUARDED_MEMBER_READ_FRESH_CLOSURE=PASS
GUARDED_MEMBER_READ_GENERIC_FALLBACK=PASS
LATE_NEARER_CREATION_RETARGETING=PASS
D179_C0_REMOVAL_FALLBACK=PASS
REMOVE_RECREATE=PASS
PRESENT_NULL_DISTINCT_FROM_ABSENT=PASS
CONTEXT_INTRINSIC=PASS
CONTEXT_REFLECTION=PASS
CAPTURE_BY_REFERENCE=PASS
LATER_MUTATION_VISIBLE=PASS
ESCAPED_CAPTURE_AFTER_OWNER_RETURN=PASS
MULTI_DEPTH_CAPTURE=PASS
TRANSITIVE_NESTED_CAPTURE=PASS
```

The maintainer then reported the repository integrated gate:

```text
make test=PASS
FULL_VALIDATION=PASS
```

After that green integrated gate, only required publication metadata
(`pom.xml` and `CHANGELOG.md`) was materialized. No executable/dependency
state changed after the full gate.

Final metadata preflight reported:

```text
PROTOS_LOCAL_HEAD=56f615628efb9c78fee06cbdba6ff21a952539f9
PROTOS_REMOTE_MAIN=56f615628efb9c78fee06cbdba6ff21a952539f9
TARGET_VERSION=0.3.160-SNAPSHOT
GIT_DIFF_CHECK=PASS
STAGED_DIFF_CHECK=PASS
FINAL_CHANGED_PATHS=17
```

The maintainer then committed and pushed:

```text
56f61562..16432b0b  HEAD -> main
PUBLICATION_PUSH=PASS
```

Therefore:

```text
MAINTAINER_REPORTED_FOCAL_VALIDATION=PASS
MAINTAINER_REPORTED_FULL_VALIDATION=PASS
INDEPENDENT_TEST_REEXECUTION_BY_COORDINATING_AGENT=NO
```

No command output beyond the maintainer-reported interaction is invented.

## Exact product delta

Exact comparison:

```text
BASE=56f615628efb9c78fee06cbdba6ff21a952539f9
HEAD=16432b0baf2fe9bd8e56acbc07bbef7ca0acf2cb
COMMITS=1
FILES_CHANGED=17
ADDITIONS=904
DELETIONS=78
```

Changed paths:

```text
M CHANGELOG.md
M pom.xml
M src/main/java/com/guillermomolina/protos/execution/CanonicalBindingAnalyzer.java
M src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
M src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java
M src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosClosureValue.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosExecutionContextValue.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosLexicalBindingAuthority.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosMapBackedLexicalBindingAuthority.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosObjectValue.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosPrelude.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosValueLookup.java
M src/test/java/com/guillermomolina/protos/execution/ProtosPerf025FrameMaterializationSliceTest.java
M src/test/java/com/guillermomolina/protos/execution/ProtosPerf025LazyLexicalCaptureTest.java
A src/test/java/com/guillermomolina/protos/runtime/ProtosPerf025ObjectModelPayAsYouGrowTest.java
```

Version publication:

```text
IMPLEMENTATION_VERSION=0.3.160-SNAPSHOT
ROOT_CHANGELOG_ENTRY=YES
SPECIFICATION_CHANGE=NO
```

## Performance-claim boundary

The maintainer explicitly requested this implementation line without
measurement. No benchmark, timing comparison, JFR campaign, allocation profile,
or exact-revision performance attribution was performed for this slice.

```text
BENCHMARK_RUN_FOR_THIS_SLICE=NO
PERFORMANCE_EFFECT_MEASURED=NO
END_TO_END_SPEEDUP_PERCENT=NOT_MEASURED
PERFORMANCE_MAGNITUDE_CLAIMED=NO
```

The durable claim is structural only: common object/member and frame-lexical
paths no longer instantiate several representations that are only needed by
less-common dynamic/general cases.

## PERF025 status

This bounded PERF025 implementation slice is complete. PERF025 itself remains
open for the continuing post-F1 pay-as-you-grow line.

```text
PERF025_OBJECT_MODEL_PAY_AS_YOU_GROW=COMPLETE
PERF025_STATUS=OPEN
OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
SHAPE_MIGRATION_IMPLEMENTED=NO
BENCHMARK_RESULT_CLAIMED=NO
```

## Materially inspected publication evidence

The coordinating publication review inspected:

- `guillermomolina/protos@16432b0baf2fe9bd8e56acbc07bbef7ca0acf2cb`;
- the exact one-commit comparison against
  `56f615628efb9c78fee06cbdba6ff21a952539f9`;
- the published `0.3.160-SNAPSHOT` version/changelog delta;
- PERF025 / `guillermomolina/protos#758`;
- the post-F1 pay-as-you-grow audit findings 4, 11, 12 and 13;
- existing D179/frame-layout/guarded-lookup semantic regression ownership; and
- the maintainer-reported focal and integrated validation outcomes.

This record is evidence only and does not replace live GitHub Issue
coordination.
