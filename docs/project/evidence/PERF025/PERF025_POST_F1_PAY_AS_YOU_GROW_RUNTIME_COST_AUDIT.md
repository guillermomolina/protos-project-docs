# PERF025 — Post-F1 pay-as-you-grow runtime-cost audit

Status: **EXHAUSTIVE STATIC AUDIT / DURABLE EVIDENCE / NO PERFORMANCE MAGNITUDE CLAIM**

Date: 2026-10-02

This durable, non-normative record retains the post-PERF025-F1 audit requested
for `guillermomolina/protos#758`: search for runtime costs caused by realizing
the most general Protos semantics physically on every ordinary execution, even
when the corresponding capability is not used.

The audit deliberately distinguishes semantic requirements from physical
representation. It does not change the Protos specification, reopen any ratified
platform decision, authorize an implementation, or claim a measured attribution
for any candidate below.

## Identity and current-head applicability

The exhaustive source audit was performed against:

```text
AUDIT_PRODUCT_REVISION=59fcb8552bf294be4e5c8ffdbe7786cf93194490
AUDIT_PRODUCT_VERSION=0.3.146-SNAPSHOT
AUDIT_PRODUCT_SUBJECT=PERF025-F1: direct-caller hosted-session execution
```

Current Protos HEAD was then checked:

```text
CURRENT_PRODUCT_REVISION=c8e0e0d59d5541007d733e8a0b53a3cf123e5818
CURRENT_PRODUCT_VERSION=0.3.147-SNAPSHOT
CURRENT_PRODUCT_SUBJECT=PERF025-G1: lazy Task child bookkeeping

AUDIT_TO_CURRENT_COMMITS=1
AUDIT_TO_CURRENT_CHANGED_PRODUCT_SURFACE=
  CHANGELOG.md
  pom.xml
  src/main/java/com/guillermomolina/protos/runtime/ProtosTask.java
  src/test/java/com/guillermomolina/protos/runtime/ProtosPerf025G1LazyTaskChildrenTest.java
```

Therefore every finding below whose implementation owner is outside
`ProtosTask.java` remains source-identical at current HEAD. G1 partially consumes
only the RootTask/Task bookkeeping line by making the structured-child set lazy
and sharing the stateless child-drain sentinel. G1 does not change call
activation, frame materialization, Closure capture, local binding, represented
values, Map/Array/Bytes/String representation, member lookup, or Polyglot-context
lookup.

## Governing implementation principle

The common architectural pattern discovered by this audit is:

```text
semantic capability exists
    !=
physical machinery for that capability must exist eagerly on every use
```

The intended pay-as-you-grow form is instead:

```text
ordinary cheap representation
    -> guard / assumption / proven static property
    -> materialize richer semantic machinery only when it becomes observable
    -> deopt/fallback when an invalidating operation actually occurs
```

This is compatible with the normative contracts inspected during the audit.
Several specifications explicitly permit scalar replacement, virtualization,
copy-on-write, physical sharing, alternative Map indexing, and other
representation strategies when observable semantics are preserved.

The audit also identified positive counterexamples in the current
implementation: Bytecode DSL instrumentation/source information is created with
`BytecodeConfig.DEFAULT` and is materialized only when requested. The problem is
therefore not that Protos cannot implement optional capability lazily; several
hot runtime paths simply do not yet use the same discipline.

## Highest-priority findings

### 1. Any frame-backed local forces unconditional Truffle frame materialization

`CanonicalToBytecodeLowerer` emits `InstallFrameLexicalAuthority` whenever the
current root has at least one declared frame local:

```text
if (!frameLocals.isEmpty())
    -> InstallFrameLexicalAuthority
```

The operation unconditionally executes:

```text
VirtualFrame.materialize()
    -> MaterializedFrame
    -> ProtosFrameLexicalBindingAuthority
```

The admission condition is not "this frame is captured", "context is observed",
"reflection is used", or "the frame escapes". A parameter alone is enough.

Consequences include:

```text
identity(value) => value
    -> materialized frame on every invocation

fibonacci(n)
    -> materialized frame on every recursive invocation
```

This is a first-order candidate because Truffle's normal optimization model
tries to keep frames virtual until an actual escaping/capture use requires
materialization.

The existing Protos binding analysis already carries exact
`CapturedResolved(owner, lexicalDepth)` information. The backend therefore
already knows which lexical accesses are genuinely captured; the missing step is
using that information to decide when a materialized owner frame is actually
needed.

### 2. Every Closure literal physically captures the enclosing execution context

`MaterializeClosure` always executes:

```text
activation.lexicalContextsForClosureCapture()
```

For a normal activation that method performs:

```text
new ArrayList
+ context()
+ copy all already-captured lexical contexts
+ unmodifiableList
```

The call to `context()` materializes a deferred
`ProtosExecutionContextValue`.

Therefore even a semantically autonomous literal such as a Closure that does not
reference an outer local can force physical enclosing-context materialization.

The language requires lexical capture semantics when a Closure actually depends
on enclosing state. It does not require every physical Closure representation to
carry an eagerly materialized capture chain when static analysis proves that the
body does not use it.

### 3. Direct Closure calls still use the eager rich activation path

PERF014 stabilized direct Closure selection and target dispatch, but
`finishDirectClosureCall(...)` still routes through
`ProtosActivation.forClosureInvocation(...)`.

That factory eagerly creates or projects:

```text
fresh execution context
captured lexical-context list
frozen guest Array for supplied arguments
return-home state
fresh ProtosActivation
Task/dynamic-control attachment
```

This applies once per direct source Closure invocation, including recursive
calls. It is therefore potentially multiplicative inside one already-prepared
host invocation and is independent of the hosted-session carrier fixed by F1.

The runtime already contains a contrasting compact/deferred factory for ordinary
source method calls:
`forImmediateMethodInvocationWithReturnHomeForRuntime(...)`.
The implementation has therefore already proved that eager guest Context and
guest Array creation are not universally required for source-call semantics.

### 4. A direct Closure call creates one context representation and immediately replaces it

The eager execution context initially owns a
`ProtosMapBackedLexicalBindingAuthority`. Root entry then installs the
frame-backed authority.

The replacement path also constructs temporary name/value arrays to preserve
already-established bindings, commonly empty at normal source-root entry.

The physical sequence is therefore approximately:

```text
create map-backed lexical authority
    -> enter source root
    -> materialize frame
    -> create frame-backed lexical authority
    -> migrate existing bindings
    -> abandon the original map-backed authority
```

This is representation churn, not an observable language requirement.

### 5. The compact ordinary method-call ABI is re-expanded immediately after entry

PLAT040 introduced a compact frame ABI:

```text
closure
receiver
methodHome
caller
returnHome
user arguments...
```

At target entry, `ProtosFrameArguments.activation(...)` reconstructs:

```text
Arrays.asList(frameArguments).subList(...)
-> ProtosActivation
-> DeferredSuppliedArguments
```

Parameter binding then reads through generic list operations and creates the
parameter as a named execution-context slot.

Thus the current path is physically:

```text
compact flat ABI
    -> list view / supplied-vector object state
    -> rich Activation
    -> generic named parameter binding
    -> frame authority / materialized frame when locals exist
```

The guest-visible context/argument Array remains partially deferred, but the
semantic call state is still substantially reified per call.

### 6. Statically known parameters are established through dynamic named-slot machinery

For an ordinary parameter such as `value`, the current binding path performs
runtime work equivalent to:

```text
LoadClosureArgument
  -> supplied.size / supplied.get

BindClosureParameter("value", value)
  -> createCurrentLocalSlotForRuntime
  -> establishmentOrder.add("value")
  -> frameBackedLayout.offsetOf("value")
  -> LocalAccessor.setObject

CheckClosureArgumentUpperBound
  -> supplied.size
```

The compiler already knows the Closure definition, parameter identity, stable
BytecodeLocal, and often the stable supplied arity.

The semantic requirement is that parameter establishment is observable as a
normal context slot at the correct point in left-to-right binding. It does not
require every optimized call to discover the parameter by String name and
maintain full reflection/order representation before such observation occurs.

### 7. Fresh return-home objects are physically established even when no return can observe them

Source invocation establishes return-home semantics, but current compact and
rich call paths commonly create a fresh `ProtosReturnHome` when the Closure did
not capture one.

The canonical AST already exposes return expressions directly. A future
optimization can therefore investigate a summary such as:

```text
NEEDS_OWN_RETURN_HOME
NEEDS_CAPTURED_RETURN_HOME
```

and preserve the semantic relationship while allowing the physical carrier to
remain virtual/lazy when no `^` path or nested Closure can observe it.

This is a representation question only; the Smalltalk-style home-activation
semantics remain unchanged.

### 8. PLAT044 inline callbacks remove the physical child root but retain rich callback activation

Eligible Boolean/while/each literal callbacks can execute their bodies inline,
but preparation still establishes a fresh semantic callback activation through
the ordinary Closure path.

The optimization therefore removes:

```text
child RootCallTarget entry
```

while retaining much of:

```text
Closure materialization
capture state
Activation
arguments
return-home/control state
callback completion state
```

For loops and recursive control-heavy code this can repeat once or more per
iteration/recursive node.

### 9. Inline callbacks deliberately disable the frame-native lexical lowering state

Current HEAD still performs the PERF026-B1-documented reset inside
`emitInlineLiteralCallback(...)`:

```text
currentRootAnalysis = null
currentRootTopScope = null
currentRootFrameLocals = Map.of()
```

The callback body therefore does not reuse the enclosing lowerer's proven
direct/captured-local lowering machinery and instead falls back toward the
general activation authority.

This means the optimization removes a root boundary while simultaneously losing
some lexical optimization knowledge.

### 10. D179 dynamic presence/removal semantics remain on every otherwise-resolved local path

A same-scope resolved read still checks whether its `LocalAccessor` is cleared.
Captured reads additionally preserve nearer-context retargeting checks.

Those checks are semantically required whenever dynamic context mutation can be
observed. The open performance question is whether an unobserved/unescaped
context can be guarded by an Assumption so that ordinary code pays the dynamic
presence machinery only after an operation capable of observing/removing/
recreating bindings actually occurs.

The audit does **not** propose weakening D179.

## Additional representation findings

### 11. Every frame lexical authority eagerly owns dynamic-overflow and establishment-order structures

`ProtosFrameLexicalBindingAuthority` constructs:

```text
LinkedHashMap<String,Object> dynamicOverflow
LinkedHashSet<String> establishmentOrder
```

for every frame-backed activation even when no dynamic binding, reflection,
remove/recreate operation, or order observation ever occurs.

These are candidate lazy-on-first-use structures.

### 12. Every ProtosObjectValue eagerly owns map-backed slot storage

Ordinary `ProtosObjectValue` construction installs a new
`ProtosMapBackedLexicalBindingAuthority`, which itself owns mutable map/order
storage.

This affects objects that may never acquire a local slot, including several
runtime value families that inherit from `ProtosObjectValue`.

A shared immutable empty authority with copy/materialization on first slot
mutation is a direct pay-as-you-grow candidate.

### 13. Ordinary member reads still cross a general dynamic lookup/materialization boundary

Member access uses general String-keyed slot/delegation lookup. The materialized
member-read path also crosses a `@TruffleBoundary`.

Stable ordinary-object properties therefore do not currently exploit a
Shape/property-cache representation comparable to mature Truffle object models.

This is a broad architectural candidate, not a small patch: Protos must retain
ordinary dynamic slot mutation, delegation, shadowing, extraction identity, and
lookup invalidation.

### 14. Integer guarded selection stabilizes *what* to call but not *how* to execute it

The represented Integer guard can prove stable standard selection, but the
successful hit still routes through the general immediate-method preparation and
native Closure invocation machinery before reaching the actual
`BigInteger` operation.

A stable canonical Integer `+`, `-`, comparison, etc. therefore still pays
substantial generic call machinery.

A stronger specialization may preserve:

```text
ordinary lookup
shadowing
selector-specific invalidation
standard-owner/implementation identity
arbitrary-precision semantics
```

while executing the already-proven standard operation directly at the
specialized operation site and falling back when the guard invalidates.

This is analogous to the Truffle DSL's intended use of specialized operations,
not a change from "everything is a message" at the language level.

### 15. Integer representation is always BigInteger-backed

Every semantic Integer is physically `ProtosIntegerValue(BigInteger)`.

The Protos numeric specification requires unbounded mathematical Integer
semantics but explicitly does not prescribe host width, boxing, tagging, or
storage representation.

A small-int primitive representation with promotion to BigInteger is therefore a
separate physical optimization candidate. It must remain distinct from the
full-call-path candidate because dispatch/activation overhead and numeric storage
are separate causes.

### 16. String operations revalidate already-proven Unicode invariants

`ProtosStringValue(String)` validates the entire Java String as a Unicode scalar
sequence.

Internal operations such as concatenating two already-valid Protos Strings
produce a value whose validity is known by construction, then invoke the same
validating constructor and scan the result again.

`String.size` also recomputes the Unicode code-point count by traversal.

A trusted internal construction path plus cached scalar count is a candidate
while untrusted/interop construction retains full validation.

### 17. Normal Map is an insertion-ordered linear list and copies the entry list for key search

`ProtosMapValue` stores entries in an `ArrayList`.

Ordinary key lookup computes the query hash and then iterates
`map.keyedSnapshot()`, where `keyedSnapshot()` is `List.copyOf(entries)`.

For equal recorded hashes, Protos must invoke directed ordinary
`queryKey == storedKey` in deterministic insertion order. The normative Map
contract explicitly defines that observable order while permitting any physical
indexing strategy that preserves it.

A hash index to insertion-ordered candidate buckets can therefore preserve the
exact logical algorithm without copying/scanning all entries for every lookup.

### 18. IdentityMap uses the same copied linear-search representation despite primitive identity semantics

`IdentityMap` also stores an insertion-ordered `ArrayList`, copies it for
search, and performs a linear scan.

Its key law uses primitive non-overridable semantic identity hash plus `===`,
so it has an even clearer opportunity for an indexed physical representation
while maintaining insertion order for iteration.

### 19. Array/Map/Bytes logical snapshots are implemented as eager physical copies

The language intentionally defines snapshot semantics for collection iteration.

The normative specification also explicitly says that the snapshot is a
semantic boundary, not a mandated eager copied representation; persistent
storage, versioned views, copy-on-write, and equivalent strategies are allowed.

The current eager snapshots are therefore correct but conservative physical
implementations.

### 20. Bytes pays synchronization even when isolated-parallel byte reservations are never used

`ProtosBytesValue` / byte-region machinery uses synchronized access broadly,
including ordinary byte operations.

The parallel-execution specification requires reservation coordination only when
`Bytes.parallelRange` / `ByteRegion.parallelRange` has established live
exclusive regions. Ordinary Bytes access outside that capability has no semantic
requirement to behave as globally concurrent shared state.

A candidate architecture is an ordinary actor-local representation with
reservation/coordinated state activated only on the first live reservation.

This requires careful validation of publication, reservation overlap, Future
completion, and parent-access failure semantics; it is not a mechanical removal
of synchronization.

### 21. Guest hot paths rediscover the entered Polyglot Context through embedding APIs

Several guarded operations bind
`ProtosLanguageContext.currentIfEnteredForRuntime()`.

That helper first checks public `org.graalvm.polyglot.Context.getCurrent()` and
then obtains the language context through a Truffle reference with a null node.

Truffle provides node-associated ContextReference access specifically so hot
compiled guest paths can specialize/constant-fold context lookup. The current
form should therefore be investigated as a host/embedding API surviving inside
guest hot execution.

No magnitude is claimed for this item.

## RootTask / Task / Actor bookkeeping status

The earlier PERF025 RootTask line remains separate because it is a per-hosted-
invocation fixed cost rather than a per-guest-operation multiplier.

Current HEAD G1 already removes two unnecessary eager Task allocations:

```text
structured child set:
  eager LinkedHashSet -> null until first child

child-drain WaitDependency:
  per Task object -> shared stateless sentinel
```

Residual RootTask/Task work remains separately investigable:

```text
fresh RootTask / Task state machine
root registration/removal in liveTasks
terminal/outcome bookkeeping
root-inapplicable generic terminal-path checks
monitor/synchronization boundaries
```

Do not merge this fixed hosted-invocation line numerically with the per-call or
per-operation candidates above.

## Important negative findings

The audit also rejected several tempting but unsupported explanations.

### Literal constants are not recreated on every iteration

Canonical numeric/String literals are materialized during lowering and loaded as
constants. Writing `1` in a loop does not by itself allocate a fresh
`BigInteger`/Protos Integer on each iteration.

Arithmetic *results* remain a separate allocation/representation issue.

### Bytecode instrumentation is already lazy by default

Protos enables instrumentation capability in the generated Bytecode root class,
but roots are created with `BytecodeConfig.DEFAULT`. Upstream Truffle defines
that default as not materializing source/instrumentation information until
requested.

Instrumentation is therefore a useful positive example of the desired
pay-as-you-grow pattern, not a current hot-path culprit.

### ProtosStaticDefinitions is tooling, not the runtime hot path

The conservative static-definition logic inspected during the audit belongs to
tooling/LSP analysis. It must not be counted as a runtime benchmark cause.

### Source-visible semantic objects do not automatically imply heap allocation

Seeing `new ProtosActivation`, `new ArrayList`, or another allocation in Java
source is not proof that the object survives Graal partial evaluation/scalar
replacement.

The findings above identify structural work and optimization barriers worth
counting/profiling. A later measured slice must distinguish physical allocation
survival from source-level construction before assigning magnitude.

## Why recursive/control-heavy workloads amplify these costs

For `fibonacci(20)`, the call tree contains 21,891 invocations of
`fibonacci` (10,945 non-base nodes and 10,946 leaves).

The current implementation therefore reaches repeated per-call/per-control
machinery on the order of tens of thousands of times inside one already-prepared
top-level invocation:

```text
~21,891 source Closure invocations
~21,891 frame-materialization opportunities from parameters/locals
~21,891 Integer comparisons
~21,891 Boolean callback-producing literal evaluations
~10,945 selected ifTrue callback executions
~21,890 Integer subtractions
~10,945 Integer additions
```

This is why eliminating only a single hosted carrier can improve the fixed floor
without explaining a remaining hundreds-times cross-runtime gap for guest-heavy
workloads.

The audit does not claim that all listed operations survive compilation or that
their individual costs add linearly. It establishes why per-call and
per-operation mechanisms are the correct next causal scale to measure.

## Priority map

Static priority by repetition frequency, structural weight, and available
compiler knowledge:

```text
P0
  EAGER_FRAME_MATERIALIZATION_FOR_ANY_LOCAL
  RICH_DIRECT_CLOSURE_INVOCATION
  UNIVERSAL_CLOSURE_CONTEXT_CAPTURE
  STANDARD_INTEGER/BOOLEAN_OPERATION_AS_FULL_CLOSURE_CALL

P1
  INLINE_CALLBACK_STILL_USES_RICH_ACTIVATION
  INLINE_CALLBACK_LOSES_FRAME_NATIVE_LEXICAL_PATH
  RETURN_HOME_PHYSICALLY_CREATED_WHEN_UNOBSERVED
  PARAMETER_BINDING_THROUGH_DYNAMIC_SLOT_AUTHORITY
  PROVEN_LEXICAL_WRITE_RUNTIME_TARGET

P2
  EAGER_FRAME_AUTHORITY_OVERFLOW_AND_ORDER_STRUCTURES
  EAGER_OBJECT_SLOT_STORAGE
  MEMBER_LOOKUP_WITHOUT_SHAPE_STYLE_PHYSICAL_SPECIALIZATION
  HOT_PATH_POLYGLOT_CONTEXT_LOOKUP
  SMALL_INTEGER_PHYSICAL_REPRESENTATION

P3
  MAP_AND_IDENTITYMAP_PHYSICAL_INDEXING
  BYTES_UNIVERSAL_SYNCHRONIZATION
  STRING_REDUNDANT_VALIDATION_AND_SCALAR_COUNT
  EAGER_COLLECTION_ITERATION_SNAPSHOTS
```

This ordering is an investigation priority, not a measured performance ranking.

## Relationship to earlier retained work

The audit explains several earlier observations without rewriting them:

- PERF010-A identified the remaining common guest-call path as activation/context/
  argument/return-home construction after smaller ablations.
- PERF014 stabilized direct Closure-call selection but did not replace the rich
  `forClosureInvocation` physical call state.
- PERF015/PERF016 stabilized represented Boolean/Integer selection; PERF016
  explicitly did not open-code the standard Integer operation.
- PLAT040/I072 established compact/deferred source-method call machinery, but
  current target entry still reconstructs significant rich state and roots with
  locals still materialize frames.
- PLAT044/PERF026 removed eligible literal callback root boundaries while
  preserving fresh semantic activation and documented the general lexical lookup
  fallback inside the inline region.
- PLAT046/PERF025-F1 correctly removed the universal hosted-session guest
  carrier. This audit is downstream of that decision and does not reopen it.
- PERF025-G1 already applies the same pay-only-when-used philosophy to Task child
  bookkeeping.

The common residual theme is therefore not "dispatch was never optimized".
Several dispatch selections are already stable. The residual question is whether
the runtime still constructs a general semantic representation after it has
proved a much narrower physical case.

## Recommended next causal slice

Do not invent another broad benchmark methodology first.

Use the existing workload/radar infrastructure and add bounded counting or
profiling that answers, for representative workloads such as primitive return,
closure-call, method-call, integer-loop, factorial and fibonacci:

```text
COUNT:
  frame.materialize / InstallFrameLexicalAuthority
  ProtosActivation.forClosureInvocation
  compact-call activation materialization
  lexicalContextsForClosureCapture / context materialization
  ProtosReturnHome creation
  rich inline-callback activation
  standard Integer/Boolean native activation
  parameter-slot establishment
  resolved lexical-write target creation
```

The purpose is attribution, not another end-to-end headline number.

After counts identify the mechanisms that repeat at the expected scale, select
one bounded mechanism, preserve exact generic fallback, implement it, and rerun
the existing exact-revision comparator.

## Formal conclusions

```text
PERF025_POST_F1_PAY_AS_YOU_GROW_AUDIT=COMPLETE

AUDIT_BASE_REVISION=59fcb8552bf294be4e5c8ffdbe7786cf93194490
CURRENT_HEAD_CHECKED=c8e0e0d59d5541007d733e8a0b53a3cf123e5818

G1_PARTIALLY_CONSUMES_ROOT_TASK_LINE=YES
NON_TASK_FINDINGS_SOURCE_IDENTICAL_AT_CURRENT_HEAD=YES

SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PLAT046_REOPENED=NO
NEW_FORMAL_ISSUE_ALLOCATED=NO

DOMINANT_CAUSE_MEASURED=NO
PER_CANDIDATE_MAGNITUDE_ESTABLISHED=NO

NEXT_REQUIRED_KIND=BOUNDED_CAUSAL_COUNTING_OR_PROFILING
NEW_BENCHMARK_METHODOLOGY_REQUIRED=NO
```

## Materially inspected sources

Product/runtime and lowering:
- `src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalLayout.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosLanguageContext.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosExecutionContextValue.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosObjectValue.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosClosureValue.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosMapBackedLexicalBindingAuthority.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosTask.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosMapValue.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosIdentityMapValue.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosArrayValue.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosBytesValue.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosByteRegionValue.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosStringValue.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosIntegerValue.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardIntegerProtocol.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardStringProtocol.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardMapProtocol.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardIdentityMapProtocol.java`
- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticDefinitions.java`

Binding analysis:
- `src/main/java/com/guillermomolina/protos/execution/CanonicalBindingAnalysis.java`
- `src/main/java/com/guillermomolina/protos/execution/CanonicalBindingResolution.java`
- `src/main/java/com/guillermomolina/protos/execution/CanonicalLexicalScope.java`

Normative owners:
- `spec/PROTOS_LANGUAGE_SPEC.md`
- `spec/semantics/CALLABLES.md`
- `spec/semantics/VALUES_AND_COLLECTIONS.md`
- `spec/concurrency/PARALLEL_EXECUTION.md`

Retained project evidence:
- PERF010-A post-I072 action/call-path evidence
- PERF014 direct Closure-call checkpoint
- PERF015/PERF016 represented selection evidence
- I072 compact/deferred call evidence
- PLAT044 / PERF026 inline-callback evidence
- PLAT046 / PERF025-F1 direct-caller evidence
- PERF025-G1 lazy Task child-bookkeeping evidence

External Truffle/Graal comparison points inspected:
- Truffle Bytecode DSL `BasicInterpreter.CreateClosure` frame materialization
- Truffle Bytecode DSL specialized `Add` operation examples
- SimpleLanguage primitive numeric specialization / promotion pattern
- Truffle language/context reference APIs intended for node-associated hot paths

No product source, specification, benchmark source, workload, or test was changed
by this audit.
