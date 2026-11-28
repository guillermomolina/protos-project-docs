# D267-C1-R1 — Approval and invariant review

2026-11-28. [D267 #1195](https://github.com/guillermomolina/protos/issues/1195), [I179 #1198](https://github.com/guillermomolina/protos/issues/1198), B013.

## Exact selection and provenance

The maintainer selected C1 on 2026-11-26, recorded in the [original architecture decision](../../decisions/language/D267_C1_STRUCTURED_HTML_COMBINATORS_SELECTED_2026_11_26.md) and [comparative research](D267_C1_COMPARATIVE_RESEARCH_AND_OWNER_APPROVAL_2026_11_26.md). On 2026-11-28 the implementation agent reported missing approved details (limit field names/defaults/ranges/counting, attribute tables, relative URI admission, unavailable external record/ledger) and stopped without file edits, tests or builds. It reported one read-only git status outside the editing-only restriction, without state change. B013 was recorded at project-docs [add291b](https://github.com/guillermomolina/protos-project-docs/commit/add291b94d8a45f05c243a4f580f5a67c588c1cc).

The coordinator then proposed C1-R1, specifying the public API, ordinary node representation, exact resource accounting and errors, attribute tables, conservative URI admission, escaping, void elements, title, doctype and pre handling. The owner explicitly replied: "apruebo D267-C1-R1", and separately authorized commit/push in guillermomolina/protos-project-docs. The [R1 contract](../../decisions/language/D267_C1_R1_STRUCTURED_HTML_EXACT_CONTRACT_SELECTED_2026_11_28.md) records that selection.

## Revision-bound observations

Product main: [475101ed0af626b3f15b63869c5f1767bab40d0c](https://github.com/guillermomolina/protos/tree/475101ed0af626b3f15b63869c5f1767bab40d0c). Project-docs main immediately before approval publication: add291b94d8a45f05c243a4f580f5a67c588c1cc.

Inspected current spec/semantics/MARKDOWN.md: ordinary semantic trees, inclusive deterministic limits (1048576 original input scalars, 262144 AST nodes, depth 128, 8388608 logical text scalars), ordinary MarkdownLimitError, explicit authority boundary. HTML does not change Markdown parsing or its AST. Current ordinary Protos implementations in protos/lib/markdown/internal/Limits.protos, Parser.protos and Builders.protos furnish examples of iterative traversal, limits and output accumulation; these are precedents, not a dependency on private Markdown modules. Current protos/lib/uri/module.protos exports parse/format/resolve, with modular internal Parser.protos and Formatter.protos. Preserve RFC 3986 semantics; browser-facing admission belongs to HTML, catalogue-aware identity and projection policy to WEB016. Current spec/io/TEXT_IO.md section 15.4 confirms default matching BOM consumption; preserve leading content U+FEFF at the HTML conversion boundary.

## Invariant and delta review

- C1 value-returning combinators, ordinary Strings as children, ordered composition, no implicit builder context or raw trusted Strings: preserved.
- Exact R1 fixes limits, error type, admissible attributes and URLs, serialization details; these are new library behavior, not changes to Core call, argument association, capture, object, String, encoding or concurrency semantics.
- D237 declared named parameters and positional rest remain unchanged. No runtime/native family, Java classes, privileged recognizer, compiler/grammar changes or legacy execution fallback authorized.
- Distinguish structural admission, HTML contextual escaping, URI admission, and consumer-specific URL/catalogue and content policy. No sanitizer guarantee or authorization to fetch, no RFC3986/WHATWG equivalence.
- D266 owns Markdown parsing and entities; D268 owns XML. No parallel parser, automatic semantic conflation or hidden dependency on private library modules.
- Exact owner approval provenance is satisfied for R1; prior comparative work limitations remain historical. No claim of full renewed GITHUB010 audit or test/implementation success. Any genuinely unresolved observable ambiguity exposed during normative reconciliation must be surfaced, not invented.

## Remaining publication gate

Publish the dedicated normative HTML contract and reconciled specification index/revision in product, independently of I179 production code, via the human-executor workflow. Re-read exact publication against the approved record and ledger; then move B013 to READY and release I179. No product code, normative files, version or test results changed in this approval step. Supply the approved record and ledger excerpt directly to the offline editing agent.
