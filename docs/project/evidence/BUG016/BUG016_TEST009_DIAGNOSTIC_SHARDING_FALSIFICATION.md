# BUG016 — TEST009 diagnostic sharding falsification checkpoint

## Scope

This snapshot records the BUG016-B harness experiment for
`guillermomolina/protos#797` (**BUG016 — TEST009 compilation harness
serializes parallel Test Tool work**).

It is non-normative defect evidence. It does not define Test Tool semantics,
Truffle/Graal policy, or a new compiler architecture.

BUG016-B was an unpublished local implementation experiment. Its process-level
logical-Case sharding mechanism was exercised far enough to falsify the
performance hypothesis that motivated the slice, so the experiment was stopped
before publication rather than waiting for an already non-viable long-running
measurement to complete.

## Evidence identities

```text
FORMAL_WORK_ITEM=BUG016
GITHUB_ISSUE=guillermomolina/protos#797
FAMILY=BUG
ISSUE_STATUS_AT_RECORDING=IN_PROGRESS
ISSUE_PRIORITY_AT_RECORDING=P1

CURRENT_PUBLISHED_PROTOS_HEAD=49dc0a4b4e70a04f7ce9d05a078b31a4bdddaa46
LOCAL_UNPUBLISHED_BUG016_B_CANDIDATE=YES
LOCAL_CANDIDATE_COMMIT=NOT_PUBLISHED
LOCAL_RUNTIME_ARTIFACT_VERSION_OBSERVED=0.3.208-SNAPSHOT

TEST_TOOL_PREREQUISITE=TOOL009-G/#798
TOOL009_G_STATUS=COMPLETED
D185_STATUS=RATIFIED

OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
STANDARD_LIBRARY_CHANGE=NO
BUG016_B_PUBLISHED=NO
```

The current published Protos revision above is recorded only as the moving
repository state at evidence publication. The sharding candidate itself was
local and unpublished, so this record does not fabricate a product commit for
that candidate.

## Historical baseline

BUG016 was triggered by a TEST009 diagnostic baseline in which the ordinary
Test Tool parallel scheduler worked but the strict Truffle diagnostic did not
scale with `--jobs`:

```text
ordinary make test-protos:
  jobs=8
  1328 passed, 0 failed
  ~31 s
  effective CPU parallelism observed

TEST009 diagnostic:
  jobs=1 -> ~715.4 s
  jobs=8 -> ~721.7 s
  focal selection -> 3 passed, 0 failed
  effective CPU use observed as serial
```

BUG016-A therefore investigated the TEST009 harness rather than reopening the
ordinary Test Tool scheduler.

## TOOL009-G prerequisite

TOOL009-G / #798 later published authoritative logical-Case discovery and exact
selection:

```text
protos test --list-cases
protos test --case CASE_REF [--case CASE_REF ...]
```

This removed the previous blocker to a clean process-level sharding experiment:
the harness could consume opaque authoritative CaseRefs without parsing Protos
test sources or assuming one physical file equals one logical Case.

## BUG016-B local candidate

The unpublished BUG016-B candidate changed only the diagnostic execution
topology conceptually:

```text
authoritative Case discovery
  -> ordered CaseRefs
  -> one independent diagnostic JVM per CaseRef
  -> bounded worker pool
  -> per-shard log/report provenance
  -> deterministic aggregate
```

The diagnostic JVMs retained the existing heavy diagnostic options, including:

```text
CompileImmediately=true
BackgroundCompilation=false
CompilationFailureAction=Print
TraceCompilation=true
TracePerformanceWarnings=all
TraceMethodExpansion=truffleTier
MethodExpansionStatistics=truffleTier
TraceNodeExpansion=truffleTier
NodeExpansionStatistics=truffleTier
TraceInlining=true
```

The candidate did not redefine Test Tool `--jobs`, Case identity, D185
CaseRefs, or ordinary Test Tool scheduling.

## First run: wrong default scope

The first real run accidentally discovered the full default corpus rather than
the historical focal baseline. Discovery returned 1328 logical CaseRefs.

With three shard workers this would have launched 1328 diagnostic JVMs three at
a time. The run was stopped because it was not measuring the historical
three-Case baseline and would have been prohibitively expensive.

That run exposed an important performance clue before termination: independent
diagnostic JVMs were observed using approximately one CPU core each rather than
scaling across the 16-core host. A trivial Case remained active for many
minutes, suggesting a large per-JVM diagnostic/compilation cost independent of
logical Case body complexity.

Because the scope was wrong, this run is not a valid wall-time comparison.

## Correct focal scope

The historical three-Case scope was recovered as:

```text
--file protos/tests/conformance/call/closure-call-and-return.protos
```

The corrected run used:

```text
python3 tools/truffle_compilation_gate.py diagnose
  --timeout 3600
  --shard-workers 3
  --artifacts target/truffle-compilation
  --report target/truffle-compilation/diagnose-report.json
  --
  --jobs 1
  --file protos/tests/conformance/call/closure-call-and-return.protos
```

The process table approximately 90 seconds into the run showed exactly three
diagnostic JVMs, all started at the same time, each with the correct focal
`--file`, each with one distinct opaque D185 CaseRef, and each with
`--jobs 1`.

Observed diagnostic JVMs:

```text
PID     ELAPSED  CPU%  RSS_KiB
184880  01:30    106   682252
184881  01:30    106   709624
184882  01:30    106   765920
```

Approximate combined RSS at that checkpoint:

```text
2157796 KiB ~= 2.06 GiB
```

The Python coordinator itself used negligible CPU relative to the JVMs.

The three JVM command lines carried three different `--case v1....` values.
This is direct evidence that the corrected focal run had materialized the
intended one-Case-per-JVM topology.

## What this proves

The focal checkpoint proves:

```text
CORRECT_FOCAL_SCOPE=YES
DISTINCT_CASE_REFS=3
CONCURRENT_DIAGNOSTIC_JVMS=3
ONE_CASE_PER_JVM=YES
BOUNDED_PROCESS_SHARDING_MATERIALIZED=YES
PER_JVM_CPU_APPROX_ONE_CORE=YES
TEST_TOOL_JOBS_VALUE=1
```

The sharding mechanism therefore does create real inter-process overlap.

## What it does not prove

The corrected run was intentionally stopped early after the performance
hypothesis was already contradicted. Therefore this record does **not** claim:

- a completed wall time for the corrected three-Case run;
- a final shard PASS/FAIL aggregate;
- a measured speedup ratio;
- that the exact dominant internal Truffle/Graal stage is already known.

Those require the next investigation, not fabrication from an aborted run.

## Falsified BUG016-B hypothesis

BUG016-B was motivated by the hypothesis that the ~12 minute diagnostic was
dominated by multiple logical Cases contending inside one JVM, particularly
around shared Truffle diagnostic logging/statistics.

The new evidence contradicts that as the dominant explanation:

1. the three Cases were successfully split into three independent JVMs;
2. all three JVMs ran concurrently;
3. each JVM nevertheless used only about one CPU core;
4. earlier wrong-scope observation showed even one trivial exact Case could
   remain in the heavy diagnostic JVM for many minutes;
5. therefore Case-level process sharding multiplies the large per-JVM startup /
   compilation / diagnostic cost rather than eliminating it.

The shared logger/statistics synchronization found by BUG016-A may still exist
and may still contribute cost. The evidence here only establishes that it is
not sufficient as the dominant explanation for the observed ~12 minute wall
time.

## Publication decision

```text
BUG016_B_MECHANISM=FUNCTIONAL_ENOUGH_TO_TEST
BUG016_B_PERFORMANCE_HYPOTHESIS=FALSIFIED
BUG016_B_PUBLICATION=REJECTED
BUG016_B_PRODUCT_COMMIT=NONE
BUG016_REMAINS_OPEN=YES
```

The local candidate should not be published as the BUG016 performance repair.

## Next investigation boundary

The next slice must investigate the cost of **one diagnostic JVM executing one
exact logical Case**, not Test Tool scheduling.

Primary question:

> Why does one TEST009 JVM under `CompileImmediately` plus full diagnostic
> tracing use approximately one core and remain expensive for many minutes even
> when it executes only one exact logical Case?

The investigation should decompose elapsed time and CPU ownership across at
least:

- packaged JVM/Test Tool startup and discovery;
- guest execution before the focal Case body;
- synchronous Truffle compilation / partial evaluation;
- number and identity of compiled roots;
- method/node expansion statistics;
- inlining tracing;
- log formatting/output volume;
- compiler queue/thread activity;
- repeated compilation or compilation of Test Tool/runtime/stdlib roots;
- shutdown/final aggregation.

It should use the cheapest discriminating evidence first and avoid another
blind 10–12 minute full-trace run unless a specific measurement requires it.

The next slice is research/investigation only. It must not change product code
or publish the rejected BUG016-B candidate.

## Current state

```text
BUG016_STATUS=IN_PROGRESS
BUG016_B=STOPPED_UNPUBLISHED
BUG016_B_SHARDING_MECHANISM=OBSERVED_WORKING
BUG016_B_PERFORMANCE_HYPOTHESIS=FALSIFIED

NEXT_SLICE=BUG016-C
NEXT_SLICE_TYPE=INVESTIGATION
IMPLEMENTATION_REPOSITORY=NOT_APPLICABLE
EXECUTION_ALLOWED=NO
```

AI assistance: this evidence record was drafted with ChatGPT from the
maintainer-provided process table and command lines, the live BUG016/TOOL009-G
GitHub coordination, and exact current repository inspection. No unobserved
wall-time result was inferred.
