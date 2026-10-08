# PERF031-B — Retained publication and focal validation evidence

Date: 2026-10-08

## Publication identity

```text
PARENT_ISSUE=guillermomolina/protos#787
IMPLEMENTATION_SLICE=PERF031-B
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=b7f7175c46134585c1b45d216b6adbcde0896220
IMPLEMENTATION_VERSION=0.3.287-SNAPSHOT
PRODUCT_SEMANTICS_CHANGED=NO
TEST_INFRASTRUCTURE_CHANGED=NO
SPECIFICATION_CHANGED=NO
IMPLEMENTATION_PUBLISHED=YES
ISSUE_CLOSED=NO
```

[Exact GitHub implementation commit](https://github.com/guillermomolina/protos/commit/b7f7175c46134585c1b45d216b6adbcde0896220).

The Protos commit changes **exactly seven tracked files**: five Java test classes, `pom.xml` and root `CHANGELOG.md`. Its reviewed diff shows new class-local lazy Core Prelude holders, retained fresh module activations and only a matching release-metadata update. The exact changed-path list is:

- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolSuiteGraphTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolResourceCatalogSchemaTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolResourceRequirementsSchemaTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolSequentialRunnerTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosExternalPackagePlanningPreflightTest.java`
- `pom.xml`
- `CHANGELOG.md`

The preceding PERF031-A investigation was published at project-record revision `6db3f746236aeeb88034c5c8484aa9b6c1e875f2` as `docs/project/evidence/PERF031/PERF031_A_JAVA_INTEGRATION_TEST_COST_CAUSAL_AUDIT.md`. It established the source and historical CI evidence that routed this bounded implementation.

## Owner-reported focal validation

The maintainer explicitly reported that the PERF031-B focal regression passed after source edits and before publication metadata finalization:

```text
FOCAL_VALIDATION=PASS
VALIDATION_PROVENANCE=HUMAN_REPORTED
TEST_COUNT=25
FAILURES=0
ERRORS=0
SKIPPED=0
REPORTED_MAVEN_BUILD_WALL_TIME_SECONDS=39.5
PER_CLASS_TIMING_EVIDENCE=NOT_PROVIDED
SAME_HOST_PRECHANGE_BASELINE=NOT_AVAILABLE
MEASURED_PERFORMANCE_WIN=NOT_ESTABLISHED
INTEGRATED_SUITE=NOT_RUN
```

The 39.5 seconds is **total Maven build time including compilation**, not a per-test-class measurement, isolated baseline, or proven gain. The human's observed focal run showed no ordering or isolation defect. This document does not claim independent agent-executed validation, a post-publication full suite, or a `git diff --check` output not supplied by the human.

The maintainer judged the full `make test` unnecessary under the changed-path adaptive policy: five self-contained test classes, no shared test infrastructure, no executable production source changes, and no parent-Issue closure. This is a **validation-scope decision**, not a claim that the full suite was run.

## Structural before/after result

| Test class / helper | Before | After |
|---|---|---|
| ProtosTestToolSuiteGraphTest | One Core bootstrap per `fixture()` call, approximately 14 | One lazy class Prelude |
| ProtosTestToolResourceCatalogSchemaTest | One per `fixture()` call | One lazy class Prelude |
| ProtosTestToolResourceRequirementsSchemaTest | One per `fixture()` call | One lazy class Prelude |
| ProtosTestToolSequentialRunnerTest | Two bootstrap calls | One lazy class Prelude |
| ProtosExternalPackagePlanningPreflightTest: `assertBorrowedCustodiesStillOpen` | Three helper Core bootstrap calls | One lazy class Prelude |

The structural improvement is fewer Core initializations in the reviewed paths. Test assertions remain, each fixture or evaluation receives its fresh `ProtosActivation`, and unrelated Package verification/planning, Process execution and custody lifetimes remain independent. These conclusions are grounded in the published source diff plus the human's successful focal results; they are **not** a performance-speedup estimate.

## Residual PERF031 scope

`ProtosPackageTestLogicalCaseExecutionFacilityTest` retains per-test Core bootstrap. At the publication revision it constructs the same Package-flavored resolver configuration for its eleven `@Test` scenarios, including a finite exact overlay of real Package Tool module identities over the bundled Test Tool and Standard Library resolvers. Each method separately constructs a `ProtosPolyglotRuntimeHost`, a `ManualSubmission` queue and a fresh activation. Tests explicitly prove isolated Case execution (a mutable counter resets), selected-case authority, project-tree physical authority, exact module resolution, and rejection paths.

The exact resolver implementation `ProtosExactModuleOverlayResolver` keeps immutable `Map.copyOf` lookup maps and a fallback; bundled Test Tool and Standard Library resolvers hold fixed path/root configuration and perform resolution and source loading on demand. This **suggests** class-local sharing of only a Prelude and possibly its immutable resolver, but it is not authority to share mutable Process/Actor state, runtime hosts, source roots or submission queues.

The next work should establish the remaining safety invariants and, if satisfied, implement one bounded local reuse without changing the tested runtime behavior. A human-owned focal validation must check all logical Case scenarios, including mutable-state isolation and physical-authority confinement, before publication.

No new formal Issue is required for this bounded continuation. Leave `guillermomolina/protos#787` **OPEN / READY**, priority unset, no artificial dependency on PERF032. Reconcile any other genuinely material Java integration-test cost owner before claiming final PERF031 closure.

## Final publication/result markers

```text
PERF031_B_PRODUCT_REVISION=b7f7175c46134585c1b45d216b6adbcde0896220
PERF031_B_IMPLEMENTATION=COMPLETE_AND_PUSHED
PERF031_B_PATCH_SCOPE=FIVE_LOCAL_TEST_FILES_PLUS_VERSION_AND_CHANGELOG
PERF031_B_FOCAL_TESTS=25
PERF031_B_FOCAL_FAILURES=0
PERF031_B_FOCAL_ERRORS=0
PERF031_B_FOCAL_SKIPS=0
PERF031_B_FOCAL_VALIDATION_PROVENANCE=HUMAN_REPORTED
PERF031_B_FULL_SUITE=NOT_REQUIRED_BY_OWNER_IMPACT_ASSESSMENT
PERF031_B_MEASURED_SPEEDUP=NOT_ESTABLISHED
PERF031_B_STRUCTURAL_BOOTSTRAP_REDUCTION=YES
NEXT_SLICE=PERF031-C
PARENT_ISSUE_CLOSE=NO
```
