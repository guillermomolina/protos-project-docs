# D268 — Secure XML research and Candidate D approval

Date: 2026-11-26.

Owning issue: [D268 #1196](https://github.com/guillermomolina/protos/issues/1196).
Motivating consumer: [WEB016 #12](https://github.com/guillermomolina/protos-website/issues/12).
Research baseline: [Protos 1afc4ceeb4b6df9878fb2d2f168235277f8e7e70](https://github.com/guillermomolina/protos/tree/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70), verified as main during the investigation and approval checkpoint.

## Approval provenance and scope

In the active D268 research conversation on 2026-11-26, the maintainer explicitly stated: “ok apruebo el candidato D.” This selects **Candidate D — an ordinary Protos XML library with one incremental parsing kernel and optional document-tree materialization by composition**, as presented in the research packet. The owner additionally authorized this bounded repository-record publication.

This is an approved architectural direction, not a final ratification of the complete public XML contract. The research packet explicitly identified unfinished normative review, absent captured Atom fixture evidence, and unresolved public representation, limit, text-event segmentation and diagnostic contracts. Selection does not erase those qualifications. This record is non-normative; it neither grants implementation authority for unresolved semantics nor closes D268.

## Problem and existing evidence

The website currently parses the public GitHub Discussions Announcements Atom feed using NGINX njs XML. Its [adapter](https://github.com/guillermomolina/protos-website/blob/main/server/github-announcements.js) rejects DTD/entity declarations before calling the external parser. The [normalizer](https://github.com/guillermomolina/protos-website/blob/main/server/github-announcements-core.js) selects by Atom namespace, checks discussion URLs and dates, deduplicates and sorts; the [service](https://github.com/guillermomolina/protos-website/blob/main/server/github-announcements-service.js) limits HTTP body size to 2 MiB, examines at most 100 Atom entries through the normalizer, publishes at most 20, and uses fresh/stale cache and failure backoff. These application-specific contracts remain outside XML. The existing [normalizer tests](https://github.com/guillermomolina/protos-website/blob/main/tests/github-announcements.test.mjs) and [service tests](https://github.com/guillermomolina/protos-website/blob/main/tests/github-announcements-service.test.mjs) provide consumer parity cases but are not an XML conformance corpus.

The baseline library tree has no identified reusable XML module. Existing [JSON](https://github.com/guillermomolina/protos/blob/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70/protos/lib/json/json/module.protos) and [CSV](https://github.com/guillermomolina/protos/blob/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70/protos/lib/csv/csv/module.protos) are ordinary Protos precedents for incremental feed/finish, eager operations, terminal failure and TextReader/TextWriter composition. See [LIB003](../LIB003/LIB003_JSON_DESIGN.md) and [LIB009](../LIB009/LIB009_CSV_DESIGN.md).

## Comparative research retained from the decision packet

| System | Material lesson |
| --- | --- |
| [Python ElementTree](https://docs.python.org/3/library/xml.etree.elementtree.html) / [XML security](https://docs.python.org/3/library/xml.html#xml-security) | Convenient trees and incremental interfaces; parser/version security matters. |
| [Java StAX](https://docs.oracle.com/en/java/javase/25/docs/api/java.xml/javax/xml/stream/XMLInputFactory.html), SAX and DOM | Push, pull and tree interfaces have distinct lifecycle and materialization costs; avoid inheriting host factory configuration. |
| [Go encoding/xml](https://pkg.go.dev/encoding/xml) | Incremental tokens and structured decoding; strict mode does not enforce every namespace constraint. |
| [Rust quick-xml](https://docs.rs/quick-xml/latest/quick_xml/reader/struct.NsReader.html) | Namespace-aware incremental processing without compulsory tree retention; do not expose Rust-specific borrow lifetimes. |
| [.NET XmlReaderSettings](https://learn.microsoft.com/en-us/dotnet/api/system.xml.xmlreadersettings) | Explicit DTD policy and document/entity character limits. |
| [DOMParser](https://developer.mozilla.org/en-US/docs/Web/API/DOMParser/parseFromString) | Convenient tree model, but browser document/error/HTML behavior is not a suitable Protos contract. |
| [NGINX njs XML](https://nginx.org/en/docs/njs/reference.html#xml) | Direct consumer precedent, not a public Protos API template. |
| Protos JSON and CSV | Local ordinary parsers, explicit encodings and I/O authority, consumer failure and streaming lifecycle precedents. |

Relevant standards: [XML 1.0 Fifth Edition](https://www.w3.org/TR/xml/), [Namespaces in XML 1.0 Third Edition](https://www.w3.org/TR/xml-names/), [XML Information Set](https://www.w3.org/TR/xml-infoset/), [Atom RFC 4287](https://www.rfc-editor.org/rfc/rfc4287), [Sitemaps](https://www.sitemaps.org/protocol.html), [OWASP XXE prevention](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html).

Candidate A (keep XML external) was eliminated as a permanent solution; it does not supply the required reusable Protos facility. Candidate F (full DOM) was eliminated as disproportionate. The surviving scorecard, in order B tree only / C events only / D composition / E external backend, uses qualitative 1–5 scores, not measurements:

| Criterion | B | C | D | E | Rationale |
| --- | ---: | ---: | ---: | ---: | --- |
| Correctness | 5 | 5 | 5 | 4 | One validated grammar in D; E depends on backend configuration. |
| Protos alignment | 5 | 5 | 5 | 2 | Ordinary local objects and callbacks vs foreign machinery. |
| Present proportionality | 5 | 4 | 4 | 3 | B meets the immediate feed; D adds a small compositional surface. |
| Incremental growth | 3 | 5 | 5 | 4 | A tree-only kernel can require replacement for streaming. |
| Future resilience | 3 | 5 | 5 | 3 | D permits optional materialization and backend independence. |
| Scalability | 2 | 5 | 5 | 5 | B mandates retention of the complete tree. |
| Conceptual simplicity | 5 | 4 | 4 | 3 | D adds composition; E adds adapters/configuration. |
| Portability | 5 | 5 | 5 | 2 | D does not choose a host library. |
| Runtime/resource cost | 2 | 5 | 5 | 4 | D pays for tree only when requested. |
| Failure/operability | 5 | 4 | 5 | 3 | Eager atomic result and incremental prefix effects can be distinguished. |
| Reversibility | 3 | 5 | 5 | 3 | D preserves extension paths without public host coupling. |
| Evidence maturity/risk | 4 | 4 | 4 | 4 | Internal parser precedents and mature external systems; XML implementation unmeasured. |
| Total | 47 | 56 | 57 | 40 | Qualitative aid only. |

Confidence is high for architectural comparison, medium for resource costs and implementation maturity. The strongest objection to D is the security and maintenance burden of implementing XML correctly despite mature external libraries. Mitigation is a bounded profile, standards corpus, differential and adversarial tests, and measured resource behavior; an alternative backend can be evaluated later without changing the public semantics.

## Approved direction and invariant preservation

The selected candidate preserves the stated boundaries: ordinary Protos composition; one grammar implementation; optional eager document construction; namespace-aware semantics; safe DTD-free initial XML 1.0 profile; no implicit filesystem/network authority; no website-local substitute parser; no speculative Java infrastructure; no conflation of XML and HTML. No earlier owner-approved invariant is reopened. Existing String/Bytes, Encoding, I/O ownership, module isolation and Error semantics remain owned by their current specifications. There is no requirement for new Core syntax, value families, bytecodes, Prelude bindings, global parser state or host dependencies.

The following are design directions or candidate surfaces from the research packet, **not finalized contracts**:

- Candidate module: `std:xml/xml`; candidate `parse(text)`, `eventParser(consumer)`, `readEvents(textReader, consumer)`.
- Ordered mixed content and repeated elements, expanded names (namespace URI plus local name), prefix retained as representation metadata, attribute lookup by expanded name, scoped prefix declarations including default-namespace rules and duplicate expanded-attribute rejection.
- Strict semantic String input using existing explicit Encoding/TextReader composition; XML declaration metadata cannot retroactively prove the original byte encoding. Byte autodetection deferred; exact declaration and initial BOM policy still to be specified.
- Predefined entities and legal numeric references; no DTD/custom entity or external resolution; comments, processing instructions and CDATA parsed as data; XML-required line and attribute normalization, no general trimming.
- Parser-local state, deterministic chunk-boundary behavior, terminal failure, no replay of delivered events; tree result only after complete validation. No hidden close, acquisition, URI dereferencing, HTML sanitization, schema validation or execution.
- Explicit finite resource budgets; exact defaults, units, validation and precedence require a further decision. Transport byte limits remain distinct.
- Separate serialization phase for sitemap and other consumers; exact writer, construction and structural round-trip contracts remain open. No DOM, XPath, XSD, XSLT, canonical XML or automatic external resolution in initial scope.

Pay-for-use: events do not allocate a complete tree; eager callers need not manage callbacks; unrelated Actors and non-importing programs do not pay XML parser state costs. Growth: add tree queries, writing and byte adapters without replacing the parsing kernel. Deferring namespace identity or mixed content would force a data-model migration or lose data, whereas deferring XPath, DTD, XML 1.1 and canonicalization preserves options without preimplementing them. Future stress cases include large documents, many independent Actors, deep nesting, prefix rebinding, truncation, cancellation and non-local return, native image, alternative runtimes and sitemap generation.

## Remaining research and ratification gates

1. Complete the exhaustive applicable normative-document review required by GITHUB010. The chat inspected targeted portions of VALUES_AND_COLLECTIONS, MODULES, ERRORS, TEXT_IO, BYTE_IO and IO_CORE; full relevant-document coverage and Actor/control/cancellation interactions are not yet established. Reconcile any findings before normative text or final candidate approval.
2. Obtain a representative raw capture of the actual [GitHub Announcements Atom feed](https://github.com/guillermomolina/protos/discussions/categories/announcements.atom) with source, capture date, size and SHA-256. The prior investigation did not obtain the bytes; do not fabricate fixture or parity results.
3. Finalize the public node/name/event representation, equality/identity and mutation/transfer rules, namespace declaration preservation, query absence rules, exact event segmentation, normalization, encoding-declaration/BOM treatment, deterministic diagnostic and limit contracts, and non-local return/cancellation/reentry behavior. The proposed numeric budgets were illustrative, not selected defaults.
4. Establish conformance plan using the [W3C XML test suite](https://www.w3.org/XML/Test/) with explicit supported-profile classification, malformed/namespaces/adversarial tests, arbitrary chunk partitions, limit boundary tests, Actor independence, and differential tests. Serialization and its round-trip law require separately precise scope.
5. Present the exact remaining contract and GITHUB021 invariant/delta review for owner approval; publish the durable ratified decision and reconcile the normative owner(s) in Protos before releasing dependent implementation. Any implementation issue must use the I family, with normal formal allocation and hierarchy/dependency rules as applicable.

## Validation and publication boundary

This is research/approval evidence, not implementation, conformance or closure evidence. No product files, version, changelog, specification, code or tests were changed or executed during the research. No performance measurements or fixture PASS are claimed. The product baseline is exact. Research-stage approval provenance is explicit; the complete normative contract and D268 closure remain pending.
