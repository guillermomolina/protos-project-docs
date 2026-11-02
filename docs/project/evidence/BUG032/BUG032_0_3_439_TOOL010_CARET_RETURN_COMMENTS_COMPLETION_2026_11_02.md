# BUG032 — TOOL010 caret non-local-return comment repair and closure evidence

- **Work item:** [BUG032 #1024](https://github.com/guillermomolina/protos/issues/1024).
- **Product:** `guillermomolina/protos`, exact published [commit `9a7234e4bd0f96293fb864b7249fc2dead770c6c`](https://github.com/guillermomolina/protos/commit/9a7234e4bd0f96293fb864b7249fc2dead770c6c) on `main`.
- **Implementation version:** `0.3.439-SNAPSHOT` (verified in the published `pom.xml` and `CHANGELOG.md`).
- **Date:** 2026-11-02.
- **Validation provenance:** maintainer-executed local commands reported in the interactive implementation discussion; the evidence recorder did not execute Maven, `make test`, or `make check`.

## Reproduced defect and bounded repair

The actual pre-fix source-layout test passed, confirming that valid comments after incomplete `^` were parsed and attached as `NON_LOCAL_RETURN_OPERAND`. The focused pre-fix formatter test failed with a real `IOException`: the bundled Protos-first TOOL010 formatter returned `state=FAILED` with a `Structural.protos` `Error().signal()` trace through the comment-admission path. This established a formatter defect, not a parser defect or an I138 regression.

The published Protos-first repair is restricted to `protos/tools/formatter/Structural.protos`. It formats caret-operand comments classified as `OWN_LINE`, `END_OF_LINE`, or `EMBEDDED_BETWEEN_TOKENS`, respecting original raw comment text, relative order, and blank-line separation without detaching the comment from the return operand. The `return` contextual spelling retains its separate same-logical-line operand rule; this fix does not permit `return` followed by a newline operand.

TOOL010 retains its established compact braced-Sequence policy for a single expression. Consequently the canonical closure example has the form:

```protos
f: () => { ^
    // keep return comment
    42 }
```

No grammar, parser, AST, source-layout Java production, Tool bridge Java production, runtime, or normative specification file was changed.

## Exact published product delta

The [published commit](https://github.com/guillermomolina/protos/commit/9a7234e4bd0f96293fb864b7249fc2dead770c6c) changes seven paths:

1. `protos/tools/formatter/Structural.protos` — Protos-first caret comment formatter and admissible boundary placements.
2. `src/test/java/com/guillermomolina/protos/analysis/ProtosSourceLayoutViewTest.java` — source attachment/placement regression.
3. `src/test/java/com/guillermomolina/protos/execution/integration/modules/ProtosFormatterToolBootstrapTest.java` — structural formatting, raw comments, combined placements, canonical equivalence, idempotence, I138 contextual-return guard.
4. `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerFormattingTest.java` — real document-formatting round trip.
5. `src/test/java/com/guillermomolina/protos/cli/ProtosBug032CliTest.java` — stdin/file output, read-only input, and `return` newline guard.
6. `pom.xml` — `0.3.439-SNAPSHOT`.
7. `CHANGELOG.md` — exact matching BUG032 entry.

## Maintainer-reported validation

| Gate | Reported result |
| --- | --- |
| Baseline source-layout regression | PASS |
| Baseline formatter regression before fix | ERROR, expected and captured (`Structural.protos`) |
| Structural formatter and contextual-return focal tests after fix | PASS |
| Combined caret comments and contextual `return` newline regression | PASS |
| CLI stdin/file and real LSP tests | PASS after correcting test expectations to the existing compact-brace style |
| Five affected JUnit test classes together | PASS (`mvn -q '-Dtest=ProtosSourceLayoutViewTest,ProtosFormatterToolBootstrapTest,ProtosLanguageServerFormattingTest,ProtosCliTest,ProtosBug032CliTest' test`) |
| Integrated `make test` | PASS (maintainer report) |
| Integrated `make check` | PASS (maintainer report) |
| `git diff --check` and staged `git diff --cached --check` | PASS (maintainer terminal output) |
| Seven-file product commit and push | PUBLISHED; exact revision independently read back from GitHub |

The first CLI and LSP expectation was overly expanded: the formatter correctly used a compact outer brace for the single return expression while retaining the intervening newline/comments. The tests were corrected without a further production change, then passed.

**Evidence limitations:** local test success is reported by the maintainer rather than independently rerun by this agent; raw full-suite logs, performance results, Native Image acceptance, a release tag and independent external review are not asserted. The changed product files, version/changelog and commit identity are GitHub-verifiable.

## Closure identity and authority

```text
WORK_ITEM=BUG032
ISSUE=guillermomolina/protos#1024
PROTOS_REVISION=9a7234e4bd0f96293fb864b7249fc2dead770c6c
IMPLEMENTATION_VERSION=0.3.439-SNAPSHOT
HUMAN_AFFECTED_JUNIT_CLASSES=PASS
HUMAN_MAKE_TEST=PASS
HUMAN_MAKE_CHECK=PASS
HUMAN_DIFF_CHECK=PASS
SPEC_CHANGE=NONE
```

GitHub Issue #1024 owns live closure, coordination and status. This project record is a revision-coupled, non-normative evidence snapshot; its own exact published commit SHA is linked from the Issue closure comment. The two exact repository revisions are required for cross-repository closure evidence.
