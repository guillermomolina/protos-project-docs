# PLAT040 — Cross-Truffle hot-call decision evidence

Status: FINAL DECISION EVIDENCE

Decision: PLAT040 / guillermomolina/protos#718

Ratified candidate: **F′ — guarded hot call with frame-argument invocation and conditional guest-state materialization**

Approval provenance:

~~~text
DATE=2026-09-25
OWNER=guillermomolina
APPROVAL="apruebo f'"
~~~

Product evidence identity:

~~~text
PROTOS_REVISION=0c9307240796260882ddca27611c91b7ffa4ce3b
PROTOS_VERSION=0.3.88-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.3.4.1
~~~

External source revisions inspected in the final comparative pass:

~~~text
APPLE_PKL_REVISION=43e132a38333b3236e925a004cc7a0bdd64d9656
ORACLE_GRAAL_REVISION=903d65852bd1a2055126dc895f9727f936b0bd31
ORACLE_GRAALPYTHON_REVISION=cbed856fe9621b4b0c01b6a020b31440c1cf19c8
ORACLE_GRAALJS_REVISION=fa11bf0c87ee63778c35b12705423b5b68003f72
TRUFFLERUBY_REVISION=0e6fa6a950dce7154d54f3c9c63056c4eb925ffd
TRUFFLESQUEAK_REVISION=4ff18896b3a243c9aabde6217b0d9b5e57e91876
FASTR_REVISION=f9d9199e51b371f076b19456933ee395d28360bf
ENSO_REVISION=4135f929f1046ee4ab62d9d09c155377e8542a1f
~~~

This evidence is non-normative. It records the comparative runtime basis for the selected platform architecture.

## Owner evidence standard

The project owner explicitly required the final PLAT040 recommendation to be based on how comparable Truffle languages actually solve the same hot-path problem, with special weight on GraalVM-native implementations and Apple Pkl.

The governing review rule was:

~~~text
A locally plausible Protos-only optimization hypothesis is suspicious unless
materially comparable Truffle implementations provide strong evidence for the
same architectural pattern.
~~~

Candidate F′ contains only principles that survived that evidence filter. Exact Protos Java representation details that lacked comparable-runtime support were deliberately left unselected.

## Current Protos hot-call anatomy

At the pinned product baseline, the ordinary call path contains two distinct late-stabilization problems.

### Stable semantic selection is recognized late

Representative send preparation:

~~~text
ordinary send
  -> D013 lookup / delegation traversal
  -> selected Closure + methodHome
  -> generic implementation classification
  -> invocation preparation
  -> target selection
  -> DirectCallNode
~~~

The direct target exists, but general semantic and implementation work precedes it on the repeated path.

### Per-invocation state is physically rich

`ProtosActivation.forClosureInvocation` and `forImmediateMethodInvocation` establish state including:

- fresh execution-context object;
- captured lexical contexts;
- receiver;
- Prelude;
- a frozen guest Array built from supplied arguments;
- return-home relationship;
- `methodHome`;
- Actor/module state;
- current module key;
- execution domain; and
- inherited dynamic-control state where applicable.

`PreparedClosureCall` then carries ordinary call state plus many mutually exclusive structured modes, including Boolean control, ensure/error handling, while, collection iteration, map operations, import/module initialization, native/control cases and continuation-related state before final target entry.

The final ordinary direct specialization still calls the selected body with the prepared activation only after this carrier has been constructed.

## Normative implementation freedom

The governing Protos specification requires exact observable semantics but does not require the current physical representation.

In particular:

- a Closure invocation has a fresh execution-context object semantically;
- capture and context escape remain by reference;
- parameters become slots of that activation context;
- `this`, `methodHome`, return-home and dynamic-control behavior remain exact; and
- the implementation may scalar-replace, virtualize, share immutable backing storage or otherwise avoid physical activation/Array copies when identity, reflection, mutation, capture, escape, evaluation order and other observations remain exact.

The complete ordinary supplied argument vector is not exposed through an ambient guest `args` intrinsic. A rest binding, when present, has its separately required fresh frozen standard Array semantics.

Therefore neither the current `ProtosActivation` aggregate nor an eager guest Array for every supplied vector is normative merely because the current implementation uses them.

## Apple Pkl

Primary inspected surfaces:

~~~text
pkl-core/src/main/java/org/pkl/core/ast/expression/member/InvokeMethodVirtualNode.java
pkl-core/src/main/java/org/pkl/core/ast/lambda/ApplyVmFunction0Node.java
pkl-core/src/main/java/org/pkl/core/ast/lambda/ApplyVmFunction1Node.java
pkl-core/src/main/java/org/pkl/core/ast/lambda/ApplyVmFunction2Node.java
pkl-core/src/main/java/org/pkl/core/runtime/VmFunction.java
pkl-core/src/main/java/org/pkl/core/ast/expression/generator/GeneratorWhenNode.java
pkl-core/src/main/java/org/pkl/core/stdlib/base/MapNodes.java
~~~

Observed architecture:

- virtual-method specializations cache receiver class, resolved method and direct call state;
- generic fallback performs resolution when the cache does not apply;
- function application guards `function.getCallTarget() == cachedCallTarget` and uses `DirectCallNode`;
- `VmFunction` retains captured/enclosing frame state on the function value rather than reconstructing a rich activation aggregate for every callback;
- structural `when` executes a Boolean condition node followed by a direct branch;
- collection `map`/`filter`/`fold`/`every`/`any` loops invoke callbacks through cached `ApplyVmFunctionNNode` children.

Relevant result:

~~~text
STABLE_SELECTION_BEFORE_DIRECT_CALL=YES
CALLABLE_OWNED_CAPTURED_STATE=YES
UNIVERSAL_RICH_ACTIVATION_BEFORE_CALLBACK=NO_EVIDENCE
STRUCTURED_CONTROL_REENTERS_GENERIC_SEND_EACH_ITERATION=NO
~~~

## GraalPy

Primary inspected surfaces:

~~~text
graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/function/PArguments.java
graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/call/CallDispatchers.java
graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/ExecutionContext.java
graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/frame/PFrame.java
graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/frame/MaterializeFrameNode.java
~~~

Observed architecture:

- Python call state is represented through an `Object[]` frame-argument ABI;
- direct function invocation caches function/code/target state under stability assumptions;
- `ExecutionContext.CallContext` checks callee caller flags;
- the `!needsFrameReference(callerFlags)` specialization does not pass caller-frame state;
- a real `PFrame` is materialized only when caller flags require a Python frame/locals/lasti;
- frame references can later be marked escaped when reflective/traceback operations require it.

This is strong evidence against treating a guest-visible frame/context as requiring eager rich-object allocation on every ordinary call.

Relevant result:

~~~text
FRAME_ARGUMENT_CALL_ABI=YES
OPTIONAL_CALLER_FRAME_STATE=CONDITIONAL
GUEST_VISIBLE_FRAME_MATERIALIZATION=CONDITIONAL
DEBUGGER_REFLECTION_REQUIRE_EAGER_FRAME_ON_EVERY_CALL=NO
~~~

## GraalJS

Primary inspected surfaces:

~~~text
graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/runtime/JSArguments.java
graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/function/JSFunctionCallNode.java
graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/runtime/builtins/JSFunctionObject.java
~~~

Observed architecture:

- ordinary calls use compact `JSArguments`;
- cached function-instance/function-data nodes own a `DirectCallNode`;
- the direct cache executes `callNode.call(arguments)`;
- function objects retain enclosing `MaterializedFrame` state where required.

Relevant result:

~~~text
FRAME_ARGUMENT_CALL_ABI=YES
CACHED_TARGET_DIRECT_CALL=YES
CAPTURED_FRAME_ASSOCIATED_WITH_FUNCTION=YES
~~~

## Espresso

Primary inspected surface:

~~~text
espresso/src/com.oracle.truffle.espresso/src/com/oracle/truffle/espresso/nodes/bytecodes/InvokeVirtual.java
~~~

Observed architecture:

- specializations cache receiver-class-dependent resolved `MethodVersion`;
- class-hierarchy and method-redefinition assumptions guard stable selection;
- the direct path invokes a cached `DirectCallNode`;
- the megamorphic fallback performs generic method lookup and uses `IndirectCallNode`.

This directly demonstrates that mutable/redefinable dispatch does not require repeating full lookup on every stable hit.

Relevant result:

~~~text
MUTABLE_DISPATCH_WITH_ASSUMPTION_GUARDS=YES
GENERIC_LOOKUP_REQUIRED_ON_MONOMORPHIC_HIT=NO
~~~

## Sulong / LLVM

Primary inspected surface:

~~~text
sulong/projects/com.oracle.truffle.llvm.runtime/src/com/oracle/truffle/llvm/runtime/nodes/func/LLVMDispatchNode.java
~~~

Observed architecture:

- cached function code or descriptor specializes to `DirectCallNode`;
- ordinary arguments cross as `Object[]`;
- indirect resolution remains as fallback.

Relevant result:

~~~text
CACHED_FUNCTION_DESCRIPTOR_OR_CODE=YES
DIRECT_CALL_WITH_ARGUMENT_ARRAY=YES
GENERIC_FALLBACK=YES
~~~

## GraalWasm

Primary inspected surface:

~~~text
wasm/src/org.graalvm.wasm/src/org/graalvm/wasm/nodes/WasmDirectCallNode.java
~~~

Observed architecture:

- direct call targets are resolved before the hot call;
- the node retains the target;
- return-call machinery is only entered when the target contains return calls.

Wasm is less semantically comparable than the dynamic-language runtimes but reinforces early target resolution and optional-state separation.

## FastR

Primary inspected surfaces:

~~~text
com.oracle.truffle.r.nodes/src/com/oracle/truffle/r/nodes/function/call/CallRFunctionNode.java
com.oracle.truffle.r.runtime/src/com/oracle/truffle/r/runtime/RArguments.java
~~~

Observed architecture:

- `CallRFunctionNode` owns a cached `DirectCallNode`;
- `RArguments.create` builds Truffle frame arguments containing function, caller/enclosing frame, signatures and user values;
- the selected call target is entered directly.

Relevant result:

~~~text
FRAME_ARGUMENT_CALL_ABI=YES
ENCLOSING_CALLER_STATE_CAN_TRAVEL_AS_FRAME_ARGUMENTS=YES
~~~

## TruffleRuby

Primary inspected surfaces:

~~~text
src/main/java/org/truffleruby/language/arguments/RubyArguments.java
src/main/java/org/truffleruby/language/methods/CallInternalMethodNode.java
src/main/java/org/truffleruby/language/yield/CallBlockNode.java
src/main/java/org/truffleruby/language/control/FrameOnStackNode.java
src/main/java/org/truffleruby/language/control/BreakNode.java
~~~

Observed architecture:

- `RubyArguments` explicitly defines the frame-argument ABI;
- it carries declaration frame, special-variable state, method, declaration context, an optional frame-on-stack marker, `self`, block, descriptor and user arguments;
- cached method calls guard call-target/method state under a method assumption and use `DirectCallNode`;
- cached block calls guard `block.callTarget == cachedCallTarget`, pack frame arguments and direct-call the block;
- the non-local-control `FrameOnStackMarker` may be null and is created by the relevant control structure rather than proving that all calls need a universal return-control object;
- `repackForCall` explicitly discusses separating argument-array lifetimes to improve escape analysis.

Relevant result:

~~~text
BLOCKS_AND_CAPTURE_WITH_FRAME_ARGUMENT_ABI=YES
METHOD_IDENTITY_AND_SELF_AS_FRAME_ARGUMENTS=YES
NONLOCAL_CONTROL_MARKER_CAN_BE_OPTIONAL=YES
UNIVERSAL_ACTIVATION_OBJECT_REQUIRED_BY_BLOCK_CONTROL=NO_EVIDENCE
~~~

## TruffleSqueak

Primary inspected surfaces:

~~~text
src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/dispatch/DispatchValueNode.java
src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/context/GetOrCreateContextWithoutFrameNode.java
src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/util/FrameAccess.java
~~~

Observed architecture:

- block dispatch caches the compiled block under `getCallTargetStable()`;
- the direct specialization creates closure frame arguments and invokes a `DirectCallNode`;
- the megamorphic fallback uses `IndirectCallNode`;
- the context helper explicitly “gets context or lazily initializes one if necessary”;
- a `ContextObject` is therefore not proof of an eager context object on every execution path.

This is especially relevant because Squeak/Smalltalk exposes first-class execution contexts and block semantics while still separating frame execution from context-object materialization.

Relevant result:

~~~text
BLOCK_DIRECT_CALL_WITH_STABILITY_ASSUMPTION=YES
LAZY_CONTEXT_OBJECT=YES
FIRST_CLASS_CONTEXT_REQUIRE_EAGER_CONTEXT_OBJECT=NO
~~~

## Enso

Primary inspected surface:

~~~text
engine/runtime/src/main/java/org/enso/interpreter/node/callable/ExecuteCallNode.java
~~~

Observed architecture:

- cached `function.getCallTarget()` selects direct or stronger inlineable paths;
- ordinary call state is assembled as argument arrays;
- the source explicitly describes the uncached indirect specialization as much slower and generally to be avoided.

## Common architecture extracted from the survey

Across the materially comparable runtimes, the recurring steady-state pattern is:

~~~text
resolve / specialize
  -> retain selected executable state
  -> guard / assumption / version
  -> DirectCallNode or stronger inlineable path
  -> compact frame arguments
  -> callable-owned captured state
  -> optional control/frame/context state only when needed
  -> lazy / escape-driven rich guest projection
  -> generic fallback on invalidation or megamorphism
~~~

No surveyed materially comparable implementation established the Protos baseline pattern as its intended monomorphic steady state:

~~~text
full general semantic lookup
  + universal rich invocation construction
  + optional-mode classification
  + late target stabilization
  + DirectCallNode
~~~

## Classification of Protos mechanisms

| Protos mechanism | Comparative result | PLAT040 classification |
|---|---|---|
| repeat authoritative D013 lookup on every stable hit | contradicted by Pkl/Espresso/Ruby/JS/Py patterns | remove from guarded hit |
| target cached only after generic classification | contradicted broadly | stabilize earlier |
| `DirectCallNode` under guard/assumption | common pattern | required |
| capture state associated with callable/frame | Pkl/JS/Ruby/Squeak | required direction |
| frame-argument call ABI | Py/JS/Ruby/R/FastR/Enso/Sulong | required direction |
| rich universal `ProtosActivation` as hot-call carrier | no comparable precedent found | rejected from ordinary hot-path contract |
| eager guest execution-context object on every ordinary call | contradicted by GraalPy/Squeak | not required |
| eager guest Array for complete supplied vector | no common precedent; Protos spec permits avoidance | representation accident |
| fresh return-home object on every ordinary owning call | Ruby shows optional non-local-control marker pattern | representation is not presumed necessary |
| full dynamic-control state in every hot-call carrier | optional-state patterns dominate | must be evidence-gated/conditional |
| universal `PreparedClosureCall` with many mutually exclusive modes | no comparable precedent found | rejected from ordinary hot-path contract |
| `methodHome` semantic provenance | comparable method/declaration metadata exists elsewhere | preserve semantics, representation open |
| Task/Actor/Process/domain state | no universal external analogue | preserve until specific evidence supports a narrower representation |
| suspension/continuation machinery on ordinary nonsuspending path | other runtimes separate optional paths | remove unrelated physical cost from ordinary path where semantically safe |

## Candidate evolution

The first PLAT040 recommendation, Candidate F, retained the current activation/context construction in order to isolate dispatch cost.

After the owner requested a stronger comparison specifically against other Truffle languages, the final research pass established that invocation representation itself is a second cross-runtime divergence.

Therefore:

~~~text
OLD_CANDIDATE_F=
  GUARDED_SELECTION
  + CURRENT_RICH_INVOCATION_CONSTRUCTION

OLD_CANDIDATE_F_STATUS=WITHDRAWN_AS_INSUFFICIENT

FINAL_CANDIDATE_F_PRIME=
  GUARDED_SELECTION
  + FRAME_ARGUMENT_INVOCATION
  + CONDITIONAL_GUEST_STATE_MATERIALIZATION
  + OPTIONAL_STATE_SEPARATION
~~~

This delta was surfaced to the owner before approval.

## PLAT037 evidence gate

PLAT037 deferred lazy execution-context physical materialization because the benefit was not established.

The final PLAT040 evidence changes that state:

- the broad gap remains after I068;
- GraalPy uses conditional frame/PFrame materialization;
- TruffleSqueak uses lazy `ContextObject` initialization;
- Pkl/JS/Ruby retain lexical/captured state without reconstructing an equivalent rich guest context before every direct call.

Therefore:

~~~text
PLAT037_EVIDENCE_GATE_SATISFIED=YES
PLAT037_SPECIFIC_CONTEXTCORE_CANDIDATE_ESTABLISHED=NO
PLAT037_REOPEN_REQUIRED=NO
~~~

The old Protos-specific `ContextCore` representation is not promoted merely because it was previously considered viable. The selected invariant comes from the common external pattern and leaves the exact Protos Java representation to implementation evidence.

## GITHUB021 invariant/delta check

Result:

~~~text
D013_AUTHORITY=PRESERVED
STANDARD_PROTOCOLS_REMAIN_ORDINARY=PRESERVED
FRESH_CONTEXT_SEMANTICS=PRESERVED
CAPTURE_BY_REFERENCE=PRESERVED
THIS_METHODHOME_ARGUMENT_SEMANTICS=PRESERVED
NONLOCAL_RETURN_DYNAMIC_CONTROL=PRESERVED
TASK_ACTOR_PROCESS_DOMAIN=PRESERVED
SUSPENSION_CONTINUATION=PRESERVED
PLAT036_I068_FRAME_AUTHORITY=PRESERVED
PLAT037_SPECIFIC_REPRESENTATION_NOT_SILENTLY_RESELECTED=PASS
PLAT039_PE_VISIBLE_GUEST_KERNEL=PRESERVED
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
MATERIAL_DELTA_FROM_OLD_F=SURFACED_AND_EXPLICITLY_APPROVED
DECISION_INVARIANT_CONSISTENCY=PASS
~~~

## Performance interpretation

PLAT040 establishes architecture, not an exact speedup.

The comparative evidence supports high confidence that Protos' current ordinary call path is structurally anomalous for Truffle and that F′ is the correct architectural direction.

It does not prove a numerical transition such as `500x -> 1x`, nor does it assign an attributable fraction to any one component.

The implementation/performance falsifier is structural first:

~~~text
ordinary stable compiled call
  -> no repeated general D013/classification on guarded hit
  -> no universal rich invocation carrier
  -> no eager guest context solely because a call occurred
  -> no guest supplied Array unless semantically observed
  -> direct/inlineable target remains PE-visible
~~~

PERF010-A then measures the attributable timing effect on the established common workloads.

## Final decision evidence

~~~text
PLAT040_HOT_PATH_ARCHITECTURE=ESTABLISHED
CROSS_TRUFFLE_COMMON_PATTERN=ESTABLISHED
RELEVANT_COUNTEREXAMPLE_MATCHING_PROTOS_STEADY_STATE=NONE_FOUND

GUARDED_SELECTION=ESTABLISHED
FRAME_ARGUMENT_INVOCATION_ABI=ESTABLISHED
CONDITIONAL_GUEST_CONTEXT_MATERIALIZATION=ESTABLISHED
OPTIONAL_STATE_SEPARATION=ESTABLISHED
GENERIC_FALLBACK=REQUIRED

OLD_CANDIDATE_F_STATUS=WITHDRAWN_AS_INSUFFICIENT
RATIFIED_CANDIDATE=F_PRIME
OBSERVABLE_PROTOS_SEMANTIC_CHANGE_REQUIRED=NO

PLAT037_EVIDENCE_GATE_SATISFIED=YES
PLAT036_COMPATIBLE=YES
PLAT039_COMPATIBLE=YES

IMPLEMENTATION_READY=YES
IMPLEMENTATION_OWNER=DEDICATED_IXXX_AFTER_RATIFICATION
~~~

