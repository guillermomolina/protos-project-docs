# D220 — Portable TLS: comparative evidence and approved initial scope (2026-10-28)

**Kind:** non-normative D220 research / owner-scope checkpoint; **NOT an exact public-API ratification**.
**Issue:** https://github.com/guillermomolina/protos/issues/975
**Product source baseline:** [guillermomolina/protos@7f3113ad5c29a15584053c6440640bff6911e60a](https://github.com/guillermomolina/protos/commit/7f3113ad5c29a15584053c6440640bff6911e60a).
**Documentation source baseline before this evidence:** guillermomolina/protos-project-docs@7377dbb53b68c5220d1cf477d28a9d7e0afdf774.
**Execution:** GitHub/repository-source and primary-publication research only; no local commands, builds, tests, product source edits or normative specification publication. No native/JVM proof-of-concept has been executed.

## 1. Precisely approved and not approved (GITHUB021)

**Earlier owner approval (active discussion, 2026-10-28):** “ok apruebo este esquema”, already documented on [D220 comment #6463516876](https://github.com/guillermomolina/protos/issues/975#issuecomment-6463516876).

Approved architectural invariants:
1. Protos defines portable TLS public behavior, authentication and trust semantics; PLAT/JSSE defaults do not define them.
2. A small Protos frontend and minimal private backend boundary, without public backend registration, speculative plugins or multiple initial engines.
3. Initial **TLS engine**: GraalVM/JSSE only; other engines remain replaceable future implementations.
4. TLS cryptographic engine and certificate trust-source/verifier policy are separate. OS trust, Java cacerts, and supplied CA anchors never silently substitute for or merge with one another.
5. Server TLS and additional engines and OS trust adapters are independently deferred and must not silently weaken security.

**Subsequent explicit owner approval (active discussion, 2026-10-28):** Asked “no podemos empezar con ‘ningun OS’?”; after the explanation of a first implementation with *zero OS trust adapters*, explicit supplied CA anchors, explicitly selected Java cacerts, and unsupported-system-policy failure rather than fallback, responded: **“ok, pues lo apruebo.”**

Thus the approved **initial-scope delta** is:
- **No Linux, Windows, macOS, BSD or other OS trust adapter is mandatory in the initial deliverable.**
- First usable TLS-client mode may use explicit application-supplied anchors or explicitly opted-in Java cacerts.
- Choosing system trust when its adapter is unavailable must fail visibly and closed; **never fall back to Java cacerts or union stores**.
- Deferral is per trust-source adapter and does not require a second TLS engine.
- Later OS adapters and server support are additive work with separate acceptance and scheduling.

**Still NOT approved:** the precise `std:tls` exported objects/selectors, default trust choice, parser/config shape, exact error-kind strings, endpoint-identity matching details, TLS 1.2 versus TLS 1.3 minimum, ALPN details, timeouts, TLS session cache semantics, detailed handshake/close/ownership/cancellation contract, test bounds, and an exact normative spec revision. The owner approved architecture and first-release scope, **not** an unpresented full candidate. D220 remains open; no Ixxx implementation has been authorized merely by this evidence.

## 2. Existing Protos authority and source audit

Read-only governing sources at the fixed product baseline:
- `AGENTS.md`; `AGENTS.work/{DESIGN,COORDINATION,REFERENCE}.md`; `spec/AGENTS.md`; `protos/AGENTS.md`.
- `spec/io/{NETWORK,HTTP,IO_CORE,BYTE_IO}.md`; `spec/concurrency/{FUTURES_AND_TASKS,ACTORS}.md`; `spec/semantics/{ERRORS,MODULES}.md`.
- `protos/lib/http/{Client,Host,Server}.protos`; `protos/lib/http/internal/ClientSession.protos`; `src/main/java/com/guillermomolina/protos/execution/facilities/ProtosHttpAsyncFacility.java`.
- `build/native/{Dockerfile,Makefile}`; confirmed Native Image target uses GraalVM 25 / Oracle Linux 10.

Constraints from normative and source owners:
- Network owns explicit capability and numeric endpoint/TCP acquisition; TLS must not introduce ambient DNS, socket/Network access or URL/redirect/proxy behavior.
- `Client.exchangeOne[.owning]` accepts an already supplied byte duplex and must stay independent of TLS and hostname resolution; the caller provides an authenticated duplex for HTTPS.
- Byte I/O has per-operation Future, bounded buffering, no lost bytes on cancelled reads, partial-output uncertainty, exact admission/snapshot obligations, and distinct clean EOF.
- `IO_CORE` determines borrowed versus owning custody, cleanup and failure precedence; Actors prohibit moving live mutable TCP/TLS state across Actor boundaries.
- `Server.serveOne` can later consume a secure server duplex, but `Host.owning` accepts a TCP listener and internally accepts/distributes raw TCP connections. HTTPS Host requires **future additive public integration and per-Worker credential provisioning**, not a zero-cost reuse claim.
- `ProtosHttpAsyncFacility` is a narrow private Task/Future bridge precedent, not authority for importing a Java HTTP/TLS client engine.

## 3. Independent primary-source research

| System | Model | Transferable observation and limitation |
| --- | --- | --- |
| [Go crypto/tls](https://pkg.go.dev/crypto/tls) | TLS over a caller-controlled network connection | `ServerName` governs identity/SNI and `RootCAs` is distinct from default roots; direct TLS connection closure is not Protos borrowed/owning semantics. |
| [tokio-rustls](https://docs.rs/tokio-rustls/latest/tokio_rustls/) + [platform verifier](https://docs.rs/rustls-platform-verifier/latest/rustls_platform_verifier/) | asynchronous TLS over supplied I/O, verifier separate | Closest layered precedent; on Linux system-root extraction does not automatically preserve all native distrust policy. |
| [.NET SslStream](https://learn.microsoft.com/en-us/dotnet/api/system.net.security.sslstream) | stream decorator with explicit endpoint authentication | Native chain policy and inner-stream leave-open support; a host API is not itself Protos semantics. |
| [Python asyncio StreamWriter.start_tls](https://docs.python.org/3/library/asyncio-stream.html#asyncio.StreamWriter.start_tls) | upgrade an existing async transport | TLS-on-duplex and bounded handshake options; SSLContext default trust varies by runtime/platform. |
| [Node.js TLS](https://nodejs.org/api/tls.html) | event-driven TLS on sockets | `secureConnect` alone need not mean authorized; the system-CA option may combine sources and is not an acceptable implicit Protos rule. |
| [OpenJDK JSSE / SSLEngine](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/javax/net/ssl/SSLEngine.html) | connection-independent wrap/unwrap state machine | Client/server modes, ALPN and replaceable `X509ExtendedTrustManager` support the first private engine; Java's default roots and endpoint algorithm are not portable Protos policy. |

Three substantially different models: native socket + TLS wrapper (Go/Node), async stream adapter with decoupled verifier (rustls/.NET/Python), and cipher/handshake engine separated from transport (JSSE).

**Truffle feasibility survey (not prescriptive):**
- [GraalJS](https://www.graalvm.org/latest/reference-manual/js/): TLS in its Node-oriented hosting ecosystem is not an authoritative portable guest-language trust policy.
- [GraalPy](https://www.graalvm.org/latest/reference-manual/python/): Python-compatible ssl/host implementation differs across execution modes and is not the Protos contract.
- [TruffleRuby](https://www.graalvm.org/latest/reference-manual/ruby/): native extension/OpenSSL integration shows another hosting option, not a universal Truffle TLS API.
- [Espresso](https://www.graalvm.org/latest/reference-manual/java-on-truffle/): Java libraries remain subject to Java security-provider policy.
- [Sulong/LLVM](https://github.com/oracle/graal/tree/master/sulong): language/native execution substrate, not a self-standing HTTPS/TLS API or trust-policy decision.
- SimpleLanguage: informative for small Truffle languages, **not materially comparable** as a TLS library model.
- Apple Pkl: useful counterexample on explicit versus packaged CA provenance, but a configuration-language HTTPS facility is not directly comparable to a portable TLS-on-byte-duplex capability.

**Normative security / runtime facts:**
- [RFC 9325](https://www.rfc-editor.org/rfc/rfc9325.html): TLS 1.0/1.1 forbidden; TLS 1.2 with safe suites and TLS 1.3 preferred for general application compatibility. A deliberate TLS-1.3-only subset would require an explicit interoperability decision.
- [RFC 9525](https://www.rfc-editor.org/rfc/rfc9525.html): reference DNS/IP identity in SubjectAltName, not unqualified legacy CN fallback. JSSE `SSLParameters.setEndpointIdentificationAlgorithm("HTTPS")` alone is not sufficient as Protos' precise RFC 9525 contract.
- [RFC 7301](https://www.rfc-editor.org/rfc/rfc7301.html): ALPN; HTTP/1.1 client must not silently run HTTP/2 after negotiation.
- [RFC 8446](https://www.rfc-editor.org/rfc/rfc8446.html): handshake, alert closure and resumption. Transport EOF without an authenticated TLS close can be truncation; resumed TLS 1.3 handshakes may omit a fresh server Certificate.
- [Java X509ExtendedTrustManager](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/javax/net/ssl/X509ExtendedTrustManager.html): trust verification can be supplied independently of the engine, subject to Native Image feasibility testing.
- [GraalVM Certificate Management in Native Image](https://www.graalvm.org/jdk25/reference-manual/native-image/dynamic-features/CertificateManagement/): default truststore and JDK blacklist can be captured at image build time.
- [GraalVM October 23, 2026 advisory](https://www.graalvm.org/uploads/native-image-advisory-oct-2026-cspu.pdf): concrete compromised-CA incident demonstrates the build-time stale-root/blocklist risk; a native executable must not claim that `system` trust updates automatically by inheriting Java defaults.
- [GraalVM JCA Security Services](https://www.graalvm.org/jdk25/reference-manual/native-image/dynamic-features/JCASecurityServices/): reachable crypto providers and their ordering can be fixed at image build time; explicitly test JVM and native-image support.
- [p11-kit trust](https://p11-glue.github.io/p11-glue/p11-kit/manual/trust-module.html), [Microsoft CertGetCertificateChain](https://learn.microsoft.com/en-us/windows/win32/api/wincrypt/nf-wincrypt-certgetcertificatechain), and [Apple SecTrustEvaluateWithError](https://developer.apple.com/documentation/security/sectrustevaluatewitherror(_:_:)): future OS trust-policy adapters are not equivalent to importing one PEM bundle.

## 4. Candidate set and GITHUB010 twelve-dimensional scores

Surviving scope alternatives on **the same owner-approved layered architecture**:
- **S — selected first-scope direction:** one JSSE motor, supplied-CA and explicit cacerts modes now, zero OS adapters initially; preserve private verifier seam, unsupported system mode fails closed.
- **L — credible larger initial scope:** same architecture plus Linux system-policy adapter validated before any first D220 implementation publication.

Hard-rejected alternatives: direct `SSLContext.getDefault()` (breaks approved trust invariants); simply calling an exported OS PEM file “effective OS policy” (can lose explicit distrust); Java HTTP client (duplicates existing Protos HTTP protocol); all OS TLS engines at once (speculative cost); write own TLS/X.509 cryptography (vast security/correctness burden); no TLS facility at all (does not meet HTTPS-client need).

Scores 1 (weak) to 5 (strong); every score is `score/confidence — justification`. H=HIGH, M=MEDIUM, L=LOW.

| GITHUB010 dimension | S: no OS adapter initially | L: Linux adapter initially |
| --- | --- | --- |
| 1 Correctness/invariants | 5/H — explicit anchors and cacerts, no implicit substitution | 4/M — correct only if Linux distrust/purpose policy is reproduced, not just roots |
| 2 Protos philosophy | 5/H — small private interfaces, ordinary supplied duplex | 4/M — same portable design, extra native trust integration |
| 3 Pay for present need | 5/H — one motor, two explicit usable trust sources | 2/H — OS policy FFI is not required for current explicit-CA HTTPS |
| 4 Grow incrementally | 5/M — add Linux verifier through already defined private seam | 4/M — future OS adapters still incremental |
| 5 Future-option resilience | 5/M — no public Java/platform lock-in | 5/M — Linux adapter also keeps future OS engines possible |
| 6 Scaling | 4/M — isolated bounded connection state; no global trust service | 4/M — native policy calls need bounded per-handshake execution |
| 7 Conceptual simplicity | 5/H — TLS engine and selected trust policy only | 3/M — host trust semantics, lifecycle and freshness in initial surface |
| 8 Portability | 5/H — portable API; unavailable system mode fails closed | 4/M — Linux-only first policy adapter adds platform-specific testing |
| 9 Runtime cost | 5/M — no native trust probe when unused | 4/M — policy evaluation and OS store/cache freshness cost |
| 10 Failure/operability | 4/M — clear trustUnavailable; no deceptive OS claim | 4/M — native policy failures need fine classification and diagnostics |
| 11 Deferral/reversibility | 5/M — additional private verifier without public break if API is specified carefully | 4/M — immediate Linux coupling can be corrected, but adds migration burden |
| 12 Evidence maturity/risk | 4/M — JSSE integration precedent strong; Protos Future pump unproven | 2/M — Linux policy fidelity + GraalVM Native Image integration unproven |

**Anti-overengineering gate:** L pays OS-native integration cost before any current required OS-specific behavior; it remains a valid later slice, not a first-version requirement. **Anti-underengineering gate:** S must preserve the private trust-verifier seam and explicit unavailable behavior now; otherwise OS trust later could force a public trust-policy redesign. Without that seam, S is disqualified. Scores are not arithmetic authority.

**Pay/grow/defer:** S pays for authenticated TLS and two deliberately selected trust sources only. Later OS support adds verifier adapters, policy snapshots/freshness tests and platform-specific tests, without changing the TLS engine or already-defined frontend semantics. A later TLS server adds certificate/private-key authority and handshake role, while `Host` needs a new opt-in acceptance/provisioning mode; do **not** promise zero public Host work.

**Adversarial/future stress:** many Actors/Contexts cannot share mutable live TLS connections or host-side context state; Native Image compiled before root removal must not claim runtime system trust; cancelled reads may already have consumed encrypted transport bytes and must retain plaintext state for later reads; failed writes may have committed an uncertain prefix; truncated transport EOF is not authenticated clean EOF; reused TLS sessions need trust-generation reasoning; native OS policy may perform hidden network fetch; future server Workers cannot receive arbitrary live key objects across Actors. When those use cases arise, the escape is adding bounded verifier adapters, opt-in trust authorities, or a server wrapper, not revising the approved portable boundary.

**Strongest counterargument to S:** HTTPS against normal public sites is less ergonomic until an OS adapter exists; explicit Java cacerts can remain stale in Native Image and is not a safe automatic default. This is an honest documented limitation, not a reason for silent fallback. A future user demand for default system roots warrants implementing one OS adapter, but does not invalidate the initial minimal API.

## 5. Exact remaining design gate and future delivery

**Next work:** continue D220 as **INVESTIGATION / exact contract design**, using source/web research only, with no commands/builds/tests/code/spec edits. The next handoff is [D220 agent/human-executor brief](../../work/D220/D220_NEXT_SLICE_AGENT_HUMAN_EXECUTOR_2026_10_28.md).

Before creating an Ixxx:
1. Present a precise `std:tls` contract: export names, arguments/options, trust descriptors and unavailable mode, expected reference identity, handshake/Future/Task, admission and ownership, close/EOF/truncation, cancellation/backpressure limits, error hierarchy/kinds, ALPN/TLS-version policy and all omitted features.
2. Reconcile the candidate against **all** approved GITHUB021 invariants above; explicitly expose any new semantic choice.
3. Obtain explicit owner approval of **that exact candidate**.
4. Only then publish normative text under the proper `spec/` owners, a durable decision/ratification record, and separately allocate a meaningful Ixxx implementation Issue under GITHUB007/015. Product implementation/publication remains a human-executor workflow in `guillermomolina/protos`.
5. OS adapters and server TLS remain deferred: do not turn their absence into a hidden D220 completion requirement or assume they are implemented.

**Evidence state:** approved architectural and initial-scope invariants recorded; exact API not ratified, Native Image trust behavior not experimentally verified, no test PASS claimed, D220 remains open.
