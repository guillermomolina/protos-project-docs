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

## Post-capture identity and node attribution — human-extracted from preserved local files

This section supersedes the **pending product identity and residual attribution** caveats above. The maintainer read the existing `primitive-object-slot-read/protos/unit.json` and `capture.json` without recapturing:

```text
PRODUCT_REVISION=3e94ba7d01cf94852555d3daa18a3208d3908e35
PRODUCT_CLEAN=True
EVIDENCE_VALID=True
STABILIZATION=STABLE
NODES=88
FAMILIES={"allocations":0,"control_flow_splits":1,"guards_deopts":9,"invokes":0,"loads":6,"loops":0}
FRAME_STATES=11
SURVIVING_INVOKES=NONE
```

The exact expected product SHA is now independently confirmed from `unit.json`, and `capture.json.product_before.clean=True`. The selected graph has **no invokes, no loops, no allocations, one control-flow split, six loads, and eleven FrameState nodes**.

**Top direct source attribution** (self-count, not the full 88-node partition):

| Component | Self nodes |
| --- | ---: |
| `ProtosSemanticBytecodeRootNodeGen$CachedBytecodeNode` | 15 |
| `ProtosSemanticBytecodeRootNodeGen$SelectCapturedMaterializedOwnerFrameAtRoot_Node` | 10 |
| `ProtosSemanticBytecodeRootNodeGen$ReadMemberAtRoot_Node` | 5 |
| `ProtosSemanticBytecodeRootNodeGen` | 4 |
| `ProtosSemanticBytecodeRootNodeGen$UncachedBytecodeNode` | 3 |

**Full top-20 class histogram supplied by the maintainer:**

```text
ConstantNode=25
FrameState=11
FixedGuardNode=8
VirtualObjectState=7
PiNode=4
VirtualArrayNode=4
LoadIndexedNode=3
BeginNode=2
EndNode=2
InstanceOfNode=2
IntegerEqualsNode=2
IsNullNode=2
LoadFieldNode=2
ObjectEqualsNode=2
GuardedUnsafeLoadNode=1
IfNode=1
MergeNode=1
NarrowNode=1
ParameterNode=1
PiArrayNode=1
```

The sorted list is only the 20 most common classes, so it is not a complete 88-node histogram.

For the **historical unchanged JS 36-node reference** the comparator was read from the published benchmark [`results/global-20261008-graphs/primitive-object-slot-read/js/unit.json`](https://github.com/guillermomolina/protos-benchmarks/blob/6160b0a0808a2d9391748bf65eba4667c958cad9/results/global-20261008-graphs/primitive-object-slot-read/js/unit.json), itself recording `harness_git_head=98abc9af7a05a45a7d4056b72f36aef5889cb76f` and `STABLE`. That JS histogram has `ConstantNode=6`, `FrameState=1`, `FixedGuardNode=5`, `PiNode=6`, `VirtualObjectState=0`, `VirtualArrayNode=0`, `IfNode=0`, `MergeNode=0`, `LoadIndexedNode=4`, `LoadFieldNode=3`, `GuardedUnsafeLoadNode=1`, `BoxNode$AllocatingBoxNode=1`. The cross-language comparator is **historical**, not remeasured for D1; the local D1 matrix remains single-language and `peer=UNRESOLVED`.

| Selected node class | Protos D1 (88) | JS baseline (36) | Difference |
| --- | ---: | ---: | ---: |
| `ConstantNode` | 25 | 6 | +19 |
| `FrameState` | 11 | 1 | +10 |
| `VirtualObjectState` | 7 | 0 | +7 |
| `VirtualArrayNode` | 4 | 0 | +4 |
| `FixedGuardNode` | 8 | 5 | +3 |
| `IfNode` | 1 | 0 | +1 |
| `MergeNode` | 1 | 0 | +1 |

**Interpretation and decision boundary:** the excess concentrated in constants, frame states and virtual carriers points toward the bytecode lowering, local selection/conditional, and generated root compilation state as the next code-reduction targets. This is a candidate hypothesis from class counts, **not** proof that any particular node is semantically removable; some classes are also present at different counts in JS and not all node classes have runtime cost. The D1 optimized owner-select operation itself has only **10 self nodes**, down from its prior C attribution **48**, while root `CachedBytecodeNode` remains **15 self nodes**. The next bundled implementation should start from these measured residuals, keep BUG018-safe `LoadLocalMaterialized` (or demonstrably equivalent tier-correct access), and preserve D179 and generic fallback. The user's still-open structural target is **at most 36 compiled nodes**, with **52 nodes to eliminate from the 88-node result**.

**No D1 runtime latency, deoptimization-count timeline or raw BGV publication is claimed.** This update provides complete metadata and selected graph attribution, not additional benchmarks.
