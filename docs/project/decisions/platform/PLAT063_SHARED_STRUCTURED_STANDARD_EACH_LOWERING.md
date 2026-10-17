# PLAT063 — Shared structural Bytecode lowering of canonical standard each

**RATIFIED Candidate C — 2026-10-17.** Exact project-owner selection: **“Apruebo C”**, selecting the immediately preceding alternative C (“Bucle Bytecode común”) rather than A (keep) or B (factor only Java). [Issue #908](https://github.com/guillermomolina/protos/issues/908). Product source snapshot: [`c00b36c0edaf6678b7d8101de3a405124f179644`](https://github.com/guillermomolina/protos/commit/c00b36c0edaf6678b7d8101de3a405124f179644). [Bounded comparative research and GITHUB010/GITHUB021 evidence](../../evidence/PLAT063/PLAT063_C_FOUR_FAMILY_AUDIT_AND_APPROVAL_2026_10_17.md).

## Approved architecture

For the **five canonical standard** `Array.each`, `Bytes.each`, `Map.each`, `IdentityMap.each`, `Environment.each`, keep each collection's already selected standard behavior, validation, snapshot and callback arguments authoritative. In **PLAT042's existing untagged structured-dispatch interpreter**, use the already defined `PreparedLocalEachCall` view (or equivalent bridge respecting its sole authority) to emit **one common Bytecode DSL structural `While`**, retaining:

```text
selected canonical each -> collection-specific preparation
TryFinally(complete original prepared call exactly once):
    While(StructuredLocalEachHasNext):
        PrepareStructuredLocalEachChildCall
        existing scoped child invocation (suspend/resume/error/NLR-safe)
        AdvanceStructuredLocalEach only after normal child completion
    FinishStructuredLocalEach -> exact original receiver
```

This is a **physical source-to-emitted-operation granularity** choice. It does not permit implementing the entire loop/callback as one opaque Java `@Operation`, moving `Yield` or `TryFinally` outside the correct semantic control scope, or creating a new semantic protocol. A Java emitter helper that still emits five separate structural loops is **B**, not ratified C. The adapter operations should be bounded and specialize appropriately; no unconditional per-call/callback wrapper, second cached semantic authority or new global state.

Do not change `CanonicalToBytecodeLowerer.emitLocalInlineLiteralEach` (already optimized by PERF026-D and PLAT044), the canonical semantic source roots/RootTags, ordinary Closure call/continuation (PLAT062), D013 callback lookup, D179/PLAT036 lexical authority, PLAT057–061 or I095/I101/I104/I106/I107. Foreign projected `each` (D188/D192) remains a separate pull/provider path. Range/Set/IdentitySet and aggressive inlining remain PERF045/#889.

## Invariant/delta review (GITHUB021)

**DECISION_APPROVAL_PROVENANCE=PASS** (exact owner “Apruebo C”). **DECISION_INVARIANT_CONSISTENCY=PASS for the selected architectural contract**, not implementation verification:

- PLAT042: one existing structured-only helper interpreter, no new physical callback root per element.
- PLAT044: semantic callback identity and already-admitted inline path unchanged; unadmitted callback retains ordinary physical invocation.
- D013 and CALLABLES: callbacks remain ordinary overridable/copiable `call` targets; selection and validation timing unchanged.
- D179/PLAT036: `context`, `this`, `methodHome`, capture by reference and frame-backed presence authority unchanged.
- D202/D203/D204, PLAT057–061: no new proof engine, optimization phase, IR, ABI or changed callback binding.
- Array/Bytes indexed snapshot; Map/IdentityMap insertion-order association snapshot; Environment complete portable conversion/validation before callback 1; original receiver return; callback once in source/snapshot order; no cursor advance on failed/suspended callback.
- Same `TryFinally`/completion, `Yield`/resumption, Error/NLR/`ensure`/cancellation; NORMAL/DEBUG instrumentation, root topology, reparse, multi-Context, Native Image and pay-as-you-grow.

**If actual implementation needs a change to a ratified invariant, STOP for owner decision** rather than treating C's name as implicit approval. No normative spec changes are authorized.

## Decision scope and follow-through

Four-family PLAT063 audit: **(1)** ordinary pre-call send already encapsulated, PLAT062 owns call; **(2)** lexical/object slots justified decomposition, PERF034/PERF036; **(3)** object/Map construction justified due to semantic and suspending boundaries; **(4)** standard collection `each` is the sole C candidate. The separate I-family implementation owner makes product edits under human-executor policy; PERF045 remains independently responsible for **measured** each-family performance and aggressive cross-library inlining. No product change, new test PASS, Graal graph, Native Image test or speedup follows from this ratification. The owner's already-green local tests and clean diff are **pre-implementation status only**. Reversal requires only restoring the former private emitted loops; no public API/guest migration.
