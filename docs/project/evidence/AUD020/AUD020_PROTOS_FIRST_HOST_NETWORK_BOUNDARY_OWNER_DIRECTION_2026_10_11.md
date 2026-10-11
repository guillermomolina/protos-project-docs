# AUD020 — Protos-first networking boundary: source-backed finding and owner-approved direction (2026-10-11)

**Record role:** immutable investigation/approval-context evidence; **not** a normative specification, full GITHUB010 ratification packet or executable validation.
**Audit issue:** [AUD020/#872](https://github.com/guillermomolina/protos/issues/872).
**Implementation owner:** [I093/#873](https://github.com/guillermomolina/protos/issues/873).
**Product source revision inspected:** `PROTOS_REVISION=1c414e08fefc2378b828bba7be7d263ea32e10ce`.
**Project documentation baseline inspected:** `e205109f105451e2d54b8be69496a77666353b90`.
**Execution:** read-only review of GitHub sources; **NO BUILD / TEST / BENCHMARK / product source mutation** in this audit/publication.

## A. Owner feedback, correction of research objective, and approval scope

The owner corrected the earlier I090-motivated framing: AUD020 is primarily about **minimizing the networking implementation in Java** and the reciprocal friction between Java and Protos. Java/NIO should use its own host data types; Protos should use its own semantic data types. Neither side is entitled to force the other to use its type hierarchy, particularly `java.math.BigInteger` or the Protos `BigInteger` prototype. This is a **networking architecture objective independent of any measured savings in I090**.

Following a source-grounded discussion of Protos-source ownership, a byte/port host descriptor boundary, removal of `integerPrototype` and guest-object creation from NIO, and reuse of Actor/Future lifecycle machinery, the owner stated **“apruebo lo que dices”** and explicitly requested issue and evidence publication and the next implementation slice. This is recorded as approval of **the bounded Protos-first implementation direction as presented**, not of an unpresented concrete Java ABI, weakened security invariant, or a novel public-language behavior.

**Approved direction:**

1. Standard IP/address and endpoint semantics (construction, parsing, formatting, numeric conversions, equality, hash and as much recognition as safely possible) belong in **Protos source**. Mechanically reuse shared canonical family prototypes and ordinary object protocols.
2. Java NIO stays responsible for `InetAddress`, `InetSocketAddress`, `ServerSocketChannel`, `SocketChannel`, poller, IPv6 host scope resolution, authority, resource custody, cancellation, kernel/host effects and small unavoidably privileged runtime integration.
3. The crossing representation is a **minimal immutable host address/endpoint descriptor**: exact four/sixteen address octets, IP version and bounded port, with defensive copies. This is not a mandatory Java `BigInteger`, Protos `Integer`/ `BigInteger`, `IpAddress`/ `IpEndpoint`, nor guest prototype.
4. The backend must not inspect or construct Protos objects; guest values are materialized/consumed on the Protos side of the boundary. No guest callback or source Closure is executed on the NIO poller. The owning Actor/Context must own completion; cancellation/resource cleanup is unchanged.
5. **Keep** observable D047/D048/D172 guarantees (canonical Process-shared frozen families, recognition without arbitrary candidate callbacks, identity, exact shape, safe cross-Actor/P copies, equality/hash) and D173 explicit Network-capability confinement, unless a separate exact owner-approved decision changes one.

This approval must not be misreported as approval of an actual full D172 normative replacement. **D172: AMEND direction / exact ratification delta TBD only if existing text mandates Java-host behavior contrary to approved design**. **D173: KEEP_AS_APPROVED**. If a viable Protos-source implementation preserves the existing normative invariants, no semantic D172 amendment is necessary merely to change implementation languages.

## B. Actual code evidence at pinned HEAD

| File (full path rooted at product repository) | Observable/current responsibility | Problem and intended disposition |
|---|---|---|
| `protos/lib/network/IpAddresses.protos` | `v4`, `v6`, numeric IPv4/IPv6 parsing and formatting, hextet arithmetic including 128-bit construction | **KEEP** source semantics. Add source-level inverse exact octet conversion, or equivalent source-backed implementation, without host-type dependency. |
| `protos/lib/network/IpEndpoints.protos` | Endpoint parsing, port checks and textual formatting | **KEEP**; host descriptor encoding belongs here or another source-backed library entry; no new public API without separate approval. |
| `protos/lib/core/IpAddress.protos`, `IpEndpoint.protos` | Canonical source prototype seeds (`IpAddress: {}`, `IpEndpoint: {}`) | **EXPAND** source-owned construction/equality/hash protocols where safe; do not introduce Actor-local replacements. |
| `src/main/java/com/guillermomolina/protos/execution/ProtosCoreBootstrap.java` | Loads source prototypes, installs `ProtosStandardIp*Protocol`, removes public Prelude IP names, publishes shared standard module members | **KEEP** existing shared-family and module publication mechanism. Reduce redundant Java protocol installation, where demonstrated possible. |
| `src/main/java/com/guillermomolina/protos/execution/ProtosStandardIpAddressProtocol.java` | Java native `init`, `recognizes`, `==`, `hash`; checks frozen/parent/exact slots; uses `ProtosNumericValueSupport` | **MOVE_TO_PROTOS** source-expressible operations; retain a minimal trusted no-callback structural recognition primitive if ordinary dynamic lookup cannot prove equivalence. |
| `src/main/java/com/guillermomolina/protos/execution/ProtosStandardIpEndpointProtocol.java` | Java native init/recognition/equality/hash, port recognition and structural checks | **MOVE_TO_PROTOS** source-expressible operations, preserve minimal secure recognition. |
| `src/main/java/com/guillermomolina/protos/execution/ProtosNioNetworkHost.java` | Constructs/provisions NIO backend and resolves scoped IPv6; `backend(addressPrototype, endpointPrototype, integerPrototype)` currently threads guest objects | **KEEP** host scope/poller; **REMOVE_PERMANENTLY** guest-prototype dependency from its backend signature. |
| `src/main/java/com/guillermomolina/protos/execution/ProtosEmbeddedNetworkCustody.java` | Lazily opens NIO host; stores IP and integer prototypes to construct backend | **KEEP** lazy network custody; remove stored/proxied guest prototypes used only for NIO. |
| `src/main/java/com/guillermomolina/protos/execution/ProtosNioNetworkBackend.java` | `HostEndpoint.decode` checks guest parent/slots; `HostEndpoint.materializeLocalEndpoint` creates/freeze guest IP objects and calls `integerFromUnsignedBigEndian(bytes, integerPrototype)`; NIO connect/listen | **KEEP** actual NIO operations, constrained host scope and lifecycle; **REMOVE_PERMANENTLY** inspection of guest slots, prototype fields, and guest integer/address/endpoint materialization. |
| `src/main/java/com/guillermomolina/protos/execution/ProtosNioTcpListenerBackend.java` | `handoffAccepted` obtains socket endpoints; `materializeEndpoint` creates/freeze guest IP objects and converts bytes to integer using `integerPrototype` | **KEEP** poller/accept/cancel/custody; **REMOVE_PERMANENTLY** guest endpoint materialization from NIO. |
| `src/main/java/com/guillermomolina/protos/runtime/ProtosNetworkConnectFlow.java` | Backend receives guest `ProtosObjectValue`; completion carries guest `localEndpoint` | **AMEND internal contract** to carry host descriptors; preserve commit/cancellation/resource cleanup. |
| `src/main/java/com/guillermomolina/protos/runtime/ProtosNetworkListenFlow.java` | `ListenRequest(int ipVersion, ProtosObjectValue addressConstraint, Integer portConstraint)` | **AMEND** host contract to receive host address representation, not guest object. |
| `src/main/java/com/guillermomolina/protos/runtime/ProtosTcpListenerFlow.java` | Accept completion carries two guest `ProtosObjectValue` endpoints | **AMEND** completion to host endpoint descriptors; final guest materialization inside owning Actor/Context. |
| `src/main/java/com/guillermomolina/protos/execution/ProtosStandardNetworkProtocol.java` | Holds explicit capability check; recognizes requested endpoint; materializes runtime TCP values | **KEEP** trusted authority gate and semantic/host transition here or within its existing execution-domain boundary. |
| `src/main/java/com/guillermomolina/protos/runtime/ProtosNumericValueSupport.java` | Guest numeric classification, exact `unsignedBigEndianOrNull`, `integerFromUnsignedBigEndian`; uses exceptional `java.math.BigInteger` when magnitude requires | **KEEP** for independent numeric clients, but remove NIO dependency on numeric prototype/minting. |
| `src/main/java/com/guillermomolina/protos/runtime/ProtosNetworkCapabilityValue.java` | Opaque host authority target outside ordinary object slots | **KEEP** current security model; no ambient authority. |
| `src/main/java/com/guillermomolina/protos/runtime/ProtosActorValueTransfer.java` | Actor copy, canonical shared standard object preservation and semantic rematerialization | **KEEP** general transfer mechanism; network data must remain portable and resources not transferable. |
| `src/main/java/com/guillermomolina/protos/runtime/ProtosActorExecutionDomain.java` | Contains `enqueueTargetedFutureCompletionForRuntime` and Actor-owned execution machinery | **REUSE/VERIFY**; do not execute Protos closures directly on NIO poller. |
| `src/main/java/com/guillermomolina/protos/execution/ProtosNioTcpConnectionBackend.java`, `ProtosNioHostIoPoller.java` | Physical byte I/O, channel readiness, poller lifecycle, cancellation | **KEEP** unless a demonstrated coupling appears; no speculative rewrite. |

Source browser root: `https://github.com/guillermomolina/protos/blob/1c414e08fefc2378b828bba7be7d263ea32e10ce/`. Each path above is relative to that exact source tree.

### Direct numeric dependency is *not* the same as guest-prototype coupling

The already-published [I091-F5 BigInteger token audit](https://github.com/guillermomolina/protos-project-docs/blob/e205109f105451e2d54b8be69496a77666353b90/docs/project/evidence/I091/I091_F5_ADVERSARIAL_BIG_INTEGER_CASE_BY_CASE_AUDIT_2026_10_11.md) reported **73 tokens in 16 Java production files** at its historical source baseline and no NIO backend among those files. This count is **not independently rerun at AUD020 baseline**. It counts text tokens, *not* allocation sites or runtime cost.

The current NIO problem is **indirect coupling**: guest numeric prototype propagation and Java construction of large guest integer values from IPv6 octets, not a direct NIO import of `java.math.BigInteger`. A raw `byte[16]` is enough for host IPv6 address transport; no transport-layer BigInteger is required. If a guest wants its 128-bit mathematical `bits`, **that is Protos semantics** and its exact large-integer normalization belongs on the Protos side of the boundary. Java might still implement a bounded runtime transfer mechanism but cannot dictate guest family.

## C. Existing normative/decision invariants and amendment classification

- [D048](https://github.com/guillermomolina/protos/issues/206) and `spec/io/NETWORK.md` require ordinary frozen address/endpoint data, shared canonical immediate parent, exact slot shape, callback-free recognition, structural equality/hash, transfer across Actors/P without Network authority; this is an observable contract, **KEEP**.
- [D172](https://github.com/guillermomolina/protos/issues/644) puts public IP families in `std:network` while retaining canonical private runtime substrate. **Recommended AMEND narrowly, if needed:** retire the *assumption* that a private substrate must implement init/equality/hash in Java. Preserve minimal trusted introspection, Process-shared prototypes, transfer and publication identity. Do not amend public family placement.
- [D173](https://github.com/guillermomolina/protos/issues/645) explicit Network opt-in, no ambient Process/Actor/P network grant, no thin speculative TCP facade. **KEEP_AS_APPROVED**, including host resource custody. No D173 amendment ratified or needed.
- [D197](https://github.com/guillermomolina/protos/issues/865) and [PLAT056](https://github.com/guillermomolina/protos/issues/866) control numeric guest semantics/carriers independently. Neither is a reason to force Java NIO to consume guest integer prototypes.
- [I090](https://github.com/guillermomolina/protos/issues/868) and [I091](https://github.com/guillermomolina/protos/issues/869) are independent; **no `blocked by I093` edge** inferred.

**GITHUB021 boundary:** The owner's approval covers the presented Protos-first architecture **with existing observable guarantees preserved**. Any discovered incompatibility with a literal ratified clause, new public protocol, changed identity/hash/recognition semantics, changed scope/cancellation/authority, or new runtime institution requires a full explicit delta, owner approval and durable ratification before implementation of that change. This record is **not** that future ratification and does not rewrite D172's historical text.

## D. Why a descriptor-only boundary rather than the alternatives

- **A: status quo** — least migration but NIO depends on guest integer/shape evolution and retains dual responsibility. Rejected for long-term ownership.
- **B: Java-only universal materializer** — removes prototype parameters from NIO but retains guest library semantics in Java; not the preferred minimal Protos-first model.
- **C: source-owned IP protocols and minimal trusted runtime checks + host data descriptors** — selected architectural direction; strongest source ownership and long-term guest/host separation while keeping authority/custody in Java.
- **D: route NIO through general public `std:interop`/foreign SPI** — not selected: would need new authority rules, expose/allocate unnecessary provider machinery, and risk implicit host privileges. Reuse existing internal mechanisms only where they satisfy confinement.
- **E: eliminate native trusted recognition altogether** — unproven. Dynamic method override and forged shapes may violate D048's callback-free canonical recognizer; investigate narrowly before deleting the privileged check.

These are **qualitative source-based assessments**, not a fabricated numerical 12-axis score or measured benchmark. AUD020's originally specified exhaustive comparative matrix, full audit of remaining classes and all external-language implementation references were **not completed in this interaction**; they remain evidence limitations rather than silently claimed PASS.

## E. Data flows, safety checks and implementation sequencing

**Outbound (guest -> host):** canonical endpoint validated on Protos side without arbitrary callbacks; source-backed exact address octet conversion (width 4 or 16), bounded port, and defensive-copy descriptor; Java NIO constructs `InetAddress/InetSocketAddress` and applies its existing scoped IPv6 resolution. IPv6 all-ones, `2^63`, `2^64`, `2^127` and `2^128-1` must remain exact with no signed wrapping or stripping leading zeros.

**Inbound (host -> guest):** Java copies `InetSocketAddress.getAddress().getAddress()` octets plus bounded port into immutable descriptor; host poller never invokes Protos guest logic. Existing Future/Io transition commits and Actor-owned completion materialize the canonical Protos address and endpoint, using the correct destination domain's integer normalization and shared family identity. Failure/cancellation must clean up physical channels and not publish a late resource.

**Recommended grouped slices under [I093/#873](https://github.com/guillermomolina/protos/issues/873):**

1. **A**: source-backed IP protocol + bidirectional exact octet conversion; preserve existing canonical shared publication and trusted recognition; focused tests and an independently green integration seam.
2. **B**: descriptor-only host backend + Flow contracts + deferred safe guest materialization; retain NIO scope, poller, custody and Future/cancel gates as one coherent integration slice.
3. **C**: full conformance + structural source checks + user-run full suite + Native Image-relevant validation + compare supported performance evidence. No gratuitous micro-slices or invented speed claims.

These are planning slices inside I093, not new formal Issues unless the repository's ISSUE-SLICE-BOUNDARY promotion triggers emerge. For parallel repository work, coordinate with I090/I091 on common numeric/bootstrap files but do not claim dependency.

## F. Pending proof / STOP list

**No executable claims:** No code has been changed in `guillermomolina/protos`; no test, build, benchmark, Native Image verification or `git diff --check` has been run by the investigating agent. Required next evidence includes:
- callback-free canonical recognition with overridden `parent`, `slotNames`, `==`, `hash` and forged receiver/slots;
- constructor arity/error equality/hash/map-key semantics, direct-parent identity and transfer;
- source-defined protocol behavior and receiver semantics under bootstrap, without actor-local module identities;
- precise Actor ownership for asynchronous network materialization and cancellation/late completion;
- byte[4]/byte[16] round trip, IPv4-mapped IPv6 and link-local scope constraints;
- no networkless poller/Task/Context penalty, no hot-path regression, no false arithmetic attribution;
- compatibility with actual current `main` and independently edited I090/I091 work.

**AUD020 closure is not asserted here:** This evidence captures the confirmed source findings and owner-approved implementation direction. The original larger audit work item should not be marked exhaustively complete merely by publishing this focused record.

```text
AUD020_ARCHITECTURAL_DIRECTION=OWNER_APPROVED_2026_10_11
AUD020_EXHAUSTIVE_AUDIT_COMPLETE=NO
D172_RECOMMENDATION=AMEND_ONLY_IF_CONTRACT_REQUIRES
D173_RECOMMENDATION=KEEP_AS_APPROVED
I093_CREATED=https://github.com/guillermomolina/protos/issues/873
I090_DEPENDENCY_ON_I093=NONE
I091_DEPENDENCY_ON_I093=NONE
SOURCE_BASELINE=1c414e08fefc2378b828bba7be7d263ea32e10ce
AGENT_EXECUTED_TESTS=NONE
PRODUCT_SOURCE_CHANGED=NO
```
