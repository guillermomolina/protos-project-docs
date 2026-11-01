# D232 — First-class directed Range values and two distinct inversion operations

**D232 state:** **RATIFIED — OWNER-APPROVED D232 FINAL CONTRACT (2026-11-01)**. This is the durable design authority for subsequent normative Standard Library specification and implementation, **not** a claim that the product's spec, source, tests or version have already changed.

- Formal owner: [D232 #1014](https://github.com/guillermomolina/protos/issues/1014), [guillermomolina/protos](https://github.com/guillermomolina/protos).
- Approval provenance: active 2026-11-01 owner conversation: (1) exact **“ok apruebo esto último”** selecting the two-operation `invert()/reverse()` proposal with concrete outputs; (2) subsequent explicit **“Si lo apruebo”** selecting the exact three-part constructor contract presented for D232. Earlier owner invariants require direct `InvertedRange(3,0)` visiting `3,2,1`, direction-sign step validation, and removing the redundant old helpers.
- Evidence: [comparative research, GITHUB010 scores, LIB017 conflict and falsification](https://github.com/guillermomolina/protos-project-docs/blob/949dd487c6931565baec7e9d6fc648f0bc7a47d8/docs/project/evidence/D232/D232_DIRECTIONAL_RANGES_COMPARATIVE_AND_INVARIANTS_2026_11_01.md).
- Product inspected: [guillermomolina/protos@fba47c40c4b79b6e580df1c1f78a3fcecbd067e2](https://github.com/guillermomolina/protos/commit/fba47c40c4b79b6e580df1c1f78a3fcecbd067e2), 2026-11-01, after I135. No repository mutation to Protos as part of this decision.

## Selected public semantics (implementation-independent)

The Standard Library shall represent two **ordinary, first-class Protos prototype-backed range values** (not new Core semantic families or parser syntax): `Range` and `InvertedRange`. Both can be directly constructed. They represent directed, bounded integer unit-step sequences, not eager Arrays or automatically materialized Iterators.

Let `a <= b` be genuine unbounded mathematical Integers:

| Constructor / value | Members in traversal order | First endpoint | Second endpoint |
| --- | --- | --- | --- |
| `Range(a,b)` | `a,a+1,...,b-1` | included | excluded |
| `InvertedRange(b,a)` | `b,b-1,...,a+1` | included | excluded |

For example, `Range(0,3)` yields `0,1,2`; direct `InvertedRange(3,0)` yields `3,2,1`. Equal endpoints represent an empty traversal in either orientation. No sign-swap heuristics and no auto-reversal on a wrong-signed step.

### Different, explicitly approved transformations

**`invert()` is a practical, endpoint-swapping inversion** (not set-preserving):

```text
Range(a,b).invert()           -> InvertedRange(b,a)
InvertedRange(b,a).invert()   -> Range(a,b)

Range(0,3)          0,1,2
  .invert()         3,2,1
```

**`reverse()` is an element-preserving reversal** (orientation changes; original unit-interval members are unchanged):

```text
Range(a,b).reverse()          -> InvertedRange(b - 1, a - 1)
InvertedRange(b,a).reverse()  -> Range(a + 1, b + 1)

Range(0,3)          0,1,2
  .reverse()        2,1,0
```

Both are involutive *in terms of directional descriptor and unit-step enumeration*: `invert().invert()` and `reverse().reverse()` recover the original represented limits/orientation, but are not required to preserve original object identity `===`. Addition/subtraction by 1 is exact even across SmallInteger/BigInteger boundaries. The formulas also apply when `a=b`: e.g. `Range(0,0).reverse()` is `InvertedRange(-1,-1)` (empty), and a second `reverse()` recreates `Range(0,0)`.

**`each(block)`** visits each member in the value's direction with magnitude 1. **`eachStep(step,block)`** uses signed genuine mathematical Integer step with mandatory matching orientation: positive on `Range`, negative on `InvertedRange`; zero or opposite sign signals ordinary `Error` even for an empty interval. Example:

```text
Range(0,11).eachStep(3)             0,3,6,9
Range(0,11).invert().eachStep(-3)  11,8,5,2
Range(0,11).reverse().eachStep(-3) 10,7,4,1
```

The `reverse` operation preserves the full underlying unit-step member set, **not the subset produced by a previously chosen step**: `0,3,6,9` reversed would be `9,6,3,0`, a separate progression operation not approved here. Iteration must use bounded local state, not materialize the interval.

All iteration callbacks retain applicable LIB017 semantics: one ordinary polymorphic invocation `block(integer)` per reached value, ignore its result, answer canonical `null` upon ordinary completion; do not check invocation eligibility for empty traversal; propagate the exact signaled Error, lexical `^` nonlocal-return/unwind and ordinary cancellation/suspension semantics, with no implicit Future await. Arguments evaluate in ordinary strict left-to-right order once before validation and iteration.

## Previously selected surface explicitly replaced

The owner requested **removal**, not retention as aliases, of existing published module helper operations:

```text
std:collections/Range.each(start,stop,block)
std:collections/Range.reverseEach(start,stop,block)
```

The replacement is first-class instance `each(block)` and `eachStep(step,block)` on the oriented ranges. This expressly reopens the LIB017 G-prime public-selector **compatibility** promise; it is a conscious source-breaking migration, not an additive compatibility-preserving change. Existing in-repository examples and conformance tests must migrate, while external code may break. The old LIB017 record stays as historical provenance, without rewriting its earlier approval.

This approval does not add arbitrary float ranges, generic Comparable ranges, inclusive-ranges, slicing, indexing, cardinality, membership, collections materialization, generic Iterator/Iterable, unbounded/endless ranges, new `for` syntax, or Java/Graal/Truffle instructions. [D231 #1013](https://github.com/guillermomolina/protos/issues/1013) owns future syntax; [PERF045 #889](https://github.com/guillermomolina/protos/issues/889) owns measured performance.

## GITHUB021 consistency check

| Ratified or explicit owner invariant | Selected D232 consequence | Disposition |
| --- | --- | --- |
| LIB017 genuine unlimited Integers and half-open bounded iteration | Same integer family, order and exact arithmetic | PRESERVED |
| LIB017 normal callback evaluation/one arg/null/errors/unwind/no await | Contract applies to both new instance selectors | PRESERVED |
| LIB017 no proportional sequence allocation | Ordinary small descriptor only; iteration constant state | PRESERVED |
| LIB017 fixed public helper selectors with empty behavior for `start>=stop` | Published helpers deliberately replaced; constructor ordering still requires a separate exact contract | REOPENED: API removal OWNER APPROVED; constructor invalid-order behavior NOT YET RATIFIED |
| LIB017 no first-class Range objects | Two ordinary prototypes and user-facing range values | REOPENED: OWNER APPROVED |
| Owner: InvertedRange(3,0) must yield 3,2,1 | Direct descending first-inclusive/last-exclusive | PRESERVED |
| Owner: signed step belongs to range direction | Positive/negative validation as defined above | PRESERVED |
| Owner: distinct practical vs mathematical inversions | `invert` swaps bounds; `reverse` preserves unit members | PRESERVED |
| Protos: one immutable delegation parent, ordinary `call` and identity, frozen publication semantics | No new runtime category or special syntax; concrete export/mutation semantics still need exact decision | PRESERVED IN PRINCIPLE / PUBLIC CONTRACT PENDING |

**OWNER_APPROVAL_PROVENANCE=PASS for the precisely recorded orientation, two transformations and public-helper-removal direction.** **DECISION_INVARIANT_CONSISTENCY=PASS for those invariants; NOT YET CHECKABLE for unresolved construction details.** These status markers are intentionally *not* an overall completed-ratification PASS.

## Final owner-approved construction, export and object contract

**Exact owner approval:** **“Si lo apruebo”**, following the assistant's concrete presentation of these three elements on 2026-11-01; this approval explicitly completes the previously outstanding public constructor gate.

### Module exposure and normal prototype invocation

The existing `std:collections/Range` import yields the ordinary **module object**, which exports two ordinary prototype objects as members named `Range` and `InvertedRange`. A consumer binds the objects explicitly before invoking them:

```protos
Ranges: import("std:collections/Range")
Range: Ranges.Range
InvertedRange: Ranges.InvertedRange

ascending: Range(0, 10)
descending: InvertedRange(3, 0)
```

This follows the ordinary module and prototype invocation pattern used in `std:math/LinearAlgebra`, not an ad hoc parser construct. `Ranges.Range(0,10)` and `Ranges.InvertedRange(3,0)` are **not** interchangeable invocation forms under ordinary Protos member-call semantics; direct use requires first extracting/binding the prototype as shown. The name `Range` on the module is **not** an old module-level iteration helper. Each constructed instance's immediate delegation parent is its respective actual prototype: `ascending.parent() === Range`, `descending.parent() === InvertedRange`. The exported prototypes use normal Protos object/delegation rules and are not new Core value-identity families.

### Strict constructor arguments and endpoints

Exactly two explicit semantic, mathematical, unbounded `Integer` arguments are required; zero, one, or three-or-more supplied positional arguments signal ordinary `Error`. `null`, Float, Fraction, Complex and objects merely delegating to `Integer` are invalid. A constructor must enforce exact arity explicitly; Core's ordinary Closure parameter binder alone would ignore surplus arguments. No implicit numeric conversion occurs. Protos evaluates the receiver and every supplied argument **once, strictly left-to-right, before any constructor validation or allocation/initialization as governed by ordinary `Object.call`**. Constructor validation signals `Error`; no partially initialized instance is normally returned. If several supplied values are invalid, the first particular failed validation need not define an additional `Error` subclass or precedence beyond ordinary evaluation effects; the only portable required failure is `Error`.

For the two constructors:
- `Range(start,stop)` requires **`start <= stop`**; otherwise `Error`. It visits `start,start+1,...,stop-1`.
- `InvertedRange(start,stop)` requires **`start >= stop`**; otherwise `Error`. It visits `start,start-1,...,stop+1`.
- `start == stop` is a valid empty instance in either orientation (and does not loosen incompatible `step` validation).
- Bound arithmetic, including `reverse()`'s ±1, is exact unbounded Integer arithmetic with no host-word truncation.

### Observable, frozen instance state

Each successfully constructed object has **own local slots named `start` and `stop`** storing the exact supplied semantic Integers; they are visible via ordinary member reads and Core reflection such as `hasSlot`, `slotValue`, and `slotNames`. After successful initialization, the instance is **structurally FROZEN** using ordinary Protos object-state semantics. Consequently its existing slots cannot be reassigned or removed and new local slots cannot be added; its two stored Integers are already immutable numeric values. `start` and `stop` are in traversal argument order, not normalized to numeric lower/upper. `invert()` and `reverse()` return **fresh ordinary frozen instances** of the opposite prototype and do not mutate the original.

Ordinary Protos identity and delegation still apply. No structural/value equality contract, `hash` override, range recognition intrinsic, host-backed storage or custom `Iterable` protocol is selected. User source is free to reference/override inherited ordinary methods according to standard object model rules; an operation cannot use a hidden runtime exception to bypass visible selector dispatch. Standard-construction invariants apply to successfully constructed instances, not to arbitrary user-authored objects deliberately delegating to a prototype.

### GITHUB021 final invariant/delta attestation

- **Exact owner-approved preserved:** direct two-prototype construction and direction-specific first-inclusive/last-exclusive intervals; `InvertedRange(3,0)` visits `3,2,1`; `invert` swaps endpoints and changes membership, `reverse` adjusts ±1 to preserve members; both are involutive on logical descriptor; `eachStep` sign constraint; explicit removal of old helpers; ordinary Protos implementation and historical LIB017 semantics that remain applicable.
- **Later approved additions:** explicit prototype members/extraction, exact two-Integer constructor arguments, strict ordering with equal empty, frozen locally reflectable `start`/`stop` instance slots, ordinary parent identities. These directly replace the previously pending public construction questions; they introduce no contradiction with prior owner-approved directional semantics.
- **LIB017 intentional reopen:** source compatibility for module `each`/`reverseEach` is deliberately broken; old `start>=stop -> empty` helper behavior does not immunize a newly constructed `Range(start,stop)` against the approved wrong-order `Error`. Original LIB017 decision remains historical; no aliases/wrappers are selected.
- **No unapproved extensions:** no new `Range` syntax, no Core prototype/identity hierarchy, no general Iterator, no Float/Comparable ranges, no hidden native/Truffle acceleration. No new semantic approval is inferred beyond this exact contract.

`OWNER_APPROVAL_PROVENANCE=PASS` for both explicit conversational approvals; `DECISION_INVARIANT_CONSISTENCY=PASS` for the exact approved contract and intentional LIB017 compatibility reopen; `EXACT_PUBLIC_CONTRACT=SELECTED`. The published project decision is sufficient to release **one** follow-up I136 implementation slice; implementation edits, product spec changes and tests remain human-executor-gated and have **not** occurred under D232.

## Delivery status

- The final owner-approved semantic model, public constructor/export/identity contract and GITHUB021 reconciliation have been durably ratified in `guillermomolina/protos-project-docs`.
- Next work: one [I136 implementation issue](https://github.com/guillermomolina/protos/issues/1015) in `guillermomolina/protos`, with a self-contained agent prompt and separate human-executor workflow. This design decision alone is **not** production implementation or normative spec publication.
- No product code, tests, benchmark, native build, spec change, version bump or product commit is claimed by this docs-only ratification. D232 Issue closure and I136 readiness must be verified through live GitHub coordination after document publication.
