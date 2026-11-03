# BUG034 — D237 MIX-C Signature Help association correction (0.3.442)

**Date:** 2026-11-03. **Formal owner:** [BUG034 #1042](https://github.com/guillermomolina/protos/issues/1042). **Normative authority:** the published Protos `spec/PROTOS_GRAMMAR.md` §17 and `spec/semantics/CALLABLES.md` D237-MIX-C/R3, following [D237 #1038](https://github.com/guillermomolina/protos/issues/1038) and [I143 #1039](https://github.com/guillermomolina/protos/issues/1039). This is a non-normative, revision-bound evidence record, not a new design decision.

## Verified product publication

- **PROTOS_REVISION:** [`b708e1ed981b2ba151a00210622b486a22a7a908`](https://github.com/guillermomolina/protos/commit/b708e1ed981b2ba151a00210622b486a22a7a908).
- **PARENT_REVISION:** `489db6e1dc18554d7b2e5c9a86eca5f8d157f660`.
- **COMMIT_TITLE:** `BUG034: associate D237 named arguments in Signature Help (0.3.442)`.
- **IMPLEMENTATION_VERSION:** `0.3.442-SNAPSHOT`, independently verified in the published `pom.xml` and `CHANGELOG.md`.
- **PUBLISHED_DELTA:** 10 paths, 518 insertions and 36 deletions, independently verified through GitHub: two production Java files, six Java test files and two release-metadata files.
- **SCOPE:** LM010-C document-local literal-Closure Signature Help. No new normative specification, parser, runtime, formatter, or VS Code extension files changed in this publication.

## Published correction and proof boundary

The pre-fix analyzer used the parenthesized source argument ordinal as the formal index, which was incompatible with D237's freely interleaved named and positional call items. The published `ProtosStaticSignatureHelp.activeParameter` now distinguishes `activeArgument` (source item index) from `activeParameter` (the formal index proved by source structure):

- A named argument selects its same-named declared non-rest formal, unless the label is unknown, denotes `rest`, or its association is invalid or uncertain because of positional collision or an unknown spread length.
- Ordinary positional arguments count only preceding positional values, not named items. The complete parsed argument list includes an attached trailing Closure as a final positional item for collision analysis, without shifting parenthesized source indices.
- Spread operands remain of unknown positional length even when written as array literals; no length is guessed. A later ordinary positional argument may still provably select the trailing rest formal when its minimum ordinal already reaches rest. A spread item itself has no uniquely proved formal.
- No misleading formal is published on proven or possible named/positional collisions, unsupported labels, undecidable spread associations, or excess positional arguments without rest.
- A narrow, transient `label:` missing-value repair inserts one synthetic identifier only into the editor analysis, then requires the normal lexer, real parser, call shape, token preservation and exact synthetic argument/name to confirm the projection.

The existing LM010-C boundary is unchanged: only provable direct invocation of a literal Closure qualifies. Name-based callees, dynamic sends, constructors, arbitrary custom `call`, native and foreign signatures are not inferred; no guest code is executed by the analyzer. The LSP adapter's fail-closed no-result behavior on an unproved nonempty signature remains intact.

## Regression evidence

The published commit extends six existing Java test classes:

1. `ProtosStaticSignatureHelpTest`: interleaved/reordered labels, defaults, explicit `null`, positional and named collisions (including trailing Closure), spread uncertainty and provable rest, grouped slot creation, nested calls, unsupported callees, incomplete named values, CRLF, UTF-16/non-BMP and snapshot invalidation.
2. `ProtosLanguageServerSignatureHelpTest`: formal indices `2, 0, 1` for `(third: 3, 1, 2)`, declaration-order parameter labels, no-result on unprovable association, unsaved changes and UTF-16/CRLF positions.
3. `ProtosLanguageServerIntegratedEditorTest`: the same D237 association and fail-closed boundaries across the real JSON-RPC connection.
4. `ProtosLanguageServerDefinitionTest`: a named-argument label is not a lexical reference, while its value expression can resolve normally.
5. `ProtosLanguageServerReferencesTest`: label exclusion, value reads and grouped positional slot-creation reads.
6. `ProtosLanguageServerDiagnosticsTest`: parser-owned duplicate-label diagnostics, no speculative static diagnostics for runtime-only collisions or unknown labels, and document-version replacement.

An additional read-only review caught two incorrect test expectations regarding positional/name collisions; both expectations were corrected before validation. The final published commit contains the corrected tests.

## Maintainer-reported validation and limits

| Gate | Outcome | Provenance |
| --- | --- | --- |
| Focused six-class Maven regression selection | **PASS** | Maintainer report after the final test corrections; no individual test count supplied |
| `make test` | **PASS** | Maintainer-reported local integrated result |
| `make check` | **PASS** | Maintainer-reported local integrated result |
| `git diff --check` | **CLEAN** | Maintainer terminal output |
| Git commit and push to `guillermomolina/protos` | **VERIFIED** | GitHub independently returns the exact published revision, parent, changed paths and diff statistics |
| Product version/changelog | **VERIFIED** | Published `pom.xml` and `CHANGELOG.md` contents |

The evidence recorder did **not** independently run local Maven tests, `make test`, `make check`, a pre-fix LSP reproduction, Native Image, installed-VSIX acceptance or performance benchmarks. The successful focal tests include a JSON-RPC integration regression as reported by the maintainer; they are not a separate independently executed editor-session reproduction. No additional runtime or language semantic contract is claimed.

## Closure and durable ownership

BUG034 is a standalone editor correctness repair to the completed I143/D237 implementation. It does not reopen [LM010 #493](https://github.com/guillermomolina/protos/issues/493), [I143 #1039](https://github.com/guillermomolina/protos/issues/1039) or the independent [BUG033 formatter repair #1041](https://github.com/guillermomolina/protos/issues/1041). Installed-extension acceptance remains separately owned by the extension's CI002 work. This record belongs to BUG034's durable publication evidence; issue state and Project synchronization remain owned by GitHub coordination.

```text
WORK_ITEM=BUG034
ISSUE=guillermomolina/protos#1042
PROTOS_REVISION=b708e1ed981b2ba151a00210622b486a22a7a908
PARENT_REVISION=489db6e1dc18554d7b2e5c9a86eca5f8d157f660
IMPLEMENTATION_VERSION=0.3.442-SNAPSHOT
PRODUCT_CHANGED_PATH_COUNT=10
PRODUCT_ADDITIONS=518
PRODUCT_DELETIONS=36
HUMAN_FOCAL_SIX_CLASS_TESTS=PASS_REPORTED
HUMAN_MAKE_TEST=PASS_REPORTED
HUMAN_MAKE_CHECK=PASS_REPORTED
HUMAN_GIT_DIFF_CHECK=CLEAN_REPORTED
PRODUCT_PUBLICATION=GITHUB_VERIFIED
SPEC_CHANGE=NONE
DURABLE_RECORD_DECISION=REQUIRED_BY_BUG034_CONTRACT_AND_OWNER
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
PROJECT_ROUTING=NOT_INDEPENDENTLY_VERIFIED
```
