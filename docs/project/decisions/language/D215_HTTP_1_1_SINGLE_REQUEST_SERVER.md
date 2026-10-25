# D215 — Standard Library HTTP/1.1 single-request server

**Status:** OWNER APPROVED / NORMATIVELY PUBLISHED on 2026-10-25.
**Formal decision:** [D215/#961](https://github.com/guillermomolina/protos/issues/961).
**Exact normative publication:** [`9dc4e50875252508e2afecf91541fb50f756d450`](https://github.com/guillermomolina/protos/commit/9dc4e50875252508e2afecf91541fb50f756d450), global Protos specification **`0.1.471`**.
**Publication evidence:** [D215 normative ratification checkpoint](../../evidence/D215/D215_NORMATIVE_PUBLICATION_2026_10_25.md).

This is a durable **non-normative** decision and rationale record. Observable semantics are defined exclusively in [`spec/io/HTTP.md`](https://github.com/guillermomolina/protos/blob/9dc4e50875252508e2afecf91541fb50f756d450/spec/io/HTTP.md) and its referenced owning specifications at the exact publication revision. This record neither grants implementation authority nor claims an HTTP runtime exists.

## Problem and selection

Protos needs a composable HTTP foundation suitable for an application-supplied web server today and possibly later package-tool HTTP use. The existing Core `Network` / TCP and Byte I/O facilities already provide explicit resource authority, binary streaming and Futures. D215 decides the smallest HTTP protocol/application layer that can be built on them without hiding network capabilities, adding privileged Core entities, or making premature listener, TLS, timer or multi-Actor policies.

The owner selected **Option A** (a portable Protos Standard Library HTTP layer), **A3-minimal** (ordinary public message values and private per-service session machinery), and **Q0** (connection work stays in its owning Actor with cooperative Tasks; no automatic socket transfer). P6's detailed F3–F8 contract was explicitly approved in [P7](https://github.com/guillermomolina/protos/issues/961#issuecomment-6385790888), following the [full comparative P6 packet](https://github.com/guillermomolina/protos/issues/961#issuecomment-6385736153). The approved labels are **F3=M1 / F4=H1 / F5=B1 / F6=L1 / F7=W1 / F8=S1**.

The owner additionally ratified **F9 A–F**, recorded in [P9](https://github.com/guillermomolina/protos/issues/961#issuecomment-6388054314), explicitly replacing P6's initial synchronous-argument-validation rule with a failed-Future outcome after successful dispatch. The subsequent F10 A–C and F11 A–C refinements were expressly approved in the normative-editing conversation before publication and are now fixed by the published specification. The older research packets and the dated [F9 editorial handoff](../../work/D215/D215_HTTP_APPROVAL_AND_NORMATIVE_EDITOR_HANDOFF_2026_10_24.md) remain historical, not competing current contracts.

## Selected public boundary

- **Module and admission:** `std:http/Server` exports `serveOne` (borrowed duplex) and `serveOne.owning` (explicitly owning duplex). Each successfully admitted operation creates **exactly one cooperative service Task**, in the same Actor, and returns its associated `Future<null>`. Failed initial argument/limits validation returns a failed Future with fresh `InvalidIOArgument`, without creating the Task or HTTP session, performing I/O, or acquiring ownership.
- **Public values:** Request and Response are ordinary structural objects, not new Core families or grammar. Ordered binary header occurrences, binary request targets and message-bounded `ByteReadable` request bodies retain their specified shapes; response bodies are `null`, `Bytes`, or `ByteReadable`. No text coercion or implicit Future-awaiting handler result.
- **HTTP/1.1 wire and errors:** strict CRLF, Host, header/framing, quota and response validation; supported 100 Continue; appropriate 400/413/414/417/431/501/505 best-effort rejection routes; HTTP/2 preface/version rejection takes 505 precedence where specified, Upgrade/h2c is 501. `HttpError` is a module-local Standard Library error directly under `Error`, not a new Prelude/Core binding. A late body-read framer error fails that logical read with its original `HttpError`, terminalizes the session and produces fresh `IOLifecycleError` on later reads, while hidden read-ahead does not advance the logical failure point.
- **Response lifecycle:** one request and no more than one final response; `Connection: close`, no keep-alive or pipelining. Streaming responses use bounded byte writes. HEAD and 204/205/304 apply exact no-content framing rules. `invalidResponse` has no status. HTTP-owned interim 100, chunk processing, I/O and cleanup create no extra semantic Tasks.
- **Custody and cancellation:** default service never closes a borrowed duplex. The owning variant establishes custody during admission before the Future becomes cancellable, and closes its duplex even on pre-start cancellation, honoring I/O lifecycle cleanup precedence. Cancellation cannot close a borrowed duplex and does not confer a second owner. Task children are cancelled/drained on abnormal owner exit before its Future terminalizes.
- **Actor-incarnation root scope:** legitimate Actor-local execution without a current Task owns new **Task-backed** asynchronous work through the Actor incarnation's non-detached root asynchronous scope. That scope is **not a Task**; synchronous host entry creates no implicit RootTask/ProtosTask. `Future.then()` creates one continuation Task in the applicable Task scope or root scope. A Future with no producing Task follows its producer's own contract. Suspended RootActor entry preserves Actor exclusivity; no implicit reentry or forward-progress guarantee is invented.
- **Security/authority:** no implicit Network acquisition, listener, socket acceptance, DNS, TLS, wall clock, timer, admission supervisor or timeout policy. Explicit byte/quota limits do not protect against silent/slow untrusted peers; first deployment is limited to trusted/protected ingress.

## GITHUB021 preservation and approved deltas

| Prior decision or invariant | Ratified disposition |
| --- | --- |
| D173 explicit Core TCP authority, nontransferable live resources | **Preserved.** HTTP operates over caller-supplied Duplex and adds no Network grant. |
| Existing IO Core / Byte I/O Future, resource and commitment rules | **Preserved**, with a separate limited HTTP message/session contract. |
| Option A, A3-minimal, Q0, F3–F8 | **Preserved** as approved; no silent host HTTP engine, adapter SPI or cross-Actor transport. |
| P6 synchronous invalid initial argument rejection | **Explicitly superseded by owner-approved F9-A**: failed Future after ordinary successful dispatch. |
| F9 body-failure, no-content, error-shape, HEAD, 205 and Expect rules | **Preserved/refined** through F10; no fallback reinterpretation of a caught terminal framer error. |
| Existing structured Task / Future ownership and RootActor exclusivity | **Clarified through F10-C/F11**: one service Task, optional Actor-incarnation root scope, no semantic implicit Task, no forced progress during suspended RootActor host entry. |
| HTTP client, keep-alive, production-grade untrusted ingress | **Deferred**; not accidentally provided by this first server specification. |

## Alternatives and proportionality

The approved [P6 evaluation](https://github.com/guillermomolina/protos/issues/961#issuecomment-6385736153) compared Go `net/http`, Node HTTP streams, Rust Hyper, Python `asyncio`/ASGI, Java HTTP hosting, .NET Kestrel, Elixir Plug/Cowboy/Bandit, and Actor-oriented Erlang/Ranch, Akka and Pony, against RFC 9110/9112. An integrated listener/supervisor would decide admission and shutdown too early; a host-provided engine would introduce a privileged host/guest bridge; an interchangeable engine SPI lacks a second justified implementation; adding HTTP to Core Network would merge transport authority with application protocol; a package-only native HTTP facility would not serve the general-language use case.

The selected `serveOne` layer pays for HTTP only when explicitly imported/used and leaves listener orchestration, robust deadlines, a client, TLS/certificate validation, connection reuse, routing/authentication and package policy to independently ratified growth. It does not claim a high-throughput or publicly Internet-safe server.

## Implementation and coordination boundary

This D215 publication **specifies behavior only**. There are no Java implementation, tests, Maven version, or runtime release changes attributable to it. A later implementation requires a separately scoped `LIBxxx` or other appropriate formal Issue, with runtime code and independently reported validation. Closing D215 records completion of the language decision and durable ratification; it does not certify HTTP execution.

**PROTOS_REVISION=`9dc4e50875252508e2afecf91541fb50f756d450`**
**SPEC_REVISION=`0.1.471`**
**Normative authority:** [`spec/io/HTTP.md`](https://github.com/guillermomolina/protos/blob/9dc4e50875252508e2afecf91541fb50f756d450/spec/io/HTTP.md), [`FUTURES_AND_TASKS.md`](https://github.com/guillermomolina/protos/blob/9dc4e50875252508e2afecf91541fb50f756d450/spec/concurrency/FUTURES_AND_TASKS.md), [`ACTORS.md`](https://github.com/guillermomolina/protos/blob/9dc4e50875252508e2afecf91541fb50f756d450/spec/concurrency/ACTORS.md), and the linked [global specification changelog](https://github.com/guillermomolina/protos/blob/9dc4e50875252508e2afecf91541fb50f756d450/spec/PROTOS_SPEC_CHANGELOG.md).
