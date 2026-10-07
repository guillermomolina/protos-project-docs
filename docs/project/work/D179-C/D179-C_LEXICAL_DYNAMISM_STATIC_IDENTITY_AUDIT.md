# D179-C — Lexical dynamism and Bytecode DSL static-identity audit

FORMAL_IDENTIFIER=D179-C
GITHUB_ISSUE=https://github.com/guillermomolina/protos/issues/706
PARENT=D179 / https://github.com/guillermomolina/protos/issues/703
WORK_STATE=COMPLETE
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


## Completion evidence

D179-C completed the lexical-dynamism/static-identity audit against the exact
product baseline:

    PROTOS_REVISION=3e8e6b565c95eb5098c2168d241536ba13ad19e9

It consumed the complete D179-A and D179-B results and reconstructed the actual
compiler/runtime information flow rather than treating the current Java object
representation as semantic authority.

### Current information-flow finding

The pinned implementation currently lowers ordinary bare names through:

    SurfaceName(name)
      -> CanonicalLookup(name, span)
      -> CanonicalToBytecodeLowerer.emitLookup
      -> Lookup(ProtosActivation, String)
      -> ProtosActivation.lookup(name)
      -> current context local-slot search
      -> captured lexical-context local-slot search
      -> receiver/member fallback

Creation and bare assignment likewise carry a source-known binding name as a
String. Bare assignment already resolves and pins the writable destination
before RHS evaluation.

The repository also contains ProtosStaticDefinitions, which proves exact source
origins for a bounded set of current-activation bindings and deliberately drops
those facts at opaque guest-invocation barriers. This demonstrates that useful
static identity information exists before execution but is not carried into the
canonical execution representation.

Therefore:

    STATIC_INFORMATION_LOST_BEFORE_DSL=YES
    STATIC_INFORMATION_LOSS_LAYER=
      CANONICAL_BINDING_ANALYSIS_AND_REPRESENTATION_BEFORE_BYTECODE_LOWERING
    BYTECODE_DSL_PREVENTS_STATIC_LEXICAL_IDENTITY=NO

### Core semantic distinction

D179-C separates physical indexed storage from semantic binding presence.

A source-known parameter/local/module binding may have a preallocated physical
slot without becoming semantically PRESENT before the language establishes it.

Required invariant:

    PHYSICAL_SLOT_EXISTS != SEMANTIC_BINDING_PRESENT

In particular:

    PRESENT(null) != ABSENT

Sequential parameter/default semantics, source-local creation, partially
initialized modules and reflection therefore require explicit semantic presence
state independently of physical slot allocation.

### Static/indexed admission set

The following cases can have compiler-visible stable binding identity without a
language change when their required presence/topology facts are proven:

    DEFINITELY_ESTABLISHED_CURRENT_PARAMETER
    DEFINITELY_ESTABLISHED_CURRENT_REST_PARAMETER
    DEFINITELY_ESTABLISHED_CURRENT_SOURCE_LOCAL
    CURRENT_EXISTING_BINDING_ASSIGNMENT
    EARLIER_ESTABLISHED_PARAMETER_IN_LATER_DEFAULT
    SOURCE_KNOWN_CURRENT_CREATION_TARGET
    EXPLICIT_CURRENT_CONTEXT_SOURCE_KNOWN_CREATION_TARGET
    FIXED_MEMBERSHIP_EXISTING_VALUE_MUTATION

Known captured bindings can likewise use fixed lexical depth plus binding
identity when the target is PRESENT and all nearer same-name candidates are
proven ABSENT:

    DEFINITELY_ESTABLISHED_CAPTURED_PARAMETER
    DEFINITELY_ESTABLISHED_CAPTURED_SOURCE_LOCAL
    CAPTURED_BINDING_WITH_MUTABLE_VALUE_AND_FIXED_MEMBERSHIP
    FIXED_OUTER_BINDING_WITH_PROVEN_NEARER_ABSENCE

Existing-value mutation does not itself invalidate binding identity, presence or
lexical depth.

### Guarded / invalidatable static set

The D179-A correction is retained:

    REMOVE_SLOT_UNIQUELY_PROBLEMATIC=NO

Both:

    PRESENT -> ABSENT
and
    ABSENT -> PRESENT in a nearer lexical context

can retarget a later bare lookup.

That does not force universal dynamic lookup. It means an optimized access whose
correctness depends on those membership facts must guard, invalidate, deopt,
re-specialize or fall back when the corresponding structural transition occurs.

Material guarded cases include:

    CURRENT_BINDING_SUBJECT_TO_REMOVE
    CAPTURED_BINDING_SUBJECT_TO_REMOVE
    OUTER_BINDING_SUBJECT_TO_NEARER_LATE_CREATION
    CLOSURE_CREATED_BEFORE_LATER_SOURCE_LOCAL_CREATION
    ESCAPED_CONTEXT_STRUCTURAL_MUTATION
    CAPTURED_CONTEXT_STRUCTURAL_MUTATION
    ASSIGNMENT_SELECTION_AFTER_POSSIBLE_STRUCTURAL_CHANGE
    PRELUDE_ACCESS_WITH_MUTABLE_NEARER_CONTEXTS
    SOURCE_KNOWN_PHYSICAL_SLOT_WITH_DYNAMIC_SEMANTIC_PRESENCE

Arbitrary structural mutation of one escaped context does not semantically force
all lexical accesses everywhere to become dynamic. Only facts depending on the
affected semantic context/name/topology must be invalidated.

### Assignment pinning

Bare assignment is especially important:

    resolve exact writable destination
      BEFORE
    RHS evaluation

The current lowerer already preserves this by storing the mutation target before
evaluating the RHS.

If the RHS creates a nearer same-name binding, the final write must still target
the binding selected before the RHS. If the RHS removes or freezes the selected
target, the final write must fail against that selected target rather than
re-resolve the name.

Therefore a direct binding token/index is compatible with current assignment
semantics and can express the required destination pinning more directly than
post-RHS String re-resolution.

### Genuine dynamic fallback set

Dynamic resolution remains required for materially real language cases:

    CURRENT_PARAMETER_DURING_OWN_DEFAULT
    LATER_PARAMETER_DURING_EARLIER_DEFAULT
    UNRESOLVED_BARE_NAME
    INVALIDATED_NEAREST_BINDING
    RECEIVER_MEMBER_FALLBACK
    DELEGATED_MEMBER_LOOKUP
    NONSTATIC_RUNTIME_NAME
    DYNAMIC_CONTEXT_OVERFLOW
    RUNTIME_NAME_CONTEXT_REFLECTION
    CONSTRUCTION_RUNTIME_STRUCTURAL_NAMES
    ASSIGNMENT_WITH_UNPROVEN_DESTINATION

First-class context identity, reflection, escape, Closure capture, debugger
observation and existing-binding value mutation do not by themselves require
ordinary guest lexical reads to use generic String lookup.

### Special contexts

Three context categories require separate treatment by PLAT036 rather than being
collapsed mechanically into one ephemeral invocation-frame representation.

#### Module context

A moduleContext has persistent Actor-local identity, is captured by Closures and
is placed in the module cache as INITIALIZING before body execution. Cyclic
imports can observe the real partially initialized module instance.

A future source-known top-level binding may therefore have a physical indexed
slot while remaining semantically ABSENT until its creation statement executes.

#### Frozen prelude

The prelude's own structural membership and values are frozen. Its own binding
identity is therefore stable. What remains conditional is whether lookup reaches
the prelude, because nearer module/invocation contexts may legally shadow it.

#### Object-construction context

The construction context is the object being constructed and its slots are
object/receiver state. Method Closures created in the object body do not capture
that construction object as an ordinary lexical environment. It must therefore
not be treated mechanically as an invocation-local lexical frame.

### Reflection and escape

D179-B's one-authority requirement is strengthened, not weakened.

Any frame/indexed representation must expose one coherent semantic authority for:

    VALUE
    PRESENCE
    MUTATION STATE
    REFLECTION
    CAPTURE
    ESCAPE

Reflection may project statically indexed bindings plus dynamic overflow. It must
not expose a second independently writable copy.

Runtime-name hasSlot/slotValue/slotNames remain genuinely dynamic reflection
operations. Debugger scope remains a semantic projection and must not expose raw
implementation temporaries as guest lexical bindings.

### Counterexample / falsification results

The required adversarial cases were tested semantically.

Late nearer creation:

    outer x PRESENT
    inner x ABSENT
    Closure initially resolves x to outer
    later inner.x: value
    Closure must now resolve x to inner

Result:

    UNCONDITIONAL_OUTER_BINDING_STATICITY=FALSE
    GUARDED_OUTER_BINDING_STATICITY=TRUE

Removal:

    outer x PRESENT
    inner x PRESENT
    Closure initially resolves x to inner
    inner.removeSlot("x")
    Closure must now resolve x to outer

Result:

    UNCONDITIONAL_INNER_BINDING_STATICITY=FALSE
    GUARDED_INNER_BINDING_STATICITY=TRUE

Parameter default:

    outer p exists
    current parameter p is being defaulted

During evaluation of p's default, the current p is semantically ABSENT even if a
physical p slot has already been reserved. Bare p must still be able to resolve
the outer binding. Only successful binding establishes current p.

Module cycle:

    module A cached INITIALIZING
    early binding created
    later binding not yet created
    recursive import returns same A instance

The early binding is PRESENT and the later binding is ABSENT regardless of
whether both have physical indexed positions.

These counterexamples reject eager semantic predeclaration, but do not reject
preallocated physical slots plus explicit presence/fallback.

### Comparative finding

The prior-art evidence retained by D179/AUD016 remains consistent with this
classification:

- TruffleRuby/Ruby preserve compiler-visible local identity/depth while exposing
  reflective Binding facilities separately.
- TruffleSqueak coordinates first-class context semantics with frame/materialized
  frame state.
- Python separates fixed lexical classification from unbound presence, although
  Python deliberately does not fall through to an outer binding when a local is
  unbound and therefore is not semantic authority for Protos.
- ECMAScript separates declarative binding identity from initialization state.
- Lua uses lexically indexed locals/upvalues while declaration visibility begins
  only at the language-defined point.
- TruffleSOM uses fixed local/depth information for normal lexical access.

The transferable pattern is:

    statically known semantic binding
      -> indexed identity

    runtime presence/value/reflection
      -> separate coherent state/authority

    genuinely unknown name or invalidated topology
      -> dynamic path

No comparison justifies deleting Protos's required fallback semantics.

### Facts that become stronger only if D179 restricts capabilities

Useful indexed/static access does not require a D179 semantic restriction.

A language restriction is required only for stronger unconditional claims such
as:

    binding identity can never retarget across arbitrary structural effects
    a future declaration counts as present before execution
    dynamic/nonstatic context names can never exist
    receiver fallback can be eliminated

In particular, restricting removeSlot alone is not sufficient for universal
static identity because nearer late creation still permits ABSENT -> PRESENT
retargeting.

### Facts remaining dynamic even if removeSlot is restricted

    NEARER_LATE_CREATION
    SEQUENTIAL_PARAMETER_ESTABLISHMENT
    OWN_DEFAULT_OUTER_LOOKUP
    LATER_PARAMETER_OUTER_LOOKUP
    SOURCE_KNOWN_ABSENT_TO_PRESENT_TRANSITION
    MODULE_PARTIAL_INITIALIZATION
    NONSTATIC_DYNAMIC_OVERFLOW
    RECEIVER_FALLBACK
    DELEGATED_MEMBER_LOOKUP
    CONSTRUCTION_DYNAMIC_STRUCTURE
    RUNTIME_NAME_REFLECTION
    VALUE_MUTATION
    CLOSE_FREEZE_MUTATION_AUTHORITY

Therefore:

    REMOVE_SLOT_RESTRICTION_ALONE_SUFFICIENT_FOR_UNIVERSAL_STATIC_IDENTITY=NO

### Facts passed to parent D179

D179-C establishes:

    CURRENT_PROTOS_SEMANTICS_PERMIT_A_USEFUL_STATIC_INDEXED_LEXICAL_ADMISSION_SET=YES
    FIRST_CLASS_CONTEXTS_ALONE_BLOCK_THAT_SET=NO
    STRUCTURAL_ADD_REMOVE_REQUIRE_GUARDED_INVALIDATABLE_STATICITY_FOR_AFFECTED_ACCESSES=YES
    GENERIC_DYNAMIC_FALLBACK_REMAINS_REQUIRED=YES
    GENERIC_DYNAMIC_FALLBACK_REQUIRED_FOR_ALL_LEXICAL_ACCESSES=NO

Combined child state:

    D179_A=COMPLETE
    D179_B=COMPLETE
    D179_C=COMPLETE
    D179_PARENT_DECISION_READY=YES
    D179_PARENT_DECISION_PERFORMED=NO
    PLAT036_STATE=BLOCKED

"Parent decision ready" means only that D179 now has enough child evidence to
rebuild its complete candidate set, GITHUB010 comparison and recommendation
packet. It does not ratify a semantic candidate.

### Facts passed forward to PLAT036 after D179 ratification

PLAT036 may later consume the following architectural constraints, but remains
blocked until parent D179 is explicitly approved/ratified:

    UNIVERSAL_STRING_KEY_LEXICAL_LOOKUP_REQUIRED=NO
    STABLE_COMPILER_VISIBLE_BINDING_IDS_REQUIRED_FOR_STATIC_SET=YES
    INDEXED_CURRENT_BINDING_REPRESENTATION_SEMANTICALLY_POSSIBLE=YES
    FIXED_DEPTH_CAPTURED_BINDING_REPRESENTATION_SEMANTICALLY_POSSIBLE=YES
    EXPLICIT_PRESENCE_STATE_REQUIRED=YES
    PRESENT_NULL_DISTINCT_FROM_ABSENT=YES
    STRUCTURAL_INVALIDATION_OR_EQUIVALENT_REQUIRED=YES
    DYNAMIC_OVERFLOW_REQUIRED_FOR_NONSTATIC_NAMES=YES
    ONE_SEMANTIC_VALUE_AUTHORITY_REQUIRED=YES
    ASSIGNMENT_DESTINATION_PINNING_REQUIRED=YES
    MODULE_CONTEXT_PERSISTENT_AUTHORITY_REQUIRED=YES
    FROZEN_PRELUDE_REQUIRES_SEPARATE_CLASSIFICATION=YES
    OBJECT_CONSTRUCTION_CONTEXT_REQUIRES_SEPARATE_CLASSIFICATION=YES
    DEBUGGER_SEMANTIC_SCOPE_PROJECTION_REQUIRED=YES

D179-C does not select BytecodeLocal layout, MaterializedLocalAccessor, cells,
frame/context adapters, presence-bit representation, invalidation granularity,
lazy context materialization or any other PLAT036 architecture.

### Performance discipline

No benchmark was run and no causal attribution is made.

    SEMANTIC_CONSTRAINT=ESTABLISHED
    ARCHITECTURAL_CONSTRAINT=ESTABLISHED
    MEASURED_PERFORMANCE_COST=NOT_ESTABLISHED
    D179_C_EXPLAINS_400X_OR_4000X=NOT_CLAIMED
    MEASURED_PERFORMANCE_CLAIM=NONE

## Completion result

    D179_C_RESULT=COMPLETE

    STATIC_INDEXED_ADMISSION_SET=
      DEFINITELY_ESTABLISHED_CURRENT_PARAMETER
      DEFINITELY_ESTABLISHED_CURRENT_REST_PARAMETER
      DEFINITELY_ESTABLISHED_CURRENT_SOURCE_LOCAL
      CURRENT_EXISTING_BINDING_ASSIGNMENT
      EARLIER_ESTABLISHED_PARAMETER_IN_LATER_DEFAULT
      SOURCE_KNOWN_CURRENT_CREATION_TARGET
      EXPLICIT_CURRENT_CONTEXT_SOURCE_KNOWN_CREATION_TARGET
      FIXED_MEMBERSHIP_EXISTING_VALUE_MUTATION

    MATERIALIZED_FIXED_DEPTH_ADMISSION_SET=
      DEFINITELY_ESTABLISHED_CAPTURED_PARAMETER
      DEFINITELY_ESTABLISHED_CAPTURED_SOURCE_LOCAL
      CAPTURED_BINDING_WITH_MUTABLE_VALUE_AND_FIXED_MEMBERSHIP
      FIXED_OUTER_BINDING_WITH_PROVEN_NEARER_ABSENCE

    GUARDED_INVALIDATABLE_STATIC_SET=
      CURRENT_BINDING_SUBJECT_TO_REMOVE
      CAPTURED_BINDING_SUBJECT_TO_REMOVE
      OUTER_BINDING_SUBJECT_TO_NEARER_LATE_CREATION
      CLOSURE_CREATED_BEFORE_LATER_SOURCE_LOCAL_CREATION
      ESCAPED_CONTEXT_STRUCTURAL_MUTATION
      CAPTURED_CONTEXT_STRUCTURAL_MUTATION
      ASSIGNMENT_SELECTION_AFTER_POSSIBLE_STRUCTURAL_CHANGE
      PRELUDE_ACCESS_WITH_MUTABLE_NEARER_CONTEXTS
      SOURCE_KNOWN_PHYSICAL_SLOT_WITH_DYNAMIC_SEMANTIC_PRESENCE

    DYNAMIC_FALLBACK_SET=
      CURRENT_PARAMETER_DURING_OWN_DEFAULT
      LATER_PARAMETER_DURING_EARLIER_DEFAULT
      UNRESOLVED_BARE_NAME
      INVALIDATED_NEAREST_BINDING
      RECEIVER_MEMBER_FALLBACK
      DELEGATED_MEMBER_LOOKUP
      NONSTATIC_RUNTIME_NAME
      DYNAMIC_CONTEXT_OVERFLOW
      RUNTIME_NAME_CONTEXT_REFLECTION
      CONSTRUCTION_RUNTIME_STRUCTURAL_NAMES
      ASSIGNMENT_WITH_UNPROVEN_DESTINATION

    SPECIAL_CONTEXT_SET=
      MODULE_CONTEXT
      FROZEN_PRELUDE
      OBJECT_CONSTRUCTION_CONTEXT
      DEBUGGER_SEMANTIC_SCOPE_PROJECTION

    SOURCE_KNOWN_LATE_CREATION_CAN_USE_PREALLOCATED_SLOT_PLUS_PRESENCE=YES
    ARBITRARY_STRUCTURAL_MUTATION_FORCES_ALL_LEXICAL_ACCESSES_DYNAMIC=NO
    EXISTING_VALUE_MUTATION_INVALIDATES_BINDING_IDENTITY=NO
    REMOVE_SLOT_UNIQUELY_BLOCKS_STATIC_IDENTITY=NO
    CONTEXT_IDENTITY_REQUIRES_GENERIC_LEXICAL_LOOKUP=NO
    REFLECTION_REQUIRES_GENERIC_LEXICAL_LOOKUP=NO
    ESCAPE_REQUIRES_GENERIC_LEXICAL_LOOKUP=NO
    CAPTURE_BY_REFERENCE_REQUIRES_GENERIC_LEXICAL_LOOKUP=NO
    STRUCTURAL_MUTATION_REQUIRES_INVALIDATION=YES
    EXPLICIT_PRESENCE_STATE_REQUIRED=YES
    PRESENT_NULL_DISTINCT_FROM_ABSENT=YES
    MODULE_CONTEXT_REQUIRES_SEPARATE_CLASSIFICATION=YES
    OBJECT_CONSTRUCTION_CONTEXT_REQUIRES_SEPARATE_CLASSIFICATION=YES
    FROZEN_PRELUDE_REQUIRES_SEPARATE_CLASSIFICATION=YES
    ONE_SEMANTIC_VALUE_AUTHORITY_REQUIRED=YES
    BYTECODE_DSL_PREVENTS_STATIC_LEXICAL_IDENTITY=NO
    STATIC_INFORMATION_LOST_BEFORE_DSL=YES
    D179_LANGUAGE_RESTRICTION_REQUIRED_FOR_USEFUL_STATIC_ADMISSION_SET=NO
    D179_LANGUAGE_RESTRICTION_REQUIRED_FOR_UNIVERSAL_UNGUARDED_STATIC_IDENTITY=YES
    REMOVE_SLOT_RESTRICTION_ALONE_SUFFICIENT_FOR_UNIVERSAL_STATIC_IDENTITY=NO

    D179_PARENT_DECISION_READY=YES
    D179_PARENT_DECISION_PERFORMED=NO
    PLAT036_STATE=BLOCKED
    MEASURED_PERFORMANCE_CLAIM=NONE

No source, specification, test, benchmark or implementation change is authorized
by this completion record.
