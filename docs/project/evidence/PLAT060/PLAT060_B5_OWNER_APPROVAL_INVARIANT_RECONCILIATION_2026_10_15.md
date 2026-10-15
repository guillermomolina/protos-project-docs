# PLAT060 — B5 comparative source audit, explicit owner approval and GITHUB021 reconciliation

**Evidence date:** 2026-10-15. **Product baseline:** `PROTOS_REVISION=df4142919d0222635ab8f9f8c54e7a1966e6edaa` (read-only, inspected; no product file mutated). **Project-record source baseline before publication:** `PROJECT_RECORD_BASE_REVISION=932e7b2042e58b3fd2121acfc3af2d310e5e23b0`. Ratified decision: [PLAT060 B5](../../decisions/platform/PLAT060_B5_UNIVERSAL_ORDINARY_CLOSURE_CALL_ABI.md). Live authority: [PLAT060/#895](https://github.com/guillermomolina/protos/issues/895).

**Approval provenance:** User/project owner explicitly stated (1) “no quiero abi diferentes para NORMAL/DEBUG”; (2) required B5-vs-C reevaluation with this as a hard premise; (3) “ok apruebo B5” and requested Issue/doc publication. This **approves** a common NORMAL/DEBUG B5 *ordinary Closure-call frame argument ABI*; it **does not approve** independent B6, compressed C, exceptional headerless ABI or eager activation. Architecture ratification ≠ implementation or performance acceptance.

## Physical-domain separation

The `VirtualFrame` and its `FrameDescriptor` represent Truffle execution/local slots. Its `getArguments()` is the `Object[]` *physical call ABI*. `ProtosActivation` is a lazily materialized exact guest invocation/Context identity. `BytecodeLocal` and captured lexical environment represent frame-native/lexical state. They are **not interchangeable**. A physically inline callback may have no independent callee frame at all and is not a competing call ABI.

## Exact implementation inventory at product baseline

[ProtosFrameArguments.java](https://github.com/guillermomolina/protos/blob/df4142919d0222635ab8f9f8c54e7a1966e6edaa/src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java) lines 33–87, 94–363, 364–496, 497–998:
- **Full 5-slot ordinary direct:** `[closure, DIRECT_CLOSURE_CALL, task|null, caller, returnHome, supplied...]`.
- **Full 5-slot immediate method:** `[closure, receiver, methodHome, caller, returnHome, supplied...]`.
- **Short direct Call0 MINIMAL_A/B/C/D:** `[closure,caller]`, `[closure,caller,returnHome]`, `[closure,task,caller]`, `[closure,task,caller,returnHome]`; discriminated by length and runtime types (`minimalLayout`, lines ~894–937).
- **Short immediate method Call0:** `[closure,receiver,methodHome,caller]` when home is unobservable, otherwise the five-slot version (`isMinimalImmediateMethodCall`, lines ~955–964).
- **Rich activation-only:** `[activation]` used in [ProtosBytecodeRootNode.java](https://github.com/guillermomolina/protos/blob/df4142919d0222635ab8f9f8c54e7a1966e6edaa/src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java) ~2765 and ~3535 for ordinary and special module ownership respectively; these need **separate classification** as ordinary Closure versus non-Closure initialization.
- **NORMAL-only host empty `Object[0]`:** [ProtosHostExecutableClosure.java](https://github.com/guillermomolina/protos/blob/df4142919d0222635ab8f9f8c54e7a1966e6edaa/src/main/java/com/guillermomolina/protos/execution/ProtosHostExecutableClosure.java) ~80–118, ~396–417; only on proven no-argument/unobservable-home routes.
- **In-place activation publication:** slot 0 `ProtosClosureValue` → exact `ProtosActivation`, with remaining positions and array length intact; slow materialization is `@TruffleBoundary`; exposed across [ProtosSemanticBytecodeRootNode.java](https://github.com/guillermomolina/protos/blob/df4142919d0222635ab8f9f8c54e7a1966e6edaa/src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java) operations, [ProtosBytecodeTagTreeNodeExports.java](https://github.com/guillermomolina/protos/blob/df4142919d0222635ab8f9f8c54e7a1966e6edaa/src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeTagTreeNodeExports.java) and the caller/interop pathways.
- **Caller fast paths:** `compactHasSuppliedArgument`, `compactSuppliedArgumentWithinCount`, `compactTask`, `unmaterializedInheritedProvenanceCaller`, `CurrentCallerReference` have layout-sensitive assumptions; do not merely edit producers.

This is a **shape census**, not a measured machine-graph branch count. Exact call-site counts, native allocations and Tier-2 residuals were **not measured** in the ratification. Consumers in `ProtosBytecodeRootNode`, `ProtosSemanticBytecodeRootNode`, `CanonicalToBytecodeLowerer`, debugger and native interop require a full current-HEAD migration search before patching.

## Prior-art source snapshot and reasoning

| System | Read-only source inspected, revision reference | Finding |
|---|---|---|
| GraalJS | [JSArguments.java @4327aa1](https://github.com/oracle/graaljs/blob/4327aa1f2914bac7f810406423d70ae7f8a1888b/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/runtime/JSArguments.java), [JSFunctionCallNode.java](https://github.com/oracle/graaljs/blob/4327aa1f2914bac7f810406423d70ae7f8a1888b/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/function/JSFunctionCallNode.java) | Fixed ordinary 2-slot `this` + function header; Call0/1/N specializations use the same ordinary header. New-target/resume are special semantic conventions, not evidence for variable ordinary Call0 headers. |
| GraalPy | [PArguments.java @a099d006](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/function/PArguments.java), [CreateArgumentsNode.java](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/argument/CreateArgumentsNode.java), [ExecutionContext.java](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/ExecutionContext.java) | Fixed 6-slot implicit header; callee/caller frame and exception-state transfer is demand/flag-based. Generator continuation is explicitly outside ordinary PArguments; frame reification avoided where not needed. |
| TruffleRuby | [RubyArguments.java @b60b0d0](https://github.com/truffleruby/truffleruby/blob/b60b0d04dd11454012d0e8a6588514410ffacdcb/src/main/java/org/truffleruby/language/arguments/RubyArguments.java) | 8-slot fixed ordinary header; optional fields; `repackForCall` deliberately clones a call array at some polymorphic sites to *improve separate escape analysis*. Reducing allocation count pre-PE can worsen final result. |
| FastR | [RArguments.java @f9d9199](https://github.com/oracle/fastr/blob/f9d9199e51b371f076b19456933ee395d28360bf/com.oracle.truffle.r.runtime/src/com/oracle/truffle/r/runtime/RArguments.java) | 8 fixed header slots; lazily created environment; explicitly irregular/internal frame forms not to be misclassified as ordinary function-call ABI. |
| Apple Pkl | [VmFunction.java @d062312](https://github.com/apple/pkl/blob/d062312df9c294e6abf7dad79cea9518fc9f22c9/pkl-core/src/main/java/org/pkl/core/runtime/VmFunction.java), [ApplyVmFunction1Node.java](https://github.com/apple/pkl/blob/d062312df9c294e6abf7dad79cea9518fc9f22c9/pkl-core/src/main/java/org/pkl/core/ast/lambda/ApplyVmFunction1Node.java) | DirectCallNode path passes `this`, function, arg at known positions; arity-specialized callers do not imply independent ordinary ABI variants. |
| TruffleSqueak | [ContextObject.java @0e4f14d](https://github.com/hpi-swa/trufflesqueak/blob/0e4f14dc935b3a5fee50780c7ce7dabbdd600981/src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/model/ContextObject.java) | Reifiable contexts and frame materialization demonstrate stronger identity/materialization stress; not proof that rich context must be eagerly allocated. |

Also surveyed relevant Espresso/Sulong/LLVM, SimpleLanguage, V8/HotSpot/Graal and Truffle PE as prior context; they provide optimization/continuation boundaries, **not a direct proof of which of Protos B5 or C compiles faster**. [Graal Partial Evaluation](https://github.com/oracle/graal/blob/master/truffle/docs/PartialEvaluation.md) and [Optimizing](https://github.com/oracle/graal/blob/master/truffle/docs/Optimizing.md) explain that monomorphic inlining and escape analysis are conditional; no guarantee that a 5-slot `Object[]` always scalar-replaces.

## Normative and owner-approved invariants (GITHUB021)

| Invariant and authority | Owner-approved B5 delta | Result |
|---|---|---|
| **PLAT060 owner:** one ABI for NORMAL and DEBUG; no physical profile split; explicit exact B5 approval | B5 same indices, header length and positional count for both | **PRESERVED** |
| **PLAT058-D:** one canonical/lowerer, Context-immutable profile-specific executable emission; DEBUG observability and late attach in DEBUG; independent handwritten Builder oracles; deterministic reparse | Header shared, emission different; no late attach in NORMAL | **PRESERVED** |
| **PLAT040 F′:** guarded ordinary hot call and conditional guest activation | Retain PIC/guard/selected target/lazy activation | **PRESERVED**, historical variable Call0 variants not owner-mandated |
| **PLAT036 + D179:** single frame-backed lexical binding state, lexical capture by reference, open-context `removeSlot` absent/recreate | No duplicate values, no eager semantic context or lexical snapshot | **PRESERVED** |
| **PLAT046:** optional Task/Actor/Process, no host RootTask tax | Nullable explicit Task slot, no Task allocation | **PRESERVED** |
| **PLAT044/057:** proof-driven in-root callback inlining; semantic scope/invocation identities | Physical inline region stays allowed if no separate CallTarget | **PRESERVED** |
| **PERF032 host NORMAL headerless:** empty `Object[0]` on selected source-root entry; DEBUG two slots; tests assert shape and `getScope` rejection by length | Both become full B5; NORMAL tooling rejection must be mode/tooling-based | **EXPLICITLY SUPERSEDED** by owner's approved PLAT060 common-ABI rule |
| **CALLABLES.md / EXECUTION_AND_CONTROL.md / OBJECT_MODEL.md / ERRORS.md** | Fresh invocation Context when observed; `this`, `methodHome`, extracted Closure identity, dynamic Object.call and error ordering, return and control | **PRESERVED; no spec delta** |
| **FUTURES_AND_TASKS.md / ACTORS.md** | Task ownership, cancellation, control continuations, cross-Actor authority | **PRESERVED; no spec delta** |

**Special care: `returnHome`.** [ProtosClosureValue.java lines 250–269](https://github.com/guillermomolina/protos/blob/df4142919d0222635ab8f9f8c54e7a1966e6edaa/src/main/java/com/guillermomolina/protos/runtime/ProtosClosureValue.java#L250-L269) may return a captured home, an unobservable marker or **allocate a new home per invocation**. Eliminating field 4 and recomputing it later is **not semantically safe**. Direct invocation's `Task` may differ from inherited caller Task. Immediate method's `receiver` and `methodHome` come from exact selected dispatch, not necessarily from original Closure captures.

**Tooling scope caveat:** [ProtosPerf032ExecutionModeTest.java](https://github.com/guillermomolina/protos/blob/df4142919d0222635ab8f9f8c54e7a1966e6edaa/src/test/java/com/guillermomolina/protos/execution/ProtosPerf032ExecutionModeTest.java) asserted NORMAL `Object[0]` and DEBUG `[closure,caller]`. [ProtosBytecodeTagTreeNodeExports.java](https://github.com/guillermomolina/protos/blob/df4142919d0222635ab8f9f8c54e7a1966e6edaa/src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeTagTreeNodeExports.java) decides headerless policy by `frame.getArguments().length == 0`. This is a physical-shape test used as semantic-profile policy and must be removed without promoting NORMAL debugger requirements. The genuinely required DEBUG scope/stepping/root-tag/late-attach behavior must remain.

## Candidate comparison and falsifiers

**A current variable layouts:** rejected by owner's one-ABI simplification. **B5:** approved, five fixed fields derived from existing full ABI, with a private marker/union in positions 1 and 2. **B6:** six-slot fully disjoint receiver/methodHome/task, technically compatible but not approved; adds a slot and more rewriting without measured necessity. **C:** 3/4-slot new aggregate/indirection/recalculation, rejected until owner explicitly reopens decision; risks ReturnHome identity. **D/E profile-distinct/hybrid/headerless:** rejected by explicit owner prohibition. **F defer:** no longer an architecture candidate after owner decision.

Scoring 1–5, `H` high, `M` medium, `L` low confidence. Each cell gives its reason and confidence; no total score substitutes for a red-flag gate.

| GITHUB010 axis | B5 approved | B6 rejected/not approved | C rejected/not approved |
|---|---|---|---|
| 1 correctness | 5/M known complete header | 5/M independent fields | 3/L extra indirections and identity |
| 2 Protos philosophy | 5/H simplify without institutions | 5/M clearer independent slots | 2/M packing magic |
| 3 present proportionality | 5/H reuse existing representation | 4/M extra ubiquitous slot | 2/M speculative aggregate |
| 4 incremental growth | 5/M same ABI can evolve internally | 5/M independent field slots | 3/L entangled metadata |
| 5 future resilience | 4/M shared tagged union | 5/M explicit task field | 3/L indirect hidden constraints |
| 6 scalability | 4/M no new shared state | 4/M same | 3/L unknown aggregate costs |
| 7 conceptual simplicity | 4/H fixed offsets, marker union | 5/H explicit named fields | 2/H more decoding |
| 8 portability | 5/H plain Truffle arguments | 5/H same | 3/M extra optimizer dependence |
| 9 resource cost | 4/L array effects not yet measured | 3/L plus one slot | 2/L allocations unproven |
| 10 failure/operability | 4/M fewer length/type variants | 5/M easiest debugging | 2/L implicit metadata |
| 11 deferral/migration | 5/H reuses full current header | 3/M wider rewrite | 2/M redesign |
| 12 evidence/risk | 4/M current full path & prior art | 3/M plausible but no local A/B | 2/L lacks causal evidence |

**Strongest objection to B5:** currently eligible NORMAL host Call0 has zero arguments; some direct Call0 has two. Making every call B5 may leave 3–5 more references in an un-inlined or escaping `Object[]` and could worsen interpreter, host, polymorphic and compiled paths. Graal may eliminate these fields after inline/escape analysis, but this is *conditional*. A causal identical-workload A/B could falsify any claim that B5 is faster; it would not unilaterally authorize restoration of split ordinary ABIs.

**Historical evidence limitation:** [primitive-closure-call unit.json](https://github.com/guillermomolina/protos-benchmarks/blob/main/results/graphs-full-0db24f00ff2d/primitive-closure-call/protos/unit.json) reports `GRAPH_NOT_STABLE`, `evidence_valid=false`, dirty harness, Protos `0db24f00ff2d92d642351d7f7535517fe01a55ce`; NOT a causal B5/old matched result. No new benchmark was run or speedup claimed.

**Future scenarios/escape routes:** many Tasks/Actors/Processes must not allocate optional capabilities; multiple Contexts retain their separate immutable profile and Context-owned projections; recursive/multicore and mixed PIC polymorphism may degrade PE due to array lifetime; cancellation/NLR/yield must not change invocation home or activation identity; Native Image/native guest JIT caveats remain independently governed by PLAT038/045; Bytecode DSL reparse keeps exact root groups; a future non-Truffle backend can map B5 semantics to a different physical representation, since B5 is internal runtime ABI, not Protos language ABI. The plausible regret is extra header cost in un-inlined calls; escape path: specialize construction, optional provenance transfer and compiled field-elimination under the same five offsets, or present a **new explicit owner decision** if truly impossible. No silent B6 or C fallback.

**Pay-as-you-grow adversarial answer:** smallest current solution is B5, reusing an already-working full ABI. Additional fields/packaging/debugger state beyond that are not justified today. Deferring a hypothetical B6 only requires changing fixed internal offsets/producers/readers if future evidence warrants it, not a guest-spec break. Building B6/C now forces extra global work and testing without measured necessity. Deferring ordinary ABI unification instead would keep multiple high-interaction discriminators and scope-policy coupling, already observed in the current source.

## Mechanism retirement plan and implementation acceptance

| Mechanism | Audit classification |
|---|---|
| Existing five-slot full ordinary header | KEEP and unify |
| `MINIMAL_A/B/C/D` and short-method length/type discrimination | REMOVE PERMANENTLY |
| NORMAL-only host `Object[0]` entry | REMOVE PERMANENTLY as ordinary ABI |
| Ordinary `[activation]` rich-only entry | REMOVE PERMANENTLY as separate ABI; retain published state |
| Slot-zero lazy publication, exact semantic identity and Boundary slow path | KEEP |
| Direct argument reads, Call0/1/N no intermediate vector, fused PIC | KEEP |
| Conditional return-home, Task/provenance and primitive-result markers | KEEP |
| NORMAL/DEBUG separate *lowering*, independent Builder oracle | KEEP |
| Inline callbacks with no independent CallTarget frame | KEEP |
| Internal non-Closure root and continuation conventions | KEEP only under distinct existing semantic contracts; explicitly audit |

**Acceptance witness matrix:** NORMAL and DEBUG Call0/1/N, nested Closure, captured by-reference locals, `context` escape/removal/recreation (D179), direct/overridden `Object.call` and invalidation, method send/extraction/receiver inheritance, optional/default/rest and arity/side-effect order, exact `returnHome` and NLR/ensure, Task/cancellation/Future/yield/resume, Actor/Process/host and guest interop, multi-Context, DEBUG session attach after preparation, NodeLibrary scopes, normal tooling rejection, no mandatory RootTask/activation, and `ensureComplete`/reparse structural oracles. Construct handwritten `ProtosSemanticBytecodeRootNodeGen.Builder` oracles independently for physical emission claims as ratified PLAT058-D; separate body minimality from call ABI. Bench A/B should retain exact product/harness SHAs, same workloads/tier/inlining admission, post-PE/post-escape/final graphs, allocations, timing and honest `NOT_STABLE` STOP. Owner/human executes all tests/builds/benchmarks/product publication. No automatic benchmark-policy changes.

**Issue transaction boundary:** PLAT060 ratification is complete when canonical decision and evidence are published and reread; the new I-family implementation issue remains separate and OPEN until all product acceptance. No downstream success is claimed by decision closure.

## Files materially inspected and binding sources

- Root `AGENTS.md`; `AGENTS.work/DESIGN.md`, `REFERENCE.md`, `AUDIT.md`, `COORDINATION.md` and implementation handoff constraints.
- Product `ProtosFrameArguments.java`, `ProtosBytecodeRootNode.java`, `ProtosSemanticBytecodeRootNode.java`, `ProtosHostExecutableClosure.java`, `ProtosBytecodeTagTreeNodeExports.java`, `ProtosClosureValue.java`, `ProtosPerf032ExecutionModeTest.java`, existing PERF025/032/038 traces and tests, and canonical lowerer profile decisions.
- Normative `spec/semantics/CALLABLES.md`, `EXECUTION_AND_CONTROL.md`, `OBJECT_MODEL.md`, `ERRORS.md`, `spec/concurrency/FUTURES_AND_TASKS.md`, `ACTORS.md`; owner-ratified PLAT036/040/044/046/057/058 and historical PERF032 evidence.
- Listed upstream GitHub source files with read-only revision pins. This record reports source observations, not executed benchmark/test results.
