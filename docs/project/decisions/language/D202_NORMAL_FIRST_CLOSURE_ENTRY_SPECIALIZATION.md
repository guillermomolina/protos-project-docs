# D202 — NORMAL-first pre-lowering Closure-entry specialization (ratified)

**Status:** **RATIFIED — exact owner approval, 2026-10-15.**  
**Live design authority:** [D202/#898](https://github.com/guillermomolina/protos/issues/898).  
**Implementation ownership:** [I104/#899](https://github.com/guillermomolina/protos/issues/899), **READY / not implemented by this publication**.  
**Research and comparative evidence:** [D202 GraalPy pre-lowering investigation](../../evidence/D202/D202_GRAALPY_PRELOWERING_RESEARCH_2026_10_15.md).  
**Exact Protos implementation revision audited:** [`5948ee7ce70d5a70636c921e31566483c4809323`](https://github.com/guillermomolina/protos/commit/5948ee7ce70d5a70636c921e31566483c4809323).  
**Authority:** this is a *non-normative architecture/decision record*, not a replacement for normative `spec/` text; product and specification files are unchanged by this decision publication.

## Exact approval provenance

Following D202's exhaustive initial D003/GraalPy study, further exploration established that:
1. `NORMAL` and `DEBUG` have **immutable Context-owned physical emission profiles** under the already-ratified PLAT058-D; `NORMAL` does not preserve complete debugger scopes for speculative future attachment, `DEBUG` does.
2. Protos can determine **static signature, frame layout and observability before the lowerer** and emit GraalPy-style direct frame-local argument handling in the Bytecode DSL lowerer, rather than implementing *runtime binding* at compile time.
3. The single universal B5 physical call ABI and guest semantics remain unchanged.
4. A narrow, independently proven single-argument case must precede broad multi-argument restructuring and is not an assumed Tier-2 gain.

The owner was presented this precise candidate in the conversation of **2026-10-15**:

> “D202 — NORMAL-first Closure-entry specialization: El compilador realizará, antes del lowering, un análisis de la firma y de la observabilidad de cada Closure. NORMAL podrá utilizar una entrada físicamente mínima al estilo GraalPy, sin pagar por DEBUG ni materializar Contexts que el programa no observa. DEBUG seguirá conservando todos sus compromisos. La ABI B5, los defaults, los errores, los handlers, los captures y el comportamiento observable de Protos permanecerán intactos. La primera implementación tendrá una puerta de aceptación estricta: comparación contra un Builder oracle independiente y evidencia válida de Tier 2. Si no hay mejora, no ampliaremos la complejidad.”

The project owner responded **“apruebo esa política”** and explicitly requested Issue/evidence publication and the next autonomous implementation slice. **Approval is of this bounded architectural policy**; it is not approval of a semantic rewrite, a Python default-binding model, a distinct call ABI, unmeasured speedup, or an unlimited new runtime infrastructure.

## Selected architectural contract

**Pre-lowerer compile-time proof:** use existing `CanonicalBindingAnalyzer`/`CanonicalBindingAnalysis` and immutable Context profile, with a small optional per-`CanonicalClosure` plan of parameter counts, ordinal mapping, local identity, required defaults/rest and observable Context/capture requirements. Bind this metadata to **the exact canonical AST/source identity**, not a process-wide singleton or guessed name. Carry/rebuild it correctly for lexical nesting, shared BytecodeRootNodes group reparse, different Context profiles and source reprojection. Prefer adding the smallest proof to the existing layer over inventing another full IR/compiler.

**NORMAL physical emission:** generate the smallest semantically sufficient path for an admitted Closure: direct frame-local argument transfer, minimal valid-arity work, conditional lazily materialized activation only if **the guest or Error/control semantics actually demand it**, and no DEBUG-only bookkeeping for a theoretical future debugger. The compiler is free to select semantically equivalent cold invalid-arity roots/branches and concise data movement **after the complete caller argument vector has been evaluated and after ordinary callable selection**. It must not replace runtime guest invocation with static evaluation. Existing zero-signature rejection-root optimization (PERF032-G6) is precedent, **not** a mandate that all signatures need another root.

**DEBUG physical emission:** retain full tooling/scope fidelity from the first invocation, including later debugger attachment in DEBUG. DEBUG-specific instructions may differ from NORMAL; neither mode changes the B5 frame argument layout or observable guest semantics. No runtime per-call mode tests as a substitute for PLAT058-D's immutable compile-time emission.

**Caller/ABI:** preserve one PLAT060-B5 five-slot header, with supplied positional values in the same slots/offsets in both modes. Honor dynamic `Object.call` lookup, canonical standard selection guards, host/rich entry, method receiver/`this`/`methodHome`, exact caller and return-home provenance, Tasks/Actors and all fallback semantics. The physical Python `PArguments` layout and Python's keyword/default rules are NOT transferable into Protos.

**Guest semantics:** D003 as amended is unchanged. `CALLABLES.md` normative sequential slot PRESENCE, default invocation evaluation in the **real current activation**, left-to-right caller argument/spread evaluation, source Error handler/trace identity, freeze/close/remove/recreate, lexical/receiver fallback, rest fresh frozen Array of only supplied suffix, `args` an ordinary identifier, captures by reference, NLR, cancellation, suspension/ensure/cleanup and contextual authority are invariants. The compiler may change **unobservable physical staging**; it cannot change guest effects or failures. No distinct parameter namespace or hoisted/definition-time defaults.

## Interpretation of D003 “not a separate arity preflight”

The statement in `CALLABLES.md` determines **guest-observable order of binding, evaluation, Error and partial Context effects**. It cannot be treated as a permanent requirement that every semantically equivalent implementation execute one particular Java operation or Bytecode DSL opcode for each parameter. The policy explicitly allows proving that an earlier physical count classification has no guest-observable difference, while retaining the exact semantic failure point/observable consequences as needed.

Under current grammar `required* defaulted* rest?`, an invalid call with too few supplied inputs reaches a missing **required** parameter before executing any optional default; an invalid excess-input call provides an input for every ordinary parameter so it does not run a default merely before the excess rejection. This supports, **but does not by itself prove**, safe early physical classification. All other effects, caller selection, host/interop, handlers and trace requirements are mandatory.

**Selected normative delta:** **NONE** to D003, `CALLABLES.md`, or PLAT060. This record interprets implementation freedom already consistent with the retained guest semantics. If a future I104 slice finds a concrete observable counterexample or an actual contradiction in normative text, **STOP and return for owner approval**; the architectural approval is not permission to erase or rewrite D003. No normative specification publication is implied or fabricated here.

## Owner-approved invariants — GITHUB021 reconciliation

| Prior authority / invariant | D202 consequence | Review |
| --- | --- | --- |
| D003 ratified parameter creation/default order | sequential **guest** presence, effects and failure remain; physically optimized NORMAL when proven | **PRESERVED** |
| D179 Context mutable slot PRESENCE/ABSENT and capture-by-reference | runtime dynamic observations always retain correct authority | **PRESERVED** |
| PLAT058-D immutable NORMAL/DEBUG profile | classify early; emit minimal NORMAL and faithful DEBUG without per-call mode choice | **PRESERVED / ACTIVELY REUSED** |
| NORMAL has no future debugger-attach guarantee | omit DEBUG-only materialization; synthetic tool inspection is not a NORMAL promise | **PRESERVED** |
| DEBUG retains complete attached-later scopes | DEBUG still emits and projects real parameter Context visibility | **PRESERVED** |
| PLAT060-B5 universal ABI and PLAT042 direct source root | same five header slots and source-identity/root conventions | **PRESERVED** |
| Ordinary polymorphic `Object.call` | no caller-side bypass before actual selected standard target | **PRESERVED** |
| Error, NLR, Task, Actor, source/diagnostic provenance | cold/error paths carry same semantic authority; performance not assumed | **PRESERVED** |
| Existing I101/PERF042 evidence and tests | keep independent Builder oracle and stable final Tier-2 gate | **PRESERVED** |
| New language surface, semantics, default timing, parameter namespace | **no new surface or semantic change** | **NOT APPROVED / NOT INTRODUCED** |

There is no reopened previously approved owner invariant requiring a replacement semantic choice. D202 expressly corrects an **incorrect implementation-review assumption** that merely *potential* future DEBUG tooling requires eager Context materialization in NORMAL; that assumption conflicts with, and was not an exception to, PLAT058-D.

## Selected implementation acceptance and staged continuation

**Implementation I104, not D202, owns all executable changes, tests and benchmark validation.** One I104 Issue owns coherent internal slices; do not manufacture an Issue per Java file:
- **A — coherent unary proof/entry slice:** reuse static analysis *before* lowerer, admit eligible `(value) => { value }` (currently excluded by `admitsNormalDirectScalarLocalBody` because it has parameters), preserve rich/host/failing arity/DEBUG; add independent handwritten Builder oracle, reparse/topology and cross-mode tests. Do not impose artificial instruction-count equality against an oracle that omits Protos semantics.
- **B — conditional multi-argument supplied-prefix transfer:** consider GraalPy `CopyArguments`-style compact `LocalRangeAccessor`/`@ExplodeLoop` only if B improves qualified code against A/actual baseline, and if per-slot contextual presence is still exact.
- **C — defaults/rest, error, capture, host, lifecycle integration:** extend only after proof, avoid replacing D003 defaults by caller-prepared Python values.
- **D — validated Tier-2/performance closure:** compare semantically matched final stable compiled graphs and end-to-end timings; STOP when evidence is NOT_STABLE or benefit absent. Revert ineffective complexity. Record exact source/harness revisions and human-executed test status before declaring success.

**Anti-overengineering gate:** do not create a standalone heavyweight call plan, new permanent activation carrier, parameter namespace, mode-specific B5 ABI or unconditional per-Closure root wrapper. No process-wide mutable caches; no cost in trivial NORMAL for an unused capability. When scalar Load/Store is already optimal after PE, accept it without creating a bulk opcode merely for visual parity with GraalPy.

## Publication/human-executor boundary

- **Documentation publication** of this exact record and its evidence is explicitly authorized for agent direct commit/push to `guillermomolina/protos-project-docs` for this bounded task. Source/spec remain untouched.
- **Product implementation and publication** in `guillermomolina/protos` remain **human-executed**. Agent edits and prepares, human runs builds/tests/checks/commit/push. Do not change `pom.xml`/`CHANGELOG.md` before tests have passed; no tests after their final promotion.
- Neither an approved architectural policy nor a ready implementation Issue is a claim that compilation, tests, benchmark, diff check or runtime conformance were executed. No tests were run by the agent while preparing D202.

```text
OWNER_APPROVAL_DATE=2026-10-15
RATIFIED=D202_NORMAL_FIRST_PRELOWERING_CLOSURE_ENTRY
D003_AMENDED=NO
CALLABLES_NORMATIVE_AMENDMENT=NO
PLAT058_PROFILE_CHANGE=NO
PLAT060_B5_ABI_CHANGE=NO
IMPLEMENTATION_ISSUE=I104/#899
PROTOS_SOURCE_REVISION=5948ee7ce70d5a70636c921e31566483c4809323
AGENT_EXECUTED_BUILDS_OR_TESTS=NO
I104_PRODUCT_PUBLICATION=NOT_YET
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
