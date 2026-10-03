# PERF025 — Post-lazy-Activation residual consumer audit

## Status

```text
STATUS=COMPLETE
AUDIT_TYPE=STATIC_ONLY
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=bd8617157d9c8cc6e1ce0edc0502b7a361e1e891
PRODUCT_VERSION=0.3.168-SNAPSHOT

CALLBACK_STATIC_FRAME_LOCAL_AUTHORITY=COMPLETE
CALLBACK_LAZY_SEMANTIC_ACTIVATION=COMPLETE
RESIDUAL_ACTIVATION_CONSUMER_SPECIALIZATION=JUSTIFIED

PLAT044_DECISION=B_PRIME_RATIFIED
NEW_PLATFORM_DECISION_REQUIRED=NO
OBSERVABLE_LANGUAGE_SEMANTIC_CHANGE=NO
IMPLEMENTATION_PERFORMED_BY_AUDIT=NO
TESTS_RUN_BY_AUDIT=NO
BENCHMARKS_RUN_BY_AUDIT=NO
```

## Purpose

Close the explicit post-Slice-2 gate left by
`PERF025_LAZY_INLINE_CALLBACK_ACTIVATION.md`.

The question was whether any frequent successful PLAT044 B′ inline-callback
paths still materialize the callback's semantic `ProtosActivation` only because
existing Bytecode operations accept a rich Activation operand, rather than
because the language or tooling actually observes that Activation.

The answer is yes.

## Decision

```text
RESIDUAL_ACTIVATION_CONSUMER_SPECIALIZATION=JUSTIFIED
OPTIONAL_SLICE_3=RELEASED_FOR_IMPLEMENTATION
NEW_PLATFORM_DECISION_REQUIRED=NO
```

Slice 2 made callback entry, fixed-parameter binding, current-local
create/read/write, and the ordinary frame-native lexical path lazy. The remaining
surface is not limited to explicit `context`, debugger scope, non-local return,
Errors, or D179 fallback.

Several common successful callback-body operations still invoke
`CanonicalToBytecodeLowerer.emitCurrentActivation(...)` before entering an
operation whose successful semantics need only a projection already available
from `PreparedInlineLiteralCall` and its compact target arguments.

## High-value residual consumers

### 1. Canonical Integer sends

Ordinary lowering still passes the current callback Activation to
`PrepareSendArguments` / `PrepareSendVector`.

For a guarded canonical Integer hit, the selected operation can finish through
the immediate-result path without observing the callback guest Context or
Activation identity.

This affects shapes already present in B′ while/each callbacks, including:

```text
n < limit
n + 1
visits + 1
acc * 10 + element
```

Classification:

```text
CANONICAL_INTEGER_SEND_ACTIVATION=
  IMPLEMENTATION_DEPENDENCY_NOT_SEMANTIC_OBSERVER
PRIORITY=VERY_HIGH
```

### 2. Statically resolved captured reads and writes

`ReadCapturedFrameLocal`,
`ReadCapturedMaterializedLocal`,
`ResolveCapturedWritableLexicalTarget`,
`ResolveCapturedMaterializedWritableLexicalTarget`,
`AssignCapturedFrameLocal`, and
`AssignCapturedMaterializedLocal`
all start from the rich callback Activation.

On their statically resolved successful path, the semantic information actually
required is the captured lexical environment plus the D179 nearer-binding
checks and selected destination. That environment is owned by the callback
Closure carried by the compact invocation.

The exact D179 ABSENT/dynamic fallback remains Activation-based.

Classification:

```text
STATIC_CAPTURED_SUCCESS_PATH=CARRIER_SPECIALIZABLE
D179_ABSENT_DYNAMIC_FALLBACK=MATERIALIZE
PERF028_A_DESTINATION_BEFORE_RHS=PRESERVE
PRIORITY=HIGH
```

### 3. Direct Closure calls and ordinary guarded source sends

`PrepareClosureCall`, `PrepareClosureCallArguments`,
`PrepareClosureCallVector`, and ordinary guarded source-send preparation still
receive a rich caller Activation before the successful child call is prepared.

The successful path needs the caller projection used for:

- prelude;
- actor/module/domain provenance;
- Task ownership;
- dynamic-control inheritance;
- captured receiver/methodHome where applicable;
- supplied arguments and ReturnHome.

Those values are already available through the callback Closure, compact target
arguments, and the compact caller/task fields.

The implementation must fall back to existing materialization whenever exact
inheritance cannot be reproduced from that existing authority.

Classification:

```text
DIRECT_CLOSURE_CALL_SUCCESS_PATH=CARRIER_SPECIALIZABLE
ORDINARY_GUARDED_SOURCE_SEND_SUCCESS_PATH=CARRIER_SPECIALIZABLE
GENERIC_STRUCTURED_ERROR_FALLBACK=MAY_MATERIALIZE
PRIORITY=HIGH
RISK=MEDIUM
```

### 4. `THIS` intrinsic and member reads

A frame-native B′ callback can evaluate `THIS`; only `CONTEXT` is excluded
by the existing persistent-frame analysis.

Today `LoadIntrinsic(THIS)` materializes only to call
`activation.receiver()`. For a direct Closure invocation that receiver is the
Closure's captured receiver and is therefore available without a rich
Activation.

Member-read success similarly needs receiver/name plus the callback prelude for
lookup/PIC behavior; a rich Activation is principally needed for the exact guest
Error fallback.

Classification:

```text
THIS_SUCCESS_PATH=CARRIER_SPECIALIZABLE
CONTEXT=REAL_OBSERVER_NOT_FRAME_NATIVE
MEMBER_READ_SUCCESS_PATH=CARRIER_SPECIALIZABLE
PRIORITY=MEDIUM
```

### 5. Inline multiple creation

The frame-native inline helper
`ProtosInlineCallbackFrameBindings.multipleCreate(...)` currently calls
`durableActivation(...)` before `observeMultipleCreatePrefix(...)`, so
successful multiple creation materializes unconditionally.

The successful prefix observation needs only the source Array and requested
count. The Activation is needed only to construct the exact guest Error when the
source is invalid/short or later creation fails.

Classification:

```text
MULTIPLE_CREATE_SUCCESS_PATH=CARRIER_SPECIALIZABLE
ERROR_PATH=MATERIALIZE
PRIORITY=MEDIUM
RISK=LOW
```

## Consumers that are already correctly lazy

The audit confirms that these ordinary paths remain Activation-free until an
actual observer/fallback:

```text
fixed supplied-argument reads
fixed parameter establishment
fixed arity upper-bound success
current-local creation
current-local read while PRESENT
PERF028-A current-local destination selection
current-local assignment while PRESENT
identity / not-identity
source sections / RootTag instrumentation by themselves
yield / resume by themselves
```

## Intentional materialization that must remain

Slice 3 must not remove materialization for genuine semantic/tooling observers
or fallback boundaries:

```text
explicit context observation
debugger/tooling scope projection
non-local return / control-transfer boundary
D179 ABSENT or dynamic lexical fallback
arity Errors
duplicate creation / mutation Errors
unsupported representation / lookup failure
an invocation already materialized by an earlier observer
non-frame-native callbacks
PLAT044 B′ fallback / non-admitted calls
```

Suspension alone remains non-observing:

```text
MATERIALIZE_ON_SUSPENSION_ALONE=NO
```

Tooling remains an intentional observer through
`ProtosBytecodeTagTreeNodeExports.getScope` and
`ProtosInlineCallbackFrameBindings.durableActivationForTooling`.

## Minimum Slice 3 scope

The released implementation slice is bounded to residual consumer
specialization against the already-ratified B′ carrier. It does not authorize a
new callback ABI, execution model, or platform decision.

Target consumers, in priority order:

1. statically resolved captured read/write success paths;
2. canonical Integer and ordinary guarded source-send success paths;
3. direct source-Closure call success paths;
4. `THIS` and successful member reads;
5. successful inline multiple creation.

A subcase must retain the current materializing fallback whenever the existing
carrier cannot reproduce the exact semantics without adding another semantic
carrier or widening PLAT044.

## Expected effect

The slice is expected to remove the *first* callback Activation materialization
from common successful bodies such as:

```text
Array(...).each((element) => { acc = acc + element })
(() => { n < limit }).whileTrue() { n = n + 1 }
environment.each((name, value) => { visits = visits + 1 })
callback bodies using THIS/member access
direct source-Closure calls from an otherwise unobserved callback
```

Pure local-only callbacks do not need further work: Slice 2 already leaves them
fully lazy.

## Required preservation

```text
PLAT044_B_PRIME_ADMISSION=UNCHANGED
FRESH_SEMANTIC_ACTIVATION_IDENTITY=UNCHANGED
MATERIALIZATION_AT_MOST_ONCE=UNCHANGED
DURABLE_TRANSFER_BEFORE_FIRST_OBSERVER=UNCHANGED
D179_PRESENT_ABSENT=UNCHANGED
PERF028_A_DESTINATION_BEFORE_RHS=UNCHANGED
RETURN_HOME_NLR=UNCHANGED
TASK_DYNAMIC_CONTROL_INHERITANCE=UNCHANGED
ACTOR_MODULE_DOMAIN_PROVENANCE=UNCHANGED
MULTI_CONTEXT_OWNERSHIP=UNCHANGED
DEBUGGER_SCOPE=UNCHANGED
TOOLING_FRAME_DELTA=UNCHANGED_FROM_PLAT044
```

## Implementation boundary

```text
PROPOSED_SLICE_3_NAME=
  PERF025: specialize residual inline callback Activation consumers

IMPLEMENTATION_REPOSITORY=guillermomolina/protos
IMPLEMENTATION_TYPE=PRODUCT
NEW_PLATFORM_DECISION_REQUIRED=NO
BENCHMARK_REQUIRED_INSIDE_SLICE=NO
```

Likely product files include:

```text
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosInlineCallbackFrameBindings.java
src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java
```

The exact touched set remains implementation-owned.

## Audit boundary

This record is a static audit only.

```text
IMPLEMENTATION_PERFORMED=NO
FILES_CHANGED_IN_PRODUCT=NO
TESTS_RUN=NO
BENCHMARKS_RUN=NO
ISSUES_MUTATED_BY_AUDIT=SEPARATE_PROJECT_BOOKKEEPING
```
