# D237 — Ratified contextual named arguments, stable MIX-C binding, and R3 invocation boundaries

**Status:** RATIFIED LANGUAGE DESIGN — owner approved 2026-11-02; durable record is **non-normative**, and product `spec/` publication and implementation remain the independent responsibility of [I143 #1039](https://github.com/guillermomolina/protos/issues/1039).
**Design authority:** [D237 #1038](https://github.com/guillermomolina/protos/issues/1038).
**Exact product revision inspected during the investigation:** [`guillermomolina/protos@2e85e8dc0b85548add6962ef6556f24513bc8d05`](https://github.com/guillermomolina/protos/commit/2e85e8dc0b85548add6962ef6556f24513bc8d05). This is a **preimplementation baseline**, not a tested D237 product revision.
**Approval record sequence:** [syntax + MIX-C checkpoint](../../work/D237/D237_NAMED_ARGUMENTS_MIX_C_OWNER_APPROVAL_2026_11_02.md); [R3 exhaustive research and falsification](../../work/D237/D237_REMAINING_SEMANTICS_FALSIFICATION_PACKET_2026_11_02.md); [syntax approval](https://github.com/guillermomolina/protos/issues/1038#issuecomment-6578019068); [MIX-C approval](https://github.com/guillermomolina/protos/issues/1038#issuecomment-6578425191); [R3 proposed candidate](https://github.com/guillermomolina/protos/issues/1038#issuecomment-6578722646).

## 1. Owner approval and exact scope

The project owner selected the contextual `f(a: value)` interpretation after the slot-creation compatibility delta was disclosed: **“si lo acepto, de hecho creo que tiene mas sentido el significado que le queremos dar”**. Subsequently the owner approved the presented fixed-position/any-order mixing candidate **D237-MIX-C**: **“Perfecto, apruebo el contrato”**.

After a further exhaustive review of **D237-REMAINDER-R3**, including the seven numbered rules explicitly proposed for D130 Array, contextual constructs, actual standard `Object.call` forwarding, declared native signatures, errors, interop and API compatibility, the project owner stated, verbatim:

> **“Apruebo D237-REMAINDER-R3 con las siete reglas anteriores.”**

This is **exact-candidate approval of R3** in addition to the already approved syntax and MIX-C contracts; it does not approve additional behavior not present in those contracts. The three approved checkpoints together close the D237 design space. The historical work/research documents remain genuine historical evidence and are not retroactively rewritten from proposal to decision.

## 2. Ratified syntax and deliberate compatibility change

- In an ordinary parenthesized or Array-literal argument-item list, a **bare leading `identifier: expression`** is a named argument: the identifier is a literal Unicode 17 NFC parameter label, not a lexical name lookup or slot-creation operation. Parsing is purely structural; it must not examine the current target, dynamic `call` or inferred signature.
- A grouped `f((a: 10))` remains **one positional argument**, evaluating the ordinary grouped slot creation. Plain `a: 10` outside the argument-label position is unchanged; `(a: 10) => body` remains the Closure default-parameter form. Member-slot creation, e.g. `f(object.a: 10)`, is not a bare-name label.
- **Source compatibility is knowingly broken:** the old `f(a:10)` that created a slot and passed the value instead becomes a named call; parenthesize the inner slot-creation expression to migrate. The identical compatibility delta applies to the D130 Array literal surface `[a:10]`, with `[(a:10)]` as its positional slot-creation escape.
- The exact established structural recognition of `if`, `while`, numeric/generic `for`, postfix `until` and contextual `return` takes precedence when the source matches the **already-ratified contextual form**; the condition/operand/header expressions keep ordinary `:` slot-creation semantics. A grouped callee, e.g. `(if)(flag: true) { body }` or `(return)(value: 10)`, uses ordinary named-call rules instead. No new reserved word or blanket special case for methods named after contextual constructs.

## 3. Ratified MIX-C argument association

For an ordinary name-bearing call, evaluate the receiver/target and each supplied argument-item expression **once in source order** before resolving the ultimate selected `call` or message; spread extraction uses existing standard Array semantics at its textual position. The values form a conceptual positional vector `P` and a labeled sequence `K`. Names do **not** affect evaluation order.

**Stable positional ordinals:** the nth positional value after expanding source spreads targets the nth non-rest formal parameter regardless of where named arguments appear. A named item targets the one matching non-rest formal's declared name. Positional and named items may appear in *any textual order*, but never shift a positional value to the next unoccupied parameter. `f(a: 9, 1)` is a double-binding error on the first parameter, not a positional binding of `b`.

Repeated explicitly written labels in the **same argument list** are a source/parse error. Unknown labels, labels equal to a rest-formal name, and collisions of named values with positional ones (including post-spread collisions) signal an error at dynamic association time. Unknown labels are not silently ignored. A Closure's parameter binding remains in declaration order and evaluates defaults only for **absent** formal positions; supplied `null` suppresses its default. No hoisting of later formals, second Closure kind or new context is introduced. Standard D203 extra positional arguments are still evaluated then ignored in the absence of rest; a trailing `...rest` captures **only** excess positional values as a fresh frozen Array.

Existing `...expression` spread remains **positional only**. No keyword spread, named-rest capture, ambient `args`/full-arguments intrinsic, or implicit Map/object-to-keywords conversion is introduced. An attached trailing `{ body }` Closure remains exactly one **last positional argument**; no inference from names or callable types, even if that causes a named/positional collision.

## 4. Ratified dynamic selection and standard Object.call bridge (R3)

D013's ordinary overridable `call` protocol remains definitive: after all argument expression and spread effects, perform ordinary lookup of the selected `call` Closure (or the actual member/`super` method) with the existing receiver, `this`, `methodHome`, extracted-method, return-home and inheritance semantics. There is no static argument-to-position lowering that assumes a stable callee and no privileged Closure-identity bypass of a custom `call`.

For an ordinary source-defined Closure with parameter names, bind `P,K` against **that dynamically selected target Closure's declared formals**. **Exception that preserves existing standard semantics:** the **actual executable standard `Object.call` implementation** transparently forwards its received `P,K` packet rather than binding names to a speculative native wrapper signature:

- When its invocation receiver is a Closure, activate that **exact Closure** once with the same `P,K`, preserving captured receiver, `methodHome`, lexical scope, return home and control/error behavior.
- Otherwise, under standard object-construction receiver validation, create the normal fresh child and send the *selected ordinary* `init` method the same `P,K`; `init` return and failure behavior remain unchanged.
- This forwarding belongs to the **actual standard executable behavior**, including if its Closure was copied, inherited, composed, aliased or extracted with ordinary receiver-binding metadata. `methodHome === Object` is **not** required for correctness; recognizing the canonical root selection is an optional guarded optimization only. Copies/extractions do not acquire other special powers.
- An arbitrary custom `call` remains a normally bound executable and **does not automatically forward labels** it did not declare. A source-defined `call: (...args)` is positional-rest only, even when a user conceptually describes its values as `start` or `stop`. User-defined generic keyword forwarding requires future separate language design, not a hidden extension of D145.

## 5. Ratified parameter signatures, foreign/host boundary and errors

Named labels are meaningful **only against semantically declared non-rest formal names** of the actual target. Source Closures have those declarations. Native/standard/projected Closure behaviors accept names only where they **explicitly publish compatible Protos-semantic signatures**; do not infer names from local Java variables, foreign-language method signatures, documentation, selector names, or `...args`. The standard `Object.call` transparent bridge is the sole already-selected special forwarding seam, not a general native-signature inference rule. There is no new automatically guest-visible Closure signature-reflection API.

A duplicate explicit label is a **parse failure**; neither supplied expression executes. For source-valid calls, a caller argument evaluation/spread/receiver error or non-local transfer precedes selection; a missing or non-Closure selected behavior fails before name validation. Unknown labels and positional/named collisions are then validated **before executing any default or body of the actual target**. Each binding-validation failure is a **fresh generic Protos `Error`** with ordinary Error ancestry and existing non-resumable handler behavior. No new required public error prototype or payload/message is introduced. Earlier caller effects are not rolled back. Standard `Object.call` forwards the packet without prevalidating against its wrapper signature; a failed selected constructor `init` lookup wins before its own name validation. Defaults run later in their actual activation, preserving prior slot/return-home/suspension/failure precedence.

**Interop boundary:** foreign projected `call` adapters, `std:interop.invoke(target, ...arguments)`/`instantiate(target, ...arguments)`, Polyglot/host positional execution and synchronous foreign callback ingress remain positional unless a separate explicitly defined Protos-facing named signature enables name association. There is **no automatic translation** of labels into Python/Ruby/C#/other foreign keywords. A Protos name-validation error before entering the foreign operation is a fresh ordinary `Error`, not `ForeignError`; failures produced after entered foreign operation retain their existing `ForeignError`/unchanged-control rules. Actor/P isolation, resource authority, task/return-home lifetime and transfer semantics remain unchanged.

**Compatibility consequence:** a publicly callable Closure's declared parameter names are now part of its source-call API: changing a formal's name may break existing named callers even when positional invocations remain valid. That is an intentional consequence of name-aware binding, not an implementation detail.

## 6. Observational examples (future behavior, not executed)

~~~protos
f: (a: 10, b: 20, c: 30) => [a, b, c]
f(c: 99)                 // [10, 20, 99]
f(c: 99, 1)              // [1, 20, 99]
f(1, c: 99)              // [1, 20, 99]
f(...[1, 2], c: 99)      // [1, 2, 99]
f(a: 99, 1)              // Error: a gets both a named and positional value
f(unknown: 1)            // Error: name is not declared
f(a: 1, a: 2)            // source error: duplicate explicit label

f((a: 10))               // positional result of ordinary slot creation
[(a: 10)]                // positional Array literal with slot creation
[a: 10]                  // same named argument as Array(a: 10): standard Array rejects
if(flag: true) { body }  // contextual if: condition expression can create slot flag
(if)(flag: true) { body }// ordinary call with named flag and trailing positional closure
~~~

`Array` shadowing still applies to `[a: 10]` exactly as to `Array(a: 10)`, so a program-supplied factory that declares formal `a` may accept the label. Standard `Array.call` is currently a native positional factory and does not automatically accept `a`. A source-based `Range.call: (...args)` similarly does not acquire named `start,stop` by magic. `Point(x: 10, y: 20)` using the *standard* inherited `Object.call` forwards `x,y` to the actual `init` and works only where that `init` declares those names. The example `(if)(flag: true) { body }` has a final positional Closure that may collide or fail according to the actual lexical `if` signature.

## 7. GITHUB021 invariant/delta reconciliation and approval scope

| Previous authority/owner-approved invariant | R3 outcome and disclosed delta |
| --- | --- |
| D237 owner's `f(name:value)` contextual selection and `f((name:value))` escape | **PRESERVED**; explicit source migration, no optional `.name:` marker |
| D237-MIX-C, fixed positional ordinals and free textual interleaving | **PRESERVED**; collisions instead of hole filling, trailing Closure strictly positional |
| D203 flexible positional arity/defaults/null/rest/spread | **PRESERVED**, only explicit name association and failures added |
| D013 ordinary `call` selection and extraction, overridden/copyable behavior | **PRESERVED** by forwarding through executable standard Object.call even when not root-owned |
| D130 exact `[...]` ↔ lexical `Array(...)` desugaring | **PRESERVED**, **EXPANDED SOURCE BREAK** for `[a:10]`, explicitly disclosed and approved by R3 rule 1 |
| D210/D211/D212/D231/D234 contextual shapes | **PRESERVED**; exact contextual recognition outranks ordinary argument-label parsing, including suffix `until` |
| D145 no ambient `args` and positional `rest` | **PRESERVED**; new generic name forwarding **NOT** introduced |
| ERRORS generic fresh `Error` taxonomy/handlers | **PRESERVED**; binding validation uses fresh generic `Error` |
| Foreign Values projected call/std:interop and callback positional rules | **PRESERVED**; no implicit foreign keyword bridge |
| PLAT060-B5 five-header-slot positional ABI | **PRESERVED as positional-only implementation invariant**; optional private name-bearing representation, no mandatory new guest object |
| D207 direct-Closure invocation reassessment still separately tracked | **NOT PREEMPTED**, no hidden alternative to ordinary D013 dispatch |

**GITHUB021 invariant check: PASS** for the exact selected syntax + MIX-C + R3 packet. The explicitly surfaced D130 source-compatibility extension and standard `Object.call` behavior-instance forwarding were **approved** with the seven-rule R3 candidate. No other approved design invariant was silently overturned. Detailed alternatives, 12-axis comparative scores/confidence, adversarial cases, future-scalability and pay-as-you-grow evidence are retained in the linked work packet (not repeated or replaced here).

## 8. Implementation and publication separation

This decision record closes the **design** and unblocks the distinct **I143** implementation scope. I143 must update normative owners (`spec/PROTOS_GRAMMAR.md`, `spec/semantics/CALLABLES.md`, values/Array, error/interop/execution cross-references and specification changelog), parser/surface AST, canonicalization, source tools, runtime standard-call bridge, dynamic binding, applicable Protos-first libraries and conformance before claiming product support. It should preserve the existing PLAT060-B5 **positional** path unless measurements demonstrate a needed change; not rewrite the core ABI on speculation.

Conformance must falsify late `call` rebinding, mixed order/effects, post-spread collisions, copied/extracted standard Object.call, custom call override, member and `super` calls, constructor/`init`, source/native signature, bracket shadowing, contextual five families, explicit-null/default rest and frozen Array behavior, Error identity and precedence, host/foreign positional boundaries, source formatter/LSP and relevant Actor/P control behavior. No new mandatory guest Map, keyword-spread, automatic foreign named transport or reflective signature API is authorized.

**No product files were edited as part of D237 ratification.** The project owner previously reported `git diff --check` clean and local `make test` and `make check` PASS; these were **pre-I143** reports with no exact tested local SHA, so they do not validate D237 implementation. No named-argument conformance, Graal compilation or benchmark PASS is claimed. Product commit/push, version bump and final test evidence belong to human-executed I143 slices.

```text
FORMAL_IDENTIFIER=D237
SELECTED_DESIGN=D237_CONTEXTUAL_NAMED_ARGUMENTS+D237_MIX_C+D237_REMAINDER_R3
OWNER_APPROVAL=EXPLICIT_2026_11_02
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
DESIGN_STATE=RATIFIED
PRODUCT_BASELINE_INSPECTED=2e85e8dc0b85548add6962ef6556f24513bc8d05
PRODUCT_NORMATIVE_DELTA=NOT_YET_PUBLISHED
PRODUCT_IMPLEMENTATION=NOT_STARTED
I143_STATUS=READY_AFTER_COORDINATION
NAMED_ARGUMENT_TESTS=NOT_RUN
```
