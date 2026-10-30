# D225-B1 — RootActor message-service bootstrap: normative and implementation publication (0.3.426)

**Date:** 2026-10-30. **Role:** durable, revision-bound completion evidence for [D225 #992](https://github.com/guillermomolina/protos/issues/992). This record is non-normative: only `guillermomolina/protos/spec/` defines Protos semantics. Live coordination remains with the Issue.

**PROTOS_REVISION:** [`0ff5c9828e64b9a127d945f4e2cb98053dcb7c23`](https://github.com/guillermomolina/protos/commit/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23)
**IMPLEMENTATION_VERSION:** `0.3.426-SNAPSHOT`
**SPECIFICATION_REVISION:** `0.1.483`
**PRODUCT_CHANGESET:** 25 files, 2,213 insertions, 23 deletions; committed and pushed to `guillermomolina/protos` `main` by the maintainer on 2026-10-30.
**Formal decision:** [D225-B1 owner-selected record](../../decisions/language/D225_B1_OPT_IN_ROOT_ACTOR_MESSAGE_SERVICE_BOOTSTRAP.md).
**Exact prior approval and GITHUB021 reconciliation:** [D225-B1 approval evidence](D225_B1_OWNER_SELECTION_AND_GITHUB021_RECONCILIATION_2026_10_30.md).
**Prior, separately published D225-A / I133:** [I133 completion](../I133/I133_0_3_422_HTTP_HOST_WORKER_FACTORY_COMPLETION_2026_10_30.md).

## Publication and normative ownership

The product revision contains the B1 implementation and one coherent normative revision, rather than merely a design selection.

- [`spec/concurrency/ACTORS.md`](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/spec/concurrency/ACTORS.md), `32: the existing RootActor has an explicit opt-in message-service initial entry. One canonical module and one destination-local callable binding establish one ordinary behavior before `READY`. Invalid bootstrap fails the Process. No implicit service, new Actor protocol or post-READY behavior replacement.
- [`spec/semantics/MODULES.md`](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/spec/semantics/MODULES.md): initial module cache, canonical module identity, relative imports and local binding selection for the service entry; ordinary direct entry remains unchanged.
- [`spec/io/PROCESS_IO.md`](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/spec/io/PROCESS_IO.md): RootActor-only bootstrap-local Process, Network and Resolver slots, with independent explicit grants; no transfer of live capabilities to imports or Worker Actors.
- [`spec/concurrency/DISTRIBUTED_RUNTIME.md`](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/spec/concurrency/DISTRIBUTED_RUNTIME.md): existing direct single-member `Actor.group(Actor.current())` GroupRef, ordinary `send`/`request` routing and no `stop`.
- [`spec/PROTOS_SPEC_CHANGELOG.md`](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/spec/PROTOS_SPEC_CHANGELOG.md) records `0.1.483`. [`CHANGELOG.md`](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/CHANGELOG.md) records `0.3.426-SNAPSHOT`.

`spec/io/HTTP.md`, `Host.owning` and `Host.owningWithFactory` retain their D215/D216/D225-A contracts. The already-committed `Closable.close()` semantics remain unchanged: the lifecycle publication fix allows the pending close outcome to reach cancellation cleanup in the owner Actor domain, never rewriting committed close to cancellation.

## Implementation surfaces

- [`ProtosRootActorServiceSession.java`](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosRootActorServiceSession.java): explicit hosted RootActor-service entry, independent host-grant selection, optional authorized `app:` catalog, terminal disposition and deterministic host/process ownership.
- [`ProtosActorBootstrap.java`](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/main/java/com/guillermomolina/protos/execution/runtime/concurrency/ProtosActorBootstrap.java): destination-local canonical-module bootstrap and scheduled RootActor control turn; preserves module identity also for ordinary Actor bootstrap imports.
- [`ProtosActorExecutionDomain.java`](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/main/java/com/guillermomolina/protos/runtime/ProtosActorExecutionDomain.java) and [`ProtosIoLifecycle.java`](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/main/java/com/guillermomolina/protos/runtime/io/ProtosIoLifecycle.java): targeted publication of committed close outcomes during a terminating Actor's live Task cleanup. Ordinary targeted runtime completions still reject Actor termination; no message dispatch resumes after the termination cutover.
- [`ProtosStandaloneHostedExecution.java`](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosStandaloneHostedExecution.java): reused hosted Process setup while preserving direct-entry defaults.
- [Root-service example](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/protos/examples/actors/root-message-service.protos), [HTTP host example](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/protos/examples/http/root-service-host.protos), [HTTP handler example](https://github.com/guillermomolina/protos/blob/0ff5c9828e64b9a127d945f4e2cb98053dcb7c23/protos/examples/http/root-service-handler.protos), and updated embedding/Actor/HTTP guides show application-owned composition.

## Validation — maintainer-executed, reported evidence

These are reports from the maintainer's local checkout, not claims that the documentation agent executed the project programs:

| Gate | Reported result |
|---|---|
| `git diff --check HEAD` / staged `git diff --cached --check` | PASS |
| D225-B1 and affected regression selection | **117 tests, 0 failures, 0 errors, 0 skipped** |
| Focused RootActor fatal termination after removing temporary diagnostics | **1 test, 0 failures, 0 errors** |
| Focused HTTP fatal termination, TCP delegation and I/O lifecycle set after the close-publication repair | **29 tests, 0 failures, 0 errors** |
| `make test` (integrated Java + Protos) | **PASS** |
| `make check` | **PASS** |
| Pre-publication version/branch/changed-file gate | **PUBLICATION_GATE=PASS** |
| Final staged product scope | **25 files**; clean staged whitespace check |
| Maintainer commit + push | **PASS**, exact product SHA above |

The maintainer obeyed the agreed ordering: complete tests and checks before updating `pom.xml` / `CHANGELOG.md`, followed by no tests after those version metadata changes. The static staged diff remained clean.

**License check:** the six newly added Protos-owned source files (three examples, one production Java class, two Java integration test classes) were inspected at the exact product revision; all contain the Protos APL 1.0 Part 5 notice. Existing modified sources retained their headers.

## B1 approval / invariant reconciliation

**Approval:** the owner explicitly selected B1 on 2026-10-30: “Apruebo D225-B1: bootstrap explícito opt-in del RootActor como servicio de mensajes y GroupRef de comunicación, con las condiciones y límites del informe. Procede con la ratificación según AGENTS.md.” The prior [selection record](D225_B1_OWNER_SELECTION_AND_GITHUB021_RECONCILIATION_2026_10_30.md) records the GITHUB010 comparison and GITHUB021 invariant/delta as PASS. The product revision implements that approved scope.

**Preserved:** D225-A Worker-local handler factory and one-argument HTTP handler; GroupRef communication-only surface without changing ActorRef authority; independently granted Root-only Network and Resolver; no implicit host grants, mutable service registry, extra service Actor or new Host API; ordinary entry modes remain direct; Host drain/abort do not terminate the service Process; fatal RootActor failure does terminate the Process. The I/O close-publication repair preserves irrevocable committed close semantics.

**Not selected or claimed:** independent capability-bearing B2 broker Actors, per-Worker egress grants (C), selector-level GroupRef attenuation, transparent failover, remote discovery, automatic cancellation propagation, a new CLI syntax, or a globally available Resolver grant in ordinary Polyglot embedding. No benchmark, Native Image conformance, or production-throughput claim is inferred from the reported local gates.

## Closure and cross-repository reconciliation

The selected D225-A factory seam had already been normatively published as I133. D225-B1 now supplies the remaining owner-selected initial RootActor service message composition, with its normative specification and implementation in the same product revision. This closes the approved D225 design outcome; unselected B2/C alternatives remain future separate design decisions, not hidden follow-up requirements.

The [D225 Issue #992](https://github.com/guillermomolina/protos/issues/992) is the live closure and status authority. Its final closure comment must reference **PROTOS_REVISION=0ff5c9828e64b9a127d945f4e2cb98053dcb7c23** and the exact **PROJECT_RECORD_REVISION** of the project-docs commit that contains this evidence and the reconciled selected decision. A moving `main` link alone does not satisfy that requirement.

`DECISION_APPROVAL_PROVENANCE=PASS`
`DECISION_INVARIANT_CONSISTENCY=PASS`
`B1_NORMATIVE_PUBLICATION=PASS`
`B1_IMPLEMENTATION_PUBLICATION=PASS`
`DURABLE_RECORD_DECISION=REQUIRED`
`D225_CLOSURE_EVIDENCE=THIS_RECORD`
