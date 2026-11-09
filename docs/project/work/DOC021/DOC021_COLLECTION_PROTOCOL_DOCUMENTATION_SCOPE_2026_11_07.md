# DOC021 — Collection capability and protocol documentation scope (2026-11-07)

## Formal ownership and rationale

- **Issue:** [DOC021 #1089](https://github.com/guillermomolina/protos/issues/1089); `family:DOC`, OPEN / READY, assignee and scheduling priority intentionally unset.
- **Trigger / independent completed research:** [AUD023 #1087](https://github.com/guillermomolina/protos/issues/1087), [maintained analysis](../AUD023/AUD023_COLLECTION_PROTOCOL_COHERENCE_AUDIT.md), [owner-approved Alternative B evidence](../../evidence/AUD023/AUD023_B_OWNER_APPROVAL_AND_CLOSURE_2026_11_07.md).
- **Reference product baseline:** [`guillermomolina/protos@08c27760eb3f85105277e5b1b1b159abbca67ebe`](https://github.com/guillermomolina/protos/commit/08c27760eb3f85105277e5b1b1b159abbca67ebe); the implementation agent must inspect actual current HEAD and document only shipped, normatively supported behavior.
- **Role:** advance *maintained public documentation* for already-ratified semantics. DOC021 does not itself become a new normative authority or approve capabilities.

## Exact owner-approved direction

The owner explicitly approved B (“ok apruebo B”): **no changes to mutability, object families, public protocols, parser/indexing or iteration behavior; create a discoverable cross-family capability/contract map and focused examples.** D213's fixed-size Array, D214's fixed-shape numeric Vector/Matrix, existing Set Map-backing, `Range` traversal, foreign projection and Process Environment all remain authoritative.

DOC021 was allocated because this teaching work is independently schedulable, needs changes in multiple maintained guides and educational paths, and can close separately from the completed AUD023 investigation. It is **standalone**, not an inferred native child/dependency of AUD023, and not an `Ixxx` or `LIBxxx` implementation.

## Guide and example scope

1. `docs/guide/05-values-identity-equality-and-collections.md`: insert a compact reader-facing capability matrix and explanations for Array, Bytes, Map, IdentityMap, Set/IdentitySet roles, Range/InvertedRange, Vector/Matrix, String, Environment, optional foreign projections and exclusions (Path/StyledText are not general containers). Explain bracket lowering, mutation vs structural change vs new object, equality/key/member laws, `open/closed/frozen`.
2. `docs/guide/04-control-flow-through-protocols.md`: make the `for in` ↔ `each` relation and family-specific callback arguments/results/snapshot or pull semantics explicit. Explain why indexed String/Vector/Matrix do **not** currently support generic `for` through a standard `each`.
3. `docs/guide/library/linear-algebra.md`: only the minimal additional clarification required for algebraic `add`, fixed Vector/Matrix shapes and Matrix's `at(row,column)` without bracket sugar.
4. Focused valid examples in existing learning collections or contextual guide code, and appropriate `docs/guide/README.md`, `protos/tutorials/README.md`, `protos/examples/README.md` links, only as needed. No per-type file explosion.

## Exclusions and non-regression gate

- **No `spec/` or `src/` changes** and no modifications of `protos/lib/` implementations, parser, runtime, compiler or Standard Library behavior.
- **No new general `Collection`/Iterator/Stream**, universal `add` or `each` law, inserted/resized Array, implicitly iterable Vector/String, matrix bracket syntax, alternate Set runtime type or equality redesign.
- No claim that `Set` is an independent runtime type; no hidden transfer/snapshot semantics for foreign projections; no treating `closed` as immutable.
- Preserve direct normative ownership in `spec/semantics/VALUES_AND_COLLECTIONS.md`, `spec/PROTOS_GRAMMAR.md` and `spec/io/PROCESS_IO.md`, with current relevant source/test evidence recorded by AUD023. If an actual contract ambiguity emerges, record and route it; do not invent semantics in documentation.
- Existing documentation work [DOC016](https://github.com/guillermomolina/protos/issues/1032) and [DOC017](https://github.com/guillermomolina/protos/issues/1036) teach contextual `for` and Array operations; do not duplicate their entire lessons.

## Publication acceptance

The implementation executor must review current HEAD, surrounding guide content, source style and applicable norm owners; make localized updates; validate edited Markdown links, valid Protos syntax and any executable example using human-executed checks; then publish an exact product commit and report validation. No `pom.xml`/implementation `CHANGELOG.md` bump for docs-only changes. A separate completion record and exact revisions are required **when DOC021's product documentation is actually published**.

This file records **allocation/scope and approval provenance, not completion or executed tests**.

```text
DOC=DOC021
ISSUE=1089
STATUS=READY
AUDIT_TRIGGER=AUD023
OWNER_APPROVED_OPTION=B
DOC_PUBLIC_GUIDE_EDITS=NOT_STARTED
PROTOS_REVISION_AT_ALLOCATION=08c27760eb3f85105277e5b1b1b159abbca67ebe
TESTS_EXECUTED=NO
COMPLETION=NOT_CLAIMED
```


---

## Later owner-selected C and D252-C1 ratification — 2026-11-09

**Historical scope preserved:** The 2026-11-07 AUD023-B approval and this DOC021 allocation were valid when issued. The text above documents the older **pre-C1 product baseline**; do not rewrite that older approval as though C1 were known in advance.

**Subsequent direction:** the owner changed the future AUD023 outcome to **C** on 2026-11-09, then expressly approved **D252-C1** (“perfecto me gusta C1, apruebo”). [D252 #1098](https://github.com/guillermomolina/protos/issues/1098) is durably ratified; its [language decision](../../decisions/language/D252_COLLECTION_TRAVERSAL_STRING_VECTOR_MATRIX_C1_RATIFIED.md) defines standard `each` on String/Vector/Matrix. The implementation is separately tracked in [I162 #1099](https://github.com/guillermomolina/protos/issues/1099).

**DOC021 remains a separate documentation-only deliverable**, not implementation authority. Interpret the earlier “do not add `each`” as **do not introduce public behavior from a DOC-only edit**, rather than as a permanent ban overriding the later owner-approved D252-C1. Until I162 actually publishes `spec/` + code, current docs must accurately describe the still-unimplemented baseline, distinguishing the **approved future contract** from shipped capabilities. Once I162 publishes, align the broad family capability/contract matrix and examples with the published revision and avoid duplicating I162's directly affected guide edits. If independent DOC021 work precedes I162, do not present `for` on String/Vector/Matrix as currently executable.

No tests, product changes, DOC021 implementation completion, new priority or blocker are asserted by this project-record correction. DOC021 remains OPEN / READY; the C1 implementation is independently OPEN / READY.
