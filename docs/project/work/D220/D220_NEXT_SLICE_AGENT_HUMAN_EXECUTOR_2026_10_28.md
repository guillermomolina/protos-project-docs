# D220 — Next slice: exact TLS contract, research agent / human-executor handoff (2026-10-28)

**Work type:** INVESTIGATION / D220 exact-design ratification packet. **No implementation.**
**Owning Issue:** [D220 #975](https://github.com/guillermomolina/protos/issues/975) — OPEN; no new Ixxx Issue yet.
**Read-only source baseline:** [guillermomolina/protos@7f3113ad5c29a15584053c6440640bff6911e60a](https://github.com/guillermomolina/protos/commit/7f3113ad5c29a15584053c6440640bff6911e60a). Re-read the **actual** current GitHub HEAD when invoked, do not assume this SHA is still HEAD.
**Approved initial-scope evidence:** [D220 research and owner scope](../../evidence/D220/D220_TLS_RESEARCH_AND_OWNER_SCOPE_2026_10_28.md).
**Authority:** repository-root `AGENTS.md`, `AGENTS.work/{DESIGN,COORDINATION,REFERENCE}.md` and owning normative specifications, not this non-normative handoff.
**Execution environment:** web and GitHub read-only access only; **run no shell or project commands**.

## Copy/paste prompt — research agent

> You are the D220 investigator, **not** the implementing agent. Work exclusively from authoritative, publicly accessible web sources and the **actual GitHub HEAD** of `guillermomolina/protos` (two “mo” in Guillermo Molina). No local command execution of any kind: do not run shell commands, tests, builds, benchmark scripts, git commands, or product programs. Do not modify product sources, specifications, issue state, GitHub documents, or repository files during this research slice.
>
> Review `AGENTS.md`, `AGENTS.work/{DESIGN,COORDINATION,REFERENCE}.md` and the relevant path `AGENTS.md` files from GitHub. Read the full affected normative owners through segmented GitHub reads where necessary:
> - `spec/io/HTTP.md` (``15–16), `spec/io/NETWORK.md`, `spec/io/IO_CORE.md`, `spec/io/BYTE_IO.md`;
> - `spec/concurrency/FUTURES_AND_TASKS.md`, `spec/concurrency/ACTORS.md`;
> - `spec/semantics/ERRORS.md`, `spec/semantics/MODULES.md`;
> - implementation-side `protos/lib/http/{Client,Host,Server}.protos`, `protos/lib/http/internal/ClientSession.protos`, `src/main/java/com/guillermomolina/protos/execution/facilities/ProtosHttpAsyncFacility.java`, and native-image packaging.
>
> **Already selected invariants; do not re-open without explicit owner approval:**
> 1. Portable Protos TLS frontend/API and security semantics independent of PLAT/Java defaults.
> 2. A minimal private TLS backend boundary, initially **only GraalVM/JSSE SSLEngine**; no public provider registry, auto-loader or speculative second engine.
> 3. Separate engine from selected certificate trust/verifier; no automatic CA union or default fallback from system trust to `cacerts`.
> 4. **First version may have zero OS trust adapters**, meaning no Linux, macOS, Windows or BSD native verifier is an initial deliverable. Supply explicit CA anchors and explicit opt-in Java `cacerts` support; no OS adapter is falsely advertised. If system trust is requested but unavailable, fail closed.
> 5. Client TLS first. Other engines, native OS trust integration, mutual TLS, server credentials/private keys, server TLS and `Host` HTTPS orchestration deferred.
>
> **Existing HTTP boundary:** `Client.exchangeOne(duplex, request, onResponse, limits)` and `.owning` take a supplied readable/writable duplex. No implicit Network/DNS/TLS or URL authority. A secure authenticated TLS duplex should compose with them without changing the HTTP module. `Server.serveOne` could later consume secure server duplex, but `Host.owning` manages its own TCP acceptors/Workers: later server TLS there needs an additive Host public option/operation and key custody design, not free reuse. Keep live TLS and TCP per Actor.
>
> **Your job:** produce the **exact implementable D220 public candidate and non-normative decision packet** for Guillermo's explicit ratification. You must decide nothing on behalf of the owner; label each not-yet-approved semantic choice PROPOSED. Specify:
>
> - Whether the public module is `std:tls`, what exact prototypes and selectors it exports, ordinary object and source-level signatures, what `Trust.system` does before any OS adapter, and whether unavailability appears when the descriptor is constructed or when TLS is negotiated.
> - What `Trust.anchors` takes (PEM/DER Bytes, error timing, snapshots, zero/multiple roots) and exact `Trust.javaCacerts` opt-in semantics. Never imply `javaCacerts` is the system store or that Native Image automatically sees post-build updates.
> - Expected peer identity as separate DNS or IP reference from numeric TCP destination; SNI, RFC 9525 SAN-only identity, IDNA/wildcard/IP policy and RFC 2818 legacy mismatch, and how HTTP Host authority alignment remains the caller's obligation.
> - TLS policy: propose TLS 1.2 secure + TLS 1.3 preferred versus deliberate 1.3-only; modern TLS 1.2 ciphers, ALPN `http/1.1`/no-selection behavior, 0-RTT and session-resumption handling without importing host defaults.
> - Handshake Future timing/admission, no unverified stream delivery, Task scoping, per-Actor/multi-Context isolation, `read(max)`/`write(bytes)` bounds, backpressure and partial-write uncertainty, cancellation after ciphertext consumption, authenticated `close_notify` vs raw TCP EOF, exactly-once cleanup for borrowed and `.owning` wrappers and failure/ensure precedence.
> - Exact error prototype/kind taxonomy and contrast with existing `ClientError`/`HttpError`/`InvalidIOArgument`. Propose the narrowest private JSSE/`SSLEngine`/`X509ExtendedTrustManager` adapter and where Protos logic remains; spell out Native Image reachability/provider and truststore-capture risk.
> - The acceptance test matrix and the small pre-implementation JVM/Native Image feasibility tests to be **executed only by the human in a later implementation slice**. Do not run them here.
> - A GITHUB010 analysis: meaningful alternative candidate set, at least five independent credible systems in three approaches, all twelve 1–5 score dimensions with score/confidence/justification for surviving candidates, anti-overengineering and anti-underengineering gates, pay-now/grow-later/cost-of-deferral, strongest counterargument, and Actor/Context/native-image/server-later future stress.
> - A GITHUB021 invariant/delta matrix matching each explicit approval above; identify all newly introduced choices awaiting approval. Provide precise normative-owner delta map and smallest coherent Ixxx implementation plan **only as a proposed future plan**, grouping a small number of touched files into one slice where practical.
>
> **External facts/source starters (no need to discover repositories outside the Protos HEAD):**
> - [RFC 8446 TLS 1.3](https://www.rfc-editor.org/rfc/rfc8446.html): handshake, close_notify, resumption/PSK.
> - [RFC 9325 TLS deployment guidance](https://www.rfc-editor.org/rfc/rfc9325.html): TLS 1.0/1.1 disabled; support 1.2/1.3 for interoperable application clients; prefer 1.3.
> - [RFC 9525 peer identity](https://www.rfc-editor.org/rfc/rfc9525.html): SAN DNS/IP identification, CN fallback deprecated.
> - [RFC 7301 ALPN](https://www.rfc-editor.org/rfc/rfc7301.html): HTTP protocol selection.
> - [Java SE25 SSLEngine](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/javax/net/ssl/SSLEngine.html) plus [X509ExtendedTrustManager](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/javax/net/ssl/X509ExtendedTrustManager.html): transport-neutral wrap/unwrap with NEED_TASK/NEED_WRAP/NEED_UNWRAP, replaceable trust check; an HTTPS endpoint-identification setting alone must not define Protos RFC9525 semantics.
> - [Native Image certificate management](https://www.graalvm.org/jdk25/reference-manual/native-image/dynamic-features/CertificateManagement/), [JCA services](https://www.graalvm.org/jdk25/reference-manual/native-image/dynamic-features/JCASecurityServices/), [Oct 23 2026 compromised-CA advisory](https://www.graalvm.org/uploads/native-image-advisory-oct-2026-cspu.pdf): built-time trust and blocklist capture, runtime trust override and provider reachability need explicit evaluation; no default-store shortcut.
> - [Go crypto/tls](https://pkg.go.dev/crypto/tls), [tokio-rustls](https://docs.rs/tokio-rustls/latest/tokio_rustls/), [rustls platform verifier](https://docs.rs/rustls-platform-verifier/latest/rustls_platform_verifier/), [.NET SslStream](https://learn.microsoft.com/en-us/dotnet/api/system.net.security.sslstream), [Python asyncio TLS upgrade](https://docs.python.org/3/library/asyncio-stream.html#asyncio.StreamWriter.start_tls), [Node TLS](https://nodejs.org/api/tls.html).
> - Truffle context: [GraalJS](https://www.graalvm.org/latest/reference-manual/js/) (Node TLS as hosting behavior), [GraalPy](https://www.graalvm.org/latest/reference-manual/python/) (Python ssl compatibility), [TruffleRuby](https://www.graalvm.org/latest/reference-manual/ruby/) (OpenSSL/native hosting), [Espresso](https://www.graalvm.org/latest/reference-manual/java-on-truffle/) (Java providers), [Sulong/LLVM](https://github.com/oracle/graal/tree/master/sulong) (native execution substrate, not independent TLS API); SimpleLanguage not materially comparable for TLS and Apple Pkl is only a limited trust-source precedent. Do not adopt their host details as Protos semantics.
> - Future OS trust references, **not first-slice implementation requirements**: [p11-kit](https://p11-glue.github.io/p11-glue/p11-kit/manual/trust-module.html), [Windows chain API](https://learn.microsoft.com/en-us/windows/win32/api/wincrypt/nf-wincrypt-certgetcertificatechain), [Apple SecTrust](https://developer.apple.com/documentation/security/sectrustevaluatewitherror(_:_:)).
>
> **Deliverable:** a reviewable exact candidate and owner-selection packet in English (development-facing content is English); concise Spanish briefing to Guillermo is fine. Mark **RESEARCH ONLY / NO COMMANDS / NO EDITS / NO TEST RESULTS**. Stop for the owner's approval. No Ixxx issue allocation, normative publication, implementing, closing D220 or implying a completed HTTPS client.

## Human executor — this investigation phase

**No human execution task is needed now.** There is no validation command, compile, test, stage, commit, push or checkout for the human to run as part of this investigation. Guillermo reviews the exact proposal and explicitly approves or requests changes.

After **exact** approval: coordinate D220's normative/editorial publication phase under its owning Issue and human-executor rules, then allocate an independent Ixxx **implementation** Issue only when the normative security/API contract is published. The future implementation-agent prompt must assume the local `guillermomolina/protos` checkout is **already at HEAD**, use only that checkout/GitHub, contain any necessary outside technical facts inline, run no external web/repository research, perform edits only, and leave all execution/build/test/commit/push to the human. The implementation-slice grouping should be based on real touched-file and independent-closure criteria, not micro-Issues.

**Current result:** research handoff prepared, **no implementation and no validation claimed**. GITHUB021 approved architecture/zero-OS scope preserved; D220 exact contract remains pending.
