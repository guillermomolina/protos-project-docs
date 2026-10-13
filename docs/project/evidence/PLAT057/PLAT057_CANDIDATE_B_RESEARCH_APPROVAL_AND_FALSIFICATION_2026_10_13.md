# PLAT057 — Candidate B comparative research, owner approval and falsification

**Date:** 2026-10-13.
**Live architecture issue:** [PLAT057/#881](https://github.com/guillermomolina/protos/issues/881).
**Ratified decision:** [PLAT057 Candidate B](../../decisions/platform/PLAT057_PROOF_DRIVEN_GENERIC_CLOSURE_LEXICAL_INLINING.md).
**Owner approval:** explicit active conversation: “ok apruebo B”. This is approval of the latest **conservative proof-driven B**, rather than earlier investigatory preferences for profile-guided D.
**Exact inspected Protos checkpoint:** `57a43b6d94697f9a6256e36c7a775dfa4a1218d7` (published I095, `0.3.343-SNAPSHOT`). No product source was changed for this research.
**Research boundary:** read-only source and retained evidence; no tests, compilation, Native Image, BGV or new benchmarks executed by the agent; no matched post-I095 graph available at ratification.

## 1. Decision and motivation

Investigate source/Canonical → Bytecode DSL **generic Closure invocation inlining**, distinct from the narrow standard-control inline PLAT044 B′ and from Truffle/Graal CallTarget PE inlining. Primary question: can caller physical-frame locals represent independently owned lexical bindings for ordinary direct Closure calls, without altering semantic activation/context identity, D179 presence, captured by-reference writes, lifetime or tooling?

Selected: **B** — bounded proof-driven inline expansion, exact ordinary dispatch and fallback, using established Protos lexical/runtime mechanisms. General speculative profile-driven inline adaptation (**D**) and universal virtual activation rematerialization (**C**) are not approved.

## 2. Direct prior-art sources and exact limits

| System | Source | Established mechanism | What it does NOT establish for Protos |
| --- | --- | --- | --- |
| SOMns (Truffle Smalltalk) | [Method.inline](https://github.com/smarr/SOMns/blob/e56f4c2815cef4cae2045a9b05dc1fd190315ba6/src/som/interpreter/Method.java), [MethodBuilder.mergeIntoScope](https://github.com/smarr/SOMns/blob/e56f4c2815cef4cae2045a9b05dc1fd190315ba6/src/som/compiler/MethodBuilder.java), [LexicalScope](https://github.com/smarr/SOMns/blob/e56f4c2815cef4cae2045a9b05dc1fd190315ba6/src/som/interpreter/LexicalScope.java) | Actual AST block inlining, physical FrameSlot relocation, nested lexical-scope adaptation, name qualification | Dynamic D179 context membership; universal arbitrary-Closure caller-frame inlining |
| SOMns read/write and NLR | [LocalVariableNode](https://github.com/smarr/SOMns/blob/e56f4c2815cef4cae2045a9b05dc1fd190315ba6/src/som/interpreter/nodes/LocalVariableNode.java), [NonLocalVariableNode](https://github.com/smarr/SOMns/blob/e56f4c2815cef4cae2045a9b05dc1fd190315ba6/src/som/interpreter/nodes/NonLocalVariableNode.java), [ReturnNonLocalNode](https://github.com/smarr/SOMns/blob/e56f4c2815cef4cae2045a9b05dc1fd190315ba6/src/som/interpreter/nodes/ReturnNonLocalNode.java) | Lexical access retargeting; nonlocal return can turn local after scope merge | Protos return-home/Task/continuation semantics automatically preserved |
| SOMns admission and fallback | [IfInlinedLiteralNode](https://github.com/smarr/SOMns/blob/e56f4c2815cef4cae2045a9b05dc1fd190315ba6/src/som/interpreter/nodes/specialized/IfInlinedLiteralNode.java), [WhileInlinedLiteralsNode](https://github.com/smarr/SOMns/blob/e56f4c2815cef4cae2045a9b05dc1fd190315ba6/src/som/interpreter/nodes/specialized/whileloops/WhileInlinedLiteralsNode.java), [BlockPrims](https://github.com/smarr/SOMns/blob/e56f4c2815cef4cae2045a9b05dc1fd190315ba6/src/som/primitives/BlockPrims.java), [BlockInlining tests](https://github.com/smarr/SOMns/blob/e56f4c2815cef4cae2045a9b05dc1fd190315ba6/core-lib/TestSuite/BasicInterpreterTests/BlockInlining.ns) | Bounded selector/literal AST expansion; ordinary blocks use guarded DirectCallNode and IndirectCallNode; shadowing/nested blocks covered by tests | It does not have a proven all-dynamic-calls rematerialization engine; selected TODO fallback cases exist |
| TruffleSqueak | [DispatchValueNode](https://github.com/hpi-swa/trufflesqueak/blob/77bbbec529e9869c3b94173235294032576547f8/src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/dispatch/DispatchValueNode.java), [Context creation](https://github.com/hpi-swa/trufflesqueak/blob/77bbbec529e9869c3b94173235294032576547f8/src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/context/GetOrCreateContextWithoutFrameNode.java), [StartContextRootNode](https://github.com/hpi-swa/trufflesqueak/blob/77bbbec529e9869c3b94173235294032576547f8/src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/StartContextRootNode.java), [ContextObject](https://github.com/hpi-swa/trufflesqueak/blob/77bbbec529e9869c3b94173235294032576547f8/src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/model/ContextObject.java) | Cached CallTarget/DirectCallNode block dispatch; lazy guest Context, frame-based materialization and invalidation when needed | No verified universal AST merge of arbitrary methods/blocks into caller physical frame |
| Dolphin Smalltalk | [LexicalScope](https://github.com/dolphinsmalltalk/Dolphin/blob/master/Core/DolphinVM/Compiler/LexicalScope.cpp), [optimized-message parser](https://github.com/dolphinsmalltalk/Dolphin/blob/master/Core/DolphinVM/Compiler/ParseInlinedMessages.cpp) | Optimized scopes; copying/relocating declarations; Stack/Copied/Shared temps, promotion for writable captured temps | Not Truffle, not dynamic Protos Context slot absence/presence |
| Self | [Self language](https://selflanguage.org/) and [Self compiler](https://www.sun.com/research/self/) historical primary literature; see PLAT044 comparative corpus | Semantically guarded ordinary-message inlining, optimized lexical representations and deoptimization/reconstruction in optimizing VM | Not an available Bytecode DSL automatic rematerialization framework |
| Io | [IoBlock activation C](https://github.com/IoLanguage/io/blob/d70be0683e10df3268c1f0a4b016c4b1fe0af129/libs/iovm/source/IoBlock.c), [Io Call C](https://github.com/IoLanguage/io/blob/d70be0683e10df3268c1f0a4b016c4b1fe0af129/libs/iovm/source/IoCall.c) | Dynamic objects for locals and a Call per activation; optional explicit recycling of uncaptured locals/Call | Recycling is not allocation avoidance or Truffle compiled inline |
| GraalJS/GraalPy/TruffleRuby | [PERF042-A2 sourced comparison](../PERF042/PERF042_A2_CURRENT_HEAD_INVOCATION_ATTRIBUTION_2026_10_10.md), [PLAT040 platform comparison](../../decisions/platform/PLAT040_TRUFFLE_HOT_PATH_INVOCATION_ARCHITECTURE.md) | Guarded direct/indirect call paths, definition/code-root caching, compact invocation/frame handling, PE | No equivalent guarantee for user-mutable `Object.call` and Protos D179 |
| V8 | Established speculative inlining/deopt literature referenced by PLAT044 evidence | Budgets, guarded speculative optimized code and fallback/deopt | Not Truffle Bytecode DSL, significant bespoke compiler-engineering surface |
| Espresso, Sulong, SimpleLanguage, Apple Pkl | [PLAT044 prior comparative survey](../PLAT044/PLAT044_SEMANTIC_CLOSURE_PHYSICAL_ROOT_DECISION_EVIDENCE.md) and [PLAT040](../../decisions/platform/PLAT040_TRUFFLE_HOT_PATH_INVOCATION_ARCHITECTURE.md) | Relevant direct target, expression/root ownership and generic Truffle optimization contrasts | Different language/call-state semantics; no demonstrated drop-in Protos lexical-context rematerializer |

The strongest source precedent is **SOMns `Method.inline` + `MethodBuilder.mergeIntoScope` + `ScopeAdaptationVisitor`**, not a Graal flag. TruffleSqueak independently supports lazy guest context projection, not proof of merged rootless activations. Dolphin validates scope/storage separation. Prior surveys of the wider Truffle ecosystem remain incorporated by reference rather than falsely claiming a fresh audit of every upstream HEAD on this date.

## 3. Protos concrete source baseline

| Current source / location | Current fact | PLAT057-B consequence |
| --- | --- | --- |
| [CanonicalLexicalScope.java](https://github.com/guillermomolina/protos/blob/57a43b6d94697f9a6256e36c7a775dfa4a1218d7/src/main/java/com/guillermomolina/protos/execution/CanonicalLexicalScope.java) | ROOT/CLOSURE/OBJECT_BODY, declaration vs progressively established bindings, genuine outer-scope skipping of object bodies | Preserve semantic scope tree; do not collapse object-body lookup |
| [CanonicalBindingResolution.java](https://github.com/guillermomolina/protos/blob/57a43b6d94697f9a6256e36c7a775dfa4a1218d7/src/main/java/com/guillermomolina/protos/execution/CanonicalBindingResolution.java) | `Resolved`, `Candidate`, `CapturedResolved`, `Dynamic` classify identity but not D179 permanent presence | Admission must preserve dynamic nearer presence/fallback |
| [CanonicalToBytecodeLowerer.java](https://github.com/guillermomolina/protos/blob/57a43b6d94697f9a6256e36c7a775dfa4a1218d7/src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java) | Lines ~1305–1336 conservatively mark nested Closure/Object/Compose persistent; ~5647+ emit B′ inline callback body with separate callback own locals; ~7396+ I095 fast path | Reuse existing lowering and prove specific new ordinary invocation classes, not a global `false` on nested-Closure |
| [ProtosFrameLexicalLayout.java](https://github.com/guillermomolina/protos/blob/57a43b6d94697f9a6256e36c7a775dfa4a1218d7/src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalLayout.java) | D179 `presentContinuity` Assumption never renewed after PRESENT→ABSENT; source reparse layout consistency | Scope-specific guards and parser replay stable layout required |
| [ProtosFrameLexicalBindingAuthority.java](https://github.com/guillermomolina/protos/blob/57a43b6d94697f9a6256e36c7a775dfa4a1218d7/src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java) | Retained `MaterializedFrame` authority preserves shared bindings across escaping children; one authoritative store | Safe durable fallback/reference capture, no value copy |
| [ProtosInlineCallbackFrameBindings.java](https://github.com/guillermomolina/protos/blob/57a43b6d94697f9a6256e36c7a775dfa4a1218d7/src/main/java/com/guillermomolina/protos/execution/ProtosInlineCallbackFrameBindings.java) | Unobserved block locals, lazily materialized activation, D179 read/writes, pre-RHS destination, captured fast path | Preferred reusable physical mechanism, only under verified lifetime and identity proof |
| [ProtosExecutionMode.java](https://github.com/guillermomolina/protos/blob/57a43b6d94697f9a6256e36c7a775dfa4a1218d7/src/main/java/com/guillermomolina/protos/execution/ProtosExecutionMode.java) | Immutable NORMAL/DEBUG behavior; NORMAL rejects unsupported debugger scopes, DEBUG has provenance | Never invent on-demand dynamic execution-mode switch |

Useful existing test: [ProtosPerf025LazyLexicalCaptureTest](https://github.com/guillermomolina/protos/blob/57a43b6d94697f9a6256e36c7a775dfa4a1218d7/src/test/java/com/guillermomolina/protos/execution/ProtosPerf025LazyLexicalCaptureTest.java) proves a pair of distinct closures share one capturable outer context identity and tests D179: remove an inner `x` (2), fall back to outer `x` (1), recreate inner `x` (3), yielding `[2,1,3]`. The same test covers escaping mutating capture, preserved null-versus-absent and nested lexical depth. Do not weaken these tests.

## 4. Historical graph evidence: applicability limits

[PERF042-A2](../PERF042/PERF042_A2_CURRENT_HEAD_INVOCATION_ATTRIBUTION_2026_10_10.md) records **one eligible historical clean 2026-10-08** `primitive-closure-call` graph at Protos product `d59da442fd9bc6cf590fb9b2c82d3ff55c0391ae`, harness `98abc9af7a05a45a7d4056b72f36aef5889cb76f`: Protos Tier 2, child already inlined, **4,533 nodes**, versus GraalJS **13** and GraalPy **50**, under that historical capture. It includes historical Protos 997 FrameState, 285 If, 201 InstanceOf, 165 FixedGuard, 45 Deoptimize and a 44-invoke count of mixed provenance. These are not proven hot-path costs, disjoint partitions, allocation counts or a current-I095 graph.

A later historical capture at product `0db24f00ff2d92d642351d7f7535517fe01a55ce`, dirty harness `d9719253b628faad1ca296daef4a7ed566196216`, remained GRAPH_NOT_STABLE/Tier 1: **not admissible for graph parity**. I095 publication `57a43b6d` includes human-reported tests/checks PASS and the published code, but **no matched accepted post-I095 graph**.

Source-only hypotheses requiring causal graph verification:
- **H1** fused `TryDirectCallOne` PIC receiver-identity miss for a newly created `identity` Closure, although definition-keyed alternative path exists; preserve overridden `Object.call` selection.
- **H2** nested `CanonicalClosure` forces `demandsPersistentFrame` conservatively; relaxation could impact graph, but has highest identity/escape risk.
- **H3** differing caller activation references and composed call staging; verify which pieces remain on compiled hot path.

Do not claim any post-B speedup, CodeSize win or allocation elimination before the human supplies matched graphs. These unknowns are expressly **not** a rejection of the B architectural choice; they gate claiming benefits and expanding implementation admission.

## 5. Complete twelve-axis decision scorecard

Scale 1 (unfavorable) to 5 (favorable); comparative architecture judgment, not performance data. A=status quo/Graal only, B=conservative proof-driven inline, C=universal virtual-activation framework, D=profile-guided speculative hybrid, E=defer. **Uncertainties:** A/E present semantics are HIGH confidence; B's source feasibility MEDIUM; C speculative performance LOW; D overhead MEDIUM–LOW.

| # / axis | A | B | C | D | E | Brief axis-specific justification |
| --- | ---: | ---: | ---: | ---: | ---: | --- |
| 1 Correctness | 5 | 4 | 2 | 4 | 5 | Existing physical path has known invariants; B/D require specific proofs; universal C multiplies identity/escape cases (B MEDIUM) |
| 2 Protos alignment | 4 | 5 | 2 | 4 | 3 | B admits arbitrary ordinary callable sites by properties, no privileged selector or extra semantic kind (B HIGH) |
| 3 Present proportionality | 5 | 4 | 1 | 3 | 5 | B reuses local mechanisms; D profiling costs more; C universal rematerialization unjustified (B MEDIUM) |
| 4 Grow-as-needed | 3 | 5 | 2 | 5 | 2 | B extends proven classes incrementally and preserves fallback; D can be added separately (B MEDIUM) |
| 5 Future resilience | 4 | 5 | 3 | 5 | 4 | B keeps normal physical path and portable semantics, defers global compiler structures (B MEDIUM) |
| 6 Scalability | 4 | 4 | 3 | 4 | 3 | B has bounded expansion/fallback; universal rematerialization scales poorly until proved (all LOW–MEDIUM) |
| 7 Simplicity | 5 | 4 | 1 | 3 | 5 | B adds a single generic proof + lowering without a virtual-stack subsystem (B MEDIUM) |
| 8 Portability | 5 | 4 | 2 | 3 | 5 | B uses Bytecode DSL internals without changing guest contract; C/D bind more to JIT mechanics (B MEDIUM) |
| 9 Runtime/resource | 2 | 4 | 5 | 4 | 2 | B might eliminate physical setup; C has highest *theoretical* potential; no current graph proof (LOW) |
| 10 Failure/operability | 5 | 4 | 2 | 3 | 5 | Explicit B fallback isolates failed proofs; C/D require larger deopt management (B MEDIUM) |
| 11 Defer/migration | 3 | 5 | 2 | 4 | 2 | B incremental site expansion preserves compatibility; indefinite E would preserve structural cost (MEDIUM) |
| 12 Evidence maturity | 5 | 4 | 2 | 3 | 5 | SOMns + existing Protos B′ supports B; C universal and D profiles not proven for D179 (B MEDIUM) |

**Qualitative gate overrides numeric totals:** C fails present-necessity/overengineering absent universal-frame evidence; E risks indefinite acceptance of physical overhead if the compiler graph justifies extension. A remains valid fallback and wins if no case can prove a benefit. D's runtime profile machinery may be separately approved later; B's incremental path is not contingent upon adopting D.

**Incremental cost accounting:** If B is not implemented now, no public compatibility break is forced, but future work must still refactor per-selector B′ predicates or add a generic admission contract to avoid repeated `ifTrue`-like specializations. Building C now instead incurs a new cross-activation lifetime registry/OSR/deopt representation plus testing on every Context/Task/Actor path before there is evidence it is needed.

**Portability/scaling/future stress:** Many Contexts/Actors and simultaneous threads must not share a mutable global activation registry. Deep recursion and large dynamic megamorphic sites must choose physical fallback/budget, not unbounded expansion. Reentrant invocation requires fresh per-call local storage; no retained ephemeral frame for escaping captures. Suspension/resumption/cancellation and `ensure` must not replay completed side effects. Alternate Truffle versions and Bytecode local index reuse must not affect guest semantics. A future full virtual debugger frame layer would require an independent owner-authorized PLAT amendment; the escape route is to keep separate physical roots where tooling requires them.

## 6. Falsification and mandatory implementation acceptance

| Example / hazard | Incorrect optimization | Required invariant or fallback |
| --- | --- | --- |
| Fresh `context` observation | Reuses physical caller identity for semantic child | Fresh child Context identity, stable on repeated observation |
| Two escaping closures from one outer invocation | Captures two value copies or unrelated contexts | Same reference-captured context/binding identity |
| `removeSlot` on inner `x` | Frozen direct slot read still returns deleted 2 | After removal observe outer 1, after recreation 3 |
| RHS mutates selected binding | Re-resolve assignment target after RHS | Retain chosen target from before RHS or signal precise error |
| Same spelling `x` in caller/callee | Overwrites caller local when scope flattened | Independent scope-qualified physical storage |
| `o.m` method extraction | Old closure identity reused | Fresh extracted Closure per ordinary lookup; immediate send can avoid wrapper only if observationally exact |
| Shadowed `call` | Bypasses overridden Closure call slot | Ordinary selection under exact invalidation |
| Supplied/default/rest, failures | Changes evaluation/error order | Exact left-to-right and pre-activation error precedence |
| Nonlocal return + `ensure` | Escapes wrong home or cleans twice | Exact return home and once-on-semantic-exit cleanup |
| Recursive/reentrant Closure | Uses same block local for two live activations | Distinct per invocation or physical fallback |
| Async suspension/continuation | Loses inlined frame state or replays effects | Preserve progress and ownership; fallback when unproven |
| NORMAL / DEBUG tags and RootTag | Fabricates frames or lies about scope | Preserve PLAT044 constrained exception and current debugger mode |

**Smallest useful first implementation:** unified proof/admission + reusable inline lowering for one or more **ordinary** direct Closure C0/C1 shapes with own locals and a captured-local discriminator; exact fallback for others. Tests should verify emitted topology *and* semantic result without asserting hardcoded benchmark constants. Add breadth (N arguments, dynamic methods, recursion, profile speculation) only after actual source/graph grounds.

## 7. Governance and decision provenance

The project owner explicitly selected **B** in the active chat on 2026-10-13 following the second source comparison and the corrected preference **B first; D only a potential later evolution**. Earlier investigatory preference for D is historical, not selected. Ratification preserves D013, D179, PLAT035/036/040/044 and does not change guest semantics, debugger frame cardinality outside approved narrow cases, or introduce a new Closure kind.

```text
FORMAL_IDENTIFIER=PLAT057
SELECTED_CANDIDATE=B
OWNER_APPROVAL=EXPLICIT_2026-10-13
DECISION_INVARIANT_CONSISTENCY=PASS
SOURCE_CHECKPOINT=57a43b6d94697f9a6256e36c7a775dfa4a1218d7
EVIDENCE_TYPE=READ_ONLY_SOURCE_AND_RETAINED_HISTORICAL_GRAPH
POST_I095_MATCHED_STABLE_GRAPH=NOT_AVAILABLE
PRODUCT_CHANGES=NONE
TESTS_OR_BENCHMARKS_EXECUTED_BY_AGENT=NONE
PRODUCT_IMPLEMENTATION_ACCEPTANCE=NOT_CLAIMED
```
