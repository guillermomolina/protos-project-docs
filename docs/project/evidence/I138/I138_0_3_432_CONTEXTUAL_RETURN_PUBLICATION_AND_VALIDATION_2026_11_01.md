# I138 — Contextual non-local return sugar: publication and validation evidence (0.3.432)

**Date:** 2026-11-01. **Implementation owner:** [I138 #1021](https://github.com/guillermomolina/protos/issues/1021). **Design authority:** [D234 #1018](https://github.com/guillermomolina/protos/issues/1018), owner-ratified D234-B2-S1, exact [durable decision](../../decisions/language/D234_RETURN_CONTEXTUAL_SUGAR_B2_S1.md) and [GITHUB021/research evidence](../D234/D234_B2_S1_RESEARCH_GITHUB021_APPROVAL_2026_11_01.md) published at docs revision `5d7f688d6554452fdf5432a59f069792daee66f4`.

**Evidence distinction:** independently inspected GitHub publication and source artifacts, plus **human-maintainer-reported** local validation results. GitHub source inspection does not execute product tests or validate the maintainer's local working tree. No new language decision was made here.

## Exact source identity

- **Product repository:** `guillermomolina/protos` (exact spelling).
- **Product published SHA / verified main HEAD at audit:** [`ab637aa264ddbb2b99fdb03cb19207cc19651f45`](https://github.com/guillermomolina/protos/commit/ab637aa264ddbb2b99fdb03cb19207cc19651f45).
- **Commit title:** `I138: implement contextual non-local return sugar (0.3.432)`.
- **Parent:** `c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5` (I137 published baseline).
- **Product version:** `0.3.432-SNAPSHOT` in [`pom.xml`](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/pom.xml) and [`CHANGELOG.md`](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/CHANGELOG.md).
- **Normative specification revision:** `0.1.487` in [`spec/PROTOS_SPEC_CHANGELOG.md`](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/spec/PROTOS_SPEC_CHANGELOG.md).
- **Commit size:** 24 paths, 1,319 additions and 34 deletions, including two newly added Protos/JUnit test sources, spec, tooling, guide and metadata.
- **GitHub observation:** the SHA, parent, message, file inventory, versions and spec revision were read from the published repository; no uncommitted implementation changes have been inspected as part of this publication confirmation.

## What was published, with immutable file references

1. [Parser structural contextual recognition](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/src/main/java/com/guillermomolina/protos/parser/TokenCursor.java) and [`ProtosParser.java`](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/src/main/java/com/guillermomolina/protos/parser/ProtosParser.java): ungrouped ordinary `return` identifier in an eligible expression-start position, with one-expression parenthesized, postfix assignment-target and logical NEWLINE discrimination. No new reserved keyword or lexer token.
2. [`SurfaceNonLocalReturn.java`](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/src/main/java/com/guillermomolina/protos/parser/ast/SurfaceNonLocalReturn.java) keeps source spelling (`^` / `return`) for tooling; the existing canonicalizer maps both to `CanonicalReturn`. The canonical runtime, return-home engine and `CanonicalReturnHomeAnalysis` remain unchanged by the I138 commit.
3. [`ProtosSourceLayoutView.java`](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/src/main/java/com/guillermomolina/protos/analysis/ProtosSourceLayoutView.java) adds the `NON_LOCAL_RETURN_OPERAND` comment boundary; [`ProtosSourceLayoutToolBridge.java`](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/src/main/java/com/guillermomolina/protos/execution/host/tooling/ProtosSourceLayoutToolBridge.java) projects spelling and comments; the [Protos-first formatter](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/protos/tools/formatter/Structural.protos) preserves both spellings and same-line comments. [`ProtosStaticLint.java`](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/src/main/java/com/guillermomolina/protos/analysis/ProtosStaticLint.java) references the actual source spelling in unreachable-code diagnostics.
4. [`spec/PROTOS_GRAMMAR.md`](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/spec/PROTOS_GRAMMAR.md) sections 11/35/36 and compact EBNF, [`spec/semantics/CALLABLES.md`](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/spec/semantics/CALLABLES.md) §13, and [`spec/PROTOS_LANGUAGE_SPEC.md`](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/spec/PROTOS_LANGUAGE_SPEC.md) make the approved D234-B2-S1 semantics normative. `return expr` is `^expr`, preserving captured return homes, `InvalidReturn`, Task confinement, normal closure completion and `ensure` effects; no JS-style callback-local return.
5. [`ProtosParserContextualReturnTest.java`](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/src/test/java/com/guillermomolina/protos/parser/ProtosParserContextualReturnTest.java), [`protos/tests/conformance/surface-sugar/contextual-return.protos`](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/protos/tests/conformance/surface-sugar/contextual-return.protos) and [manifest registration](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/protos/tests/conformance/manifest.tsv), together with lexer/canonicalizer/formatter/layout/lint/LSP/REPL regressions, exercise exact source distinctions, return-home targets, escaped/foreign Task failure, cleanup and formatting. The conformance module intentionally uses module-level helpers to prevent its own Test body from exiting before assertions.
6. [Updated closure guide](https://github.com/guillermomolina/protos/blob/ab637aa264ddbb2b99fdb03cb19207cc19651f45/docs/guide/03-closures-methods-and-receivers.md) introduces the alias and the JS callback semantic difference. **Full runnable teaching progression remains separate, owned by DOC015 #1025**.

## Acceptance evidence and scope

**Owner report, 2026-11-01:**

> `I138: implement contextual non-local return sugar (0.3.432) pushed.`  
> `el git diff check esta limpio.`  
> `make test y make check pasan en local.`

The user additionally reported **focused tests PASS** after the agent's implementation and submitted a diff that was reviewed before the full-suite report.

| Gate | Evidence | Result |
| --- | --- | --- |
| Exact product publication and SHA | GitHub fetched commit and verified main branch | VERIFIED |
| Product version and normative spec revision | Read published `pom.xml` and spec changelog | VERIFIED |
| Focused JVM tests | Maintainer report in conversation | PASS (human reported) |
| `git diff --check` / clean tree claim | Maintainer report; GitHub cannot observe local tree | PASS / clean (human reported) |
| `make test` | Maintainer report | PASS (human reported) |
| `make check` | Maintainer report | PASS (human reported) |
| Native test separately | No result supplied | NOT CLAIMED |

**GITHUB020 closure boundary:** The source and required integrated-gate reports are present and attributed; the coordinating assistant may close I138 following exact durable-publication verification and live-Issue postcondition review. This record does **not** complete [DOC015 #1025](https://github.com/guillermomolina/protos/issues/1025) or affect independent [TEST011 #1023](https://github.com/guillermomolina/protos/issues/1023) / [BUG032 #1024](https://github.com/guillermomolina/protos/issues/1024).
