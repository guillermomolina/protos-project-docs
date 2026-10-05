# I066-B — canonical IP family placement migration implementation

Date: 2026-10-05

## Work identity

~~~text
WORK_ITEM=I066
IMPLEMENTATION_SLICE=I066-B
PROTOS_ISSUE=guillermomolina/protos#669
DECISION_AUTHORITY=D172/guillermomolina/protos#644
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative implementation evidence. It does not replace
the normative Protos specification or the live GitHub Issue state.

## Exact published product state

~~~text
PROTOS_REVISION=04189acc0021ba3514e9937113efdb98b176b93e
PARENT_REVISION=5a3c5e4a274dee3b815746a21008c4cbedf4c1c1
COMMIT_SUBJECT=I066-B: move canonical IP families to std:network
IMPLEMENTATION_VERSION=0.3.212-SNAPSHOT
SPECIFICATION_REVISION=0.1.442
PRODUCT_PUBLICATION=PUSHED
~~~

The commit changes 44 paths, with 681 additions and 86 deletions.

## Maintainer-reported validation

The maintainer reported after publication:

~~~text
LOCAL_FULL_VALIDATION=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

No stronger claim is inferred from source inspection alone.

At evidence-capture time, exact-sha GitHub Actions state is:

~~~text
CI_WORKFLOW=CI
CI_RUN_ID=37310369030
CI_RUN_NUMBER=2158
CI_HEAD=04189acc0021ba3514e9937113efdb98b176b93e
CI_STATUS=in_progress
CI_CONCLUSION=NONE_YET
~~~

Therefore remote CI success is not yet claimed and I066 is not closed by this
record.

## Implemented D172 placement

The public Core Prelude no longer publishes either canonical IP family:

~~~text
Prelude.IpAddress=REMOVED
Prelude.IpEndpoint=REMOVED
~~~

The same canonical frozen identities are retained privately by `ProtosPrelude`
and are exposed to Actor-local Standard Library modules as:

~~~text
std:network/IpAddresses.IpAddress
std:network/IpEndpoints.IpEndpoint
~~~

The implementation keeps one runtime-owned canonical family of each kind per
Prelude/Process while ordinary module instances remain Actor-local.

## General Standard-Library initial-member seam

`ProtosPrelude` now owns a general immutable mapping from `ProtosModuleKey` to
initial frozen standard members. I066 configures:

~~~text
std:network/IpAddresses
    IpAddress -> runtime canonical IpAddress

std:network/IpEndpoints
    IpEndpoint -> runtime canonical IpEndpoint
~~~

The generic module lifecycle does not contain IP-specific module-name logic.

The seam is installed in every canonical module-creation path identified by the
approved I066-A investigation:

- `ProtosModuleRuntime.prepareBytecodeCanonicalModule`;
- `ProtosModuleRuntime.loadCanonicalModuleInternal`;
- `ProtosCanonicalInitialModuleExecution.execute`.

Installation occurs before cache insertion/source execution where required by
the respective lifecycle.

## Runtime-retained canonical identities

`ProtosPrelude` now retains:

~~~text
runtimeIpAddressPrototype
runtimeIpEndpointPrototype
~~~

with runtime accessors and exact identity recognition. Bootstrap removes the
temporary public bootstrap slots before the public Prelude bindings are frozen,
retains the two exact canonical family objects separately, validates them as part
of the frozen standard graph, and configures the Standard Library member seam
from those same objects.

D048 protocol implementation is not replaced by branded host values or
Actor-local replacement prototypes.

## Transfer preservation

The published implementation extends existing standard-anchor handling to the
two runtime-retained IP family identities in:

- Actor snapshot/transfer;
- P/parallel transfer;
- detached execution snapshot/copy.

Recognized IP occurrences remain fresh logical copies while their immediate
parent remains the exact canonical runtime family.

No second transfer framework is introduced.

A new deterministic placement/transfer test instruments module resolution/source
loading and proves Actor/P transfer does not resolve/load the IP Standard Library
modules as a hidden transfer effect.

## Network/TCP preservation

Network and TCP consumers no longer recover canonical IP families from public
Prelude slots. They use the runtime-retained identities.

The changed implementation covers:

- Network connect validation;
- Network listen/address validation;
- NIO Network provisioning;
- TCP connection endpoint materialization/observation;
- TCP listener endpoint materialization/accept;
- NIO TCP read/write/lifecycle/listener paths.

The existing numeric IP protocols continue to own exact construction,
recognition, equality and hashing behavior.

## Normative reconciliation

`spec/io/NETWORK.md` now defines the D172 placement explicitly:

~~~text
std:network/IpAddresses.IpAddress
std:network/IpEndpoints.IpEndpoint
~~~

It specifies that:

- there is one frozen canonical family of each kind per Process;
- Actor-local module instances remain distinct;
- those module instances may share the exact frozen canonical family identity;
- runtime transfer/Network/TCP access is not a Prelude binding;
- runtime access is not an implicit import;
- runtime access creates no module instance as a side effect;
- the data confers no Network authority;
- transfer preserves the canonical parent.

D048 construction, exact recognition, structural equality/hash and numeric-only
semantics remain unchanged.

Specification changelog revision `0.1.442` records the new placement rather than
rewriting the historical D048 record.

## Focused test evidence in the published candidate

The product delta updates existing conformance/library/network/Actor/P/TCP tests
and adds:

~~~text
src/test/java/com/guillermomolina/protos/execution/ProtosStandardIpFamilyPlacementTest.java
~~~

That test contains focused guards for:

- public Prelude absence;
- frozen retained canonical family identities;
- general/keyed Standard Library initial-member seam;
- absence of IP-specific knowledge in generic module lifecycle classes;
- Actor transfer preserving canonical parent without additional module resolve/load;
- P transfer preserving canonical parent without additional module resolve/load.

Existing D048 construction/recognition/equality/hash/Map-key and Network/TCP
regression coverage is retained/adapted to import the canonical families from
`std:network`.

## Exact changed paths

~~~text
CHANGELOG.md
pom.xml
protos/lib/core/prelude.protos
protos/tests/conformance/actor/lifecycle-and-transfer.protos
protos/tests/conformance/core-surface/required-core-bindings.protos
protos/tests/conformance/network/ip-address.protos
protos/tests/conformance/network/ip-endpoint.protos
protos/tests/library/network/ip-addresses/parse-format.protos
protos/tests/library/network/ip-addresses/surface.protos
protos/tests/library/network/ip-endpoints/parse-format-and-validation.protos
protos/tests/tooling/parallel-execution-ip-data-transfer.protos
spec/PROTOS_SPEC_CHANGELOG.md
spec/io/NETWORK.md
src/main/java/com/guillermomolina/protos/execution/ProtosCanonicalInitialModuleExecution.java
src/main/java/com/guillermomolina/protos/execution/ProtosCoreBootstrap.java
src/main/java/com/guillermomolina/protos/execution/ProtosDetachedExecutionValue.java
src/main/java/com/guillermomolina/protos/execution/ProtosModuleRuntime.java
src/main/java/com/guillermomolina/protos/execution/ProtosNioNetworkHost.java
src/main/java/com/guillermomolina/protos/execution/ProtosParallelRuntime.java
src/main/java/com/guillermomolina/protos/execution/ProtosStandardNetworkProtocol.java
src/main/java/com/guillermomolina/protos/execution/ProtosStandardTcpConnectionProtocol.java
src/main/java/com/guillermomolina/protos/execution/ProtosStandardTcpListenerProtocol.java
src/main/java/com/guillermomolina/protos/runtime/ProtosActorValueTransfer.java
src/main/java/com/guillermomolina/protos/runtime/ProtosPrelude.java
src/test/java/com/guillermomolina/protos/execution/ProtosCoreBootstrapTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosCoreNativeBoundaryArchitectureTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosExactExecutionFacilityTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosNetworkConnectAcquisitionTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosNetworkListenAcquisitionTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosNetworkingFoundationFinalConformanceTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosNetworkingIpAddressesModuleTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosNetworkingIpEndpointsModuleTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosNioNetworkBackendConnectTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosNioNetworkProvisioningTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosNioTcpConnectionLifecycleTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosNioTcpConnectionReadTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosNioTcpConnectionWriteTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosNioTcpListenerBackendTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosParallelExecutionTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosStandardIpFamilyPlacementTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosTcpConnectionEndpointObservationTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosTcpConnectionIntegratedConformanceTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosTcpListenerAcceptTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosTcpListenerIntegratedConformanceTest.java
~~~

## License compliance

The only new Protos-owned source file in this product commit,
`ProtosStandardIpFamilyPlacementTest.java`, carries the project Part 5 license
notice. Modified Protos-owned sources retain their notices. No vendored or
third-party file is introduced.

## Implementation assessment

Inspection of the exact published revision finds no missing technical scope from
the approved I066-A implementation shape.

~~~text
D172_PRELUDE_REMOVAL=IMPLEMENTED
D172_STDLIB_CANONICAL_EXPOSURE=IMPLEMENTED
D048_CONSTRUCTION_PRESERVED=IMPLEMENTED_AND_LOCALLY_VALIDATED
D048_RECOGNITION_PRESERVED=IMPLEMENTED_AND_LOCALLY_VALIDATED
EQUALITY_HASH_MAP_KEY_PRESERVED=IMPLEMENTED_AND_LOCALLY_VALIDATED
ACTOR_TRANSFER_PRESERVED=IMPLEMENTED_AND_LOCALLY_VALIDATED
P_TRANSFER_PRESERVED=IMPLEMENTED_AND_LOCALLY_VALIDATED
NETWORK_TCP_CONSUMPTION_PRESERVED=IMPLEMENTED_AND_LOCALLY_VALIDATED
NO_IP_SPECIFIC_MODULE_SYSTEM=IMPLEMENTED
NORMATIVE_SPEC_RECONCILIATION=IMPLEMENTED
FOCUSED_VALIDATION=PASS_MAINTAINER_REPORTED
REQUIRED_FULL_VALIDATION=PASS_MAINTAINER_REPORTED
PUBLICATION_VALIDATION=PENDING_REMOTE_CI
~~~

## Next activity

I066-B is a closure candidate. No further implementation slice is currently
identified.

The next activity is a falsifying final closure review against current product
HEAD and current remote CI state. That review should look for post-I066
regressions or an unfulfilled #669 closure gate; it must not invent a new product
slice merely because CI was still running when this evidence was captured.

~~~text
I066_B_IMPLEMENTATION=COMPLETE
I066_PRODUCT_CLOSURE_CANDIDATE=YES
NEXT_IMPLEMENTATION_SLICE=NONE
NEXT_ACTIVITY=I066-C_FINAL_CLOSURE_REVIEW
NEXT_ACTIVITY_TYPE=INVESTIGATION_REVIEW
ISSUE_669_STATUS=REVIEW
I066_CLOSURE_AUTHORIZED=NO
~~~
