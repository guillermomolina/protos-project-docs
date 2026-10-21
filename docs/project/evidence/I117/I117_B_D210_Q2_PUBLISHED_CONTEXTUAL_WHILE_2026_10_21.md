# I117-B — Published contextual while syntax (D210-Q2)

**Date:** 2026-10-21  
**Formal work:** [I117 / #942](https://github.com/guillermomolina/protos/issues/942), phase B  
**Ratified authority:** [D210 / #941](https://github.com/guillermomolina/protos/issues/941)  
**Product repository:** `guillermomolina/protos`  
**Exact publication:** [`7a122cdeeed91ca31690e2d39412dadad3518f72`](https://github.com/guillermomolina/protos/commit/7a122cdeeed91ca31690e2d39412dadad3518f72)  
**Product version:** `0.3.394-SNAPSHOT`  
**Normative revision:** `0.1.465`  
**Provenance:** read-only published-commit and changelog/spec inspection, combined with maintainer-reported local validations. The documentation author did not run builds/tests.

## Delivered behavior

The parser recognizes `while(condition) { body }` only when `while` is a bare, ungrouped identifier followed by exactly one ordinary non-spread condition argument and an attached, parameterless braced trailing Closure under ordinary newline rules. It remains a non-reserved identifier otherwise; the approved contextual shape takes precedence over a lexical `while` binding. Grouped `(while)(x) {body}`, member `obj.while(x) {body}`, non-attached, multiple-argument and spread calls keep their ordinary interpretation.

`SurfaceContextualWhile` preserves condition, trailing body Closure and source span. `Canonicalizer` lowers it to an ordinary zero-parameter condition `CanonicalClosure` sending `whileTrue` with the original parameterless body Closure argument: observationally `(() => condition).whileTrue() { body }`. There is no new canonical loop node, loop runtime, reserved token or hand-built bytecode. Repeated condition evaluation, exact canonical Boolean checking, last-body-result identity/null on zero loops, lexical capture, NLR, Future ownership, suspension, cancellation and Error handling remain governed by the existing `whileTrue` protocol.

AST-aware Hover, Lint, References, Definitions, Document Symbols, Source Layout and the guest tooling bridge traverse condition/body without treating the syntactic `while` as a lexical reference. Source facts now associate trailing Closures with real `SurfaceCall` or `SurfaceContextualWhile` owners. TOOL010 supports `CONTEXTUAL_WHILE` and retains comments between `while` and `(`, between `(` and condition, and between `)` and `{`, preserving parseability and idempotence.

Publication includes `protos/tests/conformance/control/while-contextual-syntax.protos`, conformance manifest registration, parser/canonicalization/tool/semantic regression tests, `spec/PROTOS_GRAMMAR.md`, `spec/PROTOS_LANGUAGE_SPEC.md`, `spec/semantics/EXECUTION_AND_CONTROL.md` and the control-flow guide. Exactly **30 changed paths** in the published commit, including `pom.xml`, `CHANGELOG.md`, `spec/PROTOS_SPEC_CHANGELOG.md`.

## Validation and publication evidence

The maintainer explicitly reported `git diff --check` clean, all local tests green and `I117-B` pushed. The published `CHANGELOG.md` records focused JUnit PASS (including formatter and PERF026), `while-contextual-syntax.protos` **20/20**, `while-basic-and-validation.protos` **13/13**, `while-loop-results.protos` **17/17**, `make test` PASS and `make check` PASS. These are **maintainer-reported validations**, not independently reproduced runs or CI verification.

## Boundaries and follow-on

- I117-A is independently published at [`5c76e4a633c9aa44067fe0b4fd68cafea365750d`](https://github.com/guillermomolina/protos/commit/5c76e4a633c9aa44067fe0b4fd68cafea365750d), version `0.3.393-SNAPSHOT`, spec `0.1.464`.
- I117-A plus I117-B complete the ratified D210 Q1/Q2 implementation scope of [#942](https://github.com/guillermomolina/protos/issues/942).
- Widespread first-party adoption of idiomatic contextual `while` is a **separate follow-on implementation task**, not part of the I117-B publication. Explicit protocol tests and non-equivalent `.whileTrue` calls must remain.
