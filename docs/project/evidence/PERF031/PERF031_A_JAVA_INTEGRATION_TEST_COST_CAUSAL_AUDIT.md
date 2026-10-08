# PERF031-A — causal audit of expensive Java integration tests

Date: 2026-10-08

## Identity and scope

```text
WORK_ITEM=PERF031-A
PARENT_ISSUE=guillermomolina/protos#787
TYPE=INVESTIGATION
PROTOS_SOURCE_REVISION=2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3
EVIDENCE_REPOSITORY=guillermomolina/protos-project-docs
PRODUCT_CHANGES=NONE
TEST_EXECUTION=NONE
BENCHMARK_EXECUTION=NONE
COMMAND_EXECUTION=NONE
GITHUB_ISSUE_MUTATIONS_DURING_INVESTIGATION=NONE
```

This is a snapshot of a read-only source/CI-log investigation. It is not an implementation validation report and it does not claim new local timings. The parent remains independently open/ready; TEST008 and PLAT047 have separate ownership. The recommendation below is an implementation candidate, not a measured performance win or a change to normative semantics.

## Current operational contract

- `Makefile` defaults to `JAVA_TEST_JOBS=6` for class-parallel JUnit execution, with a separate serial lane and a single reduced-contention confirmation mechanism.
- `tools/java_slow_test_baseline.txt` holds seven reviewed **normalized, reduced-contention** class expectations. These values are not fresh timings and must not be compared directly as if they were co-run CI wall times.
- Following the 2026-10-05 owner amendment retained in PLAT047, slow-test telemetry is advisory locally and on CI. Functional Java/Protos execution remains authoritative/fail-closed.
- Do not change TEST008 thresholds, baseline, normalization, machine controls, Makefile concurrency, assertions or semantic coverage to address PERF031.

Read-only authorities: [Makefile](https://github.com/guillermomolina/protos/blob/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3/Makefile), [baseline](https://github.com/guillermomolina/protos/blob/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3/tools/java_slow_test_baseline.txt), [PLAT047 maintained decision](https://github.com/guillermomolina/protos-project-docs/blob/a872b976bc76840e0cb22a167cef6a02798276e4/docs/project/decisions/platform/PLAT047_PORTABLE_JAVA_SLOW_TEST_ADMISSION_ARCHITECTURE.md), [TEST008-A historical audit](https://github.com/guillermomolina/protos-project-docs/blob/a872b976bc76840e0cb22a167cef6a02798276e4/docs/project/evidence/TEST008/TEST008_A_CI_LOCAL_TIMING_ADMISSION_INVESTIGATION.md), and [TEST008-B closure](https://github.com/guillermomolina/protos-project-docs/blob/a872b976bc76840e0cb22a167cef6a02798276e4/docs/project/evidence/TEST008/TEST008_B_FINAL_CLOSURE.md).

## Exact historical CI evidence recovered

GitHub Actions job logs were retrieved through read interfaces, not by executing CI, tests, or shell commands:

- [CI #2110, run 37140303155, job 111253181076](https://github.com/guillermomolina/protos/actions/runs/37140303155): 499 reported classes, 2 unallowlisted slow classes, 5 over-budget.
- [CI #2111, run 37140570878, job 111253981075](https://github.com/guillermomolina/protos/actions/runs/37140570878): 499 reported classes, 10 unallowlisted slow classes, 6 over-budget; 2,402 Java parallel tests and 7 serial tests, zero test failures/errors in both lanes.

Both checks used the *historical* absolute-seconds guard. The numbers below are Surefire test-class wall times under contention, **not CPU attribution, current costs, or isolated causal measurements**.

| Seven approved-class identities (simple class names) | CI #2110 seconds | CI #2111 seconds | Current normalized baseline seconds |
|---|---:|---:|---:|
| ProtosTestToolFileSelectionPublicIntegrationTest | 14.15 | 17.40 | 3.8 |
| ProtosWorkspaceRunCliTest | 11.77 | 22.00 | 1.6 |
| ProtosFilesystemLibraryConformanceTest | 22.99 | 35.18 | 1.0 |
| ProtosExternalPackagePlanningPreflightTest | 56.98 | 87.85 | 5.7 |
| ProtosPackageExecutionPlanAdapterTest | 24.99 | 38.50 | 2.0 |
| ProtosWorkspacePackageAuthorityIsolationIntegrationTest | 18.48 | 30.38 | 1.3 |
| ProtosWorkspacePackagePreflightTest | 21.61 | 25.78 | 1.0 |

The old CI #2110 unallowlisted offenders were `ProtosTestToolSuiteGraphTest` (15.39 s) and `ProtosTomlEncoderModuleTest` (10.12 s). CI #2111's ten unallowlisted offenders were:

| Exact historical identity (simple class name) | CI #2111 seconds | Current-source disposition |
|---|---:|---|
| ProtosPackageContentVerificationTest | 13.42 | Authentic verification/custody integration; no repair established |
| ProtosPackageTestLogicalCaseExecutionFacilityTest | 11.90 | Multiple independent Core bootstrap calls; authority-sensitive, defer |
| ProtosTestToolManifestPlanTest | 16.94 | Already shares `SharedPrelude.INSTANCE`; retain |
| ProtosTestToolResourceCatalogSchemaTest | 10.79 | Repeated Core bootstrap in fixture; repair candidate |
| ProtosTestToolResourceRequirementsSchemaTest | 14.95 | Repeated Core bootstrap in fixture; repair candidate |
| ProtosTestToolSequentialRunnerTest | 11.39 | Repeated Core bootstrap in fixture; repair candidate |
| ProtosTestToolSuiteGraphTest | 21.62 | Repeated Core bootstrap in fixture; repair candidate |
| ProtosTomlEncoderModuleTest | 14.44 | Historical exact class no longer found at current HEAD; do not guess successor |
| ProtosTomlParserFloatTemporalRegressionTest | 12.73 | Already class-scoped Core; retain |
| ProtosTomlParserModuleTest | 12.29 | Historical exact class no longer found at current HEAD; do not guess successor |

Do not reinterpret the offender count as 17 additional classes. The approved seven are reviewed separately, and CI #2111 supplies ten *additional* unallowlisted identities; several also appear in CI #2110.

## Seven-class source-grounded matrix

This section contains the full required causal dimensions in compact form. Confidence refers to source-structural findings, not measured performance attribution. Main classification: A=proportional ordinary test; B=legitimate integration cost; C=redundant initialization. Exposure to historical class-parallel contention is high for every entry.

| TEST_CLASS | TEST_COUNT | INVARIANT_PROVED | SHARED_SETUP / REPEATED_SETUP / CORE_BOOTSTRAP | PROCESS_OR_RUNTIME_STARTUP / FILESYSTEM_WORK / COMPILATION_OR_MODULE_LOADING | ISOLATION_REQUIREMENTS | HISTORICAL_COST_EVIDENCE | STATIC_COST_HYPOTHESIS / OPTIMIZATION_OPPORTUNITY | EVIDENCE_CONFIDENCE / RECOMMENDATION |
|---|---:|---|---|---|---|---|---|---|
| ProtosTestToolFileSelectionPublicIntegrationTest | 1 | Public exact file selection, four real Test Tool cases, stdout/stderr | Constants; CLI-run initialization; indirect Core | In-process CLI invocation, runtime startup and Test Tool corpus | Public-entry integration must survive | 14.15/17.40 s | Real guest CLI execution plus contention; no justified test cut | High structural; B, retain |
| ProtosWorkspaceRunCliTest | 7 | CLI entry/argument contract, exact materialization, errors, workspace run | Case-specific temp projects; indirect Core via CLI | Repeated workspace/Package execution, file copies and manifests | Project, provider, error and application authority must remain distinct | 11.77/22.00 s | Valid integration coverage; no established avoidable setup | High structural; B, retain |
| ProtosFilesystemLibraryConformanceTest | 12 | Read/write Futures, cancel/close/error precedence, bounded chunks | One `@BeforeAll` Prelude; fresh activations and backends | Guest conformance sources, temp files and controlled resource operations | Every file/capability lifecycle independent | 22.99/35.18 s | Proportional conformance; 131,089-byte write proves three bounded chunks of max 65,536 bytes | High structural; A, retain workload |
| ProtosExternalPackagePlanningPreflightTest | 2 | Verified borrowed custody, source removal, external V2 plan, process termination | Separate verification/planning bootstraps; additionally redundant `assertBorrowedCustodiesStillOpen` Prelude preparation | Multiple verified-package Processes, capture/read/delete, planning guest modules | Verification and planning Process/custody distinct; assertion activation fresh | 56.98/87.85 s | Share *only* assertion-helper Prelude, never custody/Process/runtime | High structural, gain unmeasured; C, bounded repair |
| ProtosPackageExecutionPlanAdapterTest | 2 | Detached immutable host plan, malformed-plan rejection | One `@BeforeAll` tool Prelude; plan per test | Two hosted fixture Processes and separate plan builds | Mutation in one plan must not contaminate the other | 24.99/38.50 s | Legitimate separate mutable plans | High structural; B, retain |
| ProtosWorkspacePackageAuthorityIsolationIntegrationTest | 4 | Tool/Application Process boundary, project FS non-leak, Network grant | Copied projects; repeated real preflights | Distinct Processes, bounded host/capability setup, app source loading | Different Tool and Application Process identities; networkless/default and explicit grant | 18.48/30.38 s | Expensive integration is part of invariant; no source proof for removal | High structural; B, retain |
| ProtosWorkspacePackagePreflightTest | 2 | Detached plan, immutable metadata, stale-plan fail-closed termination | One class-level RuntimeHost; two distinct preflights | Fresh Tool Process and Core per preflight; manifest/lock reads | Failure and success must have separate Process state | 21.61/25.78 s | Tool execution intrinsic to invariant; RuntimeHost already shared | High structural; B, retain |

Current-source anchors (at exact Protos revision above):

- [Public Test Tool CLI integration](https://github.com/guillermomolina/protos/blob/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3/src/test/java/com/guillermomolina/protos/cli/ProtosTestToolFileSelectionPublicIntegrationTest.java)
- [Workspace CLI](https://github.com/guillermomolina/protos/blob/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3/src/test/java/com/guillermomolina/protos/cli/ProtosWorkspaceRunCliTest.java)
- [Filesystem conformance](https://github.com/guillermomolina/protos/blob/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3/src/test/java/com/guillermomolina/protos/conformance/ProtosFilesystemLibraryConformanceTest.java)
- [External planning](https://github.com/guillermomolina/protos/blob/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3/src/test/java/com/guillermomolina/protos/execution/ProtosExternalPackagePlanningPreflightTest.java)
- [Plan adapter](https://github.com/guillermomolina/protos/blob/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3/src/test/java/com/guillermomolina/protos/execution/ProtosPackageExecutionPlanAdapterTest.java)
- [Authority isolation](https://github.com/guillermomolina/protos/blob/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3/src/test/java/com/guillermomolina/protos/execution/ProtosWorkspacePackageAuthorityIsolationIntegrationTest.java)
- [Preflight](https://github.com/guillermomolina/protos/blob/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3/src/test/java/com/guillermomolina/protos/execution/ProtosWorkspacePackagePreflightTest.java)

## Shared causal owners and normative constraints

### Candidate: class-local preparation reuse in Test Tool fixtures

[SuiteGraphTest](https://github.com/guillermomolina/protos/blob/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3/src/test/java/com/guillermomolina/protos/execution/ProtosTestToolSuiteGraphTest.java), [ResourceCatalogSchemaTest](https://github.com/guillermomolina/protos/blob/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3/src/test/java/com/guillermomolina/protos/execution/ProtosTestToolResourceCatalogSchemaTest.java), [ResourceRequirementsSchemaTest](https://github.com/guillermomolina/protos/blob/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3/src/test/java/com/guillermomolina/protos/execution/ProtosTestToolResourceRequirementsSchemaTest.java), and [SequentialRunnerTest](https://github.com/guillermomolina/protos/blob/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3/src/test/java/com/guillermomolina/protos/execution/ProtosTestToolSequentialRunnerTest.java) bootstrap Core in repeatable fixture paths. The resolved bundled Test Tool/Shared/Standard Library paths are class-invariant in these observed test helpers. A class-local lazy Prelude with a **new module activation per fixture** is the smallest supported repair. The existing [ManifestPlanTest `SharedPrelude.INSTANCE` pattern](https://github.com/guillermomolina/protos/blob/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3/src/test/java/com/guillermomolina/protos/execution/ProtosTestToolManifestPlanTest.java) demonstrates the shape already used in the repository.

### Candidate: assertion-only Core in external package planning

`ProtosExternalPackagePlanningPreflightTest.assertBorrowedCustodiesStillOpen` creates a new Core Prelude to materialize each custody for post-termination assertions. Sharing **only the helper's Prelude**, with a fresh activation for every invocation, avoids a duplicate helper bootstrap without changing independent verification, planning, captured Filesystem custody, deletion, resource scopes, Process termination, or transferred capability ownership.

### Non-candidates and deferrals

- The filesystem conformance suite already bootstraps once for its class. Its bounded large-write case proves a real chunk-boundary invariant.
- Plan adapter and workspace preflight already share class-level preparation where safe; separate Processes/plans are meaningful.
- Workspace authority isolation and public CLI startup are the public integration boundary; replacing with mocks or a single shared Process would remove evidence.
- `ProtosPackageTestLogicalCaseExecutionFacilityTest` contains multiple per-test Core bootstraps but also authority-sensitive asynchronous Case execution and a composed resolver. Evaluate separately if still relevant after the bounded repair; do not automatically mix it into the first implementation.
- Historic TOML offenders whose exact class source was removed cannot be assigned a current implementation owner by filename guesswork.

Normative authority: [MODULES.md](https://github.com/guillermomolina/protos/blob/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3/spec/semantics/MODULES.md) permits immutable shared standard preparation but forbids shared mutable Actor/module state; [PROCESS_IO.md](https://github.com/guillermomolina/protos/blob/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3/spec/io/PROCESS_IO.md) defines Process and local authority; [FILESYSTEM.md](https://github.com/guillermomolina/protos/blob/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3/spec/io/FILESYSTEM.md) owns filesystem lifecycle/capabilities. [ProtosPrelude](https://github.com/guillermomolina/protos/blob/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3/src/main/java/com/guillermomolina/protos/runtime/ProtosPrelude.java) constructs fresh execution context, ActorModuleState and domain in `newModuleActivation()`.

## Implementation recommendation: one grouped PERF031-B batch

```text
NEXT_SLICE=PERF031-B
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos
PATCH_SCOPE=FIVE_JAVA_TEST_FILES_ONLY
CHANGE_PRODUCTION_SOURCE=NO
CHANGE_TEST008=NO
CHANGE_TEST_ASSERTIONS=NO
CHANGE_LANGUAGE_SEMANTICS=NO
```

Files:

1. `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolSuiteGraphTest.java`
2. `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolResourceCatalogSchemaTest.java`
3. `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolResourceRequirementsSchemaTest.java`
4. `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolSequentialRunnerTest.java`
5. `src/test/java/com/guillermomolina/protos/execution/ProtosExternalPackagePlanningPreflightTest.java`

Transform each Test Tool fixture's repeated Prelude bootstrap into one class-local lazy immutable standard preparation, retaining fresh activations/Actor state and all existing assertions, exact input files, filesystem authorities and lifecycles. Transform only the custody-assertion helper bootstrap in External Planning. Leave package-content verification and planning Process creation alone. Do not add a global static Prelude cache or shared mutable guest state. If any source or semantic constraint prevents safe sharing, leave that candidate unchanged and document the reason.

Expected effect: fewer full Core loads and compilation/initialization operations along repaired test-only fixture paths. This is a causal hypothesis with a concrete reduction in bootstrap invocation count; **no percentage or seconds saving is asserted**. A one-time human-executed same-host reduced-contention baseline/after comparison may help, but the primary functional test invariants must still pass; class-parallel CI timing is not a valid isolated A/B result. Stop and revert a candidate if it introduces test-order dependence, mutable cross-test sharing, authority leakage, or regression. Do not manufacture extra micro-slices or new formal Issues.

Agent-editor/human-executor: the implementation agent edits the five owned files in the human's already-at-HEAD `guillermomolina/protos` checkout; the human executes focused tests, required broader validation, `git diff --check`, commit and push. The agent must not run shell commands or publish Protos source. Only after human-reported substantive validation green may the agent derive the then-current next `pom.xml` patch SNAPSHOT version and matching root `CHANGELOG.md` entry for the same final commit. Avoid repeated full-suite runs solely due to metadata finalization.

## Closure and routing

```text
PERF031_A_TYPE=INVESTIGATION
SOURCE_AUDIT=COMPLETE
REVIEWED_CLASSES=7
ADDITIONAL_CI_CLASSES_IDENTIFIED=10
INTRINSIC_TEST_COST_PROVEN=PARTIAL
REDUNDANT_WORK_FOUND=YES
RUNNER_CONTENTION_RELEVANT=YES
SEMANTIC_COVERAGE_PRESERVED_BY_PROPOSAL=YES
CURRENT_PERFORMANCE_EVIDENCE_SUFFICIENT=NO
HUMAN_MEASUREMENT_REQUIRED=YES
IMPLEMENTATION_JUSTIFIED=YES
IMPLEMENTATION_BATCHES=1
NEXT_IMPLEMENTATION_REPOSITORY=guillermomolina/protos
NEW_ISSUE_REQUIRED=NO
PERF031_CLOSURE_RECOMMENDED=NO
PRODUCT_CHANGES=NONE
TEST_EXECUTION=NONE
COMMAND_EXECUTION=NONE
```

`SOURCE_AUDIT=COMPLETE` applies to the seven required current-HEAD classes and their relevant helpers, not an exhaustive new audit of every Java test. No replacement source was established for two removed historical TOML identities. The remaining authority-sensitive Test Tool candidate, any unresolved expensive current class, and the effect of PERF031-B must be reconciled before issue closure. There is no owner-approved claim of full PERF031 completion.
