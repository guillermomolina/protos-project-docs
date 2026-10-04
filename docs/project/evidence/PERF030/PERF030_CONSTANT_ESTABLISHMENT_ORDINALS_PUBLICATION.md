# PERF030-I — constant establishment ordinal publication

## Scope

This record preserves the published PERF030-I implementation of the first
repair family selected by PERF030-H:

`F1_LOWERER_KNOWN_ESTABLISHMENT_ORDINAL_LOST`.

The owning live work item is
`guillermomolina/protos#784` (**PERF030 — Make frame-local creation ordinal
PE-constant**). PERF024 / #756 remains blocked from physical cross-runtime graph
interpretation while the remaining PERF030 compilerability families are open.

This record is non-normative implementation/publication evidence. It does not
change Protos language semantics or specification authority.

## Published authority

```text
SLICE=PERF030-I
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

PROTOS_REVISION=37bab1b034e87b79ff4475a3e4013cfb5ef3fd73
COMMIT_SUBJECT=PERF030-I: carry lowerer-known establishment ordinals as constant operands
PROTOS_VERSION=0.3.184-SNAPSHOT

FAMILY=F1_LOWERER_KNOWN_ESTABLISHMENT_ORDINAL_LOST
F1_SINK_COUNT=6

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
src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java
src/main/java/com/guillermomolina/protos/execution/ProtosInlineCallbackFrameBindings.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf025CallbackConsumerSpecializationTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf025CompactCalleeExecutionTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf025H1IndexedParameterEstablishmentTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf025LazyInlineCallbackActivationTest.java
tools/java_local_range_pe_reachability_baseline.json
```

The published commit contains 477 additions and 204 deletions.

## Repair result

PERF030-H identified six primary F1 sinks:

```text
1  ProtosBytecodeRootNode.createCurrentFrameBinding / isCleared
2  ProtosBytecodeRootNode.createCurrentFrameBinding / setObject

7  ProtosFrameLexicalBindingAuthority.createFrameBackedBindingAt / isCleared
8  ProtosFrameLexicalBindingAuthority.createFrameBackedBindingAt / setObject

19 ProtosInlineCallbackFrameBindings.create / isCleared
20 ProtosInlineCallbackFrameBindings.create / setObject
```

The root cause was loss of structural constancy between a layout ordinal known
by the lowerer and the eventual `LocalRangeAccessor` access.

PERF030-I repairs that complete family.

### Scalar establishment

Frame-native, indexed-authority and inline-callback establishment operations now
carry each lowerer-known ordinal as an `int` Bytecode DSL constant operand.

The lowerer passes those values through the generated operation builder as
instruction constants instead of emitting them as ordinary dynamic
`emitLoadConstant(ordinal)` operands.

The covered operation shapes include current HEAD's frame parameter/rest/current
creation, persistent-authority indexed parameter/rest establishment, and inline
callback parameter/current creation paths.

The six F1 sinks therefore have only:

```text
PE_CONSTANT_FROM_OPERATION
```

provenance in the exact published reachability baseline.

### Frame-native multiple creation

The former frame-native multiple-creation operations carried an ordinal array
that was indexed at runtime. PERF030-I removes that architecture.

Current lowering now uses:

```text
ObserveMultipleCreatePrefix
ObserveInlineMultipleCreatePrefix
```

to observe the complete fixed source prefix once, stores those observations, and
then emits one scalar frame-native creation per target in source order. Each
scalar creation owns its ordinal as a constant operand.

The removed array-based operations include:

```text
MultipleCreateFrameLocals
MultipleCreateInlineFrameLocals
```

and the former inline `multipleCreate` helper.

The transformation preserves the existing multiple-create contract:

```text
complete fixed prefix observed before first creation
source-order creation
no rollback of earlier successful creations if a later creation fails
same source result on success
no eager activation/materialization merely for the optimization
```

No `@ExplodeLoop` or constant-array workaround is used.

## Exact PE reachability result

Before PERF030-I, PERF030-G/H authority was:

```text
PE_REACHABLE_PROVEN_CONSTANT=4
PE_REACHABLE_RISK=24
BOUNDARY_CUT=6
NOT_PE_REACHABLE=4
```

At the published PERF030-I revision, the checked-in exact baseline is:

```text
TOTAL_LOCAL_RANGE_SINKS=38

PE_REACHABLE_PROVEN_CONSTANT=10
PE_REACHABLE_RISK=18
BOUNDARY_CUT=6
NOT_PE_REACHABLE=4
```

Thus all six F1 sinks move:

```text
PE_REACHABLE_RISK
 -> PE_REACHABLE_PROVEN_CONSTANT
```

with no change in total sink count.

The remaining 18 reachable risks are exactly the later repair families selected
by PERF030-H:

```text
F2_CAPTURED_OWNER_ORDINAL_RUNTIME_PIPELINE=3
F3_GENERIC_NAME_TO_RANGE_INDEX=3
F4_PE_VISIBLE_LIFECYCLE_RANGE_SCANS=10
F5_EMPTY_QUERY_RESCANS_FRAME=1
F6_INLINE_CAPTURE_REDUNDANT_RANGE_GUARD=1
```

## Regression coverage

The published tests add structural instruction checks for constant ordinal
arguments across:

```text
frame-native parameter/rest establishment
frame-native current creation
persistent-authority indexed parameter/rest establishment
inline callback parameter/current establishment
frame-native multiple creation
inline frame-native multiple creation
```

They also retain/extend behavioral coverage for the multiple-create ordering and
lazy-inline activation surfaces touched by the lowering change.

The maintainer reported all applicable local tests PASS after publication.

## Versioning and changelog

The published product version is:

```text
0.3.184-SNAPSHOT
```

The implementation changelog records PERF030-I as a compilerability/performance
repair with:

```text
semantic change = no
specification change = no
```

## Coordination

```text
PERF030_I=COMPLETE
F1_LOWERER_KNOWN_ESTABLISHMENT_ORDINAL_LOST=REPAIRED

F1_SINKS_REPAIRED=6
PE_REACHABLE_PROVEN_CONSTANT=10
PE_REACHABLE_RISK=18

PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
NEW_FORMAL_ISSUE_REQUIRED=NO

NEXT_SLICE=PERF030-J
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
FAMILY=F2_CAPTURED_OWNER_ORDINAL_RUNTIME_PIPELINE
SINKS_COVERED=3
```

PERF030 remains open. PERF024 remains blocked until the compilerability class is
fully repaired and accepted.

AI assistance: this durable publication record was drafted with ChatGPT from the
exact published PERF030-I commit, its checked-in reachability baseline and
changelog, the PERF030-H repair map, and the maintainer-reported local validation
result.
