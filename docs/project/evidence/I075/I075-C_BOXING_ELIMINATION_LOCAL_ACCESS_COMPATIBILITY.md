# I075-C — Bytecode DSL boxing-elimination local-access compatibility investigation

Status: **COMPLETE**

This durable, non-normative record retains the read-only investigation result for
I075-C / guillermomolina/protos#746.

## Identity

```text
DATE=2026-09-30
WORK_ITEM=I075-C
PARENT_WORK_ITEM=I075
GITHUB_ISSUE=guillermomolina/protos#746
PARENT=PERF011 / guillermomolina/protos#693

TYPE=INVESTIGATION_ONLY

PROTOS_REVISION=a89a8897ea20b10344785ee9573ed329d188ef28
PROTOS_VERSION=0.3.124-SNAPSHOT
GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1
UPSTREAM_TAG=vm-25.4.4.1.1
UPSTREAM_REVISION=95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b
I075_B_STOP_GATE_RECORD_REVISION=0b0de32529d24df0b4b294d835cc9f2170525f9f
```

The Protos product baseline remained exactly the published I074 revision.
I075-C executed no builds, tests, benchmarks, profilers, patches, product
repository mutations, commits or pushes.

## Question

I075-B established that enabling:

```java
boxingEliminationTypes = {int.class}
```

passed annotation processing/build but failed integrated validation through an
existing lexical-local access path:

```text
ReadFrameLocal.perform
  -> LocalAccessor.getObject
  -> FrameSlotTypeException
```

The failing execution was inside a resumed continuation and observed a cached
local tag in the `ILLEGAL` state.

I075-C had to distinguish:

1. a bounded API-level compatibility adjustment;
2. a broader lexical-local authority migration;
3. an upstream/runtime limitation or unsupported combination;
4. leaving boxing elimination disabled.

## Decisive upstream contract

The exact Graal/Truffle 25.4.4.1.1 source and Bytecode DSL guide establish that
a `BytecodeNode` is not a permanent root identity.

It may change while a program executes, including when:

- transitioning from uncached to cached execution;
- reparsing metadata;
- quickening.

The Bytecode DSL guide explicitly instructs runtime code to obtain the
up-to-date node through `BytecodeRootNode#getBytecodeNode()` whenever it is
needed. Custom operations may instead receive the current node through
`@Bind BytecodeNode`.

`LocalAccessor` / `LocalRangeAccessor` operations are generated against the
Bytecode-node state that owns the current cached-local metadata. The node passed
to those accessors must therefore correspond to the current declaring
Bytecode node, not a historical tier-specific node retained across replacement.

Relevant upstream authority inspected in the pinned tag includes:

```text
truffle/docs/bytecode_dsl/UserGuide.md
truffle/src/com.oracle.truffle.api.bytecode/.../BytecodeRootNode.java
truffle/src/com.oracle.truffle.api.bytecode/.../BytecodeNode.java
truffle/src/com.oracle.truffle.api.bytecode/.../LocalAccessor.java
truffle/src/com.oracle.truffle.api.bytecode/.../LocalRangeAccessor.java
truffle/src/com.oracle.truffle.api.bytecode/.../MaterializedLocalAccessor.java
truffle/src/com.oracle.truffle.dsl.processor/.../BytecodeNodeElement.java
truffle/src/com.oracle.truffle.dsl.processor/.../ContinuationRootNodeImplElement.java
truffle/src/com.oracle.truffle.api.bytecode.test/.../UncachedYieldTest.java
truffle/src/com.oracle.truffle.api.bytecode.test/.../BoxingEliminationTypeSystemTest.java
```

## Protos mismatch

Current Protos installs its frame-backed lexical authority in
`ProtosBytecodeRootNode.InstallFrameLexicalAuthority`.

The authority is constructed with:

- the stable lexical layout;
- the `LocalRangeAccessor`;
- the `BytecodeNode` bound at installation time;
- the retained/materialized frame.

`ProtosFrameLexicalBindingAuthority` then retains that installation-time
`BytecodeNode` as a field and later reuses it for frame-backed:

- presence checks;
- reads;
- writes;
- clears/removals;
- snapshots/reflection projection.

That is safe only while the root continues to use the same Bytecode node.

With the uncached interpreter enabled, a root can later replace the uncached
node with a cached node. The retained authority can then continue mutating the
same frame through the old node even though later current-scope operations are
executing with the new cached node.

## Failure mechanism established by source

With boxing elimination disabled, the stale-node defect does not require
per-node cached local-kind state and therefore remained latent.

With boxing elimination enabled, the cached node maintains local-tag
specialization metadata in addition to the physical tag/value stored in the
frame.

The bounded failure sequence is:

```text
1. root executes with uncached BytecodeNode U
2. InstallFrameLexicalAuthority retains U
3. a statically allocated lexical local is still CLEARED
4. execution suspends / later resumes
5. root transitions to cached BytecodeNode C
6. C initializes that local's cached tag as ILLEGAL because the frame local
   is still cleared at transition time
7. Protos later creates/assigns the binding through
   ProtosFrameLexicalBindingAuthority
8. that authority still calls LocalRangeAccessor with stale node U
9. the physical frame slot becomes Object/PRESENT
10. C's cached local tag remains ILLEGAL
11. a normal current operation receives C through @Bind("$bytecodeNode")
12. isCleared(C, frame) is false because the physical frame slot is PRESENT
13. getObject(C, frame) consults C's incompatible cached-local state
14. FrameSlotTypeException is raised
```

This explains the I075-B observation:

```text
physical frame state = PRESENT/Object
current cached-node local tag = ILLEGAL
```

without requiring a continuation implementation defect.

## Continuation / uncached-cached finding

The original I075-B continuation hypothesis is not retained as root cause.

Upstream 25.4.4.1.1 contains explicit continuation-local reconciliation for
boxing elimination and an `UncachedYieldTest` regression whose purpose is to
validate cached-tag updates when a continuation transitions from uncached to
cached.

That test combines:

```text
enableYield=true
enableUncachedInterpreter=true
boxingEliminationTypes={int.class}
LocalAccessor reads/writes
```

Therefore the combination itself is supported upstream.

Continuation/resumption is relevant because it makes it natural for the same
Protos activation, frame and retained lexical authority to outlive the
Bytecode-node replacement. It exposes the stale-node misuse; it is not the
established source of the bad tag state.

```text
CONTINUATION_TAG_HYPOTHESIS=REJECTED

UNCACHED_CACHED_TAG_HYPOTHESIS=
  PARTIALLY_SUPPORTED_AS_TRIGGER
  NOT_SUPPORTED_AS_UPSTREAM_RECONCILIATION_BUG
```

## LocalAccessor and built-in local operations

Upstream tests also establish that boxing elimination is not generally
incompatible with custom `LocalAccessor` operations.

`BoxingEliminationTypeSystemTest.testCustomLocals` writes a local through
custom `LocalAccessor.setInt/setLong/setObject` operations and then reads that
same local through generated `LoadLocal`.

Thus no broad rule that accessor-based locals and built-in locals must never
coexist is established by 25.4.4.1.1.

The required rule for the Protos failure is narrower:

```text
ACCESSOR_BYTECODE_NODE=MUST_BE_CURRENT
```

## Presence / D179

The physical presence mechanism selected by the current PLAT036/D179
implementation remains valid.

`LocalAccessor.isCleared` / `LocalRangeAccessor.isCleared` inspect the
physical frame-slot state. A local can therefore be:

```text
ABSENT   = frame slot cleared / Illegal
PRESENT  = frame slot initialized
```

and `PRESENT(null)` remains distinct from ABSENT because Protos guest null is
not represented by host null.

The I075-B observation where:

```text
isCleared == false
getObject -> FrameSlotTypeException
```

is consistent: `isCleared` observes the physical frame tag, while
boxing-elimination reads also depend on current-node cached local-tag metadata.

No presence bitmap, sentinel, second value store or D179 change is required.

## Materialized captured locals

The direct PERF013 materialized-local path already follows the required
current-node discipline.

`ReadCapturedMaterializedLocal`,
`ResolveCapturedMaterializedWritableLexicalTarget` and
`AssignCapturedMaterializedLocal` receive the current caller Bytecode node via
`@Bind("$bytecodeNode")`.

Upstream `MaterializedLocalAccessor` then resolves through the root group to
the current Bytecode node of the declaring root before performing the local
operation.

Those paths therefore do not require a representation redesign.

The older/runtime-authority captured fallback does inherit the stale-node defect
because it ultimately reaches the retained `ProtosFrameLexicalBindingAuthority`.

## Generic BytecodeNode local access

Replacing `LocalAccessor.getObject` with
`BytecodeNode.getLocalValue(...)` is not selected.

The generic helper is intended for uncommon direct frame access and requires a
valid current bytecode index/local offset. More importantly, replacing only
the read would leave stale-node writes intact and would merely mask one observed
exception rather than restoring coherent current-node local metadata.

```text
GENERIC_LOCAL_ACCESS_WORKAROUND=REJECTED
READ_ONLY_REPAIR=INVALID
```

## Smallest correct implementation seam

The semantic/storage authority selected by PLAT036 can remain unchanged.

The incorrect implementation detail is permanent retention of a
tier-specific `BytecodeNode`.

The bounded correction is:

```text
OLD:
  ProtosFrameLexicalBindingAuthority
    retains installation-time BytecodeNode forever

NEW:
  ProtosFrameLexicalBindingAuthority
    retains stable declaring BytecodeRootNode identity
    and obtains declaringRoot.getBytecodeNode()
    when performing frame-backed accessor operations
```

The same current declaring node must be used consistently for:

- `isCleared`;
- `getObject`;
- `setObject`;
- `clear`;
- frame-backed snapshot/reflection reads using the authority.

This is an API-lifetime compatibility repair, not a new lexical representation.

## Architecture impact

The bounded repair preserves all currently selected architecture:

```text
LEXICAL_BINDING_AUTHORITY=UNCHANGED
FRAME_BACKED_STATIC_VALUE_AUTHORITY=UNCHANGED
EXECUTION_CONTEXT_MEMBERSHIP=UNCHANGED
CAPTURE_OWNERSHIP=UNCHANGED
MATERIALIZATION_LIFETIME=UNCHANGED
CONTINUATION_STATE_MODEL=UNCHANGED
DEBUGGER_REFLECTION_AUTHORITY=UNCHANGED
SECOND_LOCAL_STORAGE_AUTHORITY=NOT_INTRODUCED

PLAT036_DELTA=NONE
D179_DELTA=NONE
ARCHITECTURE_DECISION_REQUIRED=NO
UPSTREAM_COORDINATION_REQUIRED=NO
```

No PLATxxx/Dxxx approval is required before implementation.

## Classification

```text
WORK_ITEM=I075-C

OBSERVED_I075_B_FAILURE_CLASS=
  PROTOS_API_MISUSE

BOXING_ELIMINATION_TRIGGERED_FAILURE=
  ESTABLISHED

LOCAL_ACCESS_ROOT_CAUSE=
  ProtosFrameLexicalBindingAuthority retains the BytecodeNode captured when
  the authority is installed and later passes that tier-specific node to
  LocalRangeAccessor after the Bytecode DSL has replaced it. Boxing elimination
  makes the stale per-node local-tag state observable.

CURRENT_SCOPE_LOCAL_COMPATIBILITY=
  PARTIAL_CURRENT_IMPLEMENTATION

CAPTURED_MATERIALIZED_LOCAL_COMPATIBILITY=
  COMPATIBLE_FOR_DIRECT_MATERIALIZED_ACCESSOR_PATH

RUNTIME_AUTHORITY_LOCAL_COMPATIBILITY=
  INCOMPATIBLE_UNTIL_CURRENT_DECLARING_BYTECODE_NODE_IS_USED

SMALLEST_CORRECT_COMPATIBILITY_CHANGE=
  Preserve ProtosFrameLexicalBindingAuthority and LocalRangeAccessor authority,
  but retain stable declaring-root identity and resolve/use that root's current
  BytecodeNode for every frame-backed accessor operation.

LOCAL_ACCESS_COMPATIBILITY_FIX=
  BOUNDED

BOXING_ELIMINATION_ADOPTION=
  READY_FOR_BOUNDED_IMPLEMENTATION

IMPLEMENTATION_READY=
  YES

RECOMMENDED_NEXT_SLICE=
  I075-D
```

## I075-D implementation boundary

I075-D may:

1. change `ProtosFrameLexicalBindingAuthority` so it no longer permanently
   retains a tier-specific Bytecode node;
2. establish/retain the stable declaring-root reference required to obtain the
   current node;
3. make all frame-backed authority accessor operations use that current node;
4. enable `boxingEliminationTypes = {int.class}`;
5. add focused regression coverage for the stale-node transition;
6. run the repository-required integrated validation.

I075-D must not:

- migrate lexical value authority away from the existing frame-backed model;
- add a second local-value authority;
- replace current access broadly with generic `BytecodeNode.getLocalValue`;
- redesign current/captured lexical identity;
- change D179 presence semantics;
- change PLAT036;
- introduce primitive guest Integer/Float/Boolean representation;
- coordinate upstream unless a new independent upstream defect is established.

Stop if any of those broader changes become required.

## Routing

```text
I075-A=COMPLETE_HISTORICAL
I075-B=STOPPED_NOT_PUBLISHED
I075-C=COMPLETE

I075/#746=OPEN

NEXT_SLICE=I075-D
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos

IMPLEMENTATION_READY=YES
ARCHITECTURE_DECISION_REQUIRED=NO
```
