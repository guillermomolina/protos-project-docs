# BUG037 — Published Test Tool host-failure completion and callback carried-cell regression evidence (2026-11-18)

**Non-normative, revision-bound project evidence.** This snapshot records the source publication and the maintainer's reported validation. It neither changes Protos semantics nor independently proves a 10,000-frame recursion capability.

## Publication identities

- Owning Issue: [BUG037 / protos#1131](https://github.com/guillermomolina/protos/issues/1131).
- Exact published product commit: [`guillermomolina/protos@afa18073e0fd48804c24b66c2d8d3e26aaa5cbcb`](https://github.com/guillermomolina/protos/commit/afa18073e0fd48804c24b66c2d8d3e26aaa5cbcb).
- Commit subject: `BUG037: Admit carried-cell inline callbacks, keep transported cells for custom-selected callbacks, and complete logical Cases on host stack overflow`.
- Product implementation version: **0.4.157-SNAPSHOT**; the same commit updates `pom.xml` and `CHANGELOG.md`.
- Subsequent, independently owned HTTP test synchronization: [BUG038 / #1132](https://github.com/guillermomolina/protos/issues/1132), product [`91b92332f6870fd9f0903c829c2728a644d0dca4`](https://github.com/guillermomolina/protos/commit/91b92332f6870fd9f0903c829c2728a644d0dca4) / 0.4.158-SNAPSHOT.

## Actual published delta

The commit includes four compiler/lowering source changes:

- `CanonicalBindingAnalysis.java`: allow exact outer lexical CELL captures already transported by the physical parent; distinguish transportable captured cells from merely name-selectable bindings.
- `CanonicalCompilationFacts.java`: account for an inline callback also materialized on the custom-selection path when determining persistent-frame requirements.
- `CanonicalInlineStoragePlan.java`: preserve transported CELL storage when a selected custom method receives a genuine Closure.
- `CanonicalToBytecodeLowerer.java`: choose between by-name selection and explicitly transported captured cells, without changing the canonical boolean path into ordinary message dispatch.

Two Test Tool execution files settle the logical Case's terminal Future when the host worker encounters `StackOverflowError`: `ProtosLogicalCaseCallerCompletion.java` and `ProtosTestLogicalCaseExecutionFacility.java`. The change retains the host failure for completion rather than leaving the Case pending.

JUnit regression coverage is extended in `ProtosPerf026B1BooleanInlineCallbackTest.java` (outer-cell capture, custom receiver selection and the creating home of `^7`) and `ProtosTestLogicalCaseExecutionFacilityTest.java` (host stack-overflow terminal completion). These eight Java files and the two publication metadata files are the exact ten paths recorded in the published commit.

## Maintainer-provided validation

| Check | Reported observation | Provenance |
| --- | --- | --- |
| `ProtosPerf026B1BooleanInlineCallbackTest` | **19/19 PASS** | Supplied focused Maven output on the pre-publication candidate |
| `ProtosTestLogicalCaseExecutionFacilityTest` | **5/5 PASS** | Same Maven run; includes host overflow / Future settlement |
| Combined focused set | **24/24 PASS**, `BUILD SUCCESS` | `mvn '-Dtest=ProtosPerf026B1BooleanInlineCallbackTest,ProtosTestLogicalCaseExecutionFacilityTest' test` |
| `git diff --check` | **PASS** | Maintainer terminal output on the eight-file pre-publication candidate |
| `make check` | **PASS** | Maintainer transcript prior to final version publication, including Native runtime compilation checks |
| Integrated `make test` before BUG038 | **FAIL** in parallel Java lane, unrelated HTTP socket read timeout | 6004 Java tests, 0 assertion failures, 1 error, 11 skipped; subsequent Java serial/guest lanes were not reached in that run |
| Later local suite | Maintainer says **all local tests pass** and the published source is pushed | Summary reported with BUG038 publication; exact full-suite transcript and post-commit checkout SHA not supplied |

The integrated failure was `ProtosHttpHostFactoryIntegrationTest.rootActorFailureTerminatesTheProcessAndRetiresEveryWorker`, later independently tracked and repaired as BUG038. A subsequent green claim does not erase the original failure; the separate [BUG038 evidence](../BUG038/BUG038_HTTP_HOST_TERMINATION_TEST_PUBLICATION_2026_11_18.md) records its own checks.

## Boundaries and disposition

- The Test Tool fix establishes **Future completion on host overflow**, not that every recursive Protos program executes 10,000 nested calls successfully. [BUG008 / #681](https://github.com/guillermomolina/protos/issues/681) owns historical deep-recursion stack behavior.
- The published commit deliberately includes compiler/callback changes alongside the Test Tool correction; this evidence reports the actual published delta rather than claiming they were separately committed.
- No Native executable run or benchmark improvement is inferred from the 24 focused Java tests.
- This snapshot is not a new release, branch, tag or normative design decision. Live Issue state and closure belong to the linked GitHub Issue.

**Evidence source:** maintainer-provided Maven/static validation outputs and later local-suite summary, corroborated with the exact published Protos Git commit and root changelog. No build/test was executed when preparing this record.
