# I125-A — Actor-local TcpAcceptor implementation checkpoint (0.3.405)

**Date:** 2026-10-26. **Role:** immutable implementation/publication evidence, not normative authority.
**Owning implementation:** [I125 #966](https://github.com/guillermomolina/protos/issues/966) (OPEN; I125-B remains).
**Ratified decision:** [D217 #965](https://github.com/guillermomolina/protos/issues/965), [specification 0.1.473](https://github.com/guillermomolina/protos/commit/5f95dc086fc9bc853afb712915db47ab28547417).
**Exact Protos publication:** [`bc3730539d73c94bb0f95cb9b699e4f2de0dde2d`](https://github.com/guillermomolina/protos/commit/bc3730539d73c94bb0f95cb9b699e4f2de0dde2d), **implementation 0.3.405-SNAPSHOT**.
**Parent commit:** `5f95dc086fc9bc853afb712915db47ab28547417`.
**Source verification:** GitHub commit metadata and its 17-file delta independently read; no tests were executed by the coordinating agent.

## Published scope

The atomic I125-A commit adds the private, frozen `TcpAcceptor` behavior prototype and a concrete Actor-local accept-only representation. `listener.acceptCapability()` synchronously derives a fresh capability from the same `ProtosTcpListenerFlow`, without opening another socket or acquiring `Network`. `acceptor.accept()` participates in the existing listener's pending accepts, targeted completion, lifecycle and host-resource release machinery. Acceptor structural `Object.close()` is not the listener close operation.

Runtime changes include:

- `protos/lib/core/network/TcpAcceptor.protos`: canonical internal prototype source, with no public constructor or Prelude binding.
- `ProtosCoreBootstrap` and `ProtosPrelude`: source/prototype registration, freezing, validation, runtime-only identity access.
- `ProtosTcpAcceptorValue` and `ProtosStandardTcpAcceptorProtocol`: physical Actor ownership and the single native `accept` selector.
- `ProtosTcpListenerValue`, `ProtosTcpListenerFlow` and `ProtosStandardTcpListenerProtocol`: `acceptCapability` acquisition and shared listener lifecycle/backend.
- `ProtosActorValueTransfer` and `ProtosParallelRuntime`: explicit negative transfer of a concrete acceptor through ordinary Actor and P boundaries; frozen prototype recognized as canonical standard state.
- JUnit coverage in `ProtosTcpAcceptorTest`, `ProtosTcpListenerFoundationTest`, `ProtosTcpListenerAcceptTest` plus source/native-boundary architecture guards: identity, slotless acquisition, inheritance, cutover, fresh Errors/Futures, mixed pending accepts, cancel/late resource release, listener close, actor termination, transfer exclusions.

GitHub confirms **17 changed paths** (4 new feature/test files, 11 existing source/test files, plus `pom.xml` and `CHANGELOG.md`).

## Validation and scope boundary

The maintainer reports that `git diff --check` is **clean**, **all local tests passed**, and the commit was **pushed**. This is an owner-reported validation result; it does not establish an independently inspected test transcript or a separate native/performance benchmark.

**I125 is not closed.** I125-A implements only same-Actor acceptance. Its `ProtosTcpListenerFlow` and `ProtosIoLifecycle` still enforce the creator-domain boundary; the `Actor.spawn` initialization transfer still rejects the concrete acceptor. I125-B must explicitly provision a fresh destination-Actor representation within the same Process, preserve ordinary graph atomicity/aliases while dropping source capability slots/state, route each accepted connection to the accepting Actor, and preserve `send/request/P` rejection, close/commitment and late-resource custody. Complete focused conformance in B; use a separate C only for material residual work.

Do not conflate this implementation checkpoint with the D217 normative decision's closure or with D216 continuous HTTP hosting. No continuous `std:http` API, listener delegation across Processes, ambient Network permission, generic socket transfer or guest Actor admission bottleneck is authorized by this commit.
