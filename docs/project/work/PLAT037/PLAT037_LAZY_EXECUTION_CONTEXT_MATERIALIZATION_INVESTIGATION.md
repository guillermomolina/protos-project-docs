# PLAT037 — Lazy execution-context physical materialization investigation

Status: **INVESTIGATION COMPLETE / RECOMMENDATION PENDING PROJECT-OWNER APPROVAL**

Decision Issue: `guillermomolina/protos#709`

Origin: `AUD016 / #701`

Nature: exhaustive platform-architecture investigation only. This record is
durable, non-normative project evidence. It does not authorize implementation
and does not change observable Protos semantics.

## Exact baselines

~~~text
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=f1cee2d85858804ad3775adf43a9fab97664da2a
PROTOS_VERSION=0.3.87-SNAPSHOT

PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
PRIOR_PROJECT_RECORD_REVISION=8f462172c4a84cc9099fde1737a2957ccbfdc66e

GRAALVM_TRUFFLE_VERSION=25.3.4.1

D179_SELECTED_CANDIDATE=C3_MONOTONIC_CONTEXT_MEMBERSHIP
PLAT036_SELECTED_CANDIDATE=D_FRAME_BACKED_SEMANTIC_CONTEXT_ADAPTER
I068_STATUS=COMPLETE
I068_FINAL_PRODUCT_REVISION=f1cee2d85858804ad3775adf43a9fab97664da2a

PLAT036_DOES_NOT_DECIDE_LAZY_CONTEXT_OBJECT_MATERIALIZATION=YES
I068_DOES_NOT_IMPLEMENT_LAZY_CONTEXT_OBJECT_MATERIALIZATION=YES
~~~

The live tracker contained no pre-existing PLAT owner for the exact remaining
AUD016 question. `PLAT037` was therefore allocated as
`guillermomolina/protos#709`.

The Issue records textual parent `#701`. At investigation publication time the
available GitHub integration did not expose a native Parent/Sub-issue mutation
operation, and the API still reported no native parent. Therefore the native
hierarchy postcondition remains explicitly incomplete; this does not affect the
technical conclusion and must not be mistaken for ratification.

## Machine-readable investigation result

~~~text
LAZY_CONTEXT_MATERIALIZATION_ARCHITECTURALLY_VALID=
  CONDITIONAL

CURRENT_EAGER_CONTEXT_ARCHITECTURE=
  KEEP

RECOMMENDED_CANDIDATE=
  D — DEFER / KEEP CURRENT PENDING EVIDENCE

RECOMMENDATION_STATUS=
  PENDING_PROJECT_OWNER_APPROVAL

OBSERVABLE_SEMANTIC_CHANGE_REQUIRED=
  NO

NEW_IDENTITY_SUBSTRATE_REQUIRED=
  YES

DYNAMIC_OVERFLOW_BEFORE_OBJECT_MATERIALIZATION=
  SINGLE_CONTEXT_CORE_AUTHORITY_SUBSTRATE;
  NO_SECOND_AUTHORITATIVE_COPY

DEBUGGER_FORCES_MATERIALIZATION=
  CONDITIONAL

CLOSURE_CAPTURE_FORCES_MATERIALIZATION=
  NO

SUSPENSION_FORCES_MATERIALIZATION=
  NO

MEASUREMENT_REQUIRED_BEFORE_IMPLEMENTATION=
  YES

IMPLEMENTATION_READY=
  NO

NEW_DECISION_REQUIRED_BEYOND_THIS_PLAT=
  NO
~~~

Additional classification:

~~~text
CAN_LAZY_MATERIALIZATION_BE_CORRECT=YES
IS_LAZY_MATERIALIZATION_CURRENTLY_WORTH_IMPLEMENTING=NOT_ESTABLISHED
I068_EFFECT_ON_LAZY_MATERIALIZATION=EASIER
~~~

## 1. Exact decision being requested

The question is not whether Protos may stop having one semantic execution
context per invocation. It may not: the normative model requires the activation
to have a fresh first-class execution-context object with stable semantic
identity.

The question is whether that already-existing semantic object must immediately
be represented by a physical Java `ProtosExecutionContextValue`, or whether the
invocation may first establish a smaller internal semantic-identity/state
substrate and materialize the Java guest-visible wrapper only when an operation
actually needs an object value.

The investigation therefore keeps these notions separate:

~~~text
semantic context identity
physical ProtosExecutionContextValue allocation
frame-backed lexical value authority
Truffle MaterializedFrame lifetime
Closure lexical capture
debugger scope projection
Task / suspension topology
~~~

## 2. Normative invariants

The pinned normative specification establishes all of the following:

- Every invocation has a fresh execution context, and `context` denotes that
  same context throughout parameter/default binding and body execution.
- Execution contexts are first-class Protos objects and ordinary
  identity-bearing objects. Their semantic identity cannot depend on boxing,
  physical allocation, wrapper address, or rematerialization strategy.
- Context local-slot membership follows D179/C3: ABSENT may become PRESENT while
  OPEN; PRESENT may be mutated while writable; once PRESENT it cannot return to
  ABSENT. `PRESENT(null)` remains distinct from ABSENT.
- OPEN/CLOSED/FROZEN state remains semantically observable and shallow.
- Closure capture is by reference to genuine lexical execution contexts.
  Mutation after capture and legal late creation in a nearer context remain
  visible and may retarget later lookup.
- Parameter/default binding is sequential. The activation and return-home
  already exist while defaults execute, but each parameter becomes PRESENT only
  at its semantic binding point.
- Bare assignment selects its destination before evaluating the RHS and does not
  re-resolve after RHS effects.
- Receiver, methodHome and non-local return remain activation/control concepts
  rather than hidden context slots.
- Suspension/resumption, deoptimization or host-stack unwinding does not
  semantically leave or recreate the activation.
- Debugger/tooling scope must project semantic guest-visible bindings and must
  not expose backend temporaries.
- Object construction is not a new lexical-capture scope: its construction
  activation uses the object under construction as the creation target while
  closures created there capture the enclosing genuine lexical contexts.
- Module execution uses a module execution context that is also the module
  instance and may become observable through import during initialization.

No candidate below weakens those invariants.

## 3. Current implementation map after I068

At the exact product revision, ordinary Closure/method invocation performs:

~~~text
ProtosActivation.forClosureInvocation / forImmediateMethodInvocation
  -> ProtosPrelude.newExecutionContext()
  -> new ProtosExecutionContextValue(Context prototype)
  -> new ProtosMapBackedLexicalBindingAuthority()
  -> new LinkedHashMap()
  -> ProtosActivation stores the ProtosObjectValue context
~~~

The Bytecode root later installs I068's frame-backed authority:

~~~text
InstallFrameLexicalAuthority
  -> frame.materialize()
  -> ProtosFrameLexicalBindingAuthority
       static admitted bindings -> Bytecode DSL frame/local state
       dynamic-only bindings    -> dynamicOverflow
       semantic presence/order  -> authority metadata
  -> ProtosExecutionContextValue.installFrameLexicalBindingAuthority(...)
~~~

Therefore current invocation setup distinguishes two different things but still
performs both eagerly:

~~~text
TRUFFLE_FRAME_MATERIALIZATION=EAGER_AT_ROOT_AUTHORITY_INSTALLATION
PROTOS_EXECUTION_CONTEXT_WRAPPER_ALLOCATION=EAGER_AT_ACTIVATION_CREATION
~~~

This distinction is central: making the guest context wrapper lazy would not, by
itself, remove I068's current frame materialization.

### Current ownership and observation map

~~~text
WHEN_IS_CONTEXT_OBJECT_CURRENTLY_ALLOCATED?
  Before parameter/default execution, during Closure/method activation creation.

WHO_REQUIRES_ITS_SEMANTIC_IDENTITY?
  Guest context observation, object identity/equality-sensitive operations,
  IdentityMap/default object hash once the context value is observable, escaped
  context aliases, and module-instance identity where applicable.

WHO_STORES_A_REFERENCE_TO_IT?
  ProtosActivation; captured lexical-context lists in ProtosClosureValue;
  lexical fallback; debugger scope indirectly through activation; the
  frame-backed authority is currently attached to the wrapper.

WHO_CAN_CAUSE_IT_TO_ESCAPE?
  Guest evaluation of context followed by return/store/pass/delegation/reflection;
  a default expression can expose context before the full parameter list binds.
  A Closure can retain the semantic lexical context even when guest code never
  evaluates context explicitly.

WHO_CAN_OBSERVE_LEXICAL_STATE_WITHOUT_GUEST_CODE_EVALUATING_context?
  Closure execution/capture paths, lexical fallback, parameter binding, the
  debugger scope projector, and runtime continuation paths.

WHICH_OPERATIONS_REQUIRE_AN_ACTUAL_GUEST_OBJECT_VALUE?
  Evaluating context as a value; ordinary object/delegation/reflection/mutation
  dispatched on that value; guest identity-sensitive use of that value.

WHICH_OPERATIONS_REQUIRE_ONLY_LEXICAL_AUTHORITY?
  Local read/create/assign; parameter presence; late creation and retargeting;
  dynamic overflow; debugger scope enumeration/read/write.

WHICH_OPERATIONS_REQUIRE_ONLY_ACTIVATION_TOPOLOGY?
  receiver/methodHome, return-home/non-local return, Task ownership,
  suspension/resumption and dynamic handler/ensure state.
~~~

The current `ProtosIdentity` implementation uses Java reference identity and
`System.identityHashCode` for ordinary identity-bearing objects, including
execution contexts. Consequently a lazy architecture may retain that primitive
implementation only if it creates at most one guest wrapper for each semantic
context identity and subsequently reuses that wrapper.

## 4. I068 re-audit

I068 makes lazy physical wrapper creation **EASIER**, but does not make it
automatically useful.

Before I068, the context object was also the obvious lexical-value store. After
I068, statically admitted lexical value authority is already frame-backed behind
a `ProtosLexicalBindingAuthority` seam. This reduces the semantic state that a
guest wrapper intrinsically needs to own.

However, I068 still uses `ProtosExecutionContextValue` as:

- the activation's lexical-context node;
- the object captured by Closure values;
- the attachment point for the lexical authority;
- the current-context object used by fallback;
- the source for debugger projection; and
- the stable Java identity used by current ordinary-object identity primitives.

Thus the following model is not valid merely by deleting the wrapper allocation:

~~~text
activation exists
lexical frame exists
semantic context identity exists
ProtosExecutionContextValue absent
~~~

It becomes valid only if a new durable internal component owns the semantic
identity/state boundary before the wrapper exists.

## 5. Exact Truffle 25.3.4.1 substrate

The pinned Truffle line provides the substrate needed for lexical lifetime, but
not the complete Protos semantic context abstraction.

Relevant 25.3.4.1 mechanisms include:

- `BytecodeLocal`;
- `LocalAccessor` for current-frame locals;
- `MaterializedLocalAccessor` for a `MaterializedFrame`, including outer-root
  access and cleared/presence operations;
- `enableMaterializedLocalAccesses`;
- materialized frames that may be stored beyond the current Java stack frame;
- Bytecode continuations whose saved interpreter state includes a
  `MaterializedFrame`; and
- instrumentation/debugger frame access.

These APIs are enough to keep I068's static lexical authority alive through
capture and suspension. They do not supply:

- Protos execution-context semantic identity;
- Context -> Object delegation;
- OPEN/CLOSED/FROZEN state;
- dynamic-only slot storage and semantic creation order;
- exact-one guest object projection;
- Protos identity-hash semantics; or
- module/context policy.

Therefore `MaterializedFrame` alone is not a sufficient lazy-context identity
substrate.

Pinned public authorities consulted include:

- GraalVM/Truffle Javadocs for `GenerateBytecode`, `LocalAccessor`,
  `MaterializedLocalAccessor`, `Frame`, `VirtualFrame`,
  `MaterializedFrame` and `ContinuationResult`;
- Maven Central artifact
  `org.graalvm.truffle:truffle-api:25.3.4.1`; and
- GraalVM 25.3.4.1 release metadata.

## 6. Exhaustive comparative prior art

The comparisons below distinguish ordinary lexical storage from guest-visible
execution-context identity.

| Runtime | Ordinary lexical frame | Captured environment | First-class activation/context object | Lazy/materialized pattern | Relevance to PLAT037 |
| --- | --- | --- | --- | --- | --- |
| TruffleSOM | Indexed Truffle frame state | `SBlock` carries a `MaterializedFrame` only for blocks that need outer context | No Protos-like first-class context per invocation | Context-free blocks store no frame; contextual blocks materialize it | Strong evidence for pay-only-for-capture, not for guest context identity |
| TruffleSqueak | Truffle frame | Block/context machinery | Yes, Smalltalk `ContextObject` | `GetOrCreateContextWithFrameNode` creates/reuses one Context and attaches/materializes the frame | Closest Truffle precedent for first-class lazy context reification |
| TruffleRuby | Frame slots/declaration frames | `RubyProc.declarationFrame` is a `MaterializedFrame` | `Binding` is first-class but is not the canonical object for every activation | Bindings/debugger views are created from materialized frames when needed | Supports separating captured frame state from first-class reflective wrapper |
| GraalJS | Frame slots and block-scope frames | Functions link enclosing materialized frames when ancestor scope is needed | No ordinary guest execution-context object per call | Heap/context scope is selected where lexical capture/dynamic semantics need it | Supports selective environment representation, not Protos identity |
| GraalPy | Truffle frame | Python frame references/cells | Python frame objects are guest-visible/introspectable | Lightweight `PFrame.Reference` exists first; `PFrame` is filled/materialized when stack introspection/traceback requires it | Very strong precedent for Candidate B's stable lightweight substrate plus lazy object |
| Espresso | JVM local/operand slots | JVM-defined object/closure mechanisms | No guest first-class local-scope object | Debug/instrumentation can materialize frame state | Negative control: frame materialization is not semantic context identity |
| Sulong/LLVM | LLVM frame/runtime state | Language/ABI-specific | No Protos-like guest execution context | Debugger projects source-level variables from materialized frame/debug metadata | Strong debugger-projection precedent, weak identity precedent |
| SimpleLanguage | Integer/frame-slot locals | Minimal example-language capture model | None | Ordinary locals stay frame-native | Baseline proving static locals need no object dictionary |
| Apple Pkl | Virtual/MaterializedFrame | `VmFunction` and `VmObjectLike` retain an enclosing `MaterializedFrame` | No Protos-like one-context-object-per-invocation contract | Lexical objects/functions retain materialized enclosing frames | Materially relevant to capture substrate; not a lazy first-class context analogue |
| Squeak/Cog stack VM | Stack activation | Blocks/context references | Yes, Smalltalk Context | Contexts are created lazily and may be married to a stack frame, then widowed after return | Strongest semantic family analogue |
| CPython 3.11+ | Lean interpreter frame | Cells/frame references | `PyFrameObject` is introspection-visible | Most calls avoid a full frame object; it is created when debugger/`sys._getframe`/traceback requires it | Strong non-Truffle precedent for Candidate B |
| V8 | Stack locals where possible | Heap Context for captured variables | No ordinary JS frame/context object | Compiler decides stack versus heap Context based on capture/dynamic needs | Evidence for selective lexical environment allocation, not guest identity |
| HotSpot/Graal compiler | Compiled/interpreter frame plus debug state | JVM object graph | Java objects may themselves be scalar-replaced | Deoptimization can rematerialize scalar-replaced objects when observation requires them | Important evidence that compiler escape analysis may already eliminate some eager Protos wrapper cost |

Source locations materially inspected include:

~~~text
hpi-swa/trufflesqueak:
  .../nodes/context/GetOrCreateContextWithFrameNode.java
  .../model/ContextObject.java
  .../util/FrameAccess.java

SOM-st/TruffleSOM:
  src/trufflesom/src/trufflesom/vmobjects/SBlock.java
  src/trufflesom/src/trufflesom/interpreter/nodes/literals/BlockNode.java
  src/trufflesom/src/trufflesom/interpreter/nodes/ContextualNode.java
  src/trufflesom/src/trufflesom/interpreter/FrameOnStackMarker.java

truffleruby/truffleruby:
  src/main/java/org/truffleruby/core/proc/RubyProc.java
  src/main/java/org/truffleruby/core/binding/RubyBinding.java
  src/main/java/org/truffleruby/debug/RubyScope.java

oracle/graalpython:
  docs/contributor/IMPLEMENTATION_DETAILS.md
  .../builtins/objects/frame/PFrame.java
  .../nodes/frame/MaterializeFrameNode.java

oracle/graaljs:
  .../runtime/JSFrameUtil.java
  .../nodes/access/ScopeFrameNode.java
  .../nodes/function/BlockScopeNode.java

oracle/graal:
  espresso/.../MethodWithBytecodeNode.java
  sulong/.../LLVMDebuggerScopeFactory.java
  truffle/.../com.oracle.truffle.sl/.../SLReadLocalVariableNode.java

apple/pkl:
  pkl-core/.../runtime/VmObjectLike.java
  pkl-core/.../runtime/VmFunction.java

non-Truffle:
  CPython InternalDocs/frames.md and Python 3.11 frame notes
  OpenSmalltalkVM/Squeak StackInterpreter documentation
  V8 fast-properties/runtime lexical-scope implementation documentation
  HotSpot deoptimization/rematerialization sources
~~~

The common evidence supports architectural viability, not a Protos performance
claim.

## 7. Complete candidate set

### A — keep eager semantic context allocation

Every invocation eagerly creates its stable
`ProtosExecutionContextValue`. I068 frame-backed lexical authority remains
unchanged.

This is the current implementation and therefore has the strongest correctness
evidence and simplest failure model.

Future-regret case: allocation profiling later proves that the eager wrapper and
its transient default authority/map survive partial escape analysis and are a
material cost in high-call-rate workloads.

Escape path: migrate to Candidate B without changing semantics.

### B — lazy guest object with eager stable ContextCore

Each semantic invocation creates a small non-guest-visible **ContextCore**
before parameter/default execution.

The name `ContextCore` here is architectural, not an implementation mandate.

The core must own exactly the state that must exist before guest wrapper
materialization:

~~~text
ContextCore
  stable one-invocation semantic identity reservation
  lexical-parent relation
  Context delegation-parent information needed for projection
  OPEN/CLOSED/FROZEN state
  one ProtosLexicalBindingAuthority
    static bindings -> I068 frame-backed authority
    dynamic-only bindings -> one dynamic overflow
    semantic presence/order metadata
  optional exact-once ProtosExecutionContextValue wrapper
~~~

The guest wrapper is an adapter over the core. It owns no independent lexical
copy.

Required invariants:

~~~text
ONE_CONTEXT_CORE_PER_SEMANTIC_INVOCATION=YES
ONE_GUEST_WRAPPER_MAX_PER_CONTEXT_CORE=YES
ONE_SEMANTIC_BINDING_VALUE_AUTHORITY=YES
WRAPPER_MATERIALIZATION_COPIES_AUTHORITATIVE_VALUES=NO
GLOBAL_IDENTITY_REGISTRY=NO
~~~

Before wrapper materialization:

- local creation/mutation operates on the core's single authority;
- dynamic overflow lives in that authority;
- OPEN/CLOSED/FROZEN transitions live on the core;
- Closure capture stores a core reference, not a wrapper;
- the debugger projects the core/authority directly; and
- Task/suspension paths retain activation/core/frame state as required.

Materialization is forced when guest semantics require an actual context value,
for example evaluating `context`, exporting that value to tooling, or invoking
ordinary object/reflection behavior on it.

Existing Java-reference-based ordinary-object identity can remain correct if
the core guarantees exact-once wrapper creation and all later observations
reuse that wrapper.

This candidate is architecturally valid.

Future-regret case: the core grows until it is almost the entire wrapper, or
future semantics require an actual object projection for nearly every
invocation, making the indirection permanent overhead with no allocation saving.

Escape path: eagerly create the wrapper at activation establishment while
keeping the same core/authority seam, then simplify if evidence supports it.

### C — use ProtosActivation/materialized frame itself as the lazy identity substrate

A frame-only version is not complete: a Truffle frame does not own Protos
object state, dynamic overflow, mutation state or guest identity. Once those are
added as side state it collapses into Candidate B.

An activation-backed version is technically more complete, but it couples
semantic context lifetime to control/runtime state. An escaped context or
captured Closure could then retain receiver/method metadata, Task/deferred
operation links and dynamic-control state that are not semantically part of the
context object.

This is a non-compensating coupling risk.

Future-regret case: activation/control state grows, and escaping contexts retain
large or ownership-sensitive runtime graphs.

Escape path: split the semantic state into Candidate B's ContextCore. That
migration is more invasive than adopting the separation initially.

### D — defer / keep current pending evidence

Keep current eager physical wrapper creation, but do **not** ratify eager
allocation as a permanent architectural requirement.

Candidate B remains an available, semantically valid optimization architecture.
No implementation begins until current allocation evidence shows a material
reason to pay for its extra substrate, indirection and refactor.

Future-regret case: measurement later demonstrates that the eager wrapper/default
authority allocation is a dominant or material hot-path cost and deferral has
merely postponed an obvious win.

Escape path: ratify Candidate B within PLAT037 after the discriminating evidence,
then allocate implementation work.

## 8. GITHUB010 candidate scoring

Scores are 1–5 with confidence H/M/L. They are comparison aids, not arithmetic
selection authority.

| Dimension | A eager | B ContextCore + lazy wrapper | C activation/frame substrate | D defer |
| --- | ---: | ---: | ---: | ---: |
| Correctness / invariant preservation | 5/H | 4/M | 3/L | 5/H |
| Protos alignment | 5/H | 5/H | 2/M | 5/H |
| Pay for present need | 2/M | 4/M | 4/L | 4/H |
| Grow as needed | 3/M | 4/M | 2/M | 5/H |
| Future-option resilience | 4/H | 5/M | 2/M | 5/H |
| Scalability | 3/M | 4/M | 3/L | 3/M |
| Conceptual simplicity | 5/H | 2/M | 2/M | 5/H |
| Portability / implementation freedom | 5/H | 4/M | 3/L | 5/H |
| Runtime / resource cost | 3/L | 4/L | 3/L | 3/L |
| Failure / operability | 5/H | 3/M | 2/M | 5/H |
| Migration / reversibility | 5/H | 3/M | 2/M | 5/H |
| Evidence maturity / implementation risk | 5/H | 4/M | 2/L | 5/H |

Non-compensating red flags:

~~~text
A:
  no semantic red flag;
  possible unnecessary allocation remains an unmeasured runtime risk.

B:
  mandatory new per-invocation identity/state substrate and exact-once
  projection machinery are real complexity even when the wrapper is never made.

C:
  semantic context lifetime becomes coupled to activation/control/Task lifetime.

D:
  no architecture debt is added, but a proven allocation hotspot would turn
  continued deferral into avoidable runtime debt.
~~~

## 9. Counterexamples and failure modes

Candidate B fails if any of the following is permitted:

1. the wrapper gets an independent local-slot map while the core/frame remains
   authoritative;
2. wrapper materialization copies frame/dynamic values into a second writable
   semantic store;
3. Closures sometimes capture wrappers and sometimes cores, creating two
   lexical-context representations with divergent lifetime/identity behavior;
4. the debugger materializes a wrapper merely to enumerate ordinary scope and
   therefore perturbs the very allocation policy being optimized;
5. OPEN/CLOSED/FROZEN transitions before materialization are not preserved by
   the later wrapper;
6. a wrapper is recreated after suspension/resumption or after the activation
   returns, changing ordinary-object identity;
7. late dynamic creation is stored somewhere other than the same authority seen
   after wrapper materialization;
8. object-construction activations are accidentally converted to ContextCore
   even though their creation target is the actual object under construction;
9. module contexts are generalized to lazy projection without preserving the
   cache-before-execute/cyclic-import identity rule; or
10. activation-backed identity makes escaped contexts retain unrelated Task,
    dynamic-control or deferred-operation state.

## 10. Required stress analysis

### Guest context observation

- Immediate `context`: B materializes the exact-one wrapper immediately.
- Never observed: B avoids the wrapper but still pays for ContextCore and any
  frame/authority state required by actual lexical execution.
- Repeated observation: every read returns the same wrapper.
- Escape: the wrapper retains/reaches the core; the core does not need to retain
  the entire activation merely because the guest context escaped.
- Identity/IdentityMap/default hash: the same wrapper is reused, preserving the
  current ordinary-object identity implementation.

### Lexical state

- Closure capture by reference: capture the core, not the wrapper.
- Captured mutation: reads/writes go to the same authority.
- Late dynamic creation: create directly in the core's dynamic overflow without
  forcing wrapper materialization.
- Late nearer retargeting: presence checks walk the semantic core chain exactly
  as I068 currently checks nearer context membership.
- `PRESENT(null)`: preserved through independent presence/cleared state.
- Sequential/default parameters: ContextCore exists before defaults; each
  parameter becomes present only at the existing binding point. A default that
  evaluates `context` simply materializes the wrapper.
- receiver/methodHome: remain activation metadata.
- object construction: unchanged; the actual constructed object remains the
  creation target and is not replaced by ContextCore.

### Control and concurrency

- non-local return: unchanged `ProtosReturnHome`/transfer path.
- Task ownership: ContextCore is lexical semantic state, not Task identity.
- suspension/resumption: existing `ContinuationResult`/materialized-frame
  state retains the core through activation reachability; no wrapper is required.
- dynamic handlers/ensure/cancellation: remain separate dynamic-control state.
- long-lived Tasks: may avoid a wrapper, but already retain continuation/frame
  state; therefore the incremental memory benefit is not assumed to be large.

### Tooling

- debugger attach before guest context observation: ordinary semantic scope
  enumeration/read/write should project core/authority state and need not force
  a wrapper.
- debugger after activation return: a debugger can only project state still
  reachable through a retained closure/context/task/runtime record. If the
  debugger requests the actual guest context value, B materializes it exactly
  once.
- Core reflection: reflection on the context value necessarily means a wrapper
  already exists; it projects the core rather than copying it.

### Scale and deployment

- many short-lived calls: this is B's strongest possible win, but the current
  wrapper/default authority may already be scalar-replaced by Graal; measure.
- many escaping Closures: frames/core already escape, so wrapper avoidance may
  save less than in non-capturing calls.
- many Polyglot Contexts: cores are Context-local ordinary runtime state; no
  static/global semantic-identity registry is allowed.
- Native Image/AOT: B requires no reflective global registry or unsafe identity
  mechanism, but its cost/benefit under AOT remains unmeasured and must not be
  inferred from JVM JIT behavior.

## 11. Smallest-sufficient / pay-for-what-you-need analysis

The smallest correct lazy architecture is not “allocate nothing until
`context`.” Semantic identity/state exists from activation establishment.

The minimum lazily projected design still pays per invocation for:

~~~text
ContextCore identity
lexical parent/topology
mutation state
authority reference
optional dynamic-overflow/ordering state according to actual authority needs
exact-once wrapper slot/state
~~~

It may avoid:

~~~text
ProtosExecutionContextValue wrapper
wrapper-specific ordinary-object projection fields
the current transient map-backed execution-context authority/map
  when I068 immediately replaces it with frame-backed authority
~~~

Whether those avoided allocations survive Graal partial escape analysis is not
known.

The strongest pay-for-what-you-need implementation direction, if B is ever
selected, is therefore:

- keep dynamic overflow lazy within the core/authority where possible;
- do not allocate a global identity token/UUID/registry;
- use the core object itself as the internal identity reservation;
- materialize the guest wrapper exactly once only at a real semantic/tooling
  value boundary; and
- keep static lexical values in I068's existing frame authority.

## 12. Performance evidence and smallest discriminating experiment

No Protos measurement in this investigation establishes that eager context
wrapper allocation materially affects throughput, latency, allocation rate or
GC pressure.

External evidence only establishes plausibility:

- Squeak/Cog and CPython reduced costs by avoiding or delaying full frame/context
  objects;
- Graal/HotSpot can scalar-replace and later rematerialize objects, so some
  apparently eager Java allocations may already disappear after optimization.

Neither result can be transferred quantitatively to Protos.

Before Candidate B implementation, run the smallest experiment that attributes
current allocations without changing semantics or architecture.

Required call shapes:

~~~text
1. empty/near-empty invocation, context never evaluated
2. static parameters/locals, context never evaluated
3. Closure capture, context never evaluated
4. dynamic local creation, context never evaluated
5. context evaluated and escaped (control)
~~~

Measure after warmup, with allocation attribution where available:

~~~text
ProtosExecutionContextValue
ProtosMapBackedLexicalBindingAuthority
its LinkedHashMap
ProtosFrameLexicalBindingAuthority
MaterializedFrame / continuation-related frame state
throughput
allocated bytes/op or equivalent class allocation counts
GC pressure where stable enough to interpret
optimized IR / partial-escape evidence if practical
~~~

The experiment must answer first:

> What allocation and work actually survives optimization in the
> context-never-observed cases?

Only then should a performance owner determine whether the surviving cost is
material enough to justify Candidate B. This does not reopen PERF010-A to decide
architectural correctness.

~~~text
MEASUREMENT_REQUIRED_BEFORE_IMPLEMENTATION=YES
ARBITRARY_SUCCESS_THRESHOLD_DEFINED_HERE=NO
~~~

## 13. Implementation consequences if Candidate B is later ratified

No implementation is authorized by this investigation.

If B is later selected, the likely internal migration is:

~~~text
ProtosActivation:
  internal current lexical context -> ContextCore
  guest context operation -> materialize/reuse exact-one wrapper

ProtosClosureValue:
  captured lexical contexts -> ContextCore references

ProtosLexicalFallback:
  operate on a lexical-context/core abstraction, not guest wrapper identity

ProtosFrameLexicalBindingAuthority:
  attach to ContextCore/authority substrate rather than requiring wrapper host

ProtosDebuggerScope:
  project ContextCore/authority directly

ProtosExecutionContextValue:
  become exact-one guest/object adapter over ContextCore

ProtosIdentity:
  may remain reference-based for contexts if exact-one wrapper is guaranteed;
  no separate guest-visible identity primitive is required
~~~

Module contexts should remain eager in the first bounded implementation unless
their stronger cache-before-execute/cyclic-import observation boundary is
separately proven compatible with the same core abstraction. Object construction
remains outside this transformation.

Required tests include repeated `context` identity, context escape after return,
Closure capture, debugger-before-context, dynamic creation before materialization,
OPEN/CLOSED/FROZEN transitions, suspension/resumption, PRESENT(null), late
retargeting, sequential/default parameter visibility, receiver/methodHome,
non-local return and object construction.

## 14. Migration and reversibility

A -> B is semantically non-breaking but internally broad because the current
code uses `ProtosObjectValue` references as lexical-context identity in several
subsystems.

B -> A is intentionally easy if the core remains the single authority: create
the wrapper eagerly at activation establishment and keep the same authority.

B must not encode physical laziness into the normative specification. The
optimization policy remains an implementation choice.

C -> B is substantially worse because context escape/lifetime must first be
separated from activation/control lifetime.

D preserves both A and B as available future choices and therefore has the
highest reversibility.

## RECOMMENDATION — PENDING PROJECT OWNER APPROVAL

Select:

~~~text
CANDIDATE=D
NAME=DEFER_KEEP_CURRENT_PENDING_EVIDENCE
~~~

This recommendation means:

- keep the current eager `ProtosExecutionContextValue` architecture now;
- do not declare eager physical allocation semantically required or permanently
  preferred;
- record Candidate B as architecturally valid under the exact ContextCore
  invariants above;
- run allocation/partial-escape attribution before authorizing implementation;
- if the evidence establishes a material surviving cost, return to PLAT037 for
  owner selection of Candidate B and only then allocate implementation work.

The recommendation is based on an asymmetry:

~~~text
B_CORRECTNESS_PLAUSIBILITY=STRONG
B_PERFORMANCE_VALUE_IN_PROTOS=NOT_ESTABLISHED
B_IMPLEMENTATION_COMPLEXITY=REAL
~~~

It is therefore premature to pay for a new permanent per-invocation substrate
merely to avoid an allocation that Graal may already eliminate in important
hot paths.

## Strongest argument against the recommendation

The current source demonstrably constructs a
`ProtosExecutionContextValue`, its initial map-backed authority and a
`LinkedHashMap` before every ordinary Closure/method activation, while I068 may
then install a different frame-backed authority. Mature runtimes such as
Squeak/Cog and GraalPy deliberately avoid analogous full context/frame objects
until observation requires them.

If Protos allocation profiling shows that these objects survive escape analysis
and constitute a material fraction of call cost or memory pressure, then
continued deferral would be needless conservatism. The escape path is already
defined: ratify Candidate B under PLAT037, preserving the same semantics, and
allocate a bounded implementation owner only after that approval.

## Final gate

~~~text
PLAT037_INVESTIGATION=COMPLETE
PLAT037_RECOMMENDED_CANDIDATE=D
PLAT037_RECOMMENDATION_STATUS=PENDING_PROJECT_OWNER_APPROVAL

IMPLEMENTATION_AUTHORIZED=NO
IMPLEMENTATION_OWNER_ALLOCATED=NO
NORMATIVE_SPEC_CHANGE_AUTHORIZED=NO

NEXT_GATE=
  EXPLICIT_PROJECT_OWNER APPROVAL OR REJECTION OF CANDIDATE D
~~~

STOP.
