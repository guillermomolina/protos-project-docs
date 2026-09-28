# PERF017 — Static scaling and per-Case bootstrap investigation checkpoint

Date: 2026-09-28

## Scope

This record retains the static/history checkpoint for PERF017 /
guillermomolina/protos#729 and the independent implementation finding routed to
PERF018 / guillermomolina/protos#731.

It does not close PERF017 and does not claim that PERF018 is the root cause of
the historical/current parallel CPU-scaling ceiling.

## Evidence identity

```text
WORK_ITEM=PERF017/#729
FOLLOW_UP=PERF018/#731

PRODUCT_REPOSITORY=guillermomolina/protos
INSPECTED_MAIN_REVISION=1f966bd613442bccc49f61147cbccd54c91e7de5

TOOL009_LOGICAL_CASE_FOUNDATION_REVISION=757135678b6999fbde9cf559bd313ca1fa853218
TOOL009_INITIAL_PRODUCTION_MIGRATION_REVISION=7186e9d972d44f82b9090be8123433d591fce933

RELATED_PERF014_BASELINE_REVISION=0af8960363a557dad1b87968cf8a632e4716ee8a
RELATED_PERF014_FINAL_REVISION=bcf9eda164d840b0a0b4201753fe5289347afa8a
```

The current inspected `main` revision was read directly from the repository
branch head during this investigation.

## PERF014 concurrency evidence retained

PERF014's already-published checkpoint recorded:

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

That retained evidence establishes that the then-current Test Tool could admit
16 active Case carriers. It therefore falsifies a simple eight-Case admission
cap and does not support attributing the observed utilization behavior to
PERF014.

The historical project-owner observation that an earlier Test Tool state could
drive the machine closer to CPU saturation remains useful investigation input,
but an exact reproducible high-scaling product revision has not yet been
established. PERF017 therefore must not claim a first bad product revision yet.

## Current scheduler and host-carrier boundary

The current logical Case runner is work-conserving at the admission layer. Its
pull-lane design can admit a new ready Case when a lane becomes free rather than
waiting for an unrelated running Case to finish.

The production async exact-execution substrate remains the PLAT023 topology:

```text
already-admitted exact execution
  -> fresh named platform Thread
  -> no second host queue/capacity policy
```

The Test Tool RuntimeHost's Actor carrier executor is independently sized from
`Runtime.getRuntime().availableProcessors()`.

No current-repository evidence found a Protos-owned fixed host pool of eight or
one global Protos Context execution lock that would explain a hard 50% ceiling.

## Confirmed per-Case bootstrap shape

The current
`src/main/java/com/guillermomolina/protos/execution/ProtosTestLogicalCaseAttemptBridge.java`
contains, inside one logical Case attempt:

```java
ProtosPrelude prelude =
        new ProtosCoreBootstrap().bootstrap(core, resolver);

ProtosProcessRuntime process =
        new ProtosProcessRuntime(
                prelude.actorRefPrototypeForRuntime());
```

The bridge executes this for every selected logical Case.

At the inspected revision, `ProtosCoreBootstrap.bootstrap(...)` checks whether a
`ProtosPolyglotExecutionContext` is already entered. If none is entered, it
opens a temporary host:

```java
if (ProtosPolyglotExecutionContext.hasEnteredContextForRuntime()) {
    return bootstrapEntered(coreDirectory, moduleResolver);
}

try (ProtosPolyglotExecutionContext bootstrapHost =
        ProtosPolyglotExecutionContext.open(...)) {
    return bootstrapHost.callEntered(...);
}
```

The no-Engine overload of `ProtosPolyglotExecutionContext.open(...)` delegates
with a null Engine, so the temporary Context is not created on the Test Tool's
already-owned `ProtosPolyglotRuntimeHost` Engine.

Only after that Core/Prelude bootstrap completes does the logical Case bridge
create the actual semantic Process and call:

```text
runtimeHost.hostProcess(...)
```

which creates the real per-Case Process Context on the shared Test Tool Engine.

The resulting shape is therefore:

```text
logical Case
  -> direct-file resolver
  -> CoreBootstrap
       -> temporary Polyglot Context
       -> implicit temporary Engine
       -> rebuild Core/Prelude
       -> close temporary Context/Engine
  -> fresh semantic Process
  -> fresh Process Context on shared Test Tool RuntimeHost Engine
  -> re-materialize suite source
  -> validate discovery signature
  -> resolve one selected Test
  -> execute body
```

## Why this is a separate performance issue

The ratified Test Tool architecture requires fresh semantic Process isolation
per logical Case. The investigation found no corresponding requirement that
every Case must also reconstruct Core through a separate temporary
Context/Engine before its actual Process Context exists.

The earlier exact-execution architecture already separated an
already-selected Prelude/module environment from fresh Process creation.
Therefore the following is a valid independent optimization boundary without
changing the public `--jobs` contract or weakening fresh-Process isolation:

```text
reusable safe bootstrap/module state
        +
fresh semantic Process / actual Process Context per Case
```

The current redundant work has a structurally negative performance direction:
it performs additional Core evaluation, Context creation, Engine creation and
teardown for every Case. Its quantitative cost is not yet measured.

## Bounded conclusions

```text
CURRENT_SCALING_CEILING=CONFIRMED_BY_EXISTING_OBSERVATION
PERF014_ADMISSION_CAP_AT_8=FALSIFIED
PERF014_SPECIFIC_ACTIVE_CASE_CONCURRENCY_LOSS=FALSIFIED

CURRENT_LOGICAL_RUNNER_WORK_CONSERVING=YES
HOST_SUBMISSION_SECOND_CAP_FOUND=NO
ACTOR_POOL_FIXED_AT_8=NO
GLOBAL_PROTOS_CONTEXT_EXECUTION_LOCK_FOUND=NO

PER_CASE_CORE_BOOTSTRAP=CONFIRMED
PER_CASE_TEMPORARY_POLYGLOT_CONTEXT=CONFIRMED
PER_CASE_TEMPORARY_ENGINE=CONFIRMED
INTRODUCED_WITH_TOOL009_LOGICAL_CASE_EXECUTION=YES

SEMANTIC_REQUIREMENT_FOR_EXTRA_PER_CASE_BOOTSTRAP=NOT_FOUND
PERFORMANCE_DIRECTION_OF_REDUNDANT_BOOTSTRAP=NEGATIVE
PERFORMANCE_MAGNITUDE=UNMEASURED

PERF018_ALLOCATED=YES
PERF018_ISSUE=guillermomolina/protos#731

KNOWN_HIGH_SCALING_REVISION=NOT_ESTABLISHED
FIRST_BAD_SCALING_REVISION=NOT_ESTABLISHED
PERF018_IS_ROOT_CAUSE_OF_PERF017=NOT_CLAIMED
```

## PERF017 continuation

PERF017 remains responsible for the deeper scaling question. The next useful
runtime discriminator should distinguish the work performed by admitted Case
threads from waiting, compiler/JIT activity, GC/allocation pressure and
Process/Context lifecycle.

A JFR/thread-state/compiler/GC capture at a high explicit `--jobs` value is a
suitable next discriminator because the admission and fixed-eight-pool
hypotheses are already substantially narrowed.

## PERF018 implementation boundary

PERF018 / #731 owns only the semantics-preserving removal of the redundant
per-Case Core bootstrap and temporary Engine/Context work.

It must preserve:

- one fresh semantic Process per logical Case;
- one actual Process Context per Case under the Test Tool RuntimeHost;
- suite source re-materialization inside that fresh Process;
- discovery-signature stability validation;
- exact selected-Test resolution;
- no live Test/body Closure crossing Process boundaries;
- case-private stdout/stderr;
- deterministic result/report behavior;
- work-conserving global `--jobs N` scheduling;
- existing Process/Actor/Task semantics and deterministic teardown.

Before/after timing is required to quantify the gain, but the measurement is
not a prerequisite for recognizing that the additional per-Case bootstrap is
real work.

## Specification effect

```text
NORMATIVE_SPECIFICATION_CHANGE=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
PUBLIC_TEST_TOOL_POLICY_CHANGE=NO
```

Any implementation attempt that discovers a genuine semantic or durable
platform choice must stop at that boundary and route the choice through the
normal Dxxx/PLATxxx process rather than silently changing behavior.
