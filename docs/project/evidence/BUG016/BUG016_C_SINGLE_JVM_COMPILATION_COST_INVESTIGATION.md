# BUG016-C — single-JVM TEST009 diagnostic cost investigation

## Scope

This snapshot records the read-only BUG016-C investigation for
`guillermomolina/protos#797` (**BUG016 — TEST009 compilation harness
serializes parallel Test Tool work**).

It follows the BUG016-B sharding falsification checkpoint. BUG016-C did not run
commands, tests, benchmarks, profilers, Java, Maven, Python, or Truffle
diagnostics and did not modify product code. It inspected the published Protos
source and the exact pinned GraalVM/Truffle upstream source to locate the first
source-proven serialization boundary inside one diagnostic JVM.

This record is non-normative project evidence. It does not change Protos
semantics, Test Tool semantics, or the TEST009 strict-gate contract.

## Evidence identities

```text
FORMAL_WORK_ITEM=BUG016
GITHUB_ISSUE=guillermomolina/protos#797
SLICE=BUG016-C
SLICE_TYPE=INVESTIGATION
EXECUTION_ALLOWED=NO

BUG016_C_ANALYZED_PROTOS_REVISION=49dc0a4b4e70a04f7ce9d05a078b31a4bdddaa46
CURRENT_PROTOS_HEAD_AT_PUBLICATION=a85d9ca846d7408f869916a59b72f02ee12b9222
INTERVENING_PRODUCT_CHANGE=BUG017_ONLY
RELEVANT_TEST009_TOOL_TRUFFLE_SURFACES_CHANGED_SINCE_ANALYZED_REVISION=NO

GRAAL_UPSTREAM=oracle/graal
GRAAL_TAG=vm-25.4.4.1.1
GRAAL_COMMIT_OBSERVED=088451edab2666162ae265f9994561243b57df87

OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
STANDARD_LIBRARY_CHANGE=NO
PRODUCT_CHANGE=NONE
```

The single intervening Protos commit between the analyzed revision and current
HEAD changes only `CHANGELOG.md`, `pom.xml`,
`ProtosParallelRuntime.java`, and `ProtosParallelExecutionTest.java`.
It does not change the TEST009 gate, Test Tool, Polyglot host/context path, or
the Graal/Truffle code inspected by BUG016-C.

## Established BUG016-B observation

BUG016-B had already demonstrated that three distinct exact logical Cases could
be materialized as three concurrent diagnostic JVMs while each JVM remained at
approximately one CPU core:

```text
PID     ELAPSED  CPU%  RSS_KiB
184880  01:30    106   682252
184881  01:30    106   709624
184882  01:30    106   765920
```

That falsified the hypothesis that Case-level contention inside one Test Tool
JVM was the dominant reason for the historical ~12-minute diagnostic. No final
wall time or aggregate PASS/FAIL was inferred from that aborted run.

The durable preceding evidence is:

```text
docs/project/evidence/BUG016/BUG016_TEST009_DIAGNOSTIC_SHARDING_FALSIFICATION.md
```

## Current one-JVM execution graph

The published Protos source establishes this ownership path:

```text
tools/truffle_compilation_gate.py diagnose
  -> tools/truffle_jvm_launch.py
  -> packaged JVM ProtosCli test
  -> ProtosCli.runBundledTestTool()
  -> one Test Tool Session
       -> ProtosPolyglotRuntimeHost.open()
            -> one shared Engine
       -> one Test Tool Process / Polyglot Context
       -> tools/test/Main.protos
            -> imports Tool modules
            -> resolves complete suite/corpus bindings
            -> materializes every current TestPlan
            -> applies source selectors
            -> discovers suite-native logical Cases
            -> intersects exact --case selection
            -> ProtosTestLogicalCaseExecutionFacility
                 -> protos-test-exact-* carrier
                 -> ProtosTestLogicalCaseAttemptBridge.execute()
                      -> lazy shared Core Prelude
                      -> fresh Process
                      -> fresh Polyglot Context on the same Engine
                      -> selected source rematerialization
                      -> signature validation / exact selector
                      -> Test.call()
                      -> selected Case body
```

A focal exact Case therefore does not imply that the JVM only executes and
compiles the final trivial Case body. The Test Tool performs fixed bootstrap,
suite resolution, TestPlan materialization, and discovery work before the
selected Case body is invoked.

Relevant Protos files:

- `tools/truffle_compilation_gate.py`
- `tools/truffle_jvm_launch.py`
- `src/main/java/com/guillermomolina/protos/cli/ProtosCli.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosPolyglotRuntimeHost.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosPolyglotExecutionContext.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosLanguage.java`
- `src/main/java/com/guillermomolina/protos/cli/ProtosTestToolAsyncExecutionScope.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosTestLogicalCaseExecutionFacility.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosTestLogicalCaseAttemptBridge.java`
- `protos/tools/test/Main.protos`

## Exact diagnostic option surface

The current TEST009 textual diagnostic enables:

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

The strict gate additionally uses
`compiler.TreatPerformanceWarningsAsErrors=all`; the expansion and inlining
options above are diagnostic-only and are not required by the strict gate's
fail-closed compilerability authority.

## First source-proven serialization boundary

The exact GraalVM/Truffle 25.4.4.1.1 source establishes:

```text
EngineData
  compileImmediately = true
  -> multiTier = false
  -> interpreter call threshold = 0
  -> interpreter call+loop threshold = 0

executed OptimizedCallTarget
  -> compile(...)
  -> BackgroundCompileQueue.submitCompilation(...)
  -> CompilationTask assigned to Truffle compiler executor
  -> maybeWaitForTask(task)
  -> engine.backgroundCompilation == false
  -> OptimizedTruffleRuntime.finishCompilation(..., false)
  -> uninterruptibleWaitForCompilation(task)
  -> CompilationTask.awaitCompletion()
  -> Future.get()
```

The guest/carrier does not perform the compilation itself. It waits until the
compiler worker completes the submitted task.

This establishes a causal topology in which sequential discovery of executable
roots can produce:

```text
guest reaches root A
  -> submit A
  -> one TruffleCompilerThread compiles A
  -> guest waits

guest resumes and reaches root B
  -> submit B
  -> one TruffleCompilerThread compiles B
  -> guest waits

guest resumes and reaches root C
  -> ...
```

even when the runtime owns multiple compiler threads.

## Compiler pool versus runnable work

`BackgroundCompileQueue` creates a fixed compiler executor. With the default
negative `CompilerThreads`, the worker count scales with
`Runtime.availableProcessors()` and is capped at 16.

The source explicitly gives representative examples such as six compiler
threads for 16 available processors and ten for 32.

Therefore:

```text
COMPILER_THREADS_AVAILABLE=MULTIPLE
```

does not imply:

```text
COMPILATION_TASKS_RUNNABLE_SIMULTANEOUSLY=MULTIPLE
```

If synchronous guest progress reveals only the next root after the previous
compilation finishes, the queue can contain one runnable compilation while the
remaining compiler workers are idle. That is mechanically compatible with the
BUG016-B observation of approximately one CPU core per diagnostic JVM.

## Diagnostic amplification inside each compilation

The exact upstream source also proves additional work in the full textual
diagnostic.

### Method and node expansion

When the expansion options are enabled, `ExpansionStatistics` performs work
after compiler stages. At the Truffle tier it can:

1. schedule the graph;
2. iterate graph nodes;
3. construct a method expansion tree;
4. group it by Truffle node;
5. accumulate method statistics;
6. accumulate node-class statistics;
7. accumulate specialization statistics;
8. merge aggregate maps;
9. render expansion trees; and
10. produce final histograms at compiler shutdown.

`combineExpansionStatistics(...)` is synchronized.

Enabling expansion statistics also causes the compiler option processing path
to enable node source positions.

### Inlining trace

`CallTree.trace()` walks the inlined guest call tree and calls
`runtime.logEvent()` for the relevant nodes. `TraceInlining=true` is cheaper
than `TraceInliningDetails=true`, but still adds traversal, property
construction, formatting, and logging per compilation.

### Logging

`TruffleCompilerRuntime.logEvent()` formats event text before forwarding it to
the logger. The Polyglot `StreamLogHandler.publish()` implementation is
synchronized and formats then writes the record on the producer thread.

This makes logging a real possible amplifier, but BUG016-C does not have
measurement evidence that logging or its lock is the dominant many-minute cost.

## Python pipe hypothesis

The Protos launcher uses:

```text
stdout=PIPE
stderr=STDOUT
subprocess.run(...)
```

Python `subprocess.run()` is implemented through communication with the child
and drains captured output while the process is running. Therefore the specific
hypothesis that Python waits for process termination before reading the pipe is
not supported.

Pipe backpressure remains mechanically possible if the producer generates output
faster than the reader can drain it, but BUG016-C has no evidence that this is
the dominant cost.

## Cost classification

| Cost | Scope |
|---|---|
| JVM/class loading/Graal initialization | per JVM |
| RuntimeHost / Engine | per Engine |
| compiler queue/pool initialization | per Engine |
| Test Tool bootstrap and fixed resolver/prelude preparation | per JVM / Tool invocation |
| Test Tool Process and Context | per Process / Context |
| complete suite binding resolution and TestPlan materialization | per Tool invocation |
| exact source discovery/selection | selection-dependent |
| selected Case fresh Process and Context | per Case |
| selected source rematerialization | per Case |
| selected Case body | per Case |
| Truffle compilation + PE/Graal | per compiled root |
| expansion tree/stat work | per compiled root |
| inlining trace | per compiled root |
| log formatting/write | per trace event |
| expansion histogram rendering | final aggregation |

This explains why a trivial Case body can still sit behind a large fixed
diagnostic workload.

## Revised BUG016-A classification

BUG016-A correctly identified real shared diagnostic synchronization, but
BUG016-B and BUG016-C narrow its causal status.

```text
BUG016_A_ORIGINAL_CLASSIFICATION=PARTIALLY_REVISED
```

The strongest source-supported model is now:

```text
primary structural mechanism:
  CompileImmediately
  + BackgroundCompilation=false
  + sequential discovery of executed roots
  + substantial fixed Test Tool/bootstrap execution

diagnostic amplification:
  PE graph/source metadata
  + method/node expansion traversal/statistics
  + TraceInlining
  + formatting/logging
```

The source does not establish which of those CPU consumers dominates the
historical many-minute wall time.

## Causal candidates

### H1/H5 — synchronous dependent compilation of a large fixed Tool/bootstrap surface

Strength: **very high**.

This is the best source-supported explanation for the combination of a trivial
Case, large fixed JVM cost, and approximately one-core utilization.

### H2 — each individual PE/Graal compilation is effectively one-worker work

Strength: **high**.

A `CompilationTask` is executed by one compiler worker through
`doCompile()`, Truffle tier / partial evaluation, Graal graph compilation, and
installation. A single expensive root can therefore occupy roughly one CPU
while the guest waits.

### H3 — expansion diagnostics materially amplify every root

Strength: **high as an amplifier, not proven dominant**.

### TraceInlining

Strength: **medium**.

### Logging/formatting

Strength: **medium as an amplifier, not proven dominant**.

### H4 — Python pipe backpressure

Strength: **low with current evidence**.

## Root-cause status

BUG016-C can identify the first serialization boundary and explain how a
multi-threaded compiler runtime can still expose approximately one-core process
use, but it cannot identify the dominant many-minute CPU consumer without a
bounded runtime observation.

Therefore:

```text
BUG016_ROOT_CAUSE_SOURCE_PROVEN=NO
```

A product or harness repair is not yet justified.

## Next executable slice

The next slice must be executable because the missing discriminator is runtime
evidence. Under the project workflow, it is not another no-command
investigation.

```text
NEXT_SLICE=BUG016-D
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_CHARACTER=BOUNDED_MEASUREMENT_ONLY_FIRST
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
PRODUCT_CHANGE_AUTHORIZED_BEFORE_MEASUREMENT=NO
```

The first measurement should preserve the current exact Case and full
`DIAGNOSTIC_OPTIONS` and add only a bounded 30–60 second CPU/thread-state
recording. It must answer:

- is the guest carrier waiting in `CompilationTask.awaitCompletion()`?
- how many `TruffleCompilerThread` workers are RUNNABLE?
- what stack owns most compiler CPU:
  `PartialEvaluator`/Graal phases, `ExpansionStatistics`,
  `CallTree.trace`/formatting/logging, or output write/backpressure?

Stop as soon as the thread-state and dominant stack classification is stable.
Do not wait for a full ~10–12 minute diagnostic merely to collect this
discriminator.

Only if that first measurement requires a second discriminator should BUG016-D
change one variable:

1. toggle only `BackgroundCompilation=false -> true` for at most 60 seconds
   to distinguish foreground request/wait serialization from one intrinsically
   expensive compilation; or
2. change only the merged output sink from PIPE to a regular file for at most
   30–60 seconds if the first measurement actually shows meaningful logger/write
   blocking.

No product patch should be attempted until one of those measurements identifies
a bounded repair target.

## Current state

```text
BUG016_STATUS=IN_PROGRESS
BUG016_B_SHARDING_PERFORMANCE_HYPOTHESIS=FALSIFIED
BUG016_C_SOURCE_ANALYSIS=COMPLETE
FIRST_SERIALIZATION_BOUNDARY=OptimizedCallTarget.maybeWaitForTask -> OptimizedTruffleRuntime.finishCompilation(..., false) -> CompilationTask.awaitCompletion
DOMINANT_COST_CLASS=NOT_YET_MEASURED
BUG016_ROOT_CAUSE_SOURCE_PROVEN=NO
NEXT_SLICE=BUG016-D
```

AI assistance: this evidence record was drafted with ChatGPT from live GitHub
inspection of the exact Protos revision and the pinned GraalVM/Truffle
25.4.4.1.1 source. BUG016-C itself executed no project/runtime commands and
inferred no missing wall-time result.
