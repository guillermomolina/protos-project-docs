# D225-A — Research evidence, alternatives and approval checkpoint

**Evidence class:** decision-stage research snapshot, 2026-10-30. **Status:** A owner-approved; B (authorization/provisioning of an outbound service Actor) remains unselected. **Owner:** [D225 #992](https://github.com/guillermomolina/protos/issues/992).

## Repository observations

- `protos/lib/http/Host.protos` has only `Host.owning(listener, handlerModule, handlerBinding, options)`; it spawns `std:http/internal/HostWorker` with an internal delegated TcpAcceptor, the handler Strings, quotas and captured request limits.
- `protos/lib/http/internal/HostWorker.protos` imports the handler module inside its Actor, selects one own callable, and invokes `Server.serveOne.owning` with a single-argument handler. The Worker publishes `serve`, `ready`, `drain` and `abort`.
- `protos/lib/http/internal/HostLifecycle.protos` coordinates Workers via ActorRefs and owns listener close, abort/drain, quotas and observer Futures; it does not own external services.
- `spec/concurrency/ACTORS.md` permits ActorRef transfer preserving destination identity, but mutable ordinary values copy. Native/guest Closures, Futures, Network/Resolver and most I/O resources are not transferable. Internal TcpAcceptor delegation is an intentionally narrow exception.
- `spec/io/NETWORK.md` and D223 grant `Network`/`Resolver` independently only to an authorized RootActor initial module; imports and Actor.spawn do not confer either.
- `src/main/java/com/guillermomolina/protos/execution/runtime/concurrency/ProtosActorBootstrap.java` has destination-local initialization and recognizes only RootActor bootstrap-local host authorities; `ProtosStandaloneProcessBootstrap` constructs the Process/RootActor. Running a source file as the initial module does not itself install a publicly message-serving RootActor behavior. Consequently `Actor.spawn("app:broker","start",network,resolver)` is invalid, and passing `Actor.current()` to a Worker does not prove that the RootActor handles service requests.

## Prior art (mechanisms, not copied APIs)

1. Go `net/http`: closure-composed handlers capture dependencies; direct sharing assumes Go's memory/concurrency model.
   https://pkg.go.dev/net/http#Handler
2. Rust axum `State`: explicit per-router state; `Clone` and `Arc` semantics do not correspond to Protos Actor transfer.
   https://docs.rs/axum/latest/axum/extract/struct.State.html
3. Python ASGI lifespan/scope: application startup/shutdown; state delivered with request scope, no implicit cross-event-loop live-resource transport.
   https://asgi.readthedocs.io/en/latest/specs/lifespan.html
4. ASP.NET Core DI: singleton/scoped/transient scopes and service provider; strong functionality but disproportionate conceptual/runtime surface for the present Protos requirement.
   https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
5. Erlang/OTP `gen_server`: isolated process state and explicit message/reply; lifecycle and supervision are separate contracts.
   https://www.erlang.org/doc/system/gen_server_concepts.html
6. Akka Typed: explicit ActorRef messaging and ask pattern; timeout is not a guarantee to undo downstream effect.
   https://doc.akka.io/libraries/akka-core/current/typed/interaction-patterns.html
7. Pony reference capabilities: cross-Actor transfer constrained by capability safety.
   https://tutorial.ponylang.io/reference-capabilities/

## Alternatives

| Candidate | Present need | Main advantage | Fatal or material limitation | Outcome |
|---|---|---|---|---|
| D0 status quo | No | No new surface | Cannot pass caller-owned service references | Do not choose as solution |
| A local factory | Yes, for existing ActorRef/data dependencies | Small explicit opt-in API and unchanged D215 handler | Does not provision broker Network/Resolver | **Approved partial contract** |
| B A + service broker | Potentially full | Actor-local live authority retained | Existing public service provision route not demonstrated | Defer separate authority decision |
| C Network/Resolver per Worker | Potentially full | No broker communication overhead | Reopens actor/non-transferability grants and revocation rules | Rejected absent new approval |
| E Host DI container | More than required | Scopes/discovery centralized | New privileged institution and hidden composition | Rejected overengineering |

## GITHUB010 12-dimensional comparative scoring

Scale 1 unfavorable, 5 favorable; confidence H/M/L. Every cell includes a brief reason. B scores a **hypothetical complete architecture**, not an existing feature.

| Criterion | D0 | A | B | C |
|---|---|---|---|---|
| 1. Invariants | 5H unchanged | 5M copy/ref rules retained | 3L broker grant unresolved | 1H violates current grant boundary |
| 2. Protos philosophy | 4H minimal | 5H explicit ordinary composition | 4M Actor messaging | 2H special grant institution |
| 3. Current proportionality | 5H free | 5M opt-in | 3M extra provision mechanism | 2H wide authority cost |
| 4. Incremental growth | 2H still stuck | 5M add brokers later | 4M modular in principle | 3M grant policy hardens |
| 5. Option resilience | 3H not blocking | 5M model-neutral | 4M broker options | 3M ties Host to grants |
| 6. Scalability | 1H service inaccessible | 4M per Worker | 3M broker contention | 5M direct per Worker |
| 7. Simplicity | 5H no new concepts | 4M local factory | 3M two lifecycles | 2H extra policy |
| 8. Portability | 5H unchanged | 5M semantic transfer | 4M message semantics | 2M host capabilities |
| 9. Runtime/resource cost | 5H none | 4M startup copy | 3M messaging overhead | 3M replicated resources |
| 10. Failures | 2H need unresolved | 3M startup/requests separate | 3M broker failure | 3L grant lifetime |
| 11. Deferral/migration | 2H ad hoc workarounds | 4M additive API | 3M later actor change | 2M grant migration cost |
| 12. Evidence | 5H proven status | 4M transfer/factory precedent | 2M no public grant route | 2M unratified authority |

### Counterexamples and future tests

- `Actor.spawn("app:broker","start",network,resolver)` must remain invalid. Internal `TcpAcceptor` Worker delegation cannot authorize arbitrary user initialization.
- Passing `Actor.current()` does not magically install RootActor service behavior.
- `ActorRef` carries `stop()` capability; do not claim request-only attenuation.
- On client abort, cancelling a Worker wait does not undo an already accepted broker or remote HTTP operation. Preserve `RequestOutcomeUncertain` and existing non-retry semantics.
- Do not accept arbitrary handler-supplied URLs/headers as broker authority; deny SSRF/credential escalation by service protocol.
- Multiple Hosts may reference one broker, but Host shutdown must not implicitly stop that broker.
- Hundreds of Workers can saturate a single broker; quotas and broker partitioning are application policy.
- Alternative native, scheduler, multi-Process and distributed backends may optimize representation without changing Actor-level transfer semantics.
- Protos source-mode worker factory must not capture creator Closure identity, share mutable module state or depend on Thread/Context ambient globals.

## Acceptance outline for I133 (one slice)

- Existing `Host.owning` API, module public slots, quotas, D216 failure/close semantics and D215 `handler(request)` remain unchanged.
- New opt-in factory API; one local factory invocation per Worker before READY; one normal handler invocation per admitted request; copied initialization with preserved ActorRef identity; no implicit rights.
- Strict reject recursive forbidden resources, including user-supplied TcpAcceptor, network, resolver, closures, Futures; no listener custody if the public invocation is structurally invalid.
- Startup failures caused by factory import/call/result/late transfer abort with existing cleanup, including partial Worker creation.
- Drain, abort, connection close, cancellation, service reference termination and multi-host sharing under existing semantics.
- Deterministic conformance and integration for JVM/native with no real Internet access, plus guide/spec/changelog.
- Final human-executor validation at checkout HEAD with `git diff --check`, focal tests, `make test` and `make check`, followed by human commit/push and exact SHA.

## Approval provenance and limits

The project owner explicitly said **"Apruebo D225-A"** in the active 2026-10-30 conversation after receiving the full D225 proposal including the remaining broker limitation and the explicit question whether to approve only A. This is exact approval of A, not of B1/B2, C, E or a new RootActor/Network semantic. The single-slice implementation requirement and human-executor boundaries were then explicitly reinforced.

**Publication evidence at this stage:** approval and research only; no I133 implementation tests, product commit or normative publication claimed. See [selected decision record](../../decisions/language/D225_EXPLICIT_HTTP_HOST_WORKER_FACTORY.md).
