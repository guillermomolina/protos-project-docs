# PLAT064-E — Agent / Human Executor handoff after I111 unification

**Date:** 2026-10-18
**Type:** INVESTIGATION ONLY; **no local code edits, builds, tests, execution or Git mutations in `guillermomolina/protos`**.
**Work issue:** [PLAT064-E/#917](https://github.com/guillermomolina/protos/issues/917)
**Parent:** [PLAT064/#911](https://github.com/guillermomolina/protos/issues/911)
**Current product baseline:** [`edcb758fadb0e749cd45711cd664044d0b6db6bd`](https://github.com/guillermomolina/protos/commit/edcb758fadb0e749cd45711cd664044d0b6db6bd), 0.3.377-SNAPSHOT
**Completed prerequisite:** [I111/#919](https://github.com/guillermomolina/protos/issues/919), PLAT065-A unified Bytecode DSL interpreter.

## Agent mission

Investigate whether Protos's exceptional Error trace, BytecodeLocation and source-position capture, lazy TagTree reparse, and diagnostic logical-frame projection still contain **demonstrable** divergences from supported Truffle 25.4.4.1.1 Bytecode DSL facilities. Focus on exact semantics and source evidence, not performance claims or speculative redesign.

**Mandatory rebaseline:** a single `@GenerateBytecode` now lives in `ProtosBytecodeRootNode`; the former `ProtosSemanticBytecodeRootNode` is deleted. The same generated interpreter owns semantic and auxiliary C-prime physical roots. `isSemanticActivationRoot()` identifies genuine source activations. `enableRootTagging=false` and the canonical emitter adds `StandardTags.RootTag` only to genuine source activations, with nested inline Closure semantic tags. The generator has `@Prolog EnterRoot`, `@EpilogReturn`, and I112 `InvokeClosure*Tail`. Historical PLAT064 comparison tables describing two live classes or absent prolog/epilog are not current.

**Read-only product sources:** `AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, applicable `spec/semantics/{CALLABLES,EXECUTION_AND_CONTROL,ERRORS}.md`, `spec/concurrency/{FUTURES_AND_TASKS,ACTORS}.md`; `ProtosBytecodeRootNode` interception and root identity; `ProtosDiagnosticTraceCapture`; `ProtosSignalException`; `ProtosBytecodeControlTransferException`; `ProtosTailCallContinuation`; `ProtosBytecodeTagTreeNodeExports`; `CanonicalToBytecodeLowerer`; relevant CLI008, PLAT044, PLAT058, I111, I112, debugger/uncached and source reparse tests. Follow concrete references at HEAD rather than reading unrelated large subsystems by default.

**Pinned framework and peer references (research sources, not implementation dependencies):**
- Oracle Bytecode DSL `@GenerateBytecode` API: https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/GenerateBytecode.java
- Oracle Truffle SDK revision: https://github.com/oracle/graal/tree/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle ; inspect `BytecodeLocation`, `BytecodeNode`, `BytecodeConfig`, `BytecodeRootNodes`, `ContinuationResult`, `TagTree` and stack-trace APIs at pinned revision.
- GraalPy Bytecode DSL root and related diagnostics: https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java (resolve peer revision when citing).
- Oracle SimpleLanguage Bytecode DSL: https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.sl/src/com/oracle/truffle/sl/bytecode/SLBytecodeRootNode.java
- TrufflePascal Bytecode DSL manual-tag emitter: https://github.com/d45h71/TrufflePascal/blob/fcae9007500369dd194c7e09c932fd4807d64fa8/language/src/main/java/org/trufflepascal/bytecode/PascalBytecodeEmitter.java
- GraalJS/TruffleRuby are **AST-based**, not direct Bytecode DSL root-configuration precedents.

## Required source-trace cases

1. **Synchronous semantic Error:** signal creation, first-origin recording, Truffle stack element position, lazy `ensureSourceInformation`, explicit RootTag, projection.
2. **Auxiliary C-prime Error or control transfer:** no false source frames, semantic caller retained, handler selection before cleanup.
3. **UNCACHED and reparse:** recorded per-frame bytecode position, `BytecodeLocation.update()`, local/tag/source-table availability, no change of semantic owner or source section.
4. **Nested inline Closure:** genuine logical inline RootTag frames; no double-counting of physical root.
5. **I112 terminal continuation:** suspended source root already returned, pending owner finishes once, resumed failure records logical caller with correct ordering and source, `ReturnHome`/cancellation/ensure.
6. **Debugger/multiple contexts/Native:** observable source/scopes and authority; distinguish confirmed API functionality from unsupported speculation or a diagnostic-only workaround.

For each distinct facility produce the exact PLAT064 fields:
`FACILITY`, `CURRENT_PROTOS_HEAD_AND_SOURCE`, `TRUFFLE_API_VERSION_AND_DEFAULT`, `BYTECODE_DSL_PEER_SOURCE_AND_BEHAVIOR`, `CURRENT_PROTOS_USE_AND_REQUIRED_SEMANTICS`, `DIVERGENCE`, `DISPOSITION`, `JUSTIFICATION`, `OWNER_AND_NEXT_ACTION`.

Falsify plausible simplifications. Categorize `KEEP` when current machinery is necessary, `EXISTING_OWNER` when already addressed by I111/I112, `PROPOSE_BOUNDED_Ixxx` only with a verified consumer-level benefit, `NEEDS_DECISION` for genuinely new design choices, and `INCONCLUSIVE` only with an exact missing proof. **No performance improvement or Native Tier-2 workaround is required.** Do not propose an implementation because of a superficial annotation mismatch. Do not reopen approved PLAT065-A.

## Human Executor boundary

**There is no Human Executor code batch or validation command for this investigation.** The agent reads the current product HEAD and web/peer sources, reasons from them and delivers a source-backed report. It does not edit, run Maven/tests, execute tools, change Git, or prepare product metadata. If a future change is separately approved and allocated to an Ixxx implementation, only then provide Human Executor commands for its bounded edit/test/publication sequence.

A durable report may be coordinated under `docs/project/evidence/PLAT064/` subject to the publication permission and closure-evidence checks; this handoff is not the investigation result or an owner-approved new design.

`NEXT_WORK=PLAT064_E`
`TIPO=INVESTIGACION`
`PRODUCT_SHA=edcb758fadb0e749cd45711cd664044d0b6db6bd`
`SOURCE_EDIT_BATCH=NOT_APPLICABLE`
`TEST_COMMAND_BATCH=NOT_APPLICABLE`
