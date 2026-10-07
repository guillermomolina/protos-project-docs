# PERF030-J — captured-owner constant ordinal publication

## Scope

This record preserves the published PERF030-J implementation of the second
repair family selected by PERF030-H:

`F2_CAPTURED_OWNER_ORDINAL_RUNTIME_PIPELINE`.

The owning live work item is
`guillermomolina/protos#784` (**PERF030 — Make frame-local creation ordinal
PE-constant**). PERF024 / #756 remains blocked from physical cross-runtime graph
interpretation while the remaining PERF030 compilerability families are open.

This record is non-normative implementation/publication evidence. It does not
change Protos language semantics or specification authority.

## Published authority

```text
SLICE=PERF030-J
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

PROTOS_REVISION=873c9a6ac466ddcae7eedfd7d23ed38ab17c569f
COMMIT_SUBJECT=PERF030-J: carry captured owner ordinals as constant operands
PROTOS_VERSION=0.3.186-SNAPSHOT

FAMILY=F2_CAPTURED_OWNER_ORDINAL_RUNTIME_PIPELINE
F2_SINK_COUNT=3

OBSERVABLE_LANGUAGE_SEMANTIC_CHANGE=NO
SPEC_CHANGE=NO
BENCHMARK_CHANGE=NO

MAINTAINER_REPORTED_LOCAL_TESTS=PASS
```

## Changed paths

The publication changes exactly these product-repository paths:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosInlineCallbackFrameBindings.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosI068Slice5CapturedMaterializedLexicalLoweringTest.java
tools/java_local_range_pe_reachability_baseline.json
```

## Repair result

PERF030-H identified three primary F2 sinks:

```text
ProtosFrameLexicalBindingAuthority.hasFrameBackedBindingAt / isCleared
ProtosFrameLexicalBindingAuthority.readFrameBackedBindingAt / getObject
ProtosFrameLexicalBindingAuthority.assignFrameBackedBindingAt / setObject
```

Before PERF030-J, each sink was PE-reachable through a captured-read or
captured-write path whose frame ordinal reached the `LocalRangeAccessor` as an
ordinary operation operand or another unproven runtime argument.

PERF030-J repairs the complete family.

### Captured read and destination resolution

The lowerer-known captured owner ordinal is now an `int` Bytecode DSL constant
operand on the ordinary and inline captured-read operations and on the ordinary
and inline captured writable-destination resolution operations.

The lowerer passes the statically known ordinal structurally through the
generated operation builder. It is no longer emitted as a dynamic
`emitLoadConstant(frameBackedOrdinal(...))` operand.

The resulting authority calls therefore reach the two captured-read sinks only
with:

```text
PE_CONSTANT_FROM_OPERATION
```

provenance.

### Post-RHS captured assignment

Destination selection still occurs before RHS evaluation.

`CapturedLexicalWriteTarget` retains the exact selected owner/authority or
generic destination, but it no longer stores the frame ordinal. The assignment
operation receives the same lowerer-known ordinal independently as its own
`int` constant operand and supplies it to
`assignFrameBackedBindingAt(name, ordinal, value)`.

This preserves the semantic rule:

```text
resolve destination
 -> evaluate RHS
 -> mutate exactly the already-selected destination
```

The assignment path never re-resolves after RHS effects and no longer relies on
runtime target state to satisfy the PE constant-index contract.

## Preserved semantics

The published implementation retains the existing behavior for:

```text
capture by reference
lexical depth
PRESENT / ABSENT / PRESENT(null)
late nearer-binding creation/removal retargeting at destination-resolution time
destination selection before RHS evaluation
no post-RHS retargeting
selected-owner removal failure
CLOSED captured-owner mutation
FROZEN captured-owner rejection
generic fallback behavior
inline callback activation laziness
existing guest Error timing/translation
materialized captured-local paths
```

No normative specification change is required.

## Exact PE reachability result

Before PERF030-J, after PERF030-I, the exact checked-in baseline was:

```text
TOTAL_LOCAL_RANGE_SINKS=38
PE_REACHABLE_PROVEN_CONSTANT=10
PE_REACHABLE_RISK=18
BOUNDARY_CUT=6
NOT_PE_REACHABLE=4
```

At the published PERF030-J revision it is:

```text
TOTAL_LOCAL_RANGE_SINKS=38
PE_REACHABLE_PROVEN_CONSTANT=13
PE_REACHABLE_RISK=15
BOUNDARY_CUT=6
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0
```

All three F2 sinks therefore move:

```text
PE_REACHABLE_RISK
 -> PE_REACHABLE_PROVEN_CONSTANT
```

with `PE_CONSTANT_FROM_OPERATION` provenance and no change in total sink count.

Two additional baseline entries change only their representative call-chain
signatures because the relevant root operation now owns the ordinal as a
constant operand. Their classification remains `PE_REACHABLE_RISK`.

The remaining 15 reachable risks are exactly the later families selected by
PERF030-H:

```text
F6_INLINE_CAPTURE_REDUNDANT_RANGE_GUARD=1
F5_EMPTY_QUERY_RESCANS_FRAME=1
F4_PE_VISIBLE_LIFECYCLE_RANGE_SCANS=10
F3_GENERIC_NAME_TO_RANGE_INDEX=3
```

## Validation

The maintainer reported the following local validation PASS on the final
published candidate:

```text
make compile
focused captured read/write and inline/materialized Java regressions
make test-local-range-pe-guard
make test
```

The final static guard result was:

```text
SELF_TESTS=39 PASS
TOTAL_LOCAL_RANGE_SINKS=38
PE_REACHABLE_PROVEN_CONSTANT=13
PE_REACHABLE_RISK=15
BOUNDARY_CUT=6
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0
NEW_UNBASELINED_REACHABILITY=0
REACHABILITY_DRIFT=0
STALE_REACHABILITY_ENTRIES=0
local-range-pe-guard=PASS
```

The focused regression evidence included destination-before-RHS, no
post-RHS-retargeting, captured-by-reference, CLOSED/FROZEN mutation, late-nearer
retargeting, selected-owner removal, and inline captured read/write without
activation materialization.

## Versioning and changelog

The published product version is:

```text
0.3.186-SNAPSHOT
```

The implementation changelog records PERF030-J as a compilerability/performance
repair with no semantic or specification change.

## Coordination

```text
PERF030_J=COMPLETE
F2_CAPTURED_OWNER_ORDINAL_RUNTIME_PIPELINE=REPAIRED

F2_SINKS_REPAIRED=3
PE_REACHABLE_PROVEN_CONSTANT=13
PE_REACHABLE_RISK=15

PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
NEW_FORMAL_ISSUE_REQUIRED=NO

NEXT_SLICE=PERF030-K
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
FAMILY=F6_INLINE_CAPTURE_REDUNDANT_RANGE_GUARD
SINKS_COVERED=1
```

PERF030 remains open. PERF024 remains blocked until the compilerability class is
fully repaired and accepted.

AI assistance: this durable publication record was drafted with ChatGPT from the
exact published PERF030-J commit, its checked-in reachability baseline and
changelog, the PERF030-H repair map, live GitHub Issue state, and the
maintainer-reported local validation result.
