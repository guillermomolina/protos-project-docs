# D223-A1 — normative revision 0.1.480 and closure acceptance (2026-10-29)

**Design Issue:** [D223 #987](https://github.com/guillermomolina/protos/issues/987). **Implementation Issue:** [I132 #990](https://github.com/guillermomolina/protos/issues/990). **Owner-approved exact contract:** [D223 ratification](../../decisions/language/D223_EXPLICIT_RESOLVER_A1_2026_10_29.md).

**Normative product revision:** [guillermomolina/protos@aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9](https://github.com/guillermomolina/protos/commit/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9), version `0.3.421-SNAPSHOT`, spec `0.1.480`.
**Initial implementation evidence:** [I132-A report](../I132/I132_A_0_3_421_RESOLVER_COMPLETION_2026_10_29.md) from [guillermomolina/protos-project-docs@704a9f5b8c1b21510786948a4e89545170f1be56](https://github.com/guillermomolina/protos-project-docs/commit/704a9f5b8c1b21510786948a4e89545170f1be56).
**This record:** exact GITHUB021 check after new owner approval of the published §14.9 cache exception; immutable documentation SHA is bound in final live Issue closure comments after this record is published and re-read.

## Owner approval and changed-invariant check (GITHUB021)

The original 2026-10-29 owner-approved D223-A1 item 8 included a guarantee that backend/caches would not cross authority or Context domains observably. `InetAddress` uses shared JVM host name-service caching and cannot satisfy that broad literal guarantee. The implementation therefore published a narrower §14.9: isolation of **Protos-managed** mutable cache/result state, with authorized host JVM/OS caches excluded.

This exact narrowing was separately presented to Guillermo, who replied **“Si apruebo”** to the explicit §14.9 exception question on **2026-10-29**. The new approval replaces only that part of the original cache invariant. Shared host cache may affect name answers or response timing between grants, but grants remain independently required and host cache is not exposed as an authority or public TTL/cache state. No silent exception or claim of physical cache isolation remains.

| Owner-selected invariant | Published `0.1.480` / I132-A evidence | GITHUB021 result |
| --- | --- | --- |
| 1. Separate host-granted Resolver authority | `NETWORK.md` §§14.1–14.2, `PROCESS_IO.md`; no import/Network/Process-derived grant | Preserved |
| 2. Exact module/API surface | `std:network/Resolvers`, `lookupAddresses` and `close`, `ResolutionError` | Preserved |
| 3. Strict ASCII NR-LDH/IDNA2008 names | `NETWORK.md` §14.4; one label; no U-label/search/numeric coercion, unchanged TLS identity | Preserved |
| 4. Fresh frozen bounded IP results | §14.6; 1–64 structurally distinct canonical values, atomic limit/no authority | Preserved |
| 5. Host-authorized name-service domain | §14.5, including local/split-horizon/aliases, no universal DNS provenance | Preserved |
| 6. Four errors and invalid/closed behavior | §§14.3, 14.7; explicit host inability to distinguish some negative/transient `UnknownHostException` cases | Preserved with documented host-adapter limitation |
| 7. Future/cancellation/bounded backend | §14.8; owning-Actor publication, standard cancellation, no guest host-worker execution | Preserved |
| 8. Close, transfer and state isolation | §14.9; per-Resolver Protos-managed state, Process/host retirement and transferable IP data | **Amended cache scope, explicitly reapproved by owner** |
| 9. Numeric TCP and original HTTP/TLS identity | §14.10–14.11; numeric `connectTcp`, original hostname kept separate, no egress authority amplification | Preserved |
| 10. Listed future facilities deferred | §§13 and 14.12; no connect-by-name, `std:dns` records, TTL/search, deadlines, routing, REST or TLS expansion | Preserved |

**No other recorded D223 invariant is known to have been changed by the final candidate.** The Process-owned grant lifetime is a lifetime/custody rule for a separately granted Resolver, not derivation of resolver authority from `Process`. `noAddress` host diagnostic precision is explicitly documented without adding a new public error kind. The original owner-selected limits and deferrals remain in force.

## Validation and closure checks

```text
D223_OWNER_A1_APPROVAL=PASS_2026_10_29
D223_SECTION_14_9_EXACT_OWNER_REAPPROVAL=PASS_2026_10_29
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
NORMATIVE_PRODUCT_PUBLICATION=PASS
PROTOS_REVISION=aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9
PRODUCT_VERSION=0.3.421-SNAPSHOT
GLOBAL_SPEC_REVISION=0.1.480
IMPLEMENTATION_GIT_DIFF_CHECK=PASS_MAINTAINER_REPORTED
IMPLEMENTATION_MAKE_TEST=PASS_MAINTAINER_REPORTED
IMPLEMENTATION_MAKE_CHECK=PASS_MAINTAINER_REPORTED
REQUIRED_DURABLE_PUBLICATION=THIS_RECORD
CROSS_REFERENCES=EXACT_PRODUCT_SHA_AND_PRIOR_DOCS_SHA
```

The published product commit covers 56 paths; its four Resolver guest test sources are explicitly included in the standard Test Tool suite graph as 11 logical cases. On 2026-10-29 Guillermo reported clean `git diff --check`, `make test` and `make check` PASS. These gates are not rerun by the coordinating agent, and no independent live Internet DNS, native executable or cross-OS name-service test is claimed. [TEST010 #991](https://github.com/guillermomolina/protos/issues/991) separately owns a complete repository-wide corpus reachability audit; it does not reopen the verified I132 Resolver corpus membership.

This record completes the required **non-normative durable ratification** for the now-approved A1 contract at its exact product revision. At issue closure time, re-read this published file at its exact documentation SHA and record the verified `PROTOS_REVISION` + `PROJECT_RECORD_REVISION` pair in both live Issues. No new product commit is required solely to record an already-approved specification qualification.
