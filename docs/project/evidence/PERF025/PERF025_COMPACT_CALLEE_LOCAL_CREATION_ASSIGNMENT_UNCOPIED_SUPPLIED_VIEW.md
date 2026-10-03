# PERF025 — Compact callee local creation/assignment and uncopied supplied view

## Status

Published product implementation evidence for PERF025 / `guillermomolina/protos#758`.

This record is non-normative. It retains the exact product publication identity,
the structural implementation result, and the maintainer-reported validation
outcome for the compact-callee follow-up immediately after PERF025-H1.

## Exact product publication

```text
PROTOS_REVISION=ce2db129102fd69b3bc442b647c8994e7e565d90
PROTOS_PARENT_REVISION=a7e26228543f1af65a75e2e95bd6e384b67cc7d3
PROTOS_VERSION=0.3.154-SNAPSHOT
COMMIT_SUBJECT=PERF025: compact callee local creation/assignment and uncopied supplied view
OWNING_ISSUE=guillermomolina/protos#758
BASE_IS_EXACT_PARENT=YES
```

The exact parent is PERF025-H1, which removed unconditional rich
`ProtosActivation` publication at compact source-root entry.

The published delta is exactly one commit ahead of that parent.

## Bounded objective

PERF025-H1 established this source-root shape:

```text
compact source-call frame arguments
  -> source root
  -> frame-native parameter binding / ordinary reads
  -> rich ProtosActivation only when observed
```

The residual problem was that ordinary body-level current-local creation and
assignment could still force the current rich activation even though the
lowerer already knew the exact current-root frame binding.

This slice extends compact callee execution through those operations and also
removes the supplied-argument `List.copyOf` performed when a compact call is
eventually materialized.

The bounded target is therefore:

```text
compact source call
  -> frame-native parameter binding
  -> frame-native current-local creation
  -> frame-native statically resolved current-local assignment
  -> ordinary return

materialize exact rich state only when a semantic/runtime observation or
fallback really requires it
```

No benchmark or timing claim is part of this publication.

## Compact current-local creation

For a root lowered without persistent frame authority,
`CreateCurrentFrameLocal` no longer requires an eagerly supplied
`ProtosActivation`.

While the source-call frame remains unmaterialized and the target local is
ABSENT, the operation establishes the value directly through the frame-native
local range.

Conceptually:

```text
compact frame + ABSENT statically known current local
  -> LocalRangeAccessor.setObject(...)
  -> remain compact
```

If the slot is already PRESENT, or if the invocation is no longer in the
unmaterialized compact state, the operation materializes/reuses the exact
activation and delegates to the unchanged rich creation path.

This preserves ordinary slot-creation conflict behavior rather than turning
creation into assignment.

```text
COMPACT_CURRENT_LOCAL_CREATION=YES
DUPLICATE_CREATION_GENERIC_ERROR_PATH=PRESERVED
PARAMETER_OR_LOCAL_PRESENT_NULL_IS_PRESENT=PRESERVED
```

## Compact statically resolved current-local assignment

The lowerer now emits root-level current-frame write operations when the
current activation is the root's own:

```text
ResolveRootFrameLocalWriteTarget
AssignRootFrameLocal
```

For a still-compact invocation and a PRESENT statically resolved local,
selection returns the static current-frame destination without materializing the
activation. The assignment then writes through the constant `LocalAccessor`
when that same selected destination remains valid after RHS evaluation.

Conceptually:

```text
PRESENT statically resolved current-root local
  -> select STATIC_CURRENT_FRAME_LOCAL before RHS
  -> evaluate RHS
  -> revalidate presence, never retarget
  -> LocalAccessor.setObject(...)
  -> remain compact
```

If the binding is ABSENT at selection time, becomes cleared during RHS
evaluation, the context has already materialized/frozen, or any other
non-eligible state is encountered, the exact pre-existing activation-based
selection/assignment path is used.

This preserves the PERF028 destination-before-RHS contract and D179 C0
PRESENT/ABSENT behavior.

```text
STATIC_CURRENT_ROOT_ASSIGNMENT_REQUIRES_RICH_ACTIVATION=NO
DESTINATION_SELECTED_BEFORE_RHS=PRESERVED
POST_RHS_RETARGETING=NO
D179_C0_ABSENT_FALLBACK=PRESERVED
FROZEN_CONTEXT_REJECTION=PRESERVED
GENERIC_ASSIGNMENT_FALLBACK=PRESERVED
```

## Supplied arguments remain backed by compact frame arguments

Before this slice, materializing a compact invocation constructed a list view
over the supplied frame-argument range and `DeferredSuppliedArguments` then
copied it with `List.copyOf`.

The published implementation introduces a private read-only random-access
`FrameBackedSuppliedArguments` view over the immutable supplied range of the
compact frame-argument array.

`ProtosFrameArguments.activation(...)` now passes that view to the deferred
invocation factory, and `DeferredSuppliedArguments` adopts this specific view
without copying it.

Thus materializing the rich activation no longer implies a second physical copy
of the supplied argument vector.

```text
SUPPLIED_ARGUMENT_LIST_COPY_ON_MATERIALIZATION=NO
SUPPLIED_ARGUMENT_BACKING=COMPACT_FRAME_ARGUMENT_ARRAY
SUPPLIED_VIEW_READ_ONLY=YES
GUEST_ARGUMENT_ARRAY_STILL_DEFERRED=YES
```

This does not claim that no Java object is allocated: the read-only view is a
small object. The structural claim is specifically that the supplied values are
not copied into a second list backing store.

## Task provenance after lazy materialization

`ProtosFrameArguments.compactTask(...)` now recognizes a previously published
`ProtosActivation` in frame argument 0 and returns that activation's exact Task
when present.

This preserves Task identity across the compact-to-materialized transition,
including the task-owned direct source Closure path.

```text
TASK_PROVENANCE_BEFORE_MATERIALIZATION=PRESERVED
TASK_PROVENANCE_AFTER_MATERIALIZATION=PRESERVED
SAME_TASK_IDENTITY=PASS
```

## Context, lexical authority and BUG013 boundary

This slice does not reintroduce universal activation or frame-authority
materialization.

Ordinary parameter/local-only source roots may continue using their live
`VirtualFrame` and Bytecode locals while the invocation is compact.

When a semantic/runtime operation requires the rich lexical representation, the
existing on-demand materialization path remains authoritative.

The BUG013 lifetime invariant remains unchanged:

```text
RAW_VIRTUALFRAME_RETAINED_ACROSS_ROOT_LIFETIME=NO
PERSISTENT_FRAME_AUTHORITY_USES_MATERIALIZED_FRAME=YES
BUG013_LIFETIME_INVARIANT=PRESERVED
```

The compact local operations do not introduce a second binding-value store.

```text
ONE_SEMANTIC_BINDING_VALUE_AUTHORITY=YES
NO_DUAL_BINDING_STORE=YES
```

## Parameter/default/rest and D179 semantics

The dedicated regression retains the earlier compact parameter path and covers
the semantic boundaries relevant to extending compact execution into the body:

```text
LEFT_TO_RIGHT_PARAMETER_BINDING=PASS
DEFAULT_BINDING_VISIBILITY=PASS
LATE_CONTEXT_PROJECTION=PASS
REST_ARRAY_SEMANTICS=PASS
D179_PRESENT_ABSENT=PASS
D179_REMOVE_RECREATE=PASS
```

In particular:

- a default can observe an earlier parameter while a later parameter is still
  absent;
- a default cannot observe a future later-parameter value;
- late `context` observation projects bindings already established in the
  compact frame;
- duplicate local creation retains its ordinary Error;
- `PRESENT(null)` remains distinct from ABSENT;
- remove/recreate semantics remain exact.

## Capture, Error, ReturnHome and tooling

The focal regression also exercises transitions that intentionally require or
observe rich state.

Retained behavior includes:

```text
CAPTURE_BY_REFERENCE=PASS
ERROR_HANDLER_SELECTION=PASS
NON_LOCAL_RETURN=PASS
MATERIALIZED_ACTIVATION_IDENTITY_PER_INVOCATION=ONE
FRESH_CONTEXT_IDENTITY=PASS
TASK_DYNAMIC_CONTROL=PASS
```

Debugger/tooling scope observation reuses the exact same materialized
activation after first projection.

The slice does not redesign `ProtosReturnHome`, NLR semantics, Closure capture,
Task scheduling, Actor ownership, or dynamic-control architecture.

```text
RETURN_HOME_REPRESENTATION_CHANGED=NO
CLOSURE_CAPTURE_ARCHITECTURE_CHANGED=NO
TASK_ARCHITECTURE_CHANGED=NO
ACTOR_ARCHITECTURE_CHANGED=NO
NEW_PLAT_DECISION=NO
```

## Dedicated structural regression

The product publication adds:

```text
src/test/java/com/guillermomolina/protos/execution/
  ProtosPerf025CompactCalleeExecutionTest.java
```

The regression directly checks, among other points:

```text
SOURCE_ROOT_UNIVERSAL_PUBLISH_FRAME_ACTIVATION=NO
CALLEE_ACTIVATION_EAGER=NO
FRAME_MATERIALIZATION_EAGER=NO
PARAMETER_DIRECT_LOCAL_BINDING=YES
IMMEDIATE_METHOD_COMPACT_PATH=PASS
DIRECT_SOURCE_CLOSURE_COMPACT_PATH=PASS
ORDINARY_LOCAL_ONLY_ROOT_EAGER_FRAME_MATERIALIZATION=NO
LEFT_TO_RIGHT_PARAMETER_BINDING=PASS
MISSING_ARGUMENT_PRECEDENCE=PASS
EXCESS_ARGUMENT_PRECEDENCE=PASS
DEFAULT_BINDING_VISIBILITY=PASS
LATE_CONTEXT_PROJECTION=PASS
FRESH_CONTEXT_IDENTITY=PASS
D179_PRESENT_ABSENT=PASS
D179_REMOVE_RECREATE=PASS
REST_ARRAY_SEMANTICS=PASS
CAPTURE_BY_REFERENCE=PASS
ERROR_HANDLER_SELECTION=PASS
NON_LOCAL_RETURN=PASS
MATERIALIZED_ACTIVATION_IDENTITY_PER_INVOCATION=ONE
SUPPLIED_ARGUMENT_LIST_COPY_ON_ENTRY=NO
TASK_DYNAMIC_CONTROL=PASS
```

Existing PERF025 frame-materialization and PERF028 lexical-write structural
tests were adjusted to recognize the root-level compact write operations.

## Exact product delta

The exact one-commit comparison from
`a7e26228543f1af65a75e2e95bd6e384b67cc7d3` to
`ce2db129102fd69b3bc442b647c8994e7e565d90` changes 10 paths:

```text
M CHANGELOG.md
M pom.xml
M src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
M src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java
M src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java
A src/test/java/com/guillermomolina/protos/execution/ProtosPerf025CompactCalleeExecutionTest.java
M src/test/java/com/guillermomolina/protos/execution/ProtosPerf025FrameMaterializationSliceTest.java
M src/test/java/com/guillermomolina/protos/execution/ProtosPerf028AResolvedCurrentLexicalWriteTest.java
M tools/java_slow_tests_allowlist.txt

FILES_CHANGED=10
ADDITIONS=703
DELETIONS=37
```

Version publication:

```text
IMPLEMENTATION_VERSION=0.3.154-SNAPSHOT
ROOT_CHANGELOG_ENTRY=YES
SPECIFICATION_CHANGE=NO
```

## Validation provenance

The maintainer reported in the active interaction after product publication:

```text
PERF025: compact callee local creation/assignment and uncopied supplied view pushed all tests passed
```

The coordinating publication review independently verified:

```text
REMOTE_PUBLICATION=PASS
PRODUCT_REVISION=ce2db129102fd69b3bc442b647c8994e7e565d90
COMMIT_SUBJECT_MATCH=PASS
EXACT_PARENT=a7e26228543f1af65a75e2e95bd6e384b67cc7d3
PRODUCT_VERSION=0.3.154-SNAPSHOT
FILES_CHANGED=10
ADDITIONS=703
DELETIONS=37
FOCAL_TEST_PUBLISHED=PASS
CHANGELOG_RECORD=PASS
```

Therefore:

```text
MAINTAINER_REPORTED_VALIDATION=PASS
INDEPENDENT_TEST_REEXECUTION_BY_COORDINATING_AGENT=NO
```

No command-by-command test result beyond the maintainer's report is invented.

## PLAT040 F-prime convergence

This slice continues the already selected PLAT040 F-prime architecture:

```text
stable selected source call
  -> compact frame-argument ABI
  -> compact callee parameter/local execution
  -> exact rich state only on semantic/runtime observation
  -> exact generic fallback
```

The result advances the compact representation deeper into ordinary callee
execution instead of treating the compact ABI as only a target-boundary
transport.

```text
COMPACT_ABI_ONLY_AT_BOUNDARY=NO
COMPACT_CALLEE_LOCAL_CREATION=YES
COMPACT_CALLEE_CURRENT_LOCAL_ASSIGNMENT=YES
ON_DEMAND_RICH_ACTIVATION=PRESERVED
EXACT_GENERIC_FALLBACK=PRESERVED
OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
NEW_PLATFORM_DECISION_REQUIRED=NO
```

## PERF025 audit consequence

Relative to the retained post-F1 pay-as-you-grow audit and H1:

```text
COMPACT_SOURCE_ROOT_IMMEDIATE_RICH_ACTIVATION_REINFLATION
    -> consumed by PERF025-H1

BODY_LOCAL_CREATION_FORCES_RICH_ACTIVATION
    -> consumed for eligible compact current-root frame-native locals by ce2db129...

STATIC_CURRENT_ROOT_ASSIGNMENT_FORCES_RICH_ACTIVATION
    -> consumed for eligible PRESENT resolved current-root locals by ce2db129...

SUPPLIED_ARGUMENT_VECTOR_COPIED_WHEN_RICH_ACTIVATION_MATERIALIZES
    -> consumed by ce2db129...

RETURN_HOME_PHYSICALLY_CREATED_WHEN_UNOBSERVED
    -> not addressed by this slice

INLINE_CALLBACK_STILL_USES_RICH_ACTIVATION
    -> not addressed by this slice

INLINE_CALLBACK_LOSES_FRAME_NATIVE_LEXICAL_PATH
    -> not addressed by this slice
```

The product commit remains authority for the exact current code shape; this
record retains only this bounded publication consequence.

## Performance-claim boundary

No benchmark, timing campaign, JFR profile, or attributable per-call magnitude
is part of this publication.

```text
BENCHMARK_RUN_FOR_THIS_SLICE=NO
ATTRIBUTABLE_NS_PER_CALL=NOT_MEASURED
END_TO_END_SPEEDUP_PERCENT=NOT_MEASURED
PERFORMANCE_MAGNITUDE_CLAIMED=NO
```

The durable claim is structural: eligible compact source callees can now retain
compact invocation state through ordinary body local creation and resolved
current-local assignment, and later rich activation projection no longer copies
the supplied values into a second list.

## PERF025 status

This bounded implementation slice is complete. PERF025 itself remains open.

```text
PERF025_COMPACT_CALLEE_LOCAL_SLICE=COMPLETE
PERF025_STATUS=OPEN

COMPACT_CURRENT_LOCAL_CREATION=YES
COMPACT_CURRENT_LOCAL_ASSIGNMENT=YES
SUPPLIED_ARGUMENT_LIST_COPY_ON_MATERIALIZATION=NO
ON_DEMAND_RICH_ACTIVATION=PRESERVED
BUG013_LIFETIME_INVARIANT=PRESERVED
D179_SEMANTICS=PRESERVED

OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
BENCHMARK_RESULT_CLAIMED=NO
NEXT_SLICE_NOT_SELECTED_BY_THIS_RECORD=YES
```

## Materially inspected publication evidence

The coordinating publication review inspected:

- `guillermomolina/protos@ce2db129102fd69b3bc442b647c8994e7e565d90`;
- its exact parent
  `a7e26228543f1af65a75e2e95bd6e384b67cc7d3`;
- the exact one-commit comparison and per-file statistics;
- the `0.3.154-SNAPSHOT` CHANGELOG and Maven version;
- `CanonicalToBytecodeLowerer.java`;
- `ProtosFrameArguments.java`;
- `ProtosSemanticBytecodeRootNode.java`;
- `ProtosActivation.java`;
- the new `ProtosPerf025CompactCalleeExecutionTest.java`;
- the preceding PERF025-H1 durable evidence;
- the repository PERF, coordination, reference, and durable-record
  instructions.

This record is evidence only and does not replace live GitHub Issue
coordination.
