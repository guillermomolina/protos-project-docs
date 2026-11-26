# D267-C1 comparative research and owner approval

Date: 2026-11-26.  
Decision: [D267 #1195](https://github.com/guillermomolina/protos/issues/1195), [C1 architecture selection](../../decisions/language/D267_C1_STRUCTURED_HTML_COMBINATORS_SELECTED_2026_11_26.md). Implementation tracking: [I179 #1198](https://github.com/guillermomolina/protos/issues/1198).

## Evidence and scope

The initial investigation compared explicit escaping, structured HTML generation, contextual templates and restricted Markdown projection. The owner then explicitly requested a deeper investigation of structured generation because the proposed generic element/Map/text wrapper syntax was too verbose compared with Python's Dominate. The refined investigation distinguished:

| Candidate | Mechanism | Reported comparative score (/60) |
| --- | --- | ---: |
| C1 | Value-returning tag combinators | 58 |
| C2 | Explicit Canvas and Closure construction | 53 |
| C3 | Implicit construction scope | 36 |
| C4 | Combinators plus Canvas | 55 |

Scores are qualitative architecture estimates, not measured performance. C1 is the smallest selected structured solution: ordinary values, ordered children and one serializer; explicit Canvas is a possible later extension, not an approved requirement. C3 introduces implicit parent state or receiver/capture changes and is rejected as the initial direction. C4 would incur additional construction-state and unwind semantics before need is demonstrated. C1 may allocate a tree and be less ergonomic for imperative loops; future fragment/collection composition or a separately approved Canvas can address real needs without breaking the base model.

## Comparative precedents

- [Dominate](https://github.com/Knio/dominate): nested tag constructors and context-manager syntax; the expression-composition form is directly relevant to C1.
- [Swift HTML](https://github.com/pointfreeco/swift-html): concise tag functions construct ordinary node representations independently of serialization.
- [Lucid](https://hackage.haskell.org/package/lucid): typed HTML combinators and escaping on text conversion.
- [Seaside](https://github.com/SeasideSt/Seaside): Smalltalk Canvas/brush approach; relevant to C2, not a reason to require mutable construction state.
- [Kotlin type-safe builders](https://kotlinlang.org/docs/type-safe-builders.html) and [kotlinx.html](https://kotlinlang.org/docs/typesafe-html-dsl.html): receiver-based builders, which would not justify changing Protos Closure semantics.
- [Markaby](https://github.com/markaby/markaby): Ruby block DSL and implicit receiver model; useful ergonomics comparison, not adopted semantics.
- [WHATWG HTML](https://html.spec.whatwg.org/), [WHATWG URL](https://url.spec.whatwg.org/) and [OWASP XSS prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html): structural/contextual validity and URL policy remain separate from escaping.

## Product/source baseline

Protos revision: 1afc4ceeb4b6df9878fb2d2f168235277f8e7e70. Website revision inspected: 115a2d459e508f826ef4c08dc248597de54c3c8a.

Relevant inspected Protos material: AGENTS.md; AGENTS.work/DESIGN.md, REFERENCE.md and COORDINATION.md; spec/AGENTS.md and protos/AGENTS.md; relevant sections of spec/semantics/VALUES_AND_COLLECTIONS.md, CALLABLES.md, MODULES.md and ERRORS.md; spec/PROTOS_GRAMMAR.md; spec/io/TEXT_IO.md; protos/lib/uri/module.protos; protos/lib/json/json/module.protos; closure/module guides and D237 argument tutorials. The existing website renderer and its tests are [live-docs-markdown.js](https://github.com/guillermomolina/protos-website/blob/115a2d459e508f826ef4c08dc248597de54c3c8a/src/scripts/live-docs-markdown.js) and [live-docs-markdown.test.mjs](https://github.com/guillermomolina/protos-website/blob/115a2d459e508f826ef4c08dc248597de54c3c8a/tests/live-docs-markdown.test.mjs). D266 [#1194](https://github.com/guillermomolina/protos/issues/1194), D268 [#1196](https://github.com/guillermomolina/protos/issues/1196) and WEB016 [#12](https://github.com/guillermomolina/protos-website/issues/12) are adjacent boundaries.

GITHUB021 invariant/delta review for the selected direction: preserve ordinary value composition, strict D237 association, Actor isolation, RFC3986 authority, separate consumer URL/catalogue policy and inert source HTML; no new Core type, Java infrastructure, syntax, template/parser or hidden scope. The structural direction replaces the previously recommended string-escaping/direct-projection direction, without overriding security boundaries. Exact semantic details are not ratified.

## Approval and remaining evidence

The owner wrote "ok aprubeo C1." after the four-way comparison. This explicitly selects C1; it is not approval of unresolved exact API/semantic details. Full normative document coverage, complete historical decision audit, concrete adversarial/compatibility fixtures and exact public contract remain pending. No new build, test, benchmark, product code/spec change or runtime validation was performed. No GITHUB010 full research or GITHUB021 exact-contract closure PASS is claimed. Keep D267 open and I179 blocked until exact-contract approval and the required durable/normative gates are reconciled.
