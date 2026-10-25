# LIB025 — HTTP/1.1 single-request server: 0.3.404 publication and closure gate

Date: 2026-10-25. Revision-bound, non-normative implementation evidence for [LIB025 #962](https://github.com/guillermomolina/protos/issues/962). The project owner confirms the published implementation and all local tests PASS, but **formal Issue closure remains pending** for the normative error-precedence clarification identified below.

## Exact published identity

```text
WORK_ITEM=LIB025
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=01535c8609831bcca50da481583190b657242eda
IMPLEMENTATION_VERSION=0.3.404-SNAPSHOT
PRODUCT_COMMIT=LIB025: complete std:http/Server HTTP/1.1 single-request server in Protos (0.3.404)
PRODUCT_CHANGED_PATH_COUNT=20
NORMATIVE_BASE_SPEC_REVISION=0.1.471
HTTP_TEST_CASES=247 (seven registered suite-native sources)
HTTP_PROGRESS_GROUPS=7
GIT_DIFF_CHECK=PASS (maintainer-reported)
LOCAL_TESTS=PASS (maintainer-reported)
EXACT_MAKE_TEST_TRANSCRIPT=NOT_SUPPLIED
FORMAL_LIB025_CLOSURE=PENDING_NORMATIVE_RECONCILIATION
```

The exact [0.3.404 product commit](https://github.com/guillermomolina/protos/commit/01535c8609831bcca50da481583190b657242eda) was independently fetched from GitHub; its 20 changed paths include the public Protos module, the internal body/session implementation, a minimal private Java Task/Future bridge, expanded guest/Java tests, Test Tool grouping, `pom.xml` and `CHANGELOG.md`. The previous private [LIB025-A head codec publication](https://github.com/guillermomolina/protos/commit/a6b67abdfcc031dad17acadc83e065c3a54d0bcd) remains separate historical evidence. **No `spec/io/HTTP.md` change was included in the 0.3.404 commit.**

## Published API and architecture

- Explicit `std:http/Server` import exports exactly `Server` and `HttpError`, with no Prelude binding; `Server.serveOne(duplex, handler, limits)` borrows its duplex and `Server.serveOne.owning(...)` closes its acquired duplex at terminalization. Both return `Future<null>`.
- HTTP protocol decisions and orchestration live in hand-written Protos: `protos/lib/http/Server.protos`, `protos/lib/http/internal/Session.protos`, `MessageBody.protos`, and the preceding `MessageHead.protos`. No TCP listener, new Network authority, Java HTTP engine, timer, or implicit HTTP host privilege is introduced.
- The private `ProtosHttpAsyncFacility` supplies terminal Futures, read-only capability checks, and exactly one Actor-local service Task. `ProtosTask` has an opt-in pre-start cancellation unwind entry used only by HTTP, so owning close is awaited even if cancellation occurs before the Task's first service instruction; ordinary Tasks keep their original path.
- A framed request body uses a borrowed `BufferedReader` without creating Tasks per read. Read-ahead preserves octets and late-framer error provenance; the read reaching the framing error fails with the original `HttpError`, while subsequent reads after retirement fail with fresh `IOLifecycleError`.
- Strict initial rejection and best-effort final responses, `100 Continue`, single handler invocation, HEAD/204/205/304 framing, fixed/chunked streamed response, terminal retirement and conservative no-second-final-response guard are covered by the published implementation and tests.

## Published acceptance coverage

The seven suite-native HTTP source files and report groups (the **original Case identities and scheduling remain unchanged**) are:

| Guest Test source under `protos/tests/library/http/` | Cases | Progress group |
| --- | ---: | --- |
| `request-head.protos` | 52 | `library/http/request-head` |
| `response-head.protos` | 29 | `library/http/response-head` |
| `request-body.protos` | 77 | `library/http/request-body` |
| `server-admission.protos` | 22 | `library/http/server-admission` |
| `server-request.protos` | 24 | `library/http/server-request` |
| `server-response.protos` | 25 | `library/http/server-response` |
| `server-lifecycle.protos` | 18 | `library/http/server-lifecycle` |
| **Total** | **247** | **7 groups** |

`RepositoryCorpusPlans.protos` registers the seven sources; `RepositorySuite.protos` subdivides **only progress reporting** by exact `casePath`, in the style of TOOL012/TOML. `ProtosTestToolLib025HttpProgressGroupingTest` adds eight structural tests for grouping, empty/partial plans, exact identities, invalid paths, and CLI `--directory` / `--file` selection. Java `ProtosHttpServerIntegrationTest` exercises the service and ownership/cancellation paths. No new suite-graph leaf, Case identity, scheduler partition or separate HTTP corpus is claimed.

**Validation provenance:** the maintainer states that `git diff --check` was clean and all tests passed locally before committing and pushing 0.3.404. This documentation action did **not** compile, execute Protos, rerun tests, or independently verify a `make test` command transcript; the issue's explicit integrated `make test` gate is therefore not separately evidenced here.

## Owner-approved error precedence; normative text still pending

During implementation review the project owner **explicitly approved** the following three-way priority rule for a failed best-effort error-response write, plus the existing `ensure` cleanup precedence:

| Context | Service error when best-effort `write` fails |
| --- | --- |
| Initial request rejection (400/413/414/417/431/501/505) | The duplex write I/O Error prevails over the initial `HttpError`. |
| Handler failure or `invalidResponse` (optional 500) | The duplex write I/O Error prevails over the original handler / `invalidResponse` error. |
| Late request-body framer failure (best-effort 400/413) | The original framer `HttpError` prevails even if the best-effort write fails. |
| Owning duplex `close()` fails during `ensure` | Ordinary cleanup-error precedence applies over the preceding service outcome. |

The implementation's `attemptEmptyResponse(status, preserveOriginal)` follows this choice, with `preserveOriginal = true` for late framer failures. `finalResponseAttempted` is a **conservative no-retry guard** set before the first write, not a claim that BYTE_IO commitment occurred if a write failed after an unknown prefix.

**Unresolved publication mismatch:** The product commit at `01535c86` does not change [`spec/io/HTTP.md`](https://github.com/guillermomolina/protos/blob/01535c8609831bcca50da481583190b657242eda/spec/io/HTTP.md). Its §12.2 initial-rejection and handler/`invalidResponse` bullets still say that the original error is returned, while its I/O bullet and §7.6 say write failures are reported as I/O errors. The owner-approved explicit priority must be incorporated in §§7.6 and 12.2 of the **normative** specification before claiming the inconsistency is resolved. The late-framer priority remains as required by §6.2. This documentation file is not normative authority and does not repair the mismatch.

## Coordination and closure disposition

- [D215 #961](https://github.com/guillermomolina/protos/issues/961) was previously owner-ratified, normatively published, and **closed**. No D215 reopening or new semantic decision is implied by this narrow editorial reconciliation.
- [LIB025 #962](https://github.com/guillermomolina/protos/issues/962) has its public 0.3.404 code shipped and maintainer-reported local tests green, but stays **open pending the already-approved normative correction** and verification of the issue's required integrated `make test` gate. Neither should be invented as already done.
- HTTP client, TLS, public listener, untrusted Internet deployment and multi-request keep-alive are not part of the D215 single-request server implementation and are not silently marked completed.

## Exact references

- [LIB025 formal issue #962](https://github.com/guillermomolina/protos/issues/962)
- [D215 closed normative decision #961](https://github.com/guillermomolina/protos/issues/961)
- [0.3.404 product commit](https://github.com/guillermomolina/protos/commit/01535c8609831bcca50da481583190b657242eda)
- [0.3.403 private-message-head predecessor](LIB025_A_MESSAGE_HEAD_PUBLICATION_2026_10_25.md)
- [Original D215 normative ratification evidence](../D215/D215_NORMATIVE_PUBLICATION_2026_10_25.md)
