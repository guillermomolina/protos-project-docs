# PLAT064-C — Agent / Human Executor handoff after I111

Date: 2026-10-18
Type: **INVESTIGATION ONLY**; no commands, edits, builds, tests, benchmark, or repository mutation.
Issue: https://github.com/guillermomolina/protos/issues/915
Parent: https://github.com/guillermomolina/protos/issues/911
Product repository: `guillermomolina/protos`
Evidence repository: `guillermomolina/protos-project-docs`
Product starting baseline: [`edcb758fadb0e749cd45711cd664044d0b6db6bd`](https://github.com/guillermomolina/protos/commit/edcb758fadb0e749cd45711cd664044d0b6db6bd) (`0.3.377-SNAPSHOT`); refresh actual product HEAD when investigating.
Preceding dossier: [PLAT064-E completed findings](https://github.com/guillermomolina/protos-project-docs/blob/84d80ddb1733cbc3c42433cd998602cb0543c3d5/docs/project/evidence/PLAT064/PLAT064_E_SOURCE_DIAGNOSTIC_TRACE_AUDIT_2026_10_18.md).

## Agent — investigation prompt

**TIPO=INVESTIGACIÓN.** Investigate exhaustively, as a source-only comparison, the **actual effective operation surface** of Protos's single Truffle Bytecode DSL generated interpreter after I111, including every `@Operation` and `@OperationProxy` family, builder emitters, indirect C-prime/Task/IO consumers, generated operations, tests and legitimate overlap or reuse opportunities.

**Mandatory rebaseline:** Only `ProtosBytecodeRootNode` remains a `@GenerateBytecode` root in the current starting baseline. The former `ProtosSemanticBytecodeRootNode` was removed by I111/#919. The original 235/175/76 historical operation-count comparison is not valid as a current count. I112 already provides `@Prolog`, `@EpilogReturn`, `InvokeClosure*Tail` and suspended-only continuation ownership. Do not repeat approved I111/I112 work or reintroduce semantic/helper generator duplication.

### Repository authority

Read `AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, and live PLAT064-C and PLAT064 issues. Read normative `spec/semantics/CALLABLES.md`, `ERRORS.md`, `EXECUTION_AND_CONTROL.md`, `VALUES_AND_COLLECTIONS.md`, `spec/concurrency/FUTURES_AND_TASKS.md` and adjacent domain owners only as concrete consumers demand. Don't invent semantics or unapproved platform architecture.

Inspect HEAD, beginning from:
- `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java`
- `src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java`
- `src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/ProtosStructuredDispatchLowerer.java`
- Relevant production emitters in `execution/host/resources/`, `execution/bytecode/activation/`, `execution/host/embedding/`, and `execution/bytecode/numeric/`. Some numeric Node classes already use `@OperationProxy`; do not presume it absent.
- `src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI101CIndependentBuilderReferenceTest.java`, `ProtosI112TailContinuationTest.java`, `ProtosPerf040E1NumericOperationReconciliationTest.java`, and any additional direct/synthetic builder tests established from references.

Pinned upstream Bytecode DSL API sources: [oracle/graal@95ce149](https://github.com/oracle/graal/tree/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle) — `GenerateBytecode`, `Operation`, `OperationProxy`, parser and DSL generator. Protos declares GraalVM 25.4.4.1.1; avoid assuming exact release-processor binary equivalence to this snapshot. Use pinned peer examples: [GraalPy Bytecode DSL](https://github.com/oracle/graalpython/blob/7e7182b1036d56f8e89ecc9c7a86cee292180aa6/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java), [Oracle SimpleLanguage](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.sl/src/com/oracle/truffle/sl/bytecode/SLBytecodeRootNode.java), and [TrufflePascal](https://github.com/d45h71/TrufflePascal/tree/fcae9007500369dd194c7e09c932fd4807d64fa8). GraalJS and TruffleRuby AST implementations are not direct Bytecode DSL root precedents.

### Deliverables

1. Count and enumerate live unified-root operation declarations and proxies; distinguish actual duplicated implementations from operation aliases/metadata, and historical two-root counts from current data.
2. Trace declaration -> canonical, structured, Task/IO/C-prime, host callback and test builder uses -> generated specialization -> runtime contract/test. Classify as `REQUIRED`, `TEST_ONLY_OR_CONTRACTUAL`, `NO_CONSUMER_DEMONSTRATED`, or `INCONCLUSIVE`. Missing literal grep consumers does **not** prove dead code.
3. Determine whether a demonstrably redundant operation, proxy, Node specialization, or generated DSL feature admits a narrowly scoped change without breaking cached/uncached execution, semantic root tagging, lazy source/DEBUG information, boxed vs canonical Boolean, frame lexical authority, ReturnHome, ensure, Error handling, suspension/resumption, task ownership, cross-Context or Native constraints.
4. Respect the PLAT063 boundary: emitted semantic-operation granularity is out of scope; only the generated framework-operation surface is owned by PLAT064-C.
5. Compare mechanisms against multiple true Bytecode DSL peers and pinned upstream API. For each divergence output all PLAT064 fields `FACILITY`, `CURRENT_PROTOS_HEAD_AND_SOURCE`, `TRUFFLE_API_VERSION_AND_DEFAULT`, `BYTECODE_DSL_PEER_SOURCE_AND_BEHAVIOR`, `CURRENT_PROTOS_USE_AND_REQUIRED_SEMANTICS`, `DIVERGENCE`, `DISPOSITION`, `JUSTIFICATION`, `OWNER_AND_NEXT_ACTION`.
6. Falsify proposed removals or proxy rewrites against concrete negative tests; report `KEEP`, `EXISTING_OWNER`, `PROPOSE_BOUNDED_Ixxx`, `DESIGN_GATE` or `INCONCLUSIVE`. Any Ixxx is a proposal, not automatically allocated or authorized. No performance, Native Tier-2 or speedup gate.
7. Present a finite complete source-backed dossier with exact refs and tests as *source evidence*, not as executed results. Explicitly list inspected material, missing evidence and precise next action.

**Investigations never execute:** NO shell/Maven/Git commands, tests, builds, benchmarks, scripts, source changes, docs changes or GitHub Issue edits by the investigating agent. Read GitHub or web sources only. No subagents.

## Human Executor

No code editing, validation or publication command batch applies to this research. The Human Executor receives the dossier and chooses or rejects future work after source-backed comparison. If and only if a future `Ixxx` implementation is explicitly approved, draft a separate autonomous HEAD-based implementation prompt and its first human-executed editing batch. The current research is neither an implementation nor a ratification.

`NEXT_WORK=PLAT064-C/#915`
`TIPO=INVESTIGACION`
`PRODUCT_SHA_AT_HANDOFF=edcb758fadb0e749cd45711cd664044d0b6db6bd`
`AGENT_COMMAND_BATCH=NOT_APPLICABLE`
`HUMAN_EXECUTOR_COMMAND_BATCH=NOT_APPLICABLE`
`FIRST_EDIT_PBATCH=NOT_APPLICABLE`
