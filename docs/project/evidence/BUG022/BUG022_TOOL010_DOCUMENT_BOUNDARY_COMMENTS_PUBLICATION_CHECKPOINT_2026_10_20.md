# BUG022 — TOOL010 document-boundary comment repair: publication checkpoint

Date: 2026-10-20

This revision-pinned, non-normative evidence record concerns
[BUG022 / guillermomolina/protos#860](https://github.com/guillermomolina/protos/issues/860).
The Issue is the live closure/status authority; this record does not redefine D183
formatting policy, PLAT050 tooling architecture, or Protos semantics.

## Exact product publication identity

```text
FORMAL_WORK_ITEM=BUG022
ISSUE=guillermomolina/protos#860
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=d32524bd1dec045966dc33f3a3f58abc4dcf0278
PRODUCT_COMMIT=BUG022: fix TOOL010 document-boundary comments
PRODUCT_CHANGED_PATHS=5
PROTOS_POM_VERSION_AT_REVISION=0.3.387-SNAPSHOT
BUG022_CHANGELOG_ENTRY_AT_REVISION=ABSENT
PRODUCT_FINALIZATION=INCOMPLETE
DOCS_RECORD_ROLE=PUBLICATION_CHECKPOINT_NOT_CLOSURE
SPECIFICATION_CHANGE=NO
```

The exact source revision is
[protos@d32524bd1dec045966dc33f3a3f58abc4dcf0278](https://github.com/guillermomolina/protos/commit/d32524bd1dec045966dc33f3a3f58abc4dcf0278).
The coordinator inspected the published main commit, its five changed paths,
`pom.xml`, and the root `CHANGELOG.md`.

## Defect and mechanism

The earlier bundled canonical formatter rejected an ordinary file-leading
block comment (and the block APL license header in
`protos/examples/basics/slots.protos`) with exit 70 and
`bundled TOOL010 structural formatter did not return String; state=FAILED`.
The failure originated in the `formatBody` unsupported-comment guard in
`protos/tools/formatter/Structural.protos`.

The published product commit classifies comments that precede the first
code token as `DOCUMENT_PREFIX`, and comments after the last code token as
`DOCUMENT_SUFFIX`, in `ProtosSourceLayoutView`. The bundled Protos
`Structural.protos` policy renders these boundary comments while preserving
raw comment text, line placement, ordinary blank-line relationships, and
the existing internal sequence/embedded/trailing-Closure routes. Unhandled
comment placements remain fail-closed. The source-layout bridge continues to
supply immutable projection data; the editor and CLI share the TOOL010 authority.

The commit changes exactly:

```text
protos/tools/formatter/Structural.protos
src/main/java/com/guillermomolina/protos/analysis/ProtosSourceLayoutView.java
src/test/java/com/guillermomolina/protos/analysis/ProtosSourceLayoutViewTest.java
src/test/java/com/guillermomolina/protos/execution/integration/modules/ProtosFormatterToolBootstrapTest.java
src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerFormattingTest.java
```

## Maintainer-reported verification

The maintainer reported these successful gates in the 2026-10-20 handoff:

- Initial focused Java/JUnit batch: **39 tests**, 0 failures, 0 errors.
- Expanded focused Java/JUnit batch (source layout, real bundled TOOL010
  formatter/bootstrap, standard LSP formatting): **41 tests**, 0 failures,
  0 errors, 0 skipped. Cases include prefix and suffix block comments,
  consecutive comments, real `slots.protos` APL header, idempotence,
  source-preservation projections, canonical semantic AST, and LSP.
- `mvn package -DskipTests`: **BUILD SUCCESS**, exit 0. This is an artifact
  build, **not** integrated test validation.
- Fresh built checkout CLI: `printf '/* probe */\\nx: 10\\n' | bin/protos format`
  printed the preserved block comment and `x: 10`, **exit 0**.
- Fresh built checkout CLI:
  `bin/protos format protos/examples/basics/slots.protos`:
  **exit 0**, APL header preserved in the emitted source.
- Integrated `make test`: maintainer reports **PASS**.

The earlier CLI attempts during this repair returned exit 70 even after the
focal Java tests passed; rebuilding the checkout JAR with
`mvn package -DskipTests` resolved that mismatch. Those prior failures are
not evidence of a remaining product-source failure, and the unrebuilt
JAR should not be treated as a valid post-patch CLI acceptance run.

The coordinator did not independently execute the maintainer's local tests
or acquire complete integrated-suite logs. These are explicitly
maintainer-reported outcomes, not reconstructed CI telemetry.

## Publication discrepancy and required follow-up

**BUG022 must remain OPEN.** The exact published product commit contains
the five substantive files but **does not** contain `pom.xml` or the
root `CHANGELOG.md`. At this revision the project version remains
`0.3.387-SNAPSHOT` and the latest changelog section is I115-F, not BUG022.
This violates the publication-metadata requirement of
`AGENTS.work/IMPLEMENTATION.md` for changed executable source under `src/`.

Required next action in `guillermomolina/protos`, under the human-executor
workflow: derive the then-current next implementation patch version from
the live checkout, publish the `pom.xml` increment and corresponding English
root `CHANGELOG.md` entry, avoid folding unrelated concurrent work, and
verify the resulting exact product revision. The maintainer has prohibited
running tests after these version/changelog-only edits. Previously reported
test outcomes remain attributed to the validated source candidate; assess
whether any intervening executable/dependency changes invalidate that evidence.

When the missing product metadata is actually published, append a distinct
revision-pinned finalization record or reconcile this checkpoint with an
explicit successor record. Only then post a compact Issue closure comment,
record both exact product and project-record revisions, and close BUG022.
Do not retroactively describe the five-file commit as version-complete.
