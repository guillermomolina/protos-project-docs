# PLAT037 — Lazy execution-context physical materialization

Status: `RATIFIED`

Selected architecture: **Candidate D — defer / keep current eager physical
context creation pending evidence**.

Approval: explicit project-owner approval on 2026-09-25:

~~~text
aprobado
~~~

Decision Issue: `guillermomolina/protos#709`

Origin: `AUD016 / #701`

Product baseline:

~~~text
PROTOS_REVISION=f1cee2d85858804ad3775adf43a9fab97664da2a
PROTOS_VERSION=0.3.87-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.3.4.1

D179_SELECTED_CANDIDATE=C3_MONOTONIC_CONTEXT_MEMBERSHIP
PLAT036_SELECTED_CANDIDATE=D_FRAME_BACKED_SEMANTIC_CONTEXT_ADAPTER
I068_STATUS=COMPLETE
I068_FINAL_PRODUCT_REVISION=f1cee2d85858804ad3775adf43a9fab97664da2a

PLAT037_INVESTIGATION_RATIFICATION_REVISION=
  9418214068f359a4f3f7660072803bc455c6f0a3
~~~

Nature: durable non-normative JVM / Truffle / Bytecode DSL architecture
decision. Observable Protos semantics remain owned by the normative
specification.

## Decision

Protos keeps the current eager physical
`ProtosExecutionContextValue` creation architecture for now.

This is **not** a semantic requirement that the guest-visible Java wrapper must
always be allocated eagerly. PLAT037 establishes that lazy physical
materialization is architecturally possible, but only with a stable
per-invocation semantic identity/state substrate that exists before any guest
wrapper is created.

The viable future lazy architecture is the investigated Candidate B:

~~~text
one semantic invocation
  -> one internal ContextCore identity/state substrate
       -> one lexical authority
            static admitted bindings -> I068 frame-backed authority
            dynamic-only bindings    -> dynamic overflow
       -> OPEN / CLOSED / FROZEN state
       -> lexical topology / parent
       -> optional exact-once ProtosExecutionContextValue wrapper
~~~

That architecture is **not selected for implementation now**.

The selected current boundary is:

~~~text
PLAT037_SELECTED_CANDIDATE=D
CURRENT_EAGER_CONTEXT_ARCHITECTURE=KEEP_FOR_NOW

EAGER_PHYSICAL_ALLOCATION_SEMANTICALLY_REQUIRED=NO
LAZY_CONTEXT_MATERIALIZATION_ARCHITECTURALLY_VALID=CONDITIONAL

CANDIDATE_B_CONTEXTCORE_LAZY_WRAPPER=VALID_FUTURE_OPTION
CANDIDATE_B_IMPLEMENTATION_AUTHORIZED=NO

MEASUREMENT_REQUIRED_BEFORE_LAZY_IMPLEMENTATION=YES
IMPLEMENTATION_OWNER_ALLOCATED=NO
NORMATIVE_SPEC_CHANGE_REQUIRED=NO
~~~

## Preserved semantic invariants

PLAT037 changes no observable Protos behavior.

Any future lazy implementation must preserve at least:

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

RECEIVER_AND_METHOD_HOME_SEMANTICS=PRESERVED
NON_LOCAL_RETURN_AND_DYNAMIC_CONTROL=PRESERVED
SUSPENSION_RESUMPTION=PRESERVED

DEBUGGER_SEMANTIC_SCOPE=PRESERVED
OBJECT_CONSTRUCTION_LEXICAL_BOUNDARY=PRESERVED
MODULE_IDENTITY_AND_INITIALIZATION_RULES=PRESERVED
~~~

## Relationship to PLAT036 and I068

PLAT036 selected frame-backed authority for statically admitted lexical
bindings, with dynamic-only bindings in semantic-context overflow and exactly
one semantic binding-value authority.

I068 implemented that architecture.

Therefore PLAT037 does **not** decide lexical value authority. It decides only
whether the first-class semantic context's physical Java object representation
should be created eagerly or may be projected lazily.

I068 makes lazy projection easier because the wrapper no longer needs to be the
authoritative store for statically admitted lexical values. It does not,
however, eliminate the remaining requirements for stable context identity,
mutation state, dynamic overflow, lexical topology, debugger projection and
exact-one guest object materialization.

## Identity boundary

Current ordinary-object identity uses Java reference identity for ordinary
identity-bearing objects.

A future lazy implementation may preserve that implementation strategy only if:

~~~text
ONE_CONTEXT_CORE_PER_SEMANTIC_INVOCATION=YES
ONE_GUEST_WRAPPER_MAX_PER_CONTEXT_CORE=YES
WRAPPER_RECREATION_AFTER_ESCAPE_OR_RESUME=NO
GLOBAL_SEMANTIC_IDENTITY_REGISTRY=NO
~~~

Materializing multiple independent wrappers for one semantic context is
forbidden.

## Closure, suspension and debugger boundary

A future Candidate-B implementation need not force the guest wrapper merely
because:

- a Closure captures the lexical context;
- a Task suspends or resumes; or
- the debugger enumerates or mutates ordinary semantic lexical scope.

Those operations may retain/project the internal semantic context/authority
substrate directly.

The guest wrapper is required when the actual first-class context value becomes
semantically observable, for example through evaluation of `context` or
ordinary object/reflection behavior on that value.

## Module and object-construction boundary

Object-construction activations are outside the lazy-context transformation:
the object under construction remains the actual creation target.

Module contexts have stronger cache-before-execute and cyclic-import identity
requirements. No future lazy implementation should generalize ordinary
invocation projection to module contexts without proving those invariants
explicitly.

## Performance boundary

PLAT037 makes no claim about current performance materiality.

~~~text
ARCHITECTURAL_VALIDITY != PERFORMANCE_VALUE

LAZY_CORRECTNESS_PLAUSIBILITY=STRONG
LAZY_IMPLEMENTATION_COMPLEXITY=REAL
CURRENT_PROTOS_RUNTIME_BENEFIT=NOT_ESTABLISHED
~~~

The current source eagerly constructs a
`ProtosExecutionContextValue`, its initial map-backed lexical authority and
that authority's `LinkedHashMap` before ordinary Closure/method execution, but
source-level allocation does not prove that all of those allocations survive
Graal partial escape analysis.

Before any Candidate-B implementation is authorized, performance work must
establish what allocation/work actually survives optimization on current
product code and whether the surviving cost is material.

Relevant surfaces include:

~~~text
empty/minimal invocation, context never evaluated
static parameters/locals, context never evaluated
Closure capture, context never evaluated
dynamic local creation, context never evaluated
context evaluated and escaped as control
~~~

Relevant observations include:

~~~text
ProtosExecutionContextValue allocation
ProtosMapBackedLexicalBindingAuthority allocation
LinkedHashMap allocation
ProtosFrameLexicalBindingAuthority allocation
MaterializedFrame / continuation-related state
allocated bytes/op or equivalent allocation counts
throughput
GC pressure where stable
partial-escape / optimized-IR evidence where useful
~~~

No arbitrary success threshold is ratified here.

## Performance-work ownership

PLAT037 does not create a new implementation owner.

Current project sequencing remains:

~~~text
PERF010-A / #691
  -> concrete dominant-cost attribution and causal measurement

PERF011 / #693
  -> systemic runtime-representation-fit evidence

PLAT037 / #709
  -> architecture decision only
~~~

At ratification time PERF010-A already requires a post-I068 current-baseline
remeasurement before selecting another causal intervention.

Therefore PLAT037 does not bypass that gate. The next actionable slice remains
the current post-I068 PERF010-A baseline remeasurement. Only after current
baseline evidence exists should performance work decide whether the
per-invocation physical context-allocation discriminator is the next causal
experiment.

## Migration and reversibility

The selected D boundary keeps both future paths available.

If later evidence justifies Candidate B:

1. return to PLAT037's already-defined Candidate-B invariants;
2. obtain explicit project-owner selection of Candidate B;
3. only then allocate implementation work;
4. migrate activation/Closure/fallback/debugger code toward a separate internal
   context-core abstraction with one lexical authority and exact-one guest
   wrapper.

If the experiment does not establish material surviving cost, no migration is
required.

A future Candidate B remains reversible: eager behavior can be restored by
creating the exact-one wrapper at activation establishment while retaining the
same single authority/core seam.

## Ratification summary

~~~text
PLAT037_SELECTED_CANDIDATE=D
PLAT037_SELECTED_CANDIDATE_NAME=DEFER_KEEP_CURRENT_PENDING_EVIDENCE

CURRENT_EAGER_CONTEXT_ARCHITECTURE=KEEP_FOR_NOW
EAGER_PHYSICAL_ALLOCATION_SEMANTICALLY_REQUIRED=NO

LAZY_CONTEXT_MATERIALIZATION_ARCHITECTURALLY_VALID=CONDITIONAL
CANDIDATE_B_CONTEXTCORE_LAZY_WRAPPER=VALID_FUTURE_OPTION

OBSERVABLE_SEMANTIC_CHANGE_REQUIRED=NO
NEW_IDENTITY_SUBSTRATE_REQUIRED_FOR_CANDIDATE_B=YES

DEBUGGER_FORCES_WRAPPER_MATERIALIZATION=NO_FOR_ORDINARY_SCOPE_PROJECTION
CLOSURE_CAPTURE_FORCES_WRAPPER_MATERIALIZATION=NO
SUSPENSION_FORCES_WRAPPER_MATERIALIZATION=NO

MEASUREMENT_REQUIRED_BEFORE_IMPLEMENTATION=YES
IMPLEMENTATION_READY=NO
IMPLEMENTATION_OWNER_ALLOCATED=NO
SPECIFICATION_CHANGE_REQUIRED=NO

NEXT_ACTION=
  PERF010A_POST_I068_CURRENT_BASELINE_REMEASUREMENT
~~~
