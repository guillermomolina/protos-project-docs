# LM012-C1 — Published static lint CLI/CI implementation and human validation

**Outcome: PUBLISHED; maintainer-reported local tests PASS, `git diff --check` CLEAN.**

**Work item:** [LM012/#671](https://github.com/guillermomolina/protos/issues/671), slice **LM012-C1**.
**Decision:** [D195/#843](https://github.com/guillermomolina/protos/issues/843), [ratified single-source `protos lint` contract](../../decisions/tooling/D195_SINGLE_SOURCE_LINT_PUBLIC_CLI_CI_CONTRACT.md). D195 and [D194/#842](https://github.com/guillermomolina/protos/issues/842) are already **closed as decisions** at this checkpoint; their existing native parent/closure evidence is not superseded by this record.
**Exact product SHA:** [`a326eb537ce2dc4dc20287300f2b6fd677e53e75`](https://github.com/guillermomolina/protos/commit/a326eb537ce2dc4dc20287300f2b6fd677e53e75).
**Product version:** `0.3.305-SNAPSHOT` as shown in this commit's `pom.xml` and `CHANGELOG.md`.
**Commit subject:** `LM012-C1: static protos lint CLI/CI command (D195)`.
**Historical predecessor:** `129d41785bbcad9d25da03abbcdf0a7f0a5fcf8a` (LM010-C). The compare to `a326eb5` shows exactly one new product commit, changing the seven paths below.
**Evidence authored:** 2026-10-08; read-only GitHub commit/file inspection and active maintainer report. No fresh local commands, builds or test runs by the coordinating agent.

## Exact published delta

The GitHub compare `129d41785bbcad9d25da03abbcdf0a7f0a5fcf8a..a326eb537ce2dc4dc20287300f2b6fd677e53e75` identifies:

| Path | Published change |
| --- | --- |
| `src/main/java/com/guillermomolina/protos/cli/ProtosLintCommand.java` | Added ~261-line D195 command adapter: stdin/one explicit regular file or regular symlink, strict UTF-8, one canonical parse and approved checks, text/JSON v1 reports, exit 0/1/2/3, operational failures only on stderr. Unexpected internal failures are handled by ProtosCli exit 70. |
| `src/main/java/com/guillermomolina/protos/source/SourcePosition.java` | Added neutral UTF-16 source offset mapping shared with the LSP; no dependency from lint to lsp4j. |
| `src/main/java/com/guillermomolina/protos/cli/ProtosCli.java` | Added `lint` command/help and static calling-thread dispatch **before** guest carrier creation, with other command dispatch preserved. |
| `src/main/java/com/guillermomolina/protos/lsp/ProtosLspSourcePositions.java` | Delegates to protocol-neutral `SourcePosition`; shared position authority. |
| `src/test/java/com/guillermomolina/protos/cli/ProtosCliLintTest.java` | Added ~468-line focused contract and architecture tests (22 acceptance families described by implementing agent). |
| `pom.xml` | Implementation version updated to `0.3.305-SNAPSHOT`. |
| `CHANGELOG.md` | New `0.3.305-SNAPSHOT` section covering LM012-C1, diagnostic and exit contract, no guest execution and local PASS assertion. |

The implementation calls `ProtosStaticAnalysisCore.parse(snapshot)` once; on `Parsed` calls `ProtosStaticLint.check(parsed)`, and on `Failed` emits only a parser error. There is no call to `lint(snapshot)` that would parse twice, and the static command path does not open a Polyglot Context. Exit 0 means valid source and warnings permitted, exit 1 parser failure or gated warnings, exit 2 usage, exit 3 input/UTF-8, exit 70 unexpected internal error. JSON fields comprise `schemaVersion:1`, `source`, `status`, and the ordered diagnostics with origin/code/severity/message/range. No user-code execution, file rewriting, project discovery, additional lint rules, new TOOLxxx or normative spec edits were published.

Source-level and test-source proof is distinguished from runtime observation: code inspection establishes the implemented path, and focused tests exist. **The coordinating agent has not independently executed those tests.**

## Maintainer-supplied validation evidence and provenance

The project owner explicitly reported in the active conversation:

> `LM012-C1: static protos lint CLI/CI command (D195) pushed, el git diff check esta limpio. Todos los tests han pasado en local`

This is **human-reported validation**, not a newly executed command in this evidence-publication step. The prior handoff requested `git diff --check`, focal Maven tests, and one `make test`; the latest report affirms all local tests passed without supplying individual logs or a command-by-command transcript. The published changelog also states the focal and integrated `make test` suite passed. Do not invent run timestamps, durations, runner matrices, binary/native distribution acceptance, or remote GitHub Actions PASS from this report.

```text
LM012_C1_PRODUCT_SHA=a326eb537ce2dc4dc20287300f2b6fd677e53e75
PRODUCT_VERSION=0.3.305-SNAPSHOT
LM012_C1_PUBLISHED=YES
MAINTAINER_REPORT_LOCAL_ALL_TESTS=PASS
MAINTAINER_REPORT_GIT_DIFF_CHECK=CLEAN
NEW_TEST_SOURCE=PUBLISHED
COORDINATOR_EXECUTED_TESTS=NO
COORDINATOR_EXECUTED_BUILDS=NO
REMOTE_CI_RESULT=NOT_ASSERTED
NATIVE_DISTRIBUTION_RESULT=NOT_ASSERTED
NEW_LANGUAGE_SEMANTICS=NO
```

## Relationship to remaining LM012 slices

- **LM012-A0:** diagnostics/static-analysis authority audit completed.
- **LM012-B1:** two owner-approved D194 default Warning rules and LSP diagnostic publication already implemented at `a8027f6a`. This is not work to redo under LM012-D.
- **LM012-C0:** D195 public CLI/CI contract investigated and ratified.
- **LM012-C1:** now published and locally validated according to the maintainer.
- **LM012-D:** LSP push diagnostics and the generic VS Code LanguageClient transport exist. Real end-to-end extension/Problems-panel acceptance has not been evidenced by this publication; reserve it for the integrated editor acceptance/LM012-F rather than cloning the static analysis.
- **LM012-E:** no concrete CodeAction or quick-fix is currently approved by D194. The next bounded slice is **LM012-E0, TYPE=INVESTIGATION**, to determine whether any genuinely semantics-preserving per-rule edit is worth exposing, or whether E should be explicitly deferred/closed without implementation.
- **LM012-F:** final release/editor/CLI evidence and parent closure, after resolving E. Do not claim that a JVM unit test alone proves VS Code extension/live Problems-panel behavior.

The implementation commits the ratified D195 surface only; its public user-interface details are not re-opened here. Future fixes must satisfy D194's exact-snapshot, grammar/trivia and explicit-action proof requirements. No separate Dxxx/TOOLxxx/CLIxxx is warranted merely to audit candidate edits.
