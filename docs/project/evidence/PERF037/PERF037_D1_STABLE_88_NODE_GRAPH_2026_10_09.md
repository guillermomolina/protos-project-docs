# PERF037-D1 — Stable 88-node graph, human-executed measurement (2026-10-09)

**Role:** non-normative, human-supplied local post-implementation graph evidence. This file does **not** publish the raw BGV artifacts, prove a speedup, or close the parent work item.

## Identity and provenance

- **Owning live issue:** [PERF037 / `guillermomolina/protos#851`](https://github.com/guillermomolina/protos/issues/851).
- **Published product commit expected from capture path:** [`3e94ba7d01cf94852555d3daa18a3208d3908e35`](https://github.com/guillermomolina/protos/commit/3e94ba7d01cf94852555d3daa18a3208d3908e35), `PERF037-D: profile captured owner-frame selection branches (#851)`.
- **Earlier D1 implementation evidence:** [published owner-frame branch profiling](PERF037_D1_PUBLISHED_OWNER_BRANCH_PROFILING_2026_10_09.md).
- **Prior exact C revision:** `d88ed6b83e977a1bf02f425825a015e9e577ec3e`; accepted graph has 151 nodes.
- **Benchmark checkout:** `guillermomolina/protos-benchmarks`; remote `main` inspected during coordination at [`6160b0a0808a2d9391748bf65eba4667c958cad9`](https://github.com/guillermomolina/protos-benchmarks/commit/6160b0a0808a2d9391748bf65eba4667c958cad9). The exact **local producer harness** revision is not visible in the pasted logs; do not infer it from the remote `main`.
- **Human-local directory (not published on GitHub as of checkpoint):** `results/perf037-d-3e94ba7d-graphs/`.
- **Capture environment:** benchmark devcontainer has product at `/workspaces/protos`; host uses Docker for `truffle/measure_graphs.py analyze`. Summarize/verify were run on host `~/Fuentes/protos-benchmarks`.

## Exact measurement output supplied by the maintainer

```text
ANALYZED=primitive-object-slot-read/protos/budget-16000/bgv/TruffleHotSpotCompilation-2330[ProtosSemanticBytecodeRootNodeGen@60ea6b82].bgv.gz
ANALYZED=primitive-object-slot-read/protos/budget-64000/bgv/TruffleHotSpotCompilation-2320[ProtosSemanticBytecodeRootNodeGen@1d1b25d2].bgv.gz
ANALYZE_FAILURES=0

UNIT=primitive-object-slot-read/protos valid=YES total=88 graphs=1
RUNG=primitive-object-slot-read protos=88 js=None python=None peer=UNRESOLVED protos_stabilization=STABLE protos_status=NOT_EVALUATED node_comparison=SKIPPED final_state=- signals=-
FIRST_DIVERGENT_RUNG=NONE
INVALID_UNITS=0

CASES=1
WORKING_TREE_MATCHES_PRODUCER=YES
HEAD_MATCHES_PRODUCER=YES
```

This confirms one selected **valid, stabilized Protos After-TruffleTier graph containing 88 nodes**, zero analyzer failures/invalid units and a benchmark working-tree/HEAD match to its producer. The command output alone does **not** independently display `unit.json.protos_revision`, `capture.json.product_before.clean` or the residual per-operation breakdown; these must be extracted from the preserved local metadata before asserting exact recorded product identity and class-level attribution.

The one-language matrix has `peer=UNRESOLVED` and `node_comparison=SKIPPED` because JS and Python were intentionally not captured; `FIRST_DIVERGENT_RUNG=NONE` here is **not** evidence of peer parity.

## Revision-bound structural progress

| Graph state | Protos nodes | Change from previous |
| --- | ---: | ---: |
| Before PERF037-B | 683 | — |
| After PERF037-B | 253 | −430 (−62.96%) |
| After PERF037-C | 151 | −102 (−40.32%) |
| **After PERF037-D1, current local evidence** | **88** | **−63 (−41.72%)** |
| Unchanged GraalJS peer reference | **36** | Reference target |
| Unchanged GraalPy peer reference | 103 | Secondary reference |

- **Total reduction from 683 to 88:** **−595 nodes (−87.12%)**.
- **Gap to the current owner-requested GraalJS 36-node target:** **52 additional nodes**; Protos still has **2.44x** GraalJS reference nodes.
- Compiled graph size is not runtime cost or time. No latency data have been supplied for D1.
- The D1 change was the owner-frame selection branch profiler; this is a **revision-level association**, not proof which 63 nodes were removed or which 88 survived.

## Work remaining and next exact evidence

**PERF037-D is still OPEN,** including the still-pending remainder of owner-selection/root/lowering work (D2–D5). There is no acceptance of 88 nodes as the final target.

Use **existing local**, already-analyzed `results/perf037-d-3e94ba7d-graphs/primitive-object-slot-read/protos/unit.json` and `capture.json` to report:

1. Exact `protos_revision`, `product_before.clean`, `evidence_valid`, stabilization.
2. `summary.families` and the graph `FrameState` count.
3. `units[0].attribution` sorted by descending `self.count` and `units[0].graph.node_class_histogram`.
4. Remaining `invoke_targets` and any loops, guards, deopts or allocations.

Do **not** rerun Protos capture nor unchanged GraalJS/GraalPy merely to obtain this attribution. Decide the next implementation within PERF037-D against those **actual residual nodes**, preserve BUG018/D179 and pay-as-you-grow. The original human-reported validation for the published source was clean `git diff --check` and all local tests PASS, with no coordinator-side tests executed.

Publishing the local D1 BGV/output corpus to `guillermomolina/protos-benchmarks` is a separate **human-executed** git action and has **not** happened as of this record.
