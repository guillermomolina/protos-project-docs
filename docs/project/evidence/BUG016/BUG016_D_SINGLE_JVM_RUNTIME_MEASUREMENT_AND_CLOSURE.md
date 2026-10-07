# BUG016-D — single-JVM runtime measurement, publication reconciliation, and closure

## Scope

This record closes the causal investigation for `guillermomolina/protos#797`
(**BUG016 — TEST009 compilation harness serializes parallel Test Tool work**).

BUG016-D is a bounded runtime measurement supplied by the maintainer from the
current published Protos checkout. It does not introduce a product change.
Its purpose is to identify why one TEST009 diagnostic JVM remains near one CPU
core for minutes even when only one exact logical Case is selected.

This record also reconciles an important chronology change: BUG016-B was first
measured and rejected as a performance repair while still local, but was later
published as diagnostic sharding infrastructure before BUG016-D ran. The
historical BUG016-B evidence remains a faithful snapshot of the state when it
was written; this record supplies the later publication fact rather than
rewriting that history.

## Evidence identities

```text
FORMAL_WORK_ITEM=BUG016
GITHUB_ISSUE=guillermomolina/protos#797
SLICE=BUG016-D
SLICE_TYPE=BOUNDED_RUNTIME_MEASUREMENT

MEASURED_PROTOS_HEAD=5a3c5e4a274dee3b815746a21008c4cbedf4c1c1
MEASURED_VERSION=0.3.211-SNAPSHOT
PUBLISHED_HEAD_MESSAGE=BUG016-B: shard Truffle diagnostics by logical Case

PRODUCT_CHANGE_IN_BUG016_D=NONE
HARNESS_CHANGE_IN_BUG016_D=NONE
POM_CHANGE_IN_BUG016_D=NONE
CHANGELOG_CHANGE_IN_BUG016_D=NONE
```

## BUG016-B publication reconciliation

The earlier BUG016-B checkpoint recorded:

```text
BUG016_B_PUBLICATION=REJECTED
BUG016_B_PRODUCT_COMMIT=NONE
```

That was true when the checkpoint was produced. The repository subsequently
advanced and published the sharding machinery at:

```text
BUG016_B_PUBLISHED_REVISION=5a3c5e4a274dee3b815746a21008c4cbedf4c1c1
BUG016_B_PUBLISHED_VERSION=0.3.211-SNAPSHOT
```

The published implementation:

- discovers authoritative logical CaseRefs through `--list-cases`;
- runs each CaseRef in one diagnostic JVM;
- bounds concurrent JVMs through `--shard-workers`;
- preserves one log per shard;
- aggregates exact-once coverage and failure state fail-closed; and
- leaves the strict `check` gate unchanged.

This publication does not invalidate the BUG016-B performance falsification:
multiple JVMs can overlap, but every JVM still pays the same large pre-Case
diagnostic compilation cost.

## Exact focal measurement

The maintainer rebuilt the package for the measured HEAD and used the historical
focal file:

```text
protos/tests/conformance/call/closure-call-and-return.protos
```

Authoritative `--list-cases` discovery completed in approximately 4.0 seconds
and returned three Cases.

BUG016-D selected the first current CaseRef, displayed as:

```text
plain closure call
```

The complete opaque ref was retained locally in:

```text
target/truffle-compilation/bug016-d/case_ref.txt
```

The JVM used `--jobs 1` and exactly the current HEAD
`DIAGNOSTIC_OPTIONS`, unchanged:

```text
engine.CompileImmediately=true
engine.BackgroundCompilation=false
engine.CompilationFailureAction=Print
engine.TraceCompilation=true
compiler.TracePerformanceWarnings=all
compiler.TraceMethodExpansion=truffleTier
compiler.MethodExpansionStatistics=truffleTier
compiler.TraceNodeExpansion=truffleTier
compiler.NodeExpansionStatistics=truffleTier
compiler.TraceInlining=true
```

Output was still passed through a pipe. The measurement added only external
observability.

Local measurement artifacts were retained under:

```text
target/truffle-compilation/bug016-d/
  bug016d.jfr
  dump-T10.txt
  dump-T30.txt
  dump-T50.txt
  top-T*.txt
  diagnose.log
  command.sh
  case_ref.txt
```

Those `target/` artifacts are local diagnostic evidence and are not committed
to the project-record repository.

## Thread-state evidence

JFR used the JDK `profile` settings for 45 seconds. JVM thread dumps and
per-thread CPU samples were captured around T+10, T+30 and T+50 seconds.

Observed:

| Snapshot | guest `protos-cli-guest` | TruffleCompilerThreads | RUNNABLE compiler workers | hot compiler CPU |
|---|---|---:|---:|---|
| T+10 | WAITING in `CompilationTask.awaitCompletion -> FutureTask.get` | 6 | 1 primary | ~58% + ~40% during task handoff |
| T+30 | same wait chain | 6 | 1 | ~95% in one worker |
| T+50 | same wait chain | 6, pool renewed | 1 primary | ~63% + ~37% during task handoff |

Machine CPU remained approximately 77-90% idle.

No logger/writer thread had material CPU.

Therefore:

```text
GUEST_WAITING_FOR_COMPILATION=CONFIRMED
COMPILER_WORKERS_AVAILABLE=6
COMPILER_WORKERS_SIMULTANEOUSLY_RUNNABLE=1
MULTICORE_MACHINE_MOSTLY_IDLE=YES
```

## JFR evidence

The dominant native samples were:

```text
jdk.NativeMethodSample:
  2075 / 2079 samples
  -> TruffleToLibGraalCalls.doCompile
```

Only 11 `jdk.ExecutionSample` events represented ordinary Java execution.

Also:

```text
jdk.FileWrite=0
JavaMonitorEnter=0
```

This proves that the sampled CPU belongs overwhelmingly to compilation inside
libgraal, not guest Java execution, logger contention, or output writing.

JFR cannot split the native libgraal work precisely between partial evaluation,
Graal compilation, expansion-statistics work, and inlining tracing, so BUG016-D
does not fabricate such a split.

## Compilation sequence evidence

The partial log continued for approximately 368 seconds of wall time.

Within that interval:

```text
OPT_DONE=106
SUM_OPT_DONE_TIME≈273 s
OPT_DONE_SHARE_OF_WALL≈74%
INDIVIDUAL_COMPILATION_RANGE≈0.2..7.4 s
```

The evidence is therefore not one pathological giant compilation. It is a long
sequence of foreground compilations.

Repeated roots were observed, including:

```text
id=1860 -> 16 compilations
id=1575 -> 12 compilations
several others -> 3-4 compilations each
```

The partial trace also contained:

```text
OPT_INVALIDATIONS=38
  validRootAssumption local tags updated = 29
  Profiled Argument Types = 7

OPT_DEOPT=61
  uncommon trap = 54

OPT_FAILED_CODE_TOO_LARGE=9
  each discarded compilation approximately 8-9 s

PERF_WARN≈11111
LOG_SIZE≈231 MB
LOG_LINES≈969000
```

The 11,111 performance-warning lines explain much of the log volume, but the
absence of FileWrite or writer-thread pressure rejects output backpressure as
the dominant serialization cause.

## The selected Case body had not started

At T+50 the guest stack remained in the Test Tool bootstrap/module path:

```text
ProtosCli.runBundledTestTool
  -> executeModuleSource
  -> ProtosRootTaskExecution
  -> ...
```

No logical-Case attempt bridge or fresh selected-Case Process was present.

Therefore:

```text
SELECTED_CASE_BODY_REACHED=NO
DOMINANT_OBSERVED_COST_PHASE=TEST_TOOL_PRE_CASE_BOOTSTRAP_AND_PLAN_EXECUTION
```

This is the critical result for BUG016: sharding logical Cases cannot remove
the fixed compilation chain because every shard starts a fresh JVM and repeats
that same infrastructure before reaching its selected Case.

## Hypothesis verdicts

```text
H1_GUEST_WAITS_FOR_COMPILATION=CONFIRMED

H2_MULTIPLE_COMPILER_WORKERS_PROGRESS_CONCURRENTLY=REFUTED
  six workers exist, one is RUNNABLE during the expensive interval

H3_COMPILER_CPU_OWNER=CONFIRMED_AT_LIBGRAAL_LEVEL
  precise PE-vs-Graal-vs-expansion-vs-inlining split remains opaque to JFR

H3_OUTPUT_BACKPRESSURE=REFUTED
  FileWrite=0 and no writer/logger CPU or blocking evidence

H4_ONE_GIANT_COMPILATION=REFUTED
H4_MANY_FOREGROUND_COMPILATIONS_IN_SERIES=CONFIRMED
  amplified by invalidation/recompilation churn and code-too-large bailouts

H5_COST_PRECEDES_SELECTED_CASE=CONFIRMED
```

Final classification:

```text
DOMINANT_COST=SEQUENTIAL_FOREGROUND_COMPILATION_CHAIN
DOMINANT_COST_AMPLIFIERS=INVALIDATION_RECOMPILATION_CHURN+CODE_TOO_LARGE_BAILOUTS
DOMINANT_COST_LOCATION=TEST_TOOL_PRE_CASE_EXECUTION
BUG016_ROOT_CAUSE_MEASURED=YES
```

## Why `--jobs` and Case sharding do not fix it

With:

```text
CompileImmediately=true
BackgroundCompilation=false
```

the guest reaches an eligible root, submits one compilation, waits for that
compilation to complete, resumes, reaches another root, and repeats.

The measured JVM has six compiler workers but only one runnable compilation at
a time. The Test Tool cannot admit useful Case-level parallel work while its
single guest path is repeatedly blocked before the Case body is reached.

Case sharding can run several copies of this chain in several JVMs, which
explains the earlier approximately 106% CPU per JVM, but every shard repeats the
same fixed bootstrap compilation sequence. Sharding therefore creates physical
overlap without removing the dominant per-JVM cost.

## No further BUG016 measurement is justified

Two possible experiments were considered:

1. `BackgroundCompilation=true` in textual `diagnose`;
2. constraining compiler tracing/compilation to selected Case roots, for example
   with compiler filtering.

Neither is required to classify BUG016.

Both change what the current textual diagnostic means:

- foreground synchronous compilation would no longer be the observed topology;
- or infrastructure roots would no longer receive the same diagnostic
  treatment.

Those may be valid future TEST009 policy choices, but they are not evidence
needed to close this bug.

The PIPE-vs-file A/B is specifically rejected because BUG016-D measured no
write/backpressure signal.

```text
BUG016_BACKGROUND_COMPILATION_AB_REQUIRED=NO
BUG016_PIPE_AB_REQUIRED=NO
BUG016_FURTHER_LONG_DIAGNOSTIC_REQUIRED=NO
```

## Closure against BUG016 acceptance criteria

BUG016 explicitly allowed closure when the investigation proves why safe
parallelization cannot materially improve the representative diagnostic.

That condition is now satisfied:

- ordinary Test Tool parallelism remains independently healthy;
- harness sharding materializes real multi-JVM overlap;
- fail-closed exact Case attribution exists in the published harness;
- one-JVM runtime evidence identifies the first serialization boundary;
- six compiler workers exist but only one compilation is runnable;
- the expensive chain occurs before the selected Case;
- sharding merely duplicates that fixed chain; and
- changing foreground/background or root coverage would change diagnostic
  semantics rather than execution topology alone.

Therefore no additional BUG016 product repair is justified.

```text
BUG016_STATUS=COMPLETE
BUG016_CLOSURE_MODE=SAFE_PARALLELIZATION_CANNOT_REMOVE_DOMINANT_FIXED_CHAIN_WITHOUT_CHANGING_DIAGNOSTIC_MEANING
PRODUCT_REPAIR_REQUIRED=NO
```

## Return to TEST009

BUG016 does not own the remaining compilerability failures.

TEST009-J had already recorded unresolved permanent compilerability debt:

```text
CODE_INSTALLATION_TOO_LARGE=21
TOO_DEEP_INLINING=1
GLOBAL_COMPILATION_FAILURES=22
```

BUG016-D independently re-observed nine `code is too large` failures during
the pre-Case Test Tool compilation chain.

Those failures are therefore returned to their existing owner:

```text
NEXT_OWNER=TEST009/#795
NEXT_SCOPE=REMAINING_CODE_SIZE_AND_DEEP_INLINING_COMPILERABILITY_FAILURES
BUG016_FURTHER_MEASUREMENT=NONE
```

The repeated `validRootAssumption local tags updated` invalidation/recompile
churn is also retained as an independent runtime-performance observation. It is
not allocated as a new PERF issue by this closure record.

## Cleanup note

The maintainer reported that diagnostic JVM PID `200876` was still alive when
the measurement report was handed off because the planned cleanup block had not
yet run. A termination command was supplied in the handoff. This record does
not claim process termination because a post-cleanup result was not reported.

AI assistance: this durable record was drafted with ChatGPT from the
maintainer-supplied BUG016-D JFR/thread-dump/top/log analysis and live inspection
of the exact published Protos HEAD. No unreported local measurement result is
inferred.
