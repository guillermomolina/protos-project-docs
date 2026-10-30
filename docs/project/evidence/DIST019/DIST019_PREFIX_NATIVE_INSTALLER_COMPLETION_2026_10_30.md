# DIST019 — Prefix-aware Linux Native installer, version removal and offline SNAPSHOT completion

**Date:** 2026-10-30. **Record role:** revision-pinned, non-normative product-publication and maintainer-validation evidence for [DIST019 #998](https://github.com/guillermomolina/protos/issues/998). This is neither a new public binary release nor a Linux platform certification.

## Exact publication identity

```ini
WORK_ITEM=DIST019
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=3e95b5f3699a3fa74b969304b85d3b8c14d4c9d8
PRODUCT_CHANGED_FILES=7
PRODUCT_PUBLICATION=VERIFIED_GITHUB
VALIDATION_PROVENANCE=MAINTAINER_REPORTED_LOCAL
FOCUSED_INSTALLER_TESTS=100_PASS
HUMAN_REPORTED_GIT_DIFF_CHECK=CLEAN
HUMAN_REPORTED_MAKE_TEST=PASS
HUMAN_REPORTED_MAKE_CHECK=PASS
NORMATIVE_SPECIFICATION_CHANGE=NO
NATIVE_RUNTIME_CHANGE=NO
NEW_GITHUB_RELEASE=NO
```

The exact [published product commit](https://github.com/guillermomolina/protos/commit/3e95b5f3699a3fa74b969304b85d3b8c14d4c9d8) is titled `DIST019: Prefix-aware Native installer, root layout, version pruning and local SNAPSHOTs`. GitHub confirms this seven-path delta:

- `dist/install.sh`
- `dist/tests/release/test_install_release_selection.py`
- `dist/tests/release/test_install_release_fallback.py`
- `dist/tests/release/test_install_local_snapshot.py`
- `README.md`
- `docs/guide/00-try-protos.md`
- `dist/README.md`

There is no change to `pom.xml`, `CHANGELOG.md`, `bin/protos`, `dist/build_native.py`, language specifications, the Java/Truffle runtime, or release publication machinery in this product commit. The existing published Native-image ZIP and portable JVM release models remain separate. No new release/tag is implied.

## Published installer contract

- **Prefix and layout:** ordinary users default to `$HOME/.local`; effective UID 0 defaults to `/usr/local`; `--prefix PATH` selects another validated absolute path. The executable is `<prefix>/bin/protos` and retained trees are `<prefix>/share/protos/versions/<tag>`. The managed wrapper executes the real versioned launcher rather than directly symlinking it, preserving the distribution root, caller working directory and arguments. Neither automatic `sudo` nor shell-profile mutation is introduced.
- **Installed-version lifecycle:** `--list-installed` enumerates managed versions, `--rollback --tag` reactivates a retained audited distribution, `--uninstall` deactivates the managed wrapper while retaining versions, and `--remove-version --tag` removes one inactive verified managed version. There is no bulk `--prune-inactive`; it was optional under #998.
- **Offline development Native artifact:** `--inspect --local-native ZIP` and `--local-native ZIP` validate an adjacent digest, complete internal `SHA256SUMS`, archive layout, metadata, Native executable smoke, OS/architecture and glibc suitability without contacting GitHub. SNAPSHOT tags use `vMAJOR.MINOR.PATCH-SNAPSHOT`, `artifact_kind=development-native-distribution`, `public_release=false` and `implementation_version`; official tags remain independently verified against published manifests and `public_release=true`. Ambiguous source keys or conflicting channel identity are rejected.
- **Safe activation:** private staging, required executable checks, smoke before activation, no-clobber tree publication, wrapper replacement by atomic rename, version-content immutability, and retention of the former active wrapper/version on failed activation. Stale staging is reported without silently adopting or deleting it; automatic published-release selection neither downgrades nor silently replaces an active SNAPSHOT.
- **Ownership and deletion:** managed paths are rejected when symlinked, foreign-owned or writable by untrusted parties (with stricter group-write rejection for root). The installer applies explicit umask/staging/tree modes and serializes managed operations. Removal requires a full metadata and SHA-256 inventory audit, rejects the active version and foreign files, refuses a relative `PATH`, restricts unsafe filenames, and deletes inventoried regular files with `find -P ... -execdir`; directory deletion is empty-only and incomplete removal is reported.

Published user-facing behavior and precise command examples are maintained in [Try Protos at the product revision](https://github.com/guillermomolina/protos/blob/3e95b5f3699a3fa74b969304b85d3b8c14d4c9d8/docs/guide/00-try-protos.md) and the [distribution README](https://github.com/guillermomolina/protos/blob/3e95b5f3699a3fa74b969304b85d3b8c14d4c9d8/dist/README.md).

## Focused validation history and publication gate

The maintainer's incremental checks progressed through **74, 76, 81, 85 and 86 passing installer tests** while extending SNAPSHOT channel guards, offline rollback, adversarial channel transitions and version-removal safety. After the final implementation/security edits, the maintainer independently executed:

```sh
git diff --check
sh -n dist/install.sh
python3 -m py_compile \
    dist/tests/release/test_install_release_selection.py \
    dist/tests/release/test_install_release_fallback.py \
    dist/tests/release/test_install_local_snapshot.py
python3 -m unittest -q \
    dist.tests.release.test_install_release_selection \
    dist.tests.release.test_install_release_fallback \
    dist.tests.release.test_install_local_snapshot
```

The maintainer reported **100 tests in 36.541 seconds, OK**, together with clean whitespace/shell syntax and successful Python syntax compilation. The maintainer subsequently reported, for the final published DIST019 candidate, `git diff --check` clean, `make test` PASS and `make check` PASS locally, then committed and pushed the product revision above.

These are **human-reported** validation results. The evidence-recording agent has verified publication identity and changed paths through GitHub, but has not independently executed the installer suite, Maven, `make` or a privileged host installation. No CI result or archived raw test log is claimed.

## Explicit limits and prior work

- The default root path and permissions are covered by isolated fixture checks, not by a reported real root-owned installation in `/usr/local`. Do not equate this with privileged-host acceptance or certify arbitrary installations/distributions.
- The removal logic mitigates untrusted filesystem paths and simulated symlink-swap scenarios; shell/GNU-tool operations cannot guarantee safety against all malicious concurrent writers who already control the managed tree. The documented ownership/path restrictions and fail-closed checks are part of that boundary.
- The supported Native platform remains that declared by each verified release or local Native artifact. No macOS, musl, AArch64, older glibc, Java/JAR fallback or arbitrary Native builder compatibility is inferred.
- [DIST018 #981](https://github.com/guillermomolina/protos/issues/981) and [I131 #982](https://github.com/guillermomolina/protos/issues/982) established the earlier version-independent, unprivileged official Native installer; [its revision-bound evidence](../I131/I131_0_3_417_NATIVE_INSTALLER_PUBLICATION_2026_10_28.md) remains historical. DIST019 explicitly extends that earlier scope to selected prefixes, system default and offline SNAPSHOTs; it does not rewrite the earlier acceptance record.
- [Legacy issue #10](https://github.com/guillermomolina/protos/issues/10) concerns a separately scoped user-local **portable JVM** distribution helper; this Native-only work is not evidence of completion of its distinct portable-archive requirements.

## Closure authority

[DIST019 #998](https://github.com/guillermomolina/protos/issues/998) owns the live acceptance/closure state. The Issue closure comment must cite this record, the exact Protos revision and the exact resulting project-documentation commit (`PROJECT_RECORD_REVISION`). This record is a historical evidence artifact, not a substitute for a normative specification, live issue metadata or unpublished validation.
