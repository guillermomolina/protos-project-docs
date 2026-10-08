# PERF038-B — Compact method-call specialization: publication checkpoint

Date: 2026-10-08
Issue: https://github.com/guillermomolina/protos/issues/852
Product commit: https://github.com/guillermomolina/protos/commit/248b097e968219452b1663b9ed0fce597a719a79
Product revision: `248b097e968219452b1663b9ed0fce597a719a79`
Product version: `0.3.313-SNAPSHOT`
Status: product changes pushed; post-change graph A/B **not yet available**.

## Verified implementation

The product commit changes `ProtosFrameArguments.java`, `ProtosSemanticBytecodeRootNode.java`, and adds `ProtosPerf038BCompactMethodCallTest.java`, alongside `pom.xml` and `CHANGELOG.md`.

- The compact/unmaterialized root-argument discriminator uses frame argument zero for previously admitted compact calls instead of repeating full compact-header validation.
- Supplied-argument count/read chooses compact header layout by array length.
- `BindClosureFrameParameter` separates guarded compact frame-slot store from an authoritative respecializing fallback behind a boundary, intended to prevent activation materialization and lexical authority expansion in compact-only compilation sites.
- The changes preserve fallback behavior for observed/materialized activation and error cases as intended by the implementation. Runtime conformance is not independently re-tested at this checkpoint.

## Revision-bound pre-change evidence

Published baseline: https://github.com/guillermomolina/protos-benchmarks/tree/bf5af4dc1e6a30c3e614379a5b673d2cfb7c57ff/results/global-20261008-graphs/primitive-method-call

Product: `d59da442fd9bc6cf590fb9b2c82d3ff55c0391ae`, harness: `98abc9af7a05a45a7d4056b72f36aef5889cb76f`, GraalVM: `25.4.4.1.1`; `After TruffleTier`, tier 2, STABLE, correctness PASS, evidence valid YES.

| Metric | Protos pre-change | GraalJS reference | GraalPy |
|---|---:|---:|---:|
| Nodes | 1789 | 34 | 74 |
| Allocations | 41 | 1 | 1 |
| Control splits | 107 | 0 | 0 |
| Guards/deopts | 87 | 6 | 6 |
| Invokes | 30 | 0 | 0 |
| FrameState | 389 | 1 | 6 |

The historical Protos vs GraalJS node difference is 1755. These figures are graph structure, not proven timing improvement.

## Post-change acceptance pending

At this checkpoint, the most recent visible `guillermomolina/protos-benchmarks` publication is `bf5af4dc1e6a30c3e614379a5b673d2cfb7c57ff` (2026-10-08 18:34:35Z), which precedes product commit `248b097e968219452b1663b9ed0fce597a719a79` (2026-10-08 19:20:31Z). No verified post-PERF038-B capture is yet available.

Human executor should capture the **unchanged** `primitive-method-call` graph at product revision `248b097e`, preserving harness, workload, GraalVM, phase/tier, stabilization and correctness policy. Compare node families and residual invoke targets only if admission is valid. Do not infer a percent improvement from source inspection.

PERF038 / #852 remains OPEN pending valid post-change A/B and applicable functional validation evidence. No new design decision or specification change is recorded.
