# I115-C/D — 0.3.385 publication and PLAT066-C physical-convergence handoff

**Record role:** historical implementation, validation and graph-investigation evidence; neither a normative language-specification edit nor a new platform ratification.  
**Date:** 2026-10-19.  
**Implementation issue:** [I115 #931](https://github.com/guillermomolina/protos/issues/931).  
**Research owner:** [PLAT066-C #930](https://github.com/guillermomolina/protos/issues/930), under [PLAT066 #924](https://github.com/guillermomolina/protos/issues/924); broader [PERF042 #871](https://github.com/guillermomolina/protos/issues/871).  
**Product repository:** `guillermomolina/protos` (exact spelling, two `mo`).  
**PROTOS_REVISION:** [`e2200088d42d0d8be90626a79b373ddad64978f5`](https://github.com/guillermomolina/protos/commit/e2200088d42d0d8be90626a79b373ddad64978f5) on `main`, `0.3.385-SNAPSHOT`.  
**Implementation change commit:** [`4f43b696f06921800fbc60bdd464ad47fd0d8e1f`](https://github.com/guillermomolina/protos/commit/4f43b696f06921800fbc60bdd464ad47fd0d8e1f) (seven source/test paths).  
**Integration commit:** [`96b0676a49a473a6d107f94e7b06cb20a5c6ea41`](https://github.com/guillermomolina/protos/commit/96b0676a49a473a6d107f94e7b06cb20a5c6ea41), merging I116-B into the I115-C/D branch.  
**Metadata publication:** `e2200088` updates `pom.xml` and `CHANGELOG.md`, after the human-reported integrated green suite.

## Published physical implementation

The completed I115-C/D scope is the five-point second wave previously approved in [PLAT066 wave-2](../../work/PLAT066/PLAT066_WAVE2_FIVE_POINT_CONVERGENCE_2026_10_19.md).

- **I115-C:** invalidatable receiver-local absent-`call` selection certificate and canonical root-`Object.call` assumption; Context-projection stability under its own Assumption; ReturnHome non-observability certified from the correct Closure instance and prepared template. Cached definition/target is not a license to reuse another instance's captures, home or mutable selection.
- **I115-D:** compact admitted terminal Call1 B5+ frame-argument path and conditional propagation of the internal numeric result carrier to the existing root return epilogue.
- **Preserved obligations:** D013 ordinary polymorphic `call` selection after evaluating arguments; local/inherited overrides and invalidation, generic fallback, ReturnHome/non-local control, Context identity, deferred plan handling, Task/interop, five-slot B5+ header and canonical guest-visible numeric results. No new `@Operation`, no guest execution-mode split and no normative spec change are claimed.
- **Version:** `0.3.385-SNAPSHOT` is an implementation snapshot; this record does not authorize a release or tag.

## Human-executed validation and publication

The maintainer provided the following exact outcome evidence; the agent did not execute the product suite:

1. On recovered `experiment/i115-cd` (I115-C/D without D207), `git diff --check` was clean and focused Maven tests reported `I115_RECOVERED_TEST_RC=0`. The named regression command covered `ProtosI115ARuntimeAdmissionTest`, `ProtosI115BDeferredCallerTest`, `ProtosPerf042DefinitionKeyedDirectCallTest`, `ProtosI112TailContinuationTest`, `ProtosI112TerminalCallBudgetTest`, and `ProtosI091PrimitiveCarrierRootsTest`.
2. The focused output reported `I108_ARGUMENT_MUTATION_AND_PIC_RECOVERY=PASS`; the source-root FULL budget remained **11 TOP_LEVEL + 5 identity Closure = 16**, two roots, with `I112_EXTRA_GROUP_ROOTS=0` and `I112_CEILING_EXCEEDED=false`.
3. `origin/main` at I116-B `41d45fbfc950f23b6ab56ae68fce08e854e5c0ea` was merged into the implementation branch (`96b0676a`). The deterministic impact selector returned `VALIDATION_IMPACT=FULL`, `AFFECTED_TEST_SET=ALL`, `FULL_TEST_SUITE=REQUIRED`.
4. The human reported **`make test -> PASS`** for that integrated code candidate; version and changelog were finalized afterward, with no post-metadata test assertion.
5. The maintainer reported `I115_MAIN_PUBLICATION=SUCCESS`, with local and remote `main` pointing to `e2200088` and a clean `git status --short`. GitHub independently exposes that exact product revision as the current `main` commit. No independent CI run, `make check`, Native Image result or speedup is inferred.

## Physical graph evidence and unresolved gap

The compared diagnostic is the bare two-root program:

```protos
identity: (value) => { value }
identity(1)
```

The immediately preceding I115-C/D experimental checkpoint was reported and analyzed at **645 TOP_LEVEL `After TruffleTier` nodes** and approximately **55 inclusive source-ancestry nodes associated with `InvokeClosureOneTail`**. The same diagnostic still emitted 11 + 5 FULL instructions. These measurements are historical captures made before final product metadata publication; no new BGV has been independently reproduced from `protos@e2200088`. The 55 nodes are *inclusive attribution*, not a disjoint set proven removable or a claim that 55 nodes execute on the direct hit.

For a *separately isolated, deliberately non-normative D013/D207 experiment*, the maintainer generated a Tier-1 BGV with `PERF042_RUN_CAPTURE=READY`, `PERF042_RUN_COMPILED_TIER=1`, one BGV and two root groups. Previous side-by-side BGV analysis reported **645 vs 645 After TruffleTier**, **55 vs 55 inclusive `InvokeClosureOneTail` ancestry**, and no class-distribution reduction after replacing ordinary Closure `call` selection with intrinsic Closure invocation. This is not a proof that all conceivable D013 changes can never help, but it supplies **no evidence to justify changing D013 for this workload**. The experimental code and D207-specific regression remain only on the preserved branch [`experiment/i115-cd-d207`](https://github.com/guillermomolina/protos/tree/experiment/i115-cd-d207), anchored at [`61006b57c9434e4a4a904ee971ae77c846c68497`](https://github.com/guillermomolina/protos/commit/61006b57c9434e4a4a904ee971ae77c846c68497). That branch is **not** merged into `main`; D013/#171 remains ratified; D207/#932 is an unresolved independent design investigation. The two graph files are not published here as binary attachments.

The implementation **I115-C/D is complete and accepted**, but the **PLAT066-C physical-equivalence/convergence objective is not complete**. Reopen/resume the research activity under the *already open* [PLAT066-C #930](https://github.com/guillermomolina/protos/issues/930), with [PLAT066 #924](https://github.com/guillermomolina/protos/issues/924) and [PERF042 #871](https://github.com/guillermomolina/protos/issues/871) still open. Do not open another implementation issue solely because residual nodes exist; research first.

## Research handoff — PLAT066-C

**TYPE = INVESTIGATION ONLY** until an exact source- and graph-backed mechanism has been proposed and approved. Work from current published `protos@e2200088`, its original PLAT066-C R1/R2/R3 hypotheses and matched source/toolchain provenance; do not assume that any earlier 707/125 or 645/55 counts are graph measurements of the published HEAD.

1. Recompute a matched after-Truffle phase/node attribution for `InvokeClosureOneTail`: graph nodes by Java source ancestry, live direct-hit control/data path versus deopt/fallback, ambiguous ancestry, and whole-root node count. Do not sum overlapping ancestry categories.
2. Examine surviving physical `FrameState`, `RawStoreNode`, `FrameWithoutBoxing`, caller/B5+ argument preparation, lexical frame publication, and entry/epilogue barriers. Quantify actual reachable costs and distinguish frame-state metadata from runtime allocation.
3. Compare exact transitive call preparation and partial evaluation against GraalPy's equivalent source/root/arity shape, not just class names or language semantics. New proposals must name *which* Java/DSL branch or guard survives, its preconditions, and its physically testable replacement. No new `@Operation`, no cost shifting to other DSL operations, and no unsupported assumption that D013 causes the residual.
4. Preserve D013, Context, ReturnHome, Error, cancellation/Task, one execution mode and existing approved PLAT062/B5+ architectural invariants. Where a change would affect language semantics, route it through a distinct, explicitly approved D decision; the D207 experiment is not an approval.
5. Require a comparable Tier-1 BGV plus focused semantic regressions and integrated validation before claiming a new I-family implementation solved physical convergence.

**Coordination note:** PLAT066-C/#930 currently has a declared parent PLAT066/#924 but its native GitHub parent relationship remains unverified/missing (GitHub GET parent reports 404). This is a GITHUB006/GITHUB015 hierarchy-reconciliation gap, not a physical-solution gate; do not claim a native parent exists without verification.

`PROTOS_REVISION=e2200088d42d0d8be90626a79b373ddad64978f5`; `I115_CD=HUMAN_REPORTED_MAKE_TEST_PASS_AND_PUBLISHED`; `D013=UNCHANGED`; `PLAT066_C=PHYSICAL_RESEARCH_PENDING`; `PROJECT_RECORD_REVISION=THIS_COMMIT`.
