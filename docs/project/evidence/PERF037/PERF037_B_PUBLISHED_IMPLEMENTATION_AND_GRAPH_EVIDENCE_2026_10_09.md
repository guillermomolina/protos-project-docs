# PERF037-B — Published implementation and structural graph evidence (2026-10-09)

**Role:** revision-bound, non-normative implementation and measurement evidence; the owning GitHub Issue remains the authority for live status.

- **Owning issue:** [PERF037/#851](https://github.com/guillermomolina/protos/issues/851).
- **Published Protos implementation:** [`1c02b5d11c6e771910b22e213bc5fa76bc5315f7`](https://github.com/guillermomolina/protos/commit/1c02b5d11c6e771910b22e213bc5fa76bc5315f7), `PERF037-B: eliminate avoidable object-slot-read overhead (#851)`; published after `248b097e` (PERF038-B), without overwriting that concurrent implementation.
- **Before-reference Protos revision:** [`d59da442fd9bc6cf590fb9b2c82d3ff55c0391ae`](https://github.com/guillermomolina/protos/commit/d59da442fd9bc6cf590fb9b2c82d3ff55c0391ae), `0.3.312-SNAPSHOT`.
- **Published reference graph corpus:** [`guillermomolina/protos-benchmarks/results/global-20261008-graphs`](https://github.com/guillermomolina/protos-benchmarks/tree/bf5af4dc1e6a30c3e614379a5b673d2cfb7c57ff/results/global-20261008-graphs/primitive-object-slot-read), repository HEAD `bf5af4dc1e6a30c3e614379a5b673d2cfb7c57ff); reference graph producer recorded as `98abc9af7a05a45a7d4056b72f36aef5889cb76f`.
- **New post-implementation measurement:** human-executed *locally* at product `1c02b5d11c6e771910b22e213bc5fa76bc5315f7`, directory `results/perf037-b-1c02b5d1-graphs/` within `guillermomolina/protos-benchmarks`. **No publication of this new raw capture to the benchmark repository is established here.** The result and verification output below were supplied by the maintainer, not independently rerun by the coordinating agent.

## 1. Implemented change

The product commit changes 12 files (600 insertions, 59 deletions, according to the maintainer's Git commit output). The substantive optimization, as summarized by the implementing agent and identified in the published product source, groups:

1. **B1 — captured lexical binding read:** root-level `ReadCapturedFrameLocalAtRoot`, `SelectCapturedMaterializedOwnerFrameAtRoot`, and `ReadCapturedFallbackAtRoot` avoid allocating a rich `ProtosActivation` on a successful compact-frame captured lookup; generic fallbacks remain.
2. **B2 — member read:** root-level `ReadMemberAtRoot` removes the activation operand, admitting a successful PIC lookup without materialization and deferring exact activation acquisition to unsupported/absent fallback.
3. **B3 — physical member location:** lazily create `SlotCell` inside the existing map-backed binding authority when a guarded PIC needs one. The cached selection reads the cell's current value and retains structural invalidation, rather than repeating string-keyed authority reads on each valid hit.
4. **B4 — Closure extraction:** preserve a fresh bound Closure when required while profiling the Closure branch off the ordinary integer slot path.
5. **B5 — successful-read error costs:** avoid the old `orElseThrow` path on a guarded hit.

The maintainer reports `git diff --check` clean and **all local tests PASS** after implementation. This is a human-reported validation statement; specific test transcript, immutable local test-run identity and per-test counts were not supplied. No new normative Protos semantics were approved by this optimization.

## 2. Exact graph comparison

Workload: `primitive-object-slot-read` (Protos source `holder: { value: 1 }; run: () => { holder.value }`; expected result `1`). Measured operation: prepared `Value.execute()`, Protos canonical and GraalJS/GraalPy executable-value surfaces. Graph: one selected final Tier-2 guest unit, phase `After TruffleTier`, natural-warmup stabilization.

| Graph metric | Reference Protos `d59da442` | New Protos `1c02b5d1` | Reference GraalJS | Reference GraalPy |
| --- | ---: | ---: | ---: | ---: |
| Total nodes | 683 | **253** | 36 | 103 |
| Invocations | 14 | **3** | 0 | 0 |
| Allocations | 17 | **0** | 1 | 1 |
| Control-flow splits | 37 | **16** | 0 | 1 |
| Guards/deopts | 40 | **15** | 6 | 8 |
| Loads | 47 | **24** | 7 | 12 |
| Loops | 2 | **1** | 0 | 0 |
| FrameState | 126 | **31** | 1 | 10 |

- **Measured node reduction:** `683 - 253 = 430` nodes, `430 / 683 = 62.96%` fewer nodes.
- **Residual to the requested 90% reduction target:** `253 - 68 = 185` nodes to reach at most 68.
- **Remaining ratio:** `253 / 36 = 7.03x` the GraalJS graph and `253 / 103 = 2.46x` GraalPy.
- **Reference status:** all three baseline graphs were `STABLE` and `evidence_valid=true`. The maintainer's new `summarize` output states Protos 253, JS 36 and Python 103, all `valid=YES`, `INVALID_UNITS=0`, and Protos `STABLE`; the peer reference was `CONVERGED` and the structural conclusion remains `STRUCTURAL_EXCESS`.

For the new locally measured run, the maintainer supplied:

```text
PRODUCT_REVISION=1c02b5d11c6e771910b22e213bc5fa76bc5315f7
PRODUCT_CLEAN=True
EVIDENCE_VALID=True
NODES=253
FAMILIES={'allocations': 0, 'control_flow_splits': 16, 'guards_deopts': 15, 'invokes': 3, 'loads': 24, 'loops': 1}
ANALYZE_FAILURES=0
INVALID_UNITS=0
CASES=3
WORKING_TREE_MATCHES_PRODUCER=YES
HEAD_MATCHES_PRODUCER=YES
```

This is an *observed graph reduction*, **not** proof of a runtime latency improvement. The published before-reference timing admitted Protos `337.75195 ns/call` and JS `54.9979 ns/call` in the previous product revision; **post-B timing is NOT_MEASURED**. Do not use the before-only timing pair as a claimed before/after speedup.

## 3. Remaining graph costs — post-B attribution

The maintainer's new BGV-derived `unit.json` reports:

```text
SURVIVING_INVOKES:
3 ProtosLexicalBindingAuthorityCalls.contains

TOP_ATTRIBUTION (self nodes):
150 ProtosSemanticBytecodeRootNodeGen$CachedBytecodeNode
5   ProtosSemanticBytecodeRootNodeGen$ReadMemberAtRoot_Node
5   ProtosSemanticBytecodeRootNodeGen$UncachedBytecodeNode
2   ProtosSemanticBytecodeRootNodeGen

TOP_NODE_CLASSES:
32 BeginNode
31 FrameState
25 ConstantNode
20 EndNode
20 LoadFieldNode
19 PiNode
16 IfNode
12 IsNullNode
11 FixedGuardNode
7 VirtualObjectState
6 InstanceOfNode
6 ValuePhiNode
4 MergeNode
4 ObjectEqualsNode
4 VirtualArrayNode
```

**Source-derived explanation, not a proven per-invoke hot-path count:** in product `1c02b5d1`, `ProtosBytecodeRootNode.capturedOwnerWithoutNearerBinding` traverses the captured lexical chain and checks nearer scope membership with `ProtosLexicalEnvironment.hasLocalSlotForRuntime(name)`; materialized contexts may reach `ProtosObjectValue.hasLocalSlot` and `ProtosLexicalBindingAuthorityCalls.contains`. The current root-lowered captured-materialized lookup also performs owner-frame selection, a conditional `LoadLocalMaterialized` and fallback. The three surviving `contains` calls are present in the compiled graph but **have not individually been proved to execute on every successful lookup**; attribution to individual branches needs care.

The residual 150-node attribution of `CachedBytecodeNode` is a compiler attribution bucket, **not** proof that 150 nodes are independently removable or attributable to a single operation. The specific `ReadMemberAtRoot_Node` bucket now contributes only five self nodes.

## 4. Next implementation: PERF037-C

**Type:** IMPLEMENTATION, **repository:** `guillermomolina/protos`, **same issue:** `#851`. No new issue is required for a bounded coherent slice.

Implement a semantically transparent, structurally guarded cached owner/presence selection for stable captured lexical reads, eliminating repeated membership checks on the common successful path while preserving D179 late creation/removal retargeting, lexical shadowing, `PRESENT(null)` versus `ABSENT`, escaped frames, BUG018 tier-safe retained-frame reads and exact fallback. Where useful, compact the redundant root-level Bytecode DSL selection/conditional/read pipeline without moving work into per-hit `@TruffleBoundary` calls. Avoid changes to `ReadMemberAtRoot` unless required by the demonstrated residual graph.

**Verification:** human-run focal tests, static guards and one required integrated `make test` for substantive source changes; implementation publication after green validations and late `pom.xml`/`CHANGELOG.md` finalization. Post-C, capture/analyze/verify **Protos only** against the unchanged peer/reference graphs. The 90% node-reduction target is an optimization target, not a right to weaken Protos semantics. Further work remains open until independently admissible measurement and acceptance.

## 5. Scope and publication boundaries

- This record documents **human-provided** local measured output and directly inspected source, without claiming that the coordinating agent ran commands or profilers.
- The **new raw graph corpus** is local and must be published separately in `guillermomolina/protos-benchmarks` if independently downloadable BGV/trace evidence is required. The existing public `global-20261008-graphs` corpus is the *before* reference.
- The product commit is published; the target was not fully achieved and the performance Issue is not closable yet.
- The canonical live next action belongs in [PERF037/#851](https://github.com/guillermomolina/protos/issues/851).
