# D218 — Single-exchange HTTP/1.1 Client over an explicit duplex

**Decision:** RATIFIED — OWNER-APPROVED EXACT CANDIDATE B; normative publication completed at Protos spec `0.1.476` (2026-10-28).
**Issue:** [D218/#969](https://github.com/guillermomolina/protos/issues/969).
**Approval provenance:** [owner decision recorded on the D218 Issue](https://github.com/guillermomolina/protos/issues/969#issuecomment-6460926536), quoting the owner's explicit active-chat selection.
**Approved full candidate:** [D218 research §5](../../evidence/D218/D218_EXPLICIT_DUPLEX_HTTP_CLIENT_COMPARATIVE_RESEARCH_2026_10_28.md) at [project record revision `30cbacd985726c799261d6714268a41458a0f157`](https://github.com/guillermomolina/protos-project-docs/blob/30cbacd985726c799261d6714268a41458a0f157/docs/project/evidence/D218/D218_EXPLICIT_DUPLEX_HTTP_CLIENT_COMPARATIVE_RESEARCH_2026_10_28.md).
**Protos comparison baseline:** `b0ae444f30951f8d85a6776bab7c057bed8d0a3d`, specification `0.1.474`; **not** a new Client normative revision.
**Implementation:** one consolidated [I127 #970](https://github.com/guillermomolina/protos/issues/970) slice published as [`0.3.410-SNAPSHOT`](https://github.com/guillermomolina/protos/commit/6b80a8f70325e20999f55add0ccf8e9c6ec7c8ba); human integrated validation reported PASS.

This record is a non-normative owner-approval and rationale record. The Client contract is now published in `spec/io/HTTP.md` §16 with the global `spec/PROTOS_SPEC_CHANGELOG.md` revision `0.1.476` at [`6b80a8f70325e20999f55add0ccf8e9c6ec7c8ba`](https://github.com/guillermomolina/protos/commit/6b80a8f70325e20999f55add0ccf8e9c6ec7c8ba). This decision record is **not a substitute for that normative specification**. The revision-bound [D218 publication evidence](../../evidence/D218/D218_HTTP_CLIENT_SPEC_0_1_476_RATIFICATION_2026_10_28.md) supplies the remaining durable ratification/closure checkpoint.

## Exact owner selection

The owner said (2026-10-28, in the active interaction):

> Apruebo D218-B — cliente HTTP/1.1 sobre dúplex explícito, con respuesta incremental dentro de un callback

This approves the precise Candidate B contract in the fixed research snapshot linked above, not an evolving idea, shorthand label, broad implementation delegation, or authorization to resolve new semantic ambiguities without returning to the owner.

## Selected public protocol boundary

The Standard Library adds an independent, opt-in `std:http/Client` module alongside `std:http/Server` and `std:http/Host`. It exports `Client` and `ClientError`, with no Prelude binding. `ClientError` is a distinct ordinary Error prototype whose immediate parent is `Error`; it does not replace or alter `Server.HttpError`.

```text
Client.exchangeOne(duplex, request, onResponse, limits)        -> Future<result>
Client.exchangeOne.owning(duplex, request, onResponse, limits) -> Future<result>
```

- `duplex` is explicitly caller-provided, satisfying `ByteReadable` and `ByteWritable`, plus `Closable` for owning. There is no implicit `Network`, TCP connection acquisition, URL parsing, DNS, TLS, timer, Actor creation, host HTTP engine or other authority. A caller-provisioned secure byte duplex can carry HTTP bytes, but this is not an end-to-end URL-based HTTPS facility.
- After ordinary dispatch, invalid arguments produce a fresh already-failed Future containing a fresh `InvalidIOArgument`, without Task, HTTP session, bytes contributed/consumed or ownership acquisition. Valid admission captures mutable request state, acquires custody before cancellation can be observed for owning, creates exactly one Actor-local cooperative exchange Task in the invoking Task scope or Actor root asynchronous scope, and returns its Future.
- `request` is ordinary structural data: own `method:String` (valid HTTP token), `target:Bytes` (origin-form or `OPTIONS *`), `headers:Array<[String,Bytes]>`, `body:null|Bytes`. Exactly one syntactically valid explicit `Host`. No HTTP-specific String/Bytes conversion. The driver snapshots mutable request data before I/O, writes HTTP/1.1, `Content-Length` (zero for null), `Connection: close` and the remaining valid headers in order. Caller-supplied framing/hop-by-hop fields are rejected. CONNECT, Upgrade, absolute/proxy targets, Expect waiting and streaming upload are not admitted.
- `onResponse` is an ordinary callable invoked at most once within the service Task for the final HTTP response. It receives a fresh structural `{version:"1.1",status:Integer,headers:Array<[String,Bytes]>,body:ByteReadable}`; header occurrences/order/casing and binary values are preserved. A 4xx or 5xx status is a normal received response. The callback result becomes the Future's success value under existing Future adoption, after HTTP retirement and mandatory cleanup. Returning a pending Future does not extend the body reader's validity.
- The strict parser validates interim 100–199 responses (but rejects protocol-switching 101), bounded by `maxInformationalResponses`, then exactly one final 200–599 response. It enforces response status/head syntax; no-content HEAD/1xx/204/205/304; exactly one valid Content-Length, exactly chunked Transfer-Encoding, or close-delimited body. TE+CL, repeated/comma-list CL, malformed headers/chunks/trailers, unsupported transfer codings and premature EOF fail as specified. Trailers are validated/discarded, not exposed.
- `response.body` is incrementally readable only during the callback; no whole-body buffer, forced drain, second response parsing or connection reuse. The reader terminalizes on a reached late framing failure even if caught and subsequently gives fresh `IOLifecycleError`; the original `ClientError` remains the exchange failure except for required cleanup-precedence replacement. After callback return/failure/cancellation, the body is retired and escaped reads fail with `IOLifecycleError`.
- Borrowed `exchangeOne` never closes or half-closes its duplex. Owning `exchangeOne.owning` closes exactly once on every admitted terminal path, including cancellation before Task body execution, and the Future cannot terminalize before mandatory close/cleanup. All ordinary `IOError`, `Cancelled`, `IOLifecycleError`, `InvalidIOArgument` and callback failures preserve their existing categories and identity as specified.
- Fresh `ClientError.kind` is restricted to `"malformedResponse"`, `"responseLimit"`, `"unsupportedFeature"`; it does not synthesize an HTTP status. No automatic retry of a partial/non-idempotent request, redirect, compression or shared mutable connection state.
- Limits are `null` or ordinary own-slot options. Defaults: `maxStatusLineBytes=8192`, `maxHeaderBytes=32768`, `maxHeaderFields=100`, `maxChunkLineBytes=8192`, `maxTrailerBytes=8192`, `maxTrailerFields=100`, `maxResponseBodyBytes=33554432`, `maxInformationalResponses=8`, `maxPrefetchBytes=65536`, `ioChunkBytes=16384`. All supplied values are positive signed-32-bit-range Integers. Unknown own slots are rejected, inherited slots ignored; all counters are overflow-safe.
- In this first mode request-head/body writes complete before response parsing begins. No early-final monitoring during upload, hard deadline, hostile-peer liveness bound, TLS authentication/truncation guarantee or production Internet safety is claimed.

The full research snapshot contains the precise logical error/cancellation consequences, conformance matrix, and intentionally deferred extensions. A conflict between its exact approved content and an implementing draft must be surfaced for owner correction, not silently resolved by the implementation.

## GITHUB021 owner-invariant reconciliation

| Existing approved invariant | Exact B candidate |
| --- | --- |
| D215 `Server.serveOne[.owning]`, response, HttpError and one-Task semantics | Preserved; independent Client module and ClientError |
| D216 optional Host and Worker lifecycle | Preserved; no accept or listener changes |
| D217 attenuated Actor-delegable TcpAcceptor and non-transferable TcpConnection | Preserved; no live TCP transfer |
| Core explicit Network authority, numeric-endpoint TCP | Preserved; supplied duplex only |
| Byte IO, Future, Task/Actor ownership and `ensure` cleanup precedence | Preserved; HTTP-specific response framing is the bounded new Standard Library behavior |
| Protos-first optional library, no global coordination when unused | Preserved; no pool, Java HTTP engine or ambient HTTP service |
| Owner-approved one investigation + one implementation slice | Preserved; no mechanical parser/test Issue proliferation |

**DECISION_APPROVAL_PROVENANCE=PASS** (exact active owner approval recorded on D218 issue).
**DECISION_INVARIANT_CONSISTENCY=PASS** (selected candidate preserves the applicable prior invariants as compared above).
**NORMATIVE_SPEC_PUBLICATION=PASS** (`0.1.476`), **PROTOS_CLIENT_IMPLEMENTATION=PASS** (`0.3.410-SNAPSHOT`; owner-reported `make test`/`make check` PASS), **D218_CLOSURE=READY_FOR_LIVE_ISSUE_FINALIZATION** (after durable record publication).

## Publication and closure checkpoint

I127 #970 owned the single consolidated HTTP Client change in `guillermomolina/protos`: normative Client §16, Protos-first implementation, only private Java bootstrap reuse, guest/Java/NIO conformance, human integrated validation, version/changelog finalization, and one product commit. D215/D216/D217 semantics and the deferred URL/DNS/TLS/pool/HTTP2–3 boundary remain unchanged. Human-executor mode governed product validation and publication; this project-documentation repository has the user-authorized bounded direct-publication exception.

Product publication and the normative spec revision are now verified at [`6b80a8f70325e20999f55add0ccf8e9c6ec7c8ba`](https://github.com/guillermomolina/protos/commit/6b80a8f70325e20999f55add0ccf8e9c6ec7c8ba). The companion [revision-bound ratification record](../../evidence/D218/D218_HTTP_CLIENT_SPEC_0_1_476_RATIFICATION_2026_10_28.md) documents the approval/invariant/validation chain. After that record is published, the coordinator must re-read its exact project-record revision and live D218 Issue state, then close the Issue only when the GITHUB015 cross-reference checks pass.
