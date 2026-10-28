# D218 — Explicit-duplex HTTP/1.1 client: comparative research (2026-10-28)

**State:** RESEARCH / PROPOSED CANDIDATE, **NOT OWNER-RATIFIED**.
**Issue:** https://github.com/guillermomolina/protos/issues/969
**Read-only source baseline:** `guillermomolina/protos@b0ae444f30951f8d85a6776bab7c057bed8d0a3d` (2026-10-28; I126-A, specification 0.1.474).
**Durable store:** `guillermomolina/protos-project-docs`. This record is non-normative, cannot substitute for an approved `spec/io/HTTP.md` revision, and claims no test or code execution.
**Work decomposition:** exactly one design investigation and, only after exact owner approval, one consolidated Ixxx implementation slice. No separate DNS, TLS, URL, pooling or redirects work is authorized here.

## 1. Decision and settled scope

The project owner accepted that client and server belong to the same **Standard Library HTTP domain**, while client and server modules remain independent. The immediate use case is one HTTP/1.1 request/response exchange on a duplex supplied explicitly by the application. The library does not acquire `Network`, resolve hostnames, interpret a URL to create a connection, manage certificates/TLS, follow redirects, pool connections, set a timer, or create a new Core or host HTTP engine. An externally supplied authenticated TLS duplex could carry an HTTP exchange, but this is **not** an end-to-end HTTPS URL client.

D215/#961 closed the server side (`Server.serveOne[.owning]`) with strict message framing, binary headers and bodies, one Actor-local service Task, explicit borrowed/owned lifecycle, and `Connection: close`. D216/#964 added optional `std:http/Host`; D217/#965 added same-Process Actor-delegable accept-only capabilities. None authorizes an outbound Client public contract; `HTTP.md` §14 expressly defers it. This is a new observable Standard Library design checkpoint.

## 2. Source audit (exact baseline)

Binding instructions inspected: root `AGENTS.md`, `AGENTS.work/{DESIGN,LIBRARY,IMPLEMENTATION,COORDINATION,REFERENCE}.md`, `spec/AGENTS.md`, `protos/AGENTS.md`, durable docs `AGENTS.md`, `docs/project/README.md`.

Normative owners inspected: `spec/io/{HTTP,NETWORK,IO_CORE,BYTE_IO}.md`, `spec/concurrency/{FUTURES_AND_TASKS,ACTORS}.md`, `spec/semantics/{MODULES,ERRORS}.md`, `spec/PROTOS_SPEC_CHANGELOG.md`. HTTP §§1–14 and §15 were inspected with independent entry-point, parser, lifecycle and error analysis; relevant network, byte-I/O, Future, Actor, module and error sections were cross-checked. `spec/io/NETWORK.md` §1 and §13 still contain generic pre-D216 descriptions; they are not independent authority to forbid the already-ratified Host. The `IMPLEMENTATION_BLOCKERS.md` registry was consulted for applicable normative blockers.

Implementation evidence: `protos/lib/http/Server.protos`; `protos/lib/http/internal/{Session,MessageHead,MessageBody}.protos`; `protos/lib/core/network/TcpConnection.protos`; `protos/lib/core/Future.protos`; `src/main/java/com/guillermomolina/protos/execution/facilities/ProtosHttpAsyncFacility.java`; `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosCoreBootstrap.java`; targeted HTTP test directory. Existing `MessageHead.requestHeadParser` is **server-directional**; `MessageBody.requestBodyFramer` is request-directional. Their octet/token/header validation can be shared where truly symmetric, but a client **must** have its own response parser and framing. The existing `ProtosHttpAsyncFacility` already supplies private `failed`, `resolved`, `provides`, `invokable`, `closable`, `isBytes`, `startService`, `enterService` operations and the standard-module bootstrap seam. The proposed client needs no new Java HTTP primitive.

## 3. Primary-source comparative prior art (5 systems / 3+ approaches)

| System and primary source | Approach | Lesson and limitation for Protos |
| --- | --- | --- |
| [Rust Hyper `client::conn::http1::handshake`](https://docs.rs/hyper/latest/hyper/client/conn/http1/fn.handshake.html) | Explicit asynchronous I/O passed to the HTTP connection driver, separate connector/transport | Demonstrates legitimate transport-independent HTTP protocol and a distinct connection-driving lifecycle; full Hyper connection machinery is unnecessary for one Protos exchange. |
| [Go `net/http` `RoundTripper` and `Client`](https://pkg.go.dev/net/http) | One transaction interface beneath high-level client, with streaming body ownership and transport pooling | A 4xx/5xx status is a valid received response, not a transport error. Connection pooling/redirect/cookie policy belongs above the one-exchange layer. The response body must have explicit consumption/closure ownership. |
| [Node.js `http.request`](https://nodejs.org/api/http.html) | Evented writable request plus readable response; configurable Agent/connection source | Demonstrates streaming and informational-response support. Event emitter, Agent, default DNS, timeout and socket lifecycle would introduce a second concurrency model into Protos. |
| [Python `http.client`](https://docs.python.org/3/library/http.client.html) | Stateful low-level `HTTPConnection`/`HTTPSConnection` and `HTTPResponse` | Shows separate TLS configuration atop HTTP and stream-vs-buffered body handling; its host-owning connection object and synchronous/blocking API are not the explicit capability/Actor model. |
| [Java SE 25 `java.net.http.HttpClient`](https://docs.oracle.com/en/java/javase/25/docs/api/java.net.http/java/net/http/HttpClient.html) | High-level client with optional async send, body handling, redirects, TLS and pooled reusable transport | Useful later facade but would front-load DNS/TLS/proxy/pool state and a JVM dependency; not the smallest portable Protos Standard Library protocol primitive. |

Wire authority: [RFC 9110 HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) (request targets, status semantics, Expect, 100), [RFC 9112 HTTP/1.1](https://www.rfc-editor.org/rfc/rfc9112.html) (§§2, 6–9: strict message boundaries; HEAD/1xx/204/304; conflicting TE/CL; chunked; close-delimited responses; incomplete/truncated response and connection close). These RFCs allow behaviors broader than a first portable strict Protos client; the chosen Protos rules must state any deliberate stricter subset.

Three materially distinct architectural families are represented: explicit transport/protocol drivers (Hyper), transaction/response body interfaces (Go/Python), evented stream client (Node), and high-level managed-client facade (Java).

## 4. Meaningful candidates and falsification

- **A — bounded eager response:** `Client.exchangeOne(duplex, request, limits) -> Future<Response>`, whole body eagerly collected into `Bytes` under a finite cap, optional owning variant. Small public surface, simple completion, but O(response size) memory and mandatory whole-response buffering; no incremental consumer backpressure. **Survives as a genuine minimal candidate** for small JSON/API use.
- **B — scoped response consumer (recommended):** `Client.exchangeOne(duplex, request, onResponse, limits) -> Future<result>`, callback runs in the one exchange Task, receives bounded `ByteReadable` response body, and cannot use it after exchange retirement; optional owning variant. Incremental bodies and deterministic scope/custody without a public session state machine. **Survives.**
- **C — public response session handle:** return response headers and a `ByteReadable` after the Future resolves, plus `close/complete` and ownership state. Realistic for independent consumers, but requires another public lifecycle protocol, abandoned-body policy and cancellation/Actor scope semantics now. **Eliminated as premature**; can be added as an independent later module/operation.
- **D — URL-first HTTP/HTTPS Client:** `request(url)` with DNS, TLS/certificates, deadlines, redirects, pools and defaults. Meets a different high-level need but violates missing portable authority contracts and pays immediately for future facilities. **Eliminated; later facade remains possible.**
- **E — fold client into `std:http/Server`:** shares one import but couples client to listener/handler machinery and changes D215's deliberately exact module. **Eliminated**; same library does not mean same module.
- **F — defer all client work:** preserves simplicity but leaves the explicitly approved operational use case unmet. **Eliminated** for the present work.

### GITHUB010 common scorecard: 1 (poor) to 5 (strong)

Confidence H=high, M=medium, L=low; each cell is `score/confidence — rationale`.

| Dimension | A: eager | B: scoped consumer |
| --- | --- | --- |
| 1. Correctness/invariants | 4/H — simple terminal cleanup; less body exposure | 4/M — session-bounded reader is sound if pre-start cancellation and retirement are exact |
| 2. Protos alignment | 4/H — ordinary data and Future, but forced materialization | 5/H — ordinary callable, ByteReadable, one Task and explicit capability |
| 3. Present-need proportionality | 5/H — simplest for small payloads | 4/M — modest callback/lifetime machinery earns streaming |
| 4. Incremental growth | 3/M — changing eager result to streaming would break consumers | 5/M — later request streaming and high-level facade add without changing B |
| 5. Future-option resilience | 3/M — fixed Bytes return constrains large result processing | 5/M — preserves transport and response-body stream options |
| 6. Scalability | 2/H — per-request memory tracks complete body size | 5/M — per-exchange memory bounded by head/framer chunks, not total body |
| 7. Conceptual simplicity | 5/H — direct Future<Response> | 4/M — callback and scoped body lifetime must be learned |
| 8. Portability/implementation freedom | 5/H — pure byte I/O; no host engine | 5/H — pure byte I/O and existing Actor/Task ownership |
| 9. Runtime/resource cost | 2/H — every body must be copied/materialized | 5/M — streamed chunks and backpressure; only used exchanges pay |
| 10. Failure/operability | 4/H — one terminal result; response-limit failure explicit | 4/M — cleanup and late body errors precise, but callback scope is subtler |
| 11. Deferral/migration cost | 2/M — later streaming needs new breaking result protocol | 5/M — later capability requires additive operations, not model replacement |
| 12. Evidence maturity/risk | 5/H — conventional eager result, less state | 4/M — HTTP streaming prior art strong, Protos private framer needs work |

No arithmetic total selects the design. A carries an **underengineering** red flag for unbounded real response use, even when a finite cap prevents memory exhaustion. B's extra callback/lifetime surface is the price of incremental consumption; it has no speculative pool, URL resolver, TLS engine or cross-Actor machinery. B's strongest objection is that callbacks are less convenient than a direct `Future<Response>` and the body cannot escape its scope.

### Pay/grow/defer and future stress

Smallest adequate **general** one-exchange primitive with bounded memory is B. Client code is lazy-imported, Actor-local and uses an already present Task bridge; no cost for programs importing only Server/Host. A is smaller for small fixed responses, but a future package downloader or large body forces a new streaming API; preserving a stream boundary now is justified by standard HTTP content semantics, not speculative HTTP/2.

Adding DNS/TLS/URL later requires a separate capability-aware acquisition facade that passes a ready duplex to `Client.exchangeOne`; it does not change this operation's ownership or parser. Adding chunked request uploads can extend the request-body admissible family or add a new operation without changing the response callback's lifetime. Keep-alive/pooling, by contrast, needs a **new** multi-exchange session/retention contract because this B always sends close and cannot return read-ahead; do not silently reuse it. Cross-Actor fan-out means one independent duplex in each owning Actor, not transferring a live connection. Portability across NIO, native images, different OS backends, other schedulers and non-Truffle engines survives because no engine-specific HTTP primitive is normative.

Stress counterexamples:
1. A large response must not be collected whole for B; incremental `read(maxBytes)` applies backpressure.
2. A malformed or premature-EOF chunked response must fail the logical body read (and exchange Future) and retire the session, not silently report EOF.
3. Repeated or contradictory `Content-Length`/`Transfer-Encoding` must never let two independent parsers disagree about the boundary.
4. A valid 404/500 **response** invokes the callback and is not a `ClientError`; a malformed status line is a protocol failure.
5. An unsolicited 100 or 103 must not be mistaken for final; 101 must not turn into silent protocol upgrade.
6. A callback that catches a late framer Error cannot legitimize a partial body or turn a terminal exchange into successful completion.
7. Cancellation just after owning admission, even before Task start, must close the duplex before the Future becomes terminal; borrowed exchange never closes it.
8. A non-idempotent POST partially written before failure is not automatically retried, even if no response arrived.
9. A server that sends an early final response without reading a large request body may cause blocked output; this first sequential-write client has no portable deadline and **does not promise early-response concurrency**. This is a bounded-scope usability limitation, not proof of delivery or safe retry. Do not claim hostile-peer production safety.
10. A close-delimited response can hang until a clean EOF; a TLS adapter must itself define authenticated truncation/EOF semantics. The HTTP module cannot manufacture TLS guarantees.
11. Multiple Actors may each run exchanges on their own duplex without sharing mutable modules, Parser state or socket identities.
12. A future high-level URL client must not make `http:` or `https:` an implicit authority grant to this low-level module.

A plausible regret: applications soon need early-final monitoring during upload, full streaming request bodies or keep-alive. Escape path: a separately approved concurrent-upload mode or stateful session above a reusable response framer, without changing the semantics of B's single-exchange operation. Prebuilding those today would add scheduling, cancellation and retention burdens without a present normative transport-deadline foundation.

## 5. Exact proposed candidate B contract — PENDING GUILLERMO'S APPROVAL

**Module:** `std:http/Client`, exporting `Client` and a new `ClientError` whose immediate parent is `Error`; no Prelude binding. This is a sibling of `std:http/Server`, not an import of it, and does not change `Server.HttpError` identity.

```text
Client.exchangeOne(duplex, request, onResponse, limits)        -> Future<result>
Client.exchangeOne.owning(duplex, request, onResponse, limits) -> Future<result>
```

1. Ordinary dispatch/arity/receiver rules first. After successful dispatch, invalid arguments return a fresh **failed Future** containing a fresh `InvalidIOArgument`, with **no Task, byte I/O, HTTP session, or own/close effect**. A valid invocation captures request fields and limits, admits exactly one Actor-local cooperative exchange Task under the invoker's existing structured scope (or Actor root asynchronous scope), obtains owning custody if selected, and returns that Task's Future. The own responsibility precedes exposure to cancellation.
2. `duplex` satisfies `ByteReadable` and `ByteWritable`, plus `Closable` for owning. It is explicitly provided: no `TcpConnection` type identity, Network, DNS, TLS, or URL processing. `onResponse` is an ordinary callable invoked **at most once** on the first valid final response, within the exchange Task. Its ordinary result becomes the success result of `Future<result>` under existing Future resolution/adoption rules, **after** response retirement and mandatory owning cleanup. Returned pending Futures do not extend body lifetime.
3. `request` is a structural object with own `method: String` (valid HTTP token), `target: Bytes` (valid origin-form target, or `OPTIONS *`), `headers: Array` of ordered `[String, Bytes]` pairs, and `body: null|Bytes`. Only HTTP/1.1 origin exchanges; no CONNECT, Upgrade, absolute/proxy targets, Expect/100-continue waiting, or streaming upload in this first mode. Exactly one syntactically valid `Host` header is supplied **explicitly** in the header pairs. `Content-Length`, `Transfer-Encoding`, `Connection`, `Expect`, `Upgrade`, `Trailer`, `TE`, `Keep-Alive`, `Proxy-Connection` are driver-owned/unsupported request header names and caller occurrences are rejected. Unknown other valid header names are preserved. The driver sends `HTTP/1.1`, `Connection: close`, and `Content-Length` equal to the snapshotted body length (zero for null); it never compresses, encodes text, follows redirects or retries. All mutable request parts, header pairs and body bytes are logically snapshotted before the first output effect.
4. The response exposes a fresh ordinary structural object `{version: "1.1", status: Integer, headers: Array<[String,Bytes]>, body: ByteReadable}`, with original header order, duplicates/casing, trimmed outer OWS, binary values, and a response-message-bounded body. Statuses 200–599 are normal received responses **regardless of 4xx/5xx**. Interim 100–199 responses (except 101) are validated and consumed without callback; at most eight informational heads by default. 101 and unsupported protocol switches fail with `ClientError` `unsupportedFeature`; no upgraded socket is exposed. A final response is required; EOF before one is `malformedResponse`.
5. The strict HTTP/1.1 response framer handles: no-content `HEAD`/1xx/204/205/304, exactly one valid `Content-Length`, exactly `Transfer-Encoding: chunked`, and otherwise response content delimited by duplex EOF. Both TE+CL and repeated/comma-separated CL are rejected; other transfer codings are unsupported. Chunk framing/trailers are validated and discarded. Premature EOF, malformed chunks and over-limit content fail the logical body read with a fresh `ClientError` and terminalize the exchange; independent later body reads fail with fresh `IOLifecycleError`. Private read-ahead must not advance the logical error point. No response body may be interpreted as a second message.
6. `onResponse(response)` runs during the one Task and may incrementally `response.body.read(maxBytes).value()` with byte-I/O backpressure. It is not required to drain. After it returns, fails, or is cancelled, the response body is retired; all escaped reads then fail with `IOLifecycleError`. The Client does **not** drain unread response content, restore read-ahead, nor reuse the connection. The borrowed variant never calls `close` or half-close; the caller retains close responsibility. The owning variant calls `close()` exactly once on **all** terminal paths (including pre-start cancellation and failure) and its Future does not settle before close and Task cleanup settle. An owned body source on the request is not implicitly closed.
7. Limits: `null` or an ordinary own-slot options object; default `maxStatusLineBytes=8192`, `maxHeaderBytes=32768`, `maxHeaderFields=100`, `maxChunkLineBytes=8192`, `maxTrailerBytes=8192`, `maxTrailerFields=100`, `maxResponseBodyBytes=33554432`, `maxInformationalResponses=8`, `maxPrefetchBytes=65536`, `ioChunkBytes=16384`. All supplied limits are positive Integers representable in 32-bit signed range; unknown own slots invalid, inherited slots ignored, no implicit clamping. Byte and count limits are independent; driver buffers bounded by the configured head/prefetch/chunk limits, regardless of total response size.
8. `ClientError` is a fresh occurrence with only portable `kind` values `"malformedResponse"`, `"responseLimit"`, and `"unsupportedFeature"`; no HTTP `status` field is inferred from error responses. I/O errors, `InvalidIOArgument`, `IOLifecycleError`, `Cancelled` and callback errors preserve their own categories/identities. Late response-body framer failures terminalize the exchange even if handled by the callback, and the exchange Future fails with the original framer error unless a mandatory owning `close` or structured cleanup Error supersedes it under the general `ensure` precedence. No automatic replay, ambiguous-commit retry, synthetic success, or HTTP error response is generated by the Client.
9. Request-head/body writes complete before response parsing begins. This mode promises **no concurrent response monitoring during upload**, no output deadlines, and no hostile-peer liveness bound. TCP contribution does not mean remote receipt or application processing; the returned `Future` indicates only the specified exchange and cleanup outcome. Externally supplied secure duplexes must carry their own TLS certification and secure EOF guarantees.
10. Tests must cover socket-free scripted duplexes and real TCP/Actor paths, framing, 1xx/101, HEAD/no-body, chunks, close-delimited, trailers, duplicate headers, invalid arguments, cancellation and late framer Errors, application 4xx/5xx, cleanup precedence, and no transport authority leakage.

This is one **exact candidate to approve or reject**; not normative merely because the research was published. The owner may approve B as written, select A, or request a bounded amendment. A different result requires another explicit approval checkpoint.

## 6. Implementation/publication consequences if B approved

Single consolidated **Ixxx implementation slice in `guillermomolina/protos`**, initially at its actual current HEAD, with no web or other-repository access needed. Extend `spec/io/HTTP.md` with a separate Client section and adjust only genuinely affected cross-document taxonomy/references; assign the *moving* `spec/PROTOS_SPEC_CHANGELOG.md` revision immediately before the final product commit. Add `protos/lib/http/Client.protos` and only the cohesive `internal/Client*.protos` modules needed. Reuse shared protocol validation helpers when safe; do not contort the server-directional parser into a response parser. Extend the **existing** private HTTP async facility's standard-module key map in `ProtosHttpAsyncFacility.java` and `ProtosCoreBootstrap.java` only if the new Client module requires it. This is a local Truffle bootstrap precedent; no external GraalJS/TruffleRuby architecture research is needed or authorized in the implementation slice.

Add cohesive guest HTTP tests and Java/NIO integration tests. Human-executor, not agent, runs focused tests, one final `make test` and `make check` after all code/spec/test work; only then edit `pom.xml` and root `CHANGELOG.md` once and immediately commit/push, **no tests after these metadata edits**. Publication must use the real concurrent HEAD. Update live Ixxx Issue and revision-bound evidence after the product commit. `AGENTS.md` human-executor and no-subrepository-lookup rules continue to apply. The user has explicitly authorized agent-direct commit/push only for `guillermomolina/protos-project-docs`.

## 7. GITHUB021 invariant/delta check and decision gate

| Existing invariant | Candidate B |
| --- | --- |
| D215 Server one service Task, strict own/borrow and `HttpError` | PRESERVED: separate module and separate client error; no Server API modification. |
| D216 Host lifecycle, Worker quotas and owning Server sessions | PRESERVED: no hosting/accept changes; I126 can progress independently. |
| D217 `TcpAcceptor` same-Process exception and non-transferable `TcpConnection` | PRESERVED: connection stays in the invoking Actor. |
| Core `Network` explicit authority and numeric-only `connectTcp` | PRESERVED: client receives duplex only; DNS/TLS/URL explicitly out. |
| Core Byte I/O, cancellation, Task scopes, cleanup precedence | PRESERVED: no new core mechanism; HTTP-specific framing and callback scope are new proposed semantics. |
| Pay-for-use / portable backends | PRESERVED: opt-in module, no global pool or Java HTTP engine; pure protocol on duplex. |
| Owner-approved first-client scope direction | PRESERVED: one exchange, same library, separate module, no URL-first HTTPS claim. |

**New observable decisions requiring approval:** exact Client module/API, Request/Response shapes, callback-lifetime and result, limit values, allowed request body family, strict response-framing subset, informational statuses, error categories, close/retirement semantics and sequential-write liveness tradeoff. Prior user approval of the *plan* covered scope direction, not all these detailed public semantics.

**Decision request:** `D218: approve Candidate B exactly as section 5 (and authorize its ratification + the single implementation slice), or request specified changes.`

**Verification boundary:** No commands, tests, builds, benchmarks, local git operations, normative changes or runtime changes were executed in this investigation. Source conclusions are based on public/repository read APIs and primary protocol/runtime documentation. A documentation-only `git diff --check` was not run because this research explicitly forbids command execution; no PASS is claimed.
