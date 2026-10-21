# I117-A — Published single-selector loop protocol (D210-Q1)

**Date:** 2026-10-21  
**Formal work:** [I117 / #942](https://github.com/guillermomolina/protos/issues/942) (open; I117-B pending)  
**Ratified authority:** [D210 / #941](https://github.com/guillermomolina/protos/issues/941) and [D210 decision](../../decisions/language/D210_CLOSURE_WHILE_SELECTOR_AND_CONTEXTUAL_SYNTAX.md)  
**Exact published product revision:** [`5c76e4a633c9aa44067fe0b4fd68cafea365750d`](https://github.com/guillermomolina/protos/commit/5c76e4a633c9aa44067fe0b4fd68cafea365750d)  
**Commit:** `I117-A: remove standard Object.while and keep whileTrue as the single loop protocol`  
**Evidence scope:** read-only GitHub commit/file inspection and maintainer-reported local validation; no validation rerun by the documentation author.

## What was delivered

D210-Q1 is published as a separately reviewable implementation slice, ahead of D210-Q2.

- The standard `Object.while` slot/native body is removed. Only the original, identity-checked `Object.whileTrue` on the standard Object method home is admitted as canonical pre-test loop behavior. `while` stays a legal user-defined ordinary identifier or member name.
- The pre-test loop still validates the condition Closure, exactly one body argument, and the body Closure before its first predicate evaluation; it requires strict canonical Boolean results, tests before each iteration, preserves errors/transfers/suspension semantics, and returns canonical `null` for zero iterations or the exact last normal body result (including its identity and Future value).
- Removed the dual-selector `returnsLastBodyResult`/terminal-`false` result policy from the native implementation and prepared structured-control path. The structured and inline lowerers retain their generated control flow; they receive the single standard result behavior.
- Updated normative execution, callable, grammar and Future wording, the control-flow guide, Protos conformance and Java regression coverage, including root Object surface/reflection, selection/extraction, literal inline versus structured/fallback paths, zero/one/multiple bodies, result identity, Future ownership, suspended structured execution and nonlocal/error propagation. The final I117-A commit touches 22 paths, including publication metadata.

Key files in that commit include `ProtosStandardObjectProtocol.java`, `ProtosCoreBootstrap.java`, `ProtosPreparedCalls.java`, `ProtosStructuredControlCalls.java`, `CanonicalToBytecodeLowerer.java`, `ProtosStructuredDispatchLowerer.java`, `spec/semantics/EXECUTION_AND_CONTROL.md`, `spec/semantics/CALLABLES.md`, `spec/PROTOS_GRAMMAR.md`, and `protos/tests/conformance/control/while-loop-results.protos`.

## Validation and provenance

Maintainer explicitly reported for the published I117-A delta:

- `git diff --check`: clean;
- local tests: **all PASS**;
- `I117-A`: **pushed**.

This is a maintainer report, **not** an independently reproduced test or CI result. The product changelog names the executed validation suites and counts as maintainer-reported evidence, not as an independently reproduced run.

## Final publication metadata — corrected after main moved

The final published `main` head [`5c76e4a633c9aa44067fe0b4fd68cafea365750d`](https://github.com/guillermomolina/protos/commit/5c76e4a633c9aa44067fe0b4fd68cafea365750d) supersedes prior revisions `9641eefc` and `8940e779`. It **includes** `pom.xml`, `CHANGELOG.md` and `spec/PROTOS_SPEC_CHANGELOG.md` in its changed paths: implementation version `0.3.393-SNAPSHOT` and normative specification revision `0.1.464`, with matching I117-A entries. The changelog specifically records maintainer validation: focal JUnit 25/25 PASS, learning-material JUnit 54/54 PASS, `make test` PASS and `make check` PASS. These remain claims supplied by the maintainer in published metadata; they were not independently rerun here.

Normative revision `0.1.464` records D210/I117-A in `spec/PROTOS_SPEC_CHANGELOG.md`, covering execution/control, callables, grammar and Futures/task ownership wording. The former metadata discrepancy is resolved by the final publication revision.

## Next slice

**I117-B, same Issue #942, implementation in `guillermomolina/protos`.** Implement the D210-Q2 contextual expression `while(condition) { body }`, observable as `(() => condition).whileTrue() { body }`: two fresh lexical Closures, ordinary `whileTrue` selection, per-iteration reevaluation, exact returned body value/identity; precise bare-name/call/trailing-Closure boundary and approved precedence over a lexically defined `while` only for that exact source shape. Retain `while` as an identifier and member name and keep `(while)(x) { body }`, `obj.while(x) { body }`, `while(x)`, and newline-separated source on their ordinary parsing paths.

No grammar/parser implementation or tests of Q2 were part of I117-A. I117 remains open until I117-B is implemented, validated and published.
