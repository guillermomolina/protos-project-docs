# I118 — Single-slice contextual while adoption: Claude editor and human executor

**Date:** 2026-10-21  
**Issue:** [I118 / #943](https://github.com/guillermomolina/protos/issues/943)  
**Class:** IMPLEMENTATION, **one unified slice**, **one Claude prompt**, **one product publication**  
**Product repository:** `guillermomolina/protos`  
**Published prerequisite:** [I117-B `7a122cdeeed91ca31690e2d39412dadad3518f72`](https://github.com/guillermomolina/protos/commit/7a122cdeeed91ca31690e2d39412dadad3518f72), implementation `0.3.394-SNAPSHOT`, spec `0.1.465`. Local editor HEAD is authoritative.  
**Normative authority:** D210/#941, `spec/PROTOS_GRAMMAR.md`, `spec/semantics/EXECUTION_AND_CONTROL.md`, `spec/semantics/CALLABLES.md`.  
**Style and execution policy:** `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `protos/AGENTS.md`, `docs/guide/SOURCE_STYLE.md`; human runs all tests/publication commands.

> **Supersession:** The owner's explicit instruction is to combine all originally proposed I118-A/B/C/D source migrations into **one I118 implementation slice**. The earlier limited 12-module plan under this path is obsolete. No intermediate product commits, new sub-issues, multiple Claude prompts, or separate release/checkpoints. The single final patch may exceed 30 modified files; do not reduce scope by arbitrary truncation or delete tests. The normative/runtime constraints remain unchanged.

## Scope and one-pass file inventory

Read-only GitHub code search from the prerequisite revision found **92 files referencing `whileTrue(`**. That number includes normative documents, generated/test strings, protocol-specific tests, benchmarks, and sources that must be preserved; it is **not** a target count of edits. Eligible first-party code occurs in these broad groups:

- `protos/lib/**`: core/Path, math/Integer, datetime/ISO8601, uri, collections/Range and Array, network/IpEndpoints and IpAddresses, logging/TextFormatter/ColoredFormatter/JsonFormatter, io/Files, semver/SemVer, cli/CommandLine, regex/Regex, toml/TOML, crypto/SHA256, plus any current HEAD equivalents.
- `protos/tools/**`: formatter/Structural, package/ProjectMetadata and ContentIdentity, and test-tool files ArrayAccumulator, CaseRef, Progress, LogicalCaseRunner, SuiteGraph, Main, FileSelection, Runner, RepositorySuite, Manifest, etc.
- `protos/examples/**`, `protos/tutorials/**`: ordinary teaching material and programs.
- `protos/tests/**`: ordinary library, tooling and conformance programs that merely *use* a loop; keep explicit `whileTrue` where it IS the test target (validation/selection/identity/return/Future/suspend/cancel/structured/inline/dispatch).
- `protos/benchmarks/**`: migrate only source whose measurement identity and cross-language comparability are not invalidated; preserve fixed baseline and same-source workloads when changing source would destroy longitudinal evidence.
- Java/Python/shell fixture strings: default KEEP when they encode the precise `whileTrue` spelling, expected graph/source spans, protocol behavior, baseline workloads or benchmark comparability; optional migration only when ordinary source and rigorously equivalent.
- Normative specification and old changelogs: KEEP `whileTrue` protocol examples; never rewrite historical evidence for style.

## Exact authorized edits

**A.** For fresh zero-arg expression-bodied condition and parameterless trailing body:
```protos
(() => COND).whileTrue() { BODY }
```
becomes
```protos
while(COND) { BODY }
```

**B.** For the same receiver with an explicit zero-arg braced body:
```protos
(() => COND).whileTrue(() => { BODY })
```
becomes
```protos
while(COND) { BODY }
```
Be careful about original double-closure delimiters and multi-expression body formatting.

**C.** For `(() => { EXPR }).whileTrue() { BODY }`, migrate only when its condition Closure has exactly one ordinary expression, with no slot creation, locals/declarations, comments misplaced, or other structural difference; otherwise retain the explicit protocol spelling.

**D.** Retain all receiver variables or computed/extracted Closures `condition.whileTrue(body)`, non-zero parameter Closures, condition bodies with multiple expressions, explicit protocol tests, source-location and benchmark fixtures, cases with relevant `whileTrue` extraction/shadowing/method-home behavior, or any non-equivalent/newline/comment case. Do NOT mechanically replace named receiver calls, nor turn `while` into a reserved word.

**E.** Preserve `this`, `super`, `context`, lexical captures, return-home/NLR, Future ownership, exact last-result identity and zero-run `null`, strict Booleans, Error/cancel/ensure, evaluation order and count. No runtime/Truffle/Bytecode/AST/parser/formatter changes are authorized to enable conversion.

**F.** Preserve APL notices and local file style. Use targeted guarded Python file edits with assertions; a textual regex can only help *find* a simple same-line form, not authorize nested/complex replacement. Read target file before writing, honor unrelated work. Do not add redundant source-style exceptions to justify leaving mechanically convertible code or modify baselines to force tests green.

## Human-only checks and publication

One final substantive candidate, validated in increasing cost order:
```bash
git status --short
git diff --check
git diff --stat
# focused affected test-tool, formatter, libraries and conformance, as determined by exact changed paths
make test
make check
```
The repository's `scripts/source_style_guard.py` and `scripts/validation_impact.py` are part of established source-style/validation discipline; humans run gates with current documented command-line flags, do not guess invocation syntax. The human will return PASS/FAIL (with failing excerpt only). The editor does not run tests/build/validation/git mutations.

After human green, metadata finalization reads current HEAD and `origin/main`, and updates `pom.xml` / root `CHANGELOG.md` exactly once to the current next patch version. Style-only migration introduces **no normative spec revision**. Human stages ONLY owned paths (explicit `git add` lines; never `git add -A`), commits `I118: adopt contextual while in first-party Protos sources`, and pushes. No post-metadata behavior tests unless relevant executable bytes have changed. After exact product SHA is published, append durable per-revision evidence in `docs/project/evidence/I118/`, and update/close #943 when all acceptance criteria and publication gates are met.

## Prior art included — no external lookup

Truffle SimpleLanguage's `SLBytecodeParser.visitWhile_statement` + `beginWhile/endWhile`, GraalJS `WhileNode` via `LoopNode`/`RepeatingNode`, and GraalPy's while statement all embody independent host-language statement or truthiness/loop semantics. These are explicitly **not** the required algorithm. Protos already specifies `while(COND) {BODY}` as an ordinary `whileTrue` message sent to a zero-parameter Closure. This slice is source-only.

## Editor's initial PBATCH — actual file editing

First safe group: `core/Path.protos` (2), `math/Integer.protos` (3), `collections/Range.protos` (2), `crypto/SHA256.protos` (1), `semver/SemVer.protos` (5) = 13 known `.whileTrue` sites. A guarded Python script can replace these exact one-line arrow predicates, after reading and verifying their current shape and expected counts; the complete one-slice agent prompt will own all remaining groups in the same editing session. No human code execution is required for the edits; the human remains responsible for validation.

## Final report required

One complete changed-path list; per-directory counts of transformed sites; deliberate-survivor ledger with exact path/line/semantic reason; statement of no syntax/runtime/backend changes; files materially inspected and normative/style constraints applied; human focal + integrated commands. No ungrounded performance improvement claims.
