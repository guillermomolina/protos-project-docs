# WEB011-D5B2 — Dynamic website publication and immutable production image acceptance

**Formal work item:** [WEB011 / protos-website#7](https://github.com/guillermomolina/protos-website/issues/7).
**Related canonical Protos coordination:** [WEB001-H / protos#299](https://github.com/guillermomolina/protos/issues/299) (image delivery) and [WEB004 / protos#499](https://github.com/guillermomolina/protos/issues/499) (historical static-news implementation).
**Evidence date:** 2026-10-24.
**Website repository:** `guillermomolina/protos-website`.
**Exact published website commit:** [`5c86635df801bf4f4fa61c19b7e314de33031c20`](https://github.com/guillermomolina/protos-website/commit/5c86635df801bf4f4fa61c19b7e314de33031c20).
**Commit subject:** `WEB011: publish dynamic website content and verified GHCR D064 integration`.
**GitHub Actions publication:** [run 40074308584](https://github.com/guillermomolina/protos-website/actions/runs/40074308584), publish job completed successfully.

This is non-normative publication and acceptance evidence. It neither changes Protos semantics nor substitutes for the website implementation, canonical source repositories, or live Issue coordination.

## Published website scope

The commit publishes 79 changed paths (14,777 insertions, 499 deletions). Its bounded scope includes:

- NGINX Unprivileged 1.30.6 with njs 1.0.2, dynamically backed read-only GitHub release, Announcements, project-state and canonical-document endpoints.
- Browser adapters for live releases, news and project status, and the revision-coherent live documentation page.
- Exact-revision D064 acquisition from the Protos-owned GHCR/OCI artifact, without invoking a Protos documentation producer inside the website.
- Source normalization, bounded caching, input validation, safe rendering and endpoint contract tests.
- Retirement of the build-time website news materialization pipeline in favor of GitHub Discussions Announcements as the current live news authority.

Earlier [WEB004 / protos#499](https://github.com/guillermomolina/protos/issues/499) remains valid historical evidence of its original static-news implementation. WEB011 is the subsequent, approved delivery evolution; this record does not rewrite that history.

## Immutable website production artifact

The public discovery tag is:

```text
ghcr.io/guillermomolina/protos-website:sha-5c86635df801bf4f4fa61c19b7e314de33031c20
```

The **deployment-authority reference** is the immutable OCI **index** digest:

```text
ghcr.io/guillermomolina/protos-website@sha256:1d16b873f62d628b52edf4a68bc169f9449d9dee4878bfa0976a139980dc8b64
```

The maintainer's anonymous GHCR C17 manifest inspection returned:

```text
MANIFEST_MEDIA_TYPE=application/vnd.oci.image.index.v1+json
REGISTRY_DIGEST=sha256:1d16b873f62d628b52edf4a68bc169f9449d9dee4878bfa0976a139980dc8b64
CALCULATED_DIGEST=sha256:1d16b873f62d628b52edf4a68bc169f9449d9dee4878bfa0976a139980dc8b64
DIGEST_MATCH=True
REQUIRED_PLATFORMS_PASS=True
WEB011_D5B2_C17=PASS
```

The index lists `linux/amd64` and `linux/arm64`; an additional `unknown/unknown` entry is not a requested runtime platform. The separately observed `sha256:8b49a5f2313deb13a9e71f8b6cdcf7b16954bcbac5bad10cf40594730364f5e7` is **not** the selected multiarch index identity. The publication workflow itself completed successfully, including its anonymous image-manifest verification step.

## D064 producer/consumer identity

At the exact website commit, `protos-d064-publications.json` records:

```text
PROTOS_SOURCE_REVISION=14558fd9ea6758698ae4a8d4d38a273438212176
D064_CONTENT_SHA256=236597164f28f3d735ea329a9b20fbe348f7628347cc1e3e498d1650a12e80e9
D064_MANIFEST_DIGEST=sha256:19b27bb39e2321333ec570631a821e8d43c2d4c4c98c6cfd9c8150879418bc4e
```

The D064 manifest digest belongs to the **Protos documentation artifact**, not to the website container image. The website consumes and validates that artifact using its own separate production-image digest.

## Maintainer-executed validation

The maintainer reported a clean website `git diff --check` and all local tests passing prior to publication; the earlier integrated website checkpoint recorded 334 passing tests. The exact published GitHub commit and successful multiarchitecture image workflow were verified separately.

A disposable Podman container started successfully from the published GHCR image. The C15 runtime smoke reported:

| Runtime check | Observation |
| --- | --- |
| Image OCI revision label | Exact website commit `5c86635df801bf4f4fa61c19b7e314de33031c20` |
| `GET /` | HTTP 200 |
| `GET /docs/live/` | HTTP 200 |
| First `GET /api/live/docs/d064` | HTTP 200; 42,679 bytes |
| Second `GET /api/live/docs/d064` | HTTP 200; 42,679 bytes |
| `POST /api/live/docs/d064` | HTTP 403 |
| `GET /api/live/docs/d064?revision=arbitrary` | HTTP 400 |

Earlier real NGINX/njs focused checks on the live GitHub endpoints demonstrated a release cache MISS followed by HIT for `v0.3.398`, and an Announcements cache MISS followed by HIT returning 16 real entries; those checks preceded the final published-image smoke and must not be mislabeled as post-publication C15 endpoint checks.

**Podman healthcheck qualification:** the pulled GHCR OCI image was initially reported with `.Config.Healthcheck=null`; the initial `podman healthcheck run` therefore returned RC 125. This was **not** a failure of the website's HTTP health endpoint. C16 started a temporary container with the explicit runtime healthcheck `wget -q -O /dev/null http://127.0.0.1:8080/ || exit 1`, start period 5 s, interval 30 s, timeout 3 s and three retries. It returned `HEALTH_RC=0`, `healthy`, while `GET /` returned HTTP 200. Both disposable smoke containers were stopped.

The owner subsequently stated that the website **functions correctly and is validated**. This is a maintainer acceptance statement; no private deployment topology, live public-domain rollout or exact privately deployed digest is asserted in this public record.

## Worktree and authority boundaries

- The website publication committed its explicit 79-path staged set. Five unrelated tracked generated-document modifications remained outside that commit and were preserved.
- The official NGINX image is built for two architectures. Private DNS, TLS, reverse-proxy and deployment configuration are intentionally outside this public project evidence.
- The dynamic website is a read-only consumer. GitHub Issues/Project remain authoritative for live work state; GitHub Discussions Announcements for live news; Protos `spec/` for normative semantics; Protos source at an exact revision and the Protos-owned D064 OCI artifact for documentation.
- No Protos source, normative specification, release, or public tag was changed by this WEB011 website publication.

## Result

```text
WEB011_D5B2_WEBSITE_COMMIT=5c86635df801bf4f4fa61c19b7e314de33031c20
WEB011_D5B2_GHCR_PUBLICATION=PASS
WEB011_D5B2_MULTIARCH_DIGEST=PASS
WEB011_D5B2_D064_HTTP_SMOKE=PASS
WEB011_D5B2_RUNTIME_HEALTHCHECK=PASS_WITH_EXPLICIT_PODMAN_CONFIGURATION
WEB011_D5B2_OWNER_FUNCTIONAL_ACCEPTANCE=PASS
PUBLIC_PRIVATE_DEPLOYMENT_ASSERTION=NOT_MADE
PROTOS_SEMANTICS_CHANGED=NO
```
