# WEB012 — Version-independent Try Protos homepage publication

**Evidence date:** 2026-10-31

**Owning issue:** [WEB012 / guillermomolina/protos-website#8](https://github.com/guillermomolina/protos-website/issues/8).

**Related historic work:** [WEB002 / guillermomolina/protos#387](https://github.com/guillermomolina/protos/issues/387) (closed newcomer-conversion work) and [WEB001 / protos#278](https://github.com/guillermomolina/protos/issues/278) (closed website foundation).

**Exact website commit:** [f8856ef9190bd2eb07bd59f4e9554a409c6cb4c1](https://github.com/guillermomolina/protos-website/commit/f8856ef9190bd2eb07bd59f4e9554a409c6cb4c1).

**Published image workflow:** [GitHub Actions #41229082953](https://github.com/guillermomolina/protos-website/actions/runs/41229082953), completed with success.

This is durable, non-normative publication evidence. It neither redefines Protos semantics nor replaces live Issue coordination.

## Delivered change

The owner expressly required the homepage to say **Try Protos** and show download and first-run instructions for the latest GitHub-published prerelease, without pinning any numeric release in the website HTML or copied shell commands.

The exact commit changes seven website files (481 insertions, 235 deletions), introducing a versionless release-onboarding generator, updating homepage markup/styles, adapting the cached same-origin live-release links and checksums, and replacing the relevant homepage tests. A version-independent release configuration and checker replace the former static numeric release lock.

The Native Linux x86_64 and portable JVM panels include copyable shell commands that resolve the latest published GitHub prerelease when executed, download the ZIP and adjacent SHA-256 checksum, verify the archive, extract it, run the official launcher --version and execute a hello-world expression. The JVM path explains the external supported GraalVM/JDK selection through JAVA_HOME, PROTOS_JAVA or PATH without assuming every future release has the same JDK requirement.

The browser independently uses the existing validated /api/live/release adapter to update the Native/JVM ZIP and checksum links and notes together. An unavailable endpoint leaves a versionless GitHub Releases fallback and independent shell-time release discovery; it does not present an old pinned release as latest. The website remains a read-only consumer; no Protos runtime, specification, release/tag, or canonical guide was changed.

## Human-executed local validation

| Gate | Observed result |
| --- | --- |
| Focused homepage tests | 13/13 PASS |
| Full Node test suite | 336/336 PASS; zero failures and zero skipped |
| npm run source:check | PASS |
| npm run release:check | PASS; current GitHub prerelease resolved as v0.3.428 at test time |
| npm run build | PASS, 248 static pages |
| git diff --check before commit | Clean |
| Visual local preview | Owner confirmed Try Protos works |

The source check reported canonical Protos documentation revision e99b712ecf658a527b1fe82c8beba2c1286d8dc9 and VS Code grammar revision 28c572560d51bbda33e6bd0a98f71fafbf2b2526. Those are exact source inputs, not download version pins. Astro logged non-blocking empty-i18n and missing-404-document warnings, but completed the build successfully.

The owner separately reported clean git diff checks and local make test / make check PASS after publication. Since that statement does not identify exact targets and revision, this record preserves it as owner testimony instead of treating it as a substitute for the website-specific Node/npm validation.

## Immutable GitHub Container Registry publication

The GitHub Actions workflow ran against exact source revision f8856ef9190bd2eb07bd59f4e9554a409c6cb4c1 and completed successfully at 2026-10-31 18:24 UTC.

**Deployment image:**

    ghcr.io/guillermomolina/protos-website@sha256:a07d0d87b0e39f2e163b478f0270c1f2c10a6b7cbe3512a5d81f02abf6b28c13

**Multiarchitecture OCI index digest:**

    sha256:a07d0d87b0e39f2e163b478f0270c1f2c10a6b7cbe3512a5d81f02abf6b28c13

The workflow independently reported anonymous GHCR manifest read PASS and supported platforms linux/amd64 and linux/arm64 PASS. This is the image **index** identity, not an archive checksum, per-platform manifest or source commit SHA.

## Deployment report and remaining observable boundaries

On 2026-10-31 the owner explicitly reported **published**. The exact private production deployment digest was not provided. The assistant's direct public-site read failed, so independently observed production HTTP, desktop/mobile rendering and a real downloaded Native/JVM first run remain unverified. Do not infer them merely from the local build or successful GHCR image publication.

    WEBSITE_IMAGE_GHCR_PUBLICATION=VERIFIED_SUCCESS
    WEBSITE_OWNER_REPORTED_DEPLOYMENT=PUBLISHED
    PRODUCTION_IMAGE_DIGEST_ON_HOST=NOT_INDEPENDENTLY_OBSERVED
    PUBLIC_WEB_HTTP_SMOKE=NOT_INDEPENDENTLY_OBSERVED
    REAL_DOWNLOAD_VERIFY_EXECUTE=NOT_INDEPENDENTLY_OBSERVED

## Coordination

- [Website WEB012 #8](https://github.com/guillermomolina/protos-website/issues/8) remains the live authority for any pending direct production acceptance.
- [Protos WEB002 #387](https://github.com/guillermomolina/protos/issues/387) is a completed, historic predecessor. This milestone does not reopen it or create a second owner of website-local implementation.
- [Website WEB010 #6](https://github.com/guillermomolina/protos-website/issues/6) remains independently responsible for exact-source documentation refresh and is not implicitly superseded.
- Protos specification and runtime semantics remain governed by guillermomolina/protos. GitHub Releases remain the asset authority; website publication is not a Protos release.

    WEB012_PUBLISHED_WEBSITE_SOURCE=PASS
    WEB012_NODE_TESTS=336_OF_336_PASS
    WEB012_ASTRO_BUILD=248_PAGES_PASS
    WEB012_GHCR_MULTIARCH_PUBLICATION=PASS
    WEB012_OWNER_DEPLOYMENT_REPORT=RECORDED
    WEB012_DIRECT_PRODUCTION_SMOKE=NOT_VERIFIED
    PROTOS_SEMANTICS_CHANGED=NO
