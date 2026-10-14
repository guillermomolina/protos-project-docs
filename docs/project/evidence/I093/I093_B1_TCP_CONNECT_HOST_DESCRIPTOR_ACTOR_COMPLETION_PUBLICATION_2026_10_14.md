# I093-B1 — Published TCP connect host-descriptor and Actor-completion boundary

- **Date:** 2026-10-14.
- **Work item:** [I093 / guillermomolina/protos#873](https://github.com/guillermomolina/protos/issues/873).
- **Related architectural audit:** [AUD020 / #872](https://github.com/guillermomolina/protos/issues/872).
- **Prior handoff:** [AUD020 source-pinned I093-B review](../AUD020/AUD020_I093_B_DESCRIPTOR_AND_ACTOR_HANDOFF_SOURCE_REVIEW_2026_10_14.md), published at `d11a7ce55233e3ede7d213da3a72c351e4a81d34`.
- **PROTOS_REVISION:** `cf39ddbc0921a2af0f99bcb68a396e40a03a99a5` ([published product commit](https://github.com/guillermomolina/protos/commit/cf39ddbc0921a2af0f99bcb68a396e40a03a99a5)).
- **Implementation version:** `0.3.354-SNAPSHOT`.
- **Publication status:** I093-B1 **PUBLISHED** on `main`; I093-B2 and I093-C **PENDING**. I093 and AUD020 remain **OPEN**.

## Scope of the published checkpoint

I093-B1 is a deliberately bounded **connect-only** implementation checkpoint; it does **not** claim the whole I093-B guest/host networking separation.

1. `ProtosHostIpEndpoint` is a private, immutable, defensively copied host descriptor carrying only IP version 4/6, 4/16 network-order octets, and a concrete TCP port in 1–65535. It carries no guest object, guest prototype, Prelude, or Java `BigInteger`.
2. `ProtosStandardNetworkProtocol.connectTcp` performs canonical, callback-free guest endpoint recognition and exact numeric-to-octet conversion before passing the host descriptor through `ProtosNetworkConnectFlow.Backend`.
3. The NIO **connect** path consumes `ProtosHostIpEndpoint`, materializes Java `InetAddress`/`InetSocketAddress` under host/network authority, verifies family and IPv6 scope/address-bit preservation, and reports the connected local address as another inert host descriptor. No connect-path NIO guest IP object construction remains.
4. `ProtosNetworkConnectFlow` routes asynchronous successful/failed acquisition outcomes through the existing targeted owning-Actor completion facility. The Actor-side materializer creates and freezes destination-owned canonical `IpAddress` and `IpEndpoint` objects, then materializes the TCP connection capability. It avoids invoking source closures synchronously from a Task or on the NIO poller.
5. The implementation retains precommit cancellation, duplicate/late callback release, host-custody transfer and Actor-termination handling. The existing listener/accept-dependent `integerPrototype` is intentionally retained in NIO/provisioning: removing it during B1 initially broke compilation and was corrected without expanding B1 into listen/accept.

No normative specification, public selector, public module surface or new mandatory Task mechanism was changed. The canonical Protos-source IP families and private octet codec from I093-A remain the semantic base.

## Exact published files

The product commit changes **17 paths**: seven production Java files, eight JUnit test files, and the two publication-metadata files.

**Production:**
- `src/main/java/com/guillermomolina/protos/runtime/ProtosHostIpEndpoint.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosNetworkConnectFlow.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosActorExecutionDomain.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardNetworkProtocol.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosNioNetworkBackend.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosNioNetworkHost.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosEmbeddedNetworkCustody.java`

**JUnit:**
- `src/test/java/com/guillermomolina/protos/execution/ProtosNetworkConnectAcquisitionTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTcpConnectionIntegratedConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNioNetworkBackendConnectTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNioTcpConnectionReadTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNioTcpConnectionWriteTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNioTcpConnectionLifecycleTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNetworkingFoundationFinalConformanceTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosNioNetworkProvisioningTest.java`

**Metadata:** `pom.xml`, `CHANGELOG.md` (implementation version `0.3.354-SNAPSHOT`).

## Validation and repair provenance

Validation operations were executed by the **human maintainer** against the local product checkout, not independently by this documenting agent.

- First focused Maven battery: **38 tests, 0 failures, 0 errors, 0 skipped**.
- An intermediate `make test` run failed in exactly two network provisioning/foundation conformance tests because their standalone `awaitTerminal` wait loops slept without dispatching the newly targeted Actor-completion queue. That run reported **3,589 tests; 2 failures; 0 errors; 1 skipped**.
- The two wait helpers were updated to dispatch `future.domain().dispatchOne()` before sleeping when no work is runnable; the production Actor boundary was **not** reverted to poller-thread guest materialization.
- Following the final substantive fixes, the maintainer explicitly reported **`make test -> PASS`** and **`make check -> PASS`**. The maintainer also reported **`git diff --check` clean** and all local tests passing before publication.
- Publication metadata were finalized only after green validation. No behavioral test rerun after `pom.xml`/`CHANGELOG.md` finalization is claimed.
- The product commit and its complete list of 17 changed files were subsequently verified through GitHub at the exact revision above.

This evidence is a publication and human-validation record; it is **not** an independent Native Image build, benchmark result, full I093 architectural proof, or independently retained raw `make test` log.

## Remaining work and boundaries

**I093-B2:** migrate `listenTcp`, listener acquisition, accepted connections, listener-local/remote endpoints and related provision/custody paths to the appropriate descriptor-only host interface. Eliminate remaining NIO guest-object/prototype fields, constructor parameters, numeric-value minting and callback-time guest materialization. Keep exact IP family, scope, port, authority, cancellation, isolation and resource-lifecycle semantics. Do not treat the intentional B1 listener compatibility field as final architecture.

**I093-C:** check the complete two-way guest/host networking graph; all source and Java coupling/ABI surfaces; full IPv4/IPv6, mapped/large IPv6, link-local, cancellation, Actor/P/Context and Native Image-relevant gates; compare source-level coupling evidence and measured costs rather than assuming a speedup or numerical-ABI saving. Close I093 only after its entire acceptance contract and any required project record are published.

**AUD020:** remains independently open for its broader architectural audit matrix and possible owner-level D172/D173 discussion. I093-B1 neither closes AUD020 nor ratifies a public semantic change; I090 and I091 remain independent.

```text
I093_A=PUBLISHED
I093_B1=PUBLISHED
I093_B2=PENDING
I093_C=PENDING
PROTOS_REVISION=cf39ddbc0921a2af0f99bcb68a396e40a03a99a5
IMPLEMENTATION_VERSION=0.3.354-SNAPSHOT
PRODUCT_MAKE_TEST=PASS_HUMAN_REPORTED
PRODUCT_MAKE_CHECK=PASS_HUMAN_REPORTED
PRODUCT_DIFF_CHECK=CLEAN_HUMAN_REPORTED
NATIVE_IMAGE_GATE=NOT_CLAIMED
I093_PARENT_ISSUE=OPEN
AUD020_ISSUE=OPEN
```
