# PERF010-A — Current-25.4 hot-root lifecycle and compiled-shape discriminator specification

Date: 2026-09-30

## Scope

This record retains the investigation-only result for PERF010-A / guillermomolina/protos#691 after the post-PERF010-B residual causal re-evaluation selected the current-25.4 compiled form of guest-call-bearing hot roots as the strongest remaining causal boundary.

No build, test, benchmark, Docker run, repository program, product edit, patch, commit in guillermomolina/protos, or observable Protos semantic change was performed by the investigation.

This record does not reopen PERF010-B, does not authorize PERF010-B Step 4, does not reinterpret historical PERF014/PERF015/PERF016 measurements, does not reactivate PERF011, and does not trigger PERF020 implementation.

## Evidence identity

```text
WORK_ITEM=PERF010-A/#691
PARENT=PERF010/#680
SLICE=POST_PERF010_B_25_4_HOT_ROOT_COMPILER_LIFECYCLE_DISCRIMINATOR
TYPE=INVESTIGATION

CURRENT_PROTOS_HEAD=b72778ca446b602f33af5027a0ed28ab788b39ce
CURRENT_PROTOS_VERSION=0.3.119-SNAPSHOT
CURRENT_BENCHMARK_HEAD=ac59110d23cb4724e4aa438a2a5781aaf1b31a77
PROJECT_DOCS_BASE_REVISION=9c0868834982bda24016a35d62ab84d5b9eceda1

CURRENT_TOOLCHAIN=25.4.4.1.1
JDK_VERSION=25.0.4.1.1
JVMCI=25.4-b23
EXPECTED_RUNTIME=com.oracle.truffle.runtime.hotspot.HotSpotTruffleRuntime

HEAD_RUNTIME_RELEVANT_DRIFT_SINCE_REEVALUATION=NO
```

The current Protos HEAD is exactly the revision used by the preceding re-evaluation. The current benchmark HEAD is likewise unchanged. There is therefore no intervening runtime/product implementation drift to reconcile before the next slice.

## External 25.4 compiler facts

The investigation used current official GraalVM documentation only to answer the bounded diagnostic question.

GraalVM 25.4 release notes state that guest-language inlining uses Graal IR call-site frequencies rather than runtime direct-call counters.

Official references:

- https://www.graalvm.org/release-notes/25.4/
- https://www.graalvm.org/jdk25/graalvm-as-a-platform/language-implementation-framework/Options/
- https://www.graalvm.org/jdk25/graalvm-as-a-platform/language-implementation-framework/Optimizing/

Current facilities relevant to the discriminator include:

```text
engine.TraceCompilation
engine.TraceCompilationDetails
engine.TraceAssumptions
engine.TraceCompilationPolymorphism

compiler/engine Inlining
InliningExpansionBudget
InliningInliningBudget
InliningRecursionDepth

TraceInlining
TraceInliningDetails
TraceMethodExpansion
TraceNodeExpansion
TracePerformanceWarnings
NodeSourcePositions

Graal graph dumping for Truffle compilation
```

`TraceCompilation` exposes successful compilation and the reached tier; `TraceCompilationDetails` exposes queue/start/completion lifecycle. Invalidation and assumption tracing can establish loss of optimized state. Method/node expansion traces and Graal graph dumps provide direct compiled-shape evidence after partial evaluation rather than inferring elimination from source shape or sampled stacks.

## A. Baseline

```text
CURRENT_PROTOS_HEAD=b72778ca446b602f33af5027a0ed28ab788b39ce
CURRENT_PROTOS_VERSION=0.3.119-SNAPSHOT

CURRENT_BENCHMARK_HEAD=ac59110d23cb4724e4aa438a2a5781aaf1b31a77

CURRENT_TOOLCHAIN=25.4.4.1.1

HEAD_RUNTIME_RELEVANT_DRIFT_SINCE_REEVALUATION=NO

DRIFT_DETAIL=
  Protos and protos-benchmarks remain at the exact checkpoint revisions.
  No new product/runtime delta must be attributed before this discriminator.
```

## B. Existing evidence/harness suitability

```text
CURRENT_HARNESS_SUFFICIENCY=
  REQUIRES_BOUNDED_HARNESS_CHANGE

REUSABLE_COMPONENTS=
- PERF010-A Perf010aSourceIdentityInstrument
- PERF010-A Perf010aSourceIdentityInstrumentProvider
- PERF010-A Perf010aSourceIdentitySmokeDriver
- sourceName/startOffset/endOffset/line/text/rootNodeClassName identity records
- PERF010-B Step-0 lifecycle interpretation model
- PERF013 compiler-gate interpretation model
- DIST006-D current-25.4 build/runtime-probe lineage
- PERF016 exact revision/version/toolchain verification
- PERF016 correctness/source-identity discipline
- existing CPU/network diagnostic isolation conventions

MISSING_CAPABILITY=
- current-25.4 entry point joining source identity to compiler lifecycle
- normalized per-root compilation chronology
- invalidation/recompilation classification
- current-25.4 compiled-graph capture/indexing
- compiled-shape classification for the named candidate machinery
```

The existing PERF010-A source-identity path cannot be reused unchanged because its published configuration and Docker contract are historical 25.3-era infrastructure. The semantic mechanism is reusable; the current executable wiring is not.

A new measurement design is not required. The required evidence types and source identity model already exist in repository lineage and current Truffle diagnostics.

## C. Exact future discriminator

```text
DISCRIMINATOR_READY=YES

WORKLOADS=
- shared repeat/control driver
- micro/closure-call
- micro/method-call
- runtime/monomorphic-dispatch

ROOT_IDENTITY_METHOD=
  SHA-256(source bytes)
  + sourceName
  + [startOffset,endOffset)
  + startLine
  + exact SourceSection text
  + rootNodeClassName
  + logical root role

  Compiler/root numeric IDs are run-local correlation keys only and MUST NOT
  be used as cross-build identity.

LIFECYCLE_SIGNALS=
- TraceCompilation
- TraceCompilationDetails
- tier reached
- exact compiler failure/permanent-bailout reason and stack
- TraceAssumptions / invalidation evidence
- TraceCompilationPolymorphism where specialization replacement is relevant
- TraceInlining / TraceInliningDetails
- ordered opt queued/start/done/fail/invalidation chronology
- pending or unfinished relevant compilation state at process termination

COMPILED_SHAPE_SIGNALS=
- TraceMethodExpansion
- TraceNodeExpansion
- TracePerformanceWarnings
- NodeSourcePositions
- Graal Truffle graph dump
- graph inspection after TruffleTier
- graph inspection after PartialEscape or equivalent post-escape-analysis point

NORMAL_VS_COMPILATION_FALSE_REQUIRED=NO

RATIONALE=
  A/B/C/D is determined first by compiler lifecycle and optimized graph shape.
  Compilation=false timing is not required for the smallest first discriminator.
  A bounded normal-vs-Compilation=false comparison may be added later only if a
  lifecycle result needs runtime-cost correlation.
```

The discriminator must source-correlate the semantic and helper Bytecode roots belonging to the repeat driver, callback/operation, Closure call, method call and monomorphic-dispatch paths. Root numbers are not stable identity.

For each relevant root the future measurement must emit:

```text
ROOT_SOURCE_IDENTITY=<stable identity>

OPTIMIZATION_STATE=
  OPT_DONE |
  OPT_FAILED |
  NOT_TRIGGERED |
  INCONCLUSIVE

TIER_REACHED=<observed tier(s)>

FAILURE_OR_BAILOUT=
  <exact reason/stack> |
  NONE

INVALIDATION_OR_RECOMPILATION=
  PRESENT |
  ABSENT |
  INCONCLUSIVE

STABLE_FINAL_OPTIMIZED_STATE=
  YES |
  NO |
  INCONCLUSIVE
```

The lifecycle parser must distinguish at least:

```text
never compiled
compiled only first tier
compiled final tier
permanent bailout
temporary failure/retry
compiled then invalidated
compiled then recompiled successfully
compiled then replaced by generic specialization
stable optimized state
```

A stable final optimized state requires a successful final optimized compilation, no later relevant invalidation/failure, and no unresolved/pending lifecycle event at shutdown. Ambiguous source/root correlation is `INCONCLUSIVE`, never a PASS.

## D. Compiled-shape discrimination

If all required roots reach a stable optimized state, the same evidence unit must determine whether implementation-only machinery physically survives the optimized path.

Candidate families:

```text
activation creation / ProtosActivation
execution-context creation/materialization
argument-array/list transport
return-home/control-state carriers
captured lexical/environment machinery
semantic-wrapper -> helper transition
generic selection/classification
Context-local plan/cache lookup
host Map/ConcurrentMap/Optional/String classification machinery
continueAt / Bytecode continuation/interpreter machinery
other current-HEAD machinery exposed by the graph
```

For each family the future measurement emits:

```text
SURVIVES_OPTIMIZED_GRAPH=YES|NO|INCONCLUSIVE
```

The evidence rule is direct compiler visibility:

- allocations/types/fields/invokes still present after partial evaluation/escape analysis support `YES`;
- graph evidence proving the candidate disappeared supports `NO`;
- missing, ambiguous or insufficiently attributable graph evidence is `INCONCLUSIVE`.

Source-level allocation is not proof of survival. Absence from a sampled stack is not proof of elimination.

## E. Causal classifications

```text
CAN_DISTINGUISH_COMPILATION_FAILURE=YES
CAN_DISTINGUISH_LIFECYCLE_INSTABILITY=YES
CAN_DISTINGUISH_SURVIVING_MACHINERY=YES
CAN_DISTINGUISH_MATURE_COMPILED_FORM=YES

UNRESOLVED_AFTER_PROPOSED_DISCRIMINATOR=NONE
```

Routing:

```text
A. COMPILATION_FAILURE
   One or more required roots permanently fail/bail out without a later
   successful final optimized state.

B. COMPILATION_LIFECYCLE_INSTABILITY
   Required roots compile but invalidate/recompile, replace specialization, or
   otherwise fail to retain stable optimized state.

C. COMPILED_BUT_EXPENSIVE_SURVIVING_MACHINERY
   Required roots stably compile and direct graph evidence retains material
   implementation-only machinery.

D. MATURE_COMPILED_FORM_WITH_RESIDUAL_SEMANTIC_COST
   Required roots stably compile and the investigated implementation-only
   machinery is removed/elided.
```

The measurement result selects the next causal boundary. No particular activation, Context, capture, helper-dispatch, `continueAt`, generic-lookup, or host-collection mechanism is selected in advance.

## F. Next slice

```text
NEXT_STATE=IMPLEMENTATION_READY

NEXT_SLICE=
  CURRENT_25_4_HOT_ROOT_LIFECYCLE_AND_COMPILED_SHAPE_HARNESS

NEXT_SLICE_KIND=IMPLEMENTATION

NEXT_IMPLEMENTATION_REPOSITORY=
  guillermomolina/protos-benchmarks

EXPECTED_IMPLEMENTATION_COMPONENTS=
- dedicated current-25.4 discriminator configuration
- dedicated discriminator runner
- current-25.4 diagnostic-image wiring using the DIST006-D toolchain contract
- reuse of the existing PERF010-A source-identity mechanism
- lifecycle trace parser/normalizer
- graph-dump capture plus manifest/index
- bounded runner/parser tests
- Makefile/BENCHMARKING.md wiring
```

Predeclared harness acceptance before any real measurement:

```text
1. exact b72778ca446b602f33af5027a0ed28ab788b39ce / 0.3.119-SNAPSHOT
   identity is fail-closed;
2. exact 25.4.4.1.1 / JDK 25.0.4.1.1 / JVMCI 25.4-b23 runtime identity
   is fail-closed;
3. stable SourceSection identity exists for every required semantic/helper root
   without hard-coded root numbers;
4. the selected current-25.4 lifecycle/graph options are accepted;
5. raw lifecycle logs and isolated graph outputs are retained per workload;
6. parser fixtures cover every required lifecycle class;
7. every stably compiled required root has attributable compiled-shape evidence
   or is explicitly INCONCLUSIVE;
8. implementation validation proves the harness contract but does not fabricate
   a compiler-state measurement result.
```

The implementation slice itself does not execute the authoritative discriminator and does not select a product optimization.

## G. Relationship to PERF020

The lifecycle/compiled-shape discriminator and PERF020 are separate consumers with different evidence contracts.

```text
CURRENT_DISCRIMINATOR=
  one exact Protos revision
  compiler lifecycle + compiled shape
  diagnostic evidence
  no authoritative timing comparison

PERF020=
  exact CONTROL + INTERVENTION revisions
  QUICK / REFERENCE timing comparator
  before/after performance evidence
```

They should not be merged into one harness. The implementation should, however, reuse or extract only the smallest genuinely common infrastructure needed for exact revision build/probe, identity validation, CPU/process orchestration and artifact retention, so PERF020 does not later duplicate those mechanics.

This is not authorization to implement PERF020 now, and it does not change PERF020's retained decision that reference timing is serial initially. Independent diagnostic JVM/process orchestration in this compiler discriminator must not be interpreted as evidence that retained timing measurements may be parallelized.

```text
PERF020_IMPLEMENTATION_DEFERRED=YES
PERF020_TRIGGER_STATE=NOT_READY
```

## H. Product and adjacent work state

```text
PRODUCT_INTERVENTION_SELECTED=NO

CONTROL_REVISION=NOT_YET_SELECTABLE
CONTROL_VERSION=NOT_YET_SELECTABLE

PERF011_REACTIVATE_AFTER_THIS_INVESTIGATION=NO
PERF011_RATIONALE=
  No current-25.4 representation/compiler-visibility result has yet been
  measured. Reactivate only if the discriminator establishes a material
  representation surface or a concrete intervention needs PERF011 ownership.

PERF010_B_REOPENED=NO
PERF010_B_STEP4_AUTHORIZED=NO
HISTORICAL_EVIDENCE_MUTATION=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
COMMANDS_EXECUTED=NO
```

## Final selection

Exactly one next action is selected:

```text
Implement CURRENT_25_4_HOT_ROOT_LIFECYCLE_AND_COMPILED_SHAPE_HARNESS
in guillermomolina/protos-benchmarks.

Do not modify guillermomolina/protos.
Do not implement PERF020.
Do not execute the authoritative lifecycle/compiled-shape measurement as part of
the implementation slice.
```
