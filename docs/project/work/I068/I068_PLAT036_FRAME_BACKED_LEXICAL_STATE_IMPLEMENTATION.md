# I068 — PLAT036 frame-backed lexical-state implementation

Status: **OPEN / READY — SLICES 1–3 PUBLISHED; SLICE 4 NEXT**

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


## Slice 3 publication checkpoint — 2026-09-24

~~~text
I068_SLICE_3=DEFINITELY_CURRENT_LOCAL_LOWERING
PROTOS_REVISION=1756b3d3100d54ef1627bc618cae6b7ef8da3445
PROTOS_VERSION=0.3.83-SNAPSHOT
COMMIT_MESSAGE=I068: lower definitely-current lexicals to frame-backed locals

SLICE_3_EVIDENCE_REVISION=abc6fa288d177b9a3e8781b32f7aa0e1aeedfd6f

RUNTIME_AUTHORITY_CUTOVER=CURRENT_RESOLVED_ONLY
FRAME_BACKED_CURRENT_BINDING_AUTHORITY=YES
DIRECT_CURRENT_RESOLVED_READ_PATH=YES
CURRENT_BINDING_WRITES_SHARE_FRAME_AUTHORITY=YES
DUAL_AUTHORITATIVE_COPIES=NO
PREEXISTING_CONTEXT_BINDINGS_PRESERVED=YES

CANDIDATE_DYNAMIC_FALLBACK_PRESERVED=YES
OBJECT_BODY_REMAINS_ORDINARY_STATE=YES
PRESENT_NULL_DISTINCT_FROM_ABSENT=YES
D179_C3_PRESERVED=YES
ESCAPED_CONTEXT_OBSERVES_FRAME_AUTHORITY=YES
BYTECODE_REPARSE_STATE_PRESERVED=YES

PARAMETER_FRAME_LOCAL_MIGRATION=NOT_YET_IMPLEMENTED
CAPTURED_MATERIALIZED_LEXICAL_LOWERING=NOT_YET_IMPLEMENTED
LAZY_CONTEXT_MATERIALIZATION=NO

GIT_DIFF_CHECK=PASS
MAKE_COMPILE=PASS
I068_SLICE3_FOCAL_TESTS=PASS
MAKE_TEST_JAVA=PASS
MAKE_TEST_PROTOS=PASS
PROTOS_TESTS_PASSED=1250
PROTOS_TESTS_FAILED=0
PROTOS_TESTS_TOTAL_TIME_SECONDS=113

SLICE_3_STATUS=COMPLETE
NEXT_SLICE=I068_SLICE_4_SEQUENTIAL_DEFAULT_PARAMETER_LOWERING
~~~

Slice 3 is the first runtime lexical-authority cutover under Candidate D.
Statically `Resolved` bindings owned by the genuine current execution-context
scope receive stable Bytecode DSL local storage. Eligible bare reads use the
generated local-accessor path, while context-based writes and reflection reach
the same frame-backed authority rather than a second map-backed value store.

The final integrated implementation also preserves contexts that were populated
before root execution by migrating those bindings during a single-authority
handoff, and it re-establishes root-specific lowering state whenever Truffle's
retained Bytecode parser is re-invoked for lazy source/instrumentation metadata.

`Candidate`, `Dynamic`, object-body, parameter and captured/materialized outer
lexical paths remain outside the Slice 3 direct-current admission boundary.

Maintainer-reported validation on the published product revision includes the
ordinary Java suite and the full native Protos suite. The latter completed with
1250 passed, 0 failed in 113 seconds. No remote CI PASS is claimed by this
checkpoint because no combined status or associated pull-request workflow run
was observed for the product SHA at record time.

Retained evidence:

`docs/project/evidence/I068/I068_SLICE3_DEFINITELY_CURRENT_LOCAL_LOWERING.md@abc6fa288d177b9a3e8781b32f7aa0e1aeedfd6f`

I068 remains open. Slice 4 owns sequential/default parameter lowering and must
preserve semantic absence until each parameter binding point.
