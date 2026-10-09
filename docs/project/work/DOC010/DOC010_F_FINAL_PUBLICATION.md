# DOC010-F — Final news integration and documentation closure

Date: 2026-10-09
Owner: https://github.com/guillermomolina/protos/issues/848

## Published source evidence

- Canonical Protos release news: `docs/news/2026-10-09-protos-0-3-312.md`, product commit `e99b712ecf658a527b1fe82c8beba2c1286d8dc9`, https://github.com/guillermomolina/protos/commit/e99b712ecf658a527b1fe82c8beba2c1286d8dc9
- News indexed by canonical `docs/news/README.md` in that commit.
- Website publication: `c4c7a605a3df3e046d30ad0ae5023c3ab846701f`, https://github.com/guillermomolina/protos-website/commit/c4c7a605a3df3e046d30ad0ae5023c3ab846701f.
- Website commit changes only `protos-source.lock.json` to the exact news revision `e99b712ecf658a527b1fe82c8beba2c1286d8dc9`.
- Website release lock remains `v0.3.312`, with Native `protos-0.3.312-native-linux-x86_64.zip` and Portable `protos-0.3.312-posix-jvm.zip`.
- Prior accepted stages: DOC010-B `d6efe02522f20015a4b9261bbe1bf83867bff749`, C `f8c4a0a6c706e5603b3948c2b99c7fb4fd643660`, D `751acb16a16318b7b8d1cec6d00338b9da3338ea`, E website `a93e4774c0f03c4d7d302dc9d12fb844946195fd`.

## Human-executor acceptance

Owner explicitly reports website commit pushed, `git diff --check` clean and all local tests PASS. Do not repeat them for the unchanged source. This verifies source publication and user-reported validation, not a separately observed production deployment.

## Final disposition

DOC010's canonical guides, news, and website source/release locks are reconciled for public prerelease `v0.3.312`. Parent issue #848 can be closed as completed for source publication. Live production deployment remains outside the evidenced scope; no live-site deployment claim is made.

Future Native Test Tool progress visibility and failure-on-discovery hangs are independent reliability work, not DOC010 reopeners. No administrative follow-on slices required.
