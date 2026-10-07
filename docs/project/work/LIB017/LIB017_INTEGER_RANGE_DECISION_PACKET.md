# LIB017 — Integer ranges and progression iteration decision packet

## Decision state

This is the non-normative design packet for `guillermomolina/protos#570`.

```text
LIB017_STATUS=NEEDS_USER_DECISION
PROTOS_REVISION=f61bd24cf1e935591c00a07832e006fb7a256828
PROJECT_DOCS_BASE=ae821ecb952843aac9b8a7ca84394b800591dc0a
IMPLEMENTATION_AUTHORIZED=NO
SYNTAX_AUTHORIZED=NO
```

LIB017 owns Standard Library API design. It does not own new Protos grammar.
If a later decision requires dedicated range syntax, that syntax must be routed
through a separate Dxxx.

## Exact problem

Real Protos Standard Library and Tool source repeatedly spells finite bounded
integer iteration manually:

```protos
index: start
(() => index < stop).while() {
    body(index)
    index = index + 1
}
```

There are also real reverse unit-index loops such as:

```protos
index: size - 1
(() => index >= 0).while() {
    body(index)
    index = index - 1
}
```

Representative production locations include CLI, URI, network, TOML, Test Tool,
Package Tool, Files and collection implementation code.

The repository also contains many superficially similar parser/scanner loops
whose continuation depends on discovered state, delimiters, predicates or abort
flags. Those remain condition-driven `while` loops and are not evidence for
Range.

Repeated inclusive tests such as:

```protos
(octet >= 48) && (octet <= 57)
```

are real interval-shaped code, but the current spelling is already direct and
small. Their existence alone does not prove that a Range membership API should
be installed in the first slice.

The exact decision is therefore:

> What is the smallest ordinary Standard Library API that removes recurring
> bounded unit-index loop state without creating a generic iterator hierarchy,
> speculative range-value model, arbitrary stepping system, or syntax before
> current code needs them?

## Current Protos constraints

The design must preserve:

- ordinary `Integer` is exact and unbounded;
- no host integer width may become observable;
- `Closure.while` remains the general condition-driven loop;
- Array/Bytes/String indexing remains governed by existing Core contracts;
- no generic Iterable/Iterator institution is introduced by LIB017;
- no Float ranges or implicit Float-to-Integer conversion;
- no slicing semantics;
- no beginless/endless ranges;
- no comprehensions/lazy-stream framework;
- no range literal/operator is selected here;
- Standard Library APIs remain ordinary Protos library behavior;
- callback invocation remains ordinary polymorphic invocation;
- current source style prefers explicit existing idioms over unapproved sugar.

## Repository evidence

### Positive: ascending unit iteration

Current production source contains many loops whose control state is exactly:

```text
current = start
while current < stop:
    ...
    current = current + 1
```

Examples occur in:

- `protos/lib/cli/CommandLine.protos`;
- `protos/lib/uri.protos`;
- `protos/lib/io/Files.protos`;
- `protos/lib/network/IpAddresses.protos`;
- `protos/lib/network/IpEndpoints.protos`;
- `protos/lib/toml/TOML.protos`;
- `protos/tools/test/SuiteGraph.protos`;
- `protos/tools/test/Manifest.protos`;
- `protos/tools/test/Main.protos`;
- `protos/tools/test/Runner.protos`;
- multiple Package Tool modules.

This is not test-only evidence.

### Positive: reverse unit iteration

Current production code also contains pure descending unit loops, including:

- IPv6 hextet extraction in `IpAddresses.protos`;
- reverse decimal-output traversal in `TOML.protos`;
- several descending level/chunk walks in `TOML.protos`.

Therefore an ascending-only helper leaves a demonstrated bounded-loop shape
unsolved.

### Negative: state-driven scans

Examples such as URI percent-triplet scanning can increment by 1 or 3 depending
on the current character. Other parser loops stop on digits, delimiters, quote
state, abort flags or discovered syntax.

Those are not fixed progressions and must remain ordinary `while` loops.

The existence of `index = index + 2` or `+ 3` in such state machines is not
evidence for arbitrary progression step.

### Negative: no first-class Range use

No current production source was found that needs to:

- pass a Range value as data;
- store a Range value;
- compare two Range values;
- hash a Range value;
- index into a Range;
- slice a Range;
- materialize a Range to Array;
- reflect a Range value's identity;
- attach behavior to a Range instance.

The current demonstrated requirement is bounded iteration, not Range-value
transport.

### Inclusive interval tests

ASCII classification and related code repeatedly use inclusive comparisons.

Those tests are already concise and semantically direct. A half-open helper such
as `contains(48, 58, value)` would require translating the visible inclusive
upper bound `57` to `58`, while a separate closed-range API would add a second
boundary model.

The first LIB017 slice therefore need not claim that Range membership improves
those call sites.

## Comparative research

### Python — compact first-class arithmetic progression

Python `range(start, stop, step)` represents a non-materialized arithmetic
progression. The stop is half-open. Step defaults to +1, may be negative and may
not be zero. Range supports sequence-like membership, indexing, slicing and
sequence equality.

Source:

- https://docs.python.org/3/library/stdtypes.html
- https://docs.python.org/3/tutorial/controlflow.html

Contribution:

- half-open stop is strongly aligned with index iteration;
- compact non-materialization is essential;
- negative step is coherent;
- Python also demonstrates how a useful small progression can grow into a large
  Sequence surface that Protos does not currently need;
- Python permits ranges larger than host size but some `len` behavior is still
  host-sized, which Protos must not copy because ordinary Integer is unbounded.

### Ruby — generic endpoint Range

Ruby Range is a first-class begin/end value with inclusive and exclusive end
forms. It supports membership and, where the begin value has successor behavior,
iteration. It also supports beginless/endless forms and broad comparable domains.

Source:

- https://ruby-doc.org/3.4/Range.html

Contribution:

- interval membership and progression iteration can coexist in one abstraction;
- doing so broadens the semantic universe substantially;
- generic comparable endpoints, successor-based iteration, beginless/endless
  values and dual inclusive/exclusive forms exceed current Protos evidence.

### Kotlin — ranges separated from progressions

Kotlin has closed and half-open ranges, descending `downTo`, custom `step`,
and integral progressions with first/last/step state. Progressions implement
Iterable.

Source:

- https://kotlinlang.org/docs/ranges.html

Contribution:

- descending order need not be inferred from reversed bounds;
- range membership and arithmetic progression are related but separable concepts;
- custom step and Iterable integration are additional institutions rather than
  prerequisites for simple bounded loops.

### Rust — explicit half-open and inclusive bound types

Rust `Range` is half-open and `RangeInclusive` is a separate inclusive type.
Ranges integrate deeply with iterators and slicing.

Sources:

- https://doc.rust-lang.org/std/ops/struct.Range.html
- https://doc.rust-lang.org/std/ops/struct.RangeInclusive.html
- https://doc.rust-lang.org/std/ops/trait.RangeBounds.html

Contribution:

- half-open indexing is a strong independent model;
- inclusive endpoints can remain a separate concept instead of a flag;
- slicing and generic RangeBounds integration are substantial extra surface and
  should not be imported merely because bounded iteration exists.

### Swift — generic interval value with conditional iteration

Swift `Range<Bound>` is a generic half-open interval for Comparable bounds;
`ClosedRange` is distinct. Iteration is available only when the bound type has
appropriate stride behavior.

Source:

- https://developer.apple.com/documentation/Swift/Range

Contribution:

- interval membership can be generic while iteration is conditional;
- this clean separation is powerful but requires Comparable/Strideable-style
  generic institutions that Protos does not currently need.

### Pharo / Smalltalk — bounded iteration can precede a Range value

Pharo exposes direct numeric iteration through messages such as `to:do:`, while
also having first-class `Interval` objects. Intervals can use explicit step
through `to:by:` / `from:to:by:`.

Source:

- https://books.pharo.org/pharo-by-example9/
- https://books.pharo.org/updated-pharo-by-example/

Contribution:

- eliminating manual loop counters does not require making a first-class Range
  value part of the initial API;
- direct bounded iteration is especially compatible with a message-oriented
  language;
- first-class interval/progression values can be added separately when code
  actually needs to pass them around.

## Candidate set

### Candidate A — no dedicated library abstraction

Keep manual counters, ordinary comparisons and existing collection `each`.

This has zero new API surface but leaves repeated production ceremony unchanged.

### Candidate B — ascending helper only

Add one module operation:

```protos
Ranges: import("std:collections/Range")
Ranges.each(start, stop, block)
```

It enumerates the half-open integer interval `[start, stop)` in ascending unit
order.

This addresses the dominant loop shape but leaves demonstrated pure descending
unit loops manual.

### Candidate G-prime — bounded unit iteration helpers, no Range value

Add exactly:

```protos
Ranges: import("std:collections/Range")

Ranges.each(start, stop, block)
Ranges.reverseEach(start, stop, block)
```

Both operations refer to one half-open mathematical interval:

```text
start <= x < stop
```

`each` visits its values in ascending order.

`reverseEach` visits the same values in descending order:

```text
stop - 1, stop - 2, ..., start
```

No Range instance/value is created.

This is the recommended candidate.

### Candidate C — first-class unit Range descriptor

Add `std:collections/Range` with a public constructor/factory producing a
compact immutable/frozen descriptor containing half-open integer bounds.

The descriptor can then be passed to module iteration operations in ascending or
reverse order.

This preserves one bound model and creates a future home for membership/size,
but current production code does not need to transport a Range value.

### Candidate D — first-class arithmetic progression with explicit step

Adopt a Python/Pharo-like compact progression carrying start, stop and non-zero
step.

Positive steps use `x < stop`; negative steps use `x > stop`.

This naturally covers ascending, descending and arbitrary strides, but current
repository evidence does not demonstrate a fixed progression whose magnitude is
anything other than 1.

### Candidate E — separate half-open and inclusive Range forms

Expose both half-open and closed endpoint values.

This directly models current inclusive ASCII tests but adds a second value/bound
model and is not required for iteration.

**Eliminated before final scoring:** the current inclusive comparisons are
already direct; no current code needs to pass inclusive intervals as values.

### Candidate F — generic Comparable interval plus Integer progression

Separate generic interval membership from Integer progression iteration.

**Eliminated before final scoring:** Protos has no demonstrated need for generic
Comparable intervals, successor/stride protocols or a new generic iteration
institution.

### Syntax-first Range

A range literal/operator such as `a..b`, `a..<b` or equivalent is outside
LIB017 authority.

**Eliminated from the LIB017 candidate set:** current evidence does not justify
syntax independently. If ordinary API usage later demonstrates material
ergonomic insufficiency, allocate a Dxxx and compare syntax then.

## Exact Candidate G-prime contract

### Public module

```text
std:collections/Range
```

The module initially standardizes exactly:

```text
each(start, stop, block)
reverseEach(start, stop, block)
```

No standardized `Range(...)` value constructor is added.

### Bound domain

`start` and `stop` must each be ordinary unbounded Core `Integer` values.

No Float is accepted.

Fixed-width Integer-family values are not accepted in this first slice.

Rationale:

- all demonstrated loop counters and collection sizes can be expressed as
  ordinary Integer;
- ordinary Integer arithmetic is unbounded;
- accepting and normalizing all exact-integer families is an additive future
  widening and does not need to be preimplemented.

### Half-open bound policy

Both operations define the same interval:

```text
[start, stop)
```

If `start >= stop`, the interval is empty.

Bounds are ordinary call arguments and therefore are evaluated exactly once
before iteration begins.

The library does not repeatedly re-evaluate an expression such as
`source.size()` during iteration.

This difference is important when migrating a current `while`: only loops whose
bound is intended to be stable may be mechanically replaced.

### `each`

For non-empty bounds:

```text
current = start
while current < stop:
    block(current)
    current = current + 1
```

Every callback receives an ordinary unbounded Integer.

### `reverseEach`

For non-empty bounds:

```text
current = stop - 1
while current >= start:
    block(current)
    current = current - 1
```

It traverses exactly the same set of values as `each`, only in reverse order.

This avoids:

- negative-step syntax/API;
- inferred direction from reversed bounds;
- a second inclusive endpoint convention;
- sentinel calls such as "start at 7, stop at -1" for the common index case.

For example:

```protos
Ranges.reverseEach(0, 8, (index) => {
    ...
})
```

visits `7, 6, ..., 0`.

### Callback and control semantics

The `block` expression is evaluated as an ordinary argument before the module
operation begins.

The library performs no separate eager callability or arity reflection.

For every visited Integer, `block(current)` is one ordinary polymorphic
invocation with exactly one supplied argument.

Therefore:

- a non-invokable block fails when the first invocation is attempted;
- an empty interval does not invoke or otherwise inspect the block;
- callback return values are ignored;
- normal completion returns canonical `null`;
- a callback Error propagates and prevents later callbacks;
- ordinary non-local control transfer propagates and prevents later callbacks;
- a Future returned normally by a callback is just an ignored callback result;
- no implicit await, adoption, cancellation or scheduling behavior is added.

This intentionally matches ordinary library control flow rather than introducing
a special iterator execution category.

### Allocation and host-width behavior

The operations require only bounded local iteration state.

They must not materialize an Array or another collection proportional to the
number of represented Integers.

No host-width limit may alter:

- accepted ordinary Integer bounds;
- current Integer values;
- comparison;
- increment/decrement semantics.

A mathematically enormous interval is valid. Actually iterating it may consume
correspondingly enormous time/resources; this is ordinary execution cost, not a
reason to truncate the interval or expose host integer limits.

### Equality, identity and mutability

Not applicable in Candidate G-prime because no Range value is created.

This is intentional, not unspecified.

### Membership

No Range membership operation is added initially.

Current inclusive comparisons remain ordinary comparisons.

A later demonstrated need may add a half-open membership helper or a first-class
range value without changing `each` / `reverseEach`.

### Cardinality / size

No Range-size operation is added initially.

For the current unit half-open model, callers that genuinely need cardinality can
already compute the relevant ordinary Integer expression. No current source
needs a Range object whose size is observed.

### Indexing and materialization

Not added:

- no `at`;
- no bracket indexing;
- no negative indexing;
- no `toArray`;
- no slicing/subrange API.

### Step

No public step parameter exists initially.

Current production evidence justifies ascending and reverse unit traversal, not
arbitrary stride magnitude.

A future `eachBy(start, stop, step, block)` or first-class progression can be
added if real code demonstrates fixed non-unit progression.

### Interaction with collections

No collection receives new methods.

No generic Iterable/Iterator protocol is created.

Callers explicitly import and use `std:collections/Range`.

## Why direction is order, not a negative step

Candidate G-prime deliberately models one half-open set of Integer indices and
two traversal orders.

This is smaller than treating reverse iteration as a progression with a public
negative-step parameter.

It also avoids the error-prone alternative of inferring descending behavior when
`start > stop`.

Under G-prime:

```text
each(5, 2, block)        -> zero callbacks
reverseEach(5, 2, block) -> zero callbacks
```

Reversed bounds do not silently change semantic meaning.

## Comparative scoring

Scores are 1–5. Confidence is HIGH/MEDIUM/LOW.

### A — no dedicated abstraction

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 5 | HIGH | Changes nothing. |
| Protos alignment | 4 | HIGH | Ordinary `while` is valid, but repeated loop-state ceremony obscures intent. |
| Present-need proportionality | 3 | HIGH | Zero new API, but current producers repeatedly pay manual state cost. |
| Incremental growth | 5 | HIGH | A library can be added later. |
| Future-option resilience | 5 | HIGH | Preserves every future model. |
| Scalability | 3 | HIGH | Source duplication scales poorly across stdlib/tool code, though runtime semantics are sound. |
| Conceptual simplicity | 3 | HIGH | Language is simple; application code is repeatedly more complex. |
| Portability / implementation freedom | 5 | HIGH | No host coupling. |
| Runtime / resource cost | 5 | HIGH | Manual loops are minimal. |
| Failure / operability | 4 | HIGH | Explicit state is predictable but easier to update incorrectly. |
| Cost of deferral / reversibility / migration | 4 | HIGH | Adding helpers later is easy, but current maintenance debt continues. |
| Evidence maturity / implementation risk | 5 | HIGH | Status quo is proven. |

**Underengineering red flag:** MEDIUM. The repeated bounded-loop shape is now
demonstrated across production library/tool code.

### B — ascending helper only

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 5 | HIGH | Half-open +1 iteration is simple and exact. |
| Protos alignment | 5 | HIGH | Ordinary library helper, no new institution. |
| Present-need proportionality | 5 | HIGH | Directly covers the dominant shape. |
| Incremental growth | 5 | HIGH | Reverse/custom step can be added later. |
| Future-option resilience | 5 | HIGH | No value model/syntax is frozen. |
| Scalability | 4 | HIGH | Removes many manual loops, but reverse loops remain duplicated. |
| Conceptual simplicity | 5 | HIGH | One operation, one bound policy. |
| Portability / implementation freedom | 5 | HIGH | Exact Integer semantics, no host width. |
| Runtime / resource cost | 5 | HIGH | Constant state, no materialization. |
| Failure / operability | 5 | HIGH | Ordinary callback/error flow. |
| Cost of deferral / reversibility / migration | 4 | HIGH | Reverse support is additive, but current reverse evidence already exists. |
| Evidence maturity / implementation risk | 5 | HIGH | Python half-open ranges and direct bounded-loop precedents strongly support it. |

**Underengineering red flag:** MEDIUM because multiple current pure reverse-unit
loops are already known.

### G-prime — ascending + reverse unit helpers, no value

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 5 | HIGH | One half-open domain and explicit traversal order avoid inferred-direction ambiguity. |
| Protos alignment | 5 | HIGH | Small ordinary module operations; no type/iterator/value institution. |
| Present-need proportionality | 5 | HIGH | Exactly covers demonstrated +1 and -1 bounded traversal shapes. |
| Incremental growth | 5 | HIGH | Membership, step or first-class descriptors can be added independently later. |
| Future-option resilience | 5 | HIGH | Preserves all major future Range designs without preimplementing them. |
| Scalability | 5 | HIGH | Removes repeated loop bookkeeping across modules; no global coordination or iterator state objects. |
| Conceptual simplicity | 5 | HIGH | One bound model, two orders, no step/bound flags. |
| Portability / implementation freedom | 5 | HIGH | Uses semantic unbounded Integer arithmetic only. |
| Runtime / resource cost | 5 | HIGH | O(1) local state and no proportional allocation. |
| Failure / operability | 5 | HIGH | Ordinary invocation and control/error propagation, no hidden scheduler. |
| Cost of deferral / reversibility / migration | 5 | HIGH | Larger features remain additive; no foundational public representation is frozen. |
| Evidence maturity / implementation risk | 5 | HIGH | Direct repo evidence plus Python half-open and Pharo direct bounded-iteration precedent. |

**Overengineering red flag:** none.

**Underengineering red flag:** LOW. It intentionally omits first-class values and
arbitrary step, but neither omission requires foundational redesign later.

### C — first-class unit Range descriptor

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 5 | MEDIUM | A compact frozen descriptor can model bounds exactly. |
| Protos alignment | 4 | MEDIUM | Ordinary values fit Protos, but a new standardized value identity is not currently required. |
| Present-need proportionality | 2 | HIGH | Current production code does not pass/store/compare ranges. |
| Incremental growth | 5 | HIGH | Natural home for membership/size/syntax later. |
| Future-option resilience | 4 | MEDIUM | Commits future APIs to a first-class descriptor representation earlier than needed. |
| Scalability | 5 | HIGH | Compact descriptor is cheap and reusable. |
| Conceptual simplicity | 4 | MEDIUM | Adds construction, validation, identity/mutability questions absent from helper-only design. |
| Portability / implementation freedom | 5 | HIGH | Can remain pure Protos data. |
| Runtime / resource cost | 4 | HIGH | Constant-size allocation per constructed range. |
| Failure / operability | 4 | MEDIUM | Requires exact invalid-descriptor/receiver contracts if module operations accept range objects. |
| Cost of deferral / reversibility / migration | 4 | HIGH | Adding a descriptor later is bounded and need not invalidate helper APIs. |
| Evidence maturity / implementation risk | 5 | HIGH | Python/Rust/Swift/Ruby establish first-class ranges clearly. |

**Overengineering red flag:** HIGH for current Protos because no demonstrated
first-class range-data use exists.

### D — first-class arbitrary-step progression

| Criterion | Score | Confidence | Rationale |
|---|---:|---|---|
| Correctness / invariant preservation | 5 | MEDIUM | Python/Pharo demonstrate coherent nonzero-step semantics. |
| Protos alignment | 4 | MEDIUM | Ordinary compact value is plausible, but step becomes public arithmetic policy. |
| Present-need proportionality | 1 | HIGH | No demonstrated fixed non-unit progression exists. |
| Incremental growth | 5 | HIGH | Rich progression surface grows naturally. |
| Future-option resilience | 3 | MEDIUM | Freezes step/direction semantics before actual non-unit use is known. |
| Scalability | 5 | HIGH | Compact arithmetic descriptor scales. |
| Conceptual simplicity | 3 | MEDIUM | Must define sign, zero step, last-element arithmetic, membership congruence and equality. |
| Portability / implementation freedom | 5 | HIGH | Unbounded Integer arithmetic can remain host-independent. |
| Runtime / resource cost | 4 | HIGH | Compact descriptor and O(1) iteration state. |
| Failure / operability | 4 | MEDIUM | Step-zero and direction/bound mismatches require additional contract. |
| Cost of deferral / reversibility / migration | 4 | HIGH | Arbitrary step can be added later without changing unit helpers. |
| Evidence maturity / implementation risk | 5 | HIGH | Mature precedent exists; current Protos need does not. |

**Overengineering red flag:** HIGH.

## Incremental-design gate

### A — no abstraction

**Pay for what you need:** yes at API level, but current callers repeatedly pay
manual counter/state maintenance.

**Grow as you need:** excellent; nothing is committed.

**Cost of deferral:** bounded. A library helper can be added later, but current
source debt remains.

**Smallest sufficient:** no longer clearly sufficient because current production
evidence demonstrates repeated identical bounded loops.

### B — ascending helper only

**Pay for what you need:** yes for the dominant case.

**Grow as you need:** excellent.

**Cost of deferral:** reverse helper is additive.

**Smallest sufficient:** slightly too small because reverse-unit loops already
exist as current evidence.

### G-prime — unit helpers in both orders

**Pay for what you need:** yes. Every selected capability has current production
evidence.

**Grow as you need:** excellent. New operations can be added without changing
the half-open unit helpers.

**Cost of deferral:** first-class values, arbitrary step, membership and syntax
all remain bounded additive work. No object model, identity, persistence,
scheduler or storage architecture must be rewritten.

**Smallest sufficient:** yes. It addresses every demonstrated fixed unit
progression shape without adding capabilities whose only evidence is hypothetical.

### C — first-class unit descriptor

**Pay for what you need:** no. All current uses can be served without allocating
or passing a Range value.

**Grow as you need:** excellent, but future value use is preimplemented.

**Cost of deferral:** low. A descriptor can be added later while retaining the
helper API.

**Smallest sufficient:** no.

### D — arbitrary-step descriptor

**Pay for what you need:** no. Arbitrary stride and descriptor transport are both
speculative.

**Grow as you need:** strong.

**Cost of deferral:** low to medium. Adding step later requires specifying
arithmetic progression details, but does not change existing helper semantics or
fundamental runtime architecture.

**Smallest sufficient:** no.

## Required smallest-sufficient questions

### 1. What current patterns become materially clearer?

Pure bounded index loops whose only loop control is increment/decrement toward a
stable bound.

For example:

```protos
Ranges.each(1, source.size(), (index) => {
    ...
})
```

replaces explicit counter creation, condition Closure and increment.

Reverse index traversal becomes:

```protos
Ranges.reverseEach(0, hextets.size(), (index) => {
    ...
})
```

without exposing negative step or a sentinel stop.

### 2. Which loops remain `while`?

Any loop whose continuation or increment depends on:

- input content;
- parser state;
- discovered delimiters;
- variable token width;
- retry/abort state;
- Future/task outcome;
- mutable external termination state.

URI percent-triplet scans are an explicit example.

### 3. Is half-open `[start, stop)` sufficient for current indexing?

YES for bounded forward/reverse index traversal.

It matches collection index domains and keeps one bound policy.

### 4. What current use requires custom step?

No demonstrated pure fixed progression with magnitude other than 1 was found.

State-driven jumps by 2 or 3 occur in parsers but are not arithmetic
progressions.

### 5. What current use requires inclusive endpoint Range values?

Repeated inclusive comparisons exist, but no current code needs to pass,
iterate, store or compose an inclusive Range value.

Current comparisons are already concise.

### 6. What current use requires generic Comparable intervals?

None found.

### 7. What is the cost of deferring syntax?

Bounded frontend work only:

- choose syntax;
- define grammar/precedence/lowering;
- decide whether syntax produces a value or invokes a helper;
- update parser/tooling/tests/docs.

No current object/runtime boundary must be established merely to preserve the
option.

### 8. What is the cost of deferring slicing?

A later collection-API design must define bounds, failure, materialization/view
semantics and mutation interaction.

G-prime creates no representation that must be migrated.

### 9. Can enormous Integer bounds remain compact?

YES. G-prime creates no represented collection at all and holds only current
ordinary Integer loop state.

### 10. Does G-prime force Iterable/Iterator?

NO.

## Failure modes / attempted falsification

### Stable-bound trap

A current loop such as:

```protos
(() => index < source.size()).while() {
    ...
}
```

re-evaluates `source.size()` each iteration.

```protos
Ranges.each(index, source.size(), block)
```

evaluates the bound once before the helper begins.

Therefore migration is valid only when the existing dynamic re-evaluation is not
semantically required.

This is a migration audit rule, not a reason to make Range accept a condition
Closure; doing so would recreate `while`.

### Reversed-bounds bug masking

Inferring descending direction from `start > stop` would silently turn a bound
mistake into execution.

G-prime rejects that architecture: both operations define the same interval and
are empty when `start >= stop`.

### Callback validation on empty interval

An implementation must not invent eager callability reflection merely to mirror
Core Array iteration.

An empty helper interval invokes no callback, so a non-invokable block is not
observed.

This follows the smallest ordinary library implementation model.

### Huge interval resource use

The helper must not fail because the cardinality does not fit a host-sized
integer.

Actually executing an astronomically large number of callbacks can still exhaust
time/resources normally. Compact representation does not promise bounded
execution time.

### Fixed-width input pressure

Accepting fixed-width bounds now would require defining whether values preserve
family, are normalized to ordinary Integer, or may overflow during stepping.

No current use requires that choice. G-prime rejects fixed-width inputs now and
leaves broadening additive.

## Future-scenario stress

### Larger Standard Library

The helper scales as a common explicit bounded-loop primitive without imposing a
generic collection hierarchy.

### Alternative Standard Library implementations

The contract is mathematical and can be implemented with ordinary Protos loops,
specialized native code, JIT optimization or another backend without observable
difference.

### Truffle / alternative runtime

No host iterator object or Truffle-specific representation is semantic.
Migration away from current Truffle machinery does not change the API.

### Actors, Tasks, Processes and suspension

Range helpers introduce no shared state or scheduler.

Callback invocation follows ordinary execution/control rules. If callback code
uses Futures or other concurrency facilities, their existing ownership and
suspension semantics remain authoritative.

### Distribution / serialization / persistence

G-prime creates no Range value, so there is no new serialized identity or
transfer category.

This is a positive deferral property: if first-class Range values later require
transfer/persistence behavior, that decision can be made with actual evidence.

### Future syntax

A future Dxxx may choose syntax that:

- lowers to these helper operations;
- constructs a new first-class Range value;
- adopts a different richer progression model.

G-prime does not force one of those outcomes.

### Future regret question

**What plausible future requirement would make us regret G-prime?**

A later API ecosystem may frequently pass ranges as values — for slicing,
query planning, partitioning, serialization, generic algorithms or DSL
composition — and may require equality/hash/membership independent of immediate
iteration.

**What escape path remains?**

Add a first-class compact Range descriptor to `std:collections/Range` (or route
language syntax through a Dxxx if syntax is warranted) while keeping
`each(start, stop, block)` and `reverseEach(start, stop, block)` as stable
convenience operations.

No existing Range-value representation needs migration because G-prime defines
none.

## Strongest argument against G-prime

The strongest objection is API layering:

> If Protos is clearly going to need a Range value eventually, adding only
> iteration helpers now may create a transitional API and miss the opportunity
> to establish one elegant first-class abstraction once.

Python, Rust, Swift, Ruby and Pharo all provide first-class range/interval values
in mature ecosystems.

The counterweight is current evidence and deferral cost:

- current Protos code repeatedly iterates bounds but does not transport ranges;
- a first-class value immediately forces identity/equality/mutability/validation
  choices;
- adding that value later is bounded and does not invalidate helper semantics;
- G-prime is future-compatible without being future-preimplemented.

Under current AGENTS.md anti-overengineering rules, the future-value possibility
is therefore a reason to preserve the module namespace and escape path, not to
install the value model now.

## Intentionally deferred

G-prime leaves unresolved:

- first-class Range values;
- Range equality/hash/identity;
- Range mutation/open/closed/frozen state;
- arbitrary positive/negative step;
- fixed-width Integer bounds;
- generic comparable bounds;
- membership;
- cardinality/size;
- Range indexing;
- Array materialization;
- slicing/subranges;
- inclusive Range values;
- beginless/endless ranges;
- Float ranges;
- generic Iterable/Iterator;
- lazy streams;
- comprehensions;
- range syntax/operators.

Every deferred capability can be added without invalidating the selected unit
helper model.

## Constraint consistency check

```text
ORDINARY_INTEGER_UNBOUNDED=PRESERVED
HOST_WIDTH_UNOBSERVABLE=PRESERVED
CLOSURE_WHILE_GENERAL_LOOP=PRESERVED
NO_GENERIC_ITERABLE=PRESERVED
NO_FLOAT_COERCION=PRESERVED
NO_SLICING_DECISION=PRESERVED
NO_RANGE_SYNTAX_DECISION=PRESERVED
RANGE_SYNTAX_REVISIT_AFTER_REAL_USAGE=PRESERVED

MATERIAL_NEW_PUBLIC_SURFACE=
    std:collections/Range.each
    std:collections/Range.reverseEach

LIB017_CONSTRAINT_CONSISTENCY=PASS
```

## Decision-packet checklist

Although GITHUB010's exhaustive-matrix mandate is formally scoped to Dxxx and
PLATxxx, this LIB017 packet applies the same discipline because the Standard
Library API choice is substantive.

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
    Python
    Ruby
    Kotlin
    Rust
    Swift
    Pharo/Smalltalk

RESEARCH_APPROACH_COUNT>=4
READY_FOR_EXACT_OWNER_DECISION=YES
```

## Proposal pending explicit project-owner approval

**Proposed Candidate G-prime — bounded unit iteration helpers only.**

Exact proposed decision:

```text
LIB017_CANDIDATE=G_PRIME

STANDARD_MODULE=std:collections/Range

PUBLIC_OPERATIONS=
    each(start, stop, block)
    reverseEach(start, stop, block)

FIRST_CLASS_RANGE_VALUE=NO
STANDARD_RANGE_CONSTRUCTOR=NO

BOUND_MODEL=HALF_OPEN
BOUND_SET=start <= x < stop
REVERSED_BOUNDS=EMPTY_NOT_DESCENDING

EACH_ORDER=ASCENDING_UNIT
REVERSE_EACH_ORDER=DESCENDING_UNIT_OVER_SAME_HALF_OPEN_SET

START_DOMAIN=ORDINARY_UNBOUNDED_INTEGER
STOP_DOMAIN=ORDINARY_UNBOUNDED_INTEGER
FIXED_WIDTH_INTEGER_BOUNDS=REJECT
FLOAT_BOUNDS=REJECT

BOUNDS_EVALUATED_ONCE=YES
HOST_WIDTH_LIMIT=NO
PROPORTIONAL_MATERIALIZATION=NO

CALLBACK_INVOCATION=ORDINARY_POLYMORPHIC_CALL
CALLBACK_ARGUMENT_COUNT=ONE
CALLBACK_ARGUMENT=ORDINARY_INTEGER
EAGER_CALLBACK_VALIDATION=NO
EMPTY_RANGE_INSPECTS_CALLBACK=NO
CALLBACK_RESULT=IGNORED
NORMAL_OPERATION_RESULT=NULL
ERROR_AND_NONLOCAL_CONTROL=PROPAGATE
IMPLICIT_FUTURE_AWAIT_OR_ADOPTION=NO

MEMBERSHIP=DEFERRED
SIZE_CARDINALITY=DEFERRED
INDEXING=DEFERRED
TO_ARRAY=DEFERRED
ARBITRARY_STEP=DEFERRED
INCLUSIVE_RANGE=DEFERRED
GENERIC_COMPARABLE_INTERVAL=DEFERRED
ITERABLE_ITERATOR_PROTOCOL=NOT_ADDED
SLICING=DEFERRED
RANGE_SYNTAX=DEFERRED_TO_SEPARATE_DXXX_IF_REAL_USAGE_JUSTIFIES_IT

MIGRATION_RULE=
    ONLY_REWRITE_EXISTING_WHILE_LOOPS_WHEN_THE_BOUND_IS_SEMANTICALLY_STABLE
```

No implementation, source migration or syntax work is authorized until the
project owner explicitly approves this exact candidate.
