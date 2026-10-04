# PERF030-M — lifecycle frame range scans boundary publication

## Scope

This record preserves the published PERF030-M implementation of the fifth
repair-family slice selected by PERF030-H:

`F4_PE_VISIBLE_LIFECYCLE_RANGE_SCANS`.

The owning live work item is
`guillermomolina/protos#784` (**PERF030 — Make frame-local creation ordinal
PE-constant**). PERF024 / #756 remains blocked from physical cross-runtime graph
interpretation while the final PERF030 repair family and final external
acceptance remain open.

This record is non-normative implementation/publication evidence. It does not
change Protos language semantics or specification authority.

## Published authority

```text
SLICE=PERF030-M
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

PROTOS_REVISION=d7b8cbdc85ef9474395cf6eec4e70d2488e72916
COMMIT_SUBJECT=PERF030-M: boundary lifecycle frame range scans
PROTOS_VERSION=0.3.191-SNAPSHOT

FAMILY=F4_PE_VISIBLE_LIFECYCLE_RANGE_SCANS
F4_SINK_COUNT=10
F4_BOUNDARY_CUT_COUNT=10

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
src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java
src/main/java/com/guillermomolina/protos/execution/ProtosInlineCallbackFrameBindings.java
tools/java_local_range_pe_guard_baseline.json
tools/java_local_range_pe_reachability_baseline.json
```

## Repair

PERF030-M moves the ten F4 lifecycle scans behind dedicated
`@TruffleBoundary` helpers instead of attempting to make loop induction or
runtime-name-derived indices PE constants.

The repaired lifecycle seams are:

```text
ProtosFrameLexicalBindingAuthority.ensureGeneralEstablishmentOrder
  -> materializeGeneralEstablishmentOrder(BytecodeNode)

ProtosFrameLexicalBindingAuthority.adoptPresentFrameBackedBindings
  -> adoptPresentFrameBackedBindingsSlow(BytecodeNode)

ProtosFrameLexicalBindingAuthority.appendBindingsTo
  -> appendBindingsToSlow(BytecodeNode, ArrayList, ArrayList)

ProtosInlineCallbackFrameBindings.durableActivation
  -> transferFrameBindingsToDurableActivation(
         PreparedInlineLiteralCall,
         LocalRangeAccessor,
         ProtosFrameLexicalLayout,
         BytecodeNode,
         MaterializedFrame)
```

The cheap `ensureGeneralEstablishmentOrder` guard remains outside the boundary.
The current `BytecodeNode` is still resolved at the existing authority seam
instead of being retained as stable state.

For inline callbacks, the first implementation attempt exposed a Truffle
restriction: a `@TruffleBoundary` method cannot accept a `VirtualFrame`
parameter. The published repair therefore keeps:

```text
if (child.frameBindingsTransferred()) {
    return child.activation();
}
```

outside the boundary, materializes the frame only after that fast guard, and
passes a `MaterializedFrame` into the slow transfer helper. Therefore already
transferred callbacks and completely unobserved invocations do not materialize
a frame merely because the boundary exists.

No `@ExplodeLoop`, compiler assertion, duplicated metadata, or auxiliary
authority counter was introduced.

## Preserved behavior

The publication preserves:

```text
STATIC_LOCAL_EXISTS != SEMANTIC_BINDING_PRESENT
PRESENT(null) != ABSENT
OPEN / CLOSED / FROZEN
duplicate creation behavior
capture by reference
D179 late-nearer behavior
establishment order
remove/recreate ordering
compact -> general transition
dynamic overflow semantics
authority installation over an already-live frame
authority handoff / migration
frame-authority materialization
inline callback activation laziness
first-observer materialization
transfer at most once
layout/declaration order during transfer
debugger/tooling observation
captured lexical behavior
current BytecodeNode coherence
```

No normative specification change is required.

## Exact PE guard transition

With the old baselines retained after the source repair, the guard produced
exactly the expected F4 transition:

```text
TOTAL_LOCAL_RANGE_SINKS=36

PROVEN_CONSTANT_OPERAND=4
DIRECT_CONSTANT=0
TRUFFLE_BOUNDARY=12
RUNTIME_NAME_DERIVED=8
LOOP_INDEX=3
METHOD_PARAMETER=9
UNKNOWN=0

PROVEN_SAFE=16
BASELINED_RISKS=20
NEW_UNBASELINED_RISKS=0
STALE_BASELINE_ENTRIES=10

PE_REACHABLE_PROVEN_CONSTANT=13
PE_REACHABLE_RISK=3
BOUNDARY_CUT=16
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0

NEW_UNBASELINED_REACHABILITY=10
REACHABILITY_DRIFT=0
STALE_REACHABILITY_ENTRIES=10
```

The ten stale local entries were precisely the old F4 risk identities. The ten
new reachability entries were precisely the new
`TRUFFLE_BOUNDARY / BOUNDARY_CUT` helper identities.

Baseline adoption was deliberately bounded:

- remove only the ten old F4 entries from the local-risk baseline;
- replace only the ten old F4 reachability entries with the exact new
  boundary entries emitted by the guard candidate;
- preserve all non-F4 reachability entries exactly.

The final guard result is:

```text
TOTAL_LOCAL_RANGE_SINKS=36

PROVEN_CONSTANT_OPERAND=4
DIRECT_CONSTANT=0
TRUFFLE_BOUNDARY=12
RUNTIME_NAME_DERIVED=8
LOOP_INDEX=3
METHOD_PARAMETER=9
UNKNOWN=0

PROVEN_SAFE=16
BASELINED_RISKS=20
NEW_UNBASELINED_RISKS=0
STALE_BASELINE_ENTRIES=0

PE_REACHABLE_PROVEN_CONSTANT=13
PE_REACHABLE_RISK=3
BOUNDARY_CUT=16
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0

NEW_UNBASELINED_REACHABILITY=0
REACHABILITY_DRIFT=0
STALE_REACHABILITY_ENTRIES=0
local-range-pe-guard=PASS
```

The only three remaining PE-reachable risks are F3:

```text
ProtosFrameLexicalBindingAuthority.containsBinding
    isCleared(..., offset)

ProtosFrameLexicalBindingAuthority.readBinding
    isCleared(..., offset)

ProtosFrameLexicalBindingAuthority.readBinding
    getObject(..., offset)
```

All three are genuinely runtime-name-derived indices.

## Validation

The maintainer reported all requested local validation PASS for the final
published candidate.

Observed validation included:

```text
make compile = PASS
focused Java tests = 35 tests, 0 failures, 0 errors, 0 skipped
make test-local-range-pe-guard = PASS
guard self-tests = 39 PASS
make test = PASS
```

The focused coverage included:

```text
ProtosPerf025FrameMaterializationSliceTest
ProtosI075DLexicalAuthorityCurrentBytecodeNodeTest
ProtosPerf025LazyInlineCallbackActivationTest
ProtosPerf025CallbackConsumerSpecializationTest
```

No external IGV/JFR run was performed between F4 and the final F3 repair
family.

## Versioning and changelog

The published product version is:

```text
0.3.191-SNAPSHOT
```

The implementation changelog records PERF030-M as a lifecycle
`LocalRangeAccessor` boundary repair with no semantic, specification, or
benchmark change.

## Coordination

```text
PERF030_M=COMPLETE
F4_PE_VISIBLE_LIFECYCLE_RANGE_SCANS=REPAIRED
F4_SINK_COUNT=10
F4_BOUNDARY_CUT_COUNT=10

TOTAL_LOCAL_RANGE_SINKS=36
PE_REACHABLE_PROVEN_CONSTANT=13
PE_REACHABLE_RISK=3
BOUNDARY_CUT=16
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0

PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
NEW_FORMAL_ISSUE_REQUIRED=NO

REMAINING_PE_REACHABLE_RISKS=3
NEXT_SLICE=PERF030-N
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
FAMILY=F3_GENERIC_NAME_TO_RANGE_INDEX
SINKS_COVERED=3
```

PERF030-N is the final repair family currently classified by PERF030-H.
PERF030 remains open after PERF030-M; final external compilerability acceptance
must still be decided after F3 is repaired before PERF024 physical graph
interpretation resumes.

AI assistance: this durable publication record was drafted with ChatGPT from
the exact published PERF030-M product commit, the checked-in LocalRange PE
baselines, the PERF030-H repair map, live GitHub Issue state, and
maintainer-reported local validation results.
