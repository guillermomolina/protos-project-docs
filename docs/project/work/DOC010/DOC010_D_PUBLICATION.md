# DOC010-D — v0.3.312 Getting Started, distribution and authority documentation

Recorded: 2026-10-08
Issue: https://github.com/guillermomolina/protos/issues/848
Product commit: https://github.com/guillermomolina/protos/commit/751acb16a16318b7b8d1cec6d00338b9da3338ea

## Verified GitHub publication

Commit `751acb16a16318b7b8d1cec6d00338b9da3338ea` is published on `guillermomolina/protos` with message `DOC010-D: Getting Started, distribution and authority documentation for v0.3.312 (#848)`.

Changed files (GitHub commit evidence):
- `README.md`
- `dist/README.md`
- `docs/guide/00-try-protos.md`
- `docs/guide/11-process-io-filesystems-and-authority.md`
- `docs/guide/README.md`

## Human executor acceptance

Owner explicitly reports `git diff --check` clean and all local tests passing. Exact test command logs not included; do not imply separately reexecuted tests.

## Release and remaining work

Latest verified public release: `v0.3.312`, release commit `f8ff34498f3a193c9bfe15f215a181c23504a6ad`, baseline `5c4e2ea8a2fff7b2750e4182de41db8754ce4286`, spec 0.1.451.

DOC010-B, C, D documentation edits are published on `protos/main`. Website currently pins Protos source `4a6df50e8577ebb0f3fc352df13dcbab368f955a` and public release `v0.3.0` (observed 2026-10-08); do not claim website updated.

Next DOC010-E: grouped website implementation in `guillermomolina/protos-website`: update exact source lock to a reviewed product SHA including DOC010-D, update release lock to `v0.3.312` with actual asset names, reconcile sidebar/navigation and landing/download presentation, build and validate. Website source generated from lock must not be maintained as a fork. DOC010-F news source must be committed to canonical `guillermomolina/protos/docs/news` and brought into website via fresh exact lock; avoid a stale publication or silently claiming the draft ZIP is published. Consider grouping DOC010-E/F if website build can consume the canonical news after product publication, respecting human execution boundaries.

No new administrative Issues; DOC010 #848 stays open through website/news validation and publication.
