# PLAT061 — Composable proven compiler facts and bounded emission planning

**Status:** RATIFIED — revised Candidate B, 2026-10-16.
**Authority:** [PLAT061/#903](https://github.com/guillermomolina/protos/issues/903).
**Exact owner approval:** “apruebo B”, following explicit corrections that compiler components can both consume and produce facts, that the lowerer is just one consumer, and that physical strategy can be selected before lowering. Approval is for this exact bounded B, not a generic optimizer.
**Product source checkpoint:** [guillermomolina/protos@42c602fc629873436bbc52748acc77391bdb2fc3](https://github.com/guillermomolina/protos/commit/42c602fc629873436bbc52748acc77391bdb2fc3), 0.3.368-SNAPSHOT.
**Research:** [PLAT061 GITHUB010/GITHUB021 evidence](../../evidence/PLAT061/PLAT061_B_COMPOSABLE_FACTS_RESEARCH_RATIFICATION_2026_10_16.md).

## Approved architecture

Different compiler components may **produce demonstrated facts**, **consume existing facts**, and **derive new facts**; a class is not confined to the role of producer or consumer. Knowledge is organized by validity and consumer need, not by a mandated rigid or eager whole-program pipeline.

There are three **scopes** of information, not necessarily three passes:

1. **Canonical-definition facts:** lexical binding owner, proven binding identity/presence-candidate, parameters, captured authorities, ReturnHome and other facts of the original canonical tree. Their validity is tied to that canonical source/analysis.
2. **Profile- and site-scoped compilation plans:** consume proven facts and immutable target Context/profile conditions to decide capture/own-frame strategy, parameter entry, layout requirements, candidate inlining, physical operation families and guarded/general fallbacks. Candidate target is not an actual runtime-selected target.
3. **Emitted-root facts:** describe actual emitted body/root/plan, including effective non-suspension, primitive carrier body and resulting frame representation. They may be produced by the lowerer and consumed later, but cannot retrospectively authorize earlier emission decisions.

A missing fact is **UNKNOWN**, not false. Positive and negative proofs are different; conditional facts require explicit guards, immutable assumptions or appropriate fallbacks. Components can refine previously unknown information but cannot silently reverse a proof already used by an emitter. A later emission fact may legitimately be stronger than an original-body fact because it describes a different scope.

Approve a **minimal explicit proof/planning seam before Bytecode Builder emission** where it serves actual consumers, reusing existing `CanonicalBindingAnalyzer`, `CanonicalBindingAnalysis`, ReturnHome and Closure/frame/parameter/call-site analyses. Facts that are trivial AST queries need not be copied into permanent metadata. Prefer per-definition/per-site demand-driven analysis; do not require every source file to run all analyses. Retain **one** `CanonicalToBytecodeLowerer` and one approved semantic source Bytecode DSL interpreter under PLAT058. The lowerer consumes facts/plans, checks emission invariants, allocates Builder-local artifacts and publishes emitted facts.

Neither a universal plugin/fact manager nor a transformed optimized IR is authorized. Choice of a different custom Bytecode DSL interpreter is **not** approved by this decision: shared nested-root groups, materialized local access, Source/RootTag and tooling remain binding. A future alternative requires independent justification and explicit approval.

## Exact invariant review (GITHUB021)

| Authority | Required preservation |
|---|---|
| PLAT036 and D179 | By-reference actual lexical Context, binding identity distinct from PRESENT/ABSENT, removal/recreate and receiver fallback; no captured-by-value shortcut. |
| PLAT040, PLAT044, PLAT057 / I096 | Existing guarded ordinary `Object.call` selection, in-root closure expansion and exact fallback remain the dispatch authority; no second inlining engine. |
| PLAT058-D | One lowerer, immutable NORMAL/DEBUG Context-owned profiles; complete handwritten Builder oracle (instruction + constant operands + root group), deterministic reparse. |
| PLAT059 | The approved 2026-10-14 **no general canonical value/effect optimizer** decision remains binding. PLAT061 is the explicitly deferred, separately approval-gated **bounded proof-sharing/refactoring** case, now explicitly approved on 2026-10-16; no global optimizer, transformed IR or revocation of PLAT059's 2026-10-15 canonical frozen-Core clarification. |
| PLAT060-B5 | One universal five-header-slot ordinary Closure-call ABI, regardless of NORMAL/DEBUG or call origin. |
| D202/D203 / I104/I105 | Reuse existing signature, positional/default/rest and entry proof work; no redefinition of argument semantics. |
| PERF042/#871 | Independent graph stability, stable final-tier peer measurement and causal performance attribution; approval does not imply improvement. |

Do not conflate (a) enclosing-scope dependence, (b) persistence of the callee's own activation frame, (c) guest Closure escape/identity, (d) expected definition at a site, and (e) guaranteed call selection. No explicit outer lookup is **not** proof of absence of `this`, `context`, methodHome, prelude, ReturnHome or dynamic lookup authority. Preserve all applicable `CALLABLES.md`, `EXECUTION_AND_CONTROL.md`, `OBJECT_MODEL.md`, control/error/suspension/Task/host invariants, fresh semantic Closure identity and evaluation order.

**Reparse/Context:** Compiler facts remain bound to their canonical-analysis identity; `BytecodeLocal` and accessor instances remain Builder-parse-local. A retained Bytecode parser replays exactly the same semantic emission, root-group topology/order and assumption identities. A Closure projected into another Context gets emission decisions from the **destination Context** and its immutable profile; no imported source-Context physical state or global mutable facts.

**Pay as you grow:** Do not add per-call checks, compulsory guest activation/context/arguments Array, RootTask/RootActor/RootProcess, persistent frame, per-Closure descriptor/allocation, global mutable caches, or mandatory analyses for unused features.

## First implementation boundary

[I106/#904](https://github.com/guillermomolina/protos/issues/904) should establish a small reusable pre-Builder proof/planning integration for **existing** Closure-related facts, including multiple real consumers, while preserving all existing behavior and avoiding duplicate computation. Reuse rather than replace binding, signature, ReturnHome, frame and candidate-inlining analyses. An unused `CompilationFacts` wrapper does not count. A compiler refactor that changes emitted Bytecode must pass independent handwritten Builder-oracle and complete group/reparse checks. No new physical Closure representation, ABI or benchmark-specific shortcut is part of the first approved implementation.

**Agent edits Java/tests only; human executor runs builds, tests, validation, Git commit/push in `guillermomolina/protos`.** Product `pom.xml`/`CHANGELOG.md` only after owner-reported green tests and immediately before human commit/push; no subsequent tests after those metadata edits. User reports baseline `git diff --check` clean and all local tests PASS, but no exact independent run identity; no PLAT061 product changes/tests/benchmark results are implied.

## Deferrals, migration and strongest objection

Defer general escape analysis, autonomous capture-free physical closures, universal deoptimization/rematerialization, global fact dependency/invalidation managers, SSA/optimized canonical IR, alternate Bytecode DSL interpreter families, generalized interprocedural propagation and backends. Add only upon a proven consumer/benefit and separate decision gate if substantial. B's facts may later feed such machinery without changing semantic/ABI contracts.

Strongest objection: a simpler local lowerer refactor might remove sufficient duplication without a new conceptual seam. The first I slice must demonstrate at least two genuinely reused consumers and reduced proof ownership; if not, STOP rather than institutionalize an empty abstraction. This approval authorizes bounded architecture, not speculative framework-building or assumed PERF042 gain.

**Decision outcome:** RATIFIED B; no product code, semantics, specs, ABI, benchmarks or physical performance acceptance changed by publication.
