# TEST002-A5 final non-execution legacy Java classification and closure evidence

Date: 2026-10-06

Owner: `TEST002` / `guillermomolina/protos#538`

This record is durable snapshot evidence for the final TEST002 audit. It records
the exact published Protos revision against which the remaining pre-policy Java
test population was re-evaluated. It is not a replacement for live GitHub
coordination and does not recreate the retired TEST001 semantic-ownership
registry.

## Exact audit revision

```text
PROTOS_REVISION=845a1103b031abcf95d8ba852e0d780ba6b6591a
SUBJECT=AUD006-A4: make CommandLine accumulation linear
LEGACY_POLICY_CUTOFF=e99d0baba547ac41b3894f32ddca450172ee1f8b
```

The repository advanced concurrently after TEST002-A4. The final A5 recheck
confirmed that the current HEAD did not change the non-`execution` legacy
population: the intersection with the policy cutoff remains exactly 390 legacy
Java survivors, of which 256 belong to the already-reconciled `execution`
cohort and 134 belong to A5.

```text
CURRENT_LEGACY_JAVA_SURVIVORS=390
EXECUTION_SURVIVORS_ALREADY_RECONCILED=256
A5_NON_EXECUTION_POPULATION=134

runtime=59
parser=18
semantic=17
cli=14
conformance=7
lsp=7
analysis=5
lexer=4
documentation=3
```

## Classification result

The exhaustive A5 result is:

```text
MIGRATE_TO_PROTOS=0
SPLIT=0
KEEP_JAVA_HOST_RUNTIME=64
KEEP_JAVA_BOOTSTRAP=68
HELPER=2

IMPLEMENTATION_BATCHES_REQUIRED=0
NEXT_TECHNICAL_SLICE=NONE
TEST002_CLOSABLE=YES
```

The retained Java population is not semantic duplication. Where suite-native
Protos coverage overlaps with a retained Java test, the Java test proves a
materially distinct host/runtime/bootstrap contract such as exact Java object
identity, physical prototype/home relationships, snapshots, Truffle interop,
scheduler/mailbox state, host resource lifecycle, NIO effects, protocol
framing, parser/semantic-AST shape, or source/tooling infrastructure.

The placement result therefore satisfies the current repository rule:
Protos-observable semantics live in suite-native Protos where practical, while
Java/JUnit remains for genuinely host/runtime/bootstrap evidence.

## Exhaustive package accounting

### analysis — 5 KEEP_JAVA_BOOTSTRAP

- `src/test/java/com/guillermomolina/protos/analysis/ProtosProjectBindingTest.java`
- `src/test/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisCoreTest.java`
- `src/test/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisSessionTest.java`
- `src/test/java/com/guillermomolina/protos/analysis/ProtosStaticDefinitionsTest.java`
- `src/test/java/com/guillermomolina/protos/analysis/ProtosStaticReferencesTest.java`

These tests own project-binding, static-analysis, source-span, definition and
reference machinery. Their primary evidence is tooling/bootstrap state rather
than ordinary guest execution.

### cli — 14 KEEP_JAVA_BOOTSTRAP

- `src/test/java/com/guillermomolina/protos/cli/ProtosCliLearningMaterialsTest.java`
- `src/test/java/com/guillermomolina/protos/cli/ProtosCliPolyglotRoutingArchitectureTest.java`
- `src/test/java/com/guillermomolina/protos/cli/ProtosCliTest.java`
- `src/test/java/com/guillermomolina/protos/cli/ProtosDiagnosticInspectorTest.java`
- `src/test/java/com/guillermomolina/protos/cli/ProtosLm009DPublicDebugCliTest.java`
- `src/test/java/com/guillermomolina/protos/cli/ProtosLm009FLanguageServerCliTest.java`
- `src/test/java/com/guillermomolina/protos/cli/ProtosReplTest.java`
- `src/test/java/com/guillermomolina/protos/cli/ProtosTestToolCorpusRegistryTest.java`
- `src/test/java/com/guillermomolina/protos/cli/ProtosTestToolExecutionRequirementRegistryTest.java`
- `src/test/java/com/guillermomolina/protos/cli/ProtosTestToolH2B3PublicIntegrationTest.java`
- `src/test/java/com/guillermomolina/protos/cli/ProtosTestToolI8D5AHostRegistryBootstrapTest.java`
- `src/test/java/com/guillermomolina/protos/cli/ProtosTestToolI8D5CPublicCutoverTest.java`
- `src/test/java/com/guillermomolina/protos/cli/ProtosTestToolStdoutCompletionTest.java`
- `src/test/java/com/guillermomolina/protos/cli/ProtosWorkspaceRunCliTest.java`

The contracts are CLI/REPL/Test Tool/DAP/LSP/process boundaries: argument
routing, stdout/stderr/exit behavior, protocol framing, registries, exact host
authorities, scheduler integration, public command routing and workspace/package
materialization. Guest Protos source used by these tests is fixture/input to
that boundary.

### conformance — 7 KEEP_JAVA_HOST_RUNTIME

- `src/test/java/com/guillermomolina/protos/conformance/ProtosFilesystemLanguageConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/conformance/ProtosFilesystemLibraryConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/conformance/ProtosFilesystemMaturityConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/conformance/ProtosFilesystemTreeIntegratedConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/conformance/ProtosFilesystemTreeSurfaceConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/conformance/ProtosResourceLifetimeMaturityConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/conformance/ProtosSystemResourceEndToEndConformanceTest.java`

These JUnit owners intentionally pair ordinary Protos cases with host-provisioned
backends/resources that the normal Test Tool cannot construct or observe:
confined NIO state, hard links, symlinks, Unix-socket kinds, open/read/write/close
counters, cancellation races, late completions, physical resource cleanup and
deterministic Process/filesystem stream backends.

### documentation — 3 KEEP_JAVA_BOOTSTRAP

- `src/test/java/com/guillermomolina/protos/documentation/ProtosDocumentationJsonTest.java`
- `src/test/java/com/guillermomolina/protos/documentation/ProtosDocumentationModelTest.java`
- `src/test/java/com/guillermomolina/protos/documentation/ProtosStandardLibraryDocumentationExtractorTest.java`

These own documentation-model identity/provenance, canonical serialized bytes,
source comments/spans and extractor coverage.

### lexer — 4 KEEP_JAVA_BOOTSTRAP

- `src/test/java/com/guillermomolina/protos/lexer/ProtosLexerSourceSpanTest.java`
- `src/test/java/com/guillermomolina/protos/lexer/ProtosLexerTest.java`
- `src/test/java/com/guillermomolina/protos/lexer/UnicodeNfc17ConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/lexer/UnicodeXid17ConformanceTest.java`

These own token kinds, lexical rejection, raw source spans, and exhaustive
Unicode 17 NFC/XID conformance before guest execution.

### lsp — 7 KEEP_JAVA_BOOTSTRAP

- `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerDefinitionTest.java`
- `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerDiagnosticsTest.java`
- `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerDocumentSymbolsTest.java`
- `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerFoundationTest.java`
- `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerReferencesTest.java`
- `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerWorkspaceSymbolsTest.java`
- `src/test/java/com/guillermomolina/protos/lsp/ProtosWorkspaceSymbolSearchTest.java`

These own LSP4J request/response objects, protocol capabilities, UTF-16 ranges,
diagnostic publication, overlay/project authority, server lifecycle/framing and
workspace-symbol ranking.

### parser — 18 KEEP_JAVA_BOOTSTRAP

- `src/test/java/com/guillermomolina/protos/parser/ProtosBindingSurfaceNegativeFixtureTest.java`
- `src/test/java/com/guillermomolina/protos/parser/ProtosCallableSurfaceNegativeFixtureTest.java`
- `src/test/java/com/guillermomolina/protos/parser/ProtosGrammarSurfaceNegativeFixtureTest.java`
- `src/test/java/com/guillermomolina/protos/parser/ProtosParserArgumentLayoutTest.java`
- `src/test/java/com/guillermomolina/protos/parser/ProtosParserClosureTest.java`
- `src/test/java/com/guillermomolina/protos/parser/ProtosParserEllipsisContinuationTest.java`
- `src/test/java/com/guillermomolina/protos/parser/ProtosParserExpressionSeparatorTest.java`
- `src/test/java/com/guillermomolina/protos/parser/ProtosParserFoundationTest.java`
- `src/test/java/com/guillermomolina/protos/parser/ProtosParserMatchTest.java`
- `src/test/java/com/guillermomolina/protos/parser/ProtosParserMemberContinuationTest.java`
- `src/test/java/com/guillermomolina/protos/parser/ProtosParserNonLocalReturnTest.java`
- `src/test/java/com/guillermomolina/protos/parser/ProtosParserObjectExpressionTest.java`
- `src/test/java/com/guillermomolina/protos/parser/ProtosParserPostfixFoundationTest.java`
- `src/test/java/com/guillermomolina/protos/parser/ProtosParserSlotAssignmentTest.java`
- `src/test/java/com/guillermomolina/protos/parser/ProtosParserStandardOperatorTest.java`
- `src/test/java/com/guillermomolina/protos/parser/ProtosParserSuperSendTest.java`
- `src/test/java/com/guillermomolina/protos/parser/ProtosParserTrailingClosureTest.java`
- `src/test/java/com/guillermomolina/protos/parser/ProtosReceiverSuperSurfaceNegativeFixtureTest.java`

The four LM008 negative-fixture classes remain the already-established
pre-execution rejection boundary. The other parser tests inspect exact
`Surface*` node shape, parser layout/association and `ParseError` behavior,
which is bootstrap/compiler-front-end evidence rather than ordinary guest
semantics.

### semantic — 17 KEEP_JAVA_BOOTSTRAP

- `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerArithmeticTest.java`
- `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerCallTest.java`
- `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerClosureTest.java`
- `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerComparisonTest.java`
- `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerEqualityTest.java`
- `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerFoundationTest.java`
- `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerIndexTest.java`
- `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerIndexedAssignmentTest.java`
- `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerIntrinsicTest.java`
- `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerLazyBooleanTest.java`
- `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerLiteralKindTest.java`
- `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerObjectTest.java`
- `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerReturnTest.java`
- `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerSlotWriteTest.java`
- `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerSpreadTest.java`
- `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerSuperSendTest.java`
- `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerUnaryTest.java`

These prove canonical-AST/lowering shape: exact `Canonical*` node families,
selector lowering, target shape, precedence/association and canonical recursion.
They are front-end implementation evidence, not duplicate guest-result tests.

### runtime — 57 KEEP_JAVA_HOST_RUNTIME + 2 HELPERS

KEEP_JAVA_HOST_RUNTIME:

- `src/test/java/com/guillermomolina/protos/runtime/ProtosActivationReceiverTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosActorCrossDomainConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosActorDeliveryAdmissionTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosActorExecutionDomainTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosActorGroupAcquisitionTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosActorGroupCommunicationTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosActorGroupRoutingTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosActorIdentityLifecycleTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosActorMailboxSchedulerTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosActorProcessHostRoutingTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosActorPublicApiTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosActorRefValueTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosActorRemoteTransportTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosActorRequestTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosActorSendOperationTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosActorTerminationTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosActorValueTransferTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosAdditionalIndexedInteropTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosArrayInteropTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosArrayValueTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosBufferedByteIoPlat031FoundationTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosByteIoFirstEffectGateTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosClosureInvocationActivationTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosDynamicControlStateTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosEnvironmentSnapshotTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosExecutionContextTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosFilesystemNamespaceMutationFlowTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosFilesystemOpenFlowTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosFilesystemTransferTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosFilesystemTreeObservationFlowTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosFloatInteropTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosGroupRefPublicApiTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosGroupRemoteTransportTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosIntegralInteropTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosIoLifecycleTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosMethodHomeTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosNetworkCapabilityTransferTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosObjectInteropTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosObjectValueTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosPerf006B3ATaskContinuationPublicationTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosPerf006B6A6A2TaskTerminalLifecycleTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosPerf006B6A6A3ActorGroupLifecycleIntegrationTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosPlat029IoOperationRunnableSeamTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosPreludeTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosProcessArgumentsSnapshotTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosProcessCapabilityTransferTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosProcessRuntimeTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosProcessStandardStreamBindingTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosProcessStandardStreamEncodingTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosRepresentedValueInteropCoverageTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosRepresentedValueLookupTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosReturnHomeTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosSimpleScalarInteropDisplayTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosSimpleScalarInteropTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosSlotLookupResultTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosTcpConnectionFoundationTest.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosTcpListenerFoundationTest.java`

HELPERS:

- `src/test/java/com/guillermomolina/protos/runtime/ProtosHostEncodingTestCodec.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosTestPrelude.java`

The runtime cohort owns exact host/runtime representation and mechanics:
Actor/Group/Process identity and transport, mailbox/admission/scheduler state,
Task continuation and lifecycle state, graph transfer and alias preservation,
Truffle `InteropLibrary` projections, runtime lookup/home identity, I/O
operation lifecycle, network/file authority, standard-stream host binding and
TCP opaque-resource foundations. The two helpers carry no independent semantic
contract and remain because they are shared test fixtures.

## Falsification of apparent duplicates

A5 specifically rechecked the most likely false positives rather than assuming
that a Java test should remain merely because it was already Java.

Current suite-native Protos owners cover ordinary observable behavior for
families including Array/Object/reflection/execution-context and Actor/Group.
Representative owners include:

- `protos/tests/conformance/collections/array-indexed-read-and-size.protos`;
- `protos/tests/conformance/collections/array-indexed-update.protos`;
- `protos/tests/conformance/object/local-slot-mutation.protos`;
- `protos/tests/conformance/object/composition.protos`;
- `protos/tests/conformance/object-structural/alias.protos`;
- `protos/tests/conformance/object-structural/without.protos`;
- `protos/tests/conformance/execution-context/capture-by-reference-and-late-nearer-creation-retargeting.protos`;
- `protos/tests/conformance/execution-context/escape-close-freeze-and-present-null-preserved.protos`;
- `protos/tests/conformance/actor/current-and-identity.protos`;
- `protos/tests/conformance/actor/lifecycle-and-transfer.protos`;
- `protos/tests/conformance/group/acquisition-identity-and-transfer.protos`;
- `protos/tests/conformance/group/stopped-and-surface.protos`.

The retained JUnit classes in those areas still prove a separate layer: exact
Java references, detached host snapshots, host exception classes, physical
parents/home objects, mailbox sizes, delivery-attempt states, scheduler/domain
queues, transport/process state, or Truffle interop. Removing those assertions
would reduce evidence rather than eliminate duplicate semantic ownership.

Therefore:

```text
JAVA_DUPLICATE_CAN_BE_REMOVED=0
NEW_SUITE_NATIVE_MIGRATION_OWNER_REQUIRED=0
SPLIT_RESIDUAL=0
```

## Earlier TEST002 reconciliation incorporated by closure

TEST002-A2 classified the surviving legacy `execution` cohort. TEST002-A3
migrated the coherent TOML semantic ownership batch to suite-native Protos and
retained only distinct Java host/runtime assertions. TEST002-A4 migrated the
remaining pure JSON parser semantic owner and left the explicit parser stress
harness Java-owned.

At A5 there are no remaining implementation batches.

```text
EXECUTION_AUDIT_AND_RECONCILIATION=COMPLETE
EXECUTION_KNOWN_MIGRATE_TO_PROTOS_RESIDUAL=0
NON_EXECUTION_AUDIT_AND_RECONCILIATION=COMPLETE
NON_EXECUTION_MIGRATE_TO_PROTOS_RESIDUAL=0
NON_EXECUTION_SPLIT_RESIDUAL=0
```

## Closure assessment

All closure conditions from TEST002/#538 are satisfied at the exact audit
revision:

```text
LEGACY_PRE_POLICY_JAVA_TESTS=AUDITED
MIGRATABLE_PROTOS_SEMANTICS=MIGRATED
HOST_RUNTIME_JAVA_TESTS=RETAINED
BOOTSTRAP_JAVA_TESTS=BOUNDED_AND_JUSTIFIED
COVERAGE=EQUIVALENT_OR_STRONGER
TEST_OWNERSHIP_REGISTRY_REQUIRED=NO
TEST001_RUNNER_CUTOVER_REOPENED=NO

TEST002_A5=COMPLETE
TEST002_STATUS=COMPLETED
NEXT_TECHNICAL_SLICE=NONE
```

No normative Protos specification, production implementation, Test Tool
behavior, version metadata or product repository content changed in A5. A5 is
an investigation/coordination closure step only.
