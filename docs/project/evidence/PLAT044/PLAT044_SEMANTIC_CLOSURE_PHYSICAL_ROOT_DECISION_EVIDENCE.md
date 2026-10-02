# PLAT044 — Semantic Closure activation versus physical RootTag decision evidence

Status: **FINAL DECISION EVIDENCE — B′ RATIFIED**

Date: 2026-10-02

## Identity

~~~text
WORK_ITEM=PLAT044/#766
PARENT=PERF026/#765
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=8ea87fb1794599247f0e95fd5570d7dd7e08a52f
PROTOS_VERSION=0.3.134-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1

RECOMMENDED_CANDIDATE=B_PRIME
RECOMMENDATION_STATUS=APPROVED_AND_RATIFIED
PRODUCT_CHANGES=NONE
SPECIFICATION_CHANGES=NONE
~~~

This record retains the comparative evidence for the PLAT044 decision packet.
The project owner subsequently approved exact Candidate B′ on 2026-10-02, and
that approval includes the explicit PLAT026 tooling/frame delta defined by B′.
The durable selected architecture is published separately under
`docs/project/decisions/platform/PLAT044_SEMANTIC_CLOSURE_PHYSICAL_ROOT_BOUNDARY.md`.

## Exact problem

PERF026-A established that Protos has several high-value standard-control cases
whose semantic model is ordinary message/callback invocation while mature
Smalltalk implementations commonly represent the eligible common form as local
branch/loop bytecode:

- standard Boolean control: `ifTrue`, `ifFalse`,
  `ifTrueIfFalse`, `and`, `or`;
- standard `Closure.whileTrue`; and
- synchronous standard collection `each` families.

PLAT040/I072 already authorize the first half of the optimization:

~~~text
ordinary semantic lookup/selection
    -> exact selected standard behavior
    -> guarded specialization
    -> exact generic fallback
~~~

PLAT043/PERF025-C2B removed the extra untagged Boolean orchestration helper root,
but a reached ordinary Closure callback still executes through its own semantic
Closure root/CallTarget. The post-C2B recursive shape can therefore remain:

~~~text
R_i -> B_i -> R_i+1
~~~

PLAT044 asks whether an eligible immediate literal Closure callback may keep its
full Protos activation semantics while its physical body executes as a local
resumable region in the caller root.

The issue is not whether a branch can be emitted. The issue is whether Protos may
decouple:

~~~text
semantic Closure activation
~~~

from:

~~~text
distinct Truffle FrameInstance
+ RootCallTarget
+ automatically RootTagged physical Bytecode root
~~~

for a narrowly proven standard-control case.

## Current Protos evidence

### PLAT026 current invariant

PLAT026 Candidate C-prime-plus currently classifies these as semantic roots:

~~~text
top-level/public source
module body
Closure activation
~~~

and requires semantic Bytecode roots to receive truthful automatic RootTag while
implementation/helper roots remain hidden.

The exact owner-approved invariant includes:

~~~text
CURRENT_CLOSURE_ACTIVATION=SEMANTIC_ROOT
SEMANTIC_ROOT_AUTOMATIC_ROOT_TAG=YES
HELPER_ROOTS_HIDDEN=YES
REAL_DAP_STACK_SCOPE_VALIDATION=REQUIRED
~~~

PLAT026 is explicitly non-normative language semantics, but it is durable
tooling/runtime architecture. Changing the physical treatment of an eligible
Closure activation therefore requires an explicit PLAT026 delta; PERF026 cannot
silently do it.

PLAT026 also deliberately left future inlining/materialization representation and
any Protos-visible stack-frame semantic unselected. PLAT044 is therefore a valid
place to narrow the physical-root rule, provided the tooling delta is surfaced
and approved rather than hidden.

### PLAT041 precedent and limit

PLAT041 C-prime already established this implementation shape for Object-body
execution:

~~~text
semantic/private activation state
    + BytecodeLocal active-activation authority
    + resumable local region
    - separate helper RootCallTarget
~~~

That proves current Protos lowering can preserve a distinct activation while
executing its body physically inside another semantic root.

The precedent is not sufficient by itself because an Object-construction body is
explicitly not a Closure/function semantic root. PLAT044 concerns a Closure
activation, which PLAT026 currently exposes as a semantic root/frame.

### Current debugger projection seam

`ProtosBytecodeTagTreeNodeExports` already supplies a custom Bytecode DSL
`NodeLibrary` for TagTreeNode locations. Today it projects debugger scope from
frame argument zero's `ProtosActivation`.

Current Truffle 25.4 also exposes slow-path Bytecode-node local introspection by
bytecode index and explicitly supports custom TagTreeNode NodeLibrary
implementations for:

- receiver/function-object projection;
- local-scope projection; and
- hiding implementation locals.

Therefore an inline callback activation kept in a Bytecode local has a supported
tooling projection seam. PLAT044 does not need a new global activation registry.

The important limitation is stack-frame cardinality, not local-scope access.

## Current Truffle framework evidence

Authoritative current API references:

- GenerateBytecode:
  https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/GenerateBytecode.html
- CallTarget:
  https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/CallTarget.html
- RootNode:
  https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/nodes/RootNode.html
- TruffleRuntime:
  https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/TruffleRuntime.html
- DebugStackFrame:
  https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/debug/DebugStackFrame.html
- NodeLibrary:
  https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/interop/NodeLibrary.html
- BytecodeNode:
  https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/BytecodeNode.html

### CallTarget inlining is not parser inlining

A `RootNode` executes through a `CallTarget`. A stable direct call lets the
Truffle runtime inline the target during partial evaluation/compilation.

This preserves the semantic call target in interpreter structure even when the
compiled machine-code graph physically fuses caller and callee.

That is the common Truffle function/block optimization pattern.

### Bytecode DSL explicitly supports parser-inlined roots

The current `GenerateBytecode.enableRootTagging` documentation now states that,
for inlining performed by the parser, emitting a custom root tag with builder
methods can be useful so tools continue to work correctly for inlined calls.

Therefore:

~~~text
PARSER_INLINED_ROOT_TAG=SUPPORTED_DIRECTION
MANUAL_REIMPLEMENTATION_OF_ORDINARY_ROOT_PROLOG=STILL_NOT_RECOMMENDED
~~~

This makes selective standard-control open-coding a supported Truffle direction,
not an unsupported hack.

### RootTag is not a distinct DebugStackFrame

Current debugger source at:

~~~text
oracle/graal@903f4f5e1ef353710323c9b9c0ab4961358918a9
truffle/src/com.oracle.truffle.api.debug/src/com/oracle/truffle/api/debug/
    SuspendedEvent.java
    DebugStackFrame.java
~~~

shows:

1. the current suspended location always receives a synthetic top
   `DebugStackFrame`;
2. lower guest frames are assembled by
   `TruffleRuntime.iterateFrames(...)`;
3. each lower frame is a `FrameInstance` associated with a
   `RootCallTarget`;
4. a RootTag on the instrumentable call-node path determines whether a physical
   frame is guest-visible;
5. the synthetic top frame uses the physical root of the instrumented node as
   its underlying root/call target, although a custom NodeLibrary can override
   function/root-instance naming and scope projection.

Consequently:

~~~text
INLINE_ROOTTAG_SOURCE_STEPPING=SUPPORTED
INLINE_ROOTTAG_SCOPE_PROJECTION=SUPPORTED
INLINE_ROOTTAG_ROOT_INSTANCE_PROJECTION=SUPPORTED

SEPARATE_FRAMEINSTANCE_WITHOUT_CALLTARGET=NO
EXACT_OLD_DEBUG_STACK_CARDINALITY_FROM_ROOT_TAG_ALONE=NO
EXACT_OLD_TRUFFLE_STACKTRACE_ELEMENT_FROM_ROOT_TAG_ALONE=NO
~~~

This is the exact PLAT044 tooling delta.

A custom root tag can keep an open-coded callback visible at its current
instrumentation location, but it does not by itself manufacture a separate
`FrameInstance` or `TruffleStackTraceElement` for that callback.

### HotSpot virtual-frame precedent is not the same mechanism

HotSpot deoptimization retains debug metadata for inlined Java methods and can
construct multiple virtual/inlined Java frames from one physical compiled frame.
Current OpenJDK deoptimization source explicitly creates a sequence where each
`compiledVFrame` represents an inlined Java frame.

This proves a mature VM can preserve logical frames while eliminating physical
compiled frames.

It does not provide a ready-made Truffle-language mechanism for a parser-inlined
Closure body. HotSpot's reconstruction is compiler/deoptimization infrastructure
for methods that still exist as semantic methods in compilation metadata.

Building an equivalent Protos-specific virtual-frame stack above the generic
Truffle debugger would be a substantial new tooling architecture and is not
needed to obtain standard-control open-coding.

## Comparative language/runtime survey

External source revisions inspected in this PLAT044 pass:

~~~text
ORACLE_GRAAL_REVISION=903f4f5e1ef353710323c9b9c0ab4961358918a9
ORACLE_GRAALPYTHON_REVISION=ee204f260d0295b7478ce006afd955e313265a70
ORACLE_GRAALJS_REVISION=4c2b8dd51b23919c1d4ba72aa07ad45fbfaac2f7
TRUFFLERUBY_REVISION=c734f26543003fefd4519a29adbca62d0c711a2d
TRUFFLESQUEAK_REVISION=4ff18896b3a243c9aabde6217b0d9b5e57e91876
APPLE_PKL_REVISION=6423b2add58fd770a6486d25f833815c0c4d3db9
~~~

The existing PLAT026/PLAT040 evidence for Espresso, Sulong, GraalWasm, FastR
and Enso was rechecked where the same physical-root question is material.

### GNU Smalltalk and Squeak — strongest open-coding precedent

GNU Smalltalk documents that the compiler open-codes standard control messages
when the callback block shape is eligible. It emits jump bytecodes and performs
no message send for:

~~~text
and:
or:
ifTrue:
ifFalse:
ifTrue:ifFalse:
ifFalse:ifTrue:
whileTrue:
whileFalse:
to:do:
to:by:do:
timesRepeat:
~~~

Squeak's historical compiler MacroSelectors similarly includes Boolean control,
while loops, numeric iteration, case control and nil-aware control.

Squeak performance documentation is explicit that, for common inlined
`ifTrue:` cases, no regular method lookup is performed and no BlockClosure is
created during execution.

This is direct evidence for the principle:

> semantic Smalltalk message/block structure does not require a physical message
> send or block activation in every eligible standard-control execution.

It is also evidence that stack/debugger presentation of such compiler-inlined
blocks is allowed to differ from an ordinary separately invoked block.

### TruffleSqueak

When an actual BlockClosure value is dispatched, current TruffleSqueak caches
the block's CallTarget and uses `DirectCallNode`; the megamorphic path uses an
`IndirectCallNode`.

This is consistent with Squeak's two-level model:

~~~text
compiler-recognized standard control
    -> jump/control bytecodes

actual ordinary BlockClosure invocation
    -> CallTarget
~~~

That is the closest direct precedent for PLAT044 B-prime.

### GraalJS

Current GraalJS represents `if` and loop syntax as local control nodes. The
conditional node executes the selected branch directly in the current frame and
has dedicated control-flow instrumentation tags.

Ordinary JavaScript function invocation remains a call-target architecture with
cached direct-call nodes and generic fallback.

Lesson:

~~~text
LOCAL_CONTROL=YES
ORDINARY_FUNCTION_CALLTARGET=YES
~~~

This supports keeping B-prime restricted to a proven standard-control lowering
rather than turning arbitrary Closure calls into inline regions.

### GraalPy

Current GraalPy Bytecode DSL uses ordinary call-target/direct-call machinery for
Python function calls while branches/loops are bytecode control inside the
function root. Generator/coroutine continuation state is represented explicitly
when needed.

No evidence was found that ordinary Python function invocation is parser-inlined
into the caller while pretending to retain an independent Python frame.

### TruffleRuby

`RubyProc` retains a `RootCallTarget` plus declaration
`MaterializedFrame`. `CallBlockNode` caches the block target and invokes a
`DirectCallNode`; its hot path can request aggressive compiler inlining.

Thus Ruby block invocation uses:

~~~text
SEMANTIC_ROOTCALLTARGET=PRESERVED
COMPILED_PHYSICAL_INLINING=ALLOWED
~~~

This is strong evidence for Candidate D on arbitrary blocks, but not a
counterexample to Smalltalk-style standard-control open-coding because Ruby does
not express ordinary conditionals through Boolean block-message protocols.

### Apple Pkl

`VmFunction` retains a Pkl root node/call target and enclosing frame state.
Function application caches a stable call target and uses direct calls.

Pkl structural/control nodes are local; ordinary functions remain function call
targets.

Again, the common boundary is:

~~~text
CONTROL_REGION != GENERAL_FUNCTION_INVOCATION
~~~

### Espresso

Espresso method bodies retain Java method/root identity and RootTag-compatible
instrumentation. Method dispatch and redefinition can stabilize through cached
direct calls; method calls remain semantic method calls.

This supports Candidate D for general invocations.

### Sulong / LLVM

Sulong exposes function execution through semantic LLVM function roots with
RootTag. Helper/runtime mechanics remain below that boundary.

This again supports retaining roots for ordinary function calls and says little
against special compiler-lowered control constructs.

### GraalWasm

Each WebAssembly function has a `WasmFunctionRootNode`; direct function calls
retain a target and call it. Stack overflow is translated to Wasm's call-stack
exhaustion failure.

Tail-call/return-call support is a separately defined control mechanism rather
than a silent removal of arbitrary function frames.

This is important precedent against solving Protos recursion by erasing ordinary
Closure frames indiscriminately.

### Oracle SimpleLanguage / Bytecode DSL

SimpleLanguage still gives semantic functions/builtins root/call-target identity.
Its Bytecode DSL usage also demonstrates explicit tag regions.

The framework documentation, rather than SimpleLanguage itself, is the evidence
that parser-inlined methods may carry custom RootTag regions.

## Common pattern and relevant counterexample

The broad Truffle pattern is:

~~~text
language-native branch/loop/control
    -> local AST/bytecode region

ordinary function/block invocation
    -> RootCallTarget
    -> DirectCallNode when stable
    -> compiler may physically inline
~~~

The relevant counterexample is the Smalltalk family:

~~~text
ordinary source-level standard control message
    + compiler-recognized eligible block shape
    -> local control bytecodes
    -> no ordinary send/block invocation at runtime
~~~

Protos is specifically Smalltalk/Self-like in expressing control through ordinary
messages. Therefore copying JavaScript/Python/Ruby's syntax/control boundary
without considering the Smalltalk compiler precedent would preserve an
implementation cost merely because Protos chose the message-based source model.

The appropriate boundary is not “all Closures can inline”. It is:

~~~text
authoritative ordinary selection proves exact standard control behavior
+ callback provenance proves an eligible immediate literal Closure
+ the selected standard control contract fixes invocation count/order/arity
+ no unsupported activation/tooling requirement is present
    -> B-prime local semantic activation region

otherwise
    -> ordinary Closure RootCallTarget path
~~~

## Candidate set

### A — physical semantic Closure root is mandatory

Keep every reached semantic Closure activation as a distinct physical
RootCallTarget/FrameInstance.

~~~text
CURRENT_PLAT026=PASS
TOOLING_STACK=EXACT
PERF026_ROOT_REMOVAL=NO
PERF025_R_B_R_TOPOLOGY=RETAINED
~~~

This is the safe status quo.

### B-prime — selective Smalltalk-style parser open-coding

For an eligible immediate literal callback after exact standard behavior
selection:

1. evaluate/materialize the callback value exactly as required by current
   semantics;
2. preserve the authoritative ordinary selected behavior and guard/fallback;
3. create the same fresh semantic Closure activation state, but retain it in
   Bytecode locals rather than entering a separate callback CallTarget;
4. execute the callback body as an inline lexical/resumable region;
5. emit a custom RootTag/source region for the inline semantic callback;
6. extend the existing TagTreeNode NodeLibrary projection to expose the active
   inline activation/root instance/scope;
7. use the ordinary physical Closure root for every non-eligible, invalidated,
   dynamic, copied/custom or tooling-incompatible case.

Initial implementation should remain conservative: if nested escaping Closure
capture from the inline callback's own lexical locals cannot use current
same-generation access safely, that shape falls back rather than expanding
PLAT044 into a general virtual-frame architecture.

~~~text
PROTOS_LANGUAGE_SEMANTICS=PRESERVED
FRESH_ACTIVATION=PRESERVED
CONTEXT_INTRINSIC=PRESERVED
CAPTURE_BY_REFERENCE=PRESERVED_OR_FALLBACK
THIS_METHODHOME=PRESERVED
RETURN_HOME_NLR=PRESERVED
ERROR_DYNAMIC_CONTROL=PRESERVED
SUSPENSION_CONTINUATION=PRESERVED
SOURCE_STEPPING_SCOPE=PRESERVABLE

DISTINCT_CALLBACK_FRAMEINSTANCE=REMOVED_FOR_ELIGIBLE_PATH
DISTINCT_CALLBACK_TRUFFLE_STACK_ELEMENT=REMOVED_FOR_ELIGIBLE_PATH
PLAT026_TOOLING_DELTA=EXPLICIT
~~~

### C — tooling-sensitive dual topology

Open-code only while no root-sensitive tooling/instrumentation requires the
full stack representation; use the physical callback root once tooling is
enabled.

This preserves the richest debugger view during a normal debug session but has
three weaknesses:

- execution topology changes depending on attached tooling;
- an already-active inline region is difficult to retroactively turn into a
  separate FrameInstance;
- non-debug Truffle stack traces still differ unless more machinery is added.

It is technically plausible but not the smallest sufficient architecture.

### D — physical root + compiled-only inlining

Keep the callback RootCallTarget in interpreter/uncached execution, maximize
stable DirectCallNode/PE visibility and let Graal inline it in compiled code.

~~~text
PLAT026=PASS
TOOLING_STACK=EXACT
HOT_COMPILED_CALL_OVERHEAD=REDUCIBLE
INTERPRETER_CALLBACK_ROOT=RETAINED
DEEP_RECURSIVE_R_B_R=RETAINED
~~~

This is the strongest conservative alternative and the broad Truffle default.

Its underengineering problem is concrete: PERF025/PERF026 were opened because
interpreter/runtime physical topology and stack amplification matter, not only
compiled steady-state CPU cost. Candidate D does not answer that evidence.

### E — custom virtual debugger-frame architecture

Remove the callback RootCallTarget while building a Protos-specific logical
stack-frame layer capable of reconstructing the old callback/caller frame
cardinality for DAP and stack traces.

This would approximate HotSpot virtual frames, but above Truffle's generic
language debugger rather than inside compiler deoptimization metadata.

It is rejected as recommendation because it would add substantial bespoke
tooling/state machinery solely to hide a physical optimization:

- new logical-stack metadata/lifetime rules;
- debugger/DAP integration beyond ordinary Truffle frame iteration;
- stack-trace synthesis;
- unwind/eval/scope interactions;
- Native Image footprint and testing;
- likely pressure toward the custom debugger architecture PLAT026 rejected.

## GITHUB010 scorecard

Scores are 1-5. Confidence: H=high, M=medium, L=low. The arithmetic is advisory;
hard invariant deltas and under/overengineering gates remain qualitative.

| Criterion | A physical root | B-prime selective open-code | C tooling-sensitive dual | D compiled-only inline | E virtual debugger frames |
| --- | --- | --- | --- | --- | --- |
| Correctness / invariant preservation | **5/H** — exact current invariants | **3/H** — language semantics preservable, but explicit PLAT026 tooling delta | **4/M** — debugger mode preserves frames, mode transition uncertain | **5/H** — exact current invariants | **3/L** — conceptually exact, mechanism unproven |
| Protos alignment | **3/H** — ordinary model, avoidable physical institution remains | **5/H** — ordinary semantics with specialized representation/fallback | **3/M** — attached-tool mode changes physical model | **4/H** — simple and ordinary, but leaves known representation debt | **2/H** — new bespoke virtual-stack institution |
| Present-need proportionality | **3/H** — zero new machinery, but current carrier/root cost is paid broadly | **5/M** — bounded to proven common cases | **2/M** — two topologies for one optimization | **5/H** — minimal change | **1/H** — far beyond current need |
| Incremental growth | **3/H** — later root removal still requires architectural work | **5/M** — Boolean/while first, each later, exact fallback | **3/M** — dual-mode complexity grows with families | **5/H** — can strengthen direct-call work incrementally | **2/L** — foundational virtual-stack layer first |
| Future-option resilience | **3/M** — preserves tooling, constrains stack optimization | **4/M** — fallback keeps arbitrary Closure architecture open | **4/M** — preserves both modes at permanent complexity | **4/H** — preserves current Truffle model | **3/L** — high coupling to current debugger internals |
| Scalability | **2/H** — repeated callback roots retain recursive stack amplification | **5/M** — removes one physical callback layer in eligible recursion/loops | **4/M** — ordinary execution scales, tooling path retains root | **3/H** — compiled hot path good; interpreter/deep recursion unchanged | **3/L** — metadata and stack synthesis scale risk |
| Conceptual simplicity | **5/H** — current model | **4/M** — one guarded representation plus fallback | **2/H** — tool-dependent execution topology | **5/H** — standard Truffle call model | **1/H** — second stack model |
| Portability / implementation freedom | **5/H** — semantic roots map naturally across backends | **4/M** — semantic inline-control pattern is portable; Truffle projection is backend-specific | **2/M** — depends on dynamic instrumentation mode | **4/H** — common optimizing-VM pattern | **1/H** — tightly coupled to generic debugger internals |
| Runtime / resource cost | **2/H** — extra root/stack layer remains | **5/M** — directly removes target/root overhead where proven | **4/M** — fast normally, slower with tools | **4/H** — strong compiled cost, unchanged interpreter cost | **2/L** — extra metadata/projection machinery |
| Failure / operability | **5/H** — mature current path | **4/M** — exact fallback; tooling delta must be tested/documented | **2/M** — mode transition/debug-only failures | **5/H** — mature Truffle behavior | **2/L** — complex stack synthesis failure modes |
| Cost of deferral / reversibility / migration | **2/H** — deferral retains PERF025/PERF026 stack debt and carrier decision uncertainty | **4/M** — bounded implementation, generic fallback gives retreat path | **3/M** — later simplification requires deleting one topology | **3/H** — cheap now but likely followed by B-prime if interpreter evidence matters | **1/H** — expensive to build and remove |
| Evidence maturity / implementation risk | **5/H** — current production | **4/M** — strong Smalltalk + explicit Bytecode DSL parser-inline support; Protos implementation unproven | **2/L** — no strong production precedent found | **5/H** — dominant Truffle pattern | **1/L** — no comparable language-layer precedent found |

Score totals:

~~~text
A=43/60
B_PRIME=52/60
C=35/60
D=52/60
E=22/60
~~~

B-prime and D tie arithmetically. The recommendation is not selected by the
total.

D has an explicit **underengineering red flag**: it does not remove the
interpreter/root topology that motivates PERF025/PERF026.

B-prime has an explicit **current-invariant delta**: eligible open-coded
callbacks no longer contribute a distinct generic Truffle debugger stack frame.
That delta must be consciously approved; it cannot be hidden under the word
“optimization”.

E has an explicit **overengineering red flag**: it builds a new virtual stack
institution to avoid accepting the bounded tooling consequence of the desired
physical optimization.

## Mandatory incremental-design review

### What is the smallest solution that satisfies today's requirement?

If physical callback-root removal is a real requirement, B-prime is the smallest
solution:

~~~text
existing PLAT040 selection guards
+ current Closure semantic activation constructor/state
+ existing current-activation local mechanism
+ Bytecode DSL local lexical/resumable region
+ supported custom RootTag for parser-inlined call
+ existing TagTreeNode NodeLibrary seam
+ exact ordinary physical-call fallback
~~~

It does not add generic virtual frames, a custom DAP, a global registry, a new
Closure kind, syntax, or new language semantics.

### If B-prime is omitted today, can it be added later?

Yes, but the concrete deferral cost is known rather than speculative:

- PERF026-B/C/D remain unable to remove callback roots;
- PERF025 must make its carrier/stack decision against the retained R-B-R
  topology;
- a later B-prime adoption must still revisit PLAT026 tooling/frame assumptions
  and the callback lowering path.

Therefore D preserves an escape path but does not retire the current debt.

### What would have to be rewritten later?

The durable model need not change if B-prime is adopted later; the same standard
selection/fallback seam already exists.

Implementation work would still need:

- inline Closure activation lowering;
- callback lexical-scope locals/parameters;
- inline active-activation selection;
- TagTreeNode scope/root-instance projection;
- focused DAP/stack/tooling behavior;
- per-family Boolean/while/each lowering.

This is meaningful but bounded.

### What current complexity would we regret implementing too early?

The clearest regret would be E: a generic virtual debugger stack before a
specific debugger requirement proves it necessary.

C also adds permanent dual-topology complexity merely to preserve a frame
cardinality that Protos language semantics do not currently define normatively.

B-prime deliberately avoids both.

## GITHUB021 invariant/delta check

### PLAT040 / I072

~~~text
ORDINARY_LOOKUP_SELECTION_AUTHORITY=PRESERVED
SELECTED_STANDARD_BEHAVIOR_GUARD=PRESERVED
CUSTOM_COPIED_OVERRIDE_FALLBACK=PRESERVED
NO_SELECTOR_ONLY_PRIVILEGE=PRESERVED
PLAT040_DELTA=NONE
~~~

### PLAT043

~~~text
STANDARD_BOOLEAN_ORCHESTRATION_OWNER=PRESERVED
UNTAGGED_BOOLEAN_HELPER_REMAINS_REMOVED
CALLBACK_SEMANTIC_EXECUTION=CHANGES_PHYSICAL_REPRESENTATION_ONLY_IF_B_PRIME_APPROVED
PLAT043_DELTA=NONE_TO_BOOLEAN_SEMANTICS
~~~

### PLAT041

~~~text
OBJECT_BODY_INLINE_REGION=PRESERVED
SEMANTIC_SOURCE_ROOT_UNIVERSE=PRESERVED
PLAT041_DELTA=NONE
~~~

PLAT041 is an implementation precedent for active activation state in a local
resumable region, not authority for the Closure tooling delta.

### PLAT014

B-prime requires that callback suspension/resumption remain within the containing
semantic Bytecode root's continuation state while the inline callback activation
local remains live.

~~~text
NO_REPLAY=PRESERVED
YIELD_RESUME_VALUE_PRESERVED
TASK_CONTROL_OWNERSHIP=PRESERVED
NLR_ERROR_CANCELLATION=PRESERVED
PLAT014_DELTA=NONE_IF_IMPLEMENTATION_PROVES_EQUIVALENCE
~~~

### PLAT015

The existing NodeLibrary projection mechanism remains authoritative. B-prime must
extend it to select the inline active activation at an inline callback tag
location instead of always projecting frame argument zero.

~~~text
SCOPE_CONTENT_SEMANTICS=PRESERVED
PHYSICAL_FRAME_SOURCE=IMPLEMENTATION_DELTA
PLAT015_OBSERVABLE_SCOPE_DELTA=NONE_REQUIRED
~~~

### PLAT026 — explicit reopen required for B-prime

Preserved:

~~~text
ROOTTAG_MEANS_TRUTHFUL_SEMANTIC_EXECUTION_REGION=YES
HELPER_ROOTS_REMAIN_HIDDEN=YES
ROOTBODYTAG_REMAINS_DEFERRED=YES
STATEMENT_CALL_EXISTING_TAG_SEMANTICS=PRESERVED
NO_GLOBAL_ROOT_REGISTRY=YES
MULTI_CONTEXT_MUTABLE_ROOT_AUTHORITY=NO
NATIVE_IMAGE_RUNTIME_CLASSIFICATION_REGISTRY=NO
~~~

Explicit delta:

~~~text
OLD:
  every current semantic Closure activation
  -> distinct semantic Bytecode root
  -> automatic RootTag
  -> distinct FrameInstance/RootCallTarget

B_PRIME:
  ordinary/dynamic Closure invocation
  -> unchanged physical semantic root + automatic RootTag

  eligible standard-control immediate literal callback
  -> fresh semantic Closure activation preserved
  -> callback body is parser-inlined semantic region
  -> custom RootTag at inline region
  -> no distinct callback FrameInstance/RootCallTarget
~~~

Tooling consequence:

~~~text
BREAKPOINT_SOURCE_STEPPING=PRESERVABLE
CURRENT_CALLBACK_SCOPE=PRESERVABLE
CURRENT_CALLBACK_ROOT_INSTANCE_NAME=PRESERVABLE

DISTINCT_CALLBACK_DEBUGSTACKFRAME=NO
DISTINCT_CALLBACK_TRUFFLE_STACKTRACE_ELEMENT=NO
ENCLOSING_CALLER_PLUS_CALLBACK_AS_TWO_PHYSICAL_FRAMES=NO
~~~

Therefore before approval:

~~~text
PLAT026_INVARIANT_DELTA=EXPLICIT
DECISION_INVARIANT_CONSISTENCY=REQUIRES_OWNER_REOPEN_AND_APPROVAL
~~~

If the owner approves exact Candidate B-prime with this delta, GITHUB021 can
become PASS for ratification. Approval of “open-code callbacks” without this
tooling consequence is insufficient.

## Recommendation

~~~text
PLAT044_STATUS=PROPOSED

RECOMMENDED_CANDIDATE=B_PRIME
RECOMMENDATION_STATUS=APPROVED_AND_RATIFIED

B_PRIME=
  GUARDED_STANDARD_CONTROL_SELECTION
  + ELIGIBLE_IMMEDIATE_LITERAL_CLOSURE
  + FRESH_SEMANTIC_ACTIVATION_IN_BYTECODE_LOCALS
  + PARSER_INLINED_LEXICAL_RESUMABLE_REGION
  + CUSTOM_INLINE_ROOTTAG
  + TAGTREENODE_SCOPE_ROOT_INSTANCE_PROJECTION
  + EXACT_ORDINARY_PHYSICAL_CLOSURE_FALLBACK

OBSERVABLE_PROTOS_LANGUAGE_SEMANTIC_CHANGE_REQUIRED=NO
OBSERVABLE_TOOLING_STACK_DELTA=YES
PLAT026_REOPEN_REQUIRED=YES
CUSTOM_DAP_REQUIRED=NO
VIRTUAL_DEBUGGER_STACK_REQUIRED=NO
~~~

### Why B-prime over D

D is the right architecture for arbitrary ordinary Closure calls and remains the
fallback architecture.

B-prime is recommended only because the motivating family is narrower and has
stronger precedent:

- Smalltalk standard control is deliberately expressed as ordinary messages with
  blocks;
- mature Smalltalk compilers remove those sends/block activations in eligible
  forms;
- Truffle Bytecode DSL explicitly supports parser-inlined calls carrying custom
  RootTag regions;
- Protos already has exact standard-selection guards and local active-activation
  machinery;
- the known R-B-R interpreter/stack topology is a current problem, not a
  hypothetical future concern.

Treating the source-level message model as requiring a physical Closure root in
these exact standard-control cases would conflate language semantics with one
runtime representation.

### Strongest argument against B-prime

The current PLAT026 debugger model intentionally made Closure activations
semantic roots. B-prime narrows that guarantee.

When stopped inside an open-coded callback, the debugger can still project the
callback source, current semantic activation, scope and root-instance identity,
but the generic Truffle stack is no longer the same two-frame caller/callback
shape.

If that frame cardinality is more important than eliminating the physical
callback root, Candidate D should be selected instead.

This is the owner decision PLAT044 exists to make.

## Downstream consequence if B-prime is approved

~~~text
PERF026_B=#767 -> READY_FOR_BOUNDED_IMPLEMENTATION
PERF026_C=#768 -> READY_FOR_BOUNDED_IMPLEMENTATION_AFTER_COMMON_MECHANISM
PERF026_D=#769 -> READY_FOR_LATER_IMPLEMENTATION

PERF025_CARRIER_DECISION=
  REEVALUATE_AFTER_BOOLEAN/CONTROL_ROOT_TOPOLOGY_EVIDENCE

BUG008=#681 CLOSED_DO_NOT_REOPEN
~~~

Implementation should begin with the smallest Boolean/common mechanism rather
than a generic Closure-inlining framework. While/each reuse the proven mechanism
only after Boolean establishes the architecture.

If implementation discovers that inline callback lexical capture, DAP projection
or continuation behavior requires another durable architecture choice not stated
above, it stops at a new PLAT gate.

## Approval and ratification provenance

The project owner approved exact Candidate B′ in the active interaction on
2026-10-02:

~~~text
ok apruebo b'
~~~

At the point of approval, B′ was already defined by this packet as including the
explicit PLAT026 tooling/frame delta:

~~~text
DISTINCT_CALLBACK_DEBUGSTACKFRAME=NO
DISTINCT_CALLBACK_TRUFFLE_STACKTRACE_ELEMENT=NO
~~~

The approval therefore selects the complete B′ candidate rather than a reduced
variant that preserves old frame cardinality.

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
APPROVED_CANDIDATE=B_PRIME
PLAT026_INVARIANT_DELTA=EXPLICIT_AND_APPROVED
DECISION_INVARIANT_CONSISTENCY=PASS
IMPLEMENTATION_AUTHORIZED=YES_AFTER_DURABLE_PUBLICATION
DEPENDENT_PERF_RELEASED=PERF026_B_FIRST
~~~

## Materially inspected sources

### Protos / project authority

- `AGENTS.md`
- `AGENTS.work/DESIGN.md`
- `AGENTS.work/COORDINATION.md`
- `AGENTS.work/REFERENCE.md`
- `spec/semantics/CALLABLES.md`
- `spec/semantics/EXECUTION_AND_CONTROL.md`
- `CanonicalToBytecodeLowerer.java`
- `ProtosBytecodeRootNode.java`
- `ProtosSemanticBytecodeRootNode.java`
- `ProtosBytecodeTagTreeNodeExports.java`
- PLAT014, PLAT015, PLAT026, PLAT040, PLAT041 and PLAT043 durable decisions
- PERF025/PERF026 live coordination evidence

### External runtime/source evidence

- GNU Smalltalk Performance manual
- Squeak compiler macro-selector/performance documentation
- current Truffle Bytecode DSL, RootNode, CallTarget, debugger/frame and
  NodeLibrary APIs
- current Oracle Graal debugger `SuspendedEvent` / `DebugStackFrame`
- current GraalPy Bytecode root/call surfaces
- current GraalJS function-call and control-flow nodes
- current TruffleRuby Proc/block call surfaces
- current TruffleSqueak block dispatch
- current Apple Pkl function representation/application
- current Espresso method root
- current Sulong LLVM function root
- current GraalWasm function/direct-call surfaces
- OpenJDK HotSpot deoptimization virtual-frame implementation as a
  non-Truffle contrast

## Cross references

- PLAT044: `guillermomolina/protos#766`
- PERF026: `guillermomolina/protos#765`
- PERF026-B: `guillermomolina/protos#767`
- PERF026-C: `guillermomolina/protos#768`
- PERF026-D: `guillermomolina/protos#769`
- PERF025: `guillermomolina/protos#758`
- PLAT026: `guillermomolina/protos#375`
- PLAT040: `guillermomolina/protos#718`
- PLAT041: `guillermomolina/protos#759`
- PLAT043: `guillermomolina/protos#763`
