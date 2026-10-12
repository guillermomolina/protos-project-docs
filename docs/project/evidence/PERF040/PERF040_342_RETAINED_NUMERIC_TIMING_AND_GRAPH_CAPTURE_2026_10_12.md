# PERF040 — Published 0.3.342 numeric timing and compiled graph evidence

Date: 2026-10-12
Issue: [PERF040/#862](https://github.com/guillermomolina/protos/issues/862)
Classification: non-normative post-implementation measurement checkpoint; **PERF040 remains open**.

## Revisions and source identity

```text
PRODUCT_REVISION=18f71b2a664c56ac808e2f508b6369984de02280
PRODUCT_VERSION=0.3.342-SNAPSHOT
IMMEDIATE_IMPLEMENTATION_PARENT=1fc6899bbc6e18359bd84d2357bd63e78523b53c
BENCHMARK_CAPTURE_HEAD=1725cd030d6a82167c49124d434944b7b364cd9b
BENCHMARK_EVIDENCE_PUBLISHED_REVISION=f68ba6e4e9ffca029904308315fa1c83f75b9da1
SURFACE=canonical
GRAPH_PHASE=After TruffleTier
REFERENCE_A_B_VERSUS_IMMEDIATE_PARENT=NOT_ESTABLISHED
```

Published [benchmark revision `f68ba6e4e9ffca029904308315fa1c83f75b9da1`](https://github.com/guillermomolina/protos-benchmarks/commit/f68ba6e4e9ffca029904308315fa1c83f75b9da1) retains the post-PERF040-A/B/C raw timing and graph artifacts. The final cleanup untracked an incomplete Fibonacci `.staging/` directory but preserved final evidence. At capture time, `harness_dirty=true`; maintainer-reported verification `HEAD_MATCHES_PRODUCER=YES` and `WORKING_TREE_MATCHES_PRODUCER=YES` support producer-tree consistency without misreporting the capture-time commit SHA.

## Accepted measurement and graph units

| Workload | Amortized steady p50 | Tier-2 IR nodes | Graph count | Stabilization |
| --- | ---: | ---: | ---: | --- |
| `primitive-integer-add` | **463.0944 ns/call** | **2,771** | 1 | `STABLE [16000,64000]` |
| `integer-loop` | **9,120,743 ns/call** | **28,500** | 1 | `STABLE [1000,4000]` |

Both timing units have `CORRECTNESS=PASS`, `STEADY_STATE_ADMISSION=PASS`, and `MEASUREMENT_VALID=YES`. Raw/derived paths:
- [Small Integer addition timing](https://github.com/guillermomolina/protos-benchmarks/blob/f68ba6e4e9ffca029904308315fa1c83f75b9da1/results/perf040-integer-ab/protos-canonical--primitive-integer-add--reference-none/20261012T154928391819Z/summary.json) and [graph](https://github.com/guillermomolina/protos-benchmarks/blob/f68ba6e4e9ffca029904308315fa1c83f75b9da1/results/perf040-342-graphs/primitive-integer-add/protos/unit.json).
- [Integer loop timing](https://github.com/guillermomolina/protos-benchmarks/blob/f68ba6e4e9ffca029904308315fa1c83f75b9da1/results/perf040-integer-ab/protos-canonical--integer-loop--reference-none/20261012T155002336261Z/summary.json) and [graph](https://github.com/guillermomolina/protos-benchmarks/blob/f68ba6e4e9ffca029904308315fa1c83f75b9da1/results/perf040-342-graphs/integer-loop/protos/unit.json).

The larger [PERF040-342 matrix](https://github.com/guillermomolina/protos-benchmarks/blob/f68ba6e4e9ffca029904308315fa1c83f75b9da1/results/perf040-342-graphs/matrix.json) holds **11 valid Protos-only graph units**, `INVALID_UNITS=0` and `ANALYZE_FAILURES=0` in human-run output. It also includes PERF032's `primitive-return-literal` root at 34 nodes. There are no newly captured JS/Python units: `peer=UNRESOLVED`, `node_comparison=SKIPPED`, and `FIRST_DIVERGENT_RUNG=NONE` cannot claim peer convergence.

The current `primitive-integer-add` IR still has one `IntegerAddExactNode`, one `IntegerAddExactOverflowNode`, 30 allocation-family nodes, 168 control-flow splits, 203 guard/deopt-family nodes and 2 invoke-family nodes. These counts describe a **compiled IR**, not dynamically executed allocations or proven per-call cost.

## Limits and routing

- Historical October 8 Protos `primitive-integer-add` (2,932 nodes) was captured at product `d59da442`, not immediate PERF040 parent `1fc6899b`. The decrease to 2,771 cannot be credited causally to PERF040-A/B/C.
- The directory name `perf040-integer-ab` does not establish a two-revision experiment: the new retained results contain only the `0.3.342-SNAPSHOT` product revision. No matching predecessor timing or allocation A/B was executed here.
- Historical peer cases can fold `1+2` away entirely in their selected graph. They are not an isolated cost comparator for nonconstant Integer arithmetic; [PERF043/#877](https://github.com/guillermomolina/protos/issues/877) investigates guarded constant folding separately.
- No further source, test, specification or harness change was made for this evidence publication, and the human reports clean `git diff --check` and local tests PASS without an independent rerun.

PERF040 remains under [#862](https://github.com/guillermomolina/protos/issues/862); this record is publication evidence, not full comparative/performance acceptance or closure.

This document was prepared with ChatGPT assistance using exact retained benchmark results and earlier published PERF040 evidence.
