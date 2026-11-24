# D262-A — Explicit reflective local slot creation and update

**State: OWNER-SELECTED / EXHAUSTIVE AUDIT AND NORMATIVE PUBLICATION PENDING.** This is a non-normative selection record, not a completed ratification or implementation report.

- Live decision: [D262 #1163](https://github.com/guillermomolina/protos/issues/1163).
- Owner approval: exact response "apruebo A", 2026-11-24, to the presented D262-A contract.
- Inspected product revision: [c98be4618fa5e8dc7a89f5092d3712616803e501](https://github.com/guillermomolina/protos/tree/c98be4618fa5e8dc7a89f5092d3712616803e501).
- [Comparative research, GITHUB021 review, provenance and remaining gates](https://github.com/guillermomolina/protos-project-docs/blob/b8ed47535bed22594fd11fb5ccd026ae1da070a6/docs/project/evidence/D262/D262_A_GITHUB010_GITHUB021_OWNER_SELECTION_AND_SOURCE_AUDIT_2026_11_24.md).

## Selected public contract

```protos
object.createSlot(name, value)
object.updateSlot(name, value)
```

Two ordinary inherited Object messages, independently overridable under normal lookup. Their selected standard behavior uses the same ordinary structural authority as explicit member creation and assignment.

Both take a semantic String name, compared by its exact Unicode scalar sequence, with no coercion, normalization, or source identifier restriction (including empty strings, reserved words and non-source-lexable names). Both take an ordinary value, including guest null and Closure, without copying, binding or invocation merely upon storage. On normal completion return the exact supplied value.

- createSlot: only an absent local slot on an open ordinary receiver; inherited names do not prevent creating a local override.
- updateSlot: only a present writable local slot on an open or closed ordinary receiver; never creates a slot or mutates a delegation ancestor.
- Frozen objects, non-admitted receiver domains, invalid name/argument association, duplicate creation and absent-local update signal the existing fresh generic Error; no new taxonomy. Preserve existing receiver-domain distinctions, including immutable semantic Numbers.
- Strict receiver/argument evaluation and ordinary dispatch/argument association under D237. Only selected standard behavior performs the structural checks after argument evaluation. No extra suspension, rollback, setter, conversion or read-back.
- slotValue/hasSlot/removeSlot and sorted fresh slotNames snapshots observe the same slots; alias/composition rules unchanged. No new grammar, AST category, indexed slot syntax, dynamic delegation, implicit upsert, mirror, or privileged selector.
- Stored Closures follow ordinary method binding and methodHome when reached through normal invocation. Creation/update of call, construct, init and reflection-selector names follows the ordinary semantics and cache invalidation rules.
- Execution contexts are included, not special-cased. Preserve D179 C0/E, I071, stable frame/cell layout, cleared presence, live lexical selection, original capture identity and single authoritative binding value. Names outside the compiled layout use the existing authority's supported dynamic extension; no new binding table.
- SOURCE and MODULE visibility follows the existing target object: context versus this. Frozen shared Object/prelude, Actor isolation, transfer and host interop remain unchanged. D261 remains independent.

## Observable compatibility delta

Two new ordinary local slots on the frozen standard root Object are visible to reflection and inherited lookup. Existing own overrides retain ordinary precedence. Static : and = retain exact semantics, including early bare-assignment destination selection. at/atPut remain separate. No standard Map/JSON/CBOR representation change is selected.

## Gate and scope

The owner selected A, not arbitrary future production changes. No conflicting owner-approved invariant identified in the initial GITHUB021 review; complete coverage and recheck before final ratification. Exhaustive whole-document and optimized-path audit incomplete as recorded in the evidence. The implementation owner must complete preflight before editing, reconcile normative owners, and add guest tests and performance evidence with human-executed validation. No product revision has been published for D262, no build/test result exists, and D262 must stay open until its required gates are satisfied. Any newly discovered substantive choice requires explicit owner approval; no FFM redesign or new Java infrastructure is authorized.
