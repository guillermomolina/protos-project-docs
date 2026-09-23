# AUD016 — Truffle Bytecode DSL capability-adoption audit

Status: COMPLETE — IMPLEMENTATION BACKLOG IDENTIFIED

This durable, non-normative record retains the completed read-only investigation
for guillermomolina/protos#701. It does not change Protos semantics, authorize a
product patch, create implementation children, or ratify a new platform decision.

## Evidence identity

~~~text
AUD016_PROTOS_REVISION=
  3e8e6b565c95eb5098c2168d241536ba13ad19e9

AUD016_PROJECT_DOCS_BASE_REVISION=
  187d17f28c316b23465b4d0c35680fe7b0ee434b

AUD016_GRAALVM_VERSION=
  25.3.4.1

AUD016_TRUFFLE_VERSION=
  25.3.4.1
~~~

The product dependency is pinned through pom.xml with graalvm.version=25.3.4.1.

The connected repository interface exposes remote repository state but not the
maintainer's local checkout. Therefore this audit is explicitly revision-bound
to the remote product main SHA above and does not invent local git-status,
HEAD, or origin/main observations.

## Governing authority inspected

Project instructions:

~~~text
AGENTS.md
AGENTS.work/AUDIT.md
AGENTS.work/DESIGN.md
AGENTS.work/REFERENCE.md
AGENTS.work/COORDINATION.md
src/AGENTS.md
~~~

Normative owners materially read:

~~~text
spec/PROTOS_LANGUAGE_SPEC.md
spec/semantics/OBJECT_MODEL.md
spec/semantics/EXECUTION_AND_CONTROL.md
spec/semantics/CALLABLES.md
spec/semantics/VALUES_AND_COLLECTIONS.md
spec/concurrency/FUTURES_AND_TASKS.md
~~~

Relevant ratified platform lineage reconciled:

~~~text
PLAT014
PLAT015
PLAT019
PLAT021
PLAT026
PLAT034
PLAT035
~~~

The key settled platform constraint is PLAT035: the Truffle Bytecode DSL is the
only executable guest backend. AUD016 therefore treats DSL-native representation
as completion of an already-selected backend when no new durable architecture is
being selected.

## Executive result

Protos uses the Bytecode DSL substantially, but underuses it in a central part of
guest execution: lexical state.

Current architecture contains both models:

~~~text
backend temporaries
  -> BytecodeLocal
  -> generated frame-local representation

guest lexical bindings
  -> CanonicalLookup(name)
  -> Lookup(ProtosActivation, String)
  -> ProtosActivation.lookup(name)
  -> current execution-context object
  -> captured execution-context traversal
  -> receiver/member fallback
~~~

The second path is semantically correct but over-generic for bindings whose
identity and lexical location are already known before execution.

The correct target is not "replace every lookup with a local". Protos supports
genuinely dynamic cases, including a Closure observing later creation of a slot
inside a captured execution context. The normal architecture should therefore
be hybrid:

~~~text
statically admitted lexical binding
  -> indexed DSL local/materialized-local access

genuinely dynamic or not-definitely-established name
  -> exact existing dynamic lexical/member fallback
~~~

This conclusion is architectural. It does not depend on proving that the change
explains the PERF010-A performance gap.

## Semantic constraints on the migration

### Execution context

Protos has no independent semantic category "local variable". Parameters and
local bindings are slots of the current execution context, and context denotes
that ordinary Protos object.

Implementation representation may change, but observable execution-context
semantics may not.

### Lexical lookup

The required search order remains:

~~~text
current execution-context own local slots
  -> captured lexical contexts in order
  -> receiver/member lookup
  -> receiver delegation / normal fallback
  -> failure
~~~

### Creation

Bare creation creates only in the current execution context and may shadow
outer lexical, receiver, or prelude state. The right-hand side is evaluated
before the new slot becomes visible.

### Assignment

Bare assignment selects the writable destination before evaluating the RHS:

~~~text
first existing lexical local destination
  -> receiver own slot
  -> failure
~~~

Delegated receiver ancestors are not writable destinations and RHS side effects
must not cause destination re-resolution.

### Closure capture

Closures capture execution contexts by reference, not by copied value. Mutations
after capture remain visible. A captured context can also acquire a slot later
and that later slot can become visible to the Closure. This is the principal
reason unresolved names cannot be prematurely compiled to one fixed local.

### Parameter binding

Each invocation establishes a fresh execution context. Parameter slots become
established sequentially and defaults can observe only bindings established so
far. An indexed representation therefore needs explicit presence/uninitialized
semantics rather than treating every possible slot as semantically present from
invocation entry.

### Integer

Protos Integer remains mathematically unbounded. A long fast carrier with
transparent overflow to BigInteger is semantically legal in principle, but its
performance value and implementation complexity remain empirical questions.

## Current Bytecode DSL usage map

The source/lowering path inspected is:

~~~text
source
 -> parser
 -> canonical semantic AST
 -> CanonicalToBytecodeLowerer
 -> generated Bytecode DSL builder
 -> ProtosSemanticBytecodeRootNode
 -> ProtosBytecodeRootNode
 -> generated operations/specializations
 -> ProtosActivation/runtime state
 -> Closure execution plans
 -> Task/ContinuationResult integration
 -> instrumentation/debugger scope projection
~~~

Material source owners include:

~~~text
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeClosureExecutionPlan.java
src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java
src/main/java/com/guillermomolina/protos/execution/ProtosDebuggerScope.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeTagTreeNodeExports.java
src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java
src/main/java/com/guillermomolina/protos/runtime/ProtosValueLookup.java
~~~

Protos already uses correctly:

~~~text
@GenerateBytecode
BytecodeRootNode / BytecodeRootNodes
BytecodeLocal for backend temporaries
@Operation / @Specialization
VirtualFrame / FrameDescriptor
DirectCallNode / IndirectCallNode
RootCallTarget
ContinuationResult / ContinuationRootNode
enableYield=true
tag instrumentation
source sections / BytecodeLocation
semantic/helper RootTag split
automatic quickening
instruction rewriting
threaded switch
~~~

## Static information currently lost before the DSL

Canonical lexical operations presently reduce a source name to a dynamic String
operand. For a meaningful subset of accesses, the frontend can already know:

~~~text
binding declaration identity
parameter/creation identity
current-vs-outer lexical owner
lexical depth
source name
definite establishment at a program point
~~~

Yet CanonicalLookup("x") reaches the DSL as activation + String and the runtime
rediscovers location through ProtosActivation.lookup.

Therefore:

~~~text
STATIC_INFORMATION_LOST_BEFORE_DSL=YES
~~~

## Lexical classification

### Current lexical binding

A definitely-established binding in the current lexical owner can use a
BytecodeLocal.

~~~text
CAN_CURRENT_LOCAL_USE_BYTECODE_LOCAL=YES
~~~

### Captured lexical binding

A definitely-known outer binding can use materialized local access with fixed
binding identity/depth.

~~~text
CAN_CAPTURED_LOCAL_USE_MATERIALIZED_LOCAL_ACCESS=YES_CONDITIONAL
~~~

The condition is exact preservation of capture-by-reference semantics and
binding presence.

### Dynamic resolution

The generic path must remain for:

~~~text
name not definitely established at the access point
later dynamically-created captured binding
receiver/member fallback
delegated lookup
prelude/global fallback
other reflective/dynamic name resolution
~~~

~~~text
CAN_NAME_LOOKUP_BE_REMOVED_FROM_ADMITTED_HOT_PATH=YES_CONDITIONAL
CAN_GENERIC_DYNAMIC_FALLBACK_BE_PRESERVED=YES
~~~

## Activation decomposition

Current ProtosActivation combines several responsibilities:

~~~text
semantic execution-context carrier
captured lexical-context carrier
receiver / prelude / arguments
return-home / method-home metadata
Actor/module/execution-domain state
Task state
dynamic-control state
deferred C-prime suspension/release state
lexical lookup implementation
~~~

The lexical-storage responsibility should migrate toward DSL frame state.
Task, control, module, return-home, method-home, receiver and related semantic
authority must not disappear merely because local storage changes.

The semantic existence of context is distinct from mandatory eager host-object
allocation, but lazy materialization is not automatically authorized.

## Debugger/tooling

The current custom debugger projection is correct for the current runtime:
Bytecode locals are implementation state and ProtosDebuggerScope projects guest
scope through ProtosActivation.

After guest bindings become DSL locals, raw interpreter locals still must not be
exposed directly because they include implementation temporaries.

The correct follow-on remains a semantic adapter that merges:

~~~text
guest-marked DSL locals
materialized outer guest locals
dynamic execution-context overflow slots
receiver/member fallback
~~~

Therefore the custom semantic scope adapter remains, while its backing source of
lexical state changes.

## Calls and continuations

The current call-site architecture is aligned with Truffle:

~~~text
guard/cached stable target
  -> DirectCallNode

generic/megamorphic fallback
  -> IndirectCallNode
~~~

Continuation resumption similarly caches stable ContinuationRootNode identity and
uses the DSL-native ContinuationResult frame/calling convention.

PLAT014's C-prime composition is Protos-level stackful/task semantics layered on
the DSL-native single-root continuation substrate, not a duplicate interpreter.

These mechanisms should remain.

## Quickening

Automatic quickening and instruction rewriting are already enabled by the DSL's
normal configuration.

The principal lexical problem occurs before quickening: static information is
turned into dynamic Java inputs. @ForceQuickening cannot recover information
discarded by lowering.

Result:

~~~text
AUTOMATIC_QUICKENING=KEEP_CURRENT
FORCE_QUICKENING_GENERAL_POLICY=DO_NOT_ADOPT
~~~

## Bytecode handler tail calls

The pinned DSL line provides enableTailCallHandlers, disabled by default. This is
host interpreter compilation machinery, not Protos guest tail-call semantics.

It can reduce large dispatch-loop compilation units/register pressure through
handler outlining, parameter expansion, specialized calling conventions and
tail-call threading. Its value for Protos depends on generated-code/compiler
behavior.

Result:

~~~text
BYTECODE_HANDLER_TAIL_CALLS=EXPERIMENT_FIRST
~~~

It is independent of lexical migration.

## Boxing elimination

boxingEliminationTypes is applicable in principle to primitive carriers.

Potential fit:

~~~text
Float/double              -> direct semantic fit
Boolean/boolean           -> conditional on guest-object boundaries
Integer/long              -> conditional with transparent BigInteger overflow
indices/counts/temporaries-> strong implementation fit
~~~

No architectural simplification requires enabling it, and an Integer fast path
introduces overflow, specialization and boxing-boundary complexity.

Result:

~~~text
BOXING_ELIMINATION=EXPERIMENT_FIRST
~~~

## Uncached interpreter

enableUncachedInterpreter targets startup/cold execution by avoiding cached node
allocation until a threshold is crossed. It is potentially relevant to CLI,
short scripts, tests and tool invocation, not primarily steady-state throughput.

The large Protos operation surface means uncached compatibility must be checked
operation by operation.

Result:

~~~text
UNCACHED_INTERPRETER=EXPERIMENT_FIRST
~~~

PERF010 steady-state evidence is not authority for this classification.

## Other materially relevant facilities

~~~text
threaded switch
  -> KEEP_CURRENT; default-enabled and appropriate

instruction rewriting
  -> KEEP_CURRENT; default-enabled and appropriate

serialization
  -> DO_NOT_ADOPT without an actual bytecode-cache/startup requirement

specialization introspection
  -> DO_NOT_ADOPT as a production facility without a consumer

storeBytecodeIndexInFrame
  -> KEEP_CURRENT false in production; useful as bounded migration diagnostics

captureFramesForTrace
  -> KEEP_CURRENT false until a concrete tooling requirement exists
~~~

## Comparative evidence

Exact mature-runtime revisions inspected:

~~~text
TruffleSOM=
  SOM-st/TruffleSOM@73f6d2e654022565ec7c7e8ba95ae18340a862ce

TruffleSqueak=
  hpi-swa/trufflesqueak@818519b2b6a6556bc524e9e0d08f7b51969cb61a

TruffleRuby=
  truffleruby/truffleruby@0e6fa6a950dce7154d54f3c9c63056c4eb925ffd
~~~

TruffleSOM directly represents local variables through fixed slotIndex and
non-local variables through contextLevel + slotIndex + MaterializedFrame.

TruffleRuby directly represents local reads through frameSlot and captured
declaration-frame reads through frameDepth + frameSlot.

TruffleSqueak separates ordinary frame execution from materialized/context
objects where semantic context observation requires them.

The transferable pattern is not their language semantics. It is:

~~~text
semantic lexical fact known by the frontend
  -> compiler-visible slot/depth identity

genuinely dynamic language behavior
  -> specialized dynamic lookup/invalidation/fallback
~~~

No mature-runtime comparison justifies deleting Protos's required dynamic
fallback.

## Graal/Truffle source provenance note

Protos is pinned to artifact version 25.3.4.1.

AUD016 inspected the Bytecode DSL API/documentation on the public 25.3.4.1-era
Graal source line, including oracle/graal commit:

~~~text
5f900fe691e58fd2cf7b991d9ae35d45a66bfbf2
~~~

Material upstream sources included GenerateBytecode.java, BytecodeLocal.java,
LocalAccessor.java, MaterializedLocalAccessor.java, the Bytecode DSL user guide,
the Continuations tutorial, and OneCompilationPerBytecodeHandler.md.

A previously retained PERF source comparison cited:

~~~text
oracle/graal@b3ebbbefb3289f862498beb4e84f5912aa2c0557
~~~

AUD016 found that revision on a later 25.5 development line. It is therefore not
used here as authority for Protos's pinned 25.3.4.1 capability inventory.

The public repository metadata available through the connected interface does
not establish a unique final-release source SHA for the Maven artifact itself,
so AUD016 records the exact artifact version separately from the exact public
source revision inspected rather than inventing equivalence.

## Retrospective audit of current mechanisms

~~~text
String lookup for definitely-known current lexical binding
  -> REMOVE_PERMANENTLY from admitted static path

String/captured-context traversal for definitely-known captured binding
  -> REMOVE_PERMANENTLY from admitted static path

generic unresolved lexical/member lookup
  -> KEEP

semantic context abstraction
  -> KEEP

heap context object as sole storage for every static lexical value
  -> REMOVE_NOW_RECONSIDER_LATER

capturedLexicalContexts as sole transport for all captures
  -> REMOVE_NOW_RECONSIDER_LATER

ProtosActivation as semantic/task/control carrier
  -> KEEP

ProtosActivation as universal lexical-access mechanism
  -> REMOVE_NOW_RECONSIDER_LATER

custom ProtosDebuggerScope adapter
  -> KEEP

DirectCallNode + indirect fallback
  -> KEEP

C-prime continuation composition
  -> KEEP

PLAT026/PLAT034 tooling topology
  -> KEEP
~~~

## Migration coupling

~~~text
LEXICAL_FRAME_MIGRATION_COUPLING=
  STRONGLY_COUPLED
~~~

The shared architecture consists of:

~~~text
guest lexical BytecodeLocal adoption
captured/materialized locals
frame-backed context projection
debugger scope adaptation
lexical portion of activation decomposition
~~~

These should not land as one mega-patch, but they must share one representation
contract and exact fallback seam.

Lazy context-object allocation is only partially coupled and should be decided
after frame-backed lexical storage exists.

## Minimum safe implementation sequence

1. Canonical binding identity and definite-presence classification.
   No runtime representation change yet.

2. Frame-backed semantic context-storage seam while retaining eager semantic
   context identity and an exact dynamic overflow store.

3. Current lexical reads/writes lowered to BytecodeLocal for definitely admitted
   bindings.

4. Sequential parameter binding migrated with exact presence/default semantics.

5. Captured bindings lowered through materialized local access with fixed
   lexical depth/binding identity.

6. ProtosDebuggerScope updated to merge guest DSL locals, materialized captured
   locals, dynamic overflow and receiver/member fallback.

7. ProtosActivation lexical responsibility decomposed after both static paths are
   operational.

8. Only then consider lazy semantic context-object allocation through a separate
   platform decision.

## Capability backlog

### A. ADOPT — foundational / architecture-completing

~~~text
A1 canonical lexical binding identity / definite-presence classification
   ONE_IMPLEMENTATION_WORK_ITEM

A2 frame-backed current guest lexical bindings / BytecodeLocal
   PART_OF_A_SHARED_MIGRATION

A3 captured guest lexical bindings / materialized locals
   SEVERAL_DEPENDENT_IMPLEMENTATION_CHILDREN

A4 lexical ProtosActivation decomposition
   SEVERAL_DEPENDENT_IMPLEMENTATION_CHILDREN
~~~

### B. ADOPT — useful follow-on

~~~text
B1 DSL local metadata integrated into semantic debugger scope
   PART_OF_A_SHARED_MIGRATION
~~~

### C. EXPERIMENT_FIRST

~~~text
C1 bytecode-handler tail-call compilation
C2 boxing elimination
C3 uncached interpreter/startup mode
~~~

### D. KEEP_CURRENT

~~~text
automatic quickening
instruction rewriting
threaded switch
ContinuationResult/C-prime continuation architecture
DirectCallNode/IndirectCallNode cache architecture
RootTag/tag/source architecture
production storeBytecodeIndexInFrame=false
captureFramesForTrace=false absent a concrete need
~~~

### E. DO_NOT_ADOPT

~~~text
generic @ForceQuickening policy
Bytecode serialization without a bytecode-cache requirement
production specialization introspection without a consumer
~~~

### F. REQUIRES_PLAT_DECISION

~~~text
lazy semantic execution-context object materialization
~~~

No implementation child or PLAT identifier is allocated by this record.

## Platform decision packet — lazy context materialization

~~~text
RECOMMENDATION_ONLY=YES
OWNER_APPROVAL_REQUIRED=YES
~~~

Exact problem:

Should a Protos execution context use a frame-first implementation whose ordinary
Protos context object is materialized lazily only when semantically observed?

Authoritative constraints:

- the logical execution context always exists;
- context has ordinary object identity when observed;
- context can escape;
- Closures capture lexical contexts by reference;
- later dynamic slot creation remains observable;
- debugger/tooling can observe lexical scope;
- suspension must preserve state;
- sequential parameter/default establishment remains observable.

Already-approved invariants:

~~~text
PLAT014 continuation ownership/composition
PLAT015 semantic debugger scope
PLAT019 suspension bridge
PLAT021 dynamic-control authority
PLAT026 RootTag boundary
PLAT034 generic tooling envelope
PLAT035 single Bytecode DSL backend
~~~

Alternatives:

~~~text
A keep eager context object permanently

B frame-first state with lazy semantic-object materialization

C allocate stable semantic facade identity eagerly but keep lexical storage
  frame-backed and materialize/overflow only as required
~~~

Recommendation:

Do not decide B as part of the lexical migration. First implement frame-backed
lexical storage while retaining eager stable semantic context identity. Revisit
physical context-object laziness afterward, when remaining allocation cost and
coherence requirements are isolated.

Failure modes include duplicate context identities, stale frame/object state,
incorrect later-created binding visibility, capture-by-reference breakage,
debugger-dependent semantics, and suspension/materialization incoherence.

## Relationship to PERF010-A and PERF011

AUD016 consumes their evidence but does not merge their ownership.

PERF010-A remains the causal attribution owner.

PERF011 remains the runtime-representation/polymorphism-fidelity performance
owner.

AUD016 establishes an independent architecture result:

~~~text
DSL_NATIVE_LEXICAL_REPRESENTATION_BELONGS_IN_PROTOS=YES

ARCHITECTURAL_ADOPTION_DEPENDS_ON_PERF010_DOMINANCE=NO

PERFORMANCE_MEASUREMENT_REQUIRED_BEFORE_FIRST_ADOPTION_SLICE=NO
~~~

This does not assert that lexical migration explains the 400x/4000x benchmark
ratio, and it does not close or supersede either PERF issue.

The retained PERF010-A execution-context backing-store experiment closed only
the "faster String-keyed local container" sub-hypothesis. It does not falsify a
representation change from String-keyed heap discovery to statically admitted
indexed frame state with dynamic fallback.

## Required negative findings

~~~text
FEATURES_ALREADY_USED_CORRECTLY=
  Bytecode backend
  yield/ContinuationResult
  continuation-root call caching
  automatic quickening
  instruction rewriting
  threaded switch
  tag/source machinery
  semantic/helper RootTag topology

FEATURES_NOT_APPLICABLE=
  language-level interpretation of enableTailCallHandlers
  universal fixed-local replacement for receiver/delegated lookup

FEATURES_THAT_LOOK_ATTRACTIVE_BUT_SHOULD_NOT_BE_ADOPTED=
  broad @ForceQuickening
  serialization just because it exists
  raw Bytecode-local debugger exposure
  every CanonicalLookup -> fixed local

FEATURES_REQUIRING_MEASUREMENT_BEFORE_ADOPTION=
  handler tail calls
  boxing elimination
  uncached interpreter

FEATURES_BLOCKED_BY_SEMANTICS=
  unconditional static binding of unresolved names
  eager semantic existence of not-yet-created slots
  removal of dynamic lexical/member fallback

FEATURES_BLOCKED_BY_PLATFORM_DECISION=
  lazy semantic context-object materialization
~~~

## Machine-readable result

~~~text
AUD016_RESULT=
  COMPLETE

AUD016_PROTOS_REVISION=
  3e8e6b565c95eb5098c2168d241536ba13ad19e9

AUD016_TRUFFLE_VERSION=
  25.3.4.1

BYTECODE_DSL_SELECTED_BACKEND_CONFIRMED=
  YES

BYTECODE_DSL_USAGE_OVERALL=
  SUBSTANTIALLY_UNDERUTILIZED

GUEST_LEXICAL_BINDINGS_AS_DSL_LOCALS=
  ADOPT

CAPTURED_LEXICALS_AS_MATERIALIZED_DSL_LOCALS=
  ADOPT

LAZY_EXECUTION_CONTEXT_MATERIALIZATION=
  REQUIRES_PLAT_DECISION

ACTIVATION_STATE_DECOMPOSITION=
  ADOPT

DEBUGGER_SCOPE_FRAME_INTEGRATION=
  ADOPT

BYTECODE_HANDLER_TAIL_CALLS=
  EXPERIMENT_FIRST

BOXING_ELIMINATION=
  EXPERIMENT_FIRST

UNCACHED_INTERPRETER=
  EXPERIMENT_FIRST

QUICKENING_CHANGES=
  KEEP_CURRENT

CONTINUATION_MODEL_CHANGES=
  KEEP_CURRENT

INSTRUMENTATION_MODEL_CHANGES=
  KEEP_CURRENT

ADOPT_COUNT=
  4

EXPERIMENT_FIRST_COUNT=
  3

KEEP_CURRENT_COUNT=
  6

DO_NOT_ADOPT_COUNT=
  3

REQUIRES_PLAT_DECISION_COUNT=
  1

DUPLICATED_RUNTIME_MACHINERY_FOUND=
  YES

STATIC_INFORMATION_LOST_BEFORE_DSL=
  YES

LEXICAL_FRAME_MIGRATION_COUPLING=
  STRONGLY_COUPLED

ARCHITECTURAL_ADOPTION_DEPENDS_ON_PERF010_DOMINANCE=
  NO

PERFORMANCE_MEASUREMENT_REQUIRED_BEFORE_FIRST_ADOPTION_SLICE=
  NO

NEW_PLATFORM_DECISION_REQUIRED=
  YES

AUD016_IMPLEMENTATION_READY=
  PARTIAL

NEXT_ACTION=
  create bounded implementation coordination beginning with canonical lexical
  binding identity / definite-presence classification, then frame-backed current
  lexical storage and BytecodeLocal lowering with exact dynamic fallback;
  separately route lazy context-object materialization through owner approval
~~~

## Closure interpretation

AUD016's exhaustive discovery question is complete.

The first normal architecture work can begin without another broad DSL audit and
without a prerequisite PERF010 timing result. The lexical migration must be
implemented as bounded dependent slices rather than a mega-patch.

A new platform decision is required only before selecting lazy physical
materialization of the semantic execution-context object; it is not required for
the first binding-analysis / frame-backed lexical slices.
