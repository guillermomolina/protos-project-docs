# I130-C3 — Strict IDNA2008 A-label admission: Agent / Human Executor

**Kind:** IMPLEMENTATION, **not** investigation. **Repository for all edits:** `guillermomolina/protos` (two “mo”), **only**. Owner [I130 #977](https://github.com/guillermomolina/protos/issues/977), approved [D220–S12 #975](https://github.com/guillermomolina/protos/issues/975). Baseline: the **already-synchronized local HEAD** published I130-C2 [`36a0bad894181c9534c200614361b8aa132b8df4`](https://github.com/guillermomolina/protos/commit/36a0bad894181c9534c200614361b8aa132b8df4), version `0.3.419-SNAPSHOT`; do not fetch/pull/checkout or require additional repositories or online research. [C2 exact-revision evidence](../../evidence/I130/I130_C2_0_3_419_COMPLETION_2026_10_29.md). This **single cohesive C3 batch** includes parser/policy/host-mechanics/tests/spec/documentation; do not split trivial file edits into further issues. I130 and D220 stay open until validated and expressly closed.

## Part A — Agent prompt (copy in full)

You are the **I130-C3 EDIT-ONLY IMPLEMENTATION AGENT** operating *inside* the local `guillermomolina/protos` checkout, which is **already at HEAD** with I130-C2/0.3.419. Your authorization is **only file editing**. Do not execute shell/git/search/build/test commands, browse the web, inspect other repositories, install dependencies, network-fetch, branch, commit, push or edit GitHub issues. Read only the applicable local `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/LIBRARY.md`, `spec/AGENTS.md`, `protos/AGENTS.md`, `src/AGENTS.md` and the **explicitly named product files** below (plus directly referenced helper definitions when indispensable). Keep APL headers, Protos-first language semantics, comments/formatting and unrelated user edits intact.

**Outcome:** complete the remaining, already-approved D220–S12 strict IDNA2008 A-label conformance of `std:tls/Client` without enabling U-label convenience, weakening certificate identity, adding implicit trust/network/HTTP behavior, or replacing the JSSE engine. This is not architectural investigation: implement the exact following contract and regression matrix.

### Baseline files and current behavior

1. `protos/lib/tls/internal/Identity.protos`: `dnsName(text, allowWildcard)` parses ASCII names, strips one terminal root dot, lowercases A–Z, checks label ≤63/name ≤253 octets and ≥2 labels. `closeLabel` currently rejects *every* R-LDH label, identified by `label[2]==45 && label[3]==45`, including `xn--`. `reference` returns frozen `{labels,octets,sni}`; `matches` consumes peer `subjectAltName` dNSNames and exact iPAddress octets; `sameName` only allows a complete leftmost `*` matching one label. Keep these protocol decisions **in Protos**.
2. `protos/lib/tls/Client.protos`: remove the now-stale temporary `xn--` rejection comment, updating it to strict validation.
3. `protos/tests/library/tls/client-admission.protos`: current test “reserved LDH labels, including every xn-- A-label, are inadmissible” rejects `xn--bcher-kva.test`, `xn--zz.test`, `ab--cd.test`. Replace with explicit valid/invalid cases.
4. `protos/tests/library/tls/identity.protos`: add direct name/SAN/wildcard equivalence cases.
5. `src/test/java/com/guillermomolina/protos/execution/integration/tls/ProtosTlsClientIntegrationTest.java`: current `reservedLdhReferencesAreRejectedAtAdmissionWithoutEffects`; extend for real JSSE in-memory peer, strict positive SAN identity and admission failures with zero lower-transport effects.
6. `spec/io/TLS.md` §4 and §13: current **temporary** blanket R-LDH exclusion and deferred-conformance note. Replace with precise IDNA2008 A-label behavior and remove the incompleteness caveat **only when implemented and tested**. Add spec changelog in `spec/PROTOS_SPEC_CHANGELOG.md` per local version policy.
7. `CHANGELOG.md` and `pom.xml`: Human Executor owns final release version; agent may make bounded proposed changelog/docs edits only if local AGENTS permits, and **must not pre-claim PASS**. Update `dist/README.md` only if the new acceptance boundary affects its current TLS user guidance.

### Exact public/security requirements — mandatory

- DNS reference `String` and presented X.509 dNSName are ASCII wire form only (NR-LDH labels or valid IDNA2008 `xn--` A-labels). **Reject U-label Unicode input**, even where it could be converted; no implicit NFC, casefold, UTS #46 compatibility mapping or IDNA2003 conversion of caller input. Reject empty names, empty labels, single-label names, excessive octets, leading/trailing hyphens, all R-LDH other than *valid* A-labels (`ab--cd`), malformed `xn--` payload, fake/pure-ASCII Punycode, noncanonical or nonroundtrippable labels, DISALLOWED/UNASSIGNED decoded characters, illegal CONTEXTJ/CONTEXTO, bad combining marks, invalid RFC 5893 Bidi labels, invalid Unicode scalars, and invalid wildcard positioning. Only strip at most one optional terminal root dot.
- A-label validity is **per-label**, for both reference and SAN candidate, before matching. Admission must not alter the original label except ASCII case normalization. A fully valid `XN--BCHER-KVA.Example` references `xn--bcher-kva.example`, SNI receives the canonical lowercase ASCII `xn--bcher-kva.example`, and a SAN dNSName with valid A-label matches ASCII case-insensitively. There is no CN fallback, no U-label SAN coercion; invalid presented name never authorizes a match. Keep the existing exact IP bytes / no SNI rule and RFC 9525 one-label leftmost `*` wildcard behavior. No public suffix, DNS lookup, URL parser, hostname/default trust inference or certificate chain loosening.
- **RFC 3492 Punycode**: verify exact digits/base, insertion/index overflow and Unicode scalar bounds; reject invalid ACE syntax and both syntactically invalid and semantically invalid decoded labels. Re-encode decoded Unicode and require exact `xn--` payload roundtrip modulo ASCII case; U-label decoded sequence must contain at least one non-ASCII code point, be NFC and pass IDNA2008 rules. Do not use a mere ASCII regular expression as IDNA2008 validation.
- **RFC 5891/5892/5893**: enforce leading-combining-mark rejection, hyphens 3–4 of decoded U-label, label length, PVALID/CONTEXTJ/CONTEXTO classifications and per-label context/Bidi rules. RFC 5892 derived-property precedence is: explicit exceptions; backward-compatible overrides (empty in baseline RFC); unassigned; ASCII LDH; Join_Control; unstable Unicode under NFKC(casefold(NFKC(cp))); default-ignorable/whitespace/noncharacter; three ignorable Unicode blocks; conjoining Hangul Jamo L/V/T; LetterDigits categories `Ll,Lu,Lo,Nd,Lm,Mn,Mc`; otherwise disallowed. RFC exceptions: PVALID `00DF,03C2,06FD,06FE,0F0B,3007`; CONTEXTO `00B7,0375,05F3,05F4,30FB,0660..0669,06F0..06F9`; DISALLOWED `0640,07FA,302E,302F,3031..3035,303B`; JOIN controls `200C,200D` are CONTEXTJ. Context checks include U+00B7 only between lowercase `l` letters; 0375 before Greek script; 05F3/05F4 after Hebrew script; 30FB only in labels containing Hiragana/Katakana/Han; Arabic digit families must not mix; ZWJ only after virama; ZWNJ after virama or correct joining-type sequences (transparent codepoints ignored), **not simply “always allowed”**. RFC 5893 applies its six directional checks across labels in a Bidi domain: valid initial class, allowed classes by LTR/RTL, valid final significant class and no mixing AN with EN in an RTL label.
- The product already has **ICU4J 78.1** in `pom.xml` and builds Native Image; use ICU *Unicode property/normalization primitives* if necessary. `java.net.IDN` follows IDNA2003 and is NOT a strict IDNA2008 validator. ICU `com.ibm.icu.text.IDNA.getUTS46Instance`, even with nontransitional, `CHECK_BIDI`, `CHECK_CONTEXTJ`, `CHECK_CONTEXTO` and STD3, is a **UTS #46 compatibility processor**, not independently proof of strict IDNA2008: do not accept a label just because UTS #46 mapped or accepted it. If used at all as an extra guard, require byte-for-byte roundtrip plus **independent strict RFC5892 properties and contextual tests**. Unicode tables/ICU must be frozen/deterministic for both JVM and AOT; no network data/OS locale or platform default hostname validator.
- Private host authority is restricted to *mechanical Unicode/Punycode facts*; public admission, reference matching, SAN-only, wildcard, SNI and error timing stay in Protos. If host Unicode codepoint property access is genuinely necessary, add a single deterministic, no-network helper in existing `src/main/java/com/guillermomolina/protos/execution/facilities/ProtosTlsBackendFacility.java` or a small adjacent `execution/host/tls` class and invoke only through that facility’s private backend boundary. Do not add a second TLS engine, private DNS/trust authority, implicit transport, or a new public `std:tls` API. Update relevant native reflection/reachability/test architecture guard only if required. Prefer a complete coherent implementation in existing files, not a new host subsystem.

### Truffle/language precedent — supplied; no investigation necessary

- **Protos**, like GraalJS/GraalPy/TruffleRuby, can keep actual TLS crypto/JSSE in its host while enforcing language-specific guest identity policy at its frontier. In this repository that established boundary is `Identity.protos` ↔ the private TLS backend facility. Reuse it.
- **GraalPy/Python standard-library `encodings.idna`** uses IDNA2003/RFC 3490 and cannot serve as a strict IDNA2008 reference.
- **GraalJS / JavaScript WHATWG URL domain handling** follows web-compatible UTS #46 domain normalization, which is not the strict accepted D220–S12 wire-only policy.
- **TruffleRuby** provides ordinary Ruby/OpenSSL host-backed TLS use; it does not imply silent Unicode-to-ACE conversion is compatible with this Protos API.
- Thus **do not copy** the behavior of Python’s legacy codec, the WHATWG URL parser, or the default JSSE hostname verifier. Explicit constraints above are the complete cross-language design conclusion; no external Truffle source inspection is needed.

### Minimum test matrix (agent must edit relevant tests)

1. **Positive**: `xn--bcher-kva.example` representing `bücher`, mixed-case `XN--BCHER-KVA.Example`, terminal root dot, valid A-label appearing in either reference or SAN, successful authenticating native-compatible TLS 1.3 peer using SAN dNSName as wire ACE, matching full-label `*.xn--bcher-kva.example` against exactly one leftmost DNS label, and correct lowercase ASCII SNI. Include `faß` vs `fass` as **distinct** strict IDNA2008 names; do not introduce compatibility collapsing.
2. **Negative**: `ab--cd.example`; `xn--zz.example` (validate actual decoding, do not assume it is sufficient negative alone); `xn--.example`; lone `xn--`; uppercase/invalid/punycode overflow; ACE decoding to all ASCII or U-label with leading combining mark, prohibited/dot/space/control code points, disallowed historic or old Jamo, invalid NFC, fake ACE, unassigned, illegal context/Bidi; Unicode `bücher.example` as caller reference; non-ASCII presented SAN; invalid wildcard and overlong labels/name. Use **validated concrete negative vectors** rather than self-fulfilling tests.
3. Preserve direct IP SAN octet equality, no SNI for IP, no CN fallback, ALPN, exact `TlsError` kinds and no side effects on invalid admissions. Preserve existing real peer cancellation, ownership and close tests.
4. On native side ensure added pure Unicode helper is reachable in the **real** `target/native/protos` and `target/native-tests`; no accidental fixture registration into the distributed binary. The existing `make -C build/native test` gate remains mandatory for all 3 guest suites and both Native JUnit classes (C2) and must exercise added ACE cases. Do not weaken PERF006/PLAT045 gates or any existing test.

**Important quality stop:** Full strict RFC validation is security relevant. If you cannot implement **all** the above constraints reliably with primitives present in HEAD, **stop and report the exact blocker and still-unimplemented cases; do not label a permissive UTS #46 shortcut “IDNA2008”, do not silently amend the contract to partial support, and do not mark publishable**. Do not create a new research task inside this implementation agent.

### External protocol facts — provided offline

- RFC 5890 defines NR-LDH, R-LDH, A-label and U-label (an ACE-looking label is not necessarily a valid A-label). A valid ACE `xn--` label requires Punycode decode, semantic Unicode validation, canonical re-encode and DNS label length.
- RFC 3492 is the Punycode bootstring/base-36 algorithm; lowercase ASCII wire result must roundtrip exactly, no arithmetic overflow.
- RFC 5891 is the IDNA2008 lookup/registration processing contract: NFC, restrictions on hyphens and leading combining mark, CONTEXTO/CONTEXTJ, Bidi and permitted codepoint classes; wire A-label matching does not invoke UTS #46 mapping.
- RFC 5892 uses Unicode derived properties and explicit exceptions, context rule registry (nine groups), and classification order listed above; RFC 8753 documents review against newer Unicode.
- RFC 5893 applies directional and numeral checks in a Bidi domain.
- Unicode ICU4J 78.1 supports codepoint properties, normalization and Punycode/UTS#46 processing primitives but **UTS #46 and strict IDNA2008 have distinct admissibility**.
- Primary standards, provided for human traceability only (agent **must not browse**): `https://www.rfc-editor.org/rfc/rfc3492`, `https://www.rfc-editor.org/rfc/rfc5890`, `https://www.rfc-editor.org/rfc/rfc5891`, `https://www.rfc-editor.org/rfc/rfc5892`, `https://www.rfc-editor.org/rfc/rfc5893`, `https://www.rfc-editor.org/rfc/rfc8753`, `https://www.unicode.org/reports/tr46/`, `https://unicode-org.github.io/icu-docs/apidoc/dev/icu4j/com/ibm/icu/text/IDNA.html`. **All requirements needed to edit are already transcribed in this prompt.**

**Agent handoff:** report exact edited paths, per-file changes, pass/fail expectations, remaining blocker if any, and give Human Executor commands below; mark tests **NOT RUN**. Preserve clean refusal to overclaim. No commit, push, package or issue mutation.

## Part B — Human Executor (only after agent reports complete code)

Run these in the *already current* `guillermomolina/protos` checkout. The Human Executor—not the agent—owns execution, logs, final version, changelog and publication. Preserve unrelated user changes and never push red tests.

1. Read `git diff --check` and the focused file diffs for public identity policy, mechanical host validation, tests, normative spec and changelog. Confirm no weakened PLAT045/PERF006, no hidden auto-IDNA mapping, no broad trust source, no second engine, no modified HTTP.
2. Run the smallest focused JVM tests (confirm test class names in HEAD, adapt Maven selector if necessary):

```sh
git diff --check
mvn -q test -Dtest=ProtosTlsClientIntegrationTest,ProtosTlsTrustTest
```

3. Run the repo’s existing Protos guest tests using its canonical file-based invocation; the three mandatory TLS guest suites are automatically executed in Native Image gate:

```sh
make -C build/native test
```

This is a **long** full Native distribution build and separate Native JUnit build. Require `NATIVE_TLS_CONFORMANCE=PASS`, `NATIVE_REGRESSION_SUITE=PASS`, `NATIVE_TLS_PEER_INTEGRATION=PASS` and `NATIVE_TLS_TRUST_PROVENANCE=PASS` on the final output; require all native tests successful with no skips/failures and actual successful ACE SAN handshake and strict rejection coverage.

4. Run complete gates on the final patch:

```sh
make test
make check
git diff --check
```

5. Reconcile spec revision/version/changelog with repository policy, inspect final complete diff and report concrete PASS counts. Publish to `guillermomolina/protos` only if every required gate passes and **full** strict IDNA2008 is implemented. Git commit/push are **Human Executor-only**. Then relay exact product SHA/version plus gate outputs to the coordinator for immutable evidence and issues in `guillermomolina/protos-project-docs`. Do not mark I130/D220 closed unless all accepted requirements have affirmative final evidence and owner authorizes closure.

**Current formal state:** I130-C2/0.3.419 published; I130-C3 **not yet implemented**. Neither this handoff nor the current record claims C3 PASS.
