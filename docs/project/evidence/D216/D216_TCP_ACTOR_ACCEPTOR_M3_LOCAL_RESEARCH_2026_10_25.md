# D216 / D217 — Continuous HTTP architecture and local TCP accept-only delegation: research and owner-approval evidence

**Date:** 2026-10-25. **Role:** historical evidence, not normative specification.
**Protos review baseline:** [`205420a03d4ebb1c5c125ca891c04147bd0c752c`](https://github.com/guillermomolina/protos/commit/205420a03d4ebb1c5c125ca891c04147bd0c752c), specification `0.1.472`; verified current main via repository recent commits on this date.
**Live decisions:** [D216/#964](https://github.com/guillermomolina/protos/issues/964) and [D217/#965](https://github.com/guillermomolina/protos/issues/965).
**Durable decision record:** [D217 M3-local](../../decisions/language/D217_ACTOR_LOCAL_TCP_ACCEPT_ONLY_DELEGATION.md).
**Status:** owner approved the bounded architectural directions; **normative publication and implementation pending**. No build, test, guest program, benchmark or shell validation was run in this coordination/evidence step.

## Precise owner corrections and choices

The maintainer corrected earlier C/P/A confusion: **C = mono CPU, A = multi CPU, P = multi nodo** (multi-Process will be addressed when needed; two Processes on one machine are not the preferred default). C/P/A must be transparent to the application; a developer should not have to manually copy or forward a TCP connection between Actors to exploit multicore.

For HTTP hosting the owner explicitly chose **Option B: reusable server infrastructure**, rather than a one-off application loop. Following the socket-ownership review, the owner specifically approved the bounded **M3-local** design: opt-in Actor-delegable accept-only capability, same Process, workers independently pull connections and process them, not generic transfer of accepted sockets.

The owner expressly insisted that **pay-as-you-grow also covers engineering effort**. Distributed/multiprocess support is a known future requirement, not something for which to create code, APIs, remote proxies, scheduling machinery or design obligations now.

## Current published product semantics

| Normative source (reviewed revision) | Established invariant |
| --- | --- |
| [`spec/io/NETWORK.md` §§8,11,12](https://github.com/guillermomolina/protos/blob/205420a03d4ebb1c5c125ca891c04147bd0c752c/spec/io/NETWORK.md) | Listener `accept()` already permits multiple outstanding Futures. Live `Network`, `TcpListener`, `TcpConnection` lack Actor/P transfer. A listener must not require a semantic single-accept bottleneck. |
| [`spec/io/PROCESS_IO.md` §26](https://github.com/guillermomolina/protos/blob/205420a03d4ebb1c5c125ca891c04147bd0c752c/spec/io/PROCESS_IO.md) | Explicit Actor-local authority/proxy may target one Process-local service; no silent proxy when `send/request` rejects non-transferable live resources; no RootActor hop required. |
| [`spec/concurrency/ACTORS.md`](https://github.com/guillermomolina/protos/blob/205420a03d4ebb1c5c125ca891c04147bd0c752c/spec/concurrency/ACTORS.md) | `Actor.spawn` transfers/validates initialization argument graphs; only explicitly delegable capability kinds cross; `NonTransferableValue` prevents partial Actor construction. |
| [`spec/io/IO_CORE.md` §8](https://github.com/guillermomolina/protos/blob/205420a03d4ebb1c5c125ca891c04147bd0c752c/spec/io/IO_CORE.md) | Close establishes an irreversible shared logical receiver cutover; pending pre-commit operations fail and already committed effects are not rolled back. |
| [`spec/concurrency/DISTRIBUTED_RUNTIME.md` §45](https://github.com/guillermomolina/protos/blob/205420a03d4ebb1c5c125ca891c04147bd0c752c/spec/concurrency/DISTRIBUTED_RUNTIME.md) | Placement feasibility accounts for missing special resources. The placement algorithm and public affinity surface are not specified. |
| [`spec/io/HTTP.md` §§1,14](https://github.com/guillermomolina/protos/blob/205420a03d4ebb1c5c125ca891c04147bd0c752c/spec/io/HTTP.md) | `std:http/Server.serveOne` and `.owning` handle exactly one HTTP/1.1 request over caller-provided Duplex, then retire; listener supervision and reuse remain out of scope. |

## Internal shared-queue control experiment

The repository's own [`tools/test/LogicalCaseRunner.protos`](https://github.com/guillermomolina/protos/blob/205420a03d4ebb1c5c125ca891c04147bd0c752c/tools/test/LogicalCaseRunner.protos) uses independent lanes which **pull** a next logical case from shared work state (`nextIndex`), spawn work via Futures and use `Future.all` as a *final* completion barrier, avoiding a guest-Actor round trip before every item. The older [PERF019/#736](https://github.com/guillermomolina/protos/issues/736) evidence identified an approximately **868 ms refill gap** for an Actor-serialized test-control design; that historical number is **not** a TCP benchmark. Its relevance is the architectural need to avoid unnecessary serialization when independent workers can compete for a shared admission resource.

## Primary-source comparative architecture

| System | Admission/ownership mechanism | Relation to Protos |
| --- | --- | --- |
| [Erlang OTP `gen_tcp`](https://www.erlang.org/docs/27/apps/kernel/gen_tcp.html) | Many processes can issue accept concurrently on one listening socket; explicit `controlling_process` may transfer connection ownership. | **Adopt concurrent pull; reject generic live-socket transfer.** |
| [Ranch listeners](https://ninenines.eu/docs/en/ranch/2.1/guide/listeners/) | Acceptor pool plus connection supervisors, with configurable pool sizes. | Supports reusable acceptor design; does not require copying supervisor infrastructure into first Protos slice. |
| [Netty 4.x](https://netty.io/wiki/user-guide-for-4.x.html) | Boss event-loop group accepts; worker group processes accepted channels. | Valid host-backend alternative. A *host* boss is not a requirement for one **guest Actor** to serialize every connection. |
| [Go `net/http`](https://pkg.go.dev/net/http#Server.Serve) | Server-level repeated accept and per-connection serving lifecycle. | Reusable optional HTTP hosting is justified above single-request parsing; its API/lifecycle belongs in D216, not D217. |
| [Boost.Asio `basic_socket_acceptor`](https://www.boost.org/doc/libs/release/doc/html/boost_asio/reference/basic_socket_acceptor.html) | Asynchronous accept may complete via executors; host-specific dispatch hidden from application protocol. | Host mechanism remains open; semantic capability/future contract is the Protos interface. |

These peer references are **precedents**, not adoption of their public APIs, ownership transfer rules, thread models, or supervisor hierarchies.

## Candidate and stress comparison

| Candidate | Benefit | Reason not to choose as Protos Core now |
| --- | --- | --- |
| One guest listener Actor accepts and dispatches all connections | Easy control flow | Serial admission, mandatory Actor hop, and requires moving non-transferable `TcpConnection`. |
| More Tasks under one Actor only | Fits a minimal C server | Actor exclusivity prevents genuine parallel CPU execution of handler guest work; does not satisfy A scaling. |
| General cross-Actor transfer of `TcpConnection` / `TcpListener` | Enables handoff after accept | Major changes to identity, ownership, cancellation and resource semantics; contradicts closed D173 without explicit replacement. |
| Generic multiprocess proxy/RPC substrate | Possible future distributed growth | Unproven current need; high design/runtime/engineering tax directly contrary to pay-as-you-grow. |
| SO_REUSEPORT per-Actor listeners or host-specific design | Can scale on some kernels | Adds platform-specific socket/reuse/load semantics and need for multiple listening authorities; not portable as required default. |
| **M3-local: explicit accept-only delegation, one Process** | Work-conserving A pull, no socket movement, precise attenuated authority | **Selected**; narrow new semantic capability and lifecycle/placement obligations must be normatively written and later implemented. |

Critical pressure cases for the editor: two Actors with pending accepts; listener closed while futures are pending; cancellation before/after irrevocable accept; accepted connection arrives after cancellation; Worker termination before and after Future delivery; a foreign-Process Actor request; source listener vs delegated capability identity; errant `send` with `TcpConnection`; no host network authority leakage; saturated or blocked workers; HTTP work limits and slow clients; absence of clock/timer guarantees.

## Exact selected boundary and handoff

- **D217** owns `TcpAcceptor` explicit same-Process delegation, shared logical listener, same Worker Actor ownership for accepted `TcpConnection`, close/cancellation custody and hidden compatibility placement. **It has not yet changed `NETWORK.md`.**
- **D216** owns the independently selected **Option B** reusable *optional* `std:http` continuous listener infrastructure, built out of D217 plus already published `serveOne.owning`. The `Server.serve...` public signature, bounded worker admission, shutdown/drain/completion and error precedence still require their own exact normative contract and owner approval. It has not been implemented.
- **D215/#961 and LIB025/#962 stay CLOSED**; one-request HTTP semantics, including mandatory `Connection: close`, are unchanged.
- No implementation `Ixxx` is fabricated or scheduled before the normative phase is accepted.

For this coordination and documentation turn, no code was edited in `guillermomolina/protos`, no tests were run, no source-level measured CPU scaling is claimed, and no HTTP/TCP throughput improvement is asserted.