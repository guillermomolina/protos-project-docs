# PLAT036 — Bytecode DSL lexical-state representation boundary

Status: `RATIFIED`

Selected architecture: **Candidate D — frame-backed semantic-context adapter**.

Approval: explicit project-owner approval on 2026-09-24:

~~~text
Apruebo Candidate D para PLAT036; la recomendación anterior E1 queda superseded.
~~~

Decision Issue: `guillermomolina/protos#702`

Product baseline:

~~~text
PROTOS_REVISION=75231601e458930684d4b619dd5f4722377aa65d
GRAALVM_TRUFFLE_VERSION=25.3.4.1
D179_SELECTED_CANDIDATE=C3_MONOTONIC_CONTEXT_MEMBERSHIP
D179_PROJECT_RECORD_REVISION=9a7648181b70733fc0bb97940cd1e25482b38fd6
I067_PROJECT_RECORD_REVISION=e17d2554d574d85d009133b183a25844ba769700
PLAT036_PRIOR_WORK_RECORD_REVISION=1c3b19973d216f6fea9108f63eb4c0bd4e6ed892
PRIOR_E1_RECOMMENDATION=SUPERSEDED
~~~

Nature: durable non-normative JVM / Truffle / Bytecode DSL architecture decision.
Observable Protos semantics remain owned by the normative specification and by
ratified language decisions such as D179.

## Decision

For statically admitted Protos lexical bindings, the Truffle Bytecode DSL
frame/local representation is the authoritative value storage.

The first-class Protos execution-context object remains a real semantic object
with stable identity, reflection, mutation, capture and escape semantics. It
projects the authoritative frame-backed bindings and a separate dynamic overflow;
it does not maintain a second authoritative copy of the same binding value.

The architecture is:

~~~text
ProtosExecutionContextValue
  = stable semantic identity
  + context/object reflection semantics
  + OPEN/CLOSED/FROZEN state
  + semantic slot-name projection/order
  + dynamic overflow for names outside the admitted static layout
  + adapter/reference to authoritative frame-backed lexical state

statically admitted lexical binding
  -> Bytecode DSL local / frame-backed authority

genuinely dynamic-only binding
  -> dynamic overflow authority
~~~

Hard invariant:

~~~text
ONE_SEMANTIC_BINDING_VALUE_AUTHORITY_REQUIRED=YES

FOR_THE_SAME_BINDING:
  frame value authoritative
  AND
  independent context-object value authoritative
= FORBIDDEN
~~~

Candidate E1 — a backend-neutral indexed semantic-context value store using
`values[] + present[]` as the primary lexical authority — was a serious prior
recommendation but is superseded by this decision.

## Why Candidate D is selected

The post-I067 reevaluation established that no current Protos semantic invariant
requires lexical value authority to live outside Truffle frame/local state merely
because `context` is a first-class object.

The pinned Truffle 25.3.4.1 line already provides the relevant mechanisms:

- `BytecodeLocal`;
- `LocalAccessor`;
- `MaterializedLocalAccessor`;
- `enableMaterializedLocalAccesses`;
- local name/info metadata;
- explicit local clear/presence support;
- continuation/yield frame preservation; and
- captured/escaping Bytecode frame support.

The comparative implementation survey found a recurring architecture across
current Truffle implementations: statically known lexical identity is represented
as indexed/frame-native state, while reflection, reification or genuinely dynamic
name behavior uses an explicit adapter or separate dynamic mechanism.

Particularly relevant precedents include:

- TruffleSqueak: a real first-class Smalltalk Context object coexists with and
  may retain/materialize a Truffle frame;
- TruffleRuby: a first-class Binding captures materialized frame state and may
  extend dynamic binding state without converting all ordinary locals into a
  separate semantic value array;
- TruffleSOM: non-local lexical access is represented by context level plus slot
  index;
- GraalJS: lexical frames/block scopes and closure captures use frame-native
  representations and materialized parent scopes;
- GraalPy: guest-visible frame objects are materialized/projected from execution
  frame state when required;
- Apple Pkl: lexical reads carry slot plus lexical depth;
- SimpleLanguage: statically known locals lower directly to `BytecodeLocal`.

This evidence does not make Truffle implementation precedent semantic authority.
It establishes that Candidate D uses the substrate in the direction for which it
is designed while preserving Protos semantics behind an explicit adapter.

## Preserved semantic invariants

PLAT036 changes representation, not language semantics.

The selected architecture must preserve:

~~~text
FRESH_EXECUTION_CONTEXT_PER_INVOCATION=YES
CONTEXT_IS_FIRST_CLASS_PROTOS_OBJECT=YES
CONTEXT_IDENTITY_STABLE=YES
CONTEXT_ESCAPE=YES
CONTEXT_REFLECTION=YES
ABSENT_TO_PRESENT_LATE_CREATION=YES_WHILE_OPEN
PRESENT_TO_PRESENT_MUTATION=YES_WHILE_WRITABLE
PRESENT_TO_ABSENT_REMOVAL=NO
PRESENT_NULL_DISTINCT_FROM_ABSENT=YES
CAPTURE_BY_REFERENCE=YES
LATER_MUTATION_VISIBLE_TO_CAPTURED_CLOSURE=YES
LATER_LEGAL_CREATION_CAN_RETARGET_LOOKUP=YES
SEQUENTIAL_PARAMETER_DEFAULT_VISIBILITY=PRESERVED
ASSIGNMENT_DESTINATION_SELECTED_BEFORE_RHS=YES
WRITES_NEVER_DELEGATE=YES
RECEIVER_AND_METHOD_HOME_SEMANTICS=PRESERVED
NON_LOCAL_RETURN_AND_DYNAMIC_CONTROL=PRESERVED
SUSPENSION_RESUMPTION=PRESERVED
DEBUGGER_SOURCE_TAG_SEMANTICS=PRESERVED
~~~

D179 C3 is therefore a fixed prerequisite, not something PLAT036 reinterprets.

## Presence model

Static physical identity does not imply semantic presence.

A source-known binding may have an allocated local identity before it is
semantically PRESENT. Candidate D requires a distinct presence state so that:

~~~text
PRESENT(null) != ABSENT
~~~

The implementation may use the Bytecode DSL local cleared/uninitialized state or
an equivalent frame-native mechanism, provided one authoritative value state is
maintained and the semantic distinction is exact.

Sequential/default parameter binding must therefore establish presence only at
the existing semantic binding point.

## Late creation and lexical retargeting

D179 C3 still permits a nearer execution context to acquire a previously ABSENT
binding while OPEN.

Therefore a statically identified outer binding cannot always be loaded
unconditionally. Where a nearer same-name candidate may legally become PRESENT,
the lowering must preserve correctness through one or more of:

- explicit nearer-presence tests;
- a valid topology/presence assumption invalidated by the first relevant
  `ABSENT -> PRESENT` transition; or
- exact dynamic fallback.

C3 removes only the reverse `PRESENT -> ABSENT` transition.

For genuinely dynamic names not represented by a static layout, dynamic overflow
and exact name-based fallback remain part of the architecture.

## Assignment destination

The existing ordering remains authoritative:

~~~text
destination = resolve lexical write target
evaluate RHS
write to the previously resolved target
~~~

The implementation may represent a resolved static destination through
frame/local identity and a resolved dynamic destination through context/name
identity. It must never re-resolve the destination after RHS evaluation.

## Closure capture

Where binding identity and lexical depth are statically proven, captured lexical
reads and writes should use the Bytecode DSL materialized-local/frame mechanisms
rather than String lookup through copied semantic stores.

Capture remains by reference. Existing and later legal mutations must be observed
through the same authoritative frame-backed binding.

Late nearer creation remains subject to the presence/topology rule above.

## First-class context reflection and escape

The semantic execution-context object remains the guest-visible authority for
identity and object behavior, but not a duplicate lexical value store.

Reflection must project exactly the guest-visible state:

~~~text
context.hasSlot(name)
context.slotValue(name)
context.slotNames()
context mutation APIs
debugger scope reads/writes
~~~

from:

~~~text
authoritative frame-backed static bindings
+
dynamic overflow
+
semantic mutation state
~~~

Backend-private Bytecode locals/temporaries must remain invisible.

An escaped context may retain or reach the materialized/captured frame state
needed to preserve its semantics. The exact Java carrier — for example the most
appropriate combination of `MaterializedFrame`, `BytecodeFrame`, accessors and
adapter state — is an implementation choice to prove in bounded slices.

No implementation may satisfy escape by copying frame values into an independent
authoritative semantic store and then allowing both copies to diverge.

## Lazy physical context creation remains out of scope

PLAT036 does **not** ratify lazy physical creation of the semantic
`ProtosExecutionContextValue`.

The selected architecture must remain correct if every invocation continues to
allocate/retain its stable semantic context identity eagerly.

Any future proposal to make physical context-object creation lazy remains a
separate PLAT decision as required by AUD016.

## Debugger and tooling

PLAT015 remains authoritative.

Debugger scope projection must expose the semantic lexical scope, not the
physical Bytecode frame layout. It must:

- include only guest-visible bindings that are semantically PRESENT;
- include dynamic overflow names;
- preserve semantic name/order rules;
- hide backend temporaries;
- direct writes to the same authoritative binding state; and
- preserve source/tag behavior already owned by PLAT026 and PLAT034.

## Suspension, unwind and control

PLAT014, PLAT019 and PLAT021 remain unchanged.

A suspended Bytecode continuation already retains the execution frame state.
Candidate D places lexical authority in the state that suspension preserves
rather than creating an independent lexical heap-state copy.

Suspension itself remains distinct from semantic unwind. Error, cancellation,
ensure and non-local return continue to use their already-ratified control lanes
and precedence rules.

## Polyglot Context and implementation portability

The selected architecture is intentionally JVM/Truffle specific.

A future non-Truffle implementation may use a different lexical-authority
substrate while preserving the same Protos semantic context abstraction.

The architecture must keep Truffle Java types behind the backend boundary and
must not expose frame index, storage class, `BytecodeLocal`,
`MaterializedLocalAccessor` or materialization policy as Protos semantics.

Current Protos requirements do not establish live lexical-state migration across
independent Polyglot Contexts as a language semantic requirement.

## Runtime/performance boundary

This decision is architectural, not a PERF010 attribution result.

~~~text
ARCHITECTURAL_ADOPTION_DECISION != PERFORMANCE_ATTRIBUTION_RESULT
~~~

Candidate D structurally exposes lexical identity through the compiler-visible
frame/local mechanisms supplied by Bytecode DSL. That does not establish an
end-to-end speedup, an attributable fraction, or PERF010 dominance.

PERF010-A / #691 and PERF011 / #693 retain their existing performance ownership.

## GITHUB021 invariant/delta consistency

The final consistency review after exact owner approval found:

~~~text
D179_C3_MONOTONIC_MEMBERSHIP=PRESERVED
FIRST_CLASS_CONTEXT_IDENTITY=PRESERVED
REFLECTION_AND_ESCAPE=PRESERVED
CAPTURE_BY_REFERENCE=PRESERVED
LATE_CREATION_AND_RETARGETING=PRESERVED
PRESENT_NULL_DISTINCT_FROM_ABSENT=PRESERVED
SEQUENTIAL_PARAMETER_DEFAULT_VISIBILITY=PRESERVED
ASSIGNMENT_DESTINATION_BEFORE_RHS=PRESERVED
PLAT014=PRESERVED
PLAT015=PRESERVED
PLAT019=PRESERVED
PLAT021=PRESERVED
PLAT026=PRESERVED
PLAT034=PRESERVED
PLAT035=PRESERVED
LAZY_CONTEXT_OBJECT_MATERIALIZATION_SELECTED=NO
NEW_OBSERVABLE_SEMANTIC_DELTA=NONE
DECISION_INVARIANT_CONSISTENCY=PASS
~~~

## Implementation ownership and migration contract

PLAT036 is a decision record. It does not itself own implementation execution.

Implementation is tracked by:

~~~text
IMPLEMENTATION_OWNER=I068
IMPLEMENTATION_ISSUE=guillermomolina/protos#708
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
~~~

I068 consumes this ratified decision under `AGENTS.work/IMPLEMENTATION.md`.

The architecture is intended to be implemented incrementally inside I068. The
following are bounded implementation/publication slices unless one later crosses
an independent Issue-promotion trigger from `AGENTS.work/COORDINATION.md`:

| Slice | Boundary | Classification |
| --- | --- | --- |
| 1 | canonical binding identity / presence analysis and backend-private layout metadata | `I068_SLICE` |
| 2 | frame/context single-authority seam | `I068_SLICE` |
| 3 | definitely-current local lowering | `I068_SLICE` |
| 4 | sequential/default parameter lowering | `I068_SLICE` |
| 5 | captured/materialized lexical lowering | `I068_SLICE` |
| 6 | debugger/reflection projection | `I068_SLICE` |
| 7 | ProtosActivation lexical decomposition and final generic-fallback cleanup | `I068_SLICE` |

No slice above requires another semantic/platform decision merely because it
implements Candidate D. If implementation exposes a genuinely new observable or
durable architecture choice, that point must stop at the normal Dxxx/PLATxxx
approval gate.

Issue granularity follows the repository Issue/slice boundary: the slices remain
inside I068 unless one gains independent closure, blockage, scheduling,
dependency, decision-checkpoint, or multi-publication identity that requires a
formal child Issue.

## First I068 implementation slice

The first I068 slice establishes compiler-visible canonical binding identity
without yet cutting over runtime lexical value authority:

~~~text
I068_SLICE_1=
  CANONICAL_BINDING_IDENTITY_AND_PRESENCE_METADATA

GOAL=
  preserve exact current execution behavior while carrying stable binding
  identity/owner/presence classification from canonical analysis into Bytecode
  lowering, so the next slice can establish the frame/context authority seam.

RUNTIME_AUTHORITY_CUTOVER=NO
SEMANTIC_CHANGE=NO
~~~

## Superseded alternative

Candidate E1 remains documented as historical investigation evidence at:

`docs/project/work/PLAT036/PLAT036_POST_I067_LEXICAL_STATE_REEVALUATION.md`

Its value is evidentiary: it records the earlier reasoning and the trade-offs
that were later overturned by the broader post-I067 comparison of the pinned
Bytecode DSL and current Truffle implementations.

It is not current architecture authority.

## Ratification summary

~~~text
PLAT036_SELECTED_CANDIDATE=D
PLAT036_SELECTED_CANDIDATE_NAME=FRAME_BACKED_SEMANTIC_CONTEXT_ADAPTER
PRIOR_E1_RECOMMENDATION=SUPERSEDED

ONE_SEMANTIC_BINDING_VALUE_AUTHORITY_REQUIRED=YES
STATIC_LEXICAL_BINDING_AUTHORITY=TRUFFLE_BYTECODE_DSL_FRAME_LOCAL
DYNAMIC_ONLY_BINDING_AUTHORITY=CONTEXT_DYNAMIC_OVERFLOW
FIRST_CLASS_CONTEXT_SEMANTICS=PRESERVED

LAZY_CONTEXT_OBJECT_MATERIALIZATION=NOT_SELECTED
PERF010_DOMINANCE=NOT_DECIDED

IMPLEMENTATION_AUTHORIZED_UNDER_RATIFIED_ARCHITECTURE=YES
SPECIFICATION_CHANGE_REQUIRED_BY_PLAT036=NO
~~~
