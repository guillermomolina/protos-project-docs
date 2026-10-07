# PERF011 — Final runtime-representation fit audit

Status: **COMPLETE / CLOSURE AUTHORIZED**

This durable, non-normative record closes the investigation-only PERF011 /
guillermomolina/protos#693 runtime-representation fit audit against the current
published Protos product state.

It does not define Protos semantics and does not claim a whole-language
performance improvement. Historical PERF011/PERF010-A checkpoints retain their
original observations; this record supersedes only their earlier
"PERF011_CAN_CLOSE=NO" routing in light of the later accumulated evidence and
published implementation state.

## Evidence identity

~~~text
DATE=2026-09-30
WORK_ITEM=PERF011-FINAL-CLOSURE
TYPE=INVESTIGATION_ONLY

PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=898eb8b2bafe4be99a33032ab0cf6436ce6f3e72
PROTOS_VERSION=0.3.125-SNAPSHOT
PROTOS_COMMIT=I075-D: enable int boxing elimination with current-node lexical authority

GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1

PROJECT_DOCS_REPOSITORY=guillermomolina/protos-project-docs
PROJECT_DOCS_BASELINE_REVISION=3b65602814a48a79bbc700352cc91c96fd5f2330

PERF011_ISSUE=guillermomolina/protos#693
I073_ISSUE=guillermomolina/protos#744
I074_ISSUE=guillermomolina/protos#745
I075_ISSUE=guillermomolina/protos#746

I075_FINAL_RECORD=
docs/project/evidence/I075/I075-D_INT_BOXING_ELIMINATION_CURRENT_NODE_AUTHORITY.md

I075_FINAL_RECORD_REVISION=
dc46b5b01d21e54379272e9b46c5ae7b50ca59ba
~~~

Native GitHub hierarchy was re-read before closure:

~~~text
I073/#744 -> PERF011/#693 PASS
I074/#745 -> PERF011/#693 PASS
I075/#746 -> PERF011/#693 PASS

PERF011_SUB_ISSUES_TOTAL=3
PERF011_SUB_ISSUES_COMPLETED=3
PERF011_SUB_ISSUES_PERCENT_COMPLETED=100
~~~

The current Bytecode DSL root enables:

~~~text
enableMaterializedLocalAccesses=true
enableTailCallHandlers=true
enableUncachedInterpreter=true
boxingEliminationTypes={int.class}
~~~

## Final determination

~~~text
TRUFFLE_FIT_AUDIT=ESTABLISHED
POLYMORPHISM_FIDELITY_AUDIT=COMPLETE

CURRENT_MATERIAL_RUNTIME_FIT_BLOCKER=NONE
FIRST_CAUSAL_EXPERIMENT=NONE
NEW_DECISION_REQUIRED=NO
PERF011_CAN_CLOSE=YES
~~~

"ESTABLISHED" does not mean that every representation difference is materially
expensive. It means the audit did establish concrete semantics-preserving
runtime-representation/compiler-visibility mismatches during its lifetime,
distinguished them from semantic costs, and routed or completed bounded
interventions.

Two findings were directly established rather than merely suspected:

1. Context-owned execution-plan host machinery survived into a failed hot
   partial-evaluation path.
2. A semantically monomorphic call site consumed compiler specialization
   identity through fresh Closure/methodHome object identities, creating
   implementation-induced polymorphism/megamorphism.

Both findings produced bounded semantics-preserving implementation work. The
current product no longer requires PERF011 to remain open merely to seek a
percentage attribution, compare old BGV dumps, migrate wholesale to
DynamicObject/Shape, or exhaust every future representation candidate.

## Semantic costs versus implementation-representation costs

### Semantic costs that remain authoritative

PERF011 does not classify these as optimization defects:

- exact D013 lookup and mutation visibility;
- ordinary shadowing and immutable-parent delegation semantics;
- OPEN/CLOSED/FROZEN behavior;
- current receiver and methodHome provenance;
- fresh Closure identity where guest-observable;
- fresh invocation/execution-context state where guest-observable;
- non-local return, structured control, suspension/resumption, Task and
  dynamic-control behavior;
- arbitrary-precision guest Integer semantics;
- exact override and generic fallback behavior.

### Implementation-representation costs established during PERF011

~~~text
CONTEXT_PLAN_CACHE_HOST_LEAKAGE_INTO_PE=ESTABLISHED
ACCIDENTAL_CALL_SITE_MEGAMORPHISM=ESTABLISHED
SEMANTIC_CHANGE_REQUIRED_FOR_THESE_REPAIRS=NO
~~~

Residual map-backed slots, Array storage, generic fallbacks and rich activation
state are not promoted here to material defects merely because alternative
Truffle representations exist.

## Final representation matrix

### Ordinary method send

~~~text
ABSTRACTION=
  authoritative D013 method send to selected Closure

SEMANTIC_POLYMORPHISM=
  receiver/selector/override/provenance remain dynamically observable

SEMANTIC_STABILITY_FACTS=
  executable CanonicalClosure definition may remain stable while fresh
  Closure/methodHome identities vary

CURRENT_RUNTIME_REPRESENTATION=
  guardedOrdinarySend
  + fastOrdinarySend
  + guardedStructuredSend
  + represented Integer path
  + exact generic fallback
  + compact ordinary-call frame ABI

TRUFFLE_SPECIALIZATION_POLYMORPHISM=
  bounded PICs; stable-definition fast tier separates executable stability
  from fresh guest object identity

CACHE_KEY_FAST=
  selector
  + CanonicalClosure definition
  + entered ProtosLanguageContext

CACHE_KEY_GUARDED=
  exact receiver
  + selector
  + entered Context
  + lookup Assumption

CACHE_LIMIT=3

INVALIDATION_MODEL=
  selector-specific Assumption invalidation on admitted mutable ordinary
  lookup dependencies

HOST_IMPLEMENTATION_VISIBLE_AFTER_PE=INCONCLUSIVE_AT_CURRENT_HEAD
POLYMORPHISM_FIDELITY=PARTIALLY_REPAIRED
ACCIDENTAL_MEGAMORPHISM=PREVIOUSLY_ESTABLISHED_NOW_REPAIRED
SEMANTIC_CHANGE_REQUIRED_TO_IMPROVE=NO
CURRENT_ACTION=COMPLETE
NEXT_BOUNDED_EXPERIMENT=NONE
~~~

The historical host-leakage result remains established. A new compiler-graph
campaign is not required solely to turn the current-head visibility cell into
YES or NO.

### Closure call

~~~text
ABSTRACTION=
  direct invocation of a Protos Closure through ordinary call semantics

SEMANTIC_POLYMORPHISM=
  Closure guest identity may legitimately be fresh

SEMANTIC_STABILITY_FACTS=
  CanonicalClosure executable definition may remain stable across fresh values

CURRENT_RUNTIME_REPRESENTATION=
  guardedDirect exact-receiver tier
  + fastDirect CanonicalClosure-definition tier
  + generic fallback

CACHE_KEY_FAST=
  CanonicalClosure definition
  + entered ProtosLanguageContext

CACHE_LIMIT=3

INVALIDATION_MODEL=
  authoritative call-slot selection remains observable;
  guarded tier uses lookup Assumption

POLYMORPHISM_FIDELITY=MATCH
ACCIDENTAL_MEGAMORPHISM=NOT_ESTABLISHED
SEMANTIC_CHANGE_REQUIRED_TO_IMPROVE=NO
CURRENT_ACTION=KEEP_CURRENT
NEXT_BOUNDED_EXPERIMENT=NONE
~~~

The final relevant distinction is:

~~~text
fresh Closure identity != unstable executable identity
~~~

### Ordinary slot read

~~~text
ABSTRACTION=
  ordinary member read with D013 delegation

CURRENT_RUNTIME_REPRESENTATION=
  ReadMember
  -> ProtosValueLookup.readMember
  -> String-keyed delegated lookup
  -> map-backed ordinary local-slot authority

TRUFFLE_SPECIALIZATION_POLYMORPHISM=
  no dedicated ordinary-read PIC at this operation

INVALIDATION_MODEL=
  authoritative generic lookup;
  selector-specific Assumptions exist for guarded consumers

HOST_IMPLEMENTATION_VISIBLE_AFTER_PE=INCONCLUSIVE
POLYMORPHISM_FIDELITY=INCONCLUSIVE
ACCIDENTAL_MEGAMORPHISM=NOT_ESTABLISHED
SEMANTIC_CHANGE_REQUIRED_TO_IMPROVE=NO

CURRENT_ACTION=FUTURE_CANDIDATE

NEXT_BOUNDED_EXPERIMENT=
  only if future evidence selects ordinary slot reads as a material hotspot:
  one constant-selector guarded read specialization using existing
  selector-specific Assumption invalidation and exact generic fallback
~~~

A DynamicObject/Shape migration remains a technically plausible future
representation choice, not a current PERF011 requirement.

### Ordinary slot write

~~~text
ABSTRACTION=
  assignment to an existing local slot

CURRENT_RUNTIME_REPRESENTATION=
  AssignLocalSlot
  -> ProtosObjectValue.assignLocalSlot
  -> lexical-binding authority
  -> exact-selector dependency invalidation

TRUFFLE_SPECIALIZATION_POLYMORPHISM=
  no dedicated write PIC

INVALIDATION_MODEL=
  mutation invalidates only dependencies for the exact selector

HOST_IMPLEMENTATION_VISIBLE_AFTER_PE=INCONCLUSIVE
POLYMORPHISM_FIDELITY=INCONCLUSIVE
ACCIDENTAL_MEGAMORPHISM=NOT_ESTABLISHED
SEMANTIC_CHANGE_REQUIRED_TO_IMPROVE=NO

CURRENT_ACTION=FUTURE_CANDIDATE

NEXT_BOUNDED_EXPERIMENT=
  only if selected by future evidence:
  cache one constant-name exact ordinary-object write while preserving
  mutation-state checks, presence checks and selector invalidation
~~~

### Delegated lookup

~~~text
ABSTRACTION=
  ordinary D013 local/inherited selection

SEMANTIC_STABILITY_FACTS=
  delegation parent is immutable;
  FROZEN exact ordinary objects are stable;
  canonical Boolean and Integer represented parent steps are stable

CURRENT_RUNTIME_REPRESENTATION=
  generic lookup
  + lookupGuarded for admitted ordinary chains
  + lookupGuardedCanonicalBoolean
  + lookupGuardedInteger

TRUFFLE_SPECIALIZATION_POLYMORPHISM=
  guarded paths cache selection outside compiled lookup establishment;
  generic path remains authoritative elsewhere

INVALIDATION_MODEL=
  exact-selector Assumption shared across all visited ordinary dependencies

POLYMORPHISM_FIDELITY=MATCH
ACCIDENTAL_MEGAMORPHISM=NOT_ESTABLISHED
SEMANTIC_CHANGE_REQUIRED_TO_IMPROVE=NO
CURRENT_ACTION=KEEP_CURRENT
NEXT_BOUNDED_EXPERIMENT=NONE
~~~

The initial audit's absence of a general Shape/version mechanism remains true
globally, but the current product does have precise selector-specific
Assumption invalidation for the chains it admits to guarded specialization.

### Object construction

~~~text
ABSTRACTION=
  object-construction body execution producing one fresh object

SEMANTIC_STABILITY_FACTS=
  body RootCallTarget may remain stable independently of fresh result identity

CURRENT_RUNTIME_REPRESENTATION=
  EnterObjectConstruction direct-call specialization
  + IndirectCallNode fallback

CACHE_KEY=
  prepared.bodyTarget()

CACHE_LIMIT=3

POLYMORPHISM_FIDELITY=MATCH
ACCIDENTAL_MEGAMORPHISM=NOT_ESTABLISHED
SEMANTIC_CHANGE_REQUIRED_TO_IMPROVE=NO
CURRENT_ACTION=KEEP_CURRENT
NEXT_BOUNDED_EXPERIMENT=NONE
~~~

Fresh constructed-object identity is not used as the direct-call cache key.

### Integer / arithmetic primitive dispatch

~~~text
ABSTRACTION=
  ordinary message dispatch on semantic Integer values

SEMANTIC_STABILITY_FACTS=
  all ProtosIntegerValue values for one prelude delegate through the same
  Integer prototype

CURRENT_RUNTIME_REPRESENTATION=
  semantic ProtosIntegerValue remains the guest representation;
  lookupGuardedInteger admits the entire Integer representation family;
  guardedIntegerSend caches canonical native selection

CACHE_KEY=
  selector
  + prelude
  + entered ProtosLanguageContext

CACHE_LIMIT=3

INVALIDATION_MODEL=
  Assumption over mutable ordinary prototype-chain dependencies

POLYMORPHISM_FIDELITY=MATCH
ACCIDENTAL_MEGAMORPHISM=NOT_ESTABLISHED
SEMANTIC_CHANGE_REQUIRED_TO_IMPROVE=NO
CURRENT_ACTION=COMPLETE
NEXT_BOUNDED_EXPERIMENT=NONE
~~~

The I075 carrier does not narrow guest Integer semantics:

~~~text
boxingEliminationTypes={int.class}
GUEST_INTEGER_REPRESENTATION=UNCHANGED_ARBITRARY_PRECISION
INT_CARRIER_ROLE=IMPLEMENTATION_INTERNAL_METADATA
~~~

### Array / indexed access

~~~text
ABSTRACTION=
  standard indexed protocol reached through ordinary Protos sends

SEMANTIC_POLYMORPHISM=
  indexing remains message/protocol based and overrideable

CURRENT_RUNTIME_REPRESENTATION=
  ProtosArrayValue
  -> ArrayList<Object>
  -> BigInteger index validation/conversion

TRUFFLE_SPECIALIZATION_POLYMORPHISM=
  no independently established Array-index PIC mismatch

HOST_IMPLEMENTATION_VISIBLE_AFTER_PE=INCONCLUSIVE
POLYMORPHISM_FIDELITY=INCONCLUSIVE
ACCIDENTAL_MEGAMORPHISM=NOT_ESTABLISHED
SEMANTIC_CHANGE_REQUIRED_TO_IMPROVE=NO

CURRENT_ACTION=FUTURE_CANDIDATE

NEXT_BOUNDED_EXPERIMENT=
  if indexed access becomes materially hot:
  guarded canonical standard Array at/atPut selection with exact override
  fallback before considering storage migration
~~~

No current evidence requires replacing ArrayList storage or turning guest
indexing into a privileged runtime operation.

### Activation / lexical access

~~~text
ABSTRACTION=
  invocation state and lexical binding access

SEMANTIC_STABILITY_FACTS=
  statically admitted binding owner, lexical depth and frame ordinal can be
  proven at lowering time;
  captured owner identity can remain stable

CURRENT_RUNTIME_REPRESENTATION=
  compact invocation frame ABI
  + lazy rich ProtosActivation materialization
  + BytecodeLocal/LocalAccessor current-root access
  + MaterializedLocalAccessor captured access
  + single ProtosFrameLexicalBindingAuthority
  + precomputed ProtosFrameLexicalLayout
  + exact String-keyed fallback/dynamic overflow

INVALIDATION_OR_FALLBACK_MODEL=
  Bytecode DSL local presence/cleared state
  + nearer-context presence checks
  + exact generic fallback
  + declaringRoot.getBytecodeNode() for frame-backed authority access

POLYMORPHISM_FIDELITY=MATCH
ACCIDENTAL_MEGAMORPHISM=NOT_ESTABLISHED
SEMANTIC_CHANGE_REQUIRED_TO_IMPROVE=NO
CURRENT_ACTION=COMPLETE
NEXT_BOUNDED_EXPERIMENT=NONE
~~~

I075-D closes the concrete current-node lifetime mismatch exposed when boxing
elimination was enabled: frame lexical authority retains stable declaring-root
identity and resolves the root's current BytecodeNode rather than preserving an
installation-time tier-specific node.

## Completed adoptions and repairs

The current product incorporates the following PERF011-relevant representation
work accumulated during the audit:

~~~text
frame-backed lexical representation
Bytecode DSL locals for statically admitted lexical bindings
MaterializedLocalAccessor captured-local access
single frame/context lexical-binding authority
captured binding state preserved by reference
lexical-layout precomputation
compact/lazy invocation state
guarded ordinary selector/call specialization
stable CanonicalClosure executable cache identity
guarded structured-send specialization
canonical Boolean represented selection
semantic Integer-family represented selection
selector-specific lookup Assumption invalidation
current-BytecodeNode lexical authority
enableTailCallHandlers=true
enableUncachedInterpreter=true
boxingEliminationTypes={int.class}
~~~

The AUD016 EXPERIMENT_FIRST rebaseline children are all closed:

~~~text
I073/#744=CLOSED
I074/#745=CLOSED
I075/#746=CLOSED
~~~

## Historical guarded-call timing result

The bounded guarded post-D013 bypass experiment remains valid historical
evidence:

~~~text
CALL_SITE_SPECIALIZATION_EXPERIMENT=VALID
CALL_SITE_TIMING_RESULT=NEGATIVE
COMPILER_VISIBILITY_CHANGED=INCONCLUSIVE
PRODUCTION_GUARDED_BYPASS_JUSTIFIED=NO
~~~

That experiment was correctly not selected as a production optimization. Its
negative result does not invalidate the independently established compiler
visibility and polymorphism-fidelity mismatches, nor does it require a new BGV
comparison for PERF011 closure.

## Remaining independent candidates

The following are retained as ordinary future candidates only:

~~~text
CANDIDATE_1=
  ordinary object slot/read-write representation and optional
  Shape/DynamicObject specialization

CANDIDATE_2=
  Array/indexed-access representation/specialization

CANDIDATE_3=
  residual activation scalarization/deferred materialization

CANDIDATE_4=
  broader primitive carriers beyond existing int metadata
~~~

Classification:

~~~text
FOLLOWUP_TYPE=BACKLOG_ONLY
REQUIRED_NEW_PERF_ISSUE=NO
REQUIRED_NEW_I_ISSUE=NO
REQUIRED_NEW_PLAT=NO
~~~

If later profiling, regression evidence or an architecture decision makes one
independently actionable, it should receive its own appropriate work identity
then. PERF011 does not remain open merely to reserve those possibilities.

## Closure gates

~~~text
ISSUE_CLOSURE_EVIDENCE=THIS_RECORD
REVISION_BOUND_PRODUCT_STATE=PASS

CALL_SITE_MISMATCH_IDENTIFIED=PASS
COMPILER_VISIBILITY_MISMATCH_IDENTIFIED=PASS
SEMANTIC_VS_IMPLEMENTATION_COST_SEPARATION=PASS
POLYMORPHISM_FIDELITY_MATRIX=PASS
COMPLETED_ADOPTIONS_RECONCILED=PASS
REMAINING_CANDIDATES_CLASSIFIED=PASS

PERF020_REQUIRED_FOR_CLOSURE=NO
PROTOS_BENCHMARKS_REQUIRED_FOR_CLOSURE=NO
BGV_COMPARISON_REQUIRED_FOR_CLOSURE=NO
BROAD_SHAPE_MIGRATION_REQUIRED_FOR_CLOSURE=NO
NEW_DYNAMIC_MEASUREMENT_REQUIRED_FOR_CLOSURE=NO
NEW_DECISION_REQUIRED=NO

PERF011_CAN_CLOSE=YES
~~~

## Retained predecessor evidence

This final record preserves rather than rewrites the historical evidence in:

- PERF011_RUNTIME_REPRESENTATION_FIT_AUDIT.md
- PERF011_GUARDED_CALL_DISCRIMINATION_CHECKPOINT.md
- PERF011_LIGHTWEIGHT_REACTIVATION_AND_25_4_REBASELINE.md
- PERF011_AUD016_EXPERIMENT_FIRST_25_4_REBASELINE.md
- I075-D_INT_BOXING_ELIMINATION_CURRENT_NODE_AUTHORITY.md

Their earlier "PERF011_CAN_CLOSE=NO" conclusions remain correct for the product
and evidence state they documented. This record supplies the later final
closure determination.
