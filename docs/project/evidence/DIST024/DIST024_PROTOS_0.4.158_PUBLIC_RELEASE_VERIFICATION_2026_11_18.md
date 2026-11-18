# DIST024 — Protos v0.4.158 public JVM + Native release verification (2026-11-18)

**Formal owner:** [DIST024 — guillermomolina/protos#1129](https://github.com/guillermomolina/protos/issues/1129)  
**Published GitHub Release:** [Protos v0.4.158](https://github.com/guillermomolina/protos/releases/tag/v0.4.158)  
**Evidence type:** Revision-bound public publication and release-admission evidence. This document is non-normative; live GitHub coordination remains authoritative.

## Selected immutable release identity

| Coordinate | Verified value |
| --- | --- |
| Repository | `guillermomolina/protos` |
| Public version / tag | `0.4.158` / `v0.4.158` |
| Development baseline | [`91b92332f6870fd9f0903c829c2728a644d0dca4`](https://github.com/guillermomolina/protos/commit/91b92332f6870fd9f0903c829c2728a644d0dca4) |
| Baseline Maven version | `0.4.158-SNAPSHOT` |
| Published candidate / tag target (`PROTOS_REVISION`) | [`6b66acf256157dcaec4810c5ab5ec83a5026b939`](https://github.com/guillermomolina/protos/commit/6b66acf256157dcaec4810c5ab5ec83a5026b939) |
| Candidate commit | `release: materialize Protos 0.4.158 candidate` |
| Candidate ancestry and diff | One parent (the baseline above), only `pom.xml` changed; detached worktree was reported clean |
| Specification revision | `0.1.502` |
| Distribution model | `JVM_PLUS_NATIVE` |
| Release manifest SHA-256 | `915ef9be578c8427bf5e4742df1001476217e187db830d776a60946e9e89bb74` |
| Release notes SHA-256 | `0b9758eb19b5cddbf836f0324324ad477478a3e12768a8970f142615e483365d` |
| GitHub Release ID | `449526980` |
| Published at (GitHub UTC) | `2026-11-18T04:32:02Z` |
| Release visibility | `prerelease=true`; `draft=false` |

The public lightweight Git ref `refs/tags/v0.4.158` points directly to candidate commit `6b66acf...`. GitHub's generic Release field `target_commitish=main` does **not** override the actual tag target. At the independent verification checkpoint, `main` still pointed at development baseline `91b92332...`; the release-only version transition was not merged into `main`.

## Explicit selection and authorization provenance

- The maintainer selected the new `0.4.158-SNAPSHOT` baseline after [BUG036 #1130](https://github.com/guillermomolina/protos/issues/1130) corrected the obsolete Native interpreter-only archive gate. The previous `0.4.156` candidate failed release preparation and **was not published or reused**.
- The maintainer reported `DIST024_SELECTION=PASS`, with baseline `91b92332...`, version `0.4.158`, spec `0.1.502` and `release_publication_authorized=false`.
- Following the complete preparation and read-only prepublication hash review, the maintainer explicitly replied **“autorizo”** to the request to publish **this exact candidate SHA and manifest SHA-256** as GitHub prerelease. The exact authorization was recorded [in DIST024](https://github.com/guillermomolina/protos/issues/1129#issuecomment-6969071137).
- The human executor then reported `DIST024_PUBLICATION_AUTHORIZATION=PASS` using the externally stored official authorization record, and executed the unchanged `dist/publish_release.py` with candidate and release-envelope paths.

This approval is scoped to `v0.4.158`, candidate `6b66acf...`, and manifest digest `915ef9...`; it is not authorization to publish any other revision or artifact.

## Validation and human-executed publication evidence

The maintainer supplied terminal results; these commands were **not executed by the author of this record**.

| Gate | Evidence |
| --- | --- |
| Prior development tests | Maintainer reports `git diff --check` clean and all local tests PASS, without a complete new test transcript bound to the released commit |
| Native Image and Native archive | Included in the official complete release preparation PASS |
| Native extracted archive | Positive guest JIT Tier-2 admission for Bytecode DSL and builtin-call roots required by the repaired gate; prior BUG036 ZIP admission and subsequent complete release gate PASS. Detailed per-root counters were not included in the excerpt |
| Portable JVM archive | `DIST_BUILD: PASS` |
| Portable JVM end-user acceptance | `DIST001_B5_CROSS_SLICE: PASS` |
| Portable bundled Test Tool | `TEST001_H_PORTABLE_REPOSITORY_SUITE_CHECK: PASS passed=3827 failed=0` |
| JVM optimizer | `DIST_OPTIMIZING_RUNTIME_CHECK: PASS class=com.oracle.truffle.runtime.hotspot.HotSpotTruffleRuntime`; Truffle compiler version `25.5.5.0.3` |
| Multi-asset envelope creation | `MULTI_ASSET_ENVELOPE_GENERATION: PASS` |
| Multi-asset envelope verification | `MULTI_ASSET_ENVELOPE_VERIFICATION: PASS` |
| Official preparation | **`DIST005_RELEASE_PREPARATION: PASS`**; `COMMON_CANDIDATE_IDENTITY=PASS`; `CANDIDATE_FINAL_STATUS=CLEAN` |
| Read-only prepublication audit | **`DIST024_PREPUBLICATION_INSPECTION=PASS`**, candidate ancestry, POM-only diff, two archive digests and sidecar contents verified locally |
| Exact publication authorization file | `DIST024_PUBLICATION_AUTHORIZATION=PASS`, `ARTIFACT_CHECKSUMS=PASS` |
| Official public publication | **`DIST005_RELEASE_PUBLICATION: PASS`**, `PUBLIC_TAG_IDENTITY=PASS`, `GITHUB_PRERELEASE_METADATA=PASS`, `PUBLISHED_ASSET_SET=PASS`, `PUBLISHED_ASSET_DIGESTS=PASS`, `RELEASE_MANIFEST_IDENTITY=PASS` |

## Independent public GitHub verification

After publication, GitHub's releases, Git ref, commits, assets, and branch REST endpoints were read separately from the human-supplied output.

- [GitHub Release](https://github.com/guillermomolina/protos/releases/tag/v0.4.158) is non-draft and marked prerelease.
- Lightweight `v0.4.158` ref points to `6b66acf...`, and the candidate's sole parent is `91b92332...` with **only `pom.xml` modified**.
- Exactly **five** assets have `state=uploaded`; each `assets[].digest` has the exact SHA-256 below.

| Published asset | GitHub `sha256` |
| --- | --- |
| `protos-0.4.158-native-linux-x86_64.zip` | `08ba2986c5efa82d1a4f670a9fb7c2e10bc67609e7e81836e94920a7704d5233` |
| `protos-0.4.158-native-linux-x86_64.zip.sha256` | `e988f0b4ceb1ff33608afb635faabacd32e1f89e11a64ce5bbe3c4b37e58ff3b` |
| `protos-0.4.158-posix-jvm.zip` | `fa00bd14a44cf9aac0befc2c948a8075a293783c048ff4da96d23a105696d879` |
| `protos-0.4.158-posix-jvm.zip.sha256` | `5a152896147450f9413289c3bbb71a3b4b1e61299875383f4017734ed852e701` |
| `RELEASE_MANIFEST.txt` | `915ef9be578c8427bf5e4742df1001476217e187db830d776a60946e9e89bb74` |

The published notes include the same Native and JVM archive hashes and manifest identity. The **local** sidecar contents were checked against ZIP bytes before publication. The independent GitHub evidence here consists of GitHub API digest **metadata**; this record does not claim another download, remote archive rehash or independent new run of the public binaries.

## User-visible release claims and limits

The published release is the first public **0.4.x** implementation prerelease. The declared capabilities include Protos-first runtime normalization and Truffle-specialized execution, Native guest Tier-2 JIT compilation in the tested Bytecode DSL and builtin-call root families, and public FFI, SQLite and cryptographic facilities.

Native is `recommended-first-run`, Linux x86_64, dynamic linkage, conservatively advertised with glibc minimum 2.39, no external JDK. The portable POSIX JVM archive is the `compatibility-fallback`, requiring supported GraalVM Community JDK 25.0.5.0.3 / GraalVM and Truffle 25.5.5.0.3, with `HotSpotTruffleRuntime`. Linux Landlock support governs certain filesystem grants.

The release is **experimental** (Core v0.1 specification draft). No stable cross-release compatibility, universal guest Tier-2 compilation, broader Native OS/architecture support or general durable SQLite transactions are promised.

## Durable closure markers

The exact `PROJECT_RECORD_REVISION` must be recorded from the **actual commit that publishes this file** in `guillermomolina/protos-project-docs`; do not invent it and do not place a self-referential revision in this file. Re-read the file at that exact revision and cross-reference it from [DIST024](https://github.com/guillermomolina/protos/issues/1129) before issue closure.

```text
WORK_ITEM=DIST024
PROTOS_REVISION=6b66acf256157dcaec4810c5ab5ec83a5026b939
PROTOS_BASELINE_REVISION=91b92332f6870fd9f0903c829c2728a644d0dca4
RELEASE_VERSION=0.4.158
RELEASE_TAG=v0.4.158
SPECIFICATION_REVISION=0.1.502
RELEASE_MANIFEST_SHA256=915ef9be578c8427bf5e4742df1001476217e187db830d776a60946e9e89bb74
OWNER_EXACT_PUBLICATION_AUTHORIZATION=PASS
DIST005_RELEASE_PREPARATION=PASS_REPORTED
DIST024_PREPUBLICATION_INSPECTION=PASS_REPORTED
DIST005_RELEASE_PUBLICATION=PASS_REPORTED
PUBLIC_TAG_IDENTITY=PASS_GITHUB_API
PUBLIC_GITHUB_PRERELEASE=PASS_GITHUB_API
PUBLIC_ASSET_SET_AND_DIGEST_METADATA=PASS_GITHUB_API
PUBLIC_RELEASE_ID=449526980
PUBLIC_RELEASE_PUBLISHED_AT_UTC=2026-11-18T04:32:02Z
PROJECT_RECORD_REVISION=SEE_PUBLISHING_COMMIT_AND_LIVE_ISSUE
```

## Sources

- [Public release and files](https://github.com/guillermomolina/protos/releases/tag/v0.4.158), GitHub ref/commit/asset API responses.
- [DIST024 tracker](https://github.com/guillermomolina/protos/issues/1129) and [owner authorization](https://github.com/guillermomolina/protos/issues/1129#issuecomment-6969071137).
- [BUG036 correction](https://github.com/guillermomolina/protos/commit/cac6502df4cb8b88b73fbe7ebf83adf946b8d2b8) and [BUG036 tracker](https://github.com/guillermomolina/protos/issues/1130).
- Project policy: `guillermomolina/protos` `AGENTS.md`, `AGENTS.work/RELEASE.md`, `AGENTS.work/COORDINATION.md`, `dist/prepare_release.py`, and `dist/publish_release.py`.
