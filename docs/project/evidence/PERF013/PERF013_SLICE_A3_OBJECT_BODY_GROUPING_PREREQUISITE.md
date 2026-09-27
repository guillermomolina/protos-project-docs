# PERF013 Slice A3 prerequisite — object-body Closure lexical-owner grouping

Date: 2026-09-27

## Scope

This record supplements the published PERF013 Slice A/A2 topology work and identifies one remaining
shared-root prerequisite before canonical MaterializedLocalAccessor adoption.

It is derived entirely from current Protos HEAD behavior and existing Protos tests/semantics.

## Exact baseline

```text
REPOSITORY=guillermomolina/protos
WORK_ITEM=PERF013/#724

PRODUCT_REVISION=1465c4963c4530339e3293043ab9fd937b61a634
PRODUCT_VERSION=0.3.99-SNAPSHOT

PERF013_SLICE_A=COMPLETE
PERF013_SLICE_A2=COMPLETE
```

## Existing semantic boundary

Object construction is not a genuine lexical execution-context scope.

Current runtime behavior explicitly preserves:

```text
Closure created during object construction
  -> DOES NOT capture constructed object lexically
  -> DOES capture enclosing lexical execution context(s) by reference
  -> DOES use constructed object as captured receiver
```

Existing tests retain this contract:

- `CanonicalClosureMaterializationTest.closureCreatedDuringObjectConstructionSkipsConstructionObject`;
- `CanonicalObjectExecutionTest.closureDeclaredInObjectBodyDoesNotCaptureConstructedObjectLexically`.

The constructed object is ordinary receiver/object state, not a captured lexical context.

## Current physical topology

The current lowerer still materializes an object-construction helper target through:

```text
bytecodeObjectBodyTarget(object)
  -> lowerObjectBodyRoot(object.body())
  -> lowerRoot(..., genuineExecutionContextRoot=false)
  -> independent ProtosBytecodeRootNodeGen.create(...)
```

Closures encountered while that object-body helper root's builder is open use the Slice-A shared
Closure mechanism relative to that helper root.

Therefore the physical topology is currently:

```text
lexical owner group A:
  owner lexical root

object-body group B:
  object-construction helper root
  Closure declared inside object body
```

while semantic capture for that Closure can be:

```text
Closure in group B
  -> enclosing lexical execution context owned by root in group A
```

## Why this blocks canonical MaterializedLocalAccessor

The Truffle 25.3.4.1 MaterializedLocalAccessor selected by PERF013 identifies a local using:

```text
rootIndex + localOffset + localIndex
```

and resolves the declaring root through the current root's own `BytecodeRootNodes`.

Therefore a Closure root in group B cannot use a MaterializedLocalAccessor for a frame-backed local
declared by a lexical owner root in group A.

This is structurally the same kind of prerequisite Slice A/A2 solved for ordinary/default
Closures, but object-body helper roots must remain semantically non-lexical.

## Required distinction

Physical root grouping must not be confused with semantic lexical ownership.

The intended target is:

```text
one BytecodeRootNodes group:
  genuine lexical owner root
  object-construction helper root   [genuineExecutionContextRoot=false]
  Closure declared in object body   [genuine lexical Closure root]
```

while semantics remain:

```text
OBJECT_BODY_IS_LEXICAL_CAPTURE_SCOPE=NO
CONSTRUCTED_OBJECT_IS_CAPTURED_LEXICAL_CONTEXT=NO
CONSTRUCTED_OBJECT_IS_CAPTURED_RECEIVER=YES
OUTER_LEXICAL_CONTEXT_CAPTURE_BY_REFERENCE=YES
```

Grouping the helper root with the lexical owner is backend topology only.

## Expected implementation seam

The current eager `bytecodeObjectBodyTarget(object)` path requires a real `RootCallTarget` before
runtime object construction can be emitted.

Like Slice A's Closure-plan ordering conflict, a helper root emitted inside an open shared builder
cannot expose its call target until the enclosing `create()` returns.

A3 should therefore reuse the already-proven immutable-once-frozen backend pattern:

```text
while shared builder is open:
  emit/register object-body helper root in same group
  retain backend-private target cell/descriptor

after create() returns:
  obtain real helper-root RootCallTarget
  freeze target cell exactly once

before guest execution:
  cell is resolved
```

The exact Java type/name should be derived from current HEAD. No guest-visible placeholder or second
semantic authority is permitted.

Validation/reparse must replay the object-body root in the shared group so root indices remain
stable.

## Routing

```text
NEXT_SLICE=PERF013-A3_OBJECT_BODY_HELPER_SHARED_GROUPING
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos

NEW_ISSUE_REQUIRED=NO
NEW_DECISION_REQUIRED=NO
PERF013_STATUS=IN_PROGRESS
```

After A3 closes this final known group split, PERF013 Slice B can adopt MaterializedLocalAccessor for
proven frame-backed captured locals without knowingly retaining a lexical-owner group mismatch.

## Required A3 gate

```text
OBJECT_BODY_HELPER_SHARED_GROUP=PASS

OBJECT_BODY_IS_LEXICAL_CAPTURE_SCOPE=NO
CONSTRUCTED_OBJECT_CAPTURED_LEXICALLY=NO
CONSTRUCTED_OBJECT_CAPTURED_RECEIVER=YES
OUTER_LEXICAL_CAPTURE_BY_REFERENCE=PASS

CLOSURE_IN_OBJECT_BODY_CAN_SHARE_GROUP_WITH_OUTER_LEXICAL_OWNER=PASS
MULTI_DEPTH_OBJECT_BODY_CLOSURE_GROUPING=PASS

PERF013_A_A2_GROUPING=PASS
PERF012_LAYOUT=PASS

CAPTURED_LOCAL_MECHANISM=UNCHANGED
SEMANTIC_CHANGE=NO
NEW_DECISION=NO
```
