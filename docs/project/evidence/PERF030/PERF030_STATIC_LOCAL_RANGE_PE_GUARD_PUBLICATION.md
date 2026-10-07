# PERF030 — static LocalRangeAccessor PE-index guard publication

## Scope

This record preserves the published PERF030-F repository-tooling slice that
systematically inventories indexed `LocalRangeAccessor` access in Protos
production Java.

The owning live work item is
`guillermomolina/protos#784` (**PERF030 — Make frame-local creation ordinal
PE-constant**). PERF024 / #756 remains blocked from cross-runtime physical graph
interpretation while the current Protos compilerability blocker remains
unresolved.

This is non-normative tooling/publication evidence. It does not alter Protos
language semantics, product runtime behavior, benchmark behavior, or external
compiler-acceptance evidence.

A passing guard means only that the current inventory is complete according to
the checker, `UNKNOWN=0`, and the checked-in exact baseline matches current
source. It does **not** mean that the baselined sites are PE-safe or that
PERF030 is fixed.

## Published authority

```text
SLICE=PERF030-F
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

PROTOS_REVISION=5f97539e1ae0337ebb82bd4e02d659be0e1cc11c
COMMIT_SUBJECT=PERF030-F: add opt-in static LocalRangeAccessor PE-index guard

PRODUCT_RUNTIME_CHANGE=NO
OBSERVABLE_LANGUAGE_SEMANTIC_CHANGE=NO
SPEC_CHANGE=NO
BENCHMARK_CHANGE=NO

MAINTAINER_REPORTED_LOCAL_TESTS=PASS
```

The published product revision modifies only:

```text
Makefile
tools/java_local_range_pe_guard.py
tools/java_local_range_pe_guard_baseline.json
tools/test_java_local_range_pe_guard.py
```

No `src/main/java`, `src/test/java`, `protos/lib`, specification, or
benchmark source is changed by this slice.

## Trigger

The unchanged external `primitive-closure-call` compiler diagnostics preceding
this tooling slice preserved guest correctness but did not satisfy compiler
acceptance.

The current JFR evidence contained two tier-1 permanent compilation failures:

```text
PERMANENT_FAILURES=2

Partial evaluation did not reduce value to a constant:
  36218|Pi

Partial evaluation did not reduce value to a constant:
  8231|Pi
```

Both corresponding compilation events had `success=false` and
`compiledCodeSize=0`; the paired IGV run produced no BGV files.

PERF030-F deliberately stops treating those numeric Graal node tokens as stable
source identities. Instead it establishes a static repository guard for the
whole known class of indexed `LocalRangeAccessor` accesses whose index must be
PE-constant when reached by partial evaluation.

## Published checker

The opt-in checker is:

```text
tools/java_local_range_pe_guard.py
```

It is Python-standard-library-only and scans:

```text
src/main/java/**/*.java
```

It does not run Maven, Java compilation, Graal, Protos, IGV/JFR, network access,
or one subprocess per source file.

The parser:

- lexes Java while dropping comments and preserving line numbers;
- keeps strings/characters/text blocks opaque;
- fails closed on unterminated comments/strings/text blocks and unbalanced
  brackets;
- identifies types, members, parameters and locals with lexical scope; and
- reports unsupported/ambiguous relevant receiver/provenance shapes as
  `UNKNOWN` rather than silently skipping them.

## Sink model

The checker recognizes indexed operations whose receiver resolves to a field,
parameter or local declared as `LocalRangeAccessor`.

Covered operations include:

```text
isCleared
clear
getObject / typed get*
setObject / typed set*
```

with arity validation so unrelated same-named calls are not counted as
`LocalRangeAccessor` sinks.

For every sink, the report retains source path/line, enclosing class/method,
receiver, operation, normalized index expression, classification and
classification rationale.

## Provenance classifications

The published checker distinguishes:

```text
PROVEN_CONSTANT_OPERAND
DIRECT_CONSTANT
TRUFFLE_BOUNDARY
RUNTIME_NAME_DERIVED
LOOP_INDEX
METHOD_PARAMETER
UNKNOWN
```

`PROVEN_CONSTANT_OPERAND` is deliberately strict: an index is proven only
when the enclosing method is a Bytecode DSL `@Specialization` in an
`@Operation` class and the parameter position/name matches an
`int.class @ConstantOperand`. Merely naming a variable `ordinal` does not
prove it.

Simple local aliases are followed. Reassigned locals/parameters or unsupported
provenance become `UNKNOWN`.

## Exact current inventory

The current scan covers six production Java files mentioning
`LocalRangeAccessor`:

```text
ProtosFrameLexicalBindingAuthority.java   26 sinks
ProtosInlineCallbackFrameBindings.java     6 sinks
ProtosSemanticBytecodeRootNode.java        4 sinks
ProtosBytecodeRootNode.java                2 sinks
CanonicalToBytecodeLowerer.java            0 sinks
ProtosFrameLexicalLayout.java              0 sinks
```

The exact classification totals at the published revision are:

```text
TOTAL_LOCAL_RANGE_SINKS=38

PROVEN_SAFE=6
  PROVEN_CONSTANT_OPERAND=4
  TRUFFLE_BOUNDARY=2
  DIRECT_CONSTANT=0

KNOWN_BASELINED_RISK=32
  LOOP_INDEX=12
  METHOD_PARAMETER=9
  RUNTIME_NAME_DERIVED=11

UNKNOWN=0
NEW_UNBASELINED_RISKS=0
```

The 32 non-proven sites are captured explicitly in:

```text
tools/java_local_range_pe_guard_baseline.json
schema=protos-local-range-pe-guard-baseline-v1
```

Each baseline entry is exact and includes path, class, method signature,
operation, index expression, classification, occurrence and rationale.

Wildcards, duplicate entries, safe classifications, `UNKNOWN`, stale entries,
new unbaselined risks, and changed provenance fail the guard.

The baseline is therefore a list of **known risks**, not an allowlist claiming
those sites are correct.

## Self-tests

The checker self-tests cover the required safety and failure modes, including:

- Bytecode DSL constant-operand proof;
- ordinary method parameter not being accepted as constant;
- runtime `offsetOf(name)` provenance;
- one-level local alias propagation;
- loop-index classification;
- `@TruffleBoundary` classification;
- excluding unrelated receivers;
- ambiguous provenance becoming `UNKNOWN`;
- new sink absent from the baseline failing;
- stale baseline entry failing;
- source line-number preservation;
- changed provenance failing;
- wildcard/safe baseline entries failing; and
- unparsable source failing closed.

## Opt-in Make target

The published standalone target is:

```text
make test-local-range-pe-guard
```

It executes:

1. checker self-tests;
2. current-source baseline verification; and
3. deterministic report generation at:

```text
target/local-range-pe-guard-report.json
```

The target is intentionally **not** wired into:

```text
make test
make test-java
make test-protos
make check
make verify
```

This keeps the initial adoption opt-in and allows future checks to be added to
this dedicated static guard without lengthening the ordinary test suite by
default.

## Validation

The maintainer reported all requested local validation/tests PASS after
publication.

```text
MAINTAINER_REPORTED_LOCAL_TESTS=PASS
CHECKER_SELF_TESTS=PASS_REPORTED_BY_MAINTAINER
CURRENT_SOURCE_BASELINE_CHECK=PASS_REPORTED_BY_MAINTAINER

UNKNOWN=0
NEW_UNBASELINED_RISKS=0
BASELINE_CONSISTENT=YES
```

This validation establishes only the integrity of the inventory/guard.

It explicitly does **not** establish:

```text
ALL_LOCAL_RANGE_ACCESS_PE_SAFE=YES
PERF030_FIXED=YES
EXTERNAL_IGV_JFR_ACCEPTANCE=PASS
```

Those claims remain false/unproven until later product/compilerability work
eliminates or proves the relevant PE-reachable risk class and unchanged external
diagnostics pass.

## Coordination

```text
PERF030_F=COMPLETE
STATIC_LOCAL_RANGE_PE_GUARD=PUBLISHED

PRODUCT_RUNTIME_CHANGED=NO
SEMANTIC_CHANGE=NO

TOTAL_LOCAL_RANGE_SINKS=38
PROVEN_SAFE=6
KNOWN_BASELINED_RISK=32
UNKNOWN=0

RUNTIME_NAME_DERIVED=11
LOOP_INDEX=12
METHOD_PARAMETER=9
PROVEN_CONSTANT_OPERAND=4
TRUFFLE_BOUNDARY=2

PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
NEW_FORMAL_ISSUE_REQUIRED=NO

NEXT_WORK=use the complete guard inventory to bound the PERF030 remediation of PE-reachable non-constant LocalRangeAccessor indices before another external acceptance attempt
```

AI assistance: this durable evidence record was drafted with ChatGPT from the
published PERF030-F commit, its exact checked-in baseline, live GitHub
coordination, and the maintainer-reported validation result.
