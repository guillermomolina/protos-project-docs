# I111 — A publication and B implementation / Human Executor handoff

**Date:** 2026-10-17  
**Formal owner:** [I111/#919](https://github.com/guillermomolina/protos/issues/919)  
**Decision:** [PLAT065-A/#918](https://github.com/guillermomolina/protos/issues/918) (ratified; implementation separate)  
**Product repository:** `guillermomolina/protos`  
**Evidence repository:** `guillermomolina/protos-project-docs`  
**Exact product publication:** [`63a178e3ea19af1caf3e2e43e947130c5bcfc792`](https://github.com/guillermomolina/protos/commit/63a178e3ea19af1caf3e2e43e947130c5bcfc792), `I111: implement PLAT065-A explicit semantic RootTag emission (0.3.375)`.  
**Validation provenance:** project owner reports local `git diff --check` clean and all local tests PASS after I111-A, including the I112 guard suite. This record did not run tests, re-verify CI, or independently read full console output.  
**Lifecycle:** I111-A product publication complete; **I111/#919 remains open** for single-generator migration (B) and cleanup/acceptance (C). No new issue necessary.

## I111-A published contract

- The generated semantic root `ProtosSemanticBytecodeRootNode` has `enableRootTagging=false`; true source/module/Closure roots are bracketed explicitly with `StandardTags.RootTag`, retaining nested inlined callback tags.
- `CanonicalToBytecodeLowerer.emitRootBody` declares semantic root-owned `BytecodeLocal` bindings **before** the new `RootTag → Block` so their lifetime covers the physical root. Executable code remains inside the tag; block-local temporaries retain their own scopes.
- NORMAL Error diagnostic nested-tag traversal skips the explicit physical root and retains actual nested callback frames. `RootBodyTag` remains disabled; source/tag information remains lazily materialized.
- Handwritten independent Bytecode Builders and location/debugger/inline-callback structural assertions were reconciled to explicit tagging. When `ensureComplete()` exposes an unreachable implicit fallthrough epilogue, regression tests check the original instruction prefix exactly and then only the ordered Block/root-local cleanup, optional `EpilogForReturn` and final `return`. This is **not** permission to relax DEFAULT emitted-bytecode or execution assertions.
- The `ProtosFrameLexicalBindingAuthority` and `ProtosBytecodeTagTreeNodeExports` local-read behavior was not rewritten to mask out-of-scope locals.
- The entire I112/PLAT062-T implementation from the preceding commit is retained: `@Prolog EnterRoot`, `@EpilogReturn`, compact `InvokeClosure*Tail`, delegated terminal suspended-completion/cleanup, and `ProtosTailCallContinuation`. The current semantic generator **does have** `@Prolog` and `@EpilogReturn`; older research/handoff statements to the contrary are historical and **must not be reused**.
- `ProtosI112TerminalCallBudgetTest` asserts a NORMAL two-root FULL unfiltered instruction target of at most **16**, with a separate explicit ceiling of **17**; passing the test establishes `<=16`, not an independently recorded exact census of 16. No third mandatory source root.

## I111-B — approved next work

**TIPO=IMPLEMENTACIÓN. Execution repository: `guillermomolina/protos` at current local HEAD.** No new PLAT selection or peer-source investigation is authorized/needed. Agent edits; Human Executor alone compiles, tests, validates, changes version metadata and publishes product commits. Do not revert I112 while unifying generated interpreter families.

**Single retained `@GenerateBytecode` family:** `ProtosSemanticBytecodeRootNode`; retire `ProtosBytecodeRootNode` after dependent consumers have migrated, preserving legitimate multiple physical roots/CallTargets/continuations. Legacy helper roots must be **untagged** despite sharing the generator. Source/inline callbacks retain true `RootTag`; do not introduce `RootBodyTag`.

### Production entry points and ownership

- `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosSemanticBytecodeRootNode.java`: retained generator, I112 prolog/epilog and source operations; add helper-only operations without changing existing source dispatch.
- `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java`: old helper generator; inspect its latest helper-only operations, shared types and invocation bridge before consolidation or removal.
- `src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/ProtosStructuredDispatchLowerer.java`: `ProtosBytecodeRootNodeGen.Builder` producer, structured C-prime control and resumption.
- `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosTaskCPrimeEntryExecution.java`: direct helper root `ProtosBytecodeRootNodeGen.create` producer; currently local task-entry scratch and return.
- Locate all direct I/O C-prime producer `ProtosBytecodeRootNodeGen.create` sites in the product HEAD and migrate each one, together with any group/continuation/result types and imports.
- `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeTagTreeNodeExports.java` and `src/main/java/com/guillermomolina/protos/execution/host/tooling/ProtosDiagnosticTraceCapture.java`: currently distinguish source/helper partly by concrete interpreter class. Under the unified class ensure helper roots cannot be reported as genuine guest frames or debugger source lexical scopes. Use per-physical-root role information rather than a mutable global registry. Protect existing lazy NORMAL behavior.

### Operation collision matrix — current published product

Shared names requiring signature, operand, cache, exception and specialization review include `ReadMember`, `BindClosureFrameParameter`, `PrepareSendArguments`, `EnterNestedStructuredDispatch`, `CompleteClosureCall`, `FinishClosureCall`, `IsContinuation`, `ResumeContinuation`, `ReadFrameLocal`, `InstallFrameLexicalAuthority`, `CreateCurrentFrameLocal`, `RequiresStructuredDispatch`. Incompatible same-name operations must be retained under distinct descriptive names and every affected Builder call rewritten; equivalence must not be inferred from spelling.

Helper-only operation families identified in the published helper include structured case-of/object/import/ensure, while/local and array/bytes/environment/foreign/identity-map/map each, map match/read/atPut/remove, Error/cancellation, buffered byte reader/writer, IO release, text reader/writer. Source-only operations include I112 `InvokeClosureZeroTail`, `InvokeClosureOneTail`, `InvokeClosureArgumentsTail` and I112 prolog/epilog, which must never be lost or replaced by older fallback paths.

### Explicit external implementation references, already selected

- Truffle Bytecode DSL `@GenerateBytecode` processor/API version 25.4.4.1.1 (project GraalVM line). Source: https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/GenerateBytecode.java . Annotation controls instrumentation per **generator**, not root; `enableRootTagging=true` would falsely tag helpers after the merge. Keep `enableRootTagging=false` and explicit tags only in the canonical semantic builder. Merged generator may still emit multiple physical roots.
- Oracle Bytecode DSL parser supports beginRoot/endRoot, beginBlock/endBlock, beginTag/endTag and materialization through BytecodeConfig and group update. Creating a BytecodeLocal within a Block changes its scope: preserve root declared locals outside the explicit tag's block, as A already establishes.
- TrufflePascal manual-tag precedent: https://github.com/d45h71/TrufflePascal/blob/fcae9007500369dd194c7e09c932fd4807d64fa8/language/src/main/java/org/trufflepascal/bytecode/PascalBytecodeEmitter.java .
- GraalPy single DSL generator precedent: https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java . GraalPy automatic tag strategy is **not** valid for Protos merged helper roots.
- SimpleLanguage single generator precedent: https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.sl/src/com/oracle/truffle/sl/bytecode/SLBytecodeRootNode.java . It does not prove Protos helper/source semantic equivalence.
- GraalJS is not a Bytecode DSL generator analogue; no second source interpreter or JS-style workaround.

No new external research, web, or other repository checkout is required for B; the current product checkout and this handed-off contract suffice.

### Acceptance and anti-regression

- Current normative owners: `spec/semantics/{CALLABLES,EXECUTION_AND_CONTROL,ERRORS}.md`, `spec/concurrency/{FUTURES_AND_TASKS,ACTORS}.md`, relevant `spec/io/*`; follow product `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `src/AGENTS.md`.
- Verify complete producer, consumer and operation reachability. Preserve helper continuation/yield/returnHome/ensure/error/cancellation, lexical captured roots, stack/tag scopes, debugger, NORMAL lazy trace, immutable Context profile, and zero additional successful-path debug branches.
- Preserve I112 NORMAL DEFAULT full physical instruction budget: `ProtosI112TerminalCallBudgetTest` targets `<=16`, absolute ceiling 17, exactly two source roots; preserve `ProtosI112TailContinuationTest` and related I107/I108 tests. Different DEBUG `ensureComplete()` instrumentation is counted/reconciled separately and must not be used to weaken DEFAULT budget.
- Respect the agent/human-executor split: **agent may edit only**, no build/tests/scripts/product Git state changes, no version/CHANGELOG until green human validation. After tests, human edits metadata and publishes; **no tests after metadata**.
- If full B cannot be produced as a coherent passing publication, stop and report the exact blocked contract rather than leaving a purported released half-merged generator. Keep I111-C cleanup distinct only when it can be a coherent subsequent publication.

`I111_A_PRODUCT_SHA=63a178e3ea19af1caf3e2e43e947130c5bcfc792`  
`I111_A_OWNER_REPORTED_DIFF_CHECK=PASS`  
`I111_A_OWNER_REPORTED_LOCAL_TESTS=PASS`  
`I112_DEFAULT_INSTRUCTION_BUDGET_PRESERVED_BY_TEST=LE16`  
`I111_OVERALL=OPEN_PENDING_B_C`  
`NEXT_SLICE=I111_B_IMPLEMENTATION`
