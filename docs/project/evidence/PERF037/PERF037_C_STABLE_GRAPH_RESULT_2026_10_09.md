# PERF037-C — Stabilized 151-node graph, human-executed local evidence (2026-10-09)

**Role:** revision-linked, non-normative benchmark checkpoint after the published PERF037-C optimization. This is **not** a durable publication of raw BGV files, and it is **not** end-to-end runtime speedup evidence.

- **Owning Issue:** [PERF037 / guillermomolina/protos#851](https://github.com/guillermomolina/protos/issues/851).
- **Published PERF037-C product revision intended for this capture:** [`d88ed6b83e977a1bf02f425825a015e9e577ec3e`](https://github.com/guillermomolina/protos/commit/d88ed6b83e977a1bf02f425825a015e9e577ec3e).
- **Existing implementation checkpoint:** [PERF037-C published source and tests](PERF037_C_PUBLISHED_IMPLEMENTATION_CHECKPOINT_2026_10_09.md).
- **Measurement instrument:** `guillermomolina/protos-benchmarks/truffle/measure_graphs.py`, `reference` stage, workload `primitive-object-slot-read`, `GRAPH_LANGUAGE=protos`, Truffle `After TruffleTier` graph selection. Benchmarks `main` is independently verified at `175e5b2c4bf740164cdef1f02b40e2165b61afcd` as of coordination; **this is not a claim that the local capture metadata identified this exact harness HEAD**.
- **Human-local output directory:** `results/perf037-c-d88ed6b8-graphs/`. Not published or downloaded by the coordinator. The path's revision-like suffix and current GitHub product HEAD alone do **not** prove the capture's internal `PRODUCT_REVISION`; exact identity and `PRODUCT_CLEAN` can be confirmed from its `unit.json`/`capture.json`.

## Human-provided capture, analysis and verification output

```text
ANALYZED=primitive-object-slot-read/protos/budget-16000/bgv/TruffleHotSpotCompilation-2347[ProtosSemanticBytecodeRootNodeGen@1d1b25d2].bgv.gz
ANALYZED=primitive-object-slot-read/protos/budget-64000/bgv/TruffleHotSpotCompilation-2309[ProtosSemanticBytecodeRootNodeGen@1ef04055].bgv.gz
ANALYZE_FAILURES=0

UNIT=primitive-object-slot-read/protos valid=YES total=151 graphs=1
RUNG=primitive-object-slot-read protos=151 js=None python=None peer=UNRESOLVED protos_stabilization=STABLE protos_status=NOT_EVALUATED node_comparison=SKIPPED final_state=- signals=-
FIRST_DIVERGENT_RUNG=NONE
INVALID_UNITS=0

CASES=1
WORKING_TREE_MATCHES_PRODUCER=YES
HEAD_MATCHES_PRODUCER=YES
```

Interpretation: the local capture achieved **stable accepted evidence** for exactly one selected Protos compiled graph, with zero analysis failures or invalid units. The `verify` matching statements certify benchmark-producer source correspondence; they do not substitute for checking the product revision inside the capture metadata.

The new single-language matrix reports `peer=UNRESOLVED`, `node_comparison=SKIPPED`, `FIRST_DIVERGENT_RUNG=NONE` because JS and Python were deliberately **not remeasured**, not because the new Protos graph has achieved parity. Reuse the previously published peers for a qualified external comparison.

## Structural progress across revisions

| Observation | Exact product revision | Compiled graph nodes | Change vs previous |
| --- | --- | ---: | ---: |
| Before PERF037-B | `d59da442fd9bc6cf590fb9b2c82d3ff55c0391ae` | 683 | — |
| After PERF037-B | `1c02b5d11c6e771910b22e213bc5fa76bc5315f7` | 253 | −430 (−62.96%) |
| After PERF037-C, local capture named `d88ed6b8` | `d88ed6b83e977a1bf02f425825a015e9e577ec3e` **expected; metadata confirmation pending** | **151** | **−102 (−40.32% from 253)** |
| Previous GraalJS peer | unchanged reference | 36 | reference only |
| Previous GraalPy peer | unchanged reference | 103 | reference only |

- **Cumulative Protos node reduction vs original:** `683 - 151 = 532` nodes, **77.89%**.
- **Residual to 90% reduction target:** `151 - 68 = 83` nodes to reach at most 68 (from 683).
- **Protos C vs peer reference sizes:** `151 / 36 = 4.19x` GraalJS, `151 / 103 = 1.47x` GraalPy.
- These are graph-size comparisons, not latency ratios. Cross-language status has **not** been formally recomputed in this one-language capture.

## Next bounded diagnostic, no rerun required

Read only the already-generated local `primitive-object-slot-read/protos/unit.json` and `capture.json` for:

1. Exact `protos_revision`, product `clean` state, recorded harness HEAD and producer hashes; validate expected revision.
2. Compiled graph family counts, especially `invokes`, `loops`, `control_flow_splits`, `loads`, and `FrameState`.
3. Per-target outgoing invokes and top `units[0].attribution` rows; do **not** infer all surviving graph edges run on the successful call path.
4. Only once exact remaining structural costs are attributed, choose between a further PERF037 implementation slice and timing/admission measurement. Do not repeat JS/Python or already-valid graph capture merely to get a full local matrix.

**PERF037/#851 remains OPEN.** Successful graph reduction does not independently establish stable post-C runtime latency or the complete issue acceptance criteria.
