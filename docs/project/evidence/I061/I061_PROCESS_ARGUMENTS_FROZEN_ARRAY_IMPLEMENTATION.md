# I061 — Frozen Array Process-argument implementation and closure evidence

Date: 2026-09-30

## Identity

```text
WORK_ITEM=I061/#664
DECISION=D168/#638
DECISION_STATUS=RATIFIED
SELECTED_CANDIDATE=C

PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=0a5a86965bafb713859335d17312b6bdabfbae8d
COMMIT=I061: replace ProcessArguments with frozen Array bootstrap snapshots (D168)

IMPLEMENTATION_VERSION=0.3.120-SNAPSHOT
SPECIFICATION_REVISION=0.1.435

CI_RUN=36668328660
CI_RUN_NUMBER=2038
CI_RESULT=PASS
```

This record retains the published implementation, normative reconciliation and
full-CI closure evidence for I061. D168 remains the decision authority; this
record does not redefine Process I/O semantics.

## Selected semantic result

I061 implements D168 Candidate C:

```text
process.args()                            KEEP
stable bootstrap argument contents       KEEP
stable argument order                    KEEP
String-only portable elements            KEEP
later host argv mutation invisible       KEEP

ProcessArguments semantic family         REMOVE
process.args() result                    FROZEN ORDINARY Array<String>

canonical args accessor-result identity  REMOVE
canonical Environment accessor identity  REMOVE
cross-Actor canonical reacquisition      REMOVE

Environment semantic family              KEEP
Environment native-name semantics        KEEP
Environment representability             KEEP
Environment contains/get behavior         KEEP
ordinary Map substitution                REJECT
```

The retained portable contract is stable bootstrap content, not one canonical
container identity. Separate calls to `process.args()` or
`process.environment()` have no portable identity relation in either
direction.

No replacement `Arguments`, `ImmutableSequence` or common
`BootstrapSnapshot` abstraction was introduced.

## Runtime implementation

The Process runtime now retains immutable ordered String argument content rather
than a `ProtosProcessArgumentsValue`. Each `process.args()` acquisition
constructs an ordinary Core Array from the calling Actor's Array prototype and
freezes it before exposure.

The implementation removed the dedicated semantic/runtime institution:

```text
src/main/java/com/guillermomolina/protos/runtime/ProtosProcessArgumentsValue.java
src/main/java/com/guillermomolina/protos/execution/ProtosStandardProcessArgumentsProtocol.java
```

It also removed ProcessArguments-specific paths from the affected runtime
surfaces, including:

```text
Actor value transfer
parallel/P transfer
detached execution
diagnostic inspection
indexed interop
Bytecode/C-prime structured ProcessArguments.each machinery
native-boundary architecture expectations
```

The published change deleted the dedicated Bytecode/C-prime
`ProcessArguments.each` test and lowering/runtime machinery rather than
recreating equivalent privileged handling for Array.

Ordinary Array behavior is therefore the mechanism for indexed access,
iteration, interop and transferable value semantics.

## Environment preservation

I061 does not reduce Environment to an ordinary Map.

The dedicated Environment family and its native-name identity,
representability, stable bootstrap-content, deferred conversion and
`contains`/`get` semantics remain in place. The only removed Environment
contract is canonical object identity across independent accessor
acquisitions.

Tests were reconciled to validate Environment behavior/content without requiring
either identical or deliberately different object identity between independent
acquisitions.

## Normative reconciliation

The implementation advances the global specification revision to
`0.1.435`.

`spec/io/PROCESS_IO.md` now specifies:

- `process.args()` as a frozen ordinary Core Array of application-argument
  Strings;
- ordinary Array semantics for `size`, `at`, `each` and mutation failure on
  a frozen receiver;
- stable bootstrap content without canonical accessor-result identity;
- retained specialized Environment semantics without canonical reacquisition
  identity.

The corresponding specification changelog records that these D168 rules
supersede only the D018 requirements for canonical argument-snapshot identity,
special ProcessArguments representation and canonical repeated Environment
accessor identity. Other D018 Process I/O authority and lifecycle rules remain
unchanged.

The maintained Process I/O guide was reconciled with the new public model.

## Published change surface

The publication changed the Process representation/protocol, transfer,
diagnostic, interop and Bytecode surfaces plus their focused tests and
normative/documentation ownership.

Notable changed or removed paths include:

```text
spec/io/PROCESS_IO.md
spec/PROTOS_SPEC_CHANGELOG.md
docs/guide/11-process-io-filesystems-and-authority.md

src/main/java/com/guillermomolina/protos/runtime/ProtosProcessRuntime.java
src/main/java/com/guillermomolina/protos/runtime/ProtosProcessArgumentsValue.java         REMOVED
src/main/java/com/guillermomolina/protos/runtime/ProtosActorValueTransfer.java

src/main/java/com/guillermomolina/protos/execution/ProtosStandardProcessProtocol.java
src/main/java/com/guillermomolina/protos/execution/ProtosStandardProcessArgumentsProtocol.java REMOVED
src/main/java/com/guillermomolina/protos/execution/ProtosParallelRuntime.java
src/main/java/com/guillermomolina/protos/execution/ProtosDetachedExecutionValue.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosStandaloneProcessBootstrap.java
src/main/java/com/guillermomolina/protos/execution/ProtosProcessSnapshotExecution.java

protos/tests/conformance/process/snapshot-and-process-surface.protos

pom.xml
CHANGELOG.md
```

The implementation commit contains 403 additions and 1109 deletions across 32
paths, consistent with removal rather than replacement of the dedicated
ProcessArguments machinery.

## Validation

The exact published revision passed the repository's full GitHub Actions CI:

```text
CI_RUN_ID=36668328660
CI_RUN_NUMBER=2038
CI_HEAD_SHA=0a5a86965bafb713859335d17312b6bdabfbae8d
CI_CONCLUSION=success

TOOLCHAIN_VERIFICATION=PASS
MAVEN_CACHE_RESTORE=PASS
CANONICAL_COMMAND=make test

PARALLEL_JUNIT:
  tests=2141
  failures=0
  errors=0
  skipped=1
  result=BUILD SUCCESS

SERIAL_JUNIT:
  tests=7
  failures=0
  errors=0
  result=BUILD SUCCESS

PROTOS_TESTS_REPORTED_TIME=198s
CI_REPOSITORY_TESTS=PASS
```

Relevant passing coverage visible in the integrated run includes:

```text
ProtosProcessIntegratedConformanceTest
ProtosProcessArgumentsSnapshotTest
ProtosEnvironmentSnapshotTest
ProtosProcessCapabilityTransferTest
ProtosActorValueTransferTest
ProtosParallelExecutionTest
ProtosStandaloneProcessBootstrapTest
ProtosWorkspacePackageApplicationExecutionTest
ProtosCliTest
ProtosTestToolH2B3PublicIntegrationTest
```

The serial Java lane, including the previously isolated PERF017 JFR test,
also passed.

## Closure

```text
PROCESS_ARGS_CAPABILITY=KEEP
PROCESS_ARGS_CONTENT_STABLE=PASS
PROCESS_ARGS_ORDER_STABLE=PASS
PROCESS_ARGS_STRING_ONLY=PASS
PROCESS_ARGS_RESULT=FROZEN_ORDINARY_ARRAY

PROCESS_ARGUMENTS_FAMILY=REMOVED
PROCESS_ARGUMENTS_RUNTIME_VALUE=REMOVED
PROCESS_ARGUMENTS_NATIVE_PROTOCOL=REMOVED
PROCESS_ARGUMENTS_SPECIAL_TRANSFER_PATHS=REMOVED
PROCESS_ARGUMENTS_SPECIAL_INTEROP_DIAGNOSTIC_PATHS=REMOVED
PROCESS_ARGUMENTS_SPECIAL_BYTECODE_PATHS=REMOVED

CANONICAL_ARGS_ACCESSOR_IDENTITY=REMOVED
CANONICAL_ENVIRONMENT_ACCESSOR_IDENTITY=REMOVED

ENVIRONMENT_FAMILY=KEEP
ENVIRONMENT_NATIVE_NAME_SEMANTICS=PASS
ENVIRONMENT_REPRESENTABILITY=PASS
ENVIRONMENT_CONTAINS_GET=PASS
ENVIRONMENT_MAP_SUBSTITUTION=NO

NO_REPLACEMENT_ARGUMENTS_WRAPPER=PASS
NORMATIVE_SPEC_RECONCILIATION=PASS
FULL_REQUIRED_VALIDATION=PASS
CI_REQUIRED_FOR_CLOSURE=GREEN
I061_STATUS=CLOSED_COMPLETE
```
