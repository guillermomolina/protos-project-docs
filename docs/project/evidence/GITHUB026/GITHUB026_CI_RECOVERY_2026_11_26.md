# GITHUB026 — Distributed CI recovery: phase 1 publication and acceptance

Date: 2026-11-26

- Owning issue: [GITHUB026 #1176](https://github.com/guillermomolina/protos/issues/1176).
- Related: [BUG041 #1183](https://github.com/guillermomolina/protos/issues/1183), [D264 #1180](https://github.com/guillermomolina/protos/issues/1180), [I178 #1185](https://github.com/guillermomolina/protos/issues/1185).
- PROTOS_REVISION: [234f3a41175be71409839000e99c21add4ffe786](https://github.com/guillermomolina/protos/commit/234f3a41175be71409839000e99c21add4ffe786), 0.4.226-SNAPSHOT.
- [PR #1182](https://github.com/guillermomolina/protos/pull/1182) merged by fast-forward publication to main; verified merged at 08:34:09 UTC. The final PR delta contains eight CI, tooling, documentation and metadata paths and no Java source or test delta relative to main.

## Recovery and diagnostic evidence

Initial distributed validation at 0561cf229766373af4f3d2dc6ce8a70a93c2904f used four deterministic Java partitions and a serial lane, independent compilerability, Protos suite, foreign-provider and Native Image jobs, a preparation job and a fail-closed aggregate. The canonical local make test contract remained intact. [Run 45604596726](https://github.com/guillermomolina/protos/actions/runs/45604596726) exposed post-job Maven cache hashFiles failures and a separate intermittent HTTP test failure. The cache correction computes a stable tracked-POM key, uses explicit restore and save, and does not make cache storage a mandatory validation gate.

The first isolated diagnostic runs failed because checkout removed the diagnostic log directory created before checkout; the correction initializes it after checkout. On b1c9413f62453be059c5f21f35cc5d38259db691 the following diagnostics passed:

- [45759218947](https://github.com/guillermomolina/protos/actions/runs/45759218947): Maven cache save and exact marker restore verified.
- [45759222031](https://github.com/guillermomolina/protos/actions/runs/45759222031): exact HTTP method executed 20 times, 20 successes, zero failures; this does not establish absence of an HTTP race.
- [45759224993](https://github.com/guillermomolina/protos/actions/runs/45759224993): Java shard 2, 1804 tests, zero failures/errors, two skipped.
- [45760355890](https://github.com/guillermomolina/protos/actions/runs/45760355890): manual full workflow, attempt 1 failed in shard 1; attempt 2 passed all eleven jobs. [45762242458](https://github.com/guillermomolina/protos/actions/runs/45762242458) reproduced the same shard 1 external child custody refusal on the original PR candidate. See the BUG041 record below. Neither failure was masked.

The published runtime corrections BUG041 and BUG042 were incorporated before final PR validation. The mandatory workflow uses the pinned CI image sha256:44e03c759ccd2b57135f5c0bdbb5060563e858601daa73958ab6cbba2ffc5fe3, four Java shards, two Java class workers per runner and two Protos jobs in the unsharded suite lane. Revision-bound Java plan/result reconciliation and mandatory dependency results are checked by the aggregate.

## Final automatic validation

| Event | Run | Checkout/evidence revision | Result | Run elapsed (API timestamps) | Aggregate artifact ID |
| --- | --- | --- | --- | --- | --- |
| pull_request | [45781930357](https://github.com/guillermomolina/protos/actions/runs/45781930357) | 766d81f9310373036deac00b3dc19139d9a89f76 (PR merge candidate; head 234f3a41175be71409839000e99c21add4ffe786) | 11/11 jobs PASS, attempt 1 | 15m41s | 15100540314 |
| push main | [45784270517](https://github.com/guillermomolina/protos/actions/runs/45784270517) | 234f3a41175be71409839000e99c21add4ffe786 | 11/11 jobs PASS, attempt 1 | 11m22s | 15100377343 |

All required jobs passed: prepare, compilerability, four java-parallel partitions, java-serial, protos-suite, protos-foreign, native-image and aggregate. The aggregate jobs verified mandatory dependency results and Java coverage and uploaded revision-named result.json artifacts. Artifact identities are verified from job logs; binary artifact contents were not independently downloaded. Node.js 20 action deprecation warnings were present but did not fail validation; no dependency upgrade is claimed.

Main Surefire summaries (reported tests include skips):

| Lane | Job ID | Tests | Failures | Errors | Skipped |
| --- | --- | ---: | ---: | ---: | ---: |
| Java parallel 1 | 139787580254 | 1596 | 0 | 0 | 6 |
| Java parallel 2 | 139787580085 | 1804 | 0 | 0 | 2 |
| Java parallel 3 | 139787580160 | 1717 | 0 | 0 | 2 |
| Java parallel 4 | 139787579956 | 1775 | 0 | 0 | 3 |
| Java serial | 139787579867 | 42 | 0 | 0 | 0 |
| Total | | 6934 | 0 | 0 | 13 |

Total executed Java tests excluding skips: 6921. These are sums of observed Surefire summaries, not extracted aggregate JSON fields.

## BUG041 remote acceptance

The correction was published at [28d3c1df6531e31fd795ed9673b31e18bb13fc82](https://github.com/guillermomolina/protos/commit/28d3c1df6531e31fd795ed9673b31e18bb13fc82). The prior revision-bound [local validation and negative-control evidence](https://github.com/guillermomolina/protos-project-docs/blob/908a7b60f8e6e7ba03531dbc9b8405420036e07a/docs/project/evidence/BUG041/BUG041_EXTERNAL_CHILD_CUSTODY_PUBLICATION_AND_VALIDATION_2026_11_26.md) remains historical and is supplemented by this record.

- [PR shard 1](https://github.com/guillermomolina/protos/actions/runs/45781930357/job/139779742675) and [main shard 1](https://github.com/guillermomolina/protos/actions/runs/45784270517/job/139787580254) both passed: 1596 tests, zero failures/errors, six skipped. The external launch class reported 20 tests, zero failures/errors, one skipped in both runs; the original failing method was not removed or excluded.
- [Main shard 3](https://github.com/guillermomolina/protos/actions/runs/45784270517/job/139787580160) reports the external process custody class 49/49 PASS, zero failures/errors/skips.
- Both full automatic workflows passed, including mandatory aggregate. This satisfies the outstanding remote acceptance of BUG041; attribution of historical failures to the exact deterministic interleaving remains unproven.

## Scope extension

Phase 1 CI recovery is complete. The owner explicitly requested on 2026-11-26 that GITHUB026 remain open for phase 2: integrate the ratified D264-B′ Protos Test Tool sharding implemented by I178 after it is ready and validated. This supersedes the earlier coordination assumption that GITHUB026 would close after initial recovery, without invalidating the completed phase 1 evidence or creating a new semantic decision.

Use the authoritative Test Tool CasePlan partition membership; retain unsharded local defaults, independent jobs and existing foreign-provider, compilerability, Native Image and Java coverage. Reconcile complete authoritative listing against shard listings with disjoint union, validate revision/partition identities and terminal counts, reject missing, duplicated, inconsistent or failed evidence. Listing plus counts is planned identity coverage plus completed quantity, not independently proven per-Case terminal identity; any new public structured result contract requires the existing design process. No production Java infrastructure or duplicate partition policy is authorized. Actual matrix size and integration details remain to be established against I178 and measured validation; no phase 2 implementation or validation is claimed here.

DURABLE_RECORD_DECISION=REQUIRED (owner-requested revision-bound operational recovery and BUG041 remote evidence). GITHUB026 remains open for the extended scope; this is a phase-1 evidence record, not its final closure record.
