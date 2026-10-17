# PLAT065 — Unified Bytecode DSL interpreter with selective semantic RootTag

**Status:** RATIFIED — Candidate A (project-owner approval, 2026-10-17).
**Owner:** [PLAT065/#918](https://github.com/guillermomolina/protos/issues/918).
**Implementation owner:** I111 (formal implementation issue, allocated separately).
**Product checkpoint inspected:** [`62ca4eb602432f3457c080954b716f8466b76983`](https://github.com/guillermomolina/protos/commit/62ca4eb602432f3457c080954b716f8466b76983), Protos 0.3.373-SNAPSHOT.
**Evidence:** [PLAT065 research baseline](../../evidence/PLAT065/PLAT065_INTERPRETER_UNIFICATION_BASELINE_2026_10_17.md) and [owner approval/invariant reconciliation](../../evidence/PLAT065/PLAT065_A_OWNER_APPROVAL_AND_IMPLEMENTATION_GATE_2026_10_17.md).

## Exact selection

The owner explicitly said **"apruebo A"** after the concrete A mechanism had been explained, with the following selected commitments:

1. **ONE** generated Truffle Bytecode DSL interpreter (`@GenerateBytecode`) for ordinary semantic source activations and internal structured-dispatch / Task / I/O C-prime roots. One generated *interpreter* does not mean one physical guest root or one CallTarget.
2. Set `enableRootTagging=false` and explicitly bracket **only real semantic source roots** with `StandardTags.RootTag` at emission. Keep the separate nested semantic RootTag regions needed for inlined Closure diagnostic frames. Infrastructure/helper C-prime roots must never masquerade as guest function activations.
3. Keep `enableTagInstrumentation=true` where required for supported tooling and lazy semantic Error tracing; retain `enableRootBodyTagging=false`. Do not introduce universal `RootBodyTag` or eager DEBUG instrumentation in NORMAL.
4. Preserve the **immutable, Context-owned NORMAL/DEBUG physical-emission profile** selected by PLAT058 and the separate `BytecodeConfig` materialization mechanism. No successful-NORMAL hot-path mode branch, late-DAP attach contract, or extra runtime universe.
5. Preserve exact guest semantics, source/source sections, debugger stepping, lexical/native/materialized locals and captured Closures, top-level and module identity, bytecode-group/reparse behavior, PLAT014 C-prime yield/resume/ensure/Error/ReturnHome propagation, default arguments and D013 ordinary calls. No global mutable root-kind registry, AST fallback, copied lexical authority or runtime workaround.
6. Merge shared/unique source and helper operation surfaces semantically; do not blindly merge same-named operations whose operands/guards/cache semantics diverge (notably `ReadMember`, `BindClosureFrameParameter`, `PrepareSendArguments`, `EnterNestedStructuredDispatch`, `CompleteClosureCall`, `FinishClosureCall`, `IsContinuation`). Retain legitimate physical continuation roots and CallTargets.

## GITHUB021 invariant reconciliation

- **PLAT026/#375:** explicitly and *narrowly revised* the earlier no-manual-`RootTag` choice. The underlying no-false-guest-root and truthful source/debugger commitments remain binding. Manual tags are a supported emitter mechanism, not proof of arbitrary equivalence. Protos currently has no `@Prolog` on either generated root; no speculative future prolog compatibility is a pre-implementation gate. If a prolog is later introduced, re-audit its ordering.
- **PLAT041/#759 / PLAT042/#760:** prior two-generator selection was a valid historical response to then-known root-tag and C-prime constraints; this later owner-approved decision supersedes **only** the two-generated-interpreter topology, not the inline Object-body, nested lexical grouping, continuation, or semantic-root invariants.
- **PLAT014, PLAT044/CLI008, PLAT058/#891:** explicitly preserved. Lazy `RootTag` tracing in NORMAL on terminal Error and exact inlined Closure semantic frames remain required.
- **GITHUB010:** earlier research compared alternatives (manual tags, auto-tags on all roots, eliminating helper physical roots, status quo). This owner's selection does not claim that community precedence proves debugger parity or performance. Correctness is verified as **implementation regression acceptance**, not a new research authorization checkpoint.

## Implementation and acceptance

The next work is **TIPO=IMPLEMENTACIÓN** in `guillermomolina/protos`, governed by `AGENTS.work/IMPLEMENTATION.md`; human executes builds/tests and product Git mutations. Grouped phases:

- **A — Safe RootTag changeover:** while both generator classes exist, disable automatic semantic root tagging, add explicit outer semantic-root tag to `CanonicalToBytecodeLowerer.emitRootBody`, preserve existing nested RootTags, update claims/tests and verify NORMAL Error trace + DEBUG tooling + reparse/source events. No helper tagging.
- **B — Single-generator operation and producer migration:** use the semantic generator as retained class, unify operation contracts with collision-specific adapters/names where necessary, migrate structured-dispatch / C-prime Task and I/O builders and consumer types without changing continuations, yield or sources.
- **C — Retire redundant generator and strengthen acceptance:** eliminate the old generated helper interpreter, reconcile role-based diagnostics/scope exports, assert one `@GenerateBytecode` product interpreter and no remaining references, verify affected/full tests and independent builder oracle gates from PLAT058.

A and B/C may be regrouped when required by real dependency/buildability. Every publication must leave a coherent, testable intermediate state. **No compilation, validation or product edit is claimed by this decision.** User's statement that all local tests passed and `git diff --check` was clean pertains to the existing checkout before I111 implementation, not post-change validation.

`PLAT065_CANDIDATE=A`
`OWNER_APPROVAL=EXPLICIT`
`SPEC_CHANGE=NO`
`PRODUCT_IMPLEMENTATION=PENDING_I111`
