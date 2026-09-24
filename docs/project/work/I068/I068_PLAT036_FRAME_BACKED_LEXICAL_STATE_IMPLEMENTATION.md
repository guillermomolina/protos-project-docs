# I068 — PLAT036 frame-backed lexical-state implementation

Status: **OPEN / READY — SLICES 1–2 PUBLISHED; SLICE 3 NEXT**

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


## Slice 1 publication checkpoint — 2026-09-24

The first I068 implementation slice is published in the product repository:

~~~text
I068_SLICE_1=CANONICAL_BINDING_IDENTITY_AND_PRESENCE_METADATA

PROTOS_REVISION=04a243c863cf50feb1ab541e3a49dd4dcd317039
PROTOS_VERSION=0.3.81-SNAPSHOT
COMMIT_MESSAGE=I068: retain canonical binding identity and presence metadata

STATIC_BINDING_IDENTITY_RETAINED=YES
LEXICAL_OWNER_METADATA_RETAINED=YES
LEXICAL_DEPTH_METADATA_RETAINED=YES
PRESENCE_CLASSIFICATION_RETAINED=YES
DYNAMIC_FALLBACK_PRESERVED=YES
ASSIGNMENT_DESTINATION_ORDER_PRESERVED=YES

RUNTIME_AUTHORITY_CUTOVER=NO
BYTECODELOCAL_GUEST_AUTHORITY=NO
MATERIALIZED_LOCAL_MIGRATION=NO
CONTEXT_ADAPTER_IMPLEMENTED=NO
LAZY_CONTEXT_MATERIALIZATION=NO

MAINTAINER_REPORTED_TESTS=ALL_GREEN
SLICE_1_STATUS=COMPLETE
NEXT_SLICE=I068_SLICE_2_FRAME_CONTEXT_SINGLE_AUTHORITY_SEAM
~~~

The product delta introduces backend-private canonical binding analysis and
wires it into Bytecode lowering without consuming the result in generated
execution yet. The existing String-keyed `ProtosActivation` lexical
lookup/write path therefore remains the sole runtime authority in this
checkpoint.

Retained evidence:

`docs/project/evidence/I068/I068_SLICE1_CANONICAL_BINDING_IDENTITY_AND_PRESENCE_METADATA.md`


## Slice 2 publication checkpoint — 2026-09-24

~~~text
I068_SLICE_2=FRAME_CONTEXT_SINGLE_AUTHORITY_SEAM
PROTOS_REVISION=1ccbb717fa824acd7aef71bd89c876e3bf8f6fe7
PROTOS_VERSION=0.3.82-SNAPSHOT
COMMIT_MESSAGE=I068: establish frame-context single-authority seam

AUTHORITY_SEAM_IMPLEMENTED=YES
AUTHORITY_SEAM_ACTUALLY_USED=YES
EXECUTION_CONTEXT_SINGLE_AUTHORITY=YES
DUAL_AUTHORITATIVE_COPIES=NO
D179_C3_PRESERVED=YES
PRESENT_NULL_DISTINCT_FROM_ABSENT=YES

RUNTIME_AUTHORITY_CUTOVER=NO
BYTECODELOCAL_GUEST_AUTHORITY=NO
MATERIALIZED_LOCAL_MIGRATION=NO
LAZY_CONTEXT_MATERIALIZATION=NO

SLICE_2_PUBLICATION=PUBLISHED
GITHUB_ACTIONS_RUN=35992940177
GITHUB_ACTIONS_STATUS_AT_RECORD_TIME=IN_PROGRESS
NEXT_SLICE=I068_SLICE_3_DEFINITELY_CURRENT_LOCAL_LOWERING
~~~

The product publication routes local-slot operations through one installed
`ProtosLexicalBindingAuthority`. The active authorities remain map-backed, so
the seam is established without a second binding-value store and without moving
guest lexical values into Bytecode locals.

Retained evidence:

`docs/project/evidence/I068/I068_SLICE2_FRAME_CONTEXT_SINGLE_AUTHORITY_SEAM.md`
