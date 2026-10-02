# PERF027 — Integer invocation and representation tax investigation

Status: **RETAINED / STATIC INVESTIGATION COMPLETE**  
Date: 2026-10-02  
Formal owner: `PERF027 / guillermomolina/protos#779`

This is durable, non-normative performance evidence. It records the
Integer-specific follow-up to PERF025's post-F1 pay-as-you-grow runtime-cost
audit. It does not change observable Protos semantics, approve a new Integer
representation, or claim dynamic allocation/performance attribution that was
not measured.

## Identity

```text
PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=c8e0e0d59d5541007d733e8a0b53a3cf123e5818
PROTOS_VERSION=0.3.147-SNAPSHOT
PROTOS_SUBJECT=PERF025-G1: lazy Task child bookkeeping

FORMAL_WORK=PERF027
GITHUB_ISSUE=guillermomolina/protos#779

TRIGGER=PERF025/#758 post-F1 pay-as-you-grow audit
RELATED_HISTORY=PERF016/#727,I075/#746,PERF011/#693
```

The detailed source audit that first reconstructed the Integer path used the
preceding F1 revision
`59fcb8552bf294be4e5c8ffdbe7786cf93194490`. Current HEAD was then reconciled
against the one product commit between F1 and G1. That commit changes Task
bookkeeping, its regression test, version and changelog only. The Integer,
activation, guarded-send and numeric-protocol implementation surfaces below are
therefore unchanged at current HEAD.

## Question

The investigated hypothesis was:

> Does ordinary Protos Integer arithmetic still pay both a complete/rich Protos
> invocation representation and a BigInteger/wrapper representation cost per
> operation even after PERF016 guarded Integer selection?

The answer is:

```text
HYPOTHESIS=
  SUBSTANTIALLY_CONFIRMED_WITH_OPERATION_SPECIFIC_QUALIFICATION
```

Two independent physical mechanisms remain:

```text
INVOCATION_TAX
INTEGER_REPRESENTATION_TAX
```

They must be isolated before a combined intervention is justified.

## Governing semantic authority

The relevant normative contracts establish:

1. `Integer` is one exact, mathematically unbounded semantic family.
2. Internal SmallInteger, BigInteger, tagged or machine-word representation
   categories are not observable semantic families.
3. Standard arithmetic/ordering/equality/hash behavior must remain exact.
4. Ordinary message lookup, override/shadowing, receiver/methodHome provenance
   and evaluation order remain authoritative.
5. Implementations may use unboxing, JIT specialization and
   arbitrary-precision specialization when observable semantics are unchanged.
6. A bounded machine-width Integer carrier may promote transparently to
   arbitrary precision.
7. A semantic Closure/method invocation may be physically optimized or
   materialized lazily only when every observable invocation property remains
   equivalent.

The informative abstract runtime explicitly states that exact Integer arithmetic
may specialize common operations using native machine widths and promote
transparently on overflow.

This means the current physical representation is an implementation choice, not
a semantic obligation.

## Current Integer representation

Current runtime representation remains:

```text
semantic Integer
  -> ProtosIntegerValue
     -> java.math.BigInteger
```

`ProtosIntegerValue` owns one `BigInteger value` field. Current Integer-family
membership used by guarded represented lookup is:

```java
receiver instanceof ProtosIntegerValue
```

There is no current guest SmallInteger / machine-word / `long` carrier.

Integer literals are materialized during lowering and emitted as Bytecode
constants. Therefore the investigation specifically rejects the false hypothesis
that an integer literal such as `1` is recreated on every loop iteration.

```text
LITERAL_CONSTANT_RECREATION_PER_OPERATION=NO
```

## Current guarded Integer send path

PERF016 established guarded represented selection for semantic Integer receivers.

For a stable admitted standard native send such as:

```protos
count - 1
count > 0
```

the selection can be cached by semantic Integer family, selector, entered
Truffle Context, Prelude and lookup-stability Assumption.

A valid hit therefore avoids repeated generic represented-value selection.

However the selected operation still enters the unchanged immediate method call
path:

```text
PrepareSendArguments.guardedIntegerSend
  -> prepareImmediateMethodCall
  -> ProtosActivation.forImmediateMethodInvocation
  -> finishPreparingComposedCallByImplementation
  -> PreparedClosureCall.NativeCall
  -> EnterClosureCall.nativeCall
  -> nativeBody.execute(activation, supplied)
```

PERF016's retained implementation record explicitly states that no arithmetic or
comparison operation is implemented at the send site.

Therefore:

```text
GUARDED_INTEGER_SELECTION=YES
GUARDED_INTEGER_EXECUTION_REPRESENTATION=NO
```

## Rich native invocation state

`prepareImmediateMethodCall(...)` calls
`ProtosActivation.forImmediateMethodInvocation(...)`.

At the Java source/representation level the common path establishes or
constructs state including:

```text
ProtosReturnHome
ProtosExecutionContextValue
ProtosMapBackedLexicalBindingAuthority
  -> LinkedHashMap
  -> ArrayList binding order
ProtosArrayValue for caller-supplied arguments
  -> ArrayList element storage
ProtosActivation
PreparedClosureCall.NativeCall
```

The exact surviving heap allocation count is **not** established by static
source inspection. Graal may inline, escape-analyze or scalar-replace some of
these objects.

The established claim is narrower:

> The compiled optimizer must currently reason through a rich physical
> invocation representation before the tiny native numeric body is reached.

## Existing deferred activation machinery

PLAT040/I072 already introduced a compact/lazy source-method ABI.

The existing runtime contains:

```text
ProtosActivation.forImmediateMethodInvocationWithReturnHomeForRuntime(...)
```

which can create the semantic invocation carrier while leaving:

```text
guest execution Context
guest supplied-argument Array
```

unmaterialized until an observer actually requests them.

`ProtosFrameArguments.compactImmediateMethodCall(...)` similarly carries a
flat internal ABI and materializes richer activation state only after target
entry when needed.

Therefore Protos architecture already proves:

```text
SEMANTIC_FRESH_INVOCATION
  !=
MANDATORY_EAGER_GUEST_CONTEXT_AND_ARGUMENT_ARRAY
```

Current guarded standard Integer native sends do not reuse that deferred path.

## BigInteger operation path

Standard exact Integer arithmetic extracts the `BigInteger` payloads and uses
operations such as:

```text
add
subtract
multiply
divide
remainder
compareTo
```

For Integer-producing operations the Java implementation returns a new
`ProtosIntegerValue` around the exact result.

The correct operation-specific classification is:

| standard operation family | rich native invocation | BigInteger-backed operands | Java-level Integer result wrapper |
| --- | --- | --- | --- |
| `+`, binary `-`, `*` | yes | yes | yes |
| `div`, `mod` | yes | yes | yes |
| Integer `/` | yes | yes / exact rational conversion path | no; Float result |
| `< <= > >=` | yes | yes for Integer/Integer | no; canonical Boolean |
| numeric `==` | yes | yes for Integer/Integer | no; canonical Boolean |

This investigation does **not** claim one fresh heap `BigInteger` allocation for
every operation. Java `BigInteger` implementation details and Graal allocation
elimination require dynamic evidence.

The established representation fact is:

```text
SMALL_MACHINE_WIDTH_INTEGER_ARITHMETIC_USES_BIGINTEGER_API=YES
```

## Source-backed multiplication of Integer call cost

Two standard Integer operations are especially important because their Core
source definitions compose additional messages.

### Unary negation

Core defines:

```protos
negated: () => {
    0 - this
}
```

Prefix `-x` lowers to `x.negated()`.

Therefore the physical shape includes an outer source-backed Closure invocation
plus the inner binary Integer `-` send.

### Percent

Core defines the percent helper as:

```protos
_coreIntegerPercent: argument => {
    (0 + this).mod(argument)
}
```

Therefore `a % b` can include:

```text
outer source-backed % invocation
  -> Integer + send
     -> Integer intermediate result representation
  -> Integer mod send
     -> Integer result representation
```

PERF016's guarded Integer specialization deliberately admits a selected native
Closure only. The source-backed outer `negated` and `%` behaviors therefore do
not use the same guarded Integer-family native path as the underlying canonical
native operations.

## I075 reconciliation

I075 enabled:

```text
boxingEliminationTypes={int.class}
```

but its chosen `int` carrier is implementation-internal metadata:

```text
indices
counts
lexical depths
frame ordinals
```

It did not introduce a guest Integer primitive carrier.

I075 explicitly recorded that an Integer/`long` fast path with transparent
promotion would require new numeric-representation architecture.

Therefore:

```text
I075_COMPLETE=YES
GUEST_INTEGER_PRIMITIVE_CARRIER_IMPLEMENTED=NO
PERF027_REOPENS_I075=NO
```

Simply adding `long.class` to `boxingEliminationTypes` would not solve the
problem because current guest Integer operations do not produce/consume a
primitive `long` flow.

## PERF016 reconciliation

PERF016's structural result remains valid:

```text
INTEGER_REPRESENTED_SELECTION=COMPLETE
INTEGER_VALID_HIT_GENERIC_SELECTION=NO
```

Its post-Step-3 controlled timing result classified the combined Boolean +
Integer represented-selection intervention as:

```text
STEP_3_TIMING_CLASS=ESSENTIALLY_UNCHANGED
```

This does not prove that activation or BigInteger representation is the dominant
remaining cost. It does establish that removing repeated generic selection did
not move the common workloads into a materially better timing class.

PERF027 therefore begins **after** selection.

## PERF011 reconciliation

PERF011 remains closed.

Its final audit explicitly retained the following independent future candidates:

```text
residual activation scalarization/deferred materialization
broader primitive carriers beyond existing int metadata
```

PERF027 is the Integer-specific activation of those retained candidates after
PERF025's current pay-as-you-grow audit made the surface actionable.

## Causal decomposition

The two costs must not be changed simultaneously in the first experiment.

### A — invocation representation

Keep:

```text
ProtosIntegerValue(BigInteger)
BigInteger arithmetic
existing guarded Integer selection
```

and change only eager invocation materialization.

The smallest current candidate is to reuse existing PLAT040 deferred activation
machinery for the already-admitted guarded standard Integer native common path.

This can test whether avoiding eager guest Context / guest arguments Array
materialization is material without contaminating the result with a new numeric
representation.

### B — Integer representation

Keep the same invocation path and introduce a hidden bounded small representation
with transparent arbitrary-precision promotion.

A low-contamination candidate is a dual physical representation contained behind
the semantic Integer abstraction rather than exposing host `Long` throughout
the runtime.

This experiment is **not yet authorized for production** by this record.

### C — eventual primitive Bytecode carrier

A later, higher-ceiling architecture could keep a primitive `long` across
Bytecode DSL stack/locals/operations and materialize/promote only at observation
or overflow boundaries.

Only at that stage would `boxingEliminationTypes` including `long.class`
become directly relevant.

That is not the first experiment.

## Selected next slice

```text
NEXT_SLICE=PERF027-A
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos
```

PERF027-A must isolate invocation representation first.

Target:

> Reuse the existing PLAT040 deferred immediate-method activation machinery for
> the already-admitted guarded standard Integer native-send common path, while
> leaving BigInteger and ProtosIntegerValue representation completely unchanged.

Required invariants:

```text
INTEGER_REPRESENTATION_CHANGED=NO
BIGINTEGER_ARITHMETIC_CHANGED=NO

GUARDED_INTEGER_SELECTION_PRESERVED=YES
SELECTED_NATIVE_CLOSURE_PRESERVED=YES
SELECTED_METHOD_HOME_PRESERVED=YES

SUCCESSFUL_COMMON_PATH_EAGER_GUEST_CONTEXT=NO
SUCCESSFUL_COMMON_PATH_EAGER_GUEST_ARGUMENT_ARRAY=NO

ON_DEMAND_MATERIALIZATION_SEMANTICALLY_IDENTICAL=YES
ERROR_PATH_SEMANTICS=UNCHANGED
RETURN_HOME_NLR_SEMANTICS=UNCHANGED
TASK_DYNAMIC_CONTROL_SEMANTICS=UNCHANGED
GENERIC_FALLBACK=PASS
OBSERVABLE_SEMANTIC_CHANGE=NO
```

If the implementation cannot be expressed as bounded reuse/extension of the
already-ratified PLAT040 machinery and instead requires a new durable platform
architecture decision, PERF027-A must stop before publication and route that
exact question through PLAT.

## Measurement policy after A

Do not introduce a new broad benchmark methodology.

Reuse current exact-revision/radar infrastructure and include evidence that can
distinguish:

```text
Integer comparison:
  invocation + BigInteger comparison + Boolean result

Integer-producing arithmetic:
  invocation + BigInteger arithmetic + Integer result representation
```

Dynamic claims about allocation counts, scalar replacement or compiled shape
require dynamic evidence.

## Investigation result

```text
PERF027_STATIC_INVESTIGATION=COMPLETE

CURRENT_SMALL_INTEGER_FAST_REPRESENTATION=ABSENT
FULL_SEMANTIC_MESSAGE_DISPATCH_REQUIRED=YES
FULL_CURRENT_PHYSICAL_INVOCATION_SHAPE_REQUIRED=NO

CURRENT_INTEGER_NATIVE_FAST_SELECTION=YES
CURRENT_INTEGER_NATIVE_FAST_EXECUTION=NO

EAGER_RICH_NATIVE_ACTIVATION=ESTABLISHED_AT_SOURCE_REPRESENTATION_LEVEL

BIGINTEGER_FOR_CURRENT_INTEGER_REPRESENTATION=ESTABLISHED
FRESH_INTEGER_WRAPPER_FOR_INTEGER_PRODUCING_ARITHMETIC=
  ESTABLISHED_AT_JAVA_IMPLEMENTATION_LEVEL

BIGINTEGER_RESULT_HEAP_ALLOCATION_EVERY_OPERATION=NOT_CLAIMED
REAL_HEAP_ALLOCATION_COUNT=NOT_MEASURED
INVOCATION_TAX_ATTRIBUTABLE_TIME=NOT_MEASURED
INTEGER_REPRESENTATION_TAX_ATTRIBUTABLE_TIME=NOT_MEASURED
DOMINANT_CAUSE=NOT_ESTABLISHED

SEMANTIC_CHANGE_REQUIRED_FOR_HIDDEN_MACHINE_WIDTH_SPECIALIZATION=NO
NEW_PRODUCTION_INTEGER_REPRESENTATION_ARCHITECTURE_REQUIRED=YES
LONG_BOXING_ELIMINATION_ALONE_SUFFICIENT=NO

NEXT_SLICE=PERF027-A
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
  AGENTS.work/REFERENCE.md
  spec/semantics/VALUES_AND_COLLECTIONS.md
  spec/semantics/CALLABLES.md
  spec/semantics/EXECUTION_AND_CONTROL.md
  spec/runtime/ABSTRACT_RUNTIME.md
  docs/design/STANDARD_LIBRARY_IDEAS.md
  protos/lib/core/Integer.protos
  src/main/java/com/guillermomolina/protos/runtime/ProtosIntegerValue.java
  src/main/java/com/guillermomolina/protos/runtime/ProtosIdentity.java
  src/main/java/com/guillermomolina/protos/runtime/ProtosValueLookup.java
  src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java
  src/main/java/com/guillermomolina/protos/runtime/ProtosArrayValue.java
  src/main/java/com/guillermomolina/protos/runtime/ProtosObjectValue.java
  src/main/java/com/guillermomolina/protos/runtime/ProtosExecutionContextValue.java
  src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
  src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java
  src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
  src/main/java/com/guillermomolina/protos/execution/ProtosStandardIntegerProtocol.java
  src/main/java/com/guillermomolina/protos/execution/ProtosStandardNumberEqualityProtocol.java
  src/main/java/com/guillermomolina/protos/execution/ProtosStandardNumberOrderingProtocol.java
  src/main/java/com/guillermomolina/protos/execution/ProtosStandardHashSupport.java

guillermomolina/protos-project-docs:
  docs/project/evidence/PERF025/PERF025_POST_F1_PAY_AS_YOU_GROW_RUNTIME_COST_AUDIT.md
  docs/project/evidence/PERF016/PERF016_INTEGER_GUARDED_SELECTION_IMPLEMENTATION.md
  docs/project/evidence/PERF016/PERF016_POST_STEP3_CONTROLLED_TIMING_RESULT.md
  docs/project/evidence/PERF011/PERF011_FINAL_RUNTIME_REPRESENTATION_FIT_AUDIT.md
  docs/project/evidence/I075/I075-D_INT_BOXING_ELIMINATION_CURRENT_NODE_AUTHORITY.md
  docs/project/work/AUD016/AUD016_TRUFFLE_BYTECODE_DSL_CAPABILITY_ADOPTION_AUDIT.md
```

No product build, test, benchmark or runtime command was executed as part of
this static investigation.
