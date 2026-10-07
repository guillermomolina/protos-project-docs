# AUD009-A1 — Matching protocol direction checkpoint

Status: **PAUSED_PENDING_D130**

Nature: non-normative retrospective/design checkpoint

Parent: `AUD009` / `guillermomolina/protos#522`

Execution issue: `AUD009-A1` / `guillermomolina/protos#535`

Semantic decision authority: `D131` / `guillermomolina/protos#503`

Dependency for fair ergonomics comparison: `D130` / `guillermomolina/protos#502`

Validation class: `GOVERNANCE_DOCUMENTATION_ONLY`

Specification changed: **NO**

Implementation changed: **NO**

Checkpoint approved by project owner: 2026-09-16

## Why the matching audit is paused

The current `subject match { case ... }` surface had been treated during the initial AUD009-A1 review as the likely retained source form around the already-selected matcher protocol.

That assumption is withdrawn.

The project owner expected the dedicated `match` / `case` surface to be syntactic sugar over ordinary Protos mechanisms. The current implementation/spec instead gives matching a dedicated contextual grammar with its own arm/pattern sublanguage. That distinction is architecturally significant because Protos explicitly favors ordinary objects and messages over keywords and special syntactic constructs.

The next D131 pass must therefore answer a prior question before evaluating individual pattern-syntax conveniences:

> Does Core matching require dedicated language syntax at all, or can the complete fundamental mechanism be expressed through ordinary Protos objects, messages, Closures, Arrays and Errors?

## Current architectural direction to test

The strong candidate foundation is an ordinary protocol model built from existing language mechanisms:

- pattern-owned recognition through ordinary `pattern.match(subject)`;
- ordinary matcher objects rather than a closed Pattern class/registry;
- ordinary Closures as delayed selected-arm bodies and capture consumers;
- ordinary Arrays as aggregate data where an ordered collection is needed;
- ordinary Errors for terminal no-selection/failure policy where applicable;
- ordinary messages/combinators for higher-level matcher composition where practical.

A conceptual `caseOf`-style API is useful as a research model, inspired by Smalltalk/Self message/block control, but **no exact selector names or public API spellings are selected by this checkpoint**.

## Dedicated `match` / `case` syntax is no longer presumed fundamental

The following are now separate questions:

1. **Fundamental semantics/protocol** — what ordinary-object mechanism is required for matching to work?
2. **Ergonomic syntax** — after the protocol is proven, is any dedicated sugar needed?

If a future `match` / `case` or other compact surface is added, the preferred direction is that it be demonstrable as syntactic sugar with a clear lowering onto the already-complete ordinary protocol, rather than introducing an independent semantic institution.

It is valid for Core v0.1 to ship the protocol without dedicated match sugar if the ordinary surface proves adequate.

## Why D130 comes first

`D130 — Consider syntactic sugar for Array creation` evaluates pure Array construction sugar such as:

```protos
[a, b, f()]
```

lowering onto existing ordinary `Array(...)` construction semantics.

The matching protocol candidate will likely use ordered collections of cases/components. Evaluating that candidate while forcing verbose `Array(...)` spelling would bias the ergonomics comparison against the ordinary-protocol approach for a problem D130 is already intended to solve independently.

Therefore AUD009-A1/D131 is paused until D130 is resolved and, if approved, its Array sugar is available as the normal source baseline.

This is **not a semantic dependency**: matching does not require Array literal sugar to function. It is a sequencing dependency for a fair source-ergonomics evaluation.

## D130 boundary

D130 must remain strictly about Array construction syntax and desugaring. It must not decide:

- Array pattern syntax;
- `@name` binders;
- wildcard pattern syntax;
- match arms;
- matcher extraction semantics;
- `match` / `case` keywords or contextual spellings.

In particular, `[a, b]` in D130 is an ordinary Array-producing expression. Any future use of bracket syntax in a matching surface would require separate justification after the ordinary matching protocol is settled.

## Resume plan after D130

When D130 is complete, resume D131/AUD009-A1 in this order:

1. design the smallest complete ordinary matching protocol without dedicated matching grammar;
2. verify ordinary value cases and default/no-selection behavior;
3. verify bindings/extraction through ordinary Closure/callable mechanisms;
4. verify Array and Map structural matching requirements;
5. verify custom matcher composition;
6. verify conditional/refinement/guard requirements;
7. re-audit OR, aliases, rest/remainder, exact Map, custom captures and coverage against the ordinary-protocol model;
8. only then evaluate whether dedicated matching syntax is justified;
9. if sugar is justified, require an explicit lowering/desugaring story onto the ordinary protocol wherever semantics permit.

## Previously proposed Group 1 result

`AUD009_A1_GROUP1_FOUNDATIONAL_MATCHING.md` remains evidence, but its proposal to retain the current postfix `subject match { case ... }` envelope is **withdrawn pending this protocol-first review**.

The strongest surviving architectural evidence from that group is `pattern.match(subject)` / ordinary pattern-owned recognition. Other source-level forms, including `@name`, `_`, `case`, and the outer `match` envelope, must be reconsidered in the protocol-first model rather than retained by inertia.

## Resume trigger

`D130` is ratified/rejected and the resulting ordinary Array source baseline is known.

At that point AUD009-A1 returns to `IN_PROGRESS` and D131 resumes from this checkpoint rather than from the previous syntax-retention assumption.
