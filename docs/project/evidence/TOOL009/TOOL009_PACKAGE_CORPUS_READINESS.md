# TOOL009 — Package Tool Corpus Suite-Native Migration Readiness

## Investigation identity

- Issue: [TOOL009 / #692](https://github.com/guillermomolina/protos/issues/692)
- Investigated Protos revision: `6e7d89194925ba9fa2cd9c5c45aefa72d9939621`
- Result: `TOOL009_PACKAGE_CORPUS = NOT_ESTABLISHED`
- New architectural decision required: `NO`

## Corpus inventory

```
PACKAGE_MANIFEST_ENTRIES_TOTAL=102
PACKAGE_ALREADY_SUITE_NATIVE=0
PACKAGE_REQUIRING_MIGRATION=102
PACKAGE_EXPECTATION_FAMILIES=2 (true=43, error=59)
PACKAGE_SPECIAL_CASES=59 expected-error fixtures; 27 TomlSyntax; 31 TomlDocument; 44 ManifestSchemaV1
```

All 102 manifest entries are currently legacy-expectation entries.

The 102 fixtures partition into three semantic families:

- 27 `TomlSyntax`
- 31 `TomlDocument`
- 44 `ManifestSchemaV1`

The manifest expectation families are only `true` (43 cases) and `error` (59 cases).

Representative fixtures directly inspected:

- `key-bare-dotted.protos`
- `invalid-key-error.protos`
- `string-multiline-basic.protos`
- `document-table-array-of-tables-nested-latest.protos`
- `document-table-array-of-tables-child-before-parent-error.protos`
- `manifest-schema-final-all-dependency-forms.protos`
- `manifest-schema-final-mixed-git-path-error.protos`

No fixture-level Process, Future, resource-handle, or external-command dependency was established for this corpus.

## Current execution graph

```
manifest.tsv
  -> CorpusBinding(protos/corpus/package-toml)
  -> planLoader = package-toml
  -> RepositorySuite leaf(protos/package-toml)
  -> ExecutionRequirementId = protos/test/package
  -> packageExecutionAsync
  -> Package Tool Prelude/bootstrap
  -> guest fixture
  -> boolean/error observation
  -> legacy Runner / D108 result classification
```

The suite graph already associates:

```
protos/package-toml
  -> protos/corpus/package-toml
  -> protos/test/package
```

## ExecutionRequirement / registry state

`protos/test/package` currently exposes:

```
packageExecutionAsync
packageExecutionInspectAsync
packageResourceExecutionAsync
packageResourceExecutionInspectAsync
```

It does not expose:

```
logicalCaseExecutionAsync
```

This is the direct suite-native migration blocker.

The current registry already supports suite-native logical Case execution for ordinary, Actor, Group, and Process Snapshot requirements, but Package Tool remains on the legacy-shaped D125 binding.

## Bootstrap / Prelude / resolver

Package Tool has a dedicated `packagePrelude` and Package Tool resolver/bootstrap. The existing legacy execution and resource facilities are explicitly bound to that Package Tool environment.

The generic suite-native Logical Case facility is not itself a Package Tool bootstrap authority. Therefore the Package requirement cannot simply be aliased to the ordinary Logical Case route without risking loss of the Package Tool-specific Prelude/resolver boundary.

The required implementation must preserve the existing Package Tool bootstrap as the execution authority while adapting it to the suite-native Logical Case protocol.

## Discovery boundary

Package TOML discovery remains observational:

```
packageTomlFilesystem
  -> Manifest.loadPackageToml(...)
  -> parsePackageTomlLine(...)
  -> inert CaseSpec
```

Discovery reads `manifest.tsv` and constructs inert Case metadata. It does not execute fixture bodies, provision Case resources, or run Package Tool behavior.

This is compatible with the suite-native discovery boundary.

## Case authority

The 102 TOML syntax cases do not carry Package-specific CaseAuthority descriptors or project-tree authority descriptors.

Their execution authority can therefore remain the Logical Case/Test model. Package-specific bootstrap authority is an ExecutionRequirement concern, not a new CaseAuthority dimension.

No new CaseAuthority decision is required.

## Case isolation

| State | Required treatment |
|---|---|
| guest Process/runtime | fresh per Case |
| Package Tool Prelude/runtime execution context | fresh per Case execution |
| fixture source | rematerialized per Case |
| read-only corpus filesystem | may be shared |
| immutable resolver configuration | may be shared |
| mutable guest/module state | must not be shared |
| resource handles | not consumed by this corpus |
| external subprocess state | not consumed by this corpus |

No new isolation semantics are required.

## Legacy expectations

The legacy families are:

```
true
error
```

They are naturally representable inside the suite-native Test authority:

- `true` -> successful Test completion / assertion.
- `error` -> Test must observe the expected error; lack of the error must fail the Case.

No second expectation engine or host-side expectation authority is justified.

## Execution vs inspection vs resources

The 102 TOML syntax fixtures consume execution.

They do not themselves require:

- Package Tool inspection execution;
- resource execution;
- resource inspection;
- project-tree CaseAuthority;
- external process execution.

Those Package Tool facilities remain valid host/tooling infrastructure and must not be removed or reshaped merely because this corpus moves to suite-native execution.

## Minimal implementation boundary

The next bounded implementation work is:

```
Package-specific suite-native Logical Case execution route
  -> preserve existing Package Tool Prelude/resolver bootstrap
  -> expose logicalCaseExecutionAsync for protos/test/package
  -> migrate only the 102 toml-syntax Cases
```

The implementation should reuse the existing LogicalCaseRunner/LogicalCaseMigration transport and existing Package execution/bootstrap facilities rather than introduce a new CaseAuthority or expectation protocol.

No implementation was performed as part of this investigation.

## TOOL009-B consequence

TOOL009-B / #685 remains blocked.

The Package TOML corpus is the remaining legitimate production consumer of the legacy Test Tool execution path. Its migration must occur before any global cleanup reconciliation can establish that legacy expectation/execution infrastructure is removable.

No cleanup component is declared removable by this investigation.

## Evidence inventory

Materially inspected source/test surfaces included:

```
protos/tests/package-tool/toml-syntax/manifest.tsv
protos/tests/package-tool/toml-syntax/*.protos
protos/tools/test/RepositorySuite.protos
protos/tools/test/Main.protos
protos/tools/test/Manifest.protos
src/main/java/com/guillermomolina/protos/cli/ProtosTestExecutionRequirementRegistry.java
src/main/java/com/guillermomolina/protos/cli/ProtosTestCorpusRegistry.java
src/main/java/com/guillermomolina/protos/cli/ProtosCli.java
src/main/java/com/guillermomolina/protos/execution/ProtosTestLogicalCaseExecutionFacility.java
src/main/java/com/guillermomolina/protos/execution/ProtosTestResourceExecutionScope.java
src/main/java/com/guillermomolina/protos/execution/ProtosTestCaseAuthorityExecutionFacility.java
src/main/java/com/guillermomolina/protos/cli/ProtosTestCaseAuthorityExecutionScope.java
src/test/java/com/guillermomolina/protos/cli/ProtosTestToolExecutionRequirementRegistryTest.java
src/test/java/com/guillermomolina/protos/cli/ProtosTestToolSuiteGraphTest.java
src/test/java/com/guillermomolina/protos/cli/ProtosTestToolH2B3PublicIntegrationTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosTestToolPackageExecutionEnvironmentTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosTestToolPackageFailedExecutionTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosPackageToolProtosTestSupport.java
src/test/java/com/guillermomolina/protos/execution/ProtosTestLogicalCaseExecutionFacilityTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosTestToolManifestPlanTest.java
protos/tests/tooling/tool002-e1a-package-toml-manifest-plan.protos
protos/tests/tooling/tool002-e1b-package-toml-filesystem.protos
protos/tests/tooling/tool002-e2a1-package-resolved-execution.protos
protos/tests/tooling/tool002-e2a2b-package-failed-fixture.protos
```

## Validation state

This was an investigation-only pass.

- No builds run.
- No tests run.
- No programs run.
- No Protos files changed.
- No Protos Git state changed.

This is a readiness/architecture finding, not a runtime validation result.

## Final result

```
TOOL009_PACKAGE_INVESTIGATION = COMPLETE
TOOL009_PACKAGE_CORPUS = NOT_ESTABLISHED
NEW_DECISION_REQUIRED = NO
```
