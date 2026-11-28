# D267-C1-R1 — Owner-approved structured HTML public contract

Date: 2026-11-28. Design owner: [D267 #1195](https://github.com/guillermomolina/protos/issues/1195); implementation: [I179 #1198](https://github.com/guillermomolina/protos/issues/1198); blocker B013. Product baseline [475101ed0af626b3f15b63869c5f1767bab40d0c](https://github.com/guillermomolina/protos/tree/475101ed0af626b3f15b63869c5f1767bab40d0c).

**Owner-approved exact R1 selection; product normative publication pending.** This is a durable non-normative decision record. The original [C1 architecture selection](D267_C1_STRUCTURED_HTML_COMBINATORS_SELECTED_2026_11_26.md) remains historical. R1 supersedes its pending contract questions to the extent explicitly answered below.

## Approval provenance

After the implementation agent stopped over missing limit accounting, attribute tables, relative URI rules and offline access to external records, the coordinator presented D267-C1-R1 in the active interaction. The owner explicitly replied on 2026-11-28: "apruebo D267-C1-R1". The owner separately authorized direct commit/push in this project-docs repository. The following records the selected proposal, not new implementation-agent decisions.

## Public API and representation

Authority-free synchronous ordinary Protos MODULE `std:html`, no Core/Prelude addition. Exports: `element(name, ...items)`, `fragment(...children)`, `doctype()`, `escapeText(text)`, `escapeAttribute(text)`, `render(root, limits: %{})`, `HtmlLimitError`, and the ordinary variadic tag-specific constructors below.

```protos
Html: import("std:html")
div: Html.div
h1: Html.h1
p: Html.p
document: div(%{class: "content"}, h1(title), p(description))
Html.render(document)
```

Constructor names (and admitted initial element names):
html, head, body, title, meta, link;
header, footer, main, nav, section, article, aside;
div, span, p; h1, h2, h3, h4, h5, h6;
a, img; strong, em, b, i, del, s, small, mark;
code, pre, blockquote; ul, ol, li, dl, dt, dd;
table, caption, thead, tbody, tfoot, tr, th, td;
br, hr, wbr; figure, figcaption, details, summary; input, label.

All nodes are ordinary objects: element has own slots kind: "element", name, attributes, children; fragment has kind: "fragment", children; doctype has kind: "doctype". Text is an ordinary String directly. Element name must be an admitted lowercase ASCII name. A normal standard Map in the first item position supplies attributes; otherwise all items are children. Attribute Maps elsewhere are not silently reinterpreted. Children preserve argument order; no implicit scalar/object/foreign conversion, Closure execution, raw HTML or trusted String escape hatch. No arbitrary named-argument capture.

Constructors produce fresh nodes and fresh attribute/child containers, freeze the node and those new containers, and do not mutate or freeze caller-owned objects. Freeze is shallow; externally supplied compatible descendants are not promised deeply immutable. Compatible ordinary structural data is admissible and validated by the serializer over the complete reachable structure. No privileged node family, authenticity registry, implicit parent scope, Canvas or template language. Security is treatment of untrusted data, not a sandbox against malicious guest programs redefining behavior.

## Limits

All inclusive; omitted keys default to their maximum values. Limits is a normal standard Map. Reject unknown keys, non-Integer values, explicit null, and values outside the specified ranges with ordinary Error. No conversion.

| Field | Default | Inclusive range |
| --- | ---: | --- |
| maxOutputBytes | 8388608 | 0–8388608 |
| maxDepth | 128 | 0–128 |
| maxNodes | 262144 | 1–262144 |
| maxWork | 33554432 | 1–33554432 |

The root counts once at depth zero; each parent-child edge adds one. Element, fragment, doctype and String occurrences (including empty Strings) count; attributes do not count as nodes. Shared subtree occurrences count independently. Active-path cycles signal Error; shared acyclic subtrees are admissible.

Logical work units are additive: one per node occurrence; one per attribute entry (even if omitted from output); one per Unicode scalar in element name, attribute name, String attribute value and textual child; one per emitted UTF-8 byte. No additional units for sorting, auxiliary data structures or implementation strategy. Mathematical Integer accounting without overflow/truncation; no wall-clock or host instruction accounting.

`HtmlLimitError` is an ordinary Error prototype with immediate parent Error, analogous to MarkdownLimitError. Excess signals a fresh occurrence. No partial/truncated result. Render-local state, deterministic iterative traversal, no unbounded source-dependent host recursion.

## Attribute contract

Names lowercase ASCII; no implicit name normalization. Global String attributes: id, class, title, lang, dir, role; also data- and aria- followed by one or more characters from ASCII a-z, 0-9, hyphen. Element-specific String attributes:

| Element | Attributes |
| --- | --- |
| meta | charset, name, content |
| link | rel, type, media |
| a | rel, hreflang, type |
| img | alt, width, height, loading |
| ol | start, type |
| li | value |
| th | colspan, rowspan, scope, headers, align |
| td | colspan, rowspan, headers, align |
| label | for |
| input | type |

meta.charset only "utf-8"; meta.name only "description", "author", "viewport", "robots", "keywords", "theme-color". meta has exclusively charset, or name and content, apart from admitted global attributes. link.rel only "stylesheet", "canonical", "alternate". input.type only "checkbox", with disabled obligatorily true. align only "left", "center", "right". Other admitted textual attributes accept Strings without claiming validation of every HTML microsyntax/content model.

Boolean attributes: all elements hidden; details open; ol reversed; input checked, disabled. Canonical true emits bare name, false omits it; String "false" is not a Boolean substitute.

URI attributes: a.href, link.href, img.src, blockquote.cite, del.cite. All require URI admission before attribute escaping. All other attributes rejected, explicitly including event handlers, style, srcdoc, srcset, http-equiv, action, formaction, target. No permissive alternative. Unsupported active elements and foreign markup outside the admitted element list are rejected.

## URI admission

Reuse existing std:uri strict RFC 3986 parser/formatter; do not modify it or introduce another parser. Admission adds a conservative browser-facing policy; no normalization or fetch authority. Admitted references are emitted with exact original spelling after HTML escaping.

Nonempty ASCII String, at most 2048 characters; no literal spaces, controls or backslash; reject malformed percent encodings and percent-encoded control octets. Absolute schemes only HTTP and HTTPS (ASCII case-insensitive); authority and nonempty host required, no credentials. Port absent or decimal 1–65535. Host restricted to conventional ASCII DNS labels, no empty labels, leading/trailing label hyphens, percent encoding, or final label interpretable as a number; IPv4, IPv6 and IDN outside initial policy. Relative references admitted when unambiguous: path-absolute, ordinary relative path (including literal ./ and ../), query-only, fragment-only. Reject network-path references starting //, empty reference, unsupported schemes, absolute references without authority. In path reject percent-encoded 2e, 2f, 5c, 25 (ASCII case-insensitive).

Examples admitted: https://protoslang.com/docs, http://example.com/page, /docs/guide, guide.html, ./guide.html, ../guide.html, ?page=2, #installation, docs/file%20name.html.
Rejected: empty, //example.com, javascript:..., data:..., file:..., mailto:..., https:example.com, https://user@example.com, https://example.com/%ZZ, %2e%2e/docs, /docs%2fother, /docs%5cother.

No RFC3986/WHATWG equivalence claim, destination authentication, automatic resolution, DNS, HTTP or other authority. WEB016 retains catalogue/revision identity, link rewriting/resolution and consumer-specific content policy, including inert raw HTML and images. A general renderer allowing external stylesheets/images is not itself a policy for untrusted Markdown or arbitrary untrusted node trees; callers must select their allowed destinations/contexts.

## Serialization

Compact deterministic String; attributes sorted lexicographically by ASCII name. Text escapes ampersand, less-than, greater-than as &amp; &lt; &gt;. Attribute additionally escapes double quote as &quot; and apostrophe as &#39;. Both escape U+000D as &#13; and reject U+0000; preserve other Unicode scalars without normalization, including leading U+FEFF. Already escaped-looking text is not privileged; ampersands are escaped again. Double-quoted String attributes. No pretty-print whitespace.

Void elements exactly br, hr, img, input, link, meta, wbr; reject any children, emit no end tag or XML-style slash. title accepts only String children. Doctype emits <!DOCTYPE html>, admissible only as root or first direct child of root fragment; never inside an element. If serialized pre content begins with LF, emit an extra LF to compensate for HTML parser initial-LF removal. Not a complete HTML content-model validator or arbitrary HTML sanitizer.

One ordinary Protos serializer, explicit traversal stack and render-local output accumulator. No shared mutable builder, new Java, native/builtin facility, parser/compiler/grammar change, hidden I/O, Future, Task or Actor. Preserve existing Core invocation, D237, optimized execution paths, Encoding, URI, Markdown and XML boundaries. No new language behavior.

## Publication boundary

The selection does not itself publish product semantics. Reconcile the dedicated normative Standard Library document (consistent with existing spec/semantics/MARKDOWN.md), normative index and next current specification revision under D267, independently of I179 production code. Verify current product main and B013 before releasing implementation. Supply the record and ledger text to the editing-only agent, not a request for external access. Any genuinely unresolved observable ambiguity exposed by normative transcription must be reported, not silently decided. No build, test, implementation or normative product publication is claimed by this record.

See [R1 evidence](../../evidence/D267/D267_C1_R1_OWNER_APPROVAL_AND_INVARIANT_REVIEW_2026_11_28.md).
