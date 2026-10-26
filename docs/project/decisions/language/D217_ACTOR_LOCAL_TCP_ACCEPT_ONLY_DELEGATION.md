# D217 — Actor-delegable TCP accept-only capability (M3-local)

**Status:** **RATIFIED AND NORMATIVELY PUBLISHED** (2026-10-26); runtime implementation pending.
**Canonical decision:** [D217 #965](https://github.com/guillermomolina/protos/issues/965).
**Related dependent decision:** [D216 #964](https://github.com/guillermomolina/protos/issues/964) (continuous HTTP hosting).
**Original research baseline:** [`205420a03d4ebb1c5c125ca891c04147bd0c752c`](https://github.com/guillermomolina/protos/commit/205420a03d4ebb1c5c125ca891c04147bd0c752c), specification `0.1.472` (2026-10-25).
**Exact normative publication:** [`5f95dc086fc9bc853afb712915db47ab28547417`](https://github.com/guillermomolina/protos/commit/5f95dc086fc9bc853afb712915db47ab28547417), specification **`0.1.473`** (2026-10-26).
**Publication evidence:** [D217 specification 0.1.473](../../evidence/D217/D217_SPEC_0_1_473_PUBLICATION_2026_10_26.md).
**Comparative and approval evidence:** [D216/D217 architectural research checkpoint](../../evidence/D216/D216_TCP_ACTOR_ACCEPTOR_M3_LOCAL_RESEARCH_2026_10_25.md).

This remains a **non-normative project decision record**. The authoritative semantics are now published in `spec/io/NETWORK.md`, `spec/concurrency/ACTORS.md`, and `spec/io/PROCESS_IO.md` at the exact D217 revision above. Publication does **not** establish a running `TcpAcceptor` implementation or HTTP hosting operation. `Network`, `TcpListener`, and `TcpConnection` remain non-transferable across Actor/P boundaries.

## Motivation and selected direction

The HTTP/1.1 `std:http/Server.serveOne.owning` library is complete for one request, but the language needs a reusable, scalable admission substrate. A singleton guest Actor that alone accepts every socket and sends work onward would impose a serial admission hop and cannot send a non-transferable `TcpConnection` to another Actor. The project Test Tool's successful **shared-queue, worker-pull** scheduling is a useful architectural analogy, not a permission to conflate Tasks, Actors and Processes.

Guillermo explicitly corrected the C/P/A boundaries and approved **M3-local** with engineering **pay-as-you-grow**:

- **C:** one-CPU/cooperative execution. An ordinary listener stays usable, with no mandatory delegation/scheduler machinery.
- **A:** multiple Actors on CPUs in **one Protos Process**. Workers may be independently eligible to accept and process live connections without one serial guest-Actor dispatcher.
- **P:** multiprocess and multinode execution are consciously **left for future design**. We know that growth may eventually be required, but we do not design or implement speculative remote-socket proxies, cross-Process handoff, consensus, remote RPC or multi-Process listener infrastructure today.

## Approved M3-local capability boundary

**Published normative public surface** (`NETWORK.md` §§7.1, 8, 11):

```text
listener.acceptCapability() -> TcpAcceptor
acceptor.accept() -> Future<TcpConnection>
```

1. The original `TcpListener` remains the sole owning listener capability. A `TcpAcceptor` is **a different explicitly delegated accept-only authority**, not a transferable or duplicated `TcpListener`, and never a transferable `TcpConnection`.
2. Delegation to an Actor creates/installs **that Actor's own local capability representation** denoting the same logical listener, as permitted by the existing explicit-capability model. Ordinary `ActorRef.send/request` does **not** silently proxy an encountered non-transferable resource.
3. Each eligible worker Actor performs its own pending `accept()`; each successfully accepted `TcpConnection` first belongs to the Actor whose accept completed. The connection is never routed as a live guest value through a central Actor.
4. `TcpAcceptor` cannot create listeners, initiate outbound connections, expose the Network authority, close the listener, or grant more than accept authority. No ambient network permission or inheritance from Actor creation.
5. All delegates share **one logical accept domain** and its receiver close/commitment state. Concurrent accepts may proceed without a prescribed cross-Actor winner/fairness order. Existing `IO_CORE.md` cutovers classify in-flight work; late/cancelled accepted sockets must not leak or be handed to two Actors; accepted sockets already owned by workers are not closed by listener close.
6. Placement compatibility is runtime-private: an Actor requiring this Process-local special resource is admitted only in a compatible placement, as already described in `DISTRIBUTED_RUNTIME.md` §45. The program is not asked to specify CPU affinity, Process identifiers, node addresses or a new routing layer.
7. A runtime may implement the logical accept domain directly in host primitives; **there is no mandatory ordinary guest Actor or extra guest-message hop**. No specific epoll/NIO/io_uring implementation, fairness promise, worker count, thread count or hidden globally active infrastructure is ratified.

## Invariant and delta audit (GITHUB021)

| Existing owner-approved invariant | Disposition |
| --- | --- |
| D047/D052/D173 `Network` authorizes TCP operations explicitly | **Preserved**; acquiring the initial listener still requires existing explicit authority. |
| `NETWORK.md` §11: `Network`, `TcpListener`, `TcpConnection` non-transferable across Actor/P | **Preserved**; no generic live-resource movement or remote socket support. |
| `PROCESS_IO.md` §26: never auto-proxy on failed `send/request`; distinct explicit Actor-local capability may denote one service | **Reused** for a newly *minted accept-only* capability. |
| `ACTORS.md` `NonTransferableValue` and explicit capability delegation | **Preserved**; validation and delegation are specific to `TcpAcceptor`. |
| `IO_CORE.md` §8 close receiver cutovers and Future cancellation | **Preserved**, with a narrow TCP accepted-resource custody specialization only where required. |
| `DISTRIBUTED_RUNTIME.md` §45 hard feasibility for missing special resources | **Preserved**; compatibility is hidden, no public placement knobs. |
| D215 HTTP one request, owned close, `Connection: close`, D215/LIB025 closed | **Untouched**; reusable hosting is a separate future facility (D216). |
| Owner's pay-as-you-grow principle | **Strengthened**; no engineering investment or runtime cost for hypothetical multi-Process/P growth. |

The **only new authorized design surface** is explicit same-Process Actor delegation of a distinct accept-only listener capability. It is not a decision for transferable sockets or cross-Process resource proxies.

## Ratified option A: delegation content and lifecycle

At `0.1.473` the owner-selected delegation rule is **option A — authority only**, not an ordinary object copy. `Actor.spawn` initialization provisioning creates one destination-Actor-local `TcpAcceptor` with the standard frozen protocol parent, fresh structural `OPEN` state and authority over the *same* listener. It neither copies source-added slots nor transfers the source wrapper's structural state; values reachable solely through omitted slots are not part of the delegated graph. The surrounding ordinary argument graph retains atomic transfer validation. Inherited `acceptor.close()` remains ordinary structural `Object.close()`, not listener close.

`listener.acceptCapability()` after the listener close cutover signals a fresh synchronous `IOLifecycleError`; `acceptor.accept()` after that cutover returns a fresh ordinary failed Future containing a fresh `IOLifecycleError`. Other pending, committed, cancelled or undeliverable acceptance outcomes reuse `IO_CORE.md` §§4 and 8. Distinct logical listeners in one Process never merge acceptance domains. No `send()`/`request()` extension, cross-Process delegation or HTTP behavior change is included.

## Boundary to D216 and implementation

D217 is **normatively complete** as reusable Core TCP admission substrate, but **runtime implementation remains separate**. D216 separately owns the already owner-selected **Option B**, reusable optional continuous HTTP/1.1 listener hosting in `std:http` using the D217 primitive and existing `serveOne.owning`. There is still **no owner-approved exact `Server.serve` signature**, concurrency/admission bound, shutdown/drain or terminal-error precedence, and no HTTP hosting implementation. D215/LIB025 remain closed.

An `Ixxx` runtime task may now implement the **published D217 contract** without redesigning the language; this does not authorize guessing D216 HTTP semantics. No speculative multi-Process/P architecture or eager delegation machinery is implied.

## Source and comparative references

- [Erlang OTP 27 `gen_tcp`](https://www.erlang.org/docs/27/apps/kernel/gen_tcp.html): several processes can concurrently call accept on a listening socket. Its separate `controlling_process` move is **deliberately not part of Protos**.
- [Ranch 2.1 listeners](https://ninenines.eu/docs/en/ranch/2.1/guide/listeners/): acceptor pool and separate connection-process orchestration.
- [Netty 4.x user guide](https://netty.io/wiki/user-guide-for-4.x.html): separate host-level boss and worker event-loop groups; an alternative backend, not a Protos guest-Actor contract.
- [D216/D217 research checkpoint](../../evidence/D216/D216_TCP_ACTOR_ACCEPTOR_M3_LOCAL_RESEARCH_2026_10_25.md): Test Tool shared-queue observation and bounded alternatives.

**Publication outcome:** D217 `0.1.473` is **PUBLISHED** in the product commit identified above. The maintainer reported `git diff --check` clean and all local tests PASS; the coordinating agent independently verified the exact published commit and normative delta but did not execute the tests. Its separate runtime implementation and D216's exact HTTP contract remain open.