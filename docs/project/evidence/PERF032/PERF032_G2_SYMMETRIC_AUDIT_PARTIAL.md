# PERF032-G2 — symmetric graph audit, partial causal findings

Date: 2026-10-08

## Identity and scope

```text
ISSUE=guillermomolina/protos#831
SLICE=PERF032-G2
WORK_TYPE=INVESTIGATION
RESULT=INCOMPLETE
PRODUCT_CHANGE=NO
SPECIFICATION_CHANGE=NO
NEW_BENCHMARK=NO
BENCHMARK_REVISION=e221bb056a208693c9102891df2e13d65e61eefb
PRODUCT_REVISION_MEASURED=c0ac98971df115d64b7bc9f146e8b02e11da30e6
GRAPH_PHASE=After TruffleTier
```

This document is **non-normative**, records a partial source review and outstanding evidence, and does not authorize an implementation. Published data are inherited from [PERF032-F](PERF032_F_CROSS_TRUFFLE_GRAPH_PARITY_MATRIX.md); no new benchmark, execution, test or build was performed.

## Published comparative evidence

| Metric | Protos | GraalJS | GraalPy |
| --- | ---: | ---: | ---: |
| Nodes | 282 | 13 | 49 |
| Control-flow splits | 12 | 0 | 0 |
| Guards/deopts | 15 | 0 | 0 |
| Invokes | 3 | 0 | 0 |
| FrameStates | 54 | 1 | 6 |
| Allocations (analyzer category) | 10 | 1 | 1 |
| Selected compiled graphs | 1 | 1 | 1 |

The peer graph-count disparity is real in the retained selected `After TruffleTier` graphs. These counts alone do not prove identical invocation-root semantics, source-to-node attribution or removability. In particular virtualized allocations are not necessarily heap allocations and histogram groups can overlap.

The selected Protos root is `ProtosSemanticBytecodeRootNodeGen`. The largest attribution is `CachedBytecodeNode` (`COUNT=154`, `IFS=10`, `ALLOCS=4`). Its overlap with invokes/other totals must not be summed as independent savings. The three surviving invokes are `ProtosBytecodeRootNode.selectGuestHandlerOnRootCrossing`, `ProtosFrameArguments.materializeCompactActivation`, and `List.size`.

## Harness audit — provisional

```text
HARNESS_VERDICT=PARTIALLY_VALID
COMPARISON_EQUIVALENCE=UNRESOLVED
```

PERF032-F retains three prepared callable workloads with equal observable result 1, Polyglot `Value.execute()` entry, a common GraalVM version and selected compilation phase. However the language-specific wrappers differ (Protos `ProtosHostExecutableClosure`, JS `InteropBoundFunction`, Python `PFunction`); exact root-entry and inlining equivalence requires a trace-level proof, not a single graph per language.

**Correction to a provisional concern:** `harness_dirty=true` at capture does not by itself invalidate the matrix. PERF032-F explicitly records that the producer tree was subsequently committed at `e221bb056a208693c9102891df2e13d65e61eefb` and a human-reported `graphs-verify` gate found `HEAD_MATCHES_PRODUCER=YES` for all 24 cases. Do not treat the dirty marker as an unaddressed mismatch.

The current review has not exhaustively attributed the three source roots, the complete BGV node dependencies, inline boundaries or integer boxing/return representation. It therefore does **not** certify full apples-to-apples structural equivalence.

## Source-level observations, not completed causal attribution

At measured product revision `c0ac98971df115d64b7bc9f146e8b02e11da30e6`:

- `src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java`: the semantic bytecode root enables yield, tag instrumentation, materialized local access, tail-call handlers, and uncached interpreter support. These declarations are not proof that those capabilities survive in the minimum compiled path. The `interceptTruffleException` implementation records diagnostic origin and delegates to `ProtosBytecodeRootNode.interceptGuestException`.
- `src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java`: `interceptGuestException` can call `selectGuestHandlerOnRootCrossing` for signal transfers. Its exact retained CFG and the probability/reachability of that branch on a literal return remain unresolved. This class has `List.size` call sites for closure argument checks, but no graph-to-caller attribution yet proves which is the surviving invoke.
- `src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java`: `materializeCompactActivation` is a Truffle-boundary fallback used by activation retrieval. A retained call target does **not** establish materialization on every normal call.

PERF033/#832 reports `ROOT_TASK_ON_MINIMUM_VALUE_PATH=NO`, `PROTOS_TASK_ON_MINIMUM_VALUE_PATH=NO`, `ACTOR_TASK_REGISTRATION_ON_MINIMUM_VALUE_PATH=NO`, and `RICH_CALLEE_ACTIVATION_ON_LITERAL_PATH=NO`. No inspected evidence demonstrates these mechanisms have regressed. A residual activation fallback is not evidence of universal activation.

## Structural differences still requiring explanation

- GraalJS 13 versus GraalPy 49: the reported histogram includes more frame states, virtual object state and loads for GraalPy, but the exact upstream implementation causal map remains missing. The fact that GraalPy uses Bytecode DSL does not by itself explain the delta.
- Protos 282: `CachedBytecodeNode` dominates and 12 splits/15 guards remain while neither peer retains corresponding category counts. The precise bytecode operations, receiver/argument shape dependencies, deopt branch conditions, exception paths and each invoke caller are **not yet** mapped to graph nodes.
- PAY AS YOU GROW: excess raises a strong optimization question, but each claimed violation needs a proven unused feature path, specialization opportunity and a semantics-preserving removal precondition. No block is yet proved removable in the present audit.

## Next research gate

```text
PERF032_G2_RESULT=INCOMPLETE
HARNESS_VERDICT=PARTIALLY_VALID
COMPARISON_EQUIVALENCE=UNRESOLVED
JS_PYTHON_DIFFERENCE_EXPLAINED=PARTIAL
PROTOS_DOMINANT_EXCESS_CAUSE=UNRESOLVED
FIRST_DOMINANT_REMOVABLE_BLOCK=UNRESOLVED
ROOT_TASK_PRESENT_IN_GRAPH=NOT_IDENTIFIED
PROTOS_TASK_PRESENT_IN_GRAPH=NOT_IDENTIFIED
UNIVERSAL_ACTIVATION_PRESENT=NOT_PROVEN
DESIGN_DECISION_REQUIRED=UNDETERMINED
IMPLEMENTATION_AUTHORIZED_BY_EXISTING_DESIGN=UNDETERMINED
BENCHMARK_CHANGE_REQUIRED=NOT_ESTABLISHED
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_REPOSITORY=NONE
```

The next investigation must inspect the **three** published BGV/derived graph traces symmetrically, confirm root and inline boundaries, identify Protos's exact `List.size` caller and fallback reachability, and map every candidate removable block back to concrete upstream source and normative Protos semantics. If tools cannot read binary BGV, identify the exact missing artifact and stop rather than inventing source attribution. Do not authorize product modifications or remeasure published data merely to fill a presentation gap.

## References

- [PERF032 issue](https://github.com/guillermomolina/protos/issues/831)
- [PERF033 issue](https://github.com/guillermomolina/protos/issues/832)
- [PERF032-F reference matrix](PERF032_F_CROSS_TRUFFLE_GRAPH_PARITY_MATRIX.md)
- [Raw PERF032-F results](https://github.com/guillermomolina/protos-benchmarks/tree/e221bb056a208693c9102891df2e13d65e61eefb/results/perf032-f)
- [Measured Protos Semantic Bytecode root](https://github.com/guillermomolina/protos/blob/c0ac98971df115d64b7bc9f146e8b02e11da30e6/src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java)
- [Measured Protos Bytecode root](https://github.com/guillermomolina/protos/blob/c0ac98971df115d64b7bc9f146e8b02e11da30e6/src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java)
- [Measured frame arguments](https://github.com/guillermomolina/protos/blob/c0ac98971df115d64b7bc9f146e8b02e11da30e6/src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java)

## AI assistance and validation boundary

This partial report was prepared by ChatGPT from published PERF032-F evidence and targeted read-only GitHub source inspection. No tests, builds, commands, BGV extraction or benchmark runs were performed. The GitHub contents publication interface does not execute the locally prescribed `git diff --check`; that validation gate is **not claimed as PASS**.
