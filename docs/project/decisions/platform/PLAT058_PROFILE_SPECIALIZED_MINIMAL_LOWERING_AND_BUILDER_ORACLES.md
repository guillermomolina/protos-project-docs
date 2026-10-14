# PLAT058 — Profile-specialized minimal executable lowering and independent Builder-oracle validation

**Status:** RATIFIED — Candidate D, 2026-10-14.  
**Owner approval:** Explicit active owner statement: **"Aprobada"**, immediately following the PLAT058-D recommendation and the owner-imposed independent Builder-oracle acceptance rule. This is approval of **D as scoped below**, not approval for arbitrary new guest semantics, universal rematerialization or a second compiler.
**Decision issue:** [PLAT058/#891](https://github.com/guillermomolina/protos/issues/891).  
**Implementation issue:** [I101/#892](https://github.com/guillermomolina/protos/issues/892), independent top-level I-family owner; slices remain internal until an independent-identity trigger arises.  
**Product source checkpoint (read-only):** [`guillermomolina/protos@1b4cc76f4fdfb4eae7ac14d156a10b34cdabc5c0`](https://github.com/guillermomolina/protos/commit/1b4cc76f4fdfb4eae7ac14d156a10b34cdabc5c0). This is research baseline, not a target version or publication of the new architecture.  
**Comparative research, structural-oracle model and GITHUB021:** [PLAT058 owner approval and falsification evidence](../../evidence/PLAT058/PLAT058_D_OWNER_APPROVAL_INVARIANT_AND_ORACLE_EVIDENCE_2026_10_14.md).  
**Authority:** PLAT001 (exclusive Context language instance), PLAT005 (tooling tags), PLAT035/036/040/044, PLAT057-B (proof-driven generic Closure inlining), and governing normative `spec/semantics/CALLABLES.md`, `EXECUTION_AND_CONTROL.md`, `OBJECT_MODEL.md`, `VALUES_AND_COLLECTIONS.md`, `spec/PROTOS_GRAMMAR.md`. No normative specification change.

## Problem and selected architectural boundary

The current `CanonicalToBytecodeLowerer` can emit a NORMAL Boolean inline path **and** a DEBUG/ordinary send fallback separated at runtime by `IsNormalExecutionMode`. A subsequent Graal compiler *might* remove some overhead; that possibility is explicitly **not acceptable proof** that Protos generated the best available executable representation.

The project owner imposes a stricter lowering boundary:

> Assume Graal **does not** eliminate machinery unnecessarily emitted by Protos. For each admitted optimizer transformation, prove that Protos's own lowering emits the same justified minimal Bytecode DSL execution structure as an **independent handwritten Builder oracle**, before Truffle/Graal JIT optimization.

This is a **compiler-construction and acceptance invariant**, not a promise that every guest expression reduces to a constant. Dynamic/overridable behavior retains semantically required guards, errors, branches, and fallback.

**Selected D:** One shared semantic Canonical AST and proof system, **one lowerer**, but **profile-specialized physical emission** chosen once by the owning Context's immutable NORMAL/DEBUG selection. NORMAL emits minimum proven execution structure; DEBUG emits the source/activation/return provenance needed for correct debugging and late debugger attachment **within an already-DEBUG Context**. Truffle `@Instrumentation` and `BytecodeConfig` remain orthogonal, appropriate for tracing, tags and source metadata; they do not replace semantic lowering optimizations.

A NORMAL Context has no requirement to retrofit/debug its scopes by attaching a debugger. An earlier investigation incorrectly treated possible breakpoint suspension in NORMAL as a required invariant; the owner explicitly rejected that model. Do not preserve DEBUG-only lowering solely to support such an unapproved NORMAL late-attach contract.

## Required compilation ownership and reparse stability

1. Derive the profile once from `protos.Debug` at Context creation. With ratified PLAT001 `ContextPolicy.EXCLUSIVE`, the language instance belongs to one Context. Supply an explicit typed, immutable profile to `ProtosLanguage.parse` → `ProtosSourceCompiler.compileBytecode` → `CanonicalToBytecodeLowerer`. Use the destination Context's profile for `ProtosModuleRuntime` and the Context-local `ProtosClosureExecutionPlan` / `ProtosBytecodeClosureExecutionPlan.rebuildForLanguage` projection.
2. The profile is not a global, test-only, thread-local, or ambient entered-Context lookup in the lowerer. No constructor-time `ProtosLanguageContext.currentIfEnteredForRuntime()` and no emitted `IsNormalExecutionMode` branch for a proved, profile-fixed NORMAL optimization.
3. Fix **all decisions affecting executable topology** before the retained `BytecodeParser` starts emitting its initial `BytecodeRootNodes` group: proof admission, selected path, physical Closure/arity roots, root indices, metadata, source ownership, and plan-cell references. Reparse is deterministic/idempotent. In particular, replay must not re-decide a previous folding from `ProtosObjectValue.rootObject().isFrozen()`, a mutable external source or different current Context.
4. Maintain separation: the immutable semantic-emission profile cannot be switched by `BytecodeRootNodes.update(BytecodeConfig)`. Bytecode DSL may materialize more tags/sources/`@Instrumentation` within the **same** physical semantic profile; `ensureComplete()` must not change semantic paths or group identities. Adding instrumentation can invalidate compiled code, and instrumentations are not automatically removed after detachment.
5. Preserve original canonical source/binding/return-home analyses. Prefer a small **immutable proof-backed emission decision** or equivalent narrow internal representation over rewriting canonical sequences in place. Never bypass a suspension proof using `foldedLiteralControl || provesNonSuspendingBody`. Do not introduce universal IR, clone of the whole lowerer, unconditional virtual frame/activation or global cache.

## Exact owner-approved structural oracle contract

Every new lowerer optimization, and any existing optimization whose physical-elimination claim is being ratified, needs **independent handwritten Builder proof**:

- The expected executable shape is emitted directly through `ProtosSemanticBytecodeRootNodeGen.Builder`, independent of Protos parser/lowerer. The real Protos source goes through ordinary parse/canonicalize/lower.
- Compare complete `Instruction.getName()` plus **all constant operand signatures**, without filtering branches, sends or fallback, after `ensureComplete()` (and, when relevant, also before and across reparse). This is a *Bytecode DSL physical-generation comparison*, **not** a Graal final-graph comparison.
- Compare group topology where relevant: `BytecodeRootNodes.count()`, order, root identity, arity-rejection roots, retained Closure plans/CallTargets. Equality of one root's instruction list does not establish equality of entire root groups.
- Execute separately for semantic correctness, including mode parity where required and negative/non-admitted cases. The oracle is *structural*, not a substitute for guest-observable behavior tests.
- For guarded dynamic cases, build an independent Builder oracle **with exactly the necessary guard/fallback**, rather than incorrectly demanding an unconditional constant. Compare exact shape and demonstrate miss semantics.
- `@Instrumentation` may be omitted when disabled; source/tag expansion can be requested separately. Literal-only and optimized roots must use equivalent BytecodeConfig for comparisons.

**Primary required witness in NORMAL (valid syntax):**

```protos
() => { true.ifTrue() { 1 } }
```

Its **executable activation root** must have the same complete instruction signature as both `() => { 1 }` and a handwritten Builder literal-one return (including the canonical `MaterializeReturnCarrier` where needed). No `IsNormalExecutionMode`, prepared send/Boolean control, ordinary Closure call, debug branch or fallback in this admitted root. DEBUG must preserve correct observability and semantics. Group topology must be audited independently: do **not** claim the example test proved whole-group equivalence.

The owner supplied source of `ProtosI095AggressiveConstantFoldTest` and reports it **PASS** with a prototype fold; it is **not verified as published** in product HEAD. Preserve its independent-oracle structure, improve the group/reparse/mode/semantic negative checks, and never claim product implementation or test execution before the human reports it.

## Semantic and tooling invariants (no new language rules)

- Exact ordinary overridable selection, frozen standard `Object` publication proofs and `Object.call` override/fallback. Literal canonical Boolean is not license for unconditional dispatch bypass when the selected behavior is not proven.
- `CALLABLES.md`: genuine Closure identity/capture, extracted bound Closure freshness, `this`, methodHome, parameter binding/arity/default/rest/spread, stable selected receiver and error order.
- `EXECUTION_AND_CONTROL.md` D179: lexical bindings are real Context slots; `PRESENT` differs from `ABSENT`; `removeSlot`, subsequent recreation, shadowing and pre-RHS target selection remain exact; no hidden lexical-by-value capture.
- `VALUES_AND_COLLECTIONS.md` Boolean semantics: evaluate all argument-producing expressions in order; validate/call only selected argument, exactly once; preserve `ifTrue` exact result, `if`/ `else` receiver return, `and`/`or` canonical Boolean result validation; propagate Future/Errors/return home, ensure, suspension/cancellation.
- Preserve DEBUG's stepping/RootTag/SourceSection/scope observability, required arity-rejection roots and legitimate on-stack/deopt/continuation behavior. Allow NORMAL's intentionally more compact physical topology under an explicit, proven exception.
- Do not universally install `RootTask`, `RootActor`, `RootProcess`, materialized activation, global scheduler or arguments Array for cases that do not use them.

## Scope / deliberately rejected alternatives

- **A status quo + Graal cleanup:** no longer meets explicit physical-lowering acceptance.
- **B BytecodeConfig/@Instrumentation only:** removes tracing/probes when disabled but not ordinary semantic send/Closure/fallback instructions.
- **C two full independent lowerers or modes sharing unsound canonical mutation:** excessive drift/maintenance risk.
- **D chosen:** one semantic compiler, mode-specialized physical emission, conservative reusable proofs and structural Builder oracles.
- **E dynamic mode regeneration for NORMAL late debug:** no approved need; introduces on-stack/continuation and migration complexity.

Do not rewrite GraalJS/GraalPy architecture into Protos: GraalJS uses AST factories (`JSAddNode.create` folds literals, specializes constant-operand adds; `IfNode.create` swaps `!condition` arms), and AST `materializeInstrumentableNodes` can reconstruct richer tagged nodes (`JSLeftShiftConstantNode`). GraalPy emits `TraceLine` as `@Instrumentation` and enables it via `BytecodeConfig`. They demonstrate **pre-Graal minimal physical construction** and optional instrumentation, not proof that their raw node counts or semantics match Protos. Truffle `BytecodeParser` idempotence is binding.

## Incremental delivery and stop conditions

I101 owns three cohesive internal slices: (A) explicit profile throughout **all** entry/rebuild routes plus first end-to-end physical folding and handwritten oracle; (B) reuse proof-admission machinery for broader eligible selectors/calls and their oracle/negative tests; (C) comprehensive physical group/DEBUG/reparse/Task acceptance, optional Native/compiled-graph performance evidence as **secondary**, not substitute. Do not create micro-issues for patch-sized subdivisions.

**Stop and ask for a separate design decision** if a proposed optimization needs changed observable semantics, globally modified activation identity, a mandatory universal virtual-frame system, or dynamically replacing an executing NORMAL root with DEBUG. Failure of a proof means **fallback**, not semantic approximation.

**Implementation/testing/publication:** Agent edits `guillermomolina/protos` files. Human runs builds, tests, diagnostics and Git add/commit/push. `pom.xml`/`CHANGELOG.md` only after human-reported green substantive validation immediately before commit; do not rerun tests after metadata edits. Docs publication under `guillermomolina/protos-project-docs` is a separately authorized agent task. No implementation, tests, benchmarks or performance gain claimed by this decision record.
