# UPSTREAM003-B — GraalVM / Graal / Truffle 25.3 vs 25.4 controlled comparison

Date: 2026-09-28

## Scope

This record retains the executed controlled platform comparison for UPSTREAM003 / `guillermomolina/protos#732`.

The experiment changes the upstream GraalVM/Graal/Truffle platform while holding the Protos product revision, workload sources, machine, CPU policy, warmup/steady protocol and benchmark semantics fixed.

It is the execution counterpart to:

```text
docs/project/evidence/UPSTREAM003/UPSTREAM003_A_GRAALVM_25_4_STATIC_IMPACT_INVESTIGATION.md
```

No Protos language semantic change or Protos performance optimization is part of this evidence.

## Evidence identity

```text
WORK_ITEM=UPSTREAM003/#732
SLICE=UPSTREAM003-B
TYPE=CONTROLLED_PLATFORM_COMPARISON

PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=44690b1fc8c9aed023600c6d5731f969c4507e27
PROTOS_VERSION=0.3.106-SNAPSHOT

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks

TIMING_HARNESS_REVISION=4620cb3e95155298f3cc1e6966f0b140dfabcca4
DIAGNOSTIC_HARNESS_REVISION=6eeeec25b131d7ebc31e117e2570cf11e38a0013
BENCHMARK_EVIDENCE_REVISION=61371e21acdbf23376f6cb0edece7da357c6c99f

TIMING_EVIDENCE_PATH=results/upstream003-platform-comparison/timing/
DIAGNOSTIC_EVIDENCE_PATH=results/upstream003-platform-comparison/diagnostic/
```

The clean timing run intentionally remains bound to its original harness revision. The later harness revisions only repaired diagnostic-option compatibility/parsing and do not retroactively change or invalidate the timing evidence.

## Platform identity

```text
PLATFORM_A_LABEL=25.3
PLATFORM_A_GRAALVM_TRUFFLE=25.3.4.1
PLATFORM_A_JDK=25.0.4.1
PLATFORM_A_IMAGE=ghcr.io/graalvm/graalvm-community:25i3-25.0.4.1-ol10-20260825
PLATFORM_A_IMAGE_REPODIGEST=sha256:636592a38dbd6461e71aaf019ed1ff1f389c171540bc2cb632fd73e6ec7848d7

PLATFORM_B_LABEL=25.4
PLATFORM_B_GRAALVM_TRUFFLE=25.4.4.1.1
PLATFORM_B_JAVA_VERSION=25.0.4.1.1
PLATFORM_B_JVMCI=25.4-b23
PLATFORM_B_IMAGE=ghcr.io/graalvm/graalvm-community:25i4-ol10
PLATFORM_B_IMAGE_REPODIGEST=sha256:a7b4810d7c755e9627feaa1459eb5a93338643b16d745d4f3fc86db71e5da7f5
PLATFORM_B_OBSERVED_IMAGE_ID=sha256:6ff7aca7c34fc43d1e67216620550325809bb974eb8cbf5e5fe5de4bfbddce97

MAVEN=3.9.9
ARCH=amd64
NETWORK=none
CPUSET=0
```

Both roles selected:

```text
com.oracle.truffle.runtime.hotspot.HotSpotTruffleRuntime
```

with the expected exact Truffle package/component version for the role.

## Same-product invariant

Platform B required a build-only toolchain metadata overlay because the historical Protos revision intentionally pins the 25.3 dependency/distribution contract.

Authorized B-side overlay paths were exactly:

```text
dist/build_portable.py
pom.xml
toolchain.json
```

Retained identity gates:

```text
SAME_PROTOS_REVISION=PASS
SAME_PROTOS_VERSION=PASS
SAME_EXECUTABLE_LANGUAGE_SOURCE=PASS
B_TOOLCHAIN_OVERLAY_EXACT_SCOPE=PASS
WORKLOAD_SOURCE_IDENTITY=PASS

SOURCE_GIT_REVISION_SAME=YES
EXECUTABLE_LANGUAGE_SOURCE_SAME=YES
BUILD_TOOLCHAIN_METADATA_OVERLAY_B=YES

PROTOS_OPTIMIZATION_MIXED_IN=NO
PROTOS_SEMANTIC_CHANGE=NO
HISTORICAL_EVIDENCE_REWRITTEN=NO
```

The experiment does not claim `SOURCE_TREE_BYTE_IDENTICAL=YES`, because the B build metadata overlay is intentionally present.

## Clean timing protocol

```text
operation_count=10000
warmup_iterations=120
steady_iterations=100
block_order=A,B,A,B
network=none
cpu_affinity=one physical-core representative
timing_phase=steady only
diagnostic_instrumentation_in_timing=NO
automatic_adoption_classification=NO
whole_language_aggregate=NO
```

The four workloads were:

```text
micro/slot-read
micro/closure-call
micro/method-call
runtime/monomorphic-dispatch
```

Expected observable result remained `42`.

## Retained timing result

Signed raw delta is `platform_b - platform_a`.

| workload | 25.3 median ns | 25.4 median ns | raw delta | paired-control delta | order effect |
|---|---:|---:|---:|---:|---|
| micro/slot-read | 56,422,864.5 | 58,865,707.0 | +4.329526% | +0.663829% | DETECTED |
| micro/closure-call | 62,233,293.5 | 63,877,955.0 | +2.642736% | +0.526696% | DETECTED |
| micro/method-call | 62,464,029.5 | 65,834,278.0 | +5.395503% | +3.401827% | DETECTED |
| runtime/monomorphic-dispatch | 58,709,103.0 | 59,771,443.5 | +1.809499% | -2.241223% | NOT_DETECTED |

Interpretation is deliberately per-workload.

The result is **mixed movement**:

- `slot-read` and `closure-call` raw movement is mostly shared/control movement and is order-sensitive;
- `method-call` retains a larger positive paired-control movement but also has detected order/stationarity sensitivity;
- `monomorphic-dispatch` has the cleanest directional result in this run: the paired-control measure moves negative for 25.4 and the harness does not detect an order effect.

This evidence does not support a single whole-language “25.4 is faster/slower” number.

## Separate compiler/inlining diagnostic

Diagnostic evidence was collected separately from clean timing:

```text
EVIDENCE_KIND=compiler_inlining_diagnostic
TIMING_CLAIM=NO
warmup_iterations=20
steady_iterations=20
TraceCompilation=true
TraceCompilationDetails=true
TraceInlining=true
CompilationFailureAction=Print
```

Primary diagnostic workloads:

```text
micro/closure-call
micro/method-call
runtime/monomorphic-dispatch
```

The retained trace shows a material change in effective guest-call frequency between the same Protos workload under 25.3 and 25.4.

Representative relevant observations include:

```text
25.3 guest/direct-call trace frequencies: commonly 1.00
25.4 corresponding relevant guest-call trace frequencies: materially lower at selected call sites,
including values in the approximate 0.04 / 0.14 / 0.16 range in the retained traces
```

This is consistent with the UPSTREAM003-A finding for GR-79418:

```text
25.3:
runtime DirectCallNode observations
    -> guest inlining frequency

25.4:
PE result + Graal CFG / block relative frequency
    -> guest inlining frequency
```

At the same time, the final observed guest-call inlining shape remains broadly similar on the targeted workloads. Relevant callees still reach inlined/expanded outcomes, and the `Too deep inlining` bailout class remains present rather than disappearing under 25.4.

Therefore:

```text
GUEST_CALL_FREQUENCY_CHANGE=CONFIRMED
FINAL_INLINING_SHAPE_RADICALLY_CHANGED=NO_EVIDENCE
TOO_DEEP_INLINING_ELIMINATED=NO
GR79418_EXPLAINS_EXISTING_PROTOS_PERFORMANCE_GAP=NOT_ESTABLISHED
```

Timing and diagnostic evidence together do not justify attributing the existing PERF010/PERF013/PERF014 performance gap to the 25.4 frequency change alone.

## Branch-probability implication

UPSTREAM003-A established no direct current Protos use of:

```text
CompilerDirectives.injectBranchProbability
BranchProfile
ConditionProfile
```

UPSTREAM003-B confirms that 25.4 nevertheless changes the effective guest-call frequency seen by the compiler on current Protos workloads.

Classification:

```text
EXPLICIT_PROTOS_BRANCH_PROBABILITY_CORRECTION_REQUIRED=NO_EVIDENCE
GENERATED_BYTECODE_DSL_CFG_REMAINS_RELEVANT=YES
ADD_PROTOS_PROFILES_AS_PART_OF_ADOPTION=NO
```

No speculative probability injection belongs in the runtime migration.

## Compatibility impact

UPSTREAM003-A found no required handwritten Protos source change from static inspection.

UPSTREAM003-B successfully instantiated and executed the 25.4 platform using the exact same Protos executable-language source plus the bounded build-toolchain overlay. The 25.4 Maven/runtime component closure resolved and the runtime identity probes selected the expected optimizing Truffle runtime.

This establishes:

```text
STATIC_SOURCE_COMPATIBILITY=COMPATIBLE
EXPERIMENT_BUILD_COMPATIBILITY=PASS
EXPERIMENT_RUNTIME_COMPATIBILITY=PASS
REQUIRED_PROTOS_SEMANTIC_CHANGE=NO
KNOWN_25_4_ADOPTION_BLOCKER=NONE_FROM_UPSTREAM003
```

It does **not** substitute for migration closure validation on current product HEAD. Full Maven, checkout CLI, debugger/DAP/source sections, portable distribution, Native Image/container and complete integrated validation belong to the executable adoption owner.

## Cross-impact on existing performance work

### PERF010 / PERF013 / PERF014

Historical 25.3 evidence remains valid for the exact revisions/environments that produced it.

The controlled 25.4 comparison does not establish that the upstream inlining-frequency change is the dominant cause of the existing Protos guest-call cost. The relevant Protos-side performance work therefore remains meaningful after adoption.

```text
HISTORICAL_25_3_RESULTS_REWRITTEN=NO
GR79418_DOMINANT_CAUSE=NOT_ESTABLISHED
EXISTING_PROTOS_PERF_WORK_INVALIDATED=NO
```

### PERF017

25.4 may affect current compiler/runtime scaling characteristics, but it cannot be the historical root cause of a scaling ceiling that predates 25.4.

PERF017 remains independently owned and should be resumed against the adopted 25.4 baseline rather than folded into DIST006.

## Impact classification

Final UPSTREAM003 classification:

```text
UPSTREAM003_IMPACT=ACTION_REQUIRED
```

Rationale:

- exact 25.4 artifacts and the OL10 GraalVM Community platform are available and executable;
- static and controlled-execution evidence exposes no adoption blocker;
- 25.4 materially changes compiler behavior on relevant Protos guest-call paths, so remaining on 25.3 would keep active performance work on a runtime line the project already intends to supersede;
- the controlled evidence does not justify a Protos-side workaround before adoption;
- the project-owner-selected sequence retained by UPSTREAM003-A is adoption first, then a new 25.4 baseline, then continued performance work.

## Executable consequence / owner

The concrete implementation owner allocated after the comparison is:

```text
DIST006/#733 — Adopt GraalVM / Truffle 25.4.4.1.1 runtime baseline
REPOSITORY=guillermomolina/protos
INITIAL_STATUS=READY
PRIORITY=P2
```

DIST006 preserves PLAT033's already-ratified runtime/dependency architecture and changes the exact selected upstream line within that architecture.

No new PLAT decision is required by UPSTREAM003 evidence.

The first executable slice is:

```text
DIST006-A — canonical version/toolchain migration
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
```

## UPSTREAM003 closure classification

UPSTREAM003's closure gates are satisfied by the combination of the A and B records, benchmark evidence and the allocation of DIST006:

```text
25_4_ARTIFACT_CONTAINER_AVAILABILITY=ESTABLISHED
COMPATIBILITY_IMPACT=EVALUATED
CONTROLLED_25_3_VS_25_4_PERFORMANCE_EVIDENCE=RETAINED
BRANCH_PROBABILITY_INLINING_IMPLICATIONS=CLASSIFIED
IMPACT_CLASSIFICATION=ACTION_REQUIRED
REQUIRED_CONCRETE_PROTOS_OWNER=DIST006/#733
HISTORICAL_EVIDENCE_REWRITTEN=NO

DURABLE_RECORD_DECISION=REQUIRED
```

The derived implementation does not need to finish before UPSTREAM003 closes; DIST006 owns execution from this point.

## Source inventory

Durable sources for this record:

### Project record

```text
guillermomolina/protos-project-docs
docs/project/evidence/UPSTREAM003/UPSTREAM003_A_GRAALVM_25_4_STATIC_IMPACT_INVESTIGATION.md
```

### Benchmark evidence

```text
guillermomolina/protos-benchmarks
revision 61371e21acdbf23376f6cb0edece7da357c6c99f

results/upstream003-platform-comparison/timing/README.md
results/upstream003-platform-comparison/timing/raw.json
results/upstream003-platform-comparison/timing/summary.tsv
results/upstream003-platform-comparison/timing/stationarity.tsv

results/upstream003-platform-comparison/diagnostic/README.md
results/upstream003-platform-comparison/diagnostic/raw.json
results/upstream003-platform-comparison/diagnostic/logs/**
```

### Live coordination

```text
guillermomolina/protos#732  UPSTREAM003
guillermomolina/protos#733  DIST006
```

## Specification effect

```text
NORMATIVE_SPECIFICATION_CHANGE=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
NEW_PLATFORM_ARCHITECTURE_DECISION=NO
HISTORICAL_EVIDENCE_REWRITE=NO
```

UPSTREAM003-B is evidence and routing. The actual version/toolchain migration belongs to DIST006.
