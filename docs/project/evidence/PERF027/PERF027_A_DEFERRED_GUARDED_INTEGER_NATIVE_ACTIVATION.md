# PERF027-A — Deferred activation for guarded Integer native sends

Status: **PUBLISHED / VALIDATED**  
Date: 2026-10-02  
Formal owner: `PERF027 / guillermomolina/protos#779`

This durable, non-normative record retains the published PERF027-A
implementation and the maintainer-reported validation result. It does not claim
a measured performance effect.

## Exact product identity

```text
PROTOS_REPOSITORY=guillermomolina/protos
BASE_REVISION=c8e0e0d59d5541007d733e8a0b53a3cf123e5818
BASE_VERSION=0.3.147-SNAPSHOT

PERF027_A_REVISION=d1aeea403cc7f7e5ffa7289006ceb072b025ca44
PERF027_A_VERSION=0.3.148-SNAPSHOT
COMMIT_SUBJECT=PERF027-A: deferred activation for guarded Integer native sends
```

Published changed paths:

```text
M CHANGELOG.md
M pom.xml
M src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
A src/test/java/com/guillermomolina/protos/execution/ProtosPerf027AGuardedIntegerDeferredActivationTest.java
```

No specification file changed.

## Implemented physical change

PERF016's guarded Integer selection remains the admission mechanism.

Before PERF027-A an admitted standard Integer native send continued through:

```text
guardedIntegerSend
  -> prepareImmediateMethodCall
  -> ProtosActivation.forImmediateMethodInvocation
  -> eager guest invocation state
  -> selected native Closure
```

PERF027-A changes only that admitted Integer native-send preparation path:

```text
guardedIntegerSend
  -> prepareDeferredImmediateNativeMethodCall
  -> existing immediate-method preparation
     with deferGuestInvocationState=true
  -> ProtosActivation.forImmediateMethodInvocationWithReturnHomeForRuntime
  -> selected native Closure
```

The ordinary immediate-method path remains eager. The optimization is therefore
bounded to the PERF016 guarded Integer native-send common path rather than a
global native-call architecture rewrite.

## PLAT040 reuse

The implementation reuses the already-existing PLAT040/I072 deferred activation
factory:

```text
ProtosActivation.forImmediateMethodInvocationWithReturnHomeForRuntime(...)
```

It does not introduce a second lazy activation model.

The fresh invocation ReturnHome is still established before native method entry:

```text
closure.returnHome().orElseGet(ProtosReturnHome::new)
```

The semantic activation therefore still exists, while the guest execution
Context and supplied guest Array can remain deferred until observed.

## Preserved Integer representation

PERF027-A deliberately does not touch the second PERF027 causal surface.

```text
INTEGER_REPRESENTATION=ProtosIntegerValue(BigInteger)
INTEGER_REPRESENTATION_CHANGED=NO
BIGINTEGER_ARITHMETIC_CHANGED=NO
BOXING_ELIMINATION_TYPES_CHANGED=NO
```

No `long`, SmallInteger, tagged Integer, dual `long|BigInteger`, or
machine-width arithmetic fast path was introduced.

This keeps the eventual PERF027 representation experiment causally separate from
the invocation-state intervention.

## Regression coverage

The new
`ProtosPerf027AGuardedIntegerDeferredActivationTest`
retains focused evidence for:

- exact guarded standard Integer selection;
- exact selected native Closure and methodHome;
- receiver and Prelude identity;
- fresh owned ReturnHome and completion;
- guest Context remaining unmaterialized on successful ordinary arithmetic;
- supplied guest Array remaining unmaterialized on successful ordinary arithmetic;
- on-demand Context/argument materialization remaining stable and observable;
- exact values beyond signed-`long` range;
- overflow-sensitive `Long.MAX_VALUE + 1`;
- multiplication, `div`, `mod`, and Integer comparison;
- division/modulo-by-zero Error behavior;
- wrong-domain Error behavior; and
- independent activation/Prelude state across separate Truffle Contexts.

The product changelog also records that ordinary immediate-method callers retain
the eager path.

## Validation provenance

The maintainer reported after publication:

```text
pushed and tested PERF027-A: deferred activation for guarded Integer native sends
```

Accordingly:

```text
MAINTAINER_REPORTED_VALIDATION=PASS
PUBLICATION=PASS
```

No additional build, test, benchmark, or runtime command was executed by the
coordinating agent when retaining this record.

The exact command-level output was not supplied, so this record does not invent
individual test-run identities beyond the published regression source and the
maintainer's PASS report.

## Acceptance result

```text
PERF027_A_STATUS=PASS

INTEGER_REPRESENTATION_CHANGED=NO
BIGINTEGER_ARITHMETIC_CHANGED=NO
BOXING_ELIMINATION_TYPES_CHANGED=NO

GUARDED_INTEGER_SELECTION_PRESERVED=YES
SELECTED_NATIVE_CLOSURE_PRESERVED=YES
SELECTED_METHOD_HOME_PRESERVED=YES

DEFERRED_NATIVE_ACTIVATION_REUSED=YES
SUCCESSFUL_COMMON_PATH_EAGER_GUEST_CONTEXT=NO
SUCCESSFUL_COMMON_PATH_EAGER_GUEST_ARGUMENT_ARRAY=NO

ON_DEMAND_MATERIALIZATION_PRESERVED=YES
ERROR_PATH_SEMANTICS=PASS
RETURN_HOME_SEMANTICS=PASS
GENERIC_FALLBACK=PRESERVED_BY_BOUNDED_ADMISSION
MULTI_CONTEXT_ISOLATION=PASS

OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO

PUBLICATION=PASS
MAINTAINER_REPORTED_VALIDATION=PASS
PERFORMANCE_EFFECT_MEASURED=NO
```

## Causal interpretation

PERF027-A is an implementation intervention, not yet a performance attribution.

It removes eager guest Context / guest argument-Array materialization from the
admitted guarded Integer native-send path while retaining BigInteger-backed
Integer representation.

Therefore the next causal step should measure the exact-revision effect before
changing Integer representation.

```text
NEXT_SLICE=PERF027-B
NEXT_SLICE_TYPE=MEASUREMENT
PRODUCT_CHANGE=NO

CONTROL_REVISION=c8e0e0d59d5541007d733e8a0b53a3cf123e5818
INTERVENTION_REVISION=d1aeea403cc7f7e5ffa7289006ceb072b025ca44

GOAL=
  measure the invocation-tax intervention using existing exact-revision
  Integer/primitive radar without introducing a new benchmark methodology

INTEGER_REPRESENTATION_EXPERIMENT_BEFORE_B=NO
```

Only after PERF027-B establishes the A intervention's effect should PERF027
authorize a representation-only experiment over small Integers.
