# D131 — Protocol-first bonfire candidate checkpoint

Status: **CANDIDATE_PENDING_FINAL_COMPARATIVE_AUDIT_AND_OWNER_RATIFICATION**

Nature: durable non-normative decision checkpoint

Decision authority: `D131` / `guillermomolina/protos#503`
Related audit: `AUD009-A1` / `guillermomolina/protos#535`
Prior protocol-first evidence: `docs/project/work/AUD009/AUD009_A1_MATCHING_PROTOCOL_FIRST_RESULT.md`
Protos product baseline reviewed: `6ccd8b91ca5446958841db125990e4e0756c3dd0`

This record preserves the exact candidate reached during the 2026-09-17 owner review so that final D131 ratification does not have to reconstruct design intent from chat history. It is **not** normative specification text and does **not** itself ratify D131 or authorize implementation/deimplementation.

## Process state

The earlier AUD009-A1 Candidate B remains useful evidence but is no longer the exact candidate under review. The owner review deliberately applied the stricter second-pass/bonfire burden and reduced the initial retained surface further.

The exact candidate below must still receive the final GITHUB010 comparative/invariant review required by `guillermomolina/protos/AGENTS.md`, including the D131 feature matrix, strongest counterarguments, regret/recovery paths and implementation consequences, before explicit final owner ratification.

## Exact candidate

### Recognition authority

Retain exactly one public recognition authority:

```text
pattern.match(subject)
```

Retain the inherited root behavior:

```text
Object.match(subject) -> exactly one ordinary this == subject
```

Retain the existing exact matcher-result carrier:

```text
false        -> mismatch
true         -> successful match with zero captures
[x, ...]     -> successful match with positional captures
```

No second matcher authority, capture sink or callback/CPS recognition path is introduced.

### Direct structural matching by ordinary Arrays and Maps

The candidate removes the dedicated Array-pattern / Map-pattern institution. Ordinary standard collection values themselves provide standard structural recognition through their ordinary `match` behavior.

#### `Array.match(subject)`

Initial Core v0.1 behavior:

- the receiver must own standard Array indexed state; otherwise invocation signals ordinary invalid-receiver `Error`;
- an ineligible/non-Array subject returns canonical `false`;
- fixed exact-length structural recognition only;
- one shallow observation of both matcher Array and subject Array is established before invoking child matchers;
- child matchers execute left-to-right through ordinary `child.match(childSubject)`;
- each reached child is invoked exactly once;
- captures compose positionally using the existing matcher-result carrier rules;
- no recursive flattening of captured aggregate values.

No Array remainder capability is retained initially.

#### `Map.match(subject)`

Initial Core v0.1 behavior:

- the receiver must own normal standard Map keyed-entry state; otherwise invocation signals ordinary invalid-receiver `Error`;
- an ineligible/non-Map subject returns canonical `false`;
- open/subset recognition only;
- one shallow stable observation of both matcher Map requirements and subject Map associations is established for the attempt;
- matcher requirements are considered in matcher insertion order;
- required subject associations are resolved with ordinary normal-Map key semantics (`hash` plus query-side `==` under the existing Map contract);
- missing required association returns canonical `false`;
- unrelated subject associations are ignored;
- required associations/value references are fixed before nested mapped-value child matchers execute;
- mapped-value child matchers execute in requirement order through ordinary `match`;
- captures compose positionally using the existing matcher-result carrier rules.

No exact Map or Map remainder capability is retained initially.

### Initial standard matcher objects

Retain only the smallest standard matcher set needed for ordinary case selection and structural extraction:

```text
Any.match(subject) -> true
Capture.match(subject) -> [subject]
```

`Any` and `Capture` are ordinary matcher objects available through the standard language environment, not keywords or dedicated pattern syntax.

The following are intentionally absent initially and remain future reconsideration candidates rather than dormant Core behavior:

```text
Or(...)
Guard(...)
Identity(...)
Array remainder combinators
exact Map
Map remainder
```

### Ordinary selection surface

The reviewed public selection form is:

```protos
value.caseOf(cases)
```

`caseOf` is an ordinary selector/message, not a keyword or dedicated grammar form. The naming intentionally follows the same message-oriented style already used by Protos control protocols such as `ifTrue`, even though future syntactic sugar may later provide a more JavaScript-like surface if real use justifies it.

`cases` is an ordinary normal standard Map whose associations are:

```text
matcher -> callable
```

The Map's normal properties are deliberate parts of this initial representation:

- insertion order determines case order;
- matcher keys use ordinary Map `hash` / `==` semantics;
- equal/duplicate matcher keys cannot coexist as distinct cases;
- no special matching-specific key identity/equality layer is introduced.

At the start of one `caseOf` attempt, selection fixes a shallow ordered observation of the case associations. Later mutation of the cases Map does not rewrite the already-selected sequence for that attempt.

Cases are attempted in observed insertion order:

```text
result = matcher.match(value)

false        -> continue
true         -> invoke callable()
[captures]   -> invoke callable(...captures)
other normal -> Error
```

The first successful matcher commits. The selected callable's ordinary result is returned unchanged, including `null`.

If no matcher succeeds, `caseOf` signals one fresh ordinary `Error`.

No eager whole-table validation layer is introduced. An unreached matcher/action does not fail merely because it would be invalid if reached; ordinary lookup/invocation/result validation occurs when that case is actually attempted/selected.

### Capture naming and body invocation

Capture names belong to ordinary Closure/callable parameters, not to matching syntax.

Examples:

```protos
value.caseOf(%{
    [1, Capture]: x => x
    Any: () => null
})
```

and:

```protos
value.caseOf(%{
    %{
        "type": "user"
        "name": Capture
    }: name => name
})
```

Successful positional capture Arrays are supplied through ordinary call spread / callable parameter binding. No matching-specific selected-arm binding ABI is retained.

## Initial Core v0.1 removals under this candidate

The exact final D131 feature matrix still has to map every existing feature, but this candidate's direction is to remove/deimplement the current dedicated pattern-language surface and its dependent institutions, including initially:

```text
postfix subject match { ... } grammar
case arm grammar
when guard syntax
@ binder syntax
_ wildcard syntax
Array-pattern grammar
Map-pattern grammar
Array remainder forms
exact Map mode
Map remainder forms
alias pattern syntax
p1 | p2 pattern syntax
captures(...)
OR binding-name/interface equivalence
D103 complete-arm dynamic-rest terminality
matching-specific arm-binding ABI
dedicated static coverage/exhaustiveness framework
```

Removal means actual removal from Core v0.1 specification/grammar/parser/lowering/runtime/tests/docs where applicable. D131's no-pseudo-DEFER rule still applies.

## Future-growth rule

Removing the dedicated pattern language now is **not** a permanent prohibition on future matching capability or syntax.

Capabilities may be reconsidered later when concrete use justifies them. Preferred growth direction is ordinary matcher/combinator behavior first, with optional syntax only after its value is demonstrated and preferably as explicit lowering onto the ordinary protocol.

Plausible future reconsideration examples include:

```text
Or(...) and possible `|` sugar
Guard(...)
Identity(...)
Array remainder / head-tail matching
exact Map
Map remainder
compact match/case-like syntax
binder/wildcard sugar
static analysis attached to a future sufficiently closed syntax surface
```

The foundational boundary intended to remain stable is:

```text
ordinary matcher objects
    -> pattern.match(subject)
    -> false | true | capture Array

ordinary ordered case data
    -> ordinary caseOf message
    -> ordinary callable invocation
```

## Owner-review invariants preserved by this checkpoint

The working review established the following constraints that final candidate construction must not silently reverse:

1. Matching should be protocol-first; the current dedicated `match/case` grammar is not presumed fundamental.
2. `pattern.match(subject)` remains the single recognition authority.
3. Ordinary Closure parameters, not matching syntax, own capture names.
4. `value.caseOf(...)` is the reviewed ordinary selection spelling.
5. The initial case carrier is an ordinary insertion-ordered normal Map; normal Map uniqueness/hash/equality semantics are accepted deliberately rather than repaired by a parallel case institution.
6. No-selection is a fresh ordinary `Error`; a selected body returning `null` remains a successful ordinary result.
7. Initial standard helper matchers are only `Any` and `Capture`; OR, guard, identity and remainder/exactness capability are not prebuilt merely for possible future use.
8. Standard Array/Map structural match methods require semantically eligible receiver state. Delegation grants lookup, not built-in collection membership; an inherited standard `Array.match`/`Map.match` invoked on an ineligible receiver therefore signals invalid-receiver `Error` rather than silently falling back to `Object.match`.
9. Structural attempts fix shallow observations before child matching so child effects cannot rewrite the matcher/subject structure already selected for the current attempt.
10. Future sugar/capabilities remain possible, but removal from Core v0.1 means actual despecification/deimplementation now rather than dormant support.

## Material deltas from AUD009-A1 Candidate B

Compared with `AUD009_A1_MATCHING_PROTOCOL_FIRST_RESULT.md`, this exact candidate intentionally narrows the retained initial surface:

- structural recognition moves from an unspecified standard structural-matcher construction to direct ordinary `Array.match` / `Map.match` behavior on semantically eligible standard collection values;
- terminal Array remainder is no longer retained initially;
- guard/refinement and ordered OR are no longer retained as initial standard combinator capabilities;
- the exact selector/API is now `value.caseOf(cases)`;
- the exact initial case representation is an ordinary insertion-ordered normal Map;
- the initial standard helper set is exactly `Any` + `Capture`;
- receiver eligibility and matcher/case snapshots are explicit.

These deltas require the final GITHUB010 comparison to evaluate the **exact candidate recorded here**, not merely to cite Candidate B's earlier recommendation unchanged.

## Next required step

Before explicit D131 ratification:

1. reconcile the complete D131 existing-feature inventory against this exact candidate;
2. compare the exact surviving candidates under the mandatory GITHUB010 dimensions;
3. record strongest counterarguments, regret scenarios and recovery paths;
4. perform the final owner-invariant/delta consistency check;
5. present the exact candidate for explicit project-owner ratification;
6. only after ratification, produce/execute the ordered spec + parser + AST/lowering + runtime + tests + docs reconciliation plan.
