# PERF034 — Local-read priority and measured graph progress (2026-10-08)

**Role:** non-normative evidence and owner-priority checkpoint. Live status and scope remain in [PERF034 / protos#845](https://github.com/guillermomolina/protos/issues/845). This record does not define a target node count or authorize a semantics change.

**Project-owner sequencing:** continue the `primitive-local-read` repair/investigation before resuming [PERF032/#831](https://github.com/guillermomolina/protos/issues/831) graph-node research. The latter is paused voluntarily, not technically blocked by a normative dependency. No cross-language parity or performance victory is inferred from structural changes alone.

## Exact published implementation checkpoints

- PERF034-B: [`guillermomolina/protos@c1d7de3865b9d96a465341aa5755e1de4da4a332`](https://github.com/guillermomolina/protos/commit/c1d7de3865b9d96a465341aa5755e1de4da4a332), `0.3.301-SNAPSHOT`. Published cold `CreateCurrentFrameLocal` and `ReadRootFrameLocal` fallbacks; maintainer-reported integrated `make test` PASS before metadata finalization.
- PERF034-C: [`guillermomolina/protos@497bda9da781a558ab769445054935b787fab5f5`](https://github.com/guillermomolina/protos/commit/497bda9da781a558ab769445054935b787fab5f5), `0.3.303-SNAPSHOT`. Admits parameterless straight-line literal-created scalar locals to native Bytecode DSL `StoreLocal`/`LoadLocal`, with canonical fallbacks where proof/admission is absent and for observable activation semantics; maintainer-reported focal and full integrated tests PASS before version/changelog changes.

## Human-reported, same-workload structural results

| Protos checkpoint | `primitive-local-read` selected After TruffleTier nodes | Caveat |
| --- | ---: | --- |
| G6-rest historical pre-B (product `b0776d0d...`) | 972 | Earlier Protos-only structural baseline |
| PERF034-B (`c1d7de38...`) | 374 | Stable selected root, one graph, BGV analysis PASS |
| PERF034-C (`497bda9d...`) | 103 | Stable Tier 2, one graph, BGV analysis PASS, `INVALID_UNITS=0` |

The exact PERF034-C raw result path supplied by the maintainer is `protos-benchmarks/results/perf034-c-local-read` (local only, not published as a retained benchmark-results commit). The reported graph shrink is 271 nodes from B to C and 869 nodes from the G6-rest historical baseline, but **not** a measured `ns/call` improvement. Historic GraalJS/GraalPy graph counts of 13/49 do not constitute same-capture peer parity against PERF034-C; the Protos-only matrix marks `peer=UNRESOLVED` and `protos_status=NOT_EVALUATED`.

## Next work and non-goals

Keep PERF034 **open/in progress**. Inspect existing PERF034-C `unit.json` graph classes, surviving invokes/guards, roots and Java/Truffle attribution before proposing a further bounded optimization. Establish measurable local-read costs and semantics-preserving causal evidence; do not infer that the difference to GraalJS's historical node count is entirely removable.

The owner-paused PERF032 concerns the separate `primitive-return-literal` node investigation. Do not silently merge its acceptance criteria into PERF034. PERF035's `primitive-object-slot-write` Tier-2 stabilization is independently observed at published product `203c0f91...`; its clean-checkout reference gate remains pending, as documented in [PERF035 evidence](../PERF035/PERF035_PUBLISHED_SLOT_WRITE_TIER2_STABILIZATION_2026_10_08.md).
