# D263 — Contextual match with implicit lazy arm Closures (C)

Date: 2026-11-25

Status: owner-selected and ratified design; normative specification and executable
implementation pending in guillermomolina/protos.

Design authority: [D263 / #1172](https://github.com/guillermomolina/protos/issues/1172).
Exact candidate and comparative evidence:
[research packet](https://github.com/guillermomolina/protos/issues/1172#issuecomment-7154024506).

`PROTOS_REVISION=510a2c71e9c62cc3c147dea447c70d26e18d9d36` (investigation and
ratification baseline, not an implementation publication).
Specification baseline: global revision 0.1.508.

## Owner approval and scope

In the active design conversation on 2026-11-25 the project owner explicitly
stated **"apruebo D263-C"** after presentation of the exact C candidate,
including the contextual-call compatibility break, grouping-transparent
Closure classification, object-valued RHS distinction, and ordinary
Map/Closure/caseOf equivalence. This records the owner's statement, not an
owner-authored GitHub comment. The owner also expressly authorized this bounded
project-docs commit and push and requested an independent implementation issue.

This decision authorizes only the selected syntax and its exact ordinary
semantic expansion. It does not approve any additional runtime design,
performance shortcut, or unrelated syntax.

## Selected source and mandatory desugaring

```protos
result: match(value) {
    0: 11
    1: 23
    2: 37
    Any: 53
}
```

is observably equivalent to

```protos
result: value.caseOf(%{
    0: () => 11
    1: () => 23
    2: () => 37
    Any: () => 53
})
```

The expression returns the exact ordinary send result. The equivalence is a
semantic AST desugaring, not a mandatory textual rewrite.

### 1. Contextual recognition

Recognize only the bare, ungrouped identifier `match`, exactly one explicit
non-spread subject expression within parentheses, and an attached opening
brace with no intervening logical NEWLINE after the closing parenthesis.

```ebnf
contextual-match-expression =
    "match", "(", [ layout ], expression, [ layout ], ")",
    "{", map-construction-sequence, "}" ;
```

Apply this rule in the existing postfix call position, before ordinary named
argument normalization, with the existing trailing-Closure logical-line and
comment attachment rules. The subject is an ordinary expression: in
`match(flag: true) {...}` the colon expression creates a slot, not a named
argument. The contextual token is not a lexical read of a binding named
`match`. No global reserved word is introduced.

Commit to the contextual grammar once the header and attachment match, even
when the body is malformed. Do not backtrack to an ordinary trailing Closure
based on body contents, inferred types, or lexical/dynamic bindings. The
completed result supports ordinary postfix suffixes and nesting.

All other source shapes retain ordinary grammar: `match(v)`,
`(match)(v) { body }`, `obj.match(v) { body }`, zero/multiple/spread
arguments, and braces separated by a logical newline. Ordinary named argument
association remains applicable to the ordinary forms, including
`(match)(flag: true) { body }`.

### 2. Entries and RHS classification

Reuse the Map construction sequence and entry grammar exactly: binary-expression
key, colon, ordinary RHS expression. The existing newline/semicolon separator,
continuation, grouping and nesting rules apply; no comma or trailing semicolon.
Empty tables are allowed.

Classify the complete parsed RHS expression after disregarding only outer
parenthesized grouping. If its root is an explicit Closure literal, preserve
the original expression unchanged. Otherwise wrap the whole RHS in an ordinary
zero-parameter lexical Closure whose Sequence contains that one expression.
Classification is exclusively syntactic; never evaluate or inspect the
runtime type of the RHS to decide.

```protos
result: match(value) {
    [1, Capture]: x => x
    [Capture, Capture]: (x, y) => x + y
    0: () => {
        prepare()
        execute()
        11
    }
    1: { x: 10; y: 20 }
    Any: null
}
```

Grouped Closure literals are explicit actions, not implicit Closure results.
An explicit outer Closure can return a Closure literal:
`0: () => (x => x + 1)`. A bare `0: callback` evaluates the lookup and returns
its result when selected; `0: callback()` invokes it when selected. Use
ordinary caseOf or explicit forwarding with the desired parameter/rest
signature to pass a preexisting callable as the action.

A bare object RHS is an ordinary object expression, never a statement block:
`0: { x: 10 }` is equivalent to `0: () => ({ x: 10 })`. Multiple-expression
actions use the existing explicit braced Closure form. There is no new block
shorthand, and a postfix call on a Closure expression is classified by its
complete expression root, not by the presence of an arrow somewhere inside it.

### 3. Existing semantic authority and evaluation

Produce the existing canonical Send of selector caseOf, with the subject as
receiver and one Map construction argument. Use the existing ordinary lexical
Map lookup, ordered initial associations, ordinary Closure and Sequence forms.
No matching-specific canonical node, runtime Closure check, privileged lookup,
binding or callable ABI is introduced.

Evaluate the subject once, then construct the Map under D136/D239: ordinary
factory lookup and eligibility, exactly one eligible construction, and
source-order entry evaluation (key then action Closure creation then initial
association definition with ordinary hash/equality). Initial definition is
not atPut. Duplicate equal keys signal ordinary construction Error, not
replacement or static rejection. Effects already performed are not rolled back.
All keys are evaluated before standard selection; later unreachable keys can
still signal during construction. Ordinary caseOf lookup occurs after argument
construction, so overrides and effects on selection are preserved.

The standard caseOf observes its normal Map carrier and tries matchers in
insertion order using the sole D131 recognition authority
`pattern.match(subject)`. Its first-success commitment, matcher outcomes,
shallow snapshot, capture spreading, reached-callable validation and no-match
Error are unchanged. Empty tables fail under standard caseOf; Any remains an
ordinary matcher and is neither required nor specially reordered. Capture
remains an ordinary matcher; extra positional captures passed to a parameterless
action are ignored under ordinary flexible-arity rules. Explicit Closure
parameters, defaults and rest keep their normal semantics.

Only the standard caseOf guarantees one selected action. A custom caseOf can
observe, retain or invoke the Map and Closures just as with explicit source.
No implicit await, coercion, truthiness, eager arm evaluation, new validation
or static exhaustiveness/reachability institution is introduced.

Preserve exact result identity including null, Closure and Future values,
ordinary lexical capture by reference, fresh activation, this, context, super,
methodHome, return homes, active and escaped non-local return, InvalidReturn,
Error, ensure, suspension/no replay and cancellation. No new Task, Actor,
scheduler, resource or cross-boundary authority exists.

### 4. Explicit compatibility delta

The previously valid ordinary lexical call `match(v) { x: 1 }` now parses as
contextual matching, with x evaluated as a Map key. A previously valid
`match(v) { log() }` becomes a syntax error under the entry grammar. The owner
accepted this exact source reinterpretation by approving candidate C, which
explicitly disclosed it. Migrate ordinary calls to `(match)(v) { body }` or
`match(v, () => { body })`. No other call shapes or ordinary selectors are
reinterpreted.

## GITHUB021 invariant/delta reconciliation

| Prior authority | Outcome |
|---|---|
| D131, ordinary match authority, case carrier, first success and positional captures | Preserved. Only the former absence of dedicated source sugar is narrowly reopened. |
| D136, Map initial-definition semantics | Preserved; no atPut, duplicate replacement or early static rejection. |
| D239, factory lookup and eligibility | Preserved; no privileged Map lookup. |
| D203, flexible Closure arity | Preserved; no matching-specific arity policy. |
| D210/D211/D212/D231, contextual syntax and existing control protocols | Preserved; their structural precedent is reused. |
| D237, named argument association | Preserved outside the exact new contextual shape; ordinary subject expression inside it. |
| Ordinary bare match calls with one attached trailing Closure | Explicitly reinterpreted only for the approved exact header; grouped-call escape retained. |
| Owner's requested short forms and mixed explicit capture Closures | Preserved exactly; no new hidden runtime mechanism. |

`DECISION_APPROVAL_PROVENANCE=PASS`
`DECISION_INVARIANT_CONSISTENCY=PASS`

## Alternatives and deliberately deferred scope

The research packet contains the comparison of A (status quo), B (explicit
Closures only), C (selected), D (extra case marker), and E (switch spelling).
C solves the concrete verbosity without a second recognition or execution
mechanism. Its strongest cost is the explicit source compatibility break and
syntactic distinction of literal actions; D avoids the former but adds a marker.
B would make later implicit arm semantics a source-breaking reinterpretation.

Reject separate patterns/bindings, implicit runtime callable detection,
unconditional branch lowering, changes to ordinary Map RHS semantics and a
second match/switch spelling. Defer guards, OR syntax, dedicated default/wildcard,
exhaustiveness, block shorthand and break/continue/fall-through. Ordinary
matchers and explicit caseOf provide the escape path without speculative
infrastructure.

## Normative publication and implementation handoff

The owning I item must update PROTOS_GRAMMAR.md (matching, contextual/postfix
grammar, attachment and identifier references), PROTOS_LANGUAGE_SPEC.md
(contextual overview), and MATCHING.md (source-sugar cross-reference and
D131 boundary), plus the global PROTOS_SPEC_CHANGELOG.md. Do not duplicate or
change the ordinary Map/Closure/evaluation contracts in other normative owners.
Assign the next specification revision and implementation version only at
the applicable publication gates.

Reuse ProtosParser.parseCallSuffix/isBareName/parseMapConstructionEntry,
SurfaceGroup/SurfaceClosure, Canonicalizer.lowerMapConstruction,
lowerContextualIf/lowerLazyBoolean/lowerClosure, CanonicalMapConstruction,
CanonicalSend and CanonicalClosure. A source-specific Surface AST and source
facts are permitted to preserve original source locations. No CanonicalMatch,
runtime primitive, new bytecode operation or auxiliary runtime root. Audit
parser/source facts, static analysis, formatter, LSP, diagnostics and guides.

Require paired equivalence evidence for simple and explicit capture actions,
grouped and nested Closure expressions, callable values, object RHS, nested
match, postfix composition, separators and attachment, contextual named
headers, ordinary escapes, malformed bodies, duplicate keys and effect order,
Map shadowing, custom caseOf, matcher ordering/snapshots, captures/defaults/rest,
result identity, lexical writes, non-local control, suspension/cancellation,
and source/tooling round trips. Run the focal and impact-required validation
under the human-executor rules; this record does not assert tests passed.

PERF044 / #883 remains independent performance-only work. Both source forms
must feed the same existing canonical execution path and eligible hot-path
specialization; no additional runtime cost relative to equivalent explicit
source is the design target, not a measured allocation-free guarantee.

No product repository content or specification revision was changed by this
ratification. No build, test or benchmark has been run.
