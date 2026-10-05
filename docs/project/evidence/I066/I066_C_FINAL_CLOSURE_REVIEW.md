# I066-C — final closure review

Date: 2026-10-05

## Work identity

```text
WORK_ITEM=I066
CLOSURE_ACTIVITY=I066-C
PROTOS_ISSUE=guillermomolina/protos#669
DECISION_AUTHORITY=D172/guillermomolina/protos#644
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
```

This record is durable non-normative closure evidence. It does not replace the
normative Protos specification or live GitHub coordination state.

## Exact reviewed product state

```text
CURRENT_PROTOS_HEAD=3c9738f5835cc8ed2e43d50fb8edcc7eb956ddc9
I066_B_REVISION=04189acc0021ba3514e9937113efdb98b176b93e
I066_B_IS_ANCESTOR_OF_HEAD=YES
POST_I066_COMMITS_REVIEWED=3c9738f5835cc8ed2e43d50fb8edcc7eb956ddc9
POST_I066_RELEVANT_REGRESSION=NO
I066_B_IMPLEMENTATION_VERSION=0.3.212-SNAPSHOT
I066_B_SPECIFICATION_REVISION=0.1.442
```

During closure publication, `main` advanced by one commit after I066-B:
`3c9738f5835cc8ed2e43d50fb8edcc7eb956ddc9` (`TEST009-K: cut remaining
compiler expansion debt`). The closure review therefore re-audited that exact
post-I066 delta instead of assuming I066-B remained HEAD.

The post-I066 commit changes only:

- `CHANGELOG.md`;
- `pom.xml`;
- `ProtosBytecodeRootNode.java`;
- `ProtosLanguageContext.java`;
- `ProtosLexicalFallback.java`;
- `ProtosTextReader.java`;
- `ProtosTextReaderLineProtocolTest.java`.

The potentially relevant lexical/runtime patches add Truffle host boundaries and
refactor residual bare-assignment destination selection; they do not change bare
read semantics, Prelude publication, module creation/cache semantics, `std:network`
placement, IP family identity/recognition, Actor/P transfer, or Network/TCP/NIO
consumption. The TextReader changes are unrelated. No relevant I066 regression
is present in current HEAD.

## Final validation evidence

The maintainer reported after the final review:

```text
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_FULL_VALIDATION=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

Exact-sha GitHub Actions evidence for the same product revision is:

```text
CI_WORKFLOW=CI
CI_RUN_ID=37310369030
CI_RUN_NUMBER=2158
CI_HEAD=04189acc0021ba3514e9937113efdb98b176b93e
CI_STATUS=completed
CI_CONCLUSION=success
CI_JOB=test
CI_JOB_ID=111764009359
CI_JOB_CONCLUSION=success
CI_REPOSITORY_TESTS_STEP=success
```

The repository workflow's `Run repository tests` step executes canonical
`make test`; that exact-sha job completed successfully.

## Closure review result

The falsifying final review found no missing I066 implementation scope and no
post-publication regression.

### D172 placement

- public Prelude bindings `IpAddress` and `IpEndpoint` remain absent;
- the canonical frozen runtime families are retained privately by `ProtosPrelude`;
- `std:network/IpAddresses.IpAddress` and
  `std:network/IpEndpoints.IpEndpoint` expose those exact runtime identities;
- Standard Library module instances remain Actor-local while the immutable
  canonical family objects are shared as specified.

### Module/runtime architecture

The Standard Library initial-member mechanism remains a general immutable
`ProtosModuleKey -> members` seam. The canonical module creation paths reviewed
continue to install registered standard members before their required
cache/source-execution points. Generic module lifecycle code contains no
IP-specific loader or cache path.

### D048 preservation

The D048 numeric data contract remains intact:

- fresh ordinary frozen values;
- exact immediate canonical parent;
- exact own-state recognition;
- callback-free direct recognition;
- numeric IPv4/IPv6 and endpoint validation;
- structural equality and hashing;
- ordinary object identity for `===`;
- Map-key coherence.

The final product state does not introduce branded
`ProtosIpAddressValue`/`ProtosIpEndpointValue` language semantics or shape-only
recognition.

### Actor/P transfer

Actor and P transfer preserve fresh logical copies with the exact canonical
family parent and do not resolve, load, or execute `std:network` modules as a
hidden transfer effect. The existing runtime-standard anchor mechanism is reused;
no second transfer framework exists.

The explicit cross-Prelude detached-snapshot path was also inspected as a
falsification target. The observed limitation there predates I066 and is
unchanged by I066-B; it is not a D172 regression or an unmet #669 closure gate.

### Network/TCP/NIO

Network/TCP/NIO consumers use the runtime-retained canonical identities rather
than public Prelude bindings. The reviewed paths preserve numeric-only endpoint
semantics, do not introduce DNS as a hidden effect, do not confer Network
authority from address data, and do not introduce implicit IPv4-mapped-IPv6
normalization.

### Normative convergence

`spec/io/NETWORK.md` records D172 placement explicitly and remains consistent
with Actor-local module ownership, shared frozen standard identities, D048
construction/recognition/equality/hash semantics, and the no-implicit-import /
no-authority rules. No contradictory current normative owner was found.

## Material inspection set

The final review materially inspected current product state including:

- `protos/lib/core/prelude.protos`;
- `protos/lib/network/IpAddresses.protos`;
- `protos/lib/network/IpEndpoints.protos`;
- `src/main/java/com/guillermomolina/protos/runtime/ProtosPrelude.java`;
- `src/main/java/com/guillermomolina/protos/runtime/ProtosActorValueTransfer.java`;
- `src/main/java/com/guillermomolina/protos/execution/ProtosCoreBootstrap.java`;
- `src/main/java/com/guillermomolina/protos/execution/ProtosModuleRuntime.java`;
- `src/main/java/com/guillermomolina/protos/execution/ProtosCanonicalInitialModuleExecution.java`;
- `src/main/java/com/guillermomolina/protos/execution/ProtosParallelRuntime.java`;
- `src/main/java/com/guillermomolina/protos/execution/ProtosDetachedExecutionValue.java`;
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardNetworkProtocol.java`;
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardTcpConnectionProtocol.java`;
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardTcpListenerProtocol.java`;
- `src/main/java/com/guillermomolina/protos/execution/ProtosNioNetworkHost.java`;
- `src/main/java/com/guillermomolina/protos/execution/ProtosNioNetworkBackend.java`;
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardIpAddressProtocol.java`;
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardIpEndpointProtocol.java`;
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardIpFamilyPlacementTest.java`;
- `protos/tests/conformance/network/ip-address.protos`;
- `protos/tests/conformance/network/ip-endpoint.protos`;
- `spec/io/NETWORK.md`;
- `spec/semantics/MODULES.md`;
- `spec/concurrency/ACTORS.md`;
- `spec/concurrency/PARALLEL_EXECUTION.md`;
- `spec/runtime/ABSTRACT_RUNTIME.md`;
- `.github/workflows/tests.yml`;
- `AGENTS.md`;
- `AGENTS.work/IMPLEMENTATION.md`;
- `AGENTS.work/COORDINATION.md`;
- the I066-A and I066-B durable evidence records.

## Closure matrix

```text
D172_PRELUDE_REMOVAL=PASS
D172_STDLIB_CANONICAL_EXPOSURE=PASS
D048_CONSTRUCTION_PRESERVED=PASS
D048_RECOGNITION_PRESERVED=PASS
EQUALITY_HASH_MAP_KEY_PRESERVED=PASS
ACTOR_TRANSFER_PRESERVED=PASS
P_TRANSFER_PRESERVED=PASS
NETWORK_TCP_CONSUMPTION_PRESERVED=PASS
NO_IP_SPECIFIC_MODULE_SYSTEM=PASS
NORMATIVE_SPEC_RECONCILIATION=PASS
FOCUSED_VALIDATION=PASS
REQUIRED_FULL_VALIDATION=PASS
PUBLICATION_VALIDATION=PASS
```

## Final disposition

The previous closure blocker was only missing evidence for the required
`git diff --check` gate. The maintainer has now reported that gate clean, all
local tests PASS, and the exact published product SHA has green remote CI.

```text
I066_PRODUCT_COMPLETE=YES
I066_CLOSURE_AUTHORIZED=YES
NEXT_TECHNICAL_SLICE=NONE
BLOCKER_CLASS=NONE
ISSUE_669_TARGET_STATE=COMPLETED
```

No I066-D product slice is justified.
