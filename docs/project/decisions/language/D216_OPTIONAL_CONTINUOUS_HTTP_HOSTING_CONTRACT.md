# D216 — Optional continuous HTTP/1.1 hosting contract

**Status:** **RATIFIED AND NORMATIVELY PUBLISHED** (2026-10-27).
**Owning decision:** [D216 #964](https://github.com/guillermomolina/protos/issues/964).
**Exact normative publication:** [`guillermomolina/protos@636e545de6f8ae193940d6a2138b08889c91472e`](https://github.com/guillermomolina/protos/commit/636e545de6f8ae193940d6a2138b08889c91472e), specification **0.1.474**.
**Previous TCP dependency:** [D217 #965](https://github.com/guillermomolina/protos/issues/965), specification 0.1.473; [I125 #966](https://github.com/guillermomolina/protos/issues/966) provides the implementation through product 0.3.407.
**Previous single-request HTTP:** [D215 #961](https://github.com/guillermomolina/protos/issues/961) and [LIB025 #962](https://github.com/guillermomolina/protos/issues/962), unchanged.

This is a **non-normative decision record**. Authoritative semantics are in [`spec/io/HTTP.md` §15](https://github.com/guillermomolina/protos/blob/636e545de6f8ae193940d6a2138b08889c91472e/spec/io/HTTP.md), with the revision entry in [`spec/PROTOS_SPEC_CHANGELOG.md`](https://github.com/guillermomolina/protos/blob/636e545de6f8ae193940d6a2138b08889c91472e/spec/PROTOS_SPEC_CHANGELOG.md). This record does not implement the hosting module.

## Owner approval and selected exact contract

On 2026-10-27 Guillermo explicitly approved the **complete D216-1 through D216-12 bundle** following the earlier selection of Option B (an independent, opt-in hosting facility). The publication ratifies the following jointly approved constraints, without reopening prior decisions:

1. **D216-1 — Module/API.** The separate `std:http/Host` module exports `Host` and `HostError`; `Host.owning(listener, handlerModule, handlerBinding, options)` synchronously returns Actor-local `host`; `ready()`, `shutdown("drain")`, `shutdown("abort")`, and `termination()` return observing Futures. No `Server.serve` alias or borrowed-listener host.
2. **D216-2 — Explicit provisioning.** The caller supplies a live `TcpListener`; a Worker resolves its own module and callable binding from String names. `options` admits only `workers`, `maxConnections`, `maxPendingAccepts`, and `limits`. Defaults: 1, 64, workers, and null. Require `1 <= workers <= maxPendingAccepts <= maxConnections`; invalid arguments synchronously fail before resource custody changes.
3. **D216-3 — Listener custody.** The constructing Actor retains listener ownership and close authority; the host takes exclusive logical custody and closes once on all terminal paths. Worker initialization gets only attenuated `TcpAcceptor`.
4. **D216-4 — Worker concurrency.** Exactly `workers` same-Process Actor incarnations independently accept connections and run one unchanged `Server.serveOne.owning` session per connection. Handlers are Actor-local; no cross-Actor TCP handoff or central guest acceptor.
5. **D216-5 — Bounded admission.** Enforce global hard maxima using disjoint, deterministic per-Worker quotas (division plus indexed remainder); pending accepts plus active services cannot exceed the connection cap. No unbounded backlog in guest code, load-balancing guarantee or implicit HTTP 503.
6. **D216-6 — Accepted-resource custody.** The accepting Worker must serve or close each delivered connection exactly once, including shutdown races. Undeliverable backend resources follow the existing TCP/IO cleanup rules.
7. **D216-7 — Readiness/lifecycle.** Explicit STARTING, RUNNING, DRAINING, ABORTING and TERMINATED transitions; readiness only after all Worker bootstraps succeed; termination requires all Worker and resource cleanup.
8. **D216-8 — Drain/abort.** Drain completes accepted sessions; abort requests cooperative cancellation; abort may escalate drain, never the reverse. Shutdown is idempotent; no timeout, hard kill or implicit grace period.
9. **D216-9 — Future observers.** Results resolve with host or fail with hosting-level Error only after required work; cancelling one observer affects neither lifecycle work nor other observers.
10. **D216-10 — Failure isolation.** Ordinary per-client failures remain local to sessions; fatal listener, Worker or cleanup failures trigger host abort and fresh `HostError` with `kind` of `startup`, `listener`, `worker` or `shutdown`.
11. **D216-11 — Scaling boundaries.** C = mono-CPU, A = multiple same-Process Worker Actors across CPUs, P = multiple Processes/nodes deferred. No mandatory accept-serializing guest Actor, placement control or cross-Actor transfer of live resources.
12. **D216-12 — Implementation/exclusions.** Protos-first `protos/lib/http/Host.protos` and bounded internal helpers, reusing existing Core/Network primitives. No new Java HTTP engine or primitive is authorized. HTTP keep-alive, TLS, HTTP/2–3, routing, client, WebSockets, timers and P distribution remain excluded.

## GITHUB021 owner-invariant/delta consistency

**Outcome: PASS against the approved bundle and previously recorded invariants.**

- Preserve **D215/LIB025**: one HTTP/1.1 request per provided connection, one final response, `Connection: close`, unchanged parsing, Error/Task/cancellation and cleanup precedence.
- Preserve **D217/I125**: `TcpAcceptor` carries acceptance-only authority, may be delegated solely at same-Process `Actor.spawn` initialization; live `Network`, `TcpListener`, and `TcpConnection` remain non-transferable.
- Preserve owner C/A/P correction, multi-core independent accepts, no central guest accept hop, no speculative P proxy, and pay-as-you-grow (no costs unless optional hosting is imported/used).
- Preserve explicit ingress trust boundary, no portable timer/deadline guarantee and no claims of fairness, infinite slow-client eviction, or multi-core throughput.
- Newly ratified API, options, quotas, lifecycle, observer behavior, Error categories, and exclusions are exactly the D216-1…12 package approved on 2026-10-27. No unapproved replacement for a prior invariant was introduced.

## Publication, validation and next step

- **Protos revision:** `636e545de6f8ae193940d6a2138b08889c91472e`; published normative changes: `spec/io/HTTP.md` §15, `spec/PROTOS_SPEC_CHANGELOG.md` revision 0.1.474, and the `AGENTS.md` owner map.
- **Human executor:** reported `git diff --check` clean and all local tests PASS before final specification revision publication; `make test` and `make check` reported PASS for the normative candidate. No tests ran after final metadata editing.
- **Not implemented by this ratification:** `std:http/Host` runtime/library. Allocate a separate **Ixxx** implementation Issue with its own independent acceptance criteria and review, leaving D216 itself eligible for closure after durable publication evidence is verified.
- Existing earlier [D216 investigative handoff](../../work/D216/D216_CONTINUOUS_HTTP_HOSTING_AGENT_HUMAN_HANDOFF_2026_10_27.md) remains historical as a pre-approval snapshot; its unresolved wording is superseded by this ratification and the precise normative specification.
