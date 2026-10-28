# I127 — `std:http/Client` single-exchange HTTP/1.1 implementation closure (0.3.410)

**Recorded:** 2026-10-28. **Role:** exact-revision implementation and human-validation evidence; not normative authority.
**Implementation issue:** [I127 #970](https://github.com/guillermomolina/protos/issues/970).
**Approved design:** [D218-B #969](https://github.com/guillermomolina/protos/issues/969), [exact owner approval](https://github.com/guillermomolina/protos/issues/969#issuecomment-6460926536) and [D218 decision record](../../decisions/language/D218_HTTP_1_1_EXPLICIT_DUPLEX_CLIENT.md).
**Exact published Protos revision:** [`guillermomolina/protos@6b80a8f70325e20999f55add0ccf8e9c6ec7c8ba`](https://github.com/guillermomolina/protos/commit/6b80a8f70325e20999f55add0ccf8e9c6ec7c8ba) on `main`.
**Implementation version:** `0.3.410-SNAPSHOT`. **Normative global specification revision:** `0.1.476`.
**Commit title:** `I127: add std:http/Client single-exchange HTTP/1.1 client in Protos (0.3.410)`.

## Published scope

The independently imported Standard Library `std:http/Client` exports `Client` and `ClientError`, with no Prelude binding. `Client.exchangeOne(duplex, request, onResponse, limits)` and `.owning` perform one HTTP/1.1 request/response exchange using a caller-supplied byte duplex. The Client opens no network connection, resolves no address, and implements no URL, TLS, redirect, retry, pool or Java HTTP engine.

The Protos implementation comprises `protos/lib/http/Client.protos` and internal `ClientHead`, `ClientLimits` and `ClientSession` modules. Shared `MessageSyntax` centralizes pure HTTP wire syntax; `MessageBody` uses one directional body/framing engine for server requests and client responses while preserving existing `Server` and `Host` contracts. The only Java production changes register the existing private `ProtosHttpAsyncFacility` for the Client module, without adding operations or Java parsing.

The client snapshots validated requests, emits exact-length request bodies with `Connection: close`, parses bounded informational and final response heads, and exposes an incremental `ByteReadable` body only during `onResponse`. It supports fixed-length, chunked and close-delimited response framing; preserves binary header order/casing; enforces strict framing and status-specific limits; handles `HEAD`, 1xx, 204, 205 and 304; preserves the callback-result/Future adoption and late framing-error precedence; and distinguishes borrowed from mandatory exactly-once owning close including cancellation. Final source corrections preserve external header-value OWS (including `Host`, with semantic validation on its OWS-trimmed range) and classify RFC 9110 BWS-valid parameterized `Transfer-Encoding` as unsupported rather than malformed.

The normative contract was published in `spec/io/HTTP.md` §16, with corresponding `spec/io/NETWORK.md`, `spec/semantics/ERRORS.md`, and `spec/PROTOS_SPEC_CHANGELOG.md` changes at global specification `0.1.476`. The existing D215 Server, D216 Host and D217 TcpAcceptor semantic boundaries were not revised.

## Test and release evidence

The product commit changes 26 paths (6,903 additions and 776 deletions). It adds five registered Protos HTTP Client suites (`client-admission`, `client-request`, `client-response-head`, `client-response-body`, `client-lifecycle`) plus `ProtosHttpClientIntegrationTest` for Task behavior, cancellation, Actor scope and real loopback TCP. The HTTP test corpus plan, reporting groups and their Java tests were updated.

Guillermo Adrián Molina, acting as Human Executor, reported on 2026-10-28 for the published I127 candidate:

| Gate | Reported result |
| --- | --- |
| Focused tests, including Client and affected HTTP regressions | PASS (reported earlier in the implementation thread; exact command transcripts not preserved here) |
| `git diff --check` | PASS — clean |
| `make test` | PASS — local integrated suite |
| `make check` | PASS — local validation |
| Product commit and push | PASS — exact `main` revision [`6b80a8f70325e20999f55add0ccf8e9c6ec7c8ba`](https://github.com/guillermomolina/protos/commit/6b80a8f70325e20999f55add0ccf8e9c6ec7c8ba) verified by GitHub |

These are **owner-reported local results**, not tests independently executed by this coordinating agent; no raw logs or post-metadata test run is claimed. The product commit contains the final `pom.xml`, `CHANGELOG.md` and specification changelog updates, consistent with the prescribed after-test finalization gate.

## Closure and remaining boundaries

The owner-approved D218-B implementation and its full local validation/publication gates are complete at the exact product revision above. I127 #970 is eligible for **completed** closure following publication of this durable evidence and the corresponding live-Issue verification. D218's separate normative-ratification closure is documented in [the D218 publication record](../D218/D218_HTTP_CLIENT_SPEC_0_1_476_RATIFICATION_2026_10_28.md).

Future higher-level HTTP facilities (DNS/URL-first acquisition, portable TLS, proxying, streaming uploads, pooled or persistent connections, content codings, redirects, retries, HTTP/2–3) remain out of scope. This evidence is a snapshot of one published implementation; it is not a substitute for the normative specification or a guarantee about untested external peers.
