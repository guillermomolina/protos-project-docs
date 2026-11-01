# I141 — HUMAN EXECUTOR: one atomic D231-F2-PURE contextual for implementation

**IMPLEMENTATION, not research. Product repo: guillermomolina/protos.** Local checkout already points to intended HEAD; do not checkout an old referenced SHA. Formal [I141 #1030](https://github.com/guillermomolina/protos/issues/1030), owner-approved and CLOSED design [D231-F2-PURE #1013](https://github.com/guillermomolina/protos/issues/1013), immutable [selected contract](https://github.com/guillermomolina/protos-project-docs/blob/043cab67a915da912d3cd82910c6dd0373b9b817/docs/project/decisions/language/D231_CONTEXTUAL_FOR_F2_PURE_SELECTED.md). Existing foundation I140/D236 is **published and closed** at product revision 2108e5ab2eacd1ebd0f64041b49d7245a414481e (0.3.434); actual new version and spec revision come from current local HEAD, not from any pinned old SHA.

**ONE I141 slice only**, involving parser/Surface AST, ordinary canonical lowering, source layout/static tooling/LSP/formatter, normative grammar/changelog, conformance/JVM tests and focused docs. One final product commit/push only after tests; no I141-A/B separate issues or commits.

**Paired full editor prompt:** [I141_AGENT_ONE_SLICE_PROMPT.md](I141_AGENT_ONE_SLICE_PROMPT.md). Give the editing agent the **entire content** of that file. A link alone is insufficient: the editor MUST NOT access another repository or the web. The editor is strictly EDIT-ONLY; does not execute shell commands, Git, Make, Maven, tests, scripts, validations, checkout, fetch/pull, or publish product content. It may read local repo HEAD files in its editor. The human executes everything below.

## A. Human preflight — current checkout, never switch HEAD

~~~bash
pwd
git status --short
git log -1 --format='%H %s'
git diff --name-only
git diff --cached --name-only
~~~

Check concurrent work/dirty paths: preserve all unrelated edits and don't let the editor overlap another workstream. Read AGENTS.md, AGENTS.work/IMPLEMENTATION.md and relevant src/spec/protos AGENTS; use current HEAD's affected-file validation matrix. Confirm I141's design does not require another design decision: the three semantic equalities, grammar precedence and normal lookup/return/step behavior are already RATIFIED.

## B. After agent's ONE atomic edit, inspect exact file-level diff

~~~bash
git status --short
git diff --check
git diff --stat
git diff -- src/main/java/com/guillermomolina/protos/parser src/main/java/com/guillermomolina/protos/semantic/Canonicalizer.java src/main/java/com/guillermomolina/protos/analysis
git diff -- spec/PROTOS_GRAMMAR.md spec/PROTOS_SPEC_CHANGELOG.md
git diff -- src/test/java/com/guillermomolina/protos/parser src/test/java/com/guillermomolina/protos/analysis src/test/java/com/guillermomolina/protos/lsp
git diff -- protos/tests docs/guide
~~~

Review separately all other modified/untracked paths. For new files, read their entire contents before deciding to stage. Inspect external consequence: any new Surface type requires parser, source layout, static lint/definitions/references/hover/document symbols and formatting fact support. Ensure no new runtime CanonicalFor / LoopNode / Java iteration kernel, no changed Range or Integer semantics. Validate that generated canonical call order invokes Range constructor before evaluating step, and that canonicalization does not accidentally read binder i in outer scope. Confirm custom Range shadow is preserved and Map key-only behavior is specified/tested.

## C. Human-run cheap focal validation FIRST (only actual local supported invocations)

~~~bash
git diff --check
mvn -q -Dtest='ProtosParserTrailingClosureTest,ProtosParserSourceFactsTest,ProtosSourceLayoutViewTest' test
~~~

Inspect current local pom.xml, Maven settings and available relevant classes; adjust focal selectors to the tests actually changed by the patch. Add parser contextual for tests, canonicalizer correctness, complete emitted Bytecode execution, StaticDefinitions/References/Hover, LSP formatter/diagnostics, and relevant source fact/layout tests to the next targeted human command. Examples of existing class families to select from when affected: ProtosParserClosureTest, ProtosLanguageServerDefinitionTest, ProtosLanguageServerReferencesTest, ProtosLanguageServerFormattingTest, ProtosLanguageServerDiagnosticsTest, ProtosFormatterToolBootstrapTest. Do not claim they ran without an actual human result; use the project's documented invocation syntax and run the least-cost static and focal tests first.

**Protos suite-native conformance** must be run through the **existing registered** corpus/test tool using local HEAD syntax and inventory; do not invent a CLI command or execute a raw .protos source as if it were a suite. Verify all newly authored source test cases are registered and can be selected, are not duplicated or silently skipped. Cover generic Array/Map/custom/Range, step signs/direction/equal/error order, shadowed Range, the exact contextual collision and escapes, nested Closure ^ and contextual return/InvalidReturn/ensure, context and source spans.

If a focal gate FAILS, return exact diagnostic to the **same editing agent** and repair inside I141; rerun affected focal tests, then final integrated gates. No test/build/command invocation belongs to the editing agent.

## D. Integrated human validation once patch is stable

After high-signal focal checks and changed-file static/style/formatter/corpus audits PASS, execute the integrated gates required by HEAD:

~~~bash
make test
make check
git diff --check
~~~

If applicable to parser/runtime changes, validate Native CLI bootstrap and representative sample form using already available Native build/test targets **only when actually supported by HEAD**; do not invent target names or claim a Native PASS that was not executed. Source and spec style guards (and their impacted-base/head syntax) must come from local Makefile/AGENTS instructions. Capture test count/selector and failed cases if any. If expensive gates fail due to legitimate partial edits, same agent repairs in the same atomic patch; rerun focal/required integrated on final bytes.

## E. Human version/changelog and one atomic product commit/push AFTER PASS

Derive the next pom.xml version and CHANGELOG.md entry and spec/PROTOS_SPEC_CHANGELOG.md new global revision from **current local HEAD**, never reserve a guessed number. Include required version/changelog updates with source/spec/tests/guide and rerun any required validation invalidated by that metadata change. Verify correct APL Part 5 headers on new Java/Protos source, accurate guide snippets, no invented performance improvement. The human alone synchronizes/conflict-checks origin/main under repo policy before publication.

Stage ONLY explicitly reviewed patch-owned paths (adjust to real delta; some expected paths may be unchanged, and others genuinely needed):

~~~bash
git add src/main/java/com/guillermomolina/protos/parser/ProtosParser.java
git add src/main/java/com/guillermomolina/protos/parser/TokenCursor.java
git add src/main/java/com/guillermomolina/protos/parser/ProtosParserSourceFacts.java
git add src/main/java/com/guillermomolina/protos/parser/ast/SurfaceExpression.java
# Stage newly created SurfaceContextualFor.java if it exists; inspect its exact contents first.
git add src/main/java/com/guillermomolina/protos/semantic/Canonicalizer.java
# Stage each additional modified source-layout/static/LSP/formatter/test/guide path explicitly.
git add spec/PROTOS_GRAMMAR.md spec/PROTOS_SPEC_CHANGELOG.md
git add pom.xml CHANGELOG.md
git diff --cached --check
git diff --cached --stat
git diff --cached --name-only
~~~

Do NOT run git add -A or git add .; do not stage unrelated dirty content; do not force push or rewrite history. These example git add lines are **not** a command block to run blindly: select only paths actually changed by the reviewed slice, including any untracked new tests/AST files. Once every required human gate passes and staged paths are correct:

~~~bash
git commit -m "I141: implement contextual for as ordinary iteration sugar"
git push origin main
git status --short
git log -1 --format='%H %s'
~~~

Use one-line commit message without AI trailer and one source commit. Stop if concurrent main changes make a safe normal publication impossible; resolve without sacrificing user edits.

## F. Human final report to coordinating assistant

Return exact PRODUCT_REVISION full SHA, version and normative spec revision; complete changed-path list, focal parser/runtime/static/LSP/formatter tests + suite-native cases PASS/FAIL, git diff --check, make test, make check, Native PASS/NOT_RUN, style/registration check, one-line commit message and publication status. Human attestation is evidence; assistant must not invent local results.

Coordinator then verifies published product SHA/files and separately publishes a revision-coupled implementation/validation evidence record under guillermomolina/protos-project-docs, comments on I141 with all required GITHUB015/GITHUB020 closure postconditions and closes I141 only after verified product and docs publication. D231 remains closed as its separate design authority.

## Quick semantic audit (must PASS before publication)

| Case | Required |
| --- | --- |
| for(x in array) | Ordinary each snapshot and Array result |
| for(k in map) | Map each invokes callback with key,value; one-formal gets key; Map result |
| for(i,0,4) / for(i,4,0) | Asc 0..3 / desc 4..1 |
| for(i,4,0,-2) | 4,2; normal null |
| for(i,4,0,+2) | Error; do not flip sign |
| for(i,4,4,-1) | Error from empty ascending Range |
| for(i,invalidStart,stop,sideEffectStep()) | Constructor Error prevents evaluation of step |
| Range: custom then numeric for | Ordinary lexical Range resolves custom |
| for named user function | Bare exact syntax becomes contextual; (for)(...) and obj.for(...) remain normal |
| for(i,a,b) without braces | Ordinary call |
| Logical newline between ) and { | No contextual trailing body |
| Contextual binder i | Closure formal, not read of outer i |
| Non-local ^ / return / ensure | Same home and control flow as explicit each Closure |
| Formatter/LSP/source spans | Original contextual syntax and binder declaration retained |
