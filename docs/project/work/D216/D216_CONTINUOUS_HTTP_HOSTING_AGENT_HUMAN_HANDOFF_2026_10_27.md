# D216 — continuous HTTP hosting: one decision handoff (agent / human executor)

**Date:** 2026-10-27. **Status:** research handoff only; unresolved public design, no implementation permission.
**Owning design:** [D216 #964](https://github.com/guillermomolina/protos/issues/964).
**Published dependency:** [D217 #965](https://github.com/guillermomolina/protos/issues/965) spec `0.1.473`; [I125 #966](https://github.com/guillermomolina/protos/issues/966) implementation A/B/C through [0.3.407 / `ecbcd4c1`](https://github.com/guillermomolina/protos/commit/ecbcd4c19acf9f2561ed03ff7a15351b175f7407), with explicit final integrated-gate reporting being reconciled.
**Existing HTTP primitive:** closed [D215 #961](https://github.com/guillermomolina/protos/issues/961) and [LIB025 #962](https://github.com/guillermomolina/protos/issues/962), public `std:http/Server.serveOne` and `serveOne.owning`.
**Existing design research:** [D216/D217 M3-local evidence](../../evidence/D216/D216_TCP_ACTOR_ACCEPTOR_M3_LOCAL_RESEARCH_2026_10_25.md).

## Editing agent — next work: D216 exact-contract decision packet

**Class:** INVESTIGATION. Read-only. **Not** source, normative specification, issue or repository publication; **no shell commands, builds, tests, validator/scripts, program execution, benchmarks, Git mutations or code edits**. The final output is a Spanish owner-decision packet, not a change set.

**Read first from current `guillermomolina/protos` HEAD:** `AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, `AGENTS.work/COORDINATION.md`, `spec/AGENTS.md`, `spec/io/HTTP.md`, `spec/io/NETWORK.md`, `spec/io/IO_CORE.md`, `spec/io/BYTE_IO.md`, `spec/concurrency/FUTURES_AND_TASKS.md`, `spec/concurrency/ACTORS.md`, `spec/io/PROCESS_IO.md`, and directly relevant transfer/module/error owners. Read D216 #964 and the existing evidence, and verify current `protos/lib/http/{Server.protos,internal/Session.protos,internal/MessageBody.protos,internal/MessageHead.protos}`, `protos/lib/core/network/{TcpAcceptor,TcpListener,TcpConnection,Network}.protos` and representative I125 integration coverage as feasibility evidence. Do not assign normative authority to implementation details.

**Already selected invariants (do not re-decide):**

- User approved D216 **Option B**, a reusable opt-in `std:http` continuous hosting facility; no precise public method signature or lifecycle is yet approved.
- D215 `serveOne.owning` is a one-request service, emits `Connection: close`, uses one service Task and existing borrowed/owning semantics. No keep-alive or hidden Network authority.
- D217 delegated `TcpAcceptor` is attenuated, same-Process, only via explicit `Actor.spawn` initialization; each Worker independently accepts and owns its `TcpConnection`. Live sockets/listeners and P/cross-Process transfers remain forbidden.
- Owner scaling shorthand is **C = mono CPU, A = multi CPU, P = multi nodo**. Present work must genuinely allow multicore A without guest-Actor proxy bottlenecks; P is deferred with no speculative implementation.
- Engineering/runtime **pay as you grow**: the HTTP library's orchestration, policies and algorithms should be written in **Protos**; Java remains only for indispensable host/native/Truffle mechanisms, with any proposed Java additions explicitly justified individually.
- Trusted/protected-ingress is the existing D215 security scope. Do not claim an Internet-hardened, multi-request-per-connection, TLS, DNS, routing, portable deadline, HTTP client or multi-node server.

**Unresolved exact choices requiring owner approval in one packet:**

1. Public optional module/method shape: minimal `std:http/Server` addition versus separate `std:http` host helper; constructor/arguments and HTTP handler entry point; no automatic Network acquisition.
2. Caller-supplied listener versus authority-taking acquisition: who owns and closes the listener; worker acceptance authority; borrowed/owned hosting modes, if needed.
3. Number/placement of Actor Workers, connection-admission caps and queued/busy behavior without adding generic Process affinity or a new scheduler; whether the cap includes Tasks and pending accepts.
4. Host lifecycle and returned value/Future: startup readiness, errors, shutdown request, graceful drain vs immediate cancellation, already accepted but undelivered connections, worker termination, listener/backend failure, and close once semantics.
5. Structured Tasks for each connection vs Actor worker acceptance loop, fair bounded competition, independence of handler errors, backpressure, byte limits and slow-client risk without deadlines.
6. Whether ordinary public Protos mechanisms actually suffice to implement every proposed operation; list any precise missing primitive as a **blocker**, without silently creating Java infrastructure.

**Research threshold:** reuse existing D216/D217 primary-source comparative evidence (Erlang OTP `gen_tcp`, Ranch, Netty, Go `net/http`, Boost.Asio); add only evidence required to compare distinct continuous hosting policies (e.g. Node HTTP, Python asyncio/ASGI, Rust Hyper/Tower), using primary specs/docs and concrete semantics rather than marketing. Preserve GITHUB010's 12 criteria, 1–5 scores with per-cell rationale/confidence, minimum five systems/three approaches, counterexamples, future scaling/migration, smallest-sufficient pay-as-you-grow and explicit GITHUB021 invariant/delta audit. Distinguish previously selected principles from the unselected details.

**Output contract:** one recommended *exact* API and lifecycle proposal with pseudocode in Protos, alternatives and rejected cases, testable acceptance criteria, proposed owning spec clauses, one consolidated A/B/... owner-approval checklist, implementation file-boundary projection (Protos vs justified Java) and explicit exclusions. Do **not** write code, mark D216 approved, ratify specs, create an Ixxx implementation Issue or amend GitHub.

## Human executor — after the agent returns

1. **Execute nothing for the investigation.** Read the comparative packet and approve/reject the exact numbered choices. Do not treat Option B approval as approval of defaults.
2. After explicit owner approval, hand off one **normative-editing-only** ratification in `guillermomolina/protos`: approved clauses of `spec/io/HTTP.md`, minimum required cross-references and spec changelog, with no runtime implementation. Human handles appropriate static/spec gates and publication.
3. Only after normative publication, allocate/reconcile the independent implementation Ixxx under AGENTS coordination and provide **one consolidated Protos-first implementation** prompt: optional listener hosting, worker admission, lifecycle, guest tests plus narrowly necessary native integration. Split only if a true independently valid boundary emerges; never one slice per tiny file.
4. For later **implementation** validation, human runs cheapest gates, all impacted focal tests, required `make test` and `make check`. Update `pom.xml` and `CHANGELOG.md` **only after tests are green and immediately before product commit/push; no tests after metadata changes**. Product repo publication stays human-executed.
5. Live GitHub coordination and exact-revision evidence come *after* the respective ratified/implemented product publication. Never claim a green command from an unspecific summary, invent a version or silently close D216 before the user's exact contract approval.
