# I120 — One-slice contextual if/else adoption: Claude editor and human executor

**Date:** 2026-10-21  
**Formal implementation issue:** [I120 / #946](https://github.com/guillermomolina/protos/issues/946) — **ONE atomic source-style migration slice**, not investigation.  
**Product repository:** `guillermomolina/protos` (two occurrences of `mo` in `guillermomolina`).  
**Published predecessor:** [I119 product `2a3d373d06792384f0c62cb06845b1f1d62bea6f`](https://github.com/guillermomolina/protos/commit/2a3d373d06792384f0c62cb06845b1f1d62bea6f), `0.3.396-SNAPSHOT`, spec `0.1.466`.  
**Precedent:** [I118 / #943](https://github.com/guillermomolina/protos/issues/943) — one complete contextual-while adoption patch; [I118 published evidence](../../evidence/I118/I118_SINGLE_SLICE_CONTEXTUAL_WHILE_MIGRATION_PUBLISHED_2026_10_21.md).  
**Language and style authority:** `spec/PROTOS_GRAMMAR.md` §36; `spec/PROTOS_LANGUAGE_SPEC.md` §33; `spec/semantics/VALUES_AND_COLLECTIONS.md` §16; `docs/guide/SOURCE_STYLE.md`; `protos/AGENTS.md`.

## Definitive Claude implementation prompt — FILE EDITS ONLY

**Class:** IMPLEMENTATION `I120`, repository `/workspaces/protos` (or the user's existing local Protos checkout). Assume it is **already at HEAD**. Do not check out another revision. The project owner has selected **one single slice** for the entire eligible source migration, with one final commit only after human validation. The instructions below are edits to make, not an invitation to research language alternatives.

Read the applicable in-repository `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/REFERENCE.md`, `protos/AGENTS.md`, and `docs/guide/SOURCE_STYLE.md` to honor existing file/edit rules. D211 is already ratified and implemented. Respect existing local edits; do not overwrite unrelated work.

### A. Ordinary Protos code: convert all exactly eligible one-arm invocations

In **each** eligible file under `protos/lib/**`, `protos/tools/**`, `protos/examples/**`, `protos/tutorials/**` and non-protocol source under `protos/tests/**`, replace expression-matched:

```protos
CONDITION.ifTrue(() => { BODY })
CONDITION.ifTrue() { BODY }
```

with

```protos
if(CONDITION) { BODY }
```

**Initial concrete source-file targets** already known to contain ordinary uses in the immediate predecessor lineage: `protos/lib/core/Path.protos`, `protos/lib/csv/CSV.protos`, `protos/lib/io/Files.protos`, `protos/lib/text/ANSI.protos`, `protos/lib/semver/SemVer.protos`, `protos/lib/datetime/Time.protos`, `protos/lib/datetime/Date.protos`, `protos/lib/crypto/SHA256.protos`, `protos/lib/logging/Logger.protos`, `protos/lib/logging/Level.protos`, `protos/lib/text/ColorMode.protos`, `protos/lib/text/StyledText.protos`, `protos/lib/collections/Set.protos`, `protos/lib/datetime/Period.protos`, `protos/lib/logging/LogEvent.protos`, `protos/lib/datetime/ISO8601.protos`, `protos/lib/uri.protos`, `protos/lib/collections/Array.protos`, `protos/lib/network/IpEndpoints.protos`, `protos/tools/package/LockSyntax.protos`, `protos/tools/package/ProjectFile.protos`, `protos/tools/package/LockFile.protos`, `protos/lib/logging/JsonFormatter.protos`, `protos/lib/logging/TextFormatter.protos`, `protos/lib/datetime/Offset.protos`, and any other actually present files with eligible occurrences. In particular, cover `protos/tools/formatter/Structural.protos` and all other formatter/Test Tool/Package Tool implementation sources where the explicit selector is merely ordinary branching.

These paths are an **initial target inventory**, not an exhaustive frozen list: complete the migration throughout the first-party source tree. Use read-only `grep` and direct file inspection to locate the remaining occurrences; do not use `rg` or regex-only replacement for balanced nested expressions.

Keep the condition expression in the same evaluation position, exactly once and eagerly; maintain grouping/precedence, selected result, identity, custom `ifTrue` overrides, lexical capture, `this`/`super`/`context`, Error, NLR, Future/Task and suspension semantics. Do not replace `ifFalse` using negation.

Examples of the **exact intended type of edit**, adapting whitespace and indentation to the file:

```protos
// in Path.protos, hash:
(size < 0).ifTrue(() => { Error().signal() })
(h < 0).ifTrue(() => { h = h + 4294967296 })
```

becomes

```protos
if(size < 0) { Error().signal() }
if(h < 0) { h = h + 4294967296 }
```

And a plain `ready.ifTrue() { action() }` becomes `if(ready) { action() }`.

### B. Ordinary two-arm invocations: convert every exactly eligible literal pair

In **each** eligible file under the same ordinary-source trees, replace:

```protos
CONDITION.ifTrueIfFalse(
    () => { YES },
    () => { NO }
)
```

with the new paired expression:

```protos
if(CONDITION) {
    YES
} else {
    NO
}
```

Both callbacks MUST be freshly constructed, **parameterless braced Closure literals**, not callable references, parameterized callbacks, spread arguments, call-producing expressions or nonliteral invokables. Their construction and capture remain ordinary argument semantics, in the same order; only the selected callback body executes. The value returned by the entire new conditional must remain exactly the value of the original send. Preserve any immediately following member/index/call suffix on the **whole** conditional (use grouping when necessary). Preserve existing comments with correct structural ownership; never insert a logical NEWLINE between the true branch's closing `}` and its `else`, or between `else` and `{`.

**Initial concrete two-arm targets:** `protos/lib/core/Object.protos`, `protos/lib/core/Fraction.protos`, `protos/lib/text/StyledText.protos`, `protos/lib/text/ANSI.protos`, `protos/lib/math/Integer.protos`, `protos/lib/regex/Regex.protos`, `protos/lib/cli/CommandLine.protos`, `protos/tests/library/logging/colored-formatter.protos`, plus other eligible files discovered by the same read-only source search. Nested two-arm sends can become nested contextual if expressions; preserve the original nesting and branch result.

Do **not** turn two independent `ifTrue` and `ifFalse` sends into a single paired if: that would alter evaluation order/lookup. Do **not** transform `ifTrueIfFalse` calls with an ordinary callable object/variable in either branch.

### C. Test, benchmark, fixture and documentation boundaries

Convert uses in ordinary first-party tests when `ifTrue` or `ifTrueIfFalse` serves solely as control flow **inside a test of unrelated behavior**, not when the selector/spelling is the test's subject. Keep explicit selector invocations in relevant protocol conformance/dispatch/guard/inlining/canonicalizer/source-shape tests, including `protos/tests/conformance/boolean/if-true-if-false.protos`, `protos/tests/conformance/boolean/callback-dispatch-and-control.protos`, explicit selector checks in `protos/tests/conformance/boolean/receiver-preserving-if-else.protos`, and targeted `src/test/java/**/ProtosI095*`, `ProtosI119*` suites when their assertions demand specific selector semantics or shape. A test filename alone is not a blanket exclusion: keep only cases that explicitly test the protocol, and migrate other ordinary uses if source shape is not evidence.

Preserve benchmark-workload source equivalence when a frozen baseline would change, explicitly `protos/benchmarks/concurrency/*.protos` PERF001-F; preserve any other source-shape/performance comparison fixtures where source stability is an invariant. Ordinary examples/workloads not subject to such frozen constraints can be migrated. Do not silently claim that stylistic rewriting improves runtime performance or graph size.

In Java/Python/shell and other embedded Protos source, edit only self-contained **ordinary** Protos programs whose source shape is not independently asserted or baseline material. When changing a published bundled Tool source spelling, update only Java test assertions or fixtures that otherwise hardcode that old spelling. Do not change Java runtime/compiler/parser implementation in this style migration.

For user-facing `docs/guide/**`, README and ordinary example snippets: use contextual if idiom when the example teaches an unrelated capability. Preserve expanded `ifTrue`/`ifTrueIfFalse` forms where the material **explains the canonical protocol**, demonstrates desugaring, cites historical decisions, or teaches the selector itself. Do not alter historical changelogs/decision records or normative specifications.

### D. Preserve exact D211 semantics and source syntax

D211 is **not** a universal boolean transpilation or truthiness change. Absolutely no changes to:

- `src/main/java/**` parser, canonicalizer, protocol/runtime or Truffle lowering implementation;
- normative `spec/**` or `spec/PROTOS_SPEC_CHANGELOG.md`;
- standard `ifTrue`, `ifFalse`, `ifTrueIfFalse`, `and`, `or`, `not` behavior;
- literal `ifFalse` merely by wrapping condition in `not`/`!`;
- nonliteral callbacks or contexts whose capture/evaluation/order might change;
- reserved-word policy, lexical `if` behavior for noncontextual shapes or ordinary custom members.

Avoid introducing scope changes by unwrapping a multi-expression argument Closure that was not itself one of the two callback bodies. Reformat minimally to retain comments and existing local style. In a suspicious case, leave original untouched and record the exact reason rather than guessing. Do not create semantic compensating helpers.

### E. One complete batch and handoff

**ONE migration slice:** no arbitrarily imposed 30-file or other changed-path cap and no intermediate products, phases, branches, commits or releases. The owner's request overrides prior phased proposals, but does not override scoped file-size, licensing, style, code-review or semantic invariants.

Only file edits are authorized for the Claude editor. Read-only inspection with `grep`, `git status --short`, `git diff` and direct file reads is permitted for locating exact edits, but **do not execute compilation, tests, formatter executables, style/validation scripts, package installation, or state-changing Git commands**. Do not alter `pom.xml`, `CHANGELOG.md` or `spec/PROTOS_SPEC_CHANGELOG.md`. Do not make publication changes in `guillermomolina/protos-project-docs` as part of this editor pass.

Report:

1. Exact changed paths; counts of eligible one-arm and two-arm replacements, per source tree.
2. Each deliberate retained explicit invocation, by file/category/reason; prove exclusions are intentional, not work skipped due to size.
3. Any updated source-shape-only fixtures, no semantic modifications.
4. Human validation commands in cheapest-first order, including `git diff --check`, `python3 scripts/source_style_guard.py` (check accepted CLI options in local script first), focused suites and the required integrated `make check` and `make test`. The agent must **not execute them**.

Stop after file edits and the report. Human runs all required checks and reports outcomes; after explicit human green, metadata finalization from current HEAD occurs once, human commits/pushes exactly one single-line message `I120: adopt contextual if/else in first-party Protos sources`, no tests after metadata. **Only afterward** publish revision-bound evidence to this docs repository and close [I120 / #946](https://github.com/guillermomolina/protos/issues/946).

**Known inspected authority/source paths for this handoff:** `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/COORDINATION.md`, `AGENTS.work/REFERENCE.md`, `protos/AGENTS.md`, `docs/guide/SOURCE_STYLE.md`, `spec/PROTOS_GRAMMAR.md` §36, `spec/PROTOS_LANGUAGE_SPEC.md` §33, `spec/semantics/VALUES_AND_COLLECTIONS.md` §16, `CHANGELOG.md`, `spec/PROTOS_SPEC_CHANGELOG.md`, `protos/lib/core/Path.protos`, `protos/lib/core/Fraction.protos`, `AGENTS.md` (docs repository), and I118 published evidence. Concrete source-file names above are grounded in repository code-search results; repository search indexing may lag behind today's published I119 commit, so **the local current HEAD always wins**.
