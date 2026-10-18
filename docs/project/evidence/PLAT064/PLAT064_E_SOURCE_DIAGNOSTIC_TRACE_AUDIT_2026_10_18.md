# PLAT064-E — Source locations, diagnostics and trace integration after I111/I112

**Research date:** 2026-10-18. **Issue:** [PLAT064-E/#917](https://github.com/guillermomolina/protos/issues/917); **parent:** [PLAT064/#911](https://github.com/guillermomolina/protos/issues/911).
**Exact Protos evidence baseline:** [edcb758fadb0e749cd45711cd664044d0b6db6bd](https://github.com/guillermomolina/protos/commit/edcb758fadb0e749cd45711cd664044d0b6db6bd), 0.3.377-SNAPSHOT (I111 unified interpreter published; I112 terminal continuation already present).
**Type:** INVESTIGATION ONLY. No product code/spec/test, benchmark, new platform decision or implementation issue authorized by this research. Source-only review; no builds/tests run during the investigation. The maintainer separately reports local tests PASS and git diff --check clean, which are not represented as investigation execution.

## Executive outcome

PLAT064-E found **no justification for an interpreter-wide diagnostic redesign**. The earlier two-generator and absent-prolog baseline is historical after I111/I112. The single generated root, semantic-only tagging, lazy Truffle locations and suspended-only I112 terminal transport meet the intended Truffle integration shape while preserving Protos-only semantic projections. The bridge used for resumed control-flow transfers retains one bounded provenance question. It is a **source-level metadata-loss path**, not an executed reproduction of an incorrect guest trace.

**12 facilities: 8 KEEP, 3 EXISTING_OWNER, 1 PROPOSE_BOUNDED_Ixxx (proposal only).** No new work item has been allocated and no proposed code change has been approved. PLAT064-C/D remain separately actionable research children; this record closes only the E evidence question.

## Source provenance and comparator limitations

GraalVM/Truffle declared dependency **25.4.4.1.1**; inspected compatible 25.4-branch Oracle source snapshot [95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b](https://github.com/oracle/graal/tree/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle), **not** independently byte-matched to the published processor artifact. Comparator sources: [GraalPy Bytecode DSL](https://github.com/oracle/graalpython/blob/7e7182b1036d56f8e89ecc9c7a86cee292180aa6/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java), [Oracle SimpleLanguage](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.sl/src/com/oracle/truffle/sl/bytecode/SLBytecodeRootNode.java) and [TrufflePascal](https://github.com/d45h71/TrufflePascal/tree/fcae9007500369dd194c7e09c932fd4807d64fa8). A peer is an idiom comparator, never a substitute for Protos's normative semantics.

Primary product sources: [root](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java); [capture](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/host/tooling/ProtosDiagnosticTraceCapture.java); [signal](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/runtime/ProtosSignalException.java); [transfer](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeControlTransferException.java); [tail](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosTailCallContinuation.java); [task](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeTaskExecution.java); [tag](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeTagTreeNodeExports.java); [lower](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java); [structure](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/ProtosStructuredDispatchLowerer.java); [cli](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/test/java/com/guillermomolina/protos/execution/integration/diagnostics/ProtosCli008C1DiagnosticTraceTest.java); [i112](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI112TailContinuationTest.java); [debugger](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/test/java/com/guillermomolina/protos/execution/integration/interop/ProtosI026EDebuggerIntegrationTest.java).

Normative ownership retained: `spec/semantics/ERRORS.md`, `CALLABLES.md`, `EXECUTION_AND_CONTROL.md` and `spec/concurrency/FUTURES_AND_TASKS.md`. Prior ratifications PLAT036, PLAT041–043, PLAT045, PLAT062, PLAT065 and implemented I111/I112 are not reopened.

## Per-facility evidence and dispositions

### F01 — Error origin, identity, semantic stack

- **FACILITY:** Error origin, identity, semantic stack.
- **CURRENT_PROTOS_HEAD_AND_SOURCE:** ProtosSignalException retains Error identity and first semantic-root origin; capture reconstructs only semantic frames. Evidence: [signal](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/runtime/ProtosSignalException.java), [capture](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/host/tooling/ProtosDiagnosticTraceCapture.java), [root](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java).
- **TRUFFLE_API_VERSION_AND_DEFAULT:** TruffleStackTrace supplies physical unwound frames but not discarded semantic logical caller frames. Global pinned API caveat above.
- **BYTECODE_DSL_PEER_SOURCE_AND_BEHAVIOR:** GraalPy additionally tracks Python-specific exception tracebacks. Peer references in provenance.
- **CURRENT_PROTOS_USE_AND_REQUIRED_SEMANTICS:** CLI008 guest stack identity and semantic-only frame projection.
- **DIVERGENCE:** REQUIRED_SEMANTIC_CHOICE.
- **DISPOSITION:** KEEP.
- **JUSTIFICATION:** Replacing with raw Truffle physical frames changes the guest observable stack.
- **OWNER_AND_NEXT_ACTION:** Existing CLI008/PLAT044 owners; no patch.

### F02 — Handler selection before cleanup

- **FACILITY:** Handler selection before cleanup.
- **CURRENT_PROTOS_HEAD_AND_SOURCE:** Root interceptor chooses guest Error handling before crossed ensure unwind. Evidence: [root](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java), [structure](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/ProtosStructuredDispatchLowerer.java).
- **TRUFFLE_API_VERSION_AND_DEFAULT:** BytecodeRootNode interceptTruffleException permits preprocessing before handler dispatch. Global pinned API caveat above.
- **BYTECODE_DSL_PEER_SOURCE_AND_BEHAVIOR:** GraalPy's @EpilogExceptional handles Python traceback bookkeeping, not Protos Error.handle semantics. Peer references in provenance.
- **CURRENT_PROTOS_USE_AND_REQUIRED_SEMANTICS:** Guest Error.handle must select before ensure cleanup; see spec/semantics/ERRORS.md.
- **DIVERGENCE:** REQUIRED_SEMANTIC_CHOICE.
- **DISPOSITION:** KEEP.
- **JUSTIFICATION:** An exceptional epilog is not a drop-in replacement for scoped handler selection.
- **OWNER_AND_NEXT_ACTION:** Existing root and ERROR spec; keep.

### F03 — Single generated interpreter and semantic roots

- **FACILITY:** Single generated interpreter and semantic roots.
- **CURRENT_PROTOS_HEAD_AND_SOURCE:** I111 consolidated @GenerateBytecode; semanticActivationRoot distinguishes guest activations from C-prime utility roots. Evidence: [root](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java), [lower](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java).
- **TRUFFLE_API_VERSION_AND_DEFAULT:** Root tagging can be manual; automatic tagging would include auxiliary physical roots. Global pinned API caveat above.
- **BYTECODE_DSL_PEER_SOURCE_AND_BEHAVIOR:** SimpleLanguage/TrufflePascal use explicit root/body tagging suited to their different root topology. Peer references in provenance.
- **CURRENT_PROTOS_USE_AND_REQUIRED_SEMANTICS:** No fabricated guest trace frames for C-prime; semantic roots retain RootTag.
- **DIVERGENCE:** CONFIG_ONLY.
- **DISPOSITION:** EXISTING_OWNER.
- **JUSTIFICATION:** I111 changed the baseline; no second generator is live.
- **OWNER_AND_NEXT_ACTION:** I111/#919 completed; do not reopen.

### F04 — Uncached BCI and origin stability

- **FACILITY:** Uncached BCI and origin stability.
- **CURRENT_PROTOS_HEAD_AND_SOURCE:** Per-physical-frame BCI snapshot precedes lazy ensure/reparse during exceptional capture. Evidence: [capture](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/host/tooling/ProtosDiagnosticTraceCapture.java), [root](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java).
- **TRUFFLE_API_VERSION_AND_DEFAULT:** BytecodeLocation.update() translates locations after reparse; storeBytecodeIndexInFrame defaults false. Global pinned API caveat above.
- **BYTECODE_DSL_PEER_SOURCE_AND_BEHAVIOR:** GraalPy configures frame BCI storage true; does not prove Protos requires it. Peer references in provenance.
- **CURRENT_PROTOS_USE_AND_REQUIRED_SEMANTICS:** Accurate original throw site and per-Context location without unconditional hot-path BCI storage.
- **DIVERGENCE:** REQUIRED_SEMANTIC_CHOICE.
- **DISPOSITION:** KEEP.
- **JUSTIFICATION:** The observed consumer is exceptional capture; no evidence of lost accuracy caused by the default.
- **OWNER_AND_NEXT_ACTION:** PLAT064-A/#913 KEEP; no replacement.

### F05 — Lazy source and tag materialization

- **FACILITY:** Lazy source and tag materialization.
- **CURRENT_PROTOS_HEAD_AND_SOURCE:** Trace requests source/tag information only when needed; reparse uses supported source and tag structures. Evidence: [capture](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/host/tooling/ProtosDiagnosticTraceCapture.java), [tag](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeTagTreeNodeExports.java), [lower](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java).
- **TRUFFLE_API_VERSION_AND_DEFAULT:** BytecodeRootNodes.ensureSourceInformation(), ensureComplete() and BytecodeLocation.update() are provided. Global pinned API caveat above.
- **BYTECODE_DSL_PEER_SOURCE_AND_BEHAVIOR:** SimpleLanguage emits source sections; TrufflePascal uses tags for source/tooling integration. Peer references in provenance.
- **CURRENT_PROTOS_USE_AND_REQUIRED_SEMANTICS:** NORMAL does not pay DEBUG materialization cost, yet CLI008 can resolve source positions.
- **DIVERGENCE:** REQUIRED_SEMANTIC_CHOICE.
- **DISPOSITION:** KEEP.
- **JUSTIFICATION:** Global eager COMPLETE or tags would violate pay-as-you-grow with no demonstrated necessity.
- **OWNER_AND_NEXT_ACTION:** Existing source/DEBUG consumers.

### F06 — Inline semantic callback frames

- **FACILITY:** Inline semantic callback frames.
- **CURRENT_PROTOS_HEAD_AND_SOURCE:** Capture reconstructs nested semantic inline callback regions from tagged source extents. Evidence: [capture](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/host/tooling/ProtosDiagnosticTraceCapture.java), [lower](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java).
- **TRUFFLE_API_VERSION_AND_DEFAULT:** TagTree describes nested tagged regions, not a historical stack of already-returned activations. Global pinned API caveat above.
- **BYTECODE_DSL_PEER_SOURCE_AND_BEHAVIOR:** Manual tags in SL/TrufflePascal demonstrate instrumentation usage, not this Protos inline lifetime. Peer references in provenance.
- **CURRENT_PROTOS_USE_AND_REQUIRED_SEMANTICS:** Inline callbacks must appear as logical semantic frames, once only.
- **DIVERGENCE:** REQUIRED_SEMANTIC_CHOICE.
- **DISPOSITION:** KEEP.
- **JUSTIFICATION:** Physical-only stack frames omit logical inline regions.
- **OWNER_AND_NEXT_ACTION:** CLI008 and inline callback tests.

### F07 — I112 resumed terminal Error callers

- **FACILITY:** I112 resumed terminal Error callers.
- **CURRENT_PROTOS_HEAD_AND_SOURCE:** ProtosTailCallContinuation retains pending state only upon true suspension and records resumed terminal caller for AbstractTruffleException. Evidence: [tail](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosTailCallContinuation.java), [capture](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/host/tooling/ProtosDiagnosticTraceCapture.java), [i112](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI112TailContinuationTest.java).
- **TRUFFLE_API_VERSION_AND_DEFAULT:** ContinuationResult resumes retained frames; it does not recreate the caller root which has already returned. Global pinned API caveat above.
- **BYTECODE_DSL_PEER_SOURCE_AND_BEHAVIOR:** GraalPy has different callable/suspend stack policy. Peer references in provenance.
- **CURRENT_PROTOS_USE_AND_REQUIRED_SEMANTICS:** Correct source caller of resumed errors without always allocating a continuation or RootTask.
- **DIVERGENCE:** REQUIRED_SEMANTIC_CHOICE.
- **DISPOSITION:** EXISTING_OWNER.
- **JUSTIFICATION:** I112 already owns and tests the lifecycle; no duplicate implementation.
- **OWNER_AND_NEXT_ACTION:** I112 published; existing coverage.

### F08 — Resumed terminal control-transfer bridge provenance

- **FACILITY:** Resumed terminal control-transfer bridge provenance.
- **CURRENT_PROTOS_HEAD_AND_SOURCE:** resumeOwned catches the existing bridge but passes only bridged.transfer() to completeTransfer; escaping transfers receive a newly allocated bridge. Evidence: [tail](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosTailCallContinuation.java), [transfer](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeControlTransferException.java), [task](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeTaskExecution.java), [capture](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/host/tooling/ProtosDiagnosticTraceCapture.java).
- **TRUFFLE_API_VERSION_AND_DEFAULT:** Pinned Truffle 25.4 has no selected unwindExceptions annotation and internal CFE transfers must cooperate with generated unwind handling. Global pinned API caveat above.
- **BYTECODE_DSL_PEER_SOURCE_AND_BEHAVIOR:** Peers' exception schemes cannot substitute for the pinned 25.4 contract. Peer references in provenance.
- **CURRENT_PROTOS_USE_AND_REQUIRED_SEMANTICS:** InvalidReturn at Task boundary can depend on preserved origin bytecode/BCI; resumption and ensure supersession must remain exact.
- **DIVERGENCE:** FRAMEWORK_MISALIGNMENT.
- **DISPOSITION:** PROPOSE_BOUNDED_Ixxx (not authorized).
- **JUSTIFICATION:** Code-level loss of *bridge metadata* is demonstrated, but no guest-visible regression is claimed without a targeted test; original suggestion to copy metadata is superseded as an immediate next action by upstream consideration.
- **OWNER_AND_NEXT_ACTION:** DEFER new issue until owner selects a bounded correction versus future Truffle 25.5 unwindExceptions adoption; no I number allocated.

### F09 — DEBUG scopes and event identity

- **FACILITY:** DEBUG scopes and event identity.
- **CURRENT_PROTOS_HEAD_AND_SOURCE:** NORMAL and DEBUG paths intentionally materialize source/tags at different points, with debugger integration tests. Evidence: [root](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java), [tag](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeTagTreeNodeExports.java), [capture](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/host/tooling/ProtosDiagnosticTraceCapture.java).
- **TRUFFLE_API_VERSION_AND_DEFAULT:** StandardTags/TagTree and debugger session instrumentation are supported. Global pinned API caveat above.
- **BYTECODE_DSL_PEER_SOURCE_AND_BEHAVIOR:** SL/TrufflePascal expose own tag/scoping choices. Peer references in provenance.
- **CURRENT_PROTOS_USE_AND_REQUIRED_SEMANTICS:** Debugger scope correct in DEBUG, normal CLI Error trace still exists independently.
- **DIVERGENCE:** REQUIRED_SEMANTIC_CHOICE.
- **DISPOSITION:** KEEP.
- **JUSTIFICATION:** Eager DEBUG setup in NORMAL is not required by API or tests.
- **OWNER_AND_NEXT_ACTION:** Existing debugger coverage; no action.

### F10 — Source sections, roots and call targets

- **FACILITY:** Source sections, roots and call targets.
- **CURRENT_PROTOS_HEAD_AND_SOURCE:** Canonical lowering emits tagged semantic root/source regions while C-prime remains physically executable but semantically hidden. Evidence: [lower](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java), [root](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java).
- **TRUFFLE_API_VERSION_AND_DEFAULT:** SourceSection, BytecodeRootNodes group reparse and TagTree are separate facilities. Global pinned API caveat above.
- **BYTECODE_DSL_PEER_SOURCE_AND_BEHAVIOR:** SimpleLanguage/TrufflePascal use source sections in their roots. Peer references in provenance.
- **CURRENT_PROTOS_USE_AND_REQUIRED_SEMANTICS:** Source provenance stable across reparse and group-level instrumentation.
- **DIVERGENCE:** CONFIG_ONLY.
- **DISPOSITION:** EXISTING_OWNER.
- **JUSTIFICATION:** I111 consolidated physical generator and retained logical root classifications.
- **OWNER_AND_NEXT_ACTION:** I111/#919 completed.

### F11 — Context-local authority

- **FACILITY:** Context-local authority.
- **CURRENT_PROTOS_HEAD_AND_SOURCE:** Per-Context capture ownership follows current source/activation and does not pool fake foreign-context guest frames. Evidence: [capture](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/host/tooling/ProtosDiagnosticTraceCapture.java), [cli](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/test/java/com/guillermomolina/protos/execution/integration/diagnostics/ProtosCli008C1DiagnosticTraceTest.java).
- **TRUFFLE_API_VERSION_AND_DEFAULT:** BytecodeRootNodes reparse does not supply missing cross-Context semantic authority. Global pinned API caveat above.
- **BYTECODE_DSL_PEER_SOURCE_AND_BEHAVIOR:** Peers have language-specific context boundaries. Peer references in provenance.
- **CURRENT_PROTOS_USE_AND_REQUIRED_SEMANTICS:** No cross-Context trace contamination or retained live Truffle objects.
- **DIVERGENCE:** REQUIRED_SEMANTIC_CHOICE.
- **DISPOSITION:** KEEP.
- **JUSTIFICATION:** Context isolation and bounded inert record are intentional, not missing API adoption.
- **OWNER_AND_NEXT_ACTION:** CLI008 cross-Context regressions.

### F12 — Inert bounded CLI diagnostic projection

- **FACILITY:** Inert bounded CLI diagnostic projection.
- **CURRENT_PROTOS_HEAD_AND_SOURCE:** Captured public trace stores inert bounded summaries, not live Frames/CallTargets/BytecodeNodes; CLI formatting consumes them. Evidence: [capture](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/host/tooling/ProtosDiagnosticTraceCapture.java), [cli](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/test/java/com/guillermomolina/protos/execution/integration/diagnostics/ProtosCli008C1DiagnosticTraceTest.java).
- **TRUFFLE_API_VERSION_AND_DEFAULT:** TruffleStackTrace is lazy and frame-based, not a stable immutable guest-facing diagnostic schema. Global pinned API caveat above.
- **BYTECODE_DSL_PEER_SOURCE_AND_BEHAVIOR:** GraalPy's Python stack API is tailored to Python exceptions. Peer references in provenance.
- **CURRENT_PROTOS_USE_AND_REQUIRED_SEMANTICS:** Fixed-size projections (64 entries), safe after root exit and across host boundaries.
- **DIVERGENCE:** REQUIRED_SEMANTIC_CHOICE.
- **DISPOSITION:** KEEP.
- **JUSTIFICATION:** Direct exposure of Truffle stacks introduces lifetime and semantic framing changes.
- **OWNER_AND_NEXT_ACTION:** CLI008 maintained; no implementation.

## Adversarial boundaries and negative findings

- **Error.handle / ensure:** handler selection precedes crossed cleanup; DSL exceptional epilog cannot safely replace dynamic handler tokens.
- **C-prime physical roots:** are not automatically semantic stack frames; tagging every generated physical root would be a regression.
- **No suspension:** I112 does not manufacture terminal-continuation transport on ordinary synchronous return; only true suspension retains state.
- **Resumed Error:** caller is physically gone, but logical location can be recorded on the actual Error path.
- **Resumed nonlocal return/cancellation:** original CFE identity and ownership remain distinct from bridge metadata. A code path re-creates the wrapper bridge and discards its origin metadata. Prove guest-visible InvalidReturn trace behavior before calling this a diagnosed regression.
- **Nested inline callbacks:** TagTree supplies source extent evidence, not the full logical activation stack; active-tag extent checks may deserve dedicated falsification on early exits but no defect is demonstrated at current baseline.
- **UNCACHED/reparse:** diagnostics cache locations and update after source/tag reparse; neither frame-wide BCI storage nor mandatory complete instrumentation follows from this evidence.
- **Context/Native:** bounded inert traces avoid exporting live runtime frames. This was not a Native compiler/Tier-2 investigation.

## New upstream facility — oracle/graal#14434, integrated 2026-10-14

See [PR #14434](https://github.com/oracle/graal/pull/14434) / [merge e71338c](https://github.com/oracle/graal/commit/e71338c70bd9a724b5edc25accf5dffcaaaf8305), `[GR-79562] Support configurable Bytecode DSL unwind exceptions`. `GenerateBytecode.unwindExceptions` is marked **@since 25.5** in the upstream API, not part of the declared `25.4.4.1.1` baseline.

An explicitly configured Throwable/ControlFlowException subtype executes `TryFinally` while bypassing guest catch, otherwise, interceptors and `@EpilogExceptional`; it preserves the **same exception instance** through yielding cleanup, subject to normal cleanup supersession; instrumentation exceptional notifications remain. `ThreadDeath` has reserved handling. The upstream regression corpus tests cached/uncached, nested finally, yielding cleanup, replacement transfer/guest exception/return and instrumentation.

**Impact:** this makes an upstream-native replacement for some `ProtosBytecodeControlTransferException` bridging plausible after the maintainer integrates the facility into the version Protos actually builds against. It **does not** prove blanket bridge elimination, compatibility with Protos-specific Error.handle selection, cancellation state, ReturnHome ownership or CLI008 diagnostics, or availability in the declared version today. The maintainer indicated an upstream integration is available; no artifact/branch/test verification of that integration was carried out here.

**Disposition:** leave `unwindExceptions` as a future bounded adoption question, not an authorized correction under PLAT064-E. Do not open Ixxx or implement/copy bridge metadata just because code-level provenance loss was identified. Obtain exact owner selection before an architectural replacement. This does not invalidate the 8 KEEP findings.

## Regression evidence and actual execution status

- [cli](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/test/java/com/guillermomolina/protos/execution/integration/diagnostics/ProtosCli008C1DiagnosticTraceTest.java) includes same Error across occurrences, semantic frame filtering, nested inline callbacks, resumed calls, ensure supersession, bounded projection and cross-Context testing.
- [i112](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI112TailContinuationTest.java) covers resumed Error, nonlocal return, cancellation, reparse, NORMAL/DEBUG, uncached, and cross-Context terminal continuation behavior.
- Current source-only investigation executed **no** builds, tests, benchmarks, validation scripts or repository-product edits. The user reported local tests green and `git diff --check` clean before this project-record publication. No new docs diff-check was run by the investigating agent.

## Coordination and next work

- PLAT064-E/#917: publish this immutable source-backed evidence; research may close with a final closure comment only after exact docs revision is read back. No owner approval of a new design is claimed.
- PLAT064/#911: remains open while PLAT064-C/#915 and PLAT064-D/#916 remain individually unresolved; reconcile their two-generator allocation-era language to the single-root I111 HEAD.
- **Next proposed slice:** PLAT064-C/#915, **INVESTIGATION ONLY**, current single-root effective Bytecode DSL operation surface and surviving `@OperationProxy` opportunities; no product edit or human-executor command batch. PLAT064-D remains an independent investigation.
- No implementation/patch/release issue was created. Future `unwindExceptions` use should be assessed separately against the actual upstream-enabled build and exact semantics, with explicit project-owner authorization if architectural behavior changes.

## Completion identifiers

PROTOS_REVISION=edcb758fadb0e749cd45711cd664044d0b6db6bd
TRUFFLE_PINNED_SOURCE=95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b
UPSTREAM_UNWIND_PR=oracle/graal#14434
RESULT=RESEARCH_COMPLETE_NO_PRODUCT_CHANGE
CANDIDATE_CONTROL_TRANSFER_FIX=UNAPPROVED
NEXT_WORK=PLAT064-C/#915
