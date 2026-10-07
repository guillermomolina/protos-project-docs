# D179-A — Execution-context structural mutation capability audit

FORMAL_IDENTIFIER=D179-A
GITHUB_ISSUE=https://github.com/guillermomolina/protos/issues/704
PARENT=D179 / https://github.com/guillermomolina/protos/issues/703
WORK_STATE=COMPLETE / EVIDENCE PUBLISHED
WORK_KIND=INVESTIGATION / DESIGN EVIDENCE ONLY
PROTOS_REVISION=3e8e6b565c95eb5098c2168d241536ba13ad19e9
FORMAL_IDENTIFIER_UNIQUE=PASS
NATIVE_PARENT_DECLARED=#703
NATIVE_PARENT_RELATION=PASS
IMPLEMENTATION_AUTHORIZED=NO

This is the durable opening record for D179-A. It does not select or ratify a
language candidate and does not authorize specification, runtime, test, or
benchmark changes.

## Purpose

D179-A audits every structural mutation or mutation-authority capability of a
live Protos execution context that could constrain faithful use of Truffle
Bytecode DSL locals/materialized locals.

The investigation exists because the first D179 packet concentrated too heavily
on `removeSlot`. Structural removal remains an important stress case, but it is
not assumed to be the only capability with material DSL consequences.

## Required surface

The investigation must treat independently:

- `context.removeSlot(name)`;
- late `context.name: value` creation;
- creation through escaped context references;
- creation through captured contexts after Closure creation;
- mutation of existing context-local slots through escaped/captured aliases;
- `context.close()`;
- `context.freeze()`;
- the effect of each operation on later bare read, assignment and creation;
- capture, suspension/resumption and debugger observation;
- `ABSENT` versus `PRESENT(null)`.

The audit must distinguish activation-local late creation from structural changes
performed through another escaped or captured path. Similar source effects do
not imply equal compiler/runtime constraints.

## Required evidence

For every material capability, classify:

    CURRENT_SEMANTICS
    REAL_REPOSITORY_USE
    LANGUAGE_VALUE
    STATIC_FACTS_INVALIDATED
    DSL_NATIVE_PATH_CONSTRAINED
    PARALLEL_RUNTIME_MACHINERY_REQUIRED
    SLOW_PATH_PRESERVATION_POSSIBLE
    USER_VISIBLE_CHANGE_IF_RESTRICTED
    OWNER_APPROVAL_REQUIRED
    FINDING
    EVIDENCE

Measured performance evidence must remain separate from architectural or
optimization pressure. No speedup may be claimed without measurement.

## Relationship to D179 and PLAT036

D179-A owns research evidence only. Parent D179 remains the sole owner of the
final execution-context semantic capability decision.

PLAT036 / #702 remains blocked by D179. Closing D179-A alone cannot release that
blocker.

## Coordination state

The live native GitHub hierarchy was re-read after the investigation. D179-A /
#704 has native parent D179 / #703, and #703 reports the three D179-A/B/C
children. Therefore:

    NATIVE_PARENT_RELATION=PASS

The original textual `Parent: #703` declaration remains historical/bootstrap
prose; live hierarchy authority is the native relation.


## Completion evidence

D179-A completed the requested structural-mutation capability audit against:

    PROTOS_REVISION=3e8e6b565c95eb5098c2168d241536ba13ad19e9

No specification, runtime, test, benchmark, or language decision was changed.
This section records research evidence only for parent D179.

### Executive result

The audit separates three materially different kinds of execution-context
mutation:

    VALUE MUTATION
      PRESENT(v1) -> PRESENT(v2)

    STRUCTURAL MEMBERSHIP MUTATION
      ABSENT -> PRESENT
      PRESENT -> ABSENT

    MUTATION-AUTHORITY TRANSITION
      OPEN -> CLOSED
      OPEN/CLOSED -> FROZEN

These classes do not invalidate the same facts and should not be represented as
one generic "mutable context" constraint.

The central correction to the first D179 packet is:

    REMOVE_SLOT_UNIQUELY_PROBLEMATIC=NO

`removeSlot` is unique among the audited capabilities because it permits
`PRESENT -> ABSENT`, but it is not unique in its ability to change which
binding a later bare name resolves to. Late creation in a nearer escaped or
captured context can perform the opposite transition, `ABSENT -> PRESENT`,
and thereby shadow a previously selected outer lexical binding or receiver
fallback.

Therefore historical E1 ("execution contexts may grow but not shrink") would
remove removal-driven retargeting but would not, by itself, make every lexical
binding identity static.

### Exact current implementation boundary

Current lowering still represents ordinary guest lexical reads as:

    CanonicalLookup(name)
      -> Lookup(ProtosActivation, String)
      -> ProtosActivation.lookup(name)
      -> current context local slot
      -> captured lexical context local slots
      -> receiver/member fallback

Bare assignment currently resolves the writable lexical context before RHS
evaluation and then writes the pinned destination. Bare creation targets the
current execution context and performs ordinary local-slot creation.

The runtime object model distinguishes:

    createLocalSlot:
      rejects CLOSED and FROZEN
      changes membership ABSENT -> PRESENT

    assignLocalSlot:
      permits CLOSED
      rejects FROZEN
      preserves membership

    removeLocalSlot:
      rejects CLOSED and FROZEN
      changes membership PRESENT -> ABSENT

    close:
      OPEN -> CLOSED
      preserves membership and existing-value mutability

    freeze:
      -> FROZEN
      preserves membership and rejects later value/structural mutation

### Capability findings

#### 1. context.removeSlot(name)

Current semantics:
- removes one local slot from an OPEN context;
- returns the exact removed value;
- later bare lookup observes the resulting absence and continues outward.

Material static facts invalidated:
- binding presence;
- nearest lexical binding;
- binding identity;
- lexical depth;
- context shape;
- receiver-fallback reachability;
- later bare-assignment destination.

Repository value/use:
- no guest production caller of `context.removeSlot(...)` was found;
- general Object.removeSlot remains an intentional ordinary-object capability;
- the strongest KEEP value is first-class/object-model uniformity and
  Self-style reflective activation manipulation.

Result:

    FINDING=RESTRICT_CANDIDATE
    SLOW_PATH_PRESERVATION_POSSIBLE=CONDITIONAL
    OWNER_APPROVAL_REQUIRED=YES

A no-change implementation remains plausible by guarding or invalidating
compiled assumptions that depend on an established binding remaining present.

#### 2. Bare/current-context late creation

Current semantics:
- `x: value` performs no lookup and creates on the current slot-creation
  context;
- in a Closure activation it is conceptually equivalent to
  `context.x: value`;
- source bindings can therefore be semantically absent before their creation
  point.

This does not imply that physical binding identity must be dynamic. A fixed
indexed/frame local may exist physically while carrying semantic
ABSENT/uninitialized state until establishment.

Result:

    FINDING=KEEP_CANDIDATE
    SLOW_PATH_PRESERVATION_POSSIBLE=YES

The key distinction is:

    PHYSICAL_SLOT_EXISTS != SEMANTIC_BINDING_PRESENT

#### 3. Creation through an escaped context reference

This is a material architectural constraint distinct from ordinary local
creation.

If a nearer context currently lacks `x`, a bare read may resolve to an outer
`x` or receiver member. Arbitrary code holding an escaped reference to the
nearer OPEN context can later create `x`, after which subsequent bare lookup
must resolve to that newly nearer binding.

Therefore escaped late creation can invalidate:
- absence;
- nearest binding;
- binding identity;
- lexical depth;
- context shape;
- receiver-fallback choice;
- later bare-assignment destination.

Result:

    FINDING=MOVE_TO_SLOW_PATH_CANDIDATE
    SLOW_PATH_PRESERVATION_POSSIBLE=YES

A preservation model may use structural version/assumption invalidation plus a
dynamic overflow/presence representation for runtime-introduced names.

#### 4. Creation through a captured context after Closure creation

Closures capture lexical execution contexts by reference. Later creation in an
OPEN captured context is intentionally visible to subsequent lookup through the
Closure.

This can invalidate the same resolution facts as escaped late creation when
the newly-created name is in a nearer captured context than the binding
previously selected.

Result:

    FINDING=MOVE_TO_SLOW_PATH_CANDIDATE
    SLOW_PATH_PRESERVATION_POSSIBLE=YES

The important conclusion is that captured-context growth can invalidate static
binding identity, not only static presence.

#### 5. Existing-value mutation through current/escaped/captured aliases

Changing the value of an already-established binding does not change:
- binding existence;
- binding identity;
- nearest lexical binding;
- lexical depth;
- context shape.

It invalidates value/constantness facts only.

Result:

    FINDING=NO_MATERIAL_DSL_EFFECT
    SLOW_PATH_PRESERVATION_POSSIBLE=YES

This is compatible with direct current-local stores and materialized/cell-like
captured stores, provided one coherent semantic value authority is maintained.

#### 6. context.close()

`close()` changes mutation authority but not membership:

    existing reads              allowed
    existing-value assignment   allowed
    create                      rejected
    remove                      rejected

It therefore does not require generic lexical lookup on ordinary reads or
existing writes. Creation/removal needs an OPEN-state check when openness is not
otherwise known.

Result:

    FINDING=MOVE_TO_SLOW_PATH_CANDIDATE
    SLOW_PATH_PRESERVATION_POSSIBLE=YES

No semantic restriction is justified by D179-A evidence.

#### 7. context.freeze()

`freeze()` preserves membership while rejecting subsequent existing-value
writes and structural mutation.

It does not materially burden reads. Mutation paths require a frozen-state
guard or invalidation when frozen state may change through another alias.

Result:

    FINDING=KEEP_CANDIDATE
    SLOW_PATH_PRESERVATION_POSSIBLE=YES

Freeze has independent publication/isolation value and is not merely lexical
representation baggage.

### Interaction findings

Later bare read:
- removal can reveal an outer/receiver binding;
- addition can introduce a nearer binding;
- existing-value mutation changes only the returned value;
- close/freeze do not themselves redirect reads.

Later bare assignment:
- membership topology before destination selection determines the destination;
- once selected, the destination remains pinned across RHS effects;
- close still permits writing an existing destination;
- freeze can make the final write fail.

Receiver fallback:
- add/remove can change whether fallback is reached;
- value mutation/close/freeze cannot do so by themselves.

Suspension/resumption:
- no semantic rule snapshots lexical structure at suspension;
- a frame/object implementation must therefore avoid divergent authoritative
  copies across suspension.

Debugger/tooling:
- current ProtosDebuggerScope reads/enumerates context objects directly;
- a future frame-backed representation therefore needs a semantic scope adapter
  over guest locals, materialized outers, dynamic overflow, and receiver
  fallback;
- this coherence requirement does not prove heap-object lookup must remain the
  ordinary execution path.

### Facts invalidated by opaque guest calls

D179-A refines the current broad conservative barrier used by
ProtosStaticDefinitions.

Existing-value assignment can invalidate:

    value
    constantness

Late structural add/remove can additionally invalidate:

    presence/absence
    nearest lexical binding
    binding identity
    lexical depth
    context shape
    receiver-fallback decision
    later assignment destination

close can invalidate:

    can-create
    can-remove
    OPEN state

freeze can invalidate:

    can-create
    can-remove
    can-assign
    writable state

This capability-specific model is more precise than treating every opaque call
as destroying every lexical fact for the same reason.

### No-change preservation model

D179-A found no semantic proof that current behavior requires every lexical
access to remain generic String-keyed object lookup.

A plausible semantics-preserving architecture for later PLAT036 analysis is:

    STATIC BINDING PLANE
      fixed/indexed local identity for admitted bindings
      explicit semantic presence state where required
      materialized locals for admitted captured bindings

    DYNAMIC CONTEXT PLANE
      runtime-introduced-name overflow
      structural version / assumption
      OPEN/CLOSED/FROZEN authority state

    SEMANTIC CONTEXT ADAPTER
      stable first-class context identity
      coherent access to both planes
      no divergent authoritative copies

    STRUCTURAL MUTATION
      update presence/overflow
      invalidate only assumptions that depend on changed structure

    TOOLING
      project semantic locals/materialized outers/overflow/fallback

This is architecture evidence, not a PLAT036 selection.

### Smallest plausible restrictions

D179-A ratifies none of these restrictions.

For removal, the smallest identified restriction remains:

    reject PRESENT -> ABSENT while an object serves as an execution-context
    identity

This would remove deletion-driven lexical retargeting.

However it does not remove growth-driven retargeting. Arbitrary escaped or
captured aliases can still create a nearer same-name binding.

A different possible restriction would constrain who may structurally grow a
live execution context, while preserving ordinary activation-local creation.
That would be a larger language decision and cannot be recommended before
D179-B determines the actual value/requirements of first-class context escape
and reflection.

### Required cross-capability answers

    REMOVE_SLOT_UNIQUELY_PROBLEMATIC=NO

Reason:
- uniquely permits PRESENT -> ABSENT;
- not uniquely capable of changing nearest binding identity.

    LATE_CAPTURED_GROWTH_CAN_INVALIDATE_STATIC_IDENTITY=YES

Reason:
- a nearer captured context may gain a same-name binding and shadow the
  previously selected outer binding.

    ESCAPED_MUTATION_INVALIDATES_MORE_THAN_LOCAL_VALUE_MUTATION=YES

Additional facts include:
- presence;
- shape;
- nearest binding;
- binding identity;
- lexical depth;
- mutation authority.

    CLOSE_BURDENS_ORDINARY_READS=NO
    CLOSE_BURDENS_EXISTING_WRITES=NO
    CLOSE_REQUIRES_CREATE_REMOVE_STATE_CHECK=YES

    FREEZE_BURDENS_ORDINARY_READS=NO
    FREEZE_REQUIRES_MUTATION_STATE_CHECK=YES

    STRUCTURAL_MUTATION_ARCHITECTURALLY_DISTINCT_FROM_VALUE_MUTATION=YES

### Relationship to D179-B and D179-C

D179-B is still required to determine which first-class context properties
actually require eager/object-backed physical storage versus a semantic
projection over DSL-native state.

D179-C is still required to classify every lexical case by:
- static binding identity;
- static presence;
- static lexical depth;
- direct DSL-local admissibility;
- genuine dynamic fallback requirement.

D179-C must consume this specific D179-A correction:

    removing PRESENT -> ABSENT is not the only source of lexical retargeting;
    ABSENT -> PRESENT in a nearer escaped/captured context can also retarget a
    later bare lookup.

Parent D179 remains the sole decision owner and must rebuild its candidate set
after D179-B and D179-C.

### Performance discipline

No causal performance measurement was performed or inferred.

    SEMANTIC_CONSTRAINT=ESTABLISHED
    ARCHITECTURAL_CONSTRAINT=ESTABLISHED
    OPTIMIZATION_BARRIER=ESTABLISHED_FOR_SPECIFIC_CASES
    MEASURED_PERFORMANCE_COST=NOT_ESTABLISHED

### Completion result

    D179_A_RESULT=COMPLETE

    REMOVE_SLOT_UNIQUELY_PROBLEMATIC=NO

    OTHER_MATERIAL_CONTEXT_CAPABILITIES=
      ESCAPED_CONTEXT_LATE_CREATION
      CAPTURED_CONTEXT_LATE_CREATION
      CONTEXT_CLOSE_MUTATION_AUTHORITY
      CONTEXT_FREEZE_MUTATION_AUTHORITY

    CAPABILITIES_WITH_NO_MATERIAL_DSL_EFFECT=
      EXISTING_LOCAL_VALUE_MUTATION
      EXISTING_CAPTURED_LOCAL_VALUE_MUTATION

    CAPABILITIES_PRESERVABLE_AS_SLOW_PATH=
      REMOVE_SLOT_CONDITIONAL
      EXPLICIT_CURRENT_CONTEXT_LATE_CREATION
      ESCAPED_CONTEXT_LATE_CREATION
      CAPTURED_CONTEXT_LATE_CREATION
      CONTEXT_CLOSE
      CONTEXT_FREEZE

    CAPABILITIES_REQUIRING_PARENT_D179_RECONSIDERATION=
      REMOVE_SLOT
      ARBITRARY_ESCAPED_CONTEXT_STRUCTURAL_GROWTH
      ARBITRARY_CAPTURED_CONTEXT_STRUCTURAL_GROWTH

    MEASURED_PERFORMANCE_CLAIM=NONE

    D179_PARENT_DECISION_READY=NO
    D179_B_REQUIRED=YES
    D179_C_REQUIRED=YES
    PLAT036_STATE=BLOCKED
