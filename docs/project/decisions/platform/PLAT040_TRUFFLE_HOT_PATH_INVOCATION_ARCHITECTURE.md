# PLAT040 — Truffle hot-path dispatch and invocation architecture

Status: RATIFIED

Selected architecture: **Candidate F′ — guarded hot call with frame-argument invocation and conditional guest-state materialization**.

Approval: explicit project-owner approval on 2026-09-25:

~~~text
apruebo f'
~~~

Decision Issue: guillermomolina/protos#718

Performance consumers: PERF010-A / guillermomolina/protos#691 and PERF011 / guillermomolina/protos#693

Implementation consumer: a dedicated Ixxx implementation item allocated only after this ratification is published.

Product baseline:

~~~text
PROTOS_REVISION=0c9307240796260882ddca27611c91b7ffa4ce3b
PROTOS_VERSION=0.3.88-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.3.4.1
~~~

The product baseline remained identical to current `guillermomolina/protos` `main` at ratification.

Nature: durable non-normative platform/runtime architecture decision. Observable Protos semantics remain owned by the normative specification.

## Decision

Protos adopts the following hot-call architecture for ordinary stable dispatch and invocation:

~~~text
semantic resolution
  -> retain stable selected callable / methodHome / target
  -> exact guard / assumption / invalidation dependency
  -> DirectCallNode / inlineable target
  -> compact Truffle frame-argument invocation ABI
  -> conditional materialization of guest-visible rich state
  -> exact generic fallback on invalidation, megamorphism, escape or unsupported cases
~~~

The selected architecture is intentionally a composition of mechanisms repeatedly observed in mature Truffle runtimes rather than a Protos-specific optimization invention.

## Authoritative semantic boundary

D013 and the current normative object/callable/control semantics remain authoritative.

Candidate F′ does not change:

- ordinary lookup, delegation, add/remove/replace or missing-lookup behavior;
- the single Closure executable kind;
- fresh execution-context semantics;
- capture by reference;
- `this` and `methodHome`;
- argument evaluation/binding order;
- non-local return semantics;
- Error/dynamic-control semantics;
- Task, Actor, Process, module or execution-domain semantics;
- suspension/continuation semantics;
- reflection/debugger-visible behavior; or
- multiple-Truffle-Context behavior.

Implementation machinery may change only when those observations remain exact.

## Guarded stable selection

D013 remains the semantic authority for ordinary sends.

After D013 has selected the effective Closure and `methodHome`, a stable site may retain that selection and its effective Context-appropriate target while an exact validity dependency proves that re-running D013 would select the same result.

A valid hit therefore does not repeat general lookup or generic selection/classification.

The validity dependency must be invalidated by every mutation capable of changing the selected D013 result, including as applicable:

- replacement or removal of the selected slot;
- addition of the selector at any nearer lookup position;
- delegation-parent changes or represented-delegation changes relevant to the lookup chain; and
- any other object/runtime mutation that can alter the selected result.

Unrelated mutation must not invalidate the site merely for convenience.

On miss or invalidation, execution returns to the exact generic D013 path and may establish a new specialization.

No Truffle `DynamicObject` / `Shape` representation is selected by this decision. Only the semantic requirement for a constant-cost lookup-stability dependency is selected.

## Frame-argument invocation ABI

The ordinary hot call must cross the Truffle call boundary using a compact frame-argument ABI rather than requiring a universal rich invocation carrier.

The ABI may carry directly the state genuinely required by that invocation, including where applicable:

- callable/executable identity;
- receiver / `this`;
- selected `methodHome`;
- supplied positional values;
- captured lexical/frame reference;
- module/domain references; and
- the minimum control token/state required by that executable path.

The exact Java layout is an implementation choice and is not specified here.

Closure-captured lexical state remains associated with or reachable from the callable/captured frame relationship. It is not reconstructed as a new rich lexical aggregate merely because a call occurs.

## Conditional guest-state materialization

Fresh Protos execution-context semantics remain mandatory. Eager allocation of the full guest-visible execution-context representation on every ordinary optimized call is not.

The implementation must follow the externally established Truffle pattern:

~~~text
frame / invocation state first
guest-visible context/frame projection only when observed, escaped or otherwise semantically required
~~~

A materialized guest context must preserve the exact identity, structural mutation, capture-by-reference, reflection and debugger behavior required by the specification.

The completed ordinary supplied-argument vector is likewise not required to become a guest `Array` merely to cross the internal call boundary. A guest `Array` is materialized only where observable semantics require one, such as an applicable rest binding or other explicit guest observation.

The Protos specification already permits scalar replacement, virtualization and avoidance of physical activation/Array copies when identity, reflection, mutation failure, capture, escape, evaluation order and other observations are preserved.

## Optional control and suspension state

Optional semantic capability must not force unrelated ordinary calls to carry its full physical machinery.

Non-local-return, dynamic-control, Task, suspension and continuation state remain exact where applicable, but the ordinary hot-call contract does not require a universal carrier containing all such modes simultaneously.

The implementation must separate ordinary invocation from optional/specialized state using the smallest externally supported Truffle pattern that preserves semantics.

This decision does not prescribe the exact representation of a return-home token, dynamic-control carrier, Task state or continuation carrier. A Protos-only representation change without strong comparable-runtime evidence is not authorized by PLAT040.

## Structured control

Standard protocol operations such as Boolean control and collection callbacks may specialize earlier only after ordinary semantic selection has established the expected standard behavior under an equivalent validity contract.

This does not make `ifTrue`, `ifFalse`, `while`, collection iteration or another standard selector privileged language semantics.

Copied, replaced, aliased or non-canonical behavior remains ordinary behavior unless it independently satisfies the same guarded semantic-selection contract.

The goal is that stable structured control becomes visible as stable Truffle structure rather than repeatedly traversing the full generic send/call universe.

## Cross-Truffle basis

The selected architecture is supported by a recurring pattern across materially different Truffle implementations:

- **Apple Pkl**: virtual method and function-call nodes cache receiver/method or call-target state and use `DirectCallNode`; `VmFunction` retains captured frame state; collection callbacks use cached `ApplyVmFunctionNNode`; structural `when` control executes directly.
- **GraalPy**: Python calls use a frame-argument ABI; caller-frame and Python-frame materialization are conditional on caller flags; guest-visible `PFrame` state is materialized when needed.
- **GraalJS**: function calls use compact `JSArguments`, cached function/call-target state and `DirectCallNode`; enclosing frame state belongs to the function object.
- **TruffleRuby**: method/block calls cache call targets under guards/assumptions and use `RubyArguments`; lexical frame, method, declaration context, self, block and optional non-local-control marker are frame arguments rather than a universal activation object.
- **TruffleSqueak**: block dispatch caches compiled block/call target and uses `DirectCallNode`; `ContextObject` is lazily initialized when required.
- **Espresso**: virtual invocation caches resolved method state under class/redefinition assumptions; the megamorphic path alone repeats generic lookup.
- **Sulong/LLVM**: cached function-code/descriptor specializations call `DirectCallNode` with argument arrays and retain indirect fallback.
- **FastR**: cached call targets use `RArguments` frame arrays even with enclosing frames and caller state.
- **Enso**: cached target specializations use direct or inlineable call paths; its source explicitly identifies the uncached indirect path as slower.
- **GraalWasm**: direct call targets are resolved before the hot call; optional return-call machinery is separated from ordinary call state.

No materially comparable runtime was found whose intended monomorphic steady state repeatedly performs full general semantic lookup plus universal rich invocation-state construction before finally reaching a direct target.

SimpleLanguage remains only a minimal explanatory reference and is not primary evidence for Candidate F′.

## PLAT036 / I068 composition

PLAT036/I068 frame-backed lexical-state authority remains valid.

I068 solved the lexical-representation problem but did not create the historical rich invocation scaffold. The broad historical performance gap predates I068; post-I068 work can add cost when the old scaffold also carries more frame/materialization state, but that does not make frame-backed lexical authority the historical common cause.

Candidate F′ preserves one lexical binding authority and capture-by-reference.

## PLAT037 composition

PLAT037 selected deferral of lazy execution-context physical materialization pending evidence.

PLAT040 establishes that the evidence gate is now satisfied:

~~~text
PLAT037_EVIDENCE_GATE_SATISFIED=YES
~~~

The evidence includes GraalPy's conditional frame/PFrame materialization, TruffleSqueak's lazy `ContextObject`, and the persistence of the broad Protos gap after I068.

PLAT040 does **not** automatically reselect PLAT037's earlier Protos-specific `ContextCore` candidate or any particular Java representation:

~~~text
PLAT037_SPECIFIC_CONTEXTCORE_CANDIDATE_ESTABLISHED=NO
PLAT037_REOPEN_REQUIRED=NO
~~~

Instead, Candidate F′ selects the more general externally supported architectural invariant: compact frame/invocation state first, guest-visible context materialization only when semantically required.

## PLAT039 composition

PLAT039 remains authoritative.

The compact call ABI, guard/assumption checks, semantic state required for ordinary execution and conditional materialization decision stay inside the PE-visible guest kernel.

Opaque host/runtime services continue to cross only the narrow gateways selected by PLAT039.

Candidate F′ must not enlarge or strand the partial-evaluation graph with host-only mechanics.

## Current Protos divergence

At the pinned baseline, ordinary invocation materially differs from the common pattern.

`ProtosActivation.forClosureInvocation` / `forImmediateMethodInvocation` eagerly assemble state including a fresh execution context, supplied guest array, return-home relationship, receiver/home, captured lexical relations, module/domain state and dynamic-control inheritance.

`PreparedClosureCall` is a universal carrier with ordinary-call state plus numerous mutually exclusive structured-control, collection, import, native and continuation modes before the final target entry.

The cached `DirectCallNode` therefore stabilizes too late: substantial generic resolution/classification/materialization has already occurred.

Those are representation facts, not Protos semantic requirements.

## Rejected / superseded candidate

The earlier Candidate F — guarded selected-send caching while intentionally retaining the current invocation-state construction — is **WITHDRAWN AS INSUFFICIENT** for PLAT040.

Its dispatch half remains part of F′, but the comparative evidence establishes a second independent architectural divergence in the invocation ABI/materialization path.

## Owner evidence constraint

During the final PLAT040 review the project owner explicitly required that hot-path changes be grounded in how other Truffle languages actually solve the same problem, especially GraalVM-native implementations and Apple Pkl.

A Protos-only optimization hypothesis without high evidence in materially comparable runtimes is not part of the selected architecture merely because it appears locally plausible.

This evidence constraint was applied to F′: the decision selects only the common architectural principles above and deliberately leaves Protos-specific Java representations unspecified.

## GITHUB021 invariant/delta consistency

The exact-candidate consistency review passes.

Candidate F′ preserves all previously fixed PLAT040 hard boundaries and related ratified decisions:

- D013 remains authoritative;
- no standard selector becomes privileged language semantics;
- execution-context identity and object semantics remain exact;
- PLAT036/I068 frame-backed lexical authority remains exact;
- PLAT037's specific deferred representation is not silently reselected;
- PLAT039's PE-visible guest-kernel / narrow-host-gateway boundary remains exact;
- no observable Protos semantic change is authorized.

The material delta from the earlier recommendation was surfaced before approval: F′ additionally moves the ordinary invocation ABI and guest-state materialization boundary into PLAT040 because cross-Truffle evidence showed that the earlier Candidate F left a second major hot-path divergence intact.

That exact F′ delta was explicitly approved by the project owner.

## Implementation boundary

Implementation is now authorized in a dedicated implementation work item created after this durable ratification.

The implementation should be decomposed into bounded validated slices because dispatch invalidation, invocation ABI, context materialization, optional control state and structured-call carrier separation are independently reviewable risk surfaces.

Decomposition must preserve one coherent final outcome; no slice may introduce temporary observable semantics.

The implementation work must derive exact source edits from current repository evidence. PLAT040 does not pre-authorize a particular Java class layout, Shape model, context-core class, return-home token or PIC width.

## PERF010-A / PERF011 consequence

PERF010-A remains the causal-attribution owner and PERF011 remains representation-fit evidence.

After implementation slices establish the selected structural architecture, performance evidence must distinguish:

~~~text
STRUCTURAL_HOT_PATH_CONVERGENCE
from
ATTRIBUTABLE_TIMING_EFFECT
~~~

PLAT040 does not establish a numeric speedup, a dominant-cause percentage, or a claim that Candidate F′ alone must reach parity with another runtime.

The falsifiable architectural prediction is stronger than a small local optimization claim: ordinary stable calls should stop paying repeated general lookup/classification plus universal rich invocation-state construction in their steady path.

## Ratified result

~~~text
PLAT040_HOT_PATH_ARCHITECTURE=ESTABLISHED

PROTOS_REVISION=0c9307240796260882ddca27611c91b7ffa4ce3b
PROTOS_VERSION=0.3.88-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.3.4.1

LEXICAL_STATE_STATUS=STABILIZED_BY_PLAT036_I068
DISPATCH_STABILITY_STATUS=REQUIRES_GUARDED_SELECTED_SEND
STRUCTURED_CONTROL_STABILITY_STATUS=REQUIRES_EARLY_SEMANTICALLY_GUARDED_STRUCTURE
INVOCATION_STATE_MATERIALIZATION_STATUS=REQUIRES_FRAME_ARGUMENT_ABI_AND_CONDITIONAL_GUEST_MATERIALIZATION

CROSS_TRUFFLE_COMMON_PATTERN=
  RESOLVE_AND_SPECIALIZE
  + GUARD_OR_ASSUMPTION
  + DIRECT_OR_INLINE_CALL
  + FRAME_ARGUMENT_CALL_ABI
  + CALLABLE_OWNED_CAPTURED_STATE
  + CONDITIONAL_OPTIONAL_STATE
  + LAZY_OR_ESCAPE_DRIVEN_GUEST_CONTEXT_MATERIALIZATION
  + GENERIC_FALLBACK_ON_INVALIDATION

D013_LOOKUP_CAN_BE_SKIPPED_ON_GUARDED_HIT=YES
REQUIRED_INVALIDATION_MODEL=EXACT_LOOKUP_STABILITY_DEPENDENCY
TRUFFLE_DYNAMICOBJECT_SHAPE_REQUIRED=NO

FRAME_ARGUMENT_INVOCATION_ABI=REQUIRED
EAGER_SUPPLIED_GUEST_ARRAY_REQUIRED=NO
EAGER_GUEST_CONTEXT_PHYSICAL_OBJECT_REQUIRED=NO
UNIVERSAL_RICH_ACTIVATION_HOT_PATH=REJECTED
UNIVERSAL_PREPARED_CALL_CARRIER_HOT_PATH=REJECTED

STRUCTURED_CONTROL_CAN_SPECIALIZE_EARLIER=YES
STANDARD_PROTOCOL_PRIVILEGING_REQUIRED=NO

FRESH_EXECUTION_CONTEXT_SEMANTICS_PRESERVED=YES
PLAT037_EVIDENCE_GATE_SATISFIED=YES
PLAT037_SPECIFIC_CONTEXTCORE_CANDIDATE_ESTABLISHED=NO
PLAT037_REOPEN_REQUIRED=NO

PLAT036_COMPATIBLE=YES
PLAT039_COMPATIBLE=YES
OBSERVABLE_PROTOS_SEMANTIC_CHANGE_REQUIRED=NO

OLD_CANDIDATE_F_STATUS=WITHDRAWN_AS_INSUFFICIENT
RECOMMENDED_CANDIDATE=F_PRIME
RECOMMENDED_CANDIDATE_NAME=
  GUARDED_HOT_CALL_WITH_FRAME_ARGUMENT_INVOCATION_AND_CONDITIONAL_GUEST_STATE_MATERIALIZATION
RECOMMENDATION_STATUS=APPROVED_AND_RATIFIED

IMPLEMENTATION_READY=YES
IMPLEMENTATION_OWNER=DEDICATED_IXXX_ALLOCATED_AFTER_RATIFICATION
~~~

