# I125-B — same-Process Actor.spawn TcpAcceptor delegation checkpoint (0.3.406)

**Date:** 2026-10-26. **Role:** published implementation evidence, not normative authority.
**Owning implementation:** [I125 #966](https://github.com/guillermomolina/protos/issues/966) (OPEN for final end-to-end conformance/closure).
**Ratified design:** [D217 #965](https://github.com/guillermomolina/protos/issues/965), [specification 0.1.473](https://github.com/guillermomolina/protos/commit/5f95dc086fc9bc853afb712915db47ab28547417).
**Exact product publication:** [`guillermomolina/protos@da71217a4d4ecaba4b399e905aee119aef9d14fd`](https://github.com/guillermomolina/protos/commit/da71217a4d4ecaba4b399e905aee119aef9d14fd), **0.3.406-SNAPSHOT**.
**Parent product commit:** [I125-A `bc3730539d73c94bb0f95cb9b699e4f2de0dde2d`](https://github.com/guillermomolina/protos/commit/bc3730539d73c94bb0f95cb9b699e4f2de0dde2d).
**Source verification:** GitHub's exact product commit, main ref, changed-path inventory, and selected runtime/coverage files were inspected. No local tests were independently executed by this coordinator.

## Published scope

The 15-file product commit implements the ratified **same-Process, Actor.spawn-initialization-only** delegation of an attenuated `TcpAcceptor` over the listener's existing flow.

- `ProtosStandardActorProtocol` selects the special spawn argument snapshot while ordinary snapshots, send/request/replies and P transfer remain unchanged and reject concrete acceptors.
- `ProtosActorValueTransfer` captures an inert authority descriptor without copying source-added slots or structural state, validates the complete enclosing graph, and materializes one destination-local OPEN, slotless capability per source identity. It preserves aliases, cycles, ordinary parent chains and destination Map/IdentityMap bookkeeping; a foreign Process incarnation is inadmissible.
- `ProtosTcpAcceptorValue`, `ProtosTcpListenerFlow`, `ProtosNetworkAcquisitionInvocation`, `ProtosIoOperation` and `ProtosIoLifecycle` support the shared logical listener with an operation, Future, and accepted connection owned by the accepting Actor. The listener owner alone retains close authority.
- Cancellation, cutover and discarded committed completions retire the lifecycle registration and release an undelivered host connection without duplication. A dropped foreign operation does not indefinitely hold listener close.
- TCP listener release callbacks not reported synchronously from the owner's own close call are published through the existing owner-targeted queue; guest `IOError` construction and close Future settlement happen in the owner domain. A terminating owner uses host-only cleanup. This mode is TCP-listener opt-in; Filesystem/Text I/O families retain their existing lifecycle behavior.
- No continuous `std:http` API, cross-Process acceptor RPC, ambient Network authority or generic live-socket transfer was added. The separate [D216 #964](https://github.com/guillermomolina/protos/issues/964) remains open for an exact HTTP hosting contract.

## Verified changed-path and test evidence

GitHub confirms **15 changed paths**: two publication metadata files (`pom.xml`, `CHANGELOG.md`), seven production paths, and six test paths. The new `ProtosTcpAcceptorActorDelegationTest` tests one shared backend across listener and Worker accepts, Actor-local materialization, owner publication, close/failure/cancel races, undelivered socket retirement, termination and forbidden transfers.

Expanded `ProtosActorValueTransferTest` covers spawn-only admission, exact Process compatibility, graph atomicity, aliases/cycles, source slot/state attenuation and keyed collections. Expanded `ProtosActorBootstrapTest` covers actual spawn/module bootstrap and refusal of unrelated transfer. Existing TCP listener accept, lifecycle and integrated conformance tests were adapted to the targeted owner-published asynchronous close contract.

**Human executor report (2026-10-26):** `git diff --check` clean; **all local tests PASS**; I125-B commit pushed. These are maintainer-reported results, not independently collected logs. No exact command transcript was supplied for the top-level `make test` gate or an additional real-NIO cross-Actor end-to-end integration test.

## Remaining closure boundary

I125 remains OPEN. Its optional slice C is justified only for a bounded residual end-to-end integration gap: demonstrate on the real loopback/NIO backend that one Process-hosted source listener and distinct receiving Actor can accept through the delegated capability, own the resulting connection in the receiving Actor, and close/drain without an extra backend or resource leak. Do not reproduce the extensive RecordingBackend race matrix or change production semantics just for this test.

Before closing I125, the human executor should report the required final `make test` gate and other applicable checks on the final executable delta. Preserve the published I125-A and I125-B checkpoints as separate historical evidence; publish a final closure record pinned to its exact product revision once the remaining gate is satisfied.
