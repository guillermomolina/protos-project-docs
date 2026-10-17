# PLAT062 — Candidate T approved: terminal Closure invocation and delegated completion

**Date:** 2026-10-17. **Decision owner:** [PLAT062/#907](https://github.com/guillermomolina/protos/issues/907). **Product implementation:** [I112/#920](https://github.com/guillermomolina/protos/issues/920), independent implementation issue. **Source checkpoint:** [protos@62ca4eb602432f3457c080954b716f8466b76983](https://github.com/guillermomolina/protos/commit/62ca4eb602432f3457c080954b716f8466b76983) (I108, 0.3.373-SNAPSHOT). **Decision:** explicitly **owner-approved Candidate T** in conversation following a detailed candidate presentation: “Apruebo candidato, el git diff check esta limpio. Todos los tests han pasado en local.” The clean diff/test statement describes the owner's existing checkout only; **no I112 build, execution, graph or emitted instruction result is claimed**.

## Context and corrected metric

Protos diagnostic reported **420 FULL / 176 EXECUTABLE** top-level source-root instructions for `identity: (value) => { value }\nidentity(1)` on the I108 source with experimental local PERF042 changes; the former 434 and owner-recalled ~460 are **distinct historical measurements**, not same-HEAD comparisons. Handwritten Builder 38 FULL / 15 executable is not semantically equivalent to entire emitted program. GraalPy observed **17 FULL between module (12) and identity (5)**, including returns/prolog/epilog, with no yield/resume, tag enter/leave or branch in that configuration. Its absence of instrumented tags in this capture does not imply absence of tagging capability.

**New hard acceptance:** <=17 **FULL instructions across both Protos source roots** for the same trivial source, ideally 15–16, measured as the total lengths of *unfiltered* `BytecodeNode.getInstructionsAsList()`; count all actually materialized tags, returns, epilogs, branches, and helpers. Exactly two source roots; separately census every helper, continuation and synthetic root, and reject new mandatory root / hidden generic fallback relocation. NORMAL is the principal apples-to-apples baseline, DEBUG must be independently tested and accounted for, with instrumentation config specified. This candidate's draft sequence is **not verified generated output**.

## Candidate T — approved architectural change

1. Treat **true tail/terminal position** as a runtime completion boundary rather than a requirement for caller Bytecode continuation. A source root with no further guest work emits one arity-specialized `InvokeClosureOneTail` semantic operation (parallel 0/N as needed) that receives already-evaluated receiver/arguments in D013 order and acquires the exact caller authority from its frame, only when necessary. It owns complete source/native/structured dispatch, direct and generic selection, cache invalidation and B5 ABI construction, including overridden `Object.call`.
2. Ordinary synchronous path returns its result directly, without allocating any suspension/completion carrier. If a dynamic target *actually* suspends, produce/delegate the live continuation and **the exact outstanding invocation completion/unwind owner**, avoiding reentry into a terminated terminal caller. Existing Task/C-prime/host-entry drivers must resume the original child state without replay; finish or unwind all owners **once**, in nesting order, after the final outcome. The new representation/bridge must not introduce a new evaluator, scheduler, host-stack suspension or mandatory helper root.
3. A nonterminal invocation or control context that must resume instructions in caller keeps proper physical `Yield`/continuation/BCI, and a true structured control region keeps required control ownership. Terminal path's lack of `Yield` must not mean inability to suspend, nor license to assume static non-suspension merely from an identity body.
4. Existing PLAT062-B I108 dispatch primitives are source-backed foundations, not the target architecture: I108's generic branch currently returns `PreparedClosureCall`, after which `emitComposedCall` expands Boolean/structured/ordinary prepared entry, nested continuation, cleanup. Candidate T must **subsume full fallback and end-to-end completion**, not rename this protocol or hide it in an unconditional Java helper.
5. Retain fresh Closure object identity, lexical cell capture/reference, D179 presence/invalidation, D203 defaults/flexible arity/spread, D013 post-argument mutable call selection, methodHome/this/super, cross-Context, Error/handlers, `ReturnHome` nonlocal return and exactly-once cleanup/cancellation, ordered source spans, NORMAL/DEBUG, uncached and reparse. Universal PLAT060 five-header B5 ABI remains **unchanged**.

### Explicit amendment/reopening ledger (GITHUB021)

| Existing decision/invariant | Approved Candidate T delta | Preserved invariants |
|---|---|---|
| PLAT014 / PLAT021: nested C-prime continuation composed through source-root Yield | **Amend** terminal boundary: a final call may delegate continuation + deferred completion owner to C-prime driver, so terminal source root need not yield; nonterminal continuations unchanged | No replay, cancellation/Error/ReturnHome ordering, true Bytecode-owned suspension |
| PLAT042 / PLAT043: physically separate Boolean/structured generic dispatch, source root Boolean | **Amend** terminal source call: dispatch operation owns whole selected fallback, not an inlined subtree for each possible kind; other structured roots retained when really required | Selected-callback semantics and structured-control obligations |
| PLAT044 / PLAT057: guarded inline literal callbacks and fallback | **Amend** emission shape: do not emit whole duplicate fallback under every inline candidate at the terminal site | Admission guards and observable lexical semantics |
| PLAT058 / I101: NORMAL/DEBUG tag/source topology | **Amend** physical implementation to permit equivalent metadata/Truffle instrumentation instead of mandatory emitted `tag.enter/leave` in the reference config | Truthful source sections, step/debug semantics and tag-tree/source observability |
| PLAT061 / D204 / I107: effective root and call-site non-suspension facts | **Extend** facts with **terminal-position eligibility** as a separate property from non-suspension. UNKNOWN still can suspend and must retain real continuation transport | No false body -> invocation proof |
| PLAT062-B: semantically complete invocation operations, external structurally required yields | **Supersede** external-yield assumption **for tail calls only** with Candidate T delegated completion; direct/generic owner semantics retained | Post-argument dynamic selection, PIC, fallback correctness |
| PLAT060-B5, D013, D179, D203 | No new ABI and no guest-semantic change approved | Entire prior normative contract |

This is an **explicit project-owner candidate approval**, not blanket permission to alter guest language semantics or every associated PLAT. A newly found choice outside the presented terminal-delegation scope needs another decision; a known PLAT assumption contradicted exactly by terminal delegation is in the approved amendment scope.

### Draft 15-operation *budget*, **not verified actual instruction list**

| Module source root — target order | Count |
|---|---:|
| `EnterRoot` (prolog, if represented as listed instruction) | 1 |
| Load canonical Closure definition and reusable plan constants | 2 |
| `MaterializeCurrentClosure` (fresh guest Closure) | 1 |
| `CreateCurrentFrameLocal(identity)` | 1 |
| `ReadRootFrameLocal(identity)` | 1 |
| Load literal `1` | 1 |
| `InvokeClosureOneTail` | 1 |
| `EpilogForReturn`, `return` | 2 |
| **Module subtotal (provisional)** | **10** |
| Identity: `EnterRoot`, `BindUnaryFrameParameter`, `ReadRootFrameLocal(value)`, `EpilogForReturn`, `return` | **5** |
| **Draft sum, provisional (with headroom up to 17)** | **15** |

**Important:** These are target semantic steps, not a bytecode dump. An actual `@Prolog`/`@EpilogReturn`, operand constants, invocation binding or inline Java helper may compile to differing instruction counts; require exact unfiltered generated-node signature as the arbiter. The target is not met merely by forcing 15 names into a mock Builder.

### Existing code seams (I108 HEAD)

- `CanonicalToBytecodeLowerer.emitComposedCall`, lines ~9764–10168: current `closureInvocationOutcome`, `IsPreparedClosureInvocation` and unconditional `emitPreparedInvocation`; target true terminal emission path. `emitRootBody` ~999–1430, `emitContinuationComposition` ~6947. `emitPreparedInvocation` ~6057; nested dispatch ~6521. `emitExpression` ~7200; `emitCreateCurrentBinding` ~3442.
- `ProtosSemanticBytecodeRootNode.InvokeClosureOne` ~7997: direct guarded and definition-keyed specializations, generic fallback currently calls `prepareClosureCallOneFallback`; `EnterClosureCall`, `ResumeContinuation`, `FinishClosureCall`, `IsContinuationForCall` ~8220–8503. `ProtosBytecodeRootNode.PreparedClosureCall` ~2365 and `ReturnHomeOwningCall` ~2600: existing lifecycle owner.
- `ProtosBytecodeTaskExecution.drivePreparedOutcome` and `resumePreparedPublished`; `ProtosHostEntrySuspension.drive` currently iterate nested `ContinuationResult`; must preserve exact completion/unwind when delegated. `ProtosClosureInvoker.invokePreparedInExtent` has a separate synchronous boundary: do not accidentally relax Task/native suspension requirements.
- Existing tests `ProtosI101CIndependentBuilderReferenceTest`, `ProtosI107ContinuationEmissionTest`, `ProtosPerf042DefinitionKeyedDirectCallTest` provide full-signature and dynamic-fallback oracles, not proof of <=17; independent strict census and negative adversarial cases required.

### Source-backed prior art and distinction

- [GraalPy Bytecode DSL source](https://github.com/oracle/graalpython/blob/graal-25.3.4.1/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java): `MakeFunction` ~1746, `CopyArguments` ~1593, `@Prolog` ~531, `@EpilogReturn` ~581; its `@Yield` generator paths ~1415/~1477 do **not** imply that ordinary `CallUnaryMethod` needs inlined caller yield.
- [GraalPy unary dispatcher](https://github.com/oracle/graalpython/blob/6bd9d25023bf713ae3dd7e83ee620c897a2c51d0/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/call/special/CallUnaryMethodNode.java), direct with caches and generic fallback; [GraalPy call dispatchers](https://github.com/oracle/graalpython/blob/graal-25.3.4.1/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/call/CallDispatchers.java).
- [GraalJS JSFunctionCallNode](https://github.com/oracle/graaljs/blob/4327aa1f2914bac7f810406423d70ae7f8a1888b/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/function/JSFunctionCallNode.java); [TruffleRuby DispatchNode](https://github.com/truffleruby/truffleruby/blob/b60b0d04dd11454012d0e8a6588514410ffacdcb/src/main/java/org/truffleruby/language/dispatch/DispatchNode.java); [TruffleSqueak DispatchSelector1Node](https://github.com/hpi-swa/trufflesqueak/blob/0e4f14dc935b3a5fee50780c7ce7dabbdd600981/src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/dispatch/DispatchSelector1Node.java); [Apple Pkl ApplyVmFunction1Node](https://github.com/apple/pkl/blob/ef5e34c21009fe3d200b647551dbd672d288e710/pkl-core/src/main/java/org/pkl/core/ast/lambda/ApplyVmFunction1Node.java): direct/cached vs indirect fallback; not interchangeable with Protos's C-prime semantics.
- Truffle Bytecode DSL `@Yield` / `@Operation` / instrumentation API: [Truffle Bytecode DSL User Guide](https://github.com/oracle/graal/blob/master/truffle/docs/bytecode_dsl/UserGuide.md). Do not claim a synchronous Java node can suspend its own Java stack.

### Verification and handoff

[I112/#920](https://github.com/guillermomolina/protos/issues/920) owns one cohesive implementation group. Human executor exclusively runs `make test`, `make check`, `git diff --check`, and product Git publication; its report of existing passing tests is **baseline**, not proof of new code. Source-root complete instruction signatures, exact tags/config, root census, generic-first/direct/generic, overridden-suspendible `Object.call`, Error/NLR/cancellation/finally, cross-Context, DEBUG and native are mandatory. No automatic claim of speedup or Native Image improvement.

**Status:** approved architectural selection and bounded amendment **published as evidence**; acceptance target and physical grouping verification **OPEN** until I112 code and measurement. Historical PLAT062-B decision and I108 completion must remain correctly attributable to their dates.
