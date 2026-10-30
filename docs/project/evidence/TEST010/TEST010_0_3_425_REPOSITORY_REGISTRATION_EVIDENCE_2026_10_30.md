# TEST010 — Repository corpus registration integrity (0.3.425)

**Date:** 2026-10-30. **Role:** immutable, non-normative product-publication, source-enrollment and maintainer-validation evidence for [TEST010 #991](https://github.com/guillermomolina/protos/issues/991).

## Exact publication

```text
WORK_ITEM=TEST010
ISSUE=guillermomolina/protos#991
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=e5a55588362340aba3faa39e007b0228db77f543
IMPLEMENTATION_VERSION=0.3.425-SNAPSHOT
PRODUCT_CHANGED_PATH_COUNT=11
SPECIFICATION_CHANGE=NO
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

[Published Protos revision](https://github.com/guillermomolina/protos/commit/e5a55588362340aba3faa39e007b0228db77f543) is the commit `TEST010: audit repository test registration integrity`. It contains seven other added/modified paths beyond the two removed legacy sources and the version/changelog metadata: `protos/tools/test/RepositoryCorpusAudit.protos`, `RepositoryCorpusPlans.protos`, two new JUnit completeness support/test files, two updated JUnit integration tests, and `docs/guide/tools/test-tool.md`. It removes `protos/tests/package-tool/resolution-input/lockfile-fresh.protos` and `lockfile-stale.protos`. The two behaviors are covered by registered suite-native `resolution-input-lock/fixtures/fresh.protos` and `stale.protos` from their `manifest.tsv`.

## Corpus enrollment evidence

The product revision retains explicit D122 `RepositorySuite.root` membership; it does **not** add filesystem-based automatic test enrollment. The test-side `ProtosTestToolRepositoryCompletenessSupport` inventories `.protos` files, detects suite-native declarations and verified JUnit/auxiliary/data ownership boundaries, and materializes the authorized TestPlans through `SuiteGraph.flattenLeaves`, `RepositoryCorpusPlans.load`, and the existing `Manifest` loaders. The pure Protos `RepositoryCorpusAudit.audit` reconciles those inert inputs and emits deterministic failures for missing, dangling and repeated registrations, unclassified sources, invalid exclusions and authority conflicts. No guest `Test` body is executed by the audit.

An independent read of the published manifest rows and explicit library-plan entries yields the following **source-plan entries**, not named Logical Cases:

| Registered corpus family | Explicit plan entries | Evidence owner |
| --- | ---: | --- |
| Conformance (main, process snapshot, Actor, Group) | 245 | `protos/tests/conformance/{manifest.tsv,process/manifest.tsv,actor/manifest.tsv,group/manifest.tsv}` (235 + 3 + 4 + 3) |
| Standard Library (15 explicit corpus plans) | 86 | `protos/tools/test/RepositoryCorpusPlans.protos`, including logging 8 and Resolver 4 |
| Package Tool (TOML, version, lock, resolution input and D133 project-tree plans) | 50 | `protos/tests/package-tool/**/manifest.tsv` (5 + 10 + 10 + 3 + 0 + 2 + 8 + 9 + 3) |
| **Total source-plan entries** | **381** | Published manifests and source |

The earlier working-tree investigation recorded 598 physical `.protos` files under `protos/tests/`; the publication deletes exactly two such files and adds none under that tree, implying a **596-source** post-publication inventory if the baseline remained otherwise unchanged. Under that same baseline, 381 registered source identities would leave **215 excluded/auxiliary/JUnit-owned physical sources**. These last two numbers are *reconstructed from the earlier inventory*, **not** independently measured by an archived machine-readable audit report at `e5a55588362340aba3faa39e007b0228db77f543`. Do not mistake the inferred counts for a verified output of the final guard.

A source-plan entry identifies a physical source and its CaseSpec authority; it is **not** a Logical Case count. `colored-formatter.protos` is one registration but declares **16 named Tests**. Distinct D133 project identities may legitimately select one shared fixture source. The exhaustive numerical counts of all discovered Logical Cases and of the exact default-suite reachable Logical Cases were not captured in the maintainer's reported output; they remain an explicit evidence gap under TEST010 acceptance rather than an invented result. The count of source-plan entries must not be represented as the total number of executed Tests.

## Human validation and regression evidence

| Gate | Reported result | Evidence provenance |
| --- | --- | --- |
| Focal JUnit: `ProtosTestToolRepositoryCompletenessTest` | **10 PASS; 0 failures/errors**, per-test times below 10 seconds | Maintainer-provided Surefire XML summary |
| Focal JUnit: `ProtosTestToolRepositoryCorpusPlansTest` | **7 PASS; 0 failures/errors** | Maintainer-provided Surefire XML summary |
| Focal JUnit: `ProtosTestToolCaseSelectionPublicIntegrationTest` | **4 PASS; 0 failures/errors** | Maintainer-provided Surefire XML summary |
| Explicit source execution: `bin/protos test --file protos/tests/library/logging/colored-formatter.protos` | **16 passed; 0 failed** | Maintainer terminal output |
| Integrated `make test` | **PASS** | Maintainer report |
| Static `make check` | **PASS** | Maintainer report |
| `git diff --check` | **CLEAN** | Maintainer report |
| Product commit `e5a55588362340aba3faa39e007b0228db77f543` | **PUBLISHED** | Verified GitHub revision |

The regression set includes good/bad synthetic registration inputs, reversed-input diagnostic determinism, stale and duplicate references, exclusions, separation of source from Logical Case, and special-character/authority identity behavior. The public `--list-cases --file` integration test verifies all sixteen colored-formatter selections without running their bodies; the later explicit file execution separately confirms those bodies pass.

All tests above ran **before** final `pom.xml` / `CHANGELOG.md` materialization, as required by the maintainer. No test was rerun after that metadata change. The documentation-recording agent has not independently rerun Maven, `make` or the Protos executable, and does not claim a CI run or an unreported test.

## Remaining closure boundary

The product implementation and prevention guard are published, but [TEST010 #991](https://github.com/guillermomolina/protos/issues/991) remains the live authority over acceptance and closure. The exact global Logical Case totals and fully independent revision-bound physical-inventory report have **not** been established here. The Java corpus-root/loader table is an audit adapter mirroring CLI bindings; tests fail closed for a missing or surplus suite leaf, but do not fully prove equivalence against arbitrary future independent changes to the CLI's physical roots. These limitations should be reviewed against the Issue's acceptance contract before closing it; they are not reasons to modify the ratified D122 explicit-membership design implicitly.

This record identifies the exact Protos revision, not a floating `main`, and does not change normative semantics, tests, source, or live project scheduling.
