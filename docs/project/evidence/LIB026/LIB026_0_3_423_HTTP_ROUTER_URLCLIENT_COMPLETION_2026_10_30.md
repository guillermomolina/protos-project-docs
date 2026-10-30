# LIB026 — HTTP Router and UrlClient implementation evidence (0.3.423)

**Work item:** [LIB026 #996](https://github.com/guillermomolina/protos/issues/996).  
**Owner-selected architecture:** [D222-C / D222 #986](https://github.com/guillermomolina/protos/issues/986), approved 2026-10-29.  
**Exact product revision:** [guillermomolina/protos@`8b4ccbf43a1d3208137296fccb068534d4e14019`](https://github.com/guillermomolina/protos/commit/8b4ccbf43a1d3208137296fccb068534d4e14019), committed 2026-10-30.  
**Implementation version:** `0.3.423-SNAPSHOT`. **Global specification revision:** `0.1.482`.  
**Record role:** durable, non-normative, revision-bound completion evidence, not live status or a replacement for the product specification.

## Published scope

A single 23-path Protos publication adds two independent opt-in Standard Library modules:

- `std:http/Router` exports `Router.compile(routes)`. An authority-free Protos handler dispatches method and raw target against validated, snapshotted route entries; it supports literal/capture precedence, raw `{params, query}`, 404/405 with `Allow`, explicit OPTIONS routes and HEAD/GET fallback. Duplicate and crossing routes are rejected at compilation.
- `std:http/UrlClient` exports `UrlClient.requestOne(authorities, url, request, limits)`. It composes `std:uri`, explicitly supplied `network`, `resolver` for hostnames, TLS `trust` for HTTPS, and the existing owning HTTP Client. DNS candidates are tried sequentially until a successful TCP connection. The original URL host is retained as TLS reference identity; the response is copied into bounded detached Bytes while the lower callback is live.
- The core HTTP Server/Host/Client contracts remain unchanged; no `std:rest/*` framework, implicit authority, redirects, retry/replay, cookies, automatic JSON, proxying or connection pool is introduced. D225 application-service provisioning remains separate.
- New public sources: `protos/lib/http/{Router,UrlClient}.protos`; four internal `Route*`/`Url*` helpers; two `protos/examples/io/http-*.protos` examples; three Protos HTTP suites; one Java integration class. The Java implementation change only shares the existing private `_httpAsyncFacility` for the two optional modules.
- The same commit publishes the normative `spec/io/HTTP.md` §§17–18, `spec/PROTOS_SPEC_CHANGELOG.md` `0.1.482`, the maintained HTTP guide, Test Tool corpus wiring, and `pom.xml`/`CHANGELOG.md` `0.3.423-SNAPSHOT`.

The [exact product commit](https://github.com/guillermomolina/protos/commit/8b4ccbf43a1d3208137296fccb068534d4e14019) is the authoritative 23-path inventory and change history. Public semantics are owned only by [`spec/io/HTTP.md` at this revision](https://github.com/guillermomolina/protos/blob/8b4ccbf43a1d3208137296fccb068534d4e14019/spec/io/HTTP.md).

## Validation and correction evidence

The maintainer confirmed that the published candidate had a clean `git diff --check`, **`make test` PASS** and **`make check` PASS** locally. These are maintainer-reported results, not independently reproduced by this documentation publication.

The preceding focused validation reported:

- 31/31 Router and target conformance cases and 23/23 UrlClient Protos cases;
- 16/16 Java UrlClient integration tests, including HTTP/HTTPS and TLS identity scenarios;
- 6/6 Test Tool corpus-plan tests, 8/8 HTTP progress-group tests and 70/70 CLI learning-material tests;
- 8/8 lint checks across the six modules and two examples, and clean `git diff --check`.

The first comprehensive `make test` attempt exposed two regression-test integration mismatches rather than a reported HTTP runtime defect: the HTTP progress-group test had a fixed 15-source catalog, and CLI learning-material discovery tried to execute the network-authority-dependent URL example. The final product commit includes the 18-source expectation and returns a value from the embedding-only example rather than calling `print`. The maintainer subsequently reported both complete validation commands passing.

During focused development, the URL fixture's nonexistent `Error.recognizes` was corrected, TLS handshake defaults were passed as `{}` (not `null`), and a regression verifies that rejected TLS admission releases an acquired TCP duplex exactly once. These are incorporated into the one published commit, not separate releases.

## Boundaries and provenance

The approval covers D222-C's optional HTTP composition only; it does not resolve the independent [D225 #992](https://github.com/guillermomolina/protos/issues/992) service-provisioning question or grant access to host Worker capabilities. A live external GitHub request, independently exercised Native Image URL-client behavior, and production-facing internet readiness were **not** claimed by the reported focal tests. The GitHub URL example requires an explicitly authorized embedding; deterministic fixtures underpin automated tests.

**Related D222 ratification evidence:** [D222-C normative publication reconciliation](../D222/D222_C_SPEC_0_1_482_NORMATIVE_PUBLICATION_2026_10_30.md).  
**Closure coordination:** [LIB026 #996](https://github.com/guillermomolina/protos/issues/996) is the live work log.
