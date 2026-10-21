# I117-A — Published single-selector loop protocol (D210-Q1)

**Date:** 2026-10-21  
**Formal work:** [I117 / #942](https://github.com/guillermomolina/protos/issues/942) (open; I117-B pending)  
**Ratified authority:** [D210 / #941](https://github.com/guillermomolina/protos/issues/941) and [D210 decision](../../decisions/language/D210_CLOSURE_WHILE_SELECTOR_AND_CONTEXTUAL_SYNTAX.md)  
**Exact published product revision:** [`9641eefcf11a014d41ae485d0c1884aed25df857`](https://github.com/guillermomolina/protos/commit/9641eefcf11a014d41ae485d0c1884aed25df857)  
**Commit:** `I117-A: remove standard Object.while and keep whileTrue as the single loop protocol`  
**Evidence scope:** read-only GitHub commit/file inspection and maintainer-reported local validation; no validation rerun by the documentation author.

## What was delivered

D210-Q1 is published as a separately reviewable implementation slice, ahead of D210-Q2.

- The standard `Object.while` slot/native body is removed. Only the original, identity-checked `Object.whileTrue` on the standard Object method home is admitted as canonical pre-test loop behavior. `while` stays a legal user-defined ordinary identifier or member name.
- The pre-test loop still validates the condition Closure, exactly one body argument, and the body Closure before its first predicate evaluation; it requires strict canonical Boolean results, tests before each iteration, preserves errors/transfers/suspension semantics, and returns canonical `null` for zero iterations or the exact last normal body result (including its identity and Future value).
- Removed the dual-selector `returnsLastBodyResult`/terminal-`false` result policy from the native implementation and prepared structured-control path. The structured and inline lowerers retain their generated control flow; they receive the single standard result behavior.
- Updated normative execution, callable, grammar and Future wording, the control-flow guide, Protos conformance and Java regression coverage, including root Object surface/reflection, selection/extraction, literal inline versus structured/fallback paths, zero/one/multiple bodies, result identity, Future ownership, suspended structured execution and nonlocal/error propagation. The published commit contains 19 paths.

Key files in that commit include `ProtosStandardObjectProtocol.java`, `ProtosCoreBootstrap.java`, `ProtosPreparedCalls.java`, `ProtosStructuredControlCalls.java`, `CanonicalToBytecodeLowerer.java`, `ProtosStructuredDispatchLowerer.java`, `spec/semantics/EXECUTION_AND_CONTROL.md`, `spec/semantics/CALLABLES.md`, `spec/PROTOS_GRAMMAR.md`, and `protos/tests/conformance/control/while-loop-results.protos`.

## Validation and provenance

Maintainer explicitly reported for the published I117-A delta:

- `git diff --check`: clean;
- local tests: **all PASS**;
- `I117-A`: **pushed**.

This is a maintainer report, **not** an independently reproduced test or CI result. The exact tested command list, counts and native-specific output were not provided for this publication, so no narrower claim about a particular suite's executed count is made.

## Publication metadata observation — outstanding

At the observed product HEAD `9641eefcf11a014d41ae485d0c1884aed25df857`, the commit's path list **does not include** `pom.xml`, `CHANGELOG.md`, or `spec/PROTOS_SPEC_CHANGELOG.md`. The current published files still read:

- Maven implementation version: `0.3.392-SNAPSHOT`;
- most recent implementation changelog heading: `0.3.392-SNAPSHOT`, for BUG024;
- most recent normative specification revision: `0.1.463`, dated 2026-10-19.

The publication therefore satisfies the code/test evidence reported for Q1, **but the required implementation and normative changelog metadata have not been observed for I117-A**. Do not attribute a new version/spec revision to this slice without a separately published reconciliation. This is a publication-record discrepancy, not a claim of failed runtime behavior.

## Next slice

**I117-B, same Issue #942, implementation in `guillermomolina/protos`.** Implement the D210-Q2 contextual expression `while(condition) { body }`, observable as `(() => condition).whileTrue() { body }`: two fresh lexical Closures, ordinary `whileTrue` selection, per-iteration reevaluation, exact returned body value/identity; precise bare-name/call/trailing-Closure boundary and approved precedence over a lexically defined `while` only for that exact source shape. Retain `while` as an identifier and member name and keep `(while)(x) { body }`, `obj.while(x) { body }`, `while(x)`, and newline-separated source on their ordinary parsing paths.

No grammar/parser implementation or tests of Q2 were part of I117-A. I117 remains open until I117-B is implemented, validated and published.
