# DIST025 — Protos v0.4.192 public JVM + Native prerelease verification (2026-11-23)

**Formal owner:** [DIST025 — guillermomolina/protos#1151](https://github.com/guillermomolina/protos/issues/1151)  
**Public release:** [Protos v0.4.192](https://github.com/guillermomolina/protos/releases/tag/v0.4.192)  
**Evidence role:** Immutable, revision-bound publication record. GitHub Issues remain the live work-state authority.

## Exact release identity

| Coordinate | Verified value |
| --- | --- |
| Repository | `guillermomolina/protos` |
| Release version / tag | `0.4.192` / `v0.4.192` |
| Development baseline | [`9c56aba2c8b84a9c44440cb361db85b53fcf30b4`](https://github.com/guillermomolina/protos/commit/9c56aba2c8b84a9c44440cb361db85b53fcf30b4) |
| Baseline Maven version | `0.4.192-SNAPSHOT` |
| Public candidate / tag target | [`45801061a2878256969901367057cb1be0d23971`](https://github.com/guillermomolina/protos/commit/45801061a2878256969901367057cb1be0d23971) |
| Candidate commit message | `release: materialize Protos 0.4.192 candidate` |
| Candidate ancestry and diff | Exactly one parent, the development baseline; only `pom.xml` modified (GitHub independently verified) |
| Core specification revision | `0.1.507` |
| Distribution model | `JVM_PLUS_NATIVE` |
| Release manifest SHA-256 | `63ae6ea92bd35ffc3ecd628158d0a11112388010d2c7c8bde4bc155af4e4a1b5` |
| Release notes SHA-256 | `9541c8120aca0d169efcb810d6766ced661ed8b5b1954c66ffa36ac3376deda5` |
| GitHub Release ID | `455096529` |
| Published at (UTC) | `2026-11-23T07:18:07Z` |
| Public visibility | `prerelease=true`; `draft=false` |

GitHub independently confirms that the lightweight `refs/tags/v0.4.192` points directly to the candidate SHA. The release API's `target_commitish=main` field does not change the actual tag target. Remote `main` still pointed to baseline `9c56aba2...` at verification time; the release-only POM transition was **not** merged into `main`.

## Preparation, explicit authorization and publication

The maintainer selected the clean, synchronized `0.4.192-SNAPSHOT` baseline after reviewing the 41-commit delta from the earlier public release `v0.4.158`. The existing `dist/prepare_release.py` generated a detached release-only candidate and validated Native and portable-JVM distributions. The maintainer supplied terminal markers:

- `DIST005_RELEASE_PREPARATION: PASS`
- `COMMON_CANDIDATE_IDENTITY=PASS`
- `CANDIDATE_FINAL_STATUS=CLEAN`
- `RELEASE_PUBLICATION_AUTHORIZED=NO` at the end of preparation
- `TAG_CREATED=NO` and `PUBLIC_RELEASE_CREATED=NO` at the end of preparation

The maintainer then reported the candidate's exact one-parent, POM-only lineage and clean worktrees; local `sha256sum -c` returned `OK` for both archives. After discovering an unwanted trailing character in an early display of the release-note limitation, the existing `dist/prepare_release_metadata.py` regenerated the metadata from the **unchanged** Native and JVM ZIPs. `dist/verify_release_metadata.py` returned `DIST005_D1_MULTI_ASSET_ENVELOPE: PASS`. The final notes contain the correct qualification that Native external OS process execution is not yet verified. The earlier manifest SHA-256 `083187601e1192db426b40eb7489d8be0dc1875633da791e5e414020bbd94d6f` was superseded and was **not** the authorized public artifact.

Following inspection of the exact final manifest and notes hashes, the maintainer explicitly replied **“autorizo”** to publishing **candidate `45801061...` and manifest `63ae6e...`** as `v0.4.192` prerelease. The exact authorization and its scope are recorded in [DIST025's owner-authorization comment](https://github.com/guillermomolina/protos/issues/1151#issuecomment-7094137994); the preceding [prepublication audit](https://github.com/guillermomolina/protos/issues/1151#issuecomment-7094113409) explicitly reserved publication pending this decision.

The maintainer executed the existing `dist/publish_release.py` with that authorization and supplied `DIST005_RELEASE_PUBLICATION: PASS`, `PUBLIC_TAG_IDENTITY=PASS`, and exact release candidate/version/tag lines. The terminal excerpt was truncated after those lines; this record does **not** claim the remaining individual publisher PASS markers appeared in that excerpt. Public tag, release metadata and all five published-asset digests were **independently verified** through the GitHub API and recorded in [DIST025's public verification comment](https://github.com/guillermomolina/protos/issues/1151#issuecomment-7094211152).

## Independently verified five-asset public envelope

All five GitHub Release assets had `state=uploaded`. Their GitHub API `digest` metadata matched the reviewed release envelope:

| Downloadable asset | GitHub-reported SHA-256 |
| --- | --- |
| `protos-0.4.192-native-linux-x86_64.zip` | `73c16888dd81ed9fd458c56fb0b04f849b7ae81aa24880ea1d54a56da88c897b` |
| `protos-0.4.192-native-linux-x86_64.zip.sha256` | `be34a4fbb4d688cdce92f1ec9abf7a04a48937fd2e5dc60cf6f3bb7fc05b42ff` |
| `protos-0.4.192-posix-jvm.zip` | `4b77e55d036448bcbb096083a0308ebadc09f86c48f4b7b8bdb958ce6f9247bc` |
| `protos-0.4.192-posix-jvm.zip.sha256` | `a349bb8a37b14a663b47931dc2517ca6dd6164e95b91884f1192694a9b8cc91c` |
| `RELEASE_MANIFEST.txt` | `63ae6ea92bd35ffc3ecd628158d0a11112388010d2c7c8bde4bc155af4e4a1b5` |

`RELEASE_NOTES.md` is the exact public Release body, not a sixth downloadable asset. GitHub's published notes identify the candidate/baseline/specification consistently with the approved manifest and contain no stray character in the last limitation.

The independent verification here consists of GitHub tag, commit, release and asset **metadata**; it does **not** mean that an independent verifier downloaded, re-hashed or re-executed the published ZIPs. The maintainer separately reports local `git diff --check` clean and all local tests PASS; no newly identified exact test-command transcript or test counts are inferred from that statement. The release-specific end-to-end extracted-distribution validation was reported through the official preparation PASS gate.

## Public runtime and limitations

The recommended first-run artifact is Native Image for Linux x86-64, dynamically linked, with declared minimum glibc `2.39` (while observed maximum symbol requirement is `2.34`), GraalVM `25.5.5.0.3` and no external Java dependency. The portable POSIX/JVM archive is the compatibility fallback requiring the supported GraalVM Community JDK 25 / Truffle `25.5.5.0.3` optimizing runtime.

The selected implementation baseline includes Protos-first persistent SQLite databases with manual transactions and recovery examples, host-authorized lazy loading of native FFM libraries integrated in Native Image, explicit external OS execution, Bytecode DSL / canonical collection iteration changes, Language Server navigation and Test Tool diagnostics, and a reorganized programming guide.

The public notes explicitly limit the release to experimental Core 0.1 semantics without stable compatibility guarantees. Native FFM accepts only a finite registered set of physical signatures; SQLite and other third-party C libraries are not bundled. Native libraries execute as trusted in-process code, not within Protos Filesystem confinement. Portable JVM FFM still requires a separate provider JAR. External process execution in Native Image is not yet verified as a supported capability.

## Durable record coordinates

The publishing commit in `guillermomolina/protos-project-docs` is the authority for this document's own Git revision; avoid embedding a self-referential SHA inside the evidence file. After the index is updated, refer to the exact published documentation revision from [DIST025 #1151](https://github.com/guillermomolina/protos/issues/1151).

```text
WORK_ITEM=DIST025
PROTOS_BASELINE_REVISION=9c56aba2c8b84a9c44440cb361db85b53fcf30b4
PROTOS_REVISION=45801061a2878256969901367057cb1be0d23971
RELEASE_VERSION=0.4.192
RELEASE_TAG=v0.4.192
SPECIFICATION_REVISION=0.1.507
RELEASE_MANIFEST_SHA256=63ae6ea92bd35ffc3ecd628158d0a11112388010d2c7c8bde4bc155af4e4a1b5
OWNER_EXACT_PUBLICATION_AUTHORIZATION=PASS
DIST005_RELEASE_PREPARATION=PASS_REPORTED
MULTI_ASSET_ENVELOPE_VERIFICATION=PASS_REPORTED
DIST005_RELEASE_PUBLICATION=PASS_REPORTED
PUBLIC_TAG_AND_CANDIDATE_IDENTITY=PASS_GITHUB_API
PUBLIC_RELEASE_CHANNEL=PASS_GITHUB_API
PUBLIC_ASSET_SET_AND_DIGEST_METADATA=PASS_GITHUB_API
PUBLIC_RELEASE_ID=455096529
PUBLIC_RELEASE_PUBLISHED_AT_UTC=2026-11-23T07:18:07Z
PROJECT_RECORD_REVISION=SEE_PUBLISHING_COMMIT_AND_LIVE_ISSUE
```

## Sources

- [Public release and five downloadable files](https://github.com/guillermomolina/protos/releases/tag/v0.4.192).
- [DIST025 tracker and publication audit](https://github.com/guillermomolina/protos/issues/1151).
- [Exact owner authorization](https://github.com/guillermomolina/protos/issues/1151#issuecomment-7094137994).
- [Independent GitHub verification](https://github.com/guillermomolina/protos/issues/1151#issuecomment-7094211152).
- Project release policy in `guillermomolina/protos` `AGENTS.work/RELEASE.md`, `dist/prepare_release.py` and `dist/publish_release.py`.
