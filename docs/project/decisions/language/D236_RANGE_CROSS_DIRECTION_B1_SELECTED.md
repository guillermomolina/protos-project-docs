# D236 — Cross-direction Range construction, B1 selected

**State: RATIFIED / OWNER-APPROVED D236-B1 (2026-11-01).** Durable non-normative language-design decision, not a claim that the Protos product has implemented or tested it. Normative authority remains `guillermomolina/protos/spec/`.

- [D236 #1028](https://github.com/guillermomolina/protos/issues/1028): design owner and approval provenance.
- [Exhaustive GITHUB010/GITHUB021 investigation](https://github.com/guillermomolina/protos/issues/1028#issuecomment-6555685657), including twelve scored dimensions, adversarial matrix, and alternatives A/B/C/D/E.
- [Follow-up: numeric-normalization rationale](https://github.com/guillermomolina/protos/issues/1028#issuecomment-6555736671).
- [Approval/invariants evidence](../../evidence/D236/D236_B1_OWNER_APPROVAL_INVARIANTS_2026_11_01.md).
- Source state evaluated: [`guillermomolina/protos@8af1a70a56ca113a94eea1a5935a74a80fecdfd2`](https://github.com/guillermomolina/protos/commit/8af1a70a56ca113a94eea1a5935a74a80fecdfd2) (2026-11-01). Implement from the executor's **actual local HEAD**, not this historical allocation baseline.

## Exact owner approval

The project owner said **“ok apruebo B1”** immediately after the D236 exhaustive B1 packet, and explicitly instructed issue coordination, evidence publication, and one self-contained implementation slice. Earlier, the owner challenged the assumption that wrong-bound ordering is inherently an Error by comparing range construction with exact rational normalization: **“es como cuando 1/2 + 1/2 pasa de Fraction a Integer, no?”**. This approves B1 **as fully specified in the investigation**. In particular it explicitly reopens the earlier D232 strict wrong-order-Error contract and D235 exact-five-own-methods/inherited-`Object.call` contract. It does not select a `for` syntax, a step-driven factory, a new Integer domain, a Java intrinsic, or a new Range value-family kind.

## Selected semantic contract (implementation-independent)

**B1: symmetric, canonical-only, source-backed standard factory specialization.** Each of the existing canonical **frozen, physically shared, ordinary Core** prototypes, `Range` and `InvertedRange`, owns one additional local standard Closure `call`. This is the ordinary `CALLABLES.md` polymorphic-call mechanism; no host exception or change to standard `Object.call` is introduced.

For completed, genuine two-Integer argument vector `[start,stop]`, invocation of the **canonical** prototypes returns a **fresh frozen ordinary instance of the direction selected by the exact ordered bounds**:

| Canonical invocation | Parent and own stored bounds of result | Unit traversal |
| --- | --- | --- |
| `Range(0,5)` | `Range`, `start=0,stop=5` | `0,1,2,3,4` |
| `Range(5,0)` | `InvertedRange`, `start=5,stop=0` | `5,4,3,2,1` |
| `InvertedRange(5,0)` | `InvertedRange`, `start=5,stop=0` | `5,4,3,2,1` |
| `InvertedRange(0,5)` | `Range`, `start=0,stop=5` | `0,1,2,3,4` |
| `Range(a,a)` | `Range`, `a,a` | empty |
| `InvertedRange(a,a)` | `InvertedRange`, `a,a` | empty |

No endpoint swap or arithmetic normalization is performed by construction. First endpoint **inclusive**, second **exclusive** regardless of direction. Equal bounds retain **the explicitly invoked canonical prototype** (both empty). No instance reuse, interning, coercion, materialized element Array, host-width overflow, or structural equality. The two `start`/`stop` instance slots remain exactly reflectable, frozen and unchanged; the instance's immediate parent is the actual resulting canonical directional prototype. Each normal invocation produces a new identity, even if the descriptor duplicates a previous result.

Canonical `call` admission: ordinary receiver/arguments evaluated once left-to-right; normal `call` lookup/Closure methodHome; **exactly** two caller-supplied arguments (including rejects extras); both `Integer.recognizes` (no Float/Fraction/Complex/`null`/delegated Integer). Invalid arity/type signals ordinary `Error` regardless of direction, before successful range construction or slot publication. On correctly directed or equal endpoints, the factory uses ordinary inherited `Object.call` (e.g. source-level `super.call(start,stop)`) to construct the requested type with the existing strict `init`. On opposite bounds it **invokes the other canonical prototype's ordinary factory with the original arguments**; the opposite factory takes its already-valid local branch. Result errors and control transfers have ordinary Closure/handler/ensure precedence; no implicit suspension or Future await.

**Noncanonical receivers:** a derived/custom object inheriting one of these `call` Closures must not unexpectedly become a canonical Range merely due to inherited dispatch. The factory first guards that its invocation receiver is its own **canonical frozen prototype identity**, then uses inherited `Object.call` (preserving original receiver, methodHome and its customizable `init`) for any noncanonical receiver. A nearer user-defined `call` retains precedence. `Range.init` and `InvertedRange.init` remain separately invoked **strict initializers**, including their exact-parent precondition; return values of `init` still do not control `Object.call` construction. No mutation of the immutable parent is allowed.

**Names and ownership:** canonical factories resolve the counterpart via their existing frozen **Core bootstrap lexical capture**, not the invoking Actor's mutable `std:collections/Range` export or a caller's shadowed name. Ordinary `Range: custom` lexical shadowing, `Range.call` extraction/binding, user-created local `call` overrides, reflected `slotNames` and `methodHome` follow normal rules. Both canonical prototypes continue to delegate directly to `Object`, freeze with their standard method Closures/captures before guest observation, and remain physically shared across Actors/P. The importable source-backed `std:collections/Range` stays an ordinary Actor-local module with exactly two own exports initialized to these identities; module rebinding is isolated and has no effect on canonical internal factories. No special transfer, sharing, scheduling or parser rule.

**Unchanged directional operations:** `Range.eachStep` requires a strictly positive genuine Integer step and `InvertedRange.eachStep` a strictly negative one; zero/wrong sign Error even on empty. `each` has unit direction and `null` completion. Callbacks run normally once per visited Integer, ignore result, propagate `Error`, non-local `^`, `ensure`, cancellation and explicit suspension normally. No implicit await; no proportional collection materialization; Integer arithmetic unbounded. `invert` switches endpoints and direction, potentially changing the visited set; `reverse` preserves the unit-step visited set and reverses order:

```text
Range(a,b).invert()           -> InvertedRange(b,a)
Range(a,b).reverse()          -> InvertedRange(b-1,a-1)
InvertedRange(a,b).invert()   -> Range(b,a)
InvertedRange(a,b).reverse()  -> Range(b+1,a+1)
```

Both create fresh frozen results. For every already-valid descriptor (including empty), the inputs of these transformations remain valid in the target type; no old transformation is reinterpreted through the new cross-direction branch. `invert().invert()` and `reverse().reverse()` restore the *descriptor*, not object `===` identity.

## GITHUB021 — explicitly reopened promises and preserved invariants

| Ratified earlier authority | D236-B1 disposition |
| --- | --- |
| D232: wrong-bound `Range` / `InvertedRange` constructors signal `Error` | **EXPLICITLY REOPENED AND REPLACED** by symmetric direction-selected canonical constructor result |
| D235-B1-F/D232: **five** own prototype methods, no local `call`, construction uses inherited `Object.call` | **EXPLICITLY REOPENED AND REPLACED** by exactly **six** own methods (`init,each,eachStep,invert,reverse,call`) and ordinary specialized `call` with inherited `Object.call` on normal branch |
| Two frozen canonical Core prototypes, direct Object parents, same Prelude/module export identities, Actor/P shared graph isolation | **PRESERVED** |
| Exactly two genuine unbounded Integers, eval once left-to-right, original endpoints, equal boundaries valid and empty | **PRESERVED** (equal direction retains invoked prototype) |
| Strict existing `init` and exact immediate-parent guard; previously valid ordinary derived construction | **PRESERVED** via guarded source factory |
| Fresh ordinary frozen instances, exactly two local slots, ordinary identity/hash/equality, source-backed algorithms | **PRESERVED** |
| Strict step sign, `each`, callbacks, Error/nonlocal return/ensure/Future, laziness | **PRESERVED** |
| Distinct `invert` vs `reverse` and their descriptor involutions | **PRESERVED** |
| Standard `Object.call` and ordinary overriding/extraction/super message semantics | **PRESERVED** as global language rules; local Core reflection changes are explicit |
| D231 `for(item in expression)` and numeric `for(i,start,stop[,step])` | **OUT OF SCOPE / STILL UNSELECTED**; B1 simplifies only no-step bound-direction selection, not opposite-sign runtime step |

## Implementation / compatibility boundary

This decision authorizes a **separate one-slice Ixxx implementation** in `guillermomolina/protos`, **Protos-first**. Minimum expected surfaces: `protos/lib/core/Range.protos`, `spec/semantics/VALUES_AND_COLLECTIONS.md`, `spec/PROTOS_SPEC_CHANGELOG.md`, directly affected Protos conformance and Java bootstrap reflection tests, and existing guide/source documentation. Only touch other files where an independently verified affected dependency requires it. No new Java range kernel, parser, iterator, Core value category, helper export, module identity change, D231 sugar or speculative performance redesign. Version/revision must derive from implementation-time HEAD after reported validation, never reserved here. The editing agent edits only; the human executes validations, source product commit/push and reports results.

**RATIFIED DESIGN != IMPLEMENTED PRODUCT.** This record is the approved invariant handoff. See the linked D236 issue/evidence for research breadth and approval provenance.
