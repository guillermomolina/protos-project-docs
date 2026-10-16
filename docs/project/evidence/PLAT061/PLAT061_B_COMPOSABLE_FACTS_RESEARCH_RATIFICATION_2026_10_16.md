# PLAT061 — Candidate B research, explicit approval and falsification (2026-10-16)

**Type:** GITHUB010 comparative platform investigation + GITHUB021 invariant/delta review; ratification evidence, not implementation evidence.
**Owner approval:** 2026-10-16, exact reply **“apruebo B”** following the *revised* design packet. Earlier owner clarifications materially constrain interpretation: a component may consume and produce facts; the lowerer is a consumer but work may consume them earlier, including selection of different physical operations; a pre-lowering analysis stage is a candidate, not automatically a universal optimizer.
**Product HEAD inspected:** [42c602fc629873436bbc52748acc77391bdb2fc3](https://github.com/guillermomolina/protos/commit/42c602fc629873436bbc52748acc77391bdb2fc3) (0.3.368-SNAPSHOT).
**Canonical ratification:** [PLAT061 B](../../decisions/platform/PLAT061_COMPOSABLE_COMPILER_FACTS_AND_EMISSION_PLANNING.md).
**Existing research owner:** [PLAT061/#903](https://github.com/guillermomolina/protos/issues/903).
**Performance owner:** [PERF042/#871](https://github.com/guillermomolina/protos/issues/871).

## Question and refinement from original PLAT061

Original issue asked whether proven Closure capture/execution properties should persist in executable definition metadata. The owner refined the architectural problem: **how can Protos analyses, planners, emitters and later consumers exchange and derive reliable facts, each being able both to consume and to produce, without performing the same proof repeatedly or issuing general-purpose runtime work eagerly?** Closure capture and ordinary invocation remain the first concrete use case, but are not a mandate to build universal infrastructure.

The recommended B is a narrowly scoped compile-time *proof sharing and planning seam*, not a new semantic language category, optimizer pass, mutable global registry, generic dependency engine or second Truffle Bytecode DSL implementation.

## Concrete source audit: producers, consumers and boundaries

| Fact or plan | Current source | Existing consumer / issue |
|---|---|---|
| Lexical scope, binding identity, `CapturedResolved`, candidate/presence | `CanonicalBindingAnalyzer` (two passes) / `CanonicalBindingAnalysis` | Lookup/assign/frame/capture; D179 guarantees `PRESENT` differs from `ABSENT` |
| Closure ReturnHome can be observed | `CanonicalReturnHomeAnalysis.mayObserveReturnHome` | `ProtosBytecodeClosureExecutionPlan` constructor/runtime |
| Own-activation persistent frame requirement | `CanonicalToBytecodeLowerer.requiresPersistentFrameAuthority` | Root entry, unary/prefix arguments, frame-native callback storage, inlining |
| Own binding-call candidate | `bindingClosureCallCandidates` inside lowerer | I096 `inlineClosureLiteralCallCandidate` and guarded `emitInlineClosureLiteralCall` |
| Unary NORMAL argument entry | `admitUnaryNormalEntry` in lowerer | I104 `BindUnaryFrameParameter` |
| Bounded scalar constant/known local or arithmetic hint | I101 methods in lowerer | Specialized primitive emission and proofs of pure effective bodies |
| Actual effective non-suspending body | Root emission computes `effectiveBody`, `provesNonSuspendingBody` | `ProtosSemanticBytecodeRootNode.recordProvablyNonSuspendingBody` |
| Actual direct scalar carrier body | I101 emission | Root `recordDirectScalarLocalCarrierBody` |
| NORMAL/DEBUG profile | Destination `ProtosLanguageContext` | `ProtosSourceCompiler` and `ProtosBytecodeClosureExecutionPlan.rebuildForLanguage` |
| Closure capture lowering | `useCompactClosureCapture` checks creator's physical state | `MaterializeClosure` or `MaterializeCurrentClosure`; these do not themselves prove no semantic capture |

**Observed present pipeline:** `ProtosSourceCompiler.compileBytecode`: parser → `Canonicalizer` → `CanonicalToBytecodeLowerer.lowerRoot`. The lowerer calls binding analysis and performs numerous proofs within the emitter. `ProtosBytecodeClosureExecutionPlan` also performs binding/ReturnHome analyses, with reparse rematerialization for captured resolutions and destination-Context re-projection.

**Root-group constraint:** nested Closure roots share one `BytecodeRootNodes<ProtosSemanticBytecodeRootNode>` parse and materialized-local accessor authority. Some `ProtosClosureExecutionPlanCell` entries are initially pending and frozen only after Builder `create()` returns. A prior strategy must never depend on a CallTarget that does not yet exist. Reparse reuses assumptions/layouts but regenerates parse-local `BytecodeLocal` instances; this enforces deterministic, no-global-mutable-state decisions.

**Immediate documented gap:** multiple consumers call full `requiresPersistentFrameAuthority`-style logic; `admitUnaryNormalEntry` and `bindingClosureCallCandidates` own reusable proofs inside the emitter. Move proof *ownership* to a bounded compilation seam and let old consumers continue emitting equivalent Bytecode. Do not infer that the absence of a parent frame means a Closure escapes nowhere or can drop all receiver/ReturnHome/context authority.

**Source references (exact product SHA):**
- [ProtosSourceCompiler.java](https://github.com/guillermomolina/protos/blob/42c602fc629873436bbc52748acc77391bdb2fc3/src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosSourceCompiler.java)
- [CanonicalBindingAnalyzer.java](https://github.com/guillermomolina/protos/blob/42c602fc629873436bbc52748acc77391bdb2fc3/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalBindingAnalyzer.java)
- [CanonicalToBytecodeLowerer.java](https://github.com/guillermomolina/protos/blob/42c602fc629873436bbc52748acc77391bdb2fc3/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java)
- [ProtosBytecodeClosureExecutionPlan.java](https://github.com/guillermomolina/protos/blob/42c602fc629873436bbc52748acc77391bdb2fc3/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeClosureExecutionPlan.java)
- [ProtosSemanticBytecodeRootNode.java](https://github.com/guillermomolina/protos/blob/42c602fc629873436bbc52748acc77391bdb2fc3/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosSemanticBytecodeRootNode.java)
- [ProtosPerf042IdentityBuilderOracleTest.java](https://github.com/guillermomolina/protos/blob/42c602fc629873436bbc52748acc77391bdb2fc3/src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosPerf042IdentityBuilderOracleTest.java)

## Comparative implementation survey (exact external revisions)

1. **GraalPy** [BytecodeDSLCompiler](https://github.com/oracle/graalpython/blob/1e25568a4a7ae51954a9b55c7e886847364d910c/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/BytecodeDSLCompiler.java) runs `AstOptimizer.optimize`, then two-pass [ScopeEnvironment.analyze](https://github.com/oracle/graalpython/blob/1e25568a4a7ae51954a9b55c7e886847364d910c/graalpython/com.oracle.graal.python.pegparser/src/com/oracle/graal/python/pegparser/scope/ScopeEnvironment.java), then [RootNodeCompiler](https://github.com/oracle/graalpython/blob/1e25568a4a7ae51954a9b55c7e886847364d910c/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java). Emission skips cell/freevar initialization when sets are empty, and `emitMakeFunction` avoids a freevar array when none exist. **Portable lesson:** pre-emission semantic analysis, emission based on proof, and optional physical work. **Non-transferable:** Python cell and code-unit semantics, AST rewriting.
2. **GraalJS** [GraalJSTranslator](https://github.com/oracle/graaljs/blob/4327aa1f2914bac7f810406423d70ae7f8a1888b/graal-js/src/com.oracle.truffle.js.parser/src/com/oracle/truffle/js/parser/GraalJSTranslator.java) invokes `functionNeedsParentFramePass`, `usesAncestorScope`, propagating definition capability into [JSFunctionData.needsParentFrame](https://github.com/oracle/graaljs/blob/4327aa1f2914bac7f810406423d70ae7f8a1888b/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/runtime/builtins/JSFunctionData.java); [JSFunctionExpressionNode](https://github.com/oracle/graaljs/blob/4327aa1f2914bac7f810406423d70ae7f8a1888b/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/function/JSFunctionExpressionNode.java) chooses autonomous vs capturing physical node. [JSFunctionCallNode](https://github.com/oracle/graaljs/blob/4327aa1f2914bac7f810406423d70ae7f8a1888b/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/function/JSFunctionCallNode.java) uses `DirectCallNode` and cache strategies; not every user Closure is syntactically expanded in its translator. **Lesson:** different consumers use durable scope facts, but JavaScript lexical semantics are not Protos `context`/D179 semantics.
3. **TruffleRuby** [YARPTranslator](https://github.com/truffleruby/truffleruby/blob/b60b0d04dd11454012d0e8a6588514410ffacdcb/src/main/java/org/truffleruby/parser/YARPTranslator.java) has lexical translator environment and method/block metadata; [BlockDefinitionNode](https://github.com/truffleruby/truffleruby/blob/b60b0d04dd11454012d0e8a6588514410ffacdcb/src/main/java/org/truffleruby/language/methods/BlockDefinitionNode.java) materializes frame in ordinary block creation. **Lesson:** source metadata and environment composition are useful, but autonomous capture is not universal.
4. **Apple Pkl** [AstBuilder](https://github.com/apple/pkl/blob/fe96e0b1b040e66f30ed0959be24b9ded2f41b7f/pkl-core/src/main/java/org/pkl/core/ast/builder/AstBuilder.java) and [FunctionLiteralNode](https://github.com/apple/pkl/blob/fe96e0b1b040e66f30ed0959be24b9ded2f41b7f/pkl-core/src/main/java/org/pkl/core/ast/expression/literal/FunctionLiteralNode.java) perform staged AST node construction but function creation retains `frame.materialize()`. **Lesson:** a shared phase boundary does not demand capture-free function physical variants.
5. **SimpleLanguage** [SLBytecodeRootNode](https://github.com/oracle/graal/blob/1f84566ef2a851f7510e0c0b13459b0170dd34db/truffle/src/com.oracle.truffle.sl/src/com/oracle/truffle/sl/bytecode/SLBytecodeRootNode.java) has a single `@GenerateBytecode` interpreter with specialized operations; named `SLFunctionLiteralNode` is registry lookup, not a lexical-capturing analog. **Lesson:** maintain one generator and select operations via bounded emission.
6. **Espresso** [Method.java](https://github.com/oracle/graal/blob/1f84566ef2a851f7510e0c0b13459b0170dd34db/espresso/src/com.oracle.truffle.espresso/src/com/oracle/truffle/espresso/impl/Method.java) retains descriptor/signature and resolves CallTargets. **Lesson:** method metadata and executable projection can be distinct; Java method semantics do not establish Protos capture proofs.
7. **Sulong/LLVM** [LLVMFunctionDescriptor](https://github.com/oracle/graal/blob/1f84566ef2a851f7510e0c0b13459b0170dd34db/sulong/projects/com.oracle.truffle.llvm.runtime/src/com/oracle/truffle/llvm/runtime/LLVMFunctionDescriptor.java) keeps function descriptors and distinct call paths. **Lesson:** metadata reuse across call consumers, not a direct closure model.
8. **Outside Truffle:** V8 lexical scope/context analysis and LLVM/MLIR pass-analysis managers illustrate two contrasting scales: semantic proof before physical context layout; versus general invalidating dependency DAGs needed for broad IR rewrites. The latter is specifically **not** required by Protos's current immutable canonical analysis and single emitter. HotSpot's escape/scalar replacement operates after code exists; it cannot substitute for proof-driven *pre-Graal* omission of emitted instructions. Self/Smalltalk-style open coding is already an approved influence under PLAT044/057; no fresh callable category follows.

## GITHUB010 alternatives

- **A Local refactor:** extract proof helpers and pass results manually without a shared contract. Most conservative, but little architectural reuse, continued coupling.
- **B Bounded composable facts + pre-Builder planning (SELECTED):** shared per-compilation/definition/site proofs, immutable scoped results, current lowerer as both consumer and emitted-fact producer. Minimum useful generality.
- **C Universal analysis manager:** lazy dependency graph/invalidation/plugin registration for all analyses. Attractive at LLVM scale, unjustified now.
- **D New optimized canonical IR:** separate global value/effect analysis, rewritten AST/IR and transform passes. Conflicts with PLAT059 and D179/reparse risk.
- **E Multiple generated Bytecode DSL interpreters:** different interpreter types for call origins/captures. Current shared root group and materialized local access make this an unproven complexity multiplier.
- **F Defer/no action:** zero present implementation cost, indefinitely duplicating proof and lowering responsibilities.

### Twelve-axis ratings, 1–5 (higher is better); confidence H/M/L

| GITHUB010 dimension | A | B | C | D | E | F |
|---|---|---|---|---|---|---|
| 1 Correctness | 5H | 5M | 4M | 3L | 3L | 5H |
| 2 Protos fit | 4H | 5M | 2M | 2M | 2M | 3H |
| 3 Pay for what you need | 5H | 5M | 2M | 1M | 1M | 5H |
| 4 Grow as you need | 2M | 5M | 4M | 3M | 3M | 2M |
| 5 Future options | 3M | 5M | 4M | 4M | 2M | 3M |
| 6 Scalability | 3M | 5M | 4M | 3L | 2L | 3M |
| 7 Simplicity | 4H | 4M | 2M | 1M | 1M | 5H |
| 8 Portability | 4M | 5M | 4M | 3M | 2M | 4M |
| 9 Runtime resources | 5M | 4M | 3M | 2L | 2L | 5H |
| 10 Failure/operability | 4M | 5M | 3M | 2L | 2L | 3M |
| 11 Reversibility | 3M | 5M | 3M | 2M | 2M | 2M |
| 12 Evidence maturity | 5H | 4M | 4M | 3M | 2M | 5H |

**Per-criterion justifications and confidence:**

1. **Correctness:** A/F preserve actual behavior with no new interface (H). B's unknown/conditional discipline must be tested (M). C invalidation complexity, D changed canonical identities and E new root-group semantics reduce confidence (M/L).
2. **Protos fit:** B reuses semantic proofs and supports composable independent consumers (M). A is localized (H); F tolerates duplicated rules (H). C/D/E introduce speculative infrastructure (M).
3. **Present need:** A/F have minimal immediate cost (H). B only computes consumed facts and adds no runtime guest representation (M, implementation-dependent). C/D/E impose extra maintenance and compiler infrastructure without demand (M).
4. **Incremental growth:** B enables extra fact producers/consumers without central refactors (M); C also extends but with up-front architecture (M). A/F may require future extraction of fused emitter logic (M). D/E grow along prematurely fixed infrastructure (M).
5. **Future options:** B separates semantic proofs from Truffle physical choices (M). C/D could support advanced transforms but risk lock-in to dependency/IR models (M). E binds to multiple DSL root implementations (M). A/F leave migration work (M).
6. **Scalability:** B immutable per-unit facts and Context-owned projections avoid locks/global caches (M). A/F may recompute tests and couple site decisions (M). C has dependency management overhead (M), D/E unmeasured code memory/root variants (L).
7. **Simplicity:** F adds nothing (H); A is local (H). B adds one seam but removes redundant analysis ownership (M). C/D/E multiply rules and physical categories (M).
8. **Portability:** B canonical facts do not depend on Bytecode locals and isolate emission projections (M); A/F tie proof to existing emitter (M). C portable in theory but over-general (M); D/E couple to a new IR/interpreter scheme (M).
9. **Resources:** F/A no extra per-run allocations (H/M). B adds compile-time metadata only if the implementation is truly bounded (M); C/D/E can allocate more compile-time state, caches, versions, root shapes (M/L). **No measured runtime gain claimed**.
10. **Operability:** B gives attributable proof provenance/scopes without per-call probes (M); A has less infrastructure (M). C invalidation and D/E topology changes add failure modes (M/L); F leaves diagnostics scattered (M).
11. **Reversibility/deferral:** B can replace each consumer without changing semantics/ABI (M). A/F delay an increasingly cross-cutting extraction (M). C/D/E are expensive choices to undo (M).
12. **Evidence:** A/F already deployed (H). B relies on inspected real Protos analyses plus GraalPy/JS precedent (M). C/LLVM has strong outside precedent but no Protos-specific need (M). D/E have unproven local integration (M).

Scores are comparative guidance only. **Qualitative red flags:** C/D/E fail present proportionality; F/A risk underengineering if repeated cross-consumer proofs keep growing. B survives provided the initial implementation proves actual reuse instead of adding an unused wrapper.

## Adversarial semantic/fact examples

1. `() => { () => { 42 } }`: no explicit outer lookup does **not** prove no `this`, return-home or context authority, nor absence of observable fresh Closure identity.
2. `() => { x: 10; () => { x } }`: by-reference capture; later remove/recreate/shadowing and fallback may change lookup; no copy of `10` into a fake lexical cell.
3. `() => { identity: (value) => { value }; identity(1) }`: static body candidate ≠ current target/selected `Object.call`; maintain ordinary dynamic guard/fallback unless a separate proof proves selection.
4. Source body containing a potentially suspending send that is soundly eliminated at emission: the **effective emitted root** can be proved non-suspending without contradicting conservative canonical facts. Do not feed post-emission facts back into already emitted code.
5. NORMAL vs DEBUG and cross-Context `rebuildForLanguage`: facts valid for one profile/root projection must not migrate as source facts. Bytecode DSL reparse must repeat exact chosen emission decisions while recreating parse-local locals.
6. Host invocation and call-site provenance: syntactic local/external is not a sufficient semantic discriminator for a different ABI/DSL root; B5 and physical caller/ReturnHome authority remain exact.
7. Object construction and `context` intrinsic: receiver/object-construction environment is not an ordinary lexical Closure capture scope, but current context can be observed; independent proof required.
8. Recursion, nested escaped Closure, Task/Actor/Process/continuation/ensure/error pathways: information must never trigger eager optional infrastructure on simple paths or remove required observable authority.

## Incremental and future stress tests

**Minimum useful today:** explicitly share current binding, persistent-frame, signature/ReturnHome and callsite facts across two real emission decisions, moving their *proof* away from Builder instruction emission while keeping physical semantics unchanged. Compute expensive facts only if a consumer exists; preserve cheap AST questions as direct queries. No wrapper without reuse.

**If deferred:** `CanonicalToBytecodeLowerer` (~10k lines) continues to accumulate proof-local heuristics and repeated binding/frame/Call/ReturnHome checks. Extracting later touches entry, capture, inline and root fact emission simultaneously. This is maintenance/coordination cost, not an alleged ABI or guest compatibility break.

**If prematurely generalized:** global dependency managers, extended lattices, mutable property registries, extra IR types and a second generated interpreter would charge parse-time memory/compiler maintenance to simple scripts and complicate Native Image, source scopes and deterministic reparse. Explicitly deferred.

**Stress cases:** many Closure definitions → avoid unconditional Cartesian combinations of analysis and site representations; many Contexts → immutable canonical facts versus per-destination physical projections; concurrency → no shared mutable global fact registry; Native Image/AOT → do not assume JIT or reflection for decisions; DEBUG/source metadata → keep generated root identity/tag sections; future non-Truffle backend → reuse semantic facts but select different emitter-specific facts; cancellation/continuation and Task/Actor/Process → no hidden extra per-call state. If future whole-program/interprocedural transformations require an invalidating dependency graph, add one later behind separate review, not now.

**Strongest objection:** A local refactor may suffice and B could become a do-nothing `CompilationFacts` wrapper or pseudo-IR. Falsification gate: at least two independent actual consumers must reuse a produced fact; all existing semantics/bytecode identities must remain correct, and absent facts must fail closed. If not demonstrable, stop rather than entrench B prematurely.

## Structural and execution validation gates

- Product agent edits source and tests, **human executor** runs all compilation/tests/validation and product Git publication. User reports existing checkout `git diff --check` clean and local tests PASS; exact test command/commit identity is not independently established and PLAT061 made no product changes.
- Maintain PLAT058 independent handwritten Bytecode DSL Builder oracles, **complete** instruction names and constant operand signatures, whole root group/order/arity-rejection topology and deterministic reparse/ensureComplete; preserve NORMAL/DEBUG negative controls.
- Check nested escaping capture, D179 membership and provenance, identity/ReturnHome, default/rest, dynamic `Object.call` shadowing, error precedence and fallback, source/debug, Context re-projection and optional Task.
- PERF042 retains original `primitive-closure-call` benchmark and stable admitted Tier-2 source-to-BGV/timing acceptance. Historical JS 13, Py 50 and Protos 4,533 graphs belong to pinned previous revisions; 188 Tier-1-oracle work and I104 five-instruction identity root are **different measurement classes**, no current-head graph parity assumed.
- Neither this research nor the owner approval executes builds, benchmarks or alters product files; no implementation result claimed.

## Explicit GITHUB021 delta vs PLAT059

The owner-approved PLAT059 disposition was **no new general canonical optimizer/IR phase** with a *future, separately gated* bounded fact-analysis possibility if cross-consumer duplicated proofs justify one. Source inventory now establishes multiple existing consumers, and the owner **explicitly approves the narrow composable B**. This is a **bounded follow-on** to that deferral, *not* a silent broad reopening of PLAT059. Should implementation try to introduce global value/effect optimization, rewritten canonical IR, automatic invalidation or multiple interpreters, it exceeds approval; STOP for a new exact decision. PLAT058 single lowerer, PLAT059 frozen-Core canonical receiver clarification, PLAT060 universal B5, D179/Closure semantics and all ratified owner invariants remain preserved.

**Result:** Owner-ratified B, implementation independent and pending human execution; no product code/test/benchmark effects or performance gains.
