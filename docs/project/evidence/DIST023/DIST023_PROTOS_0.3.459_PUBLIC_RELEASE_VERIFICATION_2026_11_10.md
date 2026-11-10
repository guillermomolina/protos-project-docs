# DIST023 — Protos v0.3.459 public release verification (2026-11-10)

**Formal owner:** [DIST023, guillermomolina/protos#1106](https://github.com/guillermomolina/protos/issues/1106)
**Public release:** [Protos v0.3.459](https://github.com/guillermomolina/protos/releases/tag/v0.3.459)
**Evidence state:** Public release identity **verified**; human local validation **reported**; exact-candidate approval and release-script PASS transcripts **not established**. This is not a formal closure claim.

This historical evidence is non-normative. It does not change the language specification, approve a release retrospectively, or substitute for the issue's live coordination.

## Exact released revision and lineage

| Field | Verified public value |
| --- | --- |
| Product repository | `guillermomolina/protos` |
| Released version | `0.3.459` |
| Lightweight tag | `refs/tags/v0.3.459`, targeting a commit |
| Candidate / tag target | [`5f248eb0158bbf7a3ed701929132467b502e2306`](https://github.com/guillermomolina/protos/commit/5f248eb0158bbf7a3ed701929132467b502e2306) |
| Candidate commit message | `release: materialize Protos 0.3.459 candidate` |
| Candidate's sole parent / selected baseline | [`c9da7ff3e6c95f7a88948e875f61d67007fdeaa5`](https://github.com/guillermomolina/protos/commit/c9da7ff3e6c95f7a88948e875f61d67007fdeaa5) |
| Baseline Maven version | `0.3.459-SNAPSHOT` |
| Candidate-only diff | One commit and **only** `pom.xml`: `0.3.459-SNAPSHOT` → `0.3.459` |
| Observed `main` after publication | `c9da7ff3e6c95f7a88948e875f61d67007fdeaa5`; still `0.3.459-SNAPSHOT` |
| Release title | `Protos 0.3.459` |
| Channel | `prerelease=true`; `draft=false` |
| GitHub release ID | `439941892` |
| Published at (GitHub API, UTC) | `2026-11-10T04:20:13Z` |
| Specification revision in published notes | `0.1.501` |
| Distribution model | `JVM_PLUS_NATIVE` |
| Published manifest asset SHA-256 | `3784692e036c7330d22cd2f8ba7d99eb49f120bd45f915a8dd2dd39c7e99f22d` |

The original issue's planning observation `d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18` is **not** the released baseline. The public `main` advanced to `c9da7ff3...` through `DIST023: align Native validation with package and module contracts` in `dist/validate_native.py`; the released source expressly names this later revision. The actual Git tag ref and commit comparison, rather than GitHub's generic `target_commitish=main`, establish the source identity.

## Five uploaded assets and public SHA-256 values

The GitHub Releases assets endpoint returned **exactly five assets**, all `state=uploaded`:

| Asset | GitHub `assets[].digest` (SHA-256) |
| --- | --- |
| `protos-0.3.459-native-linux-x86_64.zip` | `003ae2fa574c422adfe8759ccb465356573e402e16db488234e53418669aa2dc` |
| `protos-0.3.459-native-linux-x86_64.zip.sha256` | `6efacf107e87786a1d06d9c4b7cef99aa0cb25fd05df27614d5318ef60e149f4` |
| `protos-0.3.459-posix-jvm.zip` | `bc550c6266e40070e8fce319e516357d0ce745369962e1169d2ea2495de3a410` |
| `protos-0.3.459-posix-jvm.zip.sha256` | `87c0f4bd944e6ace1328ebd04a7097a962e811571b33fe3b5a47c76d4dd3ba1a` |
| `RELEASE_MANIFEST.txt` | `3784692e036c7330d22cd2f8ba7d99eb49f120bd45f915a8dd2dd39c7e99f22d` |

The Native and JVM archive SHA-256 values also agree with those printed in the public release notes. This is independent **GitHub API digest-metadata verification**, not an additional download, local ZIP rehash, check of the adjacent checksum files' contents, direct manifest byte audit, or installed distribution smoke. None of those latter operations is claimed.

## Disclosed runtime and supported feature envelope

The published notes identify the Native distribution as recommended first-run, Linux x86_64 with policy glibc >= 2.39, GraalVM Native Image `25.5.5.0.3` / JDK `25.0.5.0.3`, and no external JDK requirement. The portable POSIX JVM fallback requires externally installed GraalVM Community JDK 25, Truffle `25.5.5.0.3` and `HotSpotTruffleRuntime`.

The public notes claim MODULE/SOURCE and `package.toml`, explicit Network, confined Filesystem, Entropy and Clock authority, RootActor hosting/lifecycle, HMAC-SHA256, canonical Base64url, HS256 JWT and Argon2id. They disclose experimental Core v0.1, Java and Linux/glibc limits, Native interpreter-only guest execution (PLAT045), explicit grants, Filesystem kernel/Landlock requirements, breaking module/package migration, and independent RealWorld / extension / devcontainer status. These are verified **release-note declarations**, not independent functional tests performed for this record.

## Human-reported source validation

The maintainer reported in the active DIST023 follow-up, without raw logs or a commit fingerprint:

```text
git diff --check = PASS (clean)
make test = PASS
make check = PASS
```

These are **human-reported** results. They are not claimed as executed by the evidence author, nor as direct proof of binary acceptance. The message does not bind the local checkout to the exact `c9da7ff3...` baseline. A one-file release-only candidate diff does not alone establish completion of the separate release preparation gates.

## Approval and release execution evidence still pending

The original live issue had **zero prior comments** at the verification checkpoint. Neither this interaction nor the issue supplied an explicit prior owner authorization simultaneously identifying the exact candidate `5f248eb...` and manifest digest `3784692e...`, or transcripts showing `DIST005_RELEASE_PREPARATION: PASS` / `DIST005_RELEASE_PUBLICATION: PASS`.

A valid public release **does exist**, but its existence is not evidence of when or by whom the precise candidate was approved, whether the documented scripts were the executing entry points, or which local admission gates completed. These are **NOT ESTABLISHED**, not inferred PASS and not inferred FAIL.

| Gate | Recorded state |
| --- | --- |
| Public tag and one-parent POM-only release lineage | **PASS**, independently checked |
| GitHub prerelease title, channel and source identity | **PASS**, independently checked |
| Five uploaded public assets with published digests | **PASS**, GitHub API |
| Maintainer's `git diff --check`, `make test`, `make check` | **PASS REPORTED**, unpinned local checkout |
| Exact-candidate prior owner authorization | **EVIDENCE PENDING** |
| `DIST005_RELEASE_PREPARATION: PASS` output | **NOT REPORTED** |
| `DIST005_RELEASE_PUBLICATION: PASS` output | **NOT REPORTED** |
| Independently downloaded artifact/manifest validation | **NOT EXECUTED HERE** |
| Formal DIST023 closure | **PENDING** the separate release-approval/execution evidence |

## Stable evidence marker

```text
WORK_ITEM=DIST023
PROTOS_BASELINE_REVISION=c9da7ff3e6c95f7a88948e875f61d67007fdeaa5
PROTOS_RELEASE_REVISION=5f248eb0158bbf7a3ed701929132467b502e2306
RELEASE_VERSION=0.3.459
RELEASE_TAG=v0.3.459
SPECIFICATION_REVISION=0.1.501
RELEASE_MANIFEST_SHA256_GITHUB=3784692e036c7330d22cd2f8ba7d99eb49f120bd45f915a8dd2dd39c7e99f22d
PUBLIC_GITHUB_RELEASE=PASS
PUBLIC_ASSET_SET_AND_DIGEST_METADATA=PASS
HUMAN_GIT_DIFF_CHECK=PASS_REPORTED
HUMAN_MAKE_TEST=PASS_REPORTED
HUMAN_MAKE_CHECK=PASS_REPORTED
OWNER_EXACT_CANDIDATE_AUTHORIZATION=NOT_ESTABLISHED
DIST005_RELEASE_PREPARATION=NOT_REPORTED
DIST005_RELEASE_PUBLICATION=NOT_REPORTED
FORMAL_CLOSURE=PENDING
```

**Follow-up:** Cross-reference this record from [DIST023 #1106](https://github.com/guillermomolina/protos/issues/1106) and retain `status:review` pending missing authorization and execution provenance. Re-read the record by its exact `protos-project-docs` commit SHA before any eventual closure. Do not rerun the entire product test suite just to fill a missing publication transcript.

## Evidence sources and policy boundary

- [Public GitHub Release](https://github.com/guillermomolina/protos/releases/tag/v0.3.459), tag reference, Releases/Assets REST API and candidate/parent commit API.
- [Release-only candidate](https://github.com/guillermomolina/protos/commit/5f248eb0158bbf7a3ed701929132467b502e2306) and [development baseline](https://github.com/guillermomolina/protos/commit/c9da7ff3e6c95f7a88948e875f61d67007fdeaa5); one-commit comparison.
- Product `pom.xml`, `spec/PROTOS_SPEC_CHANGELOG.md`, `AGENTS.md`, `AGENTS.work/RELEASE.md`, `AGENTS.work/COORDINATION.md`, `dist/prepare_release.py`, `dist/publish_release.py`.
- `guillermomolina/protos-project-docs`: `AGENTS.md`, `docs/project/README.md`, `docs/project/evidence/README.md`, and [DIST021 precedent](../DIST021/DIST021_PROTOS_0.3.442_PUBLICATION_EVIDENCE.md).

This record modifies no file in `guillermomolina/protos`.
