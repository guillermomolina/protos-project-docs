# I116-B / 0.3.384 — Real DAP TCP optimized Bytecode DSL acceptance

**Record role:** immutable implementation and human-validation publication evidence, not a normative specification, design approval or live scheduling surface.
**Date:** 2026-10-19.
**Issue:** [I116/#935](https://github.com/guillermomolina/protos/issues/935), implementation slice I116-B.
**Product revision (exact):** [`41d45fbfc950f23b6ab56ae68fce08e854e5c0ea`](https://github.com/guillermomolina/protos/commit/41d45fbfc950f23b6ab56ae68fce08e854e5c0ea), published to `main` by the human maintainer, `0.3.384-SNAPSHOT`.
**Commit subject:** `I116-B: accept optimized Bytecode DSL roots over real DAP (0.3.384)`.
**Predecessor:** [I116-A product revision `12d9672df656bc72bb7a41a14843876bc01d20e5`](https://github.com/guillermomolina/protos/commit/12d9672df656bc72bb7a41a14843876bc01d20e5), `0.3.383-SNAPSHOT`; [historical I116-A evidence](I116_A_383_UNIFIED_BYTECODE_PUBLICATION_2026_10_19.md).

## Acceptance and implemented scope

I116-A provided one proof-optimized Bytecode DSL lowering, lazy passive source/tag instrumentation, and in-process Truffle debugger observations. The remaining I116-B boundary was **real Graal DAP over TCP** observing *optimized* source statements and frame-native locals without fabricating the eliminated guest execution.

The published test-only product commit creates [`ProtosI116OptimizedDapAcceptanceTest.java`](https://github.com/guillermomolina/protos/blob/41d45fbfc950f23b6ab56ae68fce08e854e5c0ea/src/test/java/com/guillermomolina/protos/execution/integration/interop/ProtosI116OptimizedDapAcceptanceTest.java) and exposes existing package-local DAP JSON/protocol helpers from [`ProtosI026FDapBehaviorTest.java`](https://github.com/guillermomolina/protos/blob/41d45fbfc950f23b6ab56ae68fce08e854e5c0ea/src/test/java/com/guillermomolina/protos/execution/integration/interop/ProtosI026FDapBehaviorTest.java). It changes exactly **four** product files: the two Java test classes, `pom.xml` and `CHANGELOG.md`. No production source, guest semantics, Bytecode DSL operation, DAP Engine option or normative specification changes are in this commit.

The two new named acceptance methods are:

1. `optimizedConstantReturnRemainsBreakpointVisibleOverRealDap`: real TCP breakpoint on surviving line 2 of `run: () => { 1 }`, true Protos source path/URI, exactly the real Closure and caller stack frames (caller line 4), ordinary scopes/variables response, successful continuation and final result `1`.
2. `optimizedScalarLocalsRemainVisibleOverRealDap`: real TCP breakpoints at source lines 2/3/4 of `first: 7; second: first; second`, exactly two real frames (caller line 6), local lifecycle `first=absent, second=absent` → `first=7, second=absent` → `first=7, second=7`, successful continuations and final result `7`.

The tests use the existing `ProtosDapTestSupport.DapClient` transport, physical temporary source files and a Core-bootstrapped activation. The DAP frame identifier is consumed from **the same** `stackTrace` response used for the matching scopes request, rather than compared across independent requests. The optimized tests share one helper, and the original DAP behavior test is preserved. Neither phantom receiver bindings nor a phantom callback frame are accepted.

## Validation provenance

**Human maintainer reported** the following on the candidate before the version/changelog edit:

- Initial B1 and B2 focal DAP tests PASS after test-fixture corrections (Core Prelude for polymorphic invocation; DAP frame-ID request locality).
- `ProtosI026FDapBehaviorTest`, `ProtosI026FDapTransportTest`, `ProtosI026EDebuggerIntegrationTest`, `ProtosLm009DPublicDebugCliTest`: focused integration PASS.
- `ProtosPerf025CompactCalleeExecutionTest`, independent I101-B/I101-C Builder-reference tests, and `ProtosI112TerminalCallBudgetTest`: focal regression PASS. The supplied I112 output reports `I112_TOTAL_FULL=16` (11 + 5), `I112_ROOT_COUNT=2` and `I112_CEILING_EXCEEDED=false`; `PLAT067_SCALAR_BODY_DEBUGGER_SCOPES=PASS`.
- After moving the two tests into their dedicated class, `I116_C_COMPILE_RC=0` and `I116_C_DAP_RC=0`. The old class had 637 lines, the new one 460 lines, as reported by the human executor.
- Final `make test`: maintainer reported **PASS** before changing the Maven version and changelog; `git diff --check` was clean. The user reported successful commit and push with `I116_COMMIT_RC=0` and `I116_PUSH_RC=0`.

The agent did **not** execute the tests. The report does **not** assert independent CI verification, `make check`, a separately successful Native Image test target, an end-user VS Code session, or a benchmark speedup. No tests were requested after metadata finalization.

**Read-only GitHub verification:** the exact product commit is present, with the four declared changed paths and the release metadata `0.3.384-SNAPSHOT`; the native GitHub commit identity is independent from the human execution attestations above.

## Disposition and boundaries

I116-A and I116-B together satisfy the bounded, ratified I116 acceptance; no new semantic/profile mode or further normative decision is introduced. [I116-B implementation handoff](../../work/I116/I116_B_AGENT_HUMAN_EXECUTOR_HANDOFF.md) is retained as history.

A future harness-hardening observation remains distinct from I116's semantic acceptance: the root Makefile's explicitly serialized DAP-test class list included the old DAP behavior test but did not include the newly added optimized acceptance class at this publication. The maintainer's full `make test` result is PASS; this record does not infer serial execution of the new class from that fact.

`I116_A=PUBLISHED_0.3.383`; `I116_B=PUBLISHED_0.3.384`; `PROTOS_REVISION=41d45fbfc950f23b6ab56ae68fce08e854e5c0ea`; `HUMAN_REPORTED_MAKE_TEST=PASS`; `SPEC_CHANGE=NONE`; `PROJECT_RECORD_REVISION=THIS_COMMIT`.
