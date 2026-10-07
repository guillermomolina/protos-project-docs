# PERF010-A — stable-identity timing harness readiness

Status: retained investigation evidence establishing that the currently published PERF010-A benchmark harness cannot execute the next clean stable-identity timing Evidence Unit without a bounded harness implementation slice. No Protos product change is required by this finding.

## Evidence identity

```text
PROTOS_REPOSITORY=guillermomolina/protos

CONTROL_PROTOS_REVISION=
  2b3a88389da7228caed231a90b14091cf2841115

INTERVENTION_PROTOS_REVISION=
  3e8e6b565c95eb5098c2168d241536ba13ad19e9

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
BENCHMARK_REVISION=
  e0bbf213c8140491f91913712cbf6d9cd0270e0b

PREVIOUS_PROJECT_RECORD_REVISION=
  6aafd29807e9e566c0ec1e74d4ff7e667a840333
```

The benchmark revision above is the current published `main` revision observed during this investigation.

The product comparison was revalidated before inspecting harness readiness:

```text
CONTROL -> INTERVENTION
AHEAD_BY=2
BEHIND_BY=0
TOTAL_COMMITS=2
```

The two commits are:

```text
3a4afc27a96ae4efd6f3f0c71bc0990d5d102e30
  PERF010-A: add prepared Context-owned target specialization for PrepareSendArguments

3e8e6b565c95eb5098c2168d241536ba13ad19e9
  PERF010-A: key fastOrdinarySend cache on stable Closure definition identity
```

The complete changed-file set between control and intervention is exactly:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf010APreparedTargetSpecializationTest.java
```

## Investigation boundary

This was an investigation/evidence-only slice.

No tracked file in `guillermomolina/protos`, `guillermomolina/protos-benchmarks`, or `guillermomolina/protos-project-docs` was modified as part of the harness-readiness investigation itself.

The preceding compiler-lifecycle record established:

```text
FAST_SPECIALIZATION_CACHE_CHURN=REMOVED
GENERIC_REPLACING_SPECIALIZATION_ACTIVATED=NO
CALLER_HELPER_PERMANENT_BAILOUT=REMOVED
COMPILER_LIFECYCLE_STABLE=YES
TIMING_READY=YES
```

Therefore this investigation did not reopen PIC churn, compiler-lifecycle attribution, `continueAt`, or another performance hypothesis. Its only question was whether the existing retained benchmark harness can execute the required clean timing Evidence Unit as specified.

## Existing harness capability

The low-level timing mechanism already supports the required longer warmup.

`docker/protos-perf010a/Perf010aTimingDriver.java` accepts:

```text
<source> <expected-integer> <warmup> <steady>
```

and validates the supplied warmup/steady counts directly.

The existing Python harness also already has reusable lower-level machinery for:

- JFR-free timing;
- fresh Docker container / Java process per timed unit;
- `--network none`;
- explicit `--cpuset-cpus`;
- canonical/control source construction;
- correctness checking on every timed execution;
- raw ordered `warmup_ns` and `steady_ns` retention;
- median/MAD/p95/min/max summaries; and
- deterministic A/B counterbalanced no-op discrimination blocks.

In particular, `run_discrimination_blocks(...)` already receives `warmup` and `steady` as parameters, so the Java driver and the fundamental timed-unit mechanism do not require redesign.

## Phase 1 blocker: public retained path is pinned to warmup=20

The published no-op discrimination configuration remains intentionally historical:

```text
config/perf010a-0.json

operation_count=10000
warmup_iterations=20
steady_iterations=100
block_order=A,B,A,B
```

The retained public validation path explicitly asserts:

```python
assert cfg["warmup_iterations"] == 20
assert cfg["steady_iterations"] == 100
```

and `reference_discrimination()` passes those exact config values to the timed blocks.

The CLI exposes only:

```text
discrimination-validate
discrimination-smoke
discrimination-reference
```

There is no supported retained-execution override that changes only:

```text
warmup: 20 -> 120
```

while leaving the historical config itself untouched.

Therefore the required Phase 1 measurement-system admission cannot be executed through the existing published entrypoint without either:

1. modifying the historical config; or
2. adding bounded harness wiring for an explicit warmup-120 Evidence Unit.

The first option would destroy the historical reference contract and is rejected.

## Additional Phase 1 evidence gaps

Three additional requirements of the new Evidence Unit are not retained by the current public runner.

### Stationarity diagnostics

The existing harness retains raw `steady_ns`, but it does not derive or publish first-quarter versus last-quarter diagnostics for each 100-sample steady timed unit.

The required raw information already exists; the missing capability is deterministic derivation/retention of the stationarity view.

### Real Docker image identity

The current `raw.json` records the declared container/toolchain identity and benchmark/product revisions, but it does not record the actual built Docker image identity obtained from the local Docker engine.

The next Evidence Unit requires the real built-image identity in addition to the declared base image and revision labels.

### Timed-unit output visibility plus retention

`timing()` executes each timed unit with captured stdout/stderr:

```python
run(command, capture=True, check=False)
```

and `run(..., capture=True)` uses pipes.

As a result, successful timed-unit stdout/stderr is not streamed to the human while the process runs. The runner prints only its own `TIMING BEGIN` / `TIMING PASS` progress around the hidden child output.

The next Evidence Unit requires failures and relevant progress to remain visible while complete raw output is retained. The current behavior does not satisfy that requirement.

## Phase 2 blocker: current causal runner is not a two-revision baseline comparator

The existing causal `reference()` path is built around one configured Protos revision plus two image variants:

```text
baseline
ablation
```

Both image builds use the same:

```python
cfg["protos_revision"]
```

and the ablation side is distinguished by an in-build patch.

That model cannot represent the required clean comparison:

```text
CONTROL:
  2b3a88389da7228caed231a90b14091cf2841115
  VARIANT=baseline

INTERVENTION:
  3e8e6b565c95eb5098c2168d241536ba13ad19e9
  VARIANT=baseline
```

with no historical ablation patch and with the product revision as the only intended runtime-code difference.

The existing causal runner also does not expose the required control/intervention counterbalanced block sequence for this two-product-revision comparison.

Therefore Phase 2 requires additional bounded harness support if and only if Phase 1 later establishes an interpretable measurement window.

## Result

```text
PERF010A_STABLE_IDENTITY_TIMING_HARNESS_READINESS=ESTABLISHED

CURRENT_RESEARCH_SLICE=COMPLETE

HARNESS_LIMITATION=ESTABLISHED

PHASE1_EXISTING_PUBLIC_ENTRYPOINT=INSUFFICIENT
PHASE2_EXISTING_PUBLIC_ENTRYPOINT=INSUFFICIENT

TIMING_DRIVER_CHANGE_REQUIRED=NO
DOCKERFILE_CHANGE_REQUIRED=NO
PROTOS_PRODUCT_CHANGE_REQUIRED=NO

MISSING_PHASE1_WIRING=
  warmup=120 retained execution path
  + first-quarter/last-quarter stationarity diagnostics
  + actual Docker image identity
  + visible-and-retained timed-unit stdout/stderr

MISSING_PHASE2_WIRING=
  clean two-product-revision baseline-vs-baseline paired timing
  + counterbalanced CONTROL/INTERVENTION blocks

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

## Next slice

The next slice is implementation in:

```text
guillermomolina/protos-benchmarks
```

It should remain deliberately narrow.

The next implementation should add only the Phase 1 measurement-system admission path needed to execute the retained no-op experiment with:

```text
WARMUP=120
STEADY=100
OPERATION_COUNT=10000
BLOCK_ORDER=A,B,A,B
```

while preserving `config/perf010a-0.json` as the historical 20/100 reference.

That implementation should additionally retain the Phase 1 stationarity diagnostics, actual Docker image identity, and visible-plus-retained timed-unit output required by the Evidence Unit.

It should not execute or automatically authorize Phase 2.

After the Phase 1 evidence is collected, PERF010-A must classify:

```text
WARMUP_120_STABILITY_HYPOTHESIS=
  SUPPORTED | WEAKENED | INCONCLUSIVE

MEASUREMENT_GATE=
  OPEN | CLOSED | INCONCLUSIVE
```

Only an explicit `MEASUREMENT_GATE=OPEN` should make the separate two-product-revision Phase 2 harness path actionable.

The additional Phase 2 comparator gap is retained above now so that no later investigation needs to rediscover it, but implementing that comparator is intentionally deferred until the Phase 1 gate result makes it useful.
