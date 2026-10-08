# PERF032-G6 — zero-parameter Closure root: structural convergence and validation

Date: 2026-10-08

## Authority, scope and revisions

This is **durable, non-normative implementation and measurement evidence**, not a
new Protos semantic decision, a timing result, or the closure of PERF032.

```text
WORK_ITEM=PERF032
ISSUE=guillermomolina/protos#831
SLICE=PERF032-G6
PRODUCT_BASELINE_GRAPH_REVISION=6dab6ecc08c9a2102a388e00908c710a15cf2a4c
PRODUCT_MEASURED_G6_REVISION=fc0efabe73e576946104d30f1885143fa9a2157e
PRODUCT_FINAL_TEST_RECONCILIATION_REVISION=b0776d0d8077f9914dd5aecb6b4d01caae398a3e
PRODUCT_FINAL_VERSION=0.3.300-SNAPSHOT
BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
BENCHMARK_PUBLISHED_HEAD_OBSERVED=9b64a181cea1b61ea8992aab7f0a3a995220d753
WORKLOAD=primitive-return-literal
SURFACE=canonical
SELECTED_GRAPH_PHASE=After TruffleTier
PRIMARY_METRIC=relevant_graph_nodes_total_after_truffle_tier
SEMANTIC_CHANGE=NO
TIMING_A_B=NOT_YET_PERFORMED_FOR_G6
ISSUE_CLOSURE=NOT_YET
```

The **measured product** is G6's implementation commit
[`fc0efabe`](https://github.com/guillermomolina/protos/commit/fc0efabe73e576946104d30f1885143fa9a2157e).
The later [`b0776d0d`](https://github.com/guillermomolina/protos/commit/b0776d0d8077f9914dd5aecb6b4d01caae398a3e)
changes only three PERF013 JUnit test files, `pom.xml` and `CHANGELOG.md`;
it does **not** constitute a fresh graph capture for the latter commit.
The benchmark HEAD above is the published revision observed during this
reconciliation, **not** an independently checked assertion that the local
G6 capture recorded an identical producer-tree hash.

## Implemented cause and repair

Prior [PERF032-F evidence](PERF032_F_CROSS_TRUFFLE_GRAPH_PARITY_MATRIX.md)
had established a stable 282-node Protos `primitive-return-literal` graph,
versus 13 in GraalJS and 49 in GraalPy. The surplus contained, among other
things, the zero-parameter Closure's in-root argument-count rejection path,
activation materialization, exceptional handler selection and related guards,
branches and materialized errors.

PERF032-G6 moves the excess-argument rejection of a source Closure **declaring
zero parameters** out of its ordinary root. Such a definition receives a
separate cold rejection root in its **same BytecodeRootNodes group**. Calls
supplying no arguments use the direct, compact normal source target without
the in-body upper-bound check. Calls supplying arguments select the rejection
root and retain the guest argument-count Error and source/trace semantics.
Closures declaring parameters and other existing parameter-binding paths are
not globally rewritten. No universal wrapper root was introduced.

Relevant changed implementation sources in G6:

- `CanonicalToBytecodeLowerer.java`: normal and rejection root lowering,
  stable grouping/reparse attachment;
- `ProtosSemanticBytecodeRootNode.java`: target selection and rejection-target
  reference;
- `ProtosBytecodeRootNode.java`, `ProtosFrameArguments`-related callers,
  `ProtosHostExecutableClosure.java` and
  `ProtosBytecodeClosureExecutionPlan.java`: preserved ordinary/direct and
  error-path entry behavior;
- `ProtosActivation.java`: compatible invocation handling.

The exact published commit, rather than this summary, is the source authority
for the complete changed-path list.

## Same-workload G4 versus G6 result

The maintainer captured the three languages in the devcontainer under the
natural-warmup `reference` policy, then analyzed the six retained BGV files
on a separate Docker-capable **host** using the pinned IGV analyzer image.
The host and devcontainer share/synchronize the output tree; these are
distinct execution environments.

The maintainer-provided command outputs report:

```text
WORKLOAD=primitive-return-literal
CORRECTNESS_PROTOS=PASS
CORRECTNESS_GRAALJS=PASS
CORRECTNESS_GRAALPY=PASS
CAPTURE_INVALID_CASES=0
STABILIZATION=STABLE for all three languages
STABLE_PAIR=[16000,64000]
ANALYZE_FAILURES=0
SELECTED_GRAPH_COUNT_PROTOS=1
SELECTED_GRAPH_COUNT_GRAALJS=1
SELECTED_GRAPH_COUNT_GRAALPY=1
INVALID_UNITS=0
PEER_REFERENCE=CONVERGED
PROTOS_STATUS=STRUCTURALLY_CONVERGED
FIRST_DIVERGENT_RUNG=NONE
```

| Graph metric | Protos before G6 | Protos after G6 | GraalJS G6 run | GraalPy G6 run |
| --- | ---: | ---: | ---: | ---: |
| After-TruffleTier nodes | **282** | **47** | **13** | **49** |
| `IfNode` | 12 | 0 | 0 | 0 |
| `FixedGuardNode` | 12 | 0 | 0 | 0 |
| `DeoptimizeNode` | 3 | 0 | 0 | 0 |
| `InvokeWithExceptionNode` | 3 | 0 | 0 | 0 |
| `MethodCallTargetNode` | 3 | 0 | 0 | 0 |
| `BeginNode` + `EndNode` | 48 | 0 | 0 | 0 |
| `FrameState` | 54 | 6 | 1 | 6 |
| `ConstantNode` | 29 | 16 | 4 | 16 |
| `LoadIndexedNode` | 5 | 5 | 2 | 6 |
| `PiNode` | 18 | 5 | 1 | 5 |

```text
NODES_REMOVED=235
NODE_REDUCTION_PERCENT=83.33
GRAPH_PARITY_TARGET_BAND=[13,49]
PROTOS_AFTER_G6=47
STRUCTURAL_CONVERGENCE=PASS
```

The complete, maintainer-reported node-class histogram comparison also shows
that Protos G6 and GraalPy have **identical counts in every node class except**
`BoxNode$AllocatingBoxNode` (Protos 0; GraalPy 1) and
`LoadIndexedNode` (Protos 5; GraalPy 6). Protos G6 introduces no node class
absent from GraalPy. This establishes **selected-graph structural parity**,
not machine-code identity or latency parity.

The G6 capture was stored locally as
`results/perf032-g6-current/primitive-return-literal/{protos,js,python}/`
under the benchmark checkout, with `unit.json` and `matrix.json`
produced by `graphs-summarize`. **Those G6 raw results are not verified as
published in the benchmark repository at the time of this record**. The
published old matrix remains at
[`results/perf032-g4-current`](https://github.com/guillermomolina/protos-benchmarks/tree/9b64a181cea1b61ea8992aab7f0a3a995220d753/results/perf032-g4-current)
and historical PERF032-F remains separately retained. Human-supplied G6
summary output is the provenance for the newer 47-node value.

## PERF013 test reconciliation after G6

The first integrated Maven run after G6 reported **2,945 Java tests,
6 failures, 0 errors, 1 skipped**. The six failures were all exact
`BytecodeRootNodes.count()` expectations written before cold rejection
roots existed:

| Historical PERF013 topology assertion | Old physical count | G6 physical count |
| --- | ---: | ---: |
| Default-value Closure grouping | 3 | 4 |
| Nested default-value Closures | 4 | 6 |
| Closure in inline object body | 2 | 3 |
| Nested Closures in inline object body | 3 | 5 |
| Three nested Closures in shared owner | 4 | 7 |
| Sibling Closures in shared owner | 3 | 5 |

The bounded follow-up at `b0776d0d` updates the six counts and asserts
that each new rejection root preserves its owner's exact
`BytecodeRootNodes` group. It leaves production code unchanged.
**The maintainer subsequently reports `make test` PASS** and has published
the follow-up as `0.3.300-SNAPSHOT`. This is a maintainer-reported validation
result, not an independently rerun test claim.

## Remaining PERF032 work and boundaries

1. **Do not close PERF032/#831 on graph parity alone.** Its acceptance
   criteria include a measured same-workload performance A/B at compatible
   revisions. No post-G6 `ns/call` comparison is reported here.
2. The maintainer is now measuring **only Protos** on the other seven
   PERF032-F primitive ladder workloads (local read/write, integer add,
   object slot read/write, closure call, method call). GraalJS and GraalPy
   baselines are unchanged. **No new result for those seven workloads has
   been reported at publication time.** In particular, the old
   `primitive-object-slot-write/protos` non-stabilization must not be
   rewritten as a pass.
3. The 47-node result and disappearance of exceptional graph machinery
   provide a strong structural explanation for the trivial workload, but
   no numeric speedup follows without timing. No separate G7 optimization
   is automatically required merely because 47 exceeds GraalJS's 13.

```text
PERF032_G6_IMPLEMENTATION=PUBLISHED
PERF032_G6_GRAPH_CAPTURE=HUMAN_REPORTED_STABLE
PERF032_G6_STRUCTURAL_CONVERGENCE=PASS
PERF032_G6_POST_FIX_MAKE_TEST=HUMAN_REPORTED_PASS
PERF032_G6_TIMING_A_B=PENDING
REMAINING_LADDER_PROTOS_CAPTURE=IN_PROGRESS_NOT_YET_REPORTED
PERF032_ISSUE_STATE=OPEN
NEXT_ACTION=COMPARE_REMAINING_CAPTURE_AND_MEASURE_SAME_WORKLOAD_TIMING
```

## Evidence and AI-assistance disclosure

Inspected authority: `guillermomolina/protos` GitHub Issue #831, its
coordination policy, exact published G6 and follow-up product commits, the
published PERF032-F and G4 benchmark records, and the existing durable
PERF032 project evidence. The new matrix/log outcomes and final
`make test` result were supplied by the maintainer in conversation.
The evidence author did not execute the product build, tests, capture,
IGV analysis, or timing benchmark.

This record was materially prepared with AI assistance from ChatGPT.
