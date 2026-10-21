# I119 — Published contextual if/else (D211 A+B+C)

**Date:** 2026-10-21  
**Formal work:** [I119 / #945](https://github.com/guillermomolina/protos/issues/945), implementation of ratified [D211 / #944](https://github.com/guillermomolina/protos/issues/944).  
**Exact product revision:** [`2a3d373d06792384f0c62cb06845b1f1d62bea6f`](https://github.com/guillermomolina/protos/commit/2a3d373d06792384f0c62cb06845b1f1d62bea6f).  
**Exact product commit message:** `I119: add contextual if/else lowering to ifTrue/ifTrueIfFalse and remove standard Object.if/else (0.3.396)`.  
**Implementation version:** `0.3.396-SNAPSHOT` (published `CHANGELOG.md`).  
**Specification revision:** `0.1.466` (published `spec/PROTOS_SPEC_CHANGELOG.md`).  
**Design authority:** [ratified D211](../../decisions/language/D211_CONTEXTUAL_IF_BUNDLE.md) and [falsification/approval evidence](../D211/D211_CONTEXTUAL_IF_FALSIFICATION_AND_RATIFICATION_2026_10_21.md).  
**Implementation work records:** [editor handoff](../../work/I119/I119_AGENT_EDITOR_2026_10_21.md), [human executor](../../work/I119/I119_HUMAN_EXECUTOR_2026_10_21.md).

## Published A+B+C as one product revision

- **A — standard protocol removal:** standard `Object.if(block)` and `Object.else(block)` receiver-preserving actions are removed, including their root slots, native callback cases, guard/structured specialization, Bytecode lowering rows and reflection/extraction authority. Custom ordinary `if` and `else` member names remain valid.
- **B — one-arm syntax:** the exact bare contextual `if(condition) { yes }` lowers to ordinary `condition.ifTrue(() => { yes })`. The condition is evaluated once and eagerly in the caller activation; the selected branch's exact normal result is returned; false returns `null`.
- **C — two-arm syntax:** `if(condition) { yes } else { no }` lowers to ordinary `condition.ifTrueIfFalse(() => { yes }, () => { no })`. Both parameterless branch Closures are created in ordinary argument order, but only the selected one executes. No truthiness conversion, await/adoption, keyword reservation or separate conditional runtime primitive is introduced.
- **Syntax boundary:** only one non-spread condition argument on bare ungrouped `if`, attached braced body, and immediately paired same-logical-line `else`. `(if)(...)`, `obj.if(...)`, standalone `else` and nonmatching ordinary calls retain their meanings. This intentionally supersedes the narrow D051 lexical-call interpretation and removes the D200 standard Boolean action slots while preserving D050/D199/D210 behavior.
- **Toolchain:** `SurfaceContextualIf` surface AST, canonical send lowering, Static Definitions/References/Hover/Lint, document symbols, source-facts `ContextualElseOrigin`, source-layout projections, and bundled formatter all support the new expression and comments. Conformance and Java tests cover exact branch results/identity, dispatch overrides, lexical collisions, newline pairing, nested expressions, nonlocal returns, Futures, cancellation, suspension, and formatter idempotence.

The published commit also updates `ProtosI026FDapTransportTest.java` to tolerate a narrowly identified upstream GraalVM DAP closed-socket write after confirmed session disconnect during `Context.close()`. That test-harness adjustment is documented separately in the product `CHANGELOG.md`; it is not part of D211 semantics.

## Human validation and publication evidence

The maintainer explicitly reported that `git diff --check` was clean, **all local tests passed**, and the I119 commit was pushed. The published product `CHANGELOG.md` additionally records `make check` PASS and `make test` PASS. These are **maintainer-reported** results, not new tests independently rerun by the documentation coordinator.

Read-only GitHub verification found the exact published commit, the `0.3.396-SNAPSHOT` changelog entry and the `0.1.466` normative specification changelog entry. The product's single commit contains the parser, runtime, tests, formatter, guide, normative specs and final metadata together.

## Completion and downstream boundary

I119 is completed and may close after this revision-coupled durable record is published and cross-checked. The follow-on, independently scoped **I120** source-style migration may adopt the new contextual spellings in ordinary first-party source under the existing source-style policy, but **must not** change D211 semantics or suppress direct `ifTrue`/`ifTrueIfFalse` protocol/conformance coverage. I119 is not held open for that migration.

**Product state identity:** `PROTOS_REVISION=2a3d373d06792384f0c62cb06845b1f1d62bea6f`.  
**Validation evidence:** `GIT_DIFF_CHECK=MAINTAINER_REPORTED_CLEAN`; `LOCAL_TESTS=MAINTAINER_REPORTED_PASS`; `MAKE_CHECK=PRODUCT_CHANGELOG_REPORTED_PASS`; `MAKE_TEST=PRODUCT_CHANGELOG_REPORTED_PASS`.
