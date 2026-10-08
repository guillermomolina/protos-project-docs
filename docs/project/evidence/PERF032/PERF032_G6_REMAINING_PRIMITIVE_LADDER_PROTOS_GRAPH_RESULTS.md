# PERF032-G6 — Protos-only remaining primitive ladder graph results

Date: 2026-10-08

## Scope and provenance

This is a **non-normative, partial cross-workload measurement checkpoint** under
[PERF032/#831](https://github.com/guillermomolina/protos/issues/831), following
[G6's 47-node return-literal convergence](PERF032_G6_ZERO_PARAMETER_CLOSURE_GRAPH_CONVERGENCE_AND_VALIDATION.md).
It does not supersede the historical
[PERF032-F three-language matrix](PERF032_F_CROSS_TRUFFLE_GRAPH_PARITY_MATRIX.md).

```text
WORK_ITEM=PERF032
MEASUREMENT_SET=PERF032-G6-REST-PROTOS
LANGUAGE=protos only
WORKLOADS=7 remaining primitive ladder cases
GRAPH_PHASE=After TruffleTier
HOST_IGV_ANALYZE_FAILURES=0
STABLE_VALID_UNITS=6
UNSTABLE_INVALID_UNITS=1
INVALID_UNIT=primitive-object-slot-write/protos
INVALID_REASON=GRAPH_NOT_STABLE
GRAPH_PARITY_WITH_PEERS=NOT_EVALUATED_IN_PROTOS_ONLY_CAPTURE
TIMING_A_B=NOT_PERFORMED
BENCHMARK_RESULTS_LOCAL_PATH=results/perf032-g6-rest-protos/
PRODUCT_MEASUREMENT_SHA=NOT_REPORTED_IN_USER_OUTPUT
```

The maintainer captured the seven cases in the Protos devcontainer and
analyzed BGVs on the separate Docker-capable host, using the pinned IGV
analyzer. The provided `graphs-analyze` output reports **14 BGVs analyzed**
and `ANALYZE_FAILURES=0`. The `graphs-summarize` output reports six
valid stabilized single-graph units; the object-slot-write unit remains
invalid because its graph did not stabilize. The resulting summarizer
nonzero exit is expected when `INVALID_UNITS=1`, and **does not invalidate
the six stabilized cases**.

At this record's publication time, the public Protos repository HEAD was
`b0776d0d8077f9914dd5aecb6b4d01caae398a3e`
(`0.3.300-SNAPSHOT`) and the public benchmark HEAD was
`9b64a181cea1b61ea8992aab7f0a3a995220d753`.
However, the **capture's own embedded `protos_revision` and harness
producer identity were not supplied in the shared output**; do not
silently relabel the measurement with either moving HEAD. Raw G6-rest
artifacts were retained locally, not confirmed published to the benchmark
repository. Source identity can be established from each `capture.json`.

## Historical PERF032-F versus Protos G6-rest

Numbers below compare the **published historical** PERF032-F selected-phase
Protos count against the maintainer-reported later `graphs-summarize`
result for the same named workload. This is structural context, not a
same-run three-language peer verdict.

| Workload | PERF032-F Protos | G6-rest Protos | Difference | Status |
| --- | ---: | ---: | ---: | --- |
| `primitive-return-literal` | 282 | 47 | −235 | Stable; measured separately in G6 |
| `primitive-local-read` | 1,283 | 972 | −311 | STABLE, valid, one graph |
| `primitive-local-write` | 2,343 | 2,032 | −311 | STABLE, valid, one graph |
| `primitive-integer-add` | 4,595 | 4,282 | −313 | STABLE, valid, one graph |
| `primitive-object-slot-read` | 583 | 346 | −237 | STABLE, valid, one graph |
| `primitive-object-slot-write` | N/A | N/A | N/A | GRAPH_NOT_STABLE in both sets |
| `primitive-closure-call` | 4,741 | 4,275 | −466 | STABLE, valid, one graph |
| `primitive-method-call` | 2,139 | 1,912 | −227 | STABLE, valid, one graph |

The return-literal case is repeated from the separately admitted G6 capture;
it was **not** remeasured in this seven-workload set. The G6 source change
is a plausible contributor to the reductions across workloads because it
removes zero-parameter Closure arity machinery. The summary totals alone,
however, **do not establish source-level causal attribution for each
remaining workload**.

The historical PERF032-F run retained GraalJS/GraalPy references for all
eight workloads, but the later G6-rest capture contains **only Protos**.
Accordingly its `RUNG` rows report `js=None`, `python=None`,
`PEER=UNRESOLVED`, `PROTOS_STATUS=NOT_EVALUATED` and
`FIRST_DIVERGENT_RUNG=NONE`. That last value signifies **no comparable
rung could be evaluated in this partial dataset**, not new cross-language
convergence.

## The object-slot-write non-stabilization

The new result is:

```text
primitive-object-slot-write/protos
EVIDENCE_VALID=NO
INVALID_REASON=GRAPH_NOT_STABLE
SELECTED_NODE_COUNT=NOT_ADMITTED
```

The historical PERF032-F record attributed its instability symptoms to
extensive recompilation/invalidation and a maximum-compilation-count
failure. **The provided G6-rest summarize/analyze output does not contain
the new capture's per-budget reason codes or trace-derived compilation
lifecycle**. Thus it establishes that the **instability persists**, but
does **not** establish that the exact historical 100-compilation pattern or
cause persists.

Before proposing a product repair, examine the existing G6-rest
`primitive-object-slot-write/protos/capture.json` per-budget assessments
and retained `trace.log.gz`, using the existing generic
`graph_evidence.instability_evidence` mechanism. Do not increase warmup
budgets, select an invalid graph, or rerun the already-completed capture
merely to manufacture parity.

## Status and next admission gates

- `primitive-return-literal`: 47 nodes, stable, inside 13–49 peer band
  according to the separate G6 three-language evidence.
- Six other Protos primitive rungs: graph-size reductions observed, **not
  structurally converged** on any new admitted three-language comparison.
- `primitive-object-slot-write`: unstable under the bounded policy, without
  admitted node count.
- `PERF032/#831`: **OPEN**. The issue owns the isolated return-literal
  performance question; the additional ladder results are diagnostic context,
  not automatic authorization for a new multi-workload optimization campaign.
- Actual JVM `ns/call` improvement for G6: **not measured** in this record.
- Appropriate next steps: first extract the **new existing** unstable
  workload trace identity, verdict and lifecycle; separately perform the
  PERF032-required same-workload return-literal timing A/B if pursuing issue
  closure.

## Validation disclosure

All benchmark commands and outcomes in this record were supplied by the
maintainer. The documentation author has not run benchmarks, Java tests,
IGV processing, or any fresh product measurement.

This record was materially prepared with AI assistance from ChatGPT.
