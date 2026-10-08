# DOC010-C — Standard Library and developer tooling guides publication

Date: 2026-10-08
Owning issue: https://github.com/guillermomolina/protos/issues/848

## Verified GitHub evidence

- Published product revision: `f8c4a0a6c706e5603b3948c2b99c7fb4fd643660`.
- Commit: https://github.com/guillermomolina/protos/commit/f8c4a0a6c706e5603b3948c2b99c7fb4fd643660
- Title: `DOC010-C: Standard Library and developer tooling guides (#848)`.
- GitHub verified changed paths (13):
  - `docs/guide/README.md`
  - `docs/guide/library/README.md`
  - `docs/guide/library/datetime.md`
  - `docs/guide/library/interop.md`
  - `docs/guide/library/logging.md`
  - `docs/guide/library/regex.md`
  - `docs/guide/library/semver.md`
  - `docs/guide/library/text.md`
  - `docs/guide/tools/README.md`
  - `docs/guide/tools/cli.md`
  - `docs/guide/tools/language-server.md`
  - `docs/guide/tools/package-tool.md`
  - `docs/guide/tools/test-tool.md`

## Human-executor acceptance

- Owner reports `git diff --check` clean.
- Owner reports all tests PASS locally.
- Exact test commands/logs were not supplied; these are user-confirmed results, not independently rerun tests.
- No need to repeat green tests for unchanged documentary files.

## Release and next steps

- Reference public prerelease: `v0.3.312`, release-only commit `f8ff34498f3a193c9bfe15f215a181c23504a6ad`, specification `0.1.451`.
- Product `main` may include subsequent accepted documentation edits; distinguish current HEAD from published release baseline.
- DOC010-D next grouped implementation in `guillermomolina/protos`: README, Try Protos/Getting Started, current downloads, JVM/Native runtime and support matrix, authority / filesystem / network how-to, navigation. Reuse already published DOC010-B/C guides, not duplicate them.
- DOC010-E later reconciles website source and release locks; DOC010-F completes accurate news source/publication. Keep #848 open.
- No release workflow restaging and no new administrative issues.
