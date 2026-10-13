# PLAT057 — Proof-driven generic Closure lexical inlining

**Status:** RATIFIED — Candidate B (conservative, semantics-preserving canonical/Bytecode DSL inline lowering).
**Project-owner approval:** 2026-10-13, exact active owner statement: “ok apruebo B” after the second investigation incorporating SOMns, TruffleSqueak, Dolphin and Protos's existing lexical authorities.
**Decision issue:** [PLAT057/#881](https://github.com/guillermomolina/protos/issues/881).
**Product source checkpoint:** [`guillermomolina/protos@57a43b6d94697f9a6256e36c7a775dfa4a1218d7`](https://github.com/guillermomolina/protos/commit/57a43b6d94697f9a6256e36c7a775dfa4a1218d7), `0.3.343-SNAPSHOT` (published I095 checkpoint).
**Comparative research / evidence:** [PLAT057 Candidate B evidence](../../evidence/PLAT057/PLAT057_CANDIDATE_B_RESEARCH_APPROVAL_AND_FALSIFICATION_2026_10_13.md).
**Previously ratified platform boundaries:** PLAT035 (single Bytecode DSL backend); PLAT036 (binding identity/presence and authority); PLAT040 F′ (guarded dispatch, compact call ABI, conditional guest materialization); PLAT041–044 (semantic source roots and approved narrow inline callback RootTag exception).
**Normative authority:** `guillermomolina/protos:spec/semantics/CALLABLES.md`, `EXECUTION_AND_CONTROL.md`, `OBJECT_MODEL.md`, `VALUES_AND_COLLECTIONS.md`. This record changes none of them.

## Selected architecture: B

The compiler MAY expand an **ordinary, semantically selected Closure invocation** into the containing semantic Bytecode root when a conservative, explicit proof establishes observational equivalence. This mechanism is **generic by invocation/lexical properties**, rather than a special implementation dedicated to `ifTrue`, `whileTrue`, `each`, a benchmark, or any one selector.

1. **Ordinary selection is authoritative.** Preserve D013 message lookup, `Object.call` and possible overrides, Closure identity and extraction freshness, receiver/`this`, `methodHome`, argument-evaluation order, arity/default/rest/spread rules, error timing, and exact validity/invalidation checks. Inline only the *effective selected* behavior with a sound guard. Do not infer that a syntactically known Closure can bypass a changed `call` selector.
2. **Physical storage may be shared; semantic scopes must not collapse.** The admitted invocation retains one fresh semantic activation/context identity, distinct own-binding identity, lexical parent relationships and correct capture by reference. Callback-owned bindings may occupy distinct Bytecode-local positions in the **caller physical frame**, as they already do in eligible PLAT044 B′ callback regions. Name collision, shadowing and recursive/reentrant activations must not alias semantically distinct bindings.
3. **D179 is mandatory.** Compile-time `CanonicalBindingIdentity` does not imply `PRESENT`. `removeSlot`, later creation, nearer-scope retargeting, ordinary OPEN/CLOSED/FROZEN mutation rules and the distinction between `null` and `ABSENT` remain exact. A captured write resolves and retains its destination **before** RHS evaluation and must not silently re-resolve it after a side effect.
4. **Prove durability before removing a physical root/authority.** Unobserved own locals can remain in inline storage only while no escaping or observing path needs a durable authority. `context`, reflection, a nested Closure capturing the activation, storage/escape across return, suspension/continuation, debug/tooling and return-home/unwind require either a proved identical durable materialization before the required lifetime boundary or the ordinary physical fallback. A value copy is never a substitute for capture by reference. Preserve exact identity across repeated observation.
5. **Preserve already-approved tooling scope.** Retain SourceSections, tags, breakpoints, stepping, instrumented scope projection, NORMAL/DEBUG contract, OSR/deopt and PLAT044 B′ custom RootTag/physical-frame delta. PLAT057 does **not** automatically expand the approved special-case omission of a physical `FrameInstance` to all ordinary Closures: each new admitted class must prove or separately obtain approval for its precise tooling/topology effect.
6. **General fallback and bounded budgets.** Dynamic/polymorphic/overridden, recursive or otherwise unproven call sites use the existing semantic `RootCallTarget` + `DirectCallNode`/`IndirectCallNode` and compact invocation machinery. The compiler must avoid unbounded tree expansion, code size, invalidation and excessive compile time. Ordinary Truffle/Graal inlining remains available on fallback.
7. **Pay as you grow.** No universal virtual-frame/rematerialization engine, global activation registry, new general synthetic debugger stack, eager `RootTask`/`RootActor`/`RootProcess` or mandatory `Context`/arguments Array is authorized for the trivial synchronous case. Reuse `CanonicalLexicalScope`, `CanonicalBindingResolution`, `ProtosFrameLexicalLayout`, `ProtosFrameLexicalBindingAuthority`, `ProtosInlineCallbackFrameBindings` and existing activation/materialization interfaces when their exact invariants admit reuse. Avoid parallel binding authorities.
8. **Compiler-layer distinction.** Source/Canonical → Bytecode inline expansion and physical Root omission differ from Truffle's compiled CallTarget inlining and Graal escape analysis. A Graal-inlined callee is not evidence that source-level roots/semantic storage have been eliminated.

### Admission, not blanket inclusion

B does not assert that every Closure is inlineable. Nested owned locals and read/write captures are **not categorically excluded** merely because they exist; prove the individual shape. In particular, a nested Closure currently makes `CanonicalToBytecodeLowerer.demandsPersistentFrame` true. Removing that conservative requirement globally is **not approved**; any relaxation must independently preserve shared lexical authority, durable identity and escape.

The first proof-driven increment should target a **generic ordinary direct Closure call**, not yet another Boolean-only fast path, with a bounded C0/C1 and local-binding/capture discriminator where current source permits sound admission. Broader N-argument, arbitrary dynamic methods, recursion, polymorphic PIC expansion and profile-guided adaptive admission follow only after independent evidence.

## Exclusions and stop conditions

- Candidate C (universal caller-frame virtual activations with general rematerialization) **NOT selected**.
- Candidate D (profile-guided speculative/hybrid inliner) **NOT selected** as a mandatory runtime/compiler framework; any future adoption is a separate PLAT decision/amendment.
- Candidate A (status quo only) and E (indefinite deferral) are not the selected target, but ordinary fallback retains A's implementation path.
- No Protos syntax, Boolean messages or other observable language semantics change.
- No promise of performance improvement and no benchmark-specific shortcut. If a proposed admitted shape cannot preserve escape, D179, debugger/RootTag or suspension semantics without new unapproved infrastructure, it **must fall back**, not approximate the semantics.
- No direct product code, benchmark, test execution, Native/JIT result or version change is entailed by this architecture ratification.

## Evidence and release/validation gates

Before claiming an optimization benefit, inspect **matched current-product / harness / Truffle compiler tier** After-TruffleTier BGV, call-tree inlining and compiler stability, surviving call/guard/frame-state/allocation nodes, generated code bytes, deopt and stable timing. The historical pre-I095 `primitive-closure-call` Protos T2 4,533-node capture already had an inlined child; it does not establish today's result or that a new source-level inline lowering removes those nodes.

Protect at least: `primitive-closure-call`, `primitive-if-true`, `primitive-if-false`, `primitive-return-literal`, ordinary C0/C1 and a nontrivial own-locals + mutable-capture discriminator, with tests for nested escaping capture, Context identity, `removeSlot` then recreate, shadowing, pre-RHS writable destination, extraction/overridden `call`, defaults/arity/error precedence, return-home/`ensure`, recursion, suspension and NORMAL/DEBUG tooling.

Tests/builds/product commits and benchmark execution belong to the **human executor** under product `AGENTS.md`. Implementation slices should be cohesive and independently correct; update `pom.xml`/`CHANGELOG.md` only after the human reports all required tests passing, immediately before product commit and push, without running tests afterwards, as required by the owner's established workflow.

## GITHUB021 existing-invariant reconciliation

| Authority | Retained invariant / exact PLAT057 delta |
| --- | --- |
| D013 / `CALLABLES.md` | Closure remains the only callable kind; `call` ordinary/overridable; extraction freshness and identity remain; no selector privilege |
| D179 / `EXECUTION_AND_CONTROL.md` | Context ordinary object; static identity != presence; exact remove/recreate and pre-RHS write-target selection; **no semantic delta** |
| PLAT035/036 | Single Bytecode DSL execution backend, exact binding authority and captured-reference semantics remain; **inline physical placement may extend for proven shapes only** |
| PLAT040 F′ | Guarded selection, compact ABI, conditional materialization and pay-as-you-grow retained; B may additionally eliminate the separate physical call for admitted shapes |
| PLAT041–044 B′ | Existing source-root/RootTag and special inline callback frame topology stays approved **within its original limits**; no generic debugger-stack or all-Closure frame-cardinality change |
| PERF041 / PERF042 / I095 | Historical performance/implementation owners remain separate; benchmark causality pending; published I095 source remains source checkpoint, not an automatic generic inliner |

**Approval provenance:** the project owner expressly wrote “ok apruebo B” on 2026-10-13 in the active PLAT057 follow-up after being shown candidate B's revised 12-axis evaluation, source precedents, exclusions and graph limitations. This approval is **not** implementation acceptance, authorization of C/D, a relaxation of D179 or proof of JVM benefit.

```text
PLAT057_SELECTED_CANDIDATE=B
PLAT057_OWNER_APPROVAL=EXPLICIT_2026-10-13
PLAT057_DECISION=RATIFIED
GITHUB010_PRIOR_ART=SOMNS_TRUFFLESQUEAK_DOLPHIN_SELF_IO_CROSS_TRUFFLE
GITHUB021_INVARIANT_CONSISTENCY=PASS
PROTOS_SOURCE_REVISION=57a43b6d94697f9a6256e36c7a775dfa4a1218d7
SPEC_DELTA=NONE
PRODUCT_IMPLEMENTATION=NOT_AUTHORIZED_AS_COMPLETED
MATCHED_POST_I095_GRAPHS=PENDING
HUMAN_TEST_RESULTS_FOR_PLAT057_IMPL=NOT_RUN
```
