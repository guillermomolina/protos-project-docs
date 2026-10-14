# PLAT059 — Preliminary GraalJS/GraalPy canonical-optimization architecture evidence (2026-10-14)

**Status:** PRELIMINARY / INVESTIGATION INTAKE — **NOT A SELECTED OR RATIFIED PLATFORM DECISION**.  
**Owning Issue:** [PLAT059/#894](https://github.com/guillermomolina/protos/issues/894) in `guillermomolina/protos`.  
**Protos source revision inspected:** [`8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff`](https://github.com/guillermomolina/protos/commit/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff), read-only.  
**GraalJS source revision inspected:** [`3df14182071768bd4162734ca69e8b2a81eed5d4`](https://github.com/oracle/graaljs/commit/3df14182071768bd4162734ca69e8b2a81eed5d4).  
**GraalPy source revision inspected:** [`a099d006438942d54c1856c2cce7b6c2839b0c85`](https://github.com/oracle/graalpython/commit/a099d006438942d54c1856c2cce7b6c2839b0c85).  
**Evidence type:** comparative source inspection and proposed investigation boundaries. **No tests, build, benchmark, graph capture, compatibility proof or implementation performed for PLAT059.**

## Detailed source inventories preserved for future research

The initial comparison summarizes, but does not reproduce, the full per-mechanism source audits. The two separate **revision-pinned, historical, non-normative** inventories now preserve the entire enumerated findings and their classification (literal folding vs specialized node/bytecode emission vs scope/control structure vs runtime behavior):

- [**GraalJS — 65 classified mechanisms/variants**](PLAT059_GRAALJS_DETAILED_OPTIMIZATION_INVENTORY_2026_10_14.md), revision `3df14182071768bd4162734ca69e8b2a81eed5d4`.
- [**GraalPy — 57 classified mechanisms/variants**](PLAT059_GRAALPY_DETAILED_OPTIMIZATION_INVENTORY_2026_10_14.md), revision `a099d006438942d54c1856c2cce7b6c2839b0c85`.

These are not counts of independent optimization passes, not exhaustive whole-runtime coverage and **not approval** of a new Protos architecture; the review must still falsify alternatives against ratified PLAT058 and current I101 implementation.

## 1. Why this question is distinct from existing work

- [PLAT058/#891](https://github.com/guillermomolina/protos/issues/891) has already ratified **one shared proof-driven Canonical → Bytecode DSL lowerer** with immutable Context-owned NORMAL/DEBUG physical profiles, deterministic reparse, and a **handwritten independent Builder-oracle requirement** for every claimed lowering optimization. See [ratified PLAT058 decision](../../decisions/platform/PLAT058_PROFILE_SPECIALIZED_MINIMAL_LOWERING_AND_BUILDER_ORACLES.md). This architecture is authority, not a candidate to reconsider.
- [I101/#892](https://github.com/guillermomolina/protos/issues/892) implements PLAT058. Its A slice was recorded as published at Protos `97404f1cc1c1d678c66ef598b47cf62d32014f21`; B/C remain the existing owner for generalized profile-specialized physical emission and acceptance. This intake does **not** transfer that scope.
- [PERF043/#877](https://github.com/guillermomolina/protos/issues/877) already published scoped Integer-expression folding and graph evidence; [PERF043 evidence](../PERF043/PERF043_344_PUBLISHED_INTEGER_FOLDING_AND_GRAPH_CLOSURE_2026_10_13.md). Generalizing constant/fact analysis must reuse that mechanism and its D013 guards rather than pretend the optimization does not exist.
- PLAT059 asks a narrower new **architectural question**: is a separately represented pre-bytecode abstract-value/effect analysis or emission-plan stage actually necessary beyond PLAT058's already-approved small immutable proof decisions? If not, conclude no additional architecture and route existing implementation to I101 or independently warranted work.

## 2. GraalPy findings: explicit static AST optimization

Source: [`graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) and [`BytecodeDSLCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/BytecodeDSLCompiler.java).

- `BytecodeDSLCompiler.compile` explicitly calls `AstOptimizer.optimize(...)` before `RootNodeCompiler` generates bytecode.
- `AstOptimizer.visit(BinOp)` recursively visits both operands and folds two constants through the actual GraalPy `PyNumber*` operations; covered: addition, subtraction, multiplication, true/floor division, remainder for admissible types, power, both shifts, and bitwise AND/OR/XOR. Matrix multiplication deliberately remains unfurled.
- Unary folding covers invert, not, positive, negative. Constant tuples and constant subscript access can fold. List/set literal iterables used in loops/comprehensions/membership may become tuples/frozensets when safe. Certain `not (x is y)` and `not (x in y)` forms are rewritten.
- Potentially expensive multiplication/power/left-shift/repetition are bounded by local resource limits (`MAX_INT_SIZE=128` bits; `MAX_COLLECTION_SIZE=256` for selected collection growth). Exceptions encountered during speculative static folding leave the operation for runtime, not a compile-time semantic substitute.
- There is no evidence here of a **general inter-statement constant propagation pass**; the folding check recognizes `ExprTy.Constant`, not every statically resolved local value.

GraalPy's [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) separately selects fixed-arity call operations (0–4), direct `IsNone/IsNotNone`, specialized `super()`, primitive-typed literal tuple/list constructors, scope-aware local/cell/global operations, and allocation-avoiding unpack/assignment forms. [`PBytecodeDSLRootNode.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java) separates Bytecode DSL instrumentation from ordinary operations and contains runtime specializations such as `ForIterate`. Those **runtime specializations are not AST folds**.

## 3. GraalJS findings: local factory specialization and structural simplification

Sources: [`GraalJSTranslator.java`](https://github.com/oracle/graaljs/blob/3df14182071768bd4162734ca69e8b2a81eed5d4/graal-js/src/com.oracle.truffle.js.parser/src/com/oracle/truffle/js/parser/GraalJSTranslator.java), [`NodeFactory.java`](https://github.com/oracle/graaljs/blob/3df14182071768bd4162734ca69e8b2a81eed5d4/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/NodeFactory.java), [`JSAddNode.java`](https://github.com/oracle/graaljs/blob/3df14182071768bd4162734ca69e8b2a81eed5d4/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/binary/JSAddNode.java), [`JSMultiplyNode.java`](https://github.com/oracle/graaljs/blob/3df14182071768bd4162734ca69e8b2a81eed5d4/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/binary/JSMultiplyNode.java), [`AbstractBlockNode.java`](https://github.com/oracle/graaljs/blob/3df14182071768bd4162734ca69e8b2a81eed5d4/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/control/AbstractBlockNode.java).

- Operator node factories fold certain literal computations (integer addition and multiplication, some unary conversions, equality/null checks). They also create constant-one-side operator variants such as `JSAddConstantRightNumberNode`.
- Structural simplification is performed while constructing AST/Truffle nodes: empty/discardable statement handling, nested-block flattening, compact one/two-statement sequences, terminal returns, and selective loop simplification.
- Tags/instrumentation constrain some otherwise removable structure. It is **not** legitimate to conclude a lowered program has no observable debugger obligation merely because its result is constant.
- The reviewed code does not demonstrate a general local-variable data-flow constant propagation pass or universal algebraic folder. JS/Python native operator semantics cannot be transferred verbatim to Protos sends.

## 4. Protos existing code and actual proof barriers

Read-only baseline: `guillermomolina/protos@8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff`.

- [`CanonicalToBytecodeLowerer.java`](https://github.com/guillermomolina/protos/blob/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff/src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java): `terminalKnownIntegerAddition` / `foldedIntegerAddition` track candidate small integer values but explicitly preserve a guarded runtime send; `foldedConstantControlResult` can emit a statically proven Boolean-controlled literal in NORMAL; scalar-local, inlined Closure and terminal-result physical routes already exist.
- [`CanonicalBindingAnalyzer.java`](https://github.com/guillermomolina/protos/blob/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff/src/main/java/com/guillermomolina/protos/execution/CanonicalBindingAnalyzer.java): order-independent declaration discovery, ordered progressive presence analysis, identities and lexical depth.
- [`CanonicalBindingResolution.java`](https://github.com/guillermomolina/protos/blob/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff/src/main/java/com/guillermomolina/protos/execution/CanonicalBindingResolution.java): `Resolved`, `CapturedResolved`, `Candidate`, `Dynamic` deliberately distinguish name identity from runtime `PRESENT`. A `removeSlot` or nearer shadowing invalidates naive constant-substitution assumptions.
- [`ProtosSemanticBytecodeRootNode.java`](https://github.com/guillermomolina/protos/blob/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff/src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java) already has primitive operation carriers and safeguarded specialization.

**Source-proved distinction:** `value = known integer` in a local analysis is not proof that an ordinary overridable `+` message permanently denotes the standard pure numeric operation. Every proposed reduction must account for selected method/Prelude/Context authority and invalidation (D013), observed binding presence and live capture (D179), Closure identity/escape, effects, overflow, error timing, Task suspension, ReturnHome and NORMAL/DEBUG obligations.

## 5. Decision candidates to research, not selected

| Candidate | Outline | Main advantage | Main falsification risk |
| --- | --- | --- | --- |
| A — incremental existing proof decisions | Extend PLAT058/I101's small immutable emission decisions without a separate global optimizer | Minimal compiler complexity and no duplicate phase | Repeated local reasoning and inability to propagate facts across expression/statement boundaries |
| B — bounded canonical fact/effect analysis | Produce identity-keyed immutable per-site facts and emission decisions before bytecode; preserve authoritative Canonical AST | Reusable conservative analysis across bindings, operations, control, inlining | Compile-time and integration cost, incorrectly treating identity/presence or assumed call purity as guaranteed |
| C — optimized rewritten Canonical AST | Rewrite a second AST/IR then lower | Conventional explicit optimization pipeline | Breaking source spans, binding-node identity, root-group liveness, DEBUG provenance and idempotent reparse; excessive scope |
| D — measured hybrid | Restrict B to a few proven facts and keep all partial-constant operator shape selection in the existing lowerer | Potentially best pay-as-you-grow balance | An extra abstraction that has no measured independent benefit beyond A |

**No winner selected.** A/B/D require concrete falsification against PLAT058's existing proof-based emission invariants. C is included for comparative completeness, not recommended by default.

## 6. Proposed acceptance contract for follow-on work

1. Start with `() => { 1 }`, `true.ifTrue() { 1 }`, pure arithmetic literals, a local binding fed by constants, reassignment, `removeSlot`/recreation, late nearer shadowing, by-reference closure capture, overridden numeric `+` and Boolean selectors, large Integer overflow, mixed number families, and a closure with observable effects/suspension.
2. Record `PROVEN` / `GUARDED` / `UNKNOWN` independently for values, binding presence, selected ordinary method and effects. Never substitute a result when only the operands are known and the send remains overridable.
3. For every admitted lowering rewrite, use **independently handwritten Bytecode DSL Builder oracles** comparing the **complete physical execution instruction names, all constant operands and entire root-group topology**, including reparse, NORMAL/DEBUG and negative cases. A smaller final Graal graph is not proof of pre-Graal minimal emission.
4. Preserve parser/canonical source identity and the approved immutable Context compilation profile. A missing proof must leave the current exact dynamic send and side-effect/error/suspension behavior intact.
5. Pay-as-you-grow: no mandatory observer, global cache, per-call fact object, RootTask/Actor/Process, activation or extra dispatch guard for cases that do not use it. Observe compilation resource cost separately from guest runtime performance.
6. First produce a PLAT decision packet under `AGENTS.work/DESIGN.md` and obtain **explicit owner approval** if a **new durable architectural constraint** is genuinely needed. If not, keep PLAT058 as authority and record `NO_NEW_ARCHITECTURE_REQUIRED`.

## 7. Provenance and limitations

- This is a **preliminary comparative static-source investigation** only. It is **not** an exhaustive GITHUB010 12-axis decision, final GITHUB021 invariant consistency check, benchmark, independent complete Truffle source audit, or authorization to implement.
- Peer revisions are source comparison references, not claims that peers used the same VM stage, semantic contracts or benchmark launch geometry.
- Prior Issue tests/publications are cited as existing project evidence; this record **does not independently rerun or validate** them.
- The governing normative Protos specifications and prior ratified decisions must be read exhaustively at the later design gate before making a proposal; this intake deliberately avoids inventing a semantic contract.
- The PLAT059 Issue is the **live** owner of next status, approval and scheduling. This document preserves the research starting point only.
