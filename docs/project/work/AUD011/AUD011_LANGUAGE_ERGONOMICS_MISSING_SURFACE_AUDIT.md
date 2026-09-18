# AUD011 — Language ergonomics and missing-surface audit

## Status and authority

This is the durable non-normative evidence and routing record for
`guillermomolina/protos#540` (`AUD011 — Language ergonomics and missing-surface audit`).

Operational state, priority, assignment, owner approval and decision authority remain in
`guillermomolina/protos`. This record does not ratify any Dxxx/LIBxxx outcome.

Final source sweep revision:

```text
PROTOS_REVISION=b1b5c91b365a57ed65797b78ab9a6466e7df16f5
PROJECT_DOCS_BASE=149d90f94172e7b54220f2256ef2c84058906464
AUD011_ISSUE=guillermomolina/protos#540
```

## Audit question

AUD011 asks the forward-looking complement of AUD009:

> Which missing ergonomic capabilities create enough recurring real-source friction to
> justify adding anything at all?

A familiar feature in another language is not sufficient evidence. Each candidate was
checked against current Protos source, specification and existing ordinary mechanisms
before routing.

The audit preserves these constraints:

- prefer ordinary objects, messages and Closures before adding privileged syntax;
- distinguish semantic need from sugar;
- do not prebuild generic iterable, Optional/Maybe, type-system or pattern institutions;
- do not reopen D130, D131 or other ratified decisions implicitly;
- defer syntax when an ordinary value/protocol can establish the mechanism first;
- `REVISIT_AFTER_REAL_USAGE` reserves no syntax, parser hooks or compatibility promises.

## Final routing matrix

| Candidate | Evidence result | AUD011 outcome | Formal owner / note |
|---|---|---|---|
| Concise Array construction | Real; already routed before AUD011 | Record precedent | D130/#502 + I039/#539, both closed |
| Populated Map construction | Real | Completed through existing route | D136/#542 + I040/#543, both closed |
| String interpolation / dynamic text composition | Real | `ROUTE_TO_DXXX` | D141/#569 |
| Integer ranges / progressions | Real at value/API layer; syntax not established | `ROUTE_TO_LIBXXX` | LIB017/#570; range syntax revisits only after real API usage |
| Spread outside call spread | Mostly already covered | No new owner overall | forwarding + Array spread `NO_ACTION`; object composition owned by D140/#568; Map merge + generic spreadability revisit after real usage |
| Map expected-absence/default lookup | Real | `ROUTE_TO_DXXX` | D142/#572 |
| Destructuring/binding outside match | Real only after narrowing to ordered Array local binding | `ROUTE_TO_DXXX` | D143/#573 |
| Null/default chaining | Real null-aware control/fallback pressure; optional-chain syntax not established | `ROUTE_TO_DXXX` | D144/#575 |
| Recursive local Closure preinitialization | Repetition is real, but capability already exists | `DOCUMENT_OR_STYLE_ONLY` | Direct recursive slot creation already works; no language owner |
| Fresh generic Error signaling | Broad repeated helper idiom | `ROUTE_TO_DXXX` | D146/#576 |
| Exact Core semantic-family validation | Broad repeated indirect family gates | `ROUTE_TO_DXXX` | D147/#577 |
| Dedicated async/await syntax | No independent ergonomic/semantic need demonstrated | `NO_ACTION` | Existing `Future.value`, `then`, `Future.all` remain ordinary protocols |
| Dedicated `break` / `continue` | Current evidence insufficient | `REVISIT_AFTER_REAL_USAGE` | Reconsider only for repeated otherwise-useless loop-control flags / structural rewrites |

All Dxxx/LIBxxx destination items remain independent formal owners. AUD011 does not make
them children and does not select their substantive candidate.

## Candidate findings

### Array construction — precedent

D130 established that concise Array construction was genuine friction and selected exact
ordinary-call sugar:

```text
[a, b] -> Array(a, b)
[a, ...items, b] -> Array(a, ...items, b)
```

The syntax inherits existing call spread. AUD011 records this as the reference example for
how an ergonomic need can become pure sugar only after its ordinary semantics are clear.

### Map construction

D136 resolved populated equality-keyed Map construction as `%{...}`. The decision
deliberately did not add IdentityMap sugar, generic `%Factory{...}`, spread/merge,
comprehensions or pattern semantics.

AUD011 therefore treats populated Map construction as resolved, while Map merge/spread
remains a separate evidence question.

### String interpolation — D141

Production CLI, package-tool, Test Tool and TOML source contains substantial dynamic text
composition through repeated strict String `+`.

The important separation is:

```text
NEED_A = concise dynamic text composition
NEED_B = arbitrary value -> user-visible String conversion
```

Evidence establishes `NEED_A`; it does not automatically establish `NEED_B`.

Current Core has no interpolation, `${...}` in a String is ordinary literal text, String
`+` is strict String-to-String composition, and no universal guest `toString` /
`printString` protocol exists.

D141 owns the decision among no change, String-only interpolation, interpolation plus a
text conversion protocol, formatting/library-first approaches and broader extensible
models.

### Ranges — LIB017 first, syntax deferred

Production code contains true bounded index/progression loops such as:

```protos
index: start
(() => index < end).while() {
    ...
    index = index + 1
}
```

but many other `while` loops are state-dependent scanners and are not Range evidence.

The demonstrated need can initially be represented by an ordinary finite Integer
Range/progression value. No parser/operator change is required to establish that model.
Therefore the audit routes to LIB017 first and leaves `a..b`-class syntax for a later
independent evidence pass.

### Spread ergonomics — decomposition, no umbrella feature

The original candidate does not survive as one missing capability:

- call forwarding is already expressed by rest parameters + call spread;
- Array construction inherits existing call spread through D130;
- object-body `...source` is horizontal composition and is owned by D140;
- current production source does not show recurring whole-Map merge/copy boilerplate;
- current production source does not show recurring need to spread arbitrary non-Array
  iterable values into calls.

Map merge/spread should be reconsidered only when production code repeatedly needs
"existing Map + all associations from another Map + local additions". Generic spreadability
should be reconsidered only when repeated non-Array flattening materially exposes
allocation/API problems.

### Map expected-absence/default lookup — D142

Core deliberately distinguishes:

```text
map.at(key)
    present -> exact stored value
    absent  -> Error

map.containsKey(key)
    present -> true
    absent  -> false
```

and:

```text
ABSENT != PRESENT_WITH_NULL
```

Production code repeatedly performs expected-absence reads, fallback selection and
create-on-missing behavior.

This is not automatically a library-only issue: a helper composed from `containsKey`
followed by `at` may perform two observable key searches, while a standard one-search
operation would have different `hash` / `==` effect/failure behavior.

D142 therefore owns eager/lazy fallback, get-or-insert, carrier and library-only
alternatives.

### Multiple local binding from ordered Arrays — D143

The audit found direct unpacking sequences such as:

```protos
a: state[0]
b: state[1]
c: state[2]
d: state[3]
e: state[4]
f: state[5]
g: state[6]
h: state[7]
```

and several smaller 2–4 field forms.

The problem was deliberately narrowed away from general destructuring.

D080-A remains intact: arbitrary objects do not acquire a universal positional
deconstruction protocol.

D131 remains intact: the removed dedicated pattern grammar, binders, wildcard, rest,
OR, alias and capture institutions are not reintroduced.

An immediately invoked Closure with spread can bind positions in a new activation:

```protos
((a, b) => {
    ...
})(...pair)
```

but that is not identical to creating ordinary slots in the current context. D143 owns
the exact fixed-Array local-binding question and must test the smallest sufficient form
before considering rest, nested, Map/named or assignment variants.

### Null-aware control and fallback — D144

Production code repeatedly spells:

```protos
(value === null).ifFalse(() => {
    use(value)
})
```

and:

```protos
result: fallback
(value === null).ifFalse(() => {
    result = transform(value)
})
```

Core already has a strong null model:

- exactly one canonical `null`;
- no `undefined`;
- a slot containing `null` is present;
- failed lookup signals Error;
- no truthiness.

The source demonstrates null-aware control/transform/fallback pressure but not strong
long-chain `a?.b?.c` pressure. D144 therefore decides the ordinary semantic mechanism
first. Optional-chaining and null-coalescing syntax remain deferred until protocol/API
usage demonstrates independent syntax need.

D142 remains separately authoritative for Map absence.

## Additional discoveries from the real-source sweep

### Recursive local Closure preinitialization — style only

Many modules contain:

```protos
readNext: null
readNext = () => {
    ...
    readNext()
}
```

At first sight this looked like missing recursive binding. It is not.

The repository already contains canonical direct recursive definitions such as:

```protos
factorial: (n) => {
    ...
    result = n * factorial(n - 1)
}
```

Closures capture lexical contexts by reference; direct recursive slot creation is already
supported. The two-step null preinitialization is therefore cleanup/style debt where
semantically equivalent, not a new language feature.

### Fresh generic Error signaling — D146

Many independent modules define:

```protos
fail: () => {
    Error().signal()
}
```

solely to abbreviate "create one fresh generic Error and signal that exact occurrence".

D005 remains authoritative for Error freshness, identity and shallow taxonomy. D146 asks
only whether that common operation deserves a concise ordinary standard surface, comparing
no change, a prelude callable, a factory-side message and library-only approaches. New
throw/raise syntax is only a high-cost baseline.

### Exact Core semantic-family validation — D147

Multiple modules validate exact semantic families by provoking strict family operations:

```protos
ignoredValidation: "" + value   // require semantic String
value.div(1)                    // require ordinary unbounded semantic Integer
```

Several sources explicitly describe this as a family gate, not conversion.

This is distinct from ordinary duck typing: the relevant APIs really require an existing
Core semantic family, and Core already distinguishes semantic family membership from
delegation/message availability. An object may delegate to `String` without becoming a
semantic String, so prototype-parent reflection is not an exact substitute.

D147 asks whether that existing Core distinction needs an explicit runtime query/validation
surface. It explicitly excludes static typing, annotations, flow narrowing, nominal user
classes and a generic type system.

### Dedicated async/await syntax — no action

The current Future surface is already explicit and compact:

```protos
result.value()
Future.all(...pending).value()
source.then(...)
```

The bundled tutorial explicitly teaches `value()` as the ordinary Future
observation/suspension operation. Current production code does not demonstrate additional
semantic or compositional friction that would justify privileged `await` syntax.

### Dedicated break/continue — revisit only with stronger evidence

State-driven loops use values such as `done`, `more` and `handled`, but current examples
generally represent actual parser/reader state or participate directly in the loop
condition. The audit did not find enough otherwise-useless flag machinery to justify a new
loop-specific dynamic-control institution.

Reconsider only when production code repeatedly needs deep iteration escape/skip and must
maintain otherwise-useless flags or structurally rewrite code solely because current
condition/body protocols cannot express it clearly.

## Cross-cutting conclusions

The audit produced four recurring layering lessons.

First, **mechanism before sugar** remains effective. Range values should precede range
operators; null-aware protocol semantics should precede `?.` / `??`; a text-composition
need must be separated from arbitrary text conversion.

Second, **existing ordinary mechanisms eliminate many apparent gaps**. Rest + call spread
already solves forwarding, D130 already gives Array spread construction, direct recursive
Closure definitions already work, and `Future.value()` already exposes suspension
explicitly.

Third, **semantic distinctions already present in Core can themselves create ergonomic
pressure**. D142 exists because Map absence differs from stored null and key search is
observable. D147 exists because semantic family membership differs from delegation.

Fourth, **real-source repetition is necessary but not sufficient**. Repetition may indicate
obsolete style rather than a missing feature, as shown by recursive Closure
preinitialization.

## Closure assessment

At this revision the initial AUD011 candidate set has been checked against current
source/spec/library reality, each candidate has an explicit routing outcome, additional
source-derived candidates have been classified, and surviving needs have independent
formal owners without substantive selection by the audit.

The destination decisions remain subject to their own AGENTS.md decision packets and
explicit project-owner approval.

AUD011 may therefore close as an audit/routing item once this durable record is published
and the authoritative Issue records the publication evidence and final matrix.
