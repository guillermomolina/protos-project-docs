# PERF010-A — Third causal ablation reference evidence

Status: RETAINED INVALID ATTEMPT — ESTABLISHED ABLATION 3 NOT EXECUTED

This is a durable, non-normative performance-investigation record for
`PERF010-A / #691`. It records the published third-ablation harness and
reference result without treating invalid timing as causal evidence or
authorizing a production optimization.

## Revision identity

```text
PROTOS_REVISION=6e7d89194925ba9fa2cd9c5c45aefa72d9939621
ABLATION_3_READINESS_PROJECT_RECORD_REVISION=c02f7570a027f8e07ad3adaceaa823a57f97b964
ABLATION_3_READINESS_PATH=docs/project/work/PERF010-A/PERF010-A_THIRD_CAUSAL_ABLATION_READINESS.md
HARNESS_REVISION=de3a4acbd2b1bfbb9e9b4d762e8d44bb2be269d6
BENCHMARK_EVIDENCE_REVISION=3eb46f941946c8045cdb5527af9038b52fc867e6
BENCHMARK_EVIDENCE_PATH=results/perf010a-3/
```

The harness commit is:

```text
de3a4acbd2b1bfbb9e9b4d762e8d44bb2be269d6
PERF010-A: add third causal ablation for redundant lexical slot-read lookup (#691)
```

The retained reference evidence is:

```text
3eb46f941946c8045cdb5527af9038b52fc867e6
PERF010-A: retain third causal ablation evidence (#691)
```

## Readiness contract

The prior readiness investigation established a deliberately narrow diagnostic
intervention. It required a new single-probe local-slot reader to be used only
for the current-context and captured-context reads inside
`ProtosActivation.lookup`.

It explicitly required ordinary `ProtosObjectValue.readLocalSlot` callers and
`ProtosValueLookup` receiver/member lookup to remain unchanged, because a
global replacement would mix activation lexical-resolution cost with
member/delegation lookup cost.

The readiness record therefore states:

```text
ABLATION_3_TARGET=redundant containsKey+get double map probe on successful activation lexical local-slot reads
ABLATION_3_METHOD=ProtosActivation.lookup plus diagnostic ProtosObjectValue single-probe local-slot reader
SINGLE_CAUSAL_COMPONENT=YES
```

and explicitly excludes a global `readLocalSlot` replacement.

## Published implementation divergence

The published harness did not implement that established intervention.

Its retained evidence describes the patch as transforming the existing
`ProtosObjectValue.readLocalSlot` method itself:

```text
localSlots.containsKey(name) + localSlots.get(name)
    ->
single localSlots.get(name)
```

The retained evidence also states that `readLocalSlot` is used by many other
production call sites. Therefore this patch changes all callers of
`readLocalSlot`, including paths that the readiness record explicitly required
to remain baseline behavior.

Consequently:

```text
ESTABLISHED_ABLATION_3_EXECUTED=NO
ABLATION_3_SCOPE_MATCH=FAIL
SINGLE_CAUSAL_COMPONENT_FOR_ESTABLISHED_TARGET=NOT_PROVEN
```

The representation argument that the global transformation is semantically
equivalent does not repair this attribution mismatch. Semantic equivalence is
necessary, but the established causal experiment also required isolation of the
activation lexical-read component.

## Reference result

The retained reference reports:

```text
PERF010A_ABLATION_3=INVALID
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
```

All four workload classifications have:

```text
structural_ablation_confirmed=False
```

The retained medians are:

| workload | baseline canonical median (ns) | ablation canonical median (ns) | raw removed fraction |
|---|---:|---:|---:|
| micro/slot-read | 47,121,852.5 | 44,623,479.5 | +5.30% |
| micro/closure-call | 56,038,634.0 | 60,252,002.0 | -7.52% |
| micro/method-call | 60,865,152.5 | 57,166,836.5 | +6.08% |
| runtime/monomorphic-dispatch | 54,491,520.0 | 52,069,492.5 | +4.44% |

These timing movements are retained as raw measurements only. They MUST NOT be
interpreted as attributable cost for the established Ablation 3 because the
published run is structurally invalid and the patch scope does not match the
established causal intervention.

## Additional harness inconsistency

The harness source says Ablation 3 structural confirmation is source-derived and
that `reference()` obtains per-image `source_structural_probe` results before
classification. Nevertheless, the retained evidence classifies structural
confirmation as false for all four workloads.

That discrepancy must be diagnosed before another reference run. The current
record does not infer whether the failure is in the source probe, classification,
result serialization, patch shape, or another harness detail.

## Current causal state

```text
PERF010A_ABLATION_1=VALID
ABLATION_1_CAUSAL_SIGNAL=ESTABLISHED
ABLATION_1_ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
ABLATION_1_DOMINANCE=NOT_ESTABLISHED

PERF010A_ABLATION_2=INVALID
ABLATION_2_PERFORMANCE_SIGNAL=NOT_ESTABLISHED

PERF010A_ABLATION_3_READINESS=ESTABLISHED
PERF010A_ABLATION_3_EXECUTION=INVALID
ESTABLISHED_ABLATION_3_EXECUTED=NO

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

## Next bounded step

Do not choose a production optimization from this run and do not move to a
fourth causal component yet.

The next step is to repair/reconcile the Ablation 3 harness so that it implements
the already-established narrow intervention exactly:

1. add/use the diagnostic single-probe reader only for current and captured
   lexical reads in `ProtosActivation.lookup`;
2. leave ordinary `readLocalSlot` and `ProtosValueLookup` behavior on the
   baseline implementation path;
3. make structural confirmation prove those boundaries;
4. rerun validate and cheap smoke;
5. publish the corrected harness before running a new reference.

No new Protos semantic or platform decision is required by this evidence.
