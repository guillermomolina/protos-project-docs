# D179-B — First-class execution-context reflection and escape boundary audit

FORMAL_IDENTIFIER=D179-B
GITHUB_ISSUE=https://github.com/guillermomolina/protos/issues/705
PARENT=D179 / https://github.com/guillermomolina/protos/issues/703
WORK_STATE=COMPLETE / EVIDENCE PUBLISHED
WORK_KIND=INVESTIGATION / DESIGN EVIDENCE ONLY
PROTOS_REVISION=3e8e6b565c95eb5098c2168d241536ba13ad19e9
FORMAL_IDENTIFIER_UNIQUE=PASS
NATIVE_PARENT_DECLARED=#703
NATIVE_PARENT_RELATION=PASS
IMPLEMENTATION_AUTHORIZED=NO

This is the durable opening record for D179-B. It records an investigation
boundary, not an approved semantic or platform architecture.

## Purpose

D179-B determines what Protos actually requires from the statement that
`context` is a first-class ordinary object, and separates those observable
requirements from physical representation assumptions inherited from the
current runtime.

The core distinction to test is:

    semantic requirement:
      context is observably a first-class Protos object

    possible implementation assumptions:
      the context object must eagerly own every lexical value
      execution and reflection must use the same physical storage
      every ordinary Object capability must operate directly on that storage

Only the first statement is currently taken as an established semantic premise.
The stronger statements must be proven from specification/observable behavior,
not assumed from the existing implementation.

## Required surface

Audit at minimum:

- stable `context` identity;
- escape as a normal Protos value;
- mutation through escaped aliases;
- `hasSlot`, `slotValue`, and `slotNames`;
- explicit sends through `Context -> Object`;
- `without` and `alias`;
- debugger/tooling enumeration and write;
- captured-context observation;
- synchronization requirements between reflection and lexical state;
- compatibility of a semantic object/view/projection with DSL-native locals;
- which observable operations force physical materialization, if any;
- which operations can remain uncommon or slow paths without semantic change.

Lazy physical context materialization is architecture evidence here only.
Selecting such an architecture remains a separate PLAT decision, consistent
with AUD016.

## Required evidence

For every context capability, classify:

    SEMANTIC_REQUIREMENT
    IDENTITY_REQUIRED
    ESCAPE_REQUIRED
    EAGER_PHYSICAL_STORAGE_REQUIRED
    FRAME_LOCAL_PROJECTION_COMPATIBLE
    MATERIALIZATION_TRIGGER
    MUTATION_ALIASING_REQUIREMENT
    DEBUGGER_TOOLING_REQUIREMENT
    DSL_CONSTRAINT
    LANGUAGE_VALUE
    RESTRICTING_CAPABILITY_WOULD_CHANGE_PROTOS
    FINDING
    EVIDENCE

The investigation must explicitly answer whether Protos can preserve the useful
semantics of `context is an Object` while ordinary lexical execution uses
DSL-native frame/local state as its primary physical representation.

## Relationship to D179 and platform work

D179-B feeds parent D179's semantic capability decision and may expose
requirements for a later lazy-context-materialization PLAT. It does not select
that architecture itself.

## Coordination state

The native GitHub Parent/Sub-issue relation has subsequently converged and was
re-read successfully:

    NATIVE_PARENT_RELATION=PASS
    NATIVE_PARENT=#703

The original textual `Parent: #703` declaration remains useful historical
bootstrap prose, but live hierarchy authority is the native relation.

## Completion evidence

D179-B completed the first-class execution-context reflection and escape
boundary audit against the exact product baseline:

    PROTOS_REVISION=3e8e6b565c95eb5098c2168d241536ba13ad19e9
    D179_A_INPUT_REVISION=851c168a30d192a9fa5826536376040e1f523512

The investigation reconstructs the semantic contract separately from the current
`ProtosObjectValue` implementation. No product/spec/test/benchmark change is
authorized or implied by these findings.

### Core result

The strongest semantic requirement is not that lexical execution use the
ordinary-object slot map as its physical backing. It is that every observer
and mutator of one execution context converge on the same semantic identity
and the same binding/state authority.

    CONTEXT_SEMANTIC_IDENTITY_REQUIRED=YES
    CONTEXT_IS_ORDINARY_PROTOS_VALUE=YES
    CONTEXT_ESCAPE_REQUIRED=YES
    CONTEXT_OBJECT_PROTOCOL_REQUIRED=YES
    EAGER_PHYSICAL_CONTEXT_OBJECT_REQUIRED=NO
    OBJECT_BACKED_LEXICAL_VALUES_REQUIRED=NO
    ONE_SEMANTIC_VALUE_AUTHORITY_REQUIRED=YES

`VALUES_AND_COLLECTIONS.md` defines `===` as semantic identity independent of
allocation, boxing, wrappers, proxies, handles, cache entries or other physical
representation. Execution contexts remain individual identity-bearing objects,
but that rule does not require their lexical values to live in one particular
heap layout.

`CALLABLES.md` requires Closures to capture genuine lexical execution contexts
by reference. The observable requirement is therefore shared live authority,
not capture of a Java `ProtosObjectValue` specifically.

### Exact current implementation boundary

At this product revision:

    ProtosActivation.context
      -> ProtosObjectValue

    ProtosActivation.capturedLexicalContexts
      -> List<ProtosObjectValue>

    bare lexical lookup
      -> activation.lookup(name)
      -> current context localSlots
      -> captured context localSlots
      -> receiver/member fallback

    `context` intrinsic
      -> activation.context()

    Object reflection
      -> ProtosObjectValue localSlots

    debugger scope
      -> semantic projection through ProtosActivation

This path is semantically correct, but D179-B found no normative rule requiring
`ProtosObjectValue.localSlots` to remain the physical authority for every
ordinary activation-local binding.

### Stable identity and escape

Within one activation the `context` intrinsic denotes the same semantic
execution-context object. The value may be stored, passed, returned, used as a
delegation parent, or otherwise escape as an ordinary Protos value.

`CALLABLES.md` explicitly covers the hard case in which `context`, or a Closure
capturing it, escapes during parameter binding and the invocation fails later:
the reachable partial activation context remains governed by ordinary
object/capture lifetime rules.

Therefore:

    CONTEXT_ESCAPE_REQUIRES_DURABLE_AUTHORITY=YES
    CONTEXT_ESCAPE_REQUIRES_OBJECT_MAP_BACKING=NO

A frame/local representation is semantically possible only if any escaped
context identity remains a stable view/adapter over that same live authority.

### Reflection

`hasSlot`, `slotValue`, and `slotNames` observe the receiver's semantic local
slot structure only. Delegated slots never satisfy those operations.

A frame/local plus dynamic-overflow representation can preserve these contracts
exactly:

    hasSlot(name)
      -> semantic presence of admitted/static binding
         OR dynamic-overflow membership

    slotValue(name)
      -> exact current value from the same binding authority

    slotNames()
      -> all semantically PRESENT local names
         + dynamic-overflow names
         -> Unicode-scalar lexicographic sort
         -> fresh Array snapshot

`slotNames()` already specifies observation-time snapshot/sort behavior and
explicitly does not require the receiver's physical storage to be sorted or
insertion ordered. Reflection cost therefore does not imply ordinary lexical
access cost.

    REFLECTION_CAN_PROJECT_FRAME_LOCALS=YES
    HAS_SLOT_REQUIRES_EAGER_STORAGE=NO
    SLOT_VALUE_REQUIRES_EAGER_STORAGE=NO
    SLOT_NAMES_REQUIRES_EAGER_STORAGE=NO

`ABSENT` remains distinct from `PRESENT(null)`, so any indexed representation
must carry semantic presence separately from the stored guest value.

    EXPLICIT_SEMANTIC_PRESENCE_REQUIRED=YES
    PRESENT_NULL_DISTINCT_FROM_ABSENT=YES

### Escaped/captured mutation and structural change

D179-A already established the essential split:

    existing-value mutation
      -> changes value/constantness only

    late add/remove
      -> can change presence, nearest binding, binding identity, lexical depth,
         receiver fallback and later assignment destination

    close/freeze
      -> change mutation authority

D179-B confirms that escaped aliases do not change this classification.
Every route must read/write the same semantic authority.

    EXTERNAL_VALUE_MUTATION_REQUIRES_SHARED_AUTHORITY=YES
    EXTERNAL_VALUE_MUTATION_CHANGES_BINDING_IDENTITY=NO
    EXTERNAL_STRUCTURAL_MUTATION_REQUIRES_INVALIDATION=YES
    LATE_ADD_CAN_RETARGET_STATIC_BINDING_IDENTITY=YES
    REMOVE_CAN_RETARGET_STATIC_BINDING_IDENTITY=YES

Structural changes may remain uncommon/slow paths provided they update presence
or dynamic overflow and invalidate any compiled assumptions whose correctness
depends on the previous lexical structure.

### `close()` and `freeze()`

`close()` prevents later structural add/remove while preserving existing-value
writes. `freeze()` rejects both structural and existing-value mutation.
Neither operation changes ordinary read resolution by itself.

    CLOSE_BURDENS_ORDINARY_READS=NO
    FREEZE_BURDENS_ORDINARY_READS=NO
    CLOSE_FREEZE_SHARED_AUTHORITY_REQUIRED=YES

### `without()` and `alias()`

Both operations require a complete semantic local-slot projection only when
they execute. They construct a fresh ordinary open Object and do not retain a
live structural connection to the source.

Therefore they can enumerate frame-backed PRESENT locals plus dynamic overflow
on an observation/slow path and construct their independent result without
requiring eager object-map backing for ordinary lexical execution.

    WITHOUT_REQUIRES_COMPLETE_PROJECTION=YES
    ALIAS_REQUIRES_COMPLETE_PROJECTION=YES
    WITHOUT_REQUIRES_EAGER_CONTEXT_STORAGE=NO
    ALIAS_REQUIRES_EAGER_CONTEXT_STORAGE=NO

### Explicit Object behavior through `Context -> Object`

Explicit member reads, writes, creation, removal, reflection, structural state
operations and use of a context as another object's delegation parent remain
ordinary Protos object behavior.

A semantic context adapter is compatible with that rule if it:

- exposes the stable semantic identity;
- projects semantic local slots from the authoritative lexical backing;
- routes writes to that same authority rather than a copy;
- handles dynamic overflow and presence;
- participates in ordinary `Context -> Object` delegation.

    CONTEXT_OBJECT_PROTOCOL_CAN_PROJECT_FRAME_LOCALS=YES

### Closure capture

Capture-by-reference requires that a Closure observe later value mutation and,
where current semantics permit it, later structural changes in captured
contexts. It does not normatively require `List<ProtosObjectValue>`.

    CAPTURE_REQUIRES_GUEST_CONTEXT_MATERIALIZATION=NO
    CAPTURE_REQUIRES_SHARED_OR_MATERIALIZED_BINDING_AUTHORITY=YES

Materialized frames/shared cells are therefore semantically admissible in
principle; D179-C still decides which binding identities/depths can safely use
that static path.

### Debugger/tooling

`ProtosDebuggerScope` already exposes a semantic scope projection rather than
the context object itself. It enumerates current lexical locals, captured
lexical locals and receiver/delegation names, and reads through ordinary
activation lookup.

The current tooling contract is read-only: debugger scope members are not
modifiable, insertable or removable.

    DEBUGGER_REQUIRES_CONTEXT_OBJECT_IDENTITY=NO
    DEBUGGER_REQUIRES_SEMANTIC_SCOPE_PROJECTION=YES
    DEBUGGER_WRITES_REQUIRED=NO
    DEBUGGER_REQUIRES_MATERIALIZATION=NO

A future DSL-local implementation must hide backend temporaries and reconstruct
the same guest-visible ordering/lookup projection.

### Suspension and resumption

Current Bytecode DSL suspension resumes using the continuation's frame. No
semantic rule snapshots lexical structure at suspension.

A frame-first model is therefore compatible provided continuation lifetime
preserves the same semantic binding/state authority and escaped context aliases
continue to reference it.

    SUSPENSION_EAGER_MATERIALIZATION_REQUIRED=NO
    RESUME_REQUIRES_SAME_SEMANTIC_AUTHORITY=YES

### Persistent/special context categories

Two context categories materially differ from an ordinary ephemeral invocation
activation and must reach D179-C/PLAT036 as separate cases.

Module contexts:

- the module instance is its `moduleContext` identity;
- cache-before-execute exposes that same identity during cyclic initialization;
- partial initialization exposes exactly the currently PRESENT slots;
- the authority must therefore persist beyond one transient body execution.

    MODULE_CONTEXT_PERSISTENT_AUTHORITY_REQUIRED=YES
    MODULE_CONTEXT_PROTOSOBJECTVALUE_LOCAL_MAP_REQUIRED=NO

Object-construction contexts:

- the current slot-creation context is the object being constructed;
- that object is the eventual result and owns ordinary receiver/object state;
- method Closures created there do not capture the constructed object as a
  lexical parent.

    OBJECT_CONSTRUCTION_CONTEXT_REQUIRES_SEPARATE_CLASSIFICATION=YES

The frozen standard prelude is likewise a persistent/frozen lexical-root
category and should not be forced into an activation representation merely for
physical uniformity.

    FROZEN_PRELUDE_REQUIRES_SEPARATE_CLASSIFICATION=YES

### Coherence invariant

The disallowed model is two independently writable authorities:

    frame value/state
      +
    context-object copied value/state

because that can make lexical reads, escaped aliases, Closures, reflection,
debugger observation and continuation resume disagree.

The admissible boundary is:

    ONE_AUTHORITATIVE_STORE_OR_WRITE_THROUGH_AUTHORITY=YES
    EVENTUALLY_SYNCHRONIZED_DIVERGENT_COPIES=NO

Read-only transient snapshots such as the fresh Array returned by `slotNames()`
remain valid because the language explicitly defines them as snapshots.

## Capability classification matrix

| Capability | Identity / escape requirement | Eager physical lexical storage | Frame/local projection | Trigger / slow path | DSL consequence | Finding |
| --- | --- | --- | --- | --- | --- | --- |
| stable `context` identity | stable individual semantic identity; may escape | NO | YES | observation/escape | stable identity must reference same authority | KEEP semantics |
| existing-value alias mutation | same identity/authority | NO | YES | mutation | write-through/shared cell | KEEP semantics |
| late structural creation | same identity; escaped/captured paths allowed | NO | CONDITIONAL | mutation + invalidation | presence/overflow; may retarget lookup | SLOW-PATH PRESERVABLE |
| `hasSlot` | receiver identity only | NO | YES | observation | merge presence + overflow | PROJECTION COMPATIBLE |
| `slotValue` | receiver identity only | NO | YES | observation | exact authority read | PROJECTION COMPATIBLE |
| `slotNames` | receiver identity only | NO | YES | observation/enumeration/sort | enumerate PRESENT + overflow | SLOW-PATH PRESERVABLE |
| `removeSlot` | same receiver; exact removed value | NO | CONDITIONAL | mutation + invalidation | PRESENT -> ABSENT can retarget | SLOW-PATH PRESERVABLE |
| `close` | same identity/state | NO | YES | structural mutation path | create/remove authority guard | SLOW-PATH PRESERVABLE |
| `freeze` | same identity/state | NO | YES | mutation path | all mutation authority guard | SLOW-PATH PRESERVABLE |
| `without` | complete source projection; fresh result identity | NO | YES | observation/copy | no continuing alias to source structure | SLOW-PATH PRESERVABLE |
| `alias` | complete source projection; exact stored-value identity | NO | YES | observation/copy | no continuing structural alias | SLOW-PATH PRESERVABLE |
| explicit context member access | ordinary Object semantics | NO | YES | explicit object path | semantic adapter + delegation | PROJECTION COMPATIBLE |
| Closure capture | live lexical authority by reference | NO guest object requirement | YES | capture/materialization as needed | shared/materialized binding authority | REQUIRES DURABLE BACKING |
| debugger enumeration/read | semantic scope view | NO | YES | tooling | merge guest locals/captures/receiver; hide temps | PROJECTION COMPATIBLE |
| debugger write | not supported by current contract | NO | N/A | never | none | NO MATERIAL EFFECT |
| suspension/resumption | preserve live authority | NO | YES | continuation lifetime | resume same authority | PROJECTION COMPATIBLE |
| module context | persistent module identity and partial state | persistent authority YES; Java map NO | CONDITIONAL | module lifetime | cannot be ephemeral-only | SEPARATE CASE |
| construction context | context is result object state | object authority YES | generally not lexical-local migration | construction | preserve object slots as object state | SEPARATE CASE |

## Materialization trigger classification

    first guest evaluation of `context`
      = OBSERVATION_ONLY

    context return/pass/storage escape
      = ESCAPE

    Closure capture
      = CONDITIONAL state-materialization/lifetime requirement

    hasSlot / slotValue / slotNames
      = OBSERVATION_ONLY

    explicit existing-value write
      = MUTATION

    late slot creation / removeSlot / close / freeze
      = MUTATION

    without / alias
      = OBSERVATION_ONLY

    debugger attachment/enumeration/read
      = TOOLING

    suspension/resumption
      = CONDITIONAL state-lifetime requirement; no independent guest-object trigger

    module cyclic observation
      = ESCAPE / persistent-authority requirement

## Comparative evidence

D179-B reused AUD016's exact mature-runtime revisions and inspected the
material context/frame boundaries directly where relevant:

    TruffleSOM=
      SOM-st/TruffleSOM@73f6d2e654022565ec7c7e8ba95ae18340a862ce

    TruffleSqueak=
      hpi-swa/trufflesqueak@818519b2b6a6556bc524e9e0d08f7b51969cb61a

    TruffleRuby=
      truffleruby/truffleruby@0e6fa6a950dce7154d54f3c9c63056c4eb925ffd

Additional materially different comparison families were used as semantic
counterexamples rather than authority for Protos:

- Smalltalk/Squeak `thisContext`: strong first-class activation/context model;
- Python frame / modern `f_locals`: reflective/write-through frame-local view;
- Ruby `Binding`: first-class retained lexical environment with reflective
  local get/set/enumeration;
- ECMAScript Environment Records / GraalJS: lexical frame/scope representation
  separated from ordinary object properties and tooling projection;
- Lua debug locals/upvalues: execution-state reflection without object-property
  backing;
- Pkl lexical local/property separation where materially comparable.

The transferable observation is only:

    guest-visible semantic context/environment behavior
      can coexist with
    optimized/materialized frame backing

when identity, aliasing, mutation, presence and lifetime remain exact.

No comparison is used to import another language's semantic restrictions into
Protos.

## Attempted falsifications of the no-change model

The semantics-preserving model was tested against:

- repeated `context` identity;
- arbitrary context escape/storage;
- partial activation escape before parameter-binding failure;
- existing-value mutation through escaped aliases;
- late add after capture;
- removal exposing outer/receiver lookup;
- `close` and `freeze`;
- `hasSlot`, `slotValue`, `slotNames`;
- `without` and `alias`;
- explicit Object sends and context-as-parent delegation;
- Closure capture-by-reference;
- debugger enumeration/read;
- suspension/resumption;
- `ABSENT` versus `PRESENT(null)`;
- partially initialized/cyclic module observation;
- object-construction contexts.

No case establishes that all ordinary activation lexical values must remain in
a `ProtosObjectValue.localSlots` map.

## Strongest no-change preservation model

D179-B does not select this architecture, but the strongest model not falsified
by current semantics is:

    STATIC BINDING PLANE
      fixed/indexed local identity where D179-C proves admission
      explicit PRESENT/ABSENT state
      shared/materialized backing where captured

    DYNAMIC STRUCTURAL PLANE
      runtime-introduced names not admitted statically
      structural add/remove

    SEMANTIC CONTEXT ADAPTER
      stable first-class context identity
      Context -> Object behavior
      reflection over both planes
      OPEN/CLOSED/FROZEN authority

    COHERENCE
      one semantic value/presence/state authority
      no independently writable copies

    INVALIDATION
      structural mutations invalidate assumptions about presence, nearest
      binding, lexical depth, receiver fallback and assignment destination

This is architecture evidence only. PLAT036 remains the owner of any concrete
frame/context authority architecture after D179 is ratified.

## Facts delegated to D179-C

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

D179-C must now determine, case by case, which lexical binding identity,
presence and depth facts are genuinely static and which still require dynamic
fallback/invalidation. It must not select PLAT036's physical architecture.

## Performance discipline

No causal performance measurement was performed or inferred.

    SEMANTIC_CONSTRAINT=ESTABLISHED
    ARCHITECTURAL_CONSTRAINT=ESTABLISHED
    MEASURED_PERFORMANCE_COST=NOT_ESTABLISHED
    MEASURED_PERFORMANCE_CLAIM=NONE

## Completion result

    D179_B_RESULT=COMPLETE

    ESSENTIAL_CONTEXT_IS_OBJECT_SEMANTICS=
      STABLE_SEMANTIC_IDENTITY
      ORDINARY_VALUE_ESCAPE
      CONTEXT_TO_OBJECT_DELEGATION
      EXACT_LOCAL_SLOT_REFLECTION
      EXPLICIT_MEMBER_ACCESS_AND_MUTATION
      USE_AS_DELEGATION_PARENT
      CAPTURE_BY_REFERENCE_AUTHORITY
      LIVE_ALIAS_MUTATION
      LIFETIME_WHILE_REACHABLE

    CURRENT_IMPLEMENTATION_ASSUMPTIONS_NOT_PROVEN_SEMANTIC=
      PROTOS_OBJECT_VALUE_AS_ONLY_LEXICAL_BACKING
      LOCAL_SLOTS_MAP_AS_ONLY_STATIC_BINDING_STORE
      STRING_LOOKUP_FOR_EVERY_ORDINARY_LEXICAL_READ
      LIST_OF_PROTOS_OBJECT_VALUE_AS_ONLY_CAPTURE_TRANSPORT
      SAME_JAVA_MAP_FOR_EXECUTION_AND_REFLECTION
      EAGER_OBJECT_MAP_POPULATION_FOR_EVERY_ACTIVATION

    CAPABILITIES_PRESERVABLE_AS_SLOW_PATH=
      CONTEXT_REFLECTION
      EXPLICIT_CONTEXT_MEMBER_ACCESS
      EXPLICIT_CONTEXT_MEMBER_MUTATION
      LATE_STRUCTURAL_CREATION
      REMOVE_SLOT
      CLOSE
      FREEZE
      WITHOUT
      ALIAS
      DEBUGGER_ENUMERATION
      DEBUGGER_READ
      ESCAPED_CONTEXT_STRUCTURAL_MUTATION

    CAPABILITIES_FOR_WHICH_D179_B_PROVES_RESTRICTION_NECESSARY=
      NONE

    D179_PARENT_DECISION_READY=NO
    D179_C_REQUIRED=YES
    PLAT036_STATE=BLOCKED
