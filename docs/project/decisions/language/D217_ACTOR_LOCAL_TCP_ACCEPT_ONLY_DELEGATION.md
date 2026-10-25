# D217 — Actor-delegable TCP accept-only capability (M3-local)

**Status:** OWNER-APPROVED DESIGN DIRECTION, **NORMATIVE PUBLICATION PENDING** (2026-10-25).
**Canonical decision:** [D217 #965](https://github.com/guillermomolina/protos/issues/965).
**Related dependent decision:** [D216 #964](https://github.com/guillermomolina/protos/issues/964) (continuous HTTP hosting).
**Exact reviewed Protos revision:** [`205420a03d4ebb1c5c125ca891c04147bd0c752c`](https://github.com/guillermomolina/protos/commit/205420a03d4ebb1c5c125ca891c04147bd0c752c), current project main as retrieved on 2026-10-25, global specification `0.1.472`.
**Comparative and approval evidence:** [D216/D217 architectural research checkpoint](../../evidence/D216/D216_TCP_ACTOR_ACCEPTOR_M3_LOCAL_RESEARCH_2026_10_25.md).

This is a **non-normative project decision record**. It does not amend any `spec/**` file, introduce an implemented `TcpAcceptor`, announce a usable HTTP hosting operation, or close the outstanding formal decisions. The live normative text at the reviewed revision continues to forbid Actor/P transfer of `Network`, `TcpListener` and `TcpConnection`.

## Motivation and selected direction

The HTTP/1.1 `std:http/Server.serveOne.owning` library is complete for one request, but the language needs a reusable, scalable admission substrate. A singleton guest Actor that alone accepts every socket and sends work onward would impose a serial admission hop and cannot send a non-transferable `TcpConnection` to another Actor. The project Test Tool's successful **shared-queue, worker-pull** scheduling is a useful architectural analogy, not a permission to conflate Tasks, Actors and Processes.

Guillermo explicitly corrected the C/P/A boundaries and approved **M3-local** with engineering **pay-as-you-grow**:

- **C:** one-CPU/cooperative execution. An ordinary listener stays usable, with no mandatory delegation/scheduler machinery.
- **A:** multiple Actors on CPUs in **one Protos Process**. Workers may be independently eligible to accept and process live connections without one serial guest-Actor dispatcher.
- **P:** multiprocess and multinode execution are consciously **left for future design**. We know that growth may eventually be required, but we do not design or implement speculative remote-socket proxies, cross-Process handoff, consensus, remote RPC or multi-Process listener infrastructure today.

## Approved M3-local capability boundary

Illustrative public form for the **next normative editing slice**; selector spelling and all detailed edge contracts require coherent spec ratification:

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

## Boundary to D216 and implementation

D217 ratifies the reusable TCP admission *substrate*. D216 separately owns **Option B**, reusable optional continuous HTTP/1.1 listener hosting in `std:http`, utilizing `serveOne.owning`, safe bounded admission, lifecycle/shutdown and the reusable TCP capability after D217 normative publication. There is **no approved public `Server.serve` signature** or HTTP listener implementation yet. D216 and D217 remain open.

The next slice is **normative editing, not runtime implementation**. A separate future `Ixxx`/library implementation will be allocated only after the normative public contract is published and independently validated by the human executor. Do not refactor the network runtime, reserve distributed infrastructure, change Maven version or amend the product changelog as part of research.

## Source and comparative references

- [Erlang OTP 27 `gen_tcp`](https://www.erlang.org/docs/27/apps/kernel/gen_tcp.html): several processes can concurrently call accept on a listening socket. Its separate `controlling_process` move is **deliberately not part of Protos**.
- [Ranch 2.1 listeners](https://ninenines.eu/docs/en/ranch/2.1/guide/listeners/): acceptor pool and separate connection-process orchestration.
- [Netty 4.x user guide](https://netty.io/wiki/user-guide-for-4.x.html): separate host-level boss and worker event-loop groups; an alternative backend, not a Protos guest-Actor contract.
- [D216/D217 research checkpoint](../../evidence/D216/D216_TCP_ACTOR_ACCEPTOR_M3_LOCAL_RESEARCH_2026_10_25.md): Test Tool shared-queue observation and bounded alternatives.

**Publication gate:** this document records an explicit owner-selected architecture only. Until actual normative text and revision evidence are published, the exact public selectors and corner-case semantics remain **UNPUBLISHED**.