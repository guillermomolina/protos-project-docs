# I095-D — Published eight-selector Boolean-control inline lowering (0.3.350)

**Evidence date:** 2026-10-13
**Formal owner:** [I095 / #878](https://github.com/guillermomolina/protos/issues/878)
**Causal control-graph measurement owner:** [PERF041 / #863](https://github.com/guillermomolina/protos/issues/863)
**Architectural decision:** [PLAT057 / #881](https://github.com/guillermomolina/protos/issues/881), owner-ratified proof-driven lexical inlining (Candidate B)
**Product repository:** `guillermomolina/protos`
**PROTOS_REVISION:** `045768b9d352bcc18a6132fccf84e46973abb51f` — [published commit](https://github.com/guillermomolina/protos/commit/045768b9d352bcc18a6132fccf84e46973abb51f)
**PRODUCT_VERSION:** `0.3.350-SNAPSHOT`, independently re-read at the exact product commit in `pom.xml` and `CHANGELOG.md`
**Direct predecessor:** `ffd6da2fb265f4d999ee26ddbfc19369f0f0403a`, separately published I096 C2–C8 / nested-Closure implementation `0.3.349-SNAPSHOT`; do not attribute that work to I095-D
**Human validation:** maintainer reports all local tests PASS and `git diff --check` clean, after pushing product; no agent-executed build/test/performance measurement.

## Exact published product scope

The independently verified product commit has subject:

> I095: complete canonical Boolean control inlining for ifTrueIfFalse, and, or and not (0.3.350)

Its eight changed files are:

1. `src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java` — single `NormalBooleanControl` declarative eight-selector table with arity, per-canonical-receiver selected callback, immediate-result and selected-result policy; site admission and per-branch NORMAL inline, once-staged ordinary fallback, preserving the prior I095-C routes.
2. `src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java` — `RequireCanonicalBooleanControlResult` operation requiring exact canonical Boolean callback results for `and`/`or`, resolving caller authority on the invalid-result path.
3. `src/main/java/com/guillermomolina/protos/execution/ProtosStandardBooleanProtocol.java` — standard `not` selection identity helper; `not` remains callback-free.
4. `src/main/java/com/guillermomolina/protos/execution/ProtosCoreBootstrap.java` — frozen Object root publication additionally validates exact standard `ifTrueIfFalse`, `and`, `or` and `not` selection.
5. `src/test/java/com/guillermomolina/protos/execution/ProtosI095BooleanProtocolInlineTest.java` — new **14-test JUnit suite**, independently counted from `@Test` annotations on the exact published 896-line source. The earlier pre-publication report of 13 tests is superseded by the published source count.
6. `tools/java_generated_bytecode_bci_pe_baseline.json` — updated BCI/PE topology baseline for generated Boolean-result operation and cached dispatch numbering, per the published changelog. Reported gate success is from the maintainer; no independent gate transcript is supplied.
7. `pom.xml` — `0.3.350-SNAPSHOT`.
8. `CHANGELOG.md` — I095-D change entry and accepted structural description.

The previous four-selector I095-C publication remains at `ac7c5f65800fc558843e6060cc3bc5e7c356cfbb` (`0.3.348-SNAPSHOT`), with [separate historical I095-C record](I095_C_PUBLISHED_FOUR_SELECTOR_BOOLEAN_INLINE_0_3_348_2026_10_13.md). Between I095-C and I095-D, I096 published its independent `0.3.349` implementation and a large-Integer inline regression fix.

## Implementation and semantic boundaries

The published table covers all eight standard ordinary Boolean selectors: `not`, `ifTrue`, `ifFalse`, `ifTrueIfFalse`, `and`, `or`, `if`, and `else`.

- `ifTrueIfFalse` independently proves two literal zero-argument callback bodies. Admitted TRUE/FALSE bodies are incorporated into the physical caller root; one admitted and one ordinary branch, two admitted branches, or the existing ordinary/B′ path are supported with exact branch-specific fallback.
- `and` and `or`, and transitively the existing `&&`/`||` Closure-producing desugaring, execute selected proven callback bodies in the caller root. Strict canonical `true`/`false` result validation remains after callback completion; invalid `null`, object or Future results signal standard Error. A short-circuited callback is not invoked.
- `not` and existing `!` desugaring perform canonical Boolean negation without the standard native activation on the admitted path; MISS takes ordinary dispatch.
- Literal canonical receivers selecting **no** callback admit nonliteral supplied arguments: argument expressions still evaluate, with callability validation and invocation skipped exactly as the standard protocol requires.
- The published lowerer stages receiver and nonliteral operands once ahead of classification, preserves ordinary fallback/DEBUG paths and avoids repeated nested NORMAL/DEBUG bytecode growth for eligible composed operators; literal Closure values are materialized on ordinary paths when needed. A selected callback without an adequate I095-B lexical/ReturnHome proof retains B′/ordinary treatment.
- D013 ordinary message semantics, standard root selector/home identity, D179 lexical binding and context identity, D200 `if`/`else` receiver-return policy, precise selected/unselected callability, Error/NLR/suspension, and pay-as-you-grow remain the contract. No guest syntax or normative specification changed.

The newly published test suite exercises standard selector rows, both-branch and mixed-branch topology, once-only eager argument effects, unselected dynamic arguments, strict `and`/`or` validation, short-circuit operators, canonical `not` and generic MISS, custom overrides, own/captured binding isolation, abrupt exits/ensure/suspension, NORMAL/DEBUG parity, long `&&` chains and standard `ifNull`/`ifNotNull` conformance. Structural bytecode assertions are source/bytecode-level evidence, not final-tier compiled graph proof.

## Publication and validation provenance

The owner supplied, after publication:

```text
I095: complete canonical Boolean control inlining for ifTrueIfFalse, and, or and not (0.3.350) pushed
git diff --check: clean
All local tests: PASS
```

The agent independently re-read the exact GitHub product commit, the matching POM/changelog entry, its changed-file set and the 14-test published source; the agent did **not** execute `make test`, `make check`, JUnit, Native Image, CI, benchmark or any product state-changing Git operation. The owner's general tests-PASS report is not separately evidenced as a particular command transcript or test total, and the published BCI baseline is not independently verified by agent execution.

The documentation repository content publication is authorized explicitly by the maintainer in this interaction. This record is a new non-normative snapshot: it neither changes product semantics nor retroactively rewrites I095-C's earlier checkpoint.

## Benchmark handoff — evidence not yet available

At coordination time the existing benchmark repository is at `guillermomolina/protos-benchmarks@1b28b390b61d5a59e9b1573ea30e26306a6dde86`. Its registered workloads include `primitive-return-literal`, `primitive-if-true`, `primitive-if-false`, and PERF039 `primitive-while-zero/once/counted`, but **no measured I095-D result was supplied**.

- [PERF041 / #863](https://github.com/guillermomolina/protos/issues/863) owns exact-revision post-publication `primitive-if-true` / `primitive-if-false` / `primitive-return-literal` compiled graph and CodeSize evidence; it must not use historical older-revision 5193/892/39 node counts as current figures.
- The current primitive-if workloads use `ifTrue`, so they mainly check retained I095-C performance/topology and regressions on the I095-D revision; **they do not directly isolate the new `ifTrueIfFalse`/`and`/`or`/`not` fast paths**. That is an explicit coverage limit for any claimed eight-selector performance success.
- Capture final-tier After-TruffleTier graph metrics, exact host/JDK/harness/product revision, applicable CodeSize/compilation stability and comparable steady timings only after workload correctness. Reuse established benchmark policy; no silent sample/warmup changes, no unnecessary JS/Python peer reruns.
- Do not infer speedup or compiled CallTarget disappearance from source-level bytecode topology or local conformance tests. Failed correctness, unstable compilation or NOT_STABLE timing cannot be described as an accepted performance win.

**Lifecycle:** I095 functional implementation/publication checkpoint is **COMPLETE** and human-reported validated; **I095 remains OPEN / READY**, not closed before its causal acceptance is satisfied or explicitly reallocated. PERF041 remains **OPEN / READY** pending evidence; I096's generic callable proof remains separate. The closed duplicate I099 must not be reopened.

```text
FORMAL_IDENTIFIER=I095
SLICE=I095-D
PRODUCT_VERSION=0.3.350-SNAPSHOT
PROTOS_REVISION=045768b9d352bcc18a6132fccf84e46973abb51f
PRODUCT_FILES_CHANGED=8
NEW_JUNIT_SUITE=ProtosI095BooleanProtocolInlineTest
NEW_JUNIT_TESTS_SOURCE_COUNT=14
PRODUCT_COMMIT_PUBLICATION=VERIFIED_ON_GITHUB
HUMAN_REPORTED_LOCAL_TESTS=PASS
HUMAN_REPORTED_GIT_DIFF_CHECK=CLEAN
AGENT_EXECUTED_TESTS=NO
AGENT_EXECUTED_BENCHMARKS=NO
ALL_EIGHT_SELECTOR_INLINE_CHECKPOINT=PUBLISHED
CAUSAL_COMPILED_GRAPH_CODE_SIZE_TIMING=PENDING
I095_ISSUE_LIFECYCLE=OPEN_READY
PERF041_ISSUE_LIFECYCLE=OPEN_READY
```
