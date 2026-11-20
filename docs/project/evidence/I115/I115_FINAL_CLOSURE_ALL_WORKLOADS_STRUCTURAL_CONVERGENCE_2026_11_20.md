# I115 — Final owner-authorized closure after 18-workload Protos structural campaign (2026-11-20)

**Status:** COMPLETED by explicit project-owner decision, 2026-11-20. **Type:** revision-bound, non-normative project evidence. This record does not change language semantics or ratified platform decisions.

## Authority and final product

- Formal issue: [I115 / guillermomolina/protos#931](https://github.com/guillermomolina/protos/issues/931).
- Owner closure instruction (2026-11-20): “ok ya tengo menos o casi igual de nodos que graalpy en todos los casos, quiero cerrar por fin I115. el git diff check esta limpio. todos los tests pasan en local.” The maintainer explicitly authorizes closure based on the completed campaign; the test and diff statuses are **owner-reported**, not independently rerun by the coordination agent.
- Final measured Protos source: [`guillermomolina/protos@2df251847ead94012f8d5a87c2b81d2d981882c7`](https://github.com/guillermomolina/protos/commit/2df251847ead94012f8d5a87c2b81d2d981882c7), version `0.4.186-SNAPSHOT`. This is the latest specifically measured and published I115 baseline, not an assertion about any later HEAD.
- Benchmark harness producer: `guillermomolina/protos-benchmarks@51b1f3fa137f3db1ab2498afde320ca8242b8785`.
- **Immutable publication of measurements:** [`guillermomolina/protos-benchmarks@88629ae`](https://github.com/guillermomolina/protos-benchmarks/commit/88629ae), `results/reference/protos-2df251847ead94012f8d5a87c2b81d2d981882c7/`. The versioned directory includes `README.md`, `manifest.json` (per-file SHA-256), raw correctness/timing evidence, Bytecode DSL capture logs, BGV/analyses and structural reports.
- Publication was performed by the maintainer: push reported `51b1f3f..88629ae main -> main`; separately verified from GitHub: the reference `manifest.json` and `README.md` are present.

## Completion evidence

| Admission / retained artifact | Evidence |
|---|---|
| Product correctness | 18/18 reference timing entries report correctness PASS; 54 independently retained correctness records |
| Time measurements | 18 Protos workloads on pinned runtime; cold/hot samples preserved, **not** graph metrics |
| Executed Bytecode DSL | 18/18 Protos diagnostic captures valid; 100 guest calls per workload |
| Graph compilation | Tier 1 18/18, Tier 2 18/18; requested root `all`; 0 unavailable observed compiled roots in each |
| Graph postprocessing | 36/36 cold summaries completed; 0 analysis and summary failures |
| Root pairing/reporting | 18/18 `structural-report.json` report `status=COMPLETE` and cross-tier pairing COMPLETE; 0 conflicts |
| Durable archive | 722 retained files, 30,680,924 bytes (~29.3 MiB); repository commit `88629ae` |

The published `manifest.json` names all 18 workloads and records the exact producer revisions and checksums. A valid root capture proves **observed compiled call targets only**, not coverage of interpreted-only roots; Bytecode DSL physical attribution is partial.

## Cross-language structural comparison: scope and findings

Peer **GraalPy 25.4.4.1.1** is **historical** I115-H6 evidence, not a new run on 2026-11-20, and runs on a different GraalVM line from the Protos 25.5.5.0.3 runner. Its published reference is [`results/experiments/i115-h6-2026-10-29/graalpy/nodes.csv`](https://github.com/guillermomolina/protos-benchmarks/blob/ca393f1/results/experiments/i115-h6-2026-10-29/graalpy/nodes.csv) plus the separately retained H6 `structural-reports.zip`. Comparisons use **After TruffleTier graph nodes**, never runtime latency or dynamic bytecode execution totals; host `Value.execute` and guest roots are distinct.

The 17 workloads common to H6 and the new 18-workload catalog were compared functionally root-by-root where the historical capture supports it. The final comparison discussed with the owner records:

| Guest workload / function | Protos T1 | GraalPy T1 | Protos T2 | GraalPy T2 |
|---|---:|---:|---:|---:|
| factorial / run | 161 | 161 | 312 | 407 |
| factorial / factorial | 189 | 216 | 376 | 568 |
| fibonacci / run | 161 | 158 | 1,148 | 1,334 |
| fibonacci / fibonacci | 233 | 231 | 1,407 | 1,531 |
| method-call / auxiliary closure | 107 | 109 | 50 | 52 |
| method-call / run | 244 | unavailable in admitted historical T1 | 94 | 119 |
| integer-loop / run | 189 | historically recovered only | 81 | 79 |
| primitive-return-literal / run | 105 | 106 | 48 | 49 |
| primitive-local-read / run | 105 | 106 | 48 | 49 |
| primitive-local-write / run | 105 | 106 | 48 | 49 |
| primitive-integer-add / run | 108 | 106 | 51 | 49 |
| primitive-object-slot-read / run | 125 | 162 | 68 | 103 |
| primitive-object-slot-write / run | 153 | 218 | 94 | 159 |
| primitive-closure-call / identity | 106 | 109 | 49 | 52 |
| primitive-closure-call / run | 156 | 162 | 48 | 50 |
| primitive-method-call / auxiliary closure | 107 | 109 | 50 | 52 |
| primitive-method-call / run | 158 | 181 | 60 | 82 |
| primitive-if-true / run | 105 | 106 | 48 | 49 |
| primitive-if-false / run | 105 | 106 | 48 | 49 |
| primitive-while-zero / run | 105 | 106 | 48 | 49 |
| primitive-while-once / run | 182 | 181 | 74 | 73 |
| primitive-while-counted / run | 183 | 182 | 75 | 74 |

**Bubble-sort** is the eighteenth workload absent from the historical H6 archive. The newly published guest run root is **502** Tier 1 and **348** Tier 2. Previously documented GraalPy *structural targets* for Bubble Sort were **529/376**. These are reference targets, not freshly paired H6 GraalPy root captures, and are not presented as such.

**Interpretation:** every common **Tier 2 guest root** is smaller or within +2 nodes of the historical GraalPy counterpart. The comparison does **not** prove same-toolchain binary parity, identical host ABI/adapters, identical inlining, equivalent performance, or a full 18/18 observed pair of GraalPy capture roots. Some old H6 Tier 1 observations are post-hoc recovered rather than originally admitted, and some roots have no admissible Tier 1 pairing. The graph counts are an acceptance checkpoint for the **I115 implementation scope**, not a universal efficiency theorem.

## Closing disposition and independent work

- **Close I115/#931 as COMPLETED** at owner's explicit direction. Final source implementation is published; the maintainer states all local tests PASS and `git diff --check` clean; benchmark and structural acceptance evidence is durably published.
- Historical I115-A through I115-H and later compilation/lexical/boolean/shape/storage/Native slices remain in the issue history; this final closure does not replace their detailed source-based evidence or alter D013, D179, B5/B5+, ReturnHome, Future, Error or Native semantics.
- [PLAT066/#924](https://github.com/guillermomolina/protos/issues/924) was retired independently; [PLAT066-C/#930](https://github.com/guillermomolina/protos/issues/930) was closed `NOT_PLANNED` as a false lead. Do not relabel that rejected investigation as completed.
- Independent PERF issues (e.g. PERF040 numeric hot paths, PERF042 Closure-call latency and further case-specific timing/causality) **are not implicitly completed** by structural convergence. Node count is not timing. Preserve their own acceptance gates.
- [PLAT069/#1048](https://github.com/guillermomolina/protos/issues/1048) was owner-paused until I115 finishes; this prerequisite is now met, but its issue explicitly requires **separate owner instruction to resume**. Therefore PLAT069 remains PAUSED pending a later direction, not automatically resumed or completed.
- No additional production changes, tests, compiler runs, new GraalPy captures, or semantic changes are claimed as part of this coordination action.

## Source of truth

Formal lifecycle: I115/#931. Benchmark provenance and raw evidence: `guillermomolina/protos-benchmarks@88629ae`, with per-file SHA-256 `manifest.json`. Earlier prepublication scope/physical reduction evidence: [I115 0.4.186 Bubble Sort checkpoint](I115_0_4_186_BUBBLE_SORT_PHYSICAL_CONVERGENCE_2026_11_20.md). This record is deliberately bounded to the final published campaign and owner-approved closure.
