# TOOL009-B — Final Global Legacy Cleanup Reconciliation

## Identity

- Parent workstream: `guillermomolina/protos#600`
- Cleanup slice: `guillermomolina/protos#685`
- Protos revision: `4c4aa95a5852119bd280ceb40483871d5d2cbb82`
- Protos version: `0.3.74-SNAPSHOT`
- Investigation type: repository-wide consumer reconciliation only
- Product mutations performed by this investigation: none

## Result

The final global reconciliation found that TOOL009-B is **not yet ready for
legacy cleanup implementation**.

The previously migrated Package TOML corpus is fully suite-native, but it is not
the whole Package Tool production corpus. Eight additional production Package
Tool leaves remain legacy-owned and continue to execute through the incumbent
D108 / expectation path.

```text
TOOL009B_FINAL_GLOBAL_RECONCILIATION=COMPLETE

PROTOS_REVISION=4c4aa95a5852119bd280ceb40483871d5d2cbb82
VERSION=0.3.74-SNAPSHOT

REMOVE_NOW_COUNT=0

GLOBAL_LEGACY_PRODUCTION_LEAVES=8
GLOBAL_LEGACY_PRODUCTION_CONSUMERS=211
GLOBAL_LEGACY_TRUE=116
GLOBAL_LEGACY_ERROR=95

TOOL009B_CLEANUP=BLOCKED
NEW_DESIGN_DECISION_REQUIRED=NO
FURTHER_CORPUS_MIGRATION_REQUIRED=YES
```

## Confirmed migrated ownership

The following production corpus ownership is suite-native at the reconciled
revision:

```text
Conformance             817/817
Process                  15/15
Actor                    11/11
Group                    10/10
Package TOML            102/102
RepositoryCorpusPlans    78/78
```

The Package TOML result remains valid:

```text
protos/tests/package-tool/toml-syntax/manifest.tsv
PACKAGE_SUITE_NATIVE=102
PACKAGE_LEGACY=0
```

That migration did not claim that every other Package Tool corpus had already
migrated; its durable record explicitly retained global legacy infrastructure
and routed final global liveness analysis to TOOL009-B.

## Remaining production legacy consumers

`protos/tools/test/RepositorySuite.protos` still contains eight Package Tool
production leaves bound to `protos/test/package` whose manifests retain legacy
outcome metadata.

| Production leaf | Loader | Rows | true | error |
| --- | --- | ---: | ---: | ---: |
| `protos/package-tool/version` | `case-outcomes` | 74 | 47 | 27 |
| `protos/package-tool/lock` | `case-outcomes` | 71 | 39 | 32 |
| `protos/package-tool/resolution-input` | `case-outcomes` | 12 | 6 | 6 |
| `protos/package-tool/content-identity` | `project-tree` | 12 | 12 | 0 |
| `protos/package-tool/resolution-input-lock` | `project-tree` | 2 | 2 | 0 |
| `protos/package-tool/resolution-root` | `project-tree` | 8 | 3 | 5 |
| `protos/package-tool/execution-plan` | `project-tree` | 28 | 4 | 24 |
| `protos/package-tool/project-projection` | `project-tree` | 4 | 3 | 1 |
| **Total** |  | **211** | **116** | **95** |

These are authoritative production consumers, not obsolete test-only fixture
metadata.

`src/main/java/com/guillermomolina/protos/cli/ProtosTestCorpusRegistry.java`
still registers the first three through `case-outcomes` and the remaining five
through `project-tree`.

`protos/tools/test/Main.protos` materializes those plans through:

```text
Manifest.loadCaseOutcomes(...)
Manifest.loadProjectTreeCases(...)
```

The corresponding Manifest loaders normalize retained `true` / `error`
outcomes into incumbent CaseSpec expectations rather than `suite-native`.

## Current execution-path liveness

The mixed path remains production-live:

```text
RepositorySuite leaf
  -> CorpusBinding
  -> Manifest.loadCaseOutcomes / loadProjectTreeCases
  -> legacy boolean/error CaseSpec
  -> LogicalCaseMigration.splitPlan
  -> LogicalCaseMigration.legacyPlan
  -> Runner.runD108WithResources
  -> protos/test/package execution binding
```

Therefore the following migration bridge remains a current production consumer
path and is not removable yet:

```text
LogicalCaseMigration.splitPlan
LogicalCaseMigration.legacyPlan
LogicalCaseMigration.suiteNativeSpecs
LogicalCaseMigration.sourceAssociation
LogicalCaseMigration.logicalCaseExecutorAsync
LogicalCaseMigration.neutralLegacyOutcome
LogicalCaseMigration.mergeOutcome

Main migrationExecutionSuites mixed-ownership wiring
Main legacyLifecycleObserver
Runner.runD108WithResources
Runner legacy expectation evaluation
```

The production `protos/test/package` binding still exposes:

```text
packageExecutionAsync
packageExecutionInspectAsync
packageResourceExecutionAsync
packageResourceExecutionInspectAsync
logicalCaseExecutionAsync
```

The coexistence is currently intentional because both incumbent Package Tool
cases and suite-native Package TOML cases exist.

## Removal classification

No candidate satisfies all required removal conditions at this revision:

```text
ZERO_PRODUCTION_CONSUMERS
ZERO_CURRENT_TOOL_CONSUMERS
MIGRATION_ONLY
```

For the mixed bridge and D108 path, `ZERO_PRODUCTION_CONSUMERS` is false.

### KEEP now

Keep at the reconciled revision:

- the complete live `LogicalCaseMigration` bridge listed above;
- the incumbent D108 Main branch;
- legacy lifecycle/progress presentation used by D108;
- legacy CaseSpec expectation evaluation needed by the 211 current cases;
- `Manifest.loadCaseOutcomes` and `Manifest.loadProjectTreeCases`;
- Package execution, inspection and resource execution facilities;
- independently authoritative Process, Actor and Group host/runtime facilities;
- suite-native Process, Actor, Group and Package Logical Case facilities;
- migration characterization whose subject is still production-live.

### REMOVE now

```text
REMOVE_NOW={}
```

The earlier dead helper `LogicalCaseMigration.selectSuiteNativeSpecs` was
already removed in the prior TOOL009-B checkpoint and is not reconsidered here.

## Host/runtime facility boundary

Completion of a corpus migration is not sufficient evidence to delete the host
execution facility that formerly served that corpus.

Current Java integration coverage deliberately retains independent exact,
inspection, Process Snapshot, Actor, Group and Package bootstrap/runtime
characterization. Those facilities must be assessed by their own authoritative
consumers and contracts rather than removed by association with TOOL009.

In particular, the suite-native facilities are authoritative current machinery,
while the exact/inspection/resource facilities are not demonstrated to be
migration-only simply because one or more corpora have migrated.

## Next gate

TOOL009-B remains blocked on migration/reconciliation of exactly these remaining
Package Tool production leaves:

```text
PACKAGE_TOOL_NON_TOML_LEAVES_TO_MIGRATE=8
PACKAGE_TOOL_NON_TOML_CASES_TO_MIGRATE=211
  TRUE=116
  ERROR=95
```

No new language/tooling design decision is required by this reconciliation.

After those 211 cases are migrated, rerun the global TOOL009-B consumer trace.
At that point the mixed-ownership bridge itself is expected to become the
primary cleanup candidate, but D108 internals and host-specific facilities must
still receive an independent zero-consumer / ownership proof before removal.

## Coordination consequence

`guillermomolina/protos#685` should remain open and move back to
`status:blocked` until the eight Package Tool non-TOML leaves no longer consume
the incumbent path.

Parent `guillermomolina/protos#600` remains open: the TOOL009 closure condition
that no migration-only scars remain cannot yet be evaluated after final corpus
ownership cutover because that cutover is incomplete.

## Investigation discipline

This reconciliation performed no product edits, removals, version changes,
builds, tests, programs, or Git mutations in `guillermomolina/protos`.
