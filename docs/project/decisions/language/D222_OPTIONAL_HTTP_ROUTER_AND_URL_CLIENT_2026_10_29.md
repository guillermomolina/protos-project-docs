# D222-C — Optional HTTP Router and URL Client composition

**Owner decision:** D222-C, expressly approved on 2026-10-29 by Guillermo Adrián Molina: “Apruebo D222-C”.
**Coordination:** [D222 #986](https://github.com/guillermomolina/protos/issues/986).
**Research and comparison:** [GITHUB010/GITHUB021 evidence](../../evidence/D222/D222_C_REST_HTTP_COMPARATIVE_RESEARCH_AND_OWNER_APPROVAL_2026_10_29.md).
**Product baseline considered:** [guillermomolina/protos@`aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9`](https://github.com/guillermomolina/protos/commit/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9), version 0.3.421 / specification 0.1.480.
**Status:** **owner-selected design, NOT normative specification ratification and NOT an implementation**. Both module APIs below are candidate interfaces pending line-by-line normative reconciliation and separate implementation tracking.

## Context and owner intent

The existing Standard Library provides portable HTTP/1.1 message mechanics and a single-request server/client over an explicit Duplex. It does not yet provide application-level routing or a convenient URL-to-response client for interoperability with external HTTP/REST APIs (including GitHub). A web application needs both composition surfaces without changing the deliberately explicit capabilities or ownership of the existing transport.

REST is not a standalone, universally interoperable wire protocol. Ordinary HTTP semantics, resource representations and application conventions drive compatibility; introducing an opinionated `std:rest` framework would impose choices about schema, JSON, authentication and state that are unnecessary for the first use cases. This decision keeps JSON, URI and HTTP orthogonal.

## Owner-selected candidate C

Two optional, separately importable modules under `std:http`, built over the existing explicit lower layers:

1. **`std:http/Router`** — ordinary declarative route entries compiled with a candidate `Router.compile(routes) -> handler` selector. The result is a handler compatible with `std:http/Server` and `std:http/Host`. The design should specify deterministic dispatch by method and path, extraction of path parameters and query data, precedence and ambiguity handling, 404 and 405, and HEAD/OPTIONS policy. No global route registry, ambient network authority, automatic JSON, or dependency-injection framework.
2. **`std:http/UrlClient`** — a bounded convenience facade with a candidate `UrlClient.requestOne(authorities, url, request, limits) -> Future<BufferedResponse>`. Its explicitly supplied authorities include transport `Network`, D223 `Resolver` when host names require it, and TLS `Trust` for HTTPS. URL parsing and HTTP semantics remain governed by `std:uri` and the existing HTTP/TLS contracts. Body collection must be explicitly bounded and complete while the underlying response callback is live; the facade cannot leak a callback-scoped body. No implicit redirects, retries, cookies, pooling, proxy selection, authentication or automatic JSON decoding.

These candidate selectors are **design notation only**, not guaranteed final public API; exact values, names, errors, cancellation and close behavior must be ratified in the product specification before coding.

## Retained owner invariants

- D215/D216: `Server` framing and `Host` worker/handler lifecycle, ownership and safety stay unchanged. The Router produces an ordinary handler and cannot secretly inject services into a Host Worker.
- D218: `Client.exchangeOne` remains an explicit caller-supplied Duplex exchange with response-body lifetime limited to the callback. URL Client composes above that boundary.
- D220: TLS requires caller-granted trust and identity validation. No ambient Trust default. An unavailable `Trust.system()` implementation is not implicitly made available.
- D223: host-name resolution requires explicit Resolver capability; numeric endpoints do not silently require DNS or gain authority through URL parsing.
- Object/capability and Actor transfer rules remain enforced; no hidden network, host service, credentials, global state or automatic cross-Actor object sharing.
- `std:json/JSON` and `std:uri` stay independent. REST-specific application representation decisions remain the application's responsibility.

## Explicit separations and deferrals

- A generic `std:rest` framework, schema-driven services, content negotiation framework, route middleware/dependency-injection system, OpenAPI generators and application authentication are deferred, not incidentally committed.
- Redirect policy, proxying, retry/backoff, cookie jars, connection pools, streaming convenience API and automatic JSON conversion are likewise outside this selection. Independently propose any of these with their own authority and resource contracts if needed.
- **Host Worker application-service access** is a distinct unsolved architectural question, tracked as [D225 #992](https://github.com/guillermomolina/protos/issues/992). Its design must examine explicit transfer/capabilities, ActorRef or brokered services, provisioned per-worker authority and possible deferral; this D222 decision does not prejudge D225.

## Reconciliation, falsification and closure gates

The comparison evidence surveys Go, Rust, Java, Python, JavaScript and .NET HTTP/REST composition patterns with a dimension-by-dimension GITHUB010 selection matrix, adversarial tests and the GITHUB021 invariant/delta check. **Design-level GITHUB021 check: PASS; exact normative reconciliation: pending.**

Before claiming a normative D222-C API:

1. Reconcile the exact public selectors, request/response values, method/path/URI semantics, errors, Future cancellation and resource close behavior against current `spec/io/HTTP.md`, `NETWORK.md`, `TLS.md`, `FUTURES_AND_TASKS.md`, `ACTORS.md` and standard module rules.
2. Prove hard limits and no transfer of `Network`, `Resolver`, `Trust`, Duplex or handler-local resources across Host Actor boundaries.
3. Specify route ambiguity, escaping, method mismatch, HEAD/OPTIONS, content lengths, negative/cancelled requests, DNS/TLS failures, large bodies and native-image compatibility.
4. Publish owner-approved normative specification and changelog on the product repository, then open separate implementation issues and tests. Do not infer implementation from this record.

**Validation of this record:** documentation-only, non-normative; no runtime/test execution claimed. No product repository content was changed as part of this record.
