# AUD009-B4 — Execution context, name resolution, and binding complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#606`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence baseline: `98997d173ef482210dd726a900ec75d23f02ce7b`

Closure revalidation revision: `82cc94664be79a3aac121b0babb70cf82b1c4284`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Owner approval provenance: `guillermomolina/protos#606`, issue comment
`5738732327`, 2026-09-19.

## Purpose and boundary

AUD009-B4 reviewed the existing Core execution-context, lexical-parent,
unqualified-name, bare creation/assignment, evaluation-order, multiple-slot
creation, and Sequence model under the retrospective complexity/necessity
methodology approved for AUD009.

B4 does not reopen the B1 object/delegation/state outcomes, the B2
Closure/method/non-local-return outcomes, or the B3 Error/handler/cleanup
outcomes. Actor/P/Process transfer semantics remain AUD009-C; I/O remains D;
Standard Library breadth remains E; tooling remains F; and backend/AST/Bytecode
architecture remains G.

AUD009-A2 had already owner-approved `this` and `context` as KEEP. B4 found
no contradictory evidence requiring either classification to be reopened.

## Final classification ledger

```text
execution contexts as ordinary objects              KEEP
Context prototype / Context -> Object                KEEP
public standard Context binding                      KEEP
parameters/locals/module bindings as context slots   KEEP
separate lexical-parent relation                     KEEP
lexical traversal local-only                         KEEP

object-construction creation context                 KEEP
object omitted from method lexical capture           KEEP

this                                                  KEEP
context                                               KEEP

two-phase unqualified lookup                         KEEP
receiver/delegation fallback                         KEEP
captured contexts by reference                       KEEP

bare assignment nearest lexical local                KEEP
receiver-local assignment fallback                   KEEP
no delegated/implicit-creation assignment            KEEP
assignment destination selected before RHS           KEEP
assignment destination pinning                       KEEP

bare current-context creation                        KEEP
no lookup for bare creation                          KEEP
D143 fixed-prefix Array multiple creation            KEEP

strict left-to-right evaluation                      KEEP
successful write returns exact RHS                   KEEP
no rollback                                          KEEP

Sequence exact final result                          KEEP
empty Sequence -> canonical null                     KEEP
no runtime Sequence object requirement               KEEP
```

No B4 mechanism is classified for removal. No B4 Dxxx removal route is required.

## Execution contexts and Context

Protos deliberately avoids a second hidden local-variable storage universe.
Parameters, temporaries, Closure locals, module bindings and captured lexical
state are ordinary local slots of execution-context objects.

The standard visible topology is:

```text
activation/module/prelude context -> Context -> Object
```

`Context.protos` currently owns no local behavior, so its removal was the
strongest initial simplification candidate. The audit rejected removal because
real execution contexts remain observable ordinary objects. Removing Context
would require an anonymous parent, hidden execution-context category/flag, or
host environment object plus guest wrapper. None reduces the total model.

The public `Context` binding also names an otherwise observable ordinary parent
and keeps execution contexts within the same delegation universe as all other
objects.

Classification: **KEEP**.

Confidence: **HIGH**.

## Lexical-parent relation versus delegation

Execution contexts participate in two different semantic relations:

```text
ordinary delegation:
    context -> Context -> Object

lexical relation:
    current context -> enclosing/captured context -> ... -> prelude
```

Bare lexical search traverses local slots only through the lexical relation. It
does not traverse Context/Object delegation.

Collapsing the relations would either make inherited Object/Context behavior
participate in lexical name resolution or require another exception mechanism to
recreate the distinction indirectly. Encoding lexical ancestry as delegation
would also destroy the stable Context -> Object relation.

Classification: **KEEP**.

Confidence: **HIGH**.

## Object-construction context and method capture

While an object body executes, the object under construction is the current
slot-creation context. Methods created there do not capture that object as a
lexical parent; they capture the genuine enclosing lexical context.

This is required for prototype methods to remain receiver-polymorphic. If the
definition object were captured lexically, inherited methods could resolve bare
state against the definition object before the dynamic receiver.

The retained conformance case
`regression/unqualified-receiver-fallback-method-keeps-receiver.protos`
demonstrates the intended behavior.

Classification: **KEEP**.

Confidence: **HIGH**.

## `this` and `context`

A2's approved classifications remain:

```text
this     KEEP
context  KEEP
```

`this` is the explicit receiver-state authority.

`context` is the bridge exposing the current execution-context object directly.
Direct production guest use is modest, but removing it while retaining ordinary
object-backed lexical state would make the model selectively inaccessible and
would contradict the already-approved A2 decision without contrary evidence.

## Two-phase unqualified lookup

A bare read is:

```text
1. current context local slots
2. enclosing/captured lexical contexts, local slots only
3. receiver `this`
4. ordinary receiver delegation
```

The standard prelude participates only as a lexical context. There is no hidden
Process/Actor/global namespace fallback.

This composes lexical scope and ordinary object lookup without introducing a
third name-resolution mechanism.

Classification: **KEEP**.

Confidence: **HIGH**.

## Bare assignment

D002's model remains justified:

```text
nearest lexical context owning local x
else receiver itself if it owns local x
else SlotNotFound
```

Assignment never follows receiver delegation and never creates an absent
binding. That preserves B1's rule that writes do not silently mutate ancestors
and keeps `:` distinct from `=`.

The destination is selected before RHS evaluation and remains pinned across RHS
side effects. This prevents the RHS from redirecting the lvalue by creating a
nearer shadowing slot.

If no target exists, failure precedes RHS execution. After RHS completion,
ordinary mutation validation applies to the already-selected destination; no
fallback/retry occurs.

Classification: **KEEP**.

Confidence: **HIGH**.

## Bare creation and D143 multiple creation

A bare `x: value` performs no lookup. It creates only in the current
slot-creation context and may intentionally shadow outer lexical, prelude, or
receiver names.

D143's fixed-prefix Array multiple creation reuses this same institution:
one RHS evaluation, fixed-prefix shallow Array observation, ordinary
left-to-right current-context slot creation, no hidden activation, no rollback,
and exact original RHS as result.

B4 found no contradictory evidence requiring D143 to reopen.

Classification: **KEEP**.

Confidence: **HIGH**.

## Evaluation order and write result

Strict left-to-right evaluation remains one general Core rule.

Successful `:` and `=` return the exact RHS object. A post-write re-read is
not required and could itself introduce observable lookup/method-extraction
behavior.

Effects completed before a later failure are not implicitly rolled back.
Adding transactionality would be a much larger independent institution.

Classification: **KEEP**.

Confidence: **HIGH**.

## Sequence

D001 remains the smallest total normal-result rule:

```text
non-empty Sequence -> strict left-to-right, exact final-expression result
empty Sequence     -> canonical null
escaping transfer  -> remains an escaping transfer
```

No guest-visible runtime Sequence object is required.

Removing the empty-to-null rule would require another undefined/no-result
category merely to total otherwise-valid empty bodies.

Classification: **KEEP**.

Confidence: **HIGH**.

## Strongest attempted removals

The audit tried to falsify:

1. **public Context** — removal requires a hidden category/marker or anonymous
   ordinary parent and does not reduce the total model;
2. **separate lexical-parent relation** — collapsing it into delegation mixes
   inherited object behavior into lexical scope;
3. **receiver fallback for bare reads** — removal forces explicit `this.`
   everywhere without removing receiver/delegation lookup;
4. **distinct `:` and `=`** — collapsing them makes shadowing and mutation
   context-dependent;
5. **assignment target pinning** — resolving after RHS makes side effects able
   to redirect the lvalue;
6. **empty Sequence -> null** — removal requires a second absence/result
   institution.

None produces a smaller coherent Core model.

## AUD009-C handoff

B4 found one concurrency-boundary question that remains intentionally unresolved
here.

Current Actor/P/detached transfer implementation recognizes execution-context
state using the canonical standard `Context` topology; relevant paths reject an
ordinary object whose immediate parent is the canonical Context prototype.

The concurrency specifications state the portable exclusion in terms of
`ExecutionContext` values, while the object model permits any ordinary object to
use the public `Context` object as a delegation parent.

AUD009-C must therefore verify whether user-created direct Context children are
normatively execution contexts for transfer purposes or whether implementation
recognition is broader than the intended semantic boundary.

B4 makes no concurrency decision and does not treat this observation as evidence
against Context itself.

## Reconciliation boundaries

- A2 `this` / `context` KEEP outcomes remain unchanged.
- B1 object/delegation/write-locality outcomes remain unchanged.
- B2 lexical-capture/method/receiver outcomes remain unchanged.
- B3 Error/handler/cleanup outcomes remain unchanged.
- D001 Sequence semantics remain unchanged.
- D002 creation/assignment semantics remain unchanged.
- D143 multiple-slot creation remains unchanged.
- ExecutionContext transfer recognition remains AUD009-C.
- debugger/tooling projection remains tooling/platform work.
- duplicated AST/Bytecode lowering remains AUD009-G.

## Owner approval

The project owner explicitly approved all B4 classifications.

```text
ISSUE=guillermomolina/protos#606
APPROVAL_COMMENT=5738732327
DATE=2026-09-19
```

All approved outcomes retain existing semantics. B4 introduces no normative or
implementation delta.

## Closure checklist

```text
EXECUTION_CONTEXT_OBJECT_MODEL=KEEP
CONTEXT_PROTOTYPE=KEEP
CONTEXT_PUBLIC_BINDING=KEEP
CONTEXT_TO_OBJECT=KEEP
BINDINGS_AS_CONTEXT_SLOTS=KEEP
LEXICAL_PARENT_RELATION=KEEP
LEXICAL_TRAVERSAL_LOCAL_ONLY=KEEP

OBJECT_CONSTRUCTION_CREATION_CONTEXT=KEEP
OBJECT_NOT_METHOD_LEXICAL_CAPTURE=KEEP

THIS_INTRINSIC=KEEP
CONTEXT_INTRINSIC=KEEP

TWO_PHASE_UNQUALIFIED_LOOKUP=KEEP
RECEIVER_FALLBACK=KEEP
CAPTURED_CONTEXTS_BY_REFERENCE=KEEP

BARE_ASSIGNMENT_NEAREST_LEXICAL_LOCAL=KEEP
BARE_ASSIGNMENT_RECEIVER_LOCAL_FALLBACK=KEEP
NO_DELEGATED_ASSIGNMENT=KEEP
NO_IMPLICIT_CREATION_ASSIGNMENT=KEEP
ASSIGNMENT_DESTINATION_BEFORE_RHS=KEEP
ASSIGNMENT_DESTINATION_PINNING=KEEP

BARE_CURRENT_CONTEXT_CREATION=KEEP
NO_LOOKUP_FOR_BARE_CREATION=KEEP
D143_MULTIPLE_SLOT_CREATION=KEEP

STRICT_LEFT_TO_RIGHT_EVALUATION=KEEP
WRITE_RESULT_EXACT_RHS=KEEP
NO_IMPLICIT_ROLLBACK=KEEP

SEQUENCE_FINAL_RESULT=KEEP
EMPTY_SEQUENCE_NULL=KEEP
NO_RUNTIME_SEQUENCE_OBJECT_REQUIREMENT=KEEP

EVERY_SCOPED_MECHANISM_HAS_ONE_AUD009_CATEGORY=PASS
A2_BOUNDARY=PASS
B1_BOUNDARY=PASS
B2_BOUNDARY=PASS
B3_BOUNDARY=PASS
D001_RECONCILIATION=PASS
D002_RECONCILIATION=PASS
D143_RECONCILIATION=PASS
AUD009_C_HANDOFF_IDENTIFIED=PASS
OWNER_APPROVAL_PROVENANCE=PASS
REMOVAL_ROUTES=NOT_REQUIRED
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_B4_CLASSIFICATION=COMPLETE
```

AUD009-B4 is complete once this durable record and the required live GitHub
closure postconditions are verified.