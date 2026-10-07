# PERF010-A — Paired-control causal reconciliation

Status: RECONCILIATION COMPLETE — COMMON DOMINANT COMPONENT NOT ESTABLISHED

This durable, non-normative record reconciles the valid PERF010-A causal
ablations using their retained canonical and paired-control medians before any
new benchmark is commissioned.

## Evidence

```text
ABLATION_1_EVIDENCE_REVISION=08d39b1d07b6766c5533486bf56b12d0b881fec2
ABLATION_3_EVIDENCE_REVISION=8a82117c50dbb2906ef80fac69e7421f95beb10a
ABLATION_2=INVALID
```

For this reconciliation, the descriptive paired-control correction is:

```text
canonical improvement = baseline canonical - ablation canonical
control movement       = baseline control - ablation control
paired-control effect  = canonical improvement - control movement
paired-control fraction = paired-control effect / baseline canonical
```

This subtraction is a diagnostic difference-in-differences style correction of
the retained medians. It is not a confidence interval or statistical estimator
of an attributable fraction of cross-language excess cost.

## Ablation 1 — semantic/helper dispatch

| workload | canonical improvement (ms) | control movement (ms) | paired-control effect (ms) | fraction of baseline |
|---|---:|---:|---:|---:|
| micro/slot-read | +1.4631 | -0.1825 | +1.6456 | +3.39% |
| micro/closure-call | +3.5492 | -1.4372 | +4.9863 | +8.45% |
| micro/method-call | +1.8237 | +2.0280 | -0.2044 | -0.35% |
| runtime/monomorphic-dispatch | +2.5176 | +0.9774 | +1.5402 | +2.74% |

The raw canonical result was positive in all four workloads, but paired-control
movement removes that consistency: method-call becomes slightly negative.
Ablation 1 therefore establishes causal cost but not a uniform common-path
contribution or dominance.

## Ablation 3 — activation lexical single-probe read

| workload | canonical improvement (ms) | control movement (ms) | paired-control effect (ms) | fraction of baseline |
|---|---:|---:|---:|---:|
| micro/slot-read | +1.1644 | -1.3965 | +2.5609 | +5.45% |
| micro/closure-call | +0.7495 | +0.6865 | +0.0630 | +0.11% |
| micro/method-call | +0.9547 | +0.6152 | +0.3395 | +0.62% |
| runtime/monomorphic-dispatch | -0.2550 | +0.9431 | -1.1981 | -2.17% |

Ablation 3 is structurally valid and measures the established exact component,
but its paired-control signal is strongly workload-dependent: substantial for
slot-read, approximately neutral for closure-call, small positive for
method-call, and negative for monomorphic-dispatch. It does not establish a
uniform common-path contribution or dominance.

## Cross-ablation conclusion

Neither valid ablation currently identifies the dominant common overhead sought
by PERF010-A.

Ablation 1 and Ablation 3 also do not exhibit a stable cross-workload pattern
that would justify selecting either as the production optimization target:

```text
                    ABLATION_1     ABLATION_3
slot-read              +3.39%         +5.45%
closure-call           +8.45%         +0.11%
method-call            -0.35%         +0.62%
monomorphic-dispatch   +2.74%         -2.17%
```

Ablation 2 contributes only a semantic negative constraint because its variant
was invalid.

The historical PERF004 Python/JavaScript baseline is therefore not the next
blocking measurement. A same-environment external baseline could scale a
credible causal effect into a fraction of excess cost, but it cannot turn the
current non-uniform paired-control signals into evidence that either component
is dominant. Measuring it now would answer a secondary denominator question
before the numerator/common-component question is stable.

## Reconciled state

```text
PERF010A_ABLATION_1=VALID
ABLATION_1_CAUSAL_SIGNAL=ESTABLISHED
ABLATION_1_COMMON_CONTRIBUTION=NOT_ESTABLISHED
ABLATION_1_DOMINANCE=NOT_ESTABLISHED

PERF010A_ABLATION_2=INVALID
ABLATION_2_PERFORMANCE_SIGNAL=NOT_ESTABLISHED

PERF010A_ABLATION_3=VALID
ABLATION_3_SCOPE_MATCH=PASS
ABLATION_3_COMMON_CONTRIBUTION=NOT_ESTABLISHED
ABLATION_3_DOMINANCE=NOT_ESTABLISHED

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
EXTERNAL_BASELINE_NEXT=NO
```

## Next bounded step

Before another expensive benchmark, investigate the retained profiles and
current common execution path to identify the next candidate component whose
presence and semantic-preserving diagnostic removal can be established
statically.

Do not open or implement a fourth ablation merely by sequence number. First
establish a candidate under the causal-ablation admission gate: common path,
exact operation, semantic equivalence, preserved observable result for all four
workloads, single-component isolation, and exact-scope structural contract.

Only after a candidate is established should a new diagnostic ablation be
implemented and measured.
