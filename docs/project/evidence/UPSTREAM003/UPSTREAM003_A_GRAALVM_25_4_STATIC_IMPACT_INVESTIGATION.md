# UPSTREAM003-A — GraalVM / Graal / Truffle 25.4 static impact investigation

Date: 2026-09-28

## Scope

This record retains the investigation-only result for UPSTREAM003 / guillermomolina/protos#732.

No Protos build, test, benchmark, dependency update, container update, or runtime execution was performed by this slice. Evidence came from public upstream sources and read-only inspection of current `guillermomolina/protos` and `guillermomolina/protos-benchmarks`.

## Evidence identity

```text
WORK_ITEM=UPSTREAM003/#732
SLICE=UPSTREAM003-A
TYPE=INVESTIGATION_ONLY

PRODUCT_REPOSITORY=guillermomolina/protos
INSPECTED_PRODUCT_REVISION=44690b1fc8c9aed023600c6d5731f969c4507e27

CURRENT_GRAALVM_TRUFFLE=25.3.4.1
CURRENT_JDK=25.0.4.1

CANDIDATE_GRAALVM_TRUFFLE=25.4.4.1.1
CANDIDATE_JDK_RELEASE_LINE=25.0.4.1
```

## Executive result

Static source evidence finds no required Protos source change before controlled execution.

The migration is nevertheless performance-sensitive because Graal/Truffle 25.4 changes guest-language inlining on exactly the direct-call surfaces currently exercised by the PERF010/PERF013/PERF014 line.

```text
STATIC_SOURCE_COMPATIBILITY=COMPATIBLE
BUILD_COMPATIBILITY=UNKNOWN_UNTIL_EXECUTION
RUNTIME_COMPATIBILITY=UNKNOWN_UNTIL_EXECUTION
PERFORMANCE_EQUIVALENCE=UNKNOWN_UNTIL_EXECUTION

SOURCE_CHANGES_REQUIRED_BY_STATIC_EVIDENCE=NO
CONTROLLED_PLATFORM_COMPARISON_REQUIRED=YES
```

Historical 25.3 evidence remains authoritative for the exact product, toolchain, host and protocol that produced it. It must not be relabelled as 25.4 evidence.

## Guest-language inlining change

### Upstream 25.3 behavior

At `oracle/graal@vm-25.3.4.1`, Truffle guest inlining derives a child direct-call frequency from runtime direct-call counters:

```text
OptimizedDirectCallNode.callCount
        /
target.getCallCount()
```

That relative frequency contributes to the child's root-relative frequency in the Truffle call tree.

### Upstream 25.4 behavior

At `oracle/graal@vm-25.4.4.1.1`, this direct-call-counter calculation is removed.

After partial evaluation, Truffle builds a Graal `ControlFlowGraph` with frequency computation enabled and obtains the local direct-call frequency from the relative frequency of the IR block containing the invoke. The frequency is restricted consistently with Graal priority-inlining frequency bounds and propagated through the guest-call subtree.

Therefore the effective relationship changes from:

```text
25.3:
runtime DirectCallNode observations
        -> guest inlining frequency

25.4:
PE result + Graal CFG + branch-frequency information
        -> guest inlining frequency
```

This is the implementation basis for upstream change GR-79418.

## Protos call-path relevance

Current Protos HEAD deliberately contains guarded direct/indirect call pairs. In particular, stable Closure invocation in `ProtosBytecodeRootNode.EnterClosureCall` can use a cached `DirectCallNode`, with an `IndirectCallNode` fallback.

That makes the following current workloads materially relevant to the 25.4 inlining change:

```text
closure-call=HIGH
method-call=HIGH
monomorphic-dispatch=HIGH
control/workload-control=MEDIUM
```

This is relevance for testing, not a prediction of improvement or regression.

The change can also plausibly alter previously observed compiler evidence such as inlining depth, expansion order, cutoff decisions, graph shape, or `Too deep inlining` observations. Historical 25.3 compiler evidence remains valid for 25.3, but equivalent 25.4 claims must be re-established.

## Branch-probability audit

Repository-wide read-only inspection of current Protos HEAD found no direct uses of:

```text
CompilerDirectives.injectBranchProbability
BranchProfile
ConditionProfile
ConditionProfile.createBinaryProfile
ConditionProfile.createCountingProfile
```

Classification:

```text
explicit injected numeric branch probabilities=UNLIKELY_RELEVANT
explicit BranchProfile use=UNLIKELY_RELEVANT
explicit ConditionProfile use=UNLIKELY_RELEVANT
Bytecode DSL generated control flow=POSSIBLY_RELEVANT
DSL specialization/guard-generated CFG=POSSIBLY_RELEVANT
```

The important distinction is that ordinary generated control flow is not explicit probability injection. It still matters because 25.4 derives guest-call frequencies from the resulting Graal CFG.

No Protos-side branch-probability correction is justified before execution evidence exists.

## Compiler-phase changes

25.4 enables or extends optimizer behavior relevant enough to retain in the controlled comparison:

- `PullThroughPhiPhase`: may expose branch-local type/value information by duplicating suitable floating operations across merge/phi boundaries.
- `DuplicationPhase`: may expose branch-specific optimization opportunities through controlled duplication/tail duplication.
- read elimination: extended for indexed-array reads and additional array/read patterns.
- additional low-tier read elimination opportunity after reads become fixed.

Current Protos dynamic dispatch, branch-heavy generated Bytecode DSL control flow, activation/argument/frame accesses and captured-local machinery can plausibly produce relevant IR patterns.

Static inspection cannot establish a benefit or regression.

```text
PullThroughPhiPhase=MEDIUM_RELEVANCE
DuplicationPhase=MEDIUM_RELEVANCE
indexed_array_read_elimination=MEDIUM_RELEVANCE
additional_low_tier_read_elimination=MEDIUM_RELEVANCE
```

## Bytecode DSL changes

### StackValue

25.4 adds `StackValue`, `BindStackValue`, `LoadStackValue` and `StoreStackValue` for temporary operand-stack values.

Current Protos has backend-private temporaries and local-backed structured-control state where this mechanism could be investigated later.

Classification:

```text
StackValue=POTENTIAL_FUTURE_RELEVANCE
ACTION_NOW=NO
```

Adopting StackValue inside the platform comparison would mix a Protos optimization with the upstream variable and is explicitly excluded.

### Compressed source information

25.4 Bytecode DSL source information is compressed by default through `@GenerateBytecode(enableCompressedSources = true)`.

Current Protos uses Bytecode DSL source sections, StandardTags, debugger integration, instrumentation, suspension/resume and source-location tooling, while its `@GenerateBytecode` declarations do not override `enableCompressedSources`.

Classification:

```text
COMPRESSED_SOURCE_INFORMATION=ACTIONABLE_NOW_FOR_VALIDATION
SOURCE_CHANGE_REQUIRED=NO
```

The 25.4 compatibility run must validate source sections, instrumentation and debugger behavior. There is no current evidence justifying forcing compression off.

## Removed/deprecated API compatibility

25.4 removes groups of long-deprecated Truffle APIs, including old Object/String APIs, `InteropException.initCause(Throwable)`, old debugger/instrumentation/core APIs and old profile factory methods.

Current Protos HEAD searches found no source use of the specifically relevant removed members inspected during this slice.

```text
ESTABLISHED_HANDWRITTEN_SOURCE_INCOMPATIBILITY=NONE_FOUND

BEHAVIORAL_RISK:
  guest inlining frequency semantics
  compressed Bytecode DSL source representation
  changed default Graal optimization phases
  expanded read elimination
```

Generated Bytecode DSL code remains an execution-time compatibility question because the 25.4 annotation processor has not been run in this investigation.

## Artifact/container findings

The candidate Graal/Truffle component line is `25.4.4.1.1`. Required Graal/Truffle Maven artifacts are published on the aligned 25.4 component line.

Official GraalVM Community container families provide Oracle Linux variants including Oracle Linux 10 and Linux amd64/arm64 support, with corresponding Native Image images.

The eventual experiment must retain the actual container identity observed at execution:

```text
tag
image ID
RepoDigest
java -version
native-image --version where applicable
```

A moving tag alone is not sufficient retained evidence.

## Cross-impact on current performance work

### PERF010

```text
HISTORICAL_25_3_EVIDENCE=UNAFFECTED
CURRENT_25_4_INTERPRETATION=SHOULD_BE_RETESTED
```

PERF010 investigates exactly the guest-call PE/inlining surfaces affected by 25.4.

### PERF013

PERF013's architectural conclusion selecting canonical Truffle materialized-local machinery remains valid.

```text
ARCHITECTURAL_CONCLUSION=UNAFFECTED
25_4_CORRECTNESS=REVALIDATE
25_4_COMPILER_SHAPE=RETEST
25_4_TIMING=RETEST
```

### PERF014

PERF014 established the intended direct Closure-call structural shape but falsified the stronger multiplicative timing prediction under 25.3.

```text
25_3_STRUCTURAL_CONCLUSION=UNAFFECTED
25_3_TIMING_CONCLUSION=UNAFFECTED_AS_HISTORICAL_EVIDENCE
25_4_BEHAVIOR=SHOULD_BE_RETESTED
CROSS_VERSION_INTERPRETATION=MAY_CHANGE
```

Because PERF014 deliberately creates stable direct guest-call sites, GR-79418 is especially relevant to its 25.4 behavior.

### PERF017

PERF017 already established that the deeper parallel-scaling ceiling predates PERF014/TOOL009 and is not a simple eight-Case admission cap.

```text
EXISTING_STATIC_HISTORY_CONCLUSIONS=UNAFFECTED
25_4_SCALING_BEHAVIOR=SHOULD_BE_RETESTED
25_4_IS_HISTORICAL_ROOT_CAUSE=NO
```

25.4 may change JIT/compiler/GC behavior and therefore observed scaling, but it cannot explain a historical ceiling that predates the release.

## Existing benchmark-harness capability

Current `guillermomolina/protos-benchmarks` already contains the required scientific controls in the PERF010/PERF014 harness family:

- exact product revision and version identity;
- exact harness revision;
- runtime/container identity probes;
- CPU affinity;
- network isolation;
- workload-source SHA-256 identity checks;
- correctness before timing;
- warmup/steady-state separation;
- clean timing without JFR/compiler tracing;
- raw timing samples;
- median/MAD/p95/stationarity reporting;
- deterministic A/B counterbalancing;
- canonical/workload-control pairing.

The retained PERF014 comparator varies product revision while holding toolchain fixed.

UPSTREAM003-B must invert that dimension:

```text
same exact Protos revision
same exact benchmark revision
same workload sources
same machine
same CPU policy
same harness configuration
same warmup
same measurement protocol

A = GraalVM/Truffle 25.3.4.1
B = GraalVM/Truffle 25.4.4.1.1
```

The missing capability is therefore a bounded harness implementation, not a new benchmark architecture.

## Required controlled experiment

### Evidence unit A — clean steady-state timing

Use the same Protos revision for both platform roles and include at least:

```text
workload-control / control
micro/closure-call
micro/method-call
runtime/monomorphic-dispatch
```

Retain contemporaneous A and B runs. Historical 25.3 timing must not substitute for the A side.

### Evidence unit B — compiler/inlining diagnostics

Run separately from clean timing and retain enough evidence to compare:

```text
direct versus indirect call survival
guest call-tree frequencies
expanded/inlined/cutoff decisions
recursion/inlining depth
graph/IR size where available
Too-deep-inlining presence/absence
compiler failures/bailouts
source/root identity
```

The GR-79418 discriminator is:

> Does 25.4 assign materially different Graal-IR-derived call-site frequencies to the same hot Protos direct guest calls, and does that alter guest-call expansion/inlining?

### PERF017-sensitive extension

Do not fold full PERF017 profiling into the primary microbenchmark experiment.

A bounded secondary comparison may retain `jobs=1` and a high-concurrency setting such as `jobs=16` with wall/user/system time and effective parallelism. A material movement can then be consumed by PERF017 for deeper JFR/thread-state/compiler/GC discrimination.

## Sequencing decision

The project owner confirmed that both the 25.4 upgrade and the existing performance work must happen independently of the measured direction.

The selected execution order is:

```text
UPSTREAM003-A
  -> complete static investigation

UPSTREAM003-B
  -> controlled 25.3 vs 25.4 platform comparison on one unchanged Protos HEAD

25.4 adoption implementation
  -> dependency/container/distribution migration and full compatibility validation

new 25.4 performance baseline

resume/deepen PERF010 / PERF017 / related performance work on 25.4
```

Rationale: the controlled A/B must precede adoption so the platform effect is identifiable, while substantive new Protos performance changes should follow adoption so effort is not spent tuning against compiler/inlining behavior that is about to change.

This sequencing does not invalidate or reopen completed PERF013/PERF014 implementation work.

## Next slice

```text
NEXT_SLICE=UPSTREAM003-B
TYPE=IMPLEMENTATION_BENCHMARK_HARNESS
REPOSITORY=guillermomolina/protos-benchmarks
NEW_FORMAL_ISSUE_REQUIRED=NO
OWNING_ISSUE=guillermomolina/protos#732
```

UPSTREAM003-B should add the smallest comparator/configuration needed to vary only the upstream platform identity while reusing the existing PERF010/PERF014 timing and evidence discipline.

No Protos source optimization, dependency upgrade, StackValue adoption, branch-probability injection, compiler-option override, or benchmark workload change belongs in that slice.

## Upstream sources

Primary sources used by the investigation include:

- https://www.graalvm.org/release-notes/25.3/
- https://www.graalvm.org/release-notes/25.4/
- https://www.graalvm.org/docs/getting-started/container-images/
- https://github.com/oracle/graal/blob/vm-25.3.4.1/compiler/src/jdk.graal.compiler/src/jdk/graal/compiler/truffle/phases/inlining/CallNode.java
- https://github.com/oracle/graal/blob/vm-25.4.4.1.1/compiler/src/jdk.graal.compiler/src/jdk/graal/compiler/truffle/phases/inlining/CallNode.java
- Truffle 25.4 Javadocs for `CompilerDirectives`, `GenerateBytecode`, profiles and `StackValue`.
- Maven Central metadata for the 25.4 Graal/Truffle component line.

## Repository surfaces inspected

### guillermomolina/protos

```text
AGENTS.md
AGENTS.work/DESIGN.md
AGENTS.work/COORDINATION.md
pom.xml
.devcontainer/Dockerfile
build/native/Dockerfile
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosLanguage.java
relevant PERF010/PERF013/PERF014/PERF017 Issues and retained checkpoints
```

### guillermomolina/protos-benchmarks

```text
AGENTS.md
runner/perf010a.py
runner/perf010a_post_i068_baseline.py
runner/perf010a_post_i072_fprime.py
runner/perf014_direct_closure_call.py
config/perf014-direct-closure-call.json
docker/protos-perf010a/Dockerfile
docker/protos-perf010a/Perf010aTimingDriver.java
```

## Specification effect

```text
NORMATIVE_SPECIFICATION_CHANGE=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
DURABLE_PLATFORM_DECISION=NO
```

UPSTREAM003-A records evidence and sequencing only. It does not itself adopt GraalVM 25.4.
