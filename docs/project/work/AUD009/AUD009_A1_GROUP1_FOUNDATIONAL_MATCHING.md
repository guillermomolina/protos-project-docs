# AUD009-A1 — Group 1 foundational matching second-pass packet

Status: **NEEDS_USER_DECISION**

Nature: non-normative retrospective audit evidence and recommendation

Parent: `AUD009` / `guillermomolina/protos#522`

Execution issue: `AUD009-A1` / `guillermomolina/protos#535`

Semantic decision authority: `D131` / `guillermomolina/protos#503`

Methodology: `AUD009_A1_MATCHING_SECOND_PASS.md`

Validation class: `GOVERNANCE_DOCUMENTATION_ONLY`

Specification changed: **NO**

Implementation changed: **NO**

Audit date: 2026-09-16

## Scope

This packet re-audits only the foundational matching model. It deliberately does not decide guards, Array/Map advanced forms, aliases, OR patterns, custom matcher extraction syntax, or static coverage beyond source-correctness consequences of the foundational model.

The exact questions are whether Core should retain:

1. expression-valued multi-way `match` / `case` selection;
2. exactly-once subject evaluation, source-order arms and first-success commitment;
3. terminal no-selection behavior;
4. open ordinary `pattern.match(subject)` matcher authority;
5. inherited `Object.match(subject)` ordinary equality behavior;
6. basic binding `@name`, wildcard `_`, linear binding names and binding commitment only after complete pattern success;
7. the current matcher result carrier only to the extent required by this foundational subset.

## Governing burden of proof

The second pass does not ask whether these features are already implemented or whether other languages have them. It asks whether ordinary general-purpose programming predictably needs them, whether ordinary Protos mechanisms can replace them without making common programs materially worse, and whether the current form fits Protos's mechanism-over-institution philosophy.

External precedent is supporting evidence only.

## External reference points

Representative primary references considered for this group:

- Rust Reference, `match` expressions: https://doc.rust-lang.org/reference/expressions/match-expr.html
- Rust Reference, patterns: https://doc.rust-lang.org/reference/patterns.html
- Python 3.14 tutorial/reference, `match`: https://docs.python.org/3/tutorial/controlflow.html#match-statements
- Erlang expressions / `case`: https://www.erlang.org/doc/system/expressions.html
- Erlang pattern matching: https://www.erlang.org/doc/system/patterns.html

These systems differ substantially, but all provide strong evidence that multi-way pattern selection, source-order clause selection, bindings and a wildcard/catch-all are recurring general-purpose mechanisms rather than niche completeness features.

## F1 — expression-valued multi-way selection

### Capability

Select one branch from several alternatives by matching one evaluated subject, and return the selected branch result as the value of the complete expression.

### Current surface

```protos
value match {
    case 0 => "zero"
    case 1 => "one"
    case _ => "other"
}
```

The surface is postfix/receiver-first and uses contextual `match`/`case` spellings rather than adding global reserved words.

### FUNDAMENTAL_NEED

**HIGH.** Multi-way case selection is a recurring general-purpose programming need. Without a compact form, ordinary code must grow nested Boolean conditionals or ad-hoc dispatch structures for a problem that appears in parsers, protocol/state decoding, command/result handling, data shape handling and ordinary value branching.

This conclusion does not depend on existing Protos corpus size.

### ORDINARY_PROTOS_ALTERNATIVE

Boolean protocols and Closures could mechanically encode nested conditionals, but they do not provide a reasonable replacement for general multi-way pattern selection. Requiring users to reconstruct case analysis from nested `ifTrueIfFalse` calls would make a common task materially worse.

### CORE_VS_SUGAR

The exact spelling is syntax, but a first-class multi-way matching expression is a fundamental control/data-selection mechanism. Treating the entire construct as optional library sugar would either lose lexical bindings or require a larger callback/combinator institution.

### CURRENT_TOTAL_COST

The outer envelope adds one contextual expression form and arm syntax. That cost is significant enough to audit, but it is paid for a broad capability rather than a specialized edge case.

### ADD_LATER_COST

Deferring all matching would leave Core without a satisfactory multi-way selection mechanism and would encourage interim idioms/APIs that later matching would replace. This is not a capability whose absence is harmless until a niche use appears.

### PROTOS_FIT

Strong. The current postfix form is receiver/subject-first in reading order, while recognition itself remains delegated to ordinary pattern objects. `match` and `case` are contextual, not global reserved words.

### Proposed outcome

`KEEP`

This recommendation includes the current postfix expression-valued outer shape. No competing spelling demonstrates enough present benefit to justify redesigning a small coherent surface during this audit.

## F2 — exactly-once subject evaluation, source-order arms, first accepted arm

### Capability

Evaluate the subject once, consider arms in source order, and select the first successful arm.

### FUNDAMENTAL_NEED

**HIGH.** Any effectful/dynamic language needs deterministic evaluation and branch-selection rules. Protos matcher objects may execute ordinary code; unspecified order or repeated subject evaluation would make effects, Errors, suspension and control behavior unpredictable.

### ORDINARY_PROTOS_ALTERNATIVE

None that preserves the matching construct while avoiding an equivalent rule. Any multi-way selection mechanism needs a deterministic ordering/commitment contract.

### CORE_VS_SUGAR

Fundamental semantics, not sugar.

### CURRENT_TOTAL_COST

Low conceptual cost: it is the natural left-to-right rule visible in source.

### ADD_LATER_COST

High if omitted/left unspecified because user programs could accidentally depend on implementation order and later standardization could break observable effects.

### PROTOS_FIT

Strong: visible source order, ordinary evaluation, no hidden search/backtracking institution.

### Proposed outcome

`KEEP`

## F3 — terminal no-selection signals one fresh ordinary Error

### Capability

When no arm is selected, the expression fails rather than silently manufacturing a value.

### Current behavior

D092 signals one fresh ordinary `Error`; it introduces no `MatchFailure` prototype. Users who want a total/default result can write a catch-all `_` arm explicitly.

### FUNDAMENTAL_NEED

A match expression must define what value/control result exists when no arm matches. Because Protos permits open arbitrary matchers, general static exhaustiveness cannot always prove selection.

### ORDINARY_PROTOS_ALTERNATIVE

Credible alternatives are:

- implicit `null`;
- require an explicit syntactic catch-all in every match;
- introduce a dedicated `MatchFailure` category;
- current ordinary `Error`.

Implicit `null` hides an unhandled case and manufactures an ordinary value. Mandatory `_` adds noise to intentionally partial matches. A dedicated failure prototype adds an institution with no demonstrated need.

### CORE_VS_SUGAR

Fundamental terminal semantics.

### CURRENT_TOTAL_COST

Low. It reuses the existing ordinary Error mechanism and requires no public MatchFailure hierarchy.

### ADD_LATER_COST

Changing no-selection from one normal result/control behavior to another after programs depend on it is a semantic compatibility change.

### PROTOS_FIT

Strong: fail explicitly where the selection invariant is violated and reuse the existing Error mechanism rather than inventing a matching-specific failure universe.

Erlang is useful supporting precedent: a `case` with no matching clause produces a runtime `case_clause` error. Rust instead avoids the state through static exhaustiveness, a route that is not generally available to Protos's open arbitrary matcher model. Python silently does nothing when no case of its statement form matches, but Python `match` is a statement rather than an expression that must produce a Protos value.

### Proposed outcome

`KEEP`

## F4 — ordinary `pattern.match(subject)` is the single recognition authority

### Capability

Any ordinary Protos object can define recognition behavior by responding to one ordinary message.

### FUNDAMENTAL_NEED

**HIGH for the selected Protos matching model.** Once the language has pattern matching, an open prototype language needs a principled answer for user-defined recognition. A closed compiler-owned pattern taxonomy would privilege built-ins and make extensions depend on new language/compiler forms.

### ORDINARY_PROTOS_ALTERNATIVE

This *is* the ordinary-Protos alternative. Registration tables, host reflection, matcher base classes, hidden pattern objects, compiler plug-ins or subject-side special deconstruction registries are all larger institutions.

### CORE_VS_SUGAR

Fundamental semantic authority.

### CURRENT_TOTAL_COST

One public selector and the requirement that matching respect ordinary lookup/invocation/control semantics. That is real runtime/semantic responsibility, but it composes with mechanisms Protos already owns.

### ADD_LATER_COST

High. Choosing a closed or subject-owned authority now and later moving recognition to ordinary pattern objects would change ownership, extension points, dispatch and potentially all structural/composite matching semantics.

### PROTOS_FIT

Very strong: ordinary objects, ordinary messages, no privileged matcher class/registry.

### Proposed outcome

`KEEP`

## F5 — inherited `Object.match(subject)` delegates ordinary values to `this == subject`

### Capability

Every ordinary value can act as a zero-capture value pattern without a wrapper or literal-specific matching relation.

### FUNDAMENTAL_NEED

**HIGH.** Value cases such as numbers, strings, booleans and arbitrary existing values are the most basic case-selection use. They should not require explicit matcher wrapper objects.

### ORDINARY_PROTOS_ALTERNATIVE

Compiler-special-case every literal/value family, introduce a ValuePattern wrapper, or define another equality-like operator. All are less ordinary and enlarge the universe.

### CORE_VS_SUGAR

Fundamental default behavior for the open matcher authority.

### CURRENT_TOTAL_COST

Very small public surface: inherited `match(subject)` performs exactly one ordinary `this == subject` send and returns the result unchanged.

### ADD_LATER_COST

If literals/ordinary values initially use compiler-owned recognition, later unifying them under ordinary dispatch can change observable equality overrides, call counts and effects.

### PROTOS_FIT

Very strong. It deliberately refuses identity shortcuts, reverse equality, truthiness, hidden registries and special cases.

### Proposed outcome

`KEEP`

## F6 — `@name` binder, `_` wildcard, linear names and commit-after-success

### Capability

Bind a matched value/subvalue for the selected arm, explicitly discard a matched position, and make bindings visible only after the complete candidate pattern succeeds.

### Current surface

```protos
case @value => value
case _ => fallback
```

### FUNDAMENTAL_NEED

**HIGH.** Pattern matching without bindings is only multi-way recognition; structural matching becomes substantially less useful if matched components cannot be named. A wildcard/catch-all is likewise fundamental for explicit fallback and ignored positions.

### ORDINARY_PROTOS_ALTERNATIVE

Recomputing/extracting values again in the arm body is not equivalent for nested structure, custom recognition or effectful observation. A special `default` arm could replace only the top-level catch-all role of `_`, not ignored nested positions.

### CORE_VS_SUGAR

Binding/discard semantics are foundational to useful structural patterns. Exact glyphs are surface design, but the existing forms are small and solve a Protos-specific ambiguity cleanly.

A bare identifier cannot safely mean "bind" because an identifier in pattern position may denote an ordinary matcher/value object. `@name` makes the semantic distinction visible:

```protos
case positive => ...   // use object `positive` as matcher
case @positive => ...  // bind the subject to `positive`
```

`_` has no binding identity and is wildcard only in pattern position.

### CURRENT_TOTAL_COST

Small syntax plus lexical-binding rules. Linear/unique fixed binder names avoid introducing hidden repeated-name equality/rebinding semantics. Commit-after-complete-success prevents partial failed matches from leaking arm-local binding state.

### ADD_LATER_COST

Removing bindings would make the basic pattern model incomplete. Changing the spelling later is possible, but no alternative currently demonstrates enough benefit to justify compatibility churn. Leaving duplicate-name meaning unspecified would create future ambiguity.

### PROTOS_FIT

Strong: the syntax makes the semantic distinction visible and keeps bound values as ordinary lexical values in an ordinary arm Closure/body.

### Proposed outcome

`KEEP`

This recommendation includes:

- `@name`;
- `_`;
- unique/linear fixed binder names;
- source-visible bindings only after complete candidate-pattern success.

## F7 — matcher result protocol: split the foundational boolean lane from extraction

The current complete protocol is:

```text
false        -> mismatch
true         -> success, zero captures
[x, ...]     -> success with positional captures
```

The second pass finds that these are not one indivisible foundational feature.

### F7-A — canonical `false` / `true` recognition results

The open matcher authority needs an ordinary result for recognition. Canonical Boolean is already Protos's explicit Boolean domain, avoids truthiness, and is what inherited `Object.match` naturally returns from `==`.

**Proposed outcome:** `KEEP`.

### F7-B — non-empty Array as arbitrary matcher extraction carrier

This is not required merely to support ordinary value matching, `@name`, `_`, or fixed structural patterns. It primarily enables arbitrary matcher objects to return extracted values that later source machinery names/consumes.

That capability may be valuable, but its justification is inseparable from the later audit of fixed/dynamic `captures(...)`, positional extraction ABI exposure, and whether custom matchers need extraction rather than predicate-only recognition in Core v0.1.

**Current second-pass state:** intentionally **NOT CLASSIFIED IN GROUP 1**. It moves to Group 7 (`custom matcher extraction / captures(...)`).

This is not a fourth AUD009 outcome; it simply means the mechanism has not reached its assigned decision slice yet.

## Proposed Group 1 result

Pending explicit project-owner approval:

```text
KEEP
  expression-valued postfix match/case outer form
  exactly-once subject evaluation
  source-order arm attempts
  first accepted arm wins
  terminal no-selection -> fresh ordinary Error
  pattern.match(subject) as the single ordinary recognition authority
  inherited Object.match(subject) -> exactly one this == subject
  canonical false / true recognition outcomes
  @name binder
  _ wildcard/discard
  unique/linear fixed binder names
  bindings become source-visible only after complete candidate success

DEFER TO ASSIGNED AUD009-A1 GROUP, NOT YET CLASSIFIED
  non-empty Array arbitrary-matcher extraction carrier
```

## Why this is not the retention bias of the first packet

Every proposed `KEEP` above passes a stronger test than "other languages have it":

- multi-way selection solves a broad recurring general-purpose problem;
- ordering/evaluation rules are unavoidable semantics for an effectful dynamic implementation;
- ordinary Error is the smallest explicit terminal failure behavior for an expression-valued open match;
- `pattern.match(subject)` and `Object.match` establish the fundamental ownership model and avoid closed/special matcher institutions;
- binder/wildcard are necessary to turn recognition into useful structural selection and to express fallback/discard;
- the extraction Array is *not* retained here merely because it already exists.

## Decision requested

Approve, modify, or reject the Group 1 proposed classifications above. Approval changes only the AUD009-A1/D131 decision packet state; it does not change specification or implementation yet.
