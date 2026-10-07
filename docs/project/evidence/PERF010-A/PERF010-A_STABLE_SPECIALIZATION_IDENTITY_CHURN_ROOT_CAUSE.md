# PERF010-A — Stable-specialization identity churn root cause and Smalltalk/Truffle precedent

Date: 2026-09-23

## Scope

This record retains the read-only causal analysis performed after the published
prepared Context-owned target intervention.

It does **not** change Protos semantics, product code, versions, tests, benchmark
code, or benchmark timing policy. It identifies why the exact caller helper
initially benefits from `PrepareSendArguments.fastOrdinarySend` and later
returns permanently to the generic `perform` path.

## Revision-bound inputs

```text
PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=3a4afc27a96ae4efd6f3f0c71bc0990d5d102e30
PROTOS_VERSION=0.3.78-SNAPSHOT

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
BENCHMARK_REVISION=e0bbf213c8140491f91913712cbf6d9cd0270e0b

PRODUCT_ARTIFACT=perf010a-product-prepared-target-caller-helper-lifecycle.zip
PRODUCT_ARTIFACT_SHA256=b00b40e4858c46266a52e38d164cd2516eb0e0d2bde27dd3d9183d65f0776ead
PRODUCT_STDOUT_SHA256=4dffe7a514a6001dc6a27250fa0367a6664efc5a5a6881bc7fd48b49fea53963
PRODUCT_STDERR_SHA256=5d7159552f29eaea02c63eef454478d34983906230b207c71f61f6db299a72e4

RETAINED_BASELINE_STDERR_SHA256=2c4c381ebdc515f0798b46fc2508672b113e99a072058159de1657490372647d
RETAINED_GUARDED_STDERR_SHA256=f94328e450314ec847ec8572517219cc186ff5505f6fb941d6146f1e46099ea4
```

The exact helper under investigation is:

```text
CALLER_HELPER_ID=153
CALLER_HELPER_NODE=ProtosBytecodeRootNodeGen@3c01cfa1
CALLER_HELPER_SOURCE=method-call.protos:29
```

The harness reports:

```text
source_reused=true
process_reused=true
context_reused=true
fresh_activation_per_iteration=true
```

## Established causal result

```text
PERF010A_STABLE_COMPILATION_BLOCKER=ESTABLISHED

PRIMARY_CAUSE=
  EPHEMERAL Closure + methodHome IDENTITIES ARE PART OF
  fastOrdinarySend SPECIALIZATION IDENTITY

MECHANISM=
  SOURCE REEXECUTION CREATES FRESH receiver / Closure / methodHome
  -> FAST PIC ADDS DISTINCT ENTRIES
  -> limit=3 IS EXHAUSTED
  -> perform(replaces="fastOrdinarySend") BECOMES ACTIVE
  -> FAST INSTANCES ARE REMOVED FOR THAT NODE
  -> GENERIC perform BECOMES PE-VISIBLE AGAIN
  -> PREVIOUS STANDARD-IMPORT / MODULE-RESOLUTION HOST PATH REENTERS PE
  -> PERMANENT BAILOUT RETURNS

NEW_LANGUAGE_DECISION_REQUIRED=NO
BOUNDED_IMPLEMENTATION_EXPERIMENT_READY=YES
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
TIMING_READY=NO
```

## Why the cache key churns

At the fixed Protos revision, the fast specialization guards include:

```java
selector.equals(cachedSelector)
closure == cachedClosure
methodHome == cachedMethodHome
enteredContext == cachedContext
cachedTarget != null
```

with `limit = "3"`, followed by:

```java
@Specialization(replaces = "fastOrdinarySend")
public static PreparedClosureCall perform(...)
```

The same product revision establishes the identity lifecycle:

1. `PrepareObjectConstruction.perform` creates a new
   `ProtosObjectValue(parent)` for an object literal execution.
2. `MaterializeClosure.perform` creates a new `ProtosClosureValue(...)`
   when the Closure literal executes.
3. `CanonicalClosureMaterializationTest` explicitly requires two evaluations
   of the same Closure literal to produce distinct Closure identities.
4. Authoritative D013 lookup returns the actual slot-owning
   `ProtosObjectValue` as `methodHome`, so a freshly created receiver also
   yields a fresh home identity.
5. The selector is a constant in the reused bytecode.
6. The entered `ProtosLanguageContext` is stable because the harness reuses
   the same Context.
7. The executable plan/Context-local projection is definition/template based
   and can remain compatible across these fresh semantic object identities.

Therefore the effective lifecycle is:

```text
selector          STABLE
Closure identity  CHANGES ACROSS SOURCE EXECUTIONS
methodHome        CHANGES ACROSS SOURCE EXECUTIONS
entered Context   STABLE
effective target  COMPATIBLE / STABLE
```

Inside one source execution, the 10,000 sends are effectively monomorphic.
Across executions, the current guard treats fresh semantic identities as new
specialization shapes.

## Exact helper lifecycle

The retained product trace for helper `id=153` shows:

```text
07:56:55.277  Tier1 @ 400     queued
07:56:55.938  Tier1           start
07:56:56.660  Tier1           DONE       IR 3588

07:56:57.544  Tier2 @ 10000   queued

07:56:58.963  Tier2 @ 10001   queued
07:56:58.963                  INVALIDATE uncommon trap
07:56:59.372  Tier2           start
07:57:00.347  Tier2           DONE       IR 4284

07:57:00.953                  INVALIDATE uncommon trap
07:57:00.953  Tier1 @ 15926   queued
07:57:00.956  Tier1           start
07:57:01.904  Tier1           DONE       IR 4971

07:57:01.906  Tier2 @ 20423   queued
07:57:01.907  Tier2           start
07:57:03.144                  INVALIDATE uncommon trap
07:57:03.185  Tier2           DONE       IR 4972
07:57:03.187                  INVALIDATE uncommon trap

07:57:03.187  Tier1 @ 26130   queued
07:57:03.188  Tier1           start
07:57:03.464                  PERMANENT BAILOUT
```

The compiled IR growth is:

```text
3588 -> 4284   +696
4284 -> 4971   +687
4971 -> 4972     +1
```

The two near-equal first increments are consistent with successive cached fast
instances being added. The final rewrite occurs once the three-entry fast cache
can no longer accept another distinct Closure/home pair.

The count values after code has compiled are not used as a direct global
source-execution counter. The causal source-boundary attribution comes from the
product identity lifecycle above: the receiver/Closure/home identities change
when the source body reconstructs them.

## Truffle DSL consequence

Truffle's specialization contract is material here:

- `limit` bounds the number of instances of a specialization that may be
  created;
- `replaces` removes the replaced specialization instances when the replacing
  specialization is instantiated and prevents the replaced specialization from
  being instantiated again for that node.

The numeric generated state-bit masks were not retained in the artifact and are
not required for this causal classification.

## Smalltalk-on-Truffle precedent

The user's recollection has a strong architectural basis.

This investigation did **not** find evidence that TruffleSOM or TruffleSqueak
published an issue describing the exact Protos failure sequence. What their
implementations do establish is the relevant design precedent: ordinary
Smalltalk dispatch caches **stable receiver classification / lookup state**, not
the identity of each freshly allocated receiver object.

### TruffleSOM

Repository/revision:

```text
smarr/TruffleSOM
73f6d2e654022565ec7c7e8ba95ae18340a862ce
```

Relevant sources:

- `src/trufflesom/src/trufflesom/interpreter/nodes/dispatch/DispatchGuard.java`
- `src/trufflesom/src/trufflesom/interpreter/nodes/dispatch/UninitializedDispatchNode.java`
- `src/trufflesom/src/trufflesom/interpreter/nodes/dispatch/GenericDispatchNode.java`
- `src/trufflesom/src/trufflesom/interpreter/nodes/dispatch/AbstractDispatchNode.java`

For ordinary `SObject` dispatch, `DispatchGuard.create(receiver)` captures the
receiver's `ObjectLayout`, and a cache entry matches another object when its
class/layout matches that stable layout. It does **not** require
`receiver == cachedReceiver`.

The explicit PIC has `INLINE_CACHE_SIZE = 6`. Only when the dispatch chain
contains too many distinct guarded shapes does
`UninitializedDispatchNode.specialize` replace the chain with
`GenericDispatchNode`, described in source as the call site becoming
megamorphic.

Thus repeated allocations of objects with the same dispatch-relevant layout do
not consume six PIC entries merely because their object identities differ.

Historical context is also explicit in commit
`88147060287095ade4b22c984e85b19d89f2f55d` ("Complete rewrite of message
send handling", 2014-03-08): TruffleSOM moved message sends to a separate
self-specializing dispatch node for method lookup and inline caching, based on
the SimpleLanguage design.

### TruffleSqueak

Repository/revision:

```text
hpi-swa/trufflesqueak
818519b2b6a6556bc524e9e0d08f7b51969cb61a
```

Relevant sources:

- `src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/dispatch/DispatchSelector0Node.java`
- `src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/dispatch/LookupClassGuard.java`
- `src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/CacheLimits.java`

Current zero-argument selector dispatch uses:

```java
@Specialization(
    guards = "guard.check(receiver)",
    assumptions = "dispatchDirectNode.getAssumptions()",
    limit = "INLINE_METHOD_CACHE_LIMIT")
```

where the cached guard is a `LookupClassGuard`, not receiver identity.
`INLINE_METHOD_CACHE_LIMIT` is 4. The cached direct-dispatch node stores the
resolved method/target plus assumptions. When that stable-class cache really
becomes megamorphic, `doIndirect` replaces `doDirect`.

This is particularly close to the Protos failure mechanism: both use a bounded
Truffle PIC and a generic replacing specialization. The key difference is what
counts as a distinct cache entry.

### Published Smalltalk/Graal performance pathologies

Two published results make the recollection stronger than source-code analogy alone.

1. **Cross-Language Compiler Benchmarking: Are We Fast Yet?** reports a
   TruffleSOM outlier in the CD benchmark. The slow red-black-tree element
   access was optimized by Truffle's object model for different observed types,
   which caused a **megamorphic access that was not yet correctly optimized in
   TruffleSOM**. This is not the same mechanism as Protos' Closure/home identity
   churn, but it is the same general failure class: runtime representation makes
   a hot operation appear more polymorphic to Truffle/Graal than the programmer
   would expect from the logical operation.

   Source:
   https://stefan-marr.de/papers/dls-marr-et-al-cross-language-compiler-benchmarking-are-we-fast-yet/

2. **Optimizing Communicating Event-Loop Languages with Truffle** explains that
   funneling all asynchronous messages and receiver types through one event-loop
   dispatch point would make a PIC effectively megamorphic. SOMns avoids that by
   constructing a **send-site-specific RootNode** containing the normal
   synchronous send PIC. The paper states that the ideal send is monomorphic and
   then needs only a simple identity check of the receiver's **class** before
   executing the cached method.

   Source:
   https://stefan-marr.de/papers/agere-marr-moessenboeck-optimizing-communicating-event-loop-languages-with-truffle/

These papers do not establish that Protos should adopt Smalltalk's class model.
They do establish two relevant engineering lessons:

- avoid accidental megamorphism introduced by the runtime representation or by
  funneling unrelated dynamic states through one cache identity;
- contextualize caches at the semantic call/send site and guard them with the
  most stable dispatch-relevant facts available, rather than ephemeral object
  allocation identity.

### Relevance to Protos

```text
SMALLTALK_TRUFFLE_PRECEDENT=
  STRONG_ARCHITECTURAL_PRECEDENT
  NOT AN IDENTICAL DOCUMENTED HISTORICAL BUG

TRUFFLESOM_CACHE_IDENTITY=
  RECEIVER CLASS / OBJECT LAYOUT + RESOLVED METHOD TARGET

TRUFFLESQUEAK_CACHE_IDENTITY=
  LOOKUP CLASS GUARD + METHOD/TARGET ASSUMPTIONS

PROTOS_CURRENT_CACHE_IDENTITY=
  SELECTOR + Closure OBJECT IDENTITY + methodHome OBJECT IDENTITY
  + ENTERED CONTEXT + TARGET PAYLOAD

PROTOS_PATHOLOGY=
  SEMANTICALLY MONOMORPHIC REEXECUTION IS MISCLASSIFIED AS
  POLYMORPHIC BECAUSE FRESH OBJECT IDENTITIES CONSUME PIC ENTRIES
```

The Smalltalk precedent does **not** license copying a class guard directly:
Protos' prototype/object semantics and D013 mutation visibility differ from
Smalltalk class-based method lookup.

It does support the implementation direction already derived independently from
the Protos trace:

1. keep authoritative D013 lookup on every invocation;
2. keep the currently selected Closure and current `methodHome` for the fresh
   invocation activation;
3. do not make those ephemeral semantic object identities the persistent PIC
   identity when they resolve to the same compatible executable behavior;
4. cache stable executable/lookup compatibility, with exact fallback whenever
   that compatibility no longer holds;
5. preserve native/unsupported generic behavior.

## Next bounded experiment

```text
NEXT_ACTION=IMPLEMENTATION_EXPERIMENT
REPOSITORY=guillermomolina/protos

GOAL=
  MAKE fastOrdinarySend SPECIALIZATION IDENTITY STABLE ACROSS
  REEXECUTION THAT RECREATES receiver / Closure / methodHome,
  WITHOUT WEAKENING D013 OR INVOCATION SEMANTICS

ACCEPTANCE_GATE=
  SAME method-call.protos:29 HELPER REMAINS ON THE FAST/PREPARED
  EXECUTABLE PATH AND DOES NOT REENTER PrepareSendArguments.perform
  THROUGH PIC LIMIT EXHAUSTION

TIMING=
  DEFER UNTIL COMPILER LIFECYCLE GATE PASSES
```


## Stable executable-identity implementation publication — 2026-09-23

The bounded implementation experiment derived above is now published in
`guillermomolina/protos`.

Exact product identity:

```text
PROTOS_REVISION=3e8e6b565c95eb5098c2168d241536ba13ad19e9
PROTOS_VERSION=0.3.79-SNAPSHOT
COMMIT_MESSAGE=PERF010-A: key fastOrdinarySend cache on stable Closure definition identity
```

Published implementation change:

```text
OLD_FAST_CACHE_IDENTITY=
  selector equality
  + selected ProtosClosureValue object identity
  + selected methodHome object identity
  + entered ProtosLanguageContext identity

NEW_FAST_CACHE_IDENTITY=
  selector equality
  + selected Closure CanonicalClosure definition identity
  + entered ProtosLanguageContext identity
```

The currently selected `ProtosClosureValue` and `methodHome` are still
obtained from authoritative D013 lookup on every invocation and are passed into
a fresh `ProtosActivation`. Only the persistent DSL specialization identity
changed.

The cached Context-owned `RootCallTarget` remains materialized once when the
specialization entry is populated. The cache limit remains `3`; the change
does not raise or remove the limit and does not alter the generic replacing
fallback.

Published focal regression coverage adds:

`freshClosureAndReceiverMaterializationsOfTheSameDefinitionRemainCorrectPastTheCacheLimit`

which executes ten invocations through the same lowered send site while
recreating the receiver, selected `ProtosClosureValue`, and method home around
one shared `CanonicalClosure` definition / execution-plan template. The human
executor reported all seven tests in
`ProtosPerf010APreparedTargetSpecializationTest` PASS.

This focal regression establishes semantic correctness across more fresh
materializations than the previous PIC limit. It does **not**, by itself,
replace compiler-lifecycle evidence for the generated Truffle specialization
state. The next discriminator remains the exact caller-helper lifecycle trace.

Current classification:

```text
PERF010A_STABLE_EXECUTABLE_IDENTITY_IMPLEMENTATION=PUBLISHED

EPHEMERAL_CLOSURE_IDENTITY_IN_PIC_KEY=NO
EPHEMERAL_METHOD_HOME_IDENTITY_IN_PIC_KEY=NO
STABLE_CANONICAL_DEFINITION_IDENTITY_IN_PIC_KEY=YES
ENTERED_CONTEXT_IDENTITY_IN_PIC_KEY=YES
AUTHORITATIVE_D013_LOOKUP_PER_INVOCATION=PRESERVED
CURRENT_CLOSURE_FOR_ACTIVATION=PRESERVED
CURRENT_METHOD_HOME_FOR_ACTIVATION=PRESERVED
FRESH_ACTIVATION=PRESERVED
CACHE_LIMIT_CHANGED=NO
GENERIC_FALLBACK_PRESERVED=YES

FOCAL_CORRECTNESS=PASS
GENERATED_SPECIALIZATION_LIFECYCLE_AFTER_CHANGE=NOT_MEASURED
CALLER_HELPER_PERMANENT_BAILOUT_AFTER_CHANGE=NOT_MEASURED

TIMING_READY=NO
```

## Next discriminator after publication

Run the existing caller-helper lifecycle/source-identity investigation against
exactly:

```text
PROTOS_REVISION=3e8e6b565c95eb5098c2168d241536ba13ad19e9
WORKLOAD=micro/method-call
SOURCE=method-call.protos
LINE=29
TEXT=sink = receiver.identity(42)
```

using the retained `guillermomolina/protos-benchmarks` harness and the same
process/Context/Source reuse conditions that exposed the previous
`1 -> 2 -> 3 -> generic` transition.

Classify:

```text
FAST_SPECIALIZATION_CACHE_CHURN=
  REMOVED | MATERIALLY_CHANGED | UNCHANGED | INCONCLUSIVE

GENERIC_REPLACING_SPECIALIZATION_ACTIVATED=
  YES | NO | INCONCLUSIVE

CALLER_HELPER_PERMANENT_BAILOUT=
  REMOVED | MATERIALLY_CHANGED | UNCHANGED | INCONCLUSIVE

COMPILER_LIFECYCLE_STABLE=
  YES | NO | INCONCLUSIVE

TIMING_READY=
  YES only if the lifecycle evidence is stable enough to interpret timing
```

Do not interpret timing before this lifecycle discriminator is known.
