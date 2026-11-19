# I164 — Explicit standalone CLI Resolver authority: implementation publication and validation (2026-11-19)

**Role:** revision-bound implementation evidence; not a new decision or a DNS protocol change.  
**Implementation owner:** [I164 / protos#1109](https://github.com/guillermomolina/protos/issues/1109).  
**Approved decision:** [D249-A1 / protos#1075](https://github.com/guillermomolina/protos/issues/1075); [ratified decision](../../decisions/tooling/D249_EXPLICIT_RESOLVER_CLI_RATIFIED.md).  
**Original Resolver authority:** [D223-A1 / protos#987](https://github.com/guillermomolina/protos/issues/987); `spec/io/NETWORK.md` §14 and `spec/io/PROCESS_IO.md`.  
**Exact published product revision:** [`guillermomolina/protos@5f40e736d8942465083ee844a3a5356d1dc831b1`](https://github.com/guillermomolina/protos/commit/5f40e736d8942465083ee844a3a5356d1dc831b1).  
**Commit subject:** `I164: Add explicit Resolver CLI authority across application entry modes`.  
**Recorded product version:** `0.4.177-SNAPSHOT` (matching `pom.xml` and `CHANGELOG.md` in the published commit).  
**Specification changelog revision:** `0.1.503` (CLI provisioning; existing Resolver protocol semantics unchanged).

## 1. Publication inspection

The exact commit is reachable in `guillermomolina/protos` and changes **33 paths**. It includes five existing Java production files, CLI/workspace/package/service/lifecycle/isolation JUnit tests, Native CLI regression coverage, documentation, Protos examples, the Process I/O specification wording, and finalization metadata. No new production Java class or DNS provider is required.

Production files changed:

- `src/main/java/com/guillermomolina/protos/cli/ProtosCli.java`
- `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosStandaloneHostedSession.java`
- `src/main/java/com/guillermomolina/protos/execution/packages/ProtosPackageRunDriver.java`
- `src/main/java/com/guillermomolina/protos/execution/workspace/ProtosWorkspacePackageApplicationExecution.java`
- `src/main/java/com/guillermomolina/protos/execution/workspace/ProtosWorkspaceRunDriver.java`

The CLI accepts independent, default-off `--allow-resolver` for direct SOURCE files, `-e`, REPL, `debug`, workspace/package `run`, and SOURCE/MODULE service entries. Only explicit selection supplies the already-existing `ResolverGrant.HOST_RESOLVER` to the initial application RootActor module. Prior overloads default to `ResolverGrant.NONE`. The existing `ProtosPolyglotRuntimeHost.provisionHostResolver` and Process bootstrap are reused; failure before Process handoff retires the undelivered capability.

The CLI rejects duplicated/malformed value-form flags and use in bundled Tools and non-application modes. Flags after the entry are application arguments, not authorizations. `--allow-network` alone supplies no Resolver and `--allow-resolver` alone supplies no TCP capability. Imports, newly created Actors, isolated P and newly hosted Processes do not gain implicit authority. The previously ratified D223 Future, validation, cancellation, resource lifecycle and cache semantics are not redefined.

## 2. Published regression and documentation surfaces

- New deterministic launcher regression: `src/test/java/com/guillermomolina/protos/cli/resolver/ProtosI164ResolverCliTest.java`, including SOURCE/`-e`/REPL authority matrices, independent CLI option composition, post-entry argument preservation, reused launcher, invalid/duplicate/Tool paths and service lifetime. It deliberately performs no real host DNS query.
- Existing CLI tests, `ProtosCli010InspectTest` help assertions, workspace tests, public debugger/DAP tests, package driver/custody tests, Resolver authority and RootActor service/lifecycle tests are updated.
- `scripts/native-test.sh` contains native CLI Resolver grant/denial/argument regression cases. Their presence in the commit is verified. The maintainer confirms that **all local tests pass**; individual Native execution output was not separately attached.
- `docs/guide/tools/cli.md`, networking/HTTP/TLS, embedding and Process I/O guides, example index and authority examples are updated; `protos/examples/io/resolver-only.protos` is a new Protos-first, manual hostname-resolution demonstration.
- `spec/io/PROCESS_IO.md` specifies standalone CLI grant selection; `spec/io/NETWORK.md` §14 remains the owner of Resolver's guest-visible semantics.

## 3. Validation provenance and limits

The maintainer supplied a **pre-finalization** local transcript at `0.4.176-SNAPSHOT` showing clean `git diff --check` and successful `mvn -DskipTests test-compile` (2026-11-19 11:08:34 UTC). An affected-regression run at that intermediate state compiled and ran **393 tests, with 392 passing and one failing**: `ProtosCli010InspectTest.helpDocumentsInspectOptionScopeAndModule` still expected the earlier help Usage line. The published `0.4.177-SNAPSHOT` commit contains the corrected exact help expectations and the extended independent-grant composition tests. The earlier 393-test result **must not** be re-labelled a final PASS.

After pushing the exact I164 commit, the maintainer explicitly reported on 2026-11-19:

- `git diff --check` is clean;
- **all local tests pass**;
- `I164: Add explicit Resolver CLI authority across application entry modes` was pushed.

The pushed commit, its full SHA, changed paths, version/changelog and final help-test source were independently inspected via GitHub. The final all-local-tests statement is **owner-reported**, not an independently observed post-commit full-suite transcript. No exact final test count, post-commit commands, CI run, Native image build/execution, native DNS outcome, or release artifact digest was supplied. Do not infer any of those from the presence of `scripts/native-test.sh` regressions or the local-test statement.

## 4. Final completion and owner acceptance (2026-11-19)

The maintainer explicitly stated **"todos los tests pasan en local"**, after publishing the I164 commit, and reiterated that **all** local tests have passed when the previous coordination update incorrectly treated the absence of a separate Native transcript as a new blocking condition. This is explicit owner attestation of the complete local test set, not merely the pre-finalization 393-test run.

**Disposition: I164 COMPLETE, closure authorized by the maintainer's acceptance of all local tests.** The exact published product revision, final source and corrected tests are independently verified. No further test execution or repeat confirmation is required for I164 closure. This evidence distinguishes the owner's complete local validation report from individually supplied command transcripts: no exact final test count, Native-specific log, native DNS result, release artifact, or CI status is fabricated. Neither a separate public release nor any D223 DNS protocol redesign is claimed.

No new child issue, separate Native gate or follow-up acceptance issue is created by this closure. I164's GitHub Issue owns the live completed state.

## Compact evidence

```text
WORK_ITEM=I164
DESIGN_SOURCE=D249-A1
PROTOS_REVISION=5f40e736d8942465083ee844a3a5356d1dc831b1
PROTOS_REVISION_VERIFIED=PASS
PRODUCT_VERSION=0.4.177-SNAPSHOT
SPEC_CHANGELOG_REVISION=0.1.503
PRODUCT_CHANGED_PATHS=33
NEW_PRODUCTION_JAVA_CLASSES=0
EXISTING_RESOLVER_BACKEND_REUSED=YES
PREFINAL_TEST_COMPILE=PASS
PREFINAL_JUNIT=392_PASS_1_FAILURE_OF_393
PREFINAL_FAILURE=ProtosCli010InspectTest.helpDocumentsInspectOptionScopeAndModule
PUBLISHED_HELP_TEST_CORRECTION_VERIFIED=PASS
FINAL_LOCAL_TESTS=PASS_OWNER_REPORTED
FINAL_LOCAL_GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
POST_COMMIT_FULL_TEST_TRANSCRIPT=NOT_PROVIDED
NATIVE_SPECIFIC_TRANSCRIPT=NOT_SEPARATELY_PROVIDED
ALL_LOCAL_TESTS=PASS_OWNER_ATTESTED
ISSUE_CLOSURE=COMPLETED_ON_OWNER_LOCAL_TEST_ACCEPTANCE
PUBLIC_RELEASE=NOT_CLAIMED
```
