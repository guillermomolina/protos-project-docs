# PERF030 — PE reachability guard publication

## Scope

This record preserves the published PERF030-G tooling slice that extends the
static `LocalRangeAccessor` PE-index guard with conservative interprocedural
reachability from Truffle Bytecode DSL operation specializations.

The owning live work item is
`guillermomolina/protos#784` (**PERF030 — Make frame-local creation ordinal
PE-constant**). PERF024 / #756 remains blocked from physical graph
interpretation while PERF030's compilerability class is unresolved.

This is non-normative tooling/publication evidence. It does not change Protos
language semantics, runtime/product behavior, benchmark behavior, or external
compiler-acceptance evidence.

## Published authority

```text
SLICE=PERF030-G
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

PROTOS_REVISION=ee066755219dfbe6ac7dd69a68b3ac76492bf7ee
COMMIT_SUBJECT=PERF030-G: add PE reachability analysis to LocalRangeAccessor guard

PRODUCT_RUNTIME_CHANGE=NO
OBSERVABLE_LANGUAGE_SEMANTIC_CHANGE=NO
SPEC_CHANGE=NO
BENCHMARK_CHANGE=NO

MAINTAINER_REPORTED_LOCAL_TESTS=PASS
```

Published paths:

```text
Makefile
tools/java_local_range_pe_guard.py
tools/java_local_range_pe_reachability_baseline.json
tools/test_java_local_range_pe_guard.py
```

No production/runtime source is changed by this slice.

## Starting point

PERF030-F had already established a complete local source inventory:

```text
TOTAL_LOCAL_RANGE_SINKS=38

PROVEN_SAFE=6
KNOWN_BASELINED_RISK=32
UNKNOWN=0
```

That local classification deliberately did not answer whether the 32 risks
could actually be reached from partial evaluation.

PERF030-G adds that second dimension.

## Reachability model

The checker treats:

```text
@Specialization methods inside @Operation classes
```

as PE roots.

Calls into methods annotated:

```text
@TruffleBoundary
```

cut PE traversal.

The checker builds a conservative static call graph over production Java and
propagates index provenance backwards from each `LocalRangeAccessor` sink
through ordinary helpers and method parameters.

The published analysis supports, among other current source shapes:

- static, `this`, unqualified and statically typed receiver calls;
- nested/outer helper methods;
- constructors and method references;
- interface dispatch;
- overload resolution by arity and available argument types;
- Java `instanceof T name` and switch-pattern bindings;
- contextual identifiers such as `record`;
- cycle detection;
- representative shortest call chains; and
- fail-closed unresolved edges only when they can materially connect a PE root
  to a guarded sink.

An unresolved relevant edge becomes `PE_REACHABILITY_UNKNOWN`; it cannot be
baselined.

## Exact published reachability baseline

The exact checked-in baseline is:

```text
tools/java_local_range_pe_reachability_baseline.json
schema=protos-local-range-pe-reachability-baseline-v1
entries=38
```

Its published distribution is:

```text
PE_REACHABLE_RISK=24
PE_REACHABLE_PROVEN_CONSTANT=4
BOUNDARY_CUT=6
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0
```

No wildcard entries or UNKNOWN suppressions are present.

## Important mixed-path result

A sink is not considered safe merely because one caller provides a constant
ordinal.

For example, both indexed operations inside
`ProtosFrameLexicalBindingAuthority.createFrameBackedBindingAt(int,Object)`
have a constant path:

```text
ProtosSemanticBytecodeRootNode.CreateCurrentIndexedLocalSlot.perform
 -> ProtosBytecodeRootNode.createIndexedCurrentLocalSlot
 -> ProtosFrameLexicalBindingAuthority.createFrameBackedBindingAt
 -> LocalRangeAccessor.{isCleared,setObject}
```

whose index provenance is:

```text
PE_CONSTANT_FROM_OPERATION
```

but they also have PE-reachable runtime-operand paths through:

```text
ProtosSemanticBytecodeRootNode.BindClosureIndexedParameter.perform
 -> ProtosBytecodeRootNode.bindIndexedClosureParameter
 -> ProtosFrameLexicalBindingAuthority.createFrameBackedBindingAt
 -> LocalRangeAccessor.{isCleared,setObject}
```

Therefore both sinks remain:

```text
PE_REACHABLE_RISK
```

This distinction is one of the main reasons for maintaining reachability and
local provenance as separate dimensions.

## Current risk families exposed by the guard

The 24 PE-reachable risk sinks include several different mechanisms rather than
one single source location:

### Runtime or otherwise unproven ordinal parameters

Examples include:

```text
ProtosBytecodeRootNode.createCurrentFrameBinding
ProtosFrameLexicalBindingAuthority.hasFrameBackedBindingAt
ProtosFrameLexicalBindingAuthority.readFrameBackedBindingAt
ProtosFrameLexicalBindingAuthority.assignFrameBackedBindingAt
ProtosFrameLexicalBindingAuthority.createFrameBackedBindingAt
ProtosInlineCallbackFrameBindings.create
```

The representative PE paths include ordinary operation runtime operands and
unproven call arguments.

### Runtime-name-derived ordinal lookup

Examples include:

```text
ProtosFrameLexicalBindingAuthority.containsBinding
ProtosFrameLexicalBindingAuthority.readBinding
ProtosFrameLexicalBindingAuthority.appendBindingsTo
ProtosInlineCallbackFrameBindings.admitsCapturedAccess
```

where the local index provenance comes from runtime `offsetOf(name)` lookup.

### Loop-derived ordinal access

Examples include:

```text
ProtosFrameLexicalBindingAuthority.ensureGeneralEstablishmentOrder
ProtosFrameLexicalBindingAuthority.adoptPresentFrameBackedBindings
ProtosFrameLexicalBindingAuthority.isEmpty
ProtosFrameLexicalBindingAuthority.appendBindingsTo
ProtosInlineCallbackFrameBindings.durableActivation
```

where the index is a loop induction variable not structurally proven
PE-constant by the static model.

These are risk families for subsequent analysis; this tooling publication does
not claim that every site requires the same product repair.

## Boundary and non-reachable results

Six sinks are cut from PE by a boundary:

```text
BOUNDARY_CUT=6
```

including the two accesses in `putBinding` itself and four accesses in
`bindingsSnapshot` reachable only through a boundary in the modeled graph.

Four sinks have no path from any of the modeled Bytecode DSL PE roots:

```text
NOT_PE_REACHABLE=4
```

including three `removeBinding` accesses and the
`recomputeLastCompactFrameOrdinal` loop reachable only from that removal path.

The checker explicitly documents that its PE roots are Bytecode DSL operations;
`NOT_PE_REACHABLE` is not a claim about arbitrary other Truffle entry points.

## Validation

The maintainer reported all local tests PASS.

The dedicated guard run completed with:

```text
TOTAL_LOCAL_RANGE_SINKS=38

PROVEN_CONSTANT_OPERAND=4
DIRECT_CONSTANT=0
TRUFFLE_BOUNDARY=2
RUNTIME_NAME_DERIVED=11
LOOP_INDEX=12
METHOD_PARAMETER=9

UNKNOWN=0
PROVEN_SAFE=6
BASELINED_RISKS=32
NEW_UNBASELINED_RISKS=0
STALE_BASELINE_ENTRIES=0

PE_REACHABLE_PROVEN_CONSTANT=4
PE_REACHABLE_RISK=24
BOUNDARY_CUT=6
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0

PE_ROOTS=401
CALL_GRAPH_METHODS=4786
CALL_GRAPH_SITES=27716
CALL_GRAPH_AMBIGUITIES=0
UNPARSED_FILES=0

NEW_UNBASELINED_REACHABILITY=0
REACHABILITY_DRIFT=0
STALE_REACHABILITY_ENTRIES=0

local-range-pe-guard=PASS
```

The checker self-test suite also passed with 39 tests.

A PASS means the source inventory, local baseline, reachability analysis and
exact reachability baseline are internally consistent. It does **not** mean the
24 PE-reachable risks are proven safe or that PERF030 is fixed.

## Runtime-cost property

The guard remains opt-in:

```text
make test-local-range-pe-guard
```

and is not wired into:

```text
make test
make test-java
make test-protos
make check
make verify
```

It remains Python-standard-library-only and does not launch Maven, Graal,
Protos, IGV/JFR or network operations.

## Coordination

```text
PERF030_G=COMPLETE
STATIC_PE_REACHABILITY_GUARD=PUBLISHED

PRODUCT_RUNTIME_CHANGED=NO
SEMANTIC_CHANGE=NO

TOTAL_LOCAL_RANGE_SINKS=38
PE_REACHABLE_RISK=24
PE_REACHABLE_PROVEN_CONSTANT=4
BOUNDARY_CUT=6
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0

PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
NEW_FORMAL_ISSUE_REQUIRED=NO

NEXT_WORK=PERF030-H investigation to classify the 24 PE-reachable risks into complete repair families, distinguish true contract violations from conservative static risks, and select bounded product slices without chasing individual Graal Pi node IDs
```

AI assistance: this durable evidence record was drafted with ChatGPT from the
published PERF030-G revision, the exact checked-in reachability baseline, live
GitHub coordination, and the maintainer-reported local validation result.
