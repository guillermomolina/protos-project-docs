# PLAT066 — Parent investigation retired as separately tracked work continues (2026-11-20)

**Role:** non-normative, owner-directed GitHub Issue coordination and exact-source evidence. This is a **scope handoff / retirement of a duplicate open parent**, not a demonstration that the original exhaustive research acceptance was wholly completed.

## Owner direction and live work disposition

On 2026-11-20, after the I115 0.4.186 publication and a new Bubble Sort physical graph capture, the owner asks whether [PLAT066 / #924](https://github.com/guillermomolina/protos/issues/924) can now be closed. The correct closure classification is **`closed / not_planned` for further standalone parent work**, because the continuing work has existing independently owned Issues:

- [PLAT066-C / #930](https://github.com/guillermomolina/protos/issues/930) remains **OPEN** for the residual, source-to-graph `run → identity` / `InvokeClosureOneTail` physical-convergence investigation, fallback/ABI/Context/Task/ReturnHome proofs, and any further research gates. Historical native parent-link reconciliation is not asserted repaired.
- [I115 / #931](https://github.com/guillermomolina/protos/issues/931) remains **OPEN** for implementation and subsequent cross-workload/physical validation as independently specified.
- Independent numerical/mixed-family work, e.g. [PERF040 / #862](https://github.com/guillermomolina/protos/issues/862), is unaffected.

The parent PLAT066 **does not close these issues**, waive their independent acceptance gates, ratify a new platform design or change current normative semantics.

## Why closure is retirement, not original exhaustive completion

The 2026-10-18 original PLAT066 parent intake required an *exhaustive* source-and-graph mechanism ledger, transitive graph attribution, falsifying alternatives, concrete pseudocode, exact file plan, and owner disposition. Its 2026-10-20 and 2026-11-10 checkpoints explicitly stated these goals were not yet proven in full. The project subsequently selected a bounded architecture under the separately approved [PLAT066-C terminal Closure call decision](../../decisions/platform/PLAT066_C_GRAALPY_SHAPED_TERMINAL_CLOSURE_CALL_CONVERGENCE.md) and [PLAT060 B5+ addendum](../../decisions/platform/PLAT060_B5_UNIVERSAL_ORDINARY_CLOSURE_CALL_ABI.md), and implemented successive convergence slices under I115. D013 ordinary dynamic `call` semantics remain unchanged.

Consequently the parent no longer needs to remain an additional active research queue item, **but its former exhaustive acceptance matrix is not retroactively marked PASS**. Unresolved topics remain under the open PLAT066-C and I115 owners rather than being silently discarded.

## Exact published product and observed graph checkpoint

- **Protos product revision:** [`guillermomolina/protos@2df251847ead94012f8d5a87c2b81d2d981882c7`](https://github.com/guillermomolina/protos/commit/2df251847ead94012f8d5a87c2b81d2d981882c7), version `0.4.186-SNAPSHOT`, commit subject `I115: Cache Closure Preludes and lexical selections, profile indexed error paths and refresh the BCI PE baseline`.
- **Revision-pinned evidence:** [I115 0.4.186 Bubble Sort physical convergence](../I115/I115_0_4_186_BUBBLE_SORT_PHYSICAL_CONVERGENCE_2026_11_20.md); published and indexed at docs [`7071c61be5514686388cc340681df11288c5df7e`](https://github.com/guillermomolina/protos-project-docs/commit/7071c61be5514686388cc340681df11288c5df7e).
- **Maintainer-reported final prepublication graph capture:** Bubble Sort `run`, `After TruffleTier`: Tier 1 `643 → 509`, Tier 2 `489 → 355` (−134 physical nodes per tier), compared with historical GraalPy targets `529/376`. The last negative lexical-selection change alone reduced `551/397 → 509/355` (−42 per tier).
- **Maintainer attestation:** all local tests PASS and `git diff --check` clean after publication; earlier focused Maven and 58/58 harness-unit test results supplied. No independently repeated post-publication BGV capture, current same-commit GraalPy execution, or throughput result is claimed.

## Operational classification

| Issue | Lifecycle after coordination | Reason |
| --- | --- | --- |
| PLAT066 / #924 | **CLOSED / NOT_PLANNED** | Standalone parent investigation retired; residual work is explicitly owned by surviving issues |
| PLAT066-C / #930 | **OPEN** | Independent residual `InvokeClosureOneTail` physical/PE investigation |
| I115 / #931 | **OPEN** | Ongoing implementation/validation work |
| PERF040 / #862 | **OPEN** | Independent numeric-family acceptance |

The GitHub-native PLAT066-C parent relationship was historically reported missing and is not implicitly created by prose or this publication. No new Issue, change in priority, release milestone, version, code, benchmark or language/platform decision is made.

**Validation provenance:** read-only inspection of live Issues and exact Protos commit, plus prior maintainer terminal outputs. No tests, programs, builds or benchmark captures were executed while preparing this coordination record.
