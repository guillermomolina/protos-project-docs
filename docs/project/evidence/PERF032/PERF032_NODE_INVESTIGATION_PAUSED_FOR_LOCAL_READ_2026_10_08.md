# PERF032 — Owner-paused graph-node research pending local-read work (2026-10-08)

**Role:** dated non-normative workstream-routing record. This is a new checkpoint and does **not** rewrite the earlier [2026-10-08 PERF workstream reconciliation](PERF032_PERFORMANCE_WORKSTREAM_RECONCILIATION_2026_10_08.md), which accurately records its then-current owner decision.

**Live owner:** [PERF032 / protos#831](https://github.com/guillermomolina/protos/issues/831), `primitive-return-literal` JVM-cost investigation.

## New explicit owner direction

The project owner states: now that the `primitive-object-slot-write` compilation root converges, **suspend the graph-node investigation until after the local-slot/local-read problem is repaired**.

Routing interpretation:

1. PERF032's further investigations of the surviving graph nodes and related speculative reductions are **paused**. Leave the Issue **open**, use live `status:paused`, and preserve its preexisting scheduling Priority rather than interpreting a pause as completion or cancellation.
2. [PERF034 / protos#845](https://github.com/guillermomolina/protos/issues/845), the separately tracked `primitive-local-read` workload, remains the active local-read cost/repair owner; its measured PERF034-C stable graph is 103 nodes at published product `497bda9da781a558ab769445054935b787fab5f5`. Continue source-grounded work under its own acceptance criteria. A smaller compiled graph alone is not proof of a timing benefit or issue closure.
3. [PERF035 / protos#846](https://github.com/guillermomolina/protos/issues/846), `primitive-object-slot-write`, is independent: published source `203c0f912be52db1b1adf4e528851e7622c41d95` (`0.3.302-SNAPSHOT`) showed correctness PASS, stable Tier 2 pair [16000, 64000], and matching `After TruffleTier` graphs (1122 nodes). Its captured product checkout was **dirty** (`PRODUCT_CLEAN=False`), so clean-checkout reference acceptance and formal closure remain outstanding. See [PERF035 retained evidence](../PERF035/PERF035_PUBLISHED_SLOT_WRITE_TIER2_STABILIZATION_2026_10_08.md).
4. Do **not** infer a formal GitHub dependency edge, native sub-issue relation, transfer of PERF032 acceptance criteria or shared graph-node performance target. The pause is an **owner sequencing decision**, not a proven technical or normative block.

## Resumption gate and evidence hygiene

Resume the paused PERF032 node investigation only following the owner's stated local-read repair milestone, supported by PERF034's own source, correctness/graph and (where required) timing acceptance evidence. A later explicit owner decision may refine sequencing. On resumption, reassess current product/harness revision and existing captures; do not carry forward historical node counts as present-day comparisons without compatible measurement identity.

Prior PERF032-F and G6 measurements remain historical. Cross-language structure must be compared symmetrically, and `primitive-return-literal` performance must be established by same-workload evidence. This record authorizes **no** implementation, new tests/builds, benchmark policy change, version increment or semantics change.
