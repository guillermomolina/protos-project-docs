# PERF010-A — Cross-Truffle dominant-cost synthesis

Status: **CROSS-TRUFFLE SYNTHESIS ESTABLISHED; DOMINANT ARCHITECTURAL CLASS IDENTIFIED; CAUSAL CLOSURE PENDING**

This durable, non-normative record synthesizes the retained PERF004/PERF010-A
evidence with source-level comparison against mature Truffle implementations.
It does not change Protos semantics, modify the product or benchmark
repositories, select a production optimization, or authorize a broad runtime
migration.

The purpose is narrower: explain which remaining architectural class can still
plausibly account for most of the large absolute steady-state cost after the
already-tested local mechanisms have been closed or reduced to percentage-scale
effects, and define the one causal discriminator capable of falsifying that
class as "not the big cost".

## Evidence identity

```text
CURRENT_PROTOS_REVISION=
  3e8e6b565c95eb5098c2168d241536ba13ad19e9

PREVIOUS_PROJECT_RECORD_REVISION=
  4cad80067b7909d9610d9be7e46786db0d97952f

PERF004A_EVIDENCE_REVISION=
  5e8ff21f966c6c506652eef79c684d8b286bb546
PERF004A_HARNESS_REVISION=
  60dbce7faf5510bd1bd6867a866aa7ca69c48637
PERF004A_PROTOS_REVISION=
  4a03efc15620b37b2e418b3df30b4a26486446ec

PERF004_B2C_EVIDENCE_REVISION=
  8899ec163d6c0c7133b15ffa9c323447b683dbf2
PERF004_B2C_HARNESS_REVISION=
  8be0dca8a1d2b7203663f409c35159a5bd5998b5

STABLE_IDENTITY_CAUSAL_RESULT=
  docs/project/evidence/PERF010-A/PERF010-A_STABLE_IDENTITY_PHASE2_CAUSAL_RESULT.md

CONTEXT_STORAGE_CAUSAL_RESULT=
  docs/project/evidence/PERF010-A/PERF010-A_CONTEXT_STORAGE_CAUSAL_RESULT.md
```

Cross-runtime source revisions inspected:

```text
TruffleSqueak=
  hpi-swa/trufflesqueak@818519b2b6a6556bc524e9e0d08f7b51969cb61a

TruffleSOM=
  SOM-st/TruffleSOM@73f6d2e654022565ec7c7e8ba95ae18340a862ce

TruffleRuby=
  truffleruby/truffleruby@0e6fa6a950dce7154d54f3c9c63056c4eb925ffd

Graal_Truffle=
  oracle/graal@b3ebbbefb3289f862498beb4e84f5912aa2c0557
```

The source comparison is architectural evidence, not a claim that the compared
languages have identical semantics or benchmark implementations.

## PERF004-A gap reconstruction

The retained PERF004-A steady-state medians are:

| workload | Protos ns | CPython ns | Protos/Python | Node ns | Protos/Node |
|---|---:|---:|---:|---:|---:|
| `micro/slot-read` | 49,378,637 | 1,580,361 | 31.25x | 116,250 | 424.76x |
| `micro/closure-call` | 67,611,903 | 1,767,596.5 | 38.25x | 87,400 | 773.59x |
| `micro/method-call` | 55,565,935.5 | 1,754,912 | 31.66x | 126,715.5 | 438.51x |
| `runtime/monomorphic-dispatch` | 54,842,150 | 1,759,496.5 | 31.17x | 123,245 | 444.98x |
| `algorithms/factorial/recursive` | 5,014,013.5 | 1,320 | 3798.50x | 1,040 | 4821.17x |

The important quantity for root-cause work is the absolute common-workload
excess. Against CPython it is approximately 47.8-65.8 ms; against Node it is
approximately 49.3-67.5 ms. The 31-38x versus CPython and 425-774x versus Node
therefore describe nearly the same physical Protos excess over very different
external denominators.

The 4821.17x factorial result is different in ratio but not in absolute scale:
Protos takes about 5.0 ms while the Node denominator is about 1.04 microseconds.
It is therefore retained as an amplified ratio, not as evidence for a separate
4821x physical mechanism.

## Strongest common-path causal evidence

PERF004-B2-C preserved the same outer recursive `repeat`, activation, callback,
and control structure while replacing only the selected guest operation with
`sink = 42`.

| workload | canonical median | common control median | control/canonical |
|---|---:|---:|---:|
| `micro/slot-read` | 42.52 ms | 43.54 ms | 102.4% |
| `micro/closure-call` | 49.26 ms | 41.65 ms | 84.6% |
| `micro/method-call` | 50.61 ms | 46.50 ms | 91.9% |
| `runtime/monomorphic-dispatch` | 53.79 ms | 40.00 ms | 74.4% |

This is the strongest retained scale signal for the remaining search: roughly
three quarters to essentially all of the canonical cost survives after the
logical operation under test is removed.

The common machinery therefore has the frequency and scale required to explain
the large absolute gap. A mechanism that occurs only in the removed leaf
operation cannot explain this result.

## Dynamic operation model

All four canonical workloads execute a recursive guest-level:

```text
repeat(10000, operation)
```

A conservative lower bound is:

```text
10,001 repeat invocations
10,000 operation callback invocations
10,000 conditional-control decisions
tens of thousands of count/binding reads and updates
10,000 sink writes
```

The Closure-, method-, and monomorphic-dispatch workloads add another 10,000
logical invocations. Guest control itself is represented through callable
machinery, so the physical call/activation count is higher than those lower
bounds.

With approximately 40-46 ms surviving in the B2-C common controls, a common
per-invocation cost on the order of low microseconds is sufficient to explain
the observed absolute scale when repeated tens of thousands of times. No
single tens-of-milliseconds leaf operation is required.

## Negative evidence ledger

| mechanism | retained result | disposition for dominant-gap search |
|---|---|---|
| warmup/order/stationarity tuning | movement on percentage/tens-of-percent scale | measurement concern, not root cause |
| `continueAt` as a leaf target | ubiquitous execution container | symptom / execution envelope |
| semantic/helper Bytecode double dispatch | valid causal ablation about 3-6% | real but too small |
| steady-state Builder/lowering | 0% any-depth in current steady captures | closed negative |
| historical deoptimization storm | historical, not current steady state | amplifier / historical only |
| current Truffle deoptimization | zero in retained current steady captures | closed negative |
| guarded generic call-preparation bypass | negative timing | not dominant |
| prepared Context-owned target/compiler stranding | compiler-material but later timing percentage-scale | real but not dominant |
| stable PIC/executable identity | +8.0144% method-call, +11.2206% mono paired-control medians | real but not the big cost |
| duplicate lexical map probe | approximately -0.5% to +2.5% | local microcost |
| execution-context `LinkedHashMap` backing | compact-store experiment no material gain; slot-read regressed | closed negative as dominant |
| duplicate `Optional` / native-body projections | no positive common contribution | local microcost |
| individual collection copies/wrappers | bounded per invocation | insufficient alone |

Two distinctions follow from this ledger:

1. closing the `LinkedHashMap` backing does **not** close activation/lexical
   representation as a class;
2. the duplicate-probe ablation does **not** establish that lexical lookup is
   cheap; it removes one probe while preserving String-keyed dynamic resolution,
   captured-context traversal, and eager invocation-state representation.

## Protos representation that remains live

At both the PERF004-A product revision and the current fixed revision, the
following three files have identical Git blob identities:

```text
ProtosActivation.java
  6e0720e4951381775a8504e855f94c124affc0be

ProtosObjectValue.java
  12f9f00be419ed04fe08fdea527078c17ec21798

ProtosValueLookup.java
  70dcc82422b083ce3a1cba3732cc7f78edc6c93f
```

Therefore the activation/context/lexical representation under this
investigation is not a historical mechanism that disappeared between PERF004-A
and the current fixed revision.

The relevant current structure is:

```text
invocation
  -> fresh ProtosActivation
  -> fresh execution-context ProtosObjectValue
  -> copied captured-lexical-context list
  -> argument / return-home / method-home state
  -> lexical name represented as String
  -> ProtosActivation.lookup(name)
       -> current context readLocalSlot(name)
       -> captured lexical contexts loop
       -> receiver/prelude fallback
  -> ProtosObjectValue String-keyed local-slot representation
```

Changing only the backing container leaves nearly all of this representation
intact.

## Cross-Truffle source comparison

### TruffleSOM

`LocalVariableNode` stores a compiler-known `slotIndex` and directly performs
typed `VirtualFrame` accesses. `NonLocalVariableNode` stores both
`contextLevel` and `slotIndex`; captured lexical state is therefore reached
by fixed lexical depth plus indexed frame access rather than name lookup.

For ordinary sends, `UninitializedDispatchNode` performs lookup while
specializing the site. A `CachedDispatchNode` steady hit is structurally:

```text
guard.entryMatches(receiver)
  -> cachedMethod.call(arguments)
```

The generic lookup path remains as fallback.

### TruffleSqueak

`FrameAccess` defines fixed frame-argument and frame-slot layouts. Normal
execution uses `VirtualFrame` state; `ContextObject` and a
`MaterializedFrame` are created or attached when context observability
requires them.

`GetOrCreateContextWithoutFrameNode` can create the language context object
without forcing frame materialization. `GetOrCreateContextWithFrameNode`
materializes only on the path that requires a stored frame.

Dispatch uses a receiver-class guard plus method/class stability
`Assumption[]`. The monomorphic method hit reaches a `DirectCallNode` after
the lookup result has been specialized.

### TruffleRuby

Local reads and writes retain a fixed integer `frameSlot`.
`ReadDeclarationVariableNode` additionally retains a fixed `frameDepth`;
captured lexical reads therefore resolve declaration frame by depth and then
read the fixed slot.

`CallInternalMethodNode` caches `InternalMethod`, its
`RootCallTarget`, and a method-validity `Assumption`, then calls through a
`DirectCallNode`. The uncached path remains explicit.

### Current Graal/Truffle infrastructure

The Truffle API documents `DirectCallNode` as the call node for a stable
`CallTarget`, specifically enabling additional runtime optimization.

Frame locals are integer-indexed. Current Bytecode DSL `LocalAccessor` and
`MaterializedLocalAccessor` carry fixed offsets/indices and explicitly require
their accessors / Bytecode node to be partial-evaluation constants. The
infrastructure therefore provides a direct representation for the kind of
compiler-visible lexical stability seen in TruffleSOM and TruffleRuby.

### Shared architectural pattern

The compared implementations do **not** eliminate dynamic semantics. They move
dynamic work to the place where it is required:

```text
specialization / guard / invalidation / generic fallback
```

while the steady admitted path represents semantically stable facts through
compiler-visible indexes, depths, stable targets, and assumptions.

The important difference is therefore not "dynamic language versus static
language". It is whether semantic stability is represented as compiler-visible
state early enough for partial evaluation.

## Dominant architectural interpretation

The surviving candidate is not "LinkedHashMap is slow" and not "one allocation
is slow".

The candidate is:

```text
semantically stable invocation and lexical state
represented as dynamic heap runtime state
instead of compiler-visible indexed frame state
with materialization only when observability/escape requires it
```

The most concrete current mechanism is:

```text
eager fresh ProtosActivation / execution-context representation
+ String-keyed ProtosActivation lexical lookup
+ captured-context traversal
+ invocation-owned wrapper/state materialization
```

instead of the mature-Truffle pattern:

```text
frame arguments
+ fixed local slot index
+ fixed lexical depth
+ stable/direct target
+ lazy materialization for captured/observable state
+ exact generic fallback
```

The confidence values below are investigative confidence estimates, not
statistical confidence intervals:

```text
CONFIDENCE_ARCHITECTURAL_CLASS_PERCENT=91
CONFIDENCE_SPECIFIC_MECHANISM_PERCENT=82
```

The defensible current estimate is that this architectural class could account
for roughly 65-85% of the common absolute excess. This range is an analytical
model constrained by B2-C, not a measured attributable fraction. The formal
`ATTRIBUTABLE_FRACTION` remains `NOT_ESTABLISHED`.

## Why prior micro-ablations recover little

The prior interventions remove leaves while preserving the representation
boundary. For example, removing the duplicate local-slot probe still leaves:

```text
String name
 -> ProtosActivation.lookup
 -> heap execution context
 -> captured lexical traversal
 -> dynamic local-slot resolution
```

Likewise, replacing `LinkedHashMap` with a compact diagnostic store retains the
name-based lookup contract and eager execution-context object.

Those experiments therefore falsify the local mechanisms they actually remove.
They do not falsify an indexed-frame/lazy-materialization architecture that
would allow Graal to see lexical position and non-escaping invocation state as
compile-time structure.

## Factorial interpretation

The 4821.17x factorial ratio is classified as `PARTIAL` evidence for the same
common class.

Its shared cost is repeated call/activation/lexical-state representation at each
recursive/control step. Its test-specific amplifier is unusually high semantic
work density for only twenty recursive levels: Closure/message-based control,
dynamic arithmetic/dispatch, recursion/inlining pressure, and an external Node
denominator of only about 1.04 microseconds.

The ratio therefore does not establish a distinct 4821x leaf pathology.

## Next causal discriminator

The next experiment must be capable of falsifying the **architectural class**,
not merely another leaf.

The full discriminating intervention is:

```text
semantics-preserving source-Closure invocation path
  -> indexed Frame locals for admitted hot lexical bindings
  -> fixed lexical depth for admitted captured bindings
  -> frame arguments / compiler-visible invocation state
  -> defer physical execution-context materialization
  -> on context observation/capture/tooling escape:
       materialize exactly one fresh authoritative ProtosObjectValue
  -> preserve exact generic current path for unsupported or invalidated cases
```

The semantic contract remains authoritative:

- every invocation still has the semantics of one fresh execution context;
- if context identity becomes observable, exactly one unique context object is
  materialized for that invocation;
- subsequent reads, writes and captures observe the same authoritative object by
  reference;
- lexical precedence, capture, receiver fallback, return-home, structured
  control, suspension, Task/dynamic-control and tooling semantics are preserved;
- no benchmark-only guest semantic shortcut is introduced.

### Relationship to the already queued lazy-materialization slice

The already queued #691 experiment that defers physical invocation-state
materialization remains a valid **sub-hypothesis discriminator**.

A negative result from an intervention that only defers allocation but keeps
String-keyed lexical lookup can close:

```text
TIER_B_EAGER_INVOCATION_MATERIALIZATION_BIG_COST=NO
```

but it cannot by itself close the broader architectural class defined here.

To close the complete class as `NOT_THE_BIG_COST`, the intervention must also
remove name-based lexical resolution from the admitted hot path, or an
equivalent experiment must causally cover that remaining representation cost.

This distinction prevents another false negative caused by testing only a
single leaf of the candidate architecture.

## Required signal

The next full-class experiment is deliberately coarse-grained enough to be
discriminating under the already-established measurement floor.

If the hypothesis is dominant, the expected signal is clearly multiplicative,
not a few percent:

```text
EXPECTED_SIGNAL_IF_DOMINANT_HYPOTHESIS_TRUE=
  clearly multi-x end-to-end improvement on the common workloads,
  with an initial discrimination target of about >=5x,
  and a large collapse of the B2-C common-control cost
```

The numeric `>=5x` threshold is an experiment-design discriminator, not a
prediction that the final production implementation must deliver exactly that
speedup.

If a structurally valid intervention covering the full class preserves
correctness and removes both eager materialization and name-based lexical
resolution from the admitted hot path but remains only about 1.2-1.5x, this
architectural class must be closed as `NOT_THE_BIG_COST` and the search must
move on.

## Reconciled result

```text
PERF010A_CROSS_TRUFFLE_SYNTHESIS=ESTABLISHED
PERF004A_GAP_RECONSTRUCTED=YES

COMMON_WORKLOAD_GAP_VS_CPYTHON=31.17x..38.25x
COMMON_WORKLOAD_GAP_VS_NODE=424.76x..773.59x
MAX_RETAINED_GAP_VS_NODE=4821.17x

NEGATIVE_EVIDENCE_LEDGER_COMPLETE=YES

TRUFFLESOM_SOURCE_COMPARED=YES
TRUFFLESQUEAK_SOURCE_COMPARED=YES
TRUFFLERUBY_SOURCE_COMPARED=YES
GRAAL_TRUFFLE_INFRASTRUCTURE_COMPARED=YES

PROTOS_HOT_PATH_RECONSTRUCTED=YES
DYNAMIC_OPERATION_MODEL_ESTABLISHED=YES
ABSOLUTE_COST_MODEL_ESTABLISHED=YES

DOMINANT_ARCHITECTURAL_CLASS=
  SEMANTICALLY_STABLE_CALL_ACTIVATION_AND_LEXICAL_STATE_REPRESENTED_AS_DYNAMIC_HEAP_RUNTIME_STATE_INSTEAD_OF_COMPILER_VISIBLE_INDEXED_FRAME_STATE

DOMINANT_SPECIFIC_MECHANISM=
  EAGER_FRESH_INVOCATION_STATE_AND_EXECUTION_CONTEXT_MATERIALIZATION_PLUS_STRING_KEYED_LEXICAL_LOOKUP_AND_CAPTURED_CONTEXT_TRAVERSAL_INSTEAD_OF_FRAME_SLOT_INDEX_PLUS_LEXICAL_DEPTH_WITH_LAZY_MATERIALIZATION

CONFIDENCE_ARCHITECTURAL_CLASS_PERCENT=91
CONFIDENCE_SPECIFIC_MECHANISM_PERCENT=82

EXPLAINS_SLOT_READ_GAP=YES
EXPLAINS_CLOSURE_CALL_GAP=YES
EXPLAINS_METHOD_CALL_GAP=YES
EXPLAINS_MONOMORPHIC_DISPATCH_GAP=YES
EXPLAINS_FACTORIAL_4821X=PARTIAL

HISTORICAL_DEOPT_STORM_ROLE=AMPLIFIER
CONTINUE_AT_ROLE=SYMPTOM

TIER_A_REMAINING_MATERIALITY=
  LOW_TO_MODERATE_PERCENT_SCALE

TIER_B_REMAINING_MATERIALITY=
  HIGH_ARCHITECTURAL_MATERIALITY;
  BACKING_STORE_CLOSED_NEGATIVE;
  ACTIVATION_LEXICAL_REPRESENTATION_CLASS_STILL_LIVE

TIER_C_REMAINING_MATERIALITY=
  LOW_PERCENT_SCALE_ONLY

NEXT_CAUSAL_EXPERIMENT=
  INDEXED_FRAME_PLUS_LAZY_MATERIALIZATION_ARCHITECTURAL_CLASS_DISCRIMINATOR

NEXT_CAUSAL_EXPERIMENT_CAN_CLOSE_BIG_COST_HYPOTHESIS=YES

PRODUCTION_OPTIMIZATION_SELECTED=NO
IMPLEMENTATION_AUTHORIZED=NO
PRODUCT_MODIFICATION=NONE
BENCHMARK_MODIFICATION=NONE

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PERF010_READY=NO
```

PERF010-A / #691 remains the active causal owner. PERF010 / #680 remains gated
on #691. PERF011 / #693 should consume this record as cross-implementation
representation-fit evidence and must not infer authorization for a broad
Shape/DynamicObject/Frame rewrite.
