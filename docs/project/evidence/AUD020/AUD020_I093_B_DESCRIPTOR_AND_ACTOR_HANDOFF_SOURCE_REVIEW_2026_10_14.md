# AUD020 — I093-B descriptor and Actor handoff source review (2026-10-14)

**Record role:** immutable, non-normative, read-only architecture evidence and implementation handoff. This is **not** a final AUD020 audit, a ratification of D172/D173, a product implementation, or a benchmark result.

- Audit: [AUD020/#872](https://github.com/guillermomolina/protos/issues/872).
- Implementation owner: [I093/#873](https://github.com/guillermomolina/protos/issues/873).
- Pinned current inspected product: `guillermomolina/protos@97404f1cc1c1d678c66ef598b47cf62d32014f21`.
- Inspected documentation baseline: `guillermomolina/protos-project-docs@7c1666c4739980341ac4fad6e82d912fe2a31983`.
- Earlier owner-approved bounded direction: [AUD020 evidence, 2026-10-11](./AUD020_PROTOS_FIRST_HOST_NETWORK_BOUNDARY_OWNER_DIRECTION_2026_10_11.md).
- Published prerequisite: [I093-A product b47be4b698ca27b537898abee89dd6f4c8c49fc3](https://github.com/guillermomolina/protos/commit/b47be4b698ca27b537898abee89dd6f4c8c49fc3); [I093-A durable evidence](../I093/I093_A_SOURCE_BACKED_IP_PROTOCOL_AND_OCTET_CODEC_PUBLICATION_2026_10_11.md).
- Validation provenance: maintainer reports local tests PASS and `git diff --check` clean. No specific test logs, fresh I093-B changes, Native Image run, source runtime execution, or performance measurements were supplied. This record results from static source review only.

## 1. Current source-backed and native responsibilities

At the pinned product revision, `protos/lib/core/IpAddress.protos` and `IpEndpoint.protos` implement canonical factory invocation, `==` and `hash` in Protos source. `ProtosStandardIpAddressProtocol.java` and `ProtosStandardIpEndpointProtocol.java` retain the bounded **callback-free** native `recognizes` path, checking frozen state, exact immediate canonical parent, local-slot shape and numeric/address invariants. The shared Process family identity remains in the runtime/Core bootstrap, while family publication is in `std:network`.

`protos/lib/network/IpAddresses.protos` contains private `_ipOctets.from(version, octets)` and `_ipOctets.to(address)`, which use exact Protos Integer arithmetic and 4/16 network-order octets, preserve leading zeros and 128-bit values (including above signed 64-bit). The private binding is removed from the module surface after closures capture it; it is **not** a ready-made Java-side ABI. `IpEndpoints.protos` retains endpoint text parsing/formatting. Avoid exporting the codec merely to complete a backend refactor.

Source anchors at the exact inspected revision:

- [core IpAddress](https://github.com/guillermomolina/protos/blob/97404f1cc1c1d678c66ef598b47cf62d32014f21/protos/lib/core/IpAddress.protos)
- [core IpEndpoint](https://github.com/guillermomolina/protos/blob/97404f1cc1c1d678c66ef598b47cf62d32014f21/protos/lib/core/IpEndpoint.protos)
- [IpAddresses source codec](https://github.com/guillermomolina/protos/blob/97404f1cc1c1d678c66ef598b47cf62d32014f21/protos/lib/network/IpAddresses.protos)

## 2. Concrete remaining guest/host coupling

| Internal owner | Pinned source observation | Recommended disposition |
| --- | --- | --- |
| `ProtosNioNetworkBackend.java` | Carries `listenAddressPrototype`, `listenEndpointPrototype`, `integerPrototype`; `HostEndpoint.decode` reflects guest `address`/`port` and `version`/`bits` slots; `HostEndpoint.materializeLocalEndpoint` builds/freeze guest IP values on successful connect. | Remove guest coupling through an immutable 4/16-octet + port descriptor; preserve NIO/scope and custody. |
| `ProtosNioTcpListenerBackend.java` | Constructor accepts all three guest prototypes; `materializeEndpoint(InetSocketAddress)` creates canonical guest address/endpoint for each accepted connection. | Return host descriptors; materialize in owning Actor domain. |
| `ProtosNioNetworkHost.java` | Provisioning passes canonical address/endpoint and `bigIntegerPrototype()` into NIO backend. | Stop threading guest prototypes into NIO endpoint handling. |
| `ProtosEmbeddedNetworkCustody.java` | Stores guest prototypes from Prelude before lazy NIO startup and passes them into backend creation. | Preserve first-use lazy host; remove guest dependencies when no NIO consumer remains. |
| `ProtosNetworkConnectFlow.java` | Backend `connect(ProtosObjectValue endpoint,...)`; completion `succeeded(..., ProtosObjectValue localEndpoint,...)`; result materialization immediately follows `operation.commit()` in the callback. | Transport inert host descriptors and defer guest materialization to owning Actor domain without changing commitment/cancellation contracts. |
| `ProtosNetworkListenFlow.java` | `ListenRequest(int ipVersion, ProtosObjectValue addressConstraint, Integer portConstraint)` retains guest address state in backend-facing request. | Replace address constraint with immutable host address data; keep semantics of null / ephemeral requested port. |
| `ProtosTcpListenerFlow.java` | Accept completion carries two `ProtosObjectValue` endpoints. | Change completion to immutable descriptors; preserve release on rejected/late handoff. |
| `ProtosStandardNetworkProtocol.java` | Secure authority preflight and canonical recognition occur before backend; materializers create live capability wrappers. | Own the narrow guest/host adaptation, no user-visible selectors or authority changes. |
| `ProtosStandardTcpListenerProtocol.java` | Recognizes materialized local/remote endpoints in accepted result. | Keep secure validation while using Actor-owned descriptor reconstruction. |
| `ProtosTcpConnectionValue.java` | Stores canonical guest logical-endpoint snapshots for `localEndpoint`/`remoteEndpoint` access. | Keep observable snapshot behavior and ownership; do not expose raw host descriptors. |

These are *direct guest representation/prototype coupling*, not evidence of direct `java.math.BigInteger` allocations in the NIO classes or proven CPU savings. The I091 historical token census must not be repackaged as measurements for this source revision.

## 3. Async ownership finding: existing reusable pattern

`ProtosNetworkConnectFlow`, `ProtosNetworkListenFlow` and `ProtosTcpListenerFlow` currently invoke their result materializers from backend callback continuations. A production NIO callback may originate on `ProtosNioHostIoPoller`. Simply changing endpoint parameter types is therefore insufficient: guest value creation, error occurrences and Future resolution must be delivered under the owning Actor's execution rights; terminal/cancelled delivery must release untransferred resource custody.

A concrete existing precedent is [`ProtosFilesystemOpenFlow.java`](https://github.com/guillermomolina/protos/blob/97404f1cc1c1d678c66ef598b47cf62d32014f21/src/main/java/com/guillermomolina/protos/runtime/ProtosFilesystemOpenFlow.java). Its `Invocation` distinguishes synchronous backend call from off-thread callbacks, publishes host callback work using `domain.enqueueTargetedFutureCompletionForRuntime(operation.future(), ...)`, protects pending deliveries with claim-once references, and releases custody if enqueue fails on termination or when termination discards queued completions. This is a *pattern* to reuse narrowly, not authorization to clone another scheduler or assume networking already has equivalent guards.

The existing [`ProtosActorExecutionDomain.java`](https://github.com/guillermomolina/protos/blob/97404f1cc1c1d678c66ef598b47cf62d32014f21/src/main/java/com/guillermomolina/protos/runtime/ProtosActorExecutionDomain.java) checks Future/domain match, rejects terminating Actors, and runs targeted completion only under Actor ownership. `ProtosIoOperation.commit()` and `resolve()` are distinct; ordering must be checked explicitly. Off-thread success, failure, synchronous backend completion, cancellation racing registration, duplicated callback, queued completion discarded on Actor termination and exception during materialization each need a custody outcome.

## 4. Source closure/Task constraint that must not be papered over

[`ProtosClosureInvoker.java`](https://github.com/guillermomolina/protos/blob/97404f1cc1c1d678c66ef598b47cf62d32014f21/src/main/java/com/guillermomolina/protos/execution/ProtosClosureInvoker.java) explicitly rejects synchronous ordinary method invocation inside a Task (`use C-prime`). As a result, invoking a source-backed `IpAddress.call` or `IpEndpoint.call` through that path during a Future result materialization **cannot be assumed valid** merely because it runs on the correct Actor. Prove both Task and non-Task paths using current execution entry machinery; do not introduce a compulsory RootTask, general privileged SPI or a network-specific guest invocation bypass. If compliant guest invocation cannot be achieved without modifying observable behavior, stop and raise an exact D/PLAT owner checkpoint rather than silently falling back to Java materialization.

The private `_ipOctets` codec's visibility/capture is another distinct constraint: it does not provide an exposed host converter. Investigate a minimal private runtime adapter that authenticates canonical shape and converts octets before backend entry, and after Actor-domain return constructs canonical source semantics. Any remaining bounded Java numeric byte extraction on the guest *boundary* must be explicitly justified rather than repeated inside NIO or conflated with public library semantics.

## 5. Normative invariants, test owners and falsifiers

Normative [`spec/io/NETWORK.md`](https://github.com/guillermomolina/protos/blob/97404f1cc1c1d678c66ef598b47cf62d32014f21/spec/io/NETWORK.md): canonical frozen exactly shaped IP values and callback-free recognition (§2-3), shared Process family identity (§3.1), strict IPv4/IPv6 and authorized IPv6 link-local scope (§4), explicit Network authority and host-free invalid-argument failure (§5), cancellation/late resource release (§6-7), accepted resource endpoint snapshots and non-transferability (§7.1-11), no mandatory poller/network activation in a networkless program (§12). Also re-check `spec/io/IO_CORE.md`, `spec/concurrency/FUTURES_AND_TASKS.md`, `spec/concurrency/ACTORS.md`, `spec/concurrency/PARALLEL_EXECUTION.md`, `spec/semantics/VALUES_AND_COLLECTIONS.md`, `spec/semantics/OBJECT_MODEL.md`, D048/D172/D173 and applicable Process/module authority.

Existing focal test owners inspected by name/test cases, not executed by this audit: `ProtosNetworkConnectAcquisitionTest`, `ProtosNetworkListenAcquisitionTest`, `ProtosTcpListenerAcceptTest`, `ProtosNioNetworkBackendConnectTest`, `ProtosNioTcpListenerBackendTest`, `ProtosEmbeddedNetworkCustodyTest`, `ProtosPerf019TargetedContinuationHandoffTest`. Their cases cover independent acquisitions, invalid preflight, cancellation/late/duplicate custody, IPv6 scope, unsigned bits, bind failures and poller first-use. Add focused tests for off-thread completion Actor ownership, Task/non-Task source-backed materialization, queued termination drop, and malformed host descriptor.

Falsifiers: any guest object/prototype in NIO endpoint signatures/fields after final I093-B; IPv6 bytes rewritten or mapped-family normalization; callbacks on poller; a resource leaked by queued termination; compulsory Task/poller/module import in a networkless case; or a public semantic/API change without separate owner approval.

## 6. Recommended grouped implementation handoff — not a new issue

- **I093-B1: full connect vertical slice** — one coherent green change across private host descriptor, `connectTcp` secure guest preflight/encoding, host-only NIO connect/decode/local-descriptor capture, owning-Actor completion and resource cleanup, focused connect/Task/no-Task tests. Accept/listen may still use legacy guest-coupled backend code *temporarily* until B2; never declare whole I093-B complete after B1.
- **I093-B2: listen and accept vertical slice** — migrate constrained/wildcard listen descriptors, accepted local/remote endpoints, host provisioning/custody parameter cleanup, and listener/Actor transfer tests. At B2 completion all NIO endpoint-side guest dependencies are prohibited.
- **I093-C:** exhaustive conformance/architecture acceptance, Native Image relevant validation, current-head source coupling census, and audit closure evidence. No invented performance improvement claims.

This is execution-only grouping inside existing I093; it does not create independent formal Issues. I090 and I091 remain unrelated and unblocked. AUD020 stays OPEN; its original exhaustive source-by-source matrix, comparative 12-criterion alternatives and measurements are not completed by this checkpoint.

## 7. Work-state and execution provenance

No product source, specification, Maven version, changelog, test, Git branch or issue status changed as part of this read-only source audit. The human's reported local test PASS/`git diff --check` clean is recorded without inferring I093-B code has already been tested. An implementation agent edits `guillermomolina/protos` from actual checkout HEAD; human executor alone runs builds/tests/validation, commit/push. B1's following version/CHANGELOG updates must occur only after tests pass, just before the human's publication, with no subsequent tests. Docs publication is separately authorized in `guillermomolina/protos-project-docs`.

```text
AUD020_STATUS=OPEN_PARTIAL_READ_ONLY_EVIDENCE
I093_A=PUBLISHED
I093_B=NOT_IMPLEMENTED_BY_THIS_AUDIT
NEXT=I093_B1_CONNECT_VERTICAL_SLICE
PROTOS_REVISION=97404f1cc1c1d678c66ef598b47cf62d32014f21
PRODUCT_TEST_EXECUTION_BY_THIS_AUDIT=NONE
MAINTAINER_REPORT_LOCAL_TESTS=PASS
MAINTAINER_REPORT_DIFF_CHECK=CLEAN
I090_I091_BLOCKED_BY_I093=NO
```
