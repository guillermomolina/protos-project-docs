# DIST021 — Protos v0.3.442 JVM + Native prerelease publication

**Outcome:** PUBLICATION VERIFIED / COMPLETE — 2026-11-03  
**Issue:** [guillermomolina/protos#1044](https://github.com/guillermomolina/protos/issues/1044)  
**Public release:** [Protos 0.3.442](https://github.com/guillermomolina/protos/releases/tag/v0.3.442)

This is a revision-bound historical release-execution record. It records the human executor's reported gates and the independent GitHub API checks; it is not a new language decision or a fresh test run.

## Authorized candidate and exact source lineage

| Property | Verified value |
| --- | --- |
| Repository | `guillermomolina/protos` |
| Development baseline | `b708e1ed981b2ba151a00210622b486a22a7a908` |
| Development version | `0.3.442-SNAPSHOT` |
| Candidate and public tag target | `b20271369d00f8937890a4a94048bb1754b688b8` |
| Candidate commit message | `release: materialize Protos 0.3.442 candidate` |
| Candidate's only parent | `b708e1ed981b2ba151a00210622b486a22a7a908` |
| Candidate-only diff | `pom.xml` project version `0.3.442-SNAPSHOT` → `0.3.442` |
| Tag | `refs/tags/v0.3.442` (lightweight, targeting candidate) |
| GitHub Release title | `Protos 0.3.442` |
| Channel | `prerelease=true`, `draft=false` |
| Published at (GitHub API) | `2026-11-03T05:09:25Z` |
| Specification revision | `0.1.492` |
| Distribution model | `JVM_PLUS_NATIVE` |
| Release-envelope format | `protos-public-prerelease-envelope-v2` |
| Authorized `RELEASE_MANIFEST.txt` SHA-256 | `a294d13b49afb49a610545657414c96cbf628965b9af68ca9fb0b06d7fced9e1` |
| `RELEASE_NOTES.md` SHA-256 from release envelope | `dbe4d5e5f1e3af4b3f3fb0f548881db80e53468e4c2223856531855cf56adc82` |

The maintainer explicitly authorized the **exact candidate SHA and release-manifest digest**, separately from preparation, before public publication. They executed the existing `dist/prepare_release.py` and `dist/publish_release.py`; no manual source change, additional release tooling or commit to development `main` was required. The GitHub API verified that product `main` remained at the selected baseline after publication.

## Exactly five public release assets

The GitHub Release assets endpoint independently reported **exactly** the following five assets, all with `state=uploaded`. Their GitHub `assets[].digest` values match the independently supplied and audited preparation checksums.

| Asset | SHA-256 |
| --- | --- |
| `protos-0.3.442-native-linux-x86_64.zip` | `e3aea0bc715a0351cf09d5fe3a52471d444ab17b80924cc7630bf96f2a289380` |
| `protos-0.3.442-native-linux-x86_64.zip.sha256` | `cfd1f83332ee2d614b7bbe9b37ad5e7b4ce2fc7b8723381f93ecd495593c34b6` |
| `protos-0.3.442-posix-jvm.zip` | `514bd67dcc7609db69174a8dac44789cd050884fb2e49318b0fe44652921cd05` |
| `protos-0.3.442-posix-jvm.zip.sha256` | `835f903e2f97e066798db057c5f2fe58e17dfc45bb069b059e202f10baeb3c2e` |
| `RELEASE_MANIFEST.txt` | `a294d13b49afb49a610545657414c96cbf628965b9af68ca9fb0b06d7fced9e1` |

The published GitHub Release body matches the expected release identity and claims. The maintained publisher reported that the published asset digests and authorized manifest had been verified. The independent post-publication API inspection confirms the public tag target, single-parent release-only diff, release metadata, exact asset inventory and digests; it does not claim another local build or test.

## Validation and release gates

The human executor reported a successful selected-baseline preflight and completed preparation. The existing preparator performed the Native Image and portable JVM builds; both complete distribution admissions, including extracted-bundle/smoke and associated tests; and multi-asset metadata envelope verification. The preparation log contained `DIST005_RELEASE_PREPARATION: PASS`.

The human executor subsequently reported the following after the separately approved publication invocation:

```text
DIST021_PUBLICATION_PREFLIGHT=PASS
DIST005_RELEASE_PUBLICATION: PASS
RELEASE_CANDIDATE_SOURCE_REVISION=b20271369d00f8937890a4a94048bb1754b688b8
RELEASE_VERSION=0.3.442
RELEASE_TAG=v0.3.442
PUBLIC_TAG_IDENTITY=PASS
GITHUB_PRERELEASE_METADATA=PASS
PUBLISHED_ASSET_SET=PASS
PUBLISHED_ASSET_DIGESTS=PASS
RELEASE_MANIFEST_IDENTITY=PASS
RELEASE_PUBLICATION_AUTHORIZED=YES
TAG_CREATED_OR_REUSED=YES
PUBLIC_RELEASE_CREATED_OR_REUSED=YES
```

No duplicate integrated product-suite run was requested merely because the detached release candidate changed its `pom.xml` version. Previously validated results for the exact development baseline were reused where applicable; the mandatory distribution-specific gates were still executed.

## Runtime support, capabilities and disclosed limitations

The release notes announce contextual generic/numeric `for` using `each`/`eachStep`, Core `Range`/`InvertedRange`, contextual non-local `return`, Integer bitwise/shift operations, expanded Array `flatMap`, `zip`, `partition`, `distinct`, D237 named and mixed call arguments, and the BUG033 formatter / BUG034 LSP Signature Help correctness repairs.

- **Native:** recommended first-run Linux x86_64 Native Image; dynamically linked against glibc, policy minimum ABI 2.39; GraalVM Native Image 25.4.4.1.1; no external Java runtime required. Under ratified PLAT045 it runs the interpreter-only Truffle fallback; guest JIT is unavailable due to `oracle/graal#14579`.
- **Portable JVM:** POSIX compatibility fallback, with an externally installed GraalVM Community 25.4.4.1.1 for JDK 25.0.4.1.1 and matching Truffle 25.4.4.1.1; supported optimizing runtime `HotSpotTruffleRuntime`. The archive does not bundle the JDK.
- **Disclosures:** Core v0.1 is still a draft with no stable compatibility promise. BUG025's real DAP non-local-return exception-classification issue remains open. This release does not certify a matching installed VSIX or GraalVM/Truffle 25.5 support.

These claims are drawn from the exact release notes and metadata, not extrapolated into unverified platform support.

## Independent public verification and closure provenance

GitHub REST inspection after the owner-reported publication established the exact lightweight public tag, commit and parent, `pom.xml`-only diff, the prerelease title/channel/body, five uploaded asset names and `assets[].digest` values, and unchanged development `main`.

- [Immutable release-tag target](https://github.com/guillermomolina/protos/commit/b20271369d00f8937890a4a94048bb1754b688b8)
- [Selected development baseline](https://github.com/guillermomolina/protos/commit/b708e1ed981b2ba151a00210622b486a22a7a908)
- [Public release](https://github.com/guillermomolina/protos/releases/tag/v0.3.442)
- [DIST021 issue and public-verification comment](https://github.com/guillermomolina/protos/issues/1044#issuecomment-6596040278)

This record is published in the canonical historical-evidence path `docs/project/evidence/DIST021/`. Its exact project-record commit is established by Git history and must be recorded separately in the final issue closure comment, together with `PROTOS_REVISION` and `CROSS_REFERENCES=PASS`.
