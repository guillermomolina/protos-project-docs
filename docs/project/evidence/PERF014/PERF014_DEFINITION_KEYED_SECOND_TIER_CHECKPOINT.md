# PERF014 — Definition-keyed direct Closure-call second-tier checkpoint

Date: 2026-09-27

## Scope

This record retains the published PERF014 / guillermomolina/protos#725 follow-up
that completes the direct Closure-call specialization with a second,
definition-keyed cache tier after the initial receiver-identity-only
implementation exposed identity-churn cost.

This is still not the final PERF014 timing conclusion. The predeclared retained
controlled timing checkpoint remains pending and continues to gate PERF014
closure and any routing to PERF015 / #726.

## Evidence identity

```text
WORK_ITEM=PERF014/#725
PARENT=PERF010-B/#722

PRODUCT_REPOSITORY=guillermomolina/protos

PERF014_BASELINE_REVISION=0af8960363a557dad1b87968cf8a632e4716ee8a
PERF014_BASELINE_VERSION=0.3.102-SNAPSHOT

PERF014_FIRST_TIER_REVISION=cc76159a1bc9e2e1d46363f1c9047aa1034474df
PERF014_FIRST_TIER_VERSION=0.3.103-SNAPSHOT

INTERVENING_BUG010_REVISION=eab6a367c16dea0136e1aabb13a7da681c4839b0
INTERVENING_BUG010_VERSION=0.3.104-SNAPSHOT

PERF014_FINAL_PRODUCT_REVISION=bcf9eda164d840b0a0b4201753fe5289347afa8a
PERF014_FINAL_PRODUCT_VERSION=0.3.105-SNAPSHOT
```

The final PERF014 commit message is:

```text
PERF014: add definition-keyed second cache tier for direct Closure calls
```

## Product lineage and intervening unrelated change

The published lineage is:

```text
0af8960363a557dad1b87968cf8a632e4716ee8a  PERF013 B2 / PERF014 baseline
  ->
cc76159a1bc9e2e1d46363f1c9047aa1034474df  PERF014 first tier
  ->
eab6a367c16dea0136e1aabb13a7da681c4839b0  BUG010 Test Tool progress
  ->
bcf9eda164d840b0a0b4201753fe5289347afa8a  PERF014 definition-keyed second tier
```

BUG010 is unrelated to the direct Closure-call optimization. Its product delta
is confined to:

```text
CHANGELOG.md
pom.xml
protos/tools/test/LogicalCaseRunner.protos
protos/tools/test/Main.protos
src/test/java/.../ProtosTestToolBug010ProgressTemporalVisibilityTest.java
src/test/java/.../ProtosTestToolFileSelectionMainAdoptionTest.java
src/test/java/.../ProtosTestToolTool009ALifecycleReporterTest.java
```

It does not modify `ProtosBytecodeRootNode` or the benchmark workload sources.
The final controlled timing harness must nevertheless retain workload-source
identity and artifact revision/version checks and explicitly record this
intervening commit rather than silently treating the final intervention as a
direct child of the PERF014 baseline.

## First-tier diagnosis

The first PERF014 implementation introduced guarded direct Closure-call
specializations on:

```text
PrepareClosureCall
PrepareClosureCallArguments
PrepareDefaultClosureCallArguments
```

with an exact receiver-identity tier:

```text
receiver == cachedReceiver
limit = 3
```

That tier correctly preserved D013 selection and invalidation semantics, but it
was incomplete as a performance institution for repeated fresh materializations
of one Closure definition.

The observed failure mode was:

```text
many distinct ProtosClosureValue instances
  -> same executable Closure definition
  -> receiver-identity tier pays repeated specialization establishment
  -> receiver PIC limit is consumed by semantic object identity
  -> generic fallback after identity churn
```

This is the same category of problem already avoided by the ordinary-send
`PrepareSendArguments.fastOrdinarySend` second tier.

## Final second-tier implementation

The final PERF014 follow-up adds `fastDirect` to all three direct Closure-call
preparation Operations.

The second tier is keyed by:

```text
ProtosClosureValue.definition()
  -> CanonicalClosure identity
  -> entered ProtosLanguageContext
  -> Context-owned RootCallTarget
```

while the current semantic Closure instance remains dynamic.

The authoritative D013 `call` selection is still performed fresh on every
second-tier hit through `directClosureCallSelectionOrNull`. Therefore a local
`call` override on one particular Closure instance remains observable and
cannot be hidden by a cached executable target.

The steady-state shape is now:

```text
dynamic semantic Closure
  -> authoritative canonical call-selection check
  -> stable Closure definition / entered Context
  -> cached Context-owned RootCallTarget
  -> PreparedClosureCall built from the current dynamic Closure
  -> existing EnterClosureCall.direct / DirectCallNode
```

This separates semantic object identity and capture state from stable executable
identity instead of consuming one PIC entry per materialized Closure object.

## Final published delta

The PERF014 second-tier commit itself changes exactly:

```text
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf014DirectClosureCallSpecializationTest.java
pom.xml
CHANGELOG.md
```

No normative specification file changes.

The version moves:

```text
0.3.104-SNAPSHOT -> 0.3.105-SNAPSHOT
```

The new regression coverage includes:

```text
freshClosureMaterializationsOfTheSameDefinitionRemainCorrectPastBothCacheLimits
```

which exercises more fresh Closure identities than either cache limit while
retaining one executable definition.

## Validation

The maintainer reported that `make test` passed on the same final
`ProtosBytecodeRootNode.java` and PERF014 test bytes that were published.
Temporary diagnostic sampling used during the investigation was confined to
`ProtosTestToolAsyncExecutionScope.java`, was never part of the final PERF014
delta, and was removed; that file was verified byte-identical to its published
pre-diagnostic state before the final commit.

Retained final validation state:

```text
FINAL_REQUIRED_VALIDATION=PASS
DIAGNOSTIC_INSTRUMENTATION=REMOVED_VERIFIED
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
```

## Concurrency-regression investigation

The first-tier implementation was followed by an observed Test Tool slowdown
and visibly poorer CPU scaling. The maintainer reported historical observations
approximately:

```text
pre-PERF014 Protos test phase ~118 s
PERF014 receiver-only first tier ~138 s
PERF014 + definition-keyed tier ~124-129 s in later runs
```

The historical ~118 s result was not reproducible as the current baseline on
the later host/session, so it is retained as historical observation rather than
used as a causal PERF014 timing result.

A temporary diagnostic harness then compared the exact PERF014 baseline and
the current PERF014+tier2 worktree on the same host and same harness:

```text
baseline jobs=8:
  real=147.93 s
  user=1021.94 s
  user/real=6.9x

current PERF014+tier2 jobs=8:
  real=129.71 s
  user=950.49 s
  user/real=7.3x

baseline jobs=16:
  real=145.40 s
  user=1212.16 s
  user/real=8.3x
  peak activeCaseCarriers=16

current PERF014+tier2 jobs=16:
  real=116.94 s
  user=918.12 s
  user/real=7.85x
  peak activeCaseCarriers=16
```

A later direct `make test-protos` comparison on the same current host/session
reported:

```text
baseline 0af896... = 135 s
current final path = 129 s
```

These data support only the bounded conclusions below:

```text
RECEIVER_IDENTITY_CHURN=CONFIRMED
DEFINITION_KEYED_TIER=MATERIAL_FIX
ADMISSION_CAP_AT_8=FALSIFIED
PERF014_SPECIFIC_LOSS_OF_ACTIVE_CASE_CONCURRENCY=FALSIFIED
PERF014_SPECIFIC_CPU_UTILIZATION_REGRESSION=NOT_REPRODUCED
HISTORICAL_118S_BASELINE=NOT_REPRODUCED_IN_CURRENT_SESSION
```

They are not the retained PERF014 microbenchmark timing checkpoint and must not
be used to claim a quantified steady-state Closure-call improvement.

## Remaining controlled timing gate

PERF014 remains open until the retained causal comparator measures the final
product.

Required product identities:

```text
CONTROL_PRODUCT_REVISION=0af8960363a557dad1b87968cf8a632e4716ee8a
CONTROL_PRODUCT_VERSION=0.3.102-SNAPSHOT

INTERVENTION_PRODUCT_REVISION=bcf9eda164d840b0a0b4201753fe5289347afa8a
INTERVENTION_PRODUCT_VERSION=0.3.105-SNAPSHOT
```

The final intervention contains the intervening unrelated BUG010 commit, so the
Evidence Unit must explicitly prove that the benchmark workload source identity
is unchanged and record the unrelated lineage. It must not silently describe the
two product revisions as direct parent/child revisions.

The predeclared PERF014 discriminator remains:

```text
closure-call - workload-control guest-call increment
    -> must fall by a clearly multiplicative factor
```

The harness must retain raw paired-control evidence rather than auto-classifying
that judgment.

Routing remains:

```text
material multiplicative improvement
    -> retain final PERF014 result
    -> close/route #725 through PERF010-B/#722
    -> #722 decides whether PERF015/#726 is activated

structural success but timing nearly unchanged
    -> PERF014 timing prediction falsified
    -> close/retain the valid structural result
    -> return to #722 for causal re-evaluation
    -> do not silently activate PERF015
```

## Checkpoint state

```text
PERF014_PRODUCT_IMPLEMENTATION=FINAL_PUBLISHED
PERF014_FINAL_PRODUCT_REVISION=bcf9eda164d840b0a0b4201753fe5289347afa8a
PERF014_FINAL_PRODUCT_VERSION=0.3.105-SNAPSHOT

DIRECT_CLOSURE_STABLE_SELECTION=PASS
DIRECT_CLOSURE_CONTEXT_OWNED_TARGET=PASS
DIRECT_CLOSURE_DIRECT_CALL_NODE=PASS
DEFINITION_KEYED_SECOND_TIER=PASS
RECEIVER_IDENTITY_CHURN_REGRESSION_COVERAGE=PASS
CALL_SHADOWING_PRESERVED=PASS
GENERIC_FALLBACK_PRESERVED=PASS
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
FINAL_REQUIRED_VALIDATION=PASS

CONTROLLED_TIMING_CHECKPOINT=PENDING
PERF014_STATUS=IN_PROGRESS
PERF015_STATUS=BLOCKED
```
