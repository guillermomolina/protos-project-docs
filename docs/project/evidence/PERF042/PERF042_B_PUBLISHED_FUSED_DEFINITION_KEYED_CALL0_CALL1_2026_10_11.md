# PERF042-B — Published fused definition-keyed direct Closure Call0/Call1

Date: 2026-10-11
Formal owner: [PERF042/#871](https://github.com/guillermomolina/protos/issues/871)
Classification: **published implementation checkpoint with maintainer-reported human validation; not compiled-graph or performance closure**
Product commit: [`1c414e08fefc2378b828bba7be7d263ea32e10ce`](https://github.com/guillermomolina/protos/commit/1c414e08fefc2378b828bba7be7d263ea32e10ce)
Product parent: `ced3746f9664ec5ceb562461c077bdf0627e5663`
Implementation version: **`0.3.332-SNAPSHOT`**
Prior authorized scope: [PERF042-A3 retained-trace readback and B handoff](PERF042_A3_RETAINED_COMPILATION_TRACE_READBACK_AND_IMPLEMENTATION_HANDOFF_2026_10_10.md)

## 1. Exact published scope and provenance

GitHub commit `1c414e08fefc2378b828bba7be7d263ea32e10ce`, verified on `guillermomolina/protos/main` on 2026-10-11, has the single-line title:

> PERF042: reuse the fused direct Closure Call0/Call1 target across fresh instances of one definition

It changes exactly these product files:

- `src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java` — fused direct Closure-call specialization;
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf042DefinitionKeyedDirectCallTest.java` — new focused regressions and structural cache-entry tests;
- `pom.xml` — `0.3.331-SNAPSHOT` to `0.3.332-SNAPSHOT`;
- `CHANGELOG.md` — release-history entry describing PERF042-B.

No `spec/`, workload, benchmark-harness or other Protos implementation file is changed by this commit. This implementation is separate from the concurrently preceding I091 revision `ced3746f9664ec5ceb562461c077bdf0627e5663`.

## 2. Implementation, correctness gates and retained fallback

The fused Bytecode DSL operations `TryDirectCallZero` and `TryDirectCallOne` retain their original `guardedSource` instance-identity specialization and gain `definitionSource`, a second specialization with limit 3. It caches the exact `CanonicalClosure` definition, prepared execution-plan template, whether Context-local projection is required, entered `ProtosLanguageContext`, admitted `RootCallTarget`, and `DirectCallNode`.

The cached target is derived only after checking the instance's canonical ordinary `Object.call` selection and its unobservable ReturnHome marker. A source Closure without an already prepared plan does not enter this tier or invoke its deferred plan rematerializer; a noncanonical `call` override remains authoritative. Cached instances are rechecked on each hit, and the frame receives the **current Closure**, exact caller and (for Call1) the current scalar `supplied0`. Call0 retains the no-vector compact frame. Existing non-suspension and exact-arity target checks and the control-transfer exception bridge remain in place.

A `rejectedSource` MISS specialization and a generic MISS specialization without destructive `replaces` preserve existing PIC entries when encountering temporarily unadmitted instances. The cached-only specializations use `excludeForUncached = true` so the uncached Bytecode DSL path continues through fallback. Target-null protections fail closed rather than manufacturing an invalid `DirectCallNode`.

The source correction covers the A/B/C/D review boundaries: nullable target initialization, physical target keying for different templates/projections, no eager plan rematerialization before ordinary `call` selection, and no permanent PIC loss after transient rejection. It does not claim that the extra plan/projection guards are free or redundant.

## 3. Tests in the published commit

The new `ProtosPerf042DefinitionKeyedDirectCallTest.java` contains **10** JUnit `@Test` methods (counted directly from published source):

1. `freshCallOneClosuresConvergeOnTheDefinitionTier`
2. `freshCallZeroClosuresConvergeOnTheDefinitionTier`
3. `freshInstancesKeepTheirOwnIdentityAndCapturedEnvironment`
4. `callOverridesAreObservedBeforeAndAfterWarmUp`
5. `admissionBoundariesRejectAndFallBack`
6. `definitionTierIsInstalledAtTheCanonicalCallSite`
7. `distinctExecutionPlansKeepDistinctDefinitionTierTargets`
8. `foreignContextClosureProjectsATargetOwnedByTheEnteredContext`
9. `overriddenCallDoesNotPrepareAnUnavailableOriginalPlan`
10. `transientRejectionsKeepTheDefinitionTierEntry`

The tests cover canonical `identity(1)` and C0 workloads, instance identity/capture, ordinary `call` overrides, arity/ReturnHome/non-suspending rejection, distinct plan targets, foreign Context target projection, deferred-plan override and retention after transient rejection. The structural tests inspect the generated `definitionSource_cache` on the actual Bytecode call-site node, verifying installation/retention and its target. Reflection on generated field names is deliberately test-local; the test does not count hot-path invocations.

**Human validation report:** after publication, the maintainer explicitly reported `git diff --check` clean and **all local tests PASS**. Earlier in the implementation cycle the maintainer separately reported compilation and focal tests PASS; those earlier reports are not substituted for a new exact test-run manifest. This documentation agent did **not** rerun Maven, `make test` or the product in this checkpoint. There is no independently captured test-run log, test timestamp bound to the commit, or new performance measurement. No claim is made about a particular final execution count beyond the 10 methods present in the source.

## 4. Scope deliberately not measured or changed

PERF042-B addresses fresh-Closure identity churn in fused Call0/Call1 only. It does **not** implement the broader H2/H3 persistent lexical-authority or caller/activation redesign, redefine callable semantics, remove all invocation taxes, or demonstrate a graph reduction.

The valid **historical**, pre-PERF038-H clean standalone guest primary Tier-2 graph at product `d59da442fd9bc6cf590fb9b2c82d3ff55c0391ae` / harness `98abc9af7a05a45a7d4056b72f36aef5889cb76f` measured 4,533 nodes (versus 13 GraalJS, 50 GraalPy), as recorded in the A-series reports. A later dirty-harness guest-root graph was not eligible as a stable Tier-2 baseline, even though its host `Value.execute` compiled at Tier 2 with both guest roots inlined. **Neither historic number is a PERF042-B A/B result.**

The structural entry test does not prove how often the optimized path is used in compiled code, what nodes remain, or a speedup.

## 5. Remaining PERF042/#871 acceptance and next action

The bounded PERF042-B source change is published with maintainer-reported green validation; **the top-level issue remains open** (`status:ready`, priority intentionally unset), and its graph/performance acceptance criteria are not yet satisfied.

Next, perform one revision-pinned post-publication measurement in `guillermomolina/protos-benchmarks` for exact product commit `1c414e08fefc2378b828bba7be7d263ea32e10ce` and a clean, identified harness revision, preserving the existing comparable runner policy. Require canonical `primitive-closure-call` correctness and workload equivalence first. Establish an admitted stable **standalone final-tier Tier-2** graph, or an independently actionable verified compiler-admission blocker. Retain root/tier/inlining/deopt details, exact nodes/invokes/`FrameState`/`IfNode` counts and an appropriate A/B timing comparison with pinned versions and same-policy GraalJS/GraalPy.

Report what graph edges and physical costs actually survive rather than asserting that the identity PIC accounts for all 4,533 historical nodes. Publish evidence to `protos-benchmarks`, then the durable causal/conclusion record here, and update [PERF042/#871](https://github.com/guillermomolina/protos/issues/871). Close the parent issue only if its complete original acceptance criteria are met.

## 6. Sources read for this checkpoint

- Exact [product commit](https://github.com/guillermomolina/protos/commit/1c414e08fefc2378b828bba7be7d263ea32e10ce), parent, changed-path list and published `pom.xml`/`CHANGELOG.md`;
- exact-revision `ProtosSemanticBytecodeRootNode.java` and `ProtosPerf042DefinitionKeyedDirectCallTest.java`;
- [live PERF042/#871](https://github.com/guillermomolina/protos/issues/871);
- [A3 retained trace and implementation authorization](PERF042_A3_RETAINED_COMPILATION_TRACE_READBACK_AND_IMPLEMENTATION_HANDOFF_2026_10_10.md).

Documentation changes here are historical implementation evidence, **not** a new normative design approval or independent benchmark result.
