# TEST009 intake and PERF030 handoff evidence

Date: 2026-10-04

## Purpose

This record captures the bounded transition from the surviving PERF030 bailout investigation to TEST009's family-specific partial-evaluation guard work. It is non-normative project evidence; live status and hierarchy remain authoritative in GitHub Issues.

## Exact revisions

```text
PROTOS_REVISION=a5b00b7c1bb72bacd6ef36bc714a3c8e0972dac4
BENCHMARK_REVISION=e1cb1c30bc35dd06176cc5d314cd03bbe4867c26
PREVIOUS_PROJECT_RECORD_REVISION=5788e539fcf61495e5611be73b54aa536b3253bd
```

Relevant live work:

```text
PERF024=#756
PERF030=#784
TEST009=#795
TEST009_NATIVE_PARENT=#784
```

## PERF030-S publication

`guillermomolina/protos-benchmarks` published:

```text
REVISION=e1cb1c30bc35dd06176cc5d314cd03bbe4867c26
COMMIT=PERF030-S: add cache-keyed diagnostic JVM options
CHANGED_FILES=truffle/jvm_diagnostic.py,tests/test_jvm_diagnostic.py
```

The change adds a generic diagnostic JVM option surface:

```text
GENERIC_JVM_OPTION_SURFACE=DIAGNOSTIC_JVM_OPTIONS
DIAGNOSTIC_IDENTITY_SCHEMA=jvm-diagnostic-v4
```

Its contract is:

- unset or whitespace-only input normalizes to an empty list;
- non-empty input is normalized with Python `shlex.split`;
- the normalized token list is included in the diagnostic identity before the cache key is computed;
- options are inserted immediately after the `java` executable, preserving both direct `java ...` and `taskset ... java ...` command shapes;
- existing JFR/IGV JVM options and additional diagnostic options coexist;
- the future bailout-source capture can use `-Dpolyglot.engine.CompilationFailureAction=Print` without changing the workload or product.

PERF030-S is diagnostic infrastructure only. It does not itself remove or attribute a Protos bailout.

## Validation classification

The maintainer compared the full Python test suite in the working tree against a clean `git archive HEAD` export.

Observed summaries:

```text
HEAD_EXPORT_TESTS=345
HEAD_EXPORT_FAILURES=4
HEAD_EXPORT_ERRORS=11

WORKING_TREE_TESTS=357
WORKING_TREE_FAILURES=4
WORKING_TREE_ERRORS=6
```

The six focal failures that prompted the comparison were also present at HEAD. The working tree added twelve `test_jvm_diagnostic` tests without introducing a new failure attributable to PERF030-S. The additional HEAD-export-only errors were Git-identity-dependent tests running without `.git` metadata in the archive export.

Resulting classification:

```text
PERF030_S=COMPLETE
NEW_FAILURES_INTRODUCED_BY_PERF030_S=0
FULL_TESTS=FAIL_PREEXISTING_UNRELATED
PRODUCT_CHANGE=NO
SEMANTIC_CHANGE=NO
BENCHMARK_WORKLOAD_CHANGE=NO
BENCHMARK_MEASUREMENT_POLICY_CHANGE=NO
```

## Why the existing PERF030 guard was insufficient

The existing `tools/java_local_range_pe_guard.py` is a specialized static guard for one family: indexed `LocalRangeAccessor` operations and the provenance of their index/offset.

Its own contract states that a passing guard does not prove that every current site is partial-evaluation safe. It reasons about index provenance and conservative PE reachability, but the actual `LocalRangeAccessor` contract also requires other values to be partial-evaluation constants, including the accessor receiver and supplied `BytecodeNode`. Other Bytecode/Graal constant contracts may also exist outside this guard's scope.

Therefore the repaired PERF030 bailouts did not "come back". The project had eliminated some previously discovered bailout families while older, different compilerability debt remained outside the detection surface of the existing guard.

## TEST009 promotion

TEST009 was allocated as:

```text
TEST009=#795
TITLE=Family-specific PE bailout guards and aggregate checks
FAMILY=TEST
STATUS=READY
NATIVE_PARENT=#784
PRIORITY=INTENTIONALLY_UNSET
SEMANTIC_CHANGE=NO
```

Promotion from a PERF030 implementation slice was justified by:

```text
INDEPENDENT_CLOSURE=YES
MULTI_PUBLICATION_SCOPE=YES
GENERAL_REGRESSION_VALUE_BEYOND_PERF030=YES
```

TEST009 owns the durable testing/compiler-backend strategy rather than one bailout repair.

## Guard model

The agreed model distinguishes current cleanup from future regression prevention.

During current cleanup, each Python guard owns one mechanically distinguishable bailout family and scans the relevant production code broadly enough to expose pre-existing debt. A cleanup baseline means known old debt, not PE safety, and should shrink as repairs land.

After a family is clean, the same guard becomes regression prevention for newly introduced code.

Expected surfaces are conceptually:

```text
make check-local-range-index-pe
make check-local-range-operands-pe
make check-<other-family>
```

Every guard remains individually runnable. Guards whose measured runtime is sufficiently small and deterministic become prerequisites of aggregate `make check`; expensive runtime/compiler diagnostics remain explicit or periodic checks. Aggregate placement is decided from measured cost rather than assumption.

## First TEST009 implementation family

The first missing family is:

```text
LocalRangeAccessor receiver + BytecodeNode PE provenance/constancy
```

The implementation must reuse or extend the existing static-analysis machinery where useful, inventory the complete relevant production source, conservatively model PE reachability, and expose all old in-scope risks for that family. It must then repair all such in-scope risks iteratively rather than stopping after the first failing sink.

A newly exposed sink in the same family is not a new slice by itself. The implementation iteration continues until the family is clean or progress is genuinely blocked by an external dependency or explicit design/semantic decision.

## Current dependency state

```text
PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
TEST009_INITIAL_FAMILY_READY=YES
```

PERF030 remains the trigger and first consumer of TEST009. PERF024 remains blocked until the relevant PERF030 compilerability blocker is cleared.
