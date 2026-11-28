# I182 — Secure namespace-aware XML implementation and acceptance evidence

Date: 2026-11-28.

This is revision-bound, non-normative completion evidence for [I182 #1202](https://github.com/guillermomolina/protos/issues/1202), consuming [D268 #1196](https://github.com/guillermomolina/protos/issues/1196). GitHub Issues remain live coordination authority.

## Publication identity and decision authority

- Approved D268-R1: [selected contract and GITHUB021 audit](https://github.com/guillermomolina/protos-project-docs/blob/a3d39ce4d1fa3d4fdb6e52d9678acc8e9fe7c2df/docs/project/decisions/language/D268_R1_SECURE_NAMESPACE_AWARE_XML_SELECTED_2026_11_26.md), exact project-record revision `a3d39ce4d1fa3d4fdb6e52d9678acc8e9fe7c2df`; owner approval dated 2026-11-26.
- I182-A: [a556d7c442ed9c5d9b189846704e5a203b45143e](https://github.com/guillermomolina/protos/commit/a556d7c442ed9c5d9b189846704e5a203b45143e) establishes the normative contract, specification revision 0.1.520, ownership navigation and error taxonomy.
- I182-B: [348a2bdd55b7e4c82b4a7572914030f0c8104d4b](https://github.com/guillermomolina/protos/commit/348a2bdd55b7e4c82b4a7572914030f0c8104d4b) implements XML entirely in ordinary Protos.
- I182-C: [487494130365d8610feccf6902f57bb85b1fdca6](https://github.com/guillermomolina/protos/commit/487494130365d8610feccf6902f57bb85b1fdca6) publishes official suite projection, provenance, documentation and Test Tool integration, implementation version 0.4.257-SNAPSHOT; verified on main when inspected.
- Exact normative [XML.md](https://github.com/guillermomolina/protos/blob/487494130365d8610feccf6902f57bb85b1fdca6/spec/semantics/XML.md), [public guide](https://github.com/guillermomolina/protos/blob/487494130365d8610feccf6902f57bb85b1fdca6/docs/guide/library/xml.md), [offline corpus README](https://github.com/guillermomolina/protos/blob/487494130365d8610feccf6902f57bb85b1fdca6/protos/tests/conformance/library/xml/official/README.md) and [PROVENANCE.json](https://github.com/guillermomolina/protos/blob/487494130365d8610feccf6902f57bb85b1fdca6/protos/tests/conformance/library/xml/official/PROVENANCE.json).

The normative publication preserves the selected D268-R1 direction: no new Core type, grammar, Prelude binding, Java production XML infrastructure, host XML parser or implicit I/O authority. The earlier decision packet disclosed that its normative review covered directly governing sections, not every chapter of every large normative document. This record does not enlarge that claim or retroactively assert that the live design Issue was closed before implementation began.

## Delivered surface and acceptance boundary

The ordinary Actor-local MODULE `std:xml/xml` exports exactly eleven bindings: parse, eventParser, readEvents, elements, firstElement, attribute, textContent, encode, eventWriter, writeEvents and XmlError. One incremental parser drives eager tree materialization; one writer kernel supports eager serialization. Ordinary individually frozen records and Arrays preserve expanded names, lexical prefixes, local namespace declarations and ordered mixed content. Text segmentation is deterministic at 4096 decoded Unicode scalars, with eight configurable finite budgets. Borrowed text adapters compose existing Encoding and I/O protocols without taking ownership or acquiring authority. Original examples demonstrate Atom queries, incremental parsing and sitemap generation.

The selected profile is non-validating, DTD-free XML 1.0 Fifth Edition and Namespaces 1.0 Third Edition on semantic String input. No external resolution, custom entities, XML 1.1, recovery, XPath, schema validation or automatic byte decoding. This is not unrestricted XML conformance or W3C certification.

## W3C projection and reproducibility

Pinned [W3C release 20130923](https://www.w3.org/XML/Test/xmlts20130923.zip), archive SHA-256 `f9510b3532926e1b4c2e54855b021e4b8a66ec98a5337dcf4ff07e8a41968deb`. Inventory: 2,585 logical cases, 2,584 distinct inputs, zero unresolved classifications.

| Classification | Cases |
| --- | ---: |
| Supported accept | 74 |
| Supported reject | 237 |
| Profile reject | 1,672 |
| Outside String domain | 24 |
| Outside transport requirement | 4 |
| Outside selected XML edition | 574 |
| Executable total | 1,983 |
| Nonexecutable total | 602 |

The existing Test Tool executes one ordinary Test per executable full catalog identity. Projection produces 33 local sources of at most 64 cases (2 accept, 4 reject, 27 profile reject), with shared original Fixtures and Harness helpers. Eager, whole and partitioned parsing, empty feeds, diagnostic/event-prefix equivalence and encoder/eventWriter round-trips are checked. Python performs acquisition metadata, fixture projection and integrity checking, not Protos execution. Published tools reside under `tools/xml/` and support inventory, generate, check and cleanup. Progress grouping adds no new execution mechanism.

License review preserves historical contributor notices; no conclusion that the W3C general license supersedes all contributor restrictions. License SHA-256 `d4eedcb04dc16687b6d56b49ed06e1b55f599b8bd8e518af9773d1b7d2cf5c76`; XMLTEST and IBM notice hashes retained in PROVENANCE. Local-only fixture policy preserves all executable cases, including 360 XMLTEST cases. The maintainer verified that the manifest was restored, the 34 local W3C/Atom projections were removed, upstream acquisitions and work reports were moved outside the package tree, original helpers retained, and private evidence hashes matched. No acquired fixture or fixture-bearing generated source is published by I182-C. The private development Native ZIP is not a distributable artifact.

## Executed XML validation

All results below are maintainer-reported local executions, not tests independently executed during coordination.

| Test set | JVM passed | Installed Native passed |
| --- | ---: | ---: |
| Original XML tests | 105 | 105 |
| W3C executable profile | 1,983 | 1,983 |
| Differential structural tests | 74 | Not executed |
| Real captured Atom parity | 1 | 1 |
| Total | 2,163 | 2,089 |

Zero reported failures; 4,252 successful test executions across the two runtimes, not 4,252 distinct cases. JVM W3C plus own suite: 2,088/2,088 in 157.435 s (8 jobs); structural tests: 74/74 in 10.475 s; Atom: 1/1 in 67.948 s. Installed Native XML: 2,089/2,089 in 216.372 s, return code zero. GraalVM CE 25.6.5.1+2.1 / JDK 25.0.5.1. Earlier projection unit tests 60/60 and focal Java integration tests 27/27 reported PASS; final clean diff and all local tests PASS reported after consolidation. The owner did not supply separate final make check / make test-all logs in the publication handoff; no independent CI success or release is asserted.

Native validation used an installed private development distribution with candidate sources on baseline I182-B plus local I182-C edits, version 0.4.256-SNAPSHOT before final version bump. Native payload SHA-256 `48b575fb35bd2a678db167f7eb6fbd285222318b177f5edcb515f499fc38807d`; archive SHA-256 `7a66097504e18fa0ef7b3920c69664d26e9c6db727bd32c9f701309ec9e1768c`; log SHA-256 `eda69ff54a449f32dc2eae2bc3c69b8ac5512aca83a1c1c3bf866cc0f74f9cc2`. Initial extraction failed with a write error; the verified existing archive was installed using a different filesystem location, without recompilation. This is not a claim that the post-version-bump public release artifact has been built or tested.

## Independent reference comparison

311 common-profile documents (74 accept, 237 reject); 933 admission comparisons.

| Reference | Agreement | Divergence |
| --- | ---: | ---: |
| Python Expat 2.7.3 | 301 | 10 |
| libxml2 2.12.5 | 311 | 0 |
| .NET 10.0.15 | 294 | 17 |

27 admission divergences across 17 distinct cases were adjudicated against the standard, not majority vote: 26 reference differences (20 name-repertoire comparisons involving ten cases and six invalid encoding-name comparisons) and one profile difference (xml:space policy). 201 independent accepted-document structures (74 libxml2, 64 Expat, 63 .NET) were reconciled into 74 ordinary Protos structural Tests. Two structural differences were adjudicated: physical CR normalization in CDATA (reference difference) and omission of an explicit redundant xml namespace declaration (representation difference). No Protos defect identified in these comparisons. Declaration metadata was excluded from the differential structural projection, not from original XML tests. The 74 independent structural Tests ran on JVM only.

Exact reports are identified by their hashes in the final PROVENANCE block, including DIFFERENTIAL.json `a9ec319532927f6fe9385dc06b7418e909f123faa351de3b77ecd98a105fe2d4`, DIFFERENTIAL-DOTNET.json `2c7f16e1a1049e62035bb08005d3c398318e190a752b121ba14a79a12f3caca4`, STRUCTURAL-REFERENCES.json `0086d652eab8459957835667253eb0c52cff6a3bd827bdcb22caeaf510329c2b`, DIFFERENTIAL-ADJUDICATIONS.json `286fbc758de3d5ad7a409d83e8991d449d5e8ad7ccfb944229733da941b31162`. Raw reports remain local; hashes are identities, not redistributed content.

## Real Atom consumer parity

Actual GitHub Announcements Atom [source](https://github.com/guillermomolina/protos/discussions/categories/announcements.atom), 205,055 bytes, SHA-256 `51098074360646159a7de1e9857fbce15ee30d80fdf98b25af9285e04c0ce112`; HTTP 200, application/atom+xml; charset=utf-8, HTTP Date 2026-11-28T05:58:23Z. Independent local acquisition time was not observed. Strict UTF-8 projection and explicit Protos Encoding round-trip verified; not direct binary-file reading by Protos.

Reference execution used real NGINX 1.30.6 / njs 1.0.2, the pinned website image manifest `sha256:517607ae849a46afbd400616a783e4ead0290f52d20ccc0618ccea1a49362474`, and the three verified consumer blobs from exact [website revision 67d8e311452b98fa997d738cf5852c312e539e01](https://github.com/guillermomolina/protos-website/commit/67d8e311452b98fa997d738cf5852c312e539e01). A temporary export exposed the existing parseAnnouncements function without modifying its body. The real parser and consumer produced 25 source entries and 20 normalized announcements. Protos independently extracted the same 25 entries and produced exactly the same ordered 20 title/href/date/summary records in JVM and installed Native tests. Reference report SHA-256 `e3dc9163f809c1a21134c425bb4e668d24d0290f25474ffc8e02168a813ad727`. Test source SHA-256 `0e29bdd99e8e2344b073227b3f77d1d111d576dafeed5cef3ff564d6a54630f9`.

This satisfies the captured-feed implementation acceptance gate. It does not implement, deploy, or close [WEB016 #12](https://github.com/guillermomolina/protos-website/issues/12); full website migration and general application policy remain separate.

## Evidence reconciliation and coordination limits

At the exact published revision, PROVENANCE.json's `differential_and_installed_acceptance` block records the later executed results. Earlier `evidence` fields and prose in the guide/offline README still describe prior checkpoints as pending, and the redistribution-status field remains pending reconciliation; those historical/stale summaries must not be used to deny the later recorded results or imply a broad redistribution grant. A bounded product documentation reconciliation would improve consistency; this project-record publication does not edit the product repository. No change to licensing policy or XML semantics is requested.

Both formal Issues are independent top-level items (native parent endpoint returned no parent). Live dependency reads returned empty arrays for I182 blocked-by and D268 blocking. The historical textual dependency was not established natively in the observed live state; do not claim otherwise. The available interface can read but cannot establish that historical dependency edge or verify the derived Project dashboard. Normative publication is now established and no remaining technical D268 blocker has been identified. No fabricated native relationship or Project status claim.

Read-only coordination inspected the governing root/implementation/coordination/reference instructions, the project-docs AGENTS and role-first path contract, the exact approved D268-R1 record, both Issues, the three product publications, current XML specification, guide, offline corpus README, changelog and PROVENANCE. This record preserves the evidence and its limits rather than re-running product tests.
