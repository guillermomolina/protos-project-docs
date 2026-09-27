# PERF014 — Direct Closure-call guarded-selection implementation checkpoint

Date: 2026-09-27

## Scope

This record retains the published product/structural checkpoint for
PERF014 / guillermomolina/protos#725, the PERF010-B Step-2 implementation that
stabilizes direct Closure-call selection on the hot path.

This is not the PERF014 timing conclusion. The predeclared controlled timing
checkpoint remains pending and will determine the routing back through
PERF010-B / #722 before PERF015 is activated.

## Evidence identity

```text
WORK_ITEM=PERF014/#725
PARENT=PERF010-B/#722

PRODUCT_REPOSITORY=guillermomolina/protos
BASELINE_REVISION=0af8960363a557dad1b87968cf8a632e4716ee8a
PRODUCT_REVISION=cc76159a1bc9e2e1d46363f1c9047aa1034474df
PRODUCT_VERSION=0.3.103-SNAPSHOT

COMMIT_MESSAGE=PERF014: guard direct Closure-call selection on the hot path
```

GitHub records the product revision as a direct child of the PERF013 B2
baseline.

## Published delta

The product commit changes exactly these four files:

```text
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf014DirectClosureCallSpecializationTest.java
pom.xml
CHANGELOG.md
```

No specification file is changed.

The product version moves:

```text
0.3.102-SNAPSHOT -> 0.3.103-SNAPSHOT
```

## Implementation result

The published implementation adds guarded fast-hit specializations to all three
direct Closure-call preparation operations:

```text
PrepareClosureCall
PrepareClosureCallArguments
PrepareDefaultClosureCallArguments
```

The implementation deliberately reuses the existing I072 institutions rather
than introducing a parallel call architecture:

```text
ProtosValueLookup.lookupGuarded("call")
  -> selector-specific lookup-stability Assumption
  -> exact canonical Object.call selection check
  -> existing Context-owned ordinary source target derivation
  -> existing ordinary PreparedClosureCall shape
  -> existing EnterClosureCall.direct DirectCallNode
```

The admitted hit is restricted to a non-native, ordinary source-backed Closure
whose selected `call` behavior is exactly the canonical standard
`Object.call` behavior.

The generic `prepareClosureCall` path remains the exact fallback for
non-admitted cases.

## Structural discriminator

Maintainer-reported final structural result before publication:

```text
DIRECT_CLOSURE_STABLE_SELECTION=PASS
DIRECT_CLOSURE_CONTEXT_OWNED_TARGET=PASS
DIRECT_CLOSURE_DIRECT_CALL_NODE=PASS

GENERIC_CALL_LOOKUP_ON_VALID_HIT=NO
GENERIC_IMPLEMENTATION_CLASSIFICATION_ON_VALID_HIT=NO

CALL_SHADOWING=PASS
CALL_SELECTION_INVALIDATION=PASS
GENERIC_FALLBACK=PASS

OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
FINAL_REQUIRED_VALIDATION=PASS
```

The key PERF014 distinction is not merely that a `DirectCallNode` exists; one
already existed before this work. The new valid direct-Closure hit no longer
repeats the generic D013 `call` lookup or the generic structured/native
implementation classification before reaching the existing direct-call path.

## Preserved fallback / invalidation coverage

The new focal regression class contains seven bounded cases, including retained
coverage for:

- ordinary stable direct Closure invocation;
- direct invocation with supplied arguments;
- direct invocation with default arguments;
- local `call` override shadowing canonical fast-path admission;
- introduction of a `call` override after cache warmup invalidating the
  canonical fast hit;
- remove/recreate cycles switching correctly between guarded and generic paths;
- native direct Closure invocation remaining on the generic path across
  repeated calls.

The implementation keeps the generic path for override, shadowing, native body,
non-canonical selection, unsupported/projection failure, invalidation, and
other non-admitted states.

## Validation

The maintainer reported the required integrated product gate green:

```text
COMMAND=make test
RESULT=PASS
PRODUCT_REVISION=cc76159a1bc9e2e1d46363f1c9047aa1034474df
```

The new test source retains the repository-required license notice. No
normative ambiguity or specification update was reported by the final delta.

## Relationship to prior PERF013 timing

The retained PERF013 B1 -> B2 controlled checkpoint established:

```text
SLOT_READ_EFFECT=-2.6010%
CLOSURE_CALL_EFFECT=-0.3920%
METHOD_CALL_EFFECT=+18.8301%
MONOMORPHIC_DISPATCH_EFFECT=+22.9525%
```

That result is the immediate causal backdrop for PERF014: captured-local
read/write remediation materially improved method-call and monomorphic dispatch
but left `micro/closure-call` essentially unchanged.

PERF014 therefore targets a separate direct Closure-call selection/classification
cost and must be measured independently.

## Pending timing gate

PERF014 is not complete until the predeclared controlled timing discriminator is
measured against the exact product revisions.

Required next checkpoint:

```text
CONTROL_PRODUCT_REVISION=0af8960363a557dad1b87968cf8a632e4716ee8a
INTERVENTION_PRODUCT_REVISION=cc76159a1bc9e2e1d46363f1c9047aa1034474df

PREDICTION=
  closure-call - workload-control guest-call increment
  falls by a clearly multiplicative factor

CONTROLLED_TIMING_CHECKPOINT=PENDING
```

Routing after measurement:

```text
material multiplicative improvement
    -> retain result on PERF014/#725 and route through PERF010-B/#722

structural success but timing nearly unchanged
    -> timing prediction falsified
    -> return to PERF010-B/#722 for causal re-evaluation
    -> do not silently activate PERF015

PERF015/#726
    -> remains BLOCKED until #722 explicitly routes forward
```

## Checkpoint state

```text
PERF014_PRODUCT_SLICE=IMPLEMENTED
PRODUCT_PUBLICATION=PASS

DIRECT_CLOSURE_STABLE_SELECTION=PASS
DIRECT_CLOSURE_CONTEXT_OWNED_TARGET=PASS
DIRECT_CLOSURE_DIRECT_CALL_NODE=PASS
GENERIC_CALL_LOOKUP_ON_VALID_HIT=NO
GENERIC_IMPLEMENTATION_CLASSIFICATION_ON_VALID_HIT=NO
CALL_SHADOWING=PASS
CALL_SELECTION_INVALIDATION=PASS
GENERIC_FALLBACK=PASS
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
FINAL_REQUIRED_VALIDATION=PASS

CONTROLLED_TIMING_CHECKPOINT=PENDING
PERF014_STATUS=IN_PROGRESS
PERF015_STATUS=BLOCKED
```
