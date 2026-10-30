# D225-A — Explicit HTTP Host Worker factory and transferable initialization

**Decision:** D225-A (partial, approved)
**Owner approval:** 2026-10-30, exact user statement: "Apruebo D225-A".
**Source issue:** [D225 #992](https://github.com/guillermomolina/protos/issues/992)
**Normative authority:** Not this record. I133 published [`spec/io/HTTP.md` §15.13](https://github.com/guillermomolina/protos/blob/e88f4f0e510320ede0ec61bd67dfd671f30fc2cb/spec/io/HTTP.md) and [`spec/PROTOS_SPEC_CHANGELOG.md` revision 0.1.481](https://github.com/guillermomolina/protos/blob/e88f4f0e510320ede0ec61bd67dfd671f30fc2cb/spec/PROTOS_SPEC_CHANGELOG.md) in the exact Protos product commit [`e88f4f0e510320ede0ec61bd67dfd671f30fc2cb`](https://github.com/guillermomolina/protos/commit/e88f4f0e510320ede0ec61bd67dfd671f30fc2cb). The normative specification, not this decision record, owns observable behavior.
**Implementation owner:** I133 (one atomic implementation slice; independently tracked Issue).

## Problem and exact selection

`Host.owning(listener, handlerModule, handlerBinding, options)` spawns isolated Worker Actors, each importing its own handler binding and invoking `handler(request)`. Application-owned transferable state and ActorRefs cannot currently be supplied to those handlers. The owner selected **A only**: an additional opt-in Host operation that transfers permitted initialization values to each Worker and constructs a handler by invoking a Worker-local factory once.

The selected public form is:

```protos
Host.owningWithFactory(listener, factoryModule, factoryBinding, initialization, options) -> host
```

The existing `Host.owning(listener, handlerModule, handlerBinding, options)` is unchanged.

### Selected observable contract

1. Each Worker receives its own Actor-boundary copy of `initialization` (ordinary transferable values, including `ActorRef`, but not live resources, Closures, Futures, Network, Resolver or any `TcpAcceptor` supplied as application initialization).
2. `factoryModule` is imported/instantiated inside each Worker; its own `factoryBinding` is selected locally and invoked **exactly once per Worker** with one argument: that Worker's copied initialization.
3. The factory result must be callable; no implicit Future adoption, awaiting, lazy factory execution, per-request factory invocation or module sharing is introduced.
4. The resulting callable remains the D215 `handler(request)`. No second request argument, request-context registry or global service resolution.
5. Factory resolution, invocation and callable validation occur before **Worker hosting readiness** (entering its HTTP acceptance service), not necessarily before the Actor lifecycle's `INITIALIZING -> READY` cutover; factory failure produces the existing Host startup failure/cleanup behavior. Structural validation of the new public arguments must precede listener custody as D216 requires. Failures arising during Worker transfer/bootstrap remain startup failures with mandatory cleanup. The precise two-boundary distinction is normative in published `HTTP.md` §15.13.
6. Existing D216 ownership, acceptance quotas, `ready`, `drain`, `abort`, termination, mandatory close and error-precedence contracts remain unchanged. Host does not create, supervise, stop or own any application service referenced by initialization.
7. No authority amplification, ambient service lookup, cross-Actor mutable-object identity, implicit creator/parent access, or broader exception to Actor transfer.
8. A service `ActorRef` retains its destination identity when transferred. The full existing `ActorRef` authority (including `stop`) is NOT silently attenuated; applications must assess the risk.
9. No changes to D222 Router/URL-client behavior and no new REST, TLS, DNS or networking semantics.

### Explicitly not selected

- No new host-provisioned `Network`/`Resolver` grants to Workers.
- No ability to `Actor.spawn` with non-transferable `network`/`resolver`.
- No automatic RootActor message-serving behavior, new Actor service publication API, or implicit `ActorRef` lookup.
- No dependency-injection framework, container, service registry or special service identity.
- No global broker lifecycle, retry, cancellation propagation, timeout, distributed broker provisioning, narrowed ActorRef type, or implicit rollback.
- This decision does **not** establish that a GitHub REST broker can already be created with the present public APIs.

## Invariant/delta assessment (GITHUB021)

Preserved: D215 one-argument handler; D216 original Host method and Worker ownership/quotas/cleanup; D217 special `TcpAcceptor` delegation **only** for internal Worker acceptance, not caller-provided initialization; Actor transfer by value with ActorRef reference identity; Actor-local module cache; D223 independent non-transferable Resolver; Network grant isolation; D222 orthogonality.

New, approved delta: one public opt-in operation, explicitly transferable per-Worker initialization, local factory invocation before readiness, failure handling at the existing startup boundary.

**No approved invariant was reopened or replaced.** If implementation discovers an observable choice that conflicts with these constraints, it must stop and return for separate owner approval.

## Why this boundary

Research compared Go net/http handler closures, Rust axum explicit State, Python ASGI lifespan/scope, ASP.NET Core DI, Erlang/OTP processes, Akka Typed ActorRefs and Pony reference capabilities. The minimal Worker-local factory is a small, explicit composition mechanism. A system-wide DI container or transferable networking authority would impose speculative cost and weaken established capability boundaries. A broker may be an appropriate *application* pattern, but the current public model lacks an established way to provision a message-serving Actor that retains the RootActor's live Network/Resolver authority.

**Strongest counterargument:** a factory alone cannot deliver an end-to-end GitHub-backed website. It is therefore approved only as the composition seam, never as proof that application-service provisioning has been solved.

## Publication and closure boundaries

- I133 **published** the selected A contract in one coherent source/spec/test/documentation commit [`e88f4f0e510320ede0ec61bd67dfd671f30fc2cb`](https://github.com/guillermomolina/protos/commit/e88f4f0e510320ede0ec61bd67dfd671f30fc2cb) at `0.3.422-SNAPSHOT` / spec `0.1.481`. The maintainer reports clean `git diff --check`, local `make test` PASS and `make check` PASS. See [I133 completion evidence](../../evidence/I133/I133_0_3_422_HTTP_HOST_WORKER_FACTORY_COMPLETION_2026_10_30.md).
- D225 remains **open for the residual application-service provisioning question**; do not describe A as the full D225 resolution.
- The product publication distinguishes Actor READY from **hosting readiness**: the factory runs after Actor bootstrap, before Host RUNNING and acceptance. `HostWorker` uses an `entered` gate to prevent `host.ready()` racing the acceptance service. This paragraph records the published implementation/spec clarification, not new authorization to provision live service authority.
- The remaining broker/root Actor authority question requires a separate, explicit D-level decision before any implementation; neither this record nor the I133 Issue approves it.

## Evidence and references

- [Decision investigation / comparison / acceptance matrix](../../evidence/D225/D225_A_RESEARCH_AND_APPROVAL.md)
- [D225 #992](https://github.com/guillermomolina/protos/issues/992)
- [D215 #961](https://github.com/guillermomolina/protos/issues/961), [D216 #964](https://github.com/guillermomolina/protos/issues/964), [D223 #987](https://github.com/guillermomolina/protos/issues/987), [D222 #986](https://github.com/guillermomolina/protos/issues/986)
- `guillermomolina/protos:spec/io/{HTTP,NETWORK,PROCESS_IO}.md`, `spec/concurrency/ACTORS.md`, `spec/semantics/MODULES.md`
- `guillermomolina/protos:protos/lib/http/{Host.protos,internal/HostWorker.protos,internal/HostLifecycle.protos}`

## Later D225-B1 owner approval (2026-10-30)

The exclusions above are the exact **A-only scope at the time of D225-A approval**. They are historical and are **not retroactively revised** by the later decision.

The owner has now **separately approved D225-B1**: an explicit, opt-in RootActor initial message-service bootstrap using the existing module/binding and ordinary behavior-at-READY model; application-level `Actor.group(Actor.current())` produces a communication-only `GroupRef` transferable to I133 HTTP Worker factories. It is neither automatic RootActor publication nor transferable Network/Resolver authority. See [B1 selected decision](D225_B1_OPT_IN_ROOT_ACTOR_MESSAGE_SERVICE_BOOTSTRAP.md) and [B1 approval/invariant evidence](../../evidence/D225/D225_B1_OWNER_SELECTION_AND_GITHUB021_RECONCILIATION_2026_10_30.md).

**Publication gate:** The B1 choice is durably recorded in project documentation but has **not** yet been published as normative Protos specification or implemented. D225's formal design closure still awaits that separately human-executed product normative revision; this A record continues to document the already completed I133 boundary.
