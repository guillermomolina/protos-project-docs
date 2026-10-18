# PLAT064-C — Effective Bytecode DSL operation surface after I111 (source-only evidence)

**Date:** 2026-10-18. **Issue:** [PLAT064-C/#915](https://github.com/guillermomolina/protos/issues/915); **parent:** [PLAT064/#911](https://github.com/guillermomolina/protos/issues/911).
**Type:** INVESTIGATION; no design selection or implementation authorization.
**Product snapshot / current HEAD checked during research:** [`edcb758fadb0e749cd45711cd664044d0b6db6bd`](https://github.com/guillermomolina/protos/commit/edcb758fadb0e749cd45711cd664044d0b6db6bd) (`0.3.377-SNAPSHOT`).
**Evidence owner:** `guillermomolina/protos-project-docs:docs/project/evidence/PLAT064/`.
**Framework dependency:** `org.graalvm.truffle` 25.4.4.1.1 (product `pom.xml`).
**Upstream inspection:** `oracle/graal@95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b` (25.4 branch **source snapshot**, *not* verified binary equivalence to the released annotation processor).
**Peer revisions:** `oracle/graalpython@7e7182b1036d56f8e89ecc9c7a86cee292180aa6`, `oracle/graal@95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b` (SimpleLanguage), `d45h71/TrufflePascal@fcae9007500369dd194c7e09c932fd4807d64fa8`.
**Previous related source audit:** [PLAT064-E at pinned docs commit](https://github.com/guillermomolina/protos-project-docs/blob/84d80ddb1733cbc3c42433cd998602cb0543c3d5/docs/project/evidence/PLAT064/PLAT064_E_SOURCE_DIAGNOSTIC_TRACE_AUDIT_2026_10_18.md).

## Executive result and owner gate

One `@GenerateBytecode` root remains after [I111/#919](https://github.com/guillermomolina/protos/issues/919): [`ProtosBytecodeRootNode.java`](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java). The historical 235/175/76 two-root counts are obsolete. I112 already owns `@Prolog`, `@EpilogReturn`, terminal `InvokeClosure*Tail`, and suspended-only continuation transport.

The current **declaration** census is **347 `@Operation` + 9 `@OperationProxy` = 356**. This is neither the number of generated quickened variants nor the number of instructions in guest programs. The source-emitter mapping examined yields **324 explicit `emitX`/`beginX` consumers**, **one additional Java-only required method owner** (`MaterializeReturnCarrier` from the epilogue), **one test/contractual direct Java consumer** (`AdmitsStraightLineSourceCall`), and **30 declarations in five `*Each` families without a demonstrated builder emission**. Classification: **325 REQUIRED; 1 TEST_ONLY_OR_CONTRACTUAL; 30 NO_CONSUMER_DEMONSTRATED; 0 assigned INCONCLUSIVE in this bounded census**. `NO_CONSUMER_DEMONSTRATED` does **not** assert that the backing Java protocol is dead or that elimination is safe.

**Owner response:** decision deferred on 2026-10-18. **No candidate accepted, no Ixxx allocated, no root redesign approved, no product mutation**. Keep #915 and #911 open for coordination until disposition is explicitly selected or deferred further. Propose an optional single bounded future review of 32 registrations, but this is *not* implementation authorization.

## Sources and workflow

Primary paths pinned to product HEAD:

- C: [`src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java`](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java)
- S: [`src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/ProtosStructuredDispatchLowerer.java`](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/ProtosStructuredDispatchLowerer.java)
- Task: [`src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosTaskCPrimeEntryExecution.java`](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosTaskCPrimeEntryExecution.java)
- Buffered reader: [`src/main/java/com/guillermomolina/protos/execution/host/resources/ProtosBufferedByteReaderCPrimeExecution.java`](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/host/resources/ProtosBufferedByteReaderCPrimeExecution.java)
- Buffered writer: [`src/main/java/com/guillermomolina/protos/execution/host/resources/ProtosBufferedByteWriterCPrimeExecution.java`](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/host/resources/ProtosBufferedByteWriterCPrimeExecution.java)
- IO release: [`src/main/java/com/guillermomolina/protos/execution/host/resources/ProtosIoReleaseCPrimeExecution.java`](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/host/resources/ProtosIoReleaseCPrimeExecution.java)
- Text reader: [`src/main/java/com/guillermomolina/protos/execution/host/resources/ProtosTextReaderCPrimeExecution.java`](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/host/resources/ProtosTextReaderCPrimeExecution.java)
- Text writer: [`src/main/java/com/guillermomolina/protos/execution/host/resources/ProtosTextWriterCPrimeExecution.java`](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/host/resources/ProtosTextWriterCPrimeExecution.java)
- Prepared each: [`src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosStructuredEachCalls.java`](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosStructuredEachCalls.java)
- Prepared call: [`src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosPreparedCalls.java`](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosPreparedCalls.java)
- Generated baseline: [`tools/java_generated_bytecode_bci_pe_baseline.json`](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/tools/java_generated_bytecode_bci_pe_baseline.json)
- I101C independent: [`src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI101CIndependentBuilderReferenceTest.java`](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI101CIndependentBuilderReferenceTest.java)
- PERF040-E1: [`src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosPerf040E1NumericOperationReconciliationTest.java`](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosPerf040E1NumericOperationReconciliationTest.java)
- I112: [`src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI112TailContinuationTest.java`](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI112TailContinuationTest.java)
- PERF032-G7: [`src/test/java/com/guillermomolina/protos/execution/compiler/architecture/ProtosPerf032G7MinimalDirectHeaderTest.java`](https://github.com/guillermomolina/protos/blob/edcb758fadb0e749cd45711cd664044d0b6db6bd/src/test/java/com/guillermomolina/protos/execution/compiler/architecture/ProtosPerf032G7MinimalDirectHeaderTest.java)

Comparison/API sources:

- Truffle source snapshot: [`GenerateBytecode`](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/GenerateBytecode.java), [`Operation`](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/Operation.java), [`OperationProxy`](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/OperationProxy.java), [parser](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.dsl.processor/src/com/oracle/truffle/dsl/processor/bytecode/parser/BytecodeDSLParser.java), [generator entry](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.dsl.processor/src/com/oracle/truffle/dsl/processor/bytecode/generator/BytecodeDSLCodeGenerator.java).
- [GraalPy DSL root](https://github.com/oracle/graalpython/blob/7e7182b1036d56f8e89ecc9c7a86cee292180aa6/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java), [Oracle SimpleLanguage DSL root](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.sl/src/com/oracle/truffle/sl/bytecode/SLBytecodeRootNode.java), [TrufflePascal DSL root](https://github.com/d45h71/TrufflePascal/blob/fcae9007500369dd194c7e09c932fd4807d64fa8/language/src/main/java/org/trufflepascal/bytecode/PascalBytecodeRoot.java). These are comparator idioms, not Protos semantics. GraalJS/TruffleRuby AST roots are not counted as generated DSL peers.

Research reviewed canonical Protos policy (`AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`) and the normative owners `spec/semantics/CALLABLES.md`, `ERRORS.md`, `EXECUTION_AND_CONTROL.md`, `VALUES_AND_COLLECTIONS.md`, `spec/concurrency/FUTURES_AND_TASKS.md`. The source review followed canonical, structured, Task/IO C-prime and selected independent Builder tests, plus targeted code search for apparent absences. **No product code was run; no tests, compilation, benchmarks, Native Image, or generated-source regeneration were performed.** This dossier contains source-observation classifications rather than runtime or binary proofs.

### Consumer-map method and its limits

- `C`: `CanonicalToBytecodeLowerer`; `S`: `ProtosStructuredDispatchLowerer`; `IO`: five independently authored buffered byte reader/writer, IO release, text reader/writer C-prime builders; Task: `ProtosTaskCPrimeEntryExecution`; independent test Builder sources: `ProtosI101CIndependentBuilderReferenceTest` and additional numeric / terminal tests.
- `declaration -> emitter` means a literal `emit<Operation>` or `begin<Operation>` at the inspected commit. The generated specialization is defined by the annotated Java `@Specialization` methods, guards, custom operands, quickening and boxing carriers. **Not regenerated or revalidated** at this investigation phase.
- A required domain Java method can be used independently of the generated Builder instruction, e.g. `@EpilogReturn -> MaterializeReturnCarrier.perform`. A test can call a nested class method without emitting the instruction, e.g. `AdmitsStraightLineSourceCall.perform`.
- Classes `ProtosStructuredEachCalls.PreparedArrayEachCall`, `PreparedBytesEachCall`, `PreparedEnvironmentEachCall`, `PreparedIdentityMapEachCall`, `PreparedMapEachCall` are retained by the Java prepared-call protocols; a missing `emitX` does not license deleting those classes.
- `tools/java_generated_bytecode_bci_pe_baseline.json` contains generated topology entries naming the apparently un-emitted families. Such entries are **not** guest emission, but are a build/validation dependency for any later removal. No unchanged-output guarantee is claimed.
- Source-only negative conclusions are scoped to the inspected production builders and targeted repository code-search results; they do not constitute a formal proof against reflection, future builders, generated-Java tooling or all reachable paths.

### Distribution across observed producers

| Explicit producer set (exclusive) | Count |
|---|---:|
| Canonical only (including Canonical + independent test reference) | 170 |
| Structured only | 99 |
| Canonical + Structured | 14 |
| Canonical + Structured + IO builders | 3 |
| Structured + IO builders | 2 |
| Five IO C-prime builder families | 36 |
| No `emitX`/`beginX` found in examined producers | 32 |
| **Total** | **356** |

Within the 32: `MaterializeReturnCarrier` (direct epilogue Java consumer), `AdmitsStraightLineSourceCall` (direct test consumer), and 30 `*Each` operations.

## Exhaustive declaration inventory

Every name below is present once. Intervals are **root-source line ranges**, not bytecode offsets; groups partition the 356 declarations. `R` = REQUIRED (producer emitted or direct production method), `T` = TEST_ONLY_OR_CONTRACTUAL, `N` = NO_CONSUMER_DEMONSTRATED. Names show generator operation names (proxy override names for the nine proxies). The producer column indicates owning producer(s) for the *family*, not a claim that each member appears in every listed producer.

| Family | Lines | Count | Class | Producer |
|---|---:|---:|---|---|
| Proxies | 158–166 | 9 | R | C |
| Activation / binding | 520–750 | 11 | R | C |
| Lexical / captures / writes | 763–2666 | 52 | R | C |
| Members / objects / map construction / args | 2683–3412 | 28 | R | C |
| Closure calls / sends | 3439–7754 | 24 | R | C/S |
| Boolean / inline / control | 7836–8888 | 53 | R except T:AdmitsStraightLineSourceCall | C/S |
| Direct send / numeric / terminal continuation | 8974–10773 | 18 | R | C/S/IO/Java epilogue |
| CaseOf | 10806–10879 | 6 | R | S |
| Object call | 10888–10944 | 4 | R | S |
| Import | 10954–11010 | 4 | R | S |
| Ensure | 11020–11074 | 4 | R | S |
| While / local each | 11082–11137 | 3 | R | S |
| Array each | 11229–11300 | 6 | N | not found in inspected emitters |
| Array match | 11308–11382 | 6 | R | S |
| Bytes each | 11390–11461 | 6 | N | not found in inspected emitters |
| Environment each | 11469–11540 | 6 | N | not found in inspected emitters |
| Foreign each | 11548–11626 | 6 | R | S |
| IdentityMap atIfAbsent | 11634–11709 | 6 | R | S |
| IdentityMap each | 11718–11789 | 6 | N | not found in inspected emitters |
| Map match | 11797–11948 | 15 | R | S |
| Map read lookup | 11956–12086 | 13 | R | S |
| Map atPut | 12094–12196 | 10 | R | S |
| Map remove | 12204–12307 | 10 | R | S |
| Map each | 12315–12385 | 6 | N | not found in inspected emitters |
| Error / cancellation | 12393–12499 | 8 | R | S |
| IO C-prime | 12507–13003 | 36 | R | IO builders |
| **Total** | | **356** | **325 R / 1 T / 30 N** | |

#### Proxies (`158–166`; 9 declarations; R)

`TryCompactCanonicalBooleanOne`, `TryPrimitiveNumericAdd`, `TryPrimitiveNumericSubtract`, `TryPrimitiveNumericMultiply`, `TryPrimitiveNumericDivide`, `TryPrimitiveNumericLess`, `TryPrimitiveNumericLessEqual`, `TryPrimitiveNumericGreater`, `TryPrimitiveNumericGreaterEqual`.

#### Activation / binding (`520–750`; 11 declarations; R)

`CurrentActivation`, `CurrentCallerReference`, `HasClosureArgument`, `LoadClosureArgument`, `HasFrameClosureArgument`, `LoadFrameClosureArgument`, `BindClosureParameter`, `BindClosureRest`, `BindClosureIndexedParameter`, `BindClosureIndexedRest`, `Lookup`.

#### Lexical / captures / writes (`763–2666`; 52 declarations; R)

`InstallFrameLexicalAuthority`, `BindRequiredFrameParameter`, `BindClosureFrameParameter`, `BindUnaryFrameParameter`, `MaterializeInlineCallbackActivation`, `LoadInlineClosureArgument`, `BindInlineClosureFrameParameter`, `BindClosureFrameRest`, `IsCompactLocalFrame`, `CreateCurrentFrameLocal`, `CreateInlineCurrentFrameLocal`, `ObserveMultipleCreatePrefix`, `ObserveInlineMultipleCreatePrefix`, `ObservedMultipleCreateValue`, `ReadFrameLocal`, `ReadRootFrameLocal`, `ReadInlineFrameLocal`, `ReadCapturedFrameLocalAtRoot`, `SelectCapturedMaterializedOwnerFrameAtRoot`, `ReadCapturedFallbackAtRoot`, `ReadCapturedFrameLocal`, `SelectCapturedMaterializedOwnerFrame`, `IsCapturedOwnerFrameSelected`, `ReadCapturedFallback`, `ResolveCapturedWritableLexicalTarget`, `AssignCapturedFrameLocal`, `ResolveCapturedMaterializedWritableLexicalTarget`, `AssignCapturedMaterializedLocal`, `ReadInlineCapturedFrameLocal`, `SelectInlineCapturedMaterializedOwnerFrame`, `ReadInlineCapturedFallback`, `ResolveInlineCapturedWritableLexicalTarget`, `AssignInlineCapturedFrameLocal`, `ResolveInlineCapturedMaterializedWritableLexicalTarget`, `AssignInlineCapturedMaterializedLocal`, `LoadInlineCallbackReceiver`, `ReadInlineMember`, `ResolveWritableLexicalTarget`, `AssignResolvedLexicalTarget`, `ResolveCurrentFrameLocalWriteTarget`, `AssignCurrentFrameLocal`, `ResolveRootFrameLocalWriteTarget`, `AssignRootFrameLocal`, `ResolveInlineFrameLocalWriteTarget`, `AssignInlineFrameLocal`, `RequireObjectMutationTarget`, `CreateCurrentLocalSlot`, `CreateCurrentIndexedLocalSlot`, `CreateLocalSlot`, `MultipleCreateLocalSlots`, `AssignLocalSlot`, `DiscardValue`.

#### Members / objects / map construction / args (`2683–3412`; 28 declarations; R)

`ReadMemberAtRoot`, `ReadMember`, `Identity`, `NotIdentity`, `ComplementEqualityResult`, `LoadIntrinsic`, `RaiseNonLocalReturn`, `LargeIntegerLiteral`, `MaterializeClosure`, `MaterializeCurrentClosure`, `NewConstructedObject`, `NewObjectConstructionActivation`, `ComposeLocalSlots`, `PrepareMapConstructionFactoryCall`, `FinishMapConstructionFactoryCall`, `PrepareMapInitialDefinition`, `EnterMapInitialDefinitionComparison`, `LeaveMapInitialDefinitionComparison`, `PrepareMapInitialDefinitionHashCall`, `AcceptMapInitialDefinitionHashResult`, `MapInitialDefinitionNeedsEquality`, `PrepareMapInitialDefinitionEqualityCall`, `AcceptMapInitialDefinitionEqualityResult`, `FinishMapInitialDefinition`, `CreateSuppliedArgumentVector`, `AppendSuppliedArgument`, `AppendSpreadSuppliedArgument`, `AppendInlineSpreadSuppliedArgument`.

#### Closure calls / sends (`3439–7754`; 24 declarations; R)

`PrepareClosureCall`, `PrepareClosureCallArguments`, `PrepareClosureCallOne`, `PrepareDefaultClosureCallArguments`, `PrepareInlineSendArguments`, `PrepareInlineSendZero`, `PrepareInlineSendOne`, `InvokeInlineClosureZero`, `PrepareInlineClosureCall`, `PrepareInlineClosureCallArguments`, `InvokeInlineClosureOne`, `InvokeInlineClosureArguments`, `PrepareInlineClosureCallOne`, `CollectClosureCallArguments`, `SnapshotClosureCallArguments`, `InvokeClosureArguments`, `PrepareClosureCallVector`, `PrepareSendArguments`, `PrepareSendZero`, `PrepareSendOne`, `PrepareSendVector`, `PrepareSuperSendArguments`, `PrepareSuperSendVector`, `RequiresStructuredDispatch`.

#### Boolean / inline / control (`7836–8888`; 53 declarations; R except T:AdmitsStraightLineSourceCall)

`IsCanonicalIfTrueTrue`, `IsCanonicalBooleanFalseClass`, `RequireCanonicalBooleanControlResult`, `CanonicalIfTrueNonTrueResult`, `PrepareCanonicalIfTrueCallback`, `TryCompactCanonicalBooleanVector`, `TryCompactCanonicalBooleanInlineOne`, `TryCompactCanonicalBooleanInlineVector`, `PrepareCanonicalIfTrueVectorCallback`, `PrepareCanonicalIfTrueInlineCallback`, `PrepareCanonicalIfTrueInlineVectorCallback`, `IsCompactCanonicalBooleanMiss`, `IsCompactCanonicalBooleanNoCallback`, `IsNativeIfTrueInlineCallback`, `CompleteIfTrueSiteOuter`, `AdmitsNativeIfTrueInline`, `IsStructuredBooleanCall`, `PrepareStructuredBooleanCall`, `StructuredBooleanHasCallback`, `PrepareStructuredBooleanCallbackCall`, `PrepareInlineStructuredBooleanCallbackCall`, `StructuredBooleanImmediateResult`, `FinishStructuredBooleanCallback`, `AdmitsInlineLiteralCallback`, `PrepareInlineClosureLiteralCall`, `AdmitsInlineClosureLiteralCall`, `AdmitsInlineLiteralWhile`, `PrepareStructuredWhileCall`, `PrepareStructuredWhileConditionCall`, `PrepareStructuredWhileBodyCall`, `PrepareInlineStructuredWhileConditionCall`, `PrepareInlineStructuredWhileBodyCall`, `StructuredWhileCondition`, `StructuredWhileRetainBodyResult`, `StructuredWhileResult`, `AdmitsInlineLiteralWhileCondition`, `AdmitsInlineLiteralWhileBody`, `AdmitsInlineLiteralIndexedEach`, `PrepareStructuredIndexedEachCall`, `AdmitsInlineLiteralTwoParameterEach`, `PrepareStructuredTwoParameterEachCall`, `StructuredLocalEachHasNext`, `PrepareStructuredLocalEachChildCall`, `PrepareInlineStructuredLocalEachChildCall`, `AdvanceStructuredLocalEach`, `FinishStructuredLocalEach`, `AdmitsInlineLiteralLocalEachChild`, `LoadInlineCallbackActivation`, `LoadInlineLiteralFallbackCall`, `CompleteClosureCall`, `EnterNestedStructuredDispatch`, `AdmitsStraightLineSourceCall`, `EnterStraightLineSourceCall`.

#### Direct send / numeric / terminal continuation (`8974–10773`; 18 declarations; R)

`TryDirectSendZero`, `MaterializePrimitiveNumericCarrier`, `MaterializeReturnCarrier`, `IsPrimitiveNumericSendMiss`, `TryDirectSendOne`, `InvokeClosureZero`, `InvokeClosureOne`, `InvokeClosureOneTail`, `InvokeClosureZeroTail`, `InvokeClosureArgumentsTail`, `IsPreparedClosureInvocation`, `IsDirectSendMiss`, `EnterClosureCall`, `IsContinuation`, `IsContinuationForCall`, `ResumeContinuation`, `FinishClosureCall`, `FinishCPrimeClosureCall`.

#### CaseOf (`10806–10879`; 6 declarations; R)

`IsStructuredCaseOfCall`, `PrepareStructuredCaseOfCall`, `StructuredCaseOfNeedsMatcher`, `PrepareStructuredCaseOfMatcherCall`, `AcceptStructuredCaseOfMatcherOutcome`, `PrepareStructuredCaseOfActionCall`.

#### Object call (`10888–10944`; 4 declarations; R)

`IsStructuredObjectCall`, `PrepareStructuredObjectCall`, `LoadStructuredObjectCallChild`, `FinishStructuredObjectCall`.

#### Import (`10954–11010`; 4 declarations; R)

`IsStructuredImportCall`, `PrepareStructuredImportCall`, `LoadStructuredImportCallChild`, `FinishStructuredImportCall`.

#### Ensure (`11020–11074`; 4 declarations; R)

`IsStructuredEnsureCall`, `PrepareStructuredEnsureCall`, `LoadStructuredEnsureBodyCall`, `LoadStructuredEnsureCleanupCall`.

#### While / local each (`11082–11137`; 3 declarations; R)

`IsStructuredWhileCall`, `IsStructuredLocalEachCall`, `PrepareStructuredLocalEachCall`.

#### Array each (`11229–11300`; 6 declarations; N)

`IsStructuredArrayEachCall`, `PrepareStructuredArrayEachCall`, `StructuredArrayEachHasNext`, `PrepareStructuredArrayEachElementCall`, `AdvanceStructuredArrayEach`, `FinishStructuredArrayEach`.

#### Array match (`11308–11382`; 6 declarations; R)

`IsStructuredArrayMatchCall`, `PrepareStructuredArrayMatchCall`, `StructuredArrayMatchHasNext`, `PrepareStructuredArrayMatchElementCall`, `AcceptStructuredArrayMatchOutcome`, `FinishStructuredArrayMatch`.

#### Bytes each (`11390–11461`; 6 declarations; N)

`IsStructuredBytesEachCall`, `PrepareStructuredBytesEachCall`, `StructuredBytesEachHasNext`, `PrepareStructuredBytesEachElementCall`, `AdvanceStructuredBytesEach`, `FinishStructuredBytesEach`.

#### Environment each (`11469–11540`; 6 declarations; N)

`IsStructuredEnvironmentEachCall`, `PrepareStructuredEnvironmentEachCall`, `StructuredEnvironmentEachHasNext`, `PrepareStructuredEnvironmentEachEntryCall`, `AdvanceStructuredEnvironmentEach`, `FinishStructuredEnvironmentEach`.

#### Foreign each (`11548–11626`; 6 declarations; R)

`IsStructuredForeignEachCall`, `PrepareStructuredForeignEachCall`, `StructuredForeignEachHasNext`, `PrepareStructuredForeignEachElementCall`, `AdvanceStructuredForeignEach`, `FinishStructuredForeignEach`.

#### IdentityMap atIfAbsent (`11634–11709`; 6 declarations; R)

`IsStructuredIdentityMapAtIfAbsentCall`, `PrepareStructuredIdentityMapAtIfAbsentCall`, `StructuredIdentityMapAtIfAbsentNeedsFallback`, `PrepareStructuredIdentityMapAtIfAbsentFallbackCall`, `FinishStructuredIdentityMapAtIfAbsentFallback`, `FinishStructuredIdentityMapAtIfAbsentPresent`.

#### IdentityMap each (`11718–11789`; 6 declarations; N)

`IsStructuredIdentityMapEachCall`, `PrepareStructuredIdentityMapEachCall`, `StructuredIdentityMapEachHasNext`, `PrepareStructuredIdentityMapEachEntryCall`, `AdvanceStructuredIdentityMapEach`, `FinishStructuredIdentityMapEach`.

#### Map match (`11797–11948`; 15 declarations; R)

`IsStructuredMapMatchCall`, `PrepareStructuredMapMatchCall`, `StructuredMapMatchHasRequirement`, `EnterStructuredMapMatchComparison`, `LeaveStructuredMapMatchComparison`, `PrepareStructuredMapMatchHashCall`, `AcceptStructuredMapMatchHashResult`, `StructuredMapMatchNeedsEquality`, `PrepareStructuredMapMatchEqualityCall`, `AcceptStructuredMapMatchEqualityResult`, `FinishStructuredMapMatchRequirement`, `StructuredMapMatchHasChildMatcher`, `PrepareStructuredMapMatchChildCall`, `AcceptStructuredMapMatchChildOutcome`, `FinishStructuredMapMatch`.

#### Map read lookup (`11956–12086`; 13 declarations; R)

`IsStructuredMapReadLookupCall`, `PrepareStructuredMapReadLookupCall`, `EnterStructuredMapReadLookupComparison`, `LeaveStructuredMapReadLookupComparison`, `PrepareStructuredMapReadLookupHashCall`, `AcceptStructuredMapReadLookupHashResult`, `StructuredMapReadLookupNeedsEquality`, `PrepareStructuredMapReadLookupEqualityCall`, `AcceptStructuredMapReadLookupEqualityResult`, `StructuredMapReadLookupNeedsFallback`, `PrepareStructuredMapReadLookupFallbackCall`, `FinishStructuredMapReadLookupFallback`, `FinishStructuredMapReadLookup`.

#### Map atPut (`12094–12196`; 10 declarations; R)

`IsStructuredMapAtPutCall`, `PrepareStructuredMapAtPutCall`, `EnterStructuredMapAtPutComparison`, `LeaveStructuredMapAtPutComparison`, `PrepareStructuredMapAtPutHashCall`, `AcceptStructuredMapAtPutHashResult`, `StructuredMapAtPutNeedsEquality`, `PrepareStructuredMapAtPutEqualityCall`, `AcceptStructuredMapAtPutEqualityResult`, `FinishStructuredMapAtPut`.

#### Map remove (`12204–12307`; 10 declarations; R)

`IsStructuredMapRemoveCall`, `PrepareStructuredMapRemoveCall`, `EnterStructuredMapRemoveComparison`, `LeaveStructuredMapRemoveComparison`, `PrepareStructuredMapRemoveHashCall`, `AcceptStructuredMapRemoveHashResult`, `StructuredMapRemoveNeedsEquality`, `PrepareStructuredMapRemoveEqualityCall`, `AcceptStructuredMapRemoveEqualityResult`, `FinishStructuredMapRemove`.

#### Map each (`12315–12385`; 6 declarations; N)

`IsStructuredMapEachCall`, `PrepareStructuredMapEachCall`, `StructuredMapEachHasNext`, `PrepareStructuredMapEachEntryCall`, `AdvanceStructuredMapEach`, `FinishStructuredMapEach`.

#### Error / cancellation (`12393–12499`; 8 declarations; R)

`RethrowTruffleException`, `IsCancellationUnwindActive`, `SupersedeCancellationUnwindIfActive`, `IsStructuredErrorHandlerCall`, `PrepareStructuredErrorHandlerCall`, `LoadStructuredErrorHandlerBodyCall`, `PrepareSelectedStructuredErrorHandlerCall`, `LeaveStructuredErrorHandlerFrame`.

#### IO C-prime (`12507–13003`; 36 declarations; R)

`PrepareBufferedByteReaderTargetCall`, `WrapBufferedByteReaderTargetInvocation`, `BufferedByteReaderTargetInvocationFailed`, `AwaitBufferedByteReaderTargetFuture`, `ResumeBufferedByteReaderTargetFutureWait`, `BeginBufferedByteWriterFirstEffect`, `IsBufferedByteWriterStepRequired`, `PrepareBufferedByteWriterFirstCall`, `PrepareBufferedByteWriterFlushCall`, `WrapBufferedByteWriterTargetInvocation`, `BufferedByteWriterTargetInvocationFailed`, `AwaitBufferedByteWriterTargetFuture`, `ResumeBufferedByteWriterTargetFutureWait`, `ApplyBufferedByteWriterFirstOutcome`, `ApplyBufferedByteWriterFollowupOutcome`, `IoReleaseHasNextStep`, `PrepareIoReleaseTargetCall`, `WrapIoReleaseTargetInvocation`, `IoReleaseTargetInvocationFailed`, `AwaitIoReleaseTargetFuture`, `ResumeIoReleaseTargetFutureWait`, `ApplyIoReleaseTargetOutcome`, `FinishIoReleaseSequence`, `PrepareTextWriterTargetCall`, `WrapTextWriterTargetInvocation`, `TextWriterTargetInvocationFailed`, `AwaitTextWriterTargetFuture`, `ResumeTextWriterTargetFutureWait`, `PrepareTextReaderSourceCall`, `WrapTextReaderSourceInvocation`, `TextReaderSourceInvocationFailed`, `AwaitTextReaderSourceFuture`, `ResumeTextReaderSourceFutureWait`, `TextReaderInitialNeedInput`, `IsTextReaderNeedInput`, `ApplyTextReaderLowerOutcome`.


### Selected full consumer chains and regression references

| Declaration/chain | Emitter + specialization owner | Runtime consumer / regression evidence |
|---|---|---|
| `TryCompactCanonicalBooleanOne` | C -> `ProtosCanonicalBooleanOneNode` (`@OperationProxy.Proxyable(allowUncached=true)`) | Canonical Boolean send and fallback; I101C handwritten Builder oracle |
| `TryPrimitiveNumericAdd/Subtract/Multiply/Divide/Less/LessEqual/Greater/GreaterEqual` | C -> 8 `ProtosNumeric*Node` proxies with static specializations | Numeric fast-path with guarded MISS, cached/uncached transitions; PERF040-E1 same-site tests |
| `Read*FrameLocal`, `Assign*FrameLocal` and materialized owner selection | C -> root operations (`@Operation`) | Lexical authority, owner scope, captured frame and DEBUG projection; lexical/capture regressions |
| `InvokeClosureZeroTail/OneTail/ArgumentsTail` | C -> root terminal operations | Real suspension and continuation transport, exactly-once post-resume execution, ReturnHome and Task ownership; I112 terminal tests |
| `Is/Prepare/FinishStructured*` present in S | S -> root operations -> `ProtosStructured*Calls` | Nested invocation / control, Error/ensure, maps and foreign each; structured dispatch regression families |
| `PrepareBufferedByteReaderTargetCall` -> `EnterClosureCall` -> `IsContinuation` -> `ResumeContinuation` -> `FinishCPrimeClosureCall` | IO byte reader Builder, root C-prime operations | Suspension/yield/resume of IO producer; buffered-reader C-prime tests |
| `MaterializeReturnCarrier.perform` | Called by `@EpilogReturn`, **not emitted** | Final carrier projection based on `ProtosFrameArguments.acceptsPrimitiveResult`; I091/contract tests |
| `AdmitsStraightLineSourceCall.perform` | Direct Java test invocation, **not emitted**; production `EnterStraightLineSourceCall.admitted` | PERF032-G7 direct test; non-suspending/ReturnHome admission |

## Source-level falsification and counterexamples

1. **Not 356 emitted**: 32 lack literal builder emission among producers examined. **Not 32 unneeded**: epilogue and direct test counterexamples.
2. **No duplicate generated interpreters**: I111 removed `ProtosSemanticBytecodeRootNode`, leaving one root. Source declarations may repeat Java logic or encode distinct ABI contracts; textual duplication is not two runtime implementations.
3. **OperationProxy is already adopted**: 9 proxies. Their referenced nodes have `@OperationProxy.Proxyable`, `@GenerateUncached`; Truffle parser checks proxy eligibility and uncached compatibility. Neither GraalPy nor SL imposes converting every internal operation to a proxy.
4. **A real small duplicate admission predicate exists**: `AdmitsStraightLineSourceCall.perform(OrdinarySourceCall)` and `EnterStraightLineSourceCall.admitted` both check provable non-suspension and absent ReturnHome lifecycle. Direct test access and generated-registration migration would require independent treatment.
5. **The epilogue materially needs carrier projection**: removing the `MaterializeReturnCarrier` **method** can break primitive-carrier guest-value boundaries. Assess only whether the unused **annotation/Builder registration** may be retired after source and generated checks.
6. **Five un-emitted `*Each` six-operation families** are candidates for generated-registration cleanup only. Their backing `Prepared*EachCall` Java protocols persist. `ArrayMatch`, `ForeignEach`, `LocalEach` and map read/match/update families *are* emitted by S, so family-name similarity is not an elimination argument.
7. **Build-tool negative counterexample**: existing generated topology baseline includes names for all five families and both special cases. A removal could invalidate static generated-BCI checks even if no guest bytecode emits the removed operation.
8. **Architectural preservation**: automatic RootTag remains disabled on the unified generator, semantic roots receive explicit RootTag, C-prime roots stay hidden, lazy source metadata remains lazy, and cached/uncached transitions and local-tag authority must remain unchanged. Existing I111/I112 and PLAT064-E duties are not reopened.

## Required PLAT064 facility disposition fields

### C-1: Registered but not visibly emitted five *Each families

- **FACILITY:** Truffle Bytecode DSL custom-operation registration, potential un-emitted operations.
- **CURRENT_PROTOS_HEAD_AND_SOURCE:** `edcb758fadb0e749cd45711cd664044d0b6db6bd`; root lines 11229–11540, 11718–11789, 12315–12385; inspected C/S/IO/Task emitters above.
- **TRUFFLE_API_VERSION_AND_DEFAULT:** 25.4.4.1.1; `@Operation` registers a custom operation; the processor does not require every registered operation to be emitted.
- **BYTECODE_DSL_PEER_SOURCE_AND_BEHAVIOR:** Pinned GraalPy and SimpleLanguage register operations via `@Operation` / `@OperationProxy`; neither source implies un-emitted operations are prohibited.
- **CURRENT_PROTOS_USE_AND_REQUIRED_SEMANTICS:** 30 DSL declarations without demonstrated `emitX`; preserved Java prepared-call protocols, uncached paths, Error/ensure and Task correctness are constraints.
- **DIVERGENCE:** Possible excessive *registered operation surface*; not proven dead Java behavior or semantic error.
- **DISPOSITION:** **PROPOSE_BOUNDED_Ixxx (proposal only; owner decision deferred)**.
- **JUSTIFICATION:** Removing needless generated registrations could reduce maintenance, but must first close all consumers and generated BCI baseline dependencies. No speedup gate.
- **OWNER_AND_NEXT_ACTION:** Project owner; **no issue allocated or implementation approved**. Retain open research decision gate.

### C-2: Admission predicate duplicated and separately registered

- **FACILITY:** `AdmitsStraightLineSourceCall` vs `EnterStraightLineSourceCall.admitted`.
- **CURRENT_PROTOS_HEAD_AND_SOURCE:** Same HEAD; root lines 8853–8973; PERF032-G7.
- **TRUFFLE_API_VERSION_AND_DEFAULT:** `@Operation` is a generated instruction declaration; static Java calls to method do not inherently emit instructions.
- **BYTECODE_DSL_PEER_SOURCE_AND_BEHAVIOR:** Pinned peers retain distinct invocation/control operations as needed, not universal method-to-operation registration.
- **CURRENT_PROTOS_USE_AND_REQUIRED_SEMANTICS:** Test direct call remains; production checks non-suspension and ReturnHome lifecycle inside `EnterStraightLineSourceCall`.
- **DIVERGENCE:** Duplicated Java admission predicate and apparently unused DSL registration.
- **DISPOSITION:** **PROPOSE_BOUNDED_Ixxx (proposal only; unselected)**.
- **JUSTIFICATION:** Reuse a single predicate may be narrowly justified; do not remove coverage or alter admission/fallback.
- **OWNER_AND_NEXT_ACTION:** Project owner, potentially group with C-1 after explicit approval.

### C-3: Return projection as epilogue plus operation registration

- **FACILITY:** `MaterializeReturnCarrier` and `@EpilogReturn`.
- **CURRENT_PROTOS_HEAD_AND_SOURCE:** Same HEAD; root epilogue and lines 9081 onwards.
- **TRUFFLE_API_VERSION_AND_DEFAULT:** `@EpilogReturn` and `@Operation` are separate DSL facilities.
- **BYTECODE_DSL_PEER_SOURCE_AND_BEHAVIOR:** Pinned GraalPy employs return epilogue; does not require distinct emitted projection instruction.
- **CURRENT_PROTOS_USE_AND_REQUIRED_SEMANTICS:** Direct method call from epilogue required; primitive carrier allowed only for accepted caller.
- **DIVERGENCE:** Method has required Java consumer but operation registration appears not emitted.
- **DISPOSITION:** **PROPOSE_BOUNDED_Ixxx (registration only; proposal unselected)**.
- **JUSTIFICATION:** Do not delete/alter method or carrier semantics; only inspect removable generator metadata.
- **OWNER_AND_NEXT_ACTION:** Project owner, possibly one bounded future operation-register cleanup.

### C-4: Operation vs proxy organization

- **FACILITY:** `@OperationProxy` reuse and direct `@Operation` implementations.
- **CURRENT_PROTOS_HEAD_AND_SOURCE:** Same HEAD; root lines 158–166, 520–13003; nine actual numeric/Boolean proxy Node classes.
- **TRUFFLE_API_VERSION_AND_DEFAULT:** Truffle supports proxies of eligible `Node` classes, including uncached eligibility validation.
- **BYTECODE_DSL_PEER_SOURCE_AND_BEHAVIOR:** GraalPy, SL use numerous proxies; TrufflePascal retains direct control operations. Different operation organization is supported.
- **CURRENT_PROTOS_USE_AND_REQUIRED_SEMANTICS:** Existing nine proxies; remaining operations own distinct frame/control/continuation semantics.
- **DIVERGENCE:** Different organizational mix, no additional exact eligible reuse demonstrated.
- **DISPOSITION:** **KEEP**.
- **JUSTIFICATION:** Neither number of proxies nor peer stylistic parity is evidence of redundancy; changing Node structure risks specialization/cache semantics.
- **OWNER_AND_NEXT_ACTION:** None. No mass proxy conversion.

## Negative findings, scope and pending validation

- No reason demonstrated to reopen I111/I112, reintroduce a second interpreter, add new RootTag to helper roots, change Task/ReturnHome/Error/ensure handling, or alter the published 25.4.4.1.1 dependency.
- No confirmed redundant semantic algorithm was shown across two generated interpreter roots; previous two-root counts are historical.
- No framework mismatch demonstrated for the remaining 324 explicitly emitted declarations or the nine existing proxies.
- No benchmark gain, tier-2 native compilation success or runtime measurement required for this research; **no tests were run**.
- A potential implementation would need exhaustive generated-output diffs, independent Builder regression oracles, cross-Context/DEBUG/Native compatibility, cached/uncached paths, lexical frame ownership, lazy diagnostics, cancellation and suspended resumption before making safety claims.
- Unverified: exact equivalence between the pinned 25.4 upstream source and 25.4.4.1.1 annotation-processor binary; generated interpreter output following any removal; impossible-to-have-indirect-consumer proof. Therefore this is source-backed **candidate evidence**, not safe-to-delete authorization.
- **PLAT063 boundary:** emitted *per-expression semantic-operation granularity* is a separate project issue; this dossier counts framework operation registrations.

## Coordination result

| Parent matrix area | Final research disposition | Action |
|---|---|---|
| #5 two generated roots | **EXISTING_OWNER / I111 COMPLETE** | Rebaseline: one physical generated root; no further unification |
| #6 duplicate declarations | **KEEP** for required ABI/specialization families; bounded proposal only for observed predicate/registration | No redesign; owner decision deferred |
| #7 possibly unused declarations | **30 NO_CONSUMER_DEMONSTRATED + 2 direct-consumer exceptions** | Proposed-only bounded future registration review |
| #12 OperationProxy | **KEEP** nine existing proxies; no 10th eligibility demonstrated | No action |
| PLAT064 parent | **OPEN** | Audit evidence linked; other children independently tracked |

**Research evidence publication is not a design ratification.** Owner said **no decision now**. No Ixxx, no implementation prompt/batch, no priority, schedule, or action inferred.

`NEXT_WORK=OWNER_DECISION_DEFERRED`
`TIPO=INVESTIGACION`
`FIRST_EDIT_PBATCH=NOT_APPLICABLE`
