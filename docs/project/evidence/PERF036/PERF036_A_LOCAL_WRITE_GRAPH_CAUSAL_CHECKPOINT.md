# PERF036-A — primitive-local-write structural-graph causal checkpoint

**Status: INVESTIGATION CHECKPOINT COMPLETE; NODE-LEVEL CAUSAL ATTRIBUTION PENDING.**  
**Date:** 2026-10-08  
**Issue:** [PERF036 / guillermomolina/protos#850](https://github.com/guillermomolina/protos/issues/850)  
**Next slice:** PERF036-B (investigation, without execution or implementation).

This is non-normative, revision-bound evidence. It neither redefines Protos semantics nor authorizes a production change, timing-performance claim, new Dxxx/PLATxxx decision, or PERF036 closure. The human-executor boundary is unchanged.

## Immutable sources and identity

- Product revision measured: [`997f633755574bab438897379229b75ace2bd733`](https://github.com/guillermomolina/protos/commit/997f633755574bab438897379229b75ace2bd733), `0.3.311-SNAPSHOT`, commit titled `PERF036-A: scalar-lane lowering for resolved local assignment`.
- Later product HEAD inspected during research: [`b750036119ca3af14732c658819b62b49039b350`](https://github.com/guillermomolina/protos/commit/b750036119ca3af14732c658819b62b49039b350). The compare from measured product to this HEAD showed no changes in the investigated Bytecode/lowering implementation files. This does **not** substitute current HEAD for the measured commit.
- Benchmark evidence publication: [`98abc9af7a05a45a7d4056b72f36aef5889cb76f`](https://github.com/guillermomolina/protos-benchmarks/commit/98abc9af7a05a45a7d4056b72f36aef5889cb76f), titled `PERF036-A: retain primitive-local-write graph evidence`.
- Benchmark producer HEAD recorded in the capture: `e221bb056a208693c9102891df2e13d65e61eefb`, `harness_dirty=true` due to untracked evidence/capture output during the run. Each `unit.json` and `capture.json` retains per-producer and per-workload source SHA-256 checksums; neither the floating HEAD nor the later publication commit is a claim of clean harness capture.
- GraalVM CE 25.4.4.1.1; Linux x86_64, AMD Ryzen 7 3700X, retained CPU-affinity metadata.
- Same workload `primitive-local-write` for all three languages; exact source and expected output from [`truffle/workloads/catalog.json`](https://github.com/guillermomolina/protos-benchmarks/blob/98abc9af7a05a45a7d4056b72f36aef5889cb76f/truffle/workloads/catalog.json).
- Authoritative selected phase: `After TruffleTier`, final tier 2; stable graph-signature pair at budgets 16,000 and 64,000; one relevant primary guest compilation unit in each language; correctness `PASS`, result `2`.

Unit evidence:
- [Protos unit.json](https://github.com/guillermomolina/protos-benchmarks/blob/98abc9af7a05a45a7d4056b72f36aef5889cb76f/results/perf036-a-local-write/primitive-local-write/protos/unit.json)
- [GraalJS unit.json](https://github.com/guillermomolina/protos-benchmarks/blob/98abc9af7a05a45a7d4056b72f36aef5889cb76f/results/perf036-a-local-write/primitive-local-write/js/unit.json)
- [GraalPy unit.json](https://github.com/guillermomolina/protos-benchmarks/blob/98abc9af7a05a45a7d4056b72f36aef5889cb76f/results/perf036-a-local-write/primitive-local-write/python/unit.json)

## Workload and semantic obligations

Exact Protos source:

```protos
run: () => {
    value: 1
    value = 2
    value
}
```

The GraalJS peer uses `let value = 1; value = 2; return value;` inside `function run()`; the GraalPy peer uses the corresponding function-local reassignment and return. The value observed is the post-assignment `2`, **not** merely the expression result of `value = 2`. The harness calls the prepared executable using the canonical Protos surface and the peer `executable-value` surface; exact `Value.execute()` units and entry labels are recorded.

Normative obligations reviewed: `spec/semantics/EXECUTION_AND_CONTROL.md`, `spec/semantics/OBJECT_MODEL.md`, `spec/semantics/CALLABLES.md`, and `spec/PROTOS_GRAMMAR.md`. A bare `:` creates a local slot, while `=` modifies an existing destination selected before RHS evaluation; it never implicitly creates one. Nearest lexical destination, presence versus guest `null`, capture by reference, frozen/closed state, error propagation, and no destination reselection after RHS must remain valid. A literal RHS in a non-escaping, unobserved straight-line root can justify specialization, **not** a benchmark-named semantic exception.

## Retained structural comparison

All counts are from the same selected `After TruffleTier` phase and reference-stage graph policy.

| Graph node class/family | Protos | GraalJS | GraalPy | Protos minus GraalJS |
| --- | ---: | ---: | ---: | ---: |
| **Total graph nodes** | **39** | **13** | **49** | **+26** |
| FrameState | 6 | 1 | 6 | +5 |
| TrufflePreserveFrameStateNode | 1 | 0 | 1 | +1 |
| VirtualArrayNode | 4 | 0 | 4 | +4 |
| VirtualInstanceNode | 1 | 0 | 1 | +1 |
| VirtualObjectState | 5 | 0 | 5 | +5 |
| ConstantNode | 13 | 4 | 16 | +9 |
| PiNode | 3 | 1 | 5 | +2 |
| LoadIndexedNode | 2 | 2 | 6 | 0 |
| BoxNode$AllocatingBoxNode | 0 | 1 | 1 | -1 |
| PiArrayNode | 1 | 1 | 1 | 0 |
| StartNode / ReturnNode / ParameterNode | 1 each | 1 each | 1 each | 0 |

The Protos graph has zero recorded `control_flow_splits`, `guards_deopts`, `invokes`, `loops`, and classified `allocations`, and exactly two classified `loads` (also two in GraalJS). Thus the **measured 26-node structural excess is not demonstrated to be a surviving dynamic lexical lookup, a generic assignment invocation, extra explicit guards, or extra indexed reads**.

The 16 extra frame-preservation/virtual-object representation nodes and nine extra constants are the highest-priority *suspects*, not 25 proven removable instructions. Virtual nodes are compiler IR representations, not proof of physical allocation. The peer GraalPy graph has the same 6 FrameStates, 4 VirtualArrays, 1 VirtualInstance, 5 VirtualObjectStates, and 1 TrufflePreserveFrameStateNode as Protos, suggesting (but not proving) a Bytecode DSL/frame-state representation cause.

A historical `primitive-local-read` graph in the PERF034 retention had the same Protos/Javascript 39/13 class distribution at an **earlier product revision**; it is informative about possible shared fixed-state costs but is **not** current same-revision evidence for that different workload.

## Code paths inspected and causal classification

- [`CanonicalToBytecodeLowerer.java`](https://github.com/guillermomolina/protos/blob/997f633755574bab438897379229b75ace2bd733/src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java): `scalarLocalNamesForRoot` and `assignsEstablishedScalar` admit already-established, same-scope, straight-line scalar assignment; `emitScalarAssign` emits `StoreLocal` under `IsCompactLocalFrame` and retains an authoritative materialized-activation fallback. `emitScalarLocalRead` handles direct final result. `requiresPersistentFrameAuthority` suppresses persistent authority for eligible non-escaping closure roots.
- [`ProtosSemanticBytecodeRootNode.java`](https://github.com/guillermomolina/protos/blob/997f633755574bab438897379229b75ace2bd733/src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java): generated Bytecode DSL root, `IsCompactLocalFrame`, source-instrumentation/root configuration. Its exact contribution to each surviving IR node has **not** been proved.
- [`ProtosFrameArguments.java`](https://github.com/guillermomolina/protos/blob/997f633755574bab438897379229b75ace2bd733/src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java): `isUnmaterializedCompactScalarLocalCall` discriminates the compact frame from an observed materialized activation. Its exact contribution to the residual IR node identities has **not** been proved.
- `CanonicalBindingAnalyzer.java` / `CanonicalBindingResolution.java`: distinguish static binding identity from runtime semantic presence; verified conceptually against D179.
- `ProtosPerf025CompactCalleeExecutionTest.java`: published focal regression source covers scalar assignment and rich/compact/context-observed fallback; its existence is **not** a new PASS execution report.

The scalar-write optimization was **already implemented** in the measured product. The zero-branch/zero-guard/no-invoke graph does not justify another speculative write fast path. No node-by-node BGV def-use/source attribution was completed by this checkpoint.

## Timing admission (not a speed claim)

The retained reference timing runs for `primitive-local-write` in Protos, GraalJS and GraalPy all report:

```text
measurement_valid=false
invalid_reason=NOT_STEADY_STATE_ADMITTED
admission.status=NOT_STABLE
```

They cannot support speed ratios, a measured Protos slowdown, or a performance gain claim. Graph stabilization and timing steady-state admission are separate gates. Sources: retained `results/global-20261008-timing/{protos-canonical,js-executable-value,python-executable-value}--primitive-local-write--reference-none/*invalid/summary.json` in the benchmark evidence publication, each with its exact run identifier.

## PERF036-B handoff and boundaries

**SLICE=PERF036-B; TIPO=INVESTIGACIÓN; no execution or repository mutation.** Read published `unit.json`, `capture.json`, and retained compressed BGV/filter JSON for the three languages at the same graph phase; identify actual node IDs, input/use chains, precise source and materialization/escape/deoptimization relationships; map each retained operation back to its Java class/method with evidence. Test alternative explanations (frame-state preservation, Bytecode root protocol, compact frame-argument representation, return-result boxing) against both peers, and separate unavoidable Java/Truffle compiler bookkeeping from Protos-owned removable costs.

Deliver a factual `node group -> exact BGV evidence -> Protos Java source location -> semantic obligation -> removable only if proven -> bounded proposed repair` table, plus compatibility/identity checks and a single grouped next implementation slice **only if** causal proof supports it. Otherwise report the precise missing proof and stop. Do not remeasure, re-run existing tests, reimplement PERF036-A, assume all 26 nodes are removable, or introduce a language design change.

**Agent:** investigate GitHub/published evidence read-only; do not run commands, tests, builds, benchmarks or Git operations; do not edit code/records or mutate issues in this investigation. **Human executor:** no action in PERF036-B. Only an independently authorized later implementation slice permits local product edits by agent and commands/tests/builds/commit/push by human. Before any subsequent product publication, respect HEAD/concurrency and the rule that `pom.xml` and `CHANGELOG.md` change only after human-reported green tests, immediately before commit/push, with no tests afterwards.

## Outcome

```text
PERF036_A_STRUCTURAL_INVESTIGATION=CHECKPOINT_COMPLETE
WORKLOAD_EXISTS=YES
EXPECTED_RESULT=2
GRAPH_CORRECTNESS_PROTOS_JS_PY=PASS
GRAPH_STABILIZATION_PROTOS_JS_PY=STABLE
GRAPH_PHASE=After TruffleTier
GRAPH_TIER=2
PROTOS_GRAPH_NODES=39
GRAALJS_GRAPH_NODES=13
GRAALPY_GRAPH_NODES=49
STRUCTURAL_DELTA_PROTOS_VS_JS=26
TIMING_ADMISSION_PROTOS_JS_PY=NOT_STABLE
MEASURED_SPEED_DIFFERENCE=NOT_ESTABLISHED
SURVIVING_NODE_CAUSAL_ATTRIBUTION=INCOMPLETE
FURTHER_OPTIMIZATION_AUTHORIZED=NO
NEXT_SLICE=PERF036-B_INVESTIGATION
PERF036_ISSUE_CLOSURE=NO
```
