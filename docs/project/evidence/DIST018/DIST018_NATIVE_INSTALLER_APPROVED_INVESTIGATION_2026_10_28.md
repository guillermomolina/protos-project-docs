# DIST018 — approved version-agnostic Linux Native installer investigation (2026-10-28)

Status: owner-approved implementation direction; **investigation evidence only**
Owning Issue: [DIST018 / guillermomolina/protos#981](https://github.com/guillermomolina/protos/issues/981)
Implementation handoff: [I131 / guillermomolina/protos#982](https://github.com/guillermomolina/protos/issues/982)

## Exact observation identities

```ini
OBSERVED_AT=2026-10-28
PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_MAIN_OBSERVED_REVISION=5d1a65de7fc3cf14a042b49fe1f8b0530f46c3e2
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
PROJECT_RECORD_MAIN_BEFORE_PUBLICATION=543105d532e7141cfb3065a41c85037f099743db
LATEST_OBSERVED_PUBLIC_NATIVE_RELEASE=v0.3.398
LATEST_OBSERVED_PUBLIC_RELEASE_PUBLISHED_AT=2026-10-22T13:58:16Z
LATEST_OBSERVED_NATIVE_ASSET=protos-0.3.398-native-linux-x86_64.zip
GITHUB_REPORTED_NATIVE_ASSET_SHA256=f6851e379b2e5217e2717988d4846c3f173da2bc557ccd869f450c1fcf40f95d
GITHUB_REPORTED_RELEASE_MANIFEST_SHA256=b1aa9d5b4c3a92b27ec013419a4688c65de8fef81927341f1f1a19570297c06c
INVESTIGATION_EXECUTED_COMMANDS=NONE
INSTALLER_IMPLEMENTED=NO
INSTALLER_VALIDATION=NOT_RUN
RELEASE_PUBLICATION_AUTHORIZED=NO
```

The revision and GitHub-reported digests above identify **read-only observations**, not a claim that the agent downloaded and independently rehashed the release assets or validated a new installer.

## Authority and owner direction

The owner explicitly narrowed DIST018 to a **Native Image-only** installer: no portable JVM/JAR distribution, no Java/GraalVM/JDK/Maven installation or selection, and no JVM fallback. This preserves the owner-ratified [DIST005-C JVM_PLUS_NATIVE artifact model](../DIST005/DIST005_C_ARTIFACT_MODEL_SELECTION.md): Protos continues to publish both artifacts for their separate consumers, while the one-command first-run installer consumes only Native.

On 2026-10-28, after the investigation was presented in chat, the owner replied **“perfecto, apruebo”** and requested the implementation handoff and durable evidence. The approved implementation *direction* is:

- One stable POSIX-shell bootstrap URL; **no Protos version or GraalVM version baked into the installer**.
- Dynamic discovery of published official Protos Native releases via GitHub Releases, including prereleases. Do not use GitHub `/releases/latest` (it excludes prereleases) or scrape the README as a release index.
- Validate the exact official Native archive, release manifest and checksums before unpacking/activating it.
- Preflight the supported host (Linux/architecture/glibc/CPU-ISA and shared-library requirements from the actual release metadata), then install the complete relocatable Native distribution under a versioned, unprivileged user-local root.
- Safe, idempotent install/update with previous-version retention and atomic switch only after successful staging/smoke. Existing CLI CWD and `PROTOS_HOME` semantics must be preserved.
- No `sudo` by default; no automatic host glibc upgrade, OS package-manager mutation, shell startup-file edit, or unsupported platform promise.

This approval is **not** publication authorization, a new Native runtime-model approval, or authorization for a new hosting service or release-candidate baseline. Any genuinely new durable design/security/publication choice remains subject to the appropriate owner decision gate.

## Evidence from current sources

1. [`bin/protos`](https://github.com/guillermomolina/protos/blob/main/bin/protos) computes `ROOT` from `dirname "$0"` and executes `libexec/protos-native` with `PROTOS_HOME` set to that actual distribution root. Therefore a direct `~/.local/bin/protos` symlink to versioned `bin/protos` is unsafe. Use a **managed wrapper** that execs the versioned launcher through its actual path, without modifying the Native distribution itself.
2. [`dist/build_native.py`](https://github.com/guillermomolina/protos/blob/main/dist/build_native.py) requires `bin/protos`, `libexec/protos-native`, `SOURCE.txt`, `RUNTIME.txt`, `DEPENDENCIES.txt`, `SHA256SUMS`, `protos/lib/`, `protos/tools/`, and `protos/tests/`, and rejects JAR leakage in the Native ZIP. It records the platform's observed ELF dependencies and glibc policy.
3. [`dist/release_asset_envelope.py`](https://github.com/guillermomolina/protos/blob/main/dist/release_asset_envelope.py) already encodes Native `asset.N.*` metadata in `RELEASE_MANIFEST.txt`: artifact kind, platform, target OS/arch, `external_java_required=false`, `libc_abi_min`, required shared libraries, CPU-ISA assumption, archive/checksum names, and SHA-256. The installer should consume this contract, not duplicate fixed runtime/version metadata.
4. [`dist/publish_release.py`](https://github.com/guillermomolina/protos/blob/main/dist/publish_release.py) requires an exact five-asset multi-artifact prerelease: Native ZIP and SHA-256, portable JVM ZIP and SHA-256, and `RELEASE_MANIFEST.txt`. The installer must not introduce new release assets or weaken that envelope without separately approved release-policy work.
5. [GitHub Releases API](https://docs.github.com/en/rest/releases/releases) exposes published release entries and asset digests. Its `/releases/latest` selects non-prerelease, non-draft releases and cannot select current Protos prereleases; the list-releases endpoint is the appropriate dynamic discovery input.
6. The repository README/Try Protos documentation observed during the investigation still advertised the prior `v0.3.312` while GitHub already published `v0.3.398`. A version-agnostic installer must not infer current release identity from static guides.

The current published Native target is Linux x86_64, dynamically linked to glibc with ABI floor **2.39**, plus the release's CPU-ISA/shared-library contract. The installer cannot make unsupported older-glibc systems compatible merely by downloading the Native ZIP.

## Alternatives and unresolved implementation detail

| Candidate | Result |
|---|---|
| GitHub list-releases API + reliable JSON handling | **Selected implementation direction**: no manual per-release installer/version edits and no new release-publishing channel |
| Automatically generated plain-text `latest-native` pointer | Fallback **requiring owner review** if a robust, small, dependency-acceptable API consumer proves impractical; it would add release-publication state |
| Hard-coded release version or `/releases/latest` | Rejected: manual drift or exclusion of published prereleases |

**Not yet proven**: a sufficiently reliable tiny release-JSON consumer without silently requiring `jq` or Python on a clean supported Linux host. I131 must settle this in focused implementation; if the constraint cannot be met safely, stop and request review rather than ship brittle JSON scraping or quietly add publishing infrastructure.

## Human-reported checks and limits

On 2026-10-28 the maintainer reported:

```text
git diff --check: CLEAN
make test: PASS
make check: PASS
```

Those are **maintainer-reported local baseline checks**, without supplied logs or an exact run-to-revision binding. They are not agent-executed, do not prove this record or an installer patch validated, and do not discharge the required future I131 checks.

No Protos language specification, implementation source, repository release, or installer file was changed by this investigation. Native-shell installation needs no source comparison against other Truffle languages: it consumes the already-published Protos Native artifact, not their compiler/runtime architecture.

## Next work

[I131 / #982](https://github.com/guillermomolina/protos/issues/982) is one bounded **IMPLEMENTATION** work item in `guillermomolina/protos`, covering installer, focused tests and directly necessary first-run documentation. Its agent edits the current human-maintained checkout at HEAD; the human executor runs build/test/validation and Protos content publication. No duplicate administrative I-slices are required solely because the patch touches multiple related files.

The issue remains open until the exact implemented/published delta and its applicable human-run checks have been verified. This evidence record does not close DIST018 or I131.
