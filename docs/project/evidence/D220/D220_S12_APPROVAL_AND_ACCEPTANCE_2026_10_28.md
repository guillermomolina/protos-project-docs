# D220–S12 — Approval provenance, GITHUB010 comparison and acceptance evidence

**Date:** 2026-10-28. **Kind:** non-normative approval and planned-conformance evidence, **not** a validation report.
**Live decision:** [D220 #975](https://github.com/guillermomolina/protos/issues/975).
**Product HEAD inspected:** [7f3113ad5c29a15584053c6440640bff6911e60a](https://github.com/guillermomolina/protos/commit/7f3113ad5c29a15584053c6440640bff6911e60a).
**Earlier evidence:** [D220 research and owner-scope](D220_TLS_RESEARCH_AND_OWNER_SCOPE_2026_10_28.md).
**Ratified exact contract:** [D220–S12 decision](../../decisions/language/D220_PORTABLE_TLS_HTTPS_CLIENT_S12_2026_10_28.md).

## 1. Auditable exact owner selection

Earlier approval, preserved here by reference, selected the portable frontend/private JSSE engine, distinct verifier, no speculative public registry, and a **zero-OS-adapter** initial scope: supplied CA anchors and explicitly selected Java `cacerts`; system trust unavailable with no fallback or union.

The project owner subsequently selected the exact S12 candidate in the active interaction on 2026-10-28:

> APRUEBO D220–S12: el contrato público, de seguridad, confianza, identidad DNS/IP, TLS/ALPN, Future/Byte I/O, cancelación, cierre, errores y alcance diferido presentado en el paquete de decisión D220 del 28-10-2026.

The candidate compared against every earlier invariant under GITHUB021 and introduced no conflict. The exact contract's *new* selections (names, options, limits, identity/ALPN/security policy, Future/Byte lifecycle and error kinds) are itemized in the decision record. This approval establishes design authority but **does not** evidence JVM/native feasibility, product normative publication, implementation, tests or shipping HTTPS.

## 2. GITHUB010 cross-system models and trade-offs

Primary sources, deliberately covering at least six independent systems across **three models**:

| Model | Primary precedent | Relevant result and limit of analogy |
| --- | --- | --- |
| Portable TLS session over owned/borrowed stream | [Go crypto/tls](https://pkg.go.dev/crypto/tls) | Explicit config/ServerName/root CAs and connection-level security; Go's global/system roots are not Protos authority. |
| Asynchronous TLS byte duplex | [Rust tokio-rustls](https://docs.rs/tokio-rustls/latest/tokio_rustls/) | Async stream wrapping and application/backpressure integration; no automatic portable OS trust policy. |
| Runtime SSL engine + trust manager | [OpenJDK SSLEngine](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/javax/net/ssl/SSLEngine.html) and [X509ExtendedTrustManager](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/javax/net/ssl/X509ExtendedTrustManager.html) | Transport-neutral engine, handshake statuses and provider-managed certificate checks; host defaults must not define Protos semantics. |
| Runtime SSL stream | [.NET SslStream](https://learn.microsoft.com/en-us/dotnet/api/system.net.security.sslstream) | Authenticated stream with explicit client target and options; cancellation, identity and OS trust defaults differ. |
| Existing language/runtime stream upgrade | [Python asyncio start_tls](https://docs.python.org/3/library/asyncio-stream.html#asyncio.StreamWriter.start_tls) | TLS wraps asynchronous stream transport; Python's SSL context and trust policy are not portable Protos commitments. |
| Event-loop TLSSocket | [Node TLS](https://nodejs.org/api/tls.html) | SNI/ALPN/trust and transport event integration; Node/OpenSSL runtime hosting is not a mandatory public Protos abstraction. |

Truffle host-context comparisons: [GraalJS](https://www.graalvm.org/latest/reference-manual/js/) exposes Node/JS hosting patterns when applicable; [GraalPy](https://www.graalvm.org/latest/reference-manual/python/) maps Python SSL APIs onto its host; [TruffleRuby](https://www.graalvm.org/latest/reference-manual/ruby/) uses Ruby/OpenSSL integration, not a cross-language Protos protocol; [Espresso](https://www.graalvm.org/latest/reference-manual/java-on-truffle/) can expose Java crypto hosting without changing its language's TLS contract; [Sulong/LLVM](https://github.com/oracle/graal/tree/master/sulong) is a native execution substrate, **not** an independent TLS security API. These are feasibility/architecture precedents, not permission for a host-default trust mode. Minimal initial engine and no global shared Context resources follow from Protos Actor locality.

Live OS trust policy is distinct from an exported certificate bundle: [p11-kit](https://p11-glue.github.io/p11-glue/p11-kit/manual/trust-module.html), [Windows CertGetCertificateChain](https://learn.microsoft.com/en-us/windows/win32/api/wincrypt/nf-wincrypt-certgetcertificatechain), and [Apple SecTrust](https://developer.apple.com/documentation/security/sectrustevaluatewitherror(_:_:)) would need future native adapters if actually required. No adapter is required by S12.

### Surviving alternatives

- **S12 — selected:** explicit anchors/`javaCacerts`, minimal JSSE engine, secure TLS 1.2+1.3, no initial OS adapter.
- **S13 — credible but rejected for now:** same frontend/trust machinery, TLS 1.3 *only* by default, losing interoperability for compliant TLS 1.2-only servers.
- **L12 — credible but deferred:** S12 plus validated Linux system trust-policy adapter now; adds policy/distrust fidelity, FFI and Native Image platform risk before present need.

Other investigated models rejected: `SSLContext.getDefault()` plus ambient trust (violates invariants); OS PEM bundle labeled effective OS trust (ignores distrust/purpose rules); Java's HTTP client instead of TLS duplex (duplicates Protos HTTP); all OS engines in initial release (speculative framework); own TLS/X.509 cryptography (unacceptable complexity/security risk); doing nothing (does not enable HTTPS).

12 dimensions, score **1 weak–5 strong**, H=high or M=medium confidence. Each cell gives its justification:

| Dimension | S12 | S13 | L12 |
| --- | --- | --- | --- |
| 1 Correctness/invariants | **5/H** explicit verified policy | **5/H** same constraints | **4/M** Linux trust/distrust fidelity uncertain |
| 2 Protos philosophy | **5/H** small frontend | **5/H** same frontend | **4/M** OS bridge adds policy machinery |
| 3 Pay only for current need | **5/H** HTTPS secure minimum | **5/H** fewer legacy suites | **2/H** unneeded Linux integration |
| 4 Incremental growth | **5/M** isolated seam | **5/M** TLS1.2 could be reintroduced | **4/M** later OS ports remain |
| 5 Future option resilience | **5/M** separate trust and engine | **5/M** same boundary | **5/M** private verifier remains |
| 6 Scalability | **4/M** per-Actor bounded state | **4/M** same loop costs | **3/M** verifier adapter can block |
| 7 Conceptual simplicity | **5/H** two opt-in sources | **5/H** one protocol family | **3/M** Linux store/distrust details |
| 8 Portability | **5/H** cross-OS semantics | **4/M** TLS1.2 peers excluded | **4/M** Linux-only adapter to maintain |
| 9 Runtime cost | **4/M** bounded engine; no-use lazy | **5/M** simpler suite negotiation | **3/M** extra native policy path |
| 10 Failure/operability | **4/M** explicit trustUnavailable | **4/M** clear version failures | **3/M** opaque OS trust errors |
| 11 Deferral/reversibility | **4/M** private OS seam preserved | **4/M** can relax min version | **4/M** private engine preserved |
| 12 Evidence maturity/risk | **3/M** Protos Future/Native Image unproven | **2/M** adds interoperability risk | **2/M** policy FFI unproven |
| **Total (non-compensating)** | **54** | **53** | **41** |

**Overengineering veto:** implementing Linux, Windows, macOS, BSD or a generic engine registry before any mandatory first-client behavior fails the pay-now gate. **Underengineering veto:** an unspecified trust verifier, silent fallback, missing retention/EOF guarantees or accepting TLS without identity verification would invalidate S12 even with attractive scores. **Deferral cost:** explicit public-site connections need anchors or Java cacerts configuration; a later OS verifier adds native policy source and tests behind the existing seam, not new public TLS semantics. Server-later work adds credentials/private key custody and Host Worker provisioners. **Strongest objection:** no easy automatic OS-system-trust HTTPS client and potentially stale native-image Java anchors; document honestly and only add validated adapters when demanded.

## 3. Planned acceptance matrix — all PENDING

| Area | Acceptance test, to run in later implementation | Present evidence |
| --- | --- | --- |
| Module | Only Client, Trust, TlsError exports; import no ambient Network or TLS init | NOT RUN |
| Admission | Bad duplex/closure/options/identity/Trust → fresh failed non-Task InvalidIOArgument Future, no bytes/custody | NOT RUN |
| Anchors | 1..256 DER, 64KiB each, 4MiB aggregate; snapshot mutation cannot change policy | NOT RUN |
| Trust | Valid app CA succeeds, untrusted root/invalid CA/expired certificate fails | NOT RUN |
| Java CA | Explicit javaCacerts only, no OS merge/default/fallback | NOT RUN |
| System CA | Trust.system fails trustUnavailable before TLS bytes on all initial OSes | NOT RUN |
| DNS | DNS SAN-only, no CN fallback; ASCII/IDNA2008 validation and terminal dot/case matching | NOT RUN |
| Wildcard | Only complete leftmost single-label wildcard accepted | NOT RUN |
| IP | Recognized IpAddress and exact SAN iPAddress, no IP-literal String or IP SNI | NOT RUN |
| SNI | DNS SNI exact normalized identity; IP without SNI | NOT RUN |
| Versions | TLS1.3 preferred; secure TLS1.2 works; 1.0/1.1/static RSA/CBC disallowed; 1.3-only rejects 1.2 | NOT RUN |
| ALPN | http/1.1 or absent accepted, incompatible protocol fails; null offers none | NOT RUN |
| Replay | No early data, no resumption/session cross-Actor reuse | NOT RUN |
| Handshake | Encrypted fragmented records, NEED_TASK/WRAP/UNWRAP, no secure result before authentication | NOT RUN |
| Backpressure | Finite handshake and write retention, pre-admission capacity failure leaves stream usable | NOT RUN |
| Byte read | Ordered fresh Bytes, no empty successful read; cancellation retains read-ahead plaintext | NOT RUN |
| Byte write | Exact input snapshots despite caller mutation; committed-prefix uncertainty on failure | NOT RUN |
| EOF | Authenticated close_notify → null; raw EOF/alert truncation → TlsError truncated/protocol | NOT RUN |
| Lifecycle | close_notify emission; borrowed never closes TCP; owning exactly once on all paths | NOT RUN |
| Errors | TlsError kind taxonomy and lower transport Error/failure precedence | NOT RUN |
| Actors | Deferred import, isolated per-Actor engine, no live Context/Actor transfer | NOT RUN |
| HTTP | Unchanged Client.exchangeOne[.owning] uses authenticated wrapper over scripted and real duplex | NOT RUN |
| Native | JVM and Native Image provider/PKIX/CA capture correctness incl frozen blocklist warning | NOT RUN |
| Future Host | No Host/server public behavior changed, deferred extension documented | NOT RUN |

**Proposed proof V1 (JVM, not executed):** use a local in-memory or loopback duplex with deliberately fragmented TLS records and test-only CA/certificate to demonstrate SSLEngine NEED_TASK/NEED_WRAP/NEED_UNWRAP progress, trust-manager separation, DNS/IP SAN mismatch, ALPN, TLS1.2+1.3 and clean close_notify versus unnotified transport EOF. No public HTTP/TLS semantics may be weakened to make the proof pass.

**Proposed proof V2 (Native Image, not executed):** build and run a throwaway targeted fixture under the **existing** native packaging path, with one custom CA, a deliberately rejected CA and a Java cacerts selection. Verify provider/suite availability, trust-manager reachability, and distinguish image-build-time default trust/blacklist capture from controlled runtime anchors. Do not claim system trust freshness; record exact build JDK/GraalVM, configuration and failures. See [GraalVM certificate management](https://www.graalvm.org/jdk25/reference-manual/native-image/dynamic-features/CertificateManagement/) and [JCA services](https://www.graalvm.org/jdk25/reference-manual/native-image/dynamic-features/JCASecurityServices/); the [October 2026 advisory PDF](https://www.graalvm.org/uploads/native-image-advisory-oct-2026-cspu.pdf) requires cautious provenance review but is not represented here as an independently reproduced exploit result.

## 4. Revision-bound governance state

- GITHUB021 invariant/delta check: **PASS for the exact approved decision**, based on source history and owner statement; no reopened invariant.
- GITHUB010 comparative requirement: **documented and assessed**, not empirical conformance.
- Required future normative owners: `spec/io/{TLS,HTTP,NETWORK,IO_CORE,BYTE_IO}.md`, `spec/concurrency/{FUTURES_AND_TASKS,ACTORS}.md`, `spec/semantics/{ERRORS,MODULES}.md`.
- Pending: product-side normative publication with exact `PROTOS_REVISION`, implementation issue execution, JVM/Native Image proofs, human-run tests and release validation. Do not close D220 or assert HTTPS is implemented until its particular normative publication/closure conditions are met.

**RESEARCH EVIDENCE + OWNER APPROVAL / NO COMMANDS / NO TEST RESULTS / NO PRODUCT CHANGES.**

## 5. Second exact owner approval — two error-kind mappings (2026-10-28)

Guillermo explicitly approved in the active I130-A Human Executor interaction:

> Apruebo `trustUnavailable` para `javaCacerts` no utilizable y `backendUnavailable` para ausencia local de configuración TLS compatible

The [D220–S12 decision record](../../decisions/language/D220_PORTABLE_TLS_HTTPS_CLIENT_S12_2026_10_28.md) now specifies precise early, fail-closed mappings: (a) selected Java `cacerts` missing/unreadable/empty/no usable anchors → `trustUnavailable` in handshake Future before TLS traffic, with no fallback or implicit union; available anchors that reject the peer → `untrustedPeer`; (b) local backend cannot supply an approved compatible protocol/cipher configuration → `backendUnavailable` before TLS bytes; remote version/ALPN negotiation refusal without conclusive evidence → general `handshakeFailure`, not guessed specialized kinds. This introduces no new public kind and changes no GITHUB021 invariant.

**No test, native-image proof, product command, source/spec publication or D220 closure is established by this approval.**
