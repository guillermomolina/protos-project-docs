# PERF038-A — Method-call graph evidence checkpoint

Status: investigation; causal attribution incomplete. Date: 2026-10-08. Owner: [PERF038 / guillermomolina/protos#852](https://github.com/guillermomolina/protos/issues/852).

## Authoritative evidence identity

- Workload: `primitive-method-call` (not historical loop-heavy `method-call`); expected result `1`. Three canonical sources are in `guillermomolina/protos-benchmarks:truffle/workloads/primitive-method-call/`.
- Raw locally retained location reported by human executor: `results/local/global-20261008-graphs/primitive-method-call/{protos,js,python}/`, with `capture.json`, `unit.json`, BGV and `.filter.json.gz`; these are **not claimed to be published**.
- Protos revision: `d59da442fd9bc6cf590fb9b2c82d3ff55c0391ae` (`0.3.312-SNAPSHOT`); harness revision: `98abc9af7a05a45a7d4056b72f36aef5889cb76f`, clean at capture; GraalVM CE 25.4.4.1.1.
- All three cases correctness PASS; evidence_valid=true, stabilization STABLE at budgets 16000 and 64000, Tier 2, selected phase `After TruffleTier`; instrumentation is graph-only, not timing.
- Protos canonical `run.execute()`; JS/Python executable-value `run.execute()`; the selected comparable primary guest compilation units inline the identity callee.

## Comparable graph metrics

| Metric | Protos | GraalJS | GraalPy |
| --- | ---: | ---: | ---: |
| Primary graph nodes | 1789 | 34 | 74 |
| Allocations | 41 | 1 | 1 |
| Control-flow splits | 107 | 0 | 0 |
| Guards/deopts | 87 | 6 | 6 |
| Invokes | 30 | 0 | 0 |
| Loads | 111 | 6 | 11 |
| FrameState nodes | 389 | 1 | 6 |

Protos has 1755 more nodes than GraalJS in this comparison. This is not an estimate of removable nodes, nor evidence of runtime timing superiority.

## Preliminary source-grounded attribution

The selected Protos graph retains invocation targets: `ProtosFrameArguments.materializeCompactActivation` (5), `ProtosLexicalBindingAuthorityCalls.contains` (7), `.put` (3), `.transferAll` (2), `ProtosObjectValue.invalidateTrackedLookupDependencies` (4), `ProtosBytecodeRootNode.selectGuestHandlerOnRootCrossing` (3), and six additional invokes.

Truffle trace attribution shows `ProtosSemanticBytecodeRootNodeGen$BindClosureFrameParameter_Node` with a contribution of 351 expansion nodes and 28 branches, alongside substantial `CachedBytecodeNode` contributions. Expansion attribution is not an additive final-BGV partition. Current code already has compact call arguments and lazy activation materialization: a general refactor is not warranted without a surviving-path explanation. The normative method rules require correct receiver, `methodHome`, inheritance, extraction identity, errors and authority; immediate invocation can avoid an otherwise-unobservable extracted Closure.

## Investigation gate

Obtain an exact-identity, same-stage comparison with **existing** `primitive-closure-call` graph evidence before concluding which cost belongs to method dispatch rather than underlying Closure calls. Inspect BGV live control/guard chains and relevant Truffle traces, attribute each candidate to a concrete Java method, distinguish cold/error paths from hot semantics, and propose only bounded removals with semantic guards. No implementation, tests or repeated captures were authorized in PERF038-A.

Next proposed slice: **PERF038-A2 (INVESTIGATION)** within the same Issue #852; no new formal Issue is necessary. This record does not close the issue or ratify a design decision.
