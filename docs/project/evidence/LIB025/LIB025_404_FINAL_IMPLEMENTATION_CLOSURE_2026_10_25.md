# LIB025 — Final closure of the Protos HTTP/1.1 single-request server (0.3.404)

Date: 2026-10-25. Durable non-normative closure evidence for [LIB025 #962](https://github.com/guillermomolina/protos/issues/962), following the published 0.3.404 server and D215 specification 0.1.472 error-precedence reconciliation. The user/project owner has explicitly confirmed that **both `make test` and `make check` passed**. No commands or tests are run by this documentation agent.

## Revision-bound closure identity

```text
WORK_ITEM=LIB025
PRODUCT_REPOSITORY=guillermomolina/protos
IMPLEMENTATION_REVISION=01535c8609831bcca50da481583190b657242eda
IMPLEMENTATION_VERSION=0.3.404-SNAPSHOT
NORMATIVE_CORRECTION_REVISION=205420a03d4ebb1c5c125ca891c04147bd0c752c
GLOBAL_SPEC_REVISION=0.1.472
INITIAL_D215_SPEC_REVISION=9dc4e50875252508e2afecf91541fb50f756d450
TEST_GATE=make test PASS (maintainer-confirmed)
CHECK_GATE=make check PASS (maintainer-confirmed)
DIFF_CHECK=PASS (maintainer-confirmed for LIB025 implementation)
VALIDATION_LOGS=NOT_SUPPLIED
HTTP_GUEST_TEST_CASES=247
HTTP_PROGRESS_GROUPS=7
PUBLIC_SERVER_API=std:http/Server
CLOSURE_DISPOSITION=COMPLETED
```

The exact [LIB025 implementation commit `01535c86`](https://github.com/guillermomolina/protos/commit/01535c8609831bcca50da481583190b657242eda) publishes a Protos-first HTTP/1.1 one-request server. It includes `Server.serveOne` (borrowed) and `Server.serveOne.owning` (owning), exactly the module members `Server` and `HttpError`, bounded request parsing and body framing, response streaming, session retirement and a private, minimal Future/Task host facility. The ordinary runtime does not acquire a listener or Network authority. No additional HTTP/2, keep-alive, TLS, multi-request, or public-hostile-ingress capability is claimed.

The seven registered HTTP guest sources contain 52 request-head, 29 response-head, 77 request-body, 22 server-admission, 24 server-request, 25 server-response and 18 server-lifecycle Tests: **247 cases total**. Test Tool progress reports those seven groups without changing any `CaseRef`, `SuiteId`, `CorpusId`, suite-graph leaf, Case identity or scheduling behavior. Java integration and structural tests accompany the guest cases.

## Normative alignment

The initial [D215 normative contract at `9dc4e508`](https://github.com/guillermomolina/protos/commit/9dc4e50875252508e2afecf91541fb50f756d450) established specification 0.1.471. The owner subsequently ratified the three-way precedence of failed best-effort error-response writes; the clarification is published in [spec 0.1.472 at `205420a0`](https://github.com/guillermomolina/protos/commit/205420a03d4ebb1c5c125ca891c04147bd0c752c), changing exactly `spec/io/HTTP.md` and `spec/PROTOS_SPEC_CHANGELOG.md`:

- For initial request rejections and attempted optional handler/`invalidResponse` 500, a failed best-effort write's I/O Error prevails.
- For a late framer failure under §6.2, the original `HttpError` prevails even if the best-effort 400/413 write fails.
- Ordinary `ensure` cleanup-error precedence, including owning `close()`, remains unchanged. A conservative guard against a second final response does not redefine physical commitment under `BYTE_IO.md` §6.2.

No runtime changes were needed in the normative correction; the 0.3.404 implementation already followed this owner-approved rule. D215 #961 was previously ratified, documented and closed; no new semantic decision or reopening is implied by the clarification.

## Human validation provenance and closure conclusion

The maintainer reported after the 0.3.404 product publication that all local tests passed and `git diff --check` was clean. In a later explicit response to the exact acceptance-gate question, the maintainer confirmed **yes, both `make test` and `make check` had passed**. This is the project's authorized human-executor summary of the integrated gate; no full console transcript, per-test log, independent agent test execution or second post-version-bump run is claimed.

Therefore the original [LIB025 #962](https://github.com/guillermomolina/protos/issues/962) full public-server acceptance, normative error-precedence reconciliation, revision-pinned evidence and integrated human validation are complete. Formal closure as `completed` is now justified; future HTTP client, TLS, listening-server composition and multi-request hosting are outside this Issue rather than hidden unfinished slices.

## Historical evidence retained

- [LIB025-A private message-head publication](LIB025_A_MESSAGE_HEAD_PUBLICATION_2026_10_25.md)
- [LIB025 0.3.404 publication checkpoint (before normative fix)](LIB025_404_SERVER_PUBLICATION_AND_CLOSURE_GATE_2026_10_25.md)
- [Spec 0.1.472 normative precedence reconciliation](LIB025_D215_0_1_472_NORMATIVE_RECONCILIATION_2026_10_25.md)
