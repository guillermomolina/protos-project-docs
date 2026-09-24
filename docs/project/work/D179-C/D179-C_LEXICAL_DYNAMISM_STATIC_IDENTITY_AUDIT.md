# D179-C — Lexical dynamism and Bytecode DSL static-identity audit

FORMAL_IDENTIFIER=D179-C
GITHUB_ISSUE=https://github.com/guillermomolina/protos/issues/706
PARENT=D179 / https://github.com/guillermomolina/protos/issues/703
WORK_STATE=OPEN / RESEARCH REQUIRED
WORK_KIND=INVESTIGATION / DESIGN EVIDENCE ONLY
PROTOS_REVISION=3e8e6b565c95eb5098c2168d241536ba13ad19e9
FORMAL_IDENTIFIER_UNIQUE=PASS
NATIVE_PARENT_DECLARED=#703
NATIVE_PARENT_RELATION=COORDINATION_PENDING
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

## Opening coordination state

The live Issue body declares `Parent: #703` as bootstrap coordination input.
The available GitHub connector in this session does not expose native
Parent/Sub-issue mutation. Under GITHUB006/GITHUB015, the native relationship
remains a visible coordination postcondition until established and verified by
a supported path.
