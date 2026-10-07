# TOOL009-B — Post-Migration Global Legacy Consumer Reconciliation

## Identity

- Parent workstream: `guillermomolina/protos#600`.
- Cleanup slice: `guillermomolina/protos#685`.
- Protos revision: `cd32d7d3228f88c6ebda65f357787a692b89c666`.
- Protos version: `0.3.76-SNAPSHOT`.
- Investigation type: final repository-wide post-migration consumer reconciliation.
- Product mutations performed by this investigation: none.
- Builds/tests/programs executed by this investigation: none.

This record supersedes the cleanup-readiness conclusion of
`TOOL009B_FINAL_GLOBAL_LEGACY_RECONCILIATION.md` only for the newer product
revision above. The older record remains valid historical evidence for
`4c4aa95a5852119bd280ceb40483871d5d2cbb82`.

## Result

All currently registered repository production Cases are suite-native.

```text
TOOL009B_POST_MIGRATION_RECONCILIATION=COMPLETE

PROTOS_REVISION=cd32d7d3228f88c6ebda65f357787a692b89c666
VERSION=0.3.76-SNAPSHOT

GLOBAL_PRODUCTION_SUITE_NATIVE_CASES=1244
GLOBAL_PRODUCTION_LEGACY_CASES=0

REMOVE=12
KEEP=21
REQUIRES_INVESTIGATION=0

TOOL009B_CLEANUP_SET_ESTABLISHED=YES
NEW_DESIGN_DECISION_REQUIRED=NO
TOOL009B_POST_MIGRATION_RECONCILIATION=READY_FOR_CLEANUP
```

The decisive result is not that every historical Test Tool mechanism can be
deleted. It is that the migration-only mixed repository execution path is now
dead, while independently ratified lower Test Tool capabilities remain current.

## Global production corpus inventory

Direct inspection of the registered RepositorySuite leaves, their CorpusBindings,
all production manifests, and `RepositoryCorpusPlans.protos` produced:

| Corpus family | Cases | Suite-native | Legacy |
| --- | ---: | ---: | ---: |
| main conformance | 817 | 817 | 0 |
| Process | 15 | 15 | 0 |
| Actor | 11 | 11 | 0 |
| Group | 10 | 10 | 0 |
| Package TOML | 102 | 102 | 0 |
| Standard Library explicit plans | 78 | 78 | 0 |
| Package case-outcomes | 157 | 157 | 0 |
| Package project-tree | 54 | 54 | 0 |
| **TOTAL** | **1244** | **1244** | **0** |

No currently registered production leaf generates a non-suite-native CaseSpec.

## Main mixed-path result

`Main.protos` still contains the M2 coexistence architecture:

```text
plan
  -> LogicalCaseMigration.splitPlan
  -> logical Case execution
  -> incumbent D108 execution
  -> neutral legacy outcome when empty
  -> mergeOutcome
```

At the reconciled revision every production split has an empty legacy side.

Therefore the following repository-production machinery is now migration-only:

```text
LogicalCaseMigration.isSuiteNativeSpec
LogicalCaseMigration.splitPlan
LogicalCaseMigration.legacyPlan
LogicalCaseMigration.suiteNativeSpecs
LogicalCaseMigration.neutralLegacyOutcome
LogicalCaseMigration.mergeOutcome

Main mixed partition / migrationExecutionSuites legacy half
Main incumbent D108 production branch
Main completedRuns / primaryRun / suiteD108 reconciliation
Main legacyCaseDisplayReference
Main legacyLifecycleObserver
```

Current Logical Case behavior hosted temporarily in
`LogicalCaseMigration.sourceAssociation` and
`LogicalCaseMigration.logicalCaseExecutorAsync` remains authoritative and must
be re-homed before the migration module is deleted.

## D108 and expectation engine boundary

The repository production consumer count for the incumbent D108 path is now
zero, but D108 itself is not classified as migration residue.

Keep:

```text
Runner.runD108WithResources
D108/D114/D116 infrastructure outcome and fail-stop machinery
resourceful execution/inspection
legacy expectation evaluation as retained Test Tool internal capability
generic exact execution facilities
```

These mechanisms have independent TOOL002/D077/D108/D114/D116 authority and
component/API consumers. TOOL009-B may disconnect repository suite execution
from them, but may not delete them merely because the current repository corpus
is suite-native.

## Manifest compatibility boundary

Current production manifests have no retained `true`/`error` rows in the
Package TOML, case-outcomes, or project-tree corpora.

However, the compatibility branches remain KEEP:

```text
packageTomlCaseSpec true/error compatibility = KEEP
caseOutcomeSpec true/error compatibility = KEEP
projectTreeCaseSpec true/error compatibility = KEEP
```

The migration publications explicitly added `suite-native` alongside the
existing forms rather than retracting them, and no later authority removes those
generic loader contracts.

Zero current corpus rows is therefore not sufficient removal evidence.

## Project-tree CaseAuthority boundary

The former whole-source D108 CaseAuthority execution route is dead:

```text
ProtosTestCaseAuthorityExecutionScope
ProtosTestCaseAuthorityExecutionFacility
caseAuthorityExecutionAsync CorpusBinding publication
five packageTool*CaseAuthorityExecutionAsync bootstrap slots
old CaseAuthority attempt execution/envelope machinery
ProtosTestCaseAuthorityAttemptCompletion
```

These are REMOVE candidates after shared current primitives are extracted.

The suite-native project-tree path still depends on D129/D133/D134 physical
authority semantics. Keep/re-home:

```text
trusted host casesRoot ownership
CaseAuthority descriptor validation
resolveAuthorityRoot(...)
secure read-only confinement
same-Process projectTreeFilesystem provisioning
```

The current execution path is:

```text
ProtosTestLogicalCaseExecutionFacility
  -> ProtosTestLogicalCaseAttemptBridge
  -> trusted project-tree casesRoot
  -> shared physical authority confinement
  -> fresh Process
  -> projectTreeFilesystem
  -> selected Test.call()
```

## Process / Actor / Group / Package facilities

Current logical facilities remain authoritative production machinery.

Keep:

```text
ProtosProcessSnapshotExecution
ProtosProcessSnapshotLogicalCaseExecutionFacility
Actor Logical Case execution
Group Logical Case execution
Package Logical Case execution
```

Also keep the independently owned exact/inspection/resource execution facilities
for Process/Actor/Group/Package where D125/D135/D077/D108 still own the
capability. Corpus migration alone is not deletion authority.

## Cleanup classification

The material candidate inventory reconciles to:

```text
REMOVE=12
KEEP=21
REQUIRES_INVESTIGATION=0
```

The REMOVE classes are:

1. suite-native marker predicate used only for mixed partitioning;
2. `splitPlan`;
3. `legacyPlan`;
4. `suiteNativeSpecs`;
5. neutral incumbent outcome;
6. legacy/logical merge outcome;
7. Main mixed partition/planning branch;
8. Main incumbent D108 repository-production branch;
9. Main legacy lifecycle/display adapter;
10. whole-source CaseAuthority execution scope/facility/publication;
11. old CaseAuthority attempt execution/envelope machinery;
12. stale migration-only comments/documentation.

The KEEP classes cover current Logical Case execution, D108 and expectation APIs,
manifest compatibility, progress/lifecycle, D125 bindings, D135 Process
bootstrap, resourceful/exact facilities, current project-tree authority
primitives, and current Process/Actor/Group/Package logical facilities.

No material candidate remains unclassified.

## Test consequences

Tests that exist solely to prove the deleted mixed bridge should be removed with
their subject.

Tests that currently combine dead migration assertions with current logical
invariants should be rewritten to the surviving owner.

In particular:

- remove migration-only `splitPlan`/`legacyPlan`/`mergeOutcome` fixture
  assertions;
- rewrite Main architecture tests that require one
  `Runner.runD108WithResources(...)` production call;
- rewrite lifecycle coverage from dual legacy/logical observers to the surviving
  invocation-wide logical observer;
- remove whole-source CaseAuthority facility coverage with the deleted wrapper,
  while preserving unique physical-authority/confinement evidence under the
  current logical path;
- keep independent D108, resource, exact-execution, Process bootstrap,
  Logical Case, Manifest compatibility, and project-tree authority coverage.

## Safe implementation order

The established dependency order is:

```text
1. Re-home current Logical Case helpers from LogicalCaseMigration.
2. Re-home shared CaseAuthority descriptor/confinement helpers from legacy wrappers.
3. Cut Main from mixed execution to logical-only repository execution.
4. Remove partition / neutral / merge migration machinery.
5. Remove whole-source project-tree CaseAuthority execution wrappers/bindings.
6. Reconcile tests against surviving current owners.
7. Remove/rewrite stale migration-only comments/docs/imports.
8. Human Executor runs focused and full required validation.
9. Publish product cleanup.
10. Perform final zero-reference/closure reconciliation.
```

No new Dxxx/PLATxxx decision is required by this cleanup.

## Expected post-cleanup repository execution architecture

```text
RepositorySuite
  -> CorpusBinding / planLoader
  -> suite-native TestPlans
  -> fail-closed native-plan validation
  -> source associations
  -> Logical Case discovery
  -> CasePlan
  -> ExecutionRequirementId
  -> logicalCaseExecutionAsync
  -> LogicalCaseRunner
  -> LogicalCaseResult
  -> logical-only final TestRunOutcome
  -> Progress.finishInvocation
```

Independently authoritative lower Test Tool APIs may continue to exist outside
that repository corpus execution path.

## Coordination consequence

`guillermomolina/protos#685` is no longer blocked by production legacy
consumers.

It should remain open/in-progress for the bounded cleanup implementation and
validation described above.

Parent `guillermomolina/protos#600` remains open until that cleanup is
published and TOOL009's no-migration-scars closure condition is revalidated.

```text
TOOL009B_POST_MIGRATION_RECONCILIATION=READY_FOR_CLEANUP
TOOL009_PARENT_CLOSURE=NOT_YET
```
