# I115 — PERF backlog reconciliation after GraalPy-style Bytecode migration (2026-11-19)

**Role:** Revision-pinned, non-normative coordination and historical-evidence reconciliation. Live Issue status is authoritative in `guillermomolina/protos`; this document does not certify benchmark success, change language semantics, or supersede native GitHub Issue relationships.

## Scope and source identities

- **Maintainer instruction (2026-11-19):** try to close performance issues absorbed by I115, update the remaining issues, and publish durable evidence. This authorizes retiring redundant *independent work campaigns*, not treating unmet quantitative acceptance as PASS.
- **Inspected Protos default-branch head:** [`01c8566e6481501b051ee67d475e1d67accadb4a`](https://github.com/guillermomolina/protos/commit/01c8566e6481501b051ee67d475e1d67accadb4a), dated 2026-11-18 (I168 change). This is a source/repository-history checkpoint, **not** a new correctness, timing, Tier-2 or graph run.
- **Inspected benchmark-repository head:** [`51b1f3fa137f3db1ab2498afde320ca8242b8785`](https://github.com/guillermomolina/protos-benchmarks/commit/51b1f3fa137f3db1ab2498afde320ca8242b8785), dated 2026-11-18, containing a reference-results publication for product `09589f783ddf57f31cbd1cae5efcdc8410077cd0`. Its existence alone is not admission for any of the retired PERF workloads.
- **Principal published I115 physical changes**, independently identifiable in Git history:
  - [`6b18c67b12e04e3f91dc19bb488f0128a5e256d5`](https://github.com/guillermomolina/protos/commit/6b18c67b12e04e3f91dc19bb488f0128a5e256d5): migrate control, calls and lexical execution to GraalPy-style Truffle bytecode;
  - [`60b1b81b55925f81c799824b3f20375f3c2d64ec`](https://github.com/guillermomolina/protos/commit/60b1b81b55925f81c799824b3f20375f3c2d64ec): restore shape-backed object state and guarded attributes/indexing;
  - [`11a13538c37d65692f90783698d7b401063c08b1`](https://github.com/guillermomolina/protos/commit/11a13538c37d65692f90783698d7b401063c08b1): fuse numeric operations with internal selection and overloaded dispatch;
  - [`2b8032d19457a5c1fac0224dcc87125857b66ff1`](https://github.com/guillermomolina/protos/commit/2b8032d19457a5c1fac0224dcc87125857b66ff1) and [`dcf54500767ac56fa34696126140b98b92bd5e15`](https://github.com/guillermomolina/protos/commit/dcf54500767ac56fa34696126140b98b92bd5e15): native/inline `each` paths;
  - [`1089c9eb0ae76d4ae01afc07884a4b74b5df513d`](https://github.com/guillermomolina/protos/commit/1089c9eb0ae76d4ae01afc07884a4b74b5df513d): quickening, numeric/rich-comparison fallbacks and physical graph budget;
  - [`09589f783ddf57f31cbd1cae5efcdc8410077cd0`](https://github.com/guillermomolina/protos/commit/09589f783ddf57f31cbd1cae5efcdc8410077cd0): clear invocation temporaries and disable materialized-local-access machinery.
- **Prior issue-body and issue-comment evidence inspected:** all twelve then-open `family:PERF` issues below, plus [I115 #931](https://github.com/guillermomolina/protos/issues/931) and [PLAT066-C #930](https://github.com/guillermomolina/protos/issues/930). Original measurements and earlier acceptance gates are preserved in their own histories and previously published evidence records.
- No product, harness, source, benchmark, test or specification modifications were undertaken by this reconciliation. No tests, benchmark execution or graph capture are claimed.

## Retire the independent campaigns (GitHub `closed / not_planned`, **not** `completed`)

The retirement concerns the **outdated independent optimization/investigation campaign** and its presupposed pre-migration implementation path. It does **not** assert that the original performance target, Tier-2 parity, A/B comparison, clean provenance or every checkbox has been achieved.

| Issue | Why independent follow-up is superseded | Boundaries and historical evidence |
| --- | --- | --- |
| [PERF034 #845](https://github.com/guillermomolina/protos/issues/845), primitive local read | I115 replaces lexical lowering/frame-access machinery underlying the October 2026 investigation | Historic cross-language timing was **NOT_STABLE/invalid**; old 972-node source-era graph is not a current measurement. Future lexical-read regressions belong to current I115/PLAT066 graph attribution, not resuming old local-read patch steps |
| [PERF036 #850](https://github.com/guillermomolina/protos/issues/850), primitive local write | I115 migrates local-write/inline control execution and temporary-frame handling | Historical Tier-2 graph 39 Protos / 13 GraalJS / 49 GraalPy was valid structurally at old revisions; corresponding timing invalid. No current Protos-owned runtime cost or speedup is established |
| [PERF038 #852](https://github.com/guillermomolina/protos/issues/852), method call | Current I115 call-selection, argument/Task preparation and GraalPy-shaped dispatch replace the independently prescribed method-call implementation path | October method-call and H7 experiments are historical. There is no current same-workload stable A/B or end-to-end speedup assertion; new method-call regressions need exact current-source evidence |
| [PERF041 #863](https://github.com/guillermomolina/protos/issues/863), primitive `ifTrue` | I115's structured canonical Boolean-control lowering and inline callback execution replace the former standalone control optimization implementation | Earlier I095 and experimental I115-M/N observations are not a final current-HEAD benchmark comparison. Preserve ordinary/dynamic fallback, live binding, invalidation and nonlocal-return semantics |
| [PERF042 #871](https://github.com/guillermomolina/protos/issues/871), closure call | The closure-call physical convergence and graph work moved into I115 [#931](https://github.com/guillermomolina/protos/issues/931) and residual [PLAT066-C #930](https://github.com/guillermomolina/protos/issues/930) / [PLAT066 #924](https://github.com/guillermomolina/protos/issues/924) | Existing `run → identity` cold BGV and I115 reductions are **graph/footprint** evidence, not a measured guest throughput gain or full GraalPy equivalence. Pending graph causality stays with I115/PLAT066, without importing old PERF042 acceptance as accomplished |

**Closure classification:** `not_planned` for the former standalone program, not a duplicate issue number/parent claim and not an implementation-success claim. No original benchmark acceptance checkbox is retroactively marked PASS.

## Seven separately meaningful issues retained open

| Issue | Independent remainder not discharged by I115 |
| --- | --- |
| [PERF035 #846](https://github.com/guillermomolina/protos/issues/846) | Object-slot-write compiler stability: the 2026-10-08 Tier-2 stable pair at product `203c0f912be52db1b1adf4e528851e7622c41d95` had dirty-product provenance; clean current-runtime admission is not demonstrated |
| [PERF037 #851](https://github.com/guillermomolina/protos/issues/851) | Object-slot-read selection/semantics, prior graph gap and independent current-runtime physical read costs still need re-baselining after shape/bytecode migration |
| [PERF039 #858](https://github.com/guillermomolina/protos/issues/858) | Five three-language control workloads were published in benchmark commit [`175e5b2c4bf740164cdef1f02b40e2165b61afcd`](https://github.com/guillermomolina/protos-benchmarks/commit/175e5b2c4bf740164cdef1f02b40e2165b61afcd); the [published evidence](../PERF039/PERF039_A_PUBLISHED_CONTROL_FLOW_WORKLOADS_2026_10_09.md) explicitly lacks focal catalog/correctness/smoke/diff validation PASS required for `completed` |
| [PERF040 #862](https://github.com/guillermomolina/protos/issues/862) | Numeric specializations/fallbacks changed under PERF040 and I115, but complete numerical-family current acceptance and measurable hot-path costs are not proven |
| [PERF044 #883](https://github.com/guillermomolina/protos/issues/883) | `caseOf` / `match` guarded branch-tree lowering across ordinary, dynamic matchers is a distinct unproven optimization |
| [PERF045 #889](https://github.com/guillermomolina/protos/issues/889) | Native `each`/literal callback improvements are published, but composed Range / Set / IdentitySet / foreign iteration, fallback and evidence matrix remain independent |
| [PERF046 #890](https://github.com/guillermomolina/protos/issues/890) | Whole mixed-family `==` / `!=` / ordering conformance, Float/Fraction/Complex and exact comparative performance are not discharged by primitive rich-compare specialization |

**Revalidation order for retained work:** first prove exact current source and unchanged workload semantics; then graph/Tier-2 stability and admitted timing, if demanded; only afterward attribute a remaining cost and propose a new bounded improvement. Do not reuse old node-count targets as hard constraints, assume graph size is runtime speed, reopen retired source-level patches, or silently shift independently meaningful acceptance into I115.

## Evidence and governance limitations

- This is a **lifecycle/ownership reconciliation**, not a new measured optimization; no numerical speedup and no test PASS is inferred from the act of closing an issue or publishing this record.
- Original issue histories, benchmark artifacts and published historical records remain untouched. `not_planned` preserves the difference between superseded work and objectively completed work.
- I115 [#931](https://github.com/guillermomolina/protos/issues/931) remains open for its own current implementation/convergence scope; PLAT066 [#924](https://github.com/guillermomolina/protos/issues/924) and PLAT066-C [#930](https://github.com/guillermomolina/protos/issues/930) are not declared complete. Their GitHub-native parent/dependency state is not inferred from these cross-references.
- The historical benchmark evidence retains its recorded product/harness/GraalVM/host provenance. The five retirements do not grant a blanket waiver to benchmark-related release gates elsewhere.
- This record is posted under I115 because it reconciles a *shared implementation migration*, not as a new formal PERF issue or a live status mirror. Live closure decisions, when executed, are recorded separately in the affected GitHub Issues.
