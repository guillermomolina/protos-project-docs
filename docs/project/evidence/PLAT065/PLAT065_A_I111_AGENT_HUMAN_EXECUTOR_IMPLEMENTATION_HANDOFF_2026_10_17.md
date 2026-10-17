# PLAT065-A / I111 — Implementation agent and Human Executor handoff

**Current status:** APPROVED architecture, **TIPO=IMPLEMENTACIÓN**.
**Agent work repository:** `guillermomolina/protos`, **always the current local HEAD**.
**Decision authority:** [PLAT065-A](../../decisions/platform/PLAT065_SELECTIVE_SEMANTIC_ROOT_TAG_SINGLE_BYTECODE_INTERPRETER.md), [I111/#919](https://github.com/guillermomolina/protos/issues/919).
**Historical inspected baseline:** `62ca4eb602432f3457c080954b716f8466b76983` (0.3.373-SNAPSHOT) after I108.
**Explicit role boundary:** Agent reads/edits locally but never runs project programs, builds, tests, mutation scripts or product Git push; human executes these steps, reports results and publishes `guillermomolina/protos`. Owner authorized documentation-agent commit/push **only in `guillermomolina/protos-project-docs`**.

## Agent — exact implementation mandate

Do **not** investigate again whether manual RootTag is a valid Truffle mechanism or seek another candidate approval. The owner expressly selected Candidate A. Only an actual current-code contract contradiction warrants reporting a blocker.

Read these files in the *product checkout*, without web or other repositories: `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `src/AGENTS.md`, `spec/semantics/{CALLABLES,EXECUTION_AND_CONTROL,ERRORS}.md`, `spec/concurrency/{FUTURES_AND_TASKS,ACTORS}.md`, relevant `spec/io/*` when editing IO producers, `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosSemanticBytecodeRootNode.java`, `ProtosBytecodeRootNode.java`, `ProtosBytecodeTagTreeNodeExports.java`, `src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/{CanonicalToBytecodeLowerer,ProtosStructuredDispatchLowerer}.java`, `src/main/java/com/guillermomolina/protos/execution/host/tooling/ProtosDiagnosticTraceCapture.java`, and related Task/IO C-prime producers/tests and `AGENTS.work/IMPLEMENTATION.md` blocker rules. Work only from current HEAD (not historic line numbers).

### Facts from Truffle and peers, already investigated; do not browse

- Oracle Truffle Bytecode DSL API 25.4.4.1.1: `@GenerateBytecode(enableRootTagging=true)` automatically tags **every** root from the generated class, including helper roots if merged. A single generator therefore requires `enableRootTagging=false` with explicit `Builder.beginTag(StandardTags.RootTag.class)` / `endTag` for **semantic** roots only. `enableTagInstrumentation=true` must stay enabled for tag functionality. Physical root wrappers are `Builder.beginRoot()` / `endRoot()`. Current Protos roots have **no `@Prolog`**, only control-flow and Truffle exception interceptors; Oracle advises caution if a prolog is introduced later.
- `BytecodeConfig.DEFAULT` does not materialize optional tag/source trees by default; `newConfigBuilder().addSource().addTag(RootTag.class).build()` and `BytecodeRootNodes.update(config)` materialize lazy tags on Error or tooling demand. This is orthogonal to the Context-owned physical NORMAL/DEBUG profile.
- Current `CanonicalToBytecodeLowerer.emitRootBody` brackets every genuine semantic physical `Root` inside source sections. It *also* emits nested `RootTag` for inlined Closure activations: **preserve them**. `ProtosDiagnosticTraceCapture.activeNestedRootTags` previously traverses the automatic RootTag tree, so manual outer tagging may change traversal geometry. Ensure one genuine top-level frame plus one frame per live inline Closure, not duplicate outer frame.
- `ProtosBytecodeTagTreeNodeExports` and `ProtosDiagnosticTraceCapture` currently classify semantic roots using `instanceof ProtosSemanticBytecodeRootNode`; after merge, that class will contain helpers too. Introduce a small Context-local/root-instance immutable classification bit or enum from the root producer, used only for diagnostics and source scopes; no global registry, no automatic runtime mode branches.
- `ProtosStructuredDispatchLowerer` and six direct Task/IO C-prime builders use `ProtosBytecodeRootNodeGen.Builder`; `CanonicalToBytecodeLowerer` uses `ProtosSemanticBytecodeRootNodeGen.Builder`. Migrate all to ONE retained generator without deleting real C-prime continuation targets or letting auxiliary roots acquire `RootTag`.
- I108 introduced specialized Call0/Call1/CallN/spread Closure operations in both roots. **Do not** port an obsolete pre-I108 operation inventory. Critical same-name incompatible families include `ReadMember`, `BindClosureFrameParameter`, `PrepareSendArguments`, `EnterNestedStructuredDispatch`, `CompleteClosureCall`, `FinishClosureCall`, `IsContinuation`; resolve by current signature, semantic behavior and consumer, not alphabetical name deduplication.
- Oracle reference `GenerateBytecode.java`: https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/GenerateBytecode.java . This URL is **information given**, not a dependency to browse.
- TrufflePascal manually emits tags with automatic tagging disabled: https://github.com/d45h71/TrufflePascal/blob/fcae9007500369dd194c7e09c932fd4807d64fa8/language/src/main/java/org/trufflepascal/bytecode/PascalBytecodeEmitter.java .
- GraalPy (one Bytecode DSL generator, automatic tags): https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java .
- SimpleLanguage (one generator, automatic physical RootTag and explicit body tags): https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.sl/src/com/oracle/truffle/sl/bytecode/SLBytecodeRootNode.java .
- GraalJS uses AST instrumentation, not Bytecode DSL; do not translate it into a second emitter or a workaround.

### Implementation plan

**I111-A (first self-contained publication):** before merging roots, switch semantic root generator to `enableRootTagging=false`, explicitly bracket the genuine semantic physical root in `emitRootBody`, preserve nested inline RootTag, adapt diagnostic tree recursion if required by actual resulting Bytecode TagTree (not speculation), refresh source comments and add/adjust tight regression for lazy error trace, NORMAL/DEBUG root events, debugger/source scopes and reparse; helpers stay tagless. Coherent passing intermediate, no new semantics.

**I111-B:** merge helper `@Operation` surface and all C-prime producers into the retained semantic generator; preserve conflicting op signatures via explicit distinct names/adapters; migrate direct Builder types and exports, protect lexical group, locals, continuation identity, exception/error paths. Eliminate second generator only when the product compiles coherently.

**I111-C:** retire obsolete class/declarations, ensure one `@GenerateBytecode` in product, full DEBUG/trace/source/C-prime acceptance with independent PLAT058 Builder-oracle topological checks. Group mechanically small edits inside those work stages; do not create micro-issues.

## Human Executor — first editable batch

The initial safe edit batch has three targets: semantic `@GenerateBytecode` flag, outer semantic-root tag opening and closing in `CanonicalToBytecodeLowerer`, and accuracy of current Javadoc. It does NOT touch `pom.xml` or `CHANGELOG.md`. The exact Python edit script is passed in the owner's active handoff; it asserts matches on local HEAD and aborts on concurrent drift instead of applying a guess. Agent should inspect the diff before requesting the next validation.

Follow the human-executor gates: `git status --short`, `git diff --check`, full patch review, focused tests then integrated `make test` / `make check` as required by actual changed scope. Do not run full tests after each tiny edit. After all tests pass and the code is accepted, *only then* edit `pom.xml` and `CHANGELOG.md` according to current version policy, perform no tests after metadata, and publish with explicitly scoped `git add -- <paths>`, one-line `I111: ...` message, never force push. Product Git operations belong to the human, not this documentation agent.

`TIPO=IMPLEMENTACIÓN`
`REPOSITORY=guillermomolina/protos`
`SLICE=I111-A`
`PEER_RESEARCH_REQUIRED=NO`
`FIRST_PATCH_BATCH=SEMANTIC_SELECTIVE_ROOT_TAG`
`PRODUCT_TESTS_EXECUTED_BY_THIS_HANDOFF=NO`
