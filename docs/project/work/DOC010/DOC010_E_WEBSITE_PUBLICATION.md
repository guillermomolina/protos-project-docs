# DOC010-E — Website synchronized to Protos v0.3.312

Date: 2026-10-09
Issue: https://github.com/guillermomolina/protos/issues/848

## Verified publication

Website commit: `a93e4774c0f03c4d7d302dc9d12fb844946195fd`
https://github.com/guillermomolina/protos-website/commit/a93e4774c0f03c4d7d302dc9d12fb844946195fd

Commit title: `DOC010-E: synchronize website with Protos v0.3.312 and DOC010 guides`.

Nine changed paths in GitHub:
- `.gitignore`
- `protos-release.lock.json`
- `protos-source.lock.json`
- `scripts/check-protos-release.mjs`
- `scripts/materialize-protos-guide.mjs`
- `src/content/docs/learn/getting-started/index.md`
- `src/content/docs/learn/guide/index.md`
- `src/pages/index.astro`
- `src/styles/home.css`

Verified lock contents on website main:
- Canonical Protos source revision `751acb16a16318b7b8d1cec6d00338b9da3338ea` (DOC010-D, includes DOC010-B/C).
- Public release `v0.3.312`, URL https://github.com/guillermomolina/protos/releases/tag/v0.3.312.
- Native asset `protos-0.3.312-native-linux-x86_64.zip` and JVM asset `protos-0.3.312-posix-jvm.zip`.

## Human-executor result

Owner reports pushed, `git diff --check` clean, and all local tests PASS. No tests/builds repeated by the coordinating agent. Website production deployment status has **not** been verified; pushing website source does not by itself prove deployment.

## Next: DOC010-F

The news article for v0.3.312 is not yet in canonical `guillermomolina/protos/docs/news` as of this record. Publish one dated news article and index entry in **product** `docs/news`, then advance the website exact source lock and validate generated news rendering. Do not create a manually maintained website duplicate. Keep this as one coherent handoff and avoid further gratuitous slices. The previous article draft delivered by ZIP is not itself published source.

Website release lock is already correct: do not change it again without a factual correction. Parent #848 stays open until news and closure evidence.
