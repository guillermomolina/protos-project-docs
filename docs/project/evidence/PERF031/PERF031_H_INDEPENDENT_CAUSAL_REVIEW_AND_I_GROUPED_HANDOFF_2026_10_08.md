# PERF031-H — Independent causal review and grouped PERF031-I implementation handoff

**Date:** 2026-10-08  
**Issue:** [PERF031 / #787](https://github.com/guillermomolina/protos/issues/787)  
**Product authority:** [`guillermomolina/protos@a8027f6a78fac057f409103d8a79500faead72e2`](https://github.com/guillermomolina/protos/commit/a8027f6a78fac057f409103d8a79500faead72e2) (GitHub `main` at this independent source review)  
**Investigation:** `TYPE=INVESTIGATION`; no compilation, tests, benchmarking, runtime/CLI execution, product edits or publication  
**Decision:** The proposed immediate technical closure of PERF031 is **not supported** by the examined source. Recommend **one grouped implementation slice** addressing two related Test Tool case-count-scaled algorithms; keep the issue open.  
**Normative changes proposed:** None.  
**Performance gain measured:** None.

## Why the prior PERF031-H closure argument is insufficient

An external, source-only PERF031-H report proposed `OPTION_3=TECHNICAL_CLOSURE`, classifying the 23 historic >1 s Java classes as already structurally addressed or proportional. Its rationale asserted that the remaining spread-based ordinary-Array growth in Test Tool was bounded by the number of groups/descriptors and that large Case accumulators had already adopted binary-carry chunks. These assertions are **contradicted** by the exact current `Main.protos` and `CaseSelection.protos` source.

The report's 23-row classification also contains 10 rows marked `STRUCTURALLY_ADDRESSED` and 13 marked `PROPORTIONAL_INTEGRATION`, whereas its summary says 11 and 12. This count error is secondary; the actionable finding is the overlooked case-count-scaled repeated-copy work.

The prior JFR route tallies (`Core bootstrap` in 7/1,221, 18/1,635, 19/2,827, and 13/1,419 non-exclusive stack samples) are neither upper bounds on bootstrap wall time nor a per-operation timing attribution. Fast combined ContentIdentity test methods on one warmed serial run are legitimate evidence that certain bootstraps *can* be inexpensive, but they do not establish a universal 20 ms upper bound. The generated interpreter's `continueAt` share similarly cannot distinguish necessary guest work from avoidable guest-side collection algorithms.

The independent review does **not** conclude that bootstrap and existing B/C/F/G optimizations were wrong. Those releases eliminated real redundant structural work and remain authoritative. It also does **not** conclude that any measured class-time share is dominated by the newly identified spread loops.

## Finding H1 — Main.discover retains quadratic per-Case accumulation

Current product source: [`protos/tools/test/Main.protos`](https://github.com/guillermomolina/protos/blob/a8027f6a78fac057f409103d8a79500faead72e2/protos/tools/test/Main.protos).

Within `executionSuites.each`, the `retainedCases.each` body still performs all three operations by expanding and copying each preceding Array:

```protos
logicalCaseReferences = Array(...logicalCaseReferences, CaseRef.display(retainedCase))
logicalCaseMetadata = Array(...logicalCaseMetadata, metadata)
suiteProgressCaseKeys = Array(...suiteProgressCaseKeys, progressCaseKey)
```

- `logicalCaseReferences` and `logicalCaseMetadata` accumulate across the complete retained invocation.
- `suiteProgressCaseKeys` accumulates over retained Cases **within the current suite**, then its frozen result is placed in `logicalSuiteProgressCaseKeys` in retained suite order. PERF031-F correctly removed an additional per-suite global re-filter, but it did **not** eliminate these three repeated copies.
- `logicalProjections = Array(...logicalProjections, selectedProjection)` grows per retained **projection/source**, not per Case; examine its cardinality separately rather than conflating it with the three Case growth sites.
- `logicalExecutionSuites`, `logicalSuiteProgressCaseKeys`, and group-state/observer collections generally grow with suites/groups; their cardinality should be explicitly checked, not globally rewritten for style.

For an accumulator of N Cases, expanding the whole previous Array before appending one element copies at least N(N-1)/2 prior element references in the ordinary-Array construction path, in addition to other operations. At N=884 this is 390,286 prior-reference copies **per invocation-wide accumulator**. This is algorithmic counting, *not measured JVM allocations or duration*. Per-suite growth sums the corresponding triangular counts of individual suite sizes; there is no single global N² claim for that field when Cases are distributed across multiple suites.

`--list-cases` exits before progress-group creation and execution scheduling but **after discovery**. Thus discovery's repeated-copy costs still occur on list-only invocations; do not conflate these costs with PERF031-F's pure grouping optimization.

## Finding H2 — Public CaseSelection.listing also grows quadratically

Current source: [`protos/tools/test/CaseSelection.protos`](https://github.com/guillermomolina/protos/blob/a8027f6a78fac057f409103d8a79500faead72e2/protos/tools/test/CaseSelection.protos).

`listing(projections)` traverses every retained Case in authoritative projection order, builds its public `JSON.object("ref", JSON.string(CaseRef.ref(entry)), "display", JSON.string(CaseRef.display(entry)))`, and appends by:

```protos
cases = Array(...cases, JSON.object(...))
```

The list-output array therefore also copies its prior N-element prefix on each Case, totaling N(N-1)/2 prior-reference copies for N listed Cases.

This is not purely hypothetical unrelated source: the PERF031-D measurement recorded `ProtosTestToolTool012TomlOfficialProgressGroupingTest.directorySelectionPreservesTheCorpusCaseSet` at **7.096 s** while it invoked public `--list-cases --directory` for the TOML official corpus. However, those 7.096 seconds cover discovery/manifest/source reading, selector processing, JSON generation and public CLI startup; no portion is causally attributed to these copies. `CaseSelection.listing` also feeds exact-selection/listing public integration, which must retain all D185 identity, scope and rejection invariants.

## Existing compatible algorithm, no public Array redesign

The current Tool already uses a private **binary-carry chunk builder** in:

- `protos/tools/test/SuiteGraph.protos` — `flatten` produces stable leaf order.
- `protos/tools/test/Manifest.protos` — `loadWithParser` preserves parser row order and outputs frozen plan Cases.
- `protos/tools/test/Options.protos` — `projectArguments` preserves selected token order.

These designs accumulate singleton chunks in a `Map` by binary carry level, merging adjacent chunks and flattening remaining chunks in canonical order. They use existing `Array(...left, ...right)` and `Map` semantics; they do not require an append operation on ordinary Array. Their repeated-copy element work is expected to be bounded by roughly `O(N log N)`, versus `O(N²)` for repeated prefix copying, subject to existing implementation overhead.

Normative authority: `spec/semantics/VALUES_AND_COLLECTIONS.md` requires that ordinary Array construction returns a fresh Array and `Array.atPut` can replace existing indexed entries but **cannot grow/append** or create holes. An internal chunk collector must preserve element identity/order, frozen-final-array contracts, mutation boundaries and fail-closed errors. Do not invent Array capacity constructors, `add`, `atPut(size,...)` or a public Standard Library API as part of PERF031.

`protos/AGENTS.md` additionally requires the highest-level applicable existing collection capability to be reused; check `std:collections/Array` and the module's import/layering boundaries before choosing whether to reuse a local private routine or create **one** small Test Tool-internal collector shared by the two callers. Do not blindly duplicate a full collection subsystem.

## Current and historical evidence status

| Evidence | Observed data | Safe conclusion |
|---|---|---|
| PERF031-D, product `a463429f` | 548 classes / 2,811 tests / 0 failures / 0 errors / 1 skipped, accumulated class time 143.971 s | Historical serial cohort, not a post-F/G current cost profile |
| PERF031-D Tool012 full group test | 7.192 s | Complete-corpus ownership/group assertions are expensive; isolated grouping cost not established |
| PERF031-D Tool012 public `--list-cases --directory` | 7.096 s | Real list-only CLI path is material, but causality within it is unmeasured |
| PERF031-E JFR | Four class-specific fork profiles, 17/17 tests PASS; substantial `continueAt` activity | Interpreter activity exists; no per-guest-operation wall-time attribution |
| PERF031-F `e8678245` | One group assignment per suite and elimination of repeated global filtering | Correct structurally targeted optimization; retained per-Case Arrays still grow by spread |
| PERF031-G `c8a27d2a` | Shared immutable Prelude in Package Tool/Run Driver fixtures | Avoided bootstrap repeated construction; measured effect unknown |
| Current product `a8027f6a` | `Main.protos` and `CaseSelection.protos` repeated prefix copies still present | A bounded, correctness-preserving grouped implementation candidate is demonstrable |

There is insufficient source or measurement evidence to justify another unrelated Regex/P Future busy-wait rewrite, Package Tool/Run Driver refactor, parser/runtime overhaul, broad Truffle optimization, TEST008 budget adjustment or stress-test demotion. PERF032's trivial literal-return benchmarking remains separately owned.

## Decision and acceptance for PERF031-I

**SELECT=ONE_GROUPED_IMPLEMENTATION; REPO=guillermomolina/protos; ISSUE=#787.**

Implement one coherent package of case-count-scaled collection repairs:

1. `Main.protos`: replace the three `retainedCases.each` repeated-prefix Array growth chains with semantics-equivalent accumulation. Audit `logicalProjections` by projection count and change it **only if** same-cause/material work warrants it in this batch. Preserve per-suite partition, selection and metadata/CaseRef aligned order.
2. `CaseSelection.protos`: replace repeated-prefix Array growth of `JSON.object` Case listing nodes with same-cause accumulation, retaining exact D185 reference string, display order, schema `protos.test.cases/v1` and JSON output contract.
3. Add or refine **one coherent affected regression set** in existing Test Tool and public CLI tests, including complete official corpus, selected and list-only Cases, multiple suites, empty retained Cases, duplicate/malformed/out-of-scope requests, output equivalence and absence of body scheduling/progress during listing. Do not remove vectors/assertions.

Focal public/structural test families include `ProtosTestToolTool012TomlOfficialProgressGroupingTest`, `ProtosTestToolTool011ProgressGroupingTest`, `ProtosTestToolTool004CProgressPresentationTest`, `ProtosTestToolCaseSelectionPublicIntegrationTest`, `ProtosTestToolCaseRejectionPublicIntegrationTest`, `ProtosTestToolCaseExecutionPublicIntegrationTest`, `ProtosTestToolFileSelectionPublicIntegrationTest` and other direct consumers discovered from HEAD. Public CLI integration test files live under `src/test/java/com/guillermomolina/protos/cli/`, not the execution package.

Constrain the patch to the Test Tool and its affected tests. No specification changes, no Java runtime change, no class/Case reductions, no weakening scope/authority, no new formal Issue. The implementation is grouped because both source paths share the same Case-scale collection construction problem; it should not be split into separate tiny publications simply because the sources differ.

The implementation agent may edit source/tests in the user's product checkout but executes no build/test/Git operations. The HUMAN_EXECUTOR runs diff/static/focal/full impact-selected validation, finalizes version+changelog **after** the required gates are green and performs publication. Reconciliation must preserve concurrent HEAD work and never rerun already-green tests on unchanged candidate bytes by default.

**Success:** exact output/case-identity equivalence, safe bounded collection construction, all required owner-reported gates PASS, published product revision recorded later. A speedup must **not** be declared without compatible before/after measurement. PERF031/#787 stays OPEN/READY pending publication and final proportionality reconciliation; closure is not pre-approved.

## Materials inspected in this independent investigation

- `guillermomolina/protos/AGENTS.md`, `AGENTS.work/PERFORMANCE.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/COORDINATION.md`, `protos/AGENTS.md`.
- `spec/semantics/VALUES_AND_COLLECTIONS.md`, ordinary Array construction, bounds and no-append rule.
- `protos/tools/test/Main.protos`, `CaseSelection.protos`, `RepositorySuite.protos`, `SuiteGraph.protos`, `Manifest.protos`, `Options.protos`, `std:collections/Array`, `JSON` source; relevant Test Tool CLI/Tool012 test files.
- `guillermomolina/protos#787`, prior project-record PERF031-D/E/F/G evidence, and owner-reported full local test PASS for published slices.
- `guillermomolina/protos-project-docs/AGENTS.md` and the existing evidence directory.

**Provenance:** This independent source review and handoff were drafted with ChatGPT after direct read-only GitHub inspection; no new profiling or runtime/validation execution is claimed. The project owner explicitly approved *continuing with this grouped PERF031-I implementation plan*, not performance numbers or an untested implementation.
