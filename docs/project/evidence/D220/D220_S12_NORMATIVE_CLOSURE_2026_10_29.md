# D220–S12 — normative TLS decision reconciliation and closure evidence (2026-10-29)

**Formal design owner:** [D220 #975](https://github.com/guillermomolina/protos/issues/975).
**Implementation owner:** [I130 #977](https://github.com/guillermomolina/protos/issues/977).
**Exact owner approval:** Guillermo approved the full S12 TLS client public/security/Trust/identity/ALPN/Future/Byte I/O/cancellation/close/error/deferred-scope candidate on **2026-10-28**, documented in [D220 exact S12 ratification](../../decisions/language/D220_PORTABLE_TLS_HTTPS_CLIENT_S12_2026_10_28.md) and the [GITHUB010/GITHUB021 acceptance packet](D220_S12_APPROVAL_AND_ACCEPTANCE_2026_10_28.md). The additional owner-approved `trustUnavailable` versus `backendUnavailable` clarification remains recorded in the ratification addendum; no further design selection is being inferred from tests.

**Normative product revision:** [guillermomolina/protos@5232e971ad5a85fdbfc9235c0f3273fb1912038c](https://github.com/guillermomolina/protos/commit/5232e971ad5a85fdbfc9235c0f3273fb1912038c), version `0.3.420-SNAPSHOT`, global specification revision `0.1.479`. **Normative owners:** `spec/io/TLS.md`, `spec/PROTOS_SPEC_CHANGELOG.md` and the pre-existing adjacent I/O, concurrency, errors, modules, HTTP and network specifications.
**Final implementation evidence:** [I130-C3 / 0.3.420](../I130/I130_C3_0_3_420_COMPLETION_2026_10_29.md), preceded by the I130-A/B/C1/C2 exact revisions linked there.

## GITHUB021 owner-invariant reconciliation

```text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
NORMATIVE_PRODUCT_PUBLICATION=PASS
PROTOS_REVISION=5232e971ad5a85fdbfc9235c0f3273fb1912038c
GLOBAL_SPEC_REVISION=0.1.479
CROSS_REFERENCES=PASS_WITH_EXACT_REVISION_LINKS
REQUIRED_DURABLE_RECORD=THIS_RECORD
```

- **Explicit caller authority and trust:** The caller supplies an already connected byte duplex, a separate expected DNS/IP identity and an explicit Trust descriptor. No ambient network/name resolution, plaintext fallback, implicit default TLS trust or host parser change was introduced. Selected `Trust.anchors` and `Trust.javaCacerts` remain independent; `Trust.system()` is intentionally unavailable, not silently mapped to OS/JSSE defaults.
- **Selected identity contract:** RFC 9525 SAN-only DNS/IP matching, exact IP SAN, DNS-only SNI and bounded whole-leftmost-label wildcard. The I130-B `0.1.478` **temporary** rejection of all `xn--` labels contradicted the final approved DNS-IDNA2008 scope, and was explicitly marked incomplete rather than an approved new invariant. I130-C3 `TLS.md` §4 and spec `0.1.479` replace it with strict RFC 3492/5890–5893 canonical A-label validation on both reference and SAN, while continuing to reject U-label input/conversion; the historical discrepancy is now resolved.
- **Error and lifecycle contract:** The approved precise `javaCacerts` unavailable → `trustUnavailable`, locally absent compliant backend → `backendUnavailable`, peer-trust refusal → `untrustedPeer`, and other fixed `TlsError.kind` mappings remain normative. Controlled TLS 1.2/1.3 and ALPN, handshake Future, bounded records/writes, authenticated `close_notify`, truncation, Actor confinement, borrowed/owning custody and error precedence were implemented incrementally and covered by I130 evidence.
- **Separation and deferrals:** Policy and public composition stay in Protos; minimal private JSSE/PKIX/Unicode facts stay behind a narrow facility. `std:http/Client.exchangeOne` remains unchanged and accepts a verified secure duplex. OS trust freshness adapters, server/Host TLS, mTLS, 0-RTT, session resumption, URL/DNS convenience and alternate engines remain **out of S12**; their absence is not a D220 inconsistency.

The exact approved design, later approved clarification, published spec and implementation evidence have no outstanding known invariant conflict. Therefore the bounded S12 **design ratification can close**, without treating deferred extensions or Native Image runtime limitations as unapproved scope. The human reports `git diff --check` clean, `make test` and `make check` PASS on the pushed final product; previously provided Native JUnit logs show 38/38 TLS peer/trust cases PASS. Those tests were not rerun by the coordinating agent.

**Revision binding:** This record names the immutable product commit; its immutable GitHub documentation commit and the exact cross-repository pair are recorded and read back as part of the final D220 Issue closure comment. Original 2026-10-28 selection/ratification records remain historical and are not rewritten into post-implementation claims.
