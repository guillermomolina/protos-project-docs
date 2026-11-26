# GITHUB026 — Phase 2 publication and validation

Date: 2026-11-26. Status: published; automatic main acceptance pending at inspection.

## Revision and scope

- [GITHUB026 #1176](https://github.com/guillermomolina/protos/issues/1176); [BUG043 #1189](https://github.com/guillermomolina/protos/issues/1189); [I178 #1185](https://github.com/guillermomolina/protos/issues/1185); [D264 #1180](https://github.com/guillermomolina/protos/issues/1180).
- PROTOS_REVISION: [1afc4ceeb4b6df9878fb2d2f168235277f8e7e70](https://github.com/guillermomolina/protos/commit/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70), 0.4.230-SNAPSHOT, published to main by fast-forward without changing the validated candidate.
- Phase 1 remains accepted: [prior evidence](https://github.com/guillermomolina/protos-project-docs/blob/35aa7a02b9aa49ce4ee342dc9fc272641d86a3f4/docs/project/evidence/GITHUB026/GITHUB026_CI_RECOVERY_2026_11_26.md).
- Four Protos suite runners, two workers each, independent full-plan runner; four Java partitions, serial Java, prepare, compilerability, foreign providers, Native Image, and mandatory aggregate retained. Local default remains unsharded. No duplicate Test Tool selection implementation or production Java changes in the CI integration.
- Aggregation checks revision, ordered CaseRefs, disjoint complete union, invocation configuration, raw listings, terminal counts, exit status and mandatory job outcomes. Planned identity coverage plus terminal completion quantity is not independently observed per-Case completion identity.

## Local evidence (maintainer reported)

On 21f8957d611ac7cb07944264b62798d622742bc3 plus unpublished phase-2 tooling: complete listing 1,442,281 bytes, 4,034 Cases; ordered full listing and four shard listings reconciled; plan digest `6d31a48363e646769b7a8417853a03f747740cba7c7b318b9c105a6d26ae72b3`.

| Shard | Planned | Passed | Failed | Exit | Seconds |
| --- | ---: | ---: | ---: | ---: | ---: |
| 1 | 1009 | 1009 | 0 | 0 | 105.23 |
| 2 | 1009 | 1009 | 0 | 0 | 96.59 |
| 3 | 1008 | 1008 | 0 | 0 | 103.30 |
| 4 | 1008 | 1008 | 0 | 0 | 102.90 |
| Total | 4034 | 4034 | 0 | 0 | 408.02 |

Local evidence reconstruction and coverage aggregation PASS. Earlier orchestration fixtures: Java 14/14, Protos 10/10, aggregate 24/24; toolchain verifier PASS and drift count zero after fixture correction. Maintainer reports all local tests pass and clean diff for publication; individual final integrated command logs were not supplied. Local shard timing is sequential and not a remote speedup measurement.

## Remote candidate acceptance

[Workflow dispatch 45817611325](https://github.com/guillermomolina/protos/actions/runs/45817611325), branch ci-validation, attempt 1, same exact PROTOS_REVISION as main. Created 12:13:15Z, completed 12:24:19Z, wall time 11m04s, 15/15 jobs successful.

| Job | Job ID | Observed result |
| --- | ---: | --- |
| prepare | 139901302352 | PASS |
| compilerability | 139901302534 | PASS |
| java-parallel (1) | 139901941983 | 1601 tests, 6 skipped, 0 failures/errors |
| java-parallel (2) | 139901942076 | 1804 tests, 2 skipped, 0 failures/errors |
| java-parallel (3) | 139901941916 | 1728 tests, 2 skipped, 0 failures/errors |
| java-parallel (4) | 139901942001 | 1779 tests, 3 skipped, 0 failures/errors |
| java-serial | 139901941860 | 42 tests, 0 skipped, 0 failures/errors |
| protos-plan | 139901302326 | PASS |
| protos-suite (1) | 139901302634 | 1009 passed, 0 failed |
| protos-suite (2) | 139901302380 | 1009 passed, 0 failed |
| protos-suite (3) | 139901302299 | 1008 passed, 0 failed |
| protos-suite (4) | 139901302368 | 1008 passed, 0 failed |
| protos-foreign | 139901302364 | PASS |
| native-image | 139901301822 | PASS; NATIVE_GUEST_JIT=PASS, NATIVE_REGRESSION_SUITE=PASS |
| aggregate | 139906994557 | PASS |

Java totals from Surefire logs: 6954 tests including 13 skips, 6941 executed, zero failures/errors. Protos terminal summaries: 4034 passed, zero failed. Aggregate logs show all four Protos evidence artifacts downloaded, combined validator successful and [aggregate artifact 15117181215](https://github.com/guillermomolina/protos/actions/runs/45817611325/artifacts/15117181215) uploaded (613 bytes; ZIP SHA256 `fbae1e39965a82c649383ea9ba3f71e65845d0ef8ab89c39ac665f1e86c51b66`). [Complete plan artifact 15115004476](https://github.com/guillermomolina/protos/actions/runs/45817611325/artifacts/15115004476). Shard evidence artifact IDs: 15116498002, 15116940920, 15115709612, 15116880312. Artifact metadata and execution logs verified; archive contents not independently downloaded. No claim of measured end-to-end speedup.

## BUG043 follow-up

[Original resolution](https://github.com/guillermomolina/protos-project-docs/blob/46613660da6aa32e0870552831f5520be4e143bb/docs/project/evidence/BUG043/BUG043_CASE_LISTING_PUBLICATION_AND_VALIDATION_2026_11_26.md) fixed the large listing without changing runtime admission capacity. [Run 45813534028](https://github.com/guillermomolina/protos/actions/runs/45813534028) exposed one stale architectural guard in Java shard 4; aggregate correctly failed. [4f2f2de2d6e4b8d09948a9d015705cb681f979e5](https://github.com/guillermomolina/protos/commit/4f2f2de2d6e4b8d09948a9d015705cb681f979e5), 0.4.229-SNAPSHOT, reconciled that guard to the incremental writer while preserving validation-before-publication checks. Local focal class 5/5 PASS; remote [Java shard 4](https://github.com/guillermomolina/protos/actions/runs/45817611325/job/139901942001) confirms ProtosTestToolFileSelectionMainAdoptionTest 5/5 PASS. Independent main run 45817264278 was cancelled, not passed; its correction is included in the successful integrated candidate. BUG043 remote regression resolved.

## Publication and remaining acceptance

Main points to the exact validated candidate SHA. Automatic [main push run 45819938339](https://github.com/guillermomolina/protos/actions/runs/45819938339) is in progress at inspection, not accepted yet. The owner chose direct fast-forward after successful candidate CI instead of another redundant phase-2 PR run; no phase-2 PR validation is claimed. Phase-1 automatic PR and push evidence remains valid historical evidence. Current workflow retains both triggers. Keep GITHUB026 open until the final main outcome is verified and the closure criterion is reconciled with that publication path. BUG043 can close on the verified published correction and successful full candidate run. No new priority or assignee selected.
