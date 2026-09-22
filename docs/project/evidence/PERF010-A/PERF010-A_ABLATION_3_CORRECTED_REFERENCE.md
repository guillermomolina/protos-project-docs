# PERF010-A — Corrected third causal ablation reference evidence

Status: RETAINED VALID REFERENCE — ATTRIBUTABLE FRACTION NOT ESTABLISHED

This is a durable, non-normative performance-investigation record for
`PERF010-A / #691`. It supersedes the execution conclusion of the earlier
invalid Ablation 3 attempt while preserving that attempt as historical negative
evidence. It does not authorize a production optimization.

## Revision identity

```text
PROTOS_REVISION=6e7d89194925ba9fa2cd9c5c45aefa72d9939621
CORRECTED_HARNESS_REVISION=fd22a347d24b2d3ae697846a5679c4dbd2a1e8bf
BENCHMARK_EVIDENCE_REVISION=8a82117c50dbb2906ef80fac69e7421f95beb10a
BENCHMARK_EVIDENCE_PATH=results/perf010a-3b/
PREVIOUS_INVALID_EVIDENCE_REVISION=3eb46f941946c8045cdb5527af9038b52fc867e6
```

The corrected harness implements the already-established narrow intervention:
a diagnostic single-probe local-slot reader is used only for the current and
captured lexical reads in `ProtosActivation.lookup`. Ordinary
`ProtosObjectValue.readLocalSlot` remains unchanged and
`ProtosValueLookup` remains on that ordinary path.

The retained evidence reports exact-scope structural confirmation as true for
all four workloads.

## Reference result

```text
PERF010A_ABLATION_3=VALID
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
```

| workload | baseline median (ns) | ablation median (ns) | removed (ns) | removed fraction |
|---|---:|---:|---:|---:|
| micro/slot-read | 46,977,712.0 | 45,813,333.5 | 1,164,378.5 | +2.4786% |
| micro/closure-call | 55,228,029.0 | 54,478,523.5 | 749,505.5 | +1.3571% |
| micro/method-call | 54,948,155.0 | 53,993,482.5 | 954,672.5 | +1.7374% |
| runtime/monomorphic-dispatch | 55,123,283.0 | 55,378,281.5 | -254,998.5 | -0.4626% |

The first three canonical workloads show a positive raw removed cost. The
monomorphic-dispatch workload shows a small negative raw movement. These are
baseline-relative raw measurements, not an established fraction of the
cross-language excess cost.

The retained reference deliberately does not combine the historical PERF004-A
external baseline with this evidence unit because it was measured at a
different Protos revision and container base. Therefore:

```text
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

Establishing attributable fraction requires a current-revision/current-host
external Python/JavaScript measurement under the same evidence unit.

## Structural scope

The corrected ablation preserves the established attribution boundary:

```text
current activation lexical read -> diagnostic single-probe reader
captured lexical read           -> diagnostic single-probe reader

ordinary readLocalSlot          -> unchanged baseline implementation
ProtosValueLookup               -> unchanged ordinary readLocalSlot path
receiver/delegation lookup      -> unchanged
captured traversal/order        -> unchanged
semantic/helper Bytecode        -> unchanged
```

The earlier global `readLocalSlot` transformation remains an invalid historical
attempt and is not reinterpreted by this result.

## Current causal state

```text
PERF010A_ABLATION_1=VALID
ABLATION_1_CAUSAL_SIGNAL=ESTABLISHED
ABLATION_1_ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
ABLATION_1_DOMINANCE=NOT_ESTABLISHED

PERF010A_ABLATION_2=INVALID
ABLATION_2_PERFORMANCE_SIGNAL=NOT_ESTABLISHED

PERF010A_ABLATION_3_READINESS=ESTABLISHED
PERF010A_ABLATION_3_EXECUTION=VALID
ESTABLISHED_ABLATION_3_EXECUTED=YES
ABLATION_3_SCOPE_MATCH=PASS
ABLATION_3_ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

## Next bounded step

Do not select a production optimization from the raw baseline-relative
percentages alone.

The missing evidence is now explicit: obtain a comparable external
Python/JavaScript baseline at the same current revision/host/evidence-unit
conditions needed to compute excess cost and attributable fraction. Reconcile
that evidence with Ablations 1 and 3 before deciding whether a dominant
common-overhead component has been established.
