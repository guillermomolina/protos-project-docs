# D187 — owner approval and ratification evidence

Status: **APPROVAL PROVENANCE RECORDED**

Formal decision: `guillermomolina/protos#809` — D187

Parent workstream: `guillermomolina/protos#431` — LIB014

Approval date: **2026-10-06**

## Approval chain

The project owner first approved the architectural family:

> Apruebo c5+c1.

That established:

```text
C5=APPROVED
C1=APPROVED
C3=NOT_YET_APPROVED_AT_THAT_POINT
```

A subsequent exact-contract closure bundle retained C5+C1 and supplied the remaining recommended semantics for matching, Unicode, syntax, captures, Pattern/Match values, API, complexity, host-engine freedom, reference/fallback machinery, zero-width progression, replacement, split, Core-matching separation, streaming and caching.

The project owner then answered:

> aprobado.

This second approval is the provenance for the exact D187 contract.

```text
DECISION_APPROVAL_PROVENANCE=PASS
EXACT_CONTRACT_APPROVED=YES
```

## C3 clarification

The exact approved contract does **not** make Candidate 3 a public compatibility family.

It approves only this bounded implementation-freedom rule:

```text
HOST_ENGINE_DELEGATION=OPTIONAL_INTERNAL_OPTIMIZATION
EXACT_PROTOS_EQUIVALENCE_REQUIRED=YES
PORTABLE_FALLBACK_REQUIRED=YES
```

Therefore the earlier C5+C1 approval remains intact and no host regex semantics become Protos authority.

## Invariant consistency

```text
C5_INVARIANT=PRESERVED
C1_INVARIANT=PRESERVED
UNRESTRICTED_PCRE_PERL_BASELINE=STILL_REJECTED
REOPENED_PRIOR_INVARIANTS=NONE
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Revision coupling

```text
PROTOS_REVISION=e1904bb01b61c6ee0da5fd279da0f7feb143229e
PROJECT_RECORD_BASE_REVISION=d23e87d06b343786e409899dc3f0778342ebd35c
```

The final project-record revision containing this evidence and the D187 decision record is recorded in the authoritative GitHub Issue after publication.

## Live hierarchy note

D187 was created with explicit textual parent declaration `Parent: #431`.

Project governance requires the native GitHub Parent/Sub-issue relation as the canonical live hierarchy. The connector used for publication did not expose a native sub-issue mutation action. The repository's Issue-intake workflow is expected to reconcile formal Issue parent declarations, but closure and dependent implementation must use a fresh live read and must not assume convergence.

```text
NATIVE_PARENT=VERIFY_BEFORE_CLOSURE
IMPLEMENTATION_AUTHORIZED=NO_UNTIL_D187_CLOSURE_POSTCONDITIONS_PASS
```

## AI-assistance disclosure

This evidence record was materially prepared with AI assistance from ChatGPT from the active owner-approval interaction, the LIB014-0 durable packet, and current Protos project governance. No independent human review is claimed.
