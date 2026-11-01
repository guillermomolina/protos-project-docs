# D232 — First-class directed Range values and two distinct inversion operations

**D232 state:** **OWNER-SELECTED SEMANTIC MODEL; EXACT CONSTRUCTION CONTRACT AND NORMATIVE RATIFICATION STILL PENDING**. This is a role-first durable language-decision checkpoint, not a completed normative language specification or a record of product implementation.

- Formal owner: [D232 #1014](https://github.com/guillermomolina/protos/issues/1014), [guillermomolina/protos](https://github.com/guillermomolina/protos).
- Approval provenance: active 2026-11-01 owner conversation, exact **“ok apruebo esto último”** after the two-operation `invert()/reverse()` model was explained with examples; relevant explicit earlier owner preferences: direct `InvertedRange(3,0)` must visit `3,2,1`, step-sign validation belongs to direction, and old module helpers should be removed rather than retained indefinitely.
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

## Outstanding exact public-contract gate (do not invent implementation authority)

Before closing D232 or releasing an implementation Issue, resolve at the owner approval gate:
1. **Constructor/export surface and topology.** The visible binding spelling `Range(0,10)` and `InvertedRange(3,0)` is selected, but whether the module exports two prototypes to bind explicitly (as in `std:math/LinearAlgebra`) or supplies a module-level factory/selector, and their observable `parent()`/reflection identity, is not selected. Avoid unapproved special semantics for ordinary module contexts.
2. **Wrong-ordered bounds.** The last explicit approval included equal-bound empties and direction-specific steps. Rejecting `Range(10,0)` and `InvertedRange(0,10)` with `Error`, versus preserving an empty range, changes existing behavior and must be explicitly chosen, not inferred from implementation convenience.
3. **Exact constructor error, arity and object mutability contract.** Ordinary Protos Closure binding accepts extra arguments; `Object.init` has different arity constraints. Decide whether constructor arguments are exact, validation order (including failed initializer), read-only/frozen instance semantics, fields visible via reflection, and constructor overriding/dispatch. These details are observable and cannot be delegated to arbitrary user code without a contract.

The proposed smallest matching implementation boundary is one **`I136`**, solely in `guillermomolina/protos`, with source-backed prototypes/methods, migrated examples, grouped tests, required normative specification and changelog, and any necessary version change. **I136 must remain blocked by the unresolved public-contract gate; this record is not implementation authorization.** The implementation agent should use local HEAD and this fully reproduced contract, never fetch another repo or browse.

## Delivery status

- Evidence and selected semantic checkpoint published by GitHub docs-only commits, under the owner's explicit bounded exception for `guillermomolina/protos-project-docs`.
- No production code, tests, benchmark, native build, spec change, version bump or commit in `guillermomolina/protos`.
- Do not close D232 or mark an implementation ready until the exact public-contract gate is approved, revision-bound decision is complete and cross-references verified.
