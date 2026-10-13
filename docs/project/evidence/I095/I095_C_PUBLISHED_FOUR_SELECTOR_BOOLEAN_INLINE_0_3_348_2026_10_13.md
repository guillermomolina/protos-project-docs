# I095-C — Published shared Boolean control inline lowering (0.3.348)

**Evidence date:** 2026-10-13
**Formal implementation owner:** [I095 / #878](https://github.com/guillermomolina/protos/issues/878)
**Performance evidence owner:** [PERF041 / #863](https://github.com/guillermomolina/protos/issues/863)
**Architectural authority:** [PLAT057 / #881](https://github.com/guillermomolina/protos/issues/881), approved B (bounded proof-driven Canonical → Bytecode DSL inline lowering)
**Product repo:** `guillermomolina/protos`
**PROTOS_REVISION:** `ac7c5f65800fc558843e6060cc3bc5e7c356cfbb` — [published product commit](https://github.com/guillermomolina/protos/commit/ac7c5f65800fc558843e6060cc3bc5e7c356cfbb)
**PRODUCT_VERSION:** `0.3.348-SNAPSHOT`
**Product direct parent:** `74ba4f7156d0cad909e8140c4a5a6fee632c82f8` — [preceding TEST009-V publication](https://github.com/guillermomolina/protos/commit/74ba4f7156d0cad909e8140c4a5a6fee632c82f8)
**Historical I095 checkpoint:** `57a43b6d94697f9a6256e36c7a775dfa4a1218d7`, `0.3.343-SNAPSHOT` — [earlier I095 evidence](I095_PUBLISHED_IFTRUE_INLINE_CHECKPOINT_2026_10_13.md)
**Validation attribution:** Human executor reported all local tests PASS and `git diff --check` clean; independent GitHub reading verified product commit and metadata, not test execution.

## Published scope — exact product commit

The published commit [`ac7c5f65800fc558843e6060cc3bc5e7c356cfbb`](https://github.com/guillermomolina/protos/commit/ac7c5f65800fc558843e6060cc3bc5e7c356cfbb) has the maintainer-provided subject:

> I095: share canonical Boolean control inlining for ifTrue, ifFalse, if and else (0.3.348)

The commit changes **eight files**:

- `src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java` — `NormalBooleanControl` selection/result policy, `admitsNormalBooleanControl`, shared `emitNormalBooleanControl` / `emitNormalBooleanControlBranch` / `emitNormalInlineControlBody`.
- `src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java` — `IsCanonicalBooleanFalseClass` operation using the existing canonical Boolean classifier.
- `src/main/java/com/guillermomolina/protos/execution/ProtosCoreBootstrap.java` — frozen-root publication validates the standard selection identity and home of `ifTrue`, `ifFalse`, `if`, and `else`.
- `src/test/java/com/guillermomolina/protos/execution/ProtosI095BooleanControlInlineTest.java` — **new 12-test suite** for constant/dynamic receiver behavior, direct NORMAL inline regions, fallback, tags/source, binding/capture D179 invariants, overrides, exact-once evaluation, result policy, DEBUG result parity and exceptional/suspending paths.
- `src/test/java/com/guillermomolina/protos/execution/ProtosI092CompactBooleanTest.java` — adjust an I092 branch-profile test fixture to an unproven shape, preserving its independent native/fallback exercise.
- `tools/java_generated_bytecode_bci_pe_baseline.json` — two reviewed `PROVEN_HANDLER_RETURN` topology observations for `IsCanonicalBooleanFalseClass`.
- `pom.xml` — `0.3.348-SNAPSHOT`.
- `CHANGELOG.md` — implementation entry for I095-C.

The immediately preceding product commit, `74ba4f7156d0cad909e8140c4a5a6fee632c82f8`, is **TEST009-V** (`tools/test_truffle_root_diagnostic.py`, `tools/truffle_root_diagnostic.py`). It is separate from I095, and its changes must **not** be represented as part of the I095 product commit. The earlier I096 checkpoint at `ff57d22b6fa1abdc252200c3c114d015bb80a315` is not the immediate parent because TEST009-V intervened.

## Mechanics and semantic boundaries

One NORMAL-only frontend lowering handles the four one-callback standard Boolean sends:

| Selector | Selected on | Selected normal result | Unselected normal result |
| --- | --- | --- | --- |
| `ifTrue` | canonical true | exact callback value | canonical `null` |
| `ifFalse` | canonical false | exact callback value | canonical `null` |
| `if` | canonical true | original canonical true | original canonical false |
| `else` | canonical false | original canonical false | original canonical true |

The `if` / `else` selected body executes normally and its normal result is discarded, never awaited/adopted. Abrupt exits propagate. The receiver is evaluated once and classified; canonical selected cases with admitted immediate zero-argument Closure bodies are lowered directly into the calling Bytecode root. An unselected canonical callback body is not invoked or validated. A MISS uses the existing ordinary `PrepareSendOne` / prepared invocation with the saved receiver and the supplied literal materialized on that fallback path.

The admitted callback-body lexical proof reuses I095's own-local, captured-parent, return-home and persistent-authority checks. The direct body emits `RootTag` / `SourceSection` under the call `CallTag`; saved lowerer lexical fields are restored with `finally`. DEBUG, spread, unproven callbacks, nonliteral callback objects, and unsupported activation shapes retain their prior B′ / ordinary semantics. `ifTrueIfFalse`, `and`, and `or` are unchanged. D013 ordinary method selection, D179 presence/binding identity, D200 exact return policies and pay-as-you-grow still govern.

**Evidence limitation:** bytecode emission and Java tests prove a source-level/compiler-structural implementation plus validated conformance, not a post-Graal final-tier speedup, NodeCount reduction, compiled call elimination, lower CodeSize or allocation saving. B′ itself can eliminate a separate callback CallTarget; stack depth alone does not distinguish B′ from this new aggressive NORMAL path. The 12-test suite asserts NORMAL topology and NORMAL/DEBUG result parity, but does not independently assert every possible physical debugger stack shape; existing DEBUG regression tests and the reported integrated pass remain relevant.

## Validation provenance

In the current interaction the human executor explicitly reported, **after pushing the above product commit**:

- `git diff --check`: **clean** (human report).
- **All local tests**: **PASS** (human report).
- Product publication: **pushed** (human report, exact GitHub commit subsequently verified).

Earlier in the same implementation sequence, the human supplied a partial Maven focal-test transcript at `0.3.347-SNAPSHOT`, including these completed per-class summaries before the pasted log ended:

| Test class | Tests | Failures | Errors | Skipped |
| --- | ---: | ---: | ---: | ---: |
| `ProtosI092CompactBooleanTest` | 36 | 0 | 0 | 0 |
| `ProtosI095BooleanControlInlineTest` | 12 | 0 | 0 | 0 |
| `ProtosI096InlineClosureLiteralCallTest` | 29 | 0 | 0 | 0 |
| `ProtosPerf026B3BooleanInlineCallbackTest` | 10 | 0 | 0 | 0 |

The partial transcript did not show a final Maven `BUILD SUCCESS` or final summary for `ProtosPerf032ExecutionModeTest`; **do not claim that transcript independently validates the whole focal command**. The maintainer's later explicit all-tests-PASS statement is a distinct, broader self-report. Individual `make test`, `make check`, native-image, CI output and test totals were not supplied with the final publication message; do not manufacture those individual outcomes.

No agent ran builds, tests, native binaries, benchmarks or product Git state changes in this publication evidence recording. The documentation-only change is agent-published under the maintainer's explicit `guillermomolina/protos-project-docs` authorization.

## Issue ownership and remaining acceptance

[I095/#878](https://github.com/guillermomolina/protos/issues/878) must stay **OPEN / READY** until its own revision-matched causal performance/graph acceptance, or explicitly accepted independent PERF041 ownership and closure criteria, are reconciled. The **I095-C four-selector implementation/publication slice is complete and reported validated**. This is not blanket functional implementation acceptance for all future Boolean optimizations.

[PERF041/#863](https://github.com/guillermomolina/protos/issues/863) remains OPEN / READY as the `primitive-if-true` / `primitive-if-false` / `primitive-return-literal` comparison and current graph/CodeSize/steady causal measurement owner. Historic 5193/892/39 nodes come from an older revision (`0db24f00`) and are not metrics for `ac7c5f65800fc558843e6060cc3bc5e7c356cfbb`. Use revision-matched fixtures/harness and record exact producer revisions; do not infer benefit from source-level structure or rerun unmodified peer languages solely due to a new Protos revision.

[I096/#884](https://github.com/guillermomolina/protos/issues/884) remains separate and OPEN for its own generic ordinary-Closure final-tier graph acceptance. [I099/#887](https://github.com/guillermomolina/protos/issues/887) is a closed duplicate with **zero implementation**, and this work correctly publishes under I095.

```text
FORMAL_IDENTIFIER=I095
SLICE=I095-C
PROTOS_REVISION=ac7c5f65800fc558843e6060cc3bc5e7c356cfbb
PRODUCT_VERSION=0.3.348-SNAPSHOT
HUMAN_REPORTED_GIT_DIFF_CHECK=CLEAN
HUMAN_REPORTED_ALL_LOCAL_TESTS=PASS
NEW_FOCAL_SUITE=ProtosI095BooleanControlInlineTest
NEW_FOCAL_TEST_COUNT=12
PRODUCT_COMMIT_PUBLICATION=VERIFIED_ON_GITHUB
AGENT_EXECUTED_PRODUCT_TESTS=NO
AGENT_EXECUTED_PERFORMANCE_BENCHMARKS=NO
MATCHED_POST_0_3_348_COMPILER_GRAPHS=PENDING
I095_C_IMPLEMENTATION_PUBLICATION=COMPLETE
I095_CAUSAL_PERFORMANCE_ACCEPTANCE=PENDING
I095_ISSUE_LIFECYCLE=OPEN_READY
```
