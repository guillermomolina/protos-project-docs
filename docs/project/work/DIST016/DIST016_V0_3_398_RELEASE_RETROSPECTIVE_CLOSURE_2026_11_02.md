# DIST016 — Retrospective closure of the v0.3.398 release-readiness work

**Formal owner:** [DIST016 #951](https://github.com/guillermomolina/protos/issues/951).
**Reconciliation date:** 2026-11-02.
**Classification:** completion of an old public distribution work item. This record does not create another release, retroactively amend release artifacts, or imply that newer development HEAD was included in the 2026-10-22 release.
**Public release:** [Protos v0.3.398](https://github.com/guillermomolina/protos/releases/tag/v0.3.398), published 2026-10-22 13:58:16 UTC; GitHub release ID `420792916`, `prerelease=true`, `draft=false`.
**Exact candidate SHA:** [`721b6d927f53b1d6c985cfd78a695984e5245b77`](https://github.com/guillermomolina/protos/commit/721b6d927f53b1d6c985cfd78a695984e5245b77).
**Selected development baseline:** [`0c73618170f46aabd4da8bdaf7f2e07d5c253189`](https://github.com/guillermomolina/protos/commit/0c73618170f46aabd4da8bdaf7f2e07d5c253189), `0.3.398-SNAPSHOT`; released candidate version `0.3.398`, specification `0.1.468`.

## Recovered historical acceptance and validation

The live [DIST016 #951 discussion](https://github.com/guillermomolina/protos/issues/951) contains a complete published release transaction, although its Issue state remained open:

1. The separately tracked [DOC011 #952](https://github.com/guillermomolina/protos/issues/952) documentation reconciliation was committed as [`1111ccaf83ced008a273918af94cd11627d44f6d`](https://github.com/guillermomolina/protos/commit/1111ccaf83ced008a273918af94cd11627d44f6d); its own revision-bound documentation/validation record is already published. The maintainer reported final `make test` PASS, 121/121 Markdown links and source-style guard PASS.
2. The bounded CI container Git trust repair was published at [`9c8aef61db617eb38017870648b75ce2eb1d95d2`](https://github.com/guillermomolina/protos/commit/9c8aef61db617eb38017870648b75ce2eb1d95d2). A later explicit owner correction removed any dependency on GitHub Actions as the release-preparation test oracle.
3. Local-only release-harness preparation fixes were published as [`0c73618170f46aabd4da8bdaf7f2e07d5c253189`](https://github.com/guillermomolina/protos/commit/0c73618170f46aabd4da8bdaf7f2e07d5c253189). The maintainer reported local test-suite success and release-harness focal verification.
4. The owner selected the precise development baseline and separately authorized publication of the exact `v0.3.398` candidate, as recorded in the Issue history, including the approved manifest hash and artifact identities.
5. The maintainer's `dist/prepare_release.py` result reports `DIST005_RELEASE_PREPARATION: PASS`, real extracted Linux Native and portable JVM admission PASS, and **2,802 Protos tests passed, zero failed** in each extracted distribution. The release envelope and manifest were checked, and `dist/publish_release.py` reported `DIST005_RELEASE_PUBLICATION: PASS`. These executions are **human-reported** and have not been rerun during retrospective reconciliation.

## Immutable public-release verification on 2026-11-02

An independent read through the GitHub releases API for `v0.3.398` confirms the release's timestamp, non-draft prerelease status, and exactly **five assets**. The GitHub public release asset digest fields agree with the hashes recorded in DIST016's final 2026-10-22 publication comment:

| Asset | Public SHA-256 |
| --- | --- |
| `protos-0.3.398-native-linux-x86_64.zip` | `f6851e379b2e5217e2717988d4846c3f173da2bc557ccd869f450c1fcf40f95d` |
| `protos-0.3.398-native-linux-x86_64.zip.sha256` | `3049c65d7abd70ed798b81b00a15a7bf182b8ce9b72e96ada5d8444857c11b49` |
| `protos-0.3.398-posix-jvm.zip` | `8b54a8d76c52be4f3e6e3f53a6e140df3aed1a625d913cfbb8fa0298976107eb` |
| `protos-0.3.398-posix-jvm.zip.sha256` | `744b5098b7f771f6c97f44981a0a8ffd4bada018eca7d69403ec0e6345b92636` |
| `RELEASE_MANIFEST.txt` | `b1aa9d5b4c3a92b27ec013419a4688c65de8fef81927341f1f1a19570297c06c` |

A separate GitHub Git-refs read confirms that the public lightweight `v0.3.398` tag references the exact candidate commit `721b6d927f53b1d6c985cfd78a695984e5245b77`. These are direct **remote metadata and digest-field checks**, not a new download or local rehash of the binary assets.

## Website follow-up reconciliation

The old DIST016 final comment explicitly reserved website download/source-lock work until after successful public release. By this retrospective checkpoint the [website's current `protos-release.lock.json`](https://github.com/guillermomolina/protos-website/blob/main/protos-release.lock.json) no longer pins `v0.3.312`: it declares `channel: latest-published-prerelease`. The current [website homepage](https://github.com/guillermomolina/protos-website/blob/main/src/pages/index.astro) imports the release-onboarding helpers and links to GitHub Releases, while the current website source lock points at a later Protos source revision. Thus the historical requirement **to stop advertising the old fixed 0.3.312 release** is satisfied by the newer, version-independent website implementation. This read-back does not assert which particular deployment first supplied v0.3.398 links, or independently test a running production server.

## Closure scope and provenance

The release already happened on 2026-10-22, and its required immutable GitHub identities and public assets are verified. Closing [DIST016 #951](https://github.com/guillermomolina/protos/issues/951) on 2026-11-02 is **administrative reconciliation**, not a new release, approval, tag, build or validation result. Any future release preparation belongs to the current release owner and current HEAD, not this frozen historical `v0.3.398` item.

**PUBLIC_RELEASE_TAG=`v0.3.398`**  
**RELEASE_CANDIDATE_REVISION=`721b6d927f53b1d6c985cfd78a695984e5245b77`**  
**DEVELOPMENT_BASELINE_REVISION=`0c73618170f46aabd4da8bdaf7f2e07d5c253189`**  
**RELEASE_ASSET_SET=VERIFIED (5)**  
**HUMAN_RELEASE_VALIDATION=REPORTED_PASS**  
**REQUIRED_DURABLE_PUBLICATION=PASS, after this record's own GitHub publication and exact-revision read-back.**
