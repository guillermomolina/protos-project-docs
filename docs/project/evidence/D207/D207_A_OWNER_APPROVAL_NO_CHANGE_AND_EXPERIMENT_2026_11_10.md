# D207-A — Owner approval, no-change invariant audit and experimental evidence (2026-11-10)

## Approval provenance

During a focused D207 review on 2026-11-10, the owner explained that D207 had been opened while the causes of slowness were unknown, and questioned the need to compromise Protos semantics. After the assistant explicitly recommended **“D207-A: conservar D013 sin cambios y cerrar D207 como decisión resuelta”** and requested approval of that exact candidate, the owner responded **“si apruebo”**.

This is explicit approval of **A — retain D013 without change**, not of experimental B direct invocation, C interceptors, a redesigned D mechanism or an unspecific language shortcut.

## Evidence inventory

| Source / observation | What it proves | What it does not prove |
| --- | --- | --- |
| [D207 issue #932](https://github.com/guillermomolina/protos/issues/932) | Original candidate set, semantic risks, research scope, experimental checkpoint | Full GITHUB010 research packet is complete |
| [D013 issue #171](https://github.com/guillermomolina/protos/issues/171) and `spec/semantics/CALLABLES.md` | Existing normative ordinary `call` lookup semantics remain ratified | That every implementation incurs zero lookup cost |
| [Experimental commit `61006b57c9434e4a4a904ee971ae77c846c68497`](https://github.com/guillermomolina/protos/commit/61006b57c9434e4a4a904ee971ae77c846c68497) | Human-reported isolated direct-Closure experiment was available and not merged | Any normative change or speedup is authorized |
| [I115-C/D publication evidence](../I115/I115_CD_385_FINAL_PUBLICATION_AND_PLAT066_C_RESEARCH_HANDOFF_2026_10_19.md) | Reported comparison: TOP_LEVEL Tier 1 `After TruffleTier` **645→645**, `InvokeClosureOneTail` ancestry **55→55**, and D013-preserving mainline I115-C/D | Universal lack of D013 cost, Tier 2 parity or end-to-end speed equivalence |
| [PERF042 #871](https://github.com/guillermomolina/protos/issues/871), [PLAT066-C #930](https://github.com/guillermomolina/protos/issues/930) | Independent investigations of lowerer/admission/fallback and graph costs | Approval to bypass late call-slot lookup |

## No-change delta/consistency audit (GITHUB021)

- **Before:** D013 ordinary mutable/delegating `call` lookup after argument evaluation, inherited and overridden slots, exact errors, receiver and method-home selection.
- **After D207-A:** **identical**. No syntax, Core, interoperability, object model, callback, Actor/Task/Context/ReturnHome, error/ensure or authority rule changes.
- **Product change required:** **none**.
- **Normative `spec/` change required:** **none**.
- **New I issue or benchmark prerequisite:** **none**.
- **Compatibility risk from ratification:** **none introduced**; ordinary existing invocation behavior retained.
- **Unresolved performance questions:** remain in the existing independent PERF042/PLAT066-C/I115 work. No measured speedup, throughput or zero-overhead theorem is asserted.

`DECISION_INVARIANT_CONSISTENCY=PASS` for an unchanged contract. Broad GITHUB010 comparison and 12-dimension scoring were **not** performed in this focused review; do not misstate them as completed.

## Durable outcome

The ratified language decision is [D207-A — retain D013](../../decisions/language/D207_A_KEEP_D013_ORDINARY_CALL_DISPATCH_RATIFIED_2026_11_10.md). [D207 #932](https://github.com/guillermomolina/protos/issues/932) is to be closed **completed** on the explicit approval and existing evidence; [D013 #171](https://github.com/guillermomolina/protos/issues/171) remains ratified. No new tests, product edits or version changes were executed as part of this record.
