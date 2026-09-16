# AUD009-A1 — Matching second-pass review contract

Status: **IN_PROGRESS**

Nature: non-normative retrospective audit correction and second-pass methodology

Parent: `AUD009` / `guillermomolina/protos#522`

Execution issue: `AUD009-A1` / `guillermomolina/protos#535`

Semantic decision authority: `D131` / `guillermomolina/protos#503`

Supersedes as current recommendation: `AUD009_A1_MATCHING_REVIEW.md`

Validation class: `GOVERNANCE_DOCUMENTATION_ONLY`

Specification changed: **NO**

Implementation changed: **NO**

Approved by project owner: 2026-09-16

## Why a second pass is required

The first AUD009-A1 packet correctly inventoried the current matching surface and gathered useful comparative evidence, but its retention recommendations gave too much weight to two arguments:

1. that a capability has precedent in mature languages; and
2. that its incremental implementation cost appears bounded once related matching machinery already exists.

Those facts are relevant evidence, but they do not satisfy the anti-overengineering gate in `AGENTS.md` by themselves.

The project owner explicitly rejected the implication that mature-language precedent is sufficient reason for a `KEEP` classification. The second pass therefore withdraws the first packet's feature classifications as the current recommendation while preserving its research as historical evidence.

## Correct interpretation of present need

`CURRENT_REAL_USE` must not be interpreted mechanically as "already used by existing Protos programs". Protos currently has too little production code for repository usage frequency to be a reliable proxy for language necessity.

The relevant question is instead:

> Is this a fundamental, recurring capability of general-purpose programming whose absence would predictably make ordinary programs substantially worse, or is it a specialized convenience/capability that can be added later when concrete evidence appears?

Long-established general programming needs and broad ecosystem evidence can establish present necessity even before a large Protos corpus exists. Conversely, a capability is not justified merely because one or more mature languages provide it.

## Mandatory second-pass burden of proof

Start from the smallest coherent matching model rather than from the already-implemented aggregate.

The provisional baseline to test is:

```text
- expression-valued multi-way selection (`match` / `case`)
- source-order arm selection
- ordinary value patterns through the open `pattern.match(subject)` authority
- inherited `Object.match(subject)` ordinary equality behavior
- basic binding (`@name`)
- wildcard/discard (`_`)
- basic fixed Array structural patterns
- basic open/subset Map structural patterns
```

This baseline is itself still subject to evidence, but it represents capabilities with an initially strong claim to general-purpose necessity.

Every capability beyond that baseline must independently answer all of the following.

### 1. FUNDAMENTAL_NEED

Is the capability broadly recurring in ordinary programming rather than primarily advanced/specialized convenience?

Evidence may include mature-language practice, common programming idioms, known classes of programs, and long-established programming experience. Lack of current Protos callers is weak evidence because the language corpus is young.

### 2. ORDINARY_PROTOS_ALTERNATIVE

Can the same practical requirement already be expressed reasonably through ordinary Protos objects, messages, Closures, matcher objects, Standard Library code, or duplication that is acceptable at this stage?

If yes, specialized Core syntax/semantics carries a higher burden of proof.

### 3. CORE_VS_SUGAR

Does the capability require a fundamental semantic mechanism, or could it be introduced later as pattern syntax / syntactic sugar / a library-level matcher over an already-stable core model?

A capability that can be layered later without changing the core authority model should normally not be preimplemented merely for completeness.

### 4. ADD_LATER_COST

If omitted now, what exactly must change to add it later?

A `KEEP` recommendation requires concrete evidence of material deferral cost when present necessity is otherwise weak: public semantic break, authority-model change, incompatible binding model, persisted representation, fundamental compiler/runtime boundary rewrite, or comparable migration cost.

"It already exists" and "we would need to implement it again" are not sufficient.

### 5. CURRENT_TOTAL_COST

Account for the complete ongoing cost, not only runtime lines of code:

- programmer mental model;
- syntax and grammar;
- semantic rules;
- parser and lowering;
- runtime execution;
- static validation/tooling;
- conformance/tests;
- documentation;
- cross-feature interaction rules;
- future feature drag.

A two-character syntax can still be expensive if it creates binding-equivalence, ordering, interaction or analysis rules.

### 6. PROTOS_FIT

Does the form reinforce Protos's ordinary-object/message/Closure philosophy, or introduce a special language institution where an ordinary mechanism could carry the capability?

This is especially important for guards: the **capability to condition a successful structural match** and the current special source form `case pattern when expression => ...` must be audited separately.

### 7. FUTURE_OPTION

If removed now, can the capability be reconsidered cleanly from the then-current language without reserving syntax or dormant implementation?

If yes, future usefulness is evidence for `REMOVE_NOW_RECONSIDER_LATER`, not automatically for `KEEP`.

## Important distinction: capability versus current spelling

AUD009-A1 must no longer assume that a useful capability validates its current syntax.

For example, matching may genuinely need a way to express a condition that depends on values obtained during structural matching. That does not by itself establish that the current Python-like:

```protos
case @x when x > 10 => ...
```

is the correct Protos form.

The second pass must therefore split, where applicable:

```text
CAPABILITY
CURRENT_SURFACE/SPELLING
CURRENT_INTERNAL_MECHANISM
```

and classify/recommend them independently when they can vary independently.

## Features requiring renewed scrutiny

The previous `KEEP` recommendation is explicitly withdrawn pending second-pass evidence for at least:

- guard capability;
- current `when` guard syntax and `=>` disambiguation institution;
- middle Array remainder;
- exact Map mode;
- Map remainder capture;
- whole-subject alias syntax;
- OR patterns;
- OR binding-interface equivalence rules;
- fixed opaque matcher `captures(a,b)` source exposure;
- tri-state coverage / structural diagnostics beyond the minimum needed for source correctness.

The existing recommendations to remove dynamic `captures(...rest)` and bare Map remainder discard remain useful evidence but are not considered ratified until the second pass reconciles the whole model.

## Second-pass per-feature record

For every reviewed feature, use:

```text
FEATURE
CAPABILITY
CURRENT_SURFACE

FUNDAMENTAL_NEED
ORDINARY_PROTOS_ALTERNATIVE
CORE_VS_SUGAR
CURRENT_TOTAL_COST
ADD_LATER_COST
PROTOS_FIT
EXTERNAL_PRECEDENT
CONFIDENCE

PROPOSED_OUTCOME
    KEEP |
    REMOVE_NOW_RECONSIDER_LATER |
    REMOVE_PERMANENTLY

IF_RECONSIDER_LATER:
    RECONSIDERATION_TRIGGER
    RECONSIDERATION_SCOPE

IF_PERMANENT:
    REJECTION_RATIONALE
    SUPERSEDING_MECHANISM
```

External precedent is deliberately one field rather than the selection rule.

## Required execution order

Re-audit in small groups so one feature does not justify another merely because both are already present:

1. foundational selection/matcher/binding model;
2. guards: capability first, current `when` surface separately;
3. Array patterns: fixed, terminal rest, middle rest, bare rest;
4. Map patterns: subset/open, exact, remainder capture, bare remainder;
5. aliases;
6. OR;
7. custom matcher extraction / `captures(...)` fixed and dynamic;
8. static coverage and diagnostics;
9. whole-model reconciliation.

## Decision boundary

This second-pass contract changes only the AUD009-A1 review methodology and current recommendation state.

It does **not**:

- change `spec/**`;
- change parser/runtime behavior;
- select replacement guard syntax;
- approve removal of any matching feature;
- ratify D131.

Each substantive semantic outcome still requires explicit project-owner approval through D131 or another applicable Dxxx authority.
