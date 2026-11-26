# D266-D1 — Reusable Protos-first semantic Markdown parser

Ratified design: 2026-11-26.
Governing [D266 #1194](https://github.com/guillermomolina/protos/issues/1194).
PROTOS_REVISION: [1afc4ceeb4b6df9878fb2d2f168235277f8e7e70](https://github.com/guillermomolina/protos/tree/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70).
Website reference: [67d8e311452b98fa997d738cf5852c312e539e01](https://github.com/guillermomolina/protos-website/tree/67d8e311452b98fa997d738cf5852c312e539e01).
Earlier [research and architecture approval checkpoint](https://github.com/guillermomolina/protos-project-docs/blob/a2093b165c465c396a73711ec042cf77043b386e/docs/project/work/D266/D266_MARKDOWN_RESEARCH_AND_APPROVAL.md).

## Approval and authority

The owner initially approved D266-D on 2026-11-26. In the subsequent active interaction the agent presented D266-D1 as the complete exact candidate, including the public API, nodes, profiles, Unicode, spans, limits, failure behavior, implementation acceptance and explicitly deferred features. The owner responded **“ok aprubeo la recomendación.”** and requested issue reconciliation, evidence publication and I-family allocation, expressly authorizing commit/push to guillermomolina/protos-project-docs despite AGENTS.md. This records the actual approval statement, not an invented owner-authored GitHub comment. DECISION_APPROVAL_PROVENANCE=PASS.

This is the durable selected design record, not normative language text and not an implementation claim. The first implementation slice must reconcile the selected contract into the owning proposed spec/semantics/MARKDOWN.md, affected normative cross-references and spec changelog under the ordinary human-executed product publication discipline. No parser code may outrun that reconciliation. No product files are changed by this ratification. The design decision itself has a complete selected outcome; implementation and executable verification are distinct work.

## Exact selected public contract

One ordinary source-backed MODULE: import("std:markdown"), initializer protos/lib/markdown/module.protos, using existing D242 resolution. No new resolver, prelude binding, Java runtime facility, foreign dependency, Node.js or second parser authority.

```protos
Markdown: import("std:markdown")
document: Markdown.parse(source)
document: Markdown.parse(source, profile: "commonmark-0.31.2-gfm-extensions", positions: true)
```

Public operation signature: `parse(source, profile: "commonmark-0.31.2", positions: false)`. The source must be a genuine semantic String (String.recognizes); profile must be one of the two exact Strings; positions must be canonical true or false. No coercions. Existing positional/named/default association applies, including explicit null not selecting a default and ordinary source Closure excess positional behavior. Unknown profiles, invalid domains or explicitly disallowed null signal Error. No parseWith/options carrier/parser instance is needed. The other public export is MarkdownLimitError as specified below.

### Dialects

- `commonmark-0.31.2`: complete [CommonMark 0.31.2](https://spec.commonmark.org/0.31.2/).
- `commonmark-0.31.2-gfm-extensions`: CommonMark 0.31.2 plus the four parsing extensions in [GFM 0.29](https://github.github.com/gfm/) sections 4.10 (tables), 5.3 (task lists), 6.5 (strikethrough), 6.9 (extended autolinks). CommonMark 0.31.2 prevails on shared rules; explicit D266-D1 extension choices govern where stated. This is not a promise of exact GFM 0.29, remark or GitHub.com equivalence. GFM tagfilter is a projection policy, not syntax rejection.

Both profiles include all CommonMark block and inline constructs: paragraphs, ATX/Setext headings, thematic breaks, blockquotes, nested ordered/unordered lists, indented/fenced code, code spans, emphasis/strong, direct and reference links/images, delimited autolinks, escapes/entities, HTML blocks/inline, soft/hard breaks. No silent version upgrades or latest alias. Future semantic changes require an explicit profile or compatibility decision.

### AST

Each successful parse answers a fresh tree of ordinary open objects, with fresh open standard Arrays wherever arrays occur. Ordinary identity, equality, delegation and mutation; no new Core type or constructor family. Strings are ordinary immutable values. No shared mutable node identities, parent pointers, cycles, private parser references, automatic freezing, source-object lifetime or foreign resource ownership. No CST or round-trip promise. Each node has an own String kind and exactly the applicable own fields below (plus span only when requested). Children always an Array even when empty; leaves have no children. Adjacent text nodes in the same container coalesce; no empty text nodes. Order is semantic source order.

| kind | Additional own slots |
| --- | --- |
| document | children |
| paragraph | children |
| heading | level, children |
| thematicBreak | none |
| blockquote | children |
| list | ordered, start, tight, children |
| listItem | task, children |
| codeBlock | value, info |
| htmlBlock | value |
| text | value |
| emphasis | children |
| strong | children |
| codeSpan | value |
| link | destination, title, children |
| image | destination, title, children |
| softBreak | none |
| hardBreak | none |
| htmlInline | value |
| strikethrough | children |
| table | alignments, children |
| tableRow | children |
| tableCell | children |

Document, blockquote, listItem contain block children; paragraph/heading/emphasis/strong/link/image/strikethrough/tableCell contain inline children under the selected grammar; list contains only listItem; table contains tableRow; tableRow contains tableCell. No AST representation of reference definitions. All value/info/destination/title slots are Strings except explicitly nullable info/title.

- heading.level is Integer 1–6; no generated slugs/anchors.
- list.ordered and list.tight are canonical Booleans; list.start is Integer for ordered lists and null otherwise. Tight list paragraphs remain paragraph nodes. Delimiter spelling not retained. listItem.task is always present: null ordinary item, false incomplete task, true complete task. In the GFM profile recognize a valid marker in the first paragraph followed by whitespace and nonempty content; an isolated marker is literal. In the base profile task is null.
- codeBlock.value contains CommonMark code content with LF line endings including a final LF per content line even at physical EOF; empty block is empty String. info is null for indented blocks, empty String for a fence with no info, otherwise complete processed info string, not merely its first word. codeSpan.value follows CommonMark normalization.
- link/image.destination is the semantically recognized String with escapes/entities interpreted as required by the relevant Markdown construct, without URI normalization, added percent-encoding, percent-decoding, base resolution, authorization, existence checks or fetch. title null means absent; empty String means explicitly empty title. Image description remains inline children; consumer determines alt text. Link by reference has use-site children and resolved destination/title. First equivalent definition wins, forward/complete/collapsed/shortcut references resolve as required by CommonMark, unresolved uses follow ordinary Markdown interpretation.
- htmlBlock.value is recognized literal HTML content after removal of Markdown container prefixes where applicable and logical LF separation, without synthetic final terminator. htmlInline.value is recognized literal HTML token content. Neither performs HTML parsing, entity decoding within literal HTML, sanitization, evaluation or authorization. Adjacent HTML tags may be separate nodes according to CommonMark; text coalescing applies only to text.
- table.alignments is a fresh ordinary Array of left/center/right Strings or null, one per header column. First tableRow is header; delimiter row produces no node. Every row has exactly header-column count, missing cells synthesized empty, excess cells discarded. GFM escaped pipe handling applies including in code spans. No block content in cells.
- strikethrough recognizes matching one- and two-tilde delimiter forms according to GFM examples 491–493. Three or more tildes do not produce inline strikethrough; fence precedence preserved. Extended autolinks produce ordinary link nodes, not special node kinds.

### Unicode, input and data provenance

Source is a String, not Bytes or host text. Input count and positions are original Unicode scalar units. CR, LF and CRLF follow CommonMark logical line treatment; tabs follow its column rules; U+0000 is interpreted as U+FFFD. Existing U+FEFF in a String is retained as input; decoding and BOM consumption belong to Encoding. No input mutation, no implicit normalization, no host UTF-16 semantics.

Unicode properties and case folding are pinned to 17.0.0: CommonMark whitespace (Zs plus its specified exceptions); delimiter punctuation including P and S categories and standard ASCII punctuation rule; full default case folding C/F, excluding Turkic-specific alternatives, and CommonMark label whitespace rules. No NFC/NFKC or locale behavior. Use existing ordinary Protos capability where exact; otherwise generate and package the necessary Unicode data in ordinary Protos with one owning source of data. No new public general Unicode abstraction or Java primitive selected.

Named character references use the full semicolon-terminated HTML5 catalogue, 2125 entries, reproducible by the [cmark 0.31.2 reference table](https://github.com/commonmark/cmark/blob/eec0eeba6d31189fd828314576494566d539b1e3/src/entities.inc), blob e80d9aa9a4f83e2a062c06ed7a3075c031794b4e. Semantic source is [WHATWG named character references](https://html.spec.whatwg.org/multipage/named-characters.html); verify mappings and provenance before vendoring. Decimal numeric references have 1–7 digits; hexadecimal 1–6; NUL, surrogates and out-of-range values replace with U+FFFD. Unknown or unterminated named references remain literal. Apply CommonMark context-specific recognition, not a general HTML decoder; code remains literal. Do not copy C parser/runtime. Unicode source: [Unicode 17.0.0 UCD](https://www.unicode.org/Public/17.0.0/ucd/). Pin and verify generated data and licenses.

### Positions

With positions=false no node has span. With positions=true each node has own span, either a fresh ordinary object with Integer start/end or null for a synthesized node with no physical representation. Zero-based half-open scalar offsets in original String, not UTF-8/UTF-16, logical columns or grapheme clusters. CRLF occupies two positions; expanded tab maps to original tab.

| Node | Source extent |
| --- | --- |
| document | Entire original String, including blank lines |
| block nodes | Physical lines, from start of first to end of last excluding final line terminator; may include outer container prefixes |
| inline nodes | Recognized source syntax including own delimiters |
| reference link/image | Use site, not definition |
| tableRow | Corresponding physical line |
| explicit tableCell | Physical content between delimiters, including untrimmed whitespace |
| synthesized tableCell | null |
| coalesced text | Interval encompassing all contributing source extents |
| softBreak | Original line terminator |
| hardBreak | Marker and original line terminator |

No per-decoded-scalar source mapping or reversible desindentation guarantee. For "# 😀", heading span [0,3), text [2,3).

### Resources and failures

Four fixed inclusive limits per call:
- 1,048,576 original input scalars.
- 262,144 final AST nodes (including document, synthesized rows/cells).
- 128 parent-child edges from document (depth zero).
- 8,388,608 logical textual scalars summed over non-null value, info, destination and title fields of every node. Repeated shared destination contributes once per occurrence; excludes kind and span.

Validate arguments, then input count, then parse and check final structure. Limits measure the final semantic interpretation, not discarded provisional candidates; early detection only when excess inevitable. Check inevitable table amplification before allocation. No host timeout, Java properties, global budget, bytecode-count cutoff, or implicit Actor-wide quota. Website byte admission is independent. A valid in-budget source, even incomplete markup, has normal Markdown interpretation; unknown entities/unresolved references are not syntax errors. No partial successful result.

Module exports `MarkdownLimitError`, ordinary library prototype with immediate parent Error, not Core/prelude. Fresh occurrence for any of the four limit violations. No extra per-counter taxonomy. Invalid arguments/profile are ordinary Error. Cancellation, propagation, ordinary failure and unwind unchanged.

### Execution, security and implementation freedom

Synchronous ordinary Protos call, per-invocation working state, ordinary Actor-local MODULE identity; no new Tasks/Futures, global mutable parser state or authority. Returned data is subject to existing Actor/P transfer rules; no special serialization or transferable guarantee. No network, filesystem, evaluation, code execution, DOM, automatic URL or HTML authority. A syntactically valid javascript: destination or raw HTML is untrusted data, never an authorization. [D267 #1195](https://github.com/guillermomolina/protos/issues/1195) owns structured HTML generation/projection, escaping and safe policy; WEB016 owns catalog, GitHub source authority, link rewriting, caching and website admission. D266 does not require D267 implementation to progress; D267 must consume this representation rather than reparse Markdown. Explicit anchor handling remains with projection.

Recommended two-phase block recognition and reference collection, then inline parsing using delimiter/bracket stacks; equivalent conforming algorithms permitted. Avoid unbounded host recursion, repeated full-text scans and growing concatenations. Engineering target O(N log(N+1) + V + T) work and O(N + V + T) space for input scalars N, nodes V, logical output text T; preferably linear in input/output. Justify algorithm/collision behavior and measure adversarial growth; not a measured runtime guarantee. Use ordinary Protos collection/text/Encoding operations and distribution packaging. No new production Java, new host helper or FFI parser. If a truly missing capability is exposed, route it through proper owner/decision with verified Truffle/GraalPy precedent before any Java production proposal; do not smuggle it into this implementation.

### Intentional deferrals

No public events, incremental input, lossless CST/formatter, plugin framework, footnote extension, MDX, GitHub Alerts, Mermaid processing, math, syntax highlighting, front matter or advanced configurable limits. These can be separate additive capabilities or new explicit profiles; no reservation of ordinary CommonMark syntax. Incremental parsing would require a separately selected retention/commitment contract; no speculative infrastructure now.

## Required implementation acceptance

One separate I-family item, with bounded slices A–G:
A. Reconcile normative ownership and changelog, module public surface, pinned Unicode/entity data and provenance.
B. CommonMark blocks.
C. CommonMark inline and references.
D. Full CommonMark 0.31.2 conformance.
E. Four GFM extensions.
F. Positions, resource limits and adversarial robustness.
G. Actual corpus, JVM/Native parity, documentation, final integration acceptance.

- All 652 [CommonMark 0.31.2 official examples](https://spec.commonmark.org/0.31.2/spec.json) pass; no cherry-picking. Use a test-only conformance projection of AST to expected HTML, not a public renderer/second parser or safety claim.
- All examples for the selected four GFM extensions; record explicit profile-version or D266-D1 differences case by case, no blanket exclusion. Also direct AST assertions for shape, field presence/absence, order, freshness, mutation, text coalescing, references and synthesized cells.
- Unicode full-fold/category, supplementary scalars, combining characters, CR/LF/CRLF, tabs, NUL/BOM, entities and all 2125 mappings.
- For each limit, tests at limit−1, limit and limit+1, plus large amplified tables, reference fan-out, delimiter/bracket adversarial cases and URL parentheses; scaling measurements and allocation evidence. Do not silently accept host-stack overflow or truncate.
- Parse all 90 pinned corpus files (including five excluded by current website cap); independently run existing remark AST inventory and six renderer tests, compare semantic results and reconcile unintended differences, recording declared policy differences. No assumption of exact existing website HTML parity.
- Same semantic behavior on JVM and Native, no runtime C parser/Node/npm/FFI dependency.
- Use existing Protos documentation extraction and appropriate guide/examples. Pin upstream revisions, hashes, exact fixtures and licenses following existing TOML conformance provenance pattern. CommonMark/GFM spec corpus reference CC BY-SA 4.0, Unicode data Unicode-3.0, WHATWG applicable CC BY 4.0; verify actual source/license boundaries including cmark/Python-derived table provenance before incorporation. Preserve notices and licensing separately; do not relicense third-party corpus under Protos code license. No third-party corpus vendored in this record.

## GITHUB021 invariant and delta review

Prior owner-approved D266-D invariant | D266-D1
--- | ---
Ordinary Protos, no new Java or FFI infrastructure | Preserved
Single reusable syntax authority | Preserved
Complete CommonMark plus explicitly selectable GFM parsing | Preserved; exact extension scope/version fixed
Ordinary object/Array AST, no Core type | Preserved; exact slots/freshness fixed
Optional source positions | Preserved; scalar span schema fixed
Exact String semantics, explicit Unicode policy | Preserved; Unicode 17, folding/entities/local preprocessing fixed
Verifiable deterministic resource bounds | Preserved; four inclusive counters fixed
No implicit authority, separated HTML projection | Preserved
Actor-local MODULE and ordinary transfer | Preserved

All material refinements were presented for approval: single parse signature instead of provisional parseWith, profiles, exact node/field/null representation, positions, Unicode and entity data, four fixed limits, error prototype, deferrals and validation. No reopened contrary invariant. DECISION_INVARIANT_CONSISTENCY=PASS.

## Research and evidence boundary

The prior checkpoint records eight systems, six candidates, all twelve GITHUB010 dimensions, rejected alternatives, incremental design and strongest counterargument. This ratification supplements it; do not rewrite that historical checkpoint. The selected AST approach remains preferred to production JS, reduced private dialect, FFI cmark-gfm, events-first and full CST/plugins. The strongest objection is the effort of implementing full CommonMark compared with FFI. Accept the effort to retain Protos-native ordinary values, portability and single syntax authority; do not ship a reduced dialect mislabeled CommonMark.

Second read-only scan of all 90 sources at the exact product revision: no retrieval errors; 3,424,231 blob bytes, largest 259,013 bytes (258,846 scalars, VALUES_AND_COLLECTIONS.md); longest line 3,142 scalars (ABSTRACT_RUNTIME.md); maximum pipe characters on one line 9. Five batches of 18 files, line counts 13,357 / 4,619 / 4,364 / 24,567 / 22,822 (split on CRLF/CR/LF, including final empty split where applicable). One literal [^ prefix in docs/guide/library/regex.md inside an inline code example, no actual footnote marker identified; no Alerts markers, actual NUL, U+FEFF or U+FFFD detected. Earlier lexical scan: 3227 headings, 1661 fences, 503 explicit links, 257 quote lines, 124 table delimiter candidates. These are inspection counts, not semantic AST results. Profile scope does not depend on assuming that lexical counts are exact node counts. Source admission size has about 4x headroom over largest present document, structural limits still require actual corpus validation.

Read-only inspection confirmed existing CALLABLES named/default association, VALUES_AND_COLLECTIONS exact String scalar indexing and concat, TEXT_IO strict Encoding and BOM, MODULES ordinary isolation, ERRORS ordinary library error prototypes; D242 StandardLibraryModuleResolver resolves logical/path/module.protos without registry addition; JSON/TOML/URI ordinary parser precedents and D267-C1 structured HTML boundary. Full sources/heading coverage retrieved for relevant specifications but a complete manual line-by-line normative review of every extensive owner was not claimed. No human-executed remark inventory, builds, tests, performance measurements, local git diff --check or product publication have been reported. Executable validation belongs to implementation. This record ratifies the exact selected design and does not fabricate any implementation or conformance PASS.
