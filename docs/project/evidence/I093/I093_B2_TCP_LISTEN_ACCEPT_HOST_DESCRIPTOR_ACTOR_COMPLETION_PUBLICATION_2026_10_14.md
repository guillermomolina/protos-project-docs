# I093-B2 — Published TCP listen/accept host boundary and owning-Actor completion

- **Date:** 2026-10-14.
- **Implementation owner:** [I093 / guillermomolina/protos#873](https://github.com/guillermomolina/protos/issues/873).
- **Related independent audit:** [AUD020 / #872](https://github.com/guillermomolina/protos/issues/872).
- **PROTOS_REVISION:** `f8e2d4e3dcf9b5e13d4850e14605248cae326125` ([published product commit](https://github.com/guillermomolina/protos/commit/f8e2d4e3dcf9b5e13d4850e14605248cae326125)).
- **Implementation version:** `0.3.355-SNAPSHOT`.
- **Predecessor:** [I093-B1 connect-only publication](./I093_B1_TCP_CONNECT_HOST_DESCRIPTOR_ACTOR_COMPLETION_PUBLICATION_2026_10_14.md) at `cf39ddbc0921a2af0f99bcb68a396e40a03a99a5`.
- **Architectural input:** [source-pinned AUD020 I093-B handoff](../AUD020/AUD020_I093_B_DESCRIPTOR_AND_ACTOR_HANDOFF_SOURCE_REVIEW_2026_10_14.md).
- **Status:** I093-B2 **PUBLISHED** on `main`; I093-A, B1 and B2 are published; final I093-C **PENDING**. I093 and AUD020 remain **OPEN**.

## Published implementation

The B2 release completes the TCP **listen/accept** part of the intended guest/host separation begun in B1. No change to the public `Network.listenTcp(localRequest)` or `TcpListener.accept()` protocol is asserted.

1. **Exact listen request before backend entry.** `ProtosStandardNetworkProtocol.listenTcp` continues to validate the ordinary three-slot request and recognize the canonical guest address without arbitrary candidate callbacks, then captures its 4/16 network-order octets. `ProtosNetworkListenFlow.ListenRequest` contains only `ipVersion`, optional defensively copied octets and optional `Integer` port. `null` preserves wildcard address and host-selected nonzero local-port request; the host receives neither a guest address nor a numeric-family object.
2. **Pure host NIO listener admission.** `ProtosNioNetworkBackend` decodes listen octets, selects host protocol family and authorized IPv6 scope/listen addresses, and creates `ProtosNioTcpListenerBackend` without guest address/endpoint/integer prototypes. IPv6-only checks, address-width/family checks, explicit host Network authority and component-port consistency remain in the host path.
3. **Accept returns host descriptors.** `ProtosNioTcpListenerBackend` captures both local and remote `InetSocketAddress` endpoints as validated `ProtosHostIpEndpoint` values, retaining physical connection-channel custody until a successful transfer; it does not create, freeze, inspect or retain Protos IP objects on the poller.
4. **Actor-local guest projection.** `ProtosTcpListenerFlow.AcceptCompletion` and its result materializer carry host descriptors. `ProtosStandardTcpListenerProtocol` projects both accepted endpoints through the `ProtosStandardNetworkProtocol.materializeHostEndpoint` Actor-side helper, which constructs frozen canonical guest `IpAddress`/`IpEndpoint` values with the destination Prelude's numeric family. The connection capability retains its Protos-visible local/remote endpoint behavior.
5. **Delivery and custody.** The new private `ProtosNetworkAcquisitionInvocation` coordinates targeted owning-Actor completion for listen and accept after asynchronous NIO callbacks. Synchronous backend callbacks still have the existing direct completion path. Pre-commit cancellation, host callback races, duplicate/late outcomes, Actor termination and release of untransferred resources use the existing `ProtosIoOperation` lifecycle and claim-once pending handoffs.
6. **Provisioning cleanup.** `ProtosNioNetworkHost.backend()` no longer takes guest prototypes. `ProtosEmbeddedNetworkCustody` keeps lazy first-use NIO activation and no longer stores a Prelude-derived address/endpoint/`bigIntegerPrototype` triple as backend fields.

**Static source observation at the published product revision:** the complete `ProtosNioNetworkBackend.java` and `ProtosNioTcpListenerBackend.java` files contain no textual references to `ProtosObjectValue`, `ProtosNumericValueSupport`, `ProtosPrelude`, `integerPrototype`, `addressPrototype`, `endpointPrototype` or `BigInteger`. This is a **source-token observation for these two classes**, not a transitive architecture/ABI census or a runtime/performance measurement.

## Published change surface

The GitHub product commit changes exactly **16 files**:

- **Production / execution (6):** `ProtosEmbeddedNetworkCustody.java`, `ProtosNioNetworkBackend.java`, `ProtosNioNetworkHost.java`, `ProtosNioTcpListenerBackend.java`, `ProtosStandardNetworkProtocol.java`, `ProtosStandardTcpListenerProtocol.java` in `src/main/java/com/guillermomolina/protos/execution/`.
- **Production / runtime (3):** `ProtosNetworkAcquisitionInvocation.java` (new), `ProtosNetworkListenFlow.java`, `ProtosTcpListenerFlow.java` in `src/main/java/com/guillermomolina/protos/runtime/`.
- **JUnit tests (5):** `ProtosNetworkListenAcquisitionTest.java`, `ProtosNioNetworkProvisioningTest.java`, `ProtosNioTcpListenerBackendTest.java`, `ProtosTcpListenerAcceptTest.java`, `ProtosTcpListenerIntegratedConformanceTest.java` in `src/test/java/com/guillermomolina/protos/execution/`.
- **Publication metadata (2):** `pom.xml`, `CHANGELOG.md` (new `0.3.355-SNAPSHOT` entry).

The maintainer's Git commit report counted **510 insertions and 283 deletions**. No normative `spec/` change, external Network authority expansion, public host-descriptor API or new mandatory Actor Task is claimed.

## Validation provenance — distinguish actual reports from claims

Validation was run by the **human executor** against their local checkout, not replayed by the documenting agent.

- An early B2 focused compilation attempt revealed a single outdated `ProtosTcpListenerFlow.AcceptCompletion.succeeded` test callback in `ProtosNioNetworkProvisioningTest`. The test callback signature was corrected without changing production semantics.
- The subsequent focused Java battery compiled and reported **44 tests, 4 failures, 4 errors, 0 skipped**. Failures were concentrated in three recording-backend harnesses that expected synchronous Future settlement after a post-return fake callback, but the new contract correctly queues the guest completion to the Actor.
- The three harnesses were adapted to drive the owning Actor's `dispatchOne()` for targeted completions; their rerun reported **22 tests, 0 failures, 0 errors, 0 skipped**. The other 22 from the earlier focused battery had already passed and only the three failing test classes changed at that stage.
- The human executor then confirmed **`make test -> PASS`** and **`make check -> PASS`** for the final substantive candidate. The finalization script reported **`I093_B2_DIFF_CHECK=PASS`**, `I093_B2_OWNED_PATHS=14` (before metadata), `I093_B2_VERSION=0.3.355-SNAPSHOT`; the preceding attempt failed safely because whitespace stripping corrupted one `git status --porcelain` path and wrote no metadata.
- Maven version and CHANGELOG were amended only after green validation and included atomically in the published product commit. No post-versioning behavior tests or independently retained full output logs are claimed.
- The product commit SHA, version and 16 changed paths have been checked against GitHub; the source-token census above was obtained directly from that exact revision.

**Not established by this publication:** separate Native Image compilation/smoke, a complete cross-Actor/P/Context networking matrix, an exhaustive semantic/security review of all remaining Java adapters, a before/after allocation or throughput comparison, or closure of AUD020.

## Next acceptance: I093-C

I093-C must analyze and falsify the completed boundary against current product HEAD rather than treating B2's green suite as architectural proof. Deliver a source-pinned inventory of *every* ingress/egress for `connectTcp`/`listenTcp`/`accept`, NIO/listener/backends, the precise remaining Java guest adapter duties, callback/Actor ownership and custody after cancellation/termination/close; exercise IPv4, IPv6 (including high unsigned 128-bit and mapped cases), IPv6 link-local scope/authority, no-DNS/no-ambient-Network constraints, malicious candidate recognition, future commitment and cross-domain value/capability isolation. Check Native Image-relevant validation, pay-as-you-grow and observed costs through existing suitable gates/harnesses. Report measured numbers or **NOT_MEASURED**, never assumed savings.

If any source/spec inconsistency would change public semantics, obtain an explicit owner decision rather than silently editing D172/D173. I093 may close only on published final evidence after these requirements are satisfied. AUD020 remains independent and open until its broader audit matrix is complete.

```text
I093_A=PUBLISHED
I093_B1=PUBLISHED
I093_B2=PUBLISHED
I093_C=PENDING
PROTOS_REVISION=f8e2d4e3dcf9b5e13d4850e14605248cae326125
IMPLEMENTATION_VERSION=0.3.355-SNAPSHOT
PRODUCT_MAKE_TEST=PASS_HUMAN_REPORTED
PRODUCT_MAKE_CHECK=PASS_HUMAN_REPORTED
PRODUCT_DIFF_CHECK=PASS_HUMAN_REPORTED
NIO_GUEST_TOKEN_CENSUS=ZERO_IN_TWO_CLASSES_AT_PRODUCT_REVISION
NATIVE_IMAGE_GATE=NOT_CLAIMED
CROSS_DOMAIN_FINAL_ACCEPTANCE=NOT_CLAIMED
COST_MEASUREMENT=NOT_CLAIMED
I093_ISSUE=OPEN
AUD020_ISSUE=OPEN
```
