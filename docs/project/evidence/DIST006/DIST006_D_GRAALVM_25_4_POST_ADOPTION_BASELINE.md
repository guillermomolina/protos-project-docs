# DIST006-D — GraalVM / Truffle 25.4 post-adoption performance baseline

Status: PUBLISHED
Owning Issue: \`guillermomolina/protos#733\` (\`DIST006\`)
Slice: \`DIST006-D — post-adoption 25.4 performance baseline\`

## Publication identity

\`\`\`text
PROTOS_REVISION=7aaaec6923265c99723ce5bca064e5b3ab52b8c4
HARNESS_REVISION=1e4a286f5f8fa43f96a5857818b8bf5d834ca8db
BENCHMARK_EVIDENCE_REVISION=cf72a5b88dac2c5d7ba2f0b1a1ec9e1ee19d1402
BENCHMARK_EVIDENCE_PATH=results/dist006d-baseline
GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1
JDK_VERSION=25.0.4.1.1
JVMCI=25.4-b23
MAVEN=3.9.9
EXPECTED_RUNTIME_CLASS=com.oracle.truffle.runtime.hotspot.HotSpotTruffleRuntime
\`\`\`

The retained benchmark evidence is a single-platform post-adoption baseline.
It does not recreate, rewrite or reinterpret the historical UPSTREAM003
25.3-vs-25.4 comparison.

## Retained reference result

The owner executed the DIST006-D reference run from the exact published harness
revision with a clean benchmark worktree and exact Protos revision.

\`\`\`text
DIST006D_REFERENCE=PASS
DIST006D_REFERENCE_EVIDENCE_STATUS=RETAINED
DIST006D_REFERENCE_PROTOS_REVISION=7aaaec6923265c99723ce5bca064e5b3ab52b8c4
DIST006D_REFERENCE_HARNESS_REVISION=1e4a286f5f8fa43f96a5857818b8bf5d834ca8db
DIST006D_REFERENCE_PLATFORM=25.4.4.1.1
DIST006D_REFERENCE_PAIRED_PLATFORM_FORMULA=NO
\`\`\`

The retained unit records:

- the exact GraalVM Community OL10 base image by digest;
- \`com.oracle.truffle.runtime.hotspot.HotSpotTruffleRuntime\`;
- GraalVM / Graal / Truffle 25.4.4.1.1;
- Java 25.0.4.1.1 and JVMCI 25.4-b23 for the measured runtime;
- Maven 3.9.9;
- one-CPU affinity through \`--cpuset-cpus\`;
- network disabled;
- 120 warmup iterations and 100 steady iterations;
- the four established canonical/control workloads;
- no JFR, compiler tracing, IGV, allocation profiling, source instrumentation
  or Test Tool diagnostic instrumentation in the timing phase.

The reference evidence is retained in \`guillermomolina/protos-benchmarks\` at
exact commit \`cf72a5b88dac2c5d7ba2f0b1a1ec9e1ee19d1402\`. Its \`raw.json\` is the
authoritative measurement record and \`summary.tsv\` is a derived view.

## Correctness admission evidence

Before the retained reference, the DIST006-D smoke gate validated all four
canonical workloads and their controls. Every workload produced the expected
result \`42\`.

\`\`\`text
DIST006D_SMOKE_CORRECTNESS=PASS
RETAINED_PERFORMANCE_EVIDENCE=NO
TIMING_EVIDENCE=NO
REFERENCE_EVIDENCE=NO
DIST006D_SMOKE=PASS
\`\`\`

That smoke run also established the exact measured runtime identity:

\`\`\`text
engine_implementation=GraalVM CE
engine_version=25.4.4.1.1
java_version=25.0.4.1.1
java_runtime_version=25.0.4.1.1+1-jvmci-25.4-b23
java_vm_vendor=GraalVM Community
runtime_class=com.oracle.truffle.runtime.hotspot.HotSpotTruffleRuntime
truffle_package_version=25.4.4.1.1
\`\`\`

## Build-stage Java-path finding

The retained evidence also exposed a toolchain-selection inconsistency in the
benchmark image build stage. Installing the OL10 Maven package introduces the
system OpenJDK 21 dependency. Maven itself follows the GraalVM \`JAVA_HOME\` and
reports Java 25.0.4.1.1, but the unqualified build-stage \`java\` command resolves
to Red Hat OpenJDK 21.0.12.1.

Observed build identity:

\`\`\`text
bare java:
  OpenJDK 21.0.12.1
  vendor=Red Hat

mvn -version:
  Apache Maven 3.9.9
  Java version=25.0.4.1.1
  vendor=GraalVM Community
\`\`\`

This does not invalidate the retained DIST006-D measurement: the final benchmark
runtime image is based directly on the pinned GraalVM 25.4 image, and the runtime
probe independently established Java 25.0.4.1.1 plus the exact
\`HotSpotTruffleRuntime\`.

However, the benchmark build stage also invokes unqualified \`javac\` when
compiling the benchmark helper classes. The current canonical benchmark harness
should therefore be reconciled so \`\${JAVA_HOME}/bin\` precedes the system Java
path for both \`java\` and \`javac\`, matching the development-container
selection invariant already established by DIST006-A.

This is a follow-up implementation/toolchain-coherence repair. It must not
rewrite the retained evidence or silently replace its exact harness identity.

## Scope conclusion

\`\`\`text
DIST006_D_BASELINE_STATUS=RETAINED
POST_ADOPTION_25_4_BASELINE=PASS
HISTORICAL_UPSTREAM003_EVIDENCE_REWRITTEN=NO
PAIRED_PLATFORM_FORMULA=NO
SEMANTIC_CHANGE=NO
PERFORMANCE_OPTIMIZATION_MIXED_IN=NO
BENCHMARK_BUILD_STAGE_JAVA_PATH_RECONCILIATION=PENDING
\`\`\`

The next bounded benchmark cleanup is to align the DIST006 benchmark build-stage
\`java\` and \`javac\` selection with the canonical GraalVM \`JAVA_HOME\`. The
remaining parent DIST006 closure obligations also still include DIST006-B2 and
DIST006-C.
