# WEB010 — Live-main website publication and owner-reported production activation (2026-11-23)

**Evidence date:** 2026-11-23.  
**Website work item:** [WEB010 / guillermomolina/protos-website#6](https://github.com/guillermomolina/protos-website/issues/6).  
**Related recurring guard:** [WEB013 / guillermomolina/protos-website#9](https://github.com/guillermomolina/protos-website/issues/9); independently open at the time of this record.  
**Canonical Protos coordination:** [WEB001 / guillermomolina/protos#278](https://github.com/guillermomolina/protos/issues/278), previously closed website foundation.  
**Website source commit:** [`b66e47ce05f503680ba13bffe03d18dede3ee2e9`](https://github.com/guillermomolina/protos-website/commit/b66e47ce05f503680ba13bffe03d18dede3ee2e9).  
**Image publication workflow:** [GitHub Actions run 45172072293](https://github.com/guillermomolina/protos-website/actions/runs/45172072293), completed successfully on 2026-11-23.

This is non-normative, revision-bound evidence. It does not change Protos semantics, approve a new release, or replace website-local Issue coordination.

## Website change published

WEB010's published commit changes 34 paths (783 insertions, 590 deletions) and promotes the existing NGINX/njs-backed `/docs/live/` development reader to the primary Learn/Reference navigation.

- Specification, guide, tutorials, examples, and Standard Library **source** are read-only consumers of `guillermomolina/protos/main`, resolved to one exact Git commit and tree per catalog/document view. A cached or stale state is identified; this is not a claim of instant, uncached HEAD freshness.
- Static editorial entry points point to the dynamic reader instead of presenting materialized old source-lock pages as current.
- The active Astro build pipeline is Node-only, with no Java/Maven source-producer dependency. Historical materialization utilities and `protos-source.lock.json` remain in the repository but are not called by the active build; the source lock is not the authority for live development documentation.
- Website-owned branding assets are preserved and retired static generated pages are removed. The official Native installer is used in the version-independent homepage onboarding.
- The separate D064 API documentation artifact is served from its own validated GHCR publication with independent source provenance; it is not silently identified with development `main`.
- Local Podman `make serve`, `make smoke`, and stop/status/logs commands make the production-container checks repeatable without Docker on the developer host.

## Published immutable production artifact

**Discoverable commit tag:**

```text
ghcr.io/guillermomolina/protos-website:sha-b66e47ce05f503680ba13bffe03d18dede3ee2e9
```

**Immutable multiarchitecture OCI index:**

```text
ghcr.io/guillermomolina/protos-website@sha256:d8f146279e1ddd4d86203f8250c751ec2c4c62620e9b7f7dd7109b6beb1aa845
```

GitHub Actions run 45172072293 independently reports the image build and push successful, anonymous GHCR manifest inspection `PASS`, and `linux/amd64` plus `linux/arm64` platform coverage `PASS`. The OCI index digest is the published artifact's immutable identity; it is **not**, by itself, proof of the digest running in the private production environment.

## Local build and runtime validation

The maintainer supplied the following execution evidence from the website checkout / Podman host:

| Check | Observed result |
| --- | --- |
| `node --test tests/*.test.mjs` | **351/351 PASS**, 0 failures, 0 skipped |
| `npm run build` / production image build | **PASS**, Astro emitted 17 HTML pages; NGINX production target built |
| `git diff --check` and `git diff --cached --check` | **Clean** |
| `make smoke` after readiness | **PASS**: `/`, `/docs/live/`, `/api/live/docs/development`, `/api/live/docs/catalog`, `/api/live/docs/d064` all HTTP **200** |
| Development document snapshot | Commit `89028494587d4d22309a5c048e37da3518170b18`; catalog **387** entries |
| Independently published D064 | Source revision `14558fd9ea6758698ae4a8d4d38a273438212176`, correctly reported as **different** from the development snapshot |

An initial immediate `make smoke` encountered an HTTP connection reset. The running NGINX process then returned HTTP 200 both from inside the container and from the host, and the subsequent full smoke succeeded. The Makefile readiness check and JSON/provenance validation were hardened in the later tested batch; its focused tests are included in the 351-test passing suite.

The observed development commit identifies the **local smoke-test snapshot at that time**, not a permanently pinned website source. Development `main`, the latest published Protos prerelease, and the independently published D064 artifact are distinct authorities and need not resolve to the same SHA.

## Production activation report and verification boundary

After committing and pushing `b66e47c`, the maintainer explicitly reported on 2026-11-23 that the **new website is operational in production**, with local tests passing and `git diff --check` clean. This is **owner-reported production activation**, not an independently observed external HTTP/browser or running-container digest audit. Direct external inspection of `https://protos.guillermomolina.com/` was unavailable to the assistant. The private deployment topology and configuration are intentionally omitted from public project evidence.

The live deployment's **actual running image digest was not independently supplied or verified** here. Do not confuse the published OCI digest with a proven installed digest. Likewise, the observed catalog/D064 HTTP checks were against the local production container, not separately recorded as post-deployment public-host HTTP measurements.

WEB013's ongoing release/documentation coherence and regression-prevention obligations remain independently tracked; this publication does not claim them complete or close WEB013. WEB010 closure is governed by its website-local Issue and any remaining public-deployment acceptance checks.

## Evidence identifiers

```text
WEB010_WEBSITE_COMMIT=b66e47ce05f503680ba13bffe03d18dede3ee2e9
WEB010_GHCR_OCI_INDEX=sha256:d8f146279e1ddd4d86203f8250c751ec2c4c62620e9b7f7dd7109b6beb1aa845
WEB010_GHCR_MULTIARCH_AND_ANONYMOUS_READ=PASS
WEB010_NODE_TESTS=351/351_PASS
WEB010_ASTRO_PAGES=17
WEB010_LOCAL_PRODUCTION_SMOKE=PASS
WEB010_LOCAL_DEVELOPMENT_SNAPSHOT=89028494587d4d22309a5c048e37da3518170b18
WEB010_LOCAL_CATALOG_ENTRIES=387
WEB010_D064_PUBLISHED_SOURCE=14558fd9ea6758698ae4a8d4d38a273438212176
WEB010_D064_MATCHES_SNAPSHOT=NO_EXPECTED_SEPARATE_PUBLICATION
WEB010_OWNER_REPORTED_PUBLIC_WEBSITE=OPERATIONAL
WEB010_INDEPENDENT_PUBLIC_HTTP_AUDIT=NOT_PERFORMED
WEB010_ACTUAL_DEPLOYED_DIGEST=NOT_VERIFIED
WEB013_FUTURE_COHERENCE_GUARD=OPEN_SEPARATE_WORK
PROTOS_SEMANTICS_CHANGED=NO
```
