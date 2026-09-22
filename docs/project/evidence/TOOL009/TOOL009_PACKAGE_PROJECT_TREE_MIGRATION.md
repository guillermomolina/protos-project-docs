# TOOL009 — Package project-tree suite-native migration

## Identity

- Parent workstream: `guillermomolina/protos#600`.
- Cleanup slice: `guillermomolina/protos#685`.
- Parent Protos revision: `a116176abd69ddf85e8ee52b623e90d48ef3a11e`.
- Migration Protos revision: `cd32d7d3228f88c6ebda65f357787a692b89c666`.
- Version: `0.3.76-SNAPSHOT`.
- Commit: `TOOL009: migrate Package project-tree corpora to suite-native`.
- Readiness authority: `docs/project/evidence/TOOL009/TOOL009_PACKAGE_NON_TOML_READINESS.md` at `34035efdbd595de0b1517bbe6485c3169c840d44`.

## Result

Publication 2 of the Package non-TOML migration is complete.

```text
TOOL009_PACKAGE_PROJECT_TREE_MIGRATION=COMPLETE

CONTENT_IDENTITY_SUITE_NATIVE=12
RESOLUTION_INPUT_LOCK_SUITE_NATIVE=2
RESOLUTION_ROOT_SUITE_NATIVE=8
EXECUTION_PLAN_SUITE_NATIVE=28
PROJECT_PROJECTION_SUITE_NATIVE=4

PROJECT_TREE_SUITE_NATIVE=54
PROJECT_TREE_LEGACY=0

CASE_OUTCOMES_SUITE_NATIVE=157
CASE_OUTCOMES_LEGACY=0

PACKAGE_TOML_SUITE_NATIVE=102
PACKAGE_TOML_LEGACY=0

PACKAGE_KNOWN_SUITE_NATIVE=313
PACKAGE_KNOWN_LEGACY_CASES=0
```

Direct inspection of all five project-tree manifests confirms every data row now carries the `suite-native` marker.

## Project-tree Logical Case authority adapter

The publication implements the bounded authority adapter selected by the readiness investigation.

The established D133/D134 physical project-tree authority is preserved while selected-Test Logical Case execution is moved into the same fresh semantic Process.

The published flow is:

```text
project-tree CaseSpec
  -> inert CaseAuthority descriptor
  -> sourceAssociation
  -> logical Case discovery preserves association
  -> Package logical Case execution
  -> host-owned trusted casesRoot selected
  -> existing project-tree confinement/provisioning reused
  -> fresh read-only physical authority
  -> same fresh Process
  -> projectTreeFilesystem installed
  -> suite declaration
  -> selected Test resolution
  -> selected Test.call() exactly once
```

The implementation does not introduce a second CaseAuthority model.

## Authority transport

`LogicalCaseMigration.sourceAssociation` now preserves the existing project-tree CaseAuthority descriptor as an optional third inert association element.

Conceptually:

```text
ordinary:
  [corpusId, sourcePath]

project-tree:
  [corpusId, sourcePath, authorityDescriptor]
```

No live host Path, Filesystem, capability, execution facility, or closure is transported in the plan data.

The trusted physical `casesRoot` remains selected and owned by the host.

## Project-identity-safe logical source ownership

Project-tree corpus membership can reuse the same physical fixture source under different project identities.

The published Test Tool bookkeeping therefore distinguishes logical source identity by source plus fixture/project authority identity instead of collapsing every row solely by source path.

This preserves distinct logical Cases for shared fixture sources such as the repeated execution-plan fixtures.

## Physical authority reuse

The logical adapter reuses the existing D133/D134 confinement/descriptor-validation machinery rather than duplicating it.

The product publication adjusts visibility only where required so the logical path can share the same authoritative project-tree resolution/provisioning primitive already used by the incumbent CaseAuthority execution path.

## Manifest ownership

`Manifest.projectTreeCaseSpec` now also accepts:

```text
suite-native
```

while preserving the three-column authoritative shape:

```text
projectIdentity<TAB>fixturePath<TAB>suite-native
```

It continues to derive:

```text
CaseId = namespace/projectIdentity/fixturePath
sourcePath = fixtures/fixturePath
CaseAuthorityDescriptor = projectTreeAuthorityDescriptor(projectIdentity)
```

Existing `true` / `error` handling remains present pending TOOL009-B global liveness reconciliation.

## Corpus cutover

Published manifest state:

```text
content-identity       12/12 suite-native
resolution-input-lock   2/2  suite-native
resolution-root         8/8  suite-native
execution-plan         28/28 suite-native
project-projection      4/4  suite-native

TOTAL                   54/54 suite-native
LEGACY                   0
```

The 54 logical rows are backed by 40 distinct project-tree fixture scripts; reused physical sources remain distinct logical Cases through preserved project identity.

All migrated fixture behavior is now owned by `std:test/Test` bodies using canonical `std:test/Assertions`.

## Package corpus ownership after Publication 2

Current published Package production ownership is:

```text
Package TOML          102
Package case-outcomes 157
Package project-tree   54
                     ---
TOTAL                 313

SUITE_NATIVE          313
LEGACY                  0
```

This establishes that the known Package production corpus no longer requires legacy expectation ownership.

It does **not** by itself prove which legacy Test Tool implementation facilities are globally removable.

## Published product surface

The migration commit changes 60 files and advances:

```text
0.3.75-SNAPSHOT
->
0.3.76-SNAPSHOT
```

Material production changes include:

```text
protos/tools/test/Manifest.protos
protos/tools/test/LogicalCaseMigration.protos
protos/tools/test/Main.protos

src/main/java/com/guillermomolina/protos/cli/ProtosCli.java
src/main/java/com/guillermomolina/protos/cli/ProtosTestToolAsyncExecutionScope.java

src/main/java/com/guillermomolina/protos/execution/ProtosTestCaseAuthorityAttemptBridge.java
src/main/java/com/guillermomolina/protos/execution/ProtosTestCaseAuthorityExecutionFacility.java
src/main/java/com/guillermomolina/protos/execution/ProtosTestLogicalCaseAttemptBridge.java
src/main/java/com/guillermomolina/protos/execution/ProtosTestLogicalCaseDiscoveryFacility.java
src/main/java/com/guillermomolina/protos/execution/ProtosTestLogicalCaseExecutionFacility.java
```

Relevant focal tests were updated for authority reuse and logical Case execution semantics.

## Human Executor validation

The Human Executor explicitly reported the requested implementation tests green before publication:

```text
HUMAN_EXECUTOR_VALIDATION=PASS
HUMAN_EXECUTOR_TESTS=PASS
PRODUCT_COMMIT_PUSH=PASS
```

These are maintainer-reported execution results and are recorded as such rather than inferred from static source inspection.

At the latest reconciliation, GitHub Actions `CI` run `35761973563` for the exact product revision is still `in_progress`. No remote CI PASS is inferred until GitHub reports a conclusion.

## Cleanup consequence

The Package corpus blocker that kept TOOL009-B blocked is removed:

```text
PACKAGE_KNOWN_LEGACY_CASES=0
TOOL009B_PACKAGE_CORPUS_BLOCKER=REMOVED
```

Do not delete legacy infrastructure based only on this migration.

The next required task is:

```text
TOOL009-B / #685
FINAL POST-MIGRATION GLOBAL LEGACY CONSUMER RECONCILIATION
```

That reconciliation must determine repository-wide which migration bridge, D108, expectation, lifecycle, manifest compatibility, and host execution facilities now have zero authoritative consumers and which remain current functionality.

Only proven migration-only zero-consumer infrastructure may then be removed.
