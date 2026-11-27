# DIST027 — Protos v0.4.252 public JVM + Native prerelease verification (2026-11-27)

Formal owner: [DIST027 #1219](https://github.com/guillermomolina/protos/issues/1219). Public release: [v0.4.252](https://github.com/guillermomolina/protos/releases/tag/v0.4.252). This is immutable evidence, not live issue state.

## Exact identity

- Baseline: [70a5ccf4560e7236161860bc4164748b92cdebdf](https://github.com/guillermomolina/protos/commit/70a5ccf4560e7236161860bc4164748b92cdebdf), 0.4.252-SNAPSHOT.
- Candidate: [328ca5a3ecde93ad61ed688ad6fa2790995a1ea7](https://github.com/guillermomolina/protos/commit/328ca5a3ecde93ad61ed688ad6fa2790995a1ea7).
- Specification: 0.1.519; distribution: JVM_PLUS_NATIVE; GitHub Release ID 460729388; published 2026-11-27T14:49:15Z; prerelease=true; draft=false.
- GitHub independently confirms one parent (baseline), only pom.xml changed (0.4.252-SNAPSHOT to 0.4.252), lightweight tag pointing at candidate, and main still at baseline at verification.

## Owner decision and validation

The first selected v0.4.251 candidate was interrupted during automatic Java functional validation and was not published. The owner explicitly required removal of all tests from the distribution harness, without moving them to CI or elsewhere. The published baseline removes Native execution admission, exhaustive functional validation and portable smokes from release preparation, with repository test entry points and CI unchanged. See [policy at the exact baseline](https://github.com/guillermomolina/protos/blob/70a5ccf4560e7236161860bc4164748b92cdebdf/AGENTS.work/RELEASE.md).

Maintainer supplied clean git diff check, Python and shell syntax PASS, no matching test invocations in the three release entry points, and 17/17 focal harness regressions PASS before version finalization. [CI run 46010551912](https://github.com/guillermomolina/protos/actions/runs/46010551912) was independently verified green on the earlier product baseline 509221a9d28b79e9da9accf721fad2358d81ec49, not on the final release candidate.

For v0.4.252 the maintainer supplied DIST005_RELEASE_PREPARATION: PASS, DIST001_B5_ARCHIVE_VERIFICATION: PASS, DIST005_D1_MULTI_ASSET_ENVELOPE: PASS, COMMON_CANDIDATE_IDENTITY=PASS, CANDIDATE_FINAL_STATUS=CLEAN and return code 0. No product tests were invoked by the new distribution workflow. No extracted runtime execution or final-candidate full-suite PASS is claimed.

The maintainer explicitly typed PUBLISH after the exact version, candidate SHA and manifest SHA-256 were displayed. Official publication returned DIST005_RELEASE_PUBLICATION: PASS, PUBLIC_TAG_IDENTITY=PASS, GITHUB_PRERELEASE_METADATA=PASS, PUBLISHED_ASSET_SET=PASS, PUBLISHED_ASSET_DIGESTS=PASS, RELEASE_MANIFEST_IDENTITY=PASS, RELEASE_PUBLICATION_AUTHORIZED=YES and return code 0.

## Public assets

| Asset | GitHub API SHA-256 |
| --- | --- |
| protos-0.4.252-native-linux-x86_64.zip | 499b8f1f3304db6e38fdf5f14183f1ddd87e05926ff852a755d36cbe2e81919f |
| protos-0.4.252-native-linux-x86_64.zip.sha256 | d30ec52b1210c11c46894b00878d5dd466743a9e4f601a5029049f8df40f740e |
| protos-0.4.252-posix-jvm.zip | e873f6579a01b74960c93aa8531ce1cb906e87c4215fe6f6408b10443ff52829 |
| protos-0.4.252-posix-jvm.zip.sha256 | dbdb67cd261f46b68e1180441122d9e73ca38ae297ef9dc333a6a9f891b74fa6 |
| RELEASE_MANIFEST.txt | c60281e417fa332afe6751a6bca9fb467475cd3fdb04864b1d07e869b76ae436 |

The independent GitHub verification checked public asset metadata and digests, not an additional download or execution; official publisher reported downloaded-asset digest verification PASS. Release notes are the release body, not a sixth asset.

Native is recommended for Linux x86-64, declared glibc minimum 2.39 (observed GLIBC symbol maximum 2.34), dynamic linkage, GraalVM 25.5.5.0.3, no external Java. Portable POSIX/JVM fallback requires GraalVM CE 25.5.5.0.3 for JDK 25.0.5.0.3. See public release notes for exact capabilities and limitations, including experimental Core 0.1, finite registered FFI signatures, trusted in-process native libraries, external native dependencies and separate portable JVM FFI provider. No performance or unsupported platform claim is made.

## Durable coordinates

```text
WORK_ITEM=DIST027
PROTOS_BASELINE_REVISION=70a5ccf4560e7236161860bc4164748b92cdebdf
PROTOS_REVISION=328ca5a3ecde93ad61ed688ad6fa2790995a1ea7
RELEASE_VERSION=0.4.252
RELEASE_MANIFEST_SHA256=c60281e417fa332afe6751a6bca9fb467475cd3fdb04864b1d07e869b76ae436
DIST005_RELEASE_PREPARATION=PASS_REPORTED
OWNER_EXACT_PUBLICATION_AUTHORIZATION=PASS
DIST005_RELEASE_PUBLICATION=PASS_REPORTED
PUBLIC_TAG_RELEASE_ASSET_METADATA=PASS_GITHUB_API
DISTRIBUTION_PRODUCT_TEST_EXECUTION=NONE
PROJECT_RECORD_REVISION=SEE_PUBLISHING_COMMIT_AND_LIVE_ISSUE
```

Actual GitHub Discussion, Reddit and X publication URLs are pending owner publication. See [formal issue](https://github.com/guillermomolina/protos/issues/1219) for the exact record revision and closure.

## Supplemental maintainer validation report (2026-11-27)

After public release and closure, the maintainer explicitly confirmed that `git diff --check` was clean and all local tests passed. This is additional owner-reported repository validation, not a new distribution admission requirement. The report did not supply the exact test commands, execution counts, or an independently identified tested checkout SHA; it must not be interpreted as a separately verified full suite on the detached public candidate. The earlier 17/17 focal harness transcript and exact public artifact evidence remain unchanged.

```text
MAINTAINER_GIT_DIFF_CHECK=PASS_REPORTED
MAINTAINER_LOCAL_TESTS=PASS_REPORTED
DISTRIBUTION_PRODUCT_TEST_EXECUTION=NONE
```
