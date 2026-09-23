# PERF010-A — execution-context local-storage causal result

Status: RETAINED CAUSAL RESULT — CONTEXT-STORAGE CANDIDATE CLOSED AS NOT THE BIG COST

This durable, non-normative record captures the bounded PERF010-A Tier-B
execution-context local-storage intervention. It answers whether replacing the
generic `LinkedHashMap<String,Object>` backing used by fresh execution
contexts exposes a large hidden runtime cost capable of explaining the dominant
PERF010-A gap.

## Evidence identity

```text
PRODUCT_REPOSITORY=guillermomolina/protos
BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks

BASE_PROTOS_REVISION=
  3e8e6b565c95eb5098c2168d241536ba13ad19e9

HARNESS_REVISION=
  67002c702fe293b869ba8bda34814be6b7076136

HARNESS_COMMIT_MESSAGE=
  perf010a: add execution-context storage causal experiment

CONTROL_PATCH=NONE
INTERVENTION_PATCH=
  docker/protos-perf010a/context-storage.patch

PRODUCT_COMMIT=NONE
PROTOS_REPOSITORY_MODIFICATION=NONE
```

The intervention was applied only in the diagnostic benchmark image. No
product revision was created for this experiment.

The benchmark reference runner reported the Evidence Unit as retained. No
separate benchmark evidence commit containing the generated result directory
was published as part of this checkpoint, so this record does not invent one.

## Intervention boundary

The diagnostic intervention preserves fresh `ProtosObjectValue` execution
contexts, runtime class identity, delegation parent, lexical ordering,
shadowing, receiver fallback, mutation behavior, captured-context by-reference
semantics and generic fallback behavior.

It changes only the execution-context local-slot backing representation:

```text
CONTROL=
  ProtosObjectValue local slots backed by LinkedHashMap<String,Object>

INTERVENTION=
  fresh ProtosObjectValue execution contexts backed by a private
  compact array-based local-slot store

ORDINARY_OBJECT_STORAGE=
  unchanged LinkedHashMap path
```

This was deliberately narrower than a broad Frame/DynamicObject migration and
was intended to test whether the generic context-local map representation was a
large causal component by itself.

## Admission gates

The retained run reported:

```text
SLICE_TYPE=IMPLEMENTATION_CAUSAL_EVIDENCE
IMPLEMENTATION_REPOSITORY=guillermomolina/protos-benchmarks

STRUCTURAL_SCOPE_VALID=YES
CORRECTNESS_GATE=PASS

EVIDENCE_VALID=YES
EVIDENCE_STATUS=RETAINED
```

Timing used the retained Phase-2 measurement policy:

```text
WARMUP=120
STEADY=100
OPERATION_COUNT=10000
BLOCK_ORDER=A,B,A,B
WORKLOADS=4
```

The paired-control convention is unchanged: positive values mean the
intervention is faster after subtracting workload-control movement.

## Causal results

### micro/slot-read

```text
PAIRED_EFFECT_BLOCKS_PERCENT=
  -3.4494
  -19.9263
  -38.9760
  -13.1197

PAIRED_EFFECT_MEDIAN=-16.5230%
PAIRED_EFFECT_MAD=8.2384%
```

The diagnostic storage representation is materially slower on this workload.
This is evidence that local storage representation can be runtime-visible, but
in the wrong direction for the dominant-gap hypothesis.

### micro/closure-call

```text
PAIRED_EFFECT_BLOCKS_PERCENT=
  +3.5848
  -1.8570
  -3.1761
  +11.7683

PAIRED_EFFECT_MEDIAN=+0.8639%
PAIRED_EFFECT_MAD=3.3804%
```

No clear material causal improvement is established.

### micro/method-call

```text
PAIRED_EFFECT_BLOCKS_PERCENT=
  -7.4609
  -4.0431
  +1.9584
  -5.5428

PAIRED_EFFECT_MEDIAN=-4.7930%
PAIRED_EFFECT_MAD=1.7089%
```

The intervention is percentage-scale and predominantly slower.

### runtime/monomorphic-dispatch

```text
PAIRED_EFFECT_BLOCKS_PERCENT=
  -2.7512
  -0.3411
  +6.2401
  -3.4121

PAIRED_EFFECT_MEDIAN=-1.5462%
PAIRED_EFFECT_MAD=1.5355%
```

No large positive causal effect is present.

## Materiality classification

The reference runner classified the Evidence Unit as:

```text
PRIMARY_EFFECT_SCALE=NO_CLEAR_EFFECT
CAUSAL_RUNTIME_EFFECT=NOT_ESTABLISHED

TIER_B_CONTEXT_STORAGE_BIG_COST=NO
CANDIDATE_DISPOSITION=NO_MATERIAL_CAUSAL_EFFECT
ORDER_OF_MAGNITUDE_RELEVANT=NO

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED

MORE_INVESTIGATION_OF_THIS_CANDIDATE_BEFORE_DECISION=NO
NEXT_ACTION=STOP_CONTEXT_STORAGE_CANDIDATE_AND_CONTINUE_TIER_B_SEARCH
```

The result is sufficient for the dominant-cost question even though it does not
provide a precise small-effect estimate. A representation change capable of
explaining the retained cross-runtime gap would have needed a large
multiplicative signal. Instead, the two direct call workloads are near-zero to
small negative and `slot-read` becomes substantially slower.

Therefore:

```text
PERF010A_CONTEXT_STORAGE_HYPOTHESIS=CLOSED_NEGATIVE
CONTEXT_STORAGE_CAUSAL_MATERIALITY=NO
CONTEXT_STORAGE_EXPLAINS_BIG_GAP=NO
CONTEXT_STORAGE_FURTHER_WORK=STOP
```

No follow-up array-layout tuning, binary-search variant, JFR/IGV trace,
warmup refinement or additional context-storage micro-ablation is justified for
the dominant-gap search.

## Relation to the retained Tier-B class

The earlier cross-runtime root-cause record defined Tier B as:

```text
TIER_B=
  String-keyed heap activation/lexical lookup
  and per-call activation/context materialization
```

This Evidence Unit closes only the local-slot backing-storage sub-hypothesis.
It does not establish that the broader per-call activation/context
materialization cost is small.

The remaining Tier-B search must therefore move to a distinct causal mechanism,
not another local-slot container variant.

The next candidate must satisfy all three rules:

1. directly measure causal materiality; or
2. if direct measurement cannot yet run, implement only the minimum mechanism
   strictly necessary for the immediately following causal measurement; and
3. the resulting Evidence Unit must be able to terminate the candidate as
   `NOT_THE_BIG_COST`.

A semantics-preserving diagnostic that avoids eager per-call activation/context
materialization while preserving fresh observable identity and exact
materialization fallback is the next architecture-level discriminator to
consider. Broad production migration is not authorized by this record.

## Reconciled live state

```text
TIER_A_STABLE_IDENTITY_BIG_COST=NO
TIER_B_CONTEXT_STORAGE_BIG_COST=NO

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO

NEXT_CANDIDATE_CLASS=
  TIER_B_PER_CALL_ACTIVATION_CONTEXT_MATERIALIZATION

NEXT_ACTION=
  STOP_CONTEXT_STORAGE_CANDIDATE_AND_MEASURE_NEXT_TIER_B_MECHANISM
```
