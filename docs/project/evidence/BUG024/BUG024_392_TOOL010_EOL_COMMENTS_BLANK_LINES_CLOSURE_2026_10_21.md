# BUG024 — TOOL010 end-of-line comments after blank lines: publication and closure evidence

Date: 2026-10-21. Non-normative, commit-bound evidence for
[BUG024 / guillermomolina/protos#939](https://github.com/guillermomolina/protos/issues/939).

## Immutable product identity

```text
WORK_ITEM=BUG024
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=b9513157e8def75d1a6f213b3c26d7c0b8d5dcee
PRODUCT_COMMIT=BUG024: fix TOOL010 end-of-line comments after blank lines (0.3.392)
IMPLEMENTATION_VERSION=0.3.392-SNAPSHOT
PRODUCT_CHANGED_PATH_COUNT=7
SPECIFICATION_CHANGE=NO
DESIGN_CHANGE=NO
DURABLE_RECORD_DECISION=REQUIRED
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

The exact published product commit is
[protos@b9513157e8def75d1a6f213b3c26d7c0b8d5dcee](https://github.com/guillermomolina/protos/commit/b9513157e8def75d1a6f213b3c26d7c0b8d5dcee).
The GitHub commit was independently read. It contains the formatter repair,
four affected JUnit test classes, `pom.xml`, and `CHANGELOG.md`:

```text
protos/tools/formatter/Structural.protos
src/test/java/com/guillermomolina/protos/analysis/ProtosSourceLayoutViewTest.java
src/test/java/com/guillermomolina/protos/execution/integration/modules/ProtosFormatterToolBootstrapTest.java
src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerFormattingTest.java
src/test/java/com/guillermomolina/protos/cli/ProtosCliTest.java
pom.xml
CHANGELOG.md
```

The product commit advertises `0.3.392-SNAPSHOT`, with a matching
BUG024 entry in the root changelog. No host source-layout implementation,
bridge, parser, CLI/LSP production routing, examples, or specification were changed.

## Failure, verified source facts, and repair

The incident arose while formatting a valid `std:runtime` example in VS Code
with installed `0.3.391-SNAPSHOT`: `textDocument/formatting` returned an
internal JSON-RPC failure and the bundled formatter reported
`state=FAILED; value=null`. Its original innermost guest diagnostic pointed
to the END_OF_LINE `blankLineBefore` rejection in
`Structural.protos:616` at the pre-repair revision.

The reduced *covered* regression has an expression after a blank line followed
by an end-of-line comment and another expression:

```protos
a: 1

b: 2 // note
c: 3
```

The existing tool-neutral source-layout projection correctly reports:

- the comment is `END_OF_LINE` and attached to the `b -> c`
  `SEQUENCE_SEPARATOR`;
- `comment.blankLineBefore=true`: the physical line immediately before
  the comment's line is blank;
- the `a -> b` separator contains the blank line, while `b -> c`
  has `blankLine=false`.

Previously, `formatSequenceBoundary` counted `blankLineBefore` for every
attached comment and also called `Error().signal()` if an END_OF_LINE
comment had that fact set. Both operations incorrectly attributed the blank
line *before the owning expression* to the later `b -> c` boundary.

The repair only changes bundled Protos formatter policy code: it no longer
aggregates `blankLineBefore` for `END_OF_LINE`, removes that unsupported
impossibility guard, and aggregates `blankLineBefore` only for `OWN_LINE`.
It retains `blankLineAfter`, `matched`, `pendingBlankAfter`, the
`blankLine == commentBlankLineSeen` consistency check, and existing
unsupported-layout fail-closed guards. No source-layout facts are falsified or
reclassified.

This implements the pre-existing
[D183 #791](https://github.com/guillermomolina/protos/issues/791)
formatting-preservation policy under the single bundled-Protos formatter
architecture of
[PLAT050 #792](https://github.com/guillermomolina/protos/issues/792).
It does not authorize a second Java/VS Code formatter or new public syntax.

## Regression and validation evidence

The verified product commit contains regression coverage in four classes:

1. `ProtosSourceLayoutViewTest` checks the physical before/after blank-line
   facts, `END_OF_LINE` placement, sequence-separator kinds, and exact
   preceding/following path attachment.
2. `ProtosFormatterToolBootstrapTest` checks eight focused formatting
   variants and the original four-line `std:runtime` metadata example.
   Its checks include exact canonical output, parse validity, raw comment text,
   placement/boundary/path preservation, and repeat-format idempotence.
   Variants cover multiple blank lines, own-line/adjacent comments,
   consecutive end-of-line comments, block comments, blank lines on the other
   side, and CRLF-to-LF normalization.
3. `ProtosLanguageServerFormattingTest` drives the real LSP formatter and
   checks the whole-document edit and zero subsequent edits after `didChange`.
4. `ProtosCliTest` formats stdin and reformats the result byte-identically,
   requiring successful return, exact stdout, and empty stderr.

The formatter integration harness also checks tool-process termination and
absence of a root filesystem; those checks are not independently represented
as a proof that arbitrary input code never executes. Existing isolation
regressions remain part of the test suite.

**Maintainer-reported final acceptance**, 2026-10-21:

- Focused JUnit run: **65 tests, 0 failures, 0 errors, 0 skipped**.
- All local tests: **PASS**, explicitly confirmed by the maintainer.
- Product `git diff --check`: **clean**, explicitly confirmed.
- The published product changelog separately records focused 65/65 PASS and
  integrated `make test` PASS.

The coordinating agent independently checked the published product GitHub
revision, changed-file set, formatter patch, and version/changelog diff.
It did **not** run local builds or tests and did not receive a retained raw
full-suite log. These results are therefore `MAINTAINER_REPORTED_PASS`,
not independently reproduced measurements. No new Native Image release,
actual installed-editor retest, or general LSP internal-error redaction
change is claimed.

## Publication and closure contract

The product publication is complete at the immutable `PROTOS_REVISION`
above. The requested project evidence is this snapshot record in
`guillermomolina/protos-project-docs`; its own commit identity must be
verified after publication and linked in the final GitHub Issue comment.
The most-specific live status and closure decision remain owned by
[BUG024 #939](https://github.com/guillermomolina/protos/issues/939).

BUG024 is a correctness repair to already-ratified behavior. It introduces no
normative contract changes, and no further BUG024 implementation slice is
identified by the published code/test results. The broader handling of truly
unexpected formatter internal errors, including how diagnostics are presented,
was not changed by this patch and must not be described as fixed.
