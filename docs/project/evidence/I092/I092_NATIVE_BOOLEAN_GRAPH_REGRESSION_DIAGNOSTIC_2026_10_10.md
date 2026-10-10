# I092 — Native Boolean graph-growth checkpoint and causal research gate

**Recorded:** 2026-10-10. **Classification:** retained non-normative investigation checkpoint; `I092/#870` remains open. **This document does not approve an optimization or alter Protos semantics.**

- **Formal owner:** [I092 / guillermomolina/protos#870](https://github.com/guillermomolina/protos/issues/870); upstream research [PERF041/#863](https://github.com/guillermomolina/protos/issues/863) is already completed and should remain closed.
- **Product before:** [`guillermomolina/protos@7b609c4ad0e146d6f02a7a3d036c6ff938ccad08`](https://github.com/guillermomolina/protos/commit/7b609c4ad0e146d6f02a7a3d036c6ff938ccad08), published I092 guarded compact Boolean.
- **Product commit at new capture:** [`guillermomolina/protos@70ee9504224c59c033c647f9185bf8fb9b266b93`](https://github.com/guillermomolina/protos/commit/70ee9504224c59c033c647f9185bf8fb9b266b93), independently native `ifTrue` lowering, plus **uncommitted I091 changes**. The effective product source state for **all three captured cases** is `abf804007e73e6c7e2783c337d2b68c8b3029f7519907064d50e99d92a510c0e` (not reconstructable from product HEAD alone).
- **Published benchmark evidence:** [`guillermomolina/protos-benchmarks@ce67cbd358f5edd1dcc1056ed3540d5da13fcf32/results/i092-70ee9504-dirty-diagnostic/`](https://github.com/guillermomolina/protos-benchmarks/tree/ce67cbd358f5edd1dcc1056ed3540d5da13fcf32/results/i092-70ee9504-dirty-diagnostic). This commit also publishes the I091-compatible `ProtosSurfaceSupport.integerOutcome` interop projection patch.
- **Earlier reference:** [`results/i092-7b609c4a-graphs`](https://github.com/guillermomolina/protos-benchmarks/tree/8c91b495ac82ebe4a345ef02aa0db43fa5afe8c1/results/i092-7b609c4a-graphs). Original old-producer snapshot recorded harness HEAD `527035f01107e6c71f860e95f438b7553a5428a2`; new capture recorded harness HEAD `8c91b495ac82ebe4a345ef02aa0db43fa5afe8c1`, both with `harness_dirty=true` as initially captured.
- **Previous native implementation checkpoint:** [`I092_NATIVE_CONTROL_70ee9504_PUBLICATION_CHECKPOINT.md`](I092_NATIVE_CONTROL_70ee9504_PUBLICATION_CHECKPOINT.md). Its historical `NEW_GRAPH_ACCEPTANCE=PENDING` statement refers to the time of publication and is intentionally not silently rewritten.

## 1. Graph admission, scope and comparison

Maintainer executed graph capture with `--allow-dirty-product`, `analyze` using the pinned published IGV image and `summarize`. All three cases reported `CORRECTNESS=PASS`, `CAPTURE_INVALID_CASES=0`, `evidence_valid=true`, stable tier 2 and selected `After TruffleTier` at `budget=64000`; natural-warmup stable pair `[16000,64000]`. Six compressed BGVs were successfully analyzed (`ANALYZE_FAILURES=0`); `INVALID_UNITS=0`.

Within each capture, `source_state_sha256` was unchanged before/after; across the three cases it was the **same** exact `abf80400...` state. An earlier attempt was properly invalidated when I091 modified source during capture and is not substituted for the valid set.

The **same** canonical workload sources, GraalVM CE 25.4.4.1.1 / Java 25.0.4.1.1, host (AMD Ryzen 7 3700X, 16 logical CPUs visible), pinned CPU 0, JVM graph options, `run.execute()` surface, budgets, and selected phase are recorded for both versions. Among the recorded harness producer SHA-256 values, only `truffle/measure/surfaces/protos/ProtosSurfaceSupport.java` changed: from `integer.value().toString()` to exact Truffle `InteropLibrary.asBigInteger` result normalization to remain compatible with I091. This change is in correctness/result projection, not the guest benchmark workload. Product code and source state differ; those deltas are **not controlled independently**.

| Workload | `7b609c4a` clean-product reference | `70ee9504` + dirty I091 effective source | Delta |
| --- | ---: | ---: | ---: |
| `primitive-if-true` | 2,173 | **5,441** | **+3,268 (+150.4%)** |
| `primitive-if-false` | 444 | **1,031** | **+587 (+132.2%)** |
| `primitive-return-literal` | 39 | **39** | **0** |

These are **compiled IR node counts**, not elapsed-time/per-call cost or proven I092-only regressions. Uncommitted product content makes the new corpus **diagnostic, not a publishable clean-product A/B of I092**. There are also three committed I091 changes (`2a36e671`, `7dfceb7e`, `0fec0483`) between the two product commits, prior to `70ee9504`. Current source bytes must be isolated before attributing change to I092.

Maintainer reported before the benchmark-results commit `CASES=3`, `WORKING_TREE_MATCHES_PRODUCER=YES` and `HEAD_MATCHES_PRODUCER=NO` solely for `ProtosSurfaceSupport.java`; the producer patch and full evidence were then published in benchmark commit `ce67cbd3`. **Post-publication `HEAD_MATCHES_PRODUCER=YES` was not separately reported**; do not invent its result.

## 2. Structural differences in the retained `unit.json` files

| Selected graph class / family | `ifTrue` before → after | `ifFalse` before → after |
| --- | ---: | ---: |
| `FrameState` | 393 → 953 | 67 → 184 |
| `IfNode` | 127 → 320 | 31 → 54 |
| `BeginNode` | 269 → 688 | 62 → 114 |
| `LoadFieldNode` | 90 → 288 | 11 → 42 |
| `FixedGuardNode` | 122 → 276 | 23 → 52 |
| `VirtualInstanceNode` | 14 → 123 | 4 → 30 |
| `AllocatedObjectNode` | 24 → 115 | 2 → 22 |
| `InvokeWithExceptionNode` | 9 → 46 | 0 → 8 |
| `LoopBeginNode` | 0 → 6 | 0 → 3 |

The old `ifTrue` graph attribution includes `TryCompactCanonicalBooleanOne_Node` (guarded candidate). The new graph lists `CachedBytecodeNode` with 3,560 source/expansion occurrences (versus 736 previously), `PrepareStructuredBooleanCall_Node`, and `PrepareSendOne_Node`; old `ifFalse` is dominated by `MaterializeCurrentClosure_Node` and `TryCompactCanonicalBooleanOne_Node`, whereas new `ifFalse` includes `PrepareSendOne_Node` and `PrepareStructuredBooleanCall_Node`. **Do not sum source-expansion buckets as if they were distinct final nodes; some names recur, and expansion provenance is not executable-path frequency.**

New `ifTrue` selected-graph residual invoke targets include `ProtosFrameArguments.materializeCompactActivation`, `ProtosActivation.inheritDynamicControlState`, `ProtosLanguageContext.currentIfEnteredForRuntime`, `ProtosLexicalBindingAuthorityCalls.read`, and `ProtosCoreErrors.selectMatchingHandlerFrame`. New `ifFalse` retains some of these despite the unselected callback. Their presence **does not prove that they execute on the successful canonical hit**, or that there is an intermediate standard `ifTrue` activation (I092's runtime/bytecode design explicitly removes that carrier). They can belong to cold fallback, deoptimization/exception paths, or live paths: this awaits edge-level attribution.

The unchanged `primitive-return-literal` histogram (39 nodes) rules out a wholesale change to the minimal root graph in the measured cases. It does **not** isolate I091 effects on Boolean-specific paths.

## 3. Source-level findings and hypotheses

Read pinned `guillermomolina/protos@7b609c4a` and `@70ee9504` source:

1. Former `ProtosSemanticBytecodeRootNode.TryCompactCanonicalBooleanOne` used guarded `@Specialization` with cached receiver, selector, context, Prelude and selected `Assumption` objects (limit 3). The new version performs direct canonical Boolean identity checks within one specialization and calls `nativeIfTrueCallback` only for true. The frozen standard root and its validated `ifTrue`/`call` publication support the semantic transformation. **Hypothesis H1:** the new physical shape is less PE-constant on the actual branch even though its guest semantics are correct; this is not established by source alone.
2. New `CanonicalToBytecodeLowerer.emitCanonicalIfTrueLiteralSite` physically emits distinct native-hit and full generic-miss branches, preserves one B-prime callback source region and surrounds the shared execution with `TryFinally` / `CompleteIfTrueSiteOuter`. **Hypothesis H2:** reachable generic/exception/deopt code retained by Bytecode DSL/Graal accounts for a significant fraction of the increased graph. The native Boolean hit need not traverse that code at execution time. The Bytecode DSL specification notes that `TryFinally` finalizer bytecode can be repeated on distinct exits; the IR growth contribution here remains unmeasured.
3. **Hypothesis H3:** I091's committed primitive-number lowering/carrier changes, plus the subsequently uncommitted numeric edits, materially change the interpreter branch graph independently of the I092 lowering. Both commits and uncommitted source changes are currently confounded, so this must be isolated instead of asserted.
4. **Hypothesis H4:** the difference between `true` and `false` is partly selected callback preparation and inline-body execution; source-producer/graph edge data must distinguish materialization of the callback *expression* (still semantically eager) from invocation of the selected callback (false must not execute it).

The new I092 test `ProtosI092CompactBooleanTest.canonicalAndOverrideExecuteOppositePhysicalBranches` verifies that canonical true hits the native branch and custom receiver enters fallback for the `receiver.ifTrue(() => { 41 })` shape. It is **not** a substitute for edge-level PE proof for the exact benchmark shape `true.ifTrue() { 1 }` / `false.ifTrue() { 99 }`; both workloads use the original catalog syntax.

## 4. Research gate and next slice

**Next:** `I092-GRAPH-A1` — **INVESTIGATION ONLY** under the existing open I092; do **not** allocate a new issue or reopen the completed PERF041. No product edits, tests, benchmarks or runtime execution; compare retained artifact and GitHub source data by read-only means.

Research must:

1. Examine the retained `*.filter.json.gz` IGV **graph topology** (not only `unit.json` histograms) for both old/new true and false; construct the canonical-entry → identity selection → native branch → B-prime callback vs generic-miss → prepared dispatch and exception/deopt paths. Use edge/block/source-position evidence and identify graph IDs, not speculative normal-path claims. If compressed JSON cannot be read without execution/access, explicitly stop with smallest human-run diagnostic; never infer the live path from invoke names.
2. Compare pinned source and exact `0fec0483` (after three I091 commits/before I092) against `70ee9504`; distinguish H1–H4 and the uncommitted effective `abf80400...` product state. Propose source-identity isolation/controlled measurement **only if necessary**; do not execute it in the investigation. Historical BGVs cannot reconstruct the uncommitted changes from the product commit alone.
3. Establish semantic constraints from `spec/semantics/{CALLABLES,EXECUTION_AND_CONTROL,OBJECT_MODEL,VALUES_AND_COLLECTIONS}.md`, D049 standard-root freeze, PLAT040 F′/PLAT043/PLAT044 B′, and the verified I092 override/ReturnHome/capture/tooling cases. No benchmark-only constant folding, implicit truthiness, suppressed callback creation, or changed ordinary `call` selection.
4. Return a cause-labelled node/invoke/deopt/activation attribution with confidence levels; explicitly separate live canonical execution, cold fallback, `FrameState` and exceptional machinery, plus I091 effects. If evidence is sufficient, recommend **one cohesive bounded I092 implementation slice** and all required falsification/tests. If not, state the exact discriminator and stop; don't propose speculative code edits or new D/PLAT approval.

**Current disposition:**

```text
I092_NATIVE_SOURCE_PUBLICATION=70ee9504224c59c033c647f9185bf8fb9b266b93
BENCHMARK_EVIDENCE_PUBLICATION=ce67cbd358f5edd1dcc1056ed3540d5da13fcf32
VALID_FINAL_TIER_GRAPHS=3
PRODUCT_SOURCE_STATE=abf804007e73e6c7e2783c337d2b68c8b3029f7519907064d50e99d92a510c0e
PRODUCT_DIRTY=YES
SAME_PRODUCT_SOURCE=YES
CLEAN_I092_CAUSAL_A_B=NO
LIVE_BGV_EDGE_ATTRIBUTION=NOT_ESTABLISHED
NEXT_SLICE=I092-GRAPH-A1
TYPE=INVESTIGATION
IMPLEMENTATION_AUTHORIZED=NO
SPEC_CHANGE=NO
ISSUE_NEW=NO
ISSUE_STATE=OPEN
```
