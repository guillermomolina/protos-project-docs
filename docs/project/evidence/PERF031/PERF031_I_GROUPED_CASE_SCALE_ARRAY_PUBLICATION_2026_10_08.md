# PERF031-I — Published grouped Case-scale Array accumulation

Date: 2026-10-08

## Exact identities

- **Formal Issue:** [PERF031 / #787](https://github.com/guillermomolina/protos/issues/787)
- **Product:** [`guillermomolina/protos@d0f8ea1c6f917dd1e0c16d7529d8b94e5310b039`](https://github.com/guillermomolina/protos/commit/d0f8ea1c6f917dd1e0c16d7529d8b94e5310b039)
- **Published version:** `0.3.297-SNAPSHOT`
- **Spec change:** none
- **Human-reported validation:** all local tests PASS; `git diff --check` clean
- **Independent agent-executed validation:** none; no logs/test counts provided with the handoff
- **Measured improvement:** none; no paired post-I timings or speedup claim
- **Issue lifecycle:** OPEN / READY while final class-by-class closure review remains

## Actual reviewed product commit

The exact published commit has six changed files:

| Path | Verified published change |
|---|---|
| `protos/tools/test/ArrayAccumulator.protos` | New tool-local binary-carry chunk collector; `create`, `append`, `finish` |
| `protos/tools/test/Main.protos` | Accumulate logical projections, invocation-wide metadata/references and suite-local progress Case keys; finish and freeze at the original materialization boundaries |
| `protos/tools/test/CaseSelection.protos` | Accumulate D185 public list node entries without repeated full-prefix Array copy |
| `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolCaseSelectionTest.java` | Multi-projection reversed-selector listing order, empty listing, and accumulator order/identity/independence regression cases |
| `pom.xml` | Product version `0.3.296-SNAPSHOT` → `0.3.297-SNAPSHOT` |
| `CHANGELOG.md` | Versioned description and explicit unmeasured-speedup disclaimer |

The accumulator uses the established `SuiteGraph`/`Manifest`/`Options` private binary-carry design: a `Map` of chunks indexed by merge level; equal-length chunks merge older-first; `finish` reads levels in descending order and produces a fresh ordinary Array. Each element is copied `O(log N)` times through the carries/finish rather than copying the full prefix on each append. The `finish` method does not freeze its result: consumers preserve their own original freeze responsibilities. No iteration order of Map entries defines presentation order.

**Discovery:** the `logicalCaseReferences`, `logicalCaseMetadata` and `suiteProgressCaseKeys` chains previously grew by one full-prefix spread on every retained Case; the first two span the invocation and the last is partitioned per suite. The commit replaces all three with independent accumulators; it also replaces the per-retained-projection `logicalProjections` growth. Per-suite progress keys are still finalized and frozen before joining the retained suite list. The final top-level arrays are still materialized and frozen after selection completion, so downstream execution metadata, group alignment and reporting continue to consume plain Arrays rather than exposing builders.

**Listing:** `CaseSelection.listing` now appends each `JSON.object(ref, display)` to its builder, materializes the ordered entries once, and passes the ordinary Array into the same `JSON.array(...cases)` and `JSON.encode` path. The public `protos.test.cases/v1` format, exact selected order, fail-closed selectors and list-only no-execution/progress contract remain the intended, previously tested public semantics. The changed test file adds source-backed checks for listing order across projections, reverse argument selection, empty output, and accumulator operation across carry boundaries. The maintainer confirms all local tests green; the documentation agent did not run those tests.

The untouched `logicalExecutionSuites`, `logicalSuiteProgressCaseKeys`, progress states and other smaller arrays still have their own separate suite/group cardinalities; no claim that all `Array(...prev, element)` in all Test Tool source has been removed is made.

## Evidence lineage and interpretation

1. [PERF031-D historic full Java cohort](PERF031_D_FULL_JAVA_COHORT_AND_CAUSAL_TRIAGE_2026_10_08.md): `protos@a463429f`, 548 classes / 2,811 tests / 0 failures / 0 errors / 1 skipped, 143.971 s class-time sum; this predates PERF031-F/G/I.
2. [PERF031-E JFR audit](PERF031_E_JFR_CAUSAL_AUDIT_AND_F_PUBLICATION_VERIFICATION_GATE_2026_10_08.md): 17 human-executed diagnostic tests PASS; Java stack samples do not attribute wall time to a particular guest Array operation.
3. [PERF031-F/G publication](PERF031_F_G_PUBLICATION_AND_GLOBAL_RESIDUAL_HANDOFF_2026_10_08.md): `e8678245` grouped progress assignment and `c8a27d2a` immutable fixture Prelude preparation.
4. [PERF031-H independent review](PERF031_H_INDEPENDENT_CAUSAL_REVIEW_AND_I_GROUPED_HANDOFF_2026_10_08.md): source-grounded contradiction to premature closure, identifying Case-count-scaled full-prefix array construction in Main and CaseSelection; that exact identified structural work is now addressed by I.

The original PERF031-H external 23-class disposition proposed technical closure but made an incorrect all-Arrays-group-bounded assertion and miscounted its categories (actual rows: 10 `STRUCTURALLY_ADDRESSED`, 13 `PROPORTIONAL_INTEGRATION`). This is historical evidence, not a current verified acceptance verdict. PERF031-I repairs the specific overlooked cost; other historical categories need final revision-bound acceptance reconciliation, not a new speculative optimization.

**Post-I speedup is unmeasured.** The algorithmic improvement is established by the published source structure, not by a stopwatch. The historic `ProtosTestToolTool012TomlOfficialProgressGroupingTest.directorySelectionPreservesTheCorpusCaseSet` 7.096 s was a complete public list-only invocation on the old baseline: it includes source/manifest discovery, CLI, JSON and environment work, so no fraction of it can be attributed to the removed prefix copies.

## Bounded next decision: PERF031-J final acceptance reconciliation

Next step is **one final read-only investigation / closure evidence gate**, not another source-code micro-slice. Using exact current product HEAD and this evidence lineage, reconcile the 23 historic >1 s Java classes to the formal #787 acceptance criteria. Distinguish:

- `STRUCTURALLY_ADDRESSED` by B/C/F/G/I with specific source evidence and retained regressions;
- `PROPORTIONAL_INTEGRATION` where separate Process, authority/custody, real public CLI, exact failure paths, and Regex P/Actor isolation genuinely prove the property;
- `INSUFFICIENT_EVIDENCE` where the observed samples or warm-JVM class method times do not support a claimed wall-time bound;
- `OTHER_OWNER` only with an exact existing Issue;
- any truly `MATERIAL_OPTIMIZATION_CANDIDATE` left demonstrably unresolved.

Select **closure** when each materially expensive class has an evidence-backed disposition and no material avoidable test work remains. If one contradiction survives, produce precisely one bounded acceptance blocker or decisive human-run diagnostic; do not invent incremental feature work merely to keep the Issue open. Under `AGENTS.work/COORDINATION.md` GITHUB020, closure requires a compact final Issue comment identifying product and durable-record exact revisions, confirmation that required publication is available and re-read, and live Issue state validation before closing. Until this occurs, #787 is OPEN / READY, `family:PERF`, priority unset, assignee unchanged; TEST008/PLAT047 and PERF032 remain independently owned.

## Authority and provenance

Inspected `guillermomolina/protos/AGENTS.md`, `AGENTS.work/PERFORMANCE.md`, `AGENTS.work/COORDINATION.md` closure gate, `protos/tools/test/Main.protos`, `CaseSelection.protos`, `ArrayAccumulator.protos`, the exact PERF031-I commit patch, and pre-I source/measurement evidence. The source constraints ultimately derive from `spec/semantics/VALUES_AND_COLLECTIONS.md`: ordinary Arrays cannot append via `atPut` and the replacement changes only internal accumulation. The maintainer owns product execution and publication; this project-record publication is a separately authorized documentation-repository action. Drafted with ChatGPT assistance; no human review or test execution by the recording agent is asserted.
