# I126 — Optional continuous HTTP/1.1 Host: completion evidence (0.3.409)

- **Recorded:** 2026-10-28.
- **Implementation issue:** [guillermomolina/protos#967](https://github.com/guillermomolina/protos/issues/967).
- **Exact published Protos revision:** [`4f45348c39f218fd18503ef52a40bd20b71b8cf7`](https://github.com/guillermomolina/protos/commit/4f45348c39f218fd18503ef52a40bd20b71b8cf7).
- **Product version at that revision:** `0.3.409-SNAPSHOT` (commit title: `0.3.409`).
- **Normative owner:** `spec/io/HTTP.md` §15, approved D216 specification 0.1.474; this implementation additionally carries specification 0.1.475 editorial cross-reference updates in `NETWORK.md`, `ERRORS.md` and the spec changelog.
- **Publication status:** Product commit pushed to `main`; owner reports local validation complete.

## Completed scope

I126 implemented the separate optional `std:http/Host` Standard Library module exporting `Host` and `HostError`. `Host.owning(listener, handlerModule, handlerBinding, options)` accepts an explicitly owned `TcpListener`; `ready()`, `shutdown("drain")`, `shutdown("abort")`, and `termination()` provide independent observer Futures. Cancelling an observer does not cancel the host lifecycle.

The implementation uses same-Process Worker Actors and `Actor.spawn`-delegated `TcpAcceptor` capabilities; each Worker invokes the existing `Server.serveOne.owning` for a single HTTP/1.1 request per connection. It retains per-Worker fixed quotas, bounded pending acceptance, listener and connection custody, worker-local handler resolution, no centralized accepting Actor, and cooperative draining and aborting. No Java HTTP engine, public primitive, scheduler, timer, socket transfer, keep-alive, TLS, HTTP/2–3, routing or client feature was added.

The final B slice completed the failure and termination cases:

- Fatal accept errors propagate as `listener`; failure of an obligatory connection close or non-admitted connection release propagates as `cleanup` from Worker and `HostError("shutdown")` from Host. A Worker with fatal cleanup immediately stops accepting and cooperatively aborts outstanding work.
- `HostConnection` tracks the unique owning close Future and preserves error identity and cleanup precedence. A session releases its quota exactly once, including error and cancellation exits.
- `HostLifecycle` handles synchronous partial Worker startup, readiness and termination outcomes, single stop/close effects, repeated or escalated shutdown and cancellation of internal observers. Mandatory cleanup failure takes precedence over earlier lifecycle errors.
- D121 runtime authorization was completed for **Bytecode-compiled `ensure`** during Actor termination. Its cancellation-cleanup extent is opened precisely by `BeginEnsureCleanup` and closed by generated finally; the extent depth resides lazily in the Task's `ProtosDynamicControlState`, not as a fixed field on every Task. Close authorization requires the open extent and active cancellation unwind; unrelated Tasks, a mere TERMINATING Actor, a bare UNWINDING phase and superseded cancellations do not gain authority.

## Conformance and validation evidence

The changes add and register guest `host-connection.protos` and `host-lifecycle.protos` tests (in addition to `host-construction.protos`); they exercise deterministic fatal accept, close and cleanup precedence, admission and quota release, partial startup, duplicate stop/close prevention, and shutdown escalation. `ProtosHttpHostIntegrationTest` exercises real NIO, multi-Worker acceptance, concurrent sessions, normal and fatal Worker termination, startup failures, module isolation, observer independence, shutdown modes and EOF following failed Worker termination. `ProtosTerminationCleanupCloseTest` exercises allowed and refused post-cutover NIO closes; structured-`ensure`, cancellation unwind, HTTP Server and TcpAcceptor regressions remain in the integrated suite.

**The owner reported, for the published I126-B candidate:**

| Check | Owner-reported result |
| --- | --- |
| `git diff --check` | PASS |
| `make test` | PASS |
| `make check` | PASS |
| Product `main` commit/push | PASS — `4f45348c39f218fd18503ef52a40bd20b71b8cf7` |

These are **owner-reported local results**, not independently executed by the evidence recorder. Per the repository's publication policy, version updates in `pom.xml` and `CHANGELOG.md` are applied only after all checks pass and immediately before publication; this record makes no claim of tests after those metadata-only updates.

## Scope boundaries and next work

- I126 is an **implemented** slice, not a deferred or partial Host; D216 and the unchanged `Server.serveOne.owning` remain normative.
- Investigation of whether additional Java code can be moved to Protos is subsequent architectural research, not a condition of I126 closure. In particular the Java NIO and runtime cancellation/custody machinery remains the host boundary and must not be counted as guest protocol logic.
- If later independent conformance demonstrates a regression, track and repair it on evidence rather than retroactively claiming it was tested here.
