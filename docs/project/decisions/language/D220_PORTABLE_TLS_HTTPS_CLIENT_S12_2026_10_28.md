# D220 — Portable TLS for HTTPS client: exact S12 owner ratification

**Decision:** D220–S12 / EXACT OWNER-APPROVED CONTRACT / 2026-10-28.
**Owner provenance:** Guillermo Adrián Molina, active conversation 2026-10-28, explicitly:
> APRUEBO D220–S12: el contrato público, de seguridad, confianza, identidad DNS/IP, TLS/ALPN, Future/Byte I/O, cancelación, cierre, errores y alcance diferido presentado en el paquete de decisión D220 del 28-10-2026.

**Owning coordination:** [D220 #975](https://github.com/guillermomolina/protos/issues/975). **Research/scope evidence:** [D220 previous record](../../evidence/D220/D220_TLS_RESEARCH_AND_OWNER_SCOPE_2026_10_28.md).
**Product revision inspected at ratification:** [guillermomolina/protos@7f3113ad5c29a15584053c6440640bff6911e60a](https://github.com/guillermomolina/protos/commit/7f3113ad5c29a15584053c6440640bff6911e60a).
**Status distinction:** The **decision** below is owner selected; this non-normative record is not a product implementation or a normative `spec/` publication. D220 remains open until its normative product-side publication and revision-bound closure checks are complete. No tests or feasibility probes have been executed.

## 1. Exact selected portable frontend (S12)

The optional Standard Library module is `std:tls/Client`. It exports **exactly** `Client`, `Trust`, and `TlsError`; no Prelude binding, public engine interface, registry, provider selection, Network capability, resolver, URL/HTTPS connector, or ambient trust mode. Importing the module alone has no network/TLS-engine effect.

Conceptual public selectors (normal Protos selectors and objects; not a new Core primitive):

```text
import("std:tls/Client")                 // exports Client, Trust, TlsError
Client.handshake(duplex, serverIdentity, trust, options) -> Future<SecureDuplex>
Client.handshake.owning(duplex, serverIdentity, trust, options) -> Future<SecureDuplex>
Trust.anchors(certificates)              // explicit Array of DER-encoded X.509 CA Bytes
Trust.javaCacerts()                      // explicit Java runtime cacerts opt-in
Trust.system()                           // descriptor for OS policy; unavailable in S12
```

`duplex` is an already acquired caller-supplied object with effective callable `read(maxBytes)` and `write(bytes)`. The owning selector additionally demands effective callable `close()` (not inherited root `Object.close`). No TCP, name resolution, authority acquisition, proxy, redirect, HTTP framing, timer or connection pool is supplied. `serverIdentity` is **separate from the network endpoint**: either a DNS-reference `String` or a recognized standard frozen `IpAddress` under `NETWORK.md` §2. The handshake yields a **new authenticated byte duplex only after** TLS version/cipher, chain, identity and ALPN policy succeed; never a pre-verification or plaintext duplex.

`options` is an exact ordinary object (required fields with no delegated fields; a fresh empty options object uses defaults), with only the optional own fields below; unknown own fields and invalid values reject admission. The caller must explicitly supply the `trust` argument (no default trust).

| Field | Admitted values | S12 default |
| --- | --- | --- |
| `minimumVersion` | `"1.2"` or `"1.3"` | `"1.2"` |
| `alpn` | `"http/1.1"` or canonical `null` | `"http/1.1"` |
| `maxHandshakeBytes` | positive 32-bit Integer | `1048576` |
| `maxQueuedWriteBytes` | positive 32-bit Integer | `33554432` |

All option and capability checks, reference-identity syntax checks, trust-descriptor recognition, immutable configuration snapshots, and finite write-retention admission happen **before** TLS byte effects. Bad post-dispatch arguments yield a fresh already-failed **non-Task** Future carrying a fresh `InvalidIOArgument`; they do not acquire ownership, start a Task or touch the source duplex. `Trust` constructors reject malformed descriptors/arguments synchronously under ordinary standard-object Error rules. A valid but unavailable trust source is a TLS operation failure, not a malformed descriptor.

`SecureDuplex` exposes exactly the ordinary `ByteReadable.read(maxBytes)`, `ByteWritable.write(bytes)` and `Closable.close()` semantics, not a new public generic TLS stream hierarchy. It is Actor-local and non-transferable as a live transport. Every successful read answers fresh mutable `Bytes` (1..max) or `null` at authenticated end-of-stream; every successful write answers its receiver. All Futures and operation results obey Core Future/Task rules. No implicit second read/write loop Task is introduced per operation.

## 2. Trust source, endpoint identity and certificate verification

- `Trust.anchors(certificates)` receives an Array of **1..256** DER `Bytes` certificates, each nonempty and at most **65536** bytes, with total encoded input at most **4194304** bytes. Snapshot the Array and every `Bytes` member at construction/admission so later mutations do not alter trust authority. Accept only usable X.509 CA trust anchors, not arbitrary leaf certificates masquerading as anchors. Verify the peer path against **only these** anchors under RFC 5280/PKIX validity, usages, algorithm constraints and ordinary chain rules.
- `Trust.javaCacerts()` explicitly selects the Java runtime/bundled `cacerts` material alone. It never means effective OS trust, never merges application anchors or an OS source, never silently falls back to `SSLContext.getDefault()`, and never implies that a native executable observes post-build CA or distrust-list updates. Resolve/snapshot the selected Java trust material at **each admitted handshake**, before TLS bytes, and fail closed if it cannot be used.
- `Trust.system()` is a valid descriptor, but S12 ships **no Linux, Windows, macOS or BSD trust adapter**. Selecting it returns `TlsError(kind="trustUnavailable")` in the handshake Future at trust-resolution time, **before TLS traffic**; no fallback or implicit union. Other future trust adapters will require distinct private verifier integration and platform-specific evidence.
- TLS cryptography/record engine and certificate trust-source/verifier are separate private components. Path and identity verification must not call a default Java/OS trust manager without the selected explicit authority. No online AIA fetching, implicit OCSP/CRL networking, hidden trust-store union, or security-weakening `insecureSkipVerify` in S12.
- DNS reference: a valid ASCII DNS name supplied as an IDNA2008 A-label by the caller (non-ASCII U-label is **not** silently converted). Normalize ASCII case and one final root dot for comparison; require well-formed nonempty labels and at least two labels. A numeric IPv4/IPv6 literal string is not a DNS identity and is rejected; supply a recognized `IpAddress` instead. No DNS lookup occurs.
- SNI is the normalized DNS reference (without terminal dot); no SNI for an IP reference. Verify the presented certificate's **subjectAltName DNS-ID** against a DNS reference, or byte-exact semantic `subjectAltName iPAddress` against the recognized IPv4/IPv6 `IpAddress`. The `subject CN` is **never** fallback identity.
- DNS wildcard is only one complete `*` in the **entire leftmost label**, matching exactly one nonempty leftmost label; no partial, multi-label, IDNA wildcard or public-suffix-style interpretation that broadens scope. Identity verification is based on RFC 9525 and independent of whatever JSSE `HTTPS` endpoint check happens to accept. Certificate chain trust and reference identity must **both** pass.
- The HTTP `Host` field and the TLS reference identity are independent input authorities. The application must align them with its intended origin; this module does not parse an HTTP request to infer or rewrite either.

## 3. Protocol policy and authenticated I/O lifecycle

- **TLS 1.3 preferred, secure TLS 1.2 allowed** by default; `minimumVersion:"1.3"` is explicit 1.3-only opt-in. Disable TLS 1.0/1.1, SSL, static RSA, CBC, anonymous, export, NULL and unsafe renegotiation. TLS 1.2 allowlist: ECDHE_RSA/ECDHE_ECDSA with AES_128_GCM or AES_256_GCM. TLS 1.3 allowlist: AES_128_GCM_SHA256, AES_256_GCM_SHA384, CHACHA20_POLY1305_SHA256, intersected with available secure provider support; fail if no compatible secure suite.
- `alpn:"http/1.1"` offers `http/1.1` only; an explicit `http/1.1` selection or **no ALPN selection** is accepted for HTTP/1.1, another protocol is rejected as `TlsError("alpn")`. `alpn:null` offers no ALPN. No HTTP/2, HTTP/3 or HTTP protocol parsing within TLS.
- S12 sends **no TLS 1.3 0-RTT/early application data** and performs no TLS session resumption/reuse (no shared session cache or tickets deliberately enabling abbreviated trust bypass). Every connection performs a fresh authenticated handshake. Future resumption requires explicit trust-generation, hostname, identity, session-ticket and cross-Actor policy decision.
- `handshake` returns a fresh Future and creates one structured handshake/service Task only after successful admission. Valid but unavailable system trust/engine fails closed before TLS bytes. Cancellation before first service instruction does not establish ownership or deliver a secure stream. Cancellation after work starts can leave externally observable preparation bytes but cannot yield an unverified duplex; late results and buffers are cleaned up. The admitted handshake's output stays private until all checks succeed.
- A successful wrapper preserves the supplied transport's ordered byte semantics. Individual `read(maxBytes)` and `write(bytes)` return ordinary non-Task-backed operation Futures, are serially ordered per direction and do not create per-operation Tasks. Output write admission snapshots all Bytes before return; finite retention reservation up to `maxQueuedWriteBytes` either admits the entire write or returns a fresh failed capacity Future **without contributing TLS application output or poisoning the stream**. Admitted writes may later fail after an externally committed/unknown plaintext prefix; never claim transactional rollback or automatic replay.
- Ciphertext and records retained by the engine remain bounded (record plaintext max 16384, encrypted record framing/overhead cap 18432, bounded per-stream receive/unwrap buffer 65536 and `maxHandshakeBytes` during handshake). Do not rely on unbounded `ByteReadable.read` requests, unbounded decrypted plaintext queues, or unbounded handshake accumulation. A cancelled read after internal ciphertext consumption must **retain decrypted but unreturned plaintext** for a subsequent read in order, not silently discard or duplicate it. A failed/cancelled write never replays guessed plaintext.
- A valid TLS `close_notify` from the peer authenticates clean read EOF, so reads answer `null` when all earlier plaintext has been returned. Transport EOF without the required authenticated TLS close signal is `TlsError(kind="truncated")`, not clean EOF; prior plaintext remains returnable. TLS protocol alerts/record failures do not become EOF.
- `close()` retires the TLS wrapper idempotently, attempts to emit its own `close_notify` and does **not** wait indefinitely for a peer close alert. Borrowed `handshake` never closes the supplied underlying duplex; `.owning` closes it **exactly once** on every terminal lifecycle path after custody is acquired, including handshake failure/cancellation and wrapper close. Preserve `IO_CORE.md` cleanup scope and failure precedence: already established primary failure wins over cleanup failure; absent primary failure, cleanup failure is observable. Late reads/writes fail according to standard I/O lifecycle rules, never restart TLS.
- Errors from the underlying supplied `ByteReadable`/`ByteWritable` are preserved when they cause the failure; TLS policy/engine-specific failures are fresh ordinary `TlsError` objects, with immediate parent `Error` and required `kind` in exactly:
```text
trustUnavailable, backendUnavailable, untrustedPeer, identityMismatch,
protocolVersion, alpn, handshakeFailure, protocol, limit, truncated
```
No fabricated `HttpError`/`ClientError` or HTTP-status mapping. Invalid admission remains `InvalidIOArgument`; `IOLifecycleError` remains the standard lifecycle failure.

## 4. Protos-first implementation boundary and future extension

Protos owns module selectors, exact admission/options, reference identity, trust selection, lifecycle, ordering, ownership, Future composition, result/error translation and HTTP composition. A privately deferred Actor-local runtime capability supplies the minimal `SSLEngine` lifecycle (`wrap`/`unwrap`/`NEED_TASK`), immutable certificates and PKIX chain-verifier/selected-trust-manager integration that cannot reasonably be implemented in Protos alone. No public JSSE types, default-engine registry, scheduler replacement or speculative second backend. Never leak a live TLS/Socket state across Actors/Contexts.

GraalVM Native Image feasibility is **not presumed**: JCA provider reachability and the supported cryptography sets must be proven for JVM and native variants; the Java default TrustStore and JDK `blacklisted.certs` can be frozen at **image build time**, so an image built before a trust revocation can retain stale authority. `javaCacerts()` must be documented as an explicit build/runtime-provenance selection and must not be represented as OS-live trust. The relevant Native Image path is `build/native/{Dockerfile,Makefile,generate-init-args.sh}` plus the `pom.xml` native profile.

The caller can compose `secure = await(Client.handshake(tcp, identity, trust, options))` with **unchanged** `std:http/Client.Client.exchangeOne(secure, request, onResponse, limits)`; its owning form can take the TLS wrapper's `close`, but the application must choose the borrowed/owning nesting so that the lower TCP is closed once, not twice. `std:http/Client` continues to perform no TLS, TCP, URL or DNS and sends no different HTTP frames.

The TLS engine/private record machinery can later be reused for server TLS, but server X.509 identity/private-key/certificate provisioning, client-auth and mTLS authority are **new** design and implementation work. `std:http/Server.serveOne` may consume a supplied secure server duplex unchanged. `std:http/Host.owning`, which owns listener/acceptor/Worker orchestration, will need an **additive public HTTPS opt-in** and safe Worker credential handoff. Server TLS, `Host` HTTPS, OS trust-policy adapters, additional TLS engines, 0-RTT, resumption, DNS, URL clients and HTTP/2 remain **deferred**, not hidden D220 first-version requirements.

## 5. GITHUB021 invariant and decision delta audit

| Earlier owner-approved invariant | S12 exact selection |
| --- | --- |
| Protos portable frontend and semantics, independent of PLAT defaults | **Preserved**: explicit module, options, admission, errors and identity |
| Small public frontend; minimal private backend | **Preserved**: exact 3 exports, private deferred Actor-local JSSE boundary |
| Only initial GraalVM/JSSE engine | **Preserved**: no runtime/engine registry or second backend |
| Engine separate from trust-source/verifier | **Preserved**: selected Trust descriptors, independent PKIX/identity verification |
| No initial OS CA adapter mandatory | **Preserved**: all OS system trust unavailable and fails closed |
| Explicit app anchors and explicit Java `cacerts` opt-in | **Preserved**: never implicit, never merged |
| No fallback/CA union; `system` unavailable fails closed | **Preserved**: `trustUnavailable` before TLS bytes |
| HTTPS client first; server/Host/mTLS/additional engines deferred | **Preserved**: no server public API in S12 |

**Materially new decisions approved by the exact S12 owner statement:** module/export/selector names, descriptor shapes and bounds, per-handshake trust selection timing, RFC 9525 DNS/IP/SNI/wildcard rules, version/cipher/ALPN defaults, no early data or resumption, handshake Future/Task lifecycle, byte buffer/retention caps, cancellation/EOF/close and error kinds. No earlier approved invariant is reopened or narrowed. **DECISION_INVARIANT_CONSISTENCY=PASS** for the approved candidate, distinct from implementation-test status.

## 6. Governing sources, acceptance and remaining publication

Normative product owners to publish without inventing fresh semantics:
- New `spec/io/TLS.md` or the already established appropriate `spec/io/` TLS owner, and cross-references in `spec/io/{HTTP,NETWORK,IO_CORE,BYTE_IO}.md`;
- `spec/concurrency/{FUTURES_AND_TASKS,ACTORS}.md` for service/operation Future ownership and isolation only where additive text is actually necessary;
- `spec/semantics/{ERRORS,MODULES}.md` for `TlsError` and optional module registration only where additive text is necessary;
- canonical spec navigation/changelog as required by `spec/AGENTS.md`, avoiding duplicated normative owners.

The [D220-S12 acceptance/evidence matrix](../../evidence/D220/D220_S12_APPROVAL_AND_ACCEPTANCE_2026_10_28.md) includes test expectations, two proposed feasibility checks, GITHUB010 comparison and deferred-cost review. Neither proof nor any test is claimed PASS. The implementation scope is tracked separately; its code, normative product publication, exact revision and human-executed checks still need to happen before D220 can be closed.

**State at writing:** DECISION SELECTED / NON-NORMATIVE RATIFICATION RECORD / NO PRODUCT SPEC EDIT / NO CODE / NO COMMANDS / NO TEST RESULTS.