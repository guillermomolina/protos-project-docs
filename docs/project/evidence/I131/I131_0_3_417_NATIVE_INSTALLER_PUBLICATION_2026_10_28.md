# I131 — version-agnostic Linux Native Image installer publication (2026-10-28)

Status: **published implementation and maintainer-reported validation evidence**; not a new binary release or a clean-host certification.

- Owning implementation: [I131 / #982](https://github.com/guillermomolina/protos/issues/982).
- Parent distribution work: [DIST018 / #981](https://github.com/guillermomolina/protos/issues/981).
- Approved design direction: [DIST018 investigation](../DIST018/DIST018_NATIVE_INSTALLER_APPROVED_INVESTIGATION_2026_10_28.md).
- Exact product commit: [ce6cdec09a615a31edf668e22bd022be6cac2626](https://github.com/guillermomolina/protos/commit/ce6cdec09a615a31edf668e22bd022be6cac2626).

## Exact product publication

```ini
PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_PREVIOUS_REVISION=07ca92be06c7139cfff4b35c47b2452e2ff50e28
PROTOS_REVISION=ce6cdec09a615a31edf668e22bd022be6cac2626
IMPLEMENTATION_VERSION=0.3.417-SNAPSHOT
IMPLEMENTATION_ISSUE=I131
PARENT_ISSUE=DIST018
PRODUCT_COMMIT=I131: add version-agnostic Linux Native installer
PRODUCT_CHANGED_FILES=8
PRODUCT_INSERTIONS=3757
PRODUCT_DELETIONS=157
RELEASE_CREATED=NO
PRODUCT_PUBLICATION=PASS
HUMAN_REPORTED_GIT_DIFF_CHECK=CLEAN
HUMAN_REPORTED_MAKE_TEST=PASS
HUMAN_REPORTED_MAKE_CHECK=PASS
HUMAN_REPORTED_TESTS_AFTER_VERSION_BUMP=NO
```

The exact published delta is `dist/install.sh` (executable),
`dist/tests/release/test_install_release_selection.py`,
`dist/tests/release/test_install_release_fallback.py`, `README.md`,
`dist/README.md`, `docs/guide/00-try-protos.md`, `pom.xml`, and
`CHANGELOG.md`. No normative specification or Protos runtime semantics
changed in this item.

## Implemented behavior

- A stable inspectable POSIX shell entry point at
  `https://raw.githubusercontent.com/guillermomolina/protos/main/dist/install.sh`
  discovers **published** numeric GitHub Release tags and selects the newest
  compatible Native artifact, including prereleases, without a hard-coded
  Protos release version. An explicit `--tag` is strict.
- A structural awk release-JSON reader and manifest checks reject malformed
  responses, drafts, missing or incompatible Native assets and missing
  manifest data; no `jq` or Python dependency is imposed on target hosts.
- Native archive identity, checksums, source revision, platform/ABI/ISA
  requirements and ZIP paths/layout are checked before activation. There is
  no portable JVM/JAR fallback or installation of Java, GraalVM, Maven,
  a Native Image compiler, glibc or host packages.
- An intact Native distribution is placed under
  `$HOME/.local/share/protos/versions/`; a managed
  `$HOME/.local/bin/protos` wrapper invokes the real versioned launcher
  path, preserving `PROTOS_HOME`, arguments and caller CWD. Staged smoke
  checks precede atomic activation. Previous versions remain available
  for offline rollback; uninstall deactivates the managed wrapper only.
- User documentation presents the stable one-command installer and the
  inspect-before-run, pinned-version, rollback and uninstall alternatives.

## Human-executed verification

On 2026-10-28 Guillermo reported:

1. `sh -n dist/install.sh` PASS and **44 focused Python installer tests PASS**
   (`python3 -m unittest dist.tests.release.test_install_release_selection`
   `dist.tests.release.test_install_release_fallback -v`; 44 in 3.665 seconds).
   Fixtures cover release selection, incompatible fallback, manifest parsing,
   checksum and layout corruption, traversal/JAR rejection, missing tools,
   idempotent install, upgrades, foreign wrappers, rollback and uninstall.
2. Real official `v0.3.398` Native release inspection and download
   verification PASS: `NATIVE_CHECKSUM_VERIFIED=PASS`,
   `NATIVE_ARCHIVE_VERIFIED=PASS`, `NATIVE_LAYOUT_VERIFIED=PASS`,
   `INSPECTION=PASS`, on Linux glibc 2.44 (release min glibc 2.39).
   Published Native SHA-256 observed:
   `f6851e379b2e5217e2717988d4846c3f173da2bc557ccd869f450c1fcf40f95d`.
3. Real Native installation using the installer bytes from the checkout
   with an isolated temporary `HOME` PASS:
   `INSTALL_STATUS=INSTALLED`, `ACTIVE_VERSION=v0.3.398`,
   `protos --version` → `Protos 0.3.398`,
   `protos -e 'print("Hello, Protos!")'` → `Hello, Protos!`,
   and `protos hello.protos` from unrelated CWD → `Unrelated CWD: OK`.
   Separate manual offline uninstall and rollback-to-retained-version checks
   also PASS. Truffle's interpreter-only fallback warning is an existing
   Native runtime warning, not an installer failure.
4. After the final installer and documentation candidate, the maintainer
   explicitly reported **`I131_FULL_VALIDATION=PASS`**:
   `make test` PASS, `make check` PASS and `git diff --check` clean.
   The `pom.xml` / `CHANGELOG.md` version bump to `0.3.417-SNAPSHOT`
   occurred **after** those tests. The maintainer reported no tests
   executed after the version bump.
5. The owner executed the exact product `git commit` and `git push`
   to `main` and supplied `I131_PUBLICATION=PASS` plus the full
   revision above; GitHub independently exposes that commit and its
   eight-file delta.

These are maintainer-reported local commands and results, corroborated
for publication identity by GitHub. They are **not** claims that the agent
ran the tests, that original logs were archived, or that a new release
asset was built or published.

## Remaining acceptance limits

- The host smoke used an isolated user home, but the evidence does **not**
  establish a completely clean Linux host/container without Java/GraalVM/Maven.
- Same-byte selection of a newer compatible release was covered by
  synthetic published-release fixtures; a later **real** published
  release has not yet been observed through an end-to-end update.
- Post-publication `curl | sh` execution against the public stable URL,
  an independent clean-host missing-dependency matrix and the parent
  DIST018 custom-prefix expectation have not been separately demonstrated.
- The currently observed binary release remains `v0.3.398`; publishing
  implementation `0.3.417-SNAPSHOT` does not publish another Native ZIP.

The [owning I131 Issue](https://github.com/guillermomolina/protos/issues/982)
and [DIST018 parent](https://github.com/guillermomolina/protos/issues/981)
retain live acceptance and closure authority. This revision-pinned record
captures the implemented product and validation; it does not itself
declare unobserved acceptance tests completed.
