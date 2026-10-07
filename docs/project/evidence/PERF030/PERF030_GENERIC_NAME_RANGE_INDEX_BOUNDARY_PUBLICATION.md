# PERF030-N — generic frame-name range-index boundary publication

## Scope

This record preserves the published PERF030-N implementation of the sixth and
final repair-family slice selected by PERF030-H:

`F3_GENERIC_NAME_TO_RANGE_INDEX`.

The owning live work item is
`guillermomolina/protos#784` (**PERF030 — Make frame-local creation ordinal
PE-constant**). PERF024 / #756 remains blocked from physical cross-runtime graph
interpretation until PERF030 completes final external compilerability acceptance.

This record is non-normative implementation/publication evidence. It does not
change Protos language semantics or specification authority.

## Published authority

```text
SLICE=PERF030-N
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

PROTOS_REVISION=cf6eb4c9aeaef4049fa5b56089d2dd1d2654d44b
COMMIT_SUBJECT=PERF030-N: boundary generic frame name lookup
PROTOS_VERSION=0.3.193-SNAPSHOT

FAMILY=F3_GENERIC_NAME_TO_RANGE_INDEX
F3_SINK_COUNT=3
F3_BOUNDARY_CUT_COUNT=3

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
tools/java_local_range_pe_guard_baseline.json
tools/java_local_range_pe_reachability_baseline.json
```

## Repair

PERF030-N cuts the three remaining PE-reachable runtime-name-derived
`LocalRangeAccessor` sinks at the generic frame-backed lexical authority
boundary.

The repaired generic seams are:

```text
ProtosFrameLexicalBindingAuthority.containsBinding(String)
    -> LocalRangeAccessor.isCleared(..., offset)

ProtosFrameLexicalBindingAuthority.readBinding(String)
    -> LocalRangeAccessor.isCleared(..., offset)
    -> LocalRangeAccessor.getObject(..., offset)
```

Both generic methods are now `@TruffleBoundary` slow paths. Their method bodies,
lookup rules, dynamic-overflow behavior and current-`BytecodeNode` resolution are
unchanged.

This is deliberately not a fake constantization of a genuinely runtime
`String` name. Statically selected lexical execution continues to use the
existing indexed/ordinal seams established by the earlier PERF030 repair
families; only the residual generic name-based authority fallback is cut out of
partial evaluation.

No `@ExplodeLoop`, `CompilerAsserts.partialEvaluationConstant`, duplicated
authority map, context materialization, semantic redesign or benchmark change is
introduced by this slice.

## Preserved behavior

The publication preserves:

```text
STATIC_LOCAL_EXISTS != SEMANTIC_BINDING_PRESENT
PRESENT(null) != ABSENT
OPEN / CLOSED / FROZEN
duplicate creation behavior
capture by reference
D179 late-nearer behavior
lookup order: current -> captured -> receiver
pre-RHS captured-write destination selection
no post-RHS destination retargeting
establishment order
remove/recreate behavior
dynamic overflow semantics
deferred Context laziness
current BytecodeNode coherence
debugger/reflection projection
```

No normative specification change is required.

## Exact PE guard transition

With the old baselines retained after the source repair, the guard produced
exactly the expected three-entry F3 transition:

```text
TOTAL_LOCAL_RANGE_SINKS=36
PROVEN_CONSTANT_OPERAND=4
DIRECT_CONSTANT=0
TRUFFLE_BOUNDARY=15
RUNTIME_NAME_DERIVED=5
LOOP_INDEX=3
METHOD_PARAMETER=9
UNKNOWN=0

PROVEN_SAFE=19
BASELINED_RISKS=17
NEW_UNBASELINED_RISKS=0
STALE_BASELINE_ENTRIES=3

PE_REACHABLE_PROVEN_CONSTANT=13
PE_REACHABLE_RISK=0
BOUNDARY_CUT=19
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0

NEW_UNBASELINED_REACHABILITY=3
REACHABILITY_DRIFT=0
STALE_REACHABILITY_ENTRIES=3
```

The stale local entries were exactly:

```text
containsBinding(String) / isCleared(..., offset)
readBinding(String) / isCleared(..., offset)
readBinding(String) / getObject(..., offset)
```

Their new reachability classification is exactly
`TRUFFLE_BOUNDARY / BOUNDARY_CUT`.

Baseline adoption was deliberately bounded:

- remove exactly the three old F3 `RUNTIME_NAME_DERIVED` entries from the local
  risk baseline;
- replace exactly the three corresponding reachability entries with the
  guard-emitted `TRUFFLE_BOUNDARY / BOUNDARY_CUT` entries; and
- preserve every non-F3 baseline entry unchanged.

The final guard result is:

```text
TOTAL_LOCAL_RANGE_SINKS=36
PROVEN_CONSTANT_OPERAND=4
DIRECT_CONSTANT=0
TRUFFLE_BOUNDARY=15
RUNTIME_NAME_DERIVED=5
LOOP_INDEX=3
METHOD_PARAMETER=9
UNKNOWN=0

PROVEN_SAFE=19
BASELINED_RISKS=17
NEW_UNBASELINED_RISKS=0
STALE_BASELINE_ENTRIES=0

PE_REACHABLE_PROVEN_CONSTANT=13
PE_REACHABLE_RISK=0
BOUNDARY_CUT=19
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0

NEW_UNBASELINED_REACHABILITY=0
REACHABILITY_DRIFT=0
STALE_REACHABILITY_ENTRIES=0
local-range-pe-guard=PASS
```

PERF030-H therefore has no remaining repair-family member classified as a
PE-reachable risk.

## Validation

The maintainer reported all requested local validation PASS for the published
candidate.

Observed validation included:

```text
make compile = PASS
focused Java regression set = PASS
make test-local-range-pe-guard = PASS
guard self-tests = 39 PASS
make test = PASS
git diff --check = PASS
```

The focused Java regression set included:

```text
ProtosPlat036Slice3FrameBackedCurrentLocalTest
ProtosPerf025LazyLexicalCaptureTest
ProtosI075DLexicalAuthorityCurrentBytecodeNodeTest
ProtosI068Slice6DebuggerReflectionProjectionTest
```

No external IGV/JFR acceptance was performed as part of PERF030-N.

## Versioning and changelog

The published product version is:

```text
0.3.193-SNAPSHOT
```

The implementation changelog records PERF030-N as the final F3 generic
runtime-name-to-range-index boundary repair with no semantic, specification or
benchmark change.

## Coordination

```text
PERF030_N=COMPLETE
F3_GENERIC_NAME_TO_RANGE_INDEX=REPAIRED
F3_SINK_COUNT=3
F3_BOUNDARY_CUT_COUNT=3

TOTAL_LOCAL_RANGE_SINKS=36
PE_REACHABLE_PROVEN_CONSTANT=13
PE_REACHABLE_RISK=0
BOUNDARY_CUT=19
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0

PERF030_H_REPAIR_FAMILIES_REMAINING=0
PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
NEW_FORMAL_ISSUE_REQUIRED=NO

NEXT_SLICE=PERF030-O
WORK_TYPE=INVESTIGATION
IMPLEMENTATION=NO
PURPOSE=FINAL_EXTERNAL_IGV_JFR_ACCEPTANCE
WORKLOAD=primitive-closure-call
DIAGNOSTIC_WARMUP=60
DIAGNOSTIC_STEADY=10
DIAGNOSTIC_SAMPLE_CALLS=100000
```

PERF030-O must perform the final unchanged external compilerability acceptance
against exact product revision
`cf6eb4c9aeaef4049fa5b56089d2dd1d2654d44b`. PERF030 remains open until that
acceptance proves the ordinary IGV path still emits BGV output and JFR no longer
contains the permanent compiler failure class that blocked PERF024.

PERF024 physical cross-runtime graph interpretation must remain paused until the
acceptance result is published and PERF030 coordination is reconciled.

AI assistance: this durable publication record was drafted with ChatGPT from
the exact published PERF030-N product commit, checked-in LocalRange PE
baselines, PERF030-H repair map, live GitHub Issue state, and
maintainer-reported local validation results.
