# D268-R1 — Secure namespace-aware XML Standard Library

Date: 2026-11-26.
Design owner: [D268 #1196](https://github.com/guillermomolina/protos/issues/1196).
Motivating consumer: [WEB016 #12](https://github.com/guillermomolina/protos-website/issues/12).
Product research baseline: [1afc4ceeb4b6df9878fb2d2f168235277f8e7e70](https://github.com/guillermomolina/protos/tree/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70), verified as main; specification revision 0.1.516.
Historical comparative research and architecture selection: [D268 research and approval](https://github.com/guillermomolina/protos-project-docs/blob/20b8f7736882a0c9ef6d6896b85614ac24ecec63/docs/project/work/D268/D268_XML_RESEARCH_AND_APPROVAL.md).

**Status:** exact D268-R1 recommendation explicitly owner-approved. This durable record captures that design selection; the normative Protos specification publication and implementation are not established by this document. Keep the product implementation gate closed until normative reconciliation and the applicable governance checks succeed.

## Approval provenance

On 2026-11-26, the maintainer first stated “ok apruebo el candidato D.” Following further research, the active conversation presented the 26-section exact D268-R1 packet, including the public API, record shapes, namespace and character rules, deterministic events, eight resource budgets, diagnostics, adapters, serialization, round-trip contract, normative integration, acceptance corpus and moving the real Atom capture to implementation acceptance. The maintainer replied “ok aprubeo la recomendación.” The same message expressly authorized bounded commit/push in this repository and requested an I-family implementation issue where necessary.

This approves the presented R1, not an implementation, new Core primitives, implicit authority, or future extensions. The earlier direction-only record remains historical evidence and its unresolved-public-contract statements are superseded by this selection. The packet disclosed that its normative review covered directly governing sections, not every chapter of every large normative document. No exhaustive whole-document coverage or conformance test result is manufactured by recording approval.

## 1. Profile and architecture

Implement in ordinary Protos, using one incremental parsing kernel, with complete-text parsing implemented as tree materialization from the same semantic events. No duplicated grammar, Java production infrastructure, new Core family, grammar, bytecode, Prelude slot, native XML library, ambient parser service, or new authority is required. Serialization is a separate kernel with the same structural/event vocabulary; eager serialization composes it. Each parser/writer has independent local state, and the MODULE is Actor-local under existing rules.

Accept exactly one document conforming to the DTD-free, non-validating profile of [XML 1.0 Fifth Edition](https://www.w3.org/TR/2008/REC-xml-20081126/) and [Namespaces in XML 1.0 Third Edition](https://www.w3.org/TR/2009/REC-xml-names-20091208/). Declaration absent or version exactly 1.0. Validate prolog/root/epilog, ordinary attributes, mixed content, comments, processing instructions, CDATA, five predefined entities and decimal/hex numeric references. Reject malformed syntax, XML 1.1 and other declared versions, DOCTYPE, DTD and custom or external entities. No fragments, recovery, validation against DTD/schema, external resolution, automatic XInclude, XPath, XSLT, canonical XML or HTML semantics. XML data never causes network, filesystem, URI dereferencing or execution.

## 2. Public API

Import `std:xml/xml`; product path `protos/lib/xml/xml/module.protos`. Exactly eleven exports:

| Export | Normal result |
| --- | --- |
| `parse(text, limits = null)` | Document |
| `eventParser(emit, limits = null)` | parser |
| `readEvents(textReader, emit, limits = null)` | reader adapter |
| `elements(element, namespaceURI, localName)` | fresh open Array |
| `firstElement(element, namespaceURI, localName)` | Element or null |
| `attribute(element, namespaceURI, localName)` | String or null |
| `textContent(element)` | String |
| `encode(document, limits = null)` | String |
| `eventWriter(emit, limits = null)` | writer |
| `writeEvents(textWriter, limits = null)` | writer adapter |
| `XmlError` | ordinary Error-delegating library prototype |

No additional exported constructors, types, configuration registry or implicit codec lookup.

## 3. Data records and queries

Public data uses ordinary Object-rooted records, standard Arrays and existing Core values. Record schemas (field names are exact):

| Record | Own slots |
| --- | --- |
| Document | kind="document", declaration, prolog, root, epilog |
| Declaration | kind="declaration", version="1.0", encoding, standalone |
| Name | namespaceURI, localName, prefix |
| Element | kind="element", name, attributes, namespaceDeclarations, children |
| Attribute | name, value |
| NamespaceDeclaration | prefix, namespaceURI |
| Text | kind="text", value |
| Comment | kind="comment", value |
| ProcessingInstruction | kind="processingInstruction", target, data |

Document declaration is Declaration or null; prolog and epilog are Arrays of comments and processing instructions only; exactly one root Element. Outside-root whitespace is validated and discarded. Declaration encoding is the declared String or null; standalone is true for yes, false for no, null for absent. Name localName and non-null prefix are NCNames; namespaceURI and prefix absence is null, not empty String. Non-null namespace URI is nonempty String. Attribute value, Text/Comment value and PI target/data are Strings. Element attributes, declarations and children are ordered Arrays; the original interleaving between declarations and ordinary attributes is not retained. Children contain Elements, nonempty Text, Comment, PI. Adjacent logical character data, references and CDATA combine into one Text run; empty CDATA does not create an empty node.

Every parser-produced record and Array is individually frozen before publication; no parser state, Closure, parent backlink or external capability is retained. Ordinary identity, equality, hashing and existing graph transfer apply. Two independent parses produce fresh Document identities. No new value-identity family or implicit deep-freeze mechanism. User-created structurally valid records are accepted by encode and queries; no hidden provenance token is required. Invalid structure needed for a query is rejected, and text traversal rejects cycles rather than looping forever.

Expanded-name equality compares exact namespaceURI and localName. elements selects direct matching child Elements in order, returns a fresh open independent Array of references. firstElement returns the first matching direct child or null; attribute returns matching String or null, distinguishing missing from empty. textContent concatenates descendant Text in document order, ignores comments and PI, adds no whitespace and performs no HTML interpretation.

## 4. Namespaces and lexical grammar

Apply all same-tag declarations before resolving its element and ordinary attribute names. Scope is lexical element nesting, with shadowing and restoration. Default namespace applies to unprefixed elements, never unprefixed ordinary attributes. Default clearing xmlns="" has prefix=null, namespaceURI=null; prefixed undeclaration is invalid in Namespaces 1.0. Namespace declarations are excluded from ordinary attributes. Enforce reserved xml and xmlns bindings and URIs: xml is prebound to http://www.w3.org/XML/1998/namespace; only xml may bind that URI; xmlns is reserved for declaration syntax and http://www.w3.org/2000/xmlns/ cannot be bound. Unbound prefixes, malformed QNames, duplicate lexical attributes/declarations and duplicate expanded ordinary attributes fail before startElement emission. Start/end tag lexical QNames must match exactly, even if different prefixes expand identically. Other prefixes beginning with case-insensitive xml are not rejected solely for that reason.

Namespace URI strings compare exactly after XML attribute normalization, without case folding, Unicode normalization, percent decoding, relative resolution or external access. Do not depend on std:uri for namespace validation or identity. Preserve prefixes and local declarations, including explicit legal xml binding, so QName-valued textual content is not silently changed. No automatic prefix generation or repair.

XML Char is TAB, LF, CR, U+0020–D7FF, U+E000–FFFD and U+10000–10FFFF. Validate literal and referenced characters. Name/NameStartChar/NameChar and NCName use the XML/Namespaces productions, not the Protos identifier grammar, locale, Unicode normalization or host regex categories. Decimal and lower-case x hexadecimal references require legal XML scalar values and complete semicolon-delimited grammar; predefined entities are amp, lt, gt, apos, quot only. Decode references once; replacement text is data, never re-lexed as markup.

Normalize literal CRLF and CR to LF before lexical processing, including across feed boundaries. For attributes and namespace values replace literal TAB and normalized LF with space; preserve referenced whitespace exactly; do not trim or collapse runs. Text references to CR remain CR. Comments and PI are inert; CDATA contributes ordinary text. No automatic whitespace stripping inside elements.

## 5. Encoding boundary

parse/feed accept exact semantic String, no coercion or Bytes overload. Use existing strict Encoding and TextReader for decoding, with caller-selected encoding and BOM behavior; XML String input itself does not consume an initial U+FEFF, which fails the prolog grammar. U+FEFF inside content is an ordinary legal character. Validate declaration EncName grammar and preserve spelling but do not claim to verify the original byte encoding. Byte autodetection, transport charset negotiation and byte/declaration consistency are deferred. A caller selecting replacement decoding does not gain a guarantee of byte-level XML validity from this parser. No implicit file or network acquisition.

## 6. Event parser

parser.feed(text) and parser.finish() return that parser; empty String feed is a no-op on an admitted open parser. finish declares EOF and validates one complete document, not merely a closed root. Exact event vocabulary, each fresh ordinary frozen record with individually frozen nested records/Arrays:

| kind | Other slots |
| --- | --- |
| declaration | version, encoding, standalone |
| startElement | name, attributes, namespaceDeclarations |
| text | value |
| comment | value |
| processingInstruction | target, data |
| endElement | name |

No document boundary, CDATA or entity reference events. Empty-element syntax emits startElement then endElement, as does the explicit empty pair. The event data can escape the callback and remains stable. Invoke emit(event) ordinarily and synchronously; normal return ignored, even a returned Future. No implicit await or invented eager callability primitive; actual invocation failure propagates normally.

Text is segmented into consecutive blocks of exactly 4096 decoded Unicode scalars, independent of feed partitions, entities and CDATA boundaries. Emit a complete block as soon as established; emit a residual nonempty block before the next completely validated non-text event. If invalid markup is established first, discard the unreported residual. No empty text events. Eager tree construction combines consecutive text blocks into one Text node. Concatenation and event order are invariant under chunking; do not batch validation of an entire feed ahead of earlier eligible events. Successful parse exposes a complete tree only after finish; callback effects previously delivered by an incremental parser are not rolled back.

Conceptual states OPEN, ACTIVE, FINISHED, FAILED. Admission validates state and semantic arguments before consuming input. An overlapping/reentrant call fails without disturbing the active operation if caught; uncaught callback failure abandons the active operation. Suspension retains ACTIVE and resumes the same operation without replay. On admitted operation normal completion, restore OPEN or finish terminally. On escaping Error, non-local return or cooperative cancellation mark terminal and preserve original transfer using ordinary ensure semantics. Bad input argument before admission does not consume input or poison the open parser. No reset, retry, resume, implicit yield, hidden tasks or automatic cancellation polling. Subsequent use after terminal state fails. The writer uses the same lifecycle.

## 7. TextReader composition

readEvents(reader, emit, limits) returns an adapter with read() -> Future<Boolean>. Each accepted read performs exactly one ordered reader.readText(): String (including empty if supplied by a compatible custom reader) feeds the parser and resolves true, null finishes it and resolves false. One outstanding read; reject overlap or use after terminal state. Error or cancellation of the accepted operation leaves adapter terminal; preserve original Error and ordinary Future cancellation semantics. Cancellation of a downstream continuation does not automatically cancel an upstream read Future or restore already-consumed input. No implicit readLine, close, flush, acquisition, encoding change, ownership transfer, rewind or retry. Reader custody remains with caller. Implement against actual Future pre-start cancellation, suspension, ensure, failure and adoption rules; do not assume then() runs on source failure/cancellation or that cancellation restores input.

## 8. Resource budgets

The exact eight positive Integer defaults selected in R1 (the preceding prose's reference to seven was a counting typo):

| Option | Default | Accounting |
| --- | ---: | --- |
| maxDocumentScalars | 2097152 | original source Unicode scalars before line normalization; writer emitted XML scalars including escapes and delimiters |
| maxDepth | 256 | simultaneous open elements, root=1 |
| maxAttributesPerElement | 256 | ordinary attributes in one tag |
| maxNamespaceDeclarationsPerElement | 128 | local declarations in one tag |
| maxNameScalars | 1024 | lexical name including prefix/colon |
| maxAttributeScalars | 262144 | normalized attribute value including namespace declarations |
| maxTokenScalars | 1048576 | lexical tags, comments, PI, declaration and references including delimiters |
| maxNodes | 250000 | element, logical contiguous text run, comment and PI appearances |

Document, Declaration, names and attributes are not nodes. Text blocks are not independent nodes. Ordinary character data and CDATA contents do not require single-token retention; delimiters and references remain bounded. Reference lexical cost inside an attribute also contributes to the containing start tag. Namespace declaration names and values obey name and attribute limits.

limits=null selects defaults; ordinary options record may supply any subset of the eight named slots. Unknown names, non-Integers or non-positive values fail before work. No unlimited sentinel. Snapshot option values at creation; later mutation cannot change the active operation. Count in logical order, not by entire supplied feed length; at failure stop without publishing additional events. Limit checks must precede allocation/retention beyond the accepted budget. Exact memory size or time is not promised; transport byte quotas are separate. Streaming retains only needed token, text block, scope/stack and counter state rather than a full tree. Avoid quadratic concatenation, repeated full namespace table copies and host recursion dependence. Eager materialization pays only when requested. Larger documents require explicit finite overrides.

## 9. Errors and diagnostics

One module-exported XmlError prototype directly delegates to Error; not Prelude or new Core category. Fresh occurrence owns kind, offset and limit:

- kind: syntax, namespace, unsupported, limit, argument or lifecycle.
- offset: zero-based raw String scalar offset before normalization; invalid literal at character, invalid reference at opening ampersand, malformed markup at token start, namespace or duplicate failure at tag start, mismatched close at end-tag start, unexpected EOF at input length, limit at point of established excess; null for argument/lifecycle/serialization errors.
- limit: exact option name for limit failures, otherwise null.

No normative natural-language message, line/column or source/host metadata. Ordinary binding/lookup failures and externally supplied consumer/I/O/Encoding Errors preserve their original behavior and identity; no automatic wrapping. Diagnostic positions and preceding events do not vary with feed boundaries. At each logical input step check total source, active lexical-token budget, character/grammar, name/value limits, namespace and duplicates upon complete tag, then structural budgets before emission; coincident structural checks order attributes, declarations, depth, nodes. Do not publish a start event before all checks pass.

## 10. Serialization

encode(document, limits) returns one complete String or Error, using the event writer; no partial successful result. eventWriter(emit, limits) returns writer with feed(event) -> same writer and finish() -> same writer. Same event vocabulary and structural state validation. Validate each event and all applicable output quotas before delivering its one nonempty String. finish validates complete document without additional output. Output uses explicit opening/closing pairs even for empty elements; no lookahead needed. Namespace declarations first, ordinary attributes second, each in Array order; verify supplied expanded name agrees with scope and lexical closing QName. No invented declarations or prefix repair.

Escape text ampersand, less-than and greater-than as amp/lt/gt references; CR as &#xD;; literal LF/TAB. Double-quote attributes, escaping ampersand, less-than, greater-than, double quote and literal TAB/LF/CR via amp/lt/gt/quot and &#x9;/&#xA;/&#xD; respectively. Do not generate CDATA. Preserve XML declaration presence, version and standalone; omit encoding because String serialization does not control eventual byte encoding. No implicit new declaration or BOM. Comment rejects double hyphen, trailing hyphen and invalid characters; PI target must be NCName and not case-insensitive xml, data must not contain ?> or initial separator whitespace; parser consumes initial PI separator whitespace. CR in manually constructed comments/PI is rejected because literal normalization cannot preserve it and references are not interpreted there.

Validate ordinary supplied records/Arrays, namespace coherence, character repertoire, structure and cycle absence; reject invalid graphs rather than serializing arbitrary Protos objects. Repeated acyclic node reference serializes each appearance and counts each appearance against the budgets. No preservation of Protos aliasing or object identity. Escaped output size may exceed input size, so output must meet its own budgets.

The approved round-trip is structural equivalence of profile-representable documents under selected budgets: expanded names, prefixes, local namespace declarations, attribute values/order, ordered mixed content, comments, PI, declaration presence/version/standalone. No guarantee for encoding metadata, original quoting, CDATA boundaries, character-reference spelling, empty-element syntax, outside-root whitespace or original lexical interleaving of attributes/declarations. The approved tree model uses one nonempty Text per contiguous run; it is not a lexical token stream. Normative reconciliation must preserve these restrictions rather than extending the accepted model or silently weakening equivalence.

## 11. TextWriter composition

writeEvents(textWriter, limits) returns adapter with feed(event) and finish() each returning Future<adapter>. One pending operation. Validate and serialize an accepted event into stable text before exactly one writeText(chunk), so later changes to the supplied record cannot change that write. finish validates completeness and performs one ordered writeText("") barrier, consistent with JSON precedent, not a flush, close, sync, encoder finalization or ownership transfer. No implicit retry or replay after uncertain output progress. Failure or cancellation of an admitted operation leaves the adapter terminal, retaining original Error/Future semantics; downstream cancellation is not a rollback of committed output. Use existing I/O authority and operation ordering, not a parallel XML I/O lifecycle or hidden host service.

## 12. Acceptance and implementation handoff

The implementation work is independent I-family work consuming D268. Suggested slices: normative reconciliation; lexical/incremental kernel; namespace processing; tree/queries; reader composition; serialization; writer composition; integrated acceptance. Promote to separate formal child Issues only if the coordination rules actually require it.

- Establish profile-aware manifest for [W3C XML Conformance Test Suite](https://www.w3.org/XML/Test/) release 20130923, retain origin/licensing, distinguish well-formed supported, malformed, namespace-invalid, DTD/XML 1.1 excluded, encoding outside text profile, validity-only requirements. Do not claim unrestricted XML conformance or blindly accept all valid DTD cases.
- Own tests: document/root, Unicode and exact XML names, namespace shadowing and duplicate expanded attributes, reserved bindings, references, CRLF/attribute normalization, comments/PI/CDATA, malformed delimiters and EOF, DTD/XXE, every budget N-1/N/N+1, 4095/4096/4097 text scalars, each single split of small inputs, character-at-a-time, empty feeds, reproducible random partitions, exact prefix events and diagnostics, reentry, suspension, non-local return, cancellation, terminal lifecycle, Actor independence and data transfer, writer escaping, cycles, structural round-trip, input/output adapter ordering and custody, large streaming input and output, JVM and installed Native Image parity.
- Differential comparison against Python/Expat, libxml2 and .NET within the common profile. Classify each divergence as PROTOS_DEFECT, REFERENCE_DIFFERENCE, PROFILE_DIFFERENCE or REPRESENTATION_DIFFERENCE; the standard is authoritative, not majority vote.
- Capture actual [GitHub Announcements Atom](https://github.com/guillermomolina/protos/discussions/categories/announcements.atom) bytes with URL, date, size and SHA-256, decode under explicit strict Encoding, test parser and consumer parity against the existing njs behavior. Prior research could not fetch the raw feed; website tests at [67d8e311452b98fa997d738cf5852c312e539e01](https://github.com/guillermomolina/protos-website/tree/67d8e311452b98fa997d738cf5852c312e539e01) use simulated njs records, not a captured XML fixture. Owner approved moving actual capture/parity to implementation acceptance; it is mandatory before WEB016 replaces the existing parser. Synthetic fixtures are not real capture evidence.
- [Atom RFC 4287](https://www.rfc-editor.org/rfc/rfc4287) text/html/xhtml interpretation, news URL/date validation, normalization, HTML safety and caching remain application policy. [Sitemap protocol](https://www.sitemaps.org/protocol.html) UTF-8 output and transport constraints remain caller policy. Test both consumer patterns, without embedding them in XML.
- No production Java infrastructure; if a genuine new runtime primitive becomes necessary, stop and establish its independent justification using existing Truffle languages, especially GraalPy, under the design gate.

Proposed normative owner: spec/semantics/XML.md; update spec/semantics/ERRORS.md library taxonomy, spec/PROTOS_LANGUAGE_SPEC.md ownership navigation, spec/PROTOS_SPEC_CHANGELOG.md global revision and applicable AGENTS.md normative map. No grammar change, no changed Core protocols. Recheck current main and next specification revision at publication. Normative text belongs in Protos, not this decision record; guide/examples and tests belong to implementation.

## 13. GITHUB021 invariant and delta audit

The earlier Candidate D direction is preserved: ordinary Protos, one incremental parser, optional eager tree, namespace awareness, no DTD/external entity resolution, no hidden I/O, no invented Java machinery, independent of Atom/HTML, composition with current protocols. No recorded owner invariant is reopened.

R1 adds the explicit observable module/export set; ordinary frozen records; expanded names with preserved prefixes; String-only/BOM rule; 4096-scalar deterministic segmentation; admission/terminal distinction; eight resource defaults; XmlError fields and precedence; borrowed adapters and serialization/round-trip contract. All were surfaced in the exact R1 packet before the second owner approval. Transfer uses ordinary graph semantics rather than a new value family; freeze is shallow at each existing object; errors retain non-resumable and identity rules; no new cancellation boundary or authority is implied. These additions constrain future implementation; do not infer approval of a different representation or behavior from the architecture label alone.

Comparative evidence, including eight systems/precedents, six architecture candidates, surviving 12-axis scorecard (B=47, C=56, D=57, E=40), strongest objection, scaling and deferral analysis, remains in the historical research record. R1 compared records vs behavior-bearing nodes, open vs frozen results, expanded vs lexical identity, pull/chunk-dependent vs deterministic block text, bounded vs unbounded input, generic vs library-specific errors, explicit vs automatic encoding and prefix preservation vs repair. Strongest counterargument remains security and maintenance cost of a new parser; mitigate by bounded DTD-free profile, standards corpus, differential/adversarial tests and independent backends only if later justified.

Normative source review included the directly governing parts of OBJECT_MODEL, CALLABLES, EXECUTION_AND_CONTROL, ERRORS (complete), VALUES_AND_COLLECTIONS, MODULES, ACTORS, FUTURES_AND_TASKS, PARALLEL_EXECUTION, TEXT_IO, BYTE_IO and IO_CORE, and the language specification, grammar ownership and current revision. Existing JSON/CSV implementations and design records supplied lifecycle and construction precedents; URI and website sources were examined. The packet explicitly did not certify complete reading of all unrelated sections of every extensive owner document; this limitation remains visible for GITHUB010 reconciliation.

## Publication boundary

This is durable evidence of the exact R1 selection, not product publication, final normative integration, or test evidence. No code, tests, spec files, version or changelog were modified in guillermomolina/protos by this coordination task. No conformance, fixture capture, performance or validation PASS is claimed. The previous packet made normative reconciliation and durable publication preconditions to D268 closure and dependent implementation; retain those gates, rather than silently treating this record alone as satisfying them.
