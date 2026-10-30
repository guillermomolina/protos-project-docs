# D225-B1 — Opt-in RootActor message-service bootstrap with communication-only GroupRef

**Decision family:** D225, residual application-service provisioning after D225-A.
**State:** **OWNER SELECTED / NORMATIVELY PUBLISHED / IMPLEMENTED.** D225-B1 is published in Protos `0.3.426-SNAPSHOT`, specification `0.1.483`, at the exact product revision linked below. The original 2026-10-30 selection checkpoint is preserved as historical evidence.
**Approval date:** 2026-10-30.
**Exact owner approval:** “Apruebo D225-B1: bootstrap explícito opt-in del RootActor como servicio de mensajes y GroupRef de comunicación, con las condiciones y límites del informe. Procede con la ratificación según AGENTS.md.”
**Live coordination:** [D225 #992](https://github.com/guillermomolina/protos/issues/992).
**Protos product baseline inspected:** [`e88f4f0e510320ede0ec61bd67dfd671f30fc2cb`](https://github.com/guillermomolina/protos/commit/e88f4f0e510320ede0ec61bd67dfd671f30fc2cb) (`0.3.422-SNAPSHOT`, spec `0.1.481`).
**Previously implemented slice:** [D225-A selected record](D225_EXPLICIT_HTTP_HOST_WORKER_FACTORY.md), [I133 #995](https://github.com/guillermomolina/protos/issues/995), [I133 publication evidence](../../evidence/I133/I133_0_3_422_HTTP_HOST_WORKER_FACTORY_COMPLETION_2026_10_30.md).
**Authority boundary:** This remains durable **non-normative decision/approval provenance** in `guillermomolina/protos-project-docs`. The authoritative B1 semantics were subsequently published under `guillermomolina/protos/spec/` at **PROTOS_REVISION=0ff5c9828e64b9a127d945f4e2cb98053dcb7c23**; the earlier I133 baseline above remains historical.

## Published B1 outcome and revision-bound closure reconciliation — 2026-10-30

**PROTOS_REVISION:** [`0ff5c9828e64b9a127d945f4e2cb98053dcb7c23`](https://github.com/guillermomolina/protos/commit/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23)
**IMPLEMENTATION_VERSION:** `0.3.426-SNAPSHOT`
**SPECIFICATION_REVISION:** `0.1.483`
**Exact B1 publication / validation evidence:** [D225-B1 completion evidence](../../evidence/D225/D225_B1_0_3_426_ROOT_ACTOR_SERVICE_PUBLICATION_AND_CLOSURE_2026_10_30.md).

The reviewed, maintainer-published product revision simultaneously updates `ACTORS.md` `32, `MODULES.md`, `PROCESS_IO.md` and `DISTRIBUTED_RUNTIME.md`, adds the explicit hosted `ProtosRootActorServiceSession.open(...)` API, canonical Actor-bootstrap module identity and supporting authority/HTTP integration tests. The implementation revision also repairs publication of already-committed listener close outcomes needed by live cancellation-cleanup Tasks, without changing the irrevocable `close()` contract.

The maintainer reports `make test` PASS and `make check` PASS before the version/changelog edits, plus 117 focal tests with zero failures or errors; `git diff --cached --check` was clean. New source license notices were checked at the exact product revision. The application-level B1 GroupRef path transfers neither Network nor Resolver and changes no D225-A HTTP Host method.

The following handoff section and its `NOT_PUBLISHED` fields reproduce the **earlier selection-time checkpoint**, not current repository state. The exact published completion and remaining exclusions are recorded in the linked immutable evidence. D225 closure and scheduling state are owned by GitHub Issue #992, not by this document.

## Decision and motivating gap

D225-A lets the owner of an HTTP Host pass Actor-transferable initialization to isolated Worker-local handler factories. The handler may receive an ActorRef or GroupRef, but D225-A neither creates an application service nor transfers live outbound Network/Resolver authority to the Worker. The missing capability is a **message-receiving application service which holds the RootActor's independently host-granted Network and Resolver without sending those non-transferable resources across an Actor boundary**.

**Selected B1:** define an **explicit, optional RootActor entry/bootstrap mode**, using the *existing* module-identity + local-binding bootstrap model, such that the RootActor's initial module may acquire separately granted bootstrap-local capabilities, execute a locally resolved bootstrap binding, return an ordinary valid Actor behavior object, and install it at the existing `INITIALIZING -> READY` cutover. The application may create `Actor.group(Actor.current())`, a **single-member GroupRef**, and pass this communication-only GroupRef as ordinary transferable data through the already normatively published `Host.owningWithFactory` operation to HTTP Worker factories. Workers use ordinary `GroupRef.request` / `send`; they obtain **neither** `Network`/`Resolver` nor the RootActor's `ActorRef.stop` capability.

This is **not** a new RootActor kind, mutable post-READY behavior installation, service/DI registry, additional `Actor` public selector, new Process identity, automatic service Actor, transferable resource or extra Host API. Ordinary direct-entry/REPL/`-e`/embedded evaluation remains unchanged by default. The opt-in mode must be requested explicitly by an authorized launcher/host and must not be inferred from a returned object, import, slot name or source text.

## Selected architectural invariants

1. **Explicit opt-in at Process initial-entry creation.** A normal Protos program retains its existing standalone/embedded entry semantics and pays no extra routing, Group, Actor or authority cost. A selected service-bootstrap entry uses the *existing* initial RootActor incarnation, never an additional privileged Actor or `ApplicationRoot`.
2. **Destination-local behavior.** The bootstrap identity is one canonical module identity; the local binding must exist on the destination-local initial module. Bootstrap invokes it *within that RootActor*; it returns one ordinary behavior object, fixed at the existing Actor READY cutover. No `Actor.setBehavior`, `Actor.serve`, `become`, transplanted caller Closure, or post-READY mutation of the current behavior.
3. **Independent explicit authority.** `network` exists only under its own host grant. `resolver` exists only under its separate host grant; no special Polyglot Resolver authorization is invented. The initial module, not imports or new Actors, receives only existing bootstrap-local capability slots actually granted by the host. `Process` possession grants neither. No implicit inheritance, automatic service locator or fallback.
4. **Communication-only sharing.** `Actor.group(Actor.current())` is the existing `Actor.group(...)` operation with exactly one member. Its `GroupRef` has `send`/`request` but no `stop`; it retains its ordinary identity, Process-owning lifetime and routing/backpressure/uncertainty rules, and is Actor-transferable. This is **not ActorRef attenuation**: an ActorRef previously handed out still retains all its normal authority.
5. **Preserve Actor semantics.** Before RootActor READY, no message is dispatched to application behavior. Accepted/delayed/pre-admission message handling follows current bounded Actor and Group semantics. Fatal unhandled RootActor error terminates the Protos Process and consequently its hosted Actors/resources; handled errors remain ordinary service outcomes. No transparent replay, retry, reassignment or guaranteed ordering between senders.
6. **Avoid startup deadlock.** The RootActor bootstrap may configure Host and transfer a GroupRef, but may not suspend awaiting `host.ready()` when any Worker needs a reply from the RootActor to become Host READY. The runtime must not invent an implicit nested entry or dispatch external messages during RootActor INITIALIZING to resolve this cycle.
7. **Existing HTTP boundaries.** D215 `handler(request)`, D216 worker/connection custody, quota, drain/abort, readiness and cleanup, and D225-A `owningWithFactory` remain unchanged. The RootActor service is application-owned; Host neither supervises nor stops it. Host shutdown is not Process termination.
8. **Constrain service API at application level.** A GroupRef limits generic capability *kind*, not allowed message selectors. An application receiving public HTTP input must expose bounded methods, validate outbound target/arguments, keep credentials private and explicitly configure TLS trust; avoid an unrestricted `fetch(arbitraryUrl, arbitraryHeaders)` egress proxy. `GroupRef` is not a generic selector-level authorization membrane.
9. **No implicit adoption/transport of Futures or live resources.** A service must explicitly await local operation Futures before replying with an Actor-transferable result. A Future, Closure, `Network`, `Resolver`, socket, listener or active connection is not a valid Actor reply.
10. **Portable implementation boundary.** RootActor service-bootstrap semantics must be specified without depending on JVM/Truffle/Java thread identities or a particular CLI command. The implementation may reuse `ProtosActorBootstrap.initialize`, but that implementation method is evidence, not a normative mechanism.
11. **Scalability limitation accepted.** One-member GroupRef does not create parallel service Actors. RootActor has one Actor-local execution domain, with cooperative Task interleaving, and is the Process failure root. B1 is a deliberately minimal solution, not an HA/scaling or independent-failure architecture.
12. **Scope exclusion.** D225-B1 does not supply a general host mechanism for granting `Network`/`Resolver` to child Actors, a cross-Process service broker, service discovery, mutable Group membership, pooled HTTP clients, new TLS/JSON/REST or UrlClient behavior, timeouts, reconnection or automatic HTTP/Actor request cancellation propagation.

## GITHUB021 consistency: previously approved invariant -> B1 delta

| Approved source invariant | B1 disposition |
|---|---|
| D225-A: `Host.owning` and `Host.owningWithFactory` public contracts remain; handler is `handler(request)` | **PRESERVED.** B1 supplies a pre-existing GroupRef through D225-A; no Host API revision |
| D225-A: Actor transfer copies allowed data; refuses live `Network`, `Resolver`, Closure, Future and caller `TcpAcceptor` | **PRESERVED.** Only GroupRef crosses |
| D225-A: service lifetime is independent from Host; no implicit actor owner/parent lookup | **PRESERVED.** RootActor owns its state; Host owns its Workers and listener |
| D223-A1/D047: Root-only separately granted `Network`/`Resolver` and no Actor inheritance | **PRESERVED.** B1 uses capabilities locally inside authorized RootActor |
| Actor §8/§11A: one bootstrap behavior fixed at READY; no second installation API | **PRESERVED.** New mode uses existing initial cutover |
| ActorRef: communication plus `stop()` are retained; no implicit attenuation | **PRESERVED.** Use separately existing GroupRef with different public protocol, not a restricted ActorRef |
| D225-A exclusion: no **automatic** RootActor service publication, no existing broker grant | **PRESERVED AS HISTORICAL A SCOPE.** B1 newly approves an explicit opt-in RootActor *entry*; this does not retroactively make A automatically publish a service |
| Distributed runtime: no core application/service identity or discovery registry | **PRESERVED.** GroupRef is direct acquisition from a held ActorRef, not lookup or registry |
| RootActor fatal failure terminates Process; Actor-local mutable state isolated | **PRESERVED.** This risk is acknowledged rather than patched by auto-supervision |

**New owner-approved delta only:** an explicit initial-entry RootActor service-bootstrap selection that installs its ordinary behavior before READY; application composition may transfer a direct single-member GroupRef. No previously approved invariant is reopened. **DECISION_APPROVAL_PROVENANCE=PASS; DECISION_INVARIANT_CONSISTENCY=PASS for the selected B1 architecture.**

## Why not another design

The investigation leading to owner selection compared status quo/deferral (B0), B1, independent service Actor with explicit resource provisioning (B2) and host-provisioned per-Worker live authority (C), with Go handlers, Erlang/OTP, Akka Typed, Pony, Rust/Tokio ownership, WASI component capabilities and Orleans as materially different precedents. B1 minimizes new semantic surface: it uses ordinary Actors, direct Group acquisition and existing application factory composition. B2 isolates failures and scales better but first requires an independent authorization/delegation/revocation contract, which current need does not justify; C multiplies outbound grants and couples inbound HTTP workers to egress policy; B0 cannot provision an authorized message-serving service.

The original D225-A comparative research remains [here](../../evidence/D225/D225_A_RESEARCH_AND_APPROVAL.md). The B1-specific comparison and twelve-dimension GITHUB010 scorecard were supplied to the owner in the active 2026-10-30 investigation; the exact, bounded approval above selects B1, **not** B2 or C.

### Known limits, strongest counterargument and escape path

A fatal unhandled error in the RootActor ends the whole Process, and a single RootActor serializes guest-code segments. For a high-load production service this may be an unacceptable shared failure/throughput domain. **Strongest counterargument:** provisioning an independently authorized, ordinary child service Actor would provide fault isolation and parallelism from the beginning. **Reversibility:** D225-A accepts explicit transferable communication dependencies irrespective of service placement; a future separately approved B2 broker can provide them without changing `handler(request)` or HTTP Host factory transfer semantics. A future Group strategy may require new policy/management APIs and rewriting application bootstrapping; Core's one-member GroupRef does **not** currently allow replacement or retargeting. Do not claim transparent B1 -> B2 migration.

## Historical normative-publication handoff — pending at initial B1 selection

The owner selected the architecture but did **not** approve a specific new CLI flag, command syntax, Polyglot Context option, launcher invocation signature, module locator syntax, host service registry or failure taxonomy. These are **public-entry design details** if guest-visible/portable and must be made precise under the D225 approved invariants before normative publication. A concrete CLI flag/embedding selector may require further explicit owner approval rather than being guessed during implementation.

**Required normative owners to reconcile in one coherent human-published product change:**
- `spec/concurrency/ACTORS.md` §§8, 9, 11A, 32: selected RootActor service-bootstrap mode and existing behavior/failure rules; no new Actor public selector;
- `spec/io/PROCESS_IO.md` §22/Root network and resolver provisioning: same Root-only, grant-independent authority and bootstrap locality;
- `spec/semantics/MODULES.md` initial importable module / host-initiated entries: selected initial-module identity, local binding, and no duplicate host evaluation/implicit module cache;
- `spec/concurrency/DISTRIBUTED_RUNTIME.md` §§50A, 72A-B: direct GroupRef composition without changing Group identity/lifetime/creation;
- `spec/io/HTTP.md` §15.13: cross-reference only if needed, not a Host contract revision;
- `spec/PROTOS_SPEC_CHANGELOG.md`: next current-HEAD-derived global specification revision.

**Acceptance evidence to require of an implementation work item:** opt-in and legacy modes; RootActor local binding and invalid bootstrap; one behavior at READY; grant combinations (neither, Network-only, Resolver-only, both); no ambient grant/child inheritance; GroupRef transferred into multiple Host Workers; no Worker ActorRef or stop on GroupRef; bounded messages before Root READY; no Host-ready/bootstrap circular wait; fatal RootActor Process teardown; Host drain/abort independent of Root service; handled vs fatal service errors, Actor reply transfer rejection, request late replies; standalone/embedding portability and Native gate when applicable.

**Historical publication state at the original selection checkpoint (superseded by the exact 0.3.426 publication above):**
```text
WORK_ITEM=D225-B1
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
B1_DECISION_RECORD=SELECTED
B1_PRODUCT_NORMATIVE_REVISION=NOT_PUBLISHED
B1_PRODUCT_IMPLEMENTATION=NOT_PUBLISHED
D225_FORMAL_CLOSURE=DEFER_UNTIL_SPEC_PUBLICATION
```

This record ratifies the **selected design decision as project provenance**, not a normative language revision. Once a reviewed product spec commit exists, record its exact SHA and spec revision in separately revision-bound evidence, verify live Issue state and cross-references, and only then consider D225 completed. Implementation can remain an independently tracked Ixxx after coherent normative publication if its own scope and dependencies are defined.

## References

- [D225 issue](https://github.com/guillermomolina/protos/issues/992); [D225-A selected record](D225_EXPLICIT_HTTP_HOST_WORKER_FACTORY.md); [I133 closure](../../evidence/I133/I133_0_3_422_HTTP_HOST_WORKER_FACTORY_COMPLETION_2026_10_30.md)
- [Protos ACTORS.md](https://github.com/guillermomolina/protos/blob/e88f4f0e510320ede0ec61bd67dfd671f30fc2cb/spec/concurrency/ACTORS.md), [PROCESS_IO.md](https://github.com/guillermomolina/protos/blob/e88f4f0e510320ede0ec61bd67dfd671f30fc2cb/spec/io/PROCESS_IO.md), [MODULES.md](https://github.com/guillermomolina/protos/blob/e88f4f0e510320ede0ec61bd67dfd671f30fc2cb/spec/semantics/MODULES.md), [DISTRIBUTED_RUNTIME.md](https://github.com/guillermomolina/protos/blob/e88f4f0e510320ede0ec61bd67dfd671f30fc2cb/spec/concurrency/DISTRIBUTED_RUNTIME.md), [HTTP.md §15.13](https://github.com/guillermomolina/protos/blob/e88f4f0e510320ede0ec61bd67dfd671f30fc2cb/spec/io/HTTP.md)
- [ProtosActorBootstrap.java (implementation evidence only)](https://github.com/guillermomolina/protos/blob/e88f4f0e510320ede0ec61bd67dfd671f30fc2cb/src/main/java/com/guillermomolina/protos/execution/runtime/concurrency/ProtosActorBootstrap.java), [ProtosEmbeddedProcess.java (implementation evidence only)](https://github.com/guillermomolina/protos/blob/e88f4f0e510320ede0ec61bd67dfd671f30fc2cb/src/main/java/com/guillermomolina/protos/execution/host/resources/ProtosEmbeddedProcess.java)
