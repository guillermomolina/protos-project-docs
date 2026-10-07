# LIB014-0 — C5 + C1 owner-approval evidence

Status: **PARTIAL DESIGN SELECTION — IMPLEMENTATION NOT AUTHORIZED**

Owning work item: `guillermomolina/protos#431` — `LIB014 — Regular expressions and pattern-matching library`

Record role: immutable/snapshot-like approval evidence; **non-normative**

Approval date: **2026-10-06**

Protos revision audited by the LIB014-0 decision packet:

```text
PROTOS_REVISION=e1904bb01b61c6ee0da5fd279da0f7feb143229e
```

Project-record publication base:

```text
PROJECT_RECORD_BASE_REVISION=036a9ebfce52420067cb8f412281b28118a8e7e3
```

## Approval provenance

The completed LIB014-0 comparative decision packet recommended a hybrid direction:

- **Candidate 5** as the product/evolution strategy: a portable baseline plus separately named richer extensions;
- **Candidate 1** as the baseline contract family: a deliberately restricted portable regex language designed to preserve predictable/linear-time matching properties; and
- Candidate 3 only as a constrained possible implementation-freedom mechanism, subject to exact semantic equivalence.

In the active LIB014 interaction, the project owner explicitly approved:

> Apruebo c5+c1.

This record preserves that approval **exactly in scope**. It does not broaden the statement to include Candidate 3 or any other still-open contract detail.

```text
DECISION_APPROVAL_PROVENANCE=PASS
APPROVED_CANDIDATES=C5+C1
C3_APPROVED=NO
IMPLEMENTATION_AUTHORIZED=NO
```

## Approved invariants

The owner approval establishes the following durable design invariants for the active LIB014 decision:

1. The baseline public regex facility must follow the **Candidate 1** family: a restricted, portable regex language whose contract is selected to support predictable/linear-time matching rather than unrestricted PCRE/Perl-style backtracking semantics.
2. The product/evolution strategy must follow **Candidate 5**: richer constructs that are incompatible with the baseline guarantees are not silently added to the baseline; if later justified, they belong behind a separately named and explicitly contracted richer extension/profile.
3. LIB014 must not treat a rich PCRE/Perl-compatible language as the approved baseline merely because a host engine supports it.

The approval does **not** by itself settle the exact syntax/profile, Unicode contract, capture model, public API, exact complexity wording, engine architecture, host-engine delegation policy, replacement/split semantics, or integration with the Core `pattern.match(subject)` protocol.

## Invariant consistency

Before this approval:

- `guillermomolina/protos#431` had no recorded owner-approved LIB014 candidate;
- no durable LIB014 record existed in `guillermomolina/protos-project-docs`; and
- the LIB014-0 packet explicitly marked its recommendations as pending owner approval.

No earlier owner-approved LIB014 invariant was found that conflicts with the C5 + C1 selection.

```text
DECISION_INVARIANT_CONSISTENCY=PASS
REOPENED_PRIOR_INVARIANTS=NONE
```

## Remaining owner decisions before implementation

The C5 + C1 selection does not authorize implementation yet. The remaining exact public-contract decisions from the completed packet still need owner approval, including:

- matching selection semantics;
- Unicode/indexing/property/case-folding/newline contract;
- exact baseline syntax and exclusions;
- capture numbering/naming/nonparticipation semantics;
- Pattern/Match public API and immutability/ownership model;
- exact public complexity guarantees for single and repeated matching;
- engine/fallback/host-delegation freedom;
- zero-width iteration, replacement, and split semantics; and
- whether LIB014 baseline remains separate from the Core `pattern.match(subject)` protocol.

Current project policy also requires the substantive selected contract to complete the normal Dxxx decision/ratification path before dependent implementation is released.

```text
LIB014_0_RESEARCH=COMPLETE
LIB014_C5_C1_SELECTION=APPROVED
LIB014_REMAINING_CONTRACT=NEEDS_OWNER_DECISION
FORMAL_DXXX_RATIFICATION=PENDING
LIB014_IMPLEMENTATION_AUTHORIZED=NO
```

## Next-step boundary

No additional general comparative regex research is required by this evidence.

The next design step should consume the existing LIB014-0 packet and the C5 + C1 invariant, close the remaining exact contract choices in one bounded decision pass, and only then authorize the first implementation slice.

No source, specification, tests, package metadata, or Standard Library implementation changed as part of this approval record.

## AI-assistance disclosure

This approval-evidence record was materially prepared with AI assistance from ChatGPT from the completed LIB014-0 research packet, current project policy, live `guillermomolina/protos#431` state, and the project owner's explicit C5 + C1 approval. No independent human review is claimed by this record.
