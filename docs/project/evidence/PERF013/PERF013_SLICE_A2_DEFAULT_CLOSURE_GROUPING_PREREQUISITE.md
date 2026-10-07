# PERF013 Slice A follow-up — default-parameter Closure grouping prerequisite

Date: 2026-09-27

## Scope

This record supplements, but does not rewrite, the published PERF013 Slice A checkpoint:

`docs/project/evidence/PERF013/PERF013_SLICE_A_SHARED_LEXICAL_ROOT_GROUPING.md`

Slice A correctly grouped ordinary lexically-contained Closure literals under one shared
`BytecodeRootNodes` group, but intentionally retained the historical independent path for
Closures appearing inside parameter default expressions.

A current-HEAD source audit after Slice A establishes that this retained exception can itself
contain statically proven captures and therefore must be removed before PERF013 Slice B adopts
`MaterializedLocalAccessor` as the canonical captured-local mechanism.

## Exact baseline

```text
REPOSITORY=guillermomolina/protos
WORK_ITEM=PERF013/#724

PRODUCT_REVISION=be6f310814bd2768d53e2e21c23fd95fc88d22c4
PRODUCT_VERSION=0.3.98-SNAPSHOT
SLICE_A=COMPLETE
```

## Current Slice-A exception

The published lowerer retains:

```text
validateSupportedDefaultExpression(CanonicalClosure)
  -> independentBytecodeClosurePlan(...)
  -> ProtosClosureExecutionPlan.bytecode(...)
  -> independent CanonicalToBytecodeLowerer
  -> independent ProtosBytecodeRootNodeGen.create(...)
```

The code documents this as the path for a Closure reached from a parameter default value.

Therefore an owner Closure root and a Closure literal materialized by one of that owner's
parameter default expressions can still belong to different `BytecodeRootNodes` groups.

## Why this can contain real captured locals

`CanonicalBindingAnalyzer.walkClosure` establishes the owning Closure scope as follows:

```text
declare all parameter identities

for each parameter in declared order:
  walk(parameter.defaultValue, closureScope)
  markEstablished(parameter.name)

walk body in the same closureScope
```

A Closure nested in a default expression is therefore analyzed with that exact owning
`closureScope` as its enclosing lexical scope.

Consequently a default Closure can capture:

- an earlier already-established parameter of the same invocation;
- a binding from an enclosing lexical context.

Representative shape:

```protos
f: (x, g = () => x) => g()
```

At the point the default for `g` is analyzed/evaluated, `x` is already established. The nested
`() => x` therefore has a genuine captured binding owned by the current `f` invocation.

This is not an optional or guest-invisible edge case: ordinary Closure capture semantics apply to
Closure literals in default expressions.

## Truffle constraint already governing PERF013

The Truffle 25.3.4.1 Bytecode DSL materialized-local mechanism used by PERF013 requires the current
root and the root declaring the accessed local to belong to the same generated
`BytecodeRootNodes` group.

`MaterializedLocalAccessor` represents logical identity by:

```text
rootIndex + localOffset + localIndex
```

and resolves the declaring root through the current root's `getRootNodes()`.

Therefore the Slice-A independent default-Closure path cannot support the selected canonical
materialized-local access when such a default Closure captures from its owner.

## Routing consequence

Do not start captured-local accessor replacement yet.

The next implementation unit is a bounded Slice A follow-up:

```text
NEXT_SLICE=PERF013-A2_DEFAULT_CLOSURE_SHARED_GROUPING
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
```

A2 should make Closure literals reached through parameter default-expression emission use the same
shared-root grouping mechanism as ordinary body Closure literals.

The intended result is:

```text
DEFAULT_CLOSURE_INDEPENDENT_CREATE=NO
DEFAULT_CLOSURE_CAPTURE_OWNER_GROUP=PASS
MULTI_DEPTH_DEFAULT_CLOSURE_GROUPING=PASS
```

The existing Slice-A execution-plan cell seam should be reused rather than introducing another
plan indirection.

After A2 is published and validated, PERF013 Slice B can safely adopt
`MaterializedLocalAccessor` without retaining a known captured-Closure topology exception.

## Issue granularity

A2 is a bounded completion slice inside PERF013 / #724.

It does not have independent scheduling, decision ownership, or closure from PERF013 and therefore
does not require a new GitHub Issue under the current Issue-vs-slice boundary.

```text
NEW_ISSUE_REQUIRED=NO
NEW_DECISION_REQUIRED=NO
PERF013_STATUS=IN_PROGRESS
```
