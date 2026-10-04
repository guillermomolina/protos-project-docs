# PERF030-K — inline captured range guard publication

## Scope

This record preserves the published PERF030-K implementation of the third
repair family selected by PERF030-H:

`F6_INLINE_CAPTURE_REDUNDANT_RANGE_GUARD`.

The owning live work item is
`guillermomolina/protos#784` (**PERF030 — Make frame-local creation ordinal
PE-constant**). PERF024 / #756 remains blocked from physical cross-runtime graph
interpretation while the remaining PERF030 compilerability families are open.

This record is non-normative implementation/publication evidence. It does not
change Protos language semantics or specification authority.

## Published authority

```text
SLICE=PERF030-K
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

PROTOS_REVISION=1034574c7c1081fb96cef0263523eac89657d635
COMMIT_SUBJECT=PERF030-K: remove redundant inline captured range guard
PROTOS_VERSION=0.3.188-SNAPSHOT

FAMILY=F6_INLINE_CAPTURE_REDUNDANT_RANGE_GUARD
F6_SINK_COUNT=1
CONTRACT_STATUS=STRUCTURALLY_SAFE_DESPITE_STATIC_RISK

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
src/main/java/com/guillermomolina/protos/execution/ProtosInlineCallbackFrameBindings.java
src/test/java/com/guillermomolina/protos/execution/CanonicalBindingAnalyzerTest.java
tools/java_local_range_pe_guard_baseline.json
tools/java_local_range_pe_reachability_baseline.json
```

## Structural proof retained by the repair

PERF030-H classified the single F6 sink:

```text
ProtosInlineCallbackFrameBindings.admitsCapturedAccess
 -> frameBackedLayout.offsetOf(name)
 -> LocalRangeAccessor.isCleared(..., ordinal)
```

as structurally safe despite conservative static risk.

`CanonicalBindingAnalyzer.resolve(name, scope)` stops at the nearest declaring
lexical scope. In the current scope, an already-established declaration is
`Resolved`, while a declared-but-not-yet-established declaration is
`Candidate`. `CapturedResolved` is produced only after walking outward to an
already-established owner and therefore always has positive lexical depth.

Lowering emits the inline captured operations that reach
`admitsCapturedAccess` only for `CapturedResolved`. Consequently the same name
cannot simultaneously belong to the callback's current frame-backed layout and
be the captured outer binding selected for the direct path. A later dynamic
nearer binding requires the materialized activation/context path, which the
helper rejects before taking the direct captured path.

PERF030-K makes that existing invariant explicit instead of performing a
runtime name-to-layout probe.

## Repair result

The helper now admits the direct captured path only when:

```text
activation is not materialized
AND
lexicalDepth > 0
```

The four inline captured read/write-target callers no longer pass the current
frame layout, runtime name, frame accessor, bytecode node, or frame merely to
re-prove an ownership fact already guaranteed by canonical binding analysis and
lowering.

The obsolete runtime-name-derived `LocalRangeAccessor.isCleared` sink therefore
disappears entirely rather than being reclassified or hidden behind another
lookup.

Focused `CanonicalBindingAnalyzerTest` coverage records one lexical shape with:

```text
outer established name          -> CapturedResolved, depth 1
current declaration pre-create  -> Candidate, depth 0
current declaration post-create -> Resolved
```

This makes the ownership separation relied upon by the helper directly visible
in regression coverage.

## Preserved semantics

The published implementation retains the existing behavior for:

```text
capture by reference
D179 C0 PRESENT / ABSENT / PRESENT(null)
late nearer-binding behavior
inline callback activation laziness
captured frame and materialized reads
captured writable-destination selection before RHS
no post-RHS retargeting
selected-owner mutation behavior
CLOSED/FROZEN rules
generic fallback behavior
existing guest Error timing/translation
```

No normative specification change is required.

## Exact PE reachability result

Before PERF030-K, after PERF030-J:

```text
TOTAL_LOCAL_RANGE_SINKS=38
PE_REACHABLE_PROVEN_CONSTANT=13
PE_REACHABLE_RISK=15
BOUNDARY_CUT=6
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0
```

At the published PERF030-K revision:

```text
TOTAL_LOCAL_RANGE_SINKS=37
PE_REACHABLE_PROVEN_CONSTANT=13
PE_REACHABLE_RISK=14
BOUNDARY_CUT=6
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0

NEW_UNBASELINED_RISKS=0
STALE_BASELINE_ENTRIES=0
NEW_UNBASELINED_REACHABILITY=0
REACHABILITY_DRIFT=0
STALE_REACHABILITY_ENTRIES=0
local-range-pe-guard=PASS
```

The total sink count and reachable-risk count each decrease by exactly one. No
other sink changes classification.

The remaining 14 PE-reachable risks are exactly:

```text
F5_EMPTY_QUERY_RESCANS_FRAME=1
F4_PE_VISIBLE_LIFECYCLE_RANGE_SCANS=10
F3_GENERIC_NAME_TO_RANGE_INDEX=3
```

## Validation

The maintainer reported the following local validation PASS on the final
published candidate:

```text
make compile
focused CanonicalBindingAnalyzerTest + ProtosPerf025CallbackConsumerSpecializationTest
  Tests run: 22, Failures: 0, Errors: 0, Skipped: 0
make test-local-range-pe-guard
make test
```

The static guard self-tests also passed:

```text
SELF_TESTS=39 PASS
```

No external IGV/JFR run was required for this structurally proven F6 cleanup.

## Versioning and changelog

The published product version is:

```text
0.3.188-SNAPSHOT
```

The implementation changelog records PERF030-K as a compilerability/performance
repair with no semantic, specification, or benchmark change.

## Coordination

```text
PERF030_K=COMPLETE
F6_INLINE_CAPTURE_REDUNDANT_RANGE_GUARD=REPAIRED

TOTAL_LOCAL_RANGE_SINKS=37
PE_REACHABLE_PROVEN_CONSTANT=13
PE_REACHABLE_RISK=14
BOUNDARY_CUT=6
NOT_PE_REACHABLE=4

PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
NEW_FORMAL_ISSUE_REQUIRED=NO

NEXT_SLICE=PERF030-L
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
FAMILY=F5_EMPTY_QUERY_RESCANS_FRAME
SINKS_COVERED=1
```

PERF030 remains open. PERF024 remains blocked until the compilerability class is
fully repaired and accepted.

AI assistance: this durable publication record was drafted with ChatGPT from the
exact published PERF030-K commit, the checked-in LocalRange PE baselines, the
PERF030-H repair map, live GitHub Issue state, and maintainer-reported local
validation results.
