# PERF009 immutable diagnostic evidence

## PERF009-A — IGV/JFR critical-path attribution checkpoint

Status: **IN PROGRESS — DIAGNOSTIC CHECKPOINT RETAINED**

This evidence records the PERF009-A diagnostic checkpoint reached while
attributing the current repository-test latency problem. It does not close
PERF009-A, approve a remediation architecture, or change Protos semantics,
implementation, runtime behavior, tests, or coverage.

The live work authority remains:

- `guillermomolina/protos#530` — PERF009;
- `guillermomolina/protos#531` — PERF009-A.

### Exact source and tooling identities

```text
protos_revision=e5f56c6ee829090c3ed12a9b6703f5984d872a90
protos_benchmarks_revision=9b14d807a4eab2c1f637723d1c1f9cbb9a3dcc01
graal_truffle_version=25.3.4.1
jdk_version=25.0.4.1
current_igv_analyzer=docker/igv-analyzer + scripts/igv_analyzer.sh
current_igv_analyzer_generation=Graal_25.3.4.1
historical_igv_analyzer=docker/igv-analyzer24 + scripts/igv_analyzer24.sh
historical_igv_analyzer_generation=Graal_24.0.0
```

The unsuffixed IGV analyzer is the current/default diagnostic generation for
new PERF work. The Graal 24 analyzer is explicitly historical and remains only
for retained PERF003 replay paths that require its old `analyze`/`summarize`
contract.

### IGV analyzer normalization validation

The current/default Graal 25 analyzer normalization was published in
`guillermomolina/protos-benchmarks` at exact revision
`9b14d807a4eab2c1f637723d1c1f9cbb9a3dcc01`.

Observed validation before publication:

```text
focal_igv_tests=4
focal_igv_failures=0
full_protos_benchmarks_tests=55
full_protos_benchmarks_failures=0
current_analyzer_build=PASS
current_analyzer_smoke=PASS
git_diff_check=PASS
machine_local_paths_in_persistent_analyzer_contract=NONE
```

The published naming contract is:

```text
current/default:
  docker/igv-analyzer/
  scripts/igv_analyzer.sh
  make igv-analyzer-build
  make igv-analyzer-smoke

historical PERF003 only:
  docker/igv-analyzer24/
  scripts/igv_analyzer24.sh
```

### Corrected TOML stress-path diagnosis

The original `ProtosTomlParserStressTest` path had been invoked through a test
harness that did not enter the ordinary Protos execution context. Profiling that
path showed `ProtosClosureExecutionPlan.rebuildAstForLegacyFallback` dominating
execution. Re-running the stress path through the ordinary test execution bridge
removed that legacy fallback from the hot path.

The corrected diagnosis therefore supersedes any causal conclusion based on the
legacy-fallback-heavy stress run. In particular, the earlier approximately
140-second stress-class observation is not accepted as proof that ordinary TOML
parser semantics are themselves the current repository critical path.

Corrected JFR headline:

```text
LEGACY_FALLBACK_STILL_HOT=NO
corrected_top_frame=ProtosBytecodeRootNodeGen$CachedBytecodeNode.continueAt
corrected_top_frame_percent=64.15
corrected_unsafe_put_object_percent=10.08
truffle_compilations=344
compilation_failures=118
```

### BGV structural attribution

Twelve Diagnose-mode BGV captures from the corrected production Bytecode DSL
path produced exactly two structural signature families:

```text
BGV_COUNT=12
STRUCTURAL_SIGNATURE_COUNT=2

CLOSURE_CALL_BGV=2
IMPORT_SEND_BGV=10
```

The two captured structural families are:

```text
CLOSURE_CALL
  continueAt
    -> lookup / closure preparation
    -> prepareClosureCall
    -> prepareStandardClosureCallIntrinsic
    -> finishPreparingComposedCall
    -> forClosureInvocation / dynamic-control inheritance
    -> permanent recursive-inlining bailout

IMPORT_SEND
  continueAt
    -> PrepareSendArguments
    -> prepareSend
    -> prepareImmediateMethodCall
    -> prepareBytecodeImport
    -> resolveModuleKey
    -> ProtosStandardLibraryModuleResolver
    -> permanent recursive-inlining bailout
```

The captured BGV corpus totalled 117,510,908 bytes during the diagnostic run.
The 2/12 versus 10/12 BGV split is structural evidence only; it is not treated
as a weighted estimate of failure frequency.

### Exact JFR failure classification

All 118 corrected JFR
`jdk.graal.compiler.truffle.CompilationFailure` events were classified using
signatures derived from the two BGV families.

```text
COMPILATION_FAILURES=118
PERMANENT_FAILURES=118
TOO_DEEP_INLINING=118
IMPORT_SEND=100
CLOSURE_CALL=18
OVERLAP=0
OTHER=0
```

Therefore the corrected failure corpus is fully accounted for by the two
observed production Bytecode DSL families. No third unclassified compilation
failure family remains in this recording.

The common failure is:

```text
PermanentBailoutException: Too deep inlining, probably caused by recursive inlining.
```

This establishes compiler-structural attribution for this diagnostic workload;
it does not by itself establish how much of the complete repository validation
wall clock these failures represent.

### PERF009-A consequence

The diagnostic checkpoint supports these conclusions:

```text
LEGACY_AST_FALLBACK_CAUSAL_FOR_CORRECTED_PATH=NO
PRODUCTION_BYTECODE_DSL_HOT_PATH=YES
CORRECTED_COMPILATION_FAILURES_FULLY_CLASSIFIED=YES
PRODUCTION_OPTIMIZATION_SELECTED=NO
PROTOS_SEMANTICS_CHANGED=NO
PERF009_A_CLOSED=NO
IS_PACKAGE_TOML_STILL_A_DOMINANT_CURRENT_BOTTLENECK=NOT_YET_RESOLVED
```

PERF009-A still requires reproducible measurement of the complete current
validation topology, including the current Java/JUnit lanes, the public Protos
Test Tool path, quarantine reconciliation, repeated samples/variance,
parallelism/utilization, duplicate execution, and a project-owner-approved
latency target. Those measurements must determine whether the corrected
TOML-related diagnostic path is materially dominant in the actual complete
repository wall clock.

### Retention boundary

This project-documentation record retains the durable project-level diagnostic
checkpoint and exact repository identities. Large raw JFR/BGV working artifacts
are not copied into `protos-project-docs` by this record. No claim is made that
those local working artifacts are retained reference results. Any later retained
raw performance corpus must record immutable artifact identities and the exact
Protos and benchmark-harness revisions that produced it.

## PERF009-A1 — Current test-topology inventory checkpoint

Status: **PUBLISHED — INVENTORY CONTRACT RETAINED; TIMING NOT YET RUN**

This checkpoint records the bounded PERF009-A1 inventory slice published in
`guillermomolina/protos-benchmarks`. It freezes the measurement surface for the
next PERF009-A timing slices; it is not itself a latency measurement and does
not close PERF009-A.

### Exact revisions

```text
protos_revision=e5f56c6ee829090c3ed12a9b6703f5984d872a90
protos_benchmarks_validation_repair=cdaf2ea33bf22c8afadd9ede9905f4f52d9ec9e4
protos_benchmarks_inventory_revision=ee98980ca019c124214d9e9df21f028548fc2a7a
```

The validation-repair commit precedes the inventory commit and only repairs
stale benchmark-repository validation guards exposed by the current IGV and
PERF006 lineage. It does not alter Protos behavior or PERF009 measurement
semantics.

### Frozen operational topology

The inventory runner materialized the exact Protos revision above in an isolated
work directory and extracted the current operational test topology from the
pinned source rather than relying on a hand-maintained summary.

```text
JAVA_DEFAULT_JOBS=8
PROTOS_DEFAULT_JOBS=8
JAVA_QUARANTINE_COUNT=8
JAVA_SERIAL_TEST=ProtosI026FDapBehaviorTest
TEST_TOOL_PHASE_COUNT=18
MEASUREMENT_LANE_COUNT=4
DIAGNOSTIC_PROTOS_JOBS=1,2,4,8
MINIMUM_REPETITIONS=3
```

The eight Java quarantine classes are:

```text
ProtosTomlParserStressTest
ProtosTomlEncoderModuleTest
ProtosPackageToolProtosTest
ProtosExternalPackagePlanningPreflightTest
ProtosWorkspaceRunCliTest
ProtosJsonParserModuleTest
ProtosPackageExecutionPlanAdapterTest
ProtosTestToolManifestPlanTest
```

The public Protos Test Tool exposes these 18 ordered phases at the pinned
revision:

```text
main
process-snapshot
actor
group
package-toml
uri
csv
cli
math-integer
crypto-sha256
network-ip-addresses
network-ip-endpoints
package-tool-version
package-tool-lock
package-tool-resolution-input
package-tool-resolution-root
package-tool-execution-plan
package-tool-project-projection
```

The inventory contract records four later measurement lanes:

```text
current-ci:
  make test JAVA_TEST_JOBS=4 PROTOS_TEST_JOBS=4

java-current:
  make test-java JAVA_TEST_JOBS=4

protos-current:
  make test-protos PROTOS_TEST_JOBS=4

java-complete-reference:
  mvn test
```

The complete Maven lane is deliberately labelled as a reference lane, not as the
current CI topology. Diagnostic Protos Test Tool runs are declared for
`--jobs 1`, `2`, `4`, and `8`, with at least three repetitions per measurement
condition.

### Validation evidence

Before publication, the PERF009-A1 focal suite passed 7/7 tests and the complete
`protos-benchmarks` validation passed after repairing two pre-existing stale
validation guards. The complete Python unit-test discovery reported 59/59 tests
passing, including all seven PERF009-A1 tests.

```text
PERF009_A1_FOCAL_TESTS=7
PERF009_A1_FOCAL_FAILURES=0
FULL_PYTHON_TESTS=59
FULL_PYTHON_FAILURES=0
MAKE_VALIDATE=PASS
GIT_DIFF_CHECK=PASS
```

The two stale validation guards discovered during prepublication were:

1. the historical Graal 24 IGV assertions still pointed at the unsuffixed current
   analyzer paths after the Graal 25 normalization; and
2. the PERF006-D1 historical guard incorrectly required the current
   `config/perf006d.json` to remain on slice D1 after that config had legitimately
   advanced to D2A.

Both were repaired in `cdaf2ea33bf22c8afadd9ede9905f4f52d9ec9e4` before the
A1 inventory commit. The second guard now verifies D1 lineage through the exact
`d1_harness_revision` while accepting the current D2A config state.

### PERF009-A consequence

A1 establishes the reproducible inventory contract required before timing:

```text
CURRENT_TEST_TOPOLOGY_INVENTORIED=YES
JAVA_QUARANTINE_EXPLICIT=YES
TEST_TOOL_PHASES_EXPLICIT=YES
CURRENT_CI_LANE_EXPLICIT=YES
COMPLETE_JAVA_REFERENCE_LANE_EXPLICIT=YES
DIAGNOSTIC_JOBS_MATRIX_EXPLICIT=YES
TIMING_MEASUREMENTS_RUN=NO
PERF009_A_CLOSED=NO
IS_PACKAGE_TOML_STILL_A_DOMINANT_CURRENT_BOTTLENECK=NOT_YET_RESOLVED
```

The next PERF009-A work must measure this frozen surface rather than infer the
critical path from historical runs or isolated diagnostic workloads. In
particular, the inventory alone does not establish the wall-clock share of
package/TOML work, duplicated execution, fresh Process/RootActor creation,
bootstrap/materialization, filesystem fixtures, or scheduler utilization.
