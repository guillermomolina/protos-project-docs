# PLAT059 — Final canonical optimization research, falsification and owner approval

**Date:** 2026-10-14. **VERDICT:** `NO_NEW_ARCHITECTURE_REQUIRED`. **Scope:** an approved resolution of a PLAT design question, not new architecture or source implementation.
**Issue:** [PLAT059/#894](https://github.com/guillermomolina/protos/issues/894). **Final decision:** [PLAT059 record](../../decisions/platform/PLAT059_NO_NEW_CANONICAL_OPTIMIZER_ARCHITECTURE.md).
**Owner provenance:** The owner explicitly replied **"ok apruebo"** after the PLAT059 packet that selected (A) extension of PLAT058's existing proof/emission machinery plus (E) deferral of additional optimizer architecture, with (D) a *future* contingent hybrid. This is approval of that exact negative-architecture result; no new optimizer or semantically observable change has been approved.
**Source checkpoint:** [`guillermomolina/protos@8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff`](https://github.com/guillermomolina/protos/commit/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff); [PLAT058-D ratification](../../decisions/platform/PLAT058_PROFILE_SPECIALIZED_MINIMAL_LOWERING_AND_BUILDER_ORACLES.md); [I101/#892](https://github.com/guillermomolina/protos/issues/892), slice A published, B/C pending.
**Primary upstream checkpoints:** [GraalJS `3df14182071768bd4162734ca69e8b2a81eed5d4`](https://github.com/oracle/graaljs/commit/3df14182071768bd4162734ca69e8b2a81eed5d4); [GraalPy `a099d006438942d54c1856c2cce7b6c2839b0c85`](https://github.com/oracle/graalpython/commit/a099d006438942d54c1856c2cce7b6c2839b0c85). Existing full [GraalJS 65-mechanism inventory](PLAT059_GRAALJS_DETAILED_OPTIMIZATION_INVENTORY_2026_10_14.md) and [GraalPy 57-mechanism inventory](PLAT059_GRAALPY_DETAILED_OPTIMIZATION_INVENTORY_2026_10_14.md) remain the audit authority; these numbers are mechanisms/variants, **not independent optimizer passes**.
**Evidence limits:** Read-only source and issue/document inspection, no execution; later survey outside the two upstream inventories is architectural and narrower than an exhaustive line-by-line compiler audit. No measured speedup, independently run tests or semantics-changing patch attributed to PLAT059.

## 1. Actual existing source: do not invent missing optimizations

| Mechanism | Classification | Source/proof/fallback or limitation |
| --- | --- | --- |
| Literal Boolean callback result physically removed in eligible NORMAL terminal | `PROVEN_ELIMINATION` | `CanonicalToBytecodeLowerer.foldedConstantControlResult`, `normalBooleanControlSite`; requires published standard Object, literal canonical Boolean, selected inline zero-arg Closure, pure exact literal body, NORMAL profile; otherwise ordinary send path |
| Literal Integer `+` represented as a static result in composed send | `STATIC_FOLD` **under correctness audit** | `emitComposedSendUnfolded` literal branch emits result; its D013 selection proof is not manifest at the immediate branch; suspected discrepancy below, *not proven correct or incorrect* |
| Integer values propagated through eligible scalar binding creation/assignment up to terminal addition | `GUARDED_SPECIALIZATION` | `terminalKnownIntegerAddition`; `currentRootScalarLocals` restricts to proven effect-free straight line; `IsCompactLocalFrame` controls local fold; generic send remains on miss |
| Primitive numeric selection and numeric carriers | `GUARDED_SPECIALIZATION`/`DYNAMIC_SPECIALIZATION` | `TryPrimitiveNumericSendOne` has canonical selector, Context, Prelude, assumption, operand checks and MISS fallback; more number/comparison work owned by PERF040/PERF046 |
| Frame-native/physical scalar local reads and writes | `PROVEN_ELIMINATION` of unnecessary generic lookup, not semantic slot erasure | `CanonicalBindingAnalyzer`/`CanonicalBindingAnalysis`/`CanonicalBindingResolution`, `scalarLocalNamesForRoot`, `requiresPersistentFrameAuthority`; `Resolved` does not guarantee permanent PRESENT |
| Boolean-control fast-path and selected callback inline | `GUARDED_SPECIALIZATION` or `PROVEN_ELIMINATION` per admitted site | I095/I101 Boolean selector table and `admitsNormalInlineControlBody`; negative arity, selected callback, effects, debug require ordinary path |
| Ordinary Closure call and indexed each inline candidates | `GUARDED_SPECIALIZATION` | I096/PLAT057 and existing `inlineLiteralEach`/call candidates retain actual selection and closure semantics |
| Primitive argument/result carriers and compact terminal return | `DYNAMIC_SPECIALIZATION` / `PROVEN_ELIMINATION` per site | I091/F2 and PERF037; do not equate carrier availability with guaranteed result |
| Context-owned NORMAL/DEBUG physical profiles and reparse-stable group | `PROVEN_ELIMINATION` for eligible profile-specific instructions | I101-A `ProtosSourceCompiler`, `ProtosExecutionMode`, lowerer profile and root emission; DEBUG keeps ordinary semantic/tooling paths |
| General multi-statement value/effect data-flow analysis | `NOT_ESTABLISHED` | There is no demonstrated general inter-statement constant/value/effect optimizer; **some bounded terminal Integer propagation already exists** |
| JIT cleanup of residual guards/scaffolding | `GRAAL_ONLY` when only IR shows elimination | Graph shape or timing never proves smallest pre-Graal Bytecode DSL emission |

Source: [CanonicalToBytecodeLowerer.java](https://github.com/guillermomolina/protos/blob/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff/src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java), [CanonicalBindingAnalyzer.java](https://github.com/guillermomolina/protos/blob/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff/src/main/java/com/guillermomolina/protos/execution/CanonicalBindingAnalyzer.java), [CanonicalBindingResolution.java](https://github.com/guillermomolina/protos/blob/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff/src/main/java/com/guillermomolina/protos/execution/CanonicalBindingResolution.java), [ProtosSemanticBytecodeRootNode.java](https://github.com/guillermomolina/protos/blob/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff/src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java), [ProtosI101ProfileSpecializedLoweringTest.java](https://github.com/guillermomolina/protos/blob/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff/src/test/java/com/guillermomolina/protos/execution/ProtosI101ProfileSpecializedLoweringTest.java). The committed I101-A test includes handwritten Builder-oracle expectations, mode, root/topology and reparse controls; new B/C cases still need their own proof.

### Specific static correctness risk requiring independent audit

In the pinned lowerer, `emitComposedSendUnfolded` around **8418–8429** checks `foldedIntegerAddition(send, Map.of())` and emits the computed constant, apparently *without* an explicit runtime D013 selected-method guard. Contrast `ProtosSemanticBytecodeRootNode.TryPrimitiveNumericSendOne.canonical`, approximately **6775–6840**, which requires stable canonical selection, exact Context/Prelude, and matching live operands before using `FoldedIntegerAddition`. This is a **possible bypass**, not a confirmed runtime fault: an upstream or contextual invariant may still authorize the special case. Require a valid actual guest override/Context regression and trace before filing a definitive BUG, reopening already-closed PERF043 or attributing an observed failure. An existing green suite without that adversarial witness does not settle this question.

## 2. Comparative mechanism inventory (transferability)

| System | Relevant actual mechanism | Transferability / exclusion |
| --- | --- | --- |
| GraalJS | Local node factory constant folds; constant-one-side operator nodes; conversion avoidance; block flattening and virtual lexical scope | Supports local construction-time physical specialization, *not* an automatic Protos message-fold rule; no confirmed global inter-statement constant pass in the reviewed source |
| GraalPy | `BytecodeDSLCompiler` calls explicit `AstOptimizer` before `RootNodeCompiler`; constant arithmetic/unary/collection/iterable transformations; resource caps; fixed-arity and scope-aware Bytecode DSL emission | Demonstrates a possible separate phase, but its optimizer does not establish general local value propagation; Python operations do not share D013 semantics |
| TruffleRuby | Prism/Truffle translation, lazy/differentiated AST construction and language-aware specializations | Emphasizes paying compilation costs only when needed; not evidence a rewritten Protos Canonical AST is necessary |
| SimpleLanguage | `SLAddNode` uses generated Truffle DSL specializations and overflow/big-number handling | Runtime specialization is distinct from source folding |
| Espresso | Guest Java bytecode loop, quickening to specialized operations/Truffle nodes | Demonstrates bytecode quickening, not arbitrary elimination of a Protos dynamic dispatch |
| Sulong/LLVM | Consumes LLVM bitcode whose prior compiler passes may already simplify operations | Different input representation/semantics and optimization boundary |
| Apple Pkl | `AstBuilder` maps parser forms to concrete Truffle nodes including binary arithmetic and `if` | Relevant AST-construction precedent; no verified matching pre-Bytecode whole-program fact pass |
| CPython | Selective AST optimizer with complexity/resource limits | Evaluating constants must be bounded by compilation work and size |
| V8 | Ignition bytecode generation, runtime specialization and later optimizing compiler stages | Multiple compiler stages are separate mechanisms; later JIT simplification does not prove initial bytecode minimality |
| LLVM | Sparse conditional constant propagation (SCCP) over explicit IR/CFG | Credible future data-flow technique only after Protos effects, aliasing and nonlocal control are modeled |

Anchors: [GraalPy AstOptimizer](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java), [BytecodeDSLCompiler](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/BytecodeDSLCompiler.java), [GraalJS JSAddNode](https://github.com/oracle/graaljs/blob/3df14182071768bd4162734ca69e8b2a81eed5d4/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/binary/JSAddNode.java), [SimpleLanguage SLAddNode](https://github.com/graalvm/simplelanguage/blob/5a2b35790539ab1fe5c61f02b71e3485fd0e0fb1/language/src/main/java/com/oracle/truffle/sl/nodes/expression/SLAddNode.java), [Espresso architecture](https://github.com/oracle/graal/blob/e71338c70bd9a724b5edc25accf5dffcaaaf8305/espresso/docs/how-espresso-works.md), [Pkl AstBuilder](https://github.com/apple/pkl/blob/f4ad0da658449033830f5a23906dd0a38942b73b/pkl-core/src/main/java/org/pkl/core/ast/builder/AstBuilder.java), [CPython AST optimizer](https://github.com/python/cpython/blob/25883627946b7cbcf7286acfe2a2019382375325/Python/ast_opt.c), [LLVM SCCP](https://llvm.org/docs/doxygen/Scalar_2SCCP_8cpp_source.html). Peer source SHAs for this secondary survey are read-only references, not comparative runtime measurement identities.

## 3. Required concrete abstract-value/effect falsification

Each row assumes valid Protos source; sample snippets are representative, not execution evidence.

| Case | Static knowledge and proof obligations | Physical removal vs fallback; NORMAL/DEBUG |
| --- | --- | --- |
| A: `1 + 2` | Operand constants; must still establish D013 selected method, exact arithmetic/overflow and no observable send effects | Only an independently proven operation may become a constant; otherwise canonical guard or ordinary send; DEBUG retains required source/call observability |
| B: `x: 1; x + 2` | Proven binding identity **plus** current presence/value stability and D013 dispatch; actual scalar terminal Integer case already tracked in PERF043 | Bounded guarded fold exists; cannot declare universally static just because `x` was initialized once |
| C: `x: true; x.ifTrue() { 1 }` | Presence/value of `x`; selected standard Boolean behavior and callback activation/identity/effects proof | Candidate for I101-B minimal NORMAL emission; DEBUG retains its ordinary observable control and source |
| D: `x: 1; x = 2; x + 2` | Sequential assignment updates fact **after** RHS evaluation and pre-RHS target selection | Kill/replace old fact; preserve error/side-effect ordering |
| E: `context.removeSlot("x")` after creation | PRESENT can become ABSENT; recreate or nearer late shadowing can change lookup target | Do not substitute initial value across removal; generic presence resolution/fallback |
| F: `x: 1; f: () => { x }; x = 2; f()` | Genuine Closure captures Context by reference; value can change before invocation | No by-value constant capture; preserve identity/activation/ReturnHome as needed |
| G: user override of `+`/`ifTrue` | Same literals can select different ordinary method; exact authority and invalidation required | Guard/Assumption plus MISS; no unconditional static fold without stronger proof |
| H: `(y) => { 1 + y }` | Only left operand constant; right is unknown | Specialize shape/carriers without pretending result constant; retain dynamic selection |
| I: selected callback with Context, exception, ReturnHome or suspension | Evaluation order, selected/unselected callback, Context mutation and unwind may be observable | Keep necessary physical activation, effects, suspension and cancellation; NORMAL compactness only with complete proof |

**Proposed abstract model, contingent and NOT approved as an independent phase:** `ValueFact = UNKNOWN | CONSTANT(value)`; `BindingFact = UNKNOWN | PRESENT(identity) | MAY_BE_ABSENT(identity) | DYNAMIC`; `OperationProof = UNKNOWN | GUARDED(conditions,fallback) | PROVEN(result,obligations)`. Track effects for Context reads/writes/escape, ordinary calls, exceptions, closure capture, ReturnHome, suspension/cancellation and source/debug. Facts are *not* executable authority by themselves. Merge facts only if all feasible paths agree and their obligations remain valid. Invalidations require per-root, per-Context and reparse-stable provenance.

## 4. Candidate construction, falsification and owner recommendation

| Candidate | Core design | Strong point | Falsification |
| --- | --- | --- | --- |
| **A** Extend existing PLAT058 | Small local proof-backed immutable emission decisions in the one lowerer, reusing binding/escape analyses | No new pipeline | Source lowerer already large; repeated cross-statement reasoning could become an implicit global pass |
| **B** Independent immutable canonical value/effect facts | Whole-root analysis returns per-site facts without Canonical AST rewrite | Share common proofs | Duplicates `CanonicalBindingAnalysis` unless meticulously composed; compile time and invalidation not justified now |
| **C** Rewritten optimized Canonical AST | Explicit second tree/IR then emission | General transform surface | Source identity, Closure roots, lexical maps, tag provenance and reparse become significantly harder |
| **D** Minimal hybrid | Only proven cross-statement flow facts extracted; physical specialization remains lowerer-owned | Future incremental escape from duplicated proofs | New abstraction may still charge complexity before demonstrated consumers |
| **E** Defer structural addition | Finish already-approved I101-B/C first | Zero additional framework today | Would be inadequate if interpreted as cancelling necessary minimal-emission work (it is **not** interpreted that way) |

### GITHUB010 twelve-axis comparative matrix

Each score is 1–5, followed by confidence `H/M/L` (high/medium/low). Reasons explicitly listed after the matrix. No additive score alone can overrule correctness or overengineering gates.

| Axis | A | B | C | D | E |
| --- | ---: | ---: | ---: | ---: | ---: |
| 1. Correctness and invariants | 4H | 4M | 3L | 4M | 5H |
| 2. Protos philosophical alignment | 5H | 3M | 1H | 4M | 5H |
| 3. Present-need proportionality | 5H | 2M | 1H | 4M | 5H |
| 4. Incremental growth | 4H | 4M | 2M | 5M | 4H |
| 5. Future-option resilience | 4M | 4M | 4M | 5M | 3M |
| 6. Scalability | 4M | 4M | 3L | 4M | 4M |
| 7. Conceptual simplicity | 4H | 2M | 1H | 3M | 5H |
| 8. Portability/implementation freedom | 4M | 4M | 2M | 4M | 5H |
| 9. Runtime and resource cost | 5M | 4L | 3L | 4M | 5H |
| 10. Failure/operability | 4M | 3M | 2L | 4M | 5H |
| 11. Reversibility/deferral migration | 4H | 3M | 1M | 4M | 4H |
| 12. Evidence maturity/implementation risk | 5H | 2M | 2M | 3M | 5H |

- **1 correctness:** A has established proof/fallback practice; B/D can be conservative but need invalidation proof; C risks identity drift; E introduces no new semantic risk.
- **2 philosophy:** A/E preserve ordinary things and pay-as-you-grow; B/D add architecture only if useful; C imposes two semantic trees.
- **3 immediate proportionality:** I101 already owns the real work; B and especially C are speculative now; D becomes proportional only upon concrete repeated cross-statement needs.
- **4 incremental growth:** A is extensible; B/D can acquire fact families; C forces broad migration; E remains reversible pending I101 findings.
- **5 options:** D offers narrow future extraction, B/ A preserve many options, C offers broader transformations but with upfront coupling; E risks proof duplication if deferred indefinitely.
- **6 scale:** A/E avoid global passes/state, B/D are per-root if bounded, C potentially duplicates AST allocations; no measured compilation scaling data for B/C/D.
- **7 simplicity:** A/E minimize layers; D creates one narrow abstraction, B a full analysis, C another tree.
- **8 portability:** All except C can leave canonical semantics backend-independent without significant new mapping; C requires backend/source remapping.
- **9 resources:** Guest runtime costs of B/C/D are unmeasured; E imposes none; A mostly existing local compile-time decisions; no universal Task/Actor/Process allowed.
- **10 failures:** New invalidation and stale-reparse risks increase from A/E to B/D to C; conservative fallback remains mandatory.
- **11 deferral:** No public format or confirmed irreversible semantic contract requires B/C now; later extraction of per-site facts is a bounded internal migration hypothesis, not a guaranteed zero-cost refactor.
- **12 maturity:** A/E rest on published PLAT058/I101; B/D require feasibility and new proofs; C lacks a demonstrated forcing case.

**Non-compensating overengineering gate:** reject C as disproportionate; B not justified by established need. **Underengineering gate:** E is eligible *only because* it retains I101-B/C obligations; 'do nothing to optimization' would fail. **Surviving present option:** A + E; D is a separately reviewable future option, not a preapproved architecture.

### Adversarial questions and strongest argument against the chosen path

- An optimizer duplicating `CanonicalBindingAnalysis` would create conflicting identities and presence assumptions: do not do that.
- Local proof helpers can hide a global optimizer if state begins to flow across arbitrary effects, branches and captures: stop and assess D at that point.
- Captured and removed Context slots, more-near shadowing, overriding standard sends, Context projection and suspension can invalidate naive facts.
- DEBUG may need physical provenance beyond NORMAL's minimum; `BytecodeConfig.update` cannot retroactively redefine a Context's immutable semantic profile.
- Each reparse must recompute identical decisions from stable canonical/profile/publication facts, not opportunistically consult mutable external authority.
- No universal frame/RootTask/RootActor/RootProcess and no global cache, especially for trivial programs.
- Truffle Bytecode DSL evolution and other runtimes argue for preserving original Canonical AST and back-end-specific physical emission.

**Strongest objection:** `CanonicalToBytecodeLowerer.java` is approximately 9,600 lines and accumulates localized proof responsibilities. A future independent fact analysis could reduce duplication and make multi-statement reasoning safer. **Answer:** line count alone does not establish unavoidable cross-family data-flow; finishing I101-B/C reveals whether such repeated requirements actually exist. Deferral may later require internal refactoring and oracle migration; nothing verified imposes a public-format or language-contract rewrite today.

## 5. Bounded implementation and acceptance handoff

**No new Ixxx or PLATxxx is allocated.** Existing [I101/#892](https://github.com/guillermomolina/protos/issues/892) owns (B) generalized accepted Boolean/Closure/simple-pure-arithmetic minimal emission and (C) whole-topology, NORMAL/DEBUG, Context, Task, capture/ReturnHome, source/debug and reparse conformance. Separate PERF work retains its own mandate. A potential literal-add D013 bypass should first receive read-only source/guest reproduction audit; do not count it as a confirmed bug, close/reopen PERF043 based on speculation or justify architecture from it.

For each *claimed* transformation, independently handwrite `ProtosSemanticBytecodeRootNodeGen.Builder` expected Bytecode DSL emission. Compare full `Instruction.getName()` and **all constant-operand types/values**, plus entire `BytecodeRootNodes` group count, order, relevant arity roots/Closure plans, identities, `ensureComplete()`, reparse and separate mode/Context expectations. The admitted NORMAL witness `() => { true.ifTrue() { 1 } }` must match `() => { 1 }` and literal-one oracle executable activation root, with required carrier and no unnecessary Boolean send, callback root or runtime DEBUG test. An optimized candidate with a necessary guard needs an *independent guarded* oracle, not an unconditional constant oracle.

Semantic tests must independently establish outcomes, exact argument order/count, override/invalidations, `PRESENT` vs `ABSENT`, capture/identity, overflow and mixed number correctness, selected callback effects, Error/nonlocal return, suspension/cancellation and DEBUG observability. A smaller Graal graph or faster benchmark does **not** satisfy this physical or semantic acceptance. Group related code/test modifications into coherent B/C slices, no micro-slices; never edit `pom.xml`/`CHANGELOG.md` until the human reports green tests immediately before product commit/push, with no tests after that edit.

## 6. GITHUB021 invariant review and final research boundary

| Approved owner invariant | Result |
| --- | --- |
| PLAT058-D single lowerer, proof-driven minimum physical Bytecode before Graal | **PASS — PRESERVED** |
| Immutable Context NORMAL/DEBUG profile, no runtime mode branch for admitted sites | **PASS — PRESERVED** |
| Independent handwritten Builder oracle, whole physical signature/groups | **PASS — PRESERVED** |
| D013 overridable messages and correct invalidation; D179 Context slots and capture | **PASS — PRESERVED** |
| Genuine Closure identity/ReturnHome/effects, Task suspension/cancellation | **PASS — PRESERVED** |
| Deterministic reparse, source provenance and multiple Contexts | **PASS — PRESERVED** |
| Pay-as-you-grow with no universal scheduling/activation tax | **PASS — PRESERVED** |

There is no conflicting earlier owner-approved invariant requiring reopening. No new material architectural consequence is approved implicitly. New optimizer analysis B/D would need a separate formal owner decision if later justified.

**Human-executor evidence statement:** At the resolution request the owner reports `git diff --check` clean and **all local tests PASS**. No exact suite totals, invocation or product publication SHA for that report were furnished; Protos product HEAD independently inspected is `8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff`. These reports do **not** certify that PLAT059 implemented anything. No builds, tests, benchmarks, guest examples or Git product operations were executed for this architectural closure.

**Selected result:** PLAT058-D remains the only authoritative architecture for this physical optimization boundary; no separate canonical optimizer stage or optimized canonical AST is approved. Actual implementation remains I101-B/C and the correctly scoped independent owners.
