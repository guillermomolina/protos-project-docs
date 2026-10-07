# PERF013 Slice A2 — default-parameter Closure shared lexical root grouping

Date: 2026-09-27

## Scope

This record retains the published implementation checkpoint for PERF013 / #724 Slice A2.

Slice A2 removes the last default-parameter-specific independent Closure lowering path that existed after Slice A. It does not change captured-local access yet.

## Publication

```text
REPOSITORY=guillermomolina/protos
WORK_ITEM=PERF013/#724
PARENT=PERF010-B/#722

BASELINE_REVISION=be6f310814bd2768d53e2e21c23fd95fc88d22c4
BASELINE_VERSION=0.3.98-SNAPSHOT

PRODUCT_REVISION=1465c4963c4530339e3293043ab9fd937b61a634
PRODUCT_VERSION=0.3.99-SNAPSHOT
COMMIT_MESSAGE=PERF013 Slice A2: group default-value Closures with lexical owner
```

Published changed paths:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosClosureExecutionPlanCell.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf013SliceA2DefaultClosureSharedLexicalRootGroupingTest.java
```

No normative specification path changed.

## Implemented change

Before A2, default-expression structural validation eagerly called
`independentBytecodeClosurePlan(...)`. That populated the Closure-plan cache before the owner
root's builder opened and permanently forced such Closures into an independent
`BytecodeRootNodes` group.

A2 makes default Closure validation structural-only. Real emission already occurs while the owner
root builder is open, so default-value Closures now naturally use the same Slice-A
`bytecodeClosurePlan(builder, closure)` / `lowerNestedClosureRoot(...)` path as ordinary body
Closures.

The obsolete independent-plan branch was removed from:

- `CanonicalToBytecodeLowerer`;
- `ProtosClosureExecutionPlanCell`.

There is now one coherent pending/freeze plan-cell mechanism for Closure literals reached by the
lowerer.

## Semantic/topology result

```text
PERF013_SLICE_A2=PASS

DEFAULT_CLOSURE_INDEPENDENT_CREATE=NO
DEFAULT_CLOSURE_OWNER_SHARED_GROUP=PASS
DEFAULT_CLOSURE_MULTI_DEPTH_SHARED_GROUP=PASS

DEFAULT_CAPTURE_EARLIER_PARAMETER=PASS
DEFAULT_CAPTURE_BY_REFERENCE=PASS

ORDINARY_SLICE_A_GROUPING=PASS
PERF012_LAYOUT=PASS

CAPTURED_LOCAL_MECHANISM=UNCHANGED

SEMANTIC_CHANGE=NO
SPEC_CHANGE=NO
```

A representative retained test executes the semantic equivalent of:

```protos
f: (x, g = () => x) => g
result: f(42)
result()
```

and proves both result `42` and root-group identity between the owning Closure root and the
default Closure root.

The multi-depth case proves that a Closure nested inside the default Closure remains in the same
group.

## Validation provenance

The maintainer reported the following PASS set after implementation:

```text
mvn clean test-compile=PASS
PERF013_A_A2_FOCAL_TOPOLOGY_TESTS=PASS
PARAMETER_DEFAULT_EXECUTION_TESTS=PASS
ProtosI068Slice5CapturedMaterializedLexicalLoweringTest=PASS
ALL_com.guillermomolina.protos.execution.*Test=PASS

FULL_MAKE_TEST=DEFERRED_TO_PERF013_CLOSURE
REMOTE_CI_PASS=NOT_CLAIMED
```

The intermediate-slice full-suite deferral remains explicit.

## Status

```text
PERF013_SLICE_A=COMPLETE
PERF013_SLICE_A2=COMPLETE
PERF013_STATUS=IN_PROGRESS
```

Captured-local MaterializedLocalAccessor adoption remains unimplemented at this checkpoint.
