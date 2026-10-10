# I092 — Published post-implementation Boolean graph comparison

**Owner:** [I092 / guillermomolina/protos#870](https://github.com/guillermomolina/protos/issues/870).  
**Causal reference:** [PERF041-A2](../PERF041/PERF041_A2_PRIMITIVE_IF_TRUE_BGV_TOPOLOGY.md), based on product [`0db24f00ff2d92d642351d7f7535517fe01a55ce`](https://github.com/guillermomolina/protos/commit/0db24f00ff2d92d642351d7f7535517fe01a55ce).  
**Implemented product:** [`7b609c4ad0e146d6f02a7a3d036c6ff938ccad08`](https://github.com/guillermomolina/protos/commit/7b609c4ad0e146d6f02a7a3d036c6ff938ccad08), `0.3.326-SNAPSHOT`.  
**Exact published benchmark-result revision:** [`8c91b495ac82ebe4a345ef02aa0db43fa5afe8c1`](https://github.com/guillermomolina/protos-benchmarks/commit/8c91b495ac82ebe4a345ef02aa0db43fa5afe8c1).  
**Capture-time harness HEAD:** `527035f01107e6c71f860e95f438b7553a5428a2`, as independently recorded in each `capture.json`/`unit.json`.  
**Result slot:** [`results/i092-7b609c4a-graphs/`](https://github.com/guillermomolina/protos-benchmarks/tree/8c91b495ac82ebe4a345ef02aa0db43fa5afe8c1/results/i092-7b609c4a-graphs).  
**Evidence class:** graph structure, **not** elapsed-time or per-invocation allocation measurement.  
**Status:** All three graph captures published, correct and stable. Full source-position/topology attribution of residuals remains open.

## 1. Measurement identity and admission

The published `matrix.json`, the three `capture.json`, and the three `unit.json` files have been read at the exact benchmark-result commit above.

- Language: Protos only (no peer remeasurement); canonical surface and `StructuredGraph / After TruffleTier`, compiler tier 2.
- Captures: `primitive-return-literal`, `primitive-if-false`, `primitive-if-true`.
- All three: `correctness.result=PASS`, `capture_valid=true`, `evidence_valid=true`, one primary guest graph, zero additional language-owned compiled units.
- All three: `stabilization.status=STABLE`, pair `[16000, 64000]`, `after_truffle_tier_confirmed=true`, selected budget `64000`.
- Existing reference budget policy: `[1000,4000,16000,64000,256000]`, unchanged.
- Captured JVM: GraalVM CE `25.4.4.1.1+1.1`, Java runtime `25.0.4.1.1+1-jvmci-25.4-b23`.
- Host record: Linux x86_64, AMD Ryzen 7 3700X 8-Core Processor, CPU affinity to logical CPU 0, captured inside a container.
- The metadata explicitly reports `harness_dirty=true` at capture time; do **not** mislabel the harness checkout as clean. The capture process retained producer source hashes and reported valid evidence. The later result-publication commit SHA is distinct from the recorded capture-time HEAD.
- `matrix.json` legitimately reports `PEER_REFERENCE=UNRESOLVED` and peer comparison `SKIPPED` because no JS/Python captures were requested. This does not invalidate the three Protos before/after comparisons, and no JS/Python parity conclusion follows.

## 2. Direct baseline versus I092 comparison

All numbers count nodes in the selected compiled graph at the same `After TruffleTier` phase. The historical source is PERF041-A2's published selected baseline graphs, not reconstructed measurements.

| Metric | Return before | Return after | False before | False after | True before | True after |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| **IR nodes** | **39** | **39** | **892** | **444** | **5,193** | **2,173** |
| `IfNode` | 0 | 0 | 38 | 31 | 298 | 127 |
| `FrameState` | 6 | 6 | 162 | 67 | 906 | 393 |
| `InvokeWithExceptionNode` | 0 | 0 | 9 | 0 | 47 | 9 |
| `AllocatedObjectNode` | 0 | 0 | 25 | 2 | 116 | 24 |
| `CommitAllocationNode` | 0 | 0 | 25 | 2 | 116 | 22 |

- `primitive-if-true`: **-3,020 nodes (-58.2%)**; `IfNode` **-171**; `FrameState` **-513**; invokes **-38**; `AllocatedObjectNode` **-92**.
- `primitive-if-false`: **-448 nodes (-50.2%)**; `FrameState` **-95**; invokes **-9**, leaving zero; `AllocatedObjectNode` **-23**.
- `primitive-return-literal`: no change; the 39-node baseline remains.
- True-minus-false node differential: **4,301 before, 1,729 after**, down by **2,572 (-59.8%)**.
- This is a graph-structure reduction, **not** proof of equivalent wall-clock speedup, material allocation count or per-call execution cost.
- Baseline IGV edge/block totals were 14,125/892 for true, 2,373/128 for false, and 56/1 for literal. New edge/block totals have **not** been recomputed from the retained filter exports in this comparison; do not fabricate them.

## 3. Observed retained call-target surface

The published `unit.json` for **`primitive-if-false` has zero invokes**. The `primitive-if-true` graph retains **nine `InvokeWithExceptionNode`** with recorded targets:

| Retained call target | Count |
| --- | ---: |
| `ImmutableCollections$List12.get` | 2 |
| `List.size` | 1 |
| `ProtosFrameArguments.materializeCompactActivation` | **1** |
| `ProtosInlineCallbackFrameBindings.transferFrameBindingsToDurableActivation` | 1 |
| `ProtosLanguageContext.bytecodeExecutionPlanForDefinitionMiss` | 1 |
| `ProtosLanguageContext.existingBytecodeExecutionPlan` | 1 |
| `Throwable.fillInStackTrace` | 2 |
| **Total** | **9** |

**Important limitation:** This list is from the selected graph's `invoke_targets` metadata and establishes **presence in the compiled graph only**. It does not establish normal-path dominance or execution frequency. The remaining `materializeCompactActivation` invocation must be located topologically (B0 versus conditional observer/cold fallback); **its existence alone cannot justify the claim that the original unconditional entry tax survived**, nor the claim that it was completely eliminated.

Similarly, the current `unit.json` has no disjoint Java-source-position classification comparable to PERF041-A2's origin buckets of 2,676 callback-preparation nodes, 304 outer-send nodes, and 184 structured-Boolean-preparation nodes. The large reductions are confirmed; attribution to those exact producers requires separate read-only inspection of the already-retained compressed filtered BGV exports and their Java source-position chains.

## 4. Conclusion and remaining review gate

I092's three-mechanism implementation has been published and its before/after compiled-graph benefit has been **measured and retained**; the literal baseline is unchanged and both selected/unselected Boolean cases have substantially smaller stable graphs. No new product or benchmark implementation is proposed by this record.

**Open attribution check:** inspect the published selected `primitive-if-true` and `primitive-if-false` `*.filter.json.gz` exports to classify the surviving activation invoke and the 2,173-node residual by reachable normal path, cold fallback, deoptimization state, and source-position producers. Preserve exact provenance and distinguish unknown from demonstrated causes. **No rerun is needed**: the required raw BGVs and filtered exports already exist under the published result slot.

Follow-up evidence should be appended as a new bounded evidence record (or direct issue comment) only after that read-only analysis; the implementation and benchmark result histories must remain unchanged.
