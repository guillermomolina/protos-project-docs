# PERF010-A — Remaining common-path causal candidate after Ablations 1–4

Status: CANDIDATE ESTABLISHED — MEASUREMENT NOT YET JUSTIFIED

This durable, non-normative record closes the investigation-only checkpoint after
PERF010-A Ablations 1–4. It does not authorize a fifth ablation and does not
select a production optimization.

## A. Baseline verification

```text
PROTOS_REVISION=a116176abd69ddf85e8ee52b623e90d48ef3a11e
BENCHMARK_REVISION=b829ba00a22dd06e06aaf91faf3411c8e65f4511
PROJECT_EVIDENCE_REVISION=595dfbf51b9423276b83e61b57fcd6644004cf9f
PROJECT_REPOSITORY_HEAD_INSPECTED=7bb6b33d37da07df466ffd664bafc83c7b23898d
```

The Protos revision used by Ablation 4 was
`4c4aa95a5852119bd280ceb40483871d5d2cbb82`. Current Protos is one commit
ahead. The intervening commit changes Package Tool/CLI/tests, `pom.xml`, and
`CHANGELOG.md`; it does not change the PERF010 execution/runtime classes or the
four workloads. The project-docs head is one commit ahead of the retained
PERF010-A evidence pin and adds only TOOL009 evidence. No relevant source or
evidence drift was found.

The task requested `AGENTS.work/REPRODUCIBILITY.md`; current Protos
`AGENTS.work/` contains no file with that name. `AGENTS.md`,
`AGENTS.work/PERFORMANCE.md`, and the applicable coordination/documentation
instructions were inspected.

## B. Remaining common path

After excluding the mechanisms already measured by Ablations 1, 3, and 4, the
relevant hot call path still contains:

```text
semantic/helper Bytecode execution
  -> call/send preparation
  -> closure or method selection
  -> ProtosActivation.forClosureInvocation(...)
     or ProtosActivation.forImmediateMethodInvocation(...)
       -> fresh execution context
       -> captured lexical contexts
       -> frozen argument Array
       -> return-home ownership
       -> ProtosActivation construction
  -> PreparedClosureCall
  -> body CallTarget
```

The retained Ablation 1 record identifies the closure/method activation root as
the repeatedly executed root for the hot values used by all four workloads
(`repeat`, `operation`, `identity`, `receiver.identity`,
`receiver.run`). Current source maps those invocation paths through the two
activation factories above.

## C. Profile-backed candidate search

### 1. LoadLocal / ClearLocal

```text
COMMON_PATH_EVIDENCE=YES
EXACT_OPERATION_IDENTIFIED=NO
SEMANTIC_ISOLATION=NO
REASON_REJECTED_OR_RETAINED=REJECTED
```

The retained profiles make local transport/lifetime activity a serious common
region, but current source does not expose one removable hand-written operation
corresponding to the sampled LoadLocal/ClearLocal activity. A diagnostic would
currently conflate operand transport, temporary lifetime, lowering shape, and
continuation/resume behavior.

### 2. PrepareSendArguments / argument-list materialization

```text
COMMON_PATH_EVIDENCE=YES
EXACT_OPERATION_IDENTIFIED=PARTIAL
SEMANTIC_ISOLATION=NO
REASON_REJECTED_OR_RETAINED=REJECTED
```

`PrepareSendArguments.perform` materializes `List.of(supplied)`; downstream
`PreparedClosureCall` stores `List.copyOf(supplied)`, while activation
creation also materializes a frozen Protos Array for observable arguments.
Removing or sharing one of these representations without a stronger ownership
proof can change snapshot, aliasing, mutability, or lifetime properties. The
whole region is therefore not an admissible single causal component.

### 3. General activation construction/allocation

```text
COMMON_PATH_EVIDENCE=YES
EXACT_OPERATION_IDENTIFIED=NO
SEMANTIC_ISOLATION=NO
REASON_REJECTED_OR_RETAINED=REJECTED
```

Fresh execution context, argument snapshot, return-home ownership, activation
identity, method home, receiver, dynamic-control state, and captured lexical
state are intentionally co-created. "Activation overhead" remains too broad.

### 4. Invocation-time re-projection of captured lexical contexts

```text
COMMON_PATH_EVIDENCE=YES
EXACT_OPERATION_IDENTIFIED=YES
SEMANTIC_ISOLATION=YES
REASON_REJECTED_OR_RETAINED=RETAINED_AS_THE_SINGLE_CANDIDATE
```

`ProtosClosureValue` already stores its `capturedLexicalContexts` as a final
`List.copyOf(...)` and returns that list directly. Both ordinary closure
invocation and immediate method invocation pass that already-frozen list into a
new `ProtosActivation`, whose private constructor performs a second
`List.copyOf(...)`.

This is the first remaining activation-copy operation that can be isolated from
fresh context creation, arguments, return-home ownership, receiver/method
binding, and activation identity.

## D. Selected candidate

```text
CANDIDATE_COMPONENT=invocation-time second List.copyOf of closure capturedLexicalContexts
CANDIDATE_FILE=src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java
CANDIDATE_METHOD=forClosureInvocation / forImmediateMethodInvocation -> private ProtosActivation constructor

CURRENT_OPERATION=
  closure.capturedLexicalContexts()
    -> private ProtosActivation constructor
       -> List.copyOf(capturedLexicalContexts)

DIAGNOSTIC_OPERATION=
  closure.capturedLexicalContexts()
    -> invocation-only trusted private construction path
       -> Objects.requireNonNull(capturedLexicalContexts)
       -> direct field retention
```

The generic/public activation construction paths must retain their existing
defensive copy. The diagnostic is limited to the two invocation factories whose
input is directly the already-frozen list owned by `ProtosClosureValue`.

## E. Admission gate

```text
COMMON_PATH=YES
EXACT_OPERATION=YES
SEMANTIC_EQUIVALENCE_BY_CONSTRUCTION=YES
FOUR_WORKLOAD_OBSERVABLE_RESULT_PRESERVED=YES
SINGLE_CAUSAL_COMPONENT=YES
STATIC_STRUCTURAL_CONTRACT_POSSIBLE=YES
```

### Semantic equivalence

The closure owns a final unmodifiable list created by `List.copyOf`.
`capturedLexicalContexts()` returns that same list. `ProtosActivation` does
not expose a mutable list API for the field: lookup only iterates it, and
closure-capture composition creates a new list when it needs to prepend the
current context. Retaining the closure-owned immutable list therefore preserves
the same elements, ordering, lexical precedence, shadowing, receiver fallback,
and missing-name behavior.

The diagnostic must not alter the fresh execution context, the frozen Protos
argument Array, return-home ownership/freshness, receiver, method home, actor
state, execution domain, task/dynamic-control inheritance, source/debugger
identity, continuations, errors, or native/source closure distinction.

### Four-workload observable result

- `micro/slot-read`: repeated `repeat`/callback closure activations still see
  the same captured module/outer contexts in the same order.
- `micro/closure-call`: the called closure receives the same captured contexts;
  only the redundant Java List re-projection is absent.
- `micro/method-call`: immediate method activation preserves receiver,
  method-home, arguments, return home, and captured lexical ordering.
- `runtime/monomorphic-dispatch`: repeated receiver method activations preserve
  the same receiver/method binding and lexical capture; dispatch selection is
  upstream and unchanged.

## F. Scope

```text
INCLUDED_OPERATIONS=
  second List.copyOf(capturedLexicalContexts) performed when constructing an
  activation from ProtosClosureValue.capturedLexicalContexts()

INCLUDED_CALL_SITES=
  ProtosActivation.forClosureInvocation(...)
  ProtosActivation.forImmediateMethodInvocation(...)

EXCLUDED_OPERATIONS=
  ProtosClosureValue's original List.copyOf at closure construction
  fresh ProtosObjectValue execution-context creation
  ProtosPrelude.newFrozenArray(supplied)
  return-home creation/ownership
  argument/list materialization in PrepareSendArguments and PreparedClosureCall
  activation lookup
  dynamic-control/task inheritance

EXCLUDED_CALL_SITES=
  every public/general ProtosActivation constructor/factory not directly fed by
  ProtosClosureValue.capturedLexicalContexts()

UNCHANGED_PATHS=
  lookup order and receiver/delegation fallback
  semantic/helper Bytecode topology
  CallTarget topology
  continuation/resume behavior
  RootTag and source/debugger identity
  closure/method selection and binding
```

## G. Future static contract

A future validator can fail closed before smoke/reference by proving all of the
following:

1. The only changed production file is `ProtosActivation.java`.
2. `ProtosClosureValue.java` is byte-for-byte unchanged, including the original
   closure-construction `List.copyOf`.
3. The existing defensive-copy constructor path remains unchanged for every
   non-invocation activation factory.
4. Exactly `forClosureInvocation` and `forImmediateMethodInvocation` use one
   dedicated private invocation-only path that directly retains the list
   returned by `closure.capturedLexicalContexts()`.
5. No other call site can enter the trusted path.
6. The trusted path performs `Objects.requireNonNull` and changes only the
   captured-list field assignment relative to the ordinary constructor field
   initialization.
7. Fresh context creation, frozen arguments, receiver, method home, return home,
   ownership flags, actor/module state, execution domain, and subsequent
   task/dynamic-control attachment are textually/structurally unchanged.
8. No change occurs in Bytecode lowering, `ProtosBytecodeRootNode`,
   `ProtosValueLookup`, closure selection, RootTag/tooling, continuation, or
   debugger/source machinery.
9. Scope count is exactly two admitted factory call sites and one invocation-only
   field-assignment variant; any third caller, global constructor relaxation, or
   neighboring allocation change fails validation.

## H. Relationship to prior ablations

```text
RELATION_TO_ABLATION_1=OUTSIDE
RELATION_TO_ABLATION_3=OUTSIDE
RELATION_TO_ABLATION_4=OUTSIDE
```

Ablation 1 measures semantic/helper Bytecode dispatch. Ablation 3 measures the
two lexical local-slot probes inside `ProtosActivation.lookup`. Ablation 4
measures duplicate `nativeBody()` Optional projection during call preparation.
The selected candidate is an activation-construction list projection and changes
none of those mechanisms. Their percentages must not be added or subtracted.

## I. Measurement viability

```text
CURRENT_PAIRED_CONTROL_STABILITY=INSUFFICIENT
CANDIDATE_MEASUREMENT_VALUE=PLAUSIBLE
```

The candidate has plausible information value because the operation is reached
once per hot closure/method activation in all four workloads, but retained
evidence does not quantify its standalone cost.

More importantly, the current causal harness is not sufficiently discriminating
for another expected small local effect. Its matrix executes, for each workload,
in the fixed order:

```text
baseline canonical
baseline control
ablation canonical
ablation control
```

The three valid ablations already show paired-control movement on the same scale
as the local effects being sought:

- Ablation 1 control movement ranges from about 0.18 ms to 2.03 ms.
- Ablation 3 control movement ranges from about 0.62 ms to 1.40 ms.
- Ablation 4 control movement ranges from about 1.11 ms to 2.91 ms, with sign
  reversals across workloads.

Ablation 4's control movement is roughly 2–6% of the corresponding control
medians. This is large enough to dominate or reverse the interpretation of a
small source-level transformation. The classification is therefore
`INSUFFICIENT` specifically for another small-effect local ablation; it is not
a claim that the harness cannot detect a genuinely large effect.

### Minimum next investigation

Before any Ablation 5, perform one bounded measurement-discrimination
investigation in `guillermomolina/protos-benchmarks`:

1. preserve the same four workloads, correctness checks, warmup/steady scale,
   CPU pinning, and canonical/control definitions;
2. compare two source-equivalent/no-op variants rather than changing Protos
   semantics or runtime code;
3. execute variants in a counterbalanced/interleaved order (for example ABBA
   blocks, with deterministic repetition) instead of always baseline-first;
4. retain per-block canonical and control deltas and derive a per-workload
   no-op paired-control envelope/discrimination floor;
5. declare in advance what effect magnitude must exceed that envelope before a
   small causal ablation is worth a full reference run.

Do not implement the captured-context candidate until that methodological
checkpoint demonstrates enough discrimination for its expected scale.

## Final result

```text
PERF010A_NEXT_CAUSAL_CANDIDATE=ESTABLISHED
CANDIDATE_COMPONENT=invocation-time second List.copyOf of closure capturedLexicalContexts

COMMON_PATH=YES
EXACT_OPERATION=YES
SEMANTIC_EQUIVALENCE_BY_CONSTRUCTION=YES
FOUR_WORKLOAD_OBSERVABLE_RESULT_PRESERVED=YES
SINGLE_CAUSAL_COMPONENT=YES
STATIC_STRUCTURAL_CONTRACT_POSSIBLE=YES

CURRENT_PAIRED_CONTROL_STABILITY=INSUFFICIENT
CANDIDATE_MEASUREMENT_VALUE=PLAUSIBLE

RELATION_TO_ABLATION_1=OUTSIDE
RELATION_TO_ABLATION_3=OUTSIDE
RELATION_TO_ABLATION_4=OUTSIDE

CAUSAL_COST=NOT_YET_MEASURED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO

FUTURE_ABLATION_EXACT_TRANSFORMATION=
  retain ProtosClosureValue's already-frozen capturedLexicalContexts directly
  only in closure/method invocation activation construction, leaving all generic
  activation defensive-copy paths unchanged

FUTURE_ABLATION_STATIC_CONTRACT=
  one changed production file; exactly two admitted invocation factories; the
  closure's original copy and every non-invocation activation defensive copy
  remain unchanged; all neighboring activation/call semantics unchanged

FUTURE_ABLATION_MEASUREMENT_JUSTIFIED=NO
```

A fifth ablation is therefore **not authorized by this checkpoint yet**. The
candidate is established, but the current measurement design must first
demonstrate a discrimination floor below the expected scale of this local
operation.
