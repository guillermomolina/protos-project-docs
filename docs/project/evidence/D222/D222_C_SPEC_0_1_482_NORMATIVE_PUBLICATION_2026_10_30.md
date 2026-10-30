# D222-C — Normative publication at specification 0.1.482

**Decision:** [D222 #986](https://github.com/guillermomolina/protos/issues/986), explicit project-owner approval of candidate **D222-C** on 2026-10-29.  
**Original decision:** [optional Router and UrlClient design](../../decisions/language/D222_OPTIONAL_HTTP_ROUTER_AND_URL_CLIENT_2026_10_29.md).  
**Comparison and owner-approval evidence:** [2026-10-29 research packet](D222_C_REST_HTTP_COMPARATIVE_RESEARCH_AND_OWNER_APPROVAL_2026_10_29.md).  
**Exact normative product publication:** [guillermomolina/protos@`8b4ccbf43a1d3208137296fccb068534d4e14019`](https://github.com/guillermomolina/protos/commit/8b4ccbf43a1d3208137296fccb068534d4e14019), 2026-10-30; global spec `0.1.482`, implementation `0.3.423-SNAPSHOT`.  
**Owning implementation:** [LIB026 #996](https://github.com/guillermomolina/protos/issues/996); [revision-bound implementation evidence](../LIB026/LIB026_0_3_423_HTTP_ROUTER_URLCLIENT_COMPLETION_2026_10_30.md).

## Ratification boundary

D222-C's expressly approved choice was the **split, optional, Protos-first HTTP composition**, not a general REST framework or alteration of the underlying public facilities. The single LIB026 publication reconciles that approved architecture into normative [`spec/io/HTTP.md` §17](https://github.com/guillermomolina/protos/blob/8b4ccbf43a1d3208137296fccb068534d4e14019/spec/io/HTTP.md) (`Router`) and §18 (`UrlClient`), with a global [spec changelog `0.1.482`](https://github.com/guillermomolina/protos/blob/8b4ccbf43a1d3208137296fccb068534d4e14019/spec/PROTOS_SPEC_CHANGELOG.md).

The selected public contracts are now `Router.compile(routes) -> handler` and `UrlClient.requestOne(authorities, url, request, limits) -> Future<BufferedResponse>`. `Router` is authority-free and returns an ordinary Server/Host-compatible handler; `UrlClient` requires explicit transport, DNS and TLS authorities as applicable and buffers within the low-level Client callback. Both modules are opt-in. The normative publication makes specific admission, routing, percent-encoding, HEAD/OPTIONS/405, DNS order, Host/SNI identity, buffering, cancellation and ownership rules that the earlier design evidence explicitly left to implementation-time normative reconciliation; the original 2026-10-29 decision record remains a dated snapshot of that stage, not a competing current specification.

## GITHUB021 invariant / delta comparison at published revision

| Earlier ratified boundary | Published D222-C delta | Consistency |
|---|---|---|
| D215 single-request Server: one ordinary handler and Response | §17 compiles an ordinary `handler(request)`; no Server framing or response protocol replacement | PASS |
| D216 / D225-A Host Workers: Actor-local handler provisioning | Router is authority-free and has no Worker service injection; D225 remainder stays independent | PASS |
| D218-B Client: explicit duplex, callback-scoped body and owning close | §18 composes `Client.exchangeOne.owning` and copies all Bytes within its callback | PASS |
| D220 TLS: explicit trust and DNS/IP reference identity | HTTPS requires supplied Trust, uses owning handshake and original URL host/IP as reference (DNS SNI, no IP SNI) | PASS |
| D223 Resolver / NETWORK: explicit DNS authority separate from TCP | Hostnames require supplied Resolver; numeric IPs do not; TCP uses numeric `IpEndpoint` candidates | PASS |
| Existing `std:uri`, `std:json/JSON`, Core/Actor authority rules | URL parsing delegates to `std:uri`; no automatic JSON, implicit credential/network authority or new Core primitive | PASS |

**Decision invariant consistency: PASS** against the *recorded owner-approved architectural invariants*, based on the exact published sections and commit. This comparison does **not** assert that each implementation-time edge rule received an independent, separately quoted owner approval, or replace the source tests and the canonical specification.

## Publication and validation

The same 23-path product commit delivers both optional modules, normative specification, specification changelog, conformance/integration tests, examples, guide and version/changelog. The maintainer reports clean `git diff --check` and local `make test` and `make check` PASS after correcting the two whole-suite discoveries. The normative revision is published; this evidence records it after the fact, without manufacturing a second normative decision or retroactively modifying the 2026-10-29 approval.

**Excluded:** an `std:rest/*` framework, automatic redirect/retry, implicit authority, JSON conventions, or resolution of [D225 #992](https://github.com/guillermomolina/protos/issues/992).
