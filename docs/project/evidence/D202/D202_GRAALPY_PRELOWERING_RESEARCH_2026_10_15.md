# D202 — GraalPy-style Closure entry: source evidence, falsification and decision comparison

**Role:** durable non-normative research/evidence for [D202/#898](https://github.com/guillermomolina/protos/issues/898). **Approved policy:** 2026-10-15, see [D202 ratification](../../decisions/language/D202_NORMAL_FIRST_CLOSURE_ENTRY_SPECIALIZATION.md). **Implementation:** [I104/#899](https://github.com/guillermomolina/protos/issues/899), NOT YET VERIFIED.  
**Product revision actually inspected:** [`5948ee7ce70d5a70636c921e31566483c4809323`](https://github.com/guillermomolina/protos/commit/5948ee7ce70d5a70636c921e31566483c4809323).  
**GraalPy revision inspected:** [`a099d006438942d54c1856c2cce7b6c2839b0c85`](https://github.com/oracle/graalpython/commit/a099d006438942d54c1856c2cce7b6c2839b0c85).  
**Related approved architecture:** [PLAT058-D/#891](https://github.com/guillermomolina/protos/issues/891) and [PLAT060-B5/#895](https://github.com/guillermomolina/protos/issues/895).  
**Evidence status:** source review; **no** agent-run builds, tests, Java programs, Git publication in product or benchmark measurements. Quoted pre-Graal oracle counts come from the motivating Issue, **not** a new qualified final-Tier-2 comparison.

## The actual answer to “before the lowerer”

* **Compile-time analysis** happens before emitting a Bytecode DSL root; it produces immutable signature/layout/observability facts for the exact `CanonicalClosure` and Context-owned profile.
* **Code emission** happens *inside* the lowerer, using that proof to emit a minimum NORMAL valid-call path or the more complete DEBUG/observable path.
* **Call execution** remains dynamic and subject to Protos's normal `Object.call` lookup, arity, Error and control semantics. Static analysis never executes supplied arguments and never constructs a guest activation.

[ProtosSourceCompiler.java](https://github.com/guillermomolina/protos/blob/5948ee7ce70d5a70636c921e31566483c4809323/src/main/java/com/guillermomolina/protos/execution/ProtosSourceCompiler.java) runs parser → canonicalizer → lowerer; it does not yet expose a separate pre-lowering entry plan. [ProtosBytecodeClosureExecutionPlan.java](https://github.com/guillermomolina/protos/blob/5948ee7ce70d5a70636c921e31566483c4809323/src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeClosureExecutionPlan.java) already runs `CanonicalBindingAnalyzer.analyzeClosure` **before** constructing its lowerer, including inherited whole-tree analysis and replay/reprojection paths. These are existing integration seams, not authorization to create a competing compiler.

[CanonicalBindingAnalyzer.java](https://github.com/guillermomolina/protos/blob/5948ee7ce70d5a70636c921e31566483c4809323/src/main/java/com/guillermomolina/protos/execution/CanonicalBindingAnalyzer.java) uses a two-pass model: declare static parameter identities/layout, then walk default expressions and mark parameters as semantically established **sequentially**. [CanonicalBindingAnalysis.java](https://github.com/guillermomolina/protos/blob/5948ee7ce70d5a70636c921e31566483c4809323/src/main/java/com/guillermomolina/protos/execution/CanonicalBindingAnalysis.java) retains exact AST-identity keyed maps and conservative dynamic/captured resolutions. [CanonicalLexicalScope.java](https://github.com/guillermomolina/protos/blob/5948ee7ce70d5a70636c921e31566483c4809323/src/main/java/com/guillermomolina/protos/execution/CanonicalLexicalScope.java) separates static name layout from runtime PRESENT/ABSENT, explicitly preserving D179.

[CanonicalToBytecodeLowerer.java](https://github.com/guillermomolina/protos/blob/5948ee7ce70d5a70636c921e31566483c4809323/src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java):
- `emitClosureParameterBindings` (~3259–3400) currently builds scalar `LoadFrameClosureArgument` + binding for each supplied positional parameter; defaults branch on `HasFrameClosureArgument`; final excess check is `CheckFrameClosureArgumentUpperBound`.
- `emitBindSuppliedClosureParameter` (~7034) transfers a supplied argument and may materialize a primitive numeric carrier when leaving a frame-native binding.
- `admitsNormalDirectScalarLocalBody` (~1378) explicitly refuses a Closure with *any* parameters. This is a concrete opportunity to extend the proof; it is **not** proof of a 2× compiled runtime cost.
- `requiresPersistentFrameAuthority` (~2445) already detects context-observation/capture risk and allows unobserved NORMAL live frame-native binding.
- `hasArityRejectionRoot` and `emitArityRejectionRoot` (~2003) split an invalid supplied-argument call from the **zero-parameter** valid source body. The fallback signals standard guest Error in an ordinary tagged root with unchanged source provenance.
- Source root group and reparse identity are preserved by the shared `BytecodeRootNodes` construction. Per-definition plans must remain deterministic on reparse; cloned/reprojected ASTs cannot reuse stale identity-keyed metadata.

[ProtosFrameArguments.java](https://github.com/guillermomolina/protos/blob/5948ee7ce70d5a70636c921e31566483c4809323/src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java) uses one compact B5 `Object[]` with slots [0..4] reserved, [5..] exact supplied values; publication of a rich activation changes only [0]. The header and offsets **must never vary with NORMAL/DEBUG**. [ProtosSemanticBytecodeRootNode.java](https://github.com/guillermomolina/protos/blob/5948ee7ce70d5a70636c921e31566483c4809323/src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java) already has guarded frame reads and binding specialization; it also selects the zero-argument rejection target without eager activation. [ProtosBytecodeRootNode.java](https://github.com/guillermomolina/protos/blob/5948ee7ce70d5a70636c921e31566483c4809323/src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java) maintains direct cached dispatch, ordinary `Object.call` lookup, native/rich fallback and guest Error handling.

## Primary upstream implementation precedents

**GraalPy** pinned to [RootNodeCompiler.java](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) ~1898–2015:
1. `setUpFrame` allocates contiguous `BytecodeLocal` layout from already computed scope facts.
2. `copyArguments` builds a `BytecodeLocal[]` of known parameter locals and emits **`b.emitCopyArguments(locals)`**.
3. Cell parameters are subsequently transferred to cells; captured/free variable initialization is explicitly separate from plain argument copying.

[PBytecodeDSLRootNode.java](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java) ~1614–1624 implements `CopyArguments` as `@Operation`, `@ConstantOperand(LocalRangeAccessor)`, `@ExplodeLoop`, and `locals.setObject(bytecodeNode, frame, i, PArguments.getArgument(frame, i))`. [PArguments.java](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/function/PArguments.java) uses a **different six-element header**: do not copy that offset into Protos B5.

[CreateArgumentsNode.java](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/argument/CreateArgumentsNode.java) performs caller-side signature matching, lazily activates keyword/extra-arity error paths and fills **Python** default values. Protos defaults are **invocation-time guest code in the real activation**, so that half of GraalPy is a physical-architecture precedent only, **not** a transplantable semantics algorithm.

[CallDispatchers.java](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/call/CallDispatchers.java) uses identity/root-keyed cached `DirectCallNode` and megamorphic `IndirectCallNode`. Protos already has direct PICs; do not replace dynamic `Object.call` selection by unconditional direct invocation.

**GraalJS** [AccessIndexedArgumentNode](https://github.com/oracle/graaljs/blob/4327aa1f2914bac7f810406423d70ae7f8a1888b/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/arguments/AccessIndexedArgumentNode.java): indexed frame-argument read; missing JavaScript args become `undefined` (NOT Protos Error).  
**TruffleRuby** [RubyCheckArityRootNode](https://github.com/truffleruby/truffleruby/blob/b60b0d04dd11454012d0e8a6588514410ffacdcb/src/main/java/org/truffleruby/language/RubyCheckArityRootNode.java) checks arity early; [ReadOptionalArgumentNode](https://github.com/truffleruby/truffleruby/blob/b60b0d04dd11454012d0e8a6588514410ffacdcb/src/main/java/org/truffleruby/language/arguments/ReadOptionalArgumentNode.java) executes dynamic optional expressions per call. Proc/lambda differences prevent blanket translation.  
**Apple Pkl** [FunctionNode](https://github.com/apple/pkl/blob/d062312df9c294e6abf7dad79cea9518fc9f22c9/pkl-core/src/main/java/org/pkl/core/ast/member/FunctionNode.java) uses a contiguous argument frame, `@ExplodeLoop` and early exact-arity check; its typed/strict semantics are not D003.  
**Outside Truffle:** CPython vectorcall passes contiguous supplied arguments without creating a Python tuple/dict on common positional paths; Self provides a conceptual precedent for activation argument slots but no obligation for eager physical object creation; V8/JVM families are relevant as independent PIC/frame-design analogies, not as direct Protos semantic authorities.

## Semantic counterexamples and decision table

Let `K`=required count, `M`=all ordinary parameters, `R`=has trailing rest, `N`=supplied count. The parser enforces **`required* defaulted* rest?`**. Thus valid arity is `K≤N≤M` without rest, or `N≥K` with rest. An invalid `N<K` fails before every optional default; an invalid `N>M&&!R` supplies each ordinary parameter so no default is evaluated merely before the excess check. This opens a *physical* early arity classification but does **not** authorize changed guest diagnostics, dynamic handlers, source frames or observable state.

| Scenario | Required observable property | Optimization condition |
| --- | --- | --- |
| `(x)=>x`, one input | exact value, lexical parameter identity | admit simple NORMAL path after proof |
| `(x,y)=>y`, two inputs | same, both slots exist to guest | optionally bulk-copy on proven no-context path |
| `(x,y)=>x`, one input | correct Error and Error.handle, source/caller/Task provenance | fail through semantically equivalent cold path |
| `(x)=>x`, two inputs | excess Error, no body effect | no mandatory redundant hot-path check |
| `(a=1,b=a+1)=>b`, zero inputs | execute defaults in true sequential activation | cannot prefill/hoist Python defaults |
| `(a=b,b=1)` | later b absent, lookup may resolve captured/receiver b | don't predeclare b PRESENT |
| default does `context.hasSlot`, `removeSlot`, `context` capture | D179 PRESENCE and exact escaped Context identity | materialize when guest-observable |
| default creates later parameter name, or freezes/closes context | ordinary slot creation conflict and failure precedence | no blind eager bulk-copy |
| `(a=1,...r)` with 0/1/2 inputs | r fresh frozen Array of **supplied** suffix only | do not insert default-produced values into rest |
| overridden `Object.call`, method vs direct, host/polyglot | ordinary lookup guard and selected behavior | preserve PIC/methodHome/receiver/ABI |
| NLR, cleanup, Error, suspension/cancel, return home | unchanged transfer ownership and ordering | no guessed root-splitting escape route |
| NORMAL without active debugger | no future debugger scope guarantee | omit DEBUG-only physical bookkeeping |
| DEBUG attach after earlier calls | accurate scopes from first invocation | retain full DEBUG emission, same B5 ABI |
| rich prepublished activation entry | exact activation object/captures/authority | rich fallback must remain valid |
| Bytecode DSL reparse/cross-Context reprojection | same root topology/identity, fresh correct plan | no global/stale source-identity cache |

**What the original comparison proves:** report motivating pre-Graal builder oracle **16 instructions** versus Protos **38** for an unbalanced `(value)=>value` experiment. The oracle does not implement every Protos semantic obligation; those values are not comparable Tier-2 machine work or speedup. Historical qualified final Tier-2 has been observed for other revisions/cases, but no *current stable matched* before/after Tier-2 comparison for I104 exists. Do not label source inspection as a measured win.

## GITHUB010 alternatives and 12-axis comparison

1–5 = relative strength; H/M/L = confidence. A keeps old binding, B is chosen **narrow pre-lowering NORMAL unary proof**, C is optional multi-arg compact bulk transfer, D is literal transplant of GraalPy defaults/early failure, E is ad-hoc lowerer-only patch, F defers.

| Axis | A status quo | B chosen narrow preplan | C optional bulk | D copy Python literally | E lowerer-only | F defer |
| --- | --- | --- | --- | --- | --- | --- |
| Correctness | 5H shipped | 4M needs proof | 3M more edges | 1H violates semantics | 3M dispersed guards | 5H unchanged |
| Protos alignment | 3H fixed costs | 5M local proof | 4M bounded when useful | 1H imports other model | 3M opaque | 3H conservative |
| Proportionality | 3M some overhead | 5M one needed case | 3M potential machinery | 1H extensive rewrites | 4M small patch | 4H zero new cost |
| Incremental growth | 4H can revisit | 5M seed later arities | 4M phased | 2M migration | 3M ad-hoc spread | 4H future choice |
| Future options | 4H unchanged | 5M proof extensible | 4M DSL coupled | 1H foreign semantics | 3M hidden facts | 5H open |
| Scalability | 3L unknown per-call cost | 5L expected less work | 5L wide signatures | 2L probable overhead | 3L scattered | 3L old cost |
| Simplicity | 4H familiar | 5M one proof | 3M extra operations | 1H second semantics | 3M duplicated decisions | 5H no change |
| Portability | 4H existing Truffle | 5M abstract proof | 4M DSL-specific | 2M Python coupling | 3M codegen only | 5H unaffected |
| Runtime cost | 3L partially optimized | 5L must measure | 5L unproven | 2L copying/cost | 4L unknown | 3L baseline |
| Failure/operability | 5H baseline | 5M fallback path | 3M source/handler risk | 1H wrong precedence | 3M diffuse logic | 5H no change |
| Reversibility | 5H unchanged | 5M opt-in | 4M staged | 1H semantic migration | 3M later refactor | 5H none |
| Evidence maturity | 5H existing tests | 4M concrete source seam | 3M upstream analogy | 1H disqualified | 3M needs test | 5H safe |

**Qualitative non-compensating gates:** D is rejected regardless of aggregate score for D003/default incompatibility. C must be **deferred** if B gives sufficient compiled cost or bulk copying expands hot-path guards. E risks overengineering the already complex lowerer rather than preserving one reusable proof point. A/F impose no semantic migration but preserve opportunity cost. **Selected B is approval of a bounded architecture, NOT proof of runtime speedup**; widening only with matched evidence is an explicit condition.

## Acceptance and falsification in I104

1. Independent exact handwritten Bytecode DSL Builder oracle (instruction/name/constant-operand signature), whole root-group topology, source information and deterministic reparsing; **NORMAL and DEBUG** cross-mode negative controls. Do not accept merely visually shorter Java or an instruction count without semantic parity.
2. Correctness: bound and missing/excess unary, 2+ params, dynamic default/context freeze/close/remove/recreate/lookup and lexical capture, returned Closure identity, overridden `Object.call`, ordinary methods, native rich invocation, host/polyglot and Task/Actor/Process provenance; NLR/cancellation/ensures/suspension; rest/spread; Error type/handler/source trace. The original test sources include [PERF025 compact](https://github.com/guillermomolina/protos/blob/5948ee7ce70d5a70636c921e31566483c4809323/src/test/java/com/guillermomolina/protos/execution/ProtosPerf025CompactCalleeExecutionTest.java) and [PERF032-G6](https://github.com/guillermomolina/protos/blob/5948ee7ce70d5a70636c921e31566483c4809323/src/test/java/com/guillermomolina/protos/execution/ProtosPerf032G6ZeroParameterGuestRootTest.java).
3. Compare eligible **stable final Tier-2** graphs *after* Graal partial evaluation and matched warmup against the actual product baseline; distinguish **pre-Graal Bytecode DSL** instructions, PE graph nodes and machine-level runtime. Do not claim better performance when NOT_STABLE or data absent.
4. Human executes compilation, tests, checks, benchmark and Git; agent prepares code/tests/docs. No automatic metadata version bump before tests and no tests after `pom.xml`/`CHANGELOG.md` promotion; no agent-run execution is claimed.

```text
RESEARCH_BASELINE_PROTOS=5948ee7ce70d5a70636c921e31566483c4809323
GRAALPY_UPSTREAM_SHA=a099d006438942d54c1856c2cce7b6c2839b0c85
OWNER_SELECTED=D202_NORMAL_FIRST_PRELOWERING
OWNER_APPROVAL_DATE=2026-10-15
D003_SEMANTIC_CHANGE=NO
PLAT060_B5_ABI_CHANGE=NO
NORMAL_LATE_DEBUGGER_GUARANTEE=NO
DEBUG_LATE_DEBUGGER_GUARANTEE=YES
I104_IMPLEMENTED=NO
BENCHMARK_TIER2_STABLE_AB=NOT_YET_MEASURED
AGENT_EXECUTED_TESTS=NO
```
