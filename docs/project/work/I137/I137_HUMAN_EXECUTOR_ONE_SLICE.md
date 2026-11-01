# I137 — Human executor: one atomic Protos implementation slice

## Scope and ownership

**Repository:** ONLY current local `guillermomolina/protos` checkout. **Work:** IMPLEMENTATION, not investigation. No other repository or web needed by the editing agent. **Issue:** https://github.com/guillermomolina/protos/issues/1017. **Exact owner-approved design:** D233-B1, owner statement “Apruebo este contrato” (2026-11-01), durable decision published under `guillermomolina/protos-project-docs:docs/project/decisions/language/D233_FIXED_BITWISE_OPERATOR_SUGAR_B1.md` at `9878c0bea34bcc483b77f74005861f7c18de86fb`. The paired editor prompt is `docs/project/work/I137/I137_AGENT_ONE_SLICE_PROMPT.md` in that docs repository. This handoff supplies the necessary external facts, and the editor must use **only** the local Protos repository.

**One I137 slice, one final Protos product commit/push.** Do not split lexer, parser, formatter, tests, syntax adoption, JVM and Native into several slices or auxiliary implementation issues; repair/validation loops remain within this slice.

## Explicit authority separation

**Editing agent:** reads and edits files in supplied local Protos HEAD, preserving unrelated work. No builds, tests, formatter invocations, scripts, package installs, local executions, Git add/commit/push/pull/fetch/checkout, or calls to other repositories/web.

**Human executor:** run verification/validation, synchronize current `origin/main`, inspect overlap, handle release version/changelog integration according to the agent's complete edit, stage exactly owned paths, commit/push once. Report concise PASS/FAIL or the specific failed log. Never treat docs ratification as proof that product code exists.

## Already decided contract

`&` -> `bitAnd`; `|` -> `bitOr`; binary `~` -> `bitXor`; prefix `~` -> `bitNot`; `<<` -> `shiftLeft`; `>>` -> `shiftRight`. Exactly 5 new fixed tokens, strict binary normal sends and user overrides, no new Integer kernel; relative precedence: postfix > unary(!,-,~) > multiplicative > additive > shift > & > binary ~ > | > comparisons > equality > && > ||, all new binaries left-associative. Preserve maximal munch and nonlocal `^`; no `>>>`, compound assignments, extra selectors or user-defined operators. `&&` and `||` remain lazy. Adopt new normative syntax only once both grammar and formatter tests are safe.

## Initial preflight (human)

Before asking the editor to work, ensure the checkout is the intended checkout and already at HEAD. Inspect status and paths without moving branches:

~~~bash
pwd
git status --short
git log -1 --format='%H %s'
git diff --name-only
git diff --cached --name-only
~~~

Tell the editor about any local modified patch-owned path so it can preserve it. Avoid temporary branches/worktrees and do not overwrite concurrent edits. The project requires synchronization with actual `origin/main` **before publishing**; the human owns that operation and must verify/reconcile path overlaps, not silently reset.

## After the agent has made all edits

First review the exact changed paths and source:

~~~bash
git status --short
git diff --check
git diff --stat
git diff -- spec/PROTOS_GRAMMAR.md src/main/java/com/guillermomolina/protos/lexer/ProtosLexer.java src/main/java/com/guillermomolina/protos/parser/ProtosParser.java src/main/java/com/guillermomolina/protos/semantic/Canonicalizer.java
~~~

Review all remaining patch-owned diffs including the Protos-first formatter `protos/tools/formatter/Structural.protos`, tests, docs, spec changelog, version/changelog and any newly added untracked conformance file. `git diff` alone omits untracked files. The agent must list all changed paths and licensing checks. No unrelated staged paths may be included.

Perform the cheapest applicable static checks and the **full affected focal regression set**, using current build/test conventions and `scripts/validation_impact.py`/`scripts/publication_validation.py` as specified by `AGENTS.work/IMPLEMENTATION.md` (read actual command-line usage locally; do not invent flags). At a minimum verify lexer/parser/Canonicalizer, override semantics, integer bitwise conformance, formatter round trips and LSP/REPL where impacted. Explicitly distinguish JVM and Protos Test Tool lanes. Only after focal checks pass, run the integrated gates once:

~~~bash
make test
make check
git diff --check
~~~

These are human-executed and potentially lengthy; never claim PASS until results are reported. Native Image/REPL smoke verification applies when the impact policy requires it; do not infer Native PASS from a Java-only test. If failed, provide only the error evidence necessary for the editing agent to repair this same slice, then rerun impacted checks and final gates on changed bytes.

## Mandatory acceptance observations

Verify strict canonical ordinary dispatch for `&`, `|`, `~` unary/binary and shifts; original receiver and selected user method (custom non-Integer receiver method works); left-to-right exactly-once evaluation, no new Bytecode DSL numeric operation. Test standard `(-3) >> 1 = -2`, `1 << 64`, `~0`, big shift count and negative count Error. Verify `&&`, `||`, `!` unchanged; `^` nonlocal return unchanged. Verify full-token maximal munch rejection of `a~~b`, `a&-b`, `a<<-1`, `a>>=b`, `a>>>b`, `a~=b` and continued invalid `^^a`, `!!a`. Verify `a ~ ~b`, binary line continuation and standalone prefix after completed newline. **Formatter must not turn `~ -a` into `~-a` or `- ~a` into `-~a`**, and parse/format/parse must preserve the grouping and behavior.

## One final product commit

After the integrated validation passes, have the editing agent reconcile current `pom.xml` and `spec/PROTOS_SPEC_CHANGELOG.md` version with actual HEAD and apply one complete version/changelog update as part of the same patch. **Do not reserve a speculative 0.3.xxx or 0.1.xxx in advance.** Before committing, human synchronizes with current `origin/main`, rechecks overlapping patch-owned paths and reruns affected validation if baseline materially changed.

Stage only the exact patch-owned paths reported by the agent. Example structure, **not** a command to stage files that were not actually edited:

~~~bash
git add spec/PROTOS_GRAMMAR.md spec/semantics/VALUES_AND_COLLECTIONS.md spec/PROTOS_SPEC_CHANGELOG.md
git add src/main/java/com/guillermomolina/protos/lexer/TokenType.java src/main/java/com/guillermomolina/protos/lexer/ProtosLexer.java src/main/java/com/guillermomolina/protos/parser/ProtosParser.java src/main/java/com/guillermomolina/protos/semantic/Canonicalizer.java
git add protos/tools/formatter/Structural.protos
# Add only the actual further patch-owned test, conformance-manifest, guide, pom.xml and CHANGELOG.md paths one by one.
git diff --cached --check
git diff --cached --stat
git commit -m "I137: implement fixed bitwise operator sugar"
git push origin main
git status --short
git log -1 --format='%H %s'
~~~

Inspect the staged diff **including newly added files**, and exclude all unrelated changes. Never `git add -A`/`.` or force-push. The repository's Human-executor workflow owns these commands, not the editor.

## Publication/issue reconciliation handoff

Report the exact final product SHA, product/spec versions, changed paths, `git diff --check`, `make test`, `make check`, applicable native result and SHA256/formatter tests PASS/FAIL. The coordinating assistant then updates I137 with real revision-coupled evidence in `protos-project-docs`, rereads both repositories and closes only after documented acceptance. D233's design ratification is complete regardless of pending I137 code. If new observable semantic ambiguity appears, return for exact owner approval; never change B1 silently.