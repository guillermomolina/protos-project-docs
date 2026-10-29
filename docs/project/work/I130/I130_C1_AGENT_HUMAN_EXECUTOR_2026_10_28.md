# I130-C1 — Portable TLS record-limit hardening: agent and Human Executor

**WORK:** IMPLEMENTATION, exclusively in guillermomolina/protos.
**Formal owner:** [I130 #977](https://github.com/guillermomolina/protos/issues/977), under [D220 #975](https://github.com/guillermomolina/protos/issues/975). No new Issue is required for this cohesive slice.
**Published TLS prerequisite:** protos@07ca92be06c7139cfff4b35c47b2452e2ff50e28 (I130-B; 0.3.416; global spec 0.1.478). On 2026-10-29 product HEAD had advanced to ce6cdec09a615a31edf668e22bd022be6cac2626 (I131 installer); use current local HEAD and preserve unrelated work.
**Authority:** approved D220–S12 and normative current spec/io/TLS.md. This handoff is NON-NORMATIVE; it does not approve new semantics or close D220.

## Agent: complete offline implementation prompt

You are the I130-C1 IMPLEMENTATION AGENT. Assume the local repository guillermomolina/protos is already at HEAD. Read the local root/scoped AGENTS.md, AGENTS.work/IMPLEMENTATION.md, AGENTS.work/LIBRARY.md, and the actual normative owners: spec/io/{TLS,BYTE_IO,IO_CORE,HTTP,NETWORK}.md, spec/concurrency/{FUTURES_AND_TASKS,ACTORS}.md, and spec/semantics/{ERRORS,MODULES}.md. All needed primary external facts are below. Do not browse the internet or any other repository. Do not run ANY shell/git/build/test/validator/benchmark/native-image/program command, do not commit or push. Edit relevant product files only; return them to the Human Executor.

**Objective:** A conclusively over-bound inbound TLS record must fail with fresh TlsError.kind == "limit" both during the handshake and after it. Existing I130-B can produce "protocol"/"handshakeFailure" when JSSE rejects an oversized record. Never guess a specific kind from SSLException text or a peer-supplied alert. Add deterministic adversarial JVM coverage for the same record/framing and transport path.

**Product code starting points:** Java private host/tls/{ProtosTlsClientEngine,ProtosTlsFailure}.java and facilities/ProtosTlsBackendFacility.java; Protos policy protos/lib/tls/internal/{Errors,Session,SecureDuplex}.protos and public Client.protos for an inaccurate comment only. Existing tests include ProtosTlsClientEngineTest, ProtosTlsTestPeer, ProtosTlsClientIntegrationTest and protos/tests/library/tls/*.protos. Inspect local actual paths and nearby examples before editing.

**Ratified TLS.md §10 bounds:** plaintext record 16,384 octets; encrypted record 18,432 octets; receive/unwrap buffer 65,536 octets; handshake accumulation maxHandshakeBytes; queued write snapshots maxQueuedWriteBytes. Errors §9: "limit" for exceeded bound and preserve exact underlying transport Error identity. Current Session.protos bounds a single lower read at 18,432; a bounded read is NOT an on-wire record-bound check.

**Conclusive framing:** A TLS record has five header octets: content type (1), legacy version (2), unsigned network-order payload length (2), then the indicated payload. Detect only from actual framing/buffer evidence, across partial 1/2/4/5-byte headers, fragmented payload and coalesced records; prevent false positives, duplication, replay, unbounded allocation, and record scanning inside decrypted application data. SSLEngine remains the sole crypto/authentication and general TLS-protocol implementation. RFC 8446 §5.2 TLS 1.3 encrypted payload maximum is 16,640 (=2^14+256) octets; do not conflate this with Protos 18,432. TLS 1.2 has different expansion rules. If current spec/source does not resolve whether 18,432 includes the five-octet header and that changes outcomes, STOP and request Guillermo's explicit decision rather than silently choosing an off-by-five rule.

**Boundary:** Java may provide raw framing/buffer observation and minimal conclusive backend fact; Protos internal/Errors.protos maps fact to "limit". Keep portable policy/options/identity/trust, Task/Future, transport pumping, custody, close and error precedence in Protos. Keep the first engine failure sticky. Ambiguous JSSE errors remain handshakeFailure during handshake or protocol afterward. No public engine registry, second backend or Java mirror of std:tls.

**Adversarial tests:** in-memory deterministic TLS 1.2/1.3 cases where feasible for split headers, just-under/exact/over bound, declared excess with incomplete body, valid multi-record coalescing, tiny chunks, malformed frames, legitimate large fragmentation, handshake and post-handshake, EOF/truncation, and outbound invariants if applicable. No real sockets, system CA, wall-clock dependence, sleeps or fabricated PASS. Preserve current regressions: no-admission traffic, one handshake Task, non-Task I/O Futures, cancelled-read retention, lower-write fail-closed, capacity rejection recovery, borrowed/owning custody, idempotent close, authenticated close_notify versus truncated EOF, exact transport Error, Actor locality, explicit Trust, SNI/DNS/IP, ALPN http/1.1 and unchanged Client.exchangeOne.

**IDNA note:** current TLS.md §4 admits only NR-LDH references; reserved ??-- and all xn-- A-labels are temporarily rejected. If Client.protos says generic "ASCII (A-label)", correct the comment. Do not claim full strict IDNA2008, remove the restriction, or silently make a design decision. Complete IDNA requires RFC 3492 Punycode plus RFC 5891 lookup, RFC 5892 properties/context and RFC 5893 bidi; Java IDN and ICU UTS #46 do not prove strict conformance.

**Offline external/Truffle references:** RFC 9325 permits secure TLS1.2 while preferring 1.3; RFC 7301 ALPN token is "http/1.1"; RFC 9525 requires SAN-based DNS/IP identity, not CN fallback; Java 25 SSLEngine wrap/unwrap reports OK/CLOSED/BUFFER_UNDERFLOW/BUFFER_OVERFLOW and may schedule delegated tasks. GraalJS keeps JS guest-facing API above private host primitives; GraalPy ssl/asyncio separates Python stream/cancellation semantics from backend cryptography; TruffleRuby OpenSSL/IO keeps language-facing ownership over native crypto; Espresso's guest JSSE still needs Context isolation and packaging reachability; Sulong/LLVM native TLS handles require explicit custody and interop isolation. They do not justify any extra public TLS engine.

**Out of scope:** C2 Native Image JSSE provider/CA/distrust provenance and packaged executable proof; C3 strict IDNA2008 reusable Protos capability (GITHUB023) and validation; TLS server, Host HTTPS, mTLS, mandatory OS CA adapters, implicit fallback, 0-RTT or speculative engines. GraalVM Native Image may capture CA and distrust information at build time, which must later be proven by human tests. I130 and D220 stay OPEN.

No pom.xml/CHANGELOG.md/spec changelog edits until Human Executor reports all relevant PASS. Obey APL Part 5 notices. Deliver exact diff paths, conclusive observation for "limit", tested scenarios CREATED but NOT RUN, outstanding ambiguity, and Human Executor commands. STOP without executing anything.

## Human Executor (separate, human-only)

1. Review code ownership, raw record evidence and fragmentation/off-by-five logic. Confirm no JSSE-text or peer-alert classification and no unrelated edits.
2. Run in local protos:
   - git diff --check
   - mvn test -Dtest='ProtosTlsClientEngineTest,ProtosTlsTrustTest,ProtosTlsClientIntegrationTest,ProtosHttpClientIntegrationTest'
3. Report PASS or only the first compiler error/failing assertion. Run existing guest TLS and adjacent HTTP/Byte I/O cases using the actual HEAD test runner; then run make test and make check. Agent only repairs code.
4. After all human gates PASS: finalize next real patch version and relevant changelog per HEAD rules (spec changelog only on normative change); stage explicit patch-owned paths; human alone commits a one-line English I130-C1 message without trailers, pushes without force, and reports SHA and PASS.
5. Coordinator publishes revision-bound completion evidence and updates I130 #977; do not close I130 or D220 until C2/C3.

**STATUS:** C1 PROMPT READY / NO C1 TEST RESULTS / NO NEW ISSUE.
