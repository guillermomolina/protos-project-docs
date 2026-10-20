# I115-H / PLAT066-C — experimental static Closure invocation, graph gains and semantic-divergence handoff

**Date:** 2026-10-20. **Classification:** experimental implementation evidence, **not** a ratified Protos language/platform change, release acceptance, or runtime-throughput benchmark.
**Product:** [guillermomolina/protos at c37b572dd41d84ec15448efbc4fa6f4ceef11b0c](https://github.com/guillermomolina/protos/commit/c37b572dd41d84ec15448efbc4fa6f4ceef11b0c), published to branch **I115** (not asserted merged to main). Commit message: "I115-H: invoke proven local Closure calls statically with a minimal ABI and lightweight non-capturing Closures". Parent: bca51b91d1d144d815381bd66d46edf9f67fb0f2. The branch POM currently reads 0.3.389-SNAPSHOT, inherited from prior product work; I115-H did **not** change pom.xml/CHANGELOG.md or publish a versioned product release.
**Owners:** [I115 #931](https://github.com/guillermomolina/protos/issues/931) (implementation), [PLAT066-C #930](https://github.com/guillermomolina/protos/issues/930) and [PLAT066 #924](https://github.com/guillermomolina/protos/issues/924) (research/architecture), [PERF042 #871](https://github.com/guillermomolina/protos/issues/871) (measurement). All remain open.
**Provenance:** implementation files and physical root/test code were read from exact published branch head. The physical tier logs and DSL census were **pasted by the maintainer in chat**, before the reported push. The logs do not include a checkout SHA, and the BGV binary files were not attached or hashed here; therefore the measured candidate is not independently attested as byte-for-byte identical to the subsequently pushed commit. Do not upgrade to exact-commit BGV provenance without a fresh capture/hashes.

## 1. Why I115-H was performed

The matched 2026-10-20 cold benchmark is:

~~~protos
run: () => {
    identity: (value) => { value }
    identity(1)
}
~~~

The physical source is **three roots**: module, run, identity. A prior frozen Protos 0.3.386 baseline measured module/run/identity **16/112/5 = 133** FULL instructions, versus **12/12/5 = 29** for GraalPy JVM 25.4.4.1.1. A previously published intermediate I115 lexical C1 experiment brought run to **18 FULL** with a conventional `InvokeClosureOneTail` and run-only After TruffleTier **1119** Tier 1 / **922** Tier 2; see [the intermediate I115 graph acceptance](I115_PLAT066C_LEXICAL_C1_TERMINAL_RUN_GRAPH_ACCEPTANCE_2026_10_20.md). Subsequent reported I115-G optimizations reduced run to **10 FULL**, and module/run/identity to **16/10/5 = 31**; earlier chat-reported I115-G run-only compiler measurements were After TruffleTier **472/362** (Tier 1/Tier 2). That latest G measurement has no independently inspected archived BGV in this record; it must not be conflated with the earlier 18/1119/922 snapshot.

I115-H intentionally explores how far Protos can converge toward or beat the GraalPy physical path **if selected Protos semantics and platform requirements are bypassed**. These bypasses are experiment-only and are not ratification of a new normative contract.

## 2. Implementation: four grouped changes

| Change | Physical change | Files / contract |
| --- | --- | --- |
| **H1: static invocation** | A new `InvokeStaticClosureOneTail` Bytecode DSL operation takes a constant execution-plan cell and the one actual argument. Direct PIC caches a Context-owned RootCallTarget and DirectCallNode; generic performs an indirect call to the same plan. Ordinary D013 `Object.call` lookup is bypassed. | `ProtosBytecodeRootNode.java`; `CanonicalToBytecodeLowerer.java` |
| **H2: lightweight noncapturing Closure** | `ProtosClosureValue.experimentalNonCapturing` holds definition, plan and root receiver while omitting creator environment/receiver, methodHome, ReturnHome and Prelude. Allowed only when canonical facts prove **no creator-context capture AND no ReturnHome observation**. Capturing cases retain I115-G paths. | `ProtosClosureValue.java`, `ProtosFrameArguments.java`, `MaterializeCurrentClosure` |
| **H3: elide nonescaping creation and binding** | `CanonicalCompilationFacts.StaticLocalClosureCall` admits the special local create/terminal-call pattern, based on exact binding identity, no other binding use/escape, no creator capture, one non-spread supplied argument, one required parameter without rest/default, and callee body equal to one literal or direct lookup of that parameter. No Closure object or local binding is emitted for the admitted site; the independent callee root is preserved. | `CanonicalCompilationFacts.java`, `CanonicalToBytecodeLowerer.java` |
| **H4: experimental two-element ABI** | `experimentalStaticUnaryCall(supplied0)` produces `[privateStaticMarker, supplied0]`, not B5+ five header slots. Static callee parameter binding and root-frame read consume this layout. No caller activation, Closure object, Task, ReturnHome, methodHome or receiver is transported. | `ProtosFrameArguments.java`, `ProtosBytecodeRootNode.java`, lowerer |

The new static path is not used for arbitrary Closure calls. The generic specialization still requires a Context-local static plan/target and raises an internal admission contradiction if it is unavailable or produces a suspended outcome; it does **not** fall back to D013. The direct tier requires a non-suspension proof, and the null-safe `createDirectCallNodeOrNull` initializer was added to avoid building a DirectCallNode with a null target. Reparse/root-plan state is reset as part of the lowering path.

**Product commit changed seven files:** five source files (`ProtosBytecodeRootNode.java`, `ProtosFrameArguments.java`, `CanonicalCompilationFacts.java`, `CanonicalToBytecodeLowerer.java`, `ProtosClosureValue.java`) and two tests (`ProtosI115HStaticClosureCallTest.java`, `ProtosI101CIndependentBuilderReferenceTest.java`). An earlier locally edited `docs/experiments/i115-h-static-closure-invocation.md` was discussed in the chat but is **not among the files in the verified product commit**; this project-record file is the durable report.

## 3. A real regression was discovered and corrected before push

The ordinary, **non-static** program:

~~~protos
outer: () => {
    inner: (value) => { ^value }
    inner(5)
    0
}
outer()
~~~

must return **5** because `^value` returns nonlocally from `outer`. Initially H2 classified inner as noncapturing and dropped its creator ReturnHome, causing the program to return **0**. This was a real behavioral regression, not an accepted experimental divergence.

**Repair actually present in pushed source:** `CanonicalToBytecodeLowerer.beginClosureCapture` now emits `compilationFacts.mayObserveReturnHome(closure)` as a third per-site constant operand. `MaterializeCurrentClosure` and `materializeCompactClosure` choose lightweight creation **only** when `!capturesCreatorContext && !mayObserveReturnHome`; when ReturnHome may be observed they restore the I115-G creator-home-preserving construction and the previous compact environment reuse. Tests include `nonCapturingClosurePreservesNonLocalReturnHome`, `nonLocalReturnBodyIsNotStatic`, and `staticUnaryExecutesInUncachedInterpreter`.

**Validation authority:** the maintainer reports **all local tests PASS** after correction and before push. The agent did not run the tests or independently read a complete integrated `make test` transcript. No claim of all semantic edge cases, native-image validation or throughput performance follows from this report.

## 4. Matched DSL FULL result (physical roots, not one Graal graph)

Human-pasted I115-H census was reproduced identically in the Tier 1 and Tier 2 outputs:

| Physical root | Frozen Protos 0.3.386 | Later I115-G reported | **I115-H** | GraalPy JVM comparator |
| --- | ---: | ---: | ---: | ---: |
| Module | 16 | 16 | **16** | 12 |
| `run` | 112 | 10 | **5** | 12 |
| `identity` | 5 | 5 | **5** | 5 |
| Sum of independent roots | 133 | 31 | **26** | 29 |

- `run` versus I115-G: **10 → 5 (−50.00%)**, whole three-root sum **31 → 26 (−16.13%)**.
- `run` versus frozen 0.3.386: **112 → 5 (−95.54%)**; whole three-root sum **133 → 26 (−80.45%)**. These are cumulative improvements over multiple I115 slices, **not all attributable to H**.
- I115-H has fewer **aggregate** FULL instructions than GraalPy (**26 < 29**), but the module is still larger (**16 > 12**) and identity ties (**5 = 5**).

Exact FULL of the **run** root:

~~~text
c.EnterRoot
load.constant
c.InvokeStaticClosureOneTail
c.EpilogForReturn
return
~~~

The identity root remains separate and has `c.EnterRoot`, `c.BindUnaryFrameParameter`, `c.ReadRootFrameLocal`, `c.EpilogForReturn`, `return`. Counting separate Bytecode roots does not constitute a total Graal IR count.

## 5. Matched compiled run-root measurements

Both opt-in JUnit `ProtosPerf042IdentityBuilderOracleTest#captureNestedRunRootAtRequestedTier` runs returned **RC=0**, compiled the selected physical `run` target, and reported **`PERF042_NESTED_RUN_CAPTURE=READY`**. Tier 1 and Tier 2 were separate JVM captures with **128 warmup calls**, `BytecodeTier=CACHED`. Stable run-root selector: `protos-root:83c047c2b4bd1384`; program source `perf042-nested-run-identity-program.protos`, source extent **17+46**.

| Run-root metric | I115-G earlier chat-reported | **I115-H** | GraalPy JVM frozen comparator |
| --- | ---: | ---: | ---: |
| Tier 1 After TruffleTier | 472 | **152** | 268 |
| Tier 1 final IR | 1,141 | **264** | 731 |
| Tier 1 machine CodeSize | 5,641 B | **1,030 B** | 3,544 B |
| Tier 2 After TruffleTier | 362 | **69** | 165 |
| Tier 2 final IR | 373 | **103** | 420 |
| Tier 2 machine CodeSize | 1,944 B | **458 B** | 2,188 B |

- Relative to the earlier reported I115-G: After TruffleTier nodes **−67.80%** Tier 1, **−80.94%** Tier 2; final IR nodes **−76.86%** and **−72.39%**. Comparability is provisional because that G capture's raw artifacts/revision are not independently preserved in this record.
- Relative to GraalPy for the matching run workload: After TruffleTier **−43.28%** Tier 1, **−58.18%** Tier 2; final IR **−63.89%** Tier 1, **−75.48%** Tier 2. Code sizes are **−70.94%** and **−79.07%**.
- Relative to the **frozen Protos 0.3.386** run root, After TruffleTier was **7137 → 152** Tier 1 (−97.87%) and **7078 → 69** Tier 2 (−99.03%); these are **cumulative** improvements across several slices, not just H.
- Tier 1 log reports **`Inlined 0Y 1N`**; Tier 2 **`Inlined 1Y 0N`**. Static lowering retains a separate physical identity root and leaves actual Graal inlining to the compiler.

**Captured-file evidence as reported, not archived in this repo:**
- Tier 1: `CompId 3505`, `opt done |Tier 1| IR 152/264 |CodeSize 1030`, attributed BGV count **1**, reported BGV size **568,696 bytes**, named `TruffleHotSpotCompilation-3505[protos-root:83c047c2b4bd1384...].bgv`.
- Tier 2: `CompId 3514`, `opt done |Tier 2| IR 69/103 |CodeSize 458`, attributed BGV count **1**, reported BGV size **596,186 bytes**, named `TruffleHotSpotCompilation-3514[protos-root:83c047c2b4bd1384...].bgv`.
- The root census in both logs reports **module 16**, **run 5**, **identity 5**, **TOTAL_FULL 26**. The initial failed capture (due to the globally applied experimental `TraceNodeExpansion` JVM option during Core bootstrap) generated no BGV; the corrected capture omitted that global option and passed.

These are graph/code-size measurements **only**: no claim of runtime throughput, latency, allocation rate or energy benefit.

## 6. Explicit semantic and platform departures

The normative authority remains `spec/semantics/CALLABLES.md` (Closure creation/capture, calls, ReturnHome) and `spec/semantics/EXECUTION_AND_CONTROL.md` (contexts/lookup/evaluation), plus ratified D013/D179 and PLAT060-B5+ platform decisions. Experimental tests pin an isolated performance mechanism; they do **not** approve new Protos semantics.

| Slice | Contract deliberately bypassed | Scope / risk / status |
| --- | --- | --- |
| **H1 — dynamic call selection** | D013's ordinary canonical/dynamic `Object.call` selection, instance override and its Assumption/invalidation/fallback machinery. | The static admitted site calls the definition-owned target without selecting `Object.call` after evaluating arguments. This is a **real change in dispatch behavior in principle**, not evidence of equivalent semantics for mutable method selection. The test `experimentalStaticCallHasNoSelectorAssumption` is structural and **does not** dynamically mutate `Object.call`. No normative acceptance. |
| **H2 — creator capture** | Ordinary Closure captures the creator's lexical context, receiver/this provenance, methodHome, ReturnHome and relevant prelude relationships. | On the proven noncapturing/non-ReturnHome-observing case the lightweight instance omits metadata and substitutes a root receiver; the missing data can affect contexts, method binding/super, receiver fallback, reflective/escaped behaviors and subsequent composition outside the narrow proof. This omission is **experimentally accepted as a divergence**, not generally proven harmless. |
| **H3 — Closure object/binding identity** | Creation of an ordinary Closure value, fresh identity, its binding/slot and potential later observation/mutation, plus normal activation/dispatch selection. | The admitted pattern constructs **no Closure object** and no binding at all. Structural binding analysis forbids other canonical use/escape, reducing but not ratifying the observable-equivalence question. This is an intentional identity/mutation/call-selection departure. |
| **H4 — universal invocation ABI** | Ratified universal B5/B5+ five-header-slot ABI for ordinary Closure invocations, including caller/activation and Task, receiver/methodHome/ReturnHome/Context transport. | The new special route uses `[private marker, supplied0]` only (two slots) and specially adapted consumers; no general B5+ compatibility proof. No caller/Task/ReturnHome carrier on this path. The admitted body is deliberately restricted to make these omissions hard to observe, but the platform-contract violation is explicit. |
| **H1/H4 — suspension and error transport** | Ordinary C′/Task/continuation and fallback rules for dynamically selected or suspending targets. | The direct tier requires a non-suspending destination; generic throws an internal contradiction on suspension instead of ordinary continuation handling. This is an intentional experiment-only assumption and **not** a replacement for general Task semantics. |
| **Retained protection — `^`** | Smalltalk-style nonlocal return targets captured creator ReturnHome, including `InvalidReturn` cases. | A real H2 regression (outer returned **0** instead of **5**) was **fixed** before push by `mayObserveReturnHome`. The static predicate excludes `^` bodies. This regression is **not** an approved deviation. Ordinary tested nonlocal returns now follow the I115-G route. |

**Important distinction:** listing a skipped obligation is not a proof that every listed divergence is externally observable in the admitted tiny program. The experiment deliberately drops contract machinery; full normative equivalence has **not** been shown. None of the experimental deviations changes `spec/` or ratifies D013, D179, C′ or B5+. I115-H is not ready for direct merge as a semantics-preserving production optimization merely because it produces smaller graphs and local tests pass.

## 7. Validation and coordination handoff to PLAT066-C

**Verified remotely:** branch I115 points to product SHA **c37b572dd41d84ec15448efbc4fa6f4ceef11b0c**, whose commit includes the five implementation and two test paths above. **Reported by maintainer:** all local tests PASS; opt-in Tier 1/2 test invocations RC=0, the exact printed root census and compiler `opt done` summaries above. **Not independently performed by this agent:** build/test rerun, integrated make-test transcript, Native Image acceptance, BGV parsing, allocation/profiling or throughput benchmark.

**PLAT066-C handoff:**
1. Treat the **5 FULL / 152 Tier-1 / 69 Tier-2** run-root results as an **experimental lower-bound architectural data point**, not ratified Protos semantics or a speed benchmark.
2. For each H1–H4 difference, establish the exact observable contract, and classify whether it can be recovered using reliable proofs/guards or whether it unavoidably adds nodes.
3. Pay particular attention to D013 changes **after argument effects** (the static shape's no-use proof does not itself validate future prototype/instance `call` mutation), full Closure creator capture/methodHome/Prelude, fresh identity and binding, Context/Task/ReturnHome and suspension/error transport.
4. Compare against GraalPy on the same source, runtime, warmup, selected physical root and tier, while retaining the original **0.3.386 cold** and the separately reported **I115-G** snapshots as different baselines.
5. No new D/PLAT ratification or I implementation mandate follows from this evidence alone. Keep [PLAT066-C #930](https://github.com/guillermomolina/protos/issues/930), parent [PLAT066 #924](https://github.com/guillermomolina/protos/issues/924), [I115 #931](https://github.com/guillermomolina/protos/issues/931) and [PERF042 #871](https://github.com/guillermomolina/protos/issues/871) open pending further owner decisions.
6. The pre-existing native GitHub parent-issue linkage problem for PLAT066-C remains unresolved; evidence publication must not be reported as its repair.

**Sources inspected:** product I115 commit and five implementation files + test `ProtosI115HStaticClosureCallTest`; the canonical `CALLABLES.md` and `EXECUTION_AND_CONTROL.md`; project evidence frozen 0.3.386 PERF042 baseline and prior I115 lexical C1 checkpoint; issue I115/PLAT066-C/PLAT066/PERF042 descriptions; maintainer-provided I115-H capture logs and all-local-tests statement. Source snapshot status and human-reported observations are explicitly separated throughout.
