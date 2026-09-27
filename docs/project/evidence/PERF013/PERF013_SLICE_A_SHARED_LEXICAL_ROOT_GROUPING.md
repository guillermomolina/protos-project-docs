# PERF013 Slice A — shared lexical BytecodeRootNodes grouping

Date: 2026-09-27

## Scope

This record retains the publication checkpoint for PERF013 / #724 Slice A.

Slice A changes only lowering topology so a lexical owner root and lexically-contained Closures can belong to one shared
`BytecodeRootNodes<ProtosBytecodeRootNode>` group. It intentionally does not yet replace captured-local reads/writes with
`MaterializedLocalAccessor`; that remains PERF013 Slice B.

## Publication

```text
REPOSITORY=guillermomolina/protos
PARENT_WORK_ITEM=PERF010-B/#722
WORK_ITEM=PERF013/#724
SLICE=PERF013-A_SHARED_LEXICAL_ROOT_GROUPING

BASELINE_REVISION=a85e66292c4f0314b91c07c1fec3cce058be752c
BASELINE_VERSION=0.3.97-SNAPSHOT

PRODUCT_REVISION=be6f310814bd2768d53e2e21c23fd95fc88d22c4
PRODUCT_VERSION=0.3.98-SNAPSHOT
COMMIT_MESSAGE=PERF013 Slice A: shared lexical BytecodeRootNodes grouping
```

Published changed paths:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeClosureExecutionPlan.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosClosureExecutionPlan.java
src/main/java/com/guillermomolina/protos/execution/ProtosClosureExecutionPlanCell.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf013SliceASharedLexicalRootGroupingTest.java
```

No normative specification path changed.

## Implemented topology

Before this slice, a Closure encountered while lowering another root executed:

```text
emitExpression(CanonicalClosure)
  -> bytecodeClosurePlan(...)
  -> ProtosClosureExecutionPlan.bytecode(...)
  -> new CanonicalToBytecodeLowerer(...)
  -> lowerClosureActivationRoot(...)
  -> ProtosBytecodeRootNodeGen.create(...)
```

so every nested Closure belonged to an independent `BytecodeRootNodes` group.

Slice A changes that shape to one shared `ProtosBytecodeRootNodeGen.create(...)` invocation for a lexical owner and the Closures
lexically encountered while that owner's builder is open. Nested Closures recursively emit their own `beginRoot()/endRoot()`
pairs into the same builder, so arbitrary lexical depth remains in the same group.

```text
owner root
  -> child Closure root
     -> grandchild Closure root
        -> ...
```

Each root still has its own:

- `CanonicalBindingAnalysis` / lexical scope state;
- `BytecodeLocal` set;
- frame descriptor;
- PERF012 `ProtosFrameLexicalLayout`.

The shared entity is only the enclosing `BytecodeRootNodes` group.

## Execution-plan ordering seam

A nested root's `RootNode.getCallTarget()` cannot be used while the group's `create()` parse is still in progress, but
`MaterializeClosure` bytecode must be emitted before the group closes.

The published implementation introduces backend-private `ProtosClosureExecutionPlanCell`.

Its lifecycle is:

```text
while builder/create is open:
  emit the cell as the constant-pool payload
  emit/nest the Closure root

after create() returns:
  build the real ProtosClosureExecutionPlan from the already-created root
  freeze the cell exactly once with that plan

before guest execution:
  cell is frozen
```

The cell uses a `@CompilationFinal` plan field, rejects a second freeze, and fails if read before freeze.

It is not guest-visible and carries no lexical binding value/presence authority.

Default-parameter-value Closures retain their previous independent lowering path in Slice A because they are materialized before an
enclosing shared builder is open. That path is explicitly not claimed as converted by this slice.

## Preserved boundaries

The published delta preserves the intended boundaries:

```text
PERF012_LAYOUT_PRECOMPUTED_PER_ROOT=YES
INDEPENDENT_CREATE_PER_NORMAL_NESTED_CLOSURE=NO
MULTI_DEPTH_SHARED_ROOT_GROUPING=YES

OBJECT_BODY_IS_GENUINE_LEXICAL_OWNER=NO
OBJECT_BODY_GROUPING_BOUNDARY=PRESERVED

SEMANTIC_ROOT_TAG_BOUNDARY=PRESERVED
HELPER_ROOTS_REMAIN_BACKEND_PRIVATE=YES

CAPTURED_LOCAL_READ_WRITE_MECHANISM=UNCHANGED
MATERIALIZED_LOCAL_ACCESSOR_ADOPTION=NOT_YET
SEMANTIC_CHANGE=NO
SPEC_CHANGE=NO
```

## Focal topology coverage

The new focal test
`ProtosPerf013SliceASharedLexicalRootGroupingTest` directly covers:

- owner + direct child sharing one `BytecodeRootNodes`;
- owner + multi-depth nested Closures sharing one group;
- sibling Closures sharing the owner group while retaining distinct roots;
- distinct frame descriptors/local layouts per lexical root;
- object-body grouping boundary;
- execution-plan targeting of the expected group root.

Existing execution-package tests continue to cover Closure/capture/reparse/instrumentation behavior.

## Validation provenance

The maintainer reported the following validation after the published Slice A implementation:

```text
git diff --check=PASS
mvn clean test-compile=PASS
ALL com.guillermomolina.protos.execution.*Test=PASS
STATIC_FORMAT_LINT=NOT_CONFIGURED_BEYOND_JAVAC
LICENSE_COMPLIANCE=PASS
```

The integrated `make test` full-suite gate was intentionally deferred to PERF013 final closure while Slices B/C remain open under
the project's reduced-profile allowance for intermediate publication slices.

```text
FULL_MAKE_TEST=DEFERRED_TO_PERF013_CLOSURE
REMOTE_CI_PASS=NOT_CLAIMED
```

## Slice-A gate

```text
SHARED_LEXICAL_ROOT_GROUPING=PASS
OWNER_AND_CHILD_SAME_BYTECODE_ROOT_NODES=PASS
MULTI_DEPTH_GROUPING=PASS
CLOSURE_PLAN_USES_PREBUILT_GROUP_ROOT=PASS
INDEPENDENT_CREATE_PER_NESTED_CLOSURE=NO
PERF012_LAYOUT=PASS
ROOTTAG_SEMANTICS_UNCHANGED=PASS
OBJECT_BODY_SEMANTICS_UNCHANGED=PASS
SEMANTIC_CHANGE=NO

PERF013_SLICE_A=COMPLETE
PERF013_STATUS=OPEN
```

## Next slice

```text
NEXT_SLICE=PERF013-B_MATERIALIZED_CAPTURED_LOCAL_ACCESS
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
START_FROM_HEAD=YES
```

Slice B should now enable the Bytecode DSL materialized-local facility and replace the PE-hostile captured owner
`LocalRangeAccessor` / physical `BytecodeNode` recovery path with constant `MaterializedLocalAccessor` identity plus the dynamic
owner `MaterializedFrame`, while preserving D179 C0 / I071 presence and exact fallback semantics.

Context-local whole-group rematerialization remains PERF013 Slice C unless Slice B discovers a strictly necessary mechanical seam.
