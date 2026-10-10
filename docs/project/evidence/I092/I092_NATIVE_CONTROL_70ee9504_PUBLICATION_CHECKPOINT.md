# I092-NATIVE — published native control checkpoint (70ee9504)

**Date:** 2026-10-10.
**Issue:** [I092 / guillermomolina/protos#870](https://github.com/guillermomolina/protos/issues/870).
**Published product revision:** [`70ee9504224c59c033c647f9185bf8fb9b266b93`](https://github.com/guillermomolina/protos/commit/70ee9504224c59c033c647f9185bf8fb9b266b93).
**Previous product revision:** [`7b609c4ad0e146d6f02a7a3d036c6ff938ccad08`](https://github.com/guillermomolina/protos/commit/7b609c4ad0e146d6f02a7a3d036c6ff938ccad08).
**Previous benchmark evidence:** [`protos-benchmarks@8c91b495ac82ebe4a345ef02aa0db43fa5afe8c1`](https://github.com/guillermomolina/protos-benchmarks/tree/8c91b495ac82ebe4a345ef02aa0db43fa5afe8c1/results/i092-7b609c4a-graphs).
**Benchmark harness HEAD inspected:** `8c91b495ac82ebe4a345ef02aa0db43fa5afe8c1`.
**Current compiled-graph measurement:** **PENDING; NO NEW GRAPH OR TIMING RESULT**.

## Source-level implementation evidence

At the pinned published product revision:

1. `CanonicalToBytecodeLowerer.admitsNativeIfTrue` gates intrinsic lowering on the already-frozen standard root (D049). `emitNativeIfTrueCandidate` emits the non-spread, spread and frame-native candidate operations. `emitCanonicalIfTrueLiteralSite` emits an independently native hit, with exactly one B-prime callback source region. Generic misses retain ordinary `PrepareSend` / `emitPreparedInvocation` behavior.
2. `TryCompactCanonicalBooleanOne` and corresponding vector / frame-native forms check canonical receiver identity. For `false` they return canonical null without examining callback callability; for `true` they prepare only the selected callback. These operations do not repeat the `ifTrue` lookup or register selection `Assumption` objects.
3. `ProtosBytecodeRootNode.nativeIfTrueCallback` reuses compact B-prime admission or ordinary `prepareInlineLiteralCall` selection for the supplied callback, without constructing a synthetic standard `ifTrue` activation or its ReturnHome. Ordinary `Object.call` selection on the callback remains independently applicable.
4. `ProtosCoreBootstrap.validatePublishedRoot` verifies the frozen root's standard `ifTrue` and `call` behavior identities, in addition to published root invariants. Code lowered before publication remains generic.
5. Valid spread expansion takes a snapshot at the correct argument position without needless exact-activation materialization; invalid spread retains its proper Error authority.

These are *source-level and bytecode-lowering findings*, not independently measured compiled-graph cost or speed.

## Validation provenance and qualifications

The maintainer reported **all local tests PASS**, `git diff --check` clean and the I092 product commit pushed. This record makes no unsupported assertion about independent CI, raw logs, benchmark runs or measured speedup.

The single B-prime callback region avoids duplicate source RootTags; the standard call/method fallback, arity, capture, ReturnHome and instrumentation semantics remain acceptance constraints. Closure-local `call` overriding and exact caller/dynamic-control corner cases still merit explicit focused falsification where not already covered by the tested suite.

The shared literal site's `TryFinally` still has a native null-carrier completion specialization. Whether Graal eliminates this and other cold machinery from the compiled live canonical branch is *not yet demonstrated*. Dynamic callback execution may still require its own caller activation; that is not the removed synthetic `ifTrue` activation.

## Published historical comparison and new measurement slot

| Workload | Before I092 `0db24f00` | Earlier I092 `7b609c4a` | New `70ee9504` |
| --- | ---: | ---: | --- |
| `primitive-if-true` | 5,193 | 2,173 | **PENDING** |
| `primitive-if-false` | 892 | 444 | **PENDING** |
| `primitive-return-literal` | 39 | 39 | **PENDING** |

Historical figures are admitted selected `After TruffleTier` graph node totals, not timing. The previous post-I092 graph capture cannot establish the current graph.

**Causal attribution caveat:** Between the two product revisions there were also independently published I091 numeric-architecture commits (`2a36e671`, `7dfceb7e`, `0fec0483`). Do not attribute the entire whole-revision node-count delta to I092 without more specific source/graph evidence.

## Follow-up, human executed

Capture exactly the three workloads `primitive-if-true`, `primitive-if-false`, and `primitive-return-literal` using the existing clean `guillermomolina/protos-benchmarks` `truffle/measure_graphs.py` harness without editing workloads, scripts, compiler flags, warmup/stability budgets or graph policy. Pin exact product SHA `70ee9504224c59c033c647f9185bf8fb9b266b93`, harness SHA, runtime, host/CPU and raw evidence identities. Run smoke first, reference capture second, then `analyze`, `summarize`, `verify`.

Graph analysis needs Podman/Docker on the host when unavailable in the devcontainer. Admit only correctness PASS, stable, final-tier graph units; a `GRAPH_NOT_STABLE` unit is no evidence of a numeric speedup.

Inspect `IfNode`, `FrameState`, `InvokeWithExceptionNode`, allocation/caller materialization and especially the live versus cold/deoptimization provenance of any `materializeCompactActivation` or `CompleteIfTrueSiteOuter` nodes. Unit histograms alone do not prove live-path cost.

After human publication of the benchmark evidence, update the I092 issue and durable comparative report with the **actual** benchmark commit SHA. Keep I092 **OPEN** until these gates and durable cross-references are reconciled. The [historical I092 native-branch gap report](I092_NATIVE_BRANCH_ACCEPTANCE_GAP_20261010.md) remains valid for its old exact product revision and is not rewritten.
