# PERF038-H — Published implementation and residual compiled-graph checkpoint

Date: 2026-10-09
Live issue: [guillermomolina/protos#852](https://github.com/guillermomolina/protos/issues/852)
Published product commit: [`0db24f00ff2d92d642351d7f7535517fe01a55ce`](https://github.com/guillermomolina/protos/commit/0db24f00ff2d92d642351d7f7535517fe01a55ce)
Product version: `0.3.324-SNAPSHOT`
Status: implementation committed and pushed, maintainer-reported tests green; structural optimization remains open.

## Publication and validation evidence

The human executor pushed `main` from `70c40d26` to `0db24f00` and reported `git diff --check` clean, `make check` PASS, and `make test` PASS. These are maintainer-reported local results, **not** independent CI or agent test execution. The tests were completed before the version/changelog finalization; no post-finalization test rerun is claimed.

The published commit changes exactly 12 files: five Java implementation files (`CanonicalToBytecodeLowerer`, `ProtosBytecodeRootNode`, `ProtosFrameArguments`, `ProtosSemanticBytecodeRootNode`, `ProtosStandardIntegerProtocol`), four focused Java test files (`ProtosI072PhaseDPreparedCallSeparationTest`, `ProtosPerf025CallbackConsumerSpecializationTest`, `ProtosPerf032G7MinimalDirectHeaderTest`, `ProtosPerf038FResidualPrimitiveCallTest`), the generated-bytecode BCI PE baseline, `pom.xml`, and `CHANGELOG.md`.

Per the published `0.3.324-SNAPSHOT` changelog, this slice specializes zero-/one-argument direct source calls and sends, uses straight-line source-call paths where admitted, retains general call/control-transfer/return-home/continuation fallbacks, and updates regression coverage and generated-bytecode baseline. No normative Protos specification change is claimed.

## Human-local compiled-graph evidence: primitive-method-call

The maintainer captured, analyzed, and summarized `primitive-method-call` in `protos-benchmarks/results/local/perf038-h-method-call/`. IgvUtility reported `ANALYZE_FAILURES=0`; the matrix reported `INVALID_UNITS=0`, `peer=CONVERGED`, `protos_stabilization=STABLE`, `protos_status=STRUCTURAL_EXCESS`.

The compiled graph is **After TruffleTier**. Each language has one accepted graph. The exact `unit.json` `protos_revision`, `harness_git_head`, and capture dirty-state fields were **not** supplied in this interaction, so the local diagnostic is not asserted to have been captured from clean published product `0db24f00`. The capture was performed before the final product commit; the source/build identity still needs checking before representing it as a clean, publication-grade revision-bound run. The new raw BGV corpus is **not yet published** in `protos-benchmarks`. No new latency measurement or speedup claim is made.

| Compiled IR family | PERF032-F Protos historical | PERF038-H Protos local | GraalJS peer | GraalPy peer |
| --- | ---: | ---: | ---: | ---: |
| Invokes | 34 | 1 | 0 | 0 |
| Control-flow splits | 126 | 1 | 0 | 0 |
| Allocations (IR) | 53 | 4 | 1 | 1 |
| Guards/deopts | 114 | 11 | 6 | 6 |
| Loads | 129 | 7 | 6 | 11 |
| Loops | 1 | 0 | 0 | 0 |
| Other IR nodes | 1682 | 99 | 21 | 56 |
| **Total nodes** | **2139** | **123** | **34** | **74** |

Historical published PERF032-F reference: [`results/perf032-f/primitive-method-call/protos/unit.json`](https://github.com/guillermomolina/protos-benchmarks/blob/main/results/perf032-f/primitive-method-call/protos/unit.json). The historical-to-local change of 2139 to 123 nodes spans several intervening product revisions; **it is not an isolated PERF038-H A/B effect**.

The nearest publicly retained PERF038-F Protos reference is [`protos-benchmarks@623e494`](https://github.com/guillermomolina/protos-benchmarks/commit/623e494493f0d3f91136d10048f365c24407c78c), product `89e1b038c2fd510f47ddb991496882548d1bbe44`: 132 nodes, 0 invokes, 3 splits, 4 allocations, 15 guards/deopts, 9 loads, stable. Current local graph has **123 nodes**, 1 invoke, 1 split, 4 allocations, 11 guards/deopts, 7 loads. Numerically this is nine fewer nodes (132 -> 123), but the intervening product revisions and local capture identity prevent assigning the full difference exclusively to PERF038-H. Importantly, invokes increased **0 -> 1**, despite the total reduction.

The remaining `InvokeWithExceptionNode` targets `ProtosFrameArguments.materializeCompactActivation`. The remaining explicit branch is one `IfNode`, whose controlling condition was **not** identified by the extracted node properties (empty JSON) and must **not** be attributed to argument materialization without edge evidence. The graph also contains 15 `FrameState`, 38 `ConstantNode`, 9 `FixedGuardNode`, 8 `VirtualObjectState`, 5 `VirtualArrayNode`, two `CommitAllocationNode` and two `AllocatedObjectNode` (IR representations, not necessarily actual runtime allocations).

Top own-count Truffle expansion rows in the local graph: `CachedBytecodeNode` 35, `TryDirectSendOne_Node` 9, `LoadFrameClosureArgument_Node` 6, `CheckFrameClosureArgumentUpperBound_Node` 3, `SelectCapturedMaterializedOwnerFrameAtRoot_Node` 3, and the root 2. These are *non-exhaustive source-attribution rows*, not additive explanations of the total 123 nodes. The `primitive-method-call` workload has a zero-argument `run()` entry and a **one-argument** `receiver.identity(1)` method send. Seeing parameter access in this benchmark is not evidence that the zero-argument call itself eagerly materializes an argument collection.

## Human-local compiled-graph evidence: primitive-closure-call

The maintainer also analyzed `protos-benchmarks/results/local/perf038-h-closure-call/`, `ANALYZE_FAILURES=0`. GraalJS is stable with 13 nodes and GraalPy with 50 nodes. Protos is **`GRAPH_NOT_STABLE`**, with `UNIT_NOT_AT_FINAL_TIER` at every budget [1000, 4000, 16000, 64000, 256000]. The final compilation state is `COMPILED_TIER_1`, framework edge to primary `Inlined`, primary compilations=1 (Tier 1), primary invalidations=0, primary failures=0, maximum compilation count not reached. The full run lists one overall invalidation per budget; this must not be misattributed to the primary unit.

Consequently the harness correctly records `protos=None`, `node_comparison=SKIPPED`, `protos_status=DIVERGED_BEFORE_GRAPH_PARITY`, and `INVALID_UNITS=1`. There is **no valid Protos closure-call node count** for this capture. The reason Tier 2 was not reached has not been established, and the warmup/capture policy must not be silently changed to manufacture stabilization.

## Outstanding acceptance and next action

- **Remain OPEN:** product correctness publication is complete; architectural graph parity and PERF038 acceptance are not.
- For `primitive-method-call`, identify the actual input/control edges keeping `materializeCompactActivation`, the sole `IfNode`, and frame/deoptimization states alive; separate semantically required work from unnecessary ordinary-path fallback. Do not guess that one conditional exclusively explains all 89 nodes over GraalJS.
- For `primitive-closure-call`, explain primary Tier-1-only compilation despite `Inlined` framework edge and no primary failure/invalidation. Do not count an unstable graph or repeat peer captures needlessly.
- Preserve the owner-requested **pay-as-you-grow** criterion and GraalJS structural reference without changing observable Protos semantics or specializing for the benchmark text.
- Adopt a **fail-closed compiled-graph acceptance gate** for trivial relevant call shapes: in an admitted, stable Tier-2 After-TruffleTier graph, forbid surviving `Invoke` and `If/Switch` nodes when those properties are the chosen acceptance requirements. Such a gate is a **proposed follow-up**, not asserted to be implemented or passed by this commit. Keep separate `f() { 1 }` C0 and `identity(1)` M1 cases.
- Validate local capture identity/harness SHA and clean producer state before a final published A/B claim; no repeated behavioral suite is required merely to analyze retained graph evidence. Do not treat structurally smaller graphs alone as proven latency improvements.

Related durable investigation: [PERF038-G integral call cost](PERF038_G_INTEGRAL_CALL_COST_2026_10_09.md). Live progress and final closure remain governed by [issue #852](https://github.com/guillermomolina/protos/issues/852).
