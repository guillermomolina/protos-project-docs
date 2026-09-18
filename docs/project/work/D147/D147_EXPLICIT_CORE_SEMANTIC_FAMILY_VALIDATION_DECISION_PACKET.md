# D147 — Explicit Core semantic-family and receiver-domain recognition

## Decision state and authority

This is the non-normative decision packet for `guillermomolina/protos#577`.

```text
D147_STATUS=NEEDS_USER_DECISION
TRIGGER=AUD011/#540
PROTOS_REVISION=b1b5c91b365a57ed65797b78ab9a6466e7df16f5
PROJECT_DOCS_BASE=daca5f6589e3bf6b5e7258ed98a657d02e9060df
DECISION_SELECTED=NO
```

No normative or implementation change is authorized by this packet.

## Exact problem

Core already distinguishes:

1. ordinary delegation / message availability; and
2. exact membership in a built-in semantic family or ownership of standard
   receiver-specific state.

Delegation does not confer the second property.

Real Standard Library and Tool code nevertheless has no direct guest-language
way to ask several exact receiver-domain questions. It therefore validates by
provoking unrelated strict standard operations and discarding their results.

Representative examples are:

```protos
// semantic String gate
ignoredValidation: "" + value

// ordinary unbounded Integer gate
value.div(1)

// exact Float gate
ignoredValidation: 0.0 + value
```

Other production code attempts to recognize a standard Array with:

```protos
(value.parent() === Array)
```

but that is not exact under the current object model.

The decision question is:

> Should Core expose a small explicit recognition operation for the exact
> built-in receiver-domain distinctions that production code already needs, and
> if so, what is the smallest model that does so without turning ordinary Protos
> programming into nominal type checking?

## Current Protos invariants

### Delegation is not semantic membership

`spec/semantics/OBJECT_MODEL.md` already states that semantic-family
membership, ownership of family-specific state, and delegation are distinct.

An ordinary object may delegate to:

- a Number value;
- `Integer`, `Float`, or another numeric prototype;
- `String`;
- `Array`;
- another standard family/state-bearing object;

without becoming a member of that semantic family or acquiring the corresponding
standard receiver-owned state.

Standard family-specific behavior validates the **original receiver** after
ordinary lookup has selected the behavior.

### Internal semantic classifiers already exist abstractly

`spec/runtime/ABSTRACT_RUNTIME.md` already models standard receiver validation
through implementation-independent classifiers such as:

```text
isSemanticNumberValue
isSemanticStringValue
```

Those abstract predicates are not message lookup and do not test delegation.

D147 therefore does not invent semantic classification. It decides whether a
bounded part of already-existing Core truth should become directly observable to
guest code.

### Numeric families are distinct

The current numeric model has:

```text
Number
├── Integer
│   ├── UInt8
│   ├── Int8
│   ├── UInt16
│   ├── Int16
│   ├── UInt32
│   ├── Int32
│   ├── UInt64
│   └── Int64
└── Float
```

The hierarchy above is a prototype/delegation hierarchy for standard behavior.
It does not collapse semantic families.

In particular:

- an **ordinary Integer** is the unbounded `Integer` semantic family itself;
- each fixed-width integer family is distinct;
- `Float` is distinct;
- internal SmallInteger/BigInteger representation is not a semantic family;
- standard arithmetic does no implicit cross-family promotion.

Therefore a recognition operation for `Integer` must not silently mean
"Integer or any fixed-width descendant" when the contract being validated is
ordinary unbounded Integer.

### Boolean and null already have exact simple tests

Core has exactly canonical `true` and `false` Boolean values and no standard
`Boolean` prototype/binding.

`null` is one canonical singleton.

Their exact recognition can already be expressed without probing an unrelated
family operation:

```protos
(value === true) || (value === false)
value === null
```

D147 has no current evidence requiring a new Boolean/null descriptor or owner.

## Production evidence

### Ordinary unbounded Integer

GitHub source inventory at the audited revision finds `div(1)` family-gate
usage in ten production `.protos` files, including:

- `protos/lib/math/Integer.protos`;
- `protos/lib/json/JSON.protos`;
- `protos/lib/toml/TOML.protos`;
- `protos/lib/network/IpAddresses.protos`;
- `protos/lib/cli/CommandLine.protos`;
- `protos/tools/test/Progress.protos`;
- `protos/tools/test/ResourceCatalog.protos`;
- `protos/tools/test/Runner.protos`;
- `protos/tools/test/Manifest.protos`;
- `protos/tools/test/ResourceReservation.protos`.

Several comments explicitly describe this as a family or receiver-domain gate and
state that fixed-width and delegated lookalikes must be rejected.

### String

String validation appears independently in JSON, TOML, CLI, networking and Test
Tool source through forms such as:

```protos
"" + value
text + ""
validated: "" + value
(validated === value).ifFalse(...)
```

The intent is exact semantic String recognition without conversion.

### Float

`protos/lib/toml/TOML.protos` validates Float values through:

```protos
0.0 + value
```

This is an exact Float gate under the ratified arithmetic matrix: standard
`Float +` accepts `Float` only and cross-family numeric arithmetic signals
rather than promoting.

### Array

Production validation in:

- `protos/lib/cli/CommandLine.protos`;
- `protos/tools/test/SuiteGraph.protos`;

uses:

```protos
value.parent() === Array
```

as an Array check.

That relation is not equivalent to standard Array-state ownership.

It can admit an ordinary object that merely delegates directly to `Array`, and
it can reject a genuine standard Array produced by an inherited/custom Array
factory whose immediate parent is the actual invocation receiver rather than the
canonical `Array` prototype.

The current Array specification explicitly permits such standard-state Arrays:

```protos
MyArray: Array { ... }
values: MyArray(10, 20)
```

where `values` owns standard Array indexed state but delegates to `MyArray`.

D147 therefore has evidence not only for cleaner syntax but for a currently
unavailable exact guest-level observation.

## Existing Protos precedent: owner-side recognition

Core already has a closely related public model:

```text
IpAddress.recognizes(value)  -> true | false
IpEndpoint.recognizes(value) -> true | false
```

Their standard `recognizes` behavior:

- requires the exact canonical standard owner as receiver;
- accepts one arbitrary candidate;
- returns canonical Boolean;
- observes canonical standard state directly;
- invokes no candidate getter, callback, equality, or hash behavior;
- does not confer recognition through delegation;
- treats candidate mismatch as `false`, not as an invalid-candidate Error.

This is strong internal precedent for a family-owner query without a universal
type object, syntax, registry, or user-extensible nominal relation.

## Scope delta discovered during D147

The original D147 issue is framed around semantic value families.

The Array evidence exposes the same underlying problem for a state-bearing
standard receiver domain rather than an immutable semantic value family.

The proposed Candidate B-prime therefore includes one explicit scope delta:

```text
D147_SCOPE_DELTA=
    include evidence-backed standard Array receiver-state recognition
```

This is not implementation detail and requires explicit owner approval with the
candidate.

It does **not** generalize D147 to every standard object or protocol.

## Comparative research

The survey covers prototype/message-oriented, class/nominal, branded built-in,
and static runtime-type approaches.

### Self — behaviorism first, reflection explicit when necessary

Self's programming guide recommends behaviorism for normal programming and
making reflection explicit through mirrors when representation/category
inspection is genuinely necessary.

Self mirrors also distinguish VM-known object kinds such as small integers,
floats, canonical strings, vectors, blocks, methods and ordinary objects.

Sources:

- https://handbook.selflanguage.org/2024.1/progguid.html
- https://handbook.selflanguage.org/2024.1/mirrors.html

**Contribution:** exact runtime category distinctions can coexist with a
prototype/message language, but they should not replace protocol-oriented
programming by default.

### Pharo / Smalltalk — exact membership, inherited membership and capability are separate

Pharo exposes distinct questions:

```text
isMemberOf:   exact class
isKindOf:     class or subclass
respondsTo:   message capability
```

Source:

- https://books.pharo.org/updated-pharo-by-example/

**Contribution:** exact membership and protocol capability are legitimately
different queries. Protos does not, however, have a class/metaclass institution
that should be imported merely to obtain this distinction.

### JavaScript — branded built-in recognition differs from prototype ancestry

JavaScript `instanceof` normally tests prototype-chain membership and may be
customized through `Symbol.hasInstance`.

By contrast, `Array.isArray(value)` performs Array recognition that is not
equivalent to prototype-chain ancestry; an object merely using
`Array.prototype` as its prototype is not thereby an Array.

Sources:

- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/isArray
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof

**Contribution:** this is a close analogue to Protos Array-state ownership versus
delegation and strongly supports an owner-side exact recognizer instead of
prototype-chain testing.

### Python — first-class type objects and subtype-aware isinstance

Python exposes `type(value)` and `isinstance(value, classinfo)`.
`isinstance` includes direct, indirect and virtual subclasses and is normally
preferred when subtype participation is desired.

Source:

- https://docs.python.org/3/library/functions.html

**Contribution:** a generic type-object relation is powerful, but requires a
first-class nominal type institution much broader than D147's demonstrated need.

### Ruby — exact class and kind-of relations coexist

Ruby distinguishes:

```text
instance_of?  exact class
is_a?/kind_of? class/superclass/module relation
```

Source:

- https://docs.ruby-lang.org/en/master/Object.html

**Contribution:** again demonstrates that exact category tests are useful, but
through a class/module model Protos deliberately does not have.

### Kotlin — type tests are tied to the static/runtime type system

Kotlin `is` / `!is` test runtime type compatibility and participate in smart
casts and compile-time type reasoning.

Sources:

- https://kotlinlang.org/docs/typecasts.html
- https://kotlinlang.org/spec/expressions.html

**Contribution:** dedicated `is` syntax pulls in assumptions about runtime
types, subtype relations and compiler narrowing that are unnecessary for the
current Protos problem.

### C# and Swift — runtime type tests sit on nominal type/protocol systems

C# `is` checks runtime compatibility with a type and `typeof` produces a
`System.Type` object. Swift `is` / `as` operate over class/protocol type
relationships.

Sources:

- https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/type-testing-and-cast
- https://docs.swift.org/swift-book/documentation/the-swift-programming-language/typecasting/

**Contribution:** these approaches are credible but structurally much larger:
they expose a general type universe, not only pre-existing built-in semantic
receiver domains.

### Io — prototype introspection plus hidden primitive categories

Io is prototype-based and exposes ordinary slot/prototype introspection. Its VM /
embedding layer nevertheless has direct primitive category tests such as number,
sequence and list checks.

Sources:

- https://iolanguage.org/docs/Book/Objects/
- https://iolanguage.org/docs/Guide/

**Contribution:** a prototype model does not eliminate hidden primitive-category
truth, but Io provides no reason to turn that truth into a universal guest type
system.

## Candidate set

### Candidate A — status quo indirect validation

Add no public Core surface.

Continue using:

- strict family operations such as `div(1)`, String concatenation and Float
  arithmetic as validation probes;
- ordinary reflection where currently used.

For Array, code must either retain the approximate parent check or find some
other indirect operation and accept the ordinary-dispatch interference that
comes with it.

### Candidate B-prime — evidence-scoped owner-side `recognizes(value)`

Add exactly four standard owner-side recognizers:

```text
String.recognizes(value)
Integer.recognizes(value)
Float.recognizes(value)
Array.recognizes(value)
```

No other Core family/state owner gains `recognizes` merely for symmetry.

Exact recognition contracts:

```text
String.recognizes(value)
    true iff value is a semantic String value

Integer.recognizes(value)
    true iff value is an ordinary unbounded Integer value
    fixed-width Integer-family values are false

Float.recognizes(value)
    true iff value is a semantic Float value

Array.recognizes(value)
    true iff value owns standard Array indexed state
    immediate delegation parent is irrelevant
```

Common rules:

- the receiver must be the exact canonical standard owner for the current
  execution/prelude;
- exactly one argument is required;
- an arbitrary candidate is permitted;
- candidate mismatch returns canonical `false`;
- success returns canonical `true`;
- recognition performs no candidate message lookup or dispatch;
- it invokes no candidate getter, `parent`, equality, hash, callback,
  conversion or coercion;
- it does not allocate a converted/replacement value;
- ordinary objects cannot claim Core recognition merely by delegation,
  overriding their own `recognizes`, or matching the owner's visible shape;
- standard owner objects themselves are not automatically family members:
  `String.recognizes(String)`, `Integer.recognizes(Integer)`,
  `Float.recognizes(Float)`, and `Array.recognizes(Array)` are false under
  current semantics;
- open/closed/frozen standard Arrays all remain recognized Arrays;
- a standard Array created by a descendant factory is recognized even when its
  immediate parent is that descendant factory.

The selector is intentionally the already-established Protos recognition
spelling rather than a new `is`, `typeof`, `instanceOf`, descriptor object,
or type operator.

The standard owner's frozen canonical behavior defines the Core result.
Programs remain free to define ordinary user `recognizes` messages on their own
objects, but such messages create no Core membership.

### Candidate C — uniform owner-side recognizers for all standard receiver domains

Adopt the same owner-side pattern broadly across all existing Core family/state
owners, including the numeric hierarchy, String, Array and other standard
state-bearing families such as Map/IdentityMap/Bytes where meaningful.

This maximizes regularity and avoids future piecemeal additions.

It also exposes many exact category questions for which no present production
need exists.

### Candidate D — evidence-scoped owner-side validators

Expose standard operations conceptually like:

```text
String.require(value)
Integer.require(value)
Float.require(value)
Array.require(value)
```

Each returns the exact supplied value on success and signals `Error` on
mismatch.

This directly replaces current validation gates but chooses failure semantics
rather than providing a reusable observation.

No exact selector spelling is assumed by the candidate name; `require` is
conceptual.

### Candidate E — generic value-side semantic relation

Expose a universal operation conceptually like:

```text
value.isSemanticMemberOf(String)
```

The second operand acts as a semantic-family descriptor.

This is compact and extensible but must define which objects are valid
descriptors, what hierarchy relation applies, how Array-state ownership fits,
whether users may define descriptors, and how invalid descriptors fail.

### Candidate F — explicit Core reflection descriptor

Expose a reflective operation such as conceptual:

```text
semanticFamilyOf(value)
```

returning a Core descriptor, followed by descriptor queries.

This makes classification first-class and could support tooling/introspection
beyond validation.

It creates the largest new category and risks exposing implementation
organization that is currently intentionally hidden.

## Explicitly rejected candidate families

### Standard Library `Types.requireX` helpers

A pure library helper can relocate String/Integer/Float probe idioms but cannot
derive exact standard Array-state ownership from current ordinary guest
reflection without either:

- retaining an inexact parent/shape approximation; or
- invoking behavior whose ordinary lookup can be replaced by user behavior.

If backed by a new Core primitive, the helper becomes an extra namespace wrapper
around one of the Core candidates rather than an independent solution.

It is therefore not retained as a final candidate.

### Reuse D131 matching

Under D131, inherited `Object.match(subject)` is ordinary equality behavior.
Changing standard family prototypes so that `String.match` or
`Integer.match` means semantic-family recognition would alter matching
semantics and conflate value matching with receiver-domain introspection.

No present need justifies reopening D131.

### Dedicated `typeof` / `is` syntax

Syntax adds no semantic capability beyond an ordinary operation and would cross
a 0-to-1 syntax/keyword/operator boundary while also encouraging a general type
interpretation.

The Kotlin/C#/Swift comparison shows that such syntax normally belongs to a much
larger type system. It is rejected for current D147.

### User-extensible family registration

Allowing user code to register or claim Core semantic-family membership would
contradict the current invariant that delegation and user behavior do not confer
built-in family membership.

No candidate retains this direction.

## Comparative scoring

Scores are 1–5. Confidence is HIGH/MEDIUM/LOW. Scores are comparison aids, not
decision authority.

### Candidate A — status quo

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 2 | HIGH | String/Integer/Float probes happen to reach strict domains, but current Array parent testing is not equivalent to state ownership. |
| Protos alignment | 3 | HIGH | No new institution, but unrelated computations are used as hidden introspection. |
| Present-need proportionality | 4 | HIGH | Zero new surface, but repeated production ceremony is already being paid. |
| Incremental growth | 3 | MEDIUM | More probes can be invented, but each depends on incidental strict operations. |
| Future-option resilience | 4 | MEDIUM | Leaves all future API choices open, at the cost of continued indirectness. |
| Scalability | 3 | HIGH | Repetition and inconsistent probes grow with modules and family contracts. |
| Conceptual simplicity | 2 | HIGH | `div(1)` meaning "require exact Integer" hides intent and Array checks are misleading. |
| Portability / implementation freedom | 4 | MEDIUM | Uses normative behavior rather than host types, but depends on unrelated operations remaining suitable gates. |
| Runtime / resource cost | 2 | HIGH | Gates perform arithmetic/concatenation or extra reflection rather than one classifier check. |
| Failure / operability | 3 | HIGH | Strict probes signal appropriately, but failures mention unrelated operations and Array can misclassify. |
| Deferral / reversibility / migration | 3 | HIGH | Public API can be added later, but the current repeated source debt and Array bug risk continue. |
| Evidence maturity / implementation risk | 5 | HIGH | Current behavior is implemented and widely exercised. |

**Underengineering red flag:** HIGH because exact Array recognition cannot be
expressed reliably through the current parent heuristic and the indirect family
gates are already repeated across production code.

### Candidate B-prime — four evidence-scoped owner recognizers

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 5 | HIGH | Directly exposes existing semantic classifiers/state ownership without delegation-as-membership. |
| Protos alignment | 5 | HIGH | Reuses ordinary owner-side messages and the existing `IpAddress.recognizes` pattern; no universal type object. |
| Present-need proportionality | 5 | HIGH | Adds exactly the four recognizers demonstrated by current production code. |
| Incremental growth | 5 | HIGH | Another owner can gain a local recognizer later without changing this model. |
| Future-option resilience | 5 | HIGH | Does not reserve a generic family hierarchy, descriptor system, syntax or registry. |
| Scalability | 5 | HIGH | Constant-time semantic/state classification; no registry, imports, global coordination or concurrency state. |
| Conceptual simplicity | 5 | HIGH | One existing selector spelling, local to the authority that knows the exact receiver domain. |
| Portability / implementation freedom | 5 | HIGH | Semantics are family/state truth, not Java classes, Truffle interop tags or storage representation. |
| Runtime / resource cost | 5 | HIGH | Direct Boolean recognition can avoid the discarded computations/allocation of current probes. |
| Failure / operability | 5 | HIGH | Candidate mismatch is false; invalid receiver/arity Error matches existing owner recognizer precedent. |
| Deferral / reversibility / migration | 5 | HIGH | Existing code can migrate incrementally; future owners are additive and no generic institution is frozen. |
| Evidence maturity / implementation risk | 5 | HIGH | Strong repository evidence plus direct existing Protos and JavaScript branded-recognition precedent. |

**Overengineering red flag:** none.

**Underengineering red flag:** LOW. The only deliberate incompleteness is that
unneeded family/state owners do not get the selector yet; adding them later is a
local additive change.

### Candidate C — uniform recognizers for all standard family/state owners

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 5 | HIGH | Can expose exact existing receiver-domain truth consistently. |
| Protos alignment | 4 | MEDIUM | Owner-local messages fit Protos, but systematic category introspection starts resembling a nominal taxonomy. |
| Present-need proportionality | 2 | HIGH | Most recognizers would have no demonstrated current consumer. |
| Incremental growth | 5 | HIGH | Uniformity is complete from the start. |
| Future-option resilience | 3 | MEDIUM | Broad surface commits future families to participate in one recognition convention. |
| Scalability | 5 | HIGH | Direct classifiers remain cheap and coordination-free. |
| Conceptual simplicity | 4 | MEDIUM | One uniform selector is simple, but the catalog of recognized families becomes larger public knowledge. |
| Portability / implementation freedom | 5 | HIGH | Can remain semantic rather than representation-based. |
| Runtime / resource cost | 5 | HIGH | Unused messages have negligible runtime cost after bootstrap. |
| Failure / operability | 5 | HIGH | Same predictable Boolean contract. |
| Deferral / reversibility / migration | 3 | HIGH | Adding later is cheap, so pre-installing unused owners has little deferral justification. |
| Evidence maturity / implementation risk | 4 | MEDIUM | Mechanism is proven; the broad scope is speculative. |

**Overengineering red flag:** HIGH. It preimplements exact-family queries for
contracts no current code needs.

### Candidate D — owner-side `require` validators

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 5 | HIGH | Core can validate the same exact semantic/state truth as B-prime. |
| Protos alignment | 4 | HIGH | Owner-local ordinary messages fit Protos, but hard-wire validation/failure rather than pure observation. |
| Present-need proportionality | 5 | HIGH | Current call sites are mostly validation gates. |
| Incremental growth | 4 | MEDIUM | Predicates would still be needed later for branching/inspection use cases. |
| Future-option resilience | 4 | MEDIUM | Does not create descriptors, but chooses one failure-oriented API shape. |
| Scalability | 5 | HIGH | Constant-time checks, no coordination. |
| Conceptual simplicity | 4 | HIGH | Direct at validation sites, less general than a Boolean recognizer. |
| Portability / implementation freedom | 5 | HIGH | Can use semantic classifiers independent of representation. |
| Runtime / resource cost | 5 | HIGH | Avoids probe computations. |
| Failure / operability | 3 | HIGH | Standard generic Error policy may be less useful than allowing callers to choose domain-specific failure behavior. |
| Deferral / reversibility / migration | 4 | HIGH | A predicate can be added later, but that would make `require` partly redundant. |
| Evidence maturity / implementation risk | 4 | HIGH | Straightforward, but no existing Protos `require` family-owner precedent matches `recognizes`. |

**Overengineering red flag:** LOW.

**Underengineering red flag:** MEDIUM because it solves only failure-oriented
validation and does not expose a reusable observation despite implementing the
same classifier internally.

### Candidate E — generic `isSemanticMemberOf`

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 5 | MEDIUM | Can be exact if Core owns descriptor validity and membership. |
| Protos alignment | 2 | HIGH | Creates a universal semantic-family relation and privileged descriptor role absent today. |
| Present-need proportionality | 2 | HIGH | Current code needs four concrete checks, not arbitrary family relations. |
| Incremental growth | 5 | HIGH | New descriptors/families fit the generic relation. |
| Future-option resilience | 3 | MEDIUM | Locks future categories into a descriptor model and forces Array/state domains into or beside it. |
| Scalability | 5 | HIGH | Runtime query is cheap; conceptual catalog grows globally. |
| Conceptual simplicity | 2 | HIGH | Must define valid descriptors, hierarchy, exact-vs-subfamily meaning and invalid-descriptor failure. |
| Portability / implementation freedom | 4 | MEDIUM | Semantic descriptors can be portable, but exposed categories constrain hidden representation choices. |
| Runtime / resource cost | 5 | HIGH | Direct classifier dispatch can be cheap. |
| Failure / operability | 3 | MEDIUM | Invalid descriptors and user-defined objects claiming descriptor roles require new failure rules. |
| Deferral / reversibility / migration | 2 | HIGH | The generic relation is hard to retract once libraries depend on it; adding it later remains feasible. |
| Evidence maturity / implementation risk | 4 | MEDIUM | Many type systems provide generic queries, but they rest on broader institutions Protos lacks. |

**Overengineering red flag:** HIGH.

### Candidate F — first-class reflection descriptors

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 5 | MEDIUM | Core can expose exact category descriptors if completely specified. |
| Protos alignment | 2 | HIGH | Introduces a new reflective object category and risks making hidden implementation taxonomy public. |
| Present-need proportionality | 1 | HIGH | Current code needs Boolean recognition, not descriptor identity/introspection. |
| Incremental growth | 5 | HIGH | Rich reflection can grow extensively once descriptors exist. |
| Future-option resilience | 2 | MEDIUM | Descriptor identity/category choices constrain future runtime/value organization. |
| Scalability | 5 | HIGH | Machine cost can remain small; conceptual surface scales poorly with categories. |
| Conceptual simplicity | 1 | HIGH | Adds descriptor identity, lookup, hierarchy, publication and reflection semantics. |
| Portability / implementation freedom | 3 | MEDIUM | Must prevent representation classes such as SmallInteger/BigInteger from leaking into descriptors. |
| Runtime / resource cost | 4 | MEDIUM | Descriptor values/caches can be cheap, but add permanent runtime/public metadata. |
| Failure / operability | 3 | MEDIUM | Requires new invalid/unknown descriptor and reflection contracts. |
| Deferral / reversibility / migration | 2 | HIGH | Very hard to retract; no high cost to deferring it today. |
| Evidence maturity / implementation risk | 4 | MEDIUM | Self mirrors and mainstream type objects prove feasibility, not suitability for this bounded need. |

**Overengineering red flag:** HIGH.

## Per-candidate incremental-design gate

### A — status quo

**Pay for what you need:** superficially yes because it adds no API, but current
callers already pay repeated source, computation and diagnostic complexity.

**Grow as you need:** weak. Each new exact family/domain requires finding another
incidental strict operation; some domains such as Array do not have a reliable
ordinary reflective test.

**Cost of deferral / reversibility:** adding an explicit recognizer later remains
possible, but deferral preserves current source debt and possible Array
misclassification.

**Smallest sufficient:** no. It is surface-minimal but no longer
requirement-sufficient once exact Array-state validation is included.

### B-prime — four evidence-scoped recognizers

**Pay for what you need:** yes. Four public messages correspond directly to four
demonstrated production receiver-domain questions.

**Grow as you need:** yes. A later family/state owner can add its own local
`recognizes` behavior without changing existing semantics or installing a
generic registry.

**Cost of deferral / reversibility:** omitting one of the four today keeps its
probe/approximation. Adding other owners later requires only a local normative
contract, implementation classifier exposure and tests.

**Smallest sufficient:** yes. It is the smallest candidate that answers every
demonstrated exact recognition need, including standard Array state.

### C — broad uniform owner recognizers

**Pay for what you need:** no. Most of the catalog is speculative today.

**Grow as you need:** yes, but there is little need to pre-grow because B-prime
extends locally.

**Cost of deferral / reversibility:** low cost to deferring unused recognizers;
that weakens the case for installing them now.

**Smallest sufficient:** no. It is regular but broader than present evidence.

### D — owner validators

**Pay for what you need:** mostly yes because current use is validation.

**Grow as you need:** partial. A later branch/query use requires adding a Boolean
predicate or encoding control flow through Error handling.

**Cost of deferral / reversibility:** a pure predicate can be added later, but
doing so makes the failure-oriented method less fundamental.

**Smallest sufficient:** it satisfies current failure gates but is less
orthogonal than B-prime for essentially the same Core classifier cost.

### E — generic semantic relation

**Pay for what you need:** no. Descriptor validity/hierarchy semantics are paid
for every user to solve four concrete queries.

**Grow as you need:** technically strong, but future growth is forced through
the descriptor institution.

**Cost of deferral / reversibility:** adding a generic relation later is feasible
if future evidence needs arbitrary family relations; there is no foundational
runtime rewrite required.

**Smallest sufficient:** no.

### F — reflection descriptors

**Pay for what you need:** no. Reflection metadata and category identity exceed
the problem.

**Grow as you need:** very strong but speculative.

**Cost of deferral / reversibility:** low cost to deferral because internal
semantic classifiers already exist; descriptors can be designed later from
tooling/reflection requirements.

**Smallest sufficient:** emphatically no.

## Mandatory adversarial questions

### What is the smallest solution satisfying current requirements?

Candidate B-prime.

The concrete evidence for each included owner is:

```text
String   repeated stdlib/Tool exact semantic String probes
Integer  ten production files using div(1)-style ordinary-Integer gates
Float    TOML exact Float probe under no-promotion arithmetic
Array    production parent()-based validation that is not exact
```

No current evidence justifies adding `Number.recognizes`, fixed-width-family
recognizers, Map/IdentityMap/Bytes recognizers, a Boolean descriptor, a null
descriptor, a generic family relation, or syntax.

### If omitted today, can broader capability be added later without breaking the model?

YES.

The owner-side model is locally extensible. Adding:

```text
Number.recognizes
UInt8.recognizes
Map.recognizes
Bytes.recognizes
...
```

later does not require changing the four initial recognizers.

A future generic reflection/type facility could also coexist if a distinct
tooling requirement eventually justifies it; B-prime does not claim that
`recognizes` is a universal descriptor protocol.

### What exactly must be rewritten later to add another recognizer?

For one additional standard owner:

- specify the exact semantic/state recognition predicate;
- install one standard local `recognizes` behavior on that owner;
- connect it to the existing semantic classifier/state check;
- add conformance tests and documentation;
- migrate any existing probe call sites if desired.

No object-model, delegation, identity, scheduler, Actor, persistence, transfer,
module/import or runtime representation architecture must be replaced.

### What current complexity would make preimplementing the future requirement regrettable?

A broad family catalog or descriptor relation would encourage nominal-style
branching and require Core to decide category questions that current APIs have
not asked:

- whether `Number` means union or exact base family;
- every fixed-width recognition relation;
- whether Map/IdentityMap/Bytes are "families" or state domains;
- how standard factories/prototypes participate;
- what user-defined descriptors mean;
- how future built-ins enter the registry.

B-prime avoids paying those decisions today.

## Failure modes and counterexamples

### B-prime false-positive risk

A recognizer must never implement semantic membership as:

```protos
value.parent() === Owner
```

or transitive delegation.

Counterexample:

```protos
lookalike: String {}
```

`String.recognizes(lookalike)` must be `false`.

### Array false-negative risk

Recognition must not require exact parent identity.

A genuine standard Array may be produced by an inherited Array factory:

```protos
MyArray: Array {}
values: MyArray(1, 2)
```

`Array.recognizes(values)` must be `true`.

### Numeric hierarchy ambiguity

`Integer.recognizes` is exact ordinary unbounded Integer recognition.

Therefore:

```text
Integer.recognizes(1)        true
Integer.recognizes(UInt8(1)) false
Integer.recognizes(1.0)      false
```

D147 does not silently introduce subtype-family matching.

### Owner/prototype self-recognition

A standard owner object is not automatically a member of the family it owns.

Under current semantics:

```text
String.recognizes(String)   false
Integer.recognizes(Integer) false
Float.recognizes(Float)     false
Array.recognizes(Array)     false
```

### Inherited recognizer invocation

Because ordinary lookup remains ordinary, another object may find a standard
owner's `recognizes` through delegation.

The standard behavior nevertheless requires its exact canonical owner receiver,
matching the existing `IpAddress.recognizes` precedent. A delegated invocation
with the wrong receiver signals under the ordinary standard receiver-domain
rule; it does not grant family authority to the descendant.

### User shadowing

A program may shadow the prelude name `String` or define its own
`recognizes` message.

That is ordinary Protos behavior. It does not change what the canonical standard
String recognizer would answer and grants no Core semantic membership.

## Future-scenario stress test

### Larger Standard Library and Tool ecosystem

If exact-family validation remains rare, B-prime stays small because recognition
is not placed on Object and there is no universal registry.

If more standard boundaries genuinely need exact receiver-domain recognition,
new owner-local recognizers can be added independently.

### Protocol-heavy application code

The main risk is cultural rather than runtime: programmers may start checking
families where ordinary protocol polymorphism would be better.

B-prime limits that pressure by:

- avoiding `Object.isType`;
- avoiding `is` syntax;
- avoiding universal descriptors;
- exposing recognition only through the exact standard owner whose contract
  already distinguishes the domain.

The programming rule remains:

> ask for the protocol you need unless the API contract genuinely requires an
> exact Core semantic/state domain.

### Alternative runtime representations

Recognition is defined semantically.

An implementation may use tagged integers, boxed BigIntegers, ropes, compact
strings, specialized arrays, side tables or other representations without
changing recognition.

Internal categories such as SmallInteger/BigInteger must remain unobservable.

### Actors, P workers and distributed transfer

No mutable global registry is introduced.

A transferred value is recognized according to its destination-side Protos
semantics/state, not host class identity or source allocation identity.

The recognizer itself performs no transfer, scheduling, callback or suspension.

### Future standard family hierarchies

B-prime does not commit all future family hierarchies to subtype-like
recognition.

A future broad Number query can be designed separately if a real API needs
"any semantic Number".

### Tooling/reflection growth

If debugger, documentation, serializer or FFI tooling later needs a complete
enumerable family descriptor universe, B-prime will be insufficient.

That is intentionally deferred because such tooling needs descriptors, names,
identity/lifetime and enumeration semantics that current validation does not
need.

### Mandatory future-regret question

**What plausible future requirement would make us regret Candidate B-prime?**

A future reflection/serialization/FFI/type-directed tooling layer may repeatedly
need to obtain and manipulate a complete first-class semantic-family descriptor
for arbitrary values, rather than ask a small known owner whether it recognizes
one candidate.

**If that happens, what escape path remains?**

Introduce a separately designed reflection/descriptor facility with explicit
tooling requirements.

The four `recognizes` messages can remain useful convenience/owner protocols or
be layered over that facility without changing their observable Boolean
contracts.

No foundational object-model rewrite is required.

## Intentionally deferred questions

Candidate B-prime deliberately does not decide:

- `Number.recognizes(value)`;
- `UInt8` / `Int8` / other fixed-width recognizers;
- Map/IdentityMap recognizers;
- Bytes recognition;
- Path/Encoding/resource-family recognition;
- Boolean/null descriptor objects;
- a universal family hierarchy query;
- user-defined family registration;
- family enumeration;
- a first-class family descriptor;
- static narrowing;
- casts;
- annotations;
- matching integration;
- `is` / `typeof` syntax.

Deferral is safe because every future owner-local recognizer is additive and the
existing runtime already owns the relevant exact classification internally.

## Strongest argument against B-prime

The strongest objection is not implementation cost; it is design culture.

Once `String.recognizes(value)`, `Integer.recognizes(value)`, and peers exist,
programmers may reach for exact-family tests instead of writing protocol-oriented
code. Over time that can recreate nominal programming habits by accretion even
without a formal type system.

Self's behaviorism guidance is directly relevant here.

The response is bounded scope and owner locality, not a claim that the risk does
not exist:

- no predicate is added to Object;
- no generic `is` relation exists;
- no descriptor universe exists;
- only four already-demonstrated exact receiver-domain contracts receive the
  operation;
- protocol-oriented code remains the normal model.

If that cultural boundary proves ineffective in real Protos code, D147's
recognizers can be re-evaluated before broadening the catalog.

## GITHUB021 invariant / delta check

Applicable current invariants:

```text
DELEGATION_DOES_NOT_CONFER_SEMANTIC_FAMILY_MEMBERSHIP
DELEGATION_DOES_NOT_CONFER_RECEIVER_OWNED_STANDARD_STATE
STANDARD_FAMILY_BEHAVIOR_VALIDATES_ORIGINAL_RECEIVER
ORDINARY_PROTOCOL_ORIENTED_PROGRAMMING_REMAINS_DEFAULT
NO_STATIC_TYPE_SYSTEM
NO_USER_DEFINED_NOMINAL_CLASS_HIERARCHY
NO_MATCHING_SEMANTIC_CHANGE_WITHOUT_EXPLICIT_DECISION
INTERNAL_REPRESENTATION_CATEGORIES_ARE_NOT_PORTABLE_SURFACE
```

Candidate B-prime preserves all of them.

Material new delta surfaced for approval:

```text
NEW_PUBLIC_STANDARD_RECOGNIZERS=
    String.recognizes(value)
    Integer.recognizes(value)
    Float.recognizes(value)
    Array.recognizes(value)

D147_SCOPE_DELTA=
    include evidence-backed Array receiver-state recognition

GENERIC_TYPE_RELATION=NO
TYPE_DESCRIPTORS=NO
STATIC_NARROWING=NO
USER_FAMILY_REGISTRATION=NO
MATCHING_CHANGE=NO
DELEGATION_AS_MEMBERSHIP=NO
```

No existing owner-approved invariant is contradicted.

```text
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Required packet checklist

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
    Self
    Pharo/Smalltalk
    JavaScript
    Python
    Ruby
    Kotlin
    C#
    Swift
    Io

RESEARCH_APPROACH_COUNT>=4
READY_FOR_EXACT_OWNER_DECISION=YES
```

## Proposal pending explicit owner approval

**Proposed candidate: B-prime — four evidence-scoped standard-owner
`recognizes(value)` operations.**

Exact proposed decision:

```text
D147_CANDIDATE=B_PRIME

STANDARD_RECOGNITION_SELECTOR=recognizes

STRING_RECOGNITION=ADD
INTEGER_RECOGNITION=ADD
FLOAT_RECOGNITION=ADD
ARRAY_RECOGNITION=ADD

STRING_RECOGNITION_DOMAIN=EXACT_SEMANTIC_STRING
INTEGER_RECOGNITION_DOMAIN=EXACT_ORDINARY_UNBOUNDED_INTEGER
FLOAT_RECOGNITION_DOMAIN=EXACT_SEMANTIC_FLOAT
ARRAY_RECOGNITION_DOMAIN=OWNS_STANDARD_ARRAY_INDEXED_STATE

RECOGNIZER_RECEIVER=EXACT_CANONICAL_STANDARD_OWNER
RECOGNIZER_ARITY=ONE
CANDIDATE_MISMATCH_RESULT=CANONICAL_FALSE
CANDIDATE_MATCH_RESULT=CANONICAL_TRUE

CANDIDATE_USER_MESSAGE_DISPATCH=NO
CANDIDATE_PARENT_LOOKUP=NO
CANDIDATE_EQUALITY_OR_HASH_DISPATCH=NO
CANDIDATE_CALLBACK=NO
CANDIDATE_CONVERSION_OR_COERCION=NO

DELEGATION_CONFERS_RECOGNITION=NO
USER_OVERRIDE_CONFERS_CORE_MEMBERSHIP=NO
STANDARD_OWNER_SELF_MEMBERSHIP=NO

ARRAY_PARENT_IDENTITY_REQUIRED=NO
ARRAY_OPEN_CLOSED_FROZEN_ALL_RECOGNIZED=YES
ARRAY_DESCENDANT_FACTORY_STANDARD_STATE_RECOGNIZED=YES

FIXED_WIDTH_INTEGER_RECOGNIZED_BY_INTEGER=NO
NUMBER_RECOGNIZER=DEFERRED
FIXED_WIDTH_RECOGNIZERS=DEFERRED
MAP_IDENTITYMAP_BYTES_RECOGNIZERS=DEFERRED
BOOLEAN_NULL_DESCRIPTOR_MODEL=NOT_ADDED

VALUE_SIDE_GENERIC_TYPE_QUERY=NOT_ADDED
CORE_FAMILY_DESCRIPTOR=NOT_ADDED
USER_DEFINED_FAMILY_REGISTRY=NOT_ADDED
MATCHING_SEMANTICS=UNCHANGED
TYPEOF_IS_SYNTAX=NOT_ADDED
STATIC_TYPING_OR_NARROWING=NOT_ADDED

PROGRAMMING_MODEL=
    PROTOCOL_FIRST_UNLESS_EXACT_CORE_RECEIVER_DOMAIN_IS_ACTUALLY_REQUIRED

D147_SCOPE_DELTA=
    INCLUDE_EVIDENCE_BACKED_STANDARD_ARRAY_RECEIVER_STATE_RECOGNITION
```

No normative specification or implementation change is authorized until the
project owner explicitly approves this exact candidate.
