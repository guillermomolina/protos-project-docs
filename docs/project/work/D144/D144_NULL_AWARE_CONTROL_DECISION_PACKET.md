# D144 — Null-aware control, transformation and fallback semantics

## Decision state and authority

This is the durable non-normative decision packet for
`guillermomolina/protos#575`.

```text
D144_STATUS=OWNER_APPROVED
SELECTED_CANDIDATE=B_PRIME
PROTOS_REVISION=d1bbab2c1c1023e980b43ca01e7b2adafcac05f8
PROJECT_DOCS_BASE=327f980138e90bb286a45adf084b29de71fe88b6
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

Observable Protos semantics remain authoritative only through the normative
specification under `guillermomolina/protos:spec/**`. This packet records the
research, repository evidence, comparison, exact owner-approved candidate, and
intentionally deferred surface.

## Exact decision problem

Protos has exactly one canonical `null` object and no `undefined`. A slot
containing `null` is present; a failed member lookup signals `Error` and never
produces `null`.

Repository production code nevertheless contains repeated expected-null control
patterns:

```protos
(value === null).ifFalse(() => {
    use(value)
})
```

and fallback/transform forms such as:

```protos
result: fallback
(value === null).ifFalse(() => {
    result = transform(value)
})
```

D144 asks for the smallest Protos-native mechanism that reduces this repeated
expected-null control while preserving exact-null identity, ordinary message
lookup, normal callback invocation, and the existing distinction between absence
and lookup failure.

## Repository usage classification

The audit separated the observed `=== null` sites into materially different
classes.

### Present-only action and transformation

These are positive evidence for D144. Representative sources include Package
Tool compatibility projection and Test Tool optional resource-catalog loading.
The repeated semantic shape is:

```text
receiver is exactly null
    -> skip work
receiver is not null
    -> invoke behavior using that exact value
```

### Fallback and default selection

Some production code initializes a fallback and conditionally replaces it after a
non-null transformation. This is direct evidence for a lazy null fallback
operation.

### Validation, EOF, parser state, and sentinels

Many `=== null` checks intentionally validate required values, represent EOF,
track parser state, bootstrap recursive bindings, or assert test expectations.
Those are not ergonomic evidence for optional chaining and are not migration
targets merely because D144 exists.

### Long optional navigation

The current repository does not demonstrate strong recurring pressure for
multi-step optional chains comparable to:

```text
a?.b?.c?.d
```

That absence is central to the syntax deferral.

## Current Protos invariants

The selected candidate preserves:

- exactly one canonical `null`;
- no `undefined`;
- `null` delegates directly to `Object`;
- an ordinary object delegating to `null` remains a distinct non-null object;
- semantic identity `===` is the exact null test;
- no truthiness;
- failed member lookup remains `Error`;
- ordinary message lookup and dispatch remain ordinary;
- ordinary receiver and argument expressions remain left-to-right and eagerly
  evaluated before invocation;
- callback invocation uses the ordinary polymorphic call protocol;
- callback errors, non-local control, suspension, and Future values preserve
  their existing semantics;
- D142 Map expected absence remains distinct from a stored `null`.

## Comparative research

### Smalltalk / Pharo — message-oriented null control

Pharo uses ordinary `ifNil:`, `ifNotNil:`, and combined nil-control messages.
The non-nil branch can receive the present value as an argument. This is the
closest mature precedent for solving the problem through ordinary messages
rather than grammar.

References:
- https://books.pharo.org/booklet-AMiniSchemeInPharo/html/Chapters/Scheme.html
- https://books.pharo.org/booklet-ReflectiveCore/html/Chapters/ObjV/ObjV.html

Contribution to D144: strong evidence that exact nil-sensitive control can remain
an ordinary message/callback protocol.

### Self — prototype/message-oriented minimalism

Self defines a distinguished `nil` object and is centered on ordinary objects,
messages, delegation, and behavioral factoring. Its programming guidance even
contains a dedicated "nil Considered Naughty" discussion, reinforcing that nil
handling should not automatically become a large privileged syntax institution.

References:
- https://handbook.selflanguage.org/2024.1/langref.html
- https://handbook.selflanguage.org/2024.1/

Contribution to D144: prototype-oriented evidence for preserving ordinary object
semantics and keeping null-specific machinery small.

### Ruby — safe-navigation syntax

Ruby's `&.` skips exactly the guarded call when the receiver is `nil`, returns
`nil`, and does not evaluate the skipped method call's arguments. Chaining
requires repeated safe-navigation markers.

Reference:
- https://ruby-doc.org/3.4/syntax/calling_methods_rdoc.html

Contribution to D144: demonstrates the value of syntax when navigation chains
are a real problem, but also introduces evaluation rules beyond an ordinary
eager call. Current Protos evidence does not justify that grammar cost.

### Kotlin — typed nullable values, safe calls, Elvis, and let

Kotlin uses nullable types, `?.`, the Elvis operator `?:`, and `?.let { ... }`
for present-only action/transform. The right side of Elvis is evaluated only when
the left side is null.

Reference:
- https://kotlinlang.org/docs/null-safety.html

Contribution to D144: confirms that present-only transform and fallback are
distinct useful operations, but Kotlin's solution is coupled to a static nullable
type model Protos does not have.

### Swift — Optional institution plus chaining/coalescing

Swift represents absence through `Optional<Wrapped>`, uses optional chaining,
and provides `??` fallback.

References:
- https://docs.swift.org/swift-book/documentation/the-swift-programming-language/thebasics/
- https://docs.swift.org/swift-book/documentation/the-swift-programming-language/types/

Contribution to D144: mature evidence for explicit Optional values, while also
showing the extra institution that Protos would have to add merely to duplicate
the already-established role of canonical `null`.

### C# — null-conditional and null-coalescing operators

C# provides `?.`, `?[]`, `??`, and `??=`. Null-conditional access
short-circuits member/index operations; null-coalescing evaluates the fallback
only on null.

References:
- https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/null-safety/null-operators
- https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator

Contribution to D144: strong evidence for the convenience of syntax in a
null-heavy typed ecosystem, but also evidence that navigation, fallback, and
assignment are separate capabilities rather than one necessary bundle.

### JavaScript — optional chaining and nullish coalescing

JavaScript `?.` short-circuits on either `null` or `undefined` and returns
`undefined`; `??` also treats both as nullish.

References:
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing

Contribution to D144: useful contrast. Protos must not import JavaScript's
dual-nullish model, failed-lookup behavior, or `undefined` result.

### Python — explicit singleton identity checks

Python conventionally uses explicit `is None` / `is not None`, and PEP 8
specifically warns against truthiness when the semantic question is whether a
value is `None`.

Reference:
- https://peps.python.org/pep-0008/

Contribution to D144: strongest mature status-quo precedent. It validates the
correctness of explicit checks but not their ergonomic proportionality in current
Protos source.

### Rust — explicit Option value family

Rust `Option<T>` provides eager `unwrap_or` and lazy `unwrap_or_else`,
making the eager/lazy distinction explicit.

Reference:
- https://doc.rust-lang.org/core/option/enum.Option.html

Contribution to D144: confirms that fallback eagerness is observable and should
be explicit, but an Option family would duplicate Protos canonical `null`
without present evidence for the additional value institution.

## Candidate set

### Candidate A — status quo

Keep explicit `=== null` plus Boolean control.

This preserves zero new public surface but retains demonstrated repetition.

### Candidate B′ — two ordinary null-aware Object messages

Add exactly:

```text
ifNull(block)
ifNotNull(block)
```

as standard `Object` protocol behavior with exact receiver-identity tests and
ordinary polymorphic callback invocation.

This is the owner-approved candidate.

### Candidate C — one combined two-branch message only

Add conceptually:

```text
ifNullIfNotNull(nullBlock, valueBlock)
```

This minimizes selector count but requires both callback-producing arguments for
the common one-branch cases and is not justified by present two-way-branch
pressure.

### Candidate D — Standard Library Optional helper layer

Keep Core unchanged and expose helper functions.

Rejected for the selected design: it introduces a named helper institution while
losing the receiver-oriented ordinary-message fit demonstrated by the strongest
prototype/message precedent.

### Candidate E — `??` / `?.` syntax now

Rejected for the selected design. Current evidence does not justify permanent
lexer/parser/precedence/lowering/debugger/LSP surface or the different argument
suppression semantics of optional call syntax.

### Candidate F — Option/Maybe value family

Rejected for the selected design. It duplicates the established role of canonical
`null` and adds construction/unwrapping/combinator identity without a present
requirement.

## Comparative scorecard

Scores are 1–5. Confidence is HIGH unless noted. Scores are aids, not decision
authority.

| Criterion | A status quo | B′ two messages | C combined only |
|---|---:|---:|---:|
| Correctness / invariant preservation | 5 — existing exact checks | 5 — exact identity contract | 5 — exact identity contract |
| Protos alignment | 5 — ordinary mechanisms | 5 — ordinary messages | 5 — ordinary messages |
| Present-need proportionality | 2 — repeated boilerplate remains | 5 — matches observed one-branch/fallback use | 3 — forces unused branch production |
| Incremental growth | 5 — nothing foreclosed | 5 — combined/syntax can be additive | 5 — conveniences remain additive |
| Future-option resilience | 5 | 5 | 5 |
| Scalability | 5 | 5 — local dispatch only | 5 |
| Conceptual simplicity | 3 — simple primitive pieces, repetitive use | 5 — two dual selectors | 4 — one selector, two branches |
| Portability / implementation freedom | 5 | 5 | 5 |
| Runtime / resource cost | 4 — explicit identity + Boolean callback | 5 — one dispatch + reached callback | 4 — two callback arguments produced |
| Failure / operability | 5 | 5 — ordinary call failure | 5 |
| Deferral / reversibility / migration | 4 — current boilerplate persists | 5 — future syntax/combined form additive | 5 |
| Evidence maturity / implementation risk | 5 | 5 — repository + Smalltalk precedent | 4 — weaker current two-way pressure |

### Score rationale and red flags

Candidate A has an underengineering warning: current production source already
contains the repetition D144 was created to address.

Candidate C is coherent but solves selector-count minimalism rather than the
dominant usage shape. Its two eagerly evaluated callback-producing arguments are
unnecessary in the common one-branch case.

B′ has no non-compensating overengineering red flag. It adds two ordinary
selectors, no grammar, no value family, no hidden state, and no concurrency or
lookup institution.

## Owner-approved semantics — Candidate B′

The standard owner is `Object`.

```text
PUBLIC_OPERATIONS=
    ifNull(block)
    ifNotNull(block)

NULL_TEST=EXACT_CANONICAL_NULL_IDENTITY
TRUTHINESS=NO
DELEGATION_TO_NULL_COUNTS_AS_NULL=NO
```

### `ifNull(block)`

For exact canonical `null`:

```text
invoke block() exactly once
return its exact normal result
```

For every non-null receiver:

```text
do not callability-validate block
do not invoke block
return the exact receiver
```

### `ifNotNull(block)`

For exact canonical `null`:

```text
do not callability-validate block
do not invoke block
return canonical null
```

For every non-null receiver:

```text
invoke block(receiver) exactly once
return its exact normal result
```

The one supplied callback argument is the exact receiver object.

### Evaluation and invocation

Each method accepts exactly one supplied positional argument.

Receiver and callback-producing argument expressions are evaluated under the
ordinary left-to-right eager invocation rules before the selected standard
behavior executes. Creating a Closure value is therefore eager as an argument
expression, but its body is not executed until ordinary invocation reaches it.

Callability validation is path-sensitive: an unselected callback object is not
inspected for callability.

Selected callbacks use the ordinary polymorphic invocation protocol.

Errors, non-local returns, suspension/cancellation, and other non-normal control
propagate normally. A callback's normal result, including `null`, Boolean,
ordinary objects, or a Future, is returned unchanged. The protocol performs no
implicit await, Future adoption, conversion, wrapping, or hidden suspension.

## Important composition boundary

The selected protocol deliberately does not claim that:

```protos
x.ifNotNull(transform).ifNull(fallback)
```

is equivalent to a two-way branch on the original `x`.

If `x` is non-null but `transform(x)` returns `null`, the subsequent
`ifNull` observes that actual result and invokes `fallback`.

A future combined two-branch operation may be added if real usage demonstrates
the need. B′ does not predefine it.

## Missing-member and delegation boundaries

A missing member on a non-null receiver remains the ordinary failed-lookup
`Error`. D144 does not turn failed lookup into `null`.

An ordinary object delegating to canonical `null` is still a distinct
identity-bearing object. If inherited `Object.ifNull` or `Object.ifNotNull`
behavior is selected for that object, the original receiver is non-null and the
non-null branch applies.

Custom objects remain free to define or override these ordinary selector names.
Doing so does not make the object null and does not alter canonical-null identity.

## Interaction with D142

D142 continues to own expected Map-key absence.

```text
ABSENT != PRESENT_WITH_NULL
```

A Map operation that returns an actual stored `null` value returns canonical
`null`; D144 then governs only what a caller intentionally does with that value.
D144 introduces no Map absence sentinel and changes no Map lookup semantics.

## Incremental-design gates

### Pay for what you need

B′ adds only the two operations directly supported by present production
pressure. No grammar, Optional wrapper, navigation syntax, assignment syntax, or
combined two-way selector is paid for now.

### Grow as you need

A combined two-way selector, null-coalescing syntax, or optional navigation can
be evaluated later as additive surface. B′ does not require reserving syntax or
changing the null identity model.

### Cost of deferral

Deferring `?.` and `??` leaves ordinary message syntax slightly more verbose.
Adding such syntax later would require parser/tooling work, but not a change to
canonical-null identity, lookup failure, Map absence, object identity, or the B′
protocol.

Deferring a combined two-way selector requires explicit Boolean control in the
relatively rarer case where both original branches matter. That later addition
would also be additive.

### Smallest sufficient solution

Two ordinary messages are the smallest surface that directly covers the observed
present-only transform/action and lazy null-fallback shapes without forcing both
branches or adding syntax.

### Speculation burden

Long optional chains, optional assignment, Optional/Maybe values, and null-aware
index mutation are plausible features but have no demonstrated present pressure
sufficient to justify implementation today. B′ preserves escape paths without
prebuilding them.

## Future stress tests

### More complex optional navigation

If real applications develop repeated deep optional navigation, D144's ordinary
protocol remains valid. A later syntax decision can compare exact sugar/lowering
against then-current code without changing `ifNull`/`ifNotNull`.

### Futures and suspension

B′ creates no new scheduling boundary. A selected callback may suspend according
to ordinary invocation semantics; an unselected callback cannot suspend because
it is not invoked.

### Actors and Processes

The protocol carries no shared state, authority, or cross-domain identity. It is
local value/control behavior and therefore scales without new coordination.

### Alternative runtimes

The semantics depend only on canonical-null identity, ordinary dispatch, and
ordinary invocation. They do not depend on the JVM, Truffle, representation
tagging, allocation, or host null.

## Strongest argument against B′

A dedicated `?.`/`??` syntax can be much more concise and may suppress
evaluation of whole argument/member subexpressions in ways an ordinary eager
message call cannot reproduce directly.

That is a real capability difference, not merely punctuation.

The reason not to select it now is evidentiary: current Protos source demonstrates
one-branch and fallback repetition, not pervasive deep optional navigation. The
syntax can be reconsidered later without invalidating B′, while adding it now
would permanently increase grammar and tooling surface.

## GITHUB021 invariant/delta consistency

```text
CANONICAL_NULL_SINGLETON=PASS
NO_UNDEFINED=PASS
FAILED_LOOKUP_REMAINS_ERROR=PASS
NULL_DELEGATES_DIRECTLY_TO_OBJECT=PASS
DELEGATION_DOES_NOT_CONFER_NULL_IDENTITY=PASS
NO_TRUTHINESS=PASS
ORDINARY_MESSAGE_LOOKUP_AND_DISPATCH=PASS
ORDINARY_ARGUMENT_EVALUATION=PASS
ORDINARY_POLYMORPHIC_CALLBACK_INVOCATION=PASS
D142_ABSENT_DISTINCT_FROM_PRESENT_NULL=PASS
NO_OPTIONAL_MAYBE=PASS
NO_SYNTAX_CHANGE=PASS
NO_OPTIONAL_ASSIGNMENT=PASS
NO_OPTIONAL_INDEX_MUTATION=PASS
MATERIALLY_NEW_HIDDEN_CONSEQUENCE=NONE
DECISION_INVARIANT_CONSISTENCY=PASS
```

No recorded owner-approved invariant is reopened or contradicted.

## Owner approval

The project owner explicitly approved the exact Candidate B′ presented with the
semantics above in the active D144 interaction:

```text
apruebo B′
```

```text
DECISION_APPROVAL_PROVENANCE=PASS
```

## Deferred work

D144 intentionally leaves unselected:

- optional member/message chaining syntax;
- null-coalescing syntax;
- optional assignment;
- optional indexed mutation;
- optional slot creation;
- combined `ifNullIfNotNull`-class operation;
- eager value-fallback variant;
- Option/Maybe value family;
- failed-lookup-to-null conversion;
- generalized exception-swallowing navigation.

Before executable implementation proceeds, the selected semantics must be
published through the applicable normative specification owner(s), including
`spec/semantics/VALUES_AND_COLLECTIONS.md` and the global specification
changelog.
