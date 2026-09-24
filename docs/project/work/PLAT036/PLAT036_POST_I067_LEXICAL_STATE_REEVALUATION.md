# PLAT036 — post-I067 lexical-state architecture re-evaluation

Status: **INVESTIGATION COMPLETE / RECOMMENDATION PENDING PROJECT-OWNER APPROVAL**

This is durable, non-normative investigation evidence for
`guillermomolina/protos#702`. It is **not** a ratified platform decision and
does not authorize product implementation. Observable Protos semantics remain
owned by the normative specification in `guillermomolina/protos`.

## Exact baselines

~~~text
PLAT036_ISSUE=guillermomolina/protos#702
PRODUCT_REVISION=75231601e458930684d4b619dd5f4722377aa65d
I067_PROJECT_RECORD_REVISION=e17d2554d574d85d009133b183a25844ba769700

D179_SELECTED_CANDIDATE=C3_MONOTONIC_CONTEXT_MEMBERSHIP
D179_PROJECT_RECORD_REVISION=9a7648181b70733fc0bb97940cd1e25482b38fd6
C0_STATUS=DEFER_RECONSIDER_LATER

GRAALVM_TRUFFLE_VERSION=25.3.4.1
~~~

At investigation time, `origin/main` of `guillermomolina/protos` still
resolved exactly to the product revision above. No post-I067 product delta had
to be reconciled.

## Decision question

PLAT036 must select the durable implementation architecture by which Protos
lexical execution-context bindings are represented on the sole Truffle Bytecode
DSL executable backend while preserving first-class execution-context semantics.

The design must preserve simultaneously:

- statically knowable lexical binding identity where source and lexical structure
  make that identity knowable;
- dynamic semantic presence where a source-known binding is not yet present;
- late `ABSENT -> PRESENT` creation;
- nearer-binding retargeting after late creation;
- capture by reference;
- stable first-class `context` object identity and escape;
- local-slot reflection and debugger projection;
- `PRESENT(null)` distinct from ABSENT;
- close/freeze semantics;
- assignment destination selection before RHS evaluation; and
- exact dynamic name/member fallback where static proof is unavailable.

C3 removes `PRESENT -> ABSENT` only. It does not make execution contexts
conventional immutable lexical frames.

## Current post-I067 architecture

The source-to-runtime lexical path remains:

~~~text
source
  -> parser / Surface AST
  -> Canonicalizer
  -> CanonicalLookup(String name)
  -> CanonicalToBytecodeLowerer
  -> Lookup(ProtosActivation, String)
  -> ProtosActivation.lookup(name)
      -> current context.readLocalSlot(name)
      -> capturedLexicalContexts[i].readLocalSlot(name)
      -> receiver/member fallback
~~~

Lexical assignment similarly resolves a writable context by name before RHS
evaluation and then stores through that selected object.

The current Bytecode DSL locals are primarily backend-private temporaries. Guest
lexical bindings still use execution-context objects plus String lookup.

Therefore:

~~~text
STATIC_INFORMATION_LOST_BEFORE_DSL=YES
~~~

More precisely, the frontend retains the semantic name, but it does not carry a
canonical binding identity, fixed lexical owner, layout index, lexical depth, or
presence classification into Bytecode lowering.

## What C3 changes

Ratified and implemented C3 establishes:

~~~text
ABSENT  -> PRESENT = ALLOWED_WHILE_OPEN
PRESENT -> PRESENT = ALLOWED_WHILE_WRITABLE
PRESENT -> ABSENT  = REJECTED
~~~

### Runtime machinery eliminated by C3

C3 makes the following unnecessary for genuine execution contexts:

- deletion state transitions for established lexical slots;
- invalidation caused by an established binding disappearing;
- re-resolution to an outer/receiver binding because a context-local binding was
  removed;
- captured-binding disappearance propagation;
- debugger/reflection removal synchronization for context locals;
- lexical tombstone/deleted-binding state.

Once an execution-context binding becomes PRESENT, membership is permanently
stable for that context lifetime.

### Runtime machinery still required after C3

C3 does **not** eliminate:

- explicit semantic presence;
- source-known but not-yet-present bindings;
- late `ABSENT -> PRESENT` creation;
- invalidation/guarding when a newly created nearer binding retargets lookup;
- dynamic names and overflow;
- receiver/member fallback;
- sequential/default parameter visibility;
- module partial initialization;
- first-class context identity and escape;
- capture by reference;
- debugger/reflection projection;
- close/freeze state;
- `PRESENT(null)` handling.

Result:

~~~text
C3_ARCHITECTURAL_SIMPLIFICATION=MATERIAL
C3_DOES_NOT_COLLAPSE_HYBRID_ARCHITECTURE=YES
~~~

## Static binding identity and presence

A binding may receive immutable static identity whenever its semantic name and
owning lexical scope are statically known. Static identity does not imply that
the binding is semantically PRESENT from activation entry.

The required distinction is:

~~~text
PHYSICAL_STATIC_BINDING_IDENTITY != SEMANTIC_BINDING_PRESENCE
~~~

The minimum conceptual state is:

~~~text
LexicalLayout
  binding[index] = immutable metadata(name, binding identity, owner)

ExecutionContextState
  values[index]
  present[index]
  dynamicOverflow
~~~

`PRESENT(null)` is represented by a true presence bit plus the semantic null
value. ABSENT is represented independently.

Runtime structural creation of a name already represented in the static layout
activates that indexed binding; it must not create a second authoritative
overflow binding of the same name.

## Sequential/default parameters

Parameter names may have layout indexes before invocation starts without being
semantically visible.

Conceptually:

~~~text
activation entry:
  physical indexes exist
  semantic presence = false where not yet bound

bind parameter 0:
  write value[0]
  present[0] = true

evaluate default for parameter 1:
  parameter 0 is visible
  parameter 1 is still absent
  later parameters are still absent

finish parameter 1:
  write value[1]
  present[1] = true
~~~

This preserves the current sequential/default visibility contract without using
String lookup as the physical representation.

## Late creation, shadowing, retargeting and invalidation

A statically known outer binding cannot always be loaded unconditionally because
a nearer execution context may legally acquire the same name later:

~~~text
outer.x  = PRESENT
middle.x = ABSENT

read x -> outer.x

later:
middle.x becomes PRESENT

subsequent read x -> middle.x
~~~

Correct indexed execution therefore needs one of:

- explicit nearer-presence checks; or
- a compiled absent/topology assumption that invalidates on the first relevant
  `ABSENT -> PRESENT` transition.

C3 removes the reverse transition. Once the nearer binding is present, no
future deletion invalidation is required.

For names not statically represented in a nearer context, exact overflow lookup
or a suitable structural/topology assumption remains necessary before selecting
a more distant static binding.

## Assignment destination before RHS

The existing semantic ordering remains:

~~~text
destination = resolve lexical destination/address
evaluate RHS
  RHS may create a nearer binding
write to previously selected destination
~~~

An indexed architecture must preserve the selected destination as a stable
semantic address, for example:

~~~text
(context identity, static index)
~~~

or, for dynamic overflow:

~~~text
(context identity, semantic name)
~~~

It must never re-resolve the destination after the RHS.

## One authoritative value store

The central architecture boundary is:

~~~text
ONE_SEMANTIC_BINDING_VALUE_AUTHORITY_REQUIRED=YES
~~~

A genuine execution context must not have independent authoritative copies in
both:

~~~text
Truffle frame / BytecodeLocal
and
ProtosObjectValue localSlots
~~~

The recommended candidate instead makes a backend-neutral indexed semantic
execution-context store authoritative. Reflection, capture, lexical execution
and debugger projection all observe the same values/presence state.

Ordinary non-execution-context objects remain ordinary
`ProtosObjectValue` instances and retain their current storage/removal model.

## First-class context projection

The semantic `context` object remains stable and eagerly valid under this
decision. Lazy physical creation is explicitly out of scope.

Conceptually:

~~~text
context.hasSlot(name)
  static layout name -> present[index]
  otherwise          -> overflow.contains(name)

context.slotValue(name)
  static layout name -> indexed value iff present
  otherwise          -> overflow

context.slotNames()
  enumerate PRESENT static names + overflow names
  apply normative semantic String ordering
  return fresh snapshot Array
~~~

The normative `slotNames()` order is independent of source declaration order,
physical layout and insertion history. Therefore fixed layout indexes do not
create an observable ordering problem.

## Closure capture

Closures continue to capture semantic execution contexts by reference.

Where lexical analysis proves a fixed binding identity and lexical depth, the
Bytecode lowering may carry depth/index operands directly.

Where a nearer binding may still legally appear and retarget lookup, execution
must use presence/topology guards or exact fallback.

The semantic captured object remains the authority. A Context-local executable
plan may be rematerialized without making the originating Truffle frame the
portable semantic identity of captured lexical state.

## Debugger and reflection

The debugger should enumerate/project the same semantic indexed store and
dynamic overflow used by guest execution.

Backend-private Bytecode temporaries remain hidden.

No second debugger-only variable universe is introduced.

PLAT015 therefore remains compatible: the scope is still activation/semantic
state projection even if the storage under the activation is indexed rather
than String-map based.

## Bytecode DSL 25.3.4.1 findings

The pinned Truffle line provides the relevant native mechanisms:

- `BytecodeLocal`;
- `LocalAccessor`;
- `MaterializedLocalAccessor`;
- `enableMaterializedLocalAccesses`;
- local name/info metadata;
- local clear/is-cleared state;
- yield/continuation frame materialization already used by Protos.

These APIs establish that a frame-authoritative candidate is technically
possible and that fixed-depth captured access is a supported implementation
pattern.

They do **not** establish that a Truffle frame should become the semantic
authority for Protos' first-class, reflective, escaping execution-context
objects.

No recommendation in this investigation depends on a Truffle feature newer than
25.3.4.1.

## Comparative implementation evidence

The comparative research retained the following material patterns:

- **TruffleSOM**: source/compiler variables carry fixed slot index and lexical
  context level; non-local access targets materialized outer frames.
- **TruffleSqueak**: first-class Context objects coexist with materialized
  Truffle frames; frame discovery/materialization is explicitly treated as a
  potentially deoptimizing boundary.
- **TruffleRuby**: local/declaration variables use frame slot/depth information;
  first-class Binding and debugger scope bridge through materialized frames.
- **GraalJS**: parser environments resolve static frame slots and scope depth;
  reified/dynamic scope mechanisms are separate where language dynamism
  requires them.
- **GraalPy**: indexed frame locals coexist with explicit materialized-frame and
  Python-frame/debug projection machinery.
- **Espresso**: JVM guest locals are fundamentally static indexed frame state.
- **Sulong/LLVM**: optimized frame representation is separate from local-variable
  debug metadata/projection.
- **SimpleLanguage**: parser-resolved names become integer frame slots rather
  than universal runtime dictionaries.
- **Apple Pkl**: lexical frame reads carry slot and `levelsUp`; dynamic/object
  lookup remains a separate mechanism.

The common lesson is not that Protos must copy one frame representation. The
common lesson is that stable lexical identity/depth should remain compiler
visible where known, while dynamic/reflective cases use an explicit separate
mechanism.

## Candidate set after C3

### Candidate A — status quo

Keep all guest lexical bindings in execution-context object maps and resolve
them through `ProtosActivation + String` traversal.

Advantages:

- current semantics already work;
- context/reflection/capture remain straightforward.

Costs:

- known lexical identity/depth is discarded;
- all static accesses pay for the generic mechanism;
- the Bytecode DSL cannot directly see facts already known before execution.

### Candidate B — indexed current locals only

Use indexed locals for definitely-current bindings but retain the old capture
model.

Disposition: **not a coherent durable end state**.

It either creates duplicated authority or postpones the same authority problem
to captured bindings.

### Candidate C — hybrid DSL locals/materialized locals without an authority seam

Use DSL locals for current bindings and materialized frame locals for captured
bindings, plus fallback.

Disposition: **not a complete architecture**.

Once authority is specified it collapses into Candidate D or E1.

### Candidate D — frame-backed semantic-context adapter

The Truffle frame is authoritative for statically admitted lexical bindings.
The first-class Protos context object becomes an adapter/projector over those
frame values plus dynamic overflow.

Advantages:

- strongest direct use of Truffle's lexical machinery;
- potentially excellent compiler visibility.

Costs/risks:

- semantic context lifetime/identity becomes coupled to Truffle
  frame/materialization lifetime;
- escape/reflection/debugging require substantial adapter machinery;
- source-backed Closure execution-plan rematerialization across Polyglot
  Contexts becomes more delicate;
- a future non-Truffle implementation would need a different fundamental
  authority substrate.

### Candidate E1 — backend-neutral indexed semantic-context store

The semantic execution context remains the single value/presence authority, but
uses immutable static layout indexes plus dynamic overflow.

Bytecode operations receive constant binding/depth/index information where
proven. Exact fallback remains only where semantics require it.

Advantages:

- preserves first-class context semantics directly;
- removes universal String lookup for statically resolved accesses;
- keeps backend and semantic authority separated;
- gives one representation to guest execution, capture, reflection and debugger;
- preserves a clean non-Truffle backend path;
- C3 makes established membership permanently stable.

Cost:

- Protos retains its own indexed semantic environment representation rather than
  delegating guest-binding authority entirely to Truffle frame locals.

### Candidate F — indefinite deferral

Operationally this is Candidate A with no migration boundary.

Disposition: **not distinct enough to justify separate selection**. The concrete
cost is continued universal String/name traversal and continued loss of static
identity at the sole executable backend boundary.

## GITHUB010 comparative scoring

Scores are 1–5 with confidence H/M/L. Scores are comparison aids, not arithmetic
authority.

| Dimension | A status quo | D frame authority | E1 indexed semantic store |
| --- | --- | --- | --- |
| Correctness/invariants | 5/H | 4/M | 5/H |
| Protos alignment | 4/H | 3/M | 5/H |
| Pay for present need | 3/H | 2/M | 4/H |
| Grow as needed | 2/H | 3/M | 5/H |
| Future-option resilience | 3/M | 3/M | 5/H |
| Scalability | 3/M | 4/M | 4/M |
| Conceptual simplicity | 4/H | 2/M | 4/H |
| Portability/implementation freedom | 5/H | 2/H | 5/H |
| Runtime/resource cost | 3/M | 3/L | 4/M |
| Failure/operability | 4/H | 2/M | 4/H |
| Deferral/reversibility/migration | 2/H | 2/M | 5/H |
| Evidence maturity/risk | 5/H | 3/M | 4/H |

Non-compensating red flags:

- **A underengineering:** it indefinitely preserves a representation mismatch
  after the Bytecode DSL became the sole executable backend.
- **D overcoupling:** it makes Truffle frame/materialization machinery the
  storage authority behind a portable, first-class semantic object.
- **E1:** no comparable non-compensating red flag was found. Its main tradeoff is
  foregoing frame-authoritative guest values in the initial architecture.

## Stress analysis

E1 remains coherent under:

- deep lexical nesting via immutable depth/index metadata;
- many bindings via indexed arrays/presence state rather than universal maps;
- many escaping Closures via context-reference capture rather than value copies;
- mutation after capture through the same authoritative indexed context;
- late nearer creation through presence/topology guarding and one-way
  invalidation;
- sequential/default parameters through independent presence;
- module partial initialization through the same presence model;
- suspension/resumption because lexical authority remains in semantic context
  state rather than copied continuation metadata;
- non-local return and dynamic control as independent runtime subsystems;
- debugger attachment by projection from the same semantic authority;
- many Polyglot Contexts because executable plans may be Context-local while
  semantic captured contexts are not Truffle-frame identities;
- AOT/native-image because layout/index metadata is immutable;
- future non-Truffle backends because the semantic store is backend-neutral.

## Future C0 escape path

If a future semantic decision restores execution-context removal, E1 does not
need a foundational replacement because explicit presence and dynamic fallback
already exist.

The later delta would require:

- permitting `present[index] true -> false`;
- invalidating permanence assumptions;
- reopening outer/receiver resolution;
- removing the member from reflection/debugger projections;
- adding disappearance tests across capture and lookup.

Therefore:

~~~text
C0_FUTURE_EXTENSION_COST=MODERATE_INCREMENTAL_COST
~~~

That future cost is real but does not justify paying for deletion machinery
under ratified C3 today.

## Recommended candidate

**RECOMMENDATION — PENDING PROJECT-OWNER APPROVAL**

Select:

~~~text
CANDIDATE=E1
NAME=BACKEND_NEUTRAL_INDEXED_SEMANTIC_CONTEXT_LEXICAL_STORE
~~~

Architecture:

~~~text
stable first-class ProtosExecutionContextValue identity
+
immutable lexical layout / BindingId / index metadata
+
one per-context authoritative indexed value store
+
explicit independent semantic presence
+
dynamic overflow for names outside the static layout
+
fixed lexical depth/index where proven
+
presence/topology guards for late nearer creation
+
exact name-based fallback where static proof is unavailable
+
capture of semantic contexts by reference
+
reflection/debugger projection from the same state
+
no second authoritative BytecodeLocal/frame value
~~~

The recommendation refines AUD016 rather than contradicting it:

- AUD016 correctly established that static lexical identity should reach the
  Bytecode backend;
- PLAT036 finds that compiler-visible static identity does **not** require the
  Truffle frame itself to become the semantic value authority.

For the minimum architecture:

~~~text
GUEST_BINDING_VALUE_AUTHORITY_IN_BYTECODELOCAL=NO
BYTECODELOCAL_FOR_BACKEND_PRIVATE_TEMPORARIES=YES
STATIC_BINDING_INDEX_DEPTH_OPERANDS_TO_BYTECODE=YES
FUTURE_DERIVED_FRAME_CACHE=DEFERRED_EXPERIMENT
~~~

## Minimum later implementation decomposition

No implementation is authorized before ratification.

If E1 is approved, the natural independent implementation Issues are:

1. canonical BindingId / lexical layout / presence classification;
2. indexed execution-context authority seam: values + presence + overflow;
3. definitely-current indexed lexical read/create/assign lowering, including
   pinned assignment destination;
4. sequential/default parameter indexed lowering;
5. captured depth/index lowering plus late-creation guards/fallback;
6. debugger/reflection projection over indexed state;
7. removal of obsolete universal lexical String-traversal responsibilities from
   `ProtosActivation`.

Each is independently meaningful and may be closed or blocked separately, so
each satisfies the Issue granularity threshold rather than being merely a patch
slice.

## Explicit deferrals

PLAT036 does not decide:

- lazy physical creation/materialization of the semantic `context` object;
- handler-tail-call compilation;
- boxing-elimination types;
- uncached interpreter/startup mode;
- generic force-quickening;
- serialization;
- the magnitude of any PERF010/PERF011 performance benefit;
- per-name versus coarse structural assumptions;
- optional derived `BytecodeLocal` caches.

These remain separate performance or platform questions.

## Strongest argument against E1

Truffle already provides purpose-built current and materialized lexical-local
machinery, and mature Truffle implementations successfully make frame slots the
primary lexical representation. Keeping a Protos-owned indexed semantic store
therefore means maintaining a language-specific environment layer that the
Bytecode DSL might otherwise optimize more naturally.

The reason E1 is still recommended is authority, not a performance claim:
Protos exposes `context` as a stable, mutable, reflective, escaping semantic
object, while executable Closure plans may be Context-local/rematerialized.
C3 removes deletion but does not remove those properties. Frame authority would
install a stronger Truffle coupling than current semantics require.

## Invariant consistency

The recommendation preserves:

- D179/C3 monotonic context membership;
- first-class execution-context identity;
- ordinary Context delegation behavior;
- capture by reference;
- late creation and retargeting;
- explicit presence and PRESENT(null);
- assignment destination before RHS;
- one semantic binding authority;
- exact dynamic fallback;
- PLAT014/019/021 continuation and control boundaries;
- PLAT015 debugger-scope authority;
- PLAT026/034 tooling/tagging boundaries; and
- PLAT035 single Bytecode executable backend.

~~~text
GITHUB021_INVARIANT_CHECK=PASS
~~~

## Investigation checkpoint

~~~text
PLAT036_REEVALUATION_RESULT=COMPLETE

PRODUCT_REVISION=75231601e458930684d4b619dd5f4722377aa65d
I067_PROJECT_RECORD_REVISION=e17d2554d574d85d009133b183a25844ba769700

D179_SELECTED_CANDIDATE=C3
C0_STATUS=DEFER_RECONSIDER_LATER

STATIC_INFORMATION_LOST_BEFORE_DSL=YES
USEFUL_STATIC_INDEXED_CURRENT_LOCAL_SET=YES
USEFUL_STATIC_INDEXED_CAPTURED_SET=YES

EXPLICIT_SEMANTIC_PRESENCE_REQUIRED=YES
DYNAMIC_OVERFLOW_REQUIRED=YES
DYNAMIC_FALLBACK_REQUIRED=YES
DYNAMIC_FALLBACK_REQUIRED_FOR_ALL_LEXICALS=NO

ONE_SEMANTIC_BINDING_VALUE_AUTHORITY_REQUIRED=YES

C3_REMOVES_PRESENT_TO_ABSENT_INVALIDATION=YES
C3_REMOVES_LATE_CREATION_INVALIDATION=NO
C3_REMOVES_NEED_FOR_PRESENCE=NO
C3_ARCHITECTURAL_SIMPLIFICATION=MATERIAL

C0_FUTURE_EXTENSION_COST=MODERATE_INCREMENTAL_COST

COMPLETE_CANDIDATE_SET_REBUILT=YES
GITHUB010_COMPLETE=YES
GITHUB021_INVARIANT_CHECK=PASS

CANDIDATE_RECOMMENDED=YES
CANDIDATE_RECOMMENDED_ID=E1
RECOMMENDATION_STATUS=PENDING_OWNER_APPROVAL
CANDIDATE_RATIFIED=NO

IMPLEMENTATION_AUTHORIZED=NO
NEXT_GATE=EXPLICIT_PROJECT_OWNER_APPROVAL_OR_REJECTION
~~~
