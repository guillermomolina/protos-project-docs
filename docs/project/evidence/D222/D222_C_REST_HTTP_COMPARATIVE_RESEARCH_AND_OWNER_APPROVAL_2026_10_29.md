# D222-C — Comparative HTTP/REST design investigation and explicit owner approval

**Recorded:** 2026-10-29. **Role:** non-normative, immutable research and approval evidence.
**Issue:** [D222 #986](https://github.com/guillermomolina/protos/issues/986).
**Separate Host-service design:** [D225 #992](https://github.com/guillermomolina/protos/issues/992).
**Exact product snapshot inspected:** [guillermomolina/protos@aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9](https://github.com/guillermomolina/protos/commit/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9) (I132-A, 0.3.421; NETWORK spec revision 0.1.480).
**Decision state:** **OWNER SELECTED D222-C; normative ratification and implementation not yet published**. This file does not define normative Protos semantics.

## 1. Provenance and exact scope

On 2026-10-29, after the comprehensive design review in the active conversation, Guillermo explicitly stated: **“Apruebo D222-C”** and directed coordination/evidence publication in `guillermomolina/protos` and `guillermomolina/protos-project-docs`. This approves the presented **split, optional HTTP composition architecture**, not a full REST framework and not an automatic change to the existing low-level HTTP/Host contract.

The selected direction is:

1. **`std:http/Router`**, an opt-in, authority-free route-table compiler producing an ordinary one-argument handler compatible with the current Server and Host; deterministic method/path semantics, structural match data, 404/405/HEAD handling and safe URI boundaries.
2. **`std:http/UrlClient`**, opt-in URL-aware one-shot HTTP/HTTPS acquisition using **explicit** `Network`, D223 `Resolver` and TLS `Trust`; bounded buffered response; no hidden DNS, trust, credentials, automatic redirect, HTTP replay/retry, pooling, cookies, protocol negotiation or JSON coercion.
3. Keep `std:uri` and `std:json/JSON` independent and leave `std:http/Client`, `std:http/Server`, `std:http/Host` unchanged. **No `std:rest/*` module now.**
4. Defer worker service/capability provisioning to independent **D225**. This approval does not authorize transfer of live Network or Resolver capabilities into Host Workers.

The research packet explicitly compared whole REST frameworks, direct expansion of existing HTTP modules, opt-in split modules, ordinary application composition, and a Router-only first slice. The recommendation was C. The owner statement does not independently authorize implementation, normative edits, closure, or previously unpresented exact API edge choices.

## 2. Current normative and implementation anchors

- [HTTP.md §§1–13,15,16](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/spec/io/HTTP.md): `Server.serveOne` has one `handler(request)`; `Host.owning` creates Workers by `handlerModule` and `handlerBinding`; `Client.exchangeOne` borrows/owns an explicitly supplied duplex and restricts `response.body` lifetime to `onResponse`. Returning a pending Future from the callback does **not** extend the body lifetime. The caller explicitly supplies `Host`, target Bytes and body Bytes. Router and URL acquisition are outside the current normative surface.
- [NETWORK.md §§5,6,11,14](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/spec/io/NETWORK.md): `Network.connectTcp(IpEndpoint)` is numeric-only. D223-A1 adds optional `std:network/Resolvers`, host-granted `resolver.lookupAddresses(name)` and 1–64 frozen `IpAddress` results. Resolver/Network are independent, live, Actor-local capabilities and are not generally Actor transferable.
- [TLS.md §§2–4,8,12](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/spec/io/TLS.md): `Client.handshake(.owning)` accepts an already-connected duplex, explicit trust and DNS/IP reference identity; original hostname remains reference identity and HTTP Host. The `Trust.system()` descriptor has no actual OS adapter in this release; explicit `Trust.javaCacerts()` or anchors are available as selected sources.
- [IO_CORE.md](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/spec/io/IO_CORE.md), [BYTE_IO.md](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/spec/io/BYTE_IO.md), [FUTURES_AND_TASKS.md](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/spec/concurrency/FUTURES_AND_TASKS.md), [ACTORS.md](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/spec/concurrency/ACTORS.md), [MODULES.md](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/spec/semantics/MODULES.md) and [ERRORS.md](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/spec/semantics/ERRORS.md): I/O commitment, cancellation, actor-local module instances and Error/ensure precedence remain unmodified.
- Actual Protos source: [Client.protos](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/protos/lib/http/Client.protos), [Server.protos](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/protos/lib/http/Server.protos), [Host.protos](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/protos/lib/http/Host.protos), [HostWorker.protos](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/protos/lib/http/internal/HostWorker.protos), [Resolvers.protos](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/protos/lib/network/Resolvers.protos), [uri.protos](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/protos/lib/uri.protos) (`parse/format/resolve` generic RFC3986 URI references), [JSON.protos](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/protos/lib/json/JSON.protos) (JSON node trees, `parse/encode` and events), and [http-get.protos](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/protos/examples/io/http-get.protos) (read body within callback).
- The approved low-level HTTP and TLS design histories remain [D215 #961](https://github.com/guillermomolina/protos/issues/961), [D216 #964](https://github.com/guillermomolina/protos/issues/964), [D218 #969](https://github.com/guillermomolina/protos/issues/969) and [D220 #975](https://github.com/guillermomolina/protos/issues/975).

## 3. Primary-source systems survey

The survey is six ecosystems across at least three architectural families; the cited documentation is normative/first-party or first-party project documentation, not secondary articles.

| System | Relevant primary sources | Observed boundary and significance |
|---|---|---|
| Go | [net/http](https://pkg.go.dev/net/http), [Go 1.22 routing](https://go.dev/doc/go1.22) | Standard HTTP client and server with standalone ServeMux; method/path wildcards and PathValue introduced in Go 1.22. Demonstrates incremental router growth but also a concrete migration/change-of-pattern-semantics risk. |
| Rust | [axum Router](https://docs.rs/axum/latest/axum/routing/struct.Router.html), [reqwest Client](https://docs.rs/reqwest/latest/reqwest/struct.Client.html) | HTTP services, router and URL client are distinct abstractions; route overlap checking and state/layer composition demonstrate a usable opt-in split. Reqwest also highlights that pooling, redirects and proxy policies are separate substantial choices. |
| Java / Jakarta REST | [Jakarta RESTful WS 4.0](https://jakarta.ee/specifications/restful-ws/4.0/jakarta-restful-ws-spec-4.0.html) | Annotation-based resource classes, URI templates, argument/provider conversion and client API provide a mature *framework* precedent. Its breadth is not proof that Protos needs the same public categories. |
| Python | [Starlette routing](https://starlette.dev/routing/), [HTTPX quickstart](https://www.python-httpx.org/quickstart/) | Lightweight ASGI routing and a separate general-purpose HTTP client; body and query conveniences can be layered. |
| JavaScript | [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch), [Express routing](https://expressjs.com/en/guide/routing/) | General HTTP client and optional app routing/framework, without shared REST wire protocol. Ordinary HTTP response statuses need not be transport errors. |
| .NET | [Minimal APIs routing](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/route-handlers), [HttpClient lifetime guidelines](https://learn.microsoft.com/en-us/dotnet/fundamentals/networking/http/httpclient-guidelines) | Integrated lightweight routes with optional parameter binding/DI, independently managed URL HTTP client; DNS TTL/pooling demonstrate complexity that should not be implicitly bundled today. |

Standards and adversarial evidence: [Fielding REST architectural constraints](https://ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm), [RFC 9110 HTTP semantics](https://www.rfc-editor.org/rfc/rfc9110), [RFC 9112 HTTP/1.1](https://www.rfc-editor.org/rfc/rfc9112), [RFC 3986 URI syntax/escaping](https://www.rfc-editor.org/rfc/rfc3986), [RFC 9457 optional API problem details](https://www.rfc-editor.org/rfc/rfc9457), [OWASP SSRF prevention](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html) and [GitHub REST request contract](https://docs.github.com/en/rest/using-the-rest-api/getting-started-with-the-rest-api). REST is an architectural style, not another mutually exclusive HTTP wire protocol or a JSON mandate.

## 4. Alternatives and complete GITHUB010 scoring

Candidates: **A** integral `std:rest/*` framework; **B** extend current `std:http/{Client,Server,Host}` directly; **C** optional independent `std:http/Router` and `std:http/UrlClient`; **D** no library, application-only composition; **E** Router-only now, URL client deferred. Ratings are **1 (weak) to 5 (strong)**. Confidence H=HIGH, M=MEDIUM, L=LOW; every cell includes a concise reason. These are qualitative aids, not an arithmetic selection.

| Mandatory dimension | A REST framework | B modify existing | C split HTTP modules | D application-only | E Router only |
| --- | --- | --- | --- | --- | --- |
| 1. Correctness/invariants | 3M many contracts | 3M existing APIs change | **5H preserves baselines** | 4H ordinary composition | 5H isolated change |
| 2. Protos alignment | 2H imposed institution | 3M mixed duties | **5H ordinary modules** | 5H no new privilege | 5H small mechanism |
| 3. Present need/pay-for-use | 1H speculative stack | 3M shared surface | **4M opt-in** | 5H no new runtime | 5H one feature |
| 4. Incremental growth | 3M broad framework | 2M API entanglement | **5M independent additions** | 3M divergent apps | 4M future client possible |
| 5. Future options | 3M policy coupling | 2M hard extension | **5M backend independent** | 4M no lock-in | 4M URL choice deferred |
| 6. Scalability | 4L mature pattern | 3L requires rework | **4L local state** | 2L repetitive control | 3L server-only |
| 7. Conceptual simplicity | 2H many concepts | 3M overloaded APIs | **4M two responsibilities** | 4M library simple | 5M one addition |
| 8. Portability | 3M broad semantics | 3M transport coupling | **5M existing protocols** | 4M per-app behavior | 5M pure routing |
| 9. Runtime/resource cost | 2L framework layers | 3L wider base | **4L only on import** | 5M no base cost | 5M local-only |
| 10. Failure/operability | 3M hidden policy risk | 3M mixed errors | **4M error identity preserved** | 2M repeated cleanup | 4M deterministic dispatch |
| 11. Deferral/reversibility | 2M difficult removal | 2M hard migration | **5M additive modules** | 2M duplicated clients | 3M user need unmet |
| 12. Evidence/risk | 4M mature precedents | 4M common pattern | **4M several analogues** | 5H works presently | 4M bounded work |

**Non-compensating gates:** A fails proportionality (first permanent REST-framework institution without evidence); B risks semantic conflict with D215/D216/D218; D creates repeated security/cleanup decisions in each app. E is the strongest credible alternative but does not meet the already-demonstrated GitHub URL-client use case. C still has genuine implementation/portable-backend risk, especially large-response buffering and connection failures, and must remain opt-in.

**Smallest sufficient:** one independent router and one explicitly-authorized one-shot URL client. Adding persistent pooling, HTTP2/3, streaming requests, middleware, JSON conversion, automatic redirects, cookie containers, schema/DI or a full REST SDK has no proven present requirement.

**Concrete deferral cost:** Router deferral duplicates route/path/405/HEAD policies; URL client deferral duplicates DNS/IP authorization, TLS identity/trust, URL/Host/target construction, callback-scoped body reading and mandatory closure across apps. Both can technically be added later without changing Core, so present need—not an invented foundational rewrite—is their justification. Deferring advanced policies merely requires additive library layers; implementing them now charges current users for speculative semantics.

**Strongest counterargument to C:** two public modules add maintenance and a URL client may duplicate delicate asynchronous cleanup and limits. If tests falsify its feasibility/portability, E (Router now, URL client later) remains the escape path without disturbing D218.

## 5. Falsification, threats and future stress

- **Router:** overlapping `/a/{x}` and `/{y}/b`; literal priority `/users/me` over `/users/{id}`; HEAD over GET; RFC 9110 405 with mandatory Allow; OPTIONS *; absolute-form origin target under RFC 9112; `%2F`, `%2E`, double escapes, trailing slash, query duplicates and no lossy query decoding. The application decides content type and representation; no automatic JSON.
- **URL client:** URL origin, `Host` and SNI always aligned with original DNS name; each numeric candidate independently authorized by Network; no unchecked second resolution or transport-authority amplification; IDNA A-label vs U-label and numeric IP reference rules from D223/TLS; forbid userinfo-based credentials; no automatic redirect or sensitive-header replay; do not conflate HTTP 4xx/5xx with DNS/TLS/I/O Errors.
- **Future/ownership:** cancelled DNS/TCP/TLS/HTTP leaves no late connection without custody; Client's body read occurs inside `onResponse`, never after retirement; buffered `Bytes` is a bounded owned snapshot; cleanup follows existing Error precedence; no request replay after uncertain write. Test failure and cleanup failure identities.
- **Scalability:** per-Worker route state scales with worker count; bounded response body per outstanding exchange; no import-time state/worker/cache; no hidden timers, proxies or global pools. A 10k-request load must not allocate unbounded body copies; quota/limit policy remains explicit.
- **Portability/future:** native image, JVM, alternate runtimes, several Actors/Contexts, HTTP/2/3 and long-lived clients are future scenarios. Avoid exposing HTTP/1.1 framing in the high-level response. Native SSL trust material, high concurrency and slowloris protections require independent evidence.
- **Deployment:** current D216 Host is suitable only for trusted/protected ingress; no portable timeout contract, no direct-hostile-Internet readiness claim. HTTPS hosting is not selected here. Its Workers do not receive Network or Resolver authority; D225 explicitly owns that separate integration question.

## 6. GITHUB021 owner-invariant/delta consistency check

| Approved invariant | D222-C impact | Result |
|---|---|---|
| D215 one ordinary `handler(request)`, Response data, no implicit Network | Router returns a compatible one-argument handler; route callbacks are Router-internal | PRESERVED |
| D216 Host module/binding import inside each Worker; bounded accept-only delegation; no implicit credentials | Router is constructed in each Worker; no Host bootstrap change in D222 | PRESERVED; separate D225 |
| D218 `Client.exchangeOne` receives duplex, body is callback-local; no URL acquisition | UrlClient is a separate opt-in wrapper and copies the body *within* the callback | PRESERVED |
| D220 TLS reference identity/trust are explicit and separate from address/HTTP Host | UrlClient takes explicit trust and original hostname; no `Trust.system` default | PRESERVED |
| D223 Network numeric-only and Resolver host-granted, independent authority | UrlClient takes supplied capabilities, resolves then connects by numeric `IpEndpoint` | PRESERVED |
| IO/Core Future/Actor/Module isolation; no silent replay | New wrappers must use existing operation Future/Task/cleanup semantics | PRESERVED AT ARCHITECTURAL LEVEL; normative verification pending |

**Check status:** **PASS for approved architectural scope; not a claim that all detailed future API clauses are normative or tested.** Exactly which fallback-connection error wins, explicit route URI/absolute-form validation, URL/client limit defaults and any novel cross-Actor service provision must be resolved during normative editing and, if substantively new, returned to Guillermo for explicit approval. No previous approved invariant is silently reopened here.

## 7. Outcome, separation and validation

**KEEP:** original HTTP/TLS/Network/Resolver/URI/JSON contracts.
**ADD (owner selected, not yet published):** opt-in Router and UrlClient.
**DEFER:** framework REST, streaming, redirects/retries/pooling, HTTP2/3, cookies/auth, JSON coercion, middleware/DI and timeouts.
**SEPARATE:** [D225 #992](https://github.com/guillermomolina/protos/issues/992), Host Worker application-service access, with its own GITHUB010 approval gate.

**Normative owner after approval:** primarily `spec/io/HTTP.md`, with only genuinely necessary coherence changes in `NETWORK.md`, `TLS.md` and relevant module/URI owners; `spec/PROTOS_SPEC_CHANGELOG.md` must advance only alongside a real published Protos specification revision. Implementation issue(s) follow normative reconciliation, not this evidence publication.

**Test matrix for later implementation:** route collisions/precedence, path/query escapes, 404/405/HEAD/OPTIONS, absolute targets, request/response streaming ownership, cancelled resolver/connect/TLS/HTTP, TLS mismatch, public/private IP policy, late callback/resource cleanup, native-image, local test origins, explicit GitHub-compatible headers and bounded large bodies. The final test list and commands belong to the implementation slice and human-executor gate.

**No code, specification edit, execution, testing, or source publication was performed to create this research evidence.** This evidence describes reviewed normative files and source and the explicit owner approval; it is not proof of an implemented Router or UrlClient.