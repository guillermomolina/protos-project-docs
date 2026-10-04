# PERF030-L — frame authority emptiness O(1) publication

## Scope

This record preserves the published PERF030-L implementation of the fourth
repair family slice selected by PERF030-H:

`F5_EMPTY_QUERY_RESCANS_FRAME`.

The owning live work item is
`guillermomolina/protos#784` (**PERF030 — Make frame-local creation ordinal
PE-constant**). PERF024 / #756 remains blocked from physical cross-runtime graph
interpretation while the remaining PERF030 compilerability families are open.

This record is non-normative implementation/publication evidence. It does not
change Protos language semantics or specification authority.

## Published authority

```text
SLICE=PERF030-L
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

PROTOS_REVISION=7ae6827219bfdeb55c4a365cce046ade1222f08d
COMMIT_SUBJECT=PERF030-L: make frame authority emptiness O(1)
PROTOS_VERSION=0.3.189-SNAPSHOT

FAMILY=F5_EMPTY_QUERY_RESCANS_FRAME
F5_SINK_COUNT=1

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
src/test/java/com/guillermomolina/protos/execution/ProtosPerf025FrameMaterializationSliceTest.java
tools/java_local_range_pe_guard_baseline.json
tools/java_local_range_pe_reachability_baseline.json
```

## Repair

Before PERF030-L, `ProtosFrameLexicalBindingAuthority.isEmpty()` rescanned the
entire frame-backed local range using `LocalRangeAccessor.isCleared(...,
ordinal)` even though the authority already maintained exact establishment
metadata.

The published implementation removes that range scan entirely.

General mode now answers emptiness from:

```text
establishmentOrder.isEmpty()
```

while compact mode answers it from:

```text
lastCompactFrameOrdinal < 0
```

The defensive dynamic-overflow check remains before those metadata queries.
No new counter, snapshot, compiler assertion, exploded loop, or second authority
of state was introduced.

The resulting `isEmpty()` operation is O(1) and performs no
`LocalRangeAccessor` access.

## Metadata invariants exercised

The focused regression extends the existing frame-authority pay-as-you-grow
coverage and verifies both authority modes.

Compact mode:

```text
frame-backed PRESENT bindings -> isEmpty() == false
remove all                  -> isEmpty() == true
establishmentOrder          -> null
lastCompactFrameOrdinal     -> -1
recreate in ascending order -> isEmpty() == false
compact representation      -> retained
```

General mode:

```text
materialize establishmentOrder -> isEmpty() == false
remove every binding            -> isEmpty() == true
dynamicOverflow after last remove -> null
general representation          -> retained
recreate one binding            -> isEmpty() == false
```

Existing remove/recreate establishment ordering remains covered.

## Preserved semantics

The publication preserves the existing behavior for:

```text
STATIC_LOCAL_EXISTS != SEMANTIC_BINDING_PRESENT
PRESENT(null) != ABSENT
OPEN / CLOSED / FROZEN
duplicate creation errors
capture by reference
D179 late-nearer behavior
establishment order
remove/recreate ordering
compact -> general transition
dynamic overflow
frame-authority materialization
captured lexical behavior
debugger/tooling snapshots
```

No normative specification change is required.

## Exact PE guard transition

With the old baselines retained after the source/test repair, the guard produced
the expected single stale entry in each baseline:

```text
TOTAL_LOCAL_RANGE_SINKS=36
PE_REACHABLE_PROVEN_CONSTANT=13
PE_REACHABLE_RISK=13
BOUNDARY_CUT=6
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0

NEW_UNBASELINED_RISKS=0
STALE_BASELINE_ENTRIES=1
NEW_UNBASELINED_REACHABILITY=0
REACHABILITY_DRIFT=0
STALE_REACHABILITY_ENTRIES=1
```

The only stale identity was:

```text
ProtosFrameLexicalBindingAuthority.isEmpty()
isCleared(..., ordinal)
LOOP_INDEX
occurrence=0
```

After removing only that exact F5 entry from both baselines, the final published
guard result is:

```text
TOTAL_LOCAL_RANGE_SINKS=36

PROVEN_CONSTANT_OPERAND=4
DIRECT_CONSTANT=0
TRUFFLE_BOUNDARY=2
RUNTIME_NAME_DERIVED=10
LOOP_INDEX=11
METHOD_PARAMETER=9
UNKNOWN=0

PROVEN_SAFE=6
BASELINED_RISKS=30
NEW_UNBASELINED_RISKS=0
STALE_BASELINE_ENTRIES=0

PE_REACHABLE_PROVEN_CONSTANT=13
PE_REACHABLE_RISK=13
BOUNDARY_CUT=6
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0

NEW_UNBASELINED_REACHABILITY=0
REACHABILITY_DRIFT=0
STALE_REACHABILITY_ENTRIES=0
local-range-pe-guard=PASS
```

The sink is removed, not reclassified.

The remaining 13 PE-reachable risks are exactly:

```text
F4_PE_VISIBLE_LIFECYCLE_RANGE_SCANS=10
F3_GENERIC_NAME_TO_RANGE_INDEX=3
```

## Validation

The maintainer reported all requested local validation PASS for the final
published candidate.

Observed validation included:

```text
make compile
focused ProtosPerf025FrameMaterializationSliceTest
make test-local-range-pe-guard
make test
```

The LocalRange PE guard self-tests also passed:

```text
SELF_TESTS=39 PASS
```

No external IGV/JFR run was required between this bounded F5 repair and the
remaining F4/F3 remediation families.

## Versioning and changelog

The published product version is:

```text
0.3.189-SNAPSHOT
```

The implementation changelog records PERF030-L as an O(1) frame-authority
emptiness repair with no semantic, specification, or benchmark change.

## Coordination

```text
PERF030_L=COMPLETE
F5_EMPTY_QUERY_RESCANS_FRAME=REPAIRED
isEmpty_LOCAL_RANGE_SCAN=REMOVED
isEmpty_COMPLEXITY=O(1)

TOTAL_LOCAL_RANGE_SINKS=36
PE_REACHABLE_PROVEN_CONSTANT=13
PE_REACHABLE_RISK=13
BOUNDARY_CUT=6
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0

PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
NEW_FORMAL_ISSUE_REQUIRED=NO

NEXT_SLICE=PERF030-M
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
FAMILY=F4_PE_VISIBLE_LIFECYCLE_RANGE_SCANS
SINKS_COVERED=10
```

PERF030 remains open. PERF024 remains blocked until the compilerability class is
fully repaired and accepted.

AI assistance: this durable publication record was drafted with ChatGPT from the
exact published PERF030-L commit, the checked-in LocalRange PE baselines, the
PERF030-H repair map, live GitHub Issue state, and maintainer-reported local
validation results.
