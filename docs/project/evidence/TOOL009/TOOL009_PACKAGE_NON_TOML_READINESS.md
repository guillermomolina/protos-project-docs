# TOOL009 — Remaining Package non-TOML corpus migration readiness

## Identity

- Parent workstream: `guillermomolina/protos#600` — TOOL009.
- Cleanup blocker: `guillermomolina/protos#685` — TOOL009-B.
- Investigated repository: `guillermomolina/protos`.
- Protos revision: `4c4aa95a5852119bd280ceb40483871d5d2cbb82`.
- Protos version: `0.3.74-SNAPSHOT`.
- Investigation type: implementation-readiness investigation only.
- Product mutations performed by this investigation: none.
- Builds/tests/programs executed by this investigation: none.

This record follows the completed 102-case Package TOML migration and the final
TOOL009-B global reconciliation. It investigates only the eight remaining
Package Tool production leaves that still consume the incumbent legacy
expectation path.

## Final result

```text
TOOL009_PACKAGE_NON_TOML_READINESS = REQUIRES_BOUNDED_PLUMBING

CASE_OUTCOMES_157=REQUIRES_BOUNDED_PLUMBING
PROJECT_TREE_54=REQUIRES_BOUNDED_PLUMBING

NEW_DESIGN_DECISION_REQUIRED=NO
```

The two migration classes require different bounded implementation work:

1. the 157 `case-outcomes` cases need finite Package module overlay expansion
   plus suite-native ownership support in the existing generic
   `case-outcomes` manifest loader;
2. the 54 `project-tree` cases additionally require the existing physical
   project-tree `CaseAuthority` to be transported into the same fresh Process
   that resolves and invokes the selected `Test.call()`.

No new CaseAuthority abstraction, fixture lifecycle model, D152/D153/D178
semantics, or Dxxx/PLATxxx decision is required.

## Verified baseline

Current `main` was verified as:

```text
REPOSITORY=guillermomolina/protos
PROTOS_REVISION=4c4aa95a5852119bd280ceb40483871d5d2cbb82
VERSION=0.3.74-SNAPSHOT
```

The earlier global cleanup reconciliation was also re-read at:

```text
guillermomolina/protos-project-docs
docs/project/evidence/TOOL009/TOOL009B_FINAL_GLOBAL_LEGACY_RECONCILIATION.md

PROJECT_RECORD_REVISION=
5c0538f5833f58d56592af006d45baa4d37fbfef
```

That record and current product state agree on the remaining production
inventory.

## Governance inspected

The investigation materially inspected and applied:

```text
AGENTS.md
AGENTS.work/TOOL.md
AGENTS.work/IMPLEMENTATION.md
src/AGENTS.md
protos/AGENTS.md
```

The governing implementation boundary is that already-ratified semantics must
be realized rather than redesigned. Missing host/runtime transport that
implements those semantics is bounded plumbing, not a new design decision.

## Confirmed production inventory

| Corpus | Loader | Cases | true | error |
| --- | --- | ---: | ---: | ---: |
| `package-tool/version` | `case-outcomes` | 74 | 47 | 27 |
| `package-tool/lock` | `case-outcomes` | 71 | 39 | 32 |
| `package-tool/resolution-input` | `case-outcomes` | 12 | 6 | 6 |
| `package-tool/content-identity` | `project-tree` | 12 | 12 | 0 |
| `package-tool/resolution-input-lock` | `project-tree` | 2 | 2 | 0 |
| `package-tool/resolution-root` | `project-tree` | 8 | 3 | 5 |
| `package-tool/execution-plan` | `project-tree` | 28 | 4 | 24 |
| `package-tool/project-projection` | `project-tree` | 4 | 3 | 1 |
| **Total** |  | **211** | **116** | **95** |

All eight leaves are bound to:

```text
protos/test/package
```

The first three leaves are structurally two-column `case-outcomes` plans.
The remaining five are three-column `project-tree` plans carrying project
identity, fixture path, and retained outcome.

## Migration-readiness matrix

| Corpus | Loader | Host authority | Import graph ready now | Manifest change required | Plumbing required | Readiness |
| --- | --- | --- | --- | --- | --- | --- |
| `package-tool/version` | `case-outcomes` | none | no | yes | loader + finite resolver overlay | `BOUNDED_PLUMBING` |
| `package-tool/lock` | `case-outcomes` | none | no | yes | loader + finite resolver overlay | `BOUNDED_PLUMBING` |
| `package-tool/resolution-input` | `case-outcomes` | none | no | yes | loader + finite resolver overlay | `BOUNDED_PLUMBING` |
| `package-tool/content-identity` | `project-tree` | physical project-tree | no | yes | authority transport + resolver | `BOUNDED_PLUMBING` |
| `package-tool/resolution-input-lock` | `project-tree` | physical project-tree | no | yes | authority transport + resolver | `BOUNDED_PLUMBING` |
| `package-tool/resolution-root` | `project-tree` | physical project-tree | no | yes | authority transport + resolver | `BOUNDED_PLUMBING` |
| `package-tool/execution-plan` | `project-tree` | physical project-tree | no | yes | authority transport + resolver | `BOUNDED_PLUMBING` |
| `package-tool/project-projection` | `project-tree` | physical project-tree | no | yes | authority transport + resolver | `BOUNDED_PLUMBING` |

## Case-outcomes — 157 cases

### Readiness

```text
CASE_OUTCOMES_157 = REQUIRES_BOUNDED_PLUMBING
```

Semantically these cases map cleanly to the already-ratified suite-native
authoring model.

Existing successful fixtures can move their behavior into a selected
`std:test/Test` body and terminate with `Assertions.require(...)`.
Existing expected-error fixtures can place the signaling operation inside
`Assertions.signals(...)`.

The required invariants remain:

```text
discovery observational
selected Test.call() = logical Case authority
Test body exactly once
fresh semantic Process per Case
```

No additional host capability or physical CaseAuthority dependency was found
for these three leaves.

### Direct import inventory

The 74 `version` fixtures partition exactly across these Package-local
imports:

```text
self:ReleaseVersion              19
self:DependencyConstraint        37
self:FreshVersionSelection        9
self:RetainedVersionSelection     9
                                 --
                                 74
```

The 71 `lock` fixtures partition exactly across:

```text
self:LockSyntax                  38
self:LockDocument                33
                                 --
                                 71
```

All 12 `resolution-input` fixtures use:

```text
self:ReleaseVersion
self:DependencyConstraint
self:ResolutionInput
```

### Transitive Package import graph

| Specifier | Source path | Direct/transitive dependencies | Overlay required |
| --- | --- | --- | --- |
| `self:ReleaseVersion` | `protos/tools/package/ReleaseVersion.protos` | none | yes |
| `self:DependencyConstraint` | `protos/tools/package/DependencyConstraint.protos` | `self:ReleaseVersion` | yes |
| `self:FreshVersionSelection` | `protos/tools/package/FreshVersionSelection.protos` | `ReleaseVersion`, `DependencyConstraint` | yes |
| `self:RetainedVersionSelection` | `protos/tools/package/RetainedVersionSelection.protos` | `ReleaseVersion`, `DependencyConstraint`, `FreshVersionSelection` | yes |
| `self:LockSyntax` | `protos/tools/package/LockSyntax.protos` | `ReleaseVersion` | yes |
| `self:LockDocument` | `protos/tools/package/LockDocument.protos` | `LockSyntax`, `ReleaseVersion`, `std:collections/Array` | yes |
| `self:ResolutionInput` | `protos/tools/package/ResolutionInput.protos` | `LockSyntax`, `ReleaseVersion`, `std:collections/Array`, `std:crypto/SHA256` | yes |

The standard-library dependencies are already handled by standard-library
resolution and need no Package exact overlay.

### Why the current Package logical resolver is insufficient

A suite source is rematerialized by `ProtosDirectFileModuleResolver` and has a
`direct-file:` module identity. `ProtosBundledToolModuleResolver` correctly
permits `self:` imports only from within its own bundled-tool closure.

The existing `packageLogicalCaseFallbackResolver` therefore uses an exact
overlay for the finite Package TOML graph:

```text
self:TomlSyntax
self:TomlDocument
self:ManifestSchemaV1
tool-shared:Toml10/TomlSyntax
tool-shared:Toml10/TomlDocument
```

That overlay does not cover the seven Package-local modules needed by the
remaining `case-outcomes` corpus.

The bounded implementation is to expand the exact Package overlay. The generic
`ProtosBundledToolModuleResolver` closure guard must remain unchanged.

### Manifest support

`Manifest.loadCaseOutcomes` already owns the correct generic two-column
loader and case namespace.

Its current `caseOutcomeSpec` accepts only:

```text
true
error
```

The bounded migration requires it to also normalize:

```text
<path>    suite-native
```

No new loader family or manifest syntax is needed.

## Project-tree — 54 cases

### Readiness

```text
PROJECT_TREE_54 = REQUIRES_BOUNDED_PLUMBING
```

These cases cannot be migrated by merely replacing their retained outcome with
`suite-native`.

### Incumbent authority path

The current execution path is:

```text
three-column manifest
  -> Manifest.projectTreeCaseSpec
  -> projectTreeAuthorityDescriptor(projectIdentity)
  -> CorpusBinding.caseAuthorityExecutionAsync
  -> ProtosTestCaseAuthorityExecutionFacility
  -> ProtosTestCaseAuthorityAttemptBridge
  -> trusted corpus-specific casesRoot
  -> casesRoot / fixtureIdentity
  -> fresh ProtosNioReadOnlyTreeFilesystemBackend
  -> fresh Process
  -> projectTreeFilesystem installed
  -> fixture execution
```

The physical bridge confines the fixture identity under a host-owned trusted
`casesRoot`, creates one fresh read-only authority per Case, injects the
resulting capability as `projectTreeFilesystem`, executes the Case in a fresh
Process, and tears the authority down afterward.

### Why the current suite-native route is insufficient

`LogicalCaseMigration.splitPlan` deliberately rejects a suite-native spec
whose `Manifest.caseAuthorityDescriptor(spec)` is non-null.

Removing only that guard would still be incorrect.

The ordinary suite-native path is:

```text
source
  -> ProtosTestLogicalCaseExecutionFacility
  -> ProtosTestLogicalCaseAttemptBridge
  -> fresh Process
  -> declare suite
  -> Discovery.resolveSelectedTest(...)
  -> selected Test.call()
```

That fresh Process does not receive project-tree CaseAuthority.

Conversely, `ProtosTestCaseAuthorityExecutionFacility` accepts the incumbent
whole-source protocol:

```text
(source, descriptor)
```

It does not implement the four-argument selected-Test logical Case protocol.

The selected Test must therefore run in the same Process into which
`projectTreeFilesystem` is installed. Nesting two existing facilities would
produce either incumbent whole-source ownership or two separate fresh
Processes, neither of which satisfies the ratified model.

### Established implementation precedent

`ProtosProcessSnapshotLogicalCaseExecutionFacility` already demonstrates the
required architectural pattern: adapt a specialized existing bootstrap to the
four-argument logical Case protocol while preserving its fresh per-Case
environment, then perform observational declaration, selected-Test resolution,
and exactly-one `Test.call()`.

Project-tree needs an analogous bounded composition of already-established
responsibilities, not a second CaseAuthority model.

### Exact suite-native gap

Three connected gaps are present.

#### 1. Authority descriptor transport

`LogicalCaseMigration.sourceAssociation` currently emits only:

```text
[corpusId, sourcePath]
```

The logical executor and Java logical Case facility validate that two-element
shape. No existing suite-native transport carries the project-tree descriptor.

#### 2. Authority provisioning inside selected-Test execution

The physical project-tree bridge owns:

```text
trusted casesRoot
fixtureIdentity confinement
read-only physical backend
projectTreeFilesystem provisioning
fresh physical authority lifecycle
```

The logical Case bridge owns:

```text
fresh semantic Process
suite declaration
signature validation
Discovery.resolveSelectedTest(...)
selected Test.call()
```

Those established responsibilities must be composed inside one Process.

#### 3. Project identity must remain part of source association

The project-tree manifest can intentionally reuse one fixture source under
different project identities. `package-tool/execution-plan` already contains
such rows.

Current suite-native bookkeeping keys a source by:

```text
corpusId + sourcePath
```

and rejects duplicate source paths.

Project-tree migration therefore needs project identity / authority identity in
the logical source association so distinct physical CaseAuthorities sharing a
fixture source do not collapse into one logical source record.

## Required project-tree manifest representation

The three-column structure remains authoritative after migration:

```text
project identity<TAB>fixture path<TAB>suite-native
```

`Manifest.projectTreeCaseSpec` should continue to derive:

```text
CaseId =
  namespace/projectIdentity/fixturePath

sourcePath =
  fixtures/fixturePath

CaseAuthorityDescriptor =
  projectTreeAuthorityDescriptor(projectIdentity)
```

Only expectation ownership moves from retained `true/error` metadata into the
selected `Test` body.

Project identity is therefore not obsolete metadata and must not be discarded.

## Project-tree Package import closure

Representative and complete fixture inspection established Package-local roots
including:

```text
ContentIdentity
LockFile
ResolutionRoot
ResolutionInput
ExecutionPlan
ProjectFile
ProjectDocument
ProjectMetadata
```

Their transitive Package graph reaches:

```text
LockDocument
MetadataPublication
ManifestSchemaV1
DependencyConstraint
LockSyntax
RuntimeNames
ManifestCommand
TomlDocument
ReleaseVersion
```

plus standard modules including:

```text
std:collections/Array
std:io/Files
std:crypto/SHA256
```

The same finite exact-overlay principle applies. In particular,
`ExecutionPlan` imports `self:RuntimeNames`; the TOML-specific proof alias
`package-runtime-names` does not itself satisfy that import.

The generic bundled-tool closure guard must not be relaxed.

## Canonical source-authoring pattern

Both migration classes can use the established authoring form:

```text
TestValue: import("std:test/Test")
Assertions: import("std:test/Assertions")

tests: Array(
    TestValue("<stable-name>", () => {
        <behavior under test>
        <require/signals>
    })
)
```

Existing project-tree access to `projectTreeFilesystem` must occur inside the
selected Test body. Discovery must not execute the tested filesystem/project
behavior.

## Exact bounded plumbing target

```text
MISSING_COMPONENT =
    project-tree-aware Package Logical Case authority adapter
```

The complete target is:

```text
LogicalCaseMigration
  transports the existing inert project-tree authority descriptor
    ->
protos/test/package logicalCaseExecutionAsync
  receives source association + source/signature/selector
    ->
host selects the already-trusted corpus casesRoot
    ->
existing D133/D134 physical authority machinery
  provisions projectTreeFilesystem
    ->
inside that same fresh Process:
  suite declaration
  signature validation
  Discovery.resolveSelectedTest(...)
  selected Test.call() exactly once
```

Implementation should extract or reuse the physical authority-provisioning
primitive currently owned by `ProtosTestCaseAuthorityAttemptBridge` rather
than duplicate confinement or lifecycle behavior.

No live host path belongs in `CasePlan`, no guest-visible host path is
required, and no ambient mutable registry is required.

## Recommended implementation decomposition

The minimum evidence-based decomposition is two publications.

### Publication 1 — all 157 case-outcomes

One coherent implementation:

```text
extend case-outcomes loader with suite-native
+ expand finite Package logical resolver overlay
+ convert all 157 source files to Test ownership
+ flip all 157 manifest rows to suite-native
```

This closes the complete non-authority migration class without corpus-by-corpus
micro-publications.

### Publication 2 — project-tree plumbing plus all 54 cases

Internally validate the plumbing before the manifest flip, but publish the
coherent result together:

```text
project-tree suite-native manifest support
+ authority descriptor/source-association transport
+ project-identity-safe logical source bookkeeping
+ reuse/extraction of physical CaseAuthority provisioning
+ selected-Test execution inside that authority
+ required finite Package resolver closure
+ migrate all 54 project-tree rows/sources
```

There is no semantic reason to publish five project-tree corpus slices.

## Version and changelog consequences

Publication 1 changes Java product plumbing in the Package logical resolver and
therefore requires one then-current Maven patch `-SNAPSHOT` increment and
matching root `CHANGELOG.md` entry.

Publication 2 changes Java execution plumbing for logical Case / physical
CaseAuthority composition and likewise requires one then-current implementation
version step and matching changelog entry.

A separate published version solely for the project-tree plumbing prerequisite
is not required. It can be developed and focal-validated as an internal
checkpoint, then published atomically with the 54-case migration.

## Materially inspected product surface

The investigation materially inspected at least:

```text
pom.xml

protos/tools/test/Main.protos
protos/tools/test/Manifest.protos
protos/tools/test/LogicalCaseMigration.protos
protos/tools/test/Discovery.protos
protos/tools/test/LogicalCaseRunner.protos

src/main/java/com/guillermomolina/protos/cli/ProtosCli.java
src/main/java/com/guillermomolina/protos/cli/ProtosTestCorpusRegistry.java
src/main/java/com/guillermomolina/protos/cli/ProtosTestToolAsyncExecutionScope.java
src/main/java/com/guillermomolina/protos/cli/ProtosTestExecutionRequirementRegistry.java
src/main/java/com/guillermomolina/protos/cli/ProtosTestCaseAuthorityExecutionScope.java

src/main/java/com/guillermomolina/protos/execution/ProtosBundledToolModuleResolver.java
src/main/java/com/guillermomolina/protos/execution/ProtosExactModuleOverlayResolver.java
src/main/java/com/guillermomolina/protos/execution/ProtosTestLogicalCaseExecutionFacility.java
src/main/java/com/guillermomolina/protos/execution/ProtosTestLogicalCaseAttemptBridge.java
src/main/java/com/guillermomolina/protos/execution/ProtosTestCaseAuthorityExecutionFacility.java
src/main/java/com/guillermomolina/protos/execution/ProtosTestCaseAuthorityAttemptBridge.java
src/main/java/com/guillermomolina/protos/execution/ProtosProcessSnapshotLogicalCaseExecutionFacility.java
src/main/java/com/guillermomolina/protos/execution/ProtosProcessSnapshotExecution.java

protos/tests/package-tool/version/manifest.tsv
protos/tests/package-tool/lock/manifest.tsv
protos/tests/package-tool/resolution-input/manifest.tsv
protos/tests/package-tool/content-identity/manifest.tsv
protos/tests/package-tool/resolution-input-lock/manifest.tsv
protos/tests/package-tool/resolution-root/manifest.tsv
protos/tests/package-tool/execution-plan/manifest.tsv
protos/tests/package-tool/project-projection/manifest.tsv
```

Representative case-outcome fixtures from every Package module family were
inspected. All project-tree fixture source files under the five `fixtures/`
directories were inspected for import/capability shape, and the relevant
Package Tool modules were followed transitively to establish their import
closure.

No evidence requires reopening D108, D132, D133, D134, D152, D153, or D178.

## Cleanup consequence

TOOL009-B remains blocked at this revision:

```text
TOOL009B_CLEANUP=BLOCKED
REMOVE_NOW_COUNT=0
FURTHER_CORPUS_MIGRATION_REQUIRED=YES
```

The current mixed bridge, D108 production branch, legacy expectation handling,
legacy lifecycle path, and Package execution/inspection/resource facilities
remain live until these 211 cases migrate and a fresh global consumer
reconciliation proves which migration-only components have become unreachable.

After both Package non-TOML migration publications, rerun the complete global
TOOL009-B consumer trace before deleting any legacy infrastructure.
