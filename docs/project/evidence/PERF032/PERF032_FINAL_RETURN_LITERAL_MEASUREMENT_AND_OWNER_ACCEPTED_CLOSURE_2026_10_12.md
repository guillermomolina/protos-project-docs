# PERF032 — Final return-literal measurement and owner-accepted closure

Date: 2026-10-12
Issue: [PERF032/#831](https://github.com/guillermomolina/protos/issues/831)
Classification: durable, non-normative benchmark and closure evidence.

## Exact identities

```text
WORK_ITEM=PERF032
WORKLOAD=primitive-return-literal
SURFACE=canonical
PRODUCT_REVISION=18f71b2a664c56ac808e2f508b6369984de02280
PRODUCT_VERSION=0.3.342-SNAPSHOT
BENCHMARK_PRODUCER_HEAD=1725cd030d6a82167c49124d434944b7b364cd9b
BENCHMARK_EVIDENCE_PUBLISHED_REVISION=f68ba6e4e9ffca029904308315fa1c83f75b9da1
CLOSURE=OWNER_ACCEPTED_COMPLETION
SPECIFICATION_CHANGE=NO
PRODUCT_PATCH_IN_THIS_CLOSURE=NONE
```

Raw evidence was first published in [benchmark commit `1a46a572`](https://github.com/guillermomolina/protos-benchmarks/commit/1a46a57257ecc3ce9fd2bca67d6a33db4e2321ce). The subsequent [`f68ba6e4e9ffca029904308315fa1c83f75b9da1`](https://github.com/guillermomolina/protos-benchmarks/commit/f68ba6e4e9ffca029904308315fa1c83f75b9da1) untracked an accidentally included incomplete Fibonacci `.staging/` tree; the final PERF032 artifacts are retained and unchanged. The measurement metadata records `harness_dirty=true` during execution, while human-run producer verification reported `WORKING_TREE_MATCHES_PRODUCER=YES` and `HEAD_MATCHES_PRODUCER=YES`; do not equate the capture-time harness HEAD with its later publication SHA.

## Final evidence

The unmodified source is `run: () => { 1 }`, invoked through the standard Polyglot `run.execute()` canonical surface. The user reports withdrawing an undesirable earlier primitive-`Long` return optimization; this record analyzes the exact pinned current product only and does not assume an isolated before/revert A/B.

- [Final selected BGV graph summary](https://github.com/guillermomolina/protos-benchmarks/blob/f68ba6e4e9ffca029904308315fa1c83f75b9da1/results/perf032-return-reverted-graphs/primitive-return-literal/protos/unit.json): `evidence_valid=true`, Tier 2 `After TruffleTier`, stabilization `STABLE` at `[16000,64000]`; **34 nodes and one selected guest graph**.
- The graph has zero invoke, load, loop, branch and guard/deopt family nodes. It has 11 `ConstantNode`, 6 `FrameState`, 5 `VirtualObjectState`, 4 `VirtualArrayNode`, 2 `VirtualInstanceNode`, 1 `AllocatedObjectNode`, 1 `CommitAllocationNode`, plus four entry/return/state nodes. The **two allocation-family nodes do not establish two physical allocations per call**.
- [Final timing summary](https://github.com/guillermomolina/protos-benchmarks/blob/f68ba6e4e9ffca029904308315fa1c83f75b9da1/results/perf032-return-reverted-timing/protos-canonical--primitive-return-literal--reference-none/20261012T165402527782Z/summary.json): `CORRECTNESS=PASS`, `STEADY_STATE_ADMISSION=PASS`, `MEASUREMENT_VALID=YES`, **81.596191 ns/call** median amortized steady time, reference policy of one million calls/sample, 60 warmup and 10 steady iterations.
- [Runtime and host metadata](https://github.com/guillermomolina/protos-benchmarks/blob/f68ba6e4e9ffca029904308315fa1c83f75b9da1/results/perf032-return-reverted-timing/protos-canonical--primitive-return-literal--reference-none/20261012T165402527782Z/metadata.json): GraalVM CE 25.4.4.1.1, JDK 25.0.4.1.1, AMD Ryzen 7 3700X, clean product state before/after.
- The separate [PERF040-342 capture](https://github.com/guillermomolina/protos-benchmarks/blob/f68ba6e4e9ffca029904308315fa1c83f75b9da1/results/perf040-342-graphs/primitive-return-literal/protos/unit.json) independently records **the identical 34-node graph class histogram at the same product SHA**, demonstrating structural reproducibility, **not** pre/post change improvement.
- Human-run results: `ANALYZE_FAILURES=0`, `INVALID_UNITS=0`, `HEAD_MATCHES_PRODUCER=YES`; maintainer reports local tests PASS and clean `git diff --check`. This record's publisher did not execute them.

## Historical boundaries

Historical October 8 `After TruffleTier` nodes were 39 for Protos at product `d59da442`, 13 for GraalJS, and 49 for GraalPy. The prior PERF032-G6 implementation had already reduced an older selected Protos graph from 282 to 47 nodes, and the later immutable NORMAL/DEBUG-mode entry improvement was separately published as `9b4a0750`. The latest **34-node** result is not attributable solely to removing `Long`, nor is it a current matched cross-language graph matrix.

Historical valid October 8 timing was 74.954 ns/call (Protos), 39.606 (GraalJS), 45.437 (GraalPy). The new 81.596 ns/call is **not a same-revision causal A/B** against the old primitive return or new peer timing. Parity of steady-state latency, compiler-phase causal attribution, and physical-allocation elimination **are not established**. This closure must not mark unfulfilled original funnel acceptance checkboxes as PASS.

## Explicit owner decision and routing

On 2026-10-12, the project owner stated satisfaction with PERF032's achieved outcome, requested closure and explicitly declined further work on this isolated case. Accordingly, close [PERF032/#831](https://github.com/guillermomolina/protos/issues/831) as **owner-accepted completion**, with the above limitations preserved. Independently scoped [PERF040/#862](https://github.com/guillermomolina/protos/issues/862) and [PERF043/#877](https://github.com/guillermomolina/protos/issues/877) remain under their own work items, without transferring unmet PERF032 goals.

This document was prepared with ChatGPT assistance from the retained benchmark corpus, existing published source/evidence, and the maintainer's reports. No tests, build or benchmark were rerun by the documenting agent.
