# D215 — Approved HTTP contract and normative editing handoff

**Date:** 2026-10-24. **Work item:** [D215 #961](https://github.com/guillermomolina/protos/issues/961).
**Class:** owner-approved design **through D215-F9 A–F**, **normative specification ratification pending**. This is non-normative coordination evidence, not a product implementation or a substitute for `spec/`.

## Provenance and exact state

- Architectural Option A (a general Protos Standard Library HTTP layer over existing explicit TCP/Byte I/O/Futures), A3-minimal (distinct public messages versus private session machinery), and Q0 (one connection-owning Actor with cooperating Tasks, no implicit socket transfer) were approved in the parent design workflow.
- The [exhaustive P6 comparison and F3–F8 candidate](https://github.com/guillermomolina/protos/issues/961#issuecomment-6385736153), including Erlang/Ranch/Cowboy, Elixir/Bandit, Akka, Pony, Go, Rust, ASGI, Node and Kestrel, was approved **as a whole** by the owner; [P7 records the explicit owner approval](https://github.com/guillermomolina/protos/issues/961#issuecomment-6385790888). Choices: **F3=M1, F4=H1, F5=B1, F6=L1, F7=W1, F8=S1**.
- When preparing this record, public `guillermomolina/protos` main was [`3f37ef3fdd507504720bf30008317b6304ccffa6`](https://github.com/guillermomolina/protos/commit/3f37ef3fdd507504720bf30008317b6304ccffa6); the proposed `spec/io/HTTP.md` was **not** published. This SHA is a *read-only observation*, **not** a ratification commit. The active local checkout can be ahead; its actual HEAD and dirty paths must be respected.
- The maintainer reported `git diff --check` clean and all local tests passed. **No D215 file delta, tested SHA or test list was supplied.** Those statements are human-reported checkout status, not independent verification that D215 specification or runtime exists at the observed published revision.
- No D215 runtime code or HTTP integration test is attributed to this record. The design Issue remains open while ratification is pending; do not create a `LIBxxx` implementation Issue or treat scheduling labels as proof of publication.

## Additional owner approval — D215-F9 A–F (P9, 2026-10-24)

The owner explicitly said: **“Apruebo D215-F9 completo, puntos A–F. Continúa con la ratificación normativa sin implementar el runtime.”** This is recorded in [D215 P9](https://github.com/guillermomolina/protos/issues/961#issuecomment-6388054314) and supplements F3–F8. It is **not** evidence of a published `spec/io/HTTP.md` or permission to implement the runtime.

The six approved normative refinements, for the local editor to apply exactly, are:

- **F9-A:** After ordinary successful dispatch of `serveOne` / `serveOne.owning`, invalid `duplex`, `handler` or `limits` fails the returned Future with fresh `InvalidIOArgument`, before any HTTP I/O; pre-dispatch lookup/arity language failures remain synchronous. **Supersedes P6's contradictory synchronous-initial-validation wording** and aligns with `spec/io/IO_CORE.md` Future failure-channel rule.
- **F9-B:** Framer-origin `HttpError` from a request-body read, even if caught by the handler, terminalizes the session. Before final response commitment attempt one suitable best-effort 400 (`malformedRequest`) or 413 (`requestLimit`) and fail service Future with original `HttpError`; after commitment no second final. Independent handler failure keeps the generic precommit 500 route.
- **F9-C:** With response status 204, 205, or 304, body may be only `null` or empty `Bytes()`; reject nonempty `Bytes` and every `ByteReadable` before commitment with fresh `HttpError(kind="invalidResponse")`, without probing or consuming the stream.
- **F9-D:** `HttpError.kind` is ordinary `String` restricted to `malformedRequest`, `requestLimit`, `invalidResponse`, `unsupportedFeature`; optional `HttpError.status` is ordinary `Integer`. Not an enum.
- **F9-E:** 205 emits `Content-Length: 0`, while 204 and 304 emit neither `Content-Length` nor `Transfer-Encoding`.
- **F9-F:** HEAD response with `Bytes` must emit the would-be exact `Content-Length` and no body octets; streaming `ByteReadable` is never read and omits CL; `null` emits CL0 unless status prohibition overrides.

P6 already fixes the remaining editor questions: issue `100 Continue` after admissible headers **before** calling the one-Request-argument handler; `maxPrefetchBytes` limits **HTTP-managed** lookahead, not kernel/underlying byte consumption. Borrowed transport remains open without rewind or promise of reusability.

**Invariant/delta check:** Option A / A3-minimal / Q0 and M1/H1/B1/L1/W1/S1 remain intact, except for the *explicitly approved* F9-A replacement of P6 synchronous invalid-initial-argument behavior. No new Network or Actor authority, TLS, HTTP client, keepalive, deadlines, sockets, or privileged error hierarchy. RFC 9110/9112 interpretation is unchanged.

**Local editor handoff:** Claude reported `spec/io/HTTP.md`, `spec/io/NETWORK.md`, `spec/semantics/ERRORS.md`, `AGENTS.md` edited locally and nine questions, with no compilation, tests, Git operations or specification-changelog edit. The next pass is **edit-only** on `spec/io/HTTP.md`; preserve the other three files unless an exact new inconsistency needs a minimal correction. The human reviews full new file (untracked files do not appear in ordinary `git diff`), runs the permitted validation, then assigns the moving specification revision and executes Git publication. No runtime changes.

## The selected contract — editing authority, not new decisions

1. **M1/API.** `std:http/Server` exposes ordinary `serveOne(duplex, handler, limits) -> Future<null>` and `serveOne.owning(duplex, handler, limits) -> Future<null>`. Duck-typed duplex = ByteReadable + ByteWritable; `.owning` additionally needs Closable. One HTTP/1.1 request and one final response, no listen/accept, Network acquisition, DNS, TLS, clock, background listener manager or auto Actor creation. Handler is called exactly once in the service Task once start-line/headers/framing are admitted; its result is a plain Response-shaped object, not an auto-awaited Future. Request fields: `method: String`, `target: Bytes`, `version: "1.1"`, `headers: Array<[String,Bytes]>`, `body: ByteReadable`. Response: `status: Integer 200..599`, `headers` same shape, `body: null | Bytes | ByteReadable`. Structural, ordinary objects, not privileged nominal records.
2. **H1/headers.** Preserve repeated header occurrences, order, original field-name ASCII case, value octets and raw target Bytes. Case-insensitive ASCII only for field-name semantic comparisons; remove boundary OWS, never auto-decode, merge or map. Snapshot internal request framing independently from mutable visible objects; snapshot outbound metadata and immediate Bytes body before final response commit. The driver owns outgoing CL, TE, Connection; reject conflicting control fields, CRLF injection and invalid names/values.
3. **B1/bodies.** Incoming message-bounded `ByteReadable.read(maxBytes)`: bounded fresh nonempty Bytes, null only at *logical HTTP body EOF*, ordinary future suspension and cancellation/zero-logical-consumption guarantees. A complete zero-length request body returns null immediately. A chunked body validates and consumes its wire framing/trailers below the body API. Outgoing null is empty, Bytes uses exact CL, unknown-length ByteReadable is written with bounded awaited chunked frames. No eager materialization, text transformation, retry or hidden ownership transfer.
4. **L1/ownership.** Default `serveOne` borrows duplex, never closes or half-closes it; `.owning` explicitly closes on every exit following established cleanup precedence. Outgoing ByteReadable is borrowed even if closable. Request.body expires at session retirement and later reads fail freshly with IOLifecycleError; no detached body, read-ahead rewind, socket hijack, implicit drain, P/Actor transfer or rollback. Handler and subsequent response streaming share connection Task lifetime; all structured children must terminate.
5. **W1/wire.** Strict CRLF HTTP/1.1 origin server; valid unique Host; origin-, absolute- and OPTIONS asterisk-form; CONNECT authority, Upgrade and tunnels not implemented. Reject request TE+CL, repeated/list CL, invalid CL, unsupported codings, bad chunks, obs-fold and whitespace-before-colon. Request with neither framing header has empty body, never EOF-delimited. Driver sends supported 100-continue when body is potential and header-level checks passed; unsupported Expect yields best-effort 417. Final `Connection: close`, no keepalive/pipeline. HEAD has no response content; 204/205/304 no content; no 1xx final; outbound CL and TE never coexist; trailers validated, bounded and discarded. Best-effort suitable 4xx/5xx only before final commit, with terminal failure and no second final response after commit. No promised remote delivery on TCP close.
6. **S1/limits and exposure.** `limits` = null or ordinary object containing optional positive-Integer own slots; reject unknown/invalid slots **with an already-failed Future holding fresh `InvalidIOArgument` after successful `serveOne` dispatch**, without HTTP I/O (F9-A explicitly supersedes the earlier P6 synchronous-validation wording); snapshot values before work. Defaults: request line 8192 octets, headers 32768 octets / 100 fields, chunk line 8192, trailers 8192 / 100, decoded request body 33554432, HTTP prefetch <=65536, HTTP I/O chunk <=16384. Overflow-safe quotas; incoming errors can occur during later stream reads after handler effects. No global concurrent admission or scheduling cap in serveOne: the application must bound Tasks and connections. **Do not advertise direct hostile Internet safety**: portable clock, deadline, Future race/select and timeout/admission guarantees are absent, so this initial scope is for loopback, trusted networks or genuinely protective ingress.
7. **Errors.** `std:http/Server.HttpError` is one ordinary Standard Library error prototype with direct parent Error, not an implicit Core Prelude error. Fresh HTTP failure instances have stable `kind` values `malformedRequest`, `requestLimit`, `invalidResponse`, `unsupportedFeature`, plus optional `status`. Underlying IO errors, Cancelled and handler errors preserve existing identity/category and cleanup precedence. Unexpected handler failure before final commit permits one generic zero-body 500 attempt, but still fails Future with original error; never invent post-commit second response.

**Do not resolve** new message fields, parser ownership states, generic timers, transports, TLS/DNS, HTTP client API, keepalive/multiplexing, trailer verification, routing, auth or package-tool policy by extrapolating beyond the [approved P6 packet](https://github.com/guillermomolina/protos/issues/961#issuecomment-6385736153). If specification authors discover an observable gap requiring substantive selection, record the alternatives in #961 and return for explicit owner approval.

## Editing agent — next single grouped slice

**Type:** IMPLEMENTATION OF APPROVED NORMATIVE SPECIFICATION (not HTTP runtime implementation).
**Repository:** `guillermomolina/protos`, assume actual local HEAD is authoritative. This single slice groups the four normative/document-owner surfaces and avoids speculative extra file changes.

1. Read local root `AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, `spec/AGENTS.md` and canonical `spec/io/{NETWORK,IO_CORE,BYTE_IO}.md`, `spec/concurrency/{FUTURES_AND_TASKS,ACTORS}.md`, `spec/semantics/{ERRORS,MODULES}.md`, `spec/PROTOS_SPEC_CHANGELOG.md`. These are local API/contracts; no browsing or other repository access is needed.
2. **Create** `spec/io/HTTP.md` as the normative owner of the exact selected HTTP Standard Library server semantics above, with explicit conformance examples/counterexamples for duplicate headers, HTTP body logical EOF, 100-continue, TE+CL, READ-after-retirement, borrowed-versus-owning socket, cancellation, streaming output and security boundary. Distinguish the approved first server from future client/transport/supervisor work; avoid host-Java details.
3. **Edit** `spec/io/NETWORK.md` only to add the minimum cross-reference distinguishing existing Core TCP capabilities from optional `std:http/Server` over ordinary byte-duplex; do not add Network API, DNS or TLS.
4. **Edit** `spec/semantics/ERRORS.md` only to register the portable `HttpError` direct-parent relation and *module-local non-Prelude visibility*, without widening the mandatory Core error prototype table into a new always-present Prelude binding.
5. **Edit** root `AGENTS.md` only to add `spec/io/HTTP.md` to the ownership map with a precise description. Do not rewrite the existing policy.
6. **After the human's successful verification of these normative edits**, prepare the next `spec/PROTOS_SPEC_CHANGELOG.md` revision and only then request the human to commit/push. Do not assume the public `0.1.470` base revision is still current; use actual local HEAD. No `pom.xml`, runtime Java, tests, `CHANGELOG.md` or package changes in this specification-only slice. No HTTP implementation issue until the normative commit is published.

**Agent must only edit files; no compilation, tests, validators, benchmark runs, `git add/commit/pull/push/checkout/reset` or branch creation.** Inspect current changed paths before editing and preserve all unrelated staged and unstaged work. At handoff list exact files, owned clauses, open ambiguities, and specific human validation commands without claiming results.

## Human executor — validation/publication boundary

After editing, the maintainer executes in the `protos` checkout:

```bash
git rev-parse HEAD
git status --short
git diff --check
git diff -- spec/io/HTTP.md spec/io/NETWORK.md spec/semantics/ERRORS.md AGENTS.md
```

The first new file is normally untracked, so also read its complete contents explicitly: `sed -n '1,260p' spec/io/HTTP.md` (and subsequent portions if longer). If files are staged, inspect `git diff --cached` too; this handoff does not authorize overwriting existing staged work. Static link/spec checks or tests, when applicable, are human-executed per AGENTS. After the first green review, update the **specification changelog** as the final metadata-only step; never run tests after a project version/`CHANGELOG.md` update. Stage only the exact owned paths using explicit `git add` paths, commit one line `D215: ratify portable HTTP server semantics`, and publish under normal human Git rules (including the prepublication `origin/main` synchronization requirement).

Once the product normative commit is verifiably published, update #961 with its exact SHA and then publish a **separate revision-pinned normative-ratification evidence record** here; only after that allocate the separately scoped HTTP runtime `LIBxxx` implementation Issue with its proper native relation if supported.

## Upstream references pre-researched for the editor

Use these as *facts and rationale*, not as a request for the editing agent to investigate:

- RFC 9110 semantics and 9112 HTTP/1.1 framing, Host, trailers, incomplete responses and connection close: https://www.rfc-editor.org/rfc/rfc9110 and https://www.rfc-editor.org/rfc/rfc9112 .
- Erlang OTP sockets and Ranch/Cowboy: per-connection ownership/handler execution is useful precedent, but mailbox-owned active-mode sockets or transferred controllers are **not** Protos HTTP semantics: https://www.erlang.org/doc/apps/erts/erl_dist_protocol.html ; https://ninenines.eu/docs/en/ranch/2.1/guide/introduction/ ; https://ninenines.eu/docs/en/cowboy/2.13/guide/ .
- Elixir Bandit request body flow and server Plug lifecycle: https://hexdocs.pm/bandit/Bandit.html .
- Go net/http HTTP message and stream ownership, including explicit buffered-data issue for hijacking: https://pkg.go.dev/net/http .
- Rust hyper/http-body streams and trailer frames demonstrate why message-bounded streaming cannot become ordinary TCP EOF: https://docs.rs/http-body/latest/http_body/ .
- Python ASGI request/response event contract is a separate public event alphabet **not selected**: https://asgi.readthedocs.io/en/latest/specs/www.html .
- Kestrel request/response limits and timers, and Node HTTP header/request timeout guidance demonstrate absent direct-Internet protection without portable deadlines: https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/options ; https://nodejs.org/api/http.html .
- All detailed alternatives, counterexamples and twelve-criterion scoring remain in [D215 P6](https://github.com/guillermomolina/protos/issues/961#issuecomment-6385736153).

## Pending gates

```ini
D215_OWNER_APPROVAL=YES_F3_TO_F8_AND_F9_A_TO_F
D215_F9_APPROVAL_RECORD=https://github.com/guillermomolina/protos/issues/961#issuecomment-6388054314
D215_EXACT_PUBLIC_CONTRACT=M1_H1_B1_L1_W1_S1
NORMATIVE_HTTP_SPEC_PUBLISHED=NO_AS_OF_2026_10_24_OBSERVATION
HTTP_RUNTIME_IMPLEMENTED=NOT_CLAIMED
HUMAN_LOCAL_DIFF_CHECK=REPORTED_PASS_NOT_SHA_ATTRIBUTED
HUMAN_LOCAL_TESTS=REPORTED_PASS_NOT_SHA_ATTRIBUTED
D215_CLOSURE=OPEN_PENDING_NORMATIVE_RATIFICATION
NEXT_SLICE=IMPLEMENTATION_OF_APPROVED_SPEC
NEXT_REPOSITORY=guillermomolina/protos
```
