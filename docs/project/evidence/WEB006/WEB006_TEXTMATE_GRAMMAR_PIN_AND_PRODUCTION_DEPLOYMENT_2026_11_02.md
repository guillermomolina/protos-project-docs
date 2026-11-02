# WEB006 — TextMate grammar pin refresh and production deployment (2026-11-02)

**Classification:** historical, non-normative cross-repository publication and deployment evidence.
**Work owner:** [WEB006 / guillermomolina/protos-website#2](https://github.com/guillermomolina/protos-website/issues/2).
**Authority:** [D175 / guillermomolina/protos#523](https://github.com/guillermomolina/protos/issues/523), ratified Candidate A.
**Extension regression:** [BUG002 / guillermomolina/protos-vscode-extension#4](https://github.com/guillermomolina/protos-vscode-extension/issues/4), completed.

## Exact published identities

```text
DATE=2026-11-02
WEBSITE_REPOSITORY=guillermomolina/protos-website
WEBSITE_REVISION=c02b1f6e24fda3f4ab11de4a3b373127e14c085f
WEBSITE_COMMIT=WEB006: refresh TextMate grammar and remove static version labels
EXTENSION_REPOSITORY=guillermomolina/protos-vscode-extension
EXTENSION_REVISION=82ceb83872b3ef25362b4b817352b088d21cee59
EXTENSION_VERSION=0.2.3
EXTENSION_GITHUB_RELEASE=v0.2.3
EXTENSION_RELEASE_VSIX_SHA256=0480dbd044ff5b740f3181a11270827e66202193c1088562aec94cd9097320d7

GITHUB_WORKFLOW_RUN_ID=41403528229
GITHUB_WORKFLOW_JOB_ID=125140345674
GITHUB_WORKFLOW_RESULT=SUCCESS
PRODUCTION_IMAGE=ghcr.io/guillermomolina/protos-website@sha256:67c39f28c68f8384892a343af59d6242ac529c117164cdf473940313a0f9ae1e
ANONYMOUS_GHCR_READ=PASS
PUBLISHED_PLATFORMS=PASS linux/amd64 linux/arm64
PRODUCTION_DEPLOYMENT=USER_REPORTED_COMPLETED
DIRECT_PRODUCTION_INSPECTION=NOT_PERFORMED
```

Website source: [exact commit](https://github.com/guillermomolina/protos-website/commit/c02b1f6e24fda3f4ab11de4a3b373127e14c085f). Image publication: [GitHub Actions run](https://github.com/guillermomolina/protos-website/actions/runs/41403528229). Extension: [release v0.2.3](https://github.com/guillermomolina/protos-vscode-extension/releases/tag/v0.2.3).

## Website changes

The website commit changed exactly two source paths:

- `protos-vscode-source.lock.json`: pinned the canonical TextMate grammar from the historical extension SHA `28c572560d51bbda33e6bd0a98f71fafbf2b2526` to `82ceb83872b3ef25362b4b817352b088d21cee59`. The existing read-only, exact-SHA acquisition and renderer-adaptation contract was preserved; no website-maintained independent grammar was added.
- `src/pages/index.astro`: removed the obsolete visible `v0.1` draft labels in favor of version-independent experimental status text. Live release links and GitHub-announcement news were not changed; the published release version remains available in its own news/release surface.

The extension revision fixes `args` ordinary identifier highlighting and adds valid-source coverage of recent contextual `for`, `return`, Range/InvertedRange, `each`/`eachStep`, and bitwise/shift syntax. The website does not interpret those constructs independently.

## Human-executed validation

The owner supplied the terminal output for the website candidate, before publishing the website commit:

```text
WEB_GRAMMAR_UPDATE=APPLIED
PROTOS_VSCODE_SOURCE_FETCH=PASS revision=82ceb83872b3ef25362b4b817352b088d21cee59
PROTOS_GRAMMAR_READY=82ceb83872b3ef25362b4b817352b088d21cee59
PROTOS_BROWSER_GRAMMAR_READY=82ceb83872b3ef25362b4b817352b088d21cee59
NODE_TESTS=336/336 PASS
NPM_RUN_CHECK=PASS
NPM_RUN_BUILD=PASS (Astro static build, 248 routes reported)
GIT_DIFF_CHECK=CLEAN
EXPECTED_CHANGED_FILES=2
```

The Astro build reported nonfatal warnings for the empty `i18n` collection and 404 entry; no build failure was observed. These results are human-supplied validation evidence, not an independently run local test suite by the agent.

## CI publication and reported deployment

The workflow run for exact website commit `c02b1f6e24fda3f4ab11de4a3b373127e14c085f` returned `completed/success`. The job logs recorded the immutable multi-platform image digest above and the independent checks:

```text
ANONYMOUS_GHCR_READ: PASS
PUBLISHED_PLATFORMS: PASS linux/amd64 linux/arm64
```

On 2026-11-02, the owner replied **"desplegada"**, confirming deployment to their server. This is owner-reported operational deployment; no direct HTTP check of the production website, container runtime, or live digest on the private server was performed. The image publication identity is verified in GitHub, but the private-server running image remains an owner attestation.

## Authority and closure

D175 remains ratified: the extension owns the canonical non-normative TextMate grammar; website consumption is pinned to an immutable extension revision and build preparation verifies that revision before materialization. No Protos specification, runtime, or library semantic change was made.

WEB006's prior migration work remains historically closed; this later exact-revision refresh and website deployment complete the previously noted consumer follow-up for extension BUG002. Historical 2026-11-02 records that predate this deployment remain accurate snapshots and are not rewritten.
