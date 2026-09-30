# PERF020 — Reusable iterative comparator investigation

Date: 2026-09-30

## Scope

This record retains the investigation-only result for PERF020 / guillermomolina/protos#741.

The investigation did not execute builds, tests, benchmarks, Docker, project programs, or repository-local commands, and did not modify Protos product or benchmark source.

PERF020 is performance-infrastructure work. It does not reopen PERF010-B, does not authorize PERF010-B Step 4, and does not reinterpret the retained PERF016 timing result.

## Investigation baseline

```text
WORK_ITEM=PERF020/#741

PROTOS_HEAD_AT_INVESTIGATION=7665ac7951200b68e672e6a63fbd201a5d4f6415
PROTOS_BENCHMARKS_HEAD_AT_INVESTIGATION=ac59110d23cb4724e4aa438a2a5781aaf1b31a77
PROJECT_DOCS_BASE_REVISION=781c34ee9ddcaacf84b3d62864f62c6a3c8dfddd

TRIGGERING_HARNESS=PERF016
TRIGGERING_HARNESS_REVISION=ac59110d23cb4724e4aa438a2a5781aaf1b31a77
REFERENCE_TIMED_UNITS=64
REFERENCE_BLOCK_ORDER=A,B,A,B
REFERENCE_WARMUP=120
REFERENCE_STEADY=100
REFERENCE_CPU_POLICY=one pinned logical CPU
```

The triggering PERF016 result remains:

```text
PERF016_STATUS=CLOSED_COMPLETE
PERF010_B_STATUS=CLOSED_COMPLETE
STEP_3_TIMING_CLASS=ESSENTIALLY_UNCHANGED
STEP3_NEXT_ROUTING=STOP_AND_REEVALUATE
STEP4_AUTHORIZED=NO
STEP4_ALLOCATED=NO
```

## Current-state diagnosis

PERF016 already reuses DIST006-D for product image construction, toolchain/runtime validation, Docker execution, correctness primitives, exact published-harness identity and manifest support.

However, the two-revision comparator still owns a large amount of repeated framework machinery:

- endpoint and product-version validation;
- workload-source identity and workload-control materialization;
- block scheduling;
- timing-unit orchestration;
- direct and paired effect calculation;
- stationarity and order-effect analysis;
- raw-evidence verification;
- evidence rendering;
- atomic output publication;
- CLI and Makefile plumbing.

The current PERF016 runner is approximately 2600 lines and its dedicated test module approximately 1300 lines. This is structural evidence of duplication, not a claim that LOC alone determines cost.

PERF014 provides direct historical evidence for the same problem: its runner explicitly ports generic process/build/timing/stationarity helpers from earlier runners because the existing standalone-runner structure did not expose them as reusable infrastructure.

The normal before/after comparison therefore does not require genuinely experiment-specific framework code. What normally varies is the endpoint pair, workload policy, work-item metadata and any genuinely special derived metric.

## Reusable comparator result

```text
REUSABLE_COMPARATOR_FEASIBLE=YES
```

Recommended prospective boundary:

```text
GENERIC_COMPARATOR_OWNS=
  exact endpoint validation
  product image build/acquisition and probe
  toolchain/runtime identity
  workload source identity
  correctness-before-timing
  CPU topology and deterministic selection
  A/B scheduling
  timed-unit execution
  raw sample collection
  standard direct/paired effects
  reference stationarity/order diagnostics
  evidence identity/rendering/manifest/atomicity
  CLI

WORKLOAD_POLICY_OWNS=
  workload ids
  canonical source paths
  expected observable results
  optional workload-control transforms
  variants
  operation count
  reference warmup/steady policy
  reference block order
  standard analysis views

WORK_ITEM_METADATA_OWNS=
  PERF identifier
  Issue/reference provenance
  interpretation/routing fields
  narrative
```

A new PERF identifier must not by itself imply a new runner, config schema, Docker harness, analysis implementation or duplicated framework test suite.

## Endpoint identity policy

```text
CONTROL_REVISION=required exact immutable 40-character SHA runtime input
INTERVENTION_REVISION=required exact immutable 40-character SHA runtime input

VERSION_POLICY=
  supplied expected version
  + observed version from each exact built product
  + fail closed on mismatch

HARNESS_REVISION=
  exact clean published immutable SHA for retained reference evidence

FLOATING_MAIN=FORBIDDEN
```

This preserves the current reproducibility contract while making the endpoint pair reusable input rather than runner constants.

## Quick mode

```text
QUICK_MODE_FEASIBLE=YES

QUICK_MODE_POLICY=
  blocks=2
  block_order=A,B
  warmup=120
  steady=20
  workload_policy=smallest predeclared hypothesis-relevant workload subset
  variants=canonical plus paired workload-control where the selected policy defines it
  correctness=required for every timed role/variant/workload
  cpu_policy=one deterministic physical-core representative
  execution=serial
  retained_evidence=NO
  authoritative_performance_claim=NO
```

The investigation deliberately does not shorten the established 120-iteration warmup without measurement evidence that a shorter warmup is adequate. The inexpensive mode instead reduces the number of workloads, blocks and steady samples.

For a normal one-workload hypothesis under the current four-workload PERF016 shape:

```text
REFERENCE_TIMED_UNITS=64
QUICK_TIMED_UNITS=8

REFERENCE_WARMUP_PLUS_STEADY_EXECUTIONS=14080
QUICK_WARMUP_PLUS_STEADY_EXECUTIONS=1120

MEASUREMENT_LOOP_WORK_REDUCTION_APPROX=12.6x
```

That ratio is not a wall-clock prediction because product build, JVM/container startup and other fixed costs remain.

Quick mode may reject a predeclared large/material directional hypothesis when both A and B fail to show the predicted direction and no gross drift condition invalidates the screen. Quick mode must not accept a final performance claim or substitute for retained reference evidence.

## Build reuse

Exact product images may be reused across iterative quick runs when keyed and revalidated by:

```text
exact product SHA
+ exact build-profile identity
+ exact toolchain identity
```

This does not change the timed evidence region: image construction and container startup are already outside the per-sample timing boundary.

No recommendation is made to reuse one JVM or one Protos Context across independent timed evidence units. That would alter the current Evidence Unit and requires its own causal/methodology investigation.

## CPU topology and reference parallelism

Current PERF016 conflates two separate facts operationally:

1. each timed unit is pinned to one logical CPU; and
2. all independent timed units are executed serially.

CPU affinity does not itself require global serialization.

Existing PERF001-F infrastructure already demonstrates repository-owned Linux topology discovery for:

- process allowed affinity;
- package/socket;
- physical core;
- logical CPU;
- SMT sibling set;
- deterministic representative selection.

The recommended initial reference policy nevertheless remains serial.

```text
REFERENCE_PARALLELISM_DECISION=SERIAL_ONLY

REFERENCE_WORKER_COUNT=one timed worker

PHYSICAL_CORE_SELECTION_POLICY=
  inspect current allowed affinity
  group logical CPUs by package/core identity
  choose one deterministic representative
  use the same logical CPU for all control/intervention timed units
  retain the full discovered topology

SMT_SIBLING_POLICY=
  do not allocate another benchmark worker to the sibling
  record sibling topology
  prefer externally isolated/exclusive core placement when available
  do not claim host-wide sibling isolation unless actually provided

CORE_BIAS_CONTROL=
  same logical CPU for the whole comparison

SHARED_RESOURCE_INTERFERENCE_POLICY=
  no comparator-created concurrent timed workers

HOST_QUIETNESS_POLICY=
  enforce comparator-owned affinity/network/diagnostic exclusions
  inspect and record topology and readable frequency/boost state
  treat unrelated host activity, global thermal state and package-wide contention
  as operator/environment controls unless explicitly isolated
```

Parallel reference blocks are not rejected permanently. They are deferred because different physical cores still share package/cache/memory/power/turbo/thermal resources and because core-to-core differences become a new causal variable. A future bounded serial-versus-crossed-two-core validation can revisit this once the reusable comparator exists.

## External methodology used

The repository evidence contract remains authoritative. External sources were used only to evaluate CPU/topology/interference implications:

- Linux `sched_setaffinity(2)` documentation — explicit CPU-affinity semantics and migration/cache effects:
  https://man7.org/linux/man-pages/man2/sched_setaffinity.2.html
- Linux x86 topology documentation — package/core/thread and sibling topology:
  https://www.kernel.org/doc/html/latest/arch/x86/topology.html
- Linux CPUFreq documentation — frequency/boost behavior and package/load interaction:
  https://docs.kernel.org/admin-guide/pm/cpufreq.html
- Google Benchmark variance guidance — core, turbo/frequency, SMT, cache, NUMA and scheduler variance:
  https://github.com/google/benchmark/blob/main/docs/reducing_variance.md

These sources do not override the Protos evidence-unit contract.

## Expected workflow change

```text
BEFORE:
  harness_work=new bespoke comparator/config/test stack
  quick_feedback=no useful non-retained timing tier
  retained_reference=full 64-unit serial comparison

AFTER:
  harness_work=existing comparator plus endpoint/work-item inputs
  new_workload_family=small declarative policy when genuinely required
  quick_feedback=small non-retained directional screen
  retained_reference=full reproducible serial comparison
```

The expected improvement is primarily a workflow-class change:

- routine comparator implementation work approaches zero for an existing workload policy;
- normal one-workload quick measurement reduces timed units from 64 to 8;
- retained reference evidence initially keeps the current serial measurement discipline.

No precise wall-clock duration is asserted without measured evidence.

## Implementation readiness

```text
HISTORICAL_EVIDENCE_MUTATION=NO
SEMANTIC_CHANGE=NO

IMPLEMENTATION_SLICE_READY=YES
IMPLEMENTATION_REPOSITORY=guillermomolina/protos-benchmarks

FIRST_IMPLEMENTATION_SLICE=
  one prospective generic two-revision comparator
  + one existing common-driver workload policy
  + serial QUICK A,B
  + serial REFERENCE A,B,A,B
  + reuse DIST006-D build/runtime machinery
  + reuse PERF001-F topology concepts
  + one generic comparator test suite
  + no historical harness migration
  + no reference parallel scheduler yet
```

Expected prospective implementation surfaces include a generic comparator runner/core, a small reusable CPU-topology helper where needed, stable workload-policy data, one generic test module, Makefile plumbing and benchmarking documentation.

Historical PERF014, PERF016 and DIST006-D retained harnesses/results remain unchanged.

## Owner scheduling decision

The project owner explicitly chose to defer PERF020 implementation until the next concrete performance intervention is ready for measurement.

The intended ordering is:

```text
1. establish the next bounded performance intervention
2. preserve its exact pre-intervention CONTROL revision
3. implement and validate/publish the intervention
4. preserve the exact INTERVENTION revision
5. implement PERF020 reusable comparator as the measurement infrastructure
6. run QUICK
7. run REFERENCE only when warranted
```

This makes the next real measurement the first consumer of the reusable comparator and avoids implementing the abstraction without an immediate concrete use.

```text
PERF020_INVESTIGATION=COMPLETE
PERF020_IMPLEMENTATION_DEFERRED=YES
PERF020_IMPLEMENTATION_TRIGGER=NEXT_PERF_READY_FOR_TIMING
PERF020_STATUS=PAUSED_PENDING_MEASUREMENT_CONSUMER

STEP4_RELATION=NONE
PERF010_B_REOPENED=NO
```

The exact next performance implementation is not selected by PERF020. Selection remains governed by the active performance work/evidence after the PERF010-B stop gate.
