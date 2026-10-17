# PLAT065-A — Explicit owner approval, precedent and invariant record (2026-10-17)

**Issue:** [PLAT065/#918](https://github.com/guillermomolina/protos/issues/918)
**Ratified mechanism:** [PLAT065 platform decision](../../decisions/platform/PLAT065_SELECTIVE_SEMANTIC_ROOT_TAG_SINGLE_BYTECODE_INTERPRETER.md)
**Inspected product baseline:** `62ca4eb602432f3457c080954b716f8466b76983` (0.3.373-SNAPSHOT), after I108.
**Prior records:** [baseline](PLAT065_INTERPRETER_UNIFICATION_BASELINE_2026_10_17.md), [research-only agent/human handoff (historical version)](PLAT065_AGENT_HUMAN_EXECUTOR_RESEARCH_HANDOFF_2026_10_17.md).

## Approval provenance

On 2026-10-17 the project owner answered **"apruebo A"** to a concrete choice previously presented as:

- one `@GenerateBytecode` interpreter;
- `enableRootTagging=false`;
- explicit `RootTag` on semantic root activations, not C-prime helper roots;
- retain normal error-only lazy instrumentation and PLAT058 NORMAL/DEBUG split;
- narrow supersession of PLAT026's manual-tag restriction.

The owner also stated `git diff --check` was clean and all existing local tests passed. These statements document **pre-implementation product state only**. No I111 implementation/test/build has occurred in this record.

## Source-backed selection and negative findings

- Oracle Truffle API, [GenerateBytecode.java at inspected source revision](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/GenerateBytecode.java), supports `beginTag/endTag`; it warns about manual root instrumentation with root prologs. Protos semantic/helper roots at product SHA above contain exception/control interceptors, **not `@Prolog` or `@Epilog`**.
- [TrufflePascal manual RootTag emitter](https://github.com/d45h71/TrufflePascal/blob/fcae9007500369dd194c7e09c932fd4807d64fa8/language/src/main/java/org/trufflepascal/bytecode/PascalBytecodeEmitter.java) supplies real one-generator/manual-tag precedent; the project did not validate its debugger correctness.
- [GraalPy](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java) and [Oracle SimpleLanguage](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.sl/src/com/oracle/truffle/sl/bytecode/SLBytecodeRootNode.java) exemplify one generated interpreter with automatic tagging; they are not manual-selective-tag proofs. GraalJS is AST instrumentation rather than a like-for-like Bytecode DSL emitter.
- In current [CanonicalToBytecodeLowerer.java](https://github.com/guillermomolina/protos/blob/62ca4eb602432f3457c080954b716f8466b76983/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java), `emitRootBody` emits each physical semantic root via `builder.beginRoot()` / `builder.endRoot()` (approximately lines 1021 and 1376); separate nested `RootTag` emission exists for inline callbacks. This is the precise first migration seam, not permission to remove nested regions.
- Existing `ProtosDiagnosticTraceCapture` distinguishes semantic roots by type and lazily reparses `RootTag` on Error. In a merged generator, role identification must be explicit and instance/local, never inferred from the now-shared Java type.

## GITHUB021 delta (precise)

PLAT026 manual tagging: **reopened and superseded only for semantic root emission under this unified generator**; retain truthful RootTag, no helper tags, source/debugger correctness. PLAT042 dual generators: **superseded topology only**; retain PLAT041 inline bodies, capture, grouping and PLAT014 continuation behavior. PLAT058 immutable NORMAL/DEBUG emission and PLAT044 error tracing: **unchanged**.

No new guest-semantic decision is selected or required. Debugger stepping, suspension/resumption and tag-reparse checks are **regression tests during I111**, not conditions for repeating platform research. If a concrete framework incompatibility emerges, report a precise finding rather than silently weakening semantics.

`GITHUB021=PASS_FOR_SELECTED_DELTA`
`ONE_GENERATOR_GOAL=APPROVED`
`SELECTIVE_MANUAL_ROOT_TAG=APPROVED`
`PRODUCT_MODIFIED_BY_RATIFICATION=NO`
