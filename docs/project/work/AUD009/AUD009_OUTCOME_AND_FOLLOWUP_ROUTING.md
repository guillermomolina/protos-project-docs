# AUD009 — Outcome and follow-up routing contract

Status: **APPROVED**

Nature: non-normative governance/work-routing evidence for AUD009

Approved by project owner: 2026-09-16

Tracking issue: `guillermomolina/protos#522` — AUD009 — Repository-wide complexity and necessity review

Publication base: `eccc1766a05f4f1907f1138a9a4fe733370bc27b`

Validation class: `GOVERNANCE_DOCUMENTATION_ONLY`

Specification changed: **NO**

Implementation changed: **NO**

## Purpose

Record the project-owner-approved interpretation of AUD009 outcomes and the routing rules that apply after a mechanism is classified.

This document does not add a fourth AUD009 classification and does not authorize any semantic or implementation change by itself. It clarifies how the already-approved retrospective vocabulary interacts with scheduling, specification decisions, implementation removal, and future reconsideration.

## Two independent axes

AUD009 must keep two questions separate.

### Axis 1 — Does the mechanism belong in current Protos?

Exactly one AUD009 outcome applies:

- `KEEP`
- `REMOVE_NOW_RECONSIDER_LATER`
- `REMOVE_PERMANENTLY`

### Axis 2 — What is the implementation scheduling state?

For mechanisms that remain part of current Protos, implementation may independently be:

- already implemented;
- pending implementation;
- `status:ready`;
- `status:blocked`;
- `status:paused`; or
- otherwise scheduled under the owning work family.

Scheduling state is not an AUD009 outcome.

In particular, `status:paused` and `REMOVE_NOW_RECONSIDER_LATER` are not synonyms.

## KEEP

`KEEP` means that the audited mechanism remains part of the current Protos model or current approved project architecture.

If the mechanism is already implemented, it remains implemented. Deferring later improvements does not justify de-implementing the already-retained mechanism.

If the mechanism is approved but not yet implemented, the owning work item may remain ready, blocked, paused, or otherwise scheduled normally. Deferring implementation does not remove the mechanism from current Protos.

A `KEEP` classification therefore answers the necessity question only. It does not imply immediate implementation work.

## REMOVE_NOW_RECONSIDER_LATER

`REMOVE_NOW_RECONSIDER_LATER` means that the mechanism does **not** belong in current Protos, but future evidence could justify revisiting substantially the same capability.

If the mechanism is already implemented, removal means real de-implementation within the approved scope. The project must not retain dormant implementation, compatibility behavior, parser productions, reserved syntax, runtime branches, hidden hooks, semantic placeholders, or other scaffolding merely to make later restoration easier.

The future record preserves the problem and reconsideration conditions, not the implementation.

For every such classification, the AUD009 ledger must record:

- `RECONSIDERATION_TRIGGER` — concrete future evidence or project state that should cause the capability to be reviewed again;
- `RECONSIDERATION_SCOPE` — the capability/problem to revisit, explicitly stating that the previous syntax, semantics, architecture, or implementation is not preselected.

Reconsideration means designing again from the then-current Protos model, evidence, specification, and architecture. A future solution may differ completely from the removed design.

AUD009 should not automatically allocate a future `Dxxx` merely to remember that reconsideration is possible. The durable AUD009 ledger is sufficient unless there is an actually unresolved design decision requiring active tracking or a useful domain work item that should remain visible as paused future work.

When a reconsideration trigger is later satisfied, the project routes the renewed question through the normal current authority: `Dxxx` for substantive implementation-independent semantics, `PLATxxx` for durable platform/runtime architecture, and the appropriate owning work family for implementation.

## REMOVE_PERMANENTLY

`REMOVE_PERMANENTLY` means that the mechanism or design direction should not return under substantially the same concept even if a related future requirement appears.

Use this outcome when the concept is intrinsically contrary to Protos philosophy, duplicates or is superseded by a better general mechanism, creates an undesirable semantic category, or is otherwise fundamentally inferior as the solution model.

If the mechanism is implemented, it is fully de-implemented within the approved scope under the same routing rules used for any substantive removal.

The durable record must preserve:

- the rejection rationale;
- the relevant conflicting principle, duplicated mechanism, or superseding model;
- enough evidence to prevent a future agent from accidentally rediscovering and reinstating the same rejected design direction as though it were new.

A future requirement in the same problem area may still be solved, but not by silently resurrecting the rejected concept. Reintroducing substantially the same model requires an explicit reopening of the prior rejection through the appropriate decision authority.

## Removal and de-implementation routing

AUD009 classifies and records evidence; it does not silently perform substantive removal.

When a recommended removal changes observable Protos semantics or normative specification behavior, route the semantic decision through the applicable `Dxxx` and normative specification process before dependent implementation work proceeds.

When a recommended removal changes a durable platform/runtime architecture while remaining semantically invisible, route the architecture decision through the applicable `PLATxxx`.

After the required design authority is ratified, de-implementation belongs to the work family that owns the affected implementation domain. Examples include:

- `Ixxx` for language/Core/runtime implementation work;
- `LIBxxx` for Standard Library work;
- `TOOLxxx` for bundled tools;
- `CLIxxx` for CLI work;
- `PERFxxx`, `DISTxxx`, `LMxxx`, or another existing owning family when that family naturally owns the implementation, migration, tests, and closure evidence.

Do not invent a generic `REMOVE`, `DEIMPL`, or equivalent family merely because work removes rather than adds behavior. Removal is implementation work owned by the same domain that would normally own the mechanism.

## Approved AUD009 ledger fields

Each audited mechanism should retain enough structure to make classification and follow-up unambiguous. The following fields form the approved minimum routing contract where applicable:

```text
FEATURE
CURRENT_AUTHORITY
CURRENT_IMPLEMENTATION
CURRENT_CONSUMERS

PROPOSED_OUTCOME
    KEEP |
    REMOVE_NOW_RECONSIDER_LATER |
    REMOVE_PERMANENTLY

IF_REMOVE:
    SEMANTIC_DECISION_OWNER
    IMPLEMENTATION_REMOVAL_OWNER

IF_RECONSIDER_LATER:
    RECONSIDERATION_TRIGGER
    RECONSIDERATION_SCOPE

IF_PERMANENT:
    REJECTION_RATIONALE
    SUPERSEDING_MECHANISM

STATUS
EVIDENCE
CONFIDENCE
```

Fields that do not apply may be omitted or explicitly marked not applicable, but the ledger must not lose the distinction between classification, approval authority, implementation owner, and future scheduling.

## Canonical interpretation table

| AUD009 outcome / state | Part of current Protos? | If already implemented | Future treatment |
|---|---:|---|---|
| `KEEP` | Yes | Keep implemented | Evolve normally |
| `KEEP` + paused future work | Yes | Do not de-implement | Resume when scheduling/prerequisites justify it |
| `REMOVE_NOW_RECONSIDER_LATER` | No | Fully de-implement after required approval | Preserve reconsideration trigger and capability scope only |
| `REMOVE_PERMANENTLY` | No | Fully de-implement after required approval | Preserve explicit rejection of the design direction |

## Governing distinction

The project therefore adopts this rule for AUD009:

> `status:paused` preserves approved work for later; `REMOVE_NOW_RECONSIDER_LATER` removes a capability from current Protos. They must never be used as synonyms.

And, for future-facing removal:

> Preserve the reason to reconsider the problem, not dormant machinery for the previous solution.

This contract governs subsequent AUD009 partitions and the final consolidated ledger unless the project owner explicitly revises it through the normal governance process.
