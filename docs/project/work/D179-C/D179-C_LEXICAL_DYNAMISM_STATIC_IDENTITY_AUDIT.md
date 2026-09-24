# D179-C — Lexical dynamism and Bytecode DSL static-identity audit

FORMAL_IDENTIFIER=D179-C
GITHUB_ISSUE=https://github.com/guillermomolina/protos/issues/706
PARENT=D179 / https://github.com/guillermomolina/protos/issues/703
WORK_STATE=OPEN / READY
WORK_KIND=INVESTIGATION / DESIGN EVIDENCE ONLY
PROTOS_REVISION=3e8e6b565c95eb5098c2168d241536ba13ad19e9
FORMAL_IDENTIFIER_UNIQUE=PASS
NATIVE_PARENT_DECLARED=#703
NATIVE_PARENT_RELATION=PASS
IMPLEMENTATION_AUTHORIZED=NO

This is the durable opening record for D179-C. It does not select a concrete
frame/context authority architecture and does not authorize implementation.

## Purpose

D179-C identifies every current Protos semantic reason why a lexical name cannot
always be lowered to a statically known Bytecode DSL local or materialized-local
identity.

The investigation must distinguish genuine language dynamism from uncertainty
that exists only because the current runtime represents lexical state through
names and execution-context object slots.

## Required semantic cases

Audit at minimum:

- parameters after establishment;
- sequential parameter establishment;
- defaults that can observe an outer same-name binding;
- local `x: value` creation;
- explicit `context.x: value` creation;
- late creation after Closure capture;
- creation through escaped/captured context references;
- shadowing;
- bare lexical reads;
- receiver/member fallback;
- bare assignment destination selection and RHS pinning;
- captured lexical reads/writes by reference;
- module/top-level contexts versus invocation contexts;
- names provably resolved during canonical lowering;
- genuinely dynamic names not statically resolvable;
- structural mutation conclusions from D179-A;
- reflection/escape conclusions from D179-B.

## Required evidence

For every materially distinct case, classify independently:

    BINDING_IDENTITY_STATIC
    PRESENCE_STATIC
    LEXICAL_DEPTH_STATIC
    DSL_LOCAL_POSSIBLE
    MATERIALIZED_LOCAL_POSSIBLE
    DYNAMIC_FALLBACK_REQUIRED
    STATIC_INFORMATION_CURRENTLY_LOST
    SEMANTIC_REASON_FOR_DYNAMICISM
    LANGUAGE_CHANGE_NEEDED_FOR_MORE_STATICITY
    VALUE_OF_THAT_LANGUAGE_CAPABILITY
    FINDING
    EVIDENCE

The final result must define:

    STATIC/INDEXED ADMISSION SET
    DYNAMIC FALLBACK SET
    FACTS THAT BECOME STATIC ONLY IF D179 RESTRICTS A CAPABILITY
    FACTS THAT REMAIN DYNAMIC EVEN AFTER PLAUSIBLE D179 RESTRICTIONS

Concrete frame, cell, adapter, storage-authority and materialization architecture
remain PLAT036 concerns.

## Dependency on sibling evidence

D179-C must consume D179-A and D179-B where their findings materially determine
whether presence, identity, escape, or reflective mutation remain dynamic.
It may investigate in parallel, but final classification cannot silently assume
unresolved sibling outcomes.

## Coordination state

The native GitHub Parent/Sub-issue relation has subsequently converged and was
re-read successfully:

    NATIVE_PARENT_RELATION=PASS
    NATIVE_PARENT=#703

The original textual `Parent: #703` declaration remains useful historical
bootstrap prose, but live hierarchy authority is the native relation.


## Current sibling dependency state

D179-A / #704 is complete and its durable result establishes that both
`PRESENT -> ABSENT` removal and `ABSENT -> PRESENT` growth in a nearer
escaped/captured context can retarget later bare lexical resolution.

D179-C still requires D179-B / #705 before final classification because D179-B
owns the unresolved question of which first-class context identity, reflection,
escape, aliasing, and materialization requirements must remain observable.

Current coordination therefore is:

    D179_A=#704 COMPLETE
    D179_B=#705 OPEN / READY
    D179_C=#706 BLOCKED_BY_D179_B
    NATIVE_DEPENDENCY_EDGE_706_BLOCKED_BY_705=PENDING_CONNECTOR_SUPPORT

The missing native dependency edge is a live-coordination postcondition only.
It does not change the semantic dependency itself and does not authorize D179-C
to finalize before D179-B evidence exists.

## D179-B completion input

D179-B / #705 has completed and its exact durable evidence is:

    D179_B_PROJECT_RECORD_REVISION=88f87f5d368ef0ccba1202b02def6f98ee60cc03
    D179_B_RESULT=COMPLETE

The previous `BLOCKED_BY_D179_B` coordination state above is historical and is
superseded by this section.

D179-C must consume these established D179-B facts directly:

    CONTEXT_IDENTITY_REQUIRES_EAGER_OBJECT=NO
    ESCAPE_REQUIRES_OBJECT_BACKED_VALUES=NO
    REFLECTION_CAN_PROJECT_FRAME_LOCALS=YES
    EXTERNAL_VALUE_MUTATION_REQUIRES_SHARED_AUTHORITY=YES
    EXTERNAL_STRUCTURAL_MUTATION_REQUIRES_INVALIDATION=YES
    LATE_ADD_CAN_RETARGET_STATIC_BINDING_IDENTITY=YES
    REMOVE_CAN_RETARGET_STATIC_BINDING_IDENTITY=YES
    DEBUGGER_REQUIRES_MATERIALIZATION=NO
    CAPTURE_REQUIRES_GUEST_CONTEXT_MATERIALIZATION=NO
    CAPTURE_REQUIRES_SHARED_OR_MATERIALIZED_BINDING_AUTHORITY=YES
    DYNAMIC_OVERFLOW_REQUIRED=YES_FOR_NONSTATIC_NAMES
    EXPLICIT_SEMANTIC_PRESENCE_REQUIRED=YES
    PRESENT_NULL_DISTINCT_FROM_ABSENT=YES
    MODULE_CONTEXT_PERSISTENT_AUTHORITY_REQUIRED=YES
    OBJECT_CONSTRUCTION_CONTEXT_REQUIRES_SEPARATE_CLASSIFICATION=YES
    FROZEN_PRELUDE_REQUIRES_SEPARATE_CLASSIFICATION=YES

Combined with D179-A, D179-C must not use the historical assumption that
`removeSlot` is the only source of lexical retargeting. Both
`PRESENT -> ABSENT` removal and `ABSENT -> PRESENT` growth in a nearer
escaped/captured context can invalidate static nearest-binding identity.

The child is now actionable research:

    D179_A=#704 COMPLETE
    D179_B=#705 COMPLETE
    D179_C=#706 OPEN / READY
    D179_C_BLOCKED_BY_D179_B=NO
    IMPLEMENTATION_AUTHORIZED=NO

PLAT036 remains blocked by parent D179, not released by this sibling transition.
