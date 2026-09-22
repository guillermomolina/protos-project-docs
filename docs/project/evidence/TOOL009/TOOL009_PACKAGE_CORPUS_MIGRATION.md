# TOOL009 — Package Tool TOML Corpus Suite-Native Migration

## Identity

- Parent workstream: `guillermomolina/protos#600`
- Cleanup follow-up: `guillermomolina/protos#685`
- Parent Protos revision: `2a5412bd60115d5c0ef928499668dada5cd61094`
- Migration Protos revision: `4c4aa95a5852119bd280ceb40483871d5d2cbb82`
- Version: `0.3.74-SNAPSHOT`
- Commit: `TOOL009: migrate the complete 102-case Package Tool TOML corpus to suite-native`

## Result

The remaining Package Tool TOML corpus has been migrated from the legacy Test Tool expectation path to suite-native Logical Case declarations.

```text
TOOL009_PACKAGE_CORPUS_MIGRATION=COMPLETE
PACKAGE_MANIFEST_DATA_ROWS=102
PACKAGE_SUITE_NATIVE=102
PACKAGE_LEGACY=0
LEGACY_TRUE_METADATA_REMAINING=0
LEGACY_ERROR_METADATA_REMAINING=0

TOML_SYNTAX=27
TOML_DOCUMENT=31
MANIFEST_SCHEMA_V1=44

JAVA_PRODUCTION_CHANGES=0
JAVA_TEST_CHANGES=0
LEGACY_INFRASTRUCTURE_REMOVED=NO
```

The `manifest.tsv` file contains one header row plus exactly 102 data rows, and every data row has expectation `suite-native`.

## Published product surface

The migration commit changes 109 files in total:

- 102 `.protos` fixtures under `protos/tests/package-tool/toml-syntax/`;
- `protos/tests/package-tool/toml-syntax/manifest.tsv`;
- `protos/tools/test/Manifest.protos`;
- three existing tooling fixtures:
  - `protos/tests/tooling/tool002-e1b-package-toml-filesystem.protos`;
  - `protos/tests/tooling/tool002-e2a1-package-resolved-execution.protos`;
  - `protos/tests/tooling/tool002-e2a2b-package-failed-fixture.protos`;
- `pom.xml`;
- `CHANGELOG.md`.

No file under `src/main/java/` or `src/test/java/` changes in this publication.

## Suite-native fixture shape

The published corpus fixtures now declare canonical Test Tool authoring facilities:

```text
std:test/Test
std:test/Assertions
```

Former success expectations use `Assertions.require(...)`; former error expectations use `Assertions.signals(Error, ...)`.

The behavior under test is moved inside the selected Test body. Package-local `self:` imports are likewise performed inside that body so discovery remains flavor-agnostic and the already-published Package execution resolver handles those imports only when the selected Case executes.

The published changelog records the completed family inventory as:

```text
TomlSyntax        27
TomlDocument      31
ManifestSchemaV1  44
TOTAL            102
```

## Manifest routing

`protos/tools/test/Manifest.protos` now accepts `suite-native` as the Package TOML manifest expectation and normalizes it to the established suite-native route.

The existing `true` / `error` parsing remains present for legacy callers; this migration does not remove global legacy infrastructure.

## Legacy tooling decoupling

Two legacy exact-execution tooling fixtures previously reused Package corpus source files whose content is now suite-native Test declarations.

They now execute dedicated inline source preserving their former raw execution subject, so those infrastructure tests remain about the legacy exact-execution facilities rather than depending on corpus files now owned by suite-native execution.

The filesystem fixture is updated to expect the first Package manifest entry as:

```text
expectation=suite-native
expected=-
```

## Preserved scope

This publication does not remove or redesign:

```text
LogicalCaseMigration
splitPlan
legacyPlan
mergeOutcome
Runner D108 route
packageExecutionAsync
packageExecutionInspectAsync
packageResourceExecutionAsync
packageResourceExecutionInspectAsync
```

It does not change D108, D152, D153, D178, or Case Authority semantics.

Final global liveness/removal analysis remains owned by TOOL009-B / #685.

## Validation / publication state

Direct GitHub inspection confirms:

```text
PROTOS_MAIN=4c4aa95a5852119bd280ceb40483871d5d2cbb82
VERSION=0.3.74-SNAPSHOT
PRODUCT_PUBLICATION_VERIFICATION=PASS
PACKAGE_SUITE_NATIVE_STATIC_RECONCILIATION=PASS
```

At the time this record was first written, the GitHub Actions CI run for this exact SHA was still `in_progress`. No CI PASS is inferred before GitHub reports a conclusion.

The maintainer reported this migration as uploaded after the Human Executor workflow; this record distinguishes that publication fact from CI state visible through GitHub.

## Consequence for TOOL009-B

The sole known Package-corpus blocker for TOOL009-B is removed:

```text
PACKAGE_LEGACY=0
TOOL009B_PACKAGE_BLOCKER=REMOVED
```

This does **not** itself prove that every global legacy consumer is gone.

The next work is therefore:

```text
TOOL009-B / #685
FINAL GLOBAL LEGACY CLEANUP RECONCILIATION

prove current global consumers
-> REMOVE only demonstrated migration-only residue
-> KEEP authoritative current facilities
-> investigate any ambiguous ownership
```

No legacy code should be deleted solely because the Package corpus is now suite-native.
