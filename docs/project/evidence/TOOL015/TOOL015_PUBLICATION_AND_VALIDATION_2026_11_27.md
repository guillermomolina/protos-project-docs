# TOOL015 — Test partitioning, selective CI and Maven cache validation

Date: 2026-11-27. Status: published and validated; final Issue review.

## Publication identity

- Owning work item: [TOOL015 #1216](https://github.com/guillermomolina/protos/issues/1216), standalone tooling work.
- Implementation: [c8e136778c02b8a362271c1698ee388aaa9bcc7d](https://github.com/guillermomolina/protos/commit/c8e136778c02b8a362271c1698ee388aaa9bcc7d).
- Intermediate cache correction: [8b9df4c61081d3d96dc6397e27d4d133600e5045](https://github.com/guillermomolina/protos/commit/8b9df4c61081d3d96dc6397e27d4d133600e5045), historical publication before verifier reconciliation.
- PROTOS_REVISION: [509221a9d28b79e9da9accf721fad2358d81ec49](https://github.com/guillermomolina/protos/commit/509221a9d28b79e9da9accf721fad2358d81ec49), final accepted CI candidate, containing cache and toolchain-contract corrections.
- Related infrastructure: TOOL011 #803, TOOL012 #814, I178 #1185, GITHUB026 #1176; Markdown I183 is the motivating corpus, not a parent or dependency to close here.

## Implemented scope

One TOOL015 implementation slice, with bounded CI corrections. No production Java files or normative specification files changed in the implementation commit. Repository source ownership is implemented in ordinary Protos in RepositoryTestOwnership.protos; mixed conformance and dedicated library sources are partitioned by authoritative corpus and source path. Filtering precedes logical discovery, preserving the original CaseRefs, order, execution bindings and existing shard algorithm. Public unfiltered `protos test` retains full coverage. The internal repository scope is not a new public CLI option.

- `make test`: Java, core/non-library Protos and foreign providers.
- `make test-protos`: core/non-library Protos.
- `make test-protos-libs`: complete registered Standard Library corpus.
- `make test-protos-lib LIB=toml` / `LIB=markdown`: complete selected library when registered; Markdown sources including official suite are present in the accepted revision.
- `make test-all` and `make check-all`: explicit exhaustive functional validation, with check controls in the latter.
- Existing independent foreign, compilerability and Native Image validation retained.

Impact selection, publication validation, scope-bound shard evidence and aggregation, weekly/manual exhaustive CI, release integration and development policy were reconciled in the implementation. Regression sources include repository-test-ownership.protos, ProtosTestToolRepositoryScopePublicIntegrationTest.java, repository completeness tests, impact/publication tests, CI shard/aggregate tests and release candidate tests. See [revision-bound Test Tool guide](https://github.com/guillermomolina/protos/blob/509221a9d28b79e9da9accf721fad2358d81ec49/docs/guide/tools/test-tool.md).

## Local validation

Maintainer reports clean git diff --check and all local tests pass. Focal toolchain verifier regression and development binding check were explicitly reported PASS, with TOOLCHAIN_DRIFT_COUNT: 0. No independent execution of local commands or individual final local command transcripts for every acceptance case are claimed.

## Remote validation and CI recovery

- [Run 45989553277](https://github.com/guillermomolina/protos/actions/runs/45989553277): initial attempt had 18/20 successful jobs; core shard 3 failed before discovery because Maven Central returned HTTP 429 resolving Surefire 3.5.6. Aggregate rejected missing required evidence. Second attempt completed successfully.
- [Run 45992765989](https://github.com/guillermomolina/protos/actions/runs/45992765989): intermediate cache correction exposed strict toolchain-contract drift in ci.tests.command and ci.diagnostics.contract. The verifier and negative regression fixtures were reconciled; no guard was disabled.
- [Final run 46010551912](https://github.com/guillermomolina/protos/actions/runs/46010551912): push, attempt 1, exact PROTOS_REVISION above, completed SUCCESS, all 20 jobs successful. Both scope plans, four Java parallel shards, Java serial, prepare, compilerability, foreign providers, Native Image and mandatory aggregate succeeded.

| Scope | Shard | Job ID | Passed | Failed |
| --- | ---: | ---: | ---: | ---: |
| core | 1 | 140545728677 | 385 | 0 |
| core | 2 | 140545728775 | 385 | 0 |
| core | 3 | 140545728827 | 385 | 0 |
| core | 4 | 140545728687 | 385 | 0 |
| libraries | 1 | 140545728683 | 1366 | 0 |
| libraries | 2 | 140545728729 | 1365 | 0 |
| libraries | 3 | 140545728744 | 1365 | 0 |
| libraries | 4 | 140545728772 | 1365 | 0 |
| **Total** | | | **7001** | **0** |

Core total 1540; library total 5461. The library portion is approximately 78% of these logical Cases, not an elapsed-time speedup. Planned identities plus terminal completion quantities do not establish independently observed per-Case execution identities.

Evidence identities: [requirements artifact 15203022484](https://github.com/guillermomolina/protos/actions/runs/46010551912/artifacts/15203022484), [core plan 15203243393](https://github.com/guillermomolina/protos/actions/runs/46010551912/artifacts/15203243393), [library plan 15202988132](https://github.com/guillermomolina/protos/actions/runs/46010551912/artifacts/15202988132), [aggregate job 140551224209](https://github.com/guillermomolina/protos/actions/runs/46010551912/job/140551224209), [aggregate artifact 15203064380](https://github.com/guillermomolina/protos/actions/runs/46010551912/artifacts/15203064380). Job conclusions, shard terminal logs and artifact metadata verified; archives were not independently downloaded.

## Maven cache evidence

The original cache was populated by test-compile only; package still needed missing Surefire artifacts. Prepare now runs package preparation before the explicit save, the key is versioned to maven-v2, and diagnostic saves use a separate namespace.

[Prepare job 140544639670](https://github.com/guillermomolina/protos/actions/runs/46010551912/job/140544639670) restored the prior key as fallback, downloaded the missing Surefire 3.5.6 artifacts and saved `Linux-maven-v2-f38a6b2feba78310a3f13c6500ef6646bb5adeebb00e3464c94f554a1bcffadc`. All eight Protos shard logs show exact cache hits and zero Surefire download lines from Central. This validates the observed package-dependency correction, not a guarantee of zero network access, full native/provider prefetch or immunity to future external rate limits.

## Acceptance boundaries

The accepted CI run exercised both partitions and fail-closed aggregation. Existing regression tests cover ownership, per-library selection, invalid scopes, selective impact, evidence integrity and release routing; the maintainer reports all local tests passing. Individual TOML/Markdown execution transcripts, each selective CI scenario as a separate live workflow, and an actual release candidate publication are not separately evidenced here. No performance threshold or release version is inferred. No separate work item or design ratification is requested.
