# PERF039-A — Published minimal control-flow workloads (2026-10-09)

**Role:** Revision-bound, non-normative benchmark implementation evidence. The live issue in `guillermomolina/protos` controls status and closure; this record does not define Protos semantics.

- **Issue:** [PERF039 / #858](https://github.com/guillermomolina/protos/issues/858).
- **Implementation repository:** `guillermomolina/protos-benchmarks`.
- **Verified published commit:** [`175e5b2c4bf740164cdef1f02b40e2165b61afcd`](https://github.com/guillermomolina/protos-benchmarks/commit/175e5b2c4bf740164cdef1f02b40e2165b61afcd), message `PERF039-A: add minimal ifTrue and whileTrue workloads`; inspected as `main` at evidence preparation.
- **Product repository:** `guillermomolina/protos` — no product implementation or specification files are changed by this benchmark commit.
- **Human report:** Maintainer stated that PERF039-A was pushed; the GitHub commit independently confirms repository-content publication.

## Published scope verified from commit and sources

The single published change adds **15 workload source files**: one Protos, one GraalJS `.mjs`, and one GraalPy `.py` for each of the following five names. The exact source paths are `truffle/workloads/<id>/<id>.{protos,mjs,py}`.

| Workload | Catalog expected integer | Logical condition evaluations | Body executions | Discriminator |
| --- | ---: | ---: | ---: | --- |
| `primitive-if-true` | 1 | 1 Boolean receiver selection | 1 | Selected lazy callback and propagated value |
| `primitive-if-false` | 1 | 1 Boolean receiver selection | 0 | Non-selected callback stays lazy |
| `primitive-while-zero` | 1 | 1 | 0 | Zero-iteration pre-test exit |
| `primitive-while-once` | 1 | 2 | 1 | One condition/body cycle and exit |
| `primitive-while-counted` | 8 | 9 | 8 | Eight body cycles, including captured local update, comparison and integer addition |

The counted case is a **composite** of loop control, lexical capture, local reads/writes, comparison, and addition; it is not a pure control-flow cost.

Other verified changed files:

- `truffle/workloads/catalog.json` adds five identifiers, their language paths, integer expected values and `jvm_sample_calls: 100`.
- `truffle/measure/cases.json` adds workload-scoped reference configurations matching the established primitive-case values: JFR `sample_calls: 1000000`; timing `warmup_iterations: 50`, `steady_iterations: 10`, `sample_calls: 10000`, `admission_scope: steady-only`. No global policy change is identified in this commit.
- `tests/test_measure_graphs.py` adds `test_control_flow_workloads_are_individual_and_outside_ladder`, asserting expected values, catalog/graph selectors, language-file existence, primitive policy matching and representative Protos syntax.
- `BENCHMARKING.md` explains the workload contracts, language-peer differences, selection and measurement policy.

The new workloads are individually addressable by the existing catalog and graph selection machinery; the `primitive` graph ladder in `truffle/measure/graphs.json` is **not** extended by this commit. No new harness, runtime specialization, benchmark-only privileged guest behavior or specification change is evidenced by the published changed-file set.

## Interpretation boundaries

Protos `Boolean.ifTrue` and `Object.whileTrue` remain **ordinary message sends with Closure arguments**. GraalJS and GraalPy peers use their native `if` / `while` forms: the inputs, expected values, branch/body decision and logical iteration counts are comparable, but lookup and Closure activation costs are not semantically identical across languages. Conditions known to be constant can be optimized away. The intent is to expose and attribute these differences, not to force equivalent node counts or claim whole-language performance.

The published JavaScript/Python source shapes and Protos sources were inspected for logical agreement with the five catalog entries. This is **static source review, not independently observed execution**.

## Validation provenance and pending acceptance

- **GitHub publication:** VERIFIED by exact commit and current benchmark `main` revision.
- **Changed-file scope / metadata / source presence:** VERIFIED by published commit inspection.
- **Catalog selector unit test:** test source is PUBLISHED; **execution PASS not evidenced** in this handoff.
- **Protos/JS/Python correctness:** **PASS not reported** for the five workloads in this handoff.
- **Focal smoke for the five cases:** **PASS not reported**.
- **Local `git diff --check` and other human-executor checks:** **PASS not explicitly reported**.
- **GitHub Actions/check runs:** no check runs/status results were returned for the published commit at evidence preparation; this is not proof of local test failure or success.
- **Reference timing / JFR / compiled graph BGV / node counts / speedups:** **NOT MEASURED or claimed** in this slice.

Consequently PERF039/#858 must remain **open** pending explicit focal validation evidence. Source publication alone does not discharge the correctness/smoke acceptance gates; do not mark all acceptance checkboxes complete or close the issue based solely on the pushed commit.

## Next acceptance step

The human executor should provide the already-performed focal unit-test, three-language correctness, smoke and diff-check PASS/FAIL summary tied to the published revision, or run the smallest missing gates without repeating completed checks. On verified admission, reconcile the remaining acceptance items in #858 and close if its original scope is complete. Compiled-graph comparisons, tier stability and timing are **follow-up performance investigations**, not requirements to validate this workload-definition publication.

**No new design decision, product optimization, performance conclusion, or further implementation slice is approved by this record.**
