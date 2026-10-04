# TEST009-G — causal diagnosis of strict-gate call warnings

Evidence date: **2026-10-04**

Owning work item: `TEST009 / guillermomolina/protos#795`

Trigger: `TEST009-F` strict Truffle compilation gate on
`guillermomolina/protos@45a6df118b83396f742667d05c1797ee2843e7e0`.

## Work type and publication state

TEST009-G is a read-only investigation. It did not change Protos product code,
tests, specifications, build files, version metadata, or changelog state.

Exact inspected product revision:

~~~text
PROTOS_REVISION=45a6df118b83396f742667d05c1797ee2843e7e0
COMMIT_SUBJECT=TEST009-F: add strict Truffle compilation gate
HEAD_ADVANCED_DURING_INVESTIGATION=NO
~~~

The maintainer reports the existing local test suite PASS after this
investigation. That report validates the unchanged checkout state; it is not a
claim that TEST009-G introduced executable changes.

## Question

TEST009-F intentionally left the strict compilation gate red on real Truffle
`call` performance warnings. TEST009-G determines whether those warning counts
represent many independent bugs or a smaller set of repeated architectural
causes, and identifies which next repair is causally justified without resuming
automatic `@TruffleBoundary` mutation.

## Conclusion

The warnings are repeated manifestations of a small number of causes, not one
bug per warning instance.

~~~text
ESTABLISHED_ARCHITECTURAL_CAUSES=4
UNATTRIBUTED_HOST_COLLECTION_BUCKET=1
AUTOMATIC_BOUNDARY_SEARCH=REJECTED
NEW_TRUFFLE_BOUNDARY_JUSTIFIED_BY_STATIC_EVIDENCE=NO
~~~

The four established causes are:

1. generic `ProtosLexicalBindingAuthority` interface dispatch after mutable
   authority handoff and frame-native fast-path fallback;
2. loss of concrete leaf type across `PreparedClosureCall`-typed Bytecode DSL
   operations;
3. genuinely polymorphic `ProtosRepresentedValue` delegation through the
   generic represented-value interface;
4. current-context / execution-plan stability not propagated far enough into
   call preparation, allowing Polyglot context lookup and the context-local
   execution-plan map hit to remain visible to partial evaluation.

Generic host `List` / `Collection` / `Map` / lambda warnings remain a separate
unattributed bucket until their first Protos frame is captured dynamically.

## Causal classification

### 1. Lexical binding authority

Relevant architecture:

~~~text
ProtosObjectValue.lexicalBindingAuthority
ProtosActivation.deferredContextAuthority
ProtosFrameLexicalBindingAuthority
ProtosMapBackedLexicalBindingAuthority
EmptyLexicalBindingAuthority
~~~

The concrete authority is not globally stable. Ordinary objects can promote
from the shared empty authority to map-backed storage, while genuine execution
contexts can transition to frame-backed storage and repeated roots can hand the
authority off while preserving bindings.

HEAD already contains specialized frame-native paths using proven ordinals,
`LocalAccessor`, `MaterializedLocalAccessor`, and explicit
`instanceof ProtosFrameLexicalBindingAuthority` checks. The warnings therefore
arise when hot paths fall back to the generic interface for membership, read, or
write behavior.

~~~text
LEXICAL_AUTHORITY_CAUSE=GENERIC_INTERFACE_DISPATCH_AFTER_MUTABLE_AUTHORITY_HANDOFF_AND_FAST_PATH_FALLBACK
CLASSIFICATION=HOT_POLYMORPHISM_SHOULD_SPECIALIZE
CONFIDENCE=HIGH
~~~

This evidence does **not** justify treating the authority field as globally
`@CompilationFinal`.

### 2. PreparedClosureCall

HEAD has exactly four concrete prepared-call leaves:

~~~text
OrdinarySourceCall
NativeCall
ImmediateResultCall
ModuleInitializationCall
~~~

`EnterClosureCall` already contains useful specialization machinery: immediate
and native guards, a small direct-call target cache, `DirectCallNode`, and an
indirect fallback.

The strongest unresolved seam is terminal lifecycle dispatch:

~~~text
CompleteClosureCall.perform(PreparedClosureCall prepared)
    -> prepared.complete()

FinishClosureCall.perform(PreparedClosureCall prepared, Object result)
    -> prepared.finish(result)
~~~

Both operations receive only the interface, even though the concrete leaf set
is closed and small and each leaf has distinct terminal behavior.

Structured-dispatch operations show the same broad representation-erasure
pattern, but they are not required for the first repair slice.

~~~text
PREPARED_CLOSURE_CAUSE=LEAF_TYPE_INFORMATION_LOST_ACROSS_INTERFACE_TYPED_DSL_OPERATIONS
CLASSIFICATION=HOT_POLYMORPHISM_SHOULD_SPECIALIZE
CONFIDENCE=VERY_HIGH
~~~

### 3. Represented values

`ProtosRepresentedValue` is implemented by many production value families,
including Boolean, Integer, Float, String, Path, Environment, Encoding,
ActorRef, GroupRef, process/network capabilities, standard streams, and Null.
The implementations do not all derive their delegation parent in the same way.

Therefore this generic lookup step:

~~~text
receiver instanceof ProtosRepresentedValue represented
represented.representedDelegationParent(prelude)
~~~

proves only an interface, not an exact representation.

HEAD already demonstrates the correct pattern for selected hot families through
family-specific guarded lookup for canonical Boolean and Integer values.

~~~text
REPRESENTED_VALUE_CAUSE=GENERIC_REPRESENTATION_MEGAMORPHIC_INTERFACE_DISPATCH
CLASSIFICATION=HOT_POLYMORPHISM_SHOULD_SPECIALIZE
CONFIDENCE=HIGH
~~~

Further implementation must first determine which additional represented
families remain hot enough to justify another specialized path.

### 4. Current context and execution-plan cache

The first identified Protos edge introducing Polyglot context acquisition is:

~~~text
ProtosBytecodeRootNode call preparation
  -> ProtosLanguageContext.currentIfEnteredForRuntime()
  -> org.graalvm.polyglot.Context.getCurrent()
~~~

The context-local Bytecode execution-plan projection then reaches:

~~~text
ProtosLanguageContext.bytecodeExecutionPlanForDefinition(...)
  -> sharedBytecodeExecutionPlans.get(template)
  -> ConcurrentHashMap
~~~

The cache miss/rebuild is already separated by an existing `@TruffleBoundary`.
The hit remains PE-visible.

The stronger current hypothesis is therefore not “ConcurrentHashMap needs a
boundary”, but that stable context/plan information has not been propagated or
cached at the right DSL edge.

~~~text
CONTEXT_PLAN_CAUSE=CONTEXT_AND_PLAN_STABILITY_NOT_PROPAGATED_TO_CALL_PREPARATION_SITE
CLASSIFICATION=CACHE_OR_STABILITY_LIFETIME_INVESTIGATION
CONFIDENCE=MEDIUM_HIGH
~~~

Any future `@Cached`, `@CompilationFinal`, or `Assumption` change must first
prove the correct multi-context lifetime.

### 5. Generic host collections and lambdas

Warnings naming generic JDK collection methods or generated lambdas do not by
themselves identify the Protos operation that introduced the subtree.

~~~text
HOST_COLLECTION_CAUSE=NOT_YET_ATTRIBUTED
CLASSIFICATION=INSUFFICIENT_EVIDENCE
~~~

No product change is justified for this bucket without a dynamic trace that
shows the first Protos frame.

## Dynamic evidence still useful

TEST009-G does not require another full-corpus strict run before releasing the
next implementation slice.

Two focused diagnostic runs are sufficient to unblock the remaining families:

~~~text
A: protos/tests/conformance/regression/closure-capture-and-parameters.protos
   -> lexical authority + PreparedClosureCall + probable context/host subtree

B: protos/tests/conformance/integer/prototype-and-receiver.protos
   + protos/tests/conformance/string/prototype-identity-and-size-receiver.protos
   -> represented-value generic lookup versus existing Integer specialization
~~~

Only if A/B fail to expose the current-context or generic collection subtree is
an additional object-call diagnostic warranted. IGV is not required yet.

## Next repair selected

One implementation slice is ready without additional diagnosis:

~~~text
NEXT_SLICE=TEST009-H
NEXT_SLICE_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
NEXT_SCOPE=PREPARED_CLOSURE_TERMINAL_LIFECYCLE_TYPE_SPECIALIZATION
IMPLEMENTATION_AUTHORIZED=YES
~~~

TEST009-H owns one cause only:

~~~text
PREPARED_CLOSURE_TERMINAL_INTERFACE_ERASURE
~~~

Its bounded target is `FinishClosureCall` and `CompleteClosureCall`: preserve
concrete prepared-call representation through type-specific Truffle DSL
specializations for the four existing leaf classes, without changing selection,
ReturnHome semantics, immediate-result behavior, module-initialization lifecycle,
control transfer, or structured dispatch.

It must not mix in lexical-authority, represented-value, context-plan,
structured-dispatch, host-collection, or new-boundary work.

## Materially inspected Protos files

~~~text
AGENTS.md
AGENTS.work/TEST.md
AGENTS.work/COORDINATION.md
Makefile
tools/truffle_compilation_gate.py
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java
src/main/java/com/guillermomolina/protos/execution/ProtosLanguageContext.java
src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java
src/main/java/com/guillermomolina/protos/runtime/ProtosExecutionContextValue.java
src/main/java/com/guillermomolina/protos/runtime/ProtosLexicalBindingAuthority.java
src/main/java/com/guillermomolina/protos/runtime/ProtosMapBackedLexicalBindingAuthority.java
src/main/java/com/guillermomolina/protos/runtime/ProtosObjectValue.java
src/main/java/com/guillermomolina/protos/runtime/ProtosRepresentedValue.java
src/main/java/com/guillermomolina/protos/runtime/ProtosValueLookup.java
protos/tests/conformance/call/closure-call-and-return.protos
protos/tests/conformance/regression/closure-capture-and-parameters.protos
protos/tests/conformance/integer/prototype-and-receiver.protos
protos/tests/conformance/string/prototype-identity-and-size-receiver.protos
protos/tests/conformance/call/object-call-and-init.protos
~~~

## Validation and constraints

~~~text
PRODUCT_CHANGE=NO
SPECIFICATION_CHANGE=NO
CHANGELOG_CHANGE=NO
POM_VERSION_CHANGE=NO
STRICT_GATE_RELAXATION=NO
AUTOMATIC_BOUNDARY_MUTATION=NO
NEW_FORMAL_ISSUE_REQUIRED=NO
MAINTAINER_REPORTED_LOCAL_TESTS=PASS
~~~

TEST009 remains open. TEST009-F remains the strict detector authority; G only
classifies its current warnings and releases the first bounded causal repair.

AI assistance: this durable record was drafted with ChatGPT from the exact
published TEST009-F product revision, live TEST009 issue history, current source
architecture, and maintainer-reported local validation.
