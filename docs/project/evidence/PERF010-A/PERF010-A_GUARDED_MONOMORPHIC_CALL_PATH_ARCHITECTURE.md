# PERF010 / PERF010-A — Guarded monomorphic call-path architecture

Status: ARCHITECTURE ESTABLISHED; CAUSAL DOMINANCE NOT CLOSED

This durable, non-normative record retains the bounded investigation requested
for PERF010 / #680 after the cross-runtime root-cause investigation retained for
PERF010-A / #691. The investigation asked whether current `origin/main` has a
semantics-preserving guarded monomorphic call path capable of removing a large
repeated generic-dispatch layer from stable ordinary calls before any production
optimization is selected.

The investigation was read-only. It did not modify Protos, benchmarks, tests,
versions, changelogs, or benchmark evidence.

## Evidence identity

```text
PROTOS_REVISION=cd32d7d3228f88c6ebda65f357787a692b89c666
PREVIOUS_PROJECT_RECORD_REVISION=fc8dc35a9bb6cbd38fa53a2297cbe28f3d2c841b
SOURCE_RECORD=docs/project/evidence/PERF010-A/PERF010-A_CROSS_RUNTIME_ROOT_CAUSE.md
```

The requested `AGENTS.work/REPRODUCIBILITY.md` path does not exist at this Protos
revision. The investigation instead verified the applicable repository,
performance, implementation, source-tree, coordination, and specification
governance that exists on current `main`.

## Result

```text
GUARDED_MONOMORPHIC_CALL_PATH=ESTABLISHED
PARETO_GATE=PASS

PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PERF010_READY=NO
NEW_DECISION_REQUIRED=NO
```

The architecture is sufficiently established for one bounded causal experiment.
This result does not select a production optimization and does not satisfy the
stronger causal-attribution closure contract of PERF010-A / #691.

## Current call-path boundary

For an ordinary call/send, current source reaches the stable direct target only
after substantial generic work:

```text
receiver / arguments
  -> PrepareClosureCall or PrepareSendArguments
  -> ProtosValueLookup.lookup(...)
  -> selected Closure + methodHome
  -> prepareImmediateMethodCall / canonical Closure-call handling
  -> fresh ProtosActivation
  -> Task/dynamic-control propagation
  -> finishPreparingComposedCallByImplementation(...)
  -> execution-plan / entered-context projection handling
  -> PreparedClosureCall
  -> structured-dispatch classification
  -> EnterClosureCall
  -> cached RootCallTarget guard
  -> DirectCallNode.call(...)
```

The current effective specialization boundary is therefore too late for the
Tier-A hypothesis: by `EnterClosureCall`, the invocation has already paid the
runtime-generic preparation and structured-dispatch classification costs.

## Safe specialization boundary

The earliest generally established boundary is immediately after authoritative
ordinary lookup has established:

```text
selected ProtosClosureValue
selected methodHome
```

and before `prepareImmediateMethodCall` /
`finishPreparingComposedCallByImplementation`.

At this point the runtime may specialize a stable ordinary composed call while
preserving D013 lookup semantics and retaining the exact generic path as the
guard-failure fallback.

The selected Closure itself contains stable information that makes repeated
implementation-category classification unnecessary on a cache hit. In
`ProtosClosureValue`, the native implementation reference is final. Therefore,
for the same selected Closure, the answer to whether it is an ordinary composed
Closure or a native structured implementation cannot later change.

## Existing architectural precedent

Current source already contains the relevant execution seam in the Task-owned
path:

```text
prepareTaskOwnedSelectedCallIfBytecode(...)
prepareTaskOwnedDirectClosureIfBytecode(...)
```

After D013 selection, these methods can construct the fresh activation and a
`PreparedClosureCall` directly from the effective Bytecode plan without passing
through `finishPreparingComposedCallByImplementation(...)`.

This is direct current-source evidence that the post-lookup composed-call bypass
is compatible with the existing activation, method-home, execution-plan, Task,
and control-transfer model. PERF010 does not need to invent a new semantic
invocation mechanism to test the hypothesis on the ordinary path.

## Generic work that can disappear from a guarded hit

`finishPreparingComposedCallByImplementation(...)` currently performs 16
implementation/category classifications, including standard `ensure`, `handle`,
`while`, Boolean callback, collection `each`, Map operations, `signal`,
`Object.call`, and import-related classification.

The ordinary prepared-invocation lowering subsequently contains 19 structured
call predicates before its final ordinary arm, covering Map match/case,
`Object.call`, import, control operations, collection callbacks, and Map/IdentityMap
operations.

For a correctly guarded ordinary composed-call hit:

```text
FAST_PATH_GENERIC_CATEGORY_CLASSIFIERS=0
FAST_PATH_STRUCTURED_DISPATCH_PREDICATES=0
```

The generic universe remains reachable only through guard failure/fallback.
This is the basis for `PARETO_GATE=PASS`: the candidate removes an architectural
layer rather than a single allocation, `Optional`, map probe, or helper call.

## D013 lookup and mutation audit

`ProtosObjectValue` has an immutable delegation-parent reference, but its local
slot table is mutable according to the OPEN/CLOSED/FROZEN object rules.

```text
OPEN:   create=yes, assign=yes, remove=yes
CLOSED: create=no,  assign=yes, remove=no
FROZEN: create=no,  assign=no,  remove=no
```

Consequently `CLOSED` does not imply stable lookup. Existing method values may
still be replaced on a closed object.

No general implementation-side slot-shape/version invalidation mechanism was
found on current main. Searches found no operative general use of:

```text
Assumption
CyclicAssumption
@CompilationFinal
transferToInterpreterAndInvalidate
```

for ordinary object-slot lookup invalidation.

Therefore:

```text
LOOKUP_CAN_BE_SKIPPED=CONDITIONAL
GENERIC_PREPARATION_CAN_BE_SKIPPED=YES
```

Arbitrary mutable delegation lookup must either remain authoritative per
invocation or be replaced by guards that prove the exact D013 result still
holds.

Bounded cases already admit such guards. A constant-selector local method on a
stable receiver can guard receiver identity plus the exact local selected-slot
value. Canonical Closure invocation can guard the Closure receiver and absence
of a local `call` override while relying on the published frozen standard
`Object.call` binding. Fully frozen relevant chains are likewise stable.

## Cacheable facts and required guards

Cacheable facts include:

- constant selector at an ordinary send site;
- selected Closure identity;
- selected `methodHome` when its provenance is guarded;
- immutable delegation-parent identities;
- composed/native implementation classification;
- absence of native structured-protocol categories for the selected composed
  Closure;
- effective execution-plan identity and its `RootCallTarget`;
- current entered-context projection when required.

Required guards depend on the bounded call shape, but may include:

- receiver identity;
- local selector-slot presence and exact value identity;
- selected Closure identity;
- selected `methodHome` provenance;
- frozen-path proof where applicable;
- context-local projection/effective target validity.

Guard failure returns to the exact current generic preparation and structured
fallback; it must not weaken lookup, override, import, structured-control,
continuation, or error semantics.

## Activation work remains semantically live

The investigation does not establish that activation/context construction can be
removed. `CALLABLES.md` requires a fresh activation context and current source
also establishes per-invocation receiver, `methodHome`, lexical relationships,
arguments, return-home state, execution domain, Actor/module state, and Task or
dynamic-control propagation.

Therefore:

```text
ACTIVATION_WORK_REMAINING=
  fresh execution context
  semantic argument vector/array
  receiver and methodHome
  captured lexical relations
  return-home establishment
  actor/module/execution-domain state
  Task/dynamic-control propagation
```

Tier B remains an independent later question. Establishing the Tier-A bypass
does not claim that activation cost is removable.

## Workload coverage

The four PERF010-A common-path workloads classify as:

```text
micro/slot-read=INDIRECT
micro/closure-call=DIRECT
micro/method-call=DIRECT
runtime/monomorphic-dispatch=DIRECT
```

`slot-read` is indirect because `holder.value` is still a normal member read,
but every benchmark iteration also executes repeated Closure/control calls. The
other three workloads exercise the call machinery directly.

## Compiler-facing consequence

A correct guarded ordinary fast arm can expose a hot graph containing bounded
semantic guards, required activation construction, the effective target, and a
`DirectCallNode`, while moving unrelated import/native/structured protocol
machinery behind the fallback edge.

This supports the architectural expectation:

```text
GENERIC_BRANCH_UNIVERSE_REMOVED_FROM_HOT_GRAPH=YES
```

That statement remains an implementation hypothesis until a compiler diagnostic
confirms the generic universe is absent from the specialized hot arm; it is not a
claim about measured Graal graph size at this investigation checkpoint.

## Next bounded causal experiment

The next experiment should implement exactly one diagnostic guarded fast path for
an ordinary composed method send with constant selector and a monomorphic local
method on the receiver.

The hit should require, at minimum:

```text
receiver == cachedReceiver
receiver local selector slot == cachedComposedClosure
effective Bytecode target remains valid for the entered context
```

On a hit it should reuse the existing fresh activation and control-transfer
semantics, then enter the ordinary call path directly without executing generic
implementation classification or the structured-dispatch ladder.

On a miss it must execute the exact current generic path.

The causal experiment should prove all of the following before PERF010 selects a
production optimization:

1. semantic regressions remain passing, including replacement/removal/shadowing,
   receiver changes, non-local return, and suspension/resumption cases;
2. the hot ordinary path no longer executes the generic category and structured
   dispatch universe;
3. compiler diagnostics show that machinery absent from the specialized hot arm;
4. paired benchmark evidence on at least `micro/method-call` and
   `runtime/monomorphic-dispatch` shows a material repeatable reduction attributable
   to the bypass.

No percentage threshold is preselected. A structurally correct bypass that does
not produce a material controlled reduction is evidence against Tier A being the
dominant practical cause.

## Reconciled live state

```text
GUARDED_MONOMORPHIC_CALL_PATH=ESTABLISHED
PARETO_GATE=PASS
CURRENT_SPECIALIZATION_BOUNDARY=late EnterClosureCall target guard
EARLIEST_SAFE_SPECIALIZATION_BOUNDARY=post-authoritative-lookup
LOOKUP_CAN_BE_SKIPPED=CONDITIONAL
GENERIC_PREPARATION_CAN_BE_SKIPPED=YES
STRUCTURED_PROTOCOL_RECLASSIFICATION_REMOVED=YES
D013_PRESERVED=YES
FALLBACK_PATH=current generic path
SEMANTIC_BLOCKERS=none for bounded post-lookup experiment
ARCHITECTURAL_BLOCKERS=none for bounded experiment
NEXT_CAUSAL_EXPERIMENT=guarded local-method monomorphic composed-send bypass
NEW_DECISION_REQUIRED=NO
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PERF010_READY=NO
```

PERF010-A / #691 remains open and continues to gate production selection under
PERF010 / #680. The next work is the bounded causal experiment above; it is not a
production optimization commitment.
