# LM013 — Agent and Human Executor Handoff (D209-C)

## Current status — completed and real-editor accepted (2026-10-21)

**LM013/#861 is CLOSED / Completed.** The implemented document-local feature is published at [`protos@cb027faa4205ada04ffe9f875e5242f19067e5e8`](https://github.com/guillermomolina/protos/commit/cb027faa4205ada04ffe9f875e5242f19067e5e8), `0.3.389-SNAPSHOT`. The independent BUG023 Native ICU4J repair is published at [`protos@d7e48d86698c08b0f74e176080fd2b3b4526c0cd`](https://github.com/guillermomolina/protos/commit/d7e48d86698c08b0f74e176080fd2b3b4526c0cd), `0.3.390-SNAPSHOT`; BUG023/#937 is also CLOSED / Completed.

**Final acceptance:** the maintainer reports PASS for real VS Code loose-file Hover/Completion/literal Closure Signature Help, negative loose-file Definition, and post-BUG023 Native bound-workspace Symbols/Definition/References/Hover/Completion. On 2026-10-21 the maintainer again reported all local tests PASS and `git diff --check` clean. See the [final acceptance evidence](../../evidence/LM013/LM013_390_NATIVE_VSCODE_FINAL_ACCEPTANCE_AND_BUG023_RESOLUTION_2026_10_21.md) and the [original published-feature checkpoint](../../evidence/LM013/LM013_389_DOCUMENT_LOCAL_LSP_PRODUCT_PUBLICATION_2026_10_20.md). No further LM013 implementation or release is pending.

The original agent/human instructions below are retained as a historical handoff; their earlier `PRODUCT_EDIT=NOT_YET_APPLIED` and test-pending markers describe the **pre-publication** state, not the current state.

**Type:** IMPLEMENTATION, **not** investigation.
**Owning Issue:** [LM013/#861](https://github.com/guillermomolina/protos/issues/861).
**Selected decision:** [D209 Candidate C](../../decisions/tooling/D209_UNBOUND_DOCUMENT_LOCAL_LSP_INTELLIGENCE_AUTHORITY.md), owner-approved 2026-10-20 and recorded in [D209/#936](https://github.com/guillermomolina/protos/issues/936).
**Implementation repository:** `guillermomolina/protos` (exact spelling, two occurrences of "mo").
**Project evidence repository:** `guillermomolina/protos-project-docs`. This handoff is recorded here by task-specific owner authorization; it does **not** permit agent publication to the product repository.
**Baseline note:** Source audit initially observed `d32524bd1dec045966dc33f3a3f58abc4dcf0278`; latest observed product `main` was `0bceacdfc0127453729f346382152bd9908f3de1` after the separate BUG022 publication. **The implementation agent must assume the local checkout is already the correct HEAD and must use its actual files**, never reset/check out a historical SHA.
**Scope:** one cohesive LM013 product implementation, tests and maintained language-server guide; no independent `Ixxx` issue necessary because this is already formally owned by LM013, whose rules explicitly cover implementation. If a truly independent workstream appears, coordinate before allocating an identifier.

## 1. Autonomous agent prompt

```text
TIPO = IMPLEMENTACIÓN
ISSUE = LM013 / #861
DECISIÓN = D209-C (RATIFIED)
REPOSITORIO = guillermomolina/protos

Assume the local product repository is ALREADY at its appropriate HEAD.
Do not fetch, pull, checkout, reset, rebase, open new branches, force-push,
or rely on the historical research SHA. Everything needed to implement this
task is in the current local product tree and in this prompt. Do not browse
the web or inspect other repositories. Do not request the earlier conversation.

Read and obey AGENTS.md, AGENTS.work/IMPLEMENTATION.md, src/AGENTS.md,
AGENTS.work/REFERENCE.md (source license), and any additional path-scoped
instructions. Read the relevant sections of spec/PROTOS_GRAMMAR.md,
spec/semantics/CALLABLES.md and EXECUTION_AND_CONTROL.md.
Read the current source/tests identified below and perform the targeted
blocker/spec audit. Do not alter language or runtime semantics.

D209-C exact approved contract:
- An exact current open-buffer snapshot alone authorizes Hover, Completion,
  and Signature Help results consisting only of existing LM010 generation-one
  static document-local parser/proof facts, even when ProjectBinding absent.
- Hover: syntax for literals/Closure heads/parameter declarations and proven
  same-document Closure parameters with the existing accurate labels.
- Completion: parser-accepted reserved expressions plus D110-proven local
  Closure parameters. No guessed slot/member/import/stdlib candidates.
- Signature Help: direct literal Closure callees after grouping only;
  existing bounded parser repair may be used; named/dynamic calls get none.
- didOpen confers snapshot custody only, NOT canonical ProjectBinding identity.
- Definition, References and Workspace Symbols remain project-gated by exact
  ProtosWorkspaceService::ownsCanonicalSourceUri. Preserve canonical URI
  equality, multi-root/no-inference, source authority and D110/D124 proof.
- No path/URI normalization to admit aliases into a canonical project.
  A separately opened noncanonical file: URI has only local snapshot truth.
- Keep snapshot staleness checks, didClose/unsaved behavior, UTF-16/CRLF and
  all existing negative-proof behavior; no stale/disk/guest results.
- Keep the official VS Code extension thin. No TypeScript semantic engine,
  host Truffle execution, extra project provider/index/daemon, package scan,
  runtime/backend changes, debugger/BUG022 modifications, new operations,
  additional grammar surface, or new API.
- D085 G2-only historical loose feature *coverage* is prospectively superseded
  only for the three LM010 local methods, NOT the ProjectBinding identity rule.

Source map (verified before the ratification):
src/main/java/com/guillermomolina/protos/lsp/ProtosLanguageServer.java
  installs workspaceService::ownsCanonicalSourceUri. LEAVE this unchanged.
src/main/java/com/guillermomolina/protos/lsp/ProtosWorkspaceService.java
  ownsCanonicalSourceUri exact binding identity. LEAVE unchanged.
src/main/java/com/guillermomolina/protos/lsp/ProtosTextDocumentService.java
  definition and references: retain authority gate and final authority recheck.
  hover, completion, signatureHelp: remove ONLY the ProjectBinding predicate
  rejection and authority recheck, keep current OPEN_DOCUMENTS_DOMAIN snapshot,
  position offset, core result, isCurrent, exact URI and LSP serialization.
src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisCore.java
  already project-neutral and parser/proof-only: ordinarily no code changes.
src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisSession.java
  already stores immutable exact snapshots; numeric document versions are NOT
  enforced monotonically. Do not create an invented version-order policy.

Update existing tests without just deleting them:
- ProtosLanguageServerHoverTest: loose/alias positive syntax + proven parameter;
  negative unknown name, stale/closed, invalid UTF-16/out-of-range; bound stays.
- ProtosLanguageServerCompletionTest: loose/alias proven local parameter and
  reserved-expression completion; negative member/slot guessing and closed;
  list remains complete and replacement spans exact.
- ProtosLanguageServerSignatureHelpTest: loose/alias direct literal Closure
  positive; named-call/dynamic/invalid negative; unsaved/close handling.
- ProtosLanguageServerIntegratedEditorTest: real LSP4J/JSON-RPC wire fixture
  compares bound source and truly loose open source, alias, unsaved edits,
  closure labels and conservative negatives.
- ProtosLanguageServerDefinitionTest and ReferencesTest: keep project-only
  loose/alias negative contracts, and ensure existing tests still pass.
- Current documentation: docs/guide/tools/language-server.md must distinguish
  document-local Hover/Completion/Signature Help from project-only workspace
  symbols/definition/references. Do not touch historical LM010 evidence.

Implementation approach:
  Change only the three handlers' authority branch first, then update the
  corresponding regression tests and the existing integrated protocol test,
  then current guide. A single product publication is preferable: one small
  production file, approximately four LSP tests and one maintained guide.
  Preserve all unrelated staged/unstaged/untracked changes.
  Python patch scripts with count==1 assertions; grep rather than rg.
  No new abstraction or helper unless exact HEAD proves it is necessary.

External system facts, provided here to avoid research:
  Standard LSP document methods textDocument/hover, /completion and
  /signatureHelp take a URI and position; the protocol does not itself
  require canonical project membership, and UTF-16 positions are a client/
  server protocol concern. TypeScript tsserver has an InferredProject for
  loose files, Jedi can analyze a source buffer, clangd uses fallback commands,
  and rust-analyzer can consume externally supplied rust-project.json;
  NONE of their upward search/project inference is permitted in Protos.
  Truffle-language implementation references are NOT applicable to this
  change: it is entirely editor-neutral parser + LSP edge, not a Truffle
  execution or partial-evaluation change. Do not browse GraalPy/JS/Ruby.

Validation is HUMAN-EXECUTED ONLY. Agent does not run compilations, tests,
Make/Maven, formatting checks, Protos programs, publication scripts, or
Git operations that modify state/communicate remotes.
At first, hand one self-contained code-edit batch to the human; then continue
tests/guide edits until a coherent candidate exists. Do not ask for Maven
between intentionally partial batches. Then hand git diff --check, precise
changed-path diff, focused static compilation/checks, complete affected
LSP/analysis tests, and impact-aware make test as needed. No test pass may be
invented. When human reports green, human updates pom.xml + CHANGELOG.md as
late metadata, performs explicit-path git add, single-line English
"LM013: enable proven document-local LSP intelligence" commit and normal push.
No tests after version/changelog-only finalization unless candidate executable
bytes/dependencies change. No force push. Docs evidence after the product SHA
is known remains a separate project-docs transaction.
```

## 2. Agent vs human-executor responsibilities

**Agent:** inspect actual current HEAD and changed-path set, read governing
spec/decisions/tests, edit source/tests/current guide preserving unrelated work,
explain exact patch, request and interpret only the necessary human-executed
checks. May coordinate Issues. Do not perform product Git publication.

**Human executor:** run the supplied Python source edit batch from the already
prepared local `guillermomolina/protos` checkout, then grant test/build
execution only after a coherent source+tests patch exists. Execute `git diff
--check`, inspect exact `git diff -- <paths>`, run focused source/analysis LSP
tests and the required integrated gate `make test` if the changed-file impact
policy selects FULL. Immediately before publication, update `pom.xml` and
`CHANGELOG.md` under project rules; stage ONLY patch-owned explicit paths,
commit and push normally; preserve all concurrent edits. No tests following
late metadata-only edits. Report exact commit SHA, test evidence, and Native/
VS Code UI observations separately; never substitute previous LM010-D results.

## 3. First editing batch — production LSP edge (partial)

This first batch edits only `ProtosTextDocumentService.java`. It has a
transactional in-memory check for all three exact method-local matches before
writing once. If the actual local HEAD differs, **STOP**; the agent must inspect
the current source and adjust the patch, not force a guessed replacement.

```bash
git status --short
git diff -- src/main/java/com/guillermomolina/protos/lsp/ProtosTextDocumentService.java

python3 - <<'PY'
from pathlib import Path

path = Path("src/main/java/com/guillermomolina/protos/lsp/ProtosTextDocumentService.java")
source = path.read_text(encoding="utf-8")
updated = source

methods = {
    "Hover": "    public CompletableFuture<Hover> hover(HoverParams params) {",
    "Completion": "    public CompletableFuture<Either<List<CompletionItem>, CompletionList>> completion(",
    "SignatureHelp": "    public CompletableFuture<SignatureHelp> signatureHelp(SignatureHelpParams params) {",
}
for name, start_marker in methods.items():
    if updated.count(start_marker) != 1:
        raise SystemExit(f"STOP: missing or duplicate method marker: {name}")
    start = updated.index(start_marker)
    end = updated.find("\n    @Override", start + len(start_marker))
    if end < 0:
        raise SystemExit(f"STOP: no next handler boundary: {name}")
    block = updated[start:end]
    initial = (
        "        Predicate<String> authority = navigationSourceAuthority;\n"
        "        if (!authority.test(uri)) {\n"
        f"            return no{name}();\n"
        "        }\n\n"
    )
    recheck = "                || !authority.test(uri)\n"
    if block.count(initial) != 1 or block.count(recheck) != 1:
        raise SystemExit(f"STOP: handler changed or ambiguous: {name}")
    block = block.replace(initial, "", 1).replace(recheck, "", 1)
    updated = updated[:start] + block + updated[end:]

if updated == source:
    raise SystemExit("STOP: patch made no change")
path.write_text(updated, encoding="utf-8")
print("LM013_D209_C_LSP_EDGE_EDIT=APPLIED")
PY

git diff --check -- src/main/java/com/guillermomolina/protos/lsp/ProtosTextDocumentService.java
git diff -- src/main/java/com/guillermomolina/protos/lsp/ProtosTextDocumentService.java
```

This is **not a complete passing patch**: existing LSP negative tests still
encode the former project gate and must be edited next. Do not run Maven,
`make test`, versioning or product Git publication after this partial batch.
The human-executor must return the edit/diff-check outcome; the agent then
supplies the first test-edit batch for the same cohesive LM013 slice.

## 4. Acceptance and closure

Repository Java tests must exercise positive/negative/bound/loose/alias,
snapshot replacement, close, invalid positions, parser-proven facts and
unchanged project-only definition/references, including the integrated
JSON-RPC client. The human must then validate actual Native runtime and
extension identity in a **new** real VS Code editor session, with an existing
bound fixture and truly loose file including unsaved-change and conservative
named-call negatives. The old 0.3.312/0.2.2 result is historical only.

LM013 closes **only after** verified product publication, appropriate complete
affected/integrated test evidence, real-editor acceptance, and a new exact
product revision evidence record. D209's ratification does not close LM013.

```text
SLICE=LM013_SINGLE_COHERENT_PRODUCT_IMPLEMENTATION
DECISION=D209_C_RATIFIED
AGENT_PRODUCT_PUBLICATION=FORBIDDEN
HUMAN_EXECUTOR=REQUIRED
NEW_I_ISSUE=NOT_REQUIRED
PRODUCT_TESTS=NOT_RUN
PRODUCT_EDIT=NOT_YET_APPLIED
```
