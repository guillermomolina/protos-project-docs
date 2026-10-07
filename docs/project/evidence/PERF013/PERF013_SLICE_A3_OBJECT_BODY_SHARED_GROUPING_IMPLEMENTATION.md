# PERF013 Slice A3 — object-body helper shared grouping implementation

Date: 2026-09-27

## Scope

This record retains the published implementation checkpoint for PERF013 / #724 Slice A3.

Slice A3 completes the physical root-group topology prerequisite for canonical captured-local materialized access. It does not yet change the captured-local read/write mechanism.

## Publication

```text
REPOSITORY=guillermomolina/protos
WORK_ITEM=PERF013/#724
PARENT=PERF010-B/#722

BASELINE_REVISION=1465c4963c4530339e3293043ab9fd937b61a634
BASELINE_VERSION=0.3.99-SNAPSHOT

PRODUCT_REVISION=24b41e8e03b60882f05e4a1a2105b8dbd9a4f21b
PRODUCT_VERSION=0.3.100-SNAPSHOT
COMMIT_MESSAGE=PERF013 Slice A3: group object-body helpers with lexical owner
```

Published changed paths:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosObjectBodyTargetCell.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf013SliceASharedLexicalRootGroupingTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf013SliceA3ObjectBodyHelperSharedGroupingTest.java
```

No normative specification path changed.

## Implemented topology

Before A3, object bodies were lowered by an independent:

```text
bytecodeObjectBodyTarget(object)
  -> lowerObjectBodyRoot(...)
  -> ProtosBytecodeRootNodeGen.create(...)
```

A3 removes that path.

Object-body helpers are now emitted with nested `beginRoot()/endRoot()` inside the same open
`ProtosBytecodeRootNodeGen.create(...)` invocation as the enclosing lexical unit.

```text
one BytecodeRootNodes group:
  genuine lexical owner root
  object-construction helper root
  Closure inside object body
  deeper Closure(s)
```

The helper remains lowered with:

```text
genuineExecutionContextRoot=false
```

and therefore remains semantically non-lexical and receives no frame-backed lexical authority/layout.

## Target ordering seam

The published implementation introduces backend-private `ProtosObjectBodyTargetCell`, mirroring the
existing Closure plan cell.

During group construction:

```text
emit object-body helper root
emit target cell as constant payload
```

After `create()` returns:

```text
helper root -> real RootCallTarget -> freeze cell exactly once
```

The cell is backend-private, `@CompilationFinal`, rejects double freeze, and rejects reads before
freeze.

The previous independent helper target cache/path is removed.

## Structural validation

`CanonicalObject` validation is now structural/preparation-only:

- validates parent expression;
- registers `CanonicalCompose` reserved-slot metadata;
- validates object-body structure;
- does not construct a helper root.

Real helper-root construction happens only during emission while the owning builder is open.

## Preserved semantics

```text
OBJECT_BODY_INDEPENDENT_CREATE=NO
OBJECT_BODY_HELPER_SHARED_GROUP=PASS

OBJECT_BODY_IS_GENUINE_EXECUTION_CONTEXT=NO
OBJECT_BODY_IS_LEXICAL_CAPTURE_SCOPE=NO

CONSTRUCTED_OBJECT_CAPTURED_LEXICALLY=NO
CONSTRUCTED_OBJECT_CAPTURED_RECEIVER=YES
OUTER_LEXICAL_CAPTURE_BY_REFERENCE=PASS

OBJECT_SLOT_STORAGE_UNCHANGED=PASS
PARENT_EVALUATION_ORDER=PASS
OBJECT_CONSTRUCTION_SUSPENSION_PROTOCOL=PASS

PERF013_A_A2_GROUPING=PASS
PERF012_LAYOUT=PASS

CAPTURED_LOCAL_MECHANISM=UNCHANGED
SEMANTIC_CHANGE=NO
SPEC_CHANGE=NO
```

## Validation provenance

The maintainer reported:

```text
git diff --check=PASS
mvn -q -o clean test-compile=PASS
PERF013_A_A2_A3_FOCAL_TOPOLOGY_TESTS=PASS
CanonicalObjectExecutionTest=PASS
ProtosI068Slice5CapturedMaterializedLexicalLoweringTest=PASS
ALL_com.guillermomolina.protos.execution.*Test=PASS

FULL_MAKE_TEST=DEFERRED_TO_PERF013_CLOSURE
REMOTE_CI_PASS=NOT_CLAIMED
```

The full integrated gate remains an explicit closure obligation for PERF013.

## Root-group completeness checkpoint

A current-HEAD search after A3 finds exactly one `ProtosBytecodeRootNodeGen.create(...)` inside
`CanonicalToBytecodeLowerer`.

Other production `create()` sites belong to independent C-prime driver implementations
(task/text/IO/buffered-IO roots), not to the canonical lexical Closure/object-body lowering hierarchy.

Therefore no additional known Closure/object-helper root split remains before materialized captured-local access.

```text
KNOWN_LEXICAL_ROOT_GROUPING_EXCEPTIONS=0
PERF013_TOPOLOGY_PREREQUISITE=COMPLETE
```

## Correction: lexicalDepth already skips OBJECT_BODY

During A3 planning, a possible mismatch was raised between canonical `lexicalDepth` and runtime
`capturedLexicalContexts()` when an `OBJECT_BODY` lies physically between a Closure and its
lexical owner.

Current HEAD shows that no extra translation is required for that reason.

`CanonicalLexicalScope.outwardScope()` explicitly skips every intervening
`CanonicalLexicalScope.Kind.OBJECT_BODY` before returning the next outward scope.

`CanonicalBindingAnalyzer.resolve(...)` increments depth only once per returned `outwardScope()`.

Therefore a Closure inside an object body whose next genuine lexical owner is one level outward is
classified with:

```text
lexicalDepth=1
ownerIndex=lexicalDepth-1=0
```

which matches the runtime capture list, because object construction likewise omits the constructed
object from `lexicalContextsForClosureCapture()`.

```text
OBJECT_BODY_DEPTH_TRANSLATION_REQUIRED=NO
LEXICAL_DEPTH_TO_CAPTURED_CONTEXT_INDEX=lexicalDepth-1
```

This corrects the earlier planning caution; no semantic change is involved.

## Next work

The topology prerequisite is now complete. The next bounded implementation should begin actual
MaterializedLocalAccessor adoption.

To keep mutation ordering and compiler attribution independently reviewable, route the direct
captured mechanism as two implementation slices:

```text
NEXT_SLICE=PERF013-B1_MATERIALIZED_CAPTURED_READ
FOLLOWING_SLICE=PERF013-B2_MATERIALIZED_CAPTURED_WRITE
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
```

B1 should:

- enable Bytecode DSL materialized-local accesses;
- express the proven captured owner local as a `MaterializedLocalAccessor` constant operand sourced
  from the owner `BytecodeLocal`;
- keep only the owner `MaterializedFrame` dynamic;
- preserve D179 C0 presence/nearer-context fallback;
- retain safe generic fallback for any runtime topology/group mismatch;
- remove the Step-0 captured-read PE-constant failure on the source-equivalent read root.

B2 will then migrate the write-destination path while preserving destination-before-RHS semantics.

Context-local whole-group rematerialization remains later PERF013 work if still required after B1/B2.
