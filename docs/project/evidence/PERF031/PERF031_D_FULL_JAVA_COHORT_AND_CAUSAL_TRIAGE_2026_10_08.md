# PERF031-D — Full Java reduced-contention cohort and causal triage

Date: 2026-10-08

## Authority and scope

```text
ISSUE=guillermomolina/protos#787
SLICE=PERF031-D
TYPE=INVESTIGATION_AND_HUMAN_EXECUTED_MEASUREMENT
PRODUCT_REPOSITORY=guillermomolina/protos
MEASURED_PROTOS_REVISION=a463429f34d677ad896279861fa09d9dea251c1a
PRODUCT_VERSION=0.3.288-SNAPSHOT
MEASUREMENT_RUN=target/perf031-d/full-java-20261008-063642
GITHUB_ISSUE_STATE=OPEN_READY
SOURCE_OR_TEST_MODIFICATIONS=NONE
RUNTIME_OR_SEMANTIC_CHANGES=NONE
CI_ADMISSION_POLICY_CHANGES=NONE
PERF032_DEPENDENCY=NONE
```

PERF031-A/B/C are previously published checkpoints; this is a new immutable **measured snapshot**, not proof of a pre/post improvement. The source revision above is the human-executed checkout's exact `git rev-parse HEAD`. Subsequent changes on `main` are not part of this sample. The human's initial `git status --short` was empty. A separate prior `make test` passed but showed TEST008 `ENVIRONMENT_NOT_COMPARABLE` (`CONTROL_COHERENCE=2.045`, threshold 1.5); that result does **not** establish a comparable calibration or imply a regression in these classes.

## Immutable raw evidence (complete files)

The maintainer supplied three files, incorporated under this evidence owner, preserving every measurement row; CSV line endings are normalized from CRLF to LF, without changing columns, rows or numeric observations.

- [548-class timing CSV](https://github.com/guillermomolina/protos-project-docs/blob/8f0c9ca024a4d740eeeb2c0d36dfda21d589968f/docs/project/evidence/PERF031/PERF031_D_FULL_JAVA_CLASS_TIMINGS_2026_10_08.csv) — publication commit `8f0c9ca024a4d740eeeb2c0d36dfda21d589968f`, 548 rows.
- [2,811-testcase/method timing CSV](https://github.com/guillermomolina/protos-project-docs/blob/e1239e07e612b5121f6a3975c80205cf30db9cde/docs/project/evidence/PERF031/PERF031_D_FULL_JAVA_METHOD_TIMINGS_2026_10_08.csv) — publication commit `e1239e07e612b5121f6a3975c80205cf30db9cde`, 2,811 rows.
- [Complete Maven/Surefire output](https://github.com/guillermomolina/protos-project-docs/blob/82e4c07efa640cd22ec4227a57f1bfeec4ac2358/docs/project/evidence/PERF031/PERF031_D_FULL_JAVA_MAVEN_2026_10_08.log) — publication commit `82e4c07efa640cd22ec4227a57f1bfeec4ac2358`, 2,138 lines.

Original uploaded-file SHA-256 checksums, **before** CSV newline normalization:

```text
class-timings.csv   7e2caf3e922e83afa21c5eb016fa35fc13a50287e8f99c02c7418d3394966a2b
method-timings.csv b2054b38a37252bac40d8674b4629e4d3e09bac1c4cd20476b32eb3164b4aa91
maven.log          671bd14e4d7ec59a6287a778fb7b5009602e6e4488ba95c7f58ec23641508c70
```

## Exact command and run summary

The human executed `make test-java-confirm JAVA_CONFIRM_TESTS='*Test,*Tests,*TestCase,Test*' JAVA_CONFIRM_REPORTS=target/perf031-d/full-java-20261008-063642/reports`, with `junit.jupiter.execution.parallel.enabled=false`. Reports were preserved separately from ordinary Surefire output. No performance tests or Git commands were executed by the investigating agent.

```text
SUREFIRE_CLASSES=548
SUREFIRE_TESTS=2811
FAILURES=0
ERRORS=0
SKIPPED=1
MAVEN_BUILD=SUCCESS
HUMAN_SHELL_REAL=154.197s
MAVEN_REPORTED_TOTAL=02:33min
SUM_CLASS_SECONDS=143.971
SUM_METHOD_SECONDS=138.179
CLASS_MINUS_METHOD_SECONDS=5.792
JAVA_MAIN_COMPILATION=UP_TO_DATE
TEST_JAVA_COMPILATION=562_SOURCES_RECOMPILED
```

The run selects conventional Java test-class names, not every possible test/stress entry point (e.g. `ProtosJsonParserStress` is intentionally separately selected). It is reduced-contention *serial JUnit execution of classes in a single run*, **not independent per-class JVM isolation**. Class duration includes setup not listed in test-case durations. Maven compilation and process overhead must not be charged to individual tests. JVM warmup/order effects and host load were not independently controlled, so there is no intrinsic per-class CPU attribution, no machine-independent regression, and no before/after speedup claim.

Concentration in class-reported time (`SUM_CLASS_SECONDS=143.971`): top 1 = 19.216 s (13.35%); top 5 = 55.033 s (38.23%); top 10 = 79.939 s (55.52%); top 20 = 108.300 s (75.22%); top 30 = 117.850 s (81.86%); top 50 = 127.216 s (88.36%). Only 23 of 548 classes exceed 1 s.

## Thirty slowest class identities

Times below are **sequential Surefire class times**, not CI class-parallel wall times.

| Rank | Exact simple class name | Tests | Seconds |
|---:|---|---:|---:|
| 1 | ProtosTestToolTool012TomlOfficialProgressGroupingTest | 5 | 19.216 |
| 2 | ProtosPackageToolExecutionPlanInvalidIdentityGraphTest | 1 | 10.093 |
| 3 | ProtosRegexSemanticTransferTest | 8 | 9.486 |
| 4 | ProtosTestToolCaseRejectionPublicIntegrationTest | 5 | 8.916 |
| 5 | ProtosPackageRunDriverMixedApplicationTest | 3 | 7.322 |
| 6 | ProtosTestToolTool011ProgressGroupingTest | 10 | 6.464 |
| 7 | ProtosPackageToolExecutionPlanCaptureTest | 3 | 5.347 |
| 8 | ProtosPackageRunDriverPlanningFailureCustodyTest | 2 | 4.653 |
| 9 | ProtosWorkspacePackageAuthorityIsolationIntegrationTest | 4 | 4.337 |
| 10 | ProtosPackageToolExecutionPlanInvalidGitAndEdgeGraphTest | 1 | 4.105 |
| 11 | ProtosPackageRunDriverTest | 3 | 3.796 |
| 12 | ProtosPackageRunDriverFailureCustodyTest | 2 | 3.582 |
| 13 | ProtosTestToolCaseSelectionPublicIntegrationTest | 3 | 3.538 |
| 14 | ProtosWorkspaceRunCliTest | 7 | 3.356 |
| 15 | ProtosTestToolCaseExecutionPublicIntegrationTest | 2 | 3.213 |
| 16 | ProtosPackageExecutionPlanAdapterTest | 2 | 2.712 |
| 17 | ProtosPackageToolContentIdentityTest | 7 | 2.291 |
| 18 | ProtosTestToolFileSelectionPublicIntegrationTest | 1 | 2.123 |
| 19 | ProtosExternalPackagePlanningPreflightTest | 2 | 1.936 |
| 20 | ProtosExactExternalRequirementsPreflightTest | 6 | 1.814 |
| 21 | ProtosCliTest | 19 | 1.812 |
| 22 | ProtosPackageToolManifestTest | 5 | 1.081 |
| 23 | ProtosFilesystemLibraryConformanceTest | 12 | 1.056 |
| 24 | ProtosTestToolCaseSelectionTest | 12 | 0.982 |
| 25 | UnicodeNfc17ConformanceTest | 1 | 0.914 |
| 26 | ProtosTestToolManifestPlanTest | 18 | 0.833 |
| 27 | ProtosCommandLineSpecModuleTest | 7 | 0.752 |
| 28 | ProtosWorkspacePackagePreflightTest | 2 | 0.721 |
| 29 | ProtosTestToolSequentialRunnerTest | 2 | 0.707 |
| 30 | ProtosLoggingFacilityTest | 5 | 0.692 |

## Original PERF031-D fifteen-class cohort: all accounted for

All 15 exist in the measured source revision and passed; 88 tests, 20.171 s aggregate, 14.01% of total class-reported time. Original seven reviewed identities account for 16.241 s; the eight additional historical still-existing identities account for 3.930 s. The seven original normalized baseline expectations are historical policy values and **not** current same-host measured baselines.

| Simple class name (prefix Protos unless noted) | Tests | Class seconds | Current conclusion |
|---|---:|---:|---|
| TestToolFileSelectionPublicIntegrationTest | 1 | 2.123 | Legitimate CLI case-selection integration; reducible cost not proven |
| WorkspaceRunCliTest | 7 | 3.356 | Real workspace/Process routing; reducible cost not proven |
| FilesystemLibraryConformanceTest | 12 | 1.056 | I/O conformance, lifecycle, errors; preserve |
| ExternalPackagePlanningPreflightTest | 2 | 1.936 | Exact borrowed custody, deleted-source behavior; preserve |
| PackageExecutionPlanAdapterTest | 2 | 2.712 | Detached plan/host data, immutable boundary; preserve |
| WorkspacePackageAuthorityIsolationIntegrationTest | 4 | 4.337 | Independent Process authority/capability isolation; preserve |
| WorkspacePackagePreflightTest | 2 | 0.721 | Preflight/termination; preserve |
| PackageContentVerificationTest | 3 | 0.303 | Digest and custody verification; no material measured hotspot |
| PackageTestLogicalCaseExecutionFacilityTest | 11 | 0.426 | PERF031-C's test-only Core sharing in place; preserve fresh per-test state |
| TestToolManifestPlanTest | 18 | 0.833 | Exact manifest/plan semantics; no material measured hotspot |
| TestToolResourceCatalogSchemaTest | 3 | 0.431 | Strict schema; no material measured hotspot |
| TestToolResourceRequirementsSchemaTest | 4 | 0.431 | Strict types, duplicates and requirements; no material hotspot |
| TestToolSequentialRunnerTest | 2 | 0.707 | Dependency order/stack; retain proof size |
| TestToolSuiteGraphTest | 14 | 0.484 | Graph identity/order/cycles; no material hotspot |
| TomlParserFloatTemporalRegressionTest | 3 | 0.315 | Long-form numeric/temporal parser regression; do not shorten vectors |

The next investigation must not return to small Core-bootstrap edits in the eight sub-second historical classes without new contradictory evidence.

## High-cost method evidence and causal hypotheses

Exact measured methods:

- `ProtosPackageToolExecutionPlanInvalidIdentityGraphTest.invalidIdentityAndRegistryGraphsAreRejected` = 10.093 s. The source loops over **eight** invalid identity/registry fixtures, each invoking `executeExternalCaptureFixture`, which recreates package Prelude, hosted execution and verified filesystem-custody context. Time is **combined for all eight**, not one scenario.
- `ProtosTestToolTool012TomlOfficialProgressGroupingTest.completeCorpusGroupsAreBoundedCompleteUniqueAndDeterministic` = 7.192 s. It processes the complete TOML corpus; `RepositorySuite.progressGroupNames` is invoked twice and mapping over cases checks ownership and index. It must preserve complete corpus size, ordered groups and all assertions.
- `ProtosTestToolTool012TomlOfficialProgressGroupingTest.directorySelectionPreservesTheCorpusCaseSet` = 7.096 s. This invokes public CLI `--list-cases --directory` over the complete TOML corpus; do not assume its time belongs to the pure grouping function.
- `ProtosRegexSemanticTransferTest.parallelInputsAndResultsRebuildPatternAndMatch(Path)` = 5.000 s. The source exercises multiple independent parallel operations, rebuilt Regex values and fresh destination state. It also uses a bounded Future-poll loop with `Thread.onSpinWait`; no hot-loop causality is proven.
- `ProtosPackageToolExecutionPlanInvalidGitAndEdgeGraphTest.invalidGitAndEdgeGraphsAreRejected` = 4.105 s. Seven separate invalid graph fixtures in one JUnit method.
- `ProtosPackageToolExecutionPlanCaptureTest.missingVerifiedDescriptorIsRejected` = 2.939 s; same support path, not evidence that verification can be removed.
- `ProtosTestToolTool012TomlOfficialProgressGroupingTest.exactCaseInSplitSourceDisplaysOnlyItsOwningSubgroup` = 2.909 s; selection and CLI execution, not simply grouping.
- `ProtosTestToolCaseRejectionPublicIntegrationTest.repeatedExactRefFailsBeforeScheduling` = 2.637 s; actual public CLI rejection boundary.

### Group A — Test Tool progress and corpus traversal (candidate)

The two progress-test classes total **25.680 s**. Read-only source inspection found a potentially superlinear repeated-pass structure in `protos/tools/test/RepositorySuite.protos` (`progressGroupNames` nested descriptor/Case scans, `Arrays.findIndex`, repeated spread-based array reconstruction) and `protos/tools/test/Main.protos` (re-filtering the entire `logicalProgressCaseKeys` set per suite). This is **structural evidence of candidate algorithmic costs**, not proof that these sites dominate the observed 25.680 s. Separate pure grouping, manifest/corpus parsing, CLI listing and actual logical Case execution. No reduction of Case counts or path/selector/ordered-group assertions.

### Group B — Package Tool invalid-graph and capture fixtures (candidate)

Three fixture classes (`InvalidIdentityGraph`, `InvalidGitAndEdgeGraph`, `Capture`) total **19.545 s**. `ProtosPackageToolProtosTestSupport.executeExternalCaptureFixture` creates a fresh Package-flavored Prelude and hosted execution, plus verified captured filesystem capabilities, on each vector. Distinguish immutable reusable Core/resolver material from runtime-local activation, context, Actor state, filesystem custody, selected-root ownership and fail-closed rejection. Never reduce the eight or seven distinct invalid fixtures just to improve elapsed time.

### Group C — Package Run Driver fixture preparation (candidate)

Four Run Driver classes (`MixedApplication`, `PlanningFailureCustody`, `RunDriverTest`, `FailureCustody`) total **19.353 s class time**, **14.899 s method time**? See the exact class-minus-method table below: the per-method totals are 6.307+3.657+2.788+2.147 = **14.899 s** and class-only difference = **4.454 s**. Their `ProtosPackageRunDriverTestSupport.digestExternalTemplates` is a `@BeforeAll` that builds external templates and computes identities. Class-minus-method is evidence of unaccounted-before-method overhead, not a precise JUnit lifecycle attribution. Investigate whether immutable template/digest data can be safely reused while each class/test gets independent mutable trees, Process and capability ownership.

| Run Driver class | Class s | Sum method s | Difference s |
|---|---:|---:|---:|
| ProtosPackageRunDriverMixedApplicationTest | 7.322 | 6.307 | 1.015 |
| ProtosPackageRunDriverPlanningFailureCustodyTest | 4.653 | 3.657 | 0.996 |
| ProtosPackageRunDriverTest | 3.796 | 2.788 | 1.008 |
| ProtosPackageRunDriverFailureCustodyTest | 3.582 | 2.147 | 1.435 |
| **Total** | **19.353** | **14.899** | **4.454** |

### Group D — Public integration, Process/Actor/Regex (retain pending contrary evidence)

Public CLI tests reject invalid CaseRefs before scheduling, and Process/Actor/Regex tests exercise actual authority, lifecycle and destination re-materialization. Their cost is nontrivial but cannot be classified as removable without a causal, safety-preserving proof. In particular, do **not** share live `ProtosActivation`, `ProtosPolyglotExecutionContext`, Actor module state, Processes, Futures or verified filesystem custodies across independent test fixtures as an optimization.

## Specification and project boundaries

- `spec/io/PROCESS_IO.md`: Process authority, Process-local delegation and bootstrap; a default filesystem capability is local, not implicitly available from prelude/imports.
- `spec/io/FILESYSTEM.md` and `spec/io/IO_CORE.md`: filesystem authority, capture/open commitments and resource lifetimes.
- `spec/concurrency/ACTORS.md`, `spec/concurrency/PARALLEL_EXECUTION.md`: Actor-local authority and isolated parallel transfer.
- Root `AGENTS.md`, `AGENTS.work/PERFORMANCE.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/COORDINATION.md`: preserve observable semantics, the human-executor validation boundary, formal Issue ownership and publication rules.
- No modifications to `tools/java_slow_test_baseline.txt`, `Makefile`, TEST008 admission/PLAT047, historical evidence, `JAVA_TEST_JOBS`, `pom.xml` or `CHANGELOG.md` are authorized by this investigation.

## Next PERF031-E slice: source-only causal attribution, grouped, no execution

```text
TYPE=INVESTIGATION
ISSUE=guillermomolina/protos#787
SLICE=PERF031-E
PRODUCT_CHANGES=NO
TESTS_OR_BUILDS_EXECUTED_BY_AGENT=NO
COMMANDS_EXECUTED_BY_AGENT=NO
GIT_STATE_CHANGES_IN_PROTOS=NO
NEW_FORMAL_ISSUE=NO
```

Investigate Groups A/B/C in one source-only pass, plus confirm why Group D must remain actual integration coverage. Current `main` might have moved since measured source revision; compare actual latest HEAD **read-only** and do not project measurements onto changed code without checking diff. Derive a precise per-method call-path/setup-cost matrix: invariants, reusable immutable data, prohibited shared mutable/capability state, expected changed paths, and diagnostic signals that would prove or disprove the dominant cost. Rank a **grouped** implementation only when cause is adequately supported; otherwise give the human a single bounded instrumentation/measurement command proposal and stop, without agent execution. Do not create more one- or two-file micro-slices by default.

PERF031 should remain **OPEN / READY** with existing `family:PERF` and `status:ready` labels, effective priority intentionally unset, no assigned owner and no dependency on PERF032. Closing requires a defensible disposition of every materially expensive class, not mere completion of the 15-class originally selected cohort.

## Observability limitations

One full reduced-contention run is a baseline, not statistical proof of stable intrinsic CPU costs. There is no paired pre-optimization sample on the same host under equivalent conditions. The diagnostic does not isolate JVM/fork ordering or quantify Core, filesystem capture, guest interpreter/JIT, scheduling or disk I/O separately. Hypotheses in Groups A/B/C must remain hypotheses until additional focused evidence is available.
