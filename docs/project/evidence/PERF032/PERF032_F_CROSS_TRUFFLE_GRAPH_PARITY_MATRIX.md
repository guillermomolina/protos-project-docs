# PERF032-F — cross-Truffle compiled-graph parity matrix

Date: 2026-10-07

## Work identity

~~~text
WORK_ITEM=PERF032
ISSUE=guillermomolina/protos#831
SLICE=PERF032-F
WORK_TYPE=IMPLEMENTATION_AND_MEASUREMENT
PRODUCT_REVISION=c0ac98971df115d64b7bc9f146e8b02e11da30e6
BENCHMARK_REVISION=e221bb056a208693c9102891df2e13d65e61eefb
SELECTED_GRAPH_PHASE=After TruffleTier
PRIMARY_METRIC=relevant_graph_nodes_total_after_truffle_tier
PRODUCT_CHANGE=NO
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
MICROOPTIMIZATION_PERFORMED=NO
~~~

This record is durable, non-normative performance evidence. Raw and derived
benchmark evidence remains owned by
`guillermomolina/protos-benchmarks@e221bb056a208693c9102891df2e13d65e61eefb`
under `results/perf032-f/`.

## Publication identity

The benchmark implementation and retained matrix were published as:

~~~text
BENCHMARK_REVISION=e221bb056a208693c9102891df2e13d65e61eefb
PROTOS_REVISION=c0ac98971df115d64b7bc9f146e8b02e11da30e6
~~~

The graph captures were produced while the benchmark implementation tree was
still uncommitted, so per-case evidence records the parent benchmark HEAD
`1afa23af712988db1137e1528af25661ee530704` with
`harness_dirty=true`. The completed slice then committed exactly that producer
tree as `e221bb056a208693c9102891df2e13d65e61eefb`.
The reported post-commit `graphs-verify` gate established
`HEAD_MATCHES_PRODUCER=YES` for all 24 cases. The retained evidence is therefore
revision-bound to the published benchmark commit despite the capture-time dirty
marker.

## Matrix admission

The implemented ladder contains eight workloads across Protos, GraalJS and
GraalPy: 24 language/workload cases.

Human-reported completion evidence and the retained result corpus record:

~~~text
CORRECTNESS=24/24 PASS
STABLE_CASES=23/24
COMMON_STABLE_PAIR_FOR_VALID_CASES=[16000,64000]
ANALYZE=48/48
ANALYZE_FAILURES=0
INVALID_UNITS=1
INVALID_UNIT=primitive-object-slot-write/protos
INVALID_REASON=GRAPH_NOT_STABLE
~~~

The one invalid graph unit is deliberate retained instability evidence, not a
failed correctness result.

The current analyzer identity is:

~~~text
GRAALVM=GraalVM CE 25.4.4.1.1
JAVA_RUNTIME=25.0.4.1.1+1-jvmci-25.4-b23
ANALYZER_IMAGE=ghcr.io/guillermomolina/protos-benchmarks/igv-analyzer:graal-25.4.4.1.1
~~~

For every valid case, the retained BGV `StructuredGraph`
`After TruffleTier` node count equals the corresponding trace IR count.

## Primitive graph-parity matrix

| Workload | Protos nodes | GraalJS nodes | GraalPy nodes | Peer band | Protos status |
| --- | ---: | ---: | ---: | --- | --- |
| primitive-return-literal | 282 | 13 | 49 | 13–49 | STRUCTURAL_EXCESS |
| primitive-local-read | 1283 | 13 | 49 | 13–49 | STRUCTURAL_EXCESS |
| primitive-local-write | 2343 | 13 | 49 | 13–49 | STRUCTURAL_EXCESS |
| primitive-integer-add | 4595 | 14 | 49 | 14–49 | STRUCTURAL_EXCESS |
| primitive-object-slot-read | 583 | 36 | 103 | 36–103 | STRUCTURAL_EXCESS |
| primitive-object-slot-write | N/A | 100 | 159 | 100–159 | DIVERGED_BEFORE_GRAPH_PARITY |
| primitive-closure-call | 4741 | 13 | 50 | 13–50 | STRUCTURAL_EXCESS |
| primitive-method-call | 2139 | 34 | 74 | 34–74 | STRUCTURAL_EXCESS |

GraalJS and GraalPy satisfy the PERF032-E peer-convergence rule on all eight
rungs. Therefore none of the Protos findings above is hidden by an unresolved
peer reference.

## First divergent rung

The first divergence appears at the smallest workload:

~~~text
FIRST_DIVERGENT_RUNG=primitive-return-literal
MECHANISM=baseline callable entry + return of one primitive literal

PROTOS_NODES=282
GRAALJS_NODES=13
GRAALPY_NODES=49
PEER_SPREAD=36
PROTOS_EXCESS_OVER_PEER_MAX=233

PEER_REFERENCE=CONVERGED
PROTOS_STATUS=STRUCTURAL_EXCESS
GRAPH_COUNT_PROTOS=1
GRAPH_COUNT_GRAALJS=1
GRAPH_COUNT_GRAALPY=1
~~~

The Protos graph also contains structure absent from the peer maximum:

~~~text
ALLOCATIONS_DELTA=+9
CONTROL_FLOW_SPLITS_DELTA=+12
GUARDS_DEOPTS_DELTA=+15
INVOKES_DELTA=+3
LOADS_DELTA=+9
~~~

The three surviving Protos invoke targets are:

~~~text
ProtosBytecodeRootNode.selectGuestHandlerOnRootCrossing
ProtosFrameArguments.materializeCompactActivation
List.size
~~~

The largest retained Truffle-tier attribution row is
`ProtosSemanticBytecodeRootNodeGen$CachedBytecodeNode` with 154 nodes,
10 conditionals and 4 allocations in its self attribution.

This establishes an explanation burden at callable-entry/literal-return itself.
It does not by itself establish which individual nodes are semantically required
or removable.

## No double-Bytecode graph count

PERF032-E explicitly required current evidence before reviving the historical
PERF008 double-Bytecode hypothesis.

The matrix reports one relevant Protos graph on every valid Protos rung:

~~~text
DOUBLE_BYTECODE_DISPATCH_CURRENTLY_ESTABLISHED=NO
~~~

The observed structural excess is inside the single primary compiled graph, not
an extra separately compiled semantic/helper graph.

Historical PERF008 continueAt evidence therefore remains attribution context,
not a currently established optimization owner.

## Independent object-slot-write instability

`primitive-object-slot-write/protos` passes observable correctness but never
produces an admissible stable graph under the bounded policy:

~~~text
CORRECTNESS=PASS
STABILIZATION=GRAPH_NOT_STABLE
PROTOS_STATUS=DIVERGED_BEFORE_GRAPH_PARITY
PEER_REFERENCE=CONVERGED
PEER_NODE_COMPARISON=SKIPPED
FINAL_COMPILATION_STATE=BailedOut

PRIMARY_COMPILATIONS=100
PRIMARY_COMPILATIONS_TIER_1=100
PRIMARY_INVALIDATION_EVENTS=200
MAXIMUM_COMPILATION_COUNT_REACHED=YES
FAILURE=Maximum compilation count 100 reached.
~~~

The retained signals are:

~~~text
UNIT_INVALIDATED_AFTER_FINAL
UNIT_NOT_AT_FINAL_TIER
UNIT_RECOMPILATION_FAILED
FINAL_COMPILATION_STATE_BAILED_OUT
MAXIMUM_COMPILATION_COUNT_REACHED
~~~

This is a separate divergence mode from the already-earlier
`primitive-return-literal` structural excess. PERF032-F deliberately did not
attempt a product fix, raise the 256k stabilization ceiling, force
`CompileImmediately`, or select an unstable graph merely to manufacture a
node-count comparison.

The 100-compilation cap is the observed terminal state, not yet the established
cause of each invalidation.

## Validation reported for the published slice

The completion report supplied for the benchmark commit records:

~~~text
UNIT_TESTS=88 OK
CAPTURE_SMOKE=PASS for 3 languages
ANALYZE=48/48
ANALYZE_FAILURES=0
SUMMARIZE_INVALID_UNITS=1
SUMMARIZE_INVALID_UNIT=deliberate primitive-object-slot-write/protos instability
GRAPHS_VERIFY_CASES=24
GRAPHS_VERIFY_WORKING_TREE_MATCH=PASS
GRAPHS_VERIFY_HEAD_MATCHES_PRODUCER=PASS
GIT_DIFF_CHECK=PASS
REFERENCE_MATRIX=COMPLETE
~~~

These executions were human-executed under the repository's human-executor
policy; this durable record does not claim that the documentation publisher
reran them.

## Routing

The governing PERF032 ordering remains:

~~~text
semantics / pay-as-you-grow
  -> compiled graph structure
  -> peer structural parity
  -> causal attribution
  -> only then microoptimization
~~~

PERF032-F has now established the first structural divergence before any
microoptimization.

The immediate next slice is therefore a causal **investigation**, not product
implementation:

~~~text
NEXT_SLICE=PERF032-G
NEXT_SLICE_TYPE=INVESTIGATION
PRIMARY_TARGET=primitive-return-literal
QUESTION=Which parts of the 282-node After-TruffleTier primary graph are required
         by active observable Protos semantics, and which are removable minimum-
         path machinery?
PRODUCT_CHANGE_AUTHORIZED=NO
COMMAND_EXECUTION=NONE
NEW_ISSUE_REQUIRED=NO
~~~

The investigation should begin with the concrete retained owners:

- `ProtosSemanticBytecodeRootNodeGen$CachedBytecodeNode`;
- `selectGuestHandlerOnRootCrossing`;
- `materializeCompactActivation`;
- `List.size`;
- the 12 control-flow splits;
- the 15 guards/deoptimization checks;
- the retained allocations and frame-state structure.

The independent `primitive-object-slot-write` invalidation loop remains a
second causal investigation after the earliest divergence is understood. Do not
fold a speculative fix for either finding into PERF032-G.

## Result

~~~text
PERF032_F_RESULT=COMPLETE
FIRST_DIVERGENT_RUNG=primitive-return-literal
PEER_REFERENCE_STATUS=CONVERGED_ON_ALL_8_RUNGS
PROTOS_STRUCTURAL_EXCESS_RUNGS=7
FIRST_NON_STABILIZING_PROTOS_RUNG=primitive-object-slot-write
DOUBLE_BYTECODE_DISPATCH_CURRENTLY_ESTABLISHED=NO

NEXT_SLICE=PERF032-G
NEXT_SLICE_TYPE=INVESTIGATION
NEW_ISSUE_REQUIRED=NO
MICROOPTIMIZATION_AUTHORIZED=NO
~~~

## AI-assistance disclosure

This durable record was materially prepared with AI assistance from ChatGPT
from the published PERF032-F benchmark corpus, exact benchmark/product revisions,
the prior PERF032-E research record, and the human-reported validation/push
results.
