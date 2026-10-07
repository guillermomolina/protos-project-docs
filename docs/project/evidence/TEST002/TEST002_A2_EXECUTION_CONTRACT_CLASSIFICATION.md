# TEST002-A2 execution legacy contract classification

Date: 2026-10-06

Owner: `TEST002` / `guillermomolina/protos#538`

This is snapshot evidence for the read-only TEST002-A2 investigation. It is not
a test-ownership registry, does not replace live GitHub state, and does not
reopen the retired TEST001 ownership infrastructure.

## Exact identities

- Audited Protos HEAD:
  `7f398f623a210b2bc3837d37dd3e5980dd726e75`
  (`I070-A: eliminate handwritten main compilation warnings`).
- Legacy placement-policy cutoff:
  `e99d0baba547ac41b3894f32ddca450172ee1f8b`
  (`TEST001-I: retire superseded test ownership infrastructure`).
- Prior TEST002 inventory baseline:
  `docs/project/evidence/TEST002/TEST002_LEGACY_JAVA_INVENTORY_BASELINE.md`.

The audit used GitHub repository state only. No local command, build, Maven,
Make, test, program, validator, commit or push was executed in
`guillermomolina/protos`.

## Universe reconciliation

At the cutoff there were 288 files below
`src/test/java/com/guillermomolina/protos/execution/`. Twenty-nine have since
been removed. Exactly **259 pre-policy execution files survive** at the audited
HEAD, so the A2 population has **zero delta** from the 259-file baseline.

Post-policy execution tests are outside this initial TEST002 legacy universe
except when read as evidence that an older contract has already been replaced.

File-level disposition of the 259 survivors:

| Disposition | Files |
| --- | ---: |
| `MIGRATE_TO_PROTOS` | 3 |
| `SPLIT` | 2 |
| `KEEP_JAVA_BOOTSTRAP` | 45 |
| `KEEP_JAVA_HOST_RUNTIME` | 206 |
| helper/support files, assigned to consuming Java families | 3 |
| **Total** | **259** |

The file counts are not a claim that each file is one independent semantic
contract. They are an exhaustive path accounting so no legacy survivor is left
unclassified.

## Migration candidates

### MIGRATE_TO_PROTOS

- `src/test/java/com/guillermomolina/protos/execution/ProtosJsonParserModuleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTomlEncoderModuleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTomlParserModuleTest.java`

`ProtosJsonParserModuleTest` is residual Java semantic ownership: its depth-64
nested-array and 64-element materialization assertions are guest-observable and
are already matched or exceeded by the suite-native JSON corpus, including
`library/json/parser-positive.protos`,
`library/json/final-deep-stress.protos`, and
`library/json/final-large-materialization.protos`.

The TOML parser and encoder classes execute ordinary Protos source and validate
guest-observable parsing, construction, rejection, encoding, nesting, float
spelling, escaping and round-trip behavior. No suite-native `std:toml/TOML`
cohort currently owns those contracts; the Package Tool TOML-syntax corpus is a
different boundary.

### SPLIT

- `src/test/java/com/guillermomolina/protos/execution/ProtosTomlClosureConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTomlDataModelModuleTest.java`

For `ProtosTomlClosureConformanceTest`, migrate the semantic round-trip and
fresh-module-state tests. Retain Java for
`publicTomlSourceKeepsTheD087AndHostRuntimeBoundaries`, which physically reads
`protos/lib/toml/TOML.protos` and asserts architecture/source-boundary
properties.

For `ProtosTomlDataModelModuleTest`, migrate constructor semantics, invalid
input rejection, and D104 second-60 semantic data. Retain Java for the exact
module local-slot export surface and Actor graph-transfer / physical module
identity evidence.

## KEEP_JAVA_BOOTSTRAP

These tests observe an external or pre-guest boundary: module/package/workspace
resolution, production entry architecture, source-loading/readability authority,
Core/language bootstrap, or exact Standard Library module installation.

- `src/test/java/com/guillermomolina/protos/execution/ProtosA4B3ModuleProcessHostingTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosA4B3NestedToolProcessHostingTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosA4B3ProductionEntryArchitectureTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosBundledToolModuleResolverTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosCommandLineModuleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosCommandLineResultModelTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosCommandLineSpecModuleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosCoreBootstrapTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosCoreContextSourceTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosCoreErrorInfrastructureTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosCoreNativeBoundaryArchitectureTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosCoreSourceNamingArchitectureTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosExternalPackagePlanningPreflightTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosLanguageRegistrationTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosLanguageSourceParsingTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosMathIntegerModuleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosModuleSourceIdentityTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNetworkBootstrapAuthorityTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNetworkingIpAddressesModuleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNetworkingIpEndpointsModuleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPackageContentIdentityV1ConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPackageContentVerificationTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPackageExecutionPlanAdapterTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosProjectBindingPrerequisiteClosureTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosProjectFileBindingProviderTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosSourceCompilerTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosSourceFileLoaderTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosSourceReadabilityAuthorityTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandaloneProcessBootstrapTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardLibraryModuleResolverTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosUriModuleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosWorkspaceExactDirectoryLookupTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosWorkspaceMemberLocationTraversalTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosWorkspacePackageApplicationExecutionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosWorkspacePackageAuthorityIsolationIntegrationTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosWorkspacePackageDependencyRoutingTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosWorkspacePackageDirectoryIndexTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosWorkspacePackageModuleKeyTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosWorkspacePackageModuleResolverBoundaryTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosWorkspacePackageModuleResolverTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosWorkspacePackagePreflightTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosWorkspacePackageProjectIndexTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosWorkspacePackageSourceInventoryTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosWorkspacePackageSourceLookupTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosWorkspacePackageStandardDelegationTest.java`

## KEEP_JAVA_HOST_RUNTIME

These tests retain materially distinct physical/runtime evidence: Truffle or
Bytecode lowering, CallTarget/compiler behavior, represented values and exact
prototype/home identity, scheduler/Actor/Future state, C-prime and suspension
machinery, native callback counters, host filesystem/network/process resources,
NIO backends, transfer/materialization, debugger/DAP/runtime integration, Test
Tool internals, or similar properties that ordinary guest source cannot
faithfully manufacture and inspect.

- `src/test/java/com/guillermomolina/protos/execution/CanonicalBareSlotMutationExecutionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/CanonicalClosureMaterializationTest.java`
- `src/test/java/com/guillermomolina/protos/execution/CanonicalCompositionExecutionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/CanonicalExplicitMemberMutationExecutionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/CanonicalLookupExecutionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/CanonicalMemberReadExecutionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/CanonicalObjectExecutionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ExtractedClosureBindingExecutionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosAPlusExecutionProjectionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosActorBootstrapTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosActorPolyglotContextRoutingTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosArrayConformanceCompletionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosAsyncExactExecutionFacilityTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosAsyncProcessSnapshotExecutionFacilityTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosCanonicalInitialModuleExecutionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosCapturedFilesystemCustodyTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosCapturedProcessExecutionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosClosureInvokerTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosCollectionsArrayAlgorithmsModuleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosCollectionsArrayReduceSortModuleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosCollectionsSetAlgebraModuleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosCollectionsSetModuleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosCollectionsSetMutationModuleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosCsvModuleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosDetachedExecutionValueCrossPreludeTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosDirectionalShutdownTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosEncodingTransferTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosEnvironmentParallelTransferTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosExactExecutionFacilityTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosFilesystemIntegratedConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosFilesystemParallelTransferTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosFreshProcessExecutorTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosI026EDebuggerIntegrationTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosI026EScopeTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosI026FDapBehaviorTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosI026FDapTransportTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosIdentityMapConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosInvalidSuperExecutionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosJsonDataModelModuleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosLm009DDebugRuntimeHostTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosModuleRuntimeTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNetworkAuthorityConfinementTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNetworkConnectAcquisitionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNetworkListenAcquisitionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNetworkingFoundationFinalConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNioConfinedFilesystemBackendTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNioFilesystemTreeCaptureBackendTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNioHostIoPollerTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNioNetworkBackendConnectTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNioNetworkProvisioningTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNioReadOnlyTreeFilesystemBackendTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNioTcpConnectionLifecycleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNioTcpConnectionReadTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNioTcpConnectionWriteTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNioTcpListenerBackendTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNonLocalReturnTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNumericPrototypeBridgeTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosParallelExecutionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosParallelPolyglotContextRoutingTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B1BytecodeLiteralSequenceTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2AClosureActivationTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2BClosureContinuationCompositionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2C1SinglePositionalBindingTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2C2GeneralPositionalArityTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2C3ARestBindingTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2C3B1ActivationRootSeamTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2C3B2SimpleDefaultBindingTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2C3B3CallSendDefaultBindingTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2D1OrdinarySendCompositionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2D2NestedArgumentCompositionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2D3AOrdinaryObjectCallProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2D3BSelectedStandardObjectCallIntrinsicTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2D4AComposedCallTargetTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2D4BComposedSendReceiverTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2D5ABodyCallSpreadTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2D5BBodySendSpreadTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2D5CDefaultSpreadTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B3BTaskBytecodeCompositionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B3CSuspensionCapableNativeLeafTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B3DTopLevelTaskPublicationTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B3EStandardFutureValueCPrimeTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B3FClosureEvidenceTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B4ATransferSubstrateTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B4BNonLocalReturnTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B4CStructuredEnsureTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B4DErrorHandlersTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B4ECancellationUnwindTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B4FStructuredWhileTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B4GControlUnwindClosureTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B5ABytecodeDebuggerScopeTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B5BSourceInstrumentationLocationTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B6A1ReadOnlyCanonicalCoverageTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B6A2MutatingCanonicalCoverageTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B6A3ASuperSendCanonicalCoverageTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B6A3BClosureLiteralCanonicalCoverageTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B6A3CObjectComposeCanonicalCoverageTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B6A4OrdinaryNativeFastPathTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B6A5ModuleInitializationCPrimeBridgeTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B6A6A1TaskOwnedClosureDispatchTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B6BFinalConformanceArchitectureTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006C1OptimizingRuntimeClosureTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006C2PortableRuntimeReconciliationTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006C3DParallelBytecodeRematerializationTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006C3FLifecycleReleaseCarrierTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006Plat028ArrayEachCallbackTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006Plat028BooleanCallbackTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006Plat028BytesEachCallbackTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006Plat028EnvironmentEachCallbackTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006Plat028IdentityMapEachCallbackTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006Plat028MapAtPutCallbackTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006Plat028MapEachCallbackTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006Plat028MapReadLookupCallbackTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006Plat028MapRemoveCallbackTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006Plat029TextReaderCallbackTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006Plat029TextWriterCallbackTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006Plat031BufferedReaderCPrimeTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006Plat031BufferedWriterCPrimeTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPlat029FutureValueNonTaskCPrimeTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPlat029IoOperationCPrimeDriverTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPolyglotExecutionContextTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPolyglotNestedCallTargetSharingTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPolyglotProcessHostingTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPolymorphicInvocationTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosProcessIntegratedConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosProcessSnapshotExecutionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosProcessStandardStreamParallelTransferTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosRootActorBootstrapAuthorityTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardArrayFactoryTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardBooleanProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardBufferedByteIoProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardByteIoDurabilityProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardByteIoPositioningProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardByteIoProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardBytesProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardEncodingProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardEnvironmentProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardFileProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardFilesystemProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardFilesystemTreeMaterializationTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardFloatArithmeticTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardFutureProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardIntegerArithmeticTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardMapProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardNumberEqualityTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardNumberOrderingProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardNumericConversionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardPathProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardProcessProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardStringProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardTextReaderProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardTextWriterProtocolTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTcpConnectionDuplexLifecycleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTcpConnectionEndpointObservationTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTcpConnectionIntegratedConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTcpListenerAcceptTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTcpListenerIntegratedConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTcpListenerLifecycleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolActorGroupOwnershipArchitectureTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolCatalogAcquisitionFacilityTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolClosureFreshExpectationsTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolErrorParentExpectationsTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolExactBinary64Test.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolFloatBitsExpectationsTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolFloatBitsParserTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolFloatNanExpectationsTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolFutureFreshInspectionFixtureSuiteTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolFutureObservationPolicyFixtureSuiteTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolFutureResolvedMechanismTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolFutureStoredExpectationRecognitionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolFutureStoredInspectionFixtureSuiteTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolFutureTerminalMechanismTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolH2B1BoundedSimpleSchedulingTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolH2B2BoundedDCaseSchedulingTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolHClosureReconciliationTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolI8CResourceRoundSchedulingTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolI8D1ProviderFoundationTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolI8D2ProviderTransactionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolI8D3RootActorResourceBundleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolI8D4ATerminalAttemptEnvelopeTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolI8D4BResourcefulAttemptBridgeTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolI8D4C1ResourcefulExecutionFacilityTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolI8D4C2ResourcefulInspectionFacilityTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolI8D4C3RunnerRoundDrainTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolI8D5BTestRunOutcomeProjectionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolJClosureReconciliationTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolLib011OptionsAdoptionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolManifestPlanTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolPackageExecutionEnvironmentTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolPackageFailedExecutionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolRepositoryCorpusPlansTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolResourceAdmissionBindingTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolResourceCatalogCompositionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolResourceCatalogOptionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolResourceCatalogSchemaTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolResourceRequirementsDiscoveryTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolResourceRequirementsJoinTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolResourceRequirementsSchemaTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolResourceReservationKernelTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolSequentialRunnerTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolSimpleExpectationsTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolSourceLoaderTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolSuiteGraphTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolTool004BProgressBoundaryTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolTool004CProgressPresentationTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTextIoFinalConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTextReaderLineProtocolTest.java`

## Helpers

These files have no independent TEST002 semantic disposition; they remain
assigned to the Java families that consume them.

- `src/test/java/com/guillermomolina/protos/execution/ProtosHostedExecutionTestFixture.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf006BytecodeTestSupport.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestExecutionSupport.java`

## Already-reconciled semantic ownership

The audit verified that several legacy Java tests now intentionally retain only
host/runtime/bootstrap evidence while ordinary behavior is already owned by
Protos tests. Important examples include:

- `ProtosNonLocalReturnTest`: exact host ReturnHome/control transfer;
- `ProtosIdentityMapConformanceTest`: representation and exact bootstrap binding;
- `ProtosStandardStringProtocolTest`: frozen Prelude/bootstrap topology;
- `ProtosStandardFutureProtocolTest`: real evaluator suspension bridge,
  scheduler state, peer progress and exact-once resume;
- `ProtosStandardEnvironmentProtocolTest`: host/native values and
  non-representable input;
- `ProtosArrayConformanceCompletionTest`: represented lifecycle/control-transfer
  evidence;
- `ProtosStandardMapProtocolTest` and `ProtosStandardArrayFactoryTest`:
  represented lifecycle/materialization state;
- `ProtosNumericPrototypeBridgeTest` and `ProtosPolymorphicInvocationTest`:
  exact physical topology/materialization.

TEST002 must not remigrate those already-reconciled public semantics.

## First implementation batch

The first amortized implementation slice is:

**TEST002-A3 — TOML legacy semantic ownership migration**

Repository: `guillermomolina/protos`.

Affected legacy Java:

- `ProtosTomlParserModuleTest.java`;
- `ProtosTomlEncoderModuleTest.java`;
- `ProtosTomlClosureConformanceTest.java`;
- `ProtosTomlDataModelModuleTest.java`.

Recommended suite-native cohort:

- `protos/tests/conformance/library/toml/data-model.protos`;
- `protos/tests/conformance/library/toml/parser-positive.protos`;
- `protos/tests/conformance/library/toml/parser-errors.protos`;
- `protos/tests/conformance/library/toml/encoder-positive.protos`;
- `protos/tests/conformance/library/toml/encoder-errors.protos`;
- `protos/tests/conformance/library/toml/roundtrip.protos`.

Register those sources through the existing
`protos/tests/conformance/manifest.tsv`; do not redesign Test Tool discovery or
TOOL009 execution.

Java removable after equivalent-or-stronger suite-native coverage is established:

- all of `ProtosTomlParserModuleTest`;
- all of `ProtosTomlEncoderModuleTest`;
- `officialStyleToml11DocumentRoundTripsSemantically`;
- `freshModuleBootstrapsDoNotShareParserOrSemanticState`;
- `semanticConstructorsPreserveApprovedTomlKindsAndPayloads`;
- `constructorsFailClosedOnWrongFamiliesAndInvalidTemporalData`;
- `d104PreservesSecondSixtyAsTomlSemanticDataWithoutEventValidation`.

Java that must remain:

- `publicTomlSourceKeepsTheD087AndHostRuntimeBoundaries`;
- `importedModuleExportsExactlyLib010ASemanticConstructors`;
- `semanticDataTransfersAcrossActorsWhileModuleRemainsActorLocal`.

The migration must preserve adversarial parser coverage, including forbidden
comment controls, DEL, malformed quote runs, temporal edge cases, exact
binary64/shortest-round-trip spelling, nested containers and deterministic
encoding. It must not substitute the Package Tool TOML parser corpus for the
public `std:toml/TOML` contract.

## TEST002 state

```text
TEST002_CLOSABLE=NO
LEGACY_EXECUTION_SURVIVORS=259
EXECUTION_FILE_MIGRATE_TO_PROTOS=3
EXECUTION_FILE_SPLIT=2
EXECUTION_FILE_KEEP_JAVA_BOOTSTRAP=45
EXECUTION_FILE_KEEP_JAVA_HOST_RUNTIME=206
EXECUTION_HELPERS=3
NEXT_SLICE=TEST002-A3
NEXT_SLICE_NAME=TOML legacy semantic ownership migration
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
```

After A3, TEST002 still requires reconciliation of the residual JSON parser
class and then the remaining legacy cohorts outside `execution`; therefore
TEST002 must remain open.
