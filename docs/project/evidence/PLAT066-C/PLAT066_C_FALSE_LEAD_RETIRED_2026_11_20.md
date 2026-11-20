# PLAT066-C — False-lead investigation retired by owner (2026-11-20)

**Role:** revision-aware, non-normative scope/closure evidence. This record preserves the project owner's explicit disposition, **not** a new technical proof, semantic ratification, performance benchmark, or claim that every former acceptance item was completed.

## Owner disposition

On **2026-11-20**, the project owner explicitly instructed: **"cierra PLAT066-C era una falsa pista"** ("close PLAT066-C; it was a false lead").

**Owning GitHub issue:** [PLAT066-C / #930](https://github.com/guillermomolina/protos/issues/930), titled `PLAT066-C — Residual InvokeClosureOneTail PE graph after I114`.

**Required classification:** `closed / not_planned`. The standalone hypothesis/research line about an independently actionable residual `InvokeClosureOneTail` physical-graph mechanism is abandoned. This is **not** `completed`, does not assert that the original exhaustive 429-node ledger and acceptance matrix were delivered, and does not retroactively invalidate independently tested I115 optimizations.

## Evidence, limits and separation of responsibilities

- The initial PLAT066-C material relied on old `TOP_LEVEL / identity` and `run → identity` graphs. Their node counts, compiler phases and source-revision caveats remain historical evidence. The current owner classifies continuation of that standalone line as a false lead; this is a **project direction**, not an independently reproduced quantitative refutation of each historical observation.
- [I115 / #931](https://github.com/guillermomolina/protos/issues/931) remains the implementation/validation owner for the GraalPy-shaped runtime and compiler migration. Closing PLAT066-C creates **no new issue** and imposes **no new independent acceptance gate** on I115.
- [PLAT066 / #924](https://github.com/guillermomolina/protos/issues/924) was already closed `not_planned` as redundant parent coordination, documented in [the 2026-11-20 parent retirement](../PLAT066/PLAT066_PARENT_SCOPE_RETIRED_TO_PLAT066_C_AND_I115_2026_11_20.md). The current decision supersedes that record's then-current statement that PLAT066-C remained open; historical claims are not silently rewritten.
- The selected historical [PLAT066-C terminal Closure-call decision](../../decisions/platform/PLAT066_C_GRAALPY_SHAPED_TERMINAL_CLOSURE_CALL_CONVERGENCE.md) and [PLAT060 B5+ amendment](../../decisions/platform/PLAT060_B5_UNIVERSAL_ORDINARY_CLOSURE_CALL_ABI.md) are **not rescinded by this lifecycle update**. Any normative/platform decision change requires its own explicit approval.
- The historical native Issue parent/sub-issue relationship for #930 has been reported as absent/unverified. This record does **not** assert that it was established or repaired merely through prose or issue closure.

## Published product checkpoint retained

Product [`guillermomolina/protos@2df251847ead94012f8d5a87c2b81d2d981882c7`](https://github.com/guillermomolina/protos/commit/2df251847ead94012f8d5a87c2b81d2d981882c7), version `0.4.186-SNAPSHOT`, remains published and unaffected.

[I115 0.4.186 physical-convergence evidence](../I115/I115_0_4_186_BUBBLE_SORT_PHYSICAL_CONVERGENCE_2026_11_20.md) records the maintainer's prepublication Bubble Sort selected `run` `After TruffleTier` counts: Protos Tier 1 **509**, Tier 2 **355**, against historical GraalPy structural targets **529/376**. Those are workload-specific graph counts, not a runtime-throughput claim or proof of every `InvokeClosureOneTail` path.

## Mutation/provenance limits

Only GitHub coordination and this durable documentation are in scope. There are no changes to `guillermomolina/protos` production sources, tests, `pom.xml`, changelog, issue identifiers or semantics; no new compilation/graph capture was executed for this closure. The closure grounds itself in the **explicit current owner instruction**, with historical and independent work left intact.
