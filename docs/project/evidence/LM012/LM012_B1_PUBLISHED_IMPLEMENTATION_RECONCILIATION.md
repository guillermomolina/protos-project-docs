# LM012-B1 — Published implementation and evidence reconciliation

**Status:** PRODUCT IMPLEMENTATION PUBLISHED; HUMAN TEST RESULTS NOT SUPPLIED IN THIS HANDOFF.

**Evidence date:** 2026-10-08. **Owner:** [LM012/#671](https://github.com/guillermomolina/protos/issues/671). **Decision:** [D194/#842](https://github.com/guillermomolina/protos/issues/842), modified B ratified; [LM012-B1 owner rule selection](../../work/LM012/LM012_B1_INITIAL_LINT_RULES_OWNER_APPROVAL.md).

```text
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=a8027f6a78fac057f409103d8a79500faead72e2
PRODUCT_COMMIT_TITLE=LM012-B1: initial canonical correctness lint diagnostics
IMPLEMENTATION_VERSION=0.3.295-SNAPSHOT
USER_PUBLICATION_REPORT=LM012-B1: initial canonical correctness lint diagnostics pushed.
PRODUCT_DIFF_READ=YES
HEAD_FILES_INSPECTED=YES
HUMAN_TEST_REPORT=NOT_PROVIDED
TESTS_EXECUTED_BY_THIS_INVESTIGATION=NO
BUILDS_EXECUTED_BY_THIS_INVESTIGATION=NO
REMOTE_CI_STATUS=NOT_VERIFIED
SPECIFICATION_MODIFIED=NO
```

The product revision was retrieved from the live GitHub commit record and re-read as concrete file content. **The human reported publication, not test output or a validation summary.** The diff proves tests were added but not that they were run or passed. The source and changelog indicate no normative specification change. This is a read-only reconciliation; no product or test commands were executed.

## Exact published product delta

[Commit `a8027f6a78fac057f409103d8a79500faead72e2`](https://github.com/guillermomolina/protos/commit/a8027f6a78fac057f409103d8a79500faead72e2) owns nine paths:

| Path | Observed change |
| --- | --- |
| `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticLint.java` | New source-local AST visitor, deterministic output order, exact direct-Closure `^` suffix check and right-hand fresh-object identity check |
| `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticLintDiagnostic.java` | Immutable snapshot-bound record with stable approved rule IDs, Warning level, `SourceSpan` and message |
| `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisCore.java` | Explicit `lint(snapshot)` entry point; parse success required, parse failure returns empty findings |
| `src/main/java/com/guillermomolina/protos/lsp/ProtosTextDocumentService.java` | Existing single per-version `publishDiagnostics` path now projects lint results; parser errors remain parser-owned; freshness checked |
| `src/test/java/com/guillermomolina/protos/analysis/ProtosStaticLintTest.java` | Focused soundness, exclusions, deterministic ordering, exact spans, parse-failure and snapshot-binding test source |
| `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerDiagnosticsTest.java` | LSP rule-code/Warning/source/range/version, parse-error-to-lint change/recovery/close, UTF-16 and CRLF test source |
| `pom.xml` | Published version `0.3.295-SNAPSHOT` |
| `CHANGELOG.md` | `0.3.295-SNAPSHOT` LM012-B1 description, with no specification bump |

**Corrected accounting:** there are eight paths in the observed diff, not nine: two newly introduced analysis files, one analysis core, one LSP service, two test files, and two version/changelog metadata files.

## Verified implementation boundaries

1. `protos/unreachable-after-nonlocal-return`: `ProtosStaticLint.checkUnreachableAfterNonLocalReturn` visits the real `SurfaceClosure` and calls its direct-Sequence check only when `expressionBody()==false`. It reports one source-span suffix after the earliest direct `SurfaceNonLocalReturn`. Traversal of nested Closure bodies is separate and does not infer invocation from their presence.
2. `protos/always-different-fresh-object`: only `SurfaceBinary` with `===` or `!==`; unwraps only `SurfaceGroup` on the **right** and admits a `SurfaceObject` source value. It does not convert ordinary `==` or dispatch a user-defined equality method; constant-result wording is explicitly qualified by normal completion.
3. Both findings use the stable owner-approved IDs and `WARNING`; a parser-failed snapshot returns no lint warnings. The AST visitor is instantiated **on explicit lint evaluation** and no new runtime bootstrap/Context/Actor/Task/Process path is introduced by the inspected diff.
4. The existing LSP server's `ProtosTextDocumentService` handles the current document snapshot and publishes rule codes via protocol `Diagnostic` on the same `publishDiagnostics` channel already used for parse errors. The standalone VS Code extension had **no source changes** in this implementation and continues to consume the standard server.
5. No `protos lint`/`protos check` CLI command, exit policy, quickfix or LSP CodeAction is added by this commit. `ProtosCli.SUBCOMMANDS` remains `language-server`, `debug`, `run`, `package`, `test`, `format` at this product revision.
6. Two focal test files now contain examples for return reachability versus conditional/nested execution, strict identity versus ordinary equality, parenthesized/parented fresh objects, deterministic order, UTF-16/CRLF, parse errors, updates and close. This is **observed test coverage in source**, not a PASS certification.

## Closure boundary and residual questions

- **LM012-B1 publication:** observed and committed; do **not** prescribe a repeat implementation or additional LSP diagnostic layer.
- **Validation:** maintainer test/build outcomes were **not stated** in the current handoff. Local suite `PASS`, `git diff --check` `PASS`, CI `SUCCESS` and strict end-to-end VS Code visual acceptance are **not evidenced here**; do not fabricate them. This does not invalidate the existence of the published commit.
- **Next functional decision:** whether and how an explicit CLI/CI lint surface should exist at all, and what exact public command, inputs, diagnostics, exit policy and scope it would own. D194 explicitly leaves that public tool contract unsettled. The correct next slice is **LM012-C0 INVESTIGATION** rather than inventing `protos lint` or rewriting the existing Test/Package Tool authority.
- **Future safe edits:** none of the two selected rules received a proved semantics-preserving quickfix; LM012-E remains deferred by design.
- **D194 formal Issue closure:** ratification was published, but native GitHub D194 child-of LM012/#671 graph reconciliation remains outstanding. The available connector does not expose native parent/sub-issue mutation. Do not claim full GITHUB015 closure until verified.

## Materially reviewed project/source references

- `guillermomolina/protos@a8027f6a78fac057f409103d8a79500faead72e2`: `pom.xml`, `CHANGELOG.md`, `ProtosStaticLint.java`, `ProtosStaticLintDiagnostic.java`, `ProtosStaticAnalysisCore.java`, `ProtosTextDocumentService.java`, `ProtosStaticLintTest.java`, `ProtosLanguageServerDiagnosticsTest.java`, and `ProtosCli.java`.
- Product `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/COORDINATION.md`, `AGENTS.work/DESIGN.md`, and selected `docs/design/TOOLCHAIN_TOOL_ARCHITECTURE.md`.
- Documentation `AGENTS.md`, D194 ratification, LM012-B1 owner selection, and LM012-B0 comparative research.

```text
LM012_B1_PRODUCT_PUSH=CONFIRMED
LM012_B1_PRODUCT_SHA=a8027f6a78fac057f409103d8a79500faead72e2
LM012_B1_IMPLEMENTATION=PUBLICATION_COMPLETE
LM012_B1_TEST_SOURCE=ADDED
LM012_B1_HUMAN_TEST_RESULTS=UNREPORTED
LM012_CLI_CI_CONTRACT=UNSELECTED
NEXT=LM012-C0_INVESTIGATION
```
