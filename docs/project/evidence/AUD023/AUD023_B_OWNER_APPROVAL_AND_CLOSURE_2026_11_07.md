# AUD023 — Exact owner approval of Alternative B and audit closure evidence (2026-11-07)

## Scope and authority

- **Formal audit:** [AUD023 #1087](https://github.com/guillermomolina/protos/issues/1087).
- **Independent documentation follow-up:** [DOC021 #1089](https://github.com/guillermomolina/protos/issues/1089).
- **Exact inspected product baseline:** [`guillermomolina/protos@08c27760eb3f85105277e5b1b1b159abbca67ebe`](https://github.com/guillermomolina/protos/commit/08c27760eb3f85105277e5b1b1b159abbca67ebe).
- **Prior published analysis/evidence:** `guillermomolina/protos-project-docs@0f79e40a9e0d2f1bb47ddebe486ce72704ebc184` (verified audit work record, source/test map, work and evidence indexes).
- **Maintained audit:** [AUD023 collection protocol coherence assessment](../../work/AUD023/AUD023_COLLECTION_PROTOCOL_COHERENCE_AUDIT.md).
- **Exact earlier evidence:** [AUD023 source/contract/test inventory](AUD023_REVISION_BOUND_COLLECTION_PROTOCOL_EVIDENCE_2026_11_07.md).

## Explicit owner instruction and selection

After the completed AUD023 assessment, the owner asked for a recommendation. The response proposed **“aprobar la alternativa B: coherencia de protocolos sin modificar la semántica de las colecciones”**, including retention of the old collection contracts, unified existing syntax, documentation of optional capabilities, no universal collection hierarchy, and deferral of possible `String`/`Vector` `each` changes.

The owner then stated, verbatim:

> ok apruebo B.

The same request explicitly authorized updating GitHub issues and publishing evidence with commit/push to **`guillermomolina/protos-project-docs`**. This proves approval of the exact scoped audit recommendation. It **does not** authorize an unrelated Dxxx/PLATxxx choice, modifying Protos's semantics, implementing a new `each` method, expanding `Array` in-place, or altering the ratified `add` contract of a Vector or Matrix.

## Decision / outcome reconciled to existing authority

**Selected: B — preserve all current observable semantics and make the actual capabilities explicit in maintained programmer documentation.**

- **KEEP:** Core fixed-length, dense, indexed Array with element replacement and fresh-copy algorithms; open/closed/frozen rules; growable Bytes; associative Map/IdentityMap; Map-backed Set and IdentitySet roles; computed frozen Ranges; fixed-shape numeric Vector/Matrix and their algebraic `add`; immutable String; read-only Process Environment; distinct foreign projections.
- **DOCUMENT:** `obj[i]` → `at(i)`, `obj[i]=v` → `atPut(i,v)`; generic `for(item in source)` → `source.each(item => {...})`; per-family optionality and callback/result/snapshot/order contracts; separate in-place mutators and fresh-copy operations; key equality/identity and no universal `add` contract.
- **DEFER:** new `String.each`/`Vector.each`, Matrix bracket syntax, any `Collection`/`Iterator`/`Sequence`/`Stream` hierarchy, a new set runtime family, new universal result/snapshot law, and changes to mutability. These are **not** implementation instructions.
- **Do not reopen:** [AUD009-B9 #620](https://github.com/guillermomolina/protos/issues/620), [AUD009-E1 #646](https://github.com/guillermomolina/protos/issues/646), [D213 #949](https://github.com/guillermomolina/protos/issues/949), [D214 #955](https://github.com/guillermomolina/protos/issues/955), and the D231/D235/D188/D192 invariants.

## GITHUB021-style invariant/delta gate

| Existing approved authority | B delta | Status |
|---|---|---|
| AUD009-B9 fixed Array and Map/IdentityMap | None: documentation restates existing contracts | PRESERVED |
| AUD009-E1 no general collection/iterator hierarchy | None: no new institution | PRESERVED |
| D213 Array fresh-copy API | None: no mutating Array growth or new selector | PRESERVED |
| D214 algebraic Vector/Matrix and fixed shape | None: no reinterpretation of `add` or multi-index syntax | PRESERVED |
| D231/D235 range and contextual for | None: `each` dispatch remains ordinary and family-owned | PRESERVED |
| D188/D192 foreign projection boundary | None: no invented Array/Map snapshot promise | PRESERVED |
| Process Environment distinct native-name semantics | None: no forced mapping-to-Map | PRESERVED |

**Outcome:** no known ratified invariant is contradicted by B. B is a **documentation/completion strategy**, not a new normative design ratification.

## Bounded publication and deferred acceptance

AUD023 has delivered the exhaustive comparative audit and revision-pinned evidence. The owner has accepted the B recommendation; this constitutes the independently closable **audit outcome**. The independently schedulable [DOC021](https://github.com/guillermomolina/protos/issues/1089) must still publish the public guides and examples in `guillermomolina/protos` before it can close.

This record confirms **documentation decision and coordination only**, not future DOC021 completion. No product files, tests, formatters, examples or runtime were edited/executed as part of AUD023 closure. The docs repository is the durable, **non-normative** home of this evidence. Live issue state belongs to GitHub.

```text
AUDIT=AUD023
ISSUE=1087
OWNER_SELECTION=B_APPROVED
APPROVAL_SCOPE=KEEP_SEMANTICS_DOCUMENT_CAPABILITIES
INVARIANT_DELTA=NONE
DOC_FOLLOW_UP=DOC021_1089_OPEN
PROTOS_REVISION=08c27760eb3f85105277e5b1b1b159abbca67ebe
PROTOS_PRODUCT_CHANGES=NONE
TESTS_EXECUTED_FOR_AUD023=NO
DXXX_SELECTION=NO
```
