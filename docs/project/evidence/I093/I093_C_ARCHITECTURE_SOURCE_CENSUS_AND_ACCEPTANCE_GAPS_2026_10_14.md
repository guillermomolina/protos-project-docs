# I093-C — Source-architecture census and outstanding acceptance gates (2026-10-14)

**Classification:** read-only, non-normative architectural review / partial acceptance evidence. It is **not** a performance benchmark, a Native TCP-with-Network integration test, an amendment to D172/D173, or full closure of [I093/#873](https://github.com/guillermomolina/protos/issues/873) or [AUD020/#872](https://github.com/guillermomolina/protos/issues/872).

- **Current published product inspected:** `guillermomolina/protos@8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff` ([commit](https://github.com/guillermomolina/protos/commit/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff)), version `0.3.356-SNAPSHOT`.
- **Pre-B1 comparison source:** `97404f1cc1c1d678c66ef598b47cf62d32014f21`.
- **I093-B1 source:** `cf39ddbc0921a2af0f99bcb68a396e40a03a99a5`.
- **I093-B2 source:** `f8e2d4e3dcf9b5e13d4850e14605248cae326125`.
- **I093-C1 publication:** [Actor termination custody](https://github.com/guillermomolina/protos-project-docs/blob/ab22912320469fc37793e15427f8b86193a06ece/docs/project/evidence/I093/I093_C1_ACTOR_TERMINATION_DISCARD_CUSTODY_PUBLICATION_2026_10_14.md).
- **Normative anchors:** [NETWORK.md](https://github.com/guillermomolina/protos/blob/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff/spec/io/NETWORK.md); [IO_CORE.md](https://github.com/guillermomolina/protos/blob/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff/spec/io/IO_CORE.md); [ACTORS.md](https://github.com/guillermomolina/protos/blob/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff/spec/concurrency/ACTORS.md); [FUTURES_AND_TASKS.md](https://github.com/guillermomolina/protos/blob/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff/spec/concurrency/FUTURES_AND_TASKS.md); [PROCESS_IO.md](https://github.com/guillermomolina/protos/blob/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff/spec/io/PROCESS_IO.md).

## 1. Reproducible narrow source-token delta

Counts below are *literal Java source token occurrences in the identified files* (including imports, comments and usages), not an object-allocation profile, complete dependency graph, linker ABI, or execution sample. Compare the exact same files between the two revisions. This small census is intended to test direct NIO/guest coupling, not prove transitive independence of all Java networking code.

| File | `ProtosObjectValue` before → after | `ProtosNumericValueSupport` before → after | `integerPrototype` + `addressPrototype` + `endpointPrototype` before → after |
| --- | ---: | ---: | ---: |
| `execution/ProtosNioNetworkBackend.java` | 28 → **0** | 13 → **0** | 29 → **0** |
| `execution/ProtosNioTcpListenerBackend.java` | 16 → **0** | 4 → **0** | 17 → **0** |
| **Combined direct NIO surface** | **44 → 0** | **17 → 0** | **46 → 0** |

At the published C1 revision, `ProtosNioTcpConnectionBackend.java`, `ProtosNioHostIoPoller.java` and immutable `ProtosHostIpEndpoint.java` likewise contain no literal `ProtosObjectValue`, `ProtosNumericValueSupport`, `ProtosPrelude`, or Java `BigInteger` references. The first two classes already had zero such direct references at the earlier pre-B1 revision.

## 2. Two-way responsibility matrix at the product commit

| Responsibility | Current owner | Evidence / limits |
| --- | --- | --- |
| Canonical `IpAddress` and `IpEndpoint` construction, structural `==`, `hash` | `protos/lib/core/IpAddress.protos`, `IpEndpoint.protos` | Source-backed protocol installed into the exact Process-shared frozen family; Java remains responsible for minimal secure `recognizes` recognition |
| Parsing, formatting and private octet codec | `protos/lib/network/IpAddresses.protos`, `IpEndpoints.protos` | Ordinary Protos semantics; private codec is not a public host or Java provider ABI |
| No-callback authentication of untrusted candidate objects and exact numeric ingress conversion | `ProtosStandardIpAddressProtocol.java`, `ProtosStandardIpEndpointProtocol.java`, `ProtosStandardNetworkProtocol.java` | Bounded guest-facing Java adapter; cannot be moved to the NIO poller or replaced by candidate callbacks |
| Descriptor transport | `ProtosHostIpEndpoint`; `ProtosNetworkConnectFlow`, `ProtosNetworkListenFlow`, `ProtosTcpListenerFlow` | Connect/accept carry immutable IPv4/IPv6 octets, family, nonzero port; listen carries optional address octets and optional requested port |
| Physical NIO connect, bind, listen, accept, scope and selectors | `ProtosNioNetworkBackend`, `ProtosNioTcpListenerBackend`, `ProtosNioTcpConnectionBackend`, `ProtosNioHostIoPoller` | No direct guest-endpoint/prototype/numeric references in the two principal backend classes; host-only scope and socket duties remain |
| Guest value and capability materialization after acquisition | `ProtosStandardNetworkProtocol`, `ProtosStandardTcpListenerProtocol`, owning Actor/Prelude | Native-facing adapters legitimately depend on guest object/numeric families; returned host descriptors are reconstructed only with destination Actor execution rights |
| Actor handoff and resource cleanup | `ProtosActorExecutionDomain`, `ProtosNetworkConnectFlow`, `ProtosNetworkAcquisitionInvocation`, `ProtosIoOperation` | C1 supplies host-only `onDiscard` cleanup for queued completions lost to termination after a prior post-commit cancellation request |
| Lazy authority/poller provisioning | `ProtosNioNetworkHost`, `ProtosEmbeddedNetworkCustody`, `ProtosNetworkCapabilityValue` | Prelude participation remains at the capability/provisioning boundary, but NIO backend constructors no longer accept guest prototypes or arbitrary-precision numeric families |
| Actor and P transfer boundary | `ProtosActorValueTransfer` and parallel runtime; Network/TCP resource wrappers | Canonical IP data can be copied; live `Network`, `TcpConnection` and `TcpListener` capabilities are not implicitly transferable |

**Architectural finding:** the intended source-backed guest value semantics and direct host-only NIO endpoint representation are separated. Residual `ProtosPrelude`, `ProtosObjectValue` and numeric tokens in standard-boundary Java classes are not, without further proof, violations; they are part of authentication, canonical projection and Actor-local ownership. This is a source analysis, not proof that a representation update elsewhere can never affect an adapter.

## 3. Observed contract coverage

| Contract axis | Evidence currently available | Status / limitation |
| --- | --- | --- |
| IPv4/IPv6 exact versions, unsigned IPv6 128-bit values, source equality/hash | Source-backed IP conformance; numeric and endpoint acquisition JUnit; Native IP data 10+8+2 | Covered by existing test gates; Native TCP not implied |
| Host connect/listen/accept, concurrent accepts, NIO family and scope | JUnit tests for NIO connect, NIO listener, integrated TCP lifecycle | Existing host/JVM tests, plus reported full integrated `make test` PASS |
| Invalid guest request before backend effects | Connect/listen/accept acquisition tests and no-callback recognizers | Existing JUnit and normative secure-shape recognition |
| Cancellation/late/duplicate callback custody | Network acquisition test groups; I093-C1 added post-commit cancellation + Actor termination cleanup regression for all three families | C1 focused 29 tests PASS; not a full concurrency interleaving/model-check proof |
| Actor/P data transfer and live capability non-transfer | `ProtosStandardIpFamilyPlacementTest`, `ProtosNetworkingFoundationFinalConformanceTest`, native Protos transfer conformance | Tests exercised within integrated `make test`; no new public transfer authority |
| Multiple Contexts and lazy/no-use poller | `ProtosEmbeddedNetworkTest`, `ProtosEmbeddedNetworkCustodyTest`, `ProtosNioNetworkProvisioningTest` | Tests exercised within reported integrated `make test`; direct source shows lazy acquisition |
| Native Image basic, IP data and interpreter-only policy | Maintainer reports Native PASS; 10 address + 8 endpoint + 2 endpoint parse/format cases | Native guest execution demonstrated, but **authorized Native TCP connect/listen/accept NOT_VERIFIED** |
| CPU, heap allocations, native image footprint attributable to networking, connect/accept latency | No version-matched benchmark or allocations supplied | **NOT_MEASURED**; source token reduction is not a runtime speedup |
| Complete independent AUD020 matrix and D172/D173 owner checkpoint | AUD020 remains open | **INCOMPLETE**; no normative change is ratified |

## 4. Provenance and remaining acceptance actions

The maintainer reports, for C1 substantive implementation: **29 focal tests PASS**, `make test` **PASS**, `make check` **PASS**, Native **PASS**, three Native IP files totaling **20 passed / 0 failed**, and a clean `git diff --check`. The later publishing shell reported `mvn: orden no encontrada`; this is **not** another green run. Product metadata finalized at `0.3.356-SNAPSHOT`, then code/test/metadata were committed and pushed as one eight-path product commit. All validation statements are maintainer-reported, not independently rerun or raw logs retained here.

For *full* I093-C acceptance, obtain either an actual Native TCP-with-explicit-Network gate or a precisely justified decision documenting why the current Native CLI does not provision that authority; produce grounded measurements or explicitly obtain owner agreement that `NOT_MEASURED` is an acceptable closure limitation; and finish the contract-coverage review of any remaining untested Actor/P/Context, link-local and commitment interleavings. Escalate any newly discovered externally observable semantic ambiguity to the correct formal D/PLAT owner; do not silently change D172/D173.

C1 itself is published and its repair validated by the maintainer. **No additional product patch is requested by this static checkpoint.** Do not create a new I-family issue simply to repeat acceptance tasks already owned by I093-C. **I093 and AUD020 stay OPEN** until their separate outstanding criteria are dealt with.

```text
PROTOS_REVISION=8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff
I093_C1=PUBLISHED
NIO_GUEST_DIRECT_COUPLING_SOURCE_CENSUS=ZERO_MAIN_NIO_ENDPOINT_CLASSES
NATIVE_IP_DATA_TESTS=20_PASS_HUMAN_REPORTED
NATIVE_TCP_WITH_NETWORK=NOT_VERIFIED
ALLOCATIONS_AND_LATENCY=NOT_MEASURED
I093_C_FINAL=OPEN
AUD020=OPEN
```
