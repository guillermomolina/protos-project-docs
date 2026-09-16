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
