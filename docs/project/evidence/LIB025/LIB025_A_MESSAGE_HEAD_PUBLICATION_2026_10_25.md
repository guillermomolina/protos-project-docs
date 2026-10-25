# LIB025-A — Internal HTTP/1.1 message-head codec published in Protos 0.3.403

Date: 2026-10-25. Non-normative, revision-bound implementation evidence for [LIB025 #962](https://github.com/guillermomolina/protos/issues/962). This is **slice A only**; it is not closure of LIB025 or publication of the public HTTP server. Normative authority is the previously published [D215 specification at `9dc4e508`](https://github.com/guillermomolina/protos/commit/9dc4e50875252508e2afecf91541fb50f756d450), particularly `spec/io/HTTP.md` (specification 0.1.471).

## Exact product identity

```text
WORK_ITEM=LIB025-A
PARENT_ISSUE=guillermomolina/protos#962
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=a6b67abdfcc031dad17acadc83e065c3a54d0bcd
PRODUCT_COMMIT=LIB025-A: add internal HTTP/1.1 message-head parser and preparer in Protos (0.3.403)
IMPLEMENTATION_VERSION=0.3.403-SNAPSHOT
PRODUCT_CHANGED_PATH_COUNT=16
NORMATIVE_SPEC_REVISION=0.1.471
NORMATIVE_SPEC_CHANGE_IN_THIS_COMMIT=NO
GIT_DIFF_CHECK=PASS (maintainer-reported)
LOCAL_TESTS=PASS (maintainer-reported; no full transcript supplied)
EXACT_MAKE_TEST_GATE=NOT_EVIDENCED_HERE
```

The exact [published commit](https://github.com/guillermomolina/protos/commit/a6b67abdfcc031dad17acadc83e065c3a54d0bcd) was independently located and its 16 changed paths inspected through GitHub. The maintainer explicitly confirmed that every local test passed and `git diff --check` was clean; no per-class or global execution transcript was supplied, so this record does not claim independent test execution.

## Shipped private behavior

- `protos/lib/http/internal/MessageHead.protos` defines exactly two top-level helpers: `requestHeadParser` and `prepareResponseHead`. The module is importable internally but is **not** the published `std:http/Server` public surface. There is no Java HTTP engine, new Prelude/Core binding, duplex access, Future, Task, or ambient network authority in this slice.
- `requestHeadParser(limits).feed(bytes, offset, length)` incrementally accepts HTTP/1.1 request-head octets, maintains strict bounded request-line and header parsing, validates Host, request target, framing, Expect and reserved/unsupported input, and stops at the first empty-header-line CRLF. It copies request-target and header values into new Bytes; no request body octets are consumed. Bad source ranges signal an ordinary Error before modifying parser state.
- `prepareResponseHead` validates and snapshots response status, headers and fixed Bytes body before emitting a private header-encoding/framing plan. Status rules, HEAD, forbidden framing headers and `Connection: close` are included. It performs no writes.
- `ProtosStandardLibraryDocumentationExtractor` and its regression test now exclude library modules below an `internal` path segment from **public generated Standard Library documentation**, without making those modules inaccessible to the internal resolver.
- `protos/corpus/library/http` was registered as an ordinary suite-native Test Tool corpus containing two ordered files, `http/request-head.protos` (52 Test cases) and `http/response-head.protos` (29 Test cases), with registry, suite-graph, progress-grouping and plan tests updated. These are 81 declared guest Tests; their PASS is maintainer-reported.

## Non-delivery and continuation

No public `protos/lib/http/Server.protos`, request-body `ByteReadable`, chunked transfer framer, streamed response, one-service-Task lifecycle, owning-close handling or end-to-end Duplex conformance exists as a consequence of this commit. Those are the remaining B/C acceptance requirements of [LIB025 #962](https://github.com/guillermomolina/protos/issues/962); **the issue stays OPEN**.

The next bounded edit-only slice is B1, a private Protos request-body framer (NONE, CONTENT_LENGTH and CHUNKED; fragment-safe, exact consumption, strict chunk and trailer validation, independent limits) with one registered guest suite. B2/C subsequently connect Byte I/O, laziness, error provenance, response streaming, session retirement and structured Task ownership, preserving D215 rather than weakening it. No new formal issue is allocated for the micro-slices.

## Boundaries

This is implementation evidence, not a second specification. In a conflict, the ratified `spec/io/HTTP.md` and referenced Byte I/O, concurrency and Errors specifications at the product revision govern. The exact product revision is pinned above rather than represented only by a moving `main` link.
