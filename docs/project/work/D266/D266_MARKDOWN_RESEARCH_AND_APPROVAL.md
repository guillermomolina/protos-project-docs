# D266 — Reusable Protos-first Markdown parsing: research and owner-approved direction

Date: 2026-11-26
Governing issue: [D266 #1194](https://github.com/guillermomolina/protos/issues/1194)
Product baseline: [1afc4ceeb4b6df9878fb2d2f168235277f8e7e70](https://github.com/guillermomolina/protos/tree/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70)
Website baseline: [67d8e311452b98fa997d738cf5852c312e539e01](https://github.com/guillermomolina/protos-website/tree/67d8e311452b98fa997d738cf5852c312e539e01)

**Status: architectural recommendation D266-D explicitly owner-approved; detailed public contract and final ratification pending.** This is a non-normative work record, not a normative specification or implementation authorization.

## Approval provenance and boundary

In the active D266 design investigation, following the full comparative packet and its explicitly provisional final recommendation, the project owner stated: **“ok apruebo la recomendación.”** The same message explicitly authorized bounded publication to guillermomolina/protos-project-docs and issue reconciliation. This is a record of the owner's actual statement, not a fabricated owner-authored GitHub comment.

The approved direction is a single reusable semantic Markdown parser, implemented in ordinary Protos, with full CommonMark 0.31.2 as the base, explicitly selectable GFM parsing extensions, ordinary object/Array AST, optional source positions, verifiable resource bounds, no implicit network or filesystem authority, no new Java infrastructure, and a distinct HTML projection boundary under D267. The default proposed profile is CommonMark; the extension profile is not a claim of exact GitHub.com or GFM 0.29 equivalence.

The presented recommendation explicitly left the exact API, options, limits, corpus AST verification and final ratification open. Approval of the direction does not silently settle those details. No implementation I item is released yet. No product files, tests, version or normative specification were changed.

## Motivation and scope

[WEB016](https://github.com/guillermomolina/protos-website/issues/12) needs substantive initial HTML for canonical GitHub documentation without a production JavaScript rendering dependency. D266 owns Markdown recognition and semantic representation, not the website, source acquisition, cache, HTML safety or URL authorization. [D267](https://github.com/guillermomolina/protos/issues/1195) independently owns safe HTML generation and Markdown projection; [D265](https://github.com/guillermomolina/protos/issues/1193) concerns module identity across Host workers, not parsing.

At the website baseline, [live-docs-markdown.js](https://github.com/guillermomolina/protos-website/blob/67d8e311452b98fa997d738cf5852c312e539e01/src/scripts/live-docs-markdown.js) uses unified 11.0.5, remark-parse 11.0.0, remark-gfm 4.0.1, remark-rehype 11.1.2, rehype-sanitize 6.0.0 and rehype-stringify 10.0.1. It converts raw HTML and images to text, applies catalog-aware link policy, and sanitizes the HTML tree. The six [renderer tests](https://github.com/guillermomolina/protos-website/blob/67d8e311452b98fa997d738cf5852c312e539e01/tests/live-docs-markdown.test.mjs) cover formatting, fences, tables, inert HTML/images, local and external link admission, unsafe references, catalog admission and size. They are parity evidence, not Markdown conformance.

The website catalog excludes blobs greater than 131072 bytes. This already excludes five real specification documents: PROTOS_GRAMMAR.md (171319), PROTOS_SPEC_CHANGELOG.md (178217), archived 0.1.300–399 changelog (146573), ABSTRACT_RUNTIME.md (205991), and VALUES_AND_COLLECTIONS.md (259013). The parser must support at least the largest current substantive document. Website byte admission and parser resource bounds are separate.

## Exact-revision corpus inspection

The Git tree at the product baseline was not truncated. All 90 Markdown files in the website's five source families were retrieved and lexically inspected in five batches of 18 with no reported retrieval error. This was an approximate scan excluding fenced blocks and inline code, not a CommonMark AST classification or executed conformance suite.

| Family | Markdown files | Git blob bytes |
| --- | ---: | ---: |
| spec | 37 | 2550445 |
| docs/guide | 45 | 821289 |
| protos/tutorials | 2 | 14686 |
| protos/examples | 5 | 36445 |
| protos/lib | 1 | 1366 |
| Total | 90 | 3424231 |

| Lexical feature | Candidate occurrences |
| --- | ---: |
| ATX headings | 3227 |
| Fenced code blocks | 1661 |
| Explicit links | 503 |
| Quote lines | 257 |
| Table separator lines | 124 |
| Autolink candidates | 27 |
| Raw HTML tag candidates | 8 |
| Task lists, footnote definitions, reference definitions, strikethrough, GitHub Alerts | 0 each detected |

Fence info strings: text 743; protos 481; js 298; sh 86; ebnf 37; java 10; other or empty 6. These are lexical counts, not assertions about exact syntax, nesting, AST sizes or absence of constructs. Actual remark AST inventory and website test execution remain pending human-executed validation. The explicit HTML anchor in [12-matching-expressions.md](https://github.com/guillermomolina/protos/blob/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70/docs/guide/12-matching-expressions.md) is not emitted as a functional anchor by the current raw-HTML-to-text transform; D267/WEB016 must reconcile safe anchor projection without enabling arbitrary HTML.

## Existing Protos contracts and precedents

- [VALUES_AND_COLLECTIONS](https://github.com/guillermomolina/protos/blob/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70/spec/semantics/VALUES_AND_COLLECTIONS.md), [TEXT_IO](https://github.com/guillermomolina/protos/blob/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70/spec/io/TEXT_IO.md), [MODULES](https://github.com/guillermomolina/protos/blob/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70/spec/semantics/MODULES.md), [ERRORS](https://github.com/guillermomolina/protos/blob/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70/spec/semantics/ERRORS.md): immutable exact scalar Strings, explicit encoding, ordinary collection identity and mutation, Actor-local modules, existing error propagation. Relevant sections inspected; exhaustive manual whole-document normative review has not been claimed.
- [D141](../../decisions/language/D141_STRING_COMPOSITION_MECHANISM.md), [D157](../../decisions/language/D157_STRING_INDEXING_UNIT_AND_GRAPHEME_CAPABILITY_PLACEMENT.md), [D252](../../decisions/language/D252_COLLECTION_TRAVERSAL_STRING_VECTOR_MATRIX_C1_RATIFIED.md): strict finite String composition and scalar indexing/traversal, no implicit normalization or grapheme semantics.
- [LIB003](../LIB003/LIB003_JSON_DESIGN.md), [LIB010](../LIB010/LIB010_TOML_DESIGN.md), [D087](../../decisions/tooling/D087_SHARED_BUNDLED_TOOL_TOML_BOOTSTRAP_AUTHORITY.md): format-specific ordinary Protos nodes, single parser ownership, no universal Node/serialization institution. JSON exposes whole-text and event operations; TOML exposes whole-document parsing and version selection. Existing TOML upstream corpus has license, provenance and checksums.
- [JSON module](https://github.com/guillermomolina/protos/blob/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70/protos/lib/json/json/module.protos), [TOML module](https://github.com/guillermomolina/protos/blob/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70/protos/lib/toml/toml/module.protos), [URI module](https://github.com/guillermomolina/protos/blob/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70/protos/lib/uri/module.protos): reusable ordinary implementations. URI is RFC 3986, not an automatic equivalent of WHATWG URL.
- Existing internal Unicode facts (ProtosRegexUnicodeFacility, ProtosIdnaUnicodeFactsFacility, UnicodeData17 and UnicodeNfc17) are not a public general Unicode API. Regex simple case closure is not full case folding for reference labels. Determine a reusable authority-free source of Unicode classification, folding and named entity data before implementation; do not smuggle a new private Markdown host facility into the design.

## Standards and prior art (GITHUB010)

Normative external baseline candidates: [CommonMark 0.31.2](https://spec.commonmark.org/0.31.2/) (2024-01-28); [GFM 0.29](https://github.github.com/gfm/) (2019-04-06); [WHATWG named character references](https://html.spec.whatwg.org/multipage/named-characters.html); [Unicode 17 case folding data](https://www.unicode.org/Public/17.0.0/ucd/CaseFolding.txt). The GFM specification has tables, task lists, strikethrough, extended autolinks and disallowed raw HTML; the last is a projection policy here. [remark-gfm](https://github.com/remarkjs/remark-gfm) additionally implements footnotes; neither GitHub.com behavior nor a particular library's claims are the normative dialect.

| System | Design family and lesson |
| --- | --- |
| [cmark](https://github.com/commonmark/cmark) | Reference C AST; separate renderers, pathological-input tests |
| [cmark-gfm](https://github.com/github/cmark-gfm) | C AST plus extensions; external differential oracle, native dependency tradeoffs |
| [commonmark-java](https://github.com/commonmark/commonmark-java) | JVM AST, optional source spans and separate renderer |
| [remark/micromark](https://github.com/micromark/micromark) | State-machine tokenization plus semantic mdast and transformations |
| [markdown-it](https://markdown-it.github.io/markdown-it/documents/Architecture.html) | Block/inline rule and flat token architecture |
| [markdown-it-py](https://github.com/executablebooks/markdown-it-py) | Token architecture with presets; security policies are separable |
| [pulldown-cmark](https://github.com/pulldown-cmark/pulldown-cmark) | Rust pull events and offset ranges; events need not imply constant-memory input streaming |
| [Goldmark](https://github.com/yuin/goldmark) | Go AST and source segments; source locations without full CST requirement |

Eight systems cover at least three different approaches. The original Markdown prose is too underspecified for the portable contract. No Truffle-platform work is proposed; a new host primitive, if genuinely required, must independently satisfy the GraalPy/Truffle precedent and governance gate.

## Alternatives and 12-axis scoring

Scores 1–5, in the order required by DESIGN.md. A: keep JS; B: reduced private dialect in Protos; C: FFI cmark-gfm; D: semantic Protos AST; E: public event-first parser; F: full CST and plugins.

| Axis | A | B | C | D | E | F |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Correctness/invariants | 4 | 2 | 5 | 5 | 4 | 5 |
| Protos alignment | 1 | 4 | 2 | 5 | 5 | 3 |
| Present proportionality | 2 | 3 | 3 | 4 | 3 | 1 |
| Incremental growth | 2 | 2 | 3 | 5 | 4 | 3 |
| Future options | 2 | 2 | 2 | 5 | 4 | 4 |
| Scalability | 3 | 3 | 4 | 4 | 4 | 3 |
| Conceptual simplicity | 3 | 3 | 3 | 5 | 3 | 2 |
| Portability | 1 | 5 | 2 | 5 | 5 | 5 |
| Runtime/resource cost | 3 | 4 | 5 | 4 | 4 | 2 |
| Failure/operability | 4 | 2 | 3 | 4 | 3 | 3 |
| Deferral/reversibility | 2 | 2 | 3 | 5 | 4 | 3 |
| Evidence/risk | 5 | 2 | 4 | 3 | 3 | 3 |
| Total | 32 | 34 | 39 | 54 | 46 | 37 |

Architectural alignment and portability confidence HIGH; complexity/implementation estimates MEDIUM; unmeasured Protos runtime performance LOW. Scores are judgment, not benchmarks. A is mature but retains the production runtime targeted for migration; defer/keep it as a parity oracle, not the final architecture. B appears cheaper but forces a dialect reconciliation and future rewrite. C has mature native performance but requires ABI packaging, ownership/custody and translation; its strongest case is early full conformance. D has an initial implementation cost but fits existing data-model and authority boundaries and leaves renderer/consumer options open. E enables sequential consumers but forward definitions require retention or delayed events; it is an additive later adapter, not mandatory today. F provides powerful lossless tooling at unjustified present complexity and retention cost. A/B have hard direction or underengineering concerns; F has an overengineering concern; totals cannot compensate for them.

## Approved direction and provisional contract details

A single ordinary Standard Library parser returns semantic document nodes; no Core family, host AST, global parser pool, filesystem/network access, implicit evaluation of code blocks, reflection-based universal serializer or private website parser.

Proposed module `std:markdown`, operations `parse(source)` and `parseWith(source, options)`; exact naming, option admission, default values, node schemas, error taxonomy and limits still require final reconciliation. Suggested profiles: `commonmark-0.31.2` and `commonmark-0.31.2-gfm-extensions`. Select four parsing extensions, with raw HTML retained as inert source data and tag filtering in projection. Do not equate the latter profile with exact GFM 0.29. Defer footnotes pending semantic corpus evidence, MDX, math, Mermaid execution, alerts, front matter, plugins, formatter/CST and incremental input.

Proposed ordinary fresh open objects/Arrays with kind discriminator and ordered children. Kinds include document, paragraph, heading, thematicBreak, blockquote, list/listItem, codeBlock, htmlBlock/htmlInline, text, emphasis/strong, codeSpan, link/image, softBreak/hardBreak, strikethrough, table/tableRow/tableCell. Preserve list start/tightness and task state; heading level, code value/info, link destination/title, table alignment. Resolve forward references using first-definition and dialect rules; no partial unresolved reference nodes required for a semantic tree. No special equality, implicit freezing or hidden lifetime. Optional source spans: zero-based half-open original String scalar offsets; synthetic nodes explicitly absent, not a promise of lossless reconstruction. These are proposals for the exact public contract, not ratified selectors or schema.

CommonMark is not JSON: unmatched delimiters and incomplete fences must follow Markdown interpretation instead of throwing syntax errors. Invalid arguments/profile/options and resource admission failures are separate. Proposed ordinary Error; no new Core taxonomy. No successful partial result on budget failure. Block parsing collects references before inline parsing; explicit stacks, bounded delimiter searches, avoid repeated concatenation/copies; deterministic source positions preserve CR/LF/CRLF and distinguish scalar from byte offsets. Unicode case folding/whitespace/punctuation and entity decoding depend on the exact dialect; code remains inert and does not undergo indiscriminate entity decoding.

## Stress, resource and growth analysis

Forward references disprove an irreversible single-pass constant-memory event API. Deep containers, unclosed brackets, long delimiter runs, fences, large reference sets, wide tables, long URLs, Unicode and node amplification require explicit adversarial tests. Initial study values, NOT normative: 1 MiB website UTF-8 input; 1048576 parser scalars; depth 128; 262144 nodes and cells. Define counting, error precedence, checked arithmetic and complexity bounds before selecting limits. Website input admission, parser retained state and HTML output budgets are distinct. Avoid global state, implicit Tasks/Futures, blocking coordination and foreign resource lifetimes. Independent invocations and Actor-local instances; ordinary cancellation and Actor/P transfer rules remain in force. JVM and Native must agree. No measured memory/CPU or cancellation claims yet.

Smallest sufficient solution is a complete-document semantic parser with opt-in GFM and source positions, not a general language/parser framework. Later events can traverse semantic structure; incremental input requires a separate retention/commitment contract. Lossless formatting would require concrete syntax information but not replacement of semantic parsing. More extensions can be added under explicit versioned profiles. A genuinely inadequate performance result warrants measured algorithm work before host primitives. The strongest argument against D is the time to achieve full conformance compared with FFI; accept the cost explicitly rather than silently ship a subset called CommonMark.

## GITHUB021 direction-level invariant review

| Invariant | Result |
| --- | --- |
| Ordinary objects/protocols; no new semantic family | Preserved |
| Exact Unicode scalar String semantics; no implicit normalization | Preserved; reference-label transformation is local explicit syntax policy |
| Actor-local module state, ordinary isolation and transfer | Preserved |
| No implicit authority or execution | Preserved |
| Protos-first, no invented Java infrastructure | Preserved |
| One parser authority, independent of consumer or renderer | Preserved |
| Canonical GitHub content and exact revision provenance | Preserved |
| RFC 3986 URI authority and distinct browser URL/HTML policy | Preserved |

No contradiction identified at direction level. Recheck exact candidate once API, resource and dialect details are resolved. No final GITHUB021 pass or final ratification is asserted on the basis of an architectural approval alone.

## Remaining evidence and gates

1. Human-executed AST inventory at the exact product revision using existing website remark-parse + remark-gfm, including all 90 documents, with node-type counts, task/footnote/reference/HTML examples, maximum depth and bytes; run the existing six renderer tests. The command was presented in the investigation; no output has yet been reported. Do not infer success from the lexical inventory.
2. Finish whole-document normative coverage and the exact public contract: profile/extension interaction and discrepancy policy, Unicode/entity source/version/licensing, options/schema and span/mutability/identity, precise budgets and errors. Explicitly reconcile D267 projection needs and any actual remark parity gap.
3. Record exact approval for any remaining substantive choices and final GITHUB021 review; publish and verify durable final ratification before closing D266 or releasing implementation. Allocate a separate I item for implementation, not another design item for the same unresolved contract.
4. Planned implementation validation: pinned CommonMark and selected GFM cases, semantic AST assertions, Unicode/entity/reference tests, large and adversarial inputs with growth evidence, resource boundary tests, 90-document parity, JVM/Native equivalence and D267/WEB016 integration. Upstream expected HTML examples require a controlled conformance projection distinct from website sanitization. Pin versions/SHA and record license, provenance and checksums; [CommonMark specification](https://spec.commonmark.org/0.31.2/) is CC BY-SA 4.0, do not assume the test tools or every upstream fixture has the same license. No third-party corpus copied by this record.

## Inspection and validation record

Materially inspected: root AGENTS.md; AGENTS.work/DESIGN.md, REFERENCE.md and COORDINATION.md; protos/AGENTS.md and spec/AGENTS.md; normative owners and source modules linked above; D141/D157/D252/D087/LIB003/LIB010; private Unicode facilities and lexer data; website package.json, live-docs-markdown.js and test, github-document-source-core.js, github-document-catalog-core.js, github-document-revision-core.js, live-docs-contract.js; exact Git tree and 90-document lexical inventory; related WEB016/D265/D267 and primary external standards/prior art. No repository-local builds/tests or benchmarks executed. This publication changes only a non-normative project record, contains no new source code or vendored corpus, and does not claim a human-executed git diff --check result.
