# PLAT062 — Specialized Closure creation and invocation operation boundary

**Status:** RATIFIED — Candidate B, 2026-10-16; product implementation remains pending.
**Owner:** [PLAT062/#907](https://github.com/guillermomolina/protos/issues/907).
**Exact owner decision:** “apruebo B” after the agent explicitly proposed **B, not D**: concentrate complete ordinary Closure dispatch and generic fallback in specialized Truffle DSL/Bytecode DSL operations, preserving the current Protos execution/continuation boundaries only when Bytecode DSL structurally requires them. The owner also reported `git diff --check` clean and all local tests passed, referring to the **existing product baseline**, not tests of this future change.
**Read-only product checkpoint:** [guillermomolina/protos@c00b36c0edaf6678b7d8101de3a405124f179644](https://github.com/guillermomolina/protos/commit/c00b36c0edaf6678b7d8101de3a405124f179644) (0.3.370-SNAPSHOT).
**Research/evidence:** [PLAT062-B prior-art, falsification and invariant record](../../evidence/PLAT062/PLAT062_B_ARCHITECTURE_AND_OWNER_APPROVAL_2026_10_16.md).

## Ratified selection and exact boundary

**B — Truffle-style semantic-operation ownership for Closure creation/invocation.** The canonical single lowerer emits evaluated callable/receiver/argument operands and an arity-appropriate semantic invocation operation; that operation owns *the complete semantically valid dynamic dispatch decision*: post-argument `Object.call` selection, proven/cached direct admission, guarded PIC, direct target/ABI B5 argument assembly, ordinary method/receiver/Context authority, and generic dispatch/fallback transitions. Do not leave a mechanically open-coded `TryDirectCall* → IsDirectSendMiss → Prepare* → Enter* → Finish* → Complete*` pipeline in the common proven non-suspending call path; do not add a second independent PIC or a new call engine as a veneer atop it.

Use existing `@Operation`, `@OperationProxy`, Truffle DSL `@Specialization`/guards/`@Cached` and existing `DirectCallNode`/indirect capability. Separate zero, one and multiple/variadic arities where real specialization warrants it. Preserve every legal transition (generic-first→direct, direct→generic→direct, polymorphic/megamorphic, guard invalidation, uncached, multi-Context) and the original guest-observable semantics; a terminal broad generic `replaces` shortcut is not acceptable.

**Creation is included in the architectural contract but NOT automatically a rewrite.** Existing `MaterializeClosure` already creates a fresh Closure instance while reusing immutable code/plan cells. Retain it when structurally correct; change only on a demonstrated source-level ownership gap. Shared definitions never imply shared guest Closure identity or copying lexical cells by value.

### Narrow structural exception — suspension/unwind

Truffle Bytecode DSL `Yield`, resumption, structured `TryFinally`, root/BCI state, ReturnHome and cleanup cannot simply be hidden in a synchronous Java method. Consequently **Candidate B expressly allows only structurally indispensable external Bytecode DSL control** for an invocation that can suspend/unwind. Selection, guards, PIC and generic dispatch mechanics must remain encapsulated to the maximum compatible with those constraints. A helper that merely returns `MISS` and leaves the identical whole mechanical fallback outside **does not satisfy B**.

Existing D204/I107 composable `CANONICAL_BODY`, `EFFECTIVE_ROOT`, `COMPLETE_INVOCATION` and `GUARDED_CALL_SITE` facts govern proven non-suspension. A body proof alone never proves full invocation: argument evaluation, mutable `Object.call`, parameter defaults, Context, ReturnHome and dynamic selection remain independent. The lowerer may omit structural fallback **only after complete applicable proof/guards**; UNKNOWN remains resumable. No second proof engine, hidden blocking thread, continuation kind, Java-stack recursion, or alternative guest ABI.

## Non-negotiable approved invariants (GITHUB021)

| Existing authority | Ratified B compatibility requirement |
|---|---|
| Normative `CALLABLES.md`, `EXECUTION_AND_CONTROL.md`, `OBJECT_MODEL.md`, `ERRORS.md`, D003/D013/D179/D202/D203 | Left-to-right single evaluation; `Object.call` lookup **after** argument effects; fresh identity, captured lexical Context by reference, dynamic slot remove/recreate/override, receiver/`this`/methodHome/`super`, defaults at invocation, flexible arity, Errors, cancellation and nonlocal ReturnHome. No language semantic change. |
| PLAT014/019/021/028/042/043/044 | Real nested C-prime suspension, `Yield`/resume, helper-root control authority, try/finally and native boundary remain valid. No transfer of resumable guest control to a nonresumable Java helper. |
| PLAT057 / I096 | Keep proof-driven inline Closure lexical expansion and exact guarded fallback; operation-based dynamic dispatch does not authorize deleting valid inlining. |
| PLAT058 / I101 | One canonical proof-driven lowerer and existing source-semantic Bytecode root; NORMAL/DEBUG distinct emission profiles, truthful tags, source sections, complete Builder-oracle and deterministic reparse. |
| PLAT059 | No second canonical optimizer, transformed canonical IR or global rewrite phase. |
| PLAT060-B5 | One universal five-header-slot ordinary Closure ABI and legal conditional activation publication in NORMAL/DEBUG; no minimal/second ABI. |
| PLAT061-B / D204-A / I106 / I107 | Reuse proven composable facts and emitted-root publication discipline; UNKNOWN never means non-suspending; physical continuation accounting remains truthful. |
| PERF040 / PERF046 / PLAT063 | Numeric operations and cross-family lowerer granularity separately owned; PLAT062 does not broaden to numeric or all lowering. |
| PERF038 / PERF042 | Performance/graph acceptance independently owned. No inferred speedup from fewer instructions or mere ratification. |

**Invariant reconciliation: PASS for the *approved contract*.** This is a prospective architecture, not a claim that every implementation technique is yet proven. An implementation conflicting with any authority above must STOP rather than silently weaken the owner-approved B to an unratified D.

## Prior art and selection

Across [GraalPy `CallUnaryMethodNode`](https://github.com/oracle/graalpython/blob/6bd9d25023bf713ae3dd7e83ee620c897a2c51d0/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/call/special/CallUnaryMethodNode.java), [GraalJS `JSFunctionCallNode`](https://github.com/oracle/graaljs/blob/4327aa1f2914bac7f810406423d70ae7f8a1888b/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/function/JSFunctionCallNode.java), [TruffleRuby `DispatchNode`](https://github.com/truffleruby/truffleruby/blob/b60b0d04dd11454012d0e8a6588514410ffacdcb/src/main/java/org/truffleruby/language/dispatch/DispatchNode.java), [TruffleSqueak `DispatchSelector1Node`](https://github.com/hpi-swa/trufflesqueak/blob/0e4f14dc935b3a5fee50780c7ce7dabbdd600981/src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/dispatch/DispatchSelector1Node.java), [Pkl `ApplyVmFunction1Node`](https://github.com/apple/pkl/blob/b97abe13bcb961d67002021a80674dd52a0e15ee/pkl-core/src/main/java/org/pkl/core/ast/lambda/ApplyVmFunction1Node.java) and [Espresso bytecode quickening](https://github.com/oracle/graal/blob/master/espresso/docs/how-espresso-works.md), the common mechanism is *encapsulated semantic dispatch with per-site guards/caches and generic fallback*, not necessarily a shared Bytecode DSL backend or fewer guest semantics. Thus this decision adopts a common **Truffle architecture pattern**, not Python function/default/call semantics.

Meaningful candidates: **A** keep current explicit emission; **B** semantic operation owns full dynamic dispatch (selected); **C** static proof/inlining only (already present in PLAT057/061, not an alternative to dynamic B); **D** split fast path/fallback hybrid (rejected as default because it can perpetuate the exact duplication); **E** defer (lowest migration cost but retains the known structural question). B is chosen for coherent semantic ownership and growth, not as a proven runtime optimization. Genuine structural `Yield` constraints are an exception to B, **not** prior approval of a different D architecture.

## 12-axis GITHUB010 assessment (qualitative, 1–5, H/M/L confidence)

Historical pre-approval scores assess uncertainty honestly; selection is constrained by hard invariants rather than score totals.

| Axis | A keep | B selected | C proof-only | D hybrid | E defer |
|---|---|---|---|---|---|
| Correctness/invariants | 5H established | 2L full resumability must be proven | 5H established fail-closed | 4L explicit control but untested | 5H unchanged |
| Protos alignment | 4H ordinary behavior | 3M coherent semantic operation, must avoid foreign guest assumptions | 5H bounded facts | 4M if minimal | 4H unchanged |
| Present proportionality | 3M repeated emission | 2L refactor cost before proven benefit | 5H existing proof consumers | 3L duplicate layers risk | 5H no change |
| Incremental growth | 3M coupled emission | 4M extensible arity operations | 4H fact refinements | 4M independent fast/control paths | 4M change later |
| Future resilience | 3M low-level exposure | 3L DSL coupling, semantic ABI retained | 4M portable facts | 4M escape retained | 4M avoid early lock-in |
| Scalability/Context | 3M existing bounded PIC | 4L possible per-site bounded guards | 3M dynamic unknown remains | 4L potential hybrid | 3M unchanged |
| Conceptual simplicity | 2H many emitted steps | 3L smaller caller, complex internal operation | 4M shared facts | 2M two owners | 5H no addition |
| Portability | 3M bytecode-specific | 2M proxy API coupling, semantic helpers reusable | 4M facts backend-neutral | 3M split ownership | 5H unchanged |
| Runtime/resources | 2L possible overhead | 3L no timed proof, cache allocation risk | 4L omission where proven | 4L hypothesis only | 2L current costs unknown |
| Failure/operability | 4M existing unwind | 2L preserve/debug precise continuation | 4M fail-closed | 3L cross-boundary audit | 4H unchanged |
| Deferral/migration | 3M later refactor | 2L significant current migration | 5H already adopted | 3M staged | 4M possible later migration |
| Evidence maturity | 5H working code | 2M source-backed precedent, no B product PoC | 4H I096/I107 | 2L no prototype | 5H no change |

**Non-compensating gates:** B's unresolved resumability/BCI/multi-Context feasibility is an actual proof obligation. An unsafe implementation is disqualified regardless of compact bytecode. Likewise a layered extra PIC or a "compressed listing" with worse PE/code-size/performance is not successful completion. B's present migration cost is justified only if the implementation shows genuine operation ownership and nonduplicated paths. A/E underengineering risk is unnecessary persistence of mechanical caller bytecode where all peers use specialized semantic nodes. C alone does not answer dynamic call specialization; D as default risks two competing owners.

**Pay as you grow:** no compulsory RootTask/RootActor/RootProcess, per-invocation rich activation, extra outcome wrapper or Context-global cache for a trivial synchronous Closure. **Grow as needed:** add arity or richer interop specializations to the same bounded operation family, preserving generic fallback. **Cost of deferral:** later migration touches lowerer, generated opcode/PE baselines, tests and native configurations, but should never require changing Protos guest semantics or B5. **Future regret:** a megamorphic/multi-Context workload, new scheduler, native AOT or unusual instrumentation may make overlarge handlers impractical; retain a narrow semantic helper boundary and existing structured Bytecode control as the escape hatch, with evidence and separate approval for any material departure.

**Strongest objection:** the current complex continuations may prevent complete encapsulation, and a huge operation could increase Truffle PE graph, deopt or code-size even while reducing pre-Graal bytecode. Therefore the implementation MUST prove actual semantics, bounded cache transitions, structural reduction at *representative proven-non-suspending sites*, and no demonstrated PE regression before claiming a completed B migration.

## Implementation authorization and acceptance

This ratification authorizes **separately tracked I-family implementation**, not code edits here. One coherent slice should replace, not duplicate, the real Call0/Call1 dynamic path; include CallN/variadic integration where coherent and human validation as gates. First recover current exact HEAD and local authoritative files. Preserve all published semantics and existing instrumentation. Inspect two comparable source shapes (direct `identity`, wrapper `run`) and adversarial fallback in NORMAL and DEBUG. Use independent handwritten Builder oracle (complete root-group/instructions/constants) plus targeted tests, including generic-first→direct→generic→direct, override after arguments, fresh Closure identity, Context remove/recreate, defaults/rest, suspend/resume/ensure/NLR, cross-Context, uncached/AOT paths. Closure creation changes only if necessary. PERF042 owns measured stable final-tier acceptance.

**STOP** if complete generic invocation cannot be represented without violating structured suspension/cleanup; describe the exact incompatible Truffle constraint, existing Protos normative invariant and smallest owner-reviewable exception. Do not quietly substitute D, simplify semantics or claim that a direct-only fragment satisfies B. Human runs compile/tests/validation/product Git/versioning; no product mutation or tests were carried out by this ratification.

**Outcome:** `OWNER_APPROVAL=EXPLICIT`; `SELECTED_CANDIDATE=B`; `GITHUB021_CONSISTENCY=PASS_FOR_CONTRACT`; `PRODUCT_IMPLEMENTATION=PENDING`; `PERFORMANCE_CLAIM=NONE`.
