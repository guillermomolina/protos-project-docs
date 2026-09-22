# TOOL009 — Package Tool Suite-Native Logical Case Facility

## Identity

- Issue: `guillermomolina/protos#692`
- Parent: `guillermomolina/protos#600`
- Cleanup blocker: `guillermomolina/protos#685`
- Protos publication revision: `a32acf925383a2b1748bede12586ff3d320b7bf7`
- Parent Protos revision: `6e7d89194925ba9fa2cd9c5c45aefa72d9939621`
- Version: `0.3.72-SNAPSHOT`
- Commit: `TOOL009: add Package Tool suite-native logical Case execution route`

## Result

```text
TOOL009_PACKAGE_LOGICAL_CASE_FACILITY = IMPLEMENTED
PACKAGE_LOGICAL_CASE_BINDING = PASS
PACKAGE_PRELUDE = PASS
PACKAGE_RESOLVER = PASS
PACKAGE_BOOTSTRAP = PASS
SELECTED_TEST_AUTHORITY = PASS
MODULE_COMPLETION_NOT_AUTHORITY = PASS
EXACTLY_ONCE = PASS
DISCOVERY_OBSERVATIONAL = PASS
MULTI_TEST_SELECTOR = PASS
SIGNATURE_MISMATCH = PASS
SELECTOR_MISMATCH = PASS
FRESH_PROCESS_PER_CASE = PASS
CASE_STATE_ISOLATION = PASS
LEGACY_PACKAGE_EXECUTION_UNCHANGED = PASS
PACKAGE_INSPECTION_UNCHANGED = PASS
PACKAGE_RESOURCE_EXECUTION_UNCHANGED = PASS
PACKAGE_RESOURCE_INSPECTION_UNCHANGED = PASS
PACKAGE_CORPUS_UNTOUCHED = PASS
PACKAGE_MIGRATED = 0
FULL_VALIDATION = PASS
```

## Implementation

`protos/test/package` now additively exposes `logicalCaseExecutionAsync` while preserving the existing Package Tool execution, inspection, resource-execution and resource-inspection bindings.

`ProtosCli` builds a Package-flavored Logical Case fallback resolver from the existing bundled Test Tool resolver plus an exact Package Tool `RuntimeNames.protos` overlay. This preserves a Package-specific bootstrap signal without changing the legacy `packagePrelude` or `packageExecutionAsync` path.

`ProtosTestToolAsyncExecutionScope` installs a fourth `ProtosTestLogicalCaseExecutionFacility` instance under `packageLogicalCaseExecutionAsync`, alongside ordinary, Actor and Group. It reuses the existing Logical Case execution/attempt bridge and therefore retains fresh Process/rematerialization semantics and selected `Test.call()` authority.

The Package TOML corpus was not migrated in this slice. The 102 fixtures and their manifest remain for later migration slices.

## Validation

The Human Executor reported all requested validation gates green after implementation:

```text
git diff --check = PASS
mvn -q -DskipTests compile test-compile = PASS
focal Package/Actor/Group/ordinary Logical Case tests = PASS
registry/integration/Package execution tests = PASS
make test = PASS
```

No test result is inferred here; these PASS results are the Human Executor's reported execution evidence.

## Scope preservation

The published diff preserves D108, D152, D153, D178, `CaseAuthority`, Process, Actor, Group, legacy Package execution, Package inspection, Package resource execution, and Package resource inspection. `pom.xml` advances to `0.3.72-SNAPSHOT`, with a matching `CHANGELOG.md` entry.

## Publication hygiene reconciliation

The accidental zero-byte temporary test artifact:

```text
protos/tests/library/tool008-unplanned-13881163000710449719.protos
```

was removed in Protos revision:

```text
f8cfb52afb2dd77693595fcb2099a21a9a6cc8e0
```

The file is absent from current `main`. No product behavior changed in this hygiene commit.

The facility slice is therefore fully published and publication-clean.

## Next intended work

The infrastructure is ready for separate Package corpus migration slices:

```text
TomlSyntax        27
TomlDocument      31
ManifestSchemaV1  44
```

Only after all 102 Package cases are migrated should TOOL009-B / #685 be globally reconciled again for legacy-scaffolding removal.