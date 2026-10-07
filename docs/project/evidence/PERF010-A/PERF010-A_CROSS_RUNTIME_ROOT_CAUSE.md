# PERF010-A — Cross-runtime root-cause investigation

Status: ARCHITECTURAL ROOT CAUSE ESTABLISHED; PERF010-A DOMINANCE NOT CLOSED

This durable, non-normative record retains the cross-runtime investigation of
PERF010 / #680 and its dominant-attribution child PERF010-A / #691. The purpose
was to stop treating local `continueAt`/allocation/map-copy costs as the default
next target and compare the published Protos hot path with the implementations
actually measured by the retained PERF004-A benchmark.

The investigation did not modify Protos, benchmarks, or benchmark evidence.

## Evidence identity

```text
CURRENT_PROTOS_REVISION=cd32d7d3228f88c6ebda65f357787a692b89c666
CURRENT_BENCHMARK_REVISION=25f5b52ce0feb074db1529c939b7d5aec1b5aa30
PREVIOUS_PROJECT_RECORD_REVISION=da6ece6e876b88c6ef9fcbceb71d28772643721d

PUBLISHED_PERF004A_EVIDENCE_REVISION=5e8ff21f966c6c506652eef79c684d8b286bb546
PUBLISHED_PERF004A_HARNESS_REVISION=60dbce7faf5510bd1bd6867a866aa7ca69c48637
PUBLISHED_PERF004A_PROTOS_REVISION=4a03efc15620b37b2e418b3df30b4a26486446ec

PERF004_B2D_EVIDENCE_REVISION=8e94423e938730e00614d4112288688120fd0549
PERF006_D3_EVIDENCE_REVISION=ecfa8fb3a23f5661524b7eb812e16e75c833ed7c
PERF008_EVIDENCE_REVISION=84d582fa75eecd759ba4d897fdcfcca58143ba58
```

The published cross-language comparison uses:

```text
Protos: GraalVM/Truffle 25.3.4.1, Java 25.0.4.1
Python: CPython 3.14.7
JavaScript: Node.js 24.20.0
```

It does **not** compare Protos with GraalPy or GraalJS. Any claim that the
published ratios are a same-GraalVM comparison is therefore rejected.

## Result

```text
PERF010_CROSS_RUNTIME_ROOT_CAUSE=ESTABLISHED
PUBLISHED_BENCHMARK_FOUND=YES
PRIMARY_COMPARISON_CLASSIFICATION=COMPARABLE_HOT_PATH

SHARED_ROOT_CAUSE_ACROSS_WORKLOADS=ESTABLISHED
MONOMORPHIC_DISPATCH_SPECIALIZES=PARTIAL
REPEATED_DEOPTIMIZATION_PRESENT=YES

TIER_A=generic composed-call preparation and insufficient early call-site specialization
TIER_B=String-keyed heap activation/lexical lookup and per-call activation/context materialization
TIER_C=local microcosts such as Optional/List.copyOf/duplicate map probes and continueAt-local work

SEMANTICALLY_REQUIRED_BY_PROTOS=NO
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
ABLATION_5=DEFERRED
WARMUP_20_TO_120_EXPERIMENT=DEFERRED
```

`PERF010_CROSS_RUNTIME_ROOT_CAUSE=ESTABLISHED` is an architectural conclusion,
not a claim that #691's stronger closure contract has been met. #691 still
requires attributable-fraction / causal-intervention evidence (or equivalent)
before `PERF010A_DOMINANT_CAUSE` can change to `ESTABLISHED`.

## Published performance gap

The four common-path workloads used by PERF010-A are algorithmically comparable
at the benchmark level: each executes the same basic 10,000-iteration recursive
shape and returns the same result, with startup/parsing/bootstrap outside the
steady timing interval.

Retained PERF004-A medians:

| workload | Protos ns | CPython ns | Protos/Python | Node ns | Protos/Node |
|---|---:|---:|---:|---:|---:|
| `micro/slot-read` | 49,378,637 | 1,580,361 | 31.25x | 116,250 | 424.76x |
| `micro/closure-call` | 67,611,903 | 1,767,596.5 | 38.25x | 87,400 | 773.59x |
| `micro/method-call` | 55,565,935.5 | 1,754,912 | 31.66x | 126,715.5 | 438.51x |
| `runtime/monomorphic-dispatch` | 54,842,150 | 1,759,496.5 | 31.17x | 123,245 | 444.98x |

The largest ratio in the full PERF004-A suite is
`algorithms/factorial/recursive` at 4,821.17x versus Node, but that very small
fixed program is not the primary diagnostic workload. `micro/closure-call` is
the primary diagnostic case because it exposes the shared invocation machinery
while retaining a 773.59x Node gap and 38.25x CPython gap.

The previously discussed informal "10,000x" figure is not present in retained
PERF004-A evidence.

## Protos hot-path structure

At the measured Protos revision, a stable Closure or method call does not enter
a known target directly. The common route is structurally:

```text
receiver/closure
  -> PrepareClosureCall / PrepareSendArguments
  -> ProtosValueLookup.lookup(...)
  -> selector / delegation resolution
  -> prepareImmediateMethodCall or standard Closure-call intrinsic
  -> ProtosActivation.for*Invocation(...)
  -> task/dynamic-control propagation
  -> finishPreparingComposedCallByImplementation(...)
       -> classify ensure / handle / while / Boolean callback
       -> classify Array / Bytes / ProcessArguments / Environment each
       -> classify IdentityMap / Map operations
       -> classify signal / Object.call / import runtime
  -> resolve execution plan
  -> PreparedClosureCall
  -> EnterClosureCall
  -> cached RootCallTarget guard
  -> DirectCallNode.call(...)
```

The final stable call is therefore specialized, but only after substantial
runtime-generic work. This is why the correct classification is
`MONOMORPHIC_DISPATCH_SPECIALIZES=PARTIAL`, not `NO`.

The same structural mechanism remains visible in current source at
`CURRENT_PROTOS_REVISION`; this record does not claim that every historical
optimizer symptom persists unchanged at current main.

## Compiler evidence at the measured revision

PERF006-D3 retained bounded `TraceCompilation` evidence for
`micro/method-call` at the same Protos revision used by PERF004-A.

The retained summary reports:

```text
successful_compilations=12
invalidation_markers=10727
failed_compilation_markers=6
bailout_markers=15
```

The retained log contains repeated semantic-root invalidations with
`Reason uncommon trap`, recompilation activity, and failed
`ProtosBytecodeRootNodeGen` compilation with:

```text
PermanentBailoutException: Too deep inlining, probably caused by recursive inlining.
```

The inlining trace reaches code unrelated to the logical monomorphic method
operation through the generic preparation route, including module/import
resolution and JDK infrastructure reached from that route. This establishes that
the generic invocation universe is not merely source-level abstraction: it can
remain compiler-visible deeply enough to destabilize or defeat optimization.

PERF004-B2-D independently records very large Truffle deoptimization counts for
the same revision and four canonical workloads, dominated by
`OptimizedCallTarget.doInvoke` / `transfer_to_interpreter` events.

The evidence does not assign an exact percentage of PERF004-A elapsed time to
these invalidations. They are a severe observed consequence of the architecture,
not an attributable-fraction measurement.

## Current-revision reconciliation

PERF008, at a later Protos revision, records zero Truffle deoptimizations for
the same four workloads. Therefore the historical invalidation storm must not
be generalized as a statement that current main still experiences the same
count or failure mode.

However, current steady-state stacks still retain common-path frames such as:

```text
PrepareSendArguments
PrepareClosureCall / PrepareClosureCallArguments
ProtosActivation.lookup
ProtosObjectValue.readLocalSlot
HashMap.getNode / putVal
String.hashCode
FrameExtensionsUnsafe.setObject
OptimizedDirectCallNode.call
```

This reconciles the historical and current evidence: the durable architectural
finding is not "Protos never compiles" or "all calls are indirect". It is that
substantial generic runtime machinery remains ahead of the stable direct call
and can survive into optimized execution.

## Comparison with the measured mature runtimes

### CPython 3.14.7

The measured CPython implementation maps locals and closure variables to indexed
frame/cell locations (`LOAD_FAST`, `LOAD_DEREF`) and provides adaptive
specialized attribute/call opcode families. Stable lexical positions therefore
do not require rediscovering a variable through String-keyed runtime maps on
every access.

### Node.js 24.20.0 / V8

The measured Node/V8 implementation attaches feedback to call sites and contains
compiler paths that specialize known/observed closure targets and inline suitable
small callees. The stable target is therefore represented at the call site early
enough for generic call machinery to be reduced or removed.

No claim is made that every call in the retained Node run was inlined; the
source-level architectural contrast is that V8 makes target stability an input
to call-site specialization, while Protos reaches its cached direct target only
after generic semantic preparation.

## Pareto interpretation

The cross-runtime comparison changes the priority order of PERF010-A evidence.
The previous micro-ablations remain valid evidence that local operations have
real cost, but they are now classified as secondary manifestations inside a
larger generic hot path.

```text
generic call preparation / late specialization = TIER_A
heap activation + String-key lexical lookup     = TIER_B
wrapper / Optional / List.copyOf / map probes   = TIER_C
```

This also explains why bounded A1/A3/A4 changes produced only small local
movements: removing one small cost does not collapse the surrounding generic
invocation pipeline.

## Relationship to the measurement-instability record

The no-op discrimination and measurement-instability records remain valid. They
establish that small percentage-level ablations cannot currently be interpreted
reliably without additional methodology work.

The cross-runtime investigation changes *priority*, not historical validity:

- do not run Ablation 5 now;
- do not run the warmup 20 -> 120 experiment merely to enable another small
  local ablation;
- retain those steps as deferred methodology work if a future small-effect
  attribution again requires them.

The stop condition for this investigation was met earlier: a single
architecture-level mechanism passed the Pareto plausibility gate. Continuing to
refine a measurement floor for another micro-ablation would not be the
highest-information next action.

## Next bounded investigation

Before selecting any production optimization, investigate one guarded
monomorphic ordinary-call fast path at current main.

The investigation should determine:

1. which stable facts can be cached/guarded before generic
   `prepareImmediateMethodCall` / `finishPreparingComposedCallByImplementation`;
2. which mutation/provenance/delegation changes must invalidate that fast path;
3. whether ordinary Closure and method calls can reach a known execution target
   without evaluating semantically irrelevant import/structured-protocol
   classifications on every hot invocation;
4. how D013 lookup/override semantics and existing structured-control/tooling
   contracts are preserved by fallback;
5. whether the proposed fast path actually reduces the compiler-visible graph
   and historical bailout/invalidation mechanism.

This is investigation only. It does not authorize a Protos implementation or
preselect the final fast-path design.

## Reconciled live state

```text
PERF010_CROSS_RUNTIME_ROOT_CAUSE=ESTABLISHED
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO

PREVIOUS_LOCAL_ABLATION_PROGRAM=DEFERRED
NEXT_ACTION=GUARDED_MONOMORPHIC_CALL_PATH_ARCHITECTURE_INVESTIGATION
```

PERF010-A / #691 remains open and continues to block PERF010 / #680. The next
work should test the architecture-level specialization boundary before returning
to additional local ablations or measurement-floor refinement.
