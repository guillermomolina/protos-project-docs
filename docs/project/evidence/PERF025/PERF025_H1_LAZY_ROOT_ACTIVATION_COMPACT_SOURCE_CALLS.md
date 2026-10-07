# PERF025-H1 — Lazy root Activation for compact source calls

## Status

Published product implementation evidence for PERF025 / `guillermomolina/protos#758`.

This record is non-normative. It retains the exact product publication identity,
the structural implementation result, and the maintainer-reported validation
outcome for PERF025-H1.

## Exact product publication

```text
PROTOS_REVISION=a7e26228543f1af65a75e2e95bd6e384b67cc7d3
PROTOS_PARENT_REVISION=9ecb5c7d0a7ccc47f3b7e9d0f16e2ecc84e95618
PROTOS_VERSION=0.3.153-SNAPSHOT
COMMIT_SUBJECT=PERF025-H1: lazy root Activation for compact source calls
OWNING_ISSUE=guillermomolina/protos#758
BASE_IS_EXACT_PARENT=YES
```

The exact parent is the preceding PERF025 lazy shared lexical-environment
Closure-capture publication. PERF025-H1 is therefore the next product
publication in that line.

## Trigger and bounded objective

The preceding direct-source-Closure slice established a compact frame ABI across
the source `CallTarget` boundary and deferred guest Context and complete
supplied-argument Array materialization. It intentionally retained a root
prologue that immediately reconstructed the rich `ProtosActivation`.

PERF025-H1 consumes that residual re-inflation point.

The bounded objective is:

```text
compact source call
  -> source target
  -> ordinary parameter/body execution from compact frame arguments + frame locals
  -> rich ProtosActivation only when an operation actually requires it
```

No observable Protos semantic change, specification change, ReturnHome
redesign, frame-materialization redesign, or Closure-capture redesign is part
of this slice.

## Published structural change

The product changelog records that semantic source roots no longer emit the
unconditional `PublishFrameActivation` prologue.

The root's current activation is now obtained through a lazy
`CurrentActivation` operation. When a rich activation is actually required,
the existing compact frame ABI is projected into the exact
`ProtosActivation`, that object is published into frame argument 0, and later
observers reuse the same object.

Thus:

```text
COMPACT_SOURCE_ROOT_UNCONDITIONAL_ACTIVATION_MATERIALIZATION=NO
ON_DEMAND_ACTIVATION_PROJECTION=YES
ACTIVATION_PUBLISHED_IN_FRAME_ARGUMENT_0=YES
ACTIVATION_MATERIALIZED_AT_MOST_ONCE=YES
```

The published implementation does not replace `ProtosActivation` with another
universal per-invocation carrier.

## Compact supplied-argument path

At root level, supplied-argument handling now has frame-native operations:

```text
HasFrameClosureArgument
LoadFrameClosureArgument
CheckFrameClosureArgumentUpperBound
```

These operations consume the compact source-call frame arguments directly
instead of first materializing `ProtosActivation`.

The lowering also uses frame-native parameter binding through
`BindClosureFrameParameter` where the current root's binding is statically
known, and PRESENT current-root reads can use `ReadRootFrameLocal` directly.

Consequently ordinary zero-argument, simple supplied-argument and default-
argument source Closure calls can remain physically compact through their
ordinary parameter/body path.

```text
ZERO_ARG_ORDINARY_PATH_REQUIRES_RICH_ACTIVATION=NO
SINGLE_ARG_ORDINARY_PATH_REQUIRES_RICH_ACTIVATION=NO
DEFAULT_ARGUMENT_ORDINARY_PATH_REQUIRES_RICH_ACTIVATION=NO
ORDINARY_ARGUMENT_ACCESS_REQUIRES_RICH_ACTIVATION=NO
```

The generic/rich path remains available where required.

## Lazy materialization seams

A rich activation is still materialized when semantics or runtime facilities
need the general representation.

The published delta includes lazy projection for cases including:

- an operation lowered through `CurrentActivation`;
- root Error/exception interception that needs activation state;
- debugger/tooling scope projection;
- other existing generic/rich paths.

Error-path materialization is intentionally allowed: avoiding a rich object on
the successful ordinary path does not require weakening Error construction or
control-transfer semantics.

The implementation reuses the exact materialization authority already present
in `ProtosFrameArguments.activation(...)`, so one invocation does not gain a
second activation identity.

## Context semantics

PERF025-H1 preserves the previously established lazy guest Context projection.

The dedicated regression verifies both:

```text
FRESH_CONTEXT_PER_INVOCATION=PASS
SAME_CONTEXT_WITHIN_INVOCATION=PASS
```

The ordinary compact path therefore does not create a guest execution Context
merely because a source root was entered, while later observation retains the
fresh logical Context semantics of each invocation.

```text
EAGER_GUEST_CONTEXT=NO
FRESH_LOGICAL_CONTEXT=PRESERVED
```

## Receiver, method-home, lexical capture and ReturnHome

The rich activation remains an exact lazy projection of the existing invocation
provenance.

The product changelog records unchanged Context, Task, receiver, method-home,
capture and return-home provenance.

The focal regression explicitly covers capture-by-reference, `this`, owned and
nested non-local return, and escaped-return failure behavior.

```text
CAPTURE_BY_REFERENCE=PRESERVED
THIS_SEMANTICS=PRESERVED
METHOD_HOME_PROVENANCE=PRESERVED
NON_LOCAL_RETURN=PRESERVED
INVALID_RETURN_BEHAVIOR=PRESERVED
RETURN_HOME_REPRESENTATION_CHANGED=NO
```

PERF025-H1 does not make `ProtosReturnHome` itself lazy.

## Task and generic fallback

The Task-owned compact source path remains compact and retains the exact owning
Task in the compact frame ABI. When a rich activation is later requested, its
Task provenance matches that exact Task.

The dedicated regression also enters a source Closure through a pre-existing
rich activation and verifies that the generic path remains valid.

```text
TASK_PROVENANCE=PRESERVED
TASK_OWNED_COMPACT_ENTRY=PASS
GENERIC_RICH_ENTRY=PASS
GENERIC_FALLBACK=PRESERVED
```

## Frame-materialization and Closure-capture boundaries

Two adjacent PERF025 interventions remain independent from this exact slice.

The preceding product revisions already changed conditional frame
materialization and Closure lexical capture. PERF025-H1 does not redesign
either mechanism:

```text
FRAME_MATERIALIZATION_CHANGED_BY_H1=NO
CLOSURE_CAPTURE_BEHAVIOR_CHANGED_BY_H1=NO
RETURN_HOME_REPRESENTATION_CHANGED_BY_H1=NO
```

This record therefore does not attribute those earlier structural changes to
H1.

## Exact product delta

The published commit changes 11 paths:

```text
M CHANGELOG.md
M pom.xml
M src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
M src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
M src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java
M src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
M src/test/java/com/guillermomolina/protos/execution/ProtosI068Slice7ActivationLexicalDecompositionTest.java
M src/test/java/com/guillermomolina/protos/execution/ProtosPerf025DirectSourceClosureCompactInvocationTest.java
M src/test/java/com/guillermomolina/protos/execution/ProtosPerf025FrameMaterializationSliceTest.java
A src/test/java/com/guillermomolina/protos/execution/ProtosPerf025H1LazyRootActivationTest.java
M src/test/java/com/guillermomolina/protos/execution/ProtosPerf025LazyLexicalCaptureTest.java

FILES_CHANGED=11
ADDITIONS=637
DELETIONS=43
```

Version publication:

```text
IMPLEMENTATION_VERSION=0.3.153-SNAPSHOT
ROOT_CHANGELOG_ENTRY=YES
SPECIFICATION_CHANGE=NO
```

## Dedicated structural regression evidence

The product publication adds:

```text
src/test/java/com/guillermomolina/protos/execution/
  ProtosPerf025H1LazyRootActivationTest.java
```

The test structurally observes compact-vs-materialized state through frame
argument 0 and publishes coverage for:

```text
ZERO_ARG_DIRECT_SOURCE_CALL_WITHOUT_RICH_ACTIVATION=PASS
SINGLE_ARG_DIRECT_SOURCE_CALL_WITHOUT_RICH_ACTIVATION=PASS
DEFAULT_PARAMETER_COMPACT_PATH=PASS
REPEATED_COMPACT_INVOCATIONS=PASS

ON_DEMAND_ACTIVATION_MATERIALIZATION=PASS
ACTIVATION_MATERIALIZED_AT_MOST_ONCE=PASS

FRESH_CONTEXT_PER_INVOCATION=PASS
SAME_CONTEXT_WITHIN_INVOCATION=PASS

CAPTURE_BY_REFERENCE=PASS
THIS_SEMANTICS=PASS
NON_LOCAL_RETURN=PASS

ARITY_ERROR_PATH=PASS
DEBUGGER_REFLECTION_PROJECTION=PASS
TASK_PROVENANCE=PASS
GENERIC_FALLBACK=PASS
```

The test additionally checks that ordinary compact calls leave the callee
Closure in frame argument 0, while an on-demand materialization replaces that
slot with one reusable `ProtosActivation`.

## Validation provenance

The maintainer reported in the active interaction after product publication:

```text
PERF025-H1: lazy root Activation for compact source calls pushed y test passed
```

The coordinating publication review independently verified:

```text
REMOTE_PUBLICATION=PASS
PRODUCT_REVISION=a7e26228543f1af65a75e2e95bd6e384b67cc7d3
COMMIT_SUBJECT_MATCH=PASS
EXACT_PARENT=9ecb5c7d0a7ccc47f3b7e9d0f16e2ecc84e95618
PRODUCT_VERSION=0.3.153-SNAPSHOT
FILES_CHANGED=11
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

The slice continues the already selected PLAT040 F-prime direction:

```text
stable selected source call
  -> compact frame-argument ABI
  -> ordinary execution remains compact
  -> exact rich state only on semantic/runtime observation
  -> exact generic fallback
```

This is an implementation-local convergence step, not a new platform decision.

```text
NEW_PLAT_DECISION=NO
OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
```

## PERF025 audit consequence

Relative to the retained post-F1 pay-as-you-grow audit and the later
direct-source compact-call publication:

```text
DIRECT_SOURCE_PRETARGET_RICH_ACTIVATION
    -> already consumed by fe078828412aabfc48dc1835bf34dce784307b3e

COMPACT_SOURCE_ROOT_IMMEDIATE_RICH_ACTIVATION_REINFLATION
    -> consumed by PERF025-H1 / a7e26228543f1af65a75e2e95bd6e384b67cc7d3

PARAMETER_BINDING_THROUGH_DYNAMIC_SLOT_AUTHORITY
    -> compact admitted root path further reduced by H1
    -> generic/rich fallback remains

RETURN_HOME_PHYSICALLY_CREATED_WHEN_UNOBSERVED
    -> not addressed by H1

INLINE_CALLBACK_STILL_USES_RICH_ACTIVATION
    -> not addressed by H1

INLINE_CALLBACK_LOSES_FRAME_NATIVE_LEXICAL_PATH
    -> not addressed by H1
```

The exact product commit should be used as authority for the final current code
shape; this record only retains the bounded H1 consequence.

## Performance-claim boundary

No benchmark, timing campaign, JFR profile, or attributable per-call magnitude
is part of this slice or this documentation publication.

```text
BENCHMARK_RUN_FOR_THIS_SLICE=NO
ATTRIBUTABLE_NS_PER_CALL=NOT_MEASURED
END_TO_END_SPEEDUP_PERCENT=NOT_MEASURED
PERFORMANCE_MAGNITUDE_CLAIMED=NO
```

The durable claim is structural only: the ordinary compact source-root path no
longer materializes a rich `ProtosActivation` unconditionally.

## PERF025 status

PERF025-H1 is complete as a bounded implementation slice. PERF025 itself remains
open.

```text
PERF025_H1_STATUS=COMPLETE
PERF025_STATUS=OPEN

COMPACT_SOURCE_ROOT_UNCONDITIONAL_ACTIVATION_MATERIALIZATION=NO
ORDINARY_ZERO_SIMPLE_DEFAULT_PATH_CAN_REMAIN_COMPACT=YES
ON_DEMAND_RICH_ACTIVATION=YES
EXACT_GENERIC_FALLBACK=PRESERVED

OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
BENCHMARK_RESULT_CLAIMED=NO
```

The next PERF025 slice is not selected by this evidence record.

## Materially inspected publication evidence

The coordinating publication review inspected:

- `guillermomolina/protos@a7e26228543f1af65a75e2e95bd6e384b67cc7d3`;
- its exact parent
  `9ecb5c7d0a7ccc47f3b7e9d0f16e2ecc84e95618`;
- the exact commit path/statistics payload;
- the `0.3.153-SNAPSHOT` product CHANGELOG entry;
- the published
  `ProtosPerf025H1LazyRootActivationTest.java` regression source;
- the retained PERF025 direct-source compact/deferred invocation evidence;
- the retained PERF025 conditional Closure frame-materialization evidence;
- the retained PERF025 lazy shared lexical-environment Closure-capture evidence;
- the repository PERF, coordination and durable-record instructions.

This record is evidence only and does not replace live GitHub Issue
coordination.
