# D145 — Closure caller-argument introspection and `args` intrinsic

## Decision state and authority

This is the non-normative decision packet for `guillermomolina/protos#574`.

```text
D145_STATUS=NEEDS_USER_DECISION
TRIGGER=AUD009-A2/#561
OWNER_APPROVED_AUDIT_CLASSIFICATION=ARGS_INTRINSIC_REMOVE_NOW_RECONSIDER_LATER
PROTOS_REVISION=b1b5c91b365a57ed65797b78ab9a6466e7df16f5
PROJECT_DOCS_BASE=1c5be587992ae82a43b5793b90de57c90276deab
```

This packet does not ratify a candidate. D145 remains open until the project owner
explicitly approves the exact semantic choice.

## Exact problem

Current Core reserves bare `args` as an intrinsic pseudo-identifier in every Closure
activation.

It exposes a fresh frozen standard Array containing the complete flattened
caller-supplied positional vector:

- receiver excluded;
- default-produced values excluded;
- spread elements already flattened;
- a trailing desugared Closure is included as an ordinary supplied element;
- nested Closure invocations receive independent `args` values.

Current Core also has explicit rest parameters and call spread.

The design question is therefore not whether Protos needs variadic calls or forwarding.
Those already exist. The exact residual capability is:

> Should ordinary Closure code always have ambient access to the original supplied
> positional vector, including the ability to distinguish an omitted defaulted
> argument from an explicitly supplied equal value?

For:

```protos
f: (x = 42) => {
    ...
}
```

current `args` distinguishes:

```text
f()    -> args.size() == 0
f(42)  -> args.size() == 1
```

while the bound parameter is `x == 42` in both calls.

## Owner-approved invariant

AUD009-A2 established:

```text
ARGS_INTRINSIC = REMOVE_NOW_RECONSIDER_LATER
```

That classification is an owner-approved routing invariant for D145. It is a strong
removal signal, but it is not final semantic approval.

Any candidate that retains the current ambient intrinsic must therefore explicitly show
why the A2 classification should be reopened.

## Current repository evidence

### Guest-language usage

At the audited revision, GitHub code search finds 32 `.protos` files containing `args`.

After classification:

- Standard Library bare intrinsic usage: none found;
- Tool bare intrinsic usage: none found;
- Tool/process hits are `process.args()`, a separate ordinary Process API;
- non-test bare intrinsic use is concentrated in:
  - `protos/tutorials/07-call-arguments/01-defaults-args-and-rest.protos`;
  - `protos/examples/closures/rest-arguments.protos`;
- the remaining bare-intrinsic uses are conformance/regression evidence for the
  existing semantics.

The current feature is therefore tested and documented, but no real Standard Library or
Tool code currently depends on it.

### Source-surface footprint

`args` is not an ordinary identifier.

It has dedicated source/runtime representation:

- lexer reserved word / `TokenType.ARGS`;
- parser `SurfaceIntrinsic.Kind.ARGS`;
- canonical `CanonicalIntrinsic.Kind.ARGS`;
- execution lowering and Bytecode intrinsic loading;
- normative grammar / call semantics / execution-control text;
- dedicated conformance and Java tests.

Removing the intrinsic would also make `args` available as an ordinary local/parameter
name.

A small correction to the opening D145 text is worth preserving: examples such as

```protos
forward: (...args) => target(...args)
```

describe the natural post-removal spelling, but are not valid under the current baseline
because the lexer still reserves `args`. Current code must use an ordinary identifier such
as `items` or `rest`.

### Runtime footprint

Current `ProtosActivation.forClosureInvocation(...)` and immediate method invocation
materialize:

```text
prelude.newFrozenArray(supplied)
```

for the activation's `arguments`.

The Bytecode parameter binder also reads `activation.arguments()` as the source vector for
arity/default/rest binding.

Therefore removal does not imply that an implementation can forget the caller-supplied
vector before parameter binding: binding still needs it.

The semantic simplification is narrower and more important:

> Core would no longer require the complete supplied vector to exist as a guest-visible,
> fresh, frozen standard Array for every activation.

An implementation may still use an Array internally if convenient. Conversely, retention
of `args` does not logically force eager materialization if an implementation can preserve
fresh identity and exact semantics through lazy allocation. Current implementation cost is
evidence, not semantic authority.

## Comparative evidence

### Self — declared argument slots

Self method activations contain declared argument/local slots. Actual arguments initialize
the corresponding argument slots. The normal language model is explicit signature binding,
not a universal ambient original-argument vector.

Source:
https://handbook.selflanguage.org/2024.1/langref.html

### Smalltalk / Pharo — explicit arguments, reflective contexts separately

Blocks/methods bind declared arguments. Pharo also exposes execution contexts through
`thisContext`, and method contexts contain receiver and arguments, but that is reflective
execution-state access rather than a mandatory convenient argument Array for normal code.

Source:
https://books.pharo.org/deep-into-pharo/

### JavaScript — ambient `arguments` plus modern rest

JavaScript is the strongest direct precedent for the current Protos design:
non-arrow functions receive ambient `arguments`, which contains the supplied arguments and
therefore preserves supplied/default distinction.

Modern JavaScript also has explicit rest parameters, and MDN explicitly recommends rest
parameters for modern code.

Sources:
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters

Important lesson: ambient caller-vector introspection is a real capability, but rest makes
its ordinary variadic/forwarding role largely redundant. The remaining value is call-shape
introspection and legacy/reflection-like behavior.

### Python — signature-declared varargs, reflection separate

Python captures extra positional arguments through explicit `*args`. Signature objects and
bound-argument reflection exist through `inspect`, but the function body does not receive
a universal ambient original call vector merely because the function has defaults.

`inspect.BoundArguments` also distinguishes bound arguments from defaults until defaults
are explicitly applied, demonstrating that suppliedness can live in an explicit
introspection layer rather than universal normal-call state.

Source:
https://docs.python.org/3/library/inspect.html

### Ruby — explicit splat

Ruby assigns extra positional arguments only when the method declares a `*argument`
parameter. Defaults and splat are part of the signature rather than a second ambient
caller-vector API.

Source:
https://docs.ruby-lang.org/en/3.3/syntax/methods_rdoc.html
https://ruby-doc.org/3.4/syntax/calling_methods_rdoc.html

### Lua — explicit vararg functions

Lua exposes `...` only in functions declared variadic. The vararg expression represents
the extra actual arguments; ordinary functions do not receive universal argument-vector
introspection.

Source:
https://www.lua.org/manual/5.4/manual.html

### Io — explicit counterexample: ambient call object

Io method/block activations expose a `call` object with sender, message, target and
argument-message access. Io can inspect and selectively evaluate unevaluated argument
messages and uses that capability to implement control structures as ordinary messages.

Source:
https://iolanguage.org/docs/Guide/index.html

Io therefore demonstrates that rich ambient call metadata can be coherent in a
prototype/message language. It is not direct evidence that Protos needs it: Io's call
object supports a broader language model in which the receiver controls evaluation of
message arguments. Protos already fixes eager left-to-right argument evaluation before
activation creation.

## Candidate set

### Candidate A — remove the `args` intrinsic

Delete guest-visible ambient `args`.

Keep:

- required/default/rest parameter binding;
- call spread;
- internal caller-supplied vector as required by binding;
- debugger/runtime-internal argument information where useful;
- `process.args()` unchanged.

After removal, `args` becomes an ordinary identifier.

The exact original supplied vector is no longer available to ordinary Closure code unless
it is representable by explicitly declared parameters/rest.

### Candidate B — retain current semantics

Keep universal ambient `args` with the current exact contract.

Implementation is free to optimize/lazily materialize it as long as all observable
fresh-frozen-Array semantics remain identical.

This preserves suppliedness and the complete flattened positional vector in every Closure.

### Candidate C — remove ambient `args`, add explicit opt-in call metadata now

Delete the reserved intrinsic but introduce an explicit per-Closure mechanism for call
metadata / original supplied vector.

Possible shapes include a call-metadata parameter/marker or explicit activation capability.
No spelling is implied here.

This preserves the rare capability without charging every Closure's conceptual surface,
but creates a new institution despite no current production requirement.

### Candidate D — replace full-vector introspection with parameter suppliedness

Expose only whether a particular defaulted parameter was supplied by the caller.

This targets the one residual capability that rest cannot express, but adds a parameter
binding concept and does not preserve arbitrary original-vector inspection.

### Candidate E — explicit full-vector capture in the signature

Remove ambient `args`, but allow a Closure to explicitly request the complete original
supplied positional vector as a special signature element.

This is narrower than a general call-metadata object and more exact than parameter-level
suppliedness, but it still adds new syntax/parameter semantics solely to preserve an
unused capability.

## Candidate scorecard

Scores are 1–5. They are comparison aids, not decision authority.

### A — remove ambient `args`

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 5 | HIGH | Preserves call evaluation, binding, defaults, rest and spread; removes only the explicitly reviewed introspection capability. |
| Protos alignment | 5 | HIGH | Replaces an ambient reserved pseudo-binding with explicit signature/message mechanisms. |
| Present-need proportionality | 5 | HIGH | No stdlib/tool use demonstrates need for original-vector introspection. |
| Incremental growth | 5 | HIGH | A later explicit opt-in mechanism can be added if real code needs suppliedness. |
| Future-option resilience | 4 | HIGH | Frees the `args` name; old spelling should not be promised later, but the capability can return under a better explicit model. |
| Scalability | 5 | HIGH | No universal guest-visible per-activation argument object is semantically required. |
| Conceptual simplicity | 5 | HIGH | One argument-binding model: declared parameters/rest. |
| Portability / implementation freedom | 5 | HIGH | Runtime may represent the supplied vector however binding needs without guest identity constraints. |
| Runtime / resource cost | 5 | MEDIUM | Enables removal/lazy avoidance of guest Array materialization, though binder still needs internal supplied data. |
| Failure / operability | 5 | HIGH | Removes special intrinsic lookup/identity cases; ordinary binding failures remain unchanged. |
| Deferral / reversibility / migration | 4 | HIGH | Reintroduction can be additive under another explicit API; removal breaks existing external bare-`args` source now. |
| Evidence maturity / implementation risk | 5 | HIGH | Repository evidence is strong; multiple mature languages use explicit parameter/vararg models. |

**Overengineering red flag:** none.

**Underengineering red flag:** only if exact supplied/default distinction is a latent Core
requirement. No production evidence currently demonstrates that.

### B — retain current ambient `args`

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 5 | HIGH | Preserves all current semantics exactly. |
| Protos alignment | 2 | HIGH | Ambient unshadowable invocation state is more privileged than ordinary parameter binding. |
| Present-need proportionality | 1 | HIGH | No current stdlib/tool code uses the unique capability. |
| Incremental growth | 3 | MEDIUM | It is broad enough for future use, but broadness is installed before need. |
| Future-option resilience | 3 | MEDIUM | Preserves supplied-vector introspection but permanently consumes a common reserved name and semantic concept. |
| Scalability | 4 | MEDIUM | Semantics are scalable, though every activation conceptually owns a distinct argument Array. |
| Conceptual simplicity | 2 | HIGH | Adds a second ambient view in addition to named/default/rest binding. |
| Portability / implementation freedom | 4 | MEDIUM | Lazy materialization is possible, but fresh Array identity must remain observable when read. |
| Runtime / resource cost | 3 | MEDIUM | Current implementation allocates eagerly; optimization can mitigate but cannot erase observable fresh identity. |
| Failure / operability | 4 | HIGH | Existing behavior is known and tested, but keeps intrinsic-specific machinery. |
| Deferral / reversibility / migration | 2 | HIGH | Retaining now makes later removal more expensive as external usage grows. |
| Evidence maturity / implementation risk | 5 | HIGH | JavaScript and Io prove the model is implementable and coherent. |

**Overengineering red flag:** HIGH. Current users pay public/conceptual surface for a
capability with no production use.

### C — explicit opt-in call metadata now

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 5 | MEDIUM | Can preserve suppliedness exactly if specified carefully. |
| Protos alignment | 3 | MEDIUM | Opt-in is better than ambient state, but introduces a dedicated call-metadata institution. |
| Present-need proportionality | 1 | HIGH | No real current code requires it. |
| Incremental growth | 5 | HIGH | Could grow to richer metadata if later justified. |
| Future-option resilience | 5 | HIGH | Preserves future call-introspection directions. |
| Scalability | 4 | MEDIUM | Opt-in limits cost to requesting Closures. |
| Conceptual simplicity | 3 | MEDIUM | Adds a second explicit call model next to ordinary parameters. |
| Portability / implementation freedom | 4 | MEDIUM | Requires retaining original call-shape metadata only when requested. |
| Runtime / resource cost | 4 | MEDIUM | Pay-for-use is plausible. |
| Failure / operability | 4 | MEDIUM | New metadata validity/lifetime rules would need specification. |
| Deferral / reversibility / migration | 5 | HIGH | Crucially, this mechanism can be added later at bounded cost. |
| Evidence maturity / implementation risk | 3 | MEDIUM | Io/Pharo show richer metadata models, but no Protos requirement selects one. |

**Overengineering red flag:** HIGH. This preimplements the reconsideration path before the
trigger occurs.

### D — parameter-level suppliedness

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 4 | MEDIUM | Solves omitted-vs-explicit default, but not complete original-vector inspection. |
| Protos alignment | 3 | MEDIUM | Keeps information near parameters but complicates binding semantics. |
| Present-need proportionality | 1 | HIGH | No production code needs suppliedness today. |
| Incremental growth | 3 | MEDIUM | Full call metadata later would still be a separate design. |
| Future-option resilience | 3 | MEDIUM | Optimizes around one guessed future use case. |
| Scalability | 5 | HIGH | Local per-parameter state is bounded. |
| Conceptual simplicity | 3 | MEDIUM | Adds supplied/defaulted state to parameter semantics. |
| Portability / implementation freedom | 5 | HIGH | Easy to represent internally. |
| Runtime / resource cost | 5 | HIGH | Could be represented by small binding metadata only when requested. |
| Failure / operability | 4 | MEDIUM | New access rules and default interaction would need definition. |
| Deferral / reversibility / migration | 5 | HIGH | No reason it cannot be added later. |
| Evidence maturity / implementation risk | 3 | MEDIUM | Analogues exist in reflection/binding APIs, but not as a demonstrated Protos need. |

**Overengineering red flag:** HIGH. It invents a narrower institution for an unobserved
requirement.

### E — explicit full-vector signature capture

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 5 | MEDIUM | Can preserve exactly the current vector when opted in. |
| Protos alignment | 4 | MEDIUM | Explicit request is more local than ambient `args`. |
| Present-need proportionality | 1 | HIGH | No current production use. |
| Incremental growth | 4 | MEDIUM | Could later grow to richer metadata, though syntax choice may constrain it. |
| Future-option resilience | 4 | MEDIUM | Preserves exact vector access without reserving it universally. |
| Scalability | 4 | MEDIUM | Pay-for-use materialization is natural. |
| Conceptual simplicity | 3 | MEDIUM | Adds another special parameter category beyond required/default/rest. |
| Portability / implementation freedom | 5 | HIGH | Straightforward implementation shape. |
| Runtime / resource cost | 4 | MEDIUM | Only requesting Closures need guest-visible vector materialization. |
| Failure / operability | 4 | MEDIUM | Mostly ordinary binding, but syntax/ordering rules would expand. |
| Deferral / reversibility / migration | 5 | HIGH | Can be introduced later without preserving current `args` spelling. |
| Evidence maturity / implementation risk | 3 | MEDIUM | Technically credible but not directly selected by current precedent or usage. |

**Overengineering red flag:** HIGH. It adds syntax now solely to preserve hypothetical
future introspection.

## Stress tests

### Ordinary forwarding

Requirement:

```protos
forward: (...items) => {
    target(...items)
}
```

Candidate A fully preserves this. Ambient `args` is unnecessary.

### Named/defaulted wrapper

Requirement:

```protos
f: (x = 42, ...tail) => {
    ...
}
```

A can observe `x` and `tail` but cannot distinguish `f()` from `f(42)` when the resulting
bound values are identical.

No repository production use demonstrates that distinction.

### Generic proxy/decorator

A truly generic forwarding wrapper can declare only rest:

```protos
proxy: (...items) => target(...items)
```

No ambient original vector is needed.

If a future proxy must expose both a meaningful signature and exact original call shape,
that is precisely the reconsideration trigger.

### Future named arguments

Preserving today's positional-only ambient Array is not obviously the correct foundation
for hypothetical future named arguments.

If named arguments are ever introduced, exact call-shape introspection would need to
decide whether names, ordering, omitted defaults, duplicate/error state and other metadata
are represented.

That future therefore strengthens the case for deferring call metadata rather than
freezing today's positional Array as its permanent base.

### Debugger/tooling

Debugger/runtime tooling can retain host/internal access to supplied arguments without
making that information a guest-language intrinsic.

Removing guest `args` does not require deleting all runtime observability.

## Smallest-sufficient analysis

Current requirements are satisfied by:

```text
required parameters
default parameters
rest parameter
call spread
```

No current Standard Library or Tool requirement needs:

```text
complete original supplied vector inside a Closure with named/default parameters
```

Therefore Candidate A is the smallest sufficient current language.

Candidates C, D and E preserve credible future options but preimplement them before the
AUD009 reconsideration trigger occurs.

## Deferral cost

If Candidate A is selected and a future real need appears, Protos can add an explicit
opt-in mechanism such as:

- call metadata;
- parameter suppliedness;
- explicit complete-vector capture;
- another then-current design.

The foundational call evaluation/binding model need not change: the runtime already forms
the caller-supplied vector before binding.

The main irreversible compatibility consequence is only this:

> once `args` becomes an ordinary identifier, re-reserving the bare word `args` later would
> break valid post-removal source.

That argues against promising restoration of the old spelling, not against removing the
capability now. The inherited AUD009 trigger already says to reconsider from the
then-current call model and not assume old `args` must return.

## Strongest counterargument to Candidate A

JavaScript demonstrates a real use for ambient original call data: `arguments.length` can
distinguish omitted/defaulted parameters and supports wrappers that want the exact call
shape.

The strongest objection is therefore:

> removing `args` destroys information that cannot always be reconstructed from bound
> parameters/rest.

That objection is technically correct.

The response is not that rest is semantically identical. It is that no current Protos
production requirement needs the lost information, while retaining it universally imposes
a permanent reserved pseudo-identifier and a second invocation-view concept.

If the requirement becomes real, adding an explicit opt-in call-shape mechanism is a
bounded extension that does not require changing eager argument evaluation, defaults,
rest, spread or Closure invocation authority.

## GITHUB021 invariant / delta check

Applicable owner-approved invariant:

```text
ARGS_INTRINSIC = REMOVE_NOW_RECONSIDER_LATER
```

Candidate A:

```text
PRESERVES_INVARIANT=YES
NEW_SEMANTIC_DELTA=args ceases to be guest-visible intrinsic and becomes ordinary identifier
HIDDEN_REOPENING=NO
```

Candidate B:

```text
PRESERVES_INVARIANT=NO
REQUIRES_EXPLICIT_REOPENING=YES
```

Candidates C/D/E:

```text
PRESERVES_REMOVAL_OF_CURRENT_INTRINSIC=YES
BUT_PREIMPLEMENT_RECONSIDERATION_CAPABILITY=YES
OWNER_APPROVAL_REQUIRED=YES
```

No other owner-approved D145 invariant has been identified.

## Proposal pending owner approval

**Proposed candidate: A — remove the ambient reserved `args` intrinsic.**

Exact proposed decision:

```text
D145_CANDIDATE=A
ARGS_INTRINSIC=REMOVE
ARGS_IDENTIFIER_AFTER_REMOVAL=ORDINARY_IDENTIFIER
REQUIRED_PARAMETERS=UNCHANGED
DEFAULT_PARAMETERS=UNCHANGED
REST_PARAMETERS=UNCHANGED
CALL_SPREAD=UNCHANGED
PROCESS_ARGS_API=UNCHANGED
DEBUGGER_INTERNAL_ARGUMENT_VISIBILITY=NOT_GUEST_SEMANTICS
CALLER_SUPPLIED_VECTOR_INTERNAL_BINDING_NEED=UNCHANGED
EXPLICIT_CALL_METADATA=NOT_ADDED
PARAMETER_SUPPLIEDNESS=NOT_ADDED
FULL_VECTOR_CAPTURE_MARKER=NOT_ADDED
RECONSIDER_TRIGGER=REAL_PRODUCTION_NEED_FOR_EXACT_ORIGINAL_CALL_SHAPE
OLD_ARGS_SPELLING_RESERVED_FOR_FUTURE=NO
```

The reconsideration trigger remains:

> Real Tool/stdlib/application code repeatedly needs the exact original supplied
> positional vector in addition to a meaningful named/default/rest signature,
> especially to distinguish omitted defaulted arguments from explicitly supplied ones.

At that point Protos should compare the then-current explicit alternatives rather than
automatically restoring the old ambient `args` intrinsic.
