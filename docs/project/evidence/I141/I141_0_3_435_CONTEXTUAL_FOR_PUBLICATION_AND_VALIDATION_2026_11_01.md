# I141 — Contextual generic/numeric for published as ordinary iteration sugar (0.3.435)

**Date:** 2026-11-01. **Work owner:** [I141 #1030](https://github.com/guillermomolina/protos/issues/1030). **Decision authority:** [D231-F2-PURE #1013](https://github.com/guillermomolina/protos/issues/1013), RATIFIED/CLOSED; its selected contract is [published in project-docs@043cab67](https://github.com/guillermomolina/protos-project-docs/blob/043cab67a915da912d3cd82910c6dd0373b9b817/docs/project/decisions/language/D231_CONTEXTUAL_FOR_F2_PURE_SELECTED.md). **Record role:** durable non-normative publication and verification evidence. The product spec is the language authority, not this record.

## Exact published source

- **PRODUCT_REPOSITORY:** `guillermomolina/protos`
- **PRODUCT_REVISION:** [`1c774003a2c697b7c2adae9967b562872b8d652a`](https://github.com/guillermomolina/protos/commit/1c774003a2c697b7c2adae9967b562872b8d652a)
- **PARENT_REVISION:** `2108e5ab2eacd1ebd0f64041b49d7245a414481e` (I140 0.3.434)
- **PRODUCT_COMMIT_TITLE:** `I141: implement contextual generic and numeric for as each/eachStep sugar (0.3.435)`
- **PRODUCT_VERSION:** `0.3.435-SNAPSHOT`, verified in product `pom.xml` and `CHANGELOG.md` at the published product revision.
- **NORMATIVE_SPEC_REVISION:** `0.1.490`, verified at product `spec/PROTOS_SPEC_CHANGELOG.md`; normative owner `spec/PROTOS_GRAMMAR.md` §36 and Compact EBNF.
- **SOURCE_PUBLICATION:** independently confirmed through GitHub that `main` pointed at exact PRODUCT_REVISION when this evidence was prepared.
- **CHANGESET:** 34 files (13 Java production, 15 test/conformance/manifest, 1 non-normative guide, 5 other source/spec/metadata/formatter), exactly as returned by the published commit.
- **CHANGE_CONTROL:** one atomic product commit; exact parent and file list verified, not inferred from a handoff report. No additional follow-on product edit claimed.

### Exact committed-file inventory

1. `CHANGELOG.md`
2. `docs/guide/04-control-flow-through-protocols.md`
3. `pom.xml`
4. `protos/tests/conformance/control/for-contextual-syntax.protos`
5. `protos/tests/conformance/manifest.tsv`
6. `protos/tools/formatter/Structural.protos`
7. `spec/PROTOS_GRAMMAR.md`
8. `spec/PROTOS_SPEC_CHANGELOG.md`
9. `src/main/java/com/guillermomolina/protos/analysis/ProtosDocumentSymbols.java`
10. `src/main/java/com/guillermomolina/protos/analysis/ProtosSourceLayoutView.java`
11. `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticDefinitions.java`
12. `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticHover.java`
13. `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticLint.java`
14. `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticReferences.java`
15. `src/main/java/com/guillermomolina/protos/execution/host/tooling/ProtosSourceLayoutToolBridge.java`
16. `src/main/java/com/guillermomolina/protos/parser/ProtosParser.java`
17. `src/main/java/com/guillermomolina/protos/parser/ProtosParserSourceFacts.java`
18. `src/main/java/com/guillermomolina/protos/parser/TokenCursor.java`
19. `src/main/java/com/guillermomolina/protos/parser/ast/SurfaceContextualFor.java`
20. `src/main/java/com/guillermomolina/protos/parser/ast/SurfaceExpression.java`
21. `src/main/java/com/guillermomolina/protos/semantic/Canonicalizer.java`
22. `src/test/java/com/guillermomolina/protos/analysis/ProtosSourceLayoutViewTest.java`
23. `src/test/java/com/guillermomolina/protos/analysis/ProtosStaticDefinitionsTest.java`
24. `src/test/java/com/guillermomolina/protos/analysis/ProtosStaticHoverTest.java`
25. `src/test/java/com/guillermomolina/protos/analysis/ProtosStaticReferencesTest.java`
26. `src/test/java/com/guillermomolina/protos/execution/integration/modules/ProtosFormatterToolBootstrapTest.java`
27. `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerDefinitionTest.java`
28. `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerDiagnosticsTest.java`
29. `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerFormattingTest.java`
30. `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerReferencesTest.java`
31. `src/test/java/com/guillermomolina/protos/parser/ProtosParserContextualForTest.java`
32. `src/test/java/com/guillermomolina/protos/parser/ProtosParserSourceFactsTest.java`
33. `src/test/java/com/guillermomolina/protos/parser/ProtosParserTrailingClosureTest.java`
34. `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerCallTest.java`

## Protos-first semantic evidence, not just syntactic resemblance

I141 implemented **three contextual forms and exactly three ordinary existing-message equivalents**:

~~~protos
for(item in expression) { body }         // expression.each(item => { body })
for(i, start, stop) { body }              // Range(start, stop).each(i => { body })
for(i, start, stop, step) { body }        // Range(start, stop).eachStep(step, i => { body })
~~~

- `SurfaceContextualFor` is a source/parse-only AST with one plain Closure parameter sourced from the header. The `Canonicalizer` lowers to **existing** `CanonicalLookup`/`CanonicalCall`/`CanonicalSend`/`CanonicalClosure`, no new canonical or Bytecode runtime loop. A lexical `Range` lookup and ordinary call evaluate the bounds and finish construction before step is evaluated. Existing I140/D236 chooses direction; `eachStep` validates strict step sign and equal-bound semantics.
- `for` and `in` remain identifiers and exact ungrouped bare `for` grammar is contextual. The explicitly ratified precedence over an ordinary callable named `for` applies only to the exact attached-block forms; grouped/member/body-less and other shapes remain ordinary.
- Ordinary receiver-selected `each` result, snapshot, callbacks, errors and nonlocal returns are preserved. One-parameter `for(key in map)` binds the key from Map's ordinary two-argument callback; no iterator or pair adaptation.
- The source fact/layout/bridge and static definitions/references/hover/lint/document-symbols preserve binder declaration, source spans, comments and ownership; the formatter itself is **Protos-first in `protos/tools/formatter/Structural.protos`**, retaining comments on the original side of each comma, `in`, parentheses and attached braces.
- Normative grammar and spec changelog plus existing guide, registered suite-native `control/for-contextual-syntax.protos`, parser/canonicalizer/JVM static/LSP/formatter tests were published together. No core `Range.protos`, `Object.call`, `ProtosBytecodeRootNode`, new LoopNode or Java range kernel was changed by I141.

## Validation and provenance boundary

**Human maintainer reported in the active interaction, after push:**
- `git diff --check`: **clean**.
- `make test`: **PASS (local)**.
- `make check`: **PASS (local)**.
- Earlier focal tests: **PASS (local)**; specific focal-selector totals, output logs and Native build/smoke outcomes are **not independently attached**.
- Product release: **pushed** with commit title above.

**Coordinator independently verified:** GitHub `main` at PRODUCT_REVISION; exact product commit SHA/parent/title/file inventory; version in `pom.xml` and `CHANGELOG.md`; global spec revision in `spec/PROTOS_SPEC_CHANGELOG.md`; bundled formatter and conformance source are committed. **Coordinator did not execute** Maven, suite-native Test Tool, make, git or Native smoke in the maintainer's local environment. Do not infer separate Native PASS or independently observed test counts. The suite-native file and manifest registration are source evidence, not a fabricated execution trace. Human-reported integrated make gates remain the attestation.

## Handoff and next independent work

Previous editing-agent/human executor handoff: [`protos-project-docs@f614233a`](https://github.com/guillermomolina/protos-project-docs/commit/f614233ac8d642bef8ac3c705bc1bef974419ca3). I141 product source is now published; any remaining education/migration is **independently allocated**, not a missing I141 requirement:

- [DOC016 #1032](https://github.com/guillermomolina/protos/issues/1032): guides, progressive standalone tutorials, cookbook examples, navigation; documentation content only.
- [I142 #1033](https://github.com/guillermomolina/protos/issues/1033): semantically safe idiomatic `for` adoption in first-party `protos/lib/**`, `protos/tools/**` and eligible ordinary tests; explicit `each` protocol and benchmark cases excluded. No overlap with DOC016 `docs/guide/**`, `protos/tutorials/**` or `protos/examples/**`.

I141 can close as **implementation completed** once this durable project-docs record is committed/pushed and read back at its immutable revision. These two successor issues remain OPEN/`status:ready`; neither is falsely claimed as implemented. No priority/assignee/milestone or native parent is invented.

```text
WORK_ITEM=I141
SOURCE_PUBLICATION=PASS
PRODUCT_REVISION=1c774003a2c697b7c2adae9967b562872b8d652a
PRODUCT_VERSION=0.3.435-SNAPSHOT
SPEC_REVISION=0.1.490
HUMAN_GIT_DIFF_CHECK=CLEAN_REPORTED
HUMAN_MAKE_TEST=PASS_REPORTED
HUMAN_MAKE_CHECK=PASS_REPORTED
SOURCE_COMMIT_INVENTORY=VERIFIED_34_PATHS
NATIVE_INDEPENDENT_TEST=NOT_CLAIMED
PROJECT_RECORD=THIS_EVIDENCE
DEPENDENT_FOLLOWUP_EDUCATION=DOC016
DEPENDENT_FOLLOWUP_SOURCE_STYLE=I142
```
