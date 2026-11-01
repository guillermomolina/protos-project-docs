# I138 — Human executor: one atomic Protos product implementation

**Type:** IMPLEMENTATION, not investigation. **Repository:** existing local HEAD `guillermomolina/protos` only. **Issue:** I138 (created under GitHub `guillermomolina/protos`). **Ratification:** D234-B2-S1, owner explicitly said “apruebo D234-B2-S1” on 2026-11-01; normative implementation is not yet published. Paired agent document: [I138_AGENT_ONE_SLICE_PROMPT.md](I138_AGENT_ONE_SLICE_PROMPT.md). All external information is in the paired prompt and current local Protos source; the editor must not inspect docs repo or web.

## Role separation and one-slice atomicity

Editing agent reads local instructions/spec/code and edits complete approved syntax, parser, canonicalization, source preservation/formatter, diagnostics, tests, guides, spec changelog as **one** cohesive slice; it does **not** execute commands/tests, mutate Git, browse web or read any other repository. The human executes checks, validation, packaging/version step and **one** final product commit/push. Do not split into I138-A/B/C or create a second release/temporary slice. If a real unapproved semantic conflict is discovered, stop for owner decision.

## Human preflight (before sending prompt to editor)

~~~bash
pwd
git status --short
git log -1 --format='%H %s'
git diff --name-only
git diff --cached --name-only
~~~

Ensure intended checkout already at HEAD; never wipe concurrent/dirty changes. Report patch-owned overlaps to the editor before it writes. Human owns any remote synchronization later, not the editor; do not checkout/pull inside editor role.

## Human validation after edits

Read changed paths including untracked conformance tests and manifests, inspect exact diffs, run cheapest guards and focused tests first; do not claim green based only on code reading:

~~~bash
git status --short
git diff --check
git diff --stat
git diff -- spec/PROTOS_GRAMMAR.md spec/semantics/CALLABLES.md src/main/java/com/guillermomolina/protos/parser/ProtosParser.java protos/tools/formatter/Structural.protos
~~~

Then inspect all remaining patch-owned diffs and untracked files. Use commands appropriate to real current `pom.xml`, `Makefile` and local `AGENTS.work/IMPLEMENTATION.md` / `scripts/validation_impact.py` to run focal parser, lexer, canonicalizer, control-transfer/return-home, formatter/source-layout, lint, LSP/REPL and registered Protos conformance. Test on JVM first; native smoke/tests only when applicable under existing impact policy. Do not invent Maven selectors.

**Required semantic assertions:** `return x` == `^x`; nested callback unwinds method home; escaped/foreign-Task `InvalidReturn`; `ensure` cleanup; no new local return; unchanged `^`. Lexer `return` is IDENTIFIER. Ordinary `return`/`return: x`/`return = x`/`return => x`/`return()`/`return(x,y)`/`object.return(x)`/`(return)(x)` remain normal; `return(x)`/`return -x`/`return ~x`/`return[0]`/`return {a:1}` are contextual; whole assignment targets like `return[0].name = x` stay ordinary. Emitted logical NEWLINE after bare `return` does not join operand, while multiline parenthesized operands work. Parse-format-parse preserves code meaning/spelling, comment attachment, semantic spans, and idempotence. No reserved-word, ASI, additional semantic AST/Truffle opcode or undue performance overhead.

Only once focal tests are green, run integrated project gates:

~~~bash
make test
make check
git diff --check
~~~

If a gate fails, hand only specific failure excerpts back for **same-slice** repair. Re-run impacted checks after repair and final gates on final bytes. No partial push, no optimistic PASS.

## Version and publication gate

Derive next product version from actual current HEAD `pom.xml` only after test success, update `pom.xml` and user `CHANGELOG.md` exactly once. Derive next global spec revision from actual current `spec/PROTOS_SPEC_CHANGELOG.md`; do **not** reserve historical guessed next numbers. Ensure normative spec/grammar change published in same product commit as tests/formatter. Before publishing, human verifies/synchronizes `origin/main` and resolves patch-owned overlap without wiping unrelated edits; if changed, rerun affected validation.

Stage only exact owned files (including newly added test manifest and docs/guide) using explicit `git add <paths>`. Never `git add -A` or force-push. Inspect staged diff:

~~~bash
git diff --cached --check
git diff --cached --stat
# After reviewing staged paths and final tests:
git commit -m "I138: implement contextual non-local return sugar"
git push origin main
git status --short
git log -1 --format='%H %s'
~~~

Do not run the above commit/push if tests fail, release metadata is incomplete or main moved incompatibly. The human may use local normal git preflight/sync commands under its own policy; the editor does not.

## Completion report and coordination handoff

Supply exact published Protos SHA, product and specification versions, list of changed paths, `git diff --check` PASS, focal suites PASS, `make test` PASS, `make check` PASS, applicable native PASS/NOT_RUN, and formatter/LSP regressions. The coordinating assistant then adds source-linked result evidence in `guillermomolina/protos-project-docs`, verifies live GitHub issue state and closes I138 **only after actual human publication and acceptance**. D234's ratified design remains closed independently of implementation status.
