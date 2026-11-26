# I178 — Deterministic distributed Test Tool Case sharding: publication and validation

Date: 2026-11-26

Live implementation coordination: [I178 #1185](https://github.com/guillermomolina/protos/issues/1185). Decision: [D264 #1180](https://github.com/guillermomolina/protos/issues/1180), [ratified D264-B′ record](https://github.com/guillermomolina/protos-project-docs/blob/8cc080e7d76aaa9a1c0bb74c0c2ce199b876a94d/docs/project/decisions/tooling/D264_TEST_TOOL_DISTRIBUTED_CASE_SHARDING.md).

## Exact product publication

- PROTOS_REVISION: [150b1b84429b9e06988eced0fdf9a91146a23257](https://github.com/guillermomolina/protos/commit/150b1b84429b9e06988eced0fdf9a91146a23257), independently verified as published main at inspection.
- Commit: `I178: Implement deterministic distributed Test Tool Case sharding`.
- Version: `0.4.227-SNAPSHOT`, incremented from `0.4.226-SNAPSHOT`, with matching implementation changelog.
- Exact commit delta: 16 existing files, including the two implementation metadata files; three bundled Protos production modules, one Protos option-adoption fixture, eight existing Java test classes, Makefile, Test Tool guide, pom.xml and CHANGELOG.md. No production Java, normative specification, CaseRef codec, scheduler, discovery or execution facility, CI workflow or aggregator file changed.

## Implementation and invariants

The exact published [Options.protos](https://github.com/guillermomolina/protos/blob/150b1b84429b9e06988eced0fdf9a91146a23257/protos/tools/test/Options.protos), [CaseSelection.protos](https://github.com/guillermomolina/protos/blob/150b1b84429b9e06988eced0fdf9a91146a23257/protos/tools/test/CaseSelection.protos) and [Main.protos](https://github.com/guillermomolina/protos/blob/150b1b84429b9e06988eced0fdf9a91146a23257/protos/tools/test/Main.protos) implement paired separate-token positive decimal ordinary Integer options, one-based shard indices and global round-robin after source and exact CaseRef selection. Integer values remain exact beyond signed-64. Unsupported attached shard options fail rather than silently select the complete suite; existing literal-value and unknown-argument handling is retained.

Main parses once before catalog acquisition or discovery. Attached-form prefixes are encoded and frozen once per argument projection. One invocation-local selection state spans the complete canonical source/suite/corpus traversal. Duplicate identities are checked and exact references resolved independently of shard membership. The original retained CasePlan entries and complete source discovery signatures feed the existing listing and execution paths. Absent sharding and explicit 1/1 perform no partition arithmetic. Valid empty shards succeed; invalid selection or discovery still fails. The existing scheduler, fresh Process isolation, private output, resource restrictions and jobs policy are unchanged.

The unchanged public listing schema identifies assigned Cases, not independent observations of all executed identities. Exact union, disjointness and balance within one Case require the same ordered selected plan on all runners; equal counts do not guarantee equal duration. Every invocation still discovers all in-scope sources.

Reproduction is documented in the [Test Tool guide](https://github.com/guillermomolina/protos/blob/150b1b84429b9e06988eced0fdf9a91146a23257/docs/guide/tools/test-tool.md) and [Makefile](https://github.com/guillermomolina/protos/blob/150b1b84429b9e06988eced0fdf9a91146a23257/Makefile):

```sh
protos test --shard-count 4 --shard-run 2 --jobs 1
protos test --list-cases --shard-count 4 --shard-run 2
make test-protos-suite PROTOS_TEST_SHARD_COUNT=4 PROTOS_TEST_SHARD_INDEX=2 PROTOS_TEST_JOBS=1
```

Default local make test remains unsharded.

## Validation evidence

Maintainer-supplied focal Maven transcript, 2026-11-26, after final refinement and before version/changelog finalization: 54 tests, zero failures/errors/skips, BUILD SUCCESS, total 2 min 19 s, finished 09:58:48 UTC, version 0.4.226-SNAPSHOT.

| Focal test class | Passed |
| --- | ---: |
| ProtosTestToolCaseRejectionPublicIntegrationTest | 7 |
| ProtosTestToolCaseSelectionPublicIntegrationTest | 8 |
| ProtosTestToolH2B3PublicIntegrationTest | 4 |
| ProtosTestToolCaseExecutionPublicIntegrationTest | 4 |
| ProtosTestToolCaseSelectionTest | 19 |
| ProtosTestToolHClosureReconciliationTest | 1 |
| ProtosTestToolFileSelectionMainAdoptionTest | 5 |
| ProtosTestToolResourceCatalogCompositionTest | 4 |
| ProtosTestToolLib011OptionsAdoptionTest | 2 |
| Total | 54 |

The maintainer subsequently reported make check PASS, make test PASS, make test-native PASS, and confirmed publication, clean git diff check and all local tests passing. The complete integrated and native logs and exact per-lane counts were not supplied; these are human-reported results, not independently executed agent validation. The focal log precedes the metadata increment, so it is not described as a Maven run bearing the final version. The published diff confirms the final refinement and version/changelog changes.

The automatic [push CI run 45801541443](https://github.com/guillermomolina/protos/actions/runs/45801541443) is bound to the exact product SHA and was in progress when inspected. No remote CI success or measured sharding speedup is claimed.

## Downstream coordination

The initial CI recovery was already accepted under [GITHUB026 #1176](https://github.com/guillermomolina/protos/issues/1176). Its owner-approved phase 2 owns integration of the new Protos suite matrix, revision-bound authoritative full/shard listings, complete union/disjointness and terminal count acceptance, preservation of other mandatory lanes, and remote integrated validation. I178 supplies the implementation prerequisite; no second guest partition policy or new public result schema is approved. Listing plus completion counts must not be presented as independently observed per-Case execution identity.

This snapshot is non-normative implementation and closure evidence under GITHUB020, not live lifecycle or scheduling authority.
