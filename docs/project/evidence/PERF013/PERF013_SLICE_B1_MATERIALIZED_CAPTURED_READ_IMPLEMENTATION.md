# PERF013 Slice B1 — MaterializedLocalAccessor captured-read implementation

Date: 2026-09-27

## Scope

This record retains the published implementation checkpoint for PERF013 / #724 Slice B1.

B1 is the first slice that changes the causal captured-local read path identified by PERF010-B. It does not migrate captured writes and does not yet rebuild isolated/context-local Closure execution plans as whole lexical root groups.

## Publication

```text
REPOSITORY=guillermomolina/protos
WORK_ITEM=PERF013/#724
PARENT=PERF010-B/#722

BASELINE_REVISION=24b41e8e03b60882f05e4a1a2105b8dbd9a4f21b
BASELINE_VERSION=0.3.100-SNAPSHOT

PRODUCT_REVISION=c498f35a383447c21fa0b63c3414857e777e5300
PRODUCT_VERSION=0.3.101-SNAPSHOT
COMMIT_MESSAGE=PERF013 Slice B1: use MaterializedLocalAccessor for captured reads
```

Published changed paths:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java
src/test/java/com/guillermomolina/protos/execution/ProtosI068Slice5CapturedMaterializedLexicalLoweringTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf013SliceB1MaterializedCapturedReadTest.java
```

No normative specification path changed.

## Implemented mechanism

`ProtosBytecodeRootNode` now enables:

```text
enableMaterializedLocalAccesses=true
```

and defines a dedicated captured-read operation using:

```text
@ConstantOperand(type = MaterializedLocalAccessor.class)
```

The generated builder consumes the exact owner `BytecodeLocal`, allowing Bytecode DSL to encode the stable logical local identity/root index itself.

`CanonicalToBytecodeLowerer` now retains a backend-private per-scope registry:

```text
CanonicalLexicalScope -> name -> BytecodeLocal
```

populated from the current parse/reparse before nested roots are lowered.

For a proven `CapturedResolved` read:

```text
same-group owner BytecodeLocal available
  -> ReadCapturedMaterializedLocal

owner local unavailable
  -> existing ReadCapturedFrameLocal
```

The fallback remains intentionally reachable for isolated/context-local Closure rebuilds that do not yet rebuild the whole lexical owner group.

## Runtime state boundary

The materialized fast path no longer obtains local identity through the runtime authority's:

```text
LocalRangeAccessor
owner BytecodeNode
```

Instead:

```text
local identity = generated MaterializedLocalAccessor constant
runtime state  = retained owner MaterializedFrame
```

`ProtosFrameLexicalBindingAuthority` remains the single execution-context authority and exposes only a backend-private retained-materialized-frame seam for captured access.

The existing context observation/capture boundary remains the materialization trigger; the B1 read seam does not introduce a second eager materialization point.

## Preserved semantics

The existing D179 C0 / I071 captured-read algorithm remains:

```text
check current context
check nearer captured contexts
resolve static owner position
obtain owner execution-context frame
isCleared -> fallback
not cleared -> getObject
```

Therefore:

```text
PRESENT(null) != ABSENT
late nearer creation can retarget
owner remove falls back
owner recreate becomes visible again
capture remains by reference
later mutation remains visible
escaped captures remain live
multi-depth capture remains live
```

The published implementation does not alter captured-write destination semantics.

## Local validation provenance

The maintainer reported:

```text
git diff --check=PASS
mvn compile=PASS
PERF013_B1_FOCAL_TESTS=PASS
RELATED_PERF012_013_I068_TESTS=PASS
FOCAL_TEST_COUNT=38/38
LICENSE_COMPLIANCE=PASS

FULL_MAKE_TEST=DEFERRED_TO_PERF013_CLOSURE
REMOTE_CI_PASS=NOT_CLAIMED
```

A focal owner-removal test initially exposed a missing `SlotNotFound` fixture binding; that was a test-fixture gap and was corrected before the final green result. No production semantic defect was attributed to that failure.

## Current implementation gate

```text
MATERIALIZED_LOCAL_ACCESSES_ENABLED=YES
SAME_GROUP_CAPTURED_READ=MATERIALIZED_LOCAL_ACCESSOR

CAPTURED_READ_OWNER_LOCAL_STATIC=YES
CAPTURED_READ_OWNER_FRAME_DYNAMIC=YES

CAPTURED_READ_RUNTIME_LOCAL_RANGE_ACCESSOR=NO
CAPTURED_READ_RUNTIME_OWNER_BYTECODE_NODE=NO

CAPTURED_WRITE_MECHANISM=UNCHANGED
ISOLATED_GROUP_SAFE_FALLBACK=RETAINED

SEMANTIC_CHANGE=NO
SPEC_CHANGE=NO
```

## Required post-publication causal gate

At this checkpoint there is no newer published `guillermomolina/protos-benchmarks` revision after the pre-B1 PERF010-B Step-0 evidence revision.

Therefore the following must NOT yet be claimed:

```text
CAPTURED_READ_PE_FAILURE=ABSENT
```

The immediate next action is the already-required post-B1 compiler-lifecycle validation against:

```text
PROTOS_REVISION=c498f35a383447c21fa0b63c3414857e777e5300
```

The gate must distinguish source/root role rather than relying on historical numeric root IDs.

Expected result:

```text
CAPTURED_READ_PE_FAILURE=ABSENT
CAPTURED_WRITE_PE_FAILURE=MAY_REMAIN
```

Specifically, the source-equivalent captured-read failure must no longer contain:

```text
CompilerAsserts.partialEvaluationConstant
 -> LocalRangeAccessor.isCleared
 -> ProtosFrameLexicalBindingAuthority.hasFrameBackedBindingAt
 -> ReadCapturedFrameLocal.perform
```

Only after that prediction is observed should PERF013-B2 migrate captured writes.

## Routing

```text
PERF013_SLICE_B1=IMPLEMENTED
PERF013_STATUS=IN_PROGRESS

IMMEDIATE_NEXT_STEP=PERF013_B1_COMPILER_CAUSAL_GATE
NEXT_IMPLEMENTATION_IF_GATE_PASSES=PERF013-B2_MATERIALIZED_CAPTURED_WRITE
FOLLOWING=PERF013-C_IF_REQUIRED
```
