# PERF010-A / PERF011 — caller/helper permanent-bailout root cause

Status: retained source-causal investigation; the permanent bailout mechanism is explained strongly enough to admit one bounded implementation experiment, but dominant attributable runtime cost is not yet established and no production optimization is selected.

## Evidence identity

```text
PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=2b3a88389da7228caed231a90b14091cf2841115
PROTOS_VERSION=0.3.77-SNAPSHOT

PRIOR_PROJECT_RECORD_REVISION=0c7ba299debe02080f239e39b122d566f5726cb9
PRIOR_PROJECT_RECORD=docs/project/evidence/PERF010-A/PERF010-A_GUARDED_CALLER_HELPER_COMPILER_LIFECYCLE_DISCRIMINATION.md

RUNTIME=GraalVM Community 25.3.4.1 / JDK 25.0.4.1 / Truffle 25.3.4.1
WORKLOAD=micro/method-call
SOURCE=method-call.protos
START_OFFSET=1226
END_OFFSET=1254
LINE=29
COLUMN_ONE_BASED=23
TEXT=sink = receiver.identity(42)
```

This investigation is static/source-causal. It did not rerun builds, tests, benchmarks or compiler captures. It consumes the already-retained BASELINE and GUARDED compiler-lifecycle evidence and the exact current source at the Protos revision above.

## Established lifecycle premise

The prior retained evidence establishes for both BASELINE and GUARDED:

```text
CALLER_COMPILER_TOPOLOGY=C

semantic root:
  Tier 1 compilation completes
  Tier 2 compilation completes
  helper is not inlined

helper:
  Tier 1 Count/Thres=400/400
  queued
  start
  PERMANENT BAILOUT

REPEATED_SEND_WORK_OUTSIDE_OPTIMIZED_REGION=YES

BAILOUT=
  jdk.graal.compiler.core.common.PermanentBailoutException:
  Too deep inlining, probably caused by recursive inlining.
```

The guarded intervention therefore does not remove compiler stranding. Its previously negative timing remains non-discriminating for the compiler-stranding hypothesis.

## BASELINE source trace

The BASELINE failed partial-evaluation path reaches:

```text
PrepareSendArguments.perform
  -> prepareSend
  -> prepareImmediateMethodCall
  -> ProtosStandardImportProtocol.selectedRuntimeForBytecodeIntrinsic
  -> prepareBytecodeImport
  -> prepareBytecodeImportWithTask
  -> resolveModuleKey
  -> ProtosStandardLibraryModuleResolver.resolve
  -> requireStandardLogicalName
  -> JDK Locale / Formatter / String / generic-reflection machinery
  -> PermanentBailoutException
```

### Ordinary-send transition

`PrepareSendArguments.perform` calls the generic `prepareSend(receiver, selector, caller, supplied)` path. D013 lookup selects a `ProtosClosureValue` and its `methodHome`, after which `prepareImmediateMethodCall` builds the fresh invocation activation.

That lookup and fresh activation work are part of the ordinary send semantics.

### Standard-import classification

Inside `prepareImmediateMethodCall`, every selected behavior is tested with:

```java
ProtosStandardImportProtocol.selectedRuntimeForBytecodeIntrinsic(
    receiver,
    closure,
    methodHome,
    caller.prelude().orElse(null))
```

If non-null, the generic preparation path calls:

```java
standardImportRuntime.prepareBytecodeImport(supplied, caller)
```

For the exact `receiver.identity(42)` call this branch is semantically impossible: the standard-import recognizer can return a runtime only when the selected behavior is the canonical standard import facility's native `StandardImportBody`. The selected `identity` behavior is the ordinary source-backed non-native Closure exercised by this caller.

Therefore:

```text
BASELINE_STANDARD_IMPORT_CLASSIFICATION_PRESENT=YES
STANDARD_IMPORT_BRANCH_REQUIRED_FOR_THIS_CALL=NO
STANDARD_IMPORT_BRANCH_SEMANTICALLY_REACHABLE_FOR_THIS_CALL=NO
```

The generic implementation nevertheless leaves that possibility compiler-visible during partial evaluation.

### Module resolver host trigger

If the generic branch is explored, `ProtosModuleRuntime.prepareBytecodeImportWithTask` calls:

```java
resolveModuleKey(supplied.get(0), caller)
```

which delegates to the configured resolver. For the standard-library resolver:

```java
String logicalName = requireStandardLogicalName(exactSpecifier);
```

`requireStandardLogicalName` validates each path segment and calls:

```java
isWindowsReservedSegment(segment)
```

whose concrete host-bearing expression is:

```java
WINDOWS_RESERVED_SEGMENTS.contains(segment.toUpperCase(Locale.ROOT))
```

This is the earliest concrete BASELINE source expression in the retained failed path that directly introduces the JDK Locale subsystem.

The retained compiler trace then expands through machinery including:

```text
Locale / BaseLocale
ReferencedKeyMap
ConcurrentHashMap
Class.getGenericInterfaces
sun.reflect.generics.*
String.substring / Preconditions
Formatter
```

`Formatter` is not introduced by a Protos `String.format` call on this path. It appears transitively inside the JDK recursion reached from the exposed host operation.

### BASELINE semantic classification

```text
BASELINE_BAILOUT_SOURCE_ROOT=
  generic standard-import classification retained inside ordinary call preparation

BASELINE_FIRST_CONCRETE_HOST_TRIGGER=
  ProtosStandardLibraryModuleResolver.isWindowsReservedSegment:
  segment.toUpperCase(Locale.ROOT)

SEMANTICALLY_REQUIRED_FOR_EVERY_ORDINARY_SEND=NO
SEMANTICALLY_REQUIRED_FOR_THIS_IDENTITY_SEND=NO
REQUIRED_EVERY_HOT_INVOCATION=NO
```

The semantic requirement is only to preserve correct standard-import behavior when that canonical native import behavior is actually selected. Re-running or compiler-expanding the import resolver for a proven non-native ordinary method is an implementation artifact, not a Protos semantic requirement.

## GUARDED source trace

The retained guarded specialization changes the failed PE prefix to:

```text
PrepareSendArguments.performGuardedOrdinaryComposedSend
  -> taskOwnedBytecodePlan
  -> ProtosLanguageContext.bytecodeExecutionPlanForEnteredClosure
  -> ProtosLanguageContext.bytecodeExecutionPlanForDefinition
  -> ConcurrentHashMap.computeIfAbsent
  -> JDK ConcurrentHashMap / reflection / Locale / Formatter machinery
  -> PermanentBailoutException
```

The guarded specialization re-runs authoritative D013 lookup and admits only the exact cached `ProtosClosureValue` / `methodHome` pair with a Closure whose `nativeBody()` is empty. That correctly eliminates the BASELINE standard-import classifier branch for the guarded hit.

### Context-owned plan projection

`taskOwnedBytecodePlan` starts with the Closure's existing execution plan. When an entered language context exists and either:

```text
closure.requiresContextLocalExecutionProjectionForRuntime()
```

or the template plan belongs to a different `ProtosLanguage`, the helper asks the entered context for the Context-owned Bytecode projection:

```java
enteredContext.bytecodeExecutionPlanForEnteredClosure(closure, template)
```

which delegates to:

```java
bytecodeExecutionPlanForDefinition(closure.definition(), template)
```

The exact cache is declared as:

```java
private final ConcurrentMap<
        ProtosClosureExecutionPlan,
        ProtosClosureExecutionPlan>
    sharedBytecodeExecutionPlans = new ConcurrentHashMap<>();
```

and the complete method is materially:

```java
return sharedBytecodeExecutionPlans.computeIfAbsent(
        template,
        ignored -> template.rebuildBytecodeForLanguage(definition, language));
```

### Direct compiler-trace linkage

The retained GUARDED failed trace contains the direct chain:

```text
java.util.concurrent.ConcurrentHashMap.computeIfAbsent
ProtosLanguageContext.bytecodeExecutionPlanForDefinition
ProtosLanguageContext.bytecodeExecutionPlanForEnteredClosure
ProtosBytecodeRootNode.taskOwnedBytecodePlan
PrepareSendArguments.performGuardedOrdinaryComposedSend
```

and, in the same inlined expansion:

```text
ConcurrentHashMap.comparableClassFor
Class.getGenericInterfaces
sun.reflect.generics.*
String.substring / Preconditions
Formatter
Locale
BaseLocale
ReferencedKeyMap
ConcurrentHashMap
```

Therefore `sharedBytecodeExecutionPlans.computeIfAbsent(...)` is a directly established source cause of the GUARDED host/JDK expansion, not merely a neighboring method inferred from the source prefix.

The cache-miss mapping function also contains plan reconstruction:

```text
ProtosClosureExecutionPlan.rebuildBytecodeForLanguage
  -> ProtosBytecodeClosureExecutionPlan.rebuildForLanguage
  -> new ProtosBytecodeClosureExecutionPlan(...)
  -> CanonicalToBytecodeLowerer(...).lowerClosureActivationRoot(...)
```

Thus both cache mechanics and the potential reconstruction are implementation machinery reachable from the hot preparation path.

### GUARDED semantic classification

Protos semantics require a source-backed Closure crossing Context ownership boundaries to execute through a plan owned by the entered `ProtosLanguageContext`. They do not require a `ConcurrentHashMap.computeIfAbsent` implementation, tree-bin handling, generic-interface reflection, or plan reconstruction to be partial-evaluation-visible at every monomorphic call.

```text
GUARDED_BAILOUT_SOURCE_ROOT=
  ProtosLanguageContext.bytecodeExecutionPlanForDefinition:
  sharedBytecodeExecutionPlans.computeIfAbsent(...)

SEMANTIC_REQUIREMENT=
  use the correct Context-owned execution plan

CURRENT_IMPLEMENTATION_MECHANISM=
  ConcurrentHashMap cache plus lazy rebuild

SEMANTICALLY_REQUIRED_AS_IMPLEMENTED=NO
REQUIRED_EVERY_HOT_INVOCATION=NO
```

## BASELINE versus GUARDED

The two variants do not statically converge on one Protos-owned offending operation.

```text
BASELINE:
  impossible-for-this-call standard-import branch
  -> standard-library logical-name validation
  -> Locale-bearing host path

GUARDED:
  Context-local execution-plan projection cache
  -> ConcurrentHashMap.computeIfAbsent
  -> CHM/reflection host path
```

They do converge inside a recurring JDK expansion involving the same broad families:

```text
Locale / Formatter
String bounds / substring
generic signature parsing
Class.getGenericInterfaces
ConcurrentHashMap comparable/tree/resize machinery
ReferencedKeyMap / BaseLocale
```

Accordingly the requested classification is:

```text
BASELINE_VS_GUARDED_PATH_CLASSIFICATION=B
```

where B means that the two variants independently expose two different pathological host paths. Their similar final bailout is not merely superficial, because both traces directly contain the same recursively expanding host families, but there is no single common Protos source root.

## Partial-evaluation boundary

No applicable production `@TruffleBoundary` protects either offending operation.

The failed work is reachable from Truffle Bytecode DSL `@Specialization` methods through ordinary Java helper calls. The `DirectCallNode` / `IndirectCallNode` boundary used for entering the prepared Closure occurs only after `PreparedClosureCall` preparation, so it cannot shield the preparation-time host machinery identified here.

The guarded `@Cached` values stabilize selector, selected Closure and `methodHome`, but the guarded hit still calls `taskOwnedBytecodePlan`; the effective Context-owned target is therefore not cached by the specialization itself, and `bytecodeExecutionPlanForDefinition` remains compiler-visible.

Source alone cannot prove every internal Graal inlining-policy choice, but the retained failed compiler traces directly prove that the named host operations were inlined/expanded on the failed PE path. No further inference about their visibility is necessary.

## Minimal next causal intervention

The next experiment must remove both currently established host expansions from the hot guarded hit before timing is interpreted.

The smallest bounded design is a monomorphic prepared-target specialization for `PrepareSendArguments` that:

1. preserves the exact authoritative D013 lookup on every hit;
2. guards the exact selector, selected Closure identity and `methodHome` identity;
3. guards the entered Context identity needed for Context-owned executable plans;
4. caches/materializes the effective Context-owned Bytecode activation target outside the repeatedly compiled hot preparation body;
5. preserves fresh activation creation, arguments, receiver, `methodHome`, lexical relationships, Task/dynamic-control propagation and all current control semantics;
6. uses the exact existing generic path on any guard miss or unsupported case.

For the admitted non-native ordinary Closure hit, this should make both of the following absent from the compiled hot preparation path:

```text
standardImportRuntime.prepareBytecodeImport(...)
sharedBytecodeExecutionPlans.computeIfAbsent(...)
```

The intervention is not accepted as evidence merely because source code changes. Its admission criterion is compiler lifecycle:

```text
SOURCE=method-call.protos
START_OFFSET=1226
END_OFFSET=1254

CALLER_HELPER_PERMANENT_BAILOUT=
  REMOVED | MATERIALLY_CHANGED
```

Only after that gate passes may paired timing be used to estimate the recoverable contribution of compiler stranding.

## Reconciled result

```text
PERF010A_BAILOUT_ROOT_CAUSE=ESTABLISHED

BASELINE_BAILOUT_SOURCE_ROOT=
  generic standard-import classification retained in ordinary call preparation

BASELINE_FIRST_CONCRETE_HOST_TRIGGER=
  ProtosStandardLibraryModuleResolver.isWindowsReservedSegment:
  segment.toUpperCase(Locale.ROOT)

GUARDED_BAILOUT_SOURCE_ROOT=
  ProtosLanguageContext.bytecodeExecutionPlanForDefinition:
  sharedBytecodeExecutionPlans.computeIfAbsent(...)

COMMON_BAILOUT_SOURCE_ROOT=NONE

COMMON_BAILOUT_MECHANISM=
  different implementation-only host entry paths feed the same recursively
  expanding JDK Locale/Formatter/generic-reflection/ConcurrentHashMap family

COMMON_CAUSE_CONFIDENCE=STRONG

SEMANTICALLY_REQUIRED_AS_CURRENT_HOST_WORK=NO
REQUIRED_EVERY_HOT_INVOCATION=NO

CURRENT_PE_BOUNDARY=
  none around the identified preparation-time host operations

MINIMAL_CAUSAL_INTERVENTION=
  guarded monomorphic prepared Context-owned Bytecode target specialization
  with authoritative D013 lookup and exact generic fallback

INTERVENTION_PRESERVES_SEMANTICS=YES
IMPLEMENTATION_SLICE_READY=YES
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

## PERF011 relationship

This result upgrades one part of PERF011 from source-level suspicion to direct compiler evidence:

```text
CALL_SITE_COMPILER_VISIBILITY_MISMATCH=ESTABLISHED
CONTEXT_PLAN_CACHE_HOST_LEAKAGE_INTO_PE=ESTABLISHED
```

It still does not justify a broad Shape/DynamicObject/Frame migration. The bounded next experiment is the same PERF010-A intervention above. PERF011 should remain aligned with that experiment and use its compiler/lifecycle/timing result before selecting any wider representation change.

## Work-state consequence

PERF010-A remains open because its closure criterion requires attributable cost/dominance evidence, not merely explanation of the bailout.

The next slice is an implementation experiment in `guillermomolina/protos`. It is not yet a selected production optimization. The experiment must first demonstrate that the exact caller helper's permanent bailout is removed or materially changed, preserve semantic behavior through the exact generic fallback, and only then proceed to timing attribution.
