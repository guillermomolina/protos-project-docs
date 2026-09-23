# TOOL009-B — Final Legacy Test Tool Migration Cleanup

## Identity

- Parent workstream: `guillermomolina/protos#600`.
- Cleanup issue: `guillermomolina/protos#685`.
- Parent Protos revision: `cd32d7d3228f88c6ebda65f357787a692b89c666`.
- Cleanup Protos revision: `f92489d357d863c33b62c1ca4356651b16568573`.
- Version: `0.3.77-SNAPSHOT`.
- Commit: `TOOL009: remove legacy Test Tool migration scaffolding`.
- Cleanup authority: `docs/project/evidence/TOOL009/TOOL009B_POST_MIGRATION_GLOBAL_RECONCILIATION.md` at `32f2fe2df8dada991400290a07e77c0183e3a957`.

## Result

The bounded TOOL009-B cleanup selected by the post-migration reconciliation is implemented and published.

```text
TOOL009B_FINAL_LEGACY_CLEANUP=COMPLETE

REPOSITORY_EXECUTION_LOGICAL_ONLY=YES
MIXED_PLAN_PARTITION_REMOVED=YES
REPOSITORY_D108_BRANCH_REMOVED=YES
LEGACY_OUTCOME_MERGE_REMOVED=YES
LEGACY_LIFECYCLE_ADAPTER_REMOVED=YES
WHOLE_SOURCE_PROJECT_TREE_CASEAUTHORITY_EXECUTION_REMOVED=YES

CURRENT_LOGICAL_HELPERS_REHOMED=YES
CURRENT_PROJECT_TREE_AUTHORITY_PRIMITIVES_REHOMED=YES

GLOBAL_PRODUCTION_LEGACY_CASES=0
```

The cleanup does not alter corpus ownership. The cleanup commit changes no `manifest.tsv` file. Therefore the previously reconciled production inventory remains unchanged at 1,244 suite-native Cases and zero production legacy Cases.

## Logical-only RepositorySuite execution

`Main.protos` no longer partitions repository TestPlans into suite-native and incumbent D108 sides.

The repository execution path is now:

```text
RepositorySuite
  -> CorpusBinding / planLoader
  -> suite-native TestPlan validation
  -> source associations
  -> Logical Case discovery
  -> CasePlan
  -> ExecutionRequirementId
  -> logicalCaseExecutionAsync
  -> LogicalCaseRunner
  -> LogicalCaseResult
  -> logical-only TestRunOutcome
  -> Progress.finishInvocation
```

The repository route fails closed if a non-suite-native production Case reaches this path. It no longer falls back to D108.

`Runner.runD108WithResources` remains present and independently authoritative for non-RepositorySuite Test Tool capabilities.

## Migration bridge removal

The migration container:

`protos/tools/test/LogicalCaseMigration.protos`

is deleted.

The removed migration-only behavior includes:

- `isSuiteNativeSpec`;
- `splitPlan`;
- `legacyPlan`;
- `suiteNativeSpecs`;
- `neutralLegacyOutcome`;
- `mergeOutcome`;
- mixed legacy/logical repository planning;
- mixed final-outcome reconciliation.

The two still-current helpers formerly located there were re-homed to:

`protos/tools/test/LogicalCaseDispatch.protos`

with their current logical-only ownership:

- source-association construction;
- execution-requirement / `logicalCaseExecutionAsync` dispatch.

## Lifecycle cleanup

The repository legacy coexistence adapter is removed.

`Main.protos` no longer builds the legacy display/lifecycle side or coordinates dual legacy/logical observers. The surviving invocation-wide logical lifecycle owner remains authoritative.

## Whole-source project-tree CaseAuthority cleanup

The obsolete whole-source execution path is removed.

Deleted production classes:

```text
src/main/java/com/guillermomolina/protos/cli/ProtosTestCaseAuthorityExecutionScope.java
src/main/java/com/guillermomolina/protos/execution/ProtosTestCaseAuthorityAttemptBridge.java
src/main/java/com/guillermomolina/protos/execution/ProtosTestCaseAuthorityAttemptCompletion.java
src/main/java/com/guillermomolina/protos/execution/ProtosTestCaseAuthorityExecutionFacility.java
```

Deleted wrapper-specific tests:

```text
src/test/java/com/guillermomolina/protos/execution/ProtosTestCaseAuthorityAttemptBridgeTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosTestCaseAuthorityExecutionFacilityTest.java
```

The old project-tree `caseAuthorityExecutionAsync` bootstrap/publication route is no longer part of repository corpus execution.

## Surviving project-tree authority owner

Current D133/D134 physical project-tree authority is preserved under the suite-native Logical Case owner.

The surviving responsibilities are now owned by the current logical attempt path, including:

```text
descriptor / fixture identity validation
trusted host casesRoot ownership
resolveAuthorityRoot(...)
secure read-only confinement
projectTreeFilesystem provisioning
same-Process selected Test execution
```

The Package project-tree route therefore remains:

```text
protos/test/package
  -> Package logicalCaseExecutionAsync
  -> sourceAssociation authority descriptor
  -> trusted casesRoot
  -> physical project-tree authority
  -> projectTreeFilesystem
  -> selected Test.call()
```

without depending on the deleted whole-source CaseAuthority execution facility.

## Explicit retained functionality

The cleanup intentionally retains the independently authoritative KEEP set established by reconciliation:

```text
Runner.runD108WithResources
D108/D114/D116 outcome/fail-stop machinery
legacy expectation engine
Manifest true/error compatibility
executionAsync
executionInspectAsync
resourceExecutionAsync
resourceExecutionInspectAsync
Process Snapshot execution/bootstrap
ordinary Logical Case execution
Actor Logical Case execution
Group Logical Case execution
Package Logical Case execution
Package project-tree Logical Case execution
D129/D133/D134 physical project-tree authority semantics
```

This publication removes migration scaffolding, not these lower-level APIs.

## Published diff

The product commit changes 34 files:

```text
ADDED=1
REMOVED=7
MODIFIED=26

ADDED:
  protos/tools/test/LogicalCaseDispatch.protos

REMOVED:
  protos/tools/test/LogicalCaseMigration.protos
  src/main/java/com/guillermomolina/protos/cli/ProtosTestCaseAuthorityExecutionScope.java
  src/main/java/com/guillermomolina/protos/execution/ProtosTestCaseAuthorityAttemptBridge.java
  src/main/java/com/guillermomolina/protos/execution/ProtosTestCaseAuthorityAttemptCompletion.java
  src/main/java/com/guillermomolina/protos/execution/ProtosTestCaseAuthorityExecutionFacility.java
  src/test/java/com/guillermomolina/protos/execution/ProtosTestCaseAuthorityAttemptBridgeTest.java
  src/test/java/com/guillermomolina/protos/execution/ProtosTestCaseAuthorityExecutionFacilityTest.java

MANIFEST_TSV_CHANGED=0
```

The implementation version advances from `0.3.76-SNAPSHOT` to `0.3.77-SNAPSHOT`.

## Human Executor validation

The Human Executor explicitly reported that all requested tests passed before publication and that the cleanup commit was pushed.

```text
HUMAN_EXECUTOR_VALIDATION=PASS
HUMAN_EXECUTOR_TESTS=PASS
PRODUCT_COMMIT_PUSH=PASS
```

These are maintainer-reported execution results; they are not inferred from repository inspection.

At the latest reconciliation, GitHub Actions CI run `35808175674` for the exact cleanup SHA is still `in_progress`. No remote CI PASS is inferred until GitHub reports a conclusion.

## Closure consequence

The implementation work selected by TOOL009-B is complete, but issue closure still requires the bounded final zero-reference / no-migration-scars reconciliation established in the implementation handoff.

Next gate:

```text
TOOL009-B FINAL ZERO-REFERENCE / NO-MIGRATION-SCARS CLOSURE RECONCILIATION
```

That reconciliation should verify against `f92489d357d863c33b62c1ca4356651b16568573`:

- deleted migration symbols have no authoritative current references;
- RepositorySuite remains 1,244/1,244 suite-native;
- retained D108/expectation/exact/resource/Manifest compatibility APIs remain present;
- no obsolete whole-source CaseAuthority bootstrap or wrapper remains;
- TOOL009-B closure conditions are satisfied.

If that passes, `#685` can close. Parent `#600` then requires its own final TOOL009 closure reconciliation.
