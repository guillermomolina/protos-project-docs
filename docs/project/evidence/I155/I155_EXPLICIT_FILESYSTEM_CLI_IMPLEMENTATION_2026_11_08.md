# I155 — Explicit confined Filesystem CLI (2026-11-08)

Issue: [I155 #1079](https://github.com/guillermomolina/protos/issues/1079). Approved design: [D248-B #1074](https://github.com/guillermomolina/protos/issues/1074).

The standalone CLI now offers `--allow-filesystem=<directory>` for SOURCE, `-e`, REPL, `run`, `debug` and service entry, preserving default-deny authority. The directory must already exist, and the grant is available as one RootActor bootstrap-local `filesystem` binding. No automatic grant through imports, Tools, other Actors or new Processes is introduced. Changes include `ProtosCli.java`, `ProtosStandaloneHostedExecution.java`, `ProtosI155CliFilesystemGrantTest.java`, CLI/authority guides, Native smoke tooling and the new `filesystem/cli-confined-data.protos` example.

## Revision and maintainer-reported validation

- Product revision: [`b27a925367047be04c7a272501624ff4ded462c0`](https://github.com/guillermomolina/protos/commit/b27a925367047be04c7a272501624ff4ded462c0), published to `main` on 2026-11-08.
- Version: `0.3.455-SNAPSHOT` (version bump after green tests; no tests after bump).
- Product commit: 85 files, +14,027 / -640 lines; final working tree clean.
- Final `make test`: PASS (`FINAL_MAKE_TEST_EXIT=0`); final `make check`: PASS (`FINAL_CHECK_EXIT=0`); `git diff HEAD --check`: PASS.
- `make test` runs Java and Protos tests, **not** `make test-native`. No rebuilt Native Image or installed Native executable test result was provided for this product revision. The implementation contains Native smoke harnesses, but their execution is **not claimed**.

These are exact-product-revision implementation evidence and owner-reported validation, not a new design approval or a public release certification.

Concurrent regression selection across Filesystem, FFM, host and Actor bootstrap: 134 tests, zero failures/errors, two environmental skips. I155 shares implementation dependencies with [I156 #1083](https://github.com/guillermomolina/protos/issues/1083) and [I158 #1091](https://github.com/guillermomolina/protos/issues/1091). Future cross-Process provisioning remains the separate [D250 #1080](https://github.com/guillermomolina/protos/issues/1080); nothing here selects its contract.

`PROTOS_REVISION=b27a925367047be04c7a272501624ff4ded462c0`
