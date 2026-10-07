# D146 — Fresh generic Error signaling ergonomics decision packet

## Decision state and authority

This is the non-normative decision packet for `guillermomolina/protos#576`.

```text
D146_STATUS=NEEDS_USER_DECISION
TRIGGER=AUD011/#540
PROTOS_REVISION=f61bd24cf1e935591c00a07832e006fb7a256828
PROJECT_DOCS_BASE=cd77d688e1142302389cdfe05046583457d91610
D005_REMAINS_AUTHORITATIVE=YES
IMPLEMENTATION_AUTHORIZED=NO
NORMATIVE_PUBLICATION_AUTHORIZED=NO
```

No candidate is selected by this record.

## Exact problem

Production Protos source repeatedly needs one exact operation:

```text
create one fresh ordinary generic Error occurrence
then signal that exact object
```

The direct source spelling is:

```protos
Error().signal()
```

and many production modules independently introduce:

```protos
fail: () => {
    Error().signal()
}
```

Repository-wide GitHub code search at the audited revision finds:

```text
FILES_CONTAINING_Error().signal()=133
FILES_CONTAINING_ZERO_ARG_fail_BINDING_SHAPE=30
```

The latter pattern appears across networking, URI, JSON, TOML, CLI, Package
Tool and Test Tool code. Some files define several independent local `fail`
helpers.

Not every local helper is semantically identical. Some intentionally perform
additional local state changes before signaling, for example:

```protos
fail: () => {
    failed = true
    Error().signal()
}
```

Such richer helpers are not evidence for replacing all local `fail` operations
with one global shorthand.

The decision is therefore narrowly:

> Is shortening the recurring exact operation `Error().signal()` valuable
> enough to consume one standard Core/prelude API name, and if so, what ordinary
> surface preserves D005 exactly without introducing new throw/raise syntax,
> payload semantics, taxonomy, rethrow semantics, or custom-subtype machinery?

## Current normative constraints

### D005 freshness and identity

`spec/semantics/ERRORS.md` already defines:

- each standard failure occurrence as a fresh ordinary Error object;
- generic standard failure as a fresh object whose immediate parent is the
  canonical standard `Error` prototype;
- no singleton standard failure instances;
- observable identity preservation through `===`, identity hashing, handlers,
  storage, Futures and ordinary reachability;
- re-signaling an existing Error as signaling exactly that Error;
- `Error` and other standard Error prototypes themselves as legal explicit
  receivers of `signal()`;
- `Error.signal()` as signaling the shared `Error` prototype itself;
- non-resumable signaling;
- a deliberately shallow portable Error taxonomy;
- no implicit String-to-Error, prototype-to-instance or other condition
  designator coercion;
- no required ordinary guest-visible stack/message/cause payload merely because
  an Error was signaled.

D146 must preserve all of these.

In particular, a convenience must **not** implement fresh generic failure as:

```protos
Error.signal()
```

because that expression has an already-ratified, observably different meaning.

### Existing prelude semantics

The standard prelude is:

- part of ordinary lexical lookup;
- frozen;
- readable by modules;
- explicitly shadowable through ordinary local slot creation with `:`;
- not a separate global-variable namespace.

Therefore adding a prelude binding named `fail` has a real but bounded
compatibility consequence:

- a module-local `fail: ...` still wins;
- a previously-unbound bare lookup of `fail` becomes bound;
- unqualified assignment that resolves only to standard `fail` reaches the
  ordinary frozen-prelude assignment rule instead of the previous absent-name
  path.

This cost must be approved if the prelude candidate is selected.

## Repository evidence classification

### Direct generic-failure repetition

Examples whose local helper is exactly:

```protos
fail: () => {
    Error().signal()
}
```

occur in production code including:

- `protos/lib/network/IpAddresses.protos`;
- `protos/lib/network/IpEndpoints.protos`;
- `protos/lib/uri.protos`;
- `protos/lib/json/JSON.protos`;
- `protos/lib/toml/TOML.protos`;
- `protos/lib/cli/CommandLine.protos`;
- many `protos/tools/package/*.protos`;
- multiple Test Tool modules.

This demonstrates both repetition and an independently converged name:
`fail`.

### Direct one-off generic failure

Many additional production and test call sites use `Error().signal()`
directly rather than defining a helper.

The direct spelling remains valid under every candidate except an explicitly
breaking redesign, which D146 does not consider.

### Rich local failure helpers

Some helpers mutate local parser/stream state before signaling.

Those must remain local behavior unless a separate design explicitly owns the
state transition.

### Re-signaling existing errors

Code such as:

```protos
error.signal()
```

must never migrate to a fresh-failure shorthand because it preserves exact error
identity and possibly a more specific category.

### Assertions

`std:test/Assertions` defines an `AssertionFailure` category and distinguishes
assertion failure from generic malformed-test usage.

A Core `fail()` shorthand must always mean **fresh generic Error**, not
AssertionFailure. D146 does not add or reserve `Assertions.fail()`.

## Comparative research

### Pharo / Smalltalk — factory-side fresh signaling

Pharo supports both:

```text
ZeroDivide new signal
ZeroDivide signal
```

The class-side convenience creates a fresh exception instance and signals it.
Pharo explicitly motivates fresh instances as occurrence-specific carriers.

Primary source:

- https://books.pharo.org/deep-into-pharo/

Contribution to D146:

- creation + signal is a coherent unit of convenience;
- preserving one fresh occurrence per signal matters;
- Protos cannot reuse the exact selector `Error.signal()` for factory-side
  convenience because D005 already gives that spelling exact-object semantics.

### Io — prototype-side ordinary message

Io raises exceptions through:

```io
Exception raise("generic foo exception")
```

and re-raising is separately expressed through `pass`.

Primary source:

- https://iolanguage.org/docs/Guide/index.html

Contribution:

- a message-oriented language does not need a throw keyword;
- prototype-side convenience is coherent;
- Io couples raising to text payloads and has resumable/reflective behavior that
  Protos deliberately does not inherit.

### Self — ordinary message-oriented programmer error path

Self's documented primitive-failure path can explicitly call the standard
`error:` method when a primitive failure should abort rather than recover.

Primary source:

- https://handbook.selflanguage.org/SelfHandbook2017.1.pdf

Contribution:

- error/failure convenience can remain ordinary message protocol rather than new
  syntax;
- Self's payload-oriented `error:` model does not establish a current need for
  message payloads in Protos.

### Ruby — ubiquitous bare callable `raise` / `fail`

Ruby's `Kernel` methods are available in functional form; `fail` is an alias
for `raise`. Ruby's facility can create a generic RuntimeError with no explicit
exception and also supports exception class, message, backtrace, cause and
re-raise behavior.

Primary source:

- https://ruby-doc.org/3.4/Kernel.html

Contribution:

- `fail` is established vocabulary for universally available failure;
- a bare callable can be ergonomic without new syntax;
- Ruby's many overload roles are evidence **against** importing more than the
  zero-argument behavior currently demonstrated by Protos.

### Python — dedicated statement with class/instance and re-raise semantics

Python's `raise` statement can raise an exception instance, instantiate an
exception class, or re-raise the active exception when used bare.

Primary source:

- https://docs.python.org/3/reference/simple_stmts.html

Contribution:

- dedicated syntax becomes responsible for multiple exception semantics;
- Protos already has ordinary-object `signal()` for exact existing objects and
  has no evidence requiring a new statement or implicit active-error context.

### JavaScript — throw an evaluated value

JavaScript's `throw expression` transfers control using the evaluated value;
common practice is to throw an Error instance.

Primary source:

- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/throw

Contribution:

- dedicated syntax mostly privileges the transfer operation;
- Protos already models transfer as an ordinary Error message, so syntax merely
  to shorten fresh generic creation is disproportionate.

### Java — throw syntax over nominal Throwable values

Java uses a `throw` statement requiring a `Throwable` object and has a broad
nominal exception hierarchy plus checked-exception declaration rules.

Primary source:

- https://docs.oracle.com/javase/tutorial/essential/exceptions/throwing.html

Contribution:

- throw syntax is coherent inside a much larger nominal exception institution;
- that architecture is not evidence for introducing syntax into Protos's
  ordinary-object Error model.

### C# — throw/rethrow syntax and nominal Exception hierarchy

C# uses `throw expression` for Exception values and bare `throw;` inside
`catch` for rethrowing the currently handled exception.

Primary source:

- https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/exception-handling-statements

Contribution:

- rethrow is a distinct dynamic-context operation in syntax-based systems;
- D146 has no need to merge Protos's explicit exact-object re-signaling into a
  generic convenience.

### Rust — explicit Result versus panic

Rust distinguishes anticipated recoverable failure through `Result<T,E>` from
panic, whose public macro constructs and propagates a panic.

Primary sources:

- https://doc.rust-lang.org/stable/core/error/index.html
- https://doc.rust-lang.org/stable/std/macro.panic.html

Contribution:

- error ergonomics should not collapse expected-value failures and exceptional
  transfer;
- D146 should remain solely an ergonomic layer over already-ratified Protos
  signaling.

### Go — error values versus panic/recover

Go conventionally uses returned error values for ordinary failures, while
`panic` begins stack unwinding and `recover` can stop it in a deferred
function.

Primary sources:

- https://go.dev/wiki/PanicAndRecover
- https://go.dev/doc/effective_go

Contribution:

- a concise exceptional operation does not imply that every domain failure
  should use it;
- D146 must not broaden generic Error signaling into an all-purpose error-return
  model.

## Candidate set

### Candidate A — no change

Keep:

```protos
Error().signal()
```

and continue allowing local `fail` helpers.

### Candidate B-prime — zero-argument standard prelude `fail`

Add exactly one frozen-prelude binding:

```text
fail
```

whose value is an ordinary callable standard facility.

```protos
fail()
```

means exactly:

> produce one fresh ordinary generic Error occurrence in the current execution
> domain, with immediate parent equal to the canonical standard Error prototype
> of the current Core environment, then enter the already-ratified non-resumable
> Error signaling operation with that exact fresh object.

No argument, payload, cause, category, message, rethrow or subtype behavior is
added.

This is the recommended candidate.

### Candidate C — exact canonical `Error.fail()`

Add one ordinary standard behavior on the exact canonical `Error` owner:

```protos
Error.fail()
```

It creates one fresh child of canonical `Error` and signals it.

It does not reinterpret `Error.signal()`.

### Candidate C2 — polymorphic Error-prototype factory signaling

Add a behavior conceptually allowing:

```protos
Error.fail()
MyDomainError.fail()
```

where the receiver becomes the immediate parent/category of the fresh signaled
occurrence.

This creates a general factory protocol for user Error prototypes.

### Candidate D — imported library helper

Require an import/qualification such as:

```protos
Failures: import("std:errors/Failures")
Failures.fail()
```

or an equivalent library module.

### Candidate E — dedicated `raise` / `throw` / `fail` syntax

Introduce a language-level control-flow form.

This is the explicit privileged-syntax baseline.

### Candidate F — generalized global raise facility

Add a global/prelude callable accepting some combination of:

- no arguments for generic Error;
- existing Error values;
- Error prototypes;
- message text;
- causes;
- dynamic-context rethrow.

This is the ordinary-call analogue of Ruby/Python's richer raise family.

## Rejected/non-surviving structural variants

### `Object.fail()`

Putting failure behavior on `Object` makes the selector available through
nearly every ordinary object delegation chain yet does not solve module-top-level
bare-name use cleanly.

It pollutes the universal receiver protocol more broadly than one explicit
prelude binding. It is not retained.

### `Context.fail()`

Putting failure on execution contexts couples a generic Error operation to one
incidental receiver/lookup route and still does not improve the demonstrated
ordinary bare-helper pattern enough to justify the conceptual association.

It is not retained.

### `Error.raise()` / `Error.newSignal()`

These are spelling variants of Candidate C rather than materially distinct
semantic models.

`newSignal` describes implementation sequence rather than intent.
`raise` imports alternate exception vocabulary despite Protos already using
`signal` normatively.
`fail` is therefore the clearest C spelling if C were selected.

## Candidate B-prime exact contract

### Binding

The standard frozen prelude gains exactly one new binding named:

```text
fail
```

It is ordinary lookup surface, not a keyword, reserved identifier or intrinsic
syntax form.

The bound value is an ordinary semantically immutable standard callable
facility. It may be extracted/passed/stored like another ordinary callable:

```protos
f: fail
f()
```

and retains the same standard behavior.

A module may explicitly shadow it:

```protos
fail: () => {
    localState = true
    Error().signal()
}
```

The local binding wins normally.

### Arity and evaluation

The standard `fail` callable accepts exactly zero supplied arguments.

D146 defines no user arguments and therefore introduces no new user-argument
evaluation-order rule.

A call with supplied arguments follows the ordinary standard callable
arity-failure contract and does **not** first perform a generic `fail()`
occurrence.

### Fresh occurrence

Every reached valid invocation creates one fresh generic standard Error
occurrence.

Observable contract:

```text
immediate parent = canonical standard Error prototype
fresh identity per reached fail() invocation
no pooling/reuse
no visible mandatory payload slots
```

Freshness remains semantic rather than a mandate for physical allocation when
identity cannot be observed, exactly as D005 already permits.

### Canonical Error provenance

The created generic occurrence uses the canonical standard `Error` prototype
associated with the current Core execution/prelude environment.

The operation does **not** perform caller lexical lookup of the name `Error`.

Therefore:

```protos
Error: someOtherObject
fail()
```

still signals a fresh standard generic Error.

This is a material observable consequence and part of the candidate requiring
owner approval.

Reason:

- a standard prelude facility must keep its standard meaning after extraction or
  caller-local shadowing;
- otherwise the same extracted standard callable would change category based on
  an unrelated caller-local name;
- local code that intentionally wants another error prototype remains explicit.

### Signaling

The exact fresh Error occurrence enters the existing semantic signaling
operation.

It:

- never signals canonical `Error` itself;
- never reuses an earlier occurrence;
- never returns normally;
- uses existing dynamic handler selection;
- abandons the signaling continuation;
- preserves existing `ensure` behavior;
- preserves Future failure recording/re-signaling behavior;
- preserves Actor/P/boundary rules from the existing Error specifications.

The operation does not create a second signaling mechanism.

### No user-overridable intermediate dispatch requirement

The normative contract is fresh generic standard Error occurrence + existing
semantic signal.

Implementations are free to realize this with ordinary source behavior, an
ordinary native callable, specialization, inlining, or another conforming
mechanism.

D146 does not require a user-overridable intermediate lookup of a caller-visible
`Error` binding or an arbitrary candidate object's `signal` slot.

### Result

A valid reached `fail()` invocation never returns normally.

There is therefore no normal result value to standardize.

### Error taxonomy and payload

D146 adds no Error prototype and no new visible Error slot.

Specifically not added:

- message;
- cause;
- code;
- stack;
- source;
- location;
- payload/data;
- checked/unchecked distinction;
- generic-failure subtype.

Portable handler matching for the created occurrence promises exactly `Error`
unless some separately applicable rule explicitly establishes otherwise.

### Source/debugger/stack provenance

D005 continues to permit implementation-private diagnostic stack metadata only
when it is not exposed as extra Core object structure/taxonomy.

D146 adds no guest-visible stack API.

For debugging/source presentation, the user-authored `fail()` call is the
semantic request site. A debugger may also show a standard-facility frame under
its normal frame-visibility policy; D146 does not create a special hidden-frame
contract.

An implementation may inline/specialize the facility so long as normal source
semantics, Error identity and debugger contracts remain correct.

### Assertion interaction

Bare standard:

```protos
fail()
```

always means fresh generic standard Error.

It never means AssertionFailure.

Existing or future qualified test-library behavior such as:

```text
Assertions.require(...)
Assertions.signals(...)
Assertions.fail(...)   // only if separately designed later
```

remains library-owned and does not alter Core `fail`.

### Prelude-name compatibility

After B-prime, a previously absent bare `fail` lookup is present.

This affects source that intentionally relied on the absence of that name.

Existing source declaring:

```protos
fail: ...
```

remains valid and shadows the standard binding.

The new name is **not reserved**: explicit shadowing remains ordinary behavior.

### Safe migration rule

A direct source expression may migrate:

```protos
Error().signal()
```

to:

```protos
fail()
```

only when:

- `Error` at the original expression means the canonical standard Error
  factory/prototype;
- the call intentionally creates a fresh generic Error;
- there is no additional state mutation/cleanup/bookkeeping tied to a local
  helper;
- no custom subtype/category is intended.

Never migrate:

```protos
error.signal()
Error.signal()
MyError().signal()
```

or stateful local `fail` helpers as if they were equivalent.

## Comparative scoring

Scores are 1–5. Every row includes evidence confidence.

### A — no change

| Dimension | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariants | 5 | HIGH | Existing D005 spelling is exact. |
| Protos alignment | 5 | HIGH | Pure object construction + ordinary signaling message. |
| Present-need proportionality | 2 | HIGH | Broad production repetition already pays boilerplate. |
| Incremental growth | 5 | HIGH | Convenience remains additive later. |
| Future-option resilience | 5 | HIGH | No names/protocols allocated. |
| Scalability | 3 | HIGH | Repeated helper definitions scale source/maintenance cost poorly. |
| Conceptual simplicity | 4 | HIGH | Semantics are simple, repetitive intent is less clear. |
| Portability / implementation freedom | 5 | HIGH | No host coupling. |
| Runtime/resource cost | 5 | HIGH | Existing minimal operation. |
| Failure/operability | 5 | HIGH | Exact current handler/unwind behavior. |
| Deferral/reversibility/migration | 4 | HIGH | Easy to change later, but repetition persists. |
| Evidence maturity/implementation risk | 5 | HIGH | Already implemented and heavily exercised. |

**Underengineering red flag:** MEDIUM/HIGH.

### B-prime — standard prelude `fail()`

| Dimension | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariants | 5 | HIGH | Exact fresh identity/category/non-resumable behavior is explicitly preserved. |
| Protos alignment | 5 | HIGH | Ordinary callable + ordinary lexical lookup/shadowing; no keyword. |
| Present-need proportionality | 5 | HIGH | Exact zero-arg `fail` shape already recurs across production modules. |
| Incremental growth | 5 | HIGH | Payloads/custom categories/rethrow remain independent future additions. |
| Future-option resilience | 5 | HIGH | Adds one narrow operation without changing Error.signal or handlers. |
| Scalability | 5 | HIGH | Replaces repeated definitions with one immutable shared standard facility. |
| Conceptual simplicity | 5 | HIGH | One name, one meaning: fresh generic failure. |
| Portability / implementation freedom | 5 | HIGH | Source/native/JIT realizations remain semantically equivalent. |
| Runtime/resource cost | 5 | HIGH | Same required fresh occurrence, no added payload/state. |
| Failure/operability | 5 | HIGH | Existing handler/unwind/future behavior remains authoritative. |
| Deferral/reversibility/migration | 4 | HIGH | Allocating bare `fail` is a lasting namespace commitment. |
| Evidence maturity/implementation risk | 5 | HIGH | Strong repository evidence plus message/callable precedents. |

**Overengineering red flag:** none.

**Underengineering red flag:** LOW.

### C — exact `Error.fail()`

| Dimension | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariants | 5 | HIGH | Can preserve D005 without altering Error.signal. |
| Protos alignment | 5 | HIGH | Owner-side ordinary behavior is natural. |
| Present-need proportionality | 3 | HIGH | Adds Core API but saves little versus Error().signal(). |
| Incremental growth | 5 | HIGH | Other factory behaviors remain separate. |
| Future-option resilience | 5 | HIGH | No universal bare name or syntax. |
| Scalability | 4 | HIGH | Can remove helpers, but weaker ergonomic incentive. |
| Conceptual simplicity | 4 | HIGH | Clear namespace; users learn factory fail versus instance signal. |
| Portability / implementation freedom | 5 | HIGH | Ordinary Core behavior. |
| Runtime/resource cost | 5 | HIGH | Same fresh Error requirement. |
| Failure/operability | 5 | HIGH | Existing transfer semantics. |
| Deferral/reversibility/migration | 5 | HIGH | Very low global compatibility cost. |
| Evidence maturity/implementation risk | 4 | HIGH | Pharo/Io support owner-side convenience, but repo converged on bare fail. |

**Underengineering red flag:** MEDIUM for the stated ergonomic problem.

### C2 — polymorphic Error-prototype `fail()`

| Dimension | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariants | 4 | MEDIUM | Freshness is preservable, but valid receiver/factory semantics must expand. |
| Protos alignment | 5 | MEDIUM | Prototype-polymorphic factory operation is message-oriented. |
| Present-need proportionality | 1 | HIGH | Current evidence is generic Error, not subtype shorthand. |
| Incremental growth | 5 | HIGH | Rich custom hierarchy convenience later. |
| Future-option resilience | 3 | MEDIUM | Commits custom Error construction to one protocol prematurely. |
| Scalability | 5 | HIGH | Broadly reusable once adopted. |
| Conceptual simplicity | 3 | MEDIUM | Receiver becomes category + factory + signal convenience. |
| Portability / implementation freedom | 5 | HIGH | Can remain semantic. |
| Runtime/resource cost | 5 | HIGH | Constant fresh occurrence. |
| Failure/operability | 4 | MEDIUM | Invalid receiver/inherited behavior needs extra rules. |
| Deferral/reversibility/migration | 3 | HIGH | Harder to retract after user hierarchies adopt it. |
| Evidence maturity/implementation risk | 4 | MEDIUM | Strong external precedent, weak current Protos need. |

**Overengineering red flag:** HIGH.

### D — imported helper

| Dimension | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariants | 5 | HIGH | Library can preserve D005. |
| Protos alignment | 5 | HIGH | Ordinary module/import mechanism. |
| Present-need proportionality | 2 | HIGH | Qualification/import is not shorter than current direct spelling. |
| Incremental growth | 5 | HIGH | Module can grow independently. |
| Future-option resilience | 5 | HIGH | Core remains unchanged. |
| Scalability | 3 | HIGH | Repeated imports/aliases replace repeated local helpers. |
| Conceptual simplicity | 3 | HIGH | Adds a namespace primarily to abbreviate a Core operation. |
| Portability / implementation freedom | 5 | HIGH | Pure library. |
| Runtime/resource cost | 5 | HIGH | Negligible extra cost. |
| Failure/operability | 5 | HIGH | Same signaling semantics. |
| Deferral/reversibility/migration | 5 | HIGH | Easy to add/remove as library surface. |
| Evidence maturity/implementation risk | 4 | HIGH | Feasible, but no evidence users want qualified failure syntax. |

**Underengineering red flag:** HIGH relative to the ergonomic requirement.

### E — dedicated syntax

| Dimension | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariants | 5 | MEDIUM | Syntax could lower to exact D005 behavior. |
| Protos alignment | 1 | HIGH | Privileged control syntax duplicates an ordinary-object mechanism. |
| Present-need proportionality | 1 | HIGH | Evidence only asks for concise zero-arg generic failure. |
| Incremental growth | 3 | MEDIUM | Syntax becomes pressure point for payload/category/rethrow extensions. |
| Future-option resilience | 2 | HIGH | Permanently reserves grammar/keyword space. |
| Scalability | 5 | HIGH | Concise at every call site. |
| Conceptual simplicity | 2 | HIGH | Short spelling but new semantic/syntactic category. |
| Portability / implementation freedom | 4 | HIGH | Runtime is portable; every frontend/tool must understand syntax. |
| Runtime/resource cost | 5 | HIGH | Can be direct. |
| Failure/operability | 5 | HIGH | Existing unwind model can remain. |
| Deferral/reversibility/migration | 1 | HIGH | Syntax is expensive to retract. |
| Evidence maturity/implementation risk | 5 | HIGH | Mature precedent exists, but necessity in Protos does not. |

**Overengineering red flag:** HIGH.

### F — generalized global raise facility

| Dimension | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariants | 4 | MEDIUM | Multiple modes can preserve D005 only with careful identity rules. |
| Protos alignment | 3 | MEDIUM | Ordinary calls fit; overloaded dynamic roles do not fit the narrow problem. |
| Present-need proportionality | 1 | HIGH | No evidence for message/cause/rethrow modes. |
| Incremental growth | 5 | HIGH | Broad facility can absorb later features. |
| Future-option resilience | 2 | HIGH | Precommits future Error enrichment to one global API. |
| Scalability | 5 | HIGH | One facility centralizes many cases. |
| Conceptual simplicity | 2 | HIGH | Arity/context-dependent meanings conflate separate operations. |
| Portability / implementation freedom | 4 | MEDIUM | Richer metadata/dynamic-context rules constrain implementations. |
| Runtime/resource cost | 4 | MEDIUM | Still small, but optional modes impose unused complexity. |
| Failure/operability | 3 | MEDIUM | Bare rethrow versus fresh failure needs new dynamic-context failure rules. |
| Deferral/reversibility/migration | 2 | HIGH | Hard to simplify after adoption. |
| Evidence maturity/implementation risk | 4 | MEDIUM | Ruby/Python validate the family, not current Protos demand. |

**Overengineering red flag:** HIGH.

## Per-candidate incremental-design gate

### A — no change

**Pay for what you need:** Core pays nothing new, but many source modules pay
repeated boilerplate.

**Grow as you need:** excellent.

**Cost of deferral/reversibility:** later addition is easy; ongoing repetition is
the concrete current cost.

**Smallest sufficient:** semantically yes, ergonomically no longer compelling
given repeated production convergence.

### B-prime — prelude `fail()`

**Pay for what you need:** yes. One name and one zero-argument callable correspond
exactly to demonstrated use.

**Grow as you need:** yes. Payload, custom-category and rethrow operations can be
added separately without changing zero-arg generic failure.

**Cost of deferral/reversibility:** deferral requires only continuing current
source patterns. Adoption allocates the bare name `fail`, which is the main
compatibility cost.

**Smallest sufficient:** yes. No syntax, new category, payload, custom subtype
protocol or rethrow state.

### C — `Error.fail()`

**Pay for what you need:** yes semantically; ergonomic payoff is smaller.

**Grow as you need:** yes.

**Cost of deferral/reversibility:** minimal.

**Smallest sufficient:** structurally small but does not fully address the
observed desire for bare `fail()`.

### C2 — polymorphic prototype `fail()`

**Pay for what you need:** no. User subtype factory semantics are speculative.

**Grow as you need:** strong but preimplemented.

**Cost of deferral/reversibility:** low; a subtype shorthand can be added if
custom Error hierarchies later repeat the same construction/signaling pattern.

**Smallest sufficient:** no.

### D — imported helper

**Pay for what you need:** low Core cost, higher per-caller ceremony.

**Grow as you need:** yes.

**Cost of deferral/reversibility:** trivial.

**Smallest sufficient:** not for the actual ergonomics goal.

### E — syntax

**Pay for what you need:** no.

**Grow as you need:** possible only by expanding a privileged language form.

**Cost of deferral/reversibility:** adding syntax later requires lexer/parser,
specification, tooling and migration work, but no foundational Error-model
rewrite.

**Smallest sufficient:** no.

### F — generalized global raise

**Pay for what you need:** no.

**Grow as you need:** broad, but capability is installed before evidence.

**Cost of deferral/reversibility:** message/cause/rethrow can be added later as
separate ordinary APIs if demonstrated.

**Smallest sufficient:** no.

## Mandatory adversarial incremental-design questions

### What is the smallest solution satisfying today's requirements?

Candidate B-prime if the project accepts allocating one universal name.

Every selected capability is justified by current evidence:

- one bare callable: repeated local bare `fail()` helpers;
- zero arguments: current generic helpers take none;
- fresh generic Error: exact current repeated operation;
- ordinary shadowing: required by existing prelude semantics and current local
  helpers;
- no payload/custom category/rethrow: no demonstrated need.

If allocating the name is considered too expensive, Candidate A is the coherent
fallback; C is not a materially better ergonomic compromise.

### If we omit B-prime today, can it be added later without breaking the model?

YES.

D005 already supplies all semantic machinery. A future prelude helper requires
only adding one ordinary standard callable/binding and migrating chosen source
sites.

No Error identity, handler, actor, Future, persistence, scheduling or transfer
model must be replaced.

### What exactly must be rewritten later if we defer it?

To add B-prime later:

- normative Error/prelude documentation;
- core/prelude bootstrap/source binding;
- conformance tests;
- guide/README references where applicable;
- selected source call sites;
- tooling highlighting/completion only insofar as it already exposes standard
  prelude bindings.

No parser/lexer change is needed.

### What complexity would make us regret preimplementing richer future failure features now?

Payload/message/cause/rethrow/subtype convenience would force decisions about:

- Error visible structure;
- diagnostics versus portable semantics;
- custom category construction;
- active-handler dynamic context;
- re-signaling identity;
- stack/source ownership;
- cross-Actor/P transfer of payload;
- API overload/arity behavior.

None is required to spell today's generic failure more concisely.

## D146 issue-specific anti-overengineering gates

### 1. Is shortening `Error().signal()` materially valuable enough to consume Core/prelude surface?

**YES, narrowly.**

The evidence is not a hypothetical one-line saving: many independent production
modules have already created the same bare zero-argument helper, often multiple
times.

The cost is one universal but shadowable prelude name.

### 2. Does a standard prelude callable solve the repetition with no syntax?

**YES.**

It matches the existing source idiom exactly and uses ordinary lookup/call
semantics.

### 3. Would a factory-side message be clearer and less globally invasive?

**Less globally invasive: YES.**

**Clearer for the demonstrated ergonomic problem: NO.**

`Error.fail()` is semantically clear but only modestly shorter than
`Error().signal()`, while production code independently converged on bare
`fail()`.

### 4. Is the local helper already good enough?

**Semantically yes; project-wide ergonomically no longer ideal.**

The repetition is broad enough that the helper itself has effectively become an
unstandardized convention.

### 5. Is any message/data argument independently required now?

**NO.**

### 6. Can richer Error constructors remain explicit?

**YES.**

Custom errors remain ordinary object construction + exact signaling.

### 7. Would syntax/keyword add machinery beyond the need?

**YES.**

It adds grammar, parser, tooling and a privileged control form without adding a
needed semantic capability.

### 8. What is the compatibility cost of deferring entirely?

Low foundational cost, but continued duplicated helper/source ceremony.

The decision is therefore ergonomic, not architectural necessity.

## Failure modes, counterexamples and disqualifying conditions

### B-prime: namespace collision

Failure mode:

A program intentionally relies on bare `fail` being absent.

After B-prime it resolves to the standard callable.

Mitigation:

- `fail` remains ordinary and shadowable;
- no reserved word is created;
- the compatibility delta is explicit.

Disqualifying condition:

If repository or ecosystem evidence showed that bare `fail` absence is itself
an important portable contract or that common code intentionally uses failed
lookup of `fail` as control flow, B-prime should be rejected.

No such evidence was found.

### B-prime: generic Error overuse

Failure mode:

A convenient global `fail()` may encourage generic Error where a meaningful
domain-specific standard/user Error category should be signaled.

Mitigation:

The helper is documented as exactly generic Error. Domain contracts that need a
specific category remain explicit.

This is a style/design pressure, not semantic unsoundness.

### B-prime: shadowed `Error`

Counterexample:

```protos
Error: CustomThing
fail()
```

If `fail()` used caller lexical lookup, the same standard callable would no
longer have standard meaning.

B-prime therefore explicitly uses canonical Core Error and does not perform
caller lexical lookup.

### B-prime: extracted callable

Counterexample:

```protos
f: fail
someObject.run(f)
```

The operation must retain standard generic Error behavior independent of the
receiver/caller where it is later invoked.

This is why the standard facility owns/captures its Core environment semantics.

### B-prime: Future identity

Two fresh fail occurrences captured as separate Future failures must remain
separate Error identities. Re-observation of one recorded Future failure must
preserve the exact recorded Error under D005.

### C2 disqualifier

No current production evidence demonstrates repeated generic shorthand for
arbitrary custom Error prototype instances.

The candidate therefore fails present-need proportionality.

### E disqualifier

No current semantic need requires grammar. Ordinary call semantics solve the
problem completely.

## Future-scenario and scalability stress

### Larger Standard Library / application ecosystem

A standard `fail()` removes duplicated generic-failure helpers while preserving
local shadowing for richer module-specific behavior.

Specific error categories remain ordinary explicit APIs.

### Actors

The prelude facility itself may be physically shared only under the existing
semantic-immutability rules.

Each reached `fail()` occurrence belongs to the invoking execution/value
domain exactly as an equivalent standard fresh Error occurrence does today.

No shared mutable Error singleton is introduced.

### Futures / Tasks

A `fail()` inside asynchronous work signals one fresh Error exactly as current
generic signaling does.

If that exact Error becomes a recorded Future failure, subsequent same-domain
observation preserves that exact identity under existing rules.

D146 adds no Future-specific behavior.

### P / transfer / distributed boundaries

Freshness and reconstruction across boundaries remain governed by D005 and the
applicable boundary contract.

`fail()` creates no new cross-domain identity channel and carries no mandatory
payload.

### Alternative runtime / migration away from Truffle

The public contract is independent of Java classes, Truffle exceptions, bytecode
implementation and host stack representation.

A future interpreter, AOT compiler or other VM may implement the standard
callable differently while preserving Error semantics.

### Debugger evolution

The semantic user request site is the `fail()` call.

No guest-visible stack structure is introduced.

Debugger frame presentation may evolve independently under debugger contracts.

### Future structured Error payloads

A future project decision may add explicit Error construction/prototype APIs for
message/cause/data without changing zero-argument generic `fail()`.

The shorthand should not be overloaded automatically merely because richer Error
objects later exist.

### Future resumable conditions/restarts

D005 explicitly reserves resumable recovery for a separate future control
contract.

B-prime does not capture or expose a continuation and must not be reinterpreted
as resumable.

### Mandatory future-regret question

**What plausible future requirement would make us regret B-prime?**

A future Protos style may strongly require every intentional application-level
failure to carry structured domain/category/diagnostic information, making a
bare generic Error operation undesirable for most new code.

A second possibility is that future top-level naming pressure makes the generic
bare name `fail` unusually valuable for another standard concept.

**If that happens, what escape path remains?**

- keep `fail()` as the narrow compatibility-level generic Error primitive;
- introduce explicit richer domain Error constructors/protocols separately;
- style/deprecation guidance can discourage new generic use without changing
  existing semantics;
- because `fail` is ordinary and shadowable, modules may locally replace its
  meaning where appropriate;
- no syntax or Error object representation has been frozen by B-prime.

The one cost that cannot be erased cheaply is the allocated standard prelude
name. That is the principal irreversible part of the decision.

## Implementation/runtime/resource consequences

### B-prime present cost

- one standard prelude binding;
- one immutable callable/facility;
- normative specification and conformance coverage;
- completion/highlighting/doc surface for one standard name;
- compatibility commitment for bare `fail`.

### B-prime execution cost

Semantically one fresh Error occurrence and one non-local signal, exactly as the
current operation already requires.

The extra source-level convenience must not require a second Error object,
payload allocation, registry lookup, synchronization or global mutable state.

An implementation may inline/specialize the facility when semantically
unobservable.

### Unused-capability cost

Because B-prime has only one zero-argument behavior, unused cost is essentially
one prelude name/facility rather than a framework.

C2/E/F impose larger unused conceptual/frontend commitments.

## Portability, migration, compatibility and reversibility

### Source compatibility

B-prime changes bare-name resolution for `fail`.

Existing explicit local `fail:` declarations retain precedence.

No syntax is reserved.

### Binary/runtime compatibility

No serialized data, persisted Error representation, wire format, module identity
or ABI is introduced.

### Migration

Selected exact generic call sites can migrate incrementally.

No source must migrate merely because the standard binding exists.

### Reversibility

Removing B-prime later would be source-breaking for code that adopts bare
`fail()`.

That is the main reversibility cost.

By contrast, richer payload/subtype/rethrow capabilities remain fully deferred.

## Intentionally deferred questions

D146 B-prime deliberately does not decide:

- message-bearing Error constructors;
- cause chains;
- structured payloads;
- source/stack guest reflection;
- user-defined custom Error factory shorthand;
- prototype-polymorphic `fail`;
- `raise` spelling;
- rethrow-current-error convenience;
- checked/unchecked categories;
- Result/Option redesign;
- assertion-specific `Assertions.fail`;
- error formatting;
- resumable conditions/restarts;
- throw/raise syntax;
- standard Error taxonomy expansion.

Deferral is safe because none is foundational to fresh generic signaling, and all
can be designed later without changing D005 identity or handler semantics.

## Strongest argument against B-prime

The strongest argument is **not** implementation complexity. It is namespace and
design culture.

`fail` is a very generic, attractive word. Once placed in the universal
prelude, it becomes permanent portable vocabulary and may encourage code to use a
generic `Error` where a precise domain failure or ordinary value would provide a
better contract.

Candidate C avoids that universal-name cost while remaining fully
message-oriented:

```protos
Error.fail()
```

It is therefore a credible conservative alternative.

The reason B-prime still wins is direct repository evidence: production code has
already repeatedly chosen the bare `fail()` abstraction itself. The project is
currently paying the same name/concept locally over and over, and the standard
prelude's explicit shadowing model prevents the new binding from blocking richer
local meanings.

If the owner values preservation of the bare global/prelude namespace more
highly than removing this repeated helper, Candidate A — not C — is the cleanest
alternative.

## GITHUB021 invariant / delta consistency check

Applicable ratified D005/current invariants:

```text
FRESH_STANDARD_FAILURE_OCCURRENCE=PRESERVE
STANDARD_FAILURE_IDENTITY_IS_OBSERVABLE=PRESERVE
EXISTING_ERROR_RESIGNAL_USES_EXACT_OBJECT=PRESERVE
ERROR_PROTOTYPES_ARE_NOT_FAILURE_SINGLETONS=PRESERVE
Error.signal()_SIGNALS_Error_ITSELF=PRESERVE
SIGNALING_IS_NON_RESUMABLE=PRESERVE
PORTABLE_ERROR_TAXONOMY_REMAINS_SHALLOW=PRESERVE
NO_IMPLICIT_ERROR_PAYLOAD_OR_CONVERSION=PRESERVE
NO_GUEST_VISIBLE_STACK_STRUCTURE_FROM_D146=PRESERVE
```

Candidate B-prime preserves every applicable invariant.

Material new observable consequences introduced by B-prime:

```text
NEW_STANDARD_PRELUDE_BINDING=fail
FAIL_BINDING_IS_ORDINARY_AND_SHADOWABLE=YES
FAIL_IS_EXTRACTABLE_CALLABLE=YES
FAIL_ARITY=ZERO
FAIL_VALID_INVOCATION_NEVER_RETURNS_NORMALLY=YES

FAIL_USES_CANONICAL_STANDARD_ERROR=YES
FAIL_USES_CALLER_LEXICAL_Error_BINDING=NO

PREVIOUSLY_ABSENT_BARE_fail_LOOKUP_BECOMES_BOUND=YES
LOCAL_fail_CREATION_STILL_SHADOWS_STANDARD=YES

CUSTOM_ERROR_SUBTYPE_SHORTHAND=NO
PAYLOAD_ARGUMENTS=NO
RETHROW_MODE=NO
ASSERTION_FAILURE_MEANING=NO
SYNTAX_CHANGE=NO
```

These consequences are surfaced before owner approval.

No owner-approved invariant is reopened or contradicted.

```text
DECISION_INVARIANT_CONSISTENCY=PASS
REOPENED_INVARIANT=NONE
APPROVAL_PROVENANCE_FOR_REOPENING=NOT_APPLICABLE
```

## Required Dxxx packet checklist

```text
1_EXACT_DECISION_AND_NEED=PASS
2_CURRENT_CONSTRAINTS_AND_RATIFIED_DECISIONS=PASS
3_PRIOR_ART_SURVEY_AND_CONTRIBUTION=PASS
4_COMPLETE_MEANINGFUL_CANDIDATE_SET=PASS
5_COMPARATIVE_1_TO_5_SCORING_WITH_CONFIDENCE=PASS
6_FAILURE_MODES_COUNTEREXAMPLES_DISQUALIFIERS=PASS
7_FUTURE_SCENARIO_AND_SCALABILITY_STRESS=PASS
8_INCREMENTAL_DESIGN_ANALYSIS=PASS
9_IMPLEMENTATION_RUNTIME_RESOURCE_CONSEQUENCES=PASS
10_PORTABILITY_MIGRATION_COMPATIBILITY_REVERSIBILITY=PASS
11_INTENTIONALLY_DEFERRED_QUESTIONS=PASS
12_RECOMMENDED_OPTION_AND_PROTOS_ALIGNMENT=PASS
13_STRONGEST_ARGUMENT_AGAINST_RECOMMENDATION=PASS

RESEARCH_SYSTEMS=
    Pharo/Smalltalk
    Io
    Self
    Ruby
    Python
    JavaScript
    Java
    C#
    Rust
    Go

RESEARCH_SYSTEM_COUNT=10
RESEARCH_APPROACH_COUNT>=5

D005_INVARIANT_DELTA_CHECK=PASS
READY_FOR_EXACT_OWNER_DECISION=YES
```

## Proposal pending exact project-owner approval

**Proposed Candidate B-prime — one ordinary zero-argument standard prelude
`fail` callable.**

Exact proposed decision:

```text
D146_CANDIDATE=B_PRIME

STANDARD_PRELUDE_BINDING=fail
FAIL_SURFACE=ORDINARY_CALLABLE
FAIL_IS_KEYWORD=NO
FAIL_IS_INTRINSIC_SYNTAX=NO
FAIL_IS_RESERVED_IDENTIFIER=NO
LOCAL_SHADOWING=ALLOWED

FAIL_ARITY=ZERO
FAIL_ARGUMENT_PAYLOAD=NONE
FAIL_EXISTING_ERROR_ARGUMENT=NOT_SUPPORTED
FAIL_ERROR_PROTOTYPE_ARGUMENT=NOT_SUPPORTED
FAIL_RETHROW_MODE=NOT_SUPPORTED

EACH_REACHED_VALID_INVOCATION=ONE_FRESH_GENERIC_ERROR_OCCURRENCE
FRESH_ERROR_IMMEDIATE_PARENT=CANONICAL_STANDARD_Error
FRESH_IDENTITY_PER_OCCURRENCE=YES
ERROR_POOLING_OR_SINGLETON=NO

CALLER_LEXICAL_Error_LOOKUP=NO
CALLER_SHADOWED_Error_CHANGES_FAIL_CATEGORY=NO

SIGNAL_EXACT_FRESH_OBJECT=YES
HANDLER_SELECTION=EXISTING_D005_RULES
NON_RESUMABLE=YES
ENSURE_UNWIND=EXISTING_RULES
FUTURE_ERROR_IDENTITY=EXISTING_D005_RULES
ACTOR_P_BOUNDARY_SEMANTICS=EXISTING_D005_RULES

VALID_FAIL_RETURNS_NORMALLY=NO
USER_VISIBLE_ERROR_PAYLOAD_ADDED=NO
ERROR_TAXONOMY_CHANGE=NO
GUEST_STACK_REFLECTION_ADDED=NO

EXTRACTED_fail_RETAINS_STANDARD_BEHAVIOR=YES
ASSERTION_FAILURE_MEANING=NO

PREVIOUSLY_ABSENT_BARE_fail_LOOKUP_BECOMES_BOUND=YES
EXISTING_LOCAL_fail_BINDINGS_REMAIN_VALID=YES

SAFE_MIGRATION=
    ONLY_CANONICAL_Error().signal()_FRESH_GENERIC_OCCURRENCES

CUSTOM_ERROR_FACTORY_SHORTHAND=DEFERRED
ERROR_MESSAGE_CAUSE_PAYLOAD=DEFERRED
RETHROW_CONVENIENCE=DEFERRED
THROW_RAISE_SYNTAX=NOT_ADDED
RESULT_OPTION_MODEL=UNCHANGED
```

No normative publication, implementation work or issue closure is authorized
until the project owner explicitly approves this exact candidate.
