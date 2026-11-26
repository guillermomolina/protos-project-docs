# BUG043 — Incremental Case listing publication and validation evidence

Date: 2026-11-26

Live coordination: [BUG043 #1189](https://github.com/guillermomolina/protos/issues/1189). Downstream dependency: [GITHUB026 phase 2 #1176](https://github.com/guillermomolina/protos/issues/1176); [I178 #1185](https://github.com/guillermomolina/protos/issues/1185) remains accepted.

## Exact product publication

- PROTOS_REVISION: [21f8957d611ac7cb07944264b62798d622742bc3](https://github.com/guillermomolina/protos/commit/21f8957d611ac7cb07944264b62798d622742bc3), verified as published main at inspection.
- Commit: `BUG043: Serialize Test Tool Case listings incrementally`.
- Implementation version: `0.4.228-SNAPSHOT` (previously `0.4.227-SNAPSHOT`).
- Exact published delta: seven paths — CHANGELOG.md, pom.xml, protos/tools/test/CaseSelection.protos, protos/tools/test/Main.protos, and three existing Java test files (ProtosTestToolCaseSelectionPublicIntegrationTest, ProtosTestToolCaseSelectionPublicIntegrationTestSupport, ProtosTestToolCaseSelectionTest). No production Java, normative specification, JSON library, CI workflow, or corpus membership change.

## Baseline failure and correction

At [150b1b84429b9e06988eced0fdf9a91146a23257](https://github.com/guillermomolina/protos/commit/150b1b84429b9e06988eced0fdf9a91146a23257) (0.4.227-SNAPSHOT), with local unpublished CI-only changes, the maintainer observed the packaged unsharded listing exit 1 with zero stdout bytes. Shard 1/2 succeeded with 722934 bytes and 2017 Cases. Independent sequential awaited TextWriter experiments succeeded for single writes of 524288 and 1048576 bytes, failed at 1048577 bytes with zero output, and succeeded for three 524288-byte writes (1572864 total). This is a per-admission boundary, not a total document limit.

The published [CaseSelection.protos](https://github.com/guillermomolina/protos/blob/21f8957d611ac7cb07944264b62798d622742bc3/protos/tools/test/CaseSelection.protos) replaces whole-document materialization with emitListing, which emits one document over the retained authoritative projections, and writeListing, which uses the existing JSON.writeEvents adapter and awaits every feed and finish. [Main.protos](https://github.com/guillermomolina/protos/blob/21f8957d611ac7cb07944264b62798d622742bc3/protos/tools/test/Main.protos) awaits the trailing newline on the same borrowed writer only after successful document completion. No alternate serializer, retry, changed capacity, or new public contract is introduced. Individual encoded scalars remain subject to output admission capacity. Failure after an already committed prefix does not imply atomic document output.

## Maintainer validation

The maintainer reports all local tests PASS and clean git diff check. The actual packaged CLI listing reconciliation reported COMPLETE_ORDERED_SHARD_COVERAGE: PASS:

| Invocation | Stdout bytes | Cases |
| --- | ---: | ---: |
| Unsharded | 1442281 | 4034 |
| Shard 1/4 | 361313 | 1009 |
| Shard 2/4 | 359574 | 1009 |
| Shard 3/4 | 361665 | 1008 |
| Shard 4/4 | 359861 | 1008 |

The supplied validation command requires successful CLI exit, empty stderr, JSON parsing, schema, unique refs, exact ordered Case records in each shard and pairwise disjoint complete union. The unsharded output exceeds 1048576 bytes. Shard counts differ by at most one. These results establish planned Case identity coverage, not completed execution identities.

The published tests add a public full-corpus reconstruction with strict complete JSON framing and byte-exact expected output, a synthetic 128-Case large Unicode listing with independently derived expected refs/displays, and regressions for awaiting each write, stopping after write failure and propagating finish failure. Existing exact-selection, empty-shard, ordering and body non-execution assertions are preserved. Local test results are owner-reported, not independently executed by the coordinating agent. The original public CLI failure is the before-fix reproduction; no separate execution of the newly added JUnit regression against the old implementation or individual test counts has been supplied. No remote CI acceptance is claimed.

## Downstream disposition

The BUG043 functional blocker for GITHUB026 phase 2 is cleared by the verified publication and packaged CLI evidence. Preserve the already accepted phase 1 and the unpublished phase-2 CI candidate. Reconcile the actual phase-2 candidate against this correction, run its mandatory complete listing coverage gate, and require successful automatic CI at the integrated revision before closing GITHUB026. The full CI integration is not established by this record.
