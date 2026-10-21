# I118-A — Adopt contextual while in bounded Standard Library modules

**Date:** 2026-10-21  
**Class:** IMPLEMENTATION (I118-A), not investigation  
**Product repository:** `guillermomolina/protos`  
**Issue:** [I118 / #943](https://github.com/guillermomolina/protos/issues/943)  
**Prerequisite:** I117-B product [`7a122cdeeed91ca31690e2d39412dadad3518f72`](https://github.com/guillermomolina/protos/commit/7a122cdeeed91ca31690e2d39412dadad3518f72), specification `0.1.465`; executor's current HEAD is authoritative.  
**Ratified authority:** [D210 / #941](https://github.com/guillermomolina/protos/issues/941), normative `spec/PROTOS_GRAMMAR.md` and `spec/semantics/EXECUTION_AND_CONTROL.md`.  
**Style:** `protos/AGENTS.md`, `docs/guide/SOURCE_STYLE.md`.  
**Validation authority:** `AGENTS.work/IMPLEMENTATION.md`; human executor only.

## Bounded file inventory

Read-only GitHub search and file inspection on the published I117-B HEAD found the following `.whileTrue(` occurrences in 12 ordinary `protos/lib` modules. These are **occurrence counts, not automatically approved replacements**; each call must be inspected structurally before editing.

| Owned file | Observed whileTrue spellings | Notes |
|---|---:|---|
| `protos/lib/core/Path.protos` | 2 | literal single-expression conditions, braced trailing bodies |
| `protos/lib/math/Integer.protos` | 3 | same |
| `protos/lib/datetime/ISO8601.protos` | 14 | same; nested parentheses in predicates |
| `protos/lib/collections/Range.protos` | 2 | same |
| `protos/lib/network/IpEndpoints.protos` | 4 | three simple conditions; one multiline braced Closure `}).whileTrue()` is a non-trivial case and should be preserved |
| `protos/lib/network/IpAddresses.protos` | 14 | thirteen simple conditions; one `}).whileTrue()` condition body is non-trivial; preserve |
| `protos/lib/logging/TextFormatter.protos` | 10 | zero-argument explicit `.whileTrue(() => { ... })` bodies |
| `protos/lib/logging/ColoredFormatter.protos` | 2 | zero-argument explicit bodies |
| `protos/lib/logging/JsonFormatter.protos` | 1 | zero-argument explicit body |
| `protos/lib/io/Files.protos` | 2 | zero-argument explicit bodies |
| `protos/lib/semver/SemVer.protos` | 5 | braced trailing bodies |
| `protos/lib/crypto/SHA256.protos` | 1 | braced trailing body |

No other paths are owned by this slice, except final `pom.xml` and root `CHANGELOG.md` after the human reports tests green. Do not change any normative specifications or spec changelog. The absolute 30-file limit still applies.

## Editor work — precise transformations

1. Files `core/Path.protos`, `math/Integer.protos`, `collections/Range.protos`, `semver/SemVer.protos`, `crypto/SHA256.protos`: replace only single-expression `(() => E).whileTrue() { B }` with `while(E) { B }`. Preserve nesting, evaluation count/order, body text, result identity, comments, line boundaries. No functional changes.
2. `datetime/ISO8601.protos`: apply exactly the same transformation to its 14 simple forms, including parenthesized compounds. Do not remove boolean grouping that ensures original precedence.
3. `network/IpEndpoints.protos`, `network/IpAddresses.protos`: migrate only the identifiable single-expression arrow-body predicates on the `.whileTrue() { ... }` shape. **Retain** each multi-statement `(() => { ... }).whileTrue() { ... }` form and record why it stays. Do not attempt `while({ ... })`: braces have different source semantics.
4. `logging/TextFormatter.protos`, `logging/ColoredFormatter.protos`, `logging/JsonFormatter.protos`, `io/Files.protos`: for the explicitly supplied **zero-parameter, braced** body `(() => E).whileTrue(() => { B })`, convert to `while(E) { B }` with complete matching of both Closure delimiters. Do not alter any call to a named, parameterized, computed, or non-braced Closure.
5. Keep explicit `whileTrue` when its receiver is a named/computed Closure (selector reflection, receiver method-home, protocol intent) or its condition is a braced multi-expression Closure. Such exceptions are normal and are NOT workarounds; enumerate exact surviving sites and reasons.
6. Do not modify tests, benchmark baselines, Truffle/Java/runtime/compiler, scanner, grammar, canonicalizer, formatter, source-style guard or exception count speculatively. Source-style guard checks, coverage and full suite are **human** tasks.

Safeguards: change exact spelling only when the pattern is provably equivalent to the ratified D210 desugaring. `while` remains a contextual expression, never a new keyword. Preserve `this`, `super`, `context`, local capture, NLR, Future ownership, shadowing/selection, strict Boolean tests, Error and cancellation. Do not turn a literal `true` condition into a constant expression for optimization. No 'native' `while` runtime or extra bytecode.

## Truffle-language precedent for editing (no external lookup)

Oracle Graal **SimpleLanguage** uses `SLBytecodeParser.visitWhile_statement` and `beginWhile/endWhile` for a *statement*; **GraalJS** `WhileNode` implements JavaScript loops through `LoopNode`/`RepeatingNode`; **GraalPy** `while` follows Python statement/truthiness rules. None applies to Protos's return-value, object protocol or Closure semantics. This slice must not borrow any of these mechanisms. It only uses existing Protos canonical lowering `while(E) {B}` = `(() => E).whileTrue() {B}`.

## Human executor

Agent performs edits only (not runs). Human first audits changed paths and exact diff, then checks `git diff --check`, run focused affected Standard Library/control/IO tests (selection by actual changed-file closure), and request the integrated gate if prescribed by `scripts/validation_impact.py`. No tests were independently executed in preparing this handoff.

After human reports green, editor rereads current HEAD and moving metadata and edits `pom.xml` and root `CHANGELOG.md` to the then-current next implementation version (no new normative spec revision). Human reruns any final nonbehavioral checks required by policy; do not repeat behavior tests after metadata-only changes unless executable/dependency bytes changed. Human explicitly stages owned files, commits single line `I118-A: adopt contextual while syntax in standard library modules`, pushes. Only after exact product SHA exists does documentation publication and Issue work log update occur.

## First PBATCH edit content

Safe initial edit subset: `core/Path.protos`, `math/Integer.protos`, `collections/Range.protos`, `crypto/SHA256.protos`, `semver/SemVer.protos`. Match exact old spellings from the current HEAD with assertions, apply the corresponding new spelling and no other changes. The agent must inspect ownership/overlap before patch; commands belong to the editor, never the human test step. Further files in the first slice are handled in subsequent editor PBATCHes before handing validation to the human.
