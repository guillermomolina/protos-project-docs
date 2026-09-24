# I068 — PLAT036 frame-backed lexical-state implementation

Status: **OPEN / READY**

Issue: `guillermomolina/protos#708`

Implements: PLAT036 / #702 — Candidate D, frame-backed semantic-context adapter.

## Authority

~~~text
PLAT036_SELECTED_CANDIDATE=D
PLAT036_SELECTED_CANDIDATE_NAME=FRAME_BACKED_SEMANTIC_CONTEXT_ADAPTER
PRIOR_E1_RECOMMENDATION=SUPERSEDED

PROTOS_BASELINE_REVISION=75231601e458930684d4b619dd5f4722377aa65d
PLAT036_PROJECT_RECORD_REVISION=96e093ab0e4a625c8d768a5a11c36d97d921b2c8
~~~

Canonical decision authority:

`docs/project/decisions/platform/PLAT036_BYTECODE_DSL_LEXICAL_STATE_REPRESENTATION_BOUNDARY.md`

I068 is implementation work and therefore consumes `AGENTS.work/IMPLEMENTATION.md`.
PLAT036 remains the architecture authority; I068 must not reinterpret the
ratified decision.

## Implementation objective

Implement one-authority lexical state under Candidate D:

~~~text
STATIC_LEXICAL_BINDING_AUTHORITY=
  TRUFFLE_BYTECODE_DSL_FRAME_LOCAL

DYNAMIC_ONLY_BINDING_AUTHORITY=
  CONTEXT_DYNAMIC_OVERFLOW

FIRST_CLASS_CONTEXT=
  STABLE_SEMANTIC_OBJECT_AND_ADAPTER

ONE_SEMANTIC_BINDING_VALUE_AUTHORITY_REQUIRED=YES
~~~

Preserve D179/C3, first-class context identity/reflection/escape, capture by
reference, legal late creation and retargeting, sequential/default parameter
visibility, assignment destination before RHS, exact dynamic fallback, debugger
scope semantics, suspension and dynamic-control behavior.

Lazy physical context-object materialization remains outside PLAT036 and I068.

## Slice plan

The implementation is decomposed into bounded publication slices owned by I068:

1. canonical binding identity / presence metadata;
2. frame/context single-authority seam;
3. definitely-current local lowering;
4. sequential/default parameter lowering;
5. captured/materialized lexical lowering;
6. debugger/reflection projection;
7. ProtosActivation lexical decomposition and fallback cleanup.

These are not PLAT implementation slices. They are implementation slices inside
I068. If one later crosses an Issue-promotion trigger, create a formal child
Issue and establish the native hierarchy required by coordination policy.

## First slice

~~~text
I068_SLICE_1=CANONICAL_BINDING_IDENTITY_AND_PRESENCE_METADATA

RUNTIME_AUTHORITY_CUTOVER=NO
BYTECODELOCAL_GUEST_AUTHORITY=NO
MATERIALIZED_LOCAL_MIGRATION=NO
CONTEXT_ADAPTER_IMPLEMENTED=NO
SEMANTIC_CHANGE=NO
~~~

The goal is to preserve statically knowable binding identity, owner/depth and
presence requirements through canonical analysis into Bytecode lowering while
leaving current runtime lexical lookup/write behavior authoritative.

## Completion boundary

I068 remains open across the slices above and closes only when the complete
ratified Candidate D architecture is implemented, validated and published.

Any newly discovered observable semantic choice or new durable architecture
choice leaves I068 implementation scope and returns to the normal Dxxx/PLATxxx
approval gate.
