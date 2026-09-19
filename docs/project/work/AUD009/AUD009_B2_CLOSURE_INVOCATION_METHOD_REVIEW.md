# AUD009-B2 — Closure, invocation, and method semantics complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#603`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence / closure revision: `98997d173ef482210dd726a900ec75d23f02ce7b`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Owner approval provenance: `guillermomolina/protos#603`, issue comment
`5738596812`, 2026-09-19.

## Purpose and boundary

AUD009-B2 reviewed the existing Closure, invocation, receiver binding, extracted
method, `super`, non-local-return, and polymorphic-`call` semantics under the
retrospective complexity/necessity methodology approved for AUD009.

B2 does not reopen source syntax already classified by AUD009-A2, object
topology/state/reflection closed by AUD009-B1, concurrency/execution architecture
owned by AUD009-C, or AST/Bytecode/runtime architecture owned by AUD009-G.

## Final classification ledger

```text
Closure as sole executable value kind              KEEP
lexical capture by reference                       KEEP
captured receiver                                  KEEP

method-as-Closure invocation role                  KEEP
receiver-bound extracted Closures                  KEEP
fresh identity per extraction                      KEEP
re-binding on new member read                      KEEP
methodHome                                         KEEP
super continuation from parent(methodHome)         KEEP

non-local return                                   KEEP
ReturnHome active/completed model                  KEEP
escaped non-local return -> InvalidReturn           KEEP

ordinary call-slot invocation                      KEEP
polymorphic callable ordinary objects              KEEP
callability via read-only call lookup              KEEP
Object.call Closure execution                      KEEP
Object.call default construction + init             KEEP
standard factory call specialization               KEEP
```

No B2 mechanism is classified for removal. No B2 Dxxx route is required.

## Closure as the single executable Core value kind

Core has one executable language value kind: `Closure`. A method is an
invocation role of a Closure-valued slot rather than a second value kind.

This keeps lexical capture, identity, invocation, argument binding, receiver
binding and returns in one semantic family. Introducing Function/Method/Callable
value categories would enlarge the language rather than remove duplication.

Closures capture genuine lexical execution contexts by reference. Production
Standard Library and Tool code relies heavily on nested callbacks that observe
and mutate captured bindings.

Classification: **KEEP**.

Confidence: **HIGH**.

## Method role and receiver binding

Ordinary message lookup selecting a Closure-valued slot invokes that Closure in
method role. The original receiver becomes `this`, even when the selected slot
belongs to an ancestor.

This follows the ordinary B1 delegation model and avoids a separate method table
or method declaration category.

Classification: **KEEP**.

Confidence: **HIGH**.

## Receiver-bound extracted Closures

Reading a Closure-valued member as a value must preserve enough information for
later invocation to behave like the original method reference.

The resulting value remains a Closure and carries:

```text
executable semantics = selected stored Closure
bound receiver       = original read receiver
methodHome           = selected slot owner
```

The retained conformance case
`protos/tests/conformance/maturity/object-model/deep-extracted-super-chain-mutation.protos`
shows extracted and nested Closures preserving the dynamic receiver and correct
`super` origin while receiver/ancestor state changes afterwards.

Classification: **KEEP**.

Confidence: **HIGH**.

## Fresh identity per extraction

Repeated successful member reads that select the same stored Closure produce
fresh Closure identities.

B2 tested the apparent simplifications:

- returning the stored Closure loses receiver/`methodHome` binding;
- mutating/rebinding the stored Closure makes aliases and different receivers
  overwrite one another;
- canonicalization requires a new identity cache/lifetime institution;
- a distinct Method wrapper introduces another executable value kind.

Fresh extraction is the smallest stateless rule. Immediate invocation need not
materialize an observable extracted Closure, so the cost is pay-for-use.

Classification: **KEEP**.

Confidence: **HIGH**.

## `methodHome` and `super`

Correct `super` semantics require two independent facts:

```text
receiver     = original dynamic receiver
lookup start = parent(methodHome)
```

Using the receiver's parent is incorrect for deep inherited invocation.
`methodHome` therefore represents an observable semantic requirement even
though it is intentionally not exposed by guest reflection.

Nested and extracted Closure conformance covers preservation of this origin.

Classification: **KEEP**.

Confidence: **HIGH**.

## Non-local return and ReturnHome

The A2 KEEP outcome for `^` survives the semantic audit.

This is real Standard Library capability:

- `std:collections/Set` uses `^false` inside nested traversal callbacks;
- `std:collections/IdentitySet` uses the same pattern;
- `std:collections/Array.findIndex` uses `^index` from inside `each`;
- `std:collections/Array.reduce` uses `^null` for the empty case.

The home-activation model lets nested callbacks exit the owning invocation
without requiring every intermediate higher-order operation to cooperate.

ReturnHome has a deliberately small observable state model:

```text
active
completed
```

A nested Closure may retain the owning home. A later escaped non-local return to
a completed home signals `InvalidReturn` rather than being retargeted.

Classification: **KEEP**.

Confidence: **HIGH**.

## Ordinary-slot polymorphic invocation

D013's ordinary-slot invocation model remains justified.

Parenthesized invocation performs ordinary lookup of `call`, requires the
selected value to be a Closure, and invokes it in method role with the original
target as `this` and selected slot owner as `methodHome`.

Production guest use includes:

- `std:collections/Set.call`;
- `std:collections/IdentitySet.call`;
- `std:test/Test.call`;
- ordinary callback objects accepted by collection/control APIs.

A nearer non-Closure `call` shadows an inherited valid one exactly like any
other slot. Alternatives such as a callable bit, registry, hierarchy, or
Closure-only special dispatch would introduce a second invocation authority.

Classification: **KEEP**.

Confidence: **HIGH**.

## Structural callability inspection

Where an API validates a candidate without invoking it, the rule remains:

```text
ordinary lookup of candidate.call succeeds
and selected slot value is a Closure
```

This does not execute user code, evaluate defaults, preflight arity, suspend, or
pin the selected Closure for later use.

Classification: **KEEP**.

Confidence: **HIGH**.

## Standard `Object.call`

The inherited root behavior has two branches:

```text
semantic Closure receiver
    -> execute that Closure

otherwise
    -> create fresh child with parent = receiver
    -> send child.init(args...)
    -> return child
```

The Closure branch avoids a separate standard Closure prototype, hidden callable
bit, or second invocation rule.

The construction branch lets any object serve as a prototype/factory without a
Class/Prototype kind. Initialization remains ordinary and overridable through
`init`, while successful construction always returns the fresh child.

Standard factories specialize the same model by owning nearer ordinary `call`
slots.

Splitting these branches would not remove ordinary invocation machinery and
would require another Closure execution authority while losing the uniform
prototype-construction rule.

Classification: **KEEP**.

Confidence: **MEDIUM-HIGH** for the exact root unification; **HIGH** for
ordinary polymorphic `call`.

## A2 argument-establishment reconciliation

B2 found no contradictory evidence requiring reopening:

```text
required parameters       KEEP
default parameters        KEEP
rest parameters           KEEP
call spread               KEEP
ambient args intrinsic    REMOVED by D145/I042
```

The caller-supplied vector remains legitimate activation/binding state without
being ambient guest surface.

## Runtime-complexity boundary

Current implementation carries receiver, `methodHome`, ReturnHome, captured
lexical contexts, and selected Closure state through synchronous, Bytecode,
C-prime, Task, and related paths.

B2 does not interpret duplicated backend transport as evidence against semantic
mechanisms with observable purpose. Runtime duplication belongs to AUD009-C/G.

## Strongest attempted removals

### Fresh extracted-Closure identity

Removing it requires lost receiver binding, mutable rebinding, identity-cache
machinery, or a distinct Method kind.

Result: **KEEP**.

### `methodHome`

Removing it breaks deep `super` while preserving the original dynamic receiver.

Result: **KEEP**.

### ReturnHome / non-local return

Real Standard Library code uses it to exit nested callbacks without
callback-specific propagation protocols.

Result: **KEEP**.

### Root `Object.call` dual role

Splitting it requires another Closure invocation authority/prototype/category
while discarding the no-Class default-construction rule.

Result: **KEEP**.

## Reconciliation boundaries

- AUD009-B1 object topology/state/reflection remains unchanged.
- AUD009-A2 syntax/operator/call-ergonomics outcomes remain unchanged.
- D013 remains consistent with the evidence.
- Future/Task/Actor/parallel execution remains AUD009-C.
- AST/Bytecode/backend duplication remains AUD009-G.
- Error/handler/ensure general control remains for a later bounded AUD009-B
  slice.
- missing callable ergonomics remain AUD011.

## Owner approval

The project owner explicitly approved the complete B2 classification packet.

```text
ISSUE=guillermomolina/protos#603
APPROVAL_COMMENT=5738596812
DATE=2026-09-19
```

All approved outcomes retain existing semantics, so no normative or
implementation change follows from B2 itself.

## Closure checklist

```text
SCOPED_CALLABLE_INVENTORY=COMPLETE
CLOSURE_SINGLE_VALUE_KIND=KEEP
LEXICAL_CAPTURE_BY_REFERENCE=KEEP
METHOD_ROLE_AND_RECEIVER_BINDING=KEEP
EXTRACTED_CLOSURE=KEEP
FRESH_EXTRACTION_IDENTITY=KEEP
METHOD_HOME_SUPER=KEEP
NON_LOCAL_RETURN=KEEP
RETURN_HOME=KEEP
POLYMORPHIC_CALL=KEEP
OBJECT_CALL_CLOSURE_BRANCH=KEEP
OBJECT_CALL_CONSTRUCTION_BRANCH=KEEP
STANDARD_FACTORY_SPECIALIZATION=KEEP
EVERY_SCOPED_MECHANISM_HAS_ONE_AUD009_CATEGORY=PASS
A2_BOUNDARY=PASS
B1_BOUNDARY=PASS
AUD009_C_BOUNDARY=PASS
AUD009_G_BOUNDARY=PASS
OWNER_APPROVAL_PROVENANCE=PASS
REMOVAL_ROUTES=NOT_REQUIRED
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_B2_CLASSIFICATION=COMPLETE
```

AUD009-B2 is complete once this durable record and the required live GitHub
closure postconditions are verified. The parent AUD009 audit should then advance
to the next bounded Core-semantics slice.
