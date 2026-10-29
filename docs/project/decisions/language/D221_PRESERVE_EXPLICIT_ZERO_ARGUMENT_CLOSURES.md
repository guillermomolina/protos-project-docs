# D221 — Preserve explicit zero-argument Closure syntax

**Status:** RATIFIED / CLOSED — Candidate A (no syntax change), owner-approved 2026-10-29.
**Issue:** [D221/#984](https://github.com/guillermomolina/protos/issues/984).
**Research evidence:** [D221 comparative grammar and compatibility audit](../../evidence/D221/D221_BRACE_CLOSURE_SHORTHAND_RESEARCH_2026_10_29.md).
**Observed Protos revision:** [ce6cdec09a615a31edf668e22bd022be6cac2626](https://github.com/guillermomolina/protos/commit/ce6cdec09a615a31edf668e22bd022be6cac2626) (source baseline only; no D221 product commit).
**Authority:** Non-normative decision record; unchanged normative syntax and semantics remain in `guillermomolina/protos/spec/`.

## Exact owner decision

After the exhaustive D221 comparative investigation, the owner accepted its recommended **Candidate A — retain the current grammar**, stating in the active interaction: **“perfecto lo dejamos como está.”** The owner explicitly requested Issue coordination and durable research evidence publication. This is approval provenance recorded by the agent, not a claim that the owner authored a GitHub comment or ran a test.

D221 closes the bounded question without implementing an alternate parameterless Closure syntax. This is a deliberate decision to retain the existing grammar, not a permanent prohibition on future independently approved syntax designs.

## Selected boundary

- A standalone zero-parameter Closure literal remains `() => { expression-sequence }`.
- Bare `{ ... }` in ordinary primary-expression position remains Object construction, never a Closure inferred from type, contents, selector or assignment.
- Already-ratified trailing parameterless Closures `receiver(...) { body }` remain supported only in their existing syntactic attachment conditions.
- Narrow contextual `if(condition) { ... } else { ... }`, `while(condition) { ... }` and `{ body } until(condition)` retain the exact source shapes approved in D210/D211/D212. They do not establish generic brace conversion.
- `whileTrue`, `whileFalse`, `untilTrue` and `untilFalse` retain their Closure-domain, receiver orientation, dynamic dispatch, strict Boolean and exact last-body-result semantics. Their selector names are not implicit brace-reinterpretation triggers.

Existing syntax, without a new language rule:

```protos
counter: 0
body: () => {
    counter = counter + 1
    counter
}
last: body.untilFalse() {
    counter > 0
}
```

Here the final braced argument is an already-supported trailing zero-parameter Closure. Analogously `body.untilTrue() { condition }` works as an ordinary trailing-Closure call, and D212 offers `{ body } until(condition)`. Neither validates the proposed `{ body }.untilFalse({ condition })` as a Closure invocation.

## Compatibility and falsification

The object-body grammar accepts expression content, so inspecting the body of `{ ... }` cannot distinguish Object construction from a putative Closure. Reinterpreting `x: { 42 }`, `f({ 42 })`, `{ 42 }.method()` or an ordinary user selector `holder.untilTrue({ x: 1 })` changes existing object expressions and/or dynamic protocol behavior. Direct `{ body }()` invokes an Object's ordinary `call` protocol, not a guaranteed lexical Closure. Grouping does not change value kind.

The exact D212 `{ body } until(condition)` suffix is a structurally identified, separately approved exception. Selector-sensitive, type/content-inferred, and global brace-as-Closure proposals were rejected for D221. A future explicit marker could be investigated separately if justified by a concrete workload.

## GITHUB021 invariant/delta audit

| Prior owner-approved constraint | D221-A |
| --- | --- |
| Ordinary Object `{ ... }` construction and expression grammar | **Preserved** |
| D013 ordinary `call` selection and shadowing | **Preserved** |
| D203 flexible Closure arguments and defaults | **Preserved** |
| Lexical capture, Closure identity, `this` / `methodHome` and nonlocal return home | **Preserved** |
| D199 no generic undotted/brace reinterpretation | **Preserved** |
| D210 exact contextual `while` boundary | **Preserved** |
| D211 exact contextual `if/else` boundary | **Preserved** |
| D212 `until` suffix and Object fallback | **Preserved** |
| Evaluation order, Error, Future, suspension and cancellation | **Preserved** |

**DECISION_INVARIANT_CONSISTENCY=PASS:** no prior approved invariant is reopened; no new observable or architectural consequence was introduced since approval.

## Work and validation disposition

```text
DECISION_APPROVAL_PROVENANCE=OWNER_EXPLICIT_D221_A_2026_10_29
D221_SELECTED=A_STATUS_QUO
D221_IMPLEMENTATION_REQUIRED=NO
NORMATIVE_SPEC_CHANGE=NO
PROTOS_PRODUCT_CHANGE=NO
NEW_Ixxx_ISSUE=NO
GITHUB021_INVARIANT_DELTA=NONE
```

No Protos source, tests, grammar, formatter, specification, version or changelog was edited for D221. Research/ratification are documentation-only: no compiler, test suite, CLI or runtime execution is claimed. Any later new Closure surface requires its own evidence and explicit owner approval.
