# TOOL009 — Package case-outcomes suite-native migration

## Identity

- Parent workstream: `guillermomolina/protos#600`.
- Cleanup blocker: `guillermomolina/protos#685`.
- Parent Protos revision: `4c4aa95a5852119bd280ceb40483871d5d2cbb82`.
- Migration Protos revision: `a116176abd69ddf85e8ee52b623e90d48ef3a11e`.
- Version: `0.3.75-SNAPSHOT`.
- Commit: `TOOL009: migrate Package case-outcomes corpora to suite-native`.
- Readiness authority: `docs/project/evidence/TOOL009/TOOL009_PACKAGE_NON_TOML_READINESS.md` at `34035efdbd595de0b1517bbe6485c3169c840d44`.

## Result

Publication 1 of the remaining Package non-TOML migration is complete.

```text
TOOL009_PACKAGE_CASE_OUTCOMES_MIGRATION=COMPLETE

VERSION_TOTAL=74
VERSION_SUITE_NATIVE=74
VERSION_LEGACY=0

LOCK_TOTAL=71
LOCK_SUITE_NATIVE=71
LOCK_LEGACY=0

RESOLUTION_INPUT_TOTAL=12
RESOLUTION_INPUT_SUITE_NATIVE=12
RESOLUTION_INPUT_LEGACY=0

CASE_OUTCOMES_TOTAL=157
CASE_OUTCOMES_SUITE_NATIVE=157
CASE_OUTCOMES_LEGACY=0

PROJECT_TREE_LEGACY=54
```

Direct inspection of the three published manifests confirms every current data row is `suite-native`:

- `protos/tests/package-tool/version/manifest.tsv`: 74/74.
- `protos/tests/package-tool/lock/manifest.tsv`: 71/71.
- `protos/tests/package-tool/resolution-input/manifest.tsv`: 12/12.

No `true` or `error` ownership rows remain in those three manifests.

## Published plumbing

### Generic case-outcomes loader

`protos/tools/test/Manifest.protos` now allows the established `suite-native` marker in the generic two-column `caseOutcomeSpec` / `loadCaseOutcomes` route.

Normalization is:

```text
suite-native
  -> expectation = "suite-native"
  -> expected = "-"
```

Existing `true` and `error` handling remains available for still-legacy consumers.

### Package Logical Case resolver

`packageLogicalCaseFallbackResolver` now includes finite exact overlays for:

```text
self:ReleaseVersion
self:DependencyConstraint
self:FreshVersionSelection
self:RetainedVersionSelection
self:LockSyntax
self:LockDocument
self:ResolutionInput
```

The exact-overlay map remains bounded. `ProtosBundledToolModuleResolver` generic closure semantics were not changed, and standard-library dependencies continue to resolve through the standard-library resolver.

Focal coverage in `ProtosPackageTestLogicalCaseExecutionFacilityTest` exercises real Package module behavior through the newly exposed dependency graph.

## Published source ownership

All 157 migrated fixtures now own their result inside a selected `std:test/Test` body using the existing `std:test/Assertions` API.

Former success cases use `Assertions.require(...)`; former expected-error cases use `Assertions.signals(Error, ...)`.

Package-local `self:` imports used by tested behavior are performed inside the selected Test body, preserving the established observational discovery / flavor-specific execution boundary.

## Published product surface

The product commit changes 166 files.

Direct commit inspection confirms:

```text
VERSION_FIXTURES_CHANGED=74
LOCK_FIXTURES_CHANGED=71
RESOLUTION_INPUT_FIXTURES_CHANGED=12

PRODUCT_JAVA_CHANGED:
  src/main/java/com/guillermomolina/protos/cli/ProtosCli.java

JAVA_TESTS_CHANGED:
  src/test/java/com/guillermomolina/protos/execution/ProtosPackageTestLogicalCaseExecutionFacilityTest.java
  src/test/java/com/guillermomolina/protos/execution/ProtosTestToolManifestPlanTest.java

PROJECT_TREE_CORPUS_FILES_CHANGED=0
```

The implementation version advances from `0.3.74-SNAPSHOT` to `0.3.75-SNAPSHOT`.

## Human Executor validation

The Human Executor explicitly reported all requested validation green before publication:

```text
STATIC_VALIDATION=PASS
FOCAL_TESTS=PASS
PACKAGE_CASE_OUTCOMES_157=PASS
MAKE_TEST=PASS
PRODUCT_COMMIT_PUSH=PASS
```

These are maintainer-reported execution results and are recorded as such; they are not inferred from static repository inspection.

At the latest reconciliation, GitHub Actions `CI` run `35751387475` for the exact product SHA is still `in_progress`. No remote CI PASS is inferred before GitHub reports a conclusion.

## Remaining Package legacy ownership

Publication 1 intentionally leaves the five project-tree production leaves on the incumbent D133/D134 authority path:

```text
package-tool/content-identity       12
package-tool/resolution-input-lock   2
package-tool/resolution-root         8
package-tool/execution-plan         28
package-tool/project-projection      4
                                    --
                                    54
```

Therefore:

```text
GLOBAL_PACKAGE_CASE_OUTCOMES_LEGACY=0
GLOBAL_PACKAGE_PROJECT_TREE_LEGACY=54
TOOL009B_CLEANUP=BLOCKED
```

The mixed migration bridge, D108 production path, legacy expectation handling, legacy lifecycle observation, and current Package exact/inspection/resource execution facilities remain live and must not be removed yet.

## Next gate

Proceed directly to Publication 2 established by the readiness investigation:

```text
project-tree-aware Package Logical Case authority adapter
+
all 54 remaining project-tree cases
```

Only after those 54 cases migrate should TOOL009-B rerun the complete global legacy consumer reconciliation before deleting any migration-only infrastructure.
