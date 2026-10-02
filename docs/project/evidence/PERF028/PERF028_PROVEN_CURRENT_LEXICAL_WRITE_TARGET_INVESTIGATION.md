# PERF028 — Proven current lexical write-target investigation

Status: **RETAINED / STATIC INVESTIGATION COMPLETE**
Date: 2026-10-02
Formal owner: `PERF028 / guillermomolina/protos#780`

This is durable, non-normative performance evidence. It records the exhaustive
follow-up to PERF025's `PROVEN_LEXICAL_WRITE_RUNTIME_TARGET` finding.

It does not change observable Protos semantics, modify product source, approve a
new lexical-state architecture, or claim runtime allocation/timing magnitude
that was not measured.

## Identity

```text
PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=d1aeea403cc7f7e5ffa7289006ceb072b025ca44
PROTOS_VERSION=0.3.148-SNAPSHOT
PROTOS_SUBJECT=PERF027-A: deferred activation for guarded Integer native sends

FORMAL_WORK=PERF028
GITHUB_ISSUE=guillermomolina/protos#780

TRIGGER=PERF025/#758 post-F1 pay-as-you-grow audit
AUDIT_FINDING=P1 PROVEN_LEXICAL_WRITE_RUNTIME_TARGET

RELATED_HISTORY=
  PLAT036/#702
  I068/#708
  PERF013/#724
```

The detailed lexical audit began against
`c8e0e0d59d5541007d733e8a0b53a3cf123e5818`. Before publication it was
reconciled against current product HEAD
`d1aeea403cc7f7e5ffa7289006ceb072b025ca44`.

The intervening PERF027-A product delta changes guarded Integer invocation
preparation. Current HEAD was re-read at the exact lexical assignment,
binding-analysis and frame-local operation sections. The PERF028 findings below
remain live.

## Question

The investigated question was:

> For an unqualified assignment whose current lexical binding identity is already
> statically proven, why does Protos still resolve and construct a general runtime
> write target on every execution, and how far can that machinery be removed
> without weakening D179 C0 or destination-before-RHS semantics?

Result:

```text
FINDING_CONFIRMED=YES
BOUNDED_IMPLEMENTATION_CANDIDATE=YES
NEW_SEMANTIC_DECISION_REQUIRED=NO
NEW_PLATFORM_DECISION_REQUIRED=NO
```

The correct optimization is **not** an unconditional raw `StoreLocal`.

The correct bounded direction is:

```text
Resolved current lexical identity
  -> constant LocalAccessor
  -> pre-RHS presence selection
  -> evaluate RHS exactly once
  -> post-RHS revalidation of the exact selected destination
  -> direct accessor write when the static current destination was selected

Candidate/Dynamic/ABSENT-at-selection cases
  -> exact existing generic fallback
```

## Governing normative semantics

The primary normative owner is
`spec/semantics/EXECUTION_AND_CONTROL.md`, section 7.

For bare assignment:

```protos
x = value
```

the specification requires:

1. select the nearest existing local lexical destination before evaluating the
   RHS;
2. if lexical search is exhausted, only the receiver's own local slot may be a
   destination;
3. never follow receiver delegation for writes;
4. if no destination exists, signal `SlotNotFound` before RHS evaluation;
5. after successful selection, evaluate the RHS;
6. attempt the write against that **same selected destination**;
7. never re-resolve or retarget because of RHS effects.

The RHS may create, remove, recreate, shadow, freeze, close or otherwise alter
same-named slots. Those effects do not restart destination selection.

This means the implementation has two distinct runtime questions:

```text
PRE_RHS:
  which exact destination is selected now?

POST_RHS:
  is mutation of that exact selected destination legal now?
```

They must not be collapsed into one after-RHS lookup.

## Binding analysis already carries the proof

`CanonicalBindingAnalyzer.walkAssign(...)` deliberately records assignment
resolution before walking the RHS:

```text
assignResolutions.put(assign, resolve(assign.name(), scope))
walk(assign.value(), scope)
```

The resolution classes are:

```text
Resolved(identity)
  current scope already established this binding

CapturedResolved(identity, lexicalDepth)
  outer genuine lexical owner already established

Candidate(identity, lexicalDepth)
  statically known possible owner, runtime presence not established

Dynamic
  no static owner in the known lexical chain
```

A same-scope `Resolved(identity)` therefore already carries the identity needed
to locate the root's frame-backed `BytecodeLocal`.

Under D179 C0, that proof establishes stable binding identity/layout, **not
permanent runtime presence**.

## Read/write asymmetry at current HEAD

For a proven current lexical read, `CanonicalToBytecodeLowerer.emitLookup(...)`
already consumes `Resolved(identity)`, finds the corresponding current-root
`BytecodeLocal`, and emits:

```text
ReadFrameLocal(LocalAccessor constant)
```

`ReadFrameLocal` checks the local's cleared state and takes exact lexical/
receiver fallback when the statically known current binding is runtime ABSENT.

For a proven current lexical assignment, `emitBodyAssign(...)` and
`emitDefaultAssign(...)` currently do not have the equivalent same-scope
`Resolved` path.

Unless the assignment is an explicit member target or a `CapturedResolved`
case, current lowering emits:

```text
ResolveWritableLexicalTarget(activation, name)
StoreLocal(assignMutationTarget)

evaluate RHS

AssignResolvedLexicalTarget(
  activation,
  assignMutationTarget,
  name,
  value)
```

Therefore current reads exploit static current-binding identity while current
writes still enter the general runtime write-target machinery.

## Current generic write-target representation

`ProtosBytecodeRootNode.ResolvedLexicalWriteTarget` carries either:

```text
currentContext = true
object = null
```

or an exact selected `ProtosObjectValue`.

`ResolveWritableLexicalTarget` performs:

```text
currentContextHasLocalSlotForRuntime(name)

else:
  scan captured lexical contexts by name

else:
  inspect receiver own local slot

else:
  SlotNotFound
```

and creates a `ResolvedLexicalWriteTarget`.

After RHS evaluation, `AssignResolvedLexicalTarget` branches on that carrier
and writes through either:

```text
activation.assignCurrentLocalSlotForRuntime(name, value)
```

or:

```text
destination.object.assignLocalSlot(name, value)
```

For an ordinary PRESENT statically resolved current local, this is a general
runtime representation of information the lowerer already knows.

The static audit does **not** claim one surviving heap allocation per
assignment. Graal may inline/scalar-replace Java-level construction. The
established result is that the optimizer currently receives the general target
resolution/carrier/authority shape instead of a direct constant-accessor current
write shape.

## Why raw StoreLocal is not the correct repair

The existing `ReadFrameLocal` implementation records a critical Bytecode DSL
constraint.

Protos frame-backed lexical bindings are read and written through
`LocalAccessor` / `LocalRangeAccessor` APIs. A local mutated only through the
dynamic accessor API does not participate in the same frame-slot-kind
speculation as the generated literal `StoreLocal` / `LoadLocal` pair.

Current source therefore explicitly warns that mixing the two mechanisms for
the same local is unsafe.

Consequently the intended specialized write should be a custom Bytecode
operation with a constant `LocalAccessor`, not a blind generated
`StoreLocal`.

## D179 C0 requires a pre-RHS presence guard

Consider:

```protos
x: 1
context.removeSlot("x")
x = rhs
```

The canonical assignment site can still classify as
`Resolved(current-x-identity)`, but at runtime the current local is ABSENT.

Bare assignment must then continue the exact writable-target search through
captured lexical contexts and, after lexical exhaustion, the receiver's own
local slot.

It must never silently recreate the cleared current local.

Required pre-RHS shape:

```text
if current LocalAccessor PRESENT:
  select STATIC_CURRENT
else:
  resolve exact existing generic writable destination now
  retain that exact fallback target
```

If no fallback exists, `SlotNotFound` occurs before RHS evaluation.

## The selected destination must survive RHS effects

If the current binding is PRESENT when destination selection occurs, it is the
selected destination for the entire assignment expression.

### RHS removes the selected binding

```text
select current x
RHS removes current x
RHS completes normally
```

The write must fail as mutation of the previously selected destination.

It must not search for an outer or receiver `x`.

### RHS removes and recreates the same owner binding

D179 C0 uses stable frame-local identity plus cleared/not-cleared presence.

Therefore:

```text
select current x
RHS removes current x
RHS recreates current x on the same execution context
RHS completes
```

can legally write the recreated same-owner binding, exactly as the existing
PERF013-B2 captured-write regression proves for the captured equivalent.

### Current binding is ABSENT before selection, then recreated by RHS

This is the converse case and is crucial:

```text
current x ABSENT
outer x PRESENT

pre-RHS selection -> outer x

RHS recreates current x

post-RHS write -> outer x
```

The recreated current binding must not steal the assignment after selection.

PERF013-B2's durable record identified the equivalent captured-owner scenario
as a historical standalone focal-coverage gap. PERF028-A should publish an
explicit same-scope regression for it.

## Mutation-state requirements

Ordinary object-state assignment rules apply to the already-selected
destination at the write point.

For execution contexts:

```text
CLOSED
  existing local mutation remains allowed

FROZEN
  existing local mutation fails
```

A specialized current write must therefore preserve:

```text
POST_RHS_SELECTED_CURRENT_PRESENT=required
POST_RHS_SELECTED_CURRENT_FROZEN=false required
CLOSED_SELECTED_CURRENT=write allowed
```

If the guest `ProtosExecutionContextValue` has not been materialized, current
runtime architecture already knows that no guest-visible freeze/close operation
has been applied through that object. Any helper introduced here should preserve
that lazy-observation property rather than materializing the context solely to
check mutation state.

## PLAT036 authority

PLAT036 Candidate D is already ratified and remains sufficient.

Its selected architecture requires:

```text
statically admitted lexical binding
  -> Bytecode DSL local / frame-backed authority

ONE_SEMANTIC_BINDING_VALUE_AUTHORITY_REQUIRED=YES
```

For assignment it explicitly retains:

```text
destination = resolve lexical write target
evaluate RHS
write to previously resolved target
```

and permits a resolved static destination to be represented by frame/local
identity while dynamic destinations use context/name identity.

Therefore PERF028 does not need to reopen PLAT036. It completes an already
authorized use of that representation boundary on the current lexical
write-side.

## PERF013-B2 precedent

PERF013-B2 migrated proven captured writes from:

```text
ResolveCapturedWritableLexicalTarget
AssignCapturedFrameLocal
```

to a same-generation fast path using:

```text
ResolveCapturedMaterializedWritableLexicalTarget
AssignCapturedMaterializedLocal
MaterializedLocalAccessor constant
```

while preserving:

```text
resolve exact destination
retain exact destination
evaluate RHS
revalidate exact retained destination
never re-resolve
write exact retained destination
```

That implementation established the closest available product precedent for
PERF028.

The associated compiler gate changed from:

```text
CAPTURED_WRITE_PE_FAILURE=PRESENT
```

to:

```text
CAPTURED_WRITE_PE_FAILURE=ABSENT
```

and the controlled B1->B2 timing checkpoint retained approximately:

```text
method-call            +18.8301% paired control effect
monomorphic-dispatch   +22.9525% paired control effect
```

Those measurements establish that runtime-authority lexical-write shape has
previously mattered to Graal PE and timing in Protos. They do **not** predict
PERF028 magnitude.

## Selected bounded implementation shape

The selected direction is:

```text
CanonicalBindingResolution.Resolved
owner == currentRootTopScope
current BytecodeLocal available
        |
        v
constant LocalAccessor current-write operation
        |
        +-- PRESENT before RHS
        |     select STATIC_CURRENT
        |
        +-- ABSENT before RHS
              exact generic writable resolution
              retain exact fallback target
        |
        v
evaluate RHS exactly once
        |
        v
write previously selected destination
        |
        +-- STATIC_CURRENT
        |     exact current owner
        |     require not FROZEN
        |     require not cleared
        |     accessor.setObject(...)
        |
        +-- GENERIC_FALLBACK
              exact retained target.assignLocalSlot(...)
```

The exact Java carrier for the two-way pre-RHS selection is implementation
machinery, not semantics.

The normal PRESENT current-local case should not require:

```text
String-keyed lexical-chain search
fresh general ResolvedLexicalWriteTarget
name -> layout offset lookup through the generic authority
```

A fallback target may still require a dynamic carrier because it is genuinely
selected at runtime before the RHS.

## Why not duplicate the RHS

The current lowerer stages `assignMutationTarget` across arbitrary RHS
evaluation.

Bytecode roots support yield/continuation. Therefore a selected destination may
need to survive suspension/resumption across the RHS.

A specialized implementation should not duplicate or split arbitrary RHS
lowering merely to eliminate a small carrier. Reuse the existing continuation
shape and make only the selection representation as narrow/PE-friendly as
necessary.

## Explicit exclusions

PERF028-A does not authorize:

```text
Candidate write specialization
Dynamic write specialization
explicit member-assignment changes
captured-write redesign
second lexical value authority
raw StoreLocal mixing for accessor-owned locals
D179 weakening
broad presence Assumption/invalidation architecture
lazy ContextCore redesign
numeric/call/Task performance changes
```

A later assumption-based optimization could investigate whether an
unobserved/unescaped context can avoid some D179 presence checks until the first
invalidating operation. That is independent of removing the current generic
write-target shape and is deliberately out of PERF028-A.

## Required PERF028-A regression matrix

At minimum:

| Case | Required result |
| --- | --- |
| current PRESENT | constant-accessor specialized path |
| assignment result | exact RHS |
| current ABSENT, outer PRESENT | outer selected pre-RHS |
| current ABSENT, receiver own slot PRESENT | receiver selected pre-RHS |
| current ABSENT, no fallback | SlotNotFound before RHS |
| current selected, RHS creates same-name elsewhere | no retarget |
| current selected, RHS removes current | mutation Error |
| current selected, RHS remove+recreate same owner | recreated same owner receives write |
| current ABSENT, farther fallback selected, RHS recreates current | farther fallback still receives write |
| selected current CLOSED | write allowed |
| selected current FROZEN | Error |
| PRESENT null | distinct from ABSENT |
| RHS control transfer | no write attempted |
| RHS yield/resume | exact pre-RHS selection retained |
| default-expression assignment | same semantics |
| context reflection/debugger | same single authoritative value |
| uncached/cached node transition | accessor metadata remains coherent |

## Blocker check

The durable implementation-blocker registry was inspected against the lexical
assignment surface.

No active blocker was found for PERF028's current lexical write specialization.

The normative and platform requirements needed for PERF028-A are already closed
by the current specification, D179 C0/I071 implementation, and ratified PLAT036
Candidate D.

## Selected next slice

```text
NEXT_SLICE=PERF028-A
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos

GOAL=
  specialize same-scope Resolved lexical assignments through the current
  binding's constant LocalAccessor while preserving exact pre-RHS destination
  selection, D179 presence, post-RHS exact-destination revalidation, mutation
  state and generic fallback

PRODUCT_CHANGE=YES
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
NEW_PLAT_DECISION=NO
```

PERF028-A should remain one bounded product slice unless current HEAD exposes a
new semantic/platform choice. If that occurs, stop rather than embedding the new
choice in performance implementation.

## Structural acceptance

```text
CURRENT_RESOLVED_WRITE_USES_CONSTANT_LOCAL_ACCESSOR=YES
ORDINARY_PRESENT_CURRENT_WRITE_GENERIC_NAME_SEARCH=NO
ORDINARY_PRESENT_CURRENT_WRITE_GENERAL_TARGET_OBJECT=NO_OR_PE_CONSTANT_EQUIVALENT

DESTINATION_SELECTED_BEFORE_RHS=PASS
POST_RHS_DESTINATION_RE_RESOLUTION=NO

D179_CLEAR_RECREATE_SEMANTICS=PASS
PRESENT_NULL_DISTINCT_FROM_ABSENT=PASS

CLOSED_WRITE_ALLOWED=PASS
FROZEN_WRITE_REJECTED=PASS

CANDIDATE_FALLBACK_PRESERVED=YES
DYNAMIC_FALLBACK_PRESERVED=YES
EXPLICIT_MEMBER_ASSIGNMENT_UNCHANGED=YES
CAPTURED_WRITE_SEMANTICS_PRESERVED=YES

ONE_SEMANTIC_BINDING_VALUE_AUTHORITY=PASS

OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
NEW_PLAT_DECISION=NO
```

## Performance claims deliberately not made

```text
RESOLVED_TARGET_HEAP_ALLOCATION_PER_ASSIGNMENT=NOT_PROVEN
CURRENT_WRITE_ATTRIBUTABLE_TIME=NOT_MEASURED
PERF028_EXPECTED_PERCENT_SPEEDUP=NOT_CLAIMED
DOMINANT_RUNTIME_CAUSE=NOT_ESTABLISHED
```

PERF013-B2 is supporting precedent, not a percentage forecast.

After an exact PERF028-A candidate is validated and published, causal timing may
reuse existing exact-revision/radar infrastructure if needed. No new broad
benchmark methodology is required by this investigation.

## Investigation result

```text
PERF028_STATIC_INVESTIGATION=COMPLETE

PROVEN_CURRENT_WRITE_GENERIC_RUNTIME_PATH=CONFIRMED
STATIC_CURRENT_BINDING_IDENTITY_ALREADY_AVAILABLE=YES
READ_SIDE_ALREADY_USES_LOCALACCESSOR=YES
WRITE_SIDE_EQUIVALENT_MISSING=YES

RAW_STORELOCAL_REPLACEMENT=REJECTED
CONSTANT_LOCALACCESSOR_FAST_PATH=SELECTED

PRE_RHS_PRESENCE_SELECTION_REQUIRED=YES
POST_RHS_EXACT_TARGET_REVALIDATION_REQUIRED=YES
POST_RHS_RESEARCH_OR_RETARGET=FORBIDDEN

ABSENT_AT_SELECTION_GENERIC_FALLBACK_REQUIRED=YES
CANDIDATE_DYNAMIC_GENERIC_PATH_REQUIRED=YES

PLAT036_REOPEN_REQUIRED=NO
D179_REOPEN_REQUIRED=NO
SPEC_CHANGE_REQUIRED=NO

NEXT_SLICE=PERF028-A
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos
```

## Materially inspected authority

The investigation materially inspected/reconciled at least:

```text
guillermomolina/protos:
  AGENTS.md
  AGENTS.work/PERFORMANCE.md
  AGENTS.work/IMPLEMENTATION.md
  AGENTS.work/COORDINATION.md
  AGENTS.work/REFERENCE.md
  spec/semantics/EXECUTION_AND_CONTROL.md
  spec/semantics/OBJECT_MODEL.md
  docs/guide/01-bindings-contexts-and-state.md
  docs/guide/03-closures-methods-and-receivers.md
  src/main/java/com/guillermomolina/protos/execution/CanonicalBindingAnalyzer.java
  src/main/java/com/guillermomolina/protos/execution/CanonicalBindingResolution.java
  src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
  src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
  src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
  src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java
  src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalLayout.java
  src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java
  src/main/java/com/guillermomolina/protos/runtime/ProtosExecutionContextValue.java
  src/main/java/com/guillermomolina/protos/runtime/ProtosObjectValue.java
  src/main/java/com/guillermomolina/protos/runtime/ProtosLexicalFallback.java
  src/test/java/com/guillermomolina/protos/execution/ProtosPerf013SliceB2MaterializedCapturedWriteTest.java
  CHANGELOG.md

guillermomolina/protos-project-docs:
  AGENTS.md
  docs/project/README.md
  docs/project/registries/IMPLEMENTATION_BLOCKERS.md
  docs/project/decisions/platform/PLAT036_BYTECODE_DSL_LEXICAL_STATE_REPRESENTATION_BOUNDARY.md
  docs/project/decisions/platform/PLAT037_LAZY_EXECUTION_CONTEXT_PHYSICAL_MATERIALIZATION.md
  docs/project/evidence/PERF025/PERF025_POST_F1_PAY_AS_YOU_GROW_RUNTIME_COST_AUDIT.md
  docs/project/evidence/PERF013/PERF013_SLICE_B2_MATERIALIZED_CAPTURED_WRITE_IMPLEMENTATION.md
  docs/project/evidence/PERF013/PERF013_B1_B2_CONTROLLED_TIMING_CHECKPOINT.md
```

No product build, test, benchmark or runtime command was executed as part of
this static investigation.

