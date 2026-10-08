# LM012-E0 — Quick-fix safety investigation and owner-selected no-action outcome

**Outcome: CLOSED BY EXPLICIT OWNER SELECTION OF CANDIDATE A — no CodeActions for the two D194 lint diagnostics.**

- Parent: [LM012 / guillermomolina/protos#671](https://github.com/guillermomolina/protos/issues/671).
- Governing policy: [D194 / #842](https://github.com/guillermomolina/protos/issues/842), [canonical ratification](../../decisions/tooling/D194_CANONICAL_LINT_DIAGNOSTICS_RULE_AND_FIX_POLICY.md).
- Separate unchanged CLI/CI authority: [D195 / #843](https://github.com/guillermomolina/protos/issues/843), [canonical contract](../../decisions/tooling/D195_SINGLE_SOURCE_LINT_PUBLIC_CLI_CI_CONTRACT.md).
- Investigation date and owner selection: **2026-10-08**, active conversation. The owner explicitly replied **"aprobada A"** to the presented LM012-E0 recommendation.
- Product revision inspected in read-only GitHub: [`5b09b5d9bfee66d0df760288e10cc26cb1bd2342`](https://github.com/guillermomolina/protos/commit/5b09b5d9bfee66d0df760288e10cc26cb1bd2342) (`PERF034-D`); its direct parent is [`a326eb537ce2dc4dc20287300f2b6fd677e53e75`](https://github.com/guillermomolina/protos/commit/a326eb537ce2dc4dc20287300f2b6fd677e53e75), the published LM012-C1 CLI revision (`0.3.305-SNAPSHOT`). The moving HEAD is not a proxy for either immutable reference.
- Source of validation assertions: owner's subsequent active-conversation statement **"el git diff check esta limpio. Todos los tests han pasado en local"**. This is a maintainer-reported local-clean/full-tests-PASS statement, **not** an independently executed verification, not proof of editor/VS Code integration, and not tied to an asserted test-run commit SHA.
- Changes from this selection: **no product source, CLI, specification, LSP implementation, or tests**.

## Decision scope and existing state

D194 conditionally permits an explicitly selected quick fix only when independently proven source- and semantics-preserving, snapshot-exact, and applied solely after a user's affirmative action. The separately approved LM012-B1 implementation supplies precisely two default-on `Warning` diagnostics, **without** a CodeAction: `protos/unreachable-after-nonlocal-return` and `protos/always-different-fresh-object`. LM012-C1 already implements D195's `protos lint [--output text|json] [--fail-on-warning] [<file>]`; E0 does not change its command, output, or exit contract.

The editor transport is not missing: `ProtosTextDocumentService.publishCurrentDiagnostics` reuses the canonical parser and `ProtosStaticLint.check`, projects rule IDs, severities and ranges, and uses versioned diagnostic publication. `ProtosLanguageServer.initialize` intentionally does **not** advertise `codeActionProvider`. Neither `textDocument/codeAction` nor any edit-generation API exists in the inspected path. Preserve this state under candidate A.

## Static audit and rule-specific proof boundary

Source files inspected at the exact Protos revision:

- `src/main/java/com/guillermomolina/protos/analysis/{ProtosStaticLint,ProtosStaticLintDiagnostic,ProtosStaticAnalysisCore,ProtosStaticParseResult,ProtosDocumentSnapshot,ProtosStaticAnalysisSession,ProtosSourceLayoutView}.java`
- `src/main/java/com/guillermomolina/protos/parser/ProtosParser.java`, relevant `parser/ast/Surface{Sequence,Closure,NonLocalReturn,Binary,Object}.java`, `lexer/ProtosLexer.java`, `source/{SourceSpan,SourcePosition}.java`
- `src/main/java/com/guillermomolina/protos/lsp/{ProtosTextDocumentService,ProtosLanguageServer,ProtosLspSourcePositions}.java`
- `src/test/java/com/guillermomolina/protos/analysis/ProtosStaticLintTest.java`, `src/test/java/com/guillermomolina/protos/parser/ProtosParserExpressionSeparatorTest.java`, `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerDiagnosticsTest.java`
- `spec/PROTOS_GRAMMAR.md` (expression separators, newlines, comments, nonlocal return, Closures), `spec/semantics/CALLABLES.md` (nonlocal return and `InvalidReturn`), `spec/semantics/OBJECT_MODEL.md` (fresh ordinary-object identity).

Read-only protocol reference: [LSP 3.17 Code Action](https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/#textDocument_codeAction) and [TextDocumentEdit](https://github.com/microsoft/language-server-protocol/blob/gh-pages/_specifications/lsp/3.17/types/textDocumentEdit.md).

### Candidate B — delete the unreachable suffix

**The diagnostic span is not an edit span.** In `f: () => { ^1; a }`, the lint diagnostic covers only `a`. Deleting that span leaves `f: () => { ^1; }`, a grammar-invalid trailing semicolon. The canonical parser requires an expression on each side of `;`; a separator across lines is a logical `NEWLINE` and is not interchangeable. The lexer treats `//` comments as ending before the newline, whereas `/* ... */` comments consume their internal newlines without producing `NEWLINE` tokens. The `SurfaceSequence` spans do not themselves identify the separator/trivia edit boundary.

A future bounded subset may be feasible, using opt-in `ProtosSourceLayoutView` token/trivia/comment attachments, the exact braced Closure sequence, a candidate source rewrite, reparsing and an independently demonstrated prefix/semantic equivalence proof. Positive reparse alone does **not** establish equivalence; comment association, scope, grammar-significant separators, literal payloads, nested syntax, CRLF, supplementary Unicode, and snapshots also matter. This proof and its practical utility have **not** been established for an approved edit domain. **No implementation authorized.**

### Candidate C — replace fresh-object identity with a constant

The rule proves only conditional normal-result value: `left === { ... }` yields `false`, and `left !== { ... }` yields `true` **if evaluation completes normally**. Replacing the binary expression with a boolean drops possibly effectful left evaluation, object parent/member initialization, errors, suspension and control transfers; replacing `===` with `==` changes primitive identity to a different operator. Wrapping evaluation to preserve order, single execution and return value introduces additional Closure/scope/control-flow semantics not proven equivalent. **Reject the constant-replacement quick fix.**

### Candidate A — diagnostics, no edits

No new source transformation exists, no runtime or LSP surface is added, and the two approved correctness warnings continue to provide information without unsound automated modification. No `fixAll`, fix-on-save, generic code action provider, source edit, formatter change, or extra lint rule is introduced. Future independently proven fixes remain possible under the existing D194 policy without requiring a placeholder implementation now.

## GITHUB010 twelve-dimensional comparison

The comparison below is qualitative evidence, not proof by scoring. B refers to a hypothetical narrowly bounded safe-delete action, not a demonstrated implementation. C refers to a proposed constant or equivalent expression rewrite.

| Dimension | A: no quick fixes | B: delete suffix | C: identity rewrite |
| --- | --- | --- | --- |
| 1. Correctness | Preserves established semantics | Not yet proven across edit boundaries | Direct constant replacement unsound |
| 2. Protos philosophy | Small and proof-first | Potentially suitable if proven | Elaborate rewrites disproportionate |
| 3. Current proportionality | No new machinery | LSP edits/trivia cost for narrow benefit | High complexity versus a warning |
| 4. Incremental growth | Optional actions remain possible | May admit a proven subset later | Poor initial slice |
| 5. Future options | No lock-in | Requires explicit proof boundary | Avoids premature transform contract |
| 6. Scalability | No new work | On-demand analysis feasible | Broad contextual rewrites difficult |
| 7. Conceptual simplicity | Highest | Separator/trivia/AST equivalence | Scope/control-flow complexity |
| 8. Portability | Editor-neutral unchanged | LSP capabilities/version gating | More semantic dependencies |
| 9. Runtime/resource cost | Zero added ordinary-runtime cost | On-demand only if implemented properly | Transformation complexity, no benefit today |
| 10. Operability/failure | Existing diagnostics only | Stale or malformed edit risks | Effects/errors can disappear |
| 11. Reversibility/defer cost | Cheap to revisit | Deferred without irreversible choice | Deferral avoids unsafe contract |
| 12. Evidence/risk | Existing implementation demonstrated | Adversarial cases; equivalence unproven | Direct-rewrite unsoundness demonstrated |

**Strongest argument against A:** dead-code removal could be useful and the on-demand source-layout infrastructure already supplies ingredients for a future narrow fix. **Response:** current evidence does not demonstrate both semantic edit safety and enough value to justify the added protocol and testing surface; deferral preserves this future option.

## LSP contract required only if a future fix is independently approved

`CodeActionContext.diagnostics` is not authoritative current-source evidence. A future provider would have to negotiate client `codeActionLiteralSupport` and versioned `workspace.workspaceEdit.documentChanges`, recalculate a matching rule on the currently open snapshot, validate URI/range/structure and trivia, and fail closed on stale or unsupported clients. A proposed `WorkspaceEdit.documentChanges` must use one non-overlapping `TextDocumentEdit` with a **concrete current document version**, not a null version or unversioned `changes` fallback. The client must reject mismatched versions between proposal and application. UTF-16 offsets, CRLF, closed documents and concurrent updates require test evidence. No `codeActionProvider` should be advertised until an independently approved implementation actually exists.

## Explicit owner selection and invariant/delta reconciliation

**Owner selection:** **A** — leave both D194 warnings intact and **do not implement Quick Fix / CodeAction** for LM012-E. Defer B without opening a speculative issue. Reject C and any D combination without new proof. Move to LM012-F final editor/CLI acceptance; do not infer acceptance from unit tests alone.

D194's requirement to withhold unproven fixes is **preserved**; its conditional future possibility is **not revoked**. D195 CLI/CI, LM012-B1 rule severity/identity/default behavior, LM009 diagnostic ownership, PLAT024 on-demand/pay-as-you-grow and LM011 formatter semantics are unchanged. **No Dxxx/PLATxxx allocation is necessary.** Existing closed D194/#842 and D195/#843 issues remain closed. LM012/#671 stays open until independent final acceptance evidence and closure gates are met.

The user-provided clean diff and local PASS are an explicitly attributed report; E0 itself authored **no executable artifact**, so no new program tests or builds are required for the owner selection. A coordinating assistant did not run commands, tests, builds, or state-changing product Git operations during the safety investigation. Publication of this non-normative evidence in the *explicitly authorized* `guillermomolina/protos-project-docs` repository is a separate post-approval coordination action.

```text
LM012_E0=COMPLETE
OWNER_SELECTION=A_EXPLICIT_2026-10-08
CODE_ACTIONS=NO
SOURCE_EDITS=NO
NEW_LINT_RULES=NO
PRODUCT_SHA_INSPECTED=5b09b5d9bfee66d0df760288e10cc26cb1bd2342
LM012_C1_PUBLISHED_SHA=a326eb537ce2dc4dc20287300f2b6fd677e53e75
HUMAN_REPORTED_LOCAL_TESTS=PASS
HUMAN_REPORTED_GIT_DIFF_CHECK=CLEAN
COORDINATOR_RAN_TESTS=NO
COORDINATOR_RAN_BUILDS=NO
REMOTE_CI_PASS=NOT_ASSERTED
VS_CODE_PROBLEMS_PANEL_PASS=NOT_ASSERTED
PORTABLE_DISTRIBUTION_PASS=NOT_ASSERTED
NATIVE_DISTRIBUTION_PASS=NOT_ASSERTED
NEXT=LM012-F0_INVESTIGATION_OF_FINAL_ACCEPTANCE
```

## LM012-F0 handoff (research only)

Audit actual product and thin VS Code extension HEAD and published acceptance procedures, enumerate missing **real editor** and **packaged portable/Native CLI** gates, define the smallest human-executable exact acceptance matrix, and distinguish tests that already have evidence from tests still pending. Do not reimplement existing diagnostic transport, run commands, mutate repos/issues, or close the parent during investigation. After human validation, reconcile real results; only propose a small implementation slice if a concrete defect is evidenced. Do not claim an editor-acceptance PASS merely from published test-source coverage.
