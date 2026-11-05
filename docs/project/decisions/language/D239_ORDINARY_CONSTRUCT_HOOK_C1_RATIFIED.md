# D239 — Ordinary `construct` hook, selected C1

**Decision status:** OWNER-APPROVED C1, 2026-11-05; durable non-normative ratification record. **Implementation and normative specification publication remain pending I145.**
**Governing issue:** [D239 #1055](https://github.com/guillermomolina/protos/issues/1055).
**Implementation owner:** [I145 #1056](https://github.com/guillermomolina/protos/issues/1056), a separate single-slice implementation issue.
**Exact inspected product baseline:** [guillermomolina/protos@5fc18249ea425d99fcfcfe387be950678b7ac5c5](https://github.com/guillermomolina/protos/commit/5fc18249ea425d99fcfcfe387be950678b7ac5c5). This is a **pre-implementation** revision.
**Selection provenance:** On 2026-11-05, after the exhaustive D239 investigation and the exact presented recommendation “D239-C1 — Ordinary construct hook”, the project owner replied **“Apruebo C1, Actualiza issues en guillermomolina/protos y agrega evidencias en guillermomolina/protos-project-docs.”** The prior decision packet explicitly disclosed the changed meaning of pre-existing `construct` slots, the replacement of `Range.call` with `Range.construct`, the two forwarding layers, and incompatibility with some reflection. This is approval of C1, not approval of unmentioned future features.
**Comparative/GITHUB021 evidence:** [D239 C1 approval and implementation boundary](../../evidence/D239/D239_C1_APPROVAL_INVARIANTS_AND_PROTOS_FIRST_IMPLEMENTATION_2026_11_05.md).
**Authority:** This repository records approved design rationale; only the corresponding ratified `guillermomolina/protos/spec/` edits establish normative published language semantics. No code, tests, benchmark or version bump is claimed here.

## Selected semantic model

Preserve the single ordinary-slot D013 invocation universe. Parenthesized `receiver(args)` still evaluates the receiver and each argument once (including named/spread) before **ordinary `call` lookup** on the evaluated receiver. The exact selected Closure and its receiver/`methodHome` govern execution and failure precedence. An explicit, nearer user `call` continues to take precedence. Non-Closure `call` shadows still signal Error rather than being skipped. No new constructor kind, callable bit, class/metaclass hierarchy, `new` keyword, syntax transform, registry or separate invocation dispatch is introduced.

When the **actual standard `Object.call` implementation** is selected:

1. On a Closure receiver, execute that Closure as currently specified by D013; do not perform `construct` lookup, change closure override/receiver binding, or preempt D207.
2. On a receiver eligible for present-day ordinary default construction (non-Closure ordinary object, excluding the existing special non-ordinary receiver domains), send the **ordinary message** `construct` to that same receiver with the exact already-evaluated positional/named packet. Its selected Closure is activated with normal receiver/home/lookup, signature, Error and control-transfer semantics.
3. On an ineligible receiver, keep the existing error-domain behavior; do not make primitive or foreign values constructible simply by installing the hook.

`Object` owns **one ordinary local standard `construct` Closure-valued slot**. The actual selected standard `Object.construct`, when invoked on an eligible ordinary receiver, creates one fresh ordinary object whose immutable immediate delegation parent is that receiver, sends `init` to the fresh child using the exact supplied packet, ignores `init`'s normal return and answers the child. An Error, non-local return, cancellation, or other transfer from `init` propagates; no successful child is returned. Standard inherited `Object.init` remains zero-argument and answers its receiver.

An overriding **custom `construct`** is an ordinary method: it may return any Protos value, including an object delegating from a different prototype, a cached identity or a scalar. Its exact normal result is the result of the original `receiver(args)` when standard `Object.call` chose it. **No automatic allocation or `init` happens before or after a custom `construct`.** The custom method must explicitly perform any construction, freezing or initialization it wants. Its argument names bind to its own real source signature. Do not invent an automatic compatible-type check or conditional `init` analogous to Python `__new__`.

The actual standard `Object.call` forwards the complete evaluated `(P,K)` packet transparently to the selected `construct`; the actual standard `Object.construct` likewise forwards transparently to the selected child `init`. Transparency is tied to **standard selected executable identity**, not the selector spelling or matching method home. It survives ordinary copying, aliasing, extraction and inheritance of the standard implementation; a custom user Closure or unrelated native factory does not become transparent merely by being named `call` or `construct`. Preserve D237 name association, collision/default/rest behavior, one-time source evaluation and the ordering of lookup failure versus argument-binding failure at both levels.

`super.construct(...)` is an ordinary super send, preserving the current invocation receiver while searching above `methodHome`. It must not recurse through `Object.call`. Explicit `object.construct(...)` is an ordinary method send, and reading `object.construct` extracts a bound Closure by the existing rule. This does not modify `Parent { ... }` literal construction, which neither sends `call` nor `construct` nor `init`.

## Mandatory source-library and Core effects

- Ordinary `Point` needs only `init` and retains `Point(0, 0)`.
- Canonical `Range` and `InvertedRange` move their directional factory from own `call` to own `construct`; canonical same-direction creation uses `super.construct`; opposite-direction results still have the opposite canonical prototype as immediate parent. Their own slot sets change from `init,call,each,eachStep,invert,reverse` to `init,construct,each,eachStep,invert,reverse`. Derived, noncanonical receivers still use default construction with their own `init`.
- Protos-source factory `call` slots in Core and Standard Library migrate to source-defined `construct` where semantically a factory. Genuinely executable `call` slots (including the constructed Test descriptor's executable behavior) are retained.
- Native standard Array, Map, IdentityMap, numeric conversion, TextReader/TextWriter and any other canonical specialized factory require an ordinary `construct` slot rather than a privileged second dispatch; keep native work only for host-owned value representation and the two transparent built-in forwarding primitives. Standard receiver-domain, Array indexed state, Map keyed state, object identity and errors remain independently normative.
- `%{...}` must still reject noncanonical factory selection **before** evaluating source entries: eligibility now accounts for the selected canonical ordinary `Object.call` bridge and canonical standard Map `construct`, not a now-removed own Map `call`. `[]` remains the ordinary Array invocation desugaring, with normal lexical shadowing and call lookup.
- No compatibility aliases that leave both a factory `call` and `construct` as two competing public construction surfaces. Deliberately preserve genuine ordinary custom-call precedence.

## Explicitly approved compatibility breaks

A pre-existing ordinary object having a local or inherited `construct` slot but previously inheriting `Object.call` changes behavior under `object(args)`: its `construct` is now invoked. A nearest non-Closure `construct` causes the normal selected-method invocation Error rather than silently falling back to allocation. The root `Object` gains a reflectable `construct`; `Range` and other migrated factories lose own `call` and gain own `construct`. `Range.call`, when read, now resolves the inherited standard bridge instead of the former bespoke source Closure. Reflective slot lists, method extraction, copied factories, and code that expected an old factory `call` closure identity change accordingly.

These are deliberate public semantic deltas and require explicit normative edits and regression updates; they must not be described as compatibility-preserving refactors. Arbitrary user-defined own `call` methods and Closure overrides remain fully ordinary.

## GITHUB021 invariant/delta reconciliation

| Existing owner-approved invariant | C1 disposition |
|---|---|
| D013: `receiver(...)` performs ordinary late `call` lookup, selected Closure activation, dynamic shadowing and D013 receiver/home semantics | **PRESERVED** |
| D013: standard `Object.call` itself allocates a child then sends `init` | **EXPLICITLY REOPENED**: the standard implementation instead delegates to ordinary `construct` |
| D013: no hidden callable kind, extra class hierarchy or second call universe | **PRESERVED** |
| D207: parenthesized Closure invocation remains a pending independent design question | **PRESERVED**: standard Closure path and local `call` overrides unchanged |
| D236: directional Range result identities and behavior | **PRESERVED** |
| D236: exact six own Range/InvertedRange slot names, including own `call` | **EXPLICITLY REOPENED**: replace own `call` with `construct`; preserve count and canonical frozen identities |
| D237: named packet forwarded through actual standard `Object.call` to initialization | **REFINED IN SELECTED C1**: transparent `Object.call` → selected `construct` → selected `init`; errors/argument effects preserved |
| Standard factories: native state domains, identity, arity, shallow semantics and Map literal canonical eligibility | **PRESERVED**, with selected factory method renamed and Map eligibility updated |
| Module/Actor: source module contexts are ordinary, Actor-local state and shared frozen Prelude remain immutable | **PRESERVED** |

C1's observable compatibility deltas were explicitly disclosed to the owner before “Apruebo C1”. There is **no approval** for a separate `[[Construct]]` mechanism, implicit await, a `factory:` parser construct, universal clone/copy semantics, hidden class flags, fallback on incompatible shadows, or altered Closure invocation.

## Implementation handoff and validation boundary

[I145 #1056](https://github.com/guillermomolina/protos/issues/1056) owns one maximal Protos-first substantive slice: normative `CALLABLES.md`/`VALUES_AND_COLLECTIONS.md`/grammar, ordinary `Object.construct` installation, native prepared/structured invocation and named forwarding, Core/Standard Library factory migration, conformance/regression coverage and guides. Update the informative `ABSTRACT_RUNTIME.md` sketch consistently. Do not prematurely materialize `pom.xml`, root `CHANGELOG.md` or global spec revision; under the human-executor policy, derive and write those only after required substantive validation is reported green, atomically with eventual product publication.

**Outstanding:** product edits, human-executed tests/builds, source/spec publication and post-publication evidence. Approval and this project-record commit alone do not claim an implemented feature.
