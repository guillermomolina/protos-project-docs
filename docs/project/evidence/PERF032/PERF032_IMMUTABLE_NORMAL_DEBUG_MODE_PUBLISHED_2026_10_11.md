# PERF032 — Immutable NORMAL/DEBUG host-entry ABI: published implementation checkpoint

Date: 2026-10-11

## Scope, authority and exact revision

This record is durable **non-normative implementation evidence** for [PERF032/#831](https://github.com/guillermomolina/protos/issues/831). It is not a new language specification, a claim of timing speedup, or final closure of the broader primitive-return-literal investigation.

```text
WORK_ITEM=PERF032
ISSUE=guillermomolina/protos#831
WORKLOAD=primitive-return-literal
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_COMMIT=9b4a0750d759ead541cde5fe90d1d67ee24c68a9
PRODUCT_VERSION=0.3.336-SNAPSHOT
IMPLEMENTATION=IMMUTABLE_NORMAL_DEBUG_EXECUTION_MODE
HUMAN_GIT_DIFF_CHECK=CLEAN_REPORTED
HUMAN_LOCAL_TESTS=ALL_PASS_REPORTED
POST_COMMIT_SAME_WORKLOAD_TIMING_A_B=NOT_REPORTED
POST_COMMIT_TIER2_GRAPH_RECAPTURE=NOT_REPORTED
PERF032_ISSUE_CLOSURE=NO
```

The exact published implementation is [product commit `9b4a0750`](https://github.com/guillermomolina/protos/commit/9b4a0750d759ead541cde5fe90d1d67ee24c68a9). The maintainer reports that `git diff --check` is clean and **all local tests passed**, and that the source was pushed. The documentation author verified the published commit and inspected its source/test surfaces, but did **not** independently execute compilation, tests, DAP or a benchmark. No exact test command/output file was provided for the final published revision.

## Implemented boundary and source-grounded behavior

- `ProtosExecutionMode` defines `NORMAL` and `DEBUG`. `ProtosPolyglotRuntimeHost` retains the mode selected at creation and exposes `isDebugMode()`; `ProtosLanguageContext` fixes its mode at Context creation from the `protos.Debug` language option, defaulting to `NORMAL`.
- The normal CLI path opens no debugger DAP listener. `protos debug <file>` opens a DEBUG RuntimeHost using the existing DAP transport on `127.0.0.1:0`. Explicit standard embedding can select `protos.Debug=true` (an experimental option) without itself starting DAP.
- `ProtosHostExecutableClosure` resolves the eligible headerless-entry decision **once when the adapter is prepared**, from the immutable Context mode and the proven source-root capability. The normal eligible trivial return-literal call uses `Object[0]`; the DEBUG call retains the ordinary compact `[closure, caller]` provenance header.
- The ABI choice is not governed by `getTagTree()`, live breakpoints, or session attachment. There is no general per-invocation debugger-mode query on the optimized host entry. Rich activation still materializes under actual semantic/tooling demand; no extra `RootTask` is required by this change.
- In a headerless NORMAL frame, `ProtosBytecodeTagTreeNodeExports.getScope` throws an explicit mode-related `IllegalStateException` instead of silently returning an absent scope. A framework-owned Java `DebuggerSession` can still be created in-process; Protos does **not** intercept its creation, and its incompatible scope observation surfaces a `DebugException`. That is deliberately narrower than claiming that the Java debugger cannot attach at all.
- An immutable DEBUG execution carries provenance **before** the client connects: a Closure call prepared before the debugger attaches can subsequently be observed with the correct lexical binding and without creating a Task.
- The earlier temporary `protos.perf032.headerlessLiteralOne` switch is no longer the selector of this published mechanism.

The implementation commit includes production source, the `ProtosPerf032ExecutionModeTest` regressions, and the `pom.xml` / `CHANGELOG.md` version metadata. Its full changed-path set is authoritative over this summary.

## Published test coverage and human validation

`ProtosPerf032ExecutionModeTest` at the pinned commit defines checks for:

1. NORMAL eligible Closure uses an empty target-argument array.
2. DEBUG eligible Closure transports the exact closure/caller pair.
3. NORMAL RuntimeHost does not offer a DAP endpoint.
4. A real in-process Truffle debugger in DEBUG observes the exact captured lexical binding.
5. A DEBUG call prepared **before** debugger attachment still exposes that binding.
6. A NORMAL headerless scope request fails explicitly; the real debugger reports the failure rather than silently exposing `null`.
7. Separate concurrent NORMAL and DEBUG RuntimeHosts retain isolated mode behavior.
8. A standard Polyglot embedding selects DEBUG with the `protos.Debug` option; both modes execute the literal correctly.
9. No additional Task is created in the relevant tested paths.

The maintainer's report covers the local tests as passing. The above is **published test coverage plus maintainer-reported results**, not an independent reproduction by this record's author.

## Earlier local A/B: historical structural evidence only

Before the final immutable-mode publication, the maintainer reported a local Protos-only comparison of the *experimental* headerless ABI, before and after the tag-tree guard. The most recent pre-publication post-tag-gate A/B was:

| Diagnostic metric | Normal-header A | Experimental headerless B | B minus A |
| --- | ---: | ---: | ---: |
| Reported Tier-2 IR nodes | 39 | 31 | -8 |
| Reported Total IR | 50 | 43 | -7 |
| Reported native code bytes | 217 | 189 | -28 |
| BGV files | 1 | 1 | 0 |

The experimental debugger scope, activation identity, no-Task, and real attachment checks also passed when the call was prepared after instrumentation. A **late** attach to an already-headerless-prepared call failed to recover the scope; that failure motivated the approved immutable execution-mode distinction, **not** a change that made dynamically attaching a debugger to NORMAL supported.

**These measurements predate product commit `9b4a0750` and are not an A/B capture of its finalized implementation.** They show a structural optimization target and an earlier prototype result; they establish neither post-commit IR/code-size preservation nor lower `ns/call`. No claim of a benchmark speedup is made.

## Residual acceptance and coordination

- Keep PERF032/#831 **open**. The issue's existing `status:paused` records the separately approved pause of further graph-node research pending local-read work; this completed bounded implementation does not itself reverse that scheduling decision.
- Exact current-main same-workload timing A/B, profiling-funnel reconciliation and any needed post-commit Tier-2 graph capture remain subject to PERF032's original acceptance criteria. Do not equate local JUnit PASS or fewer graph nodes with resolved latency.
- The implementation does not add an invariant requiring zero performance warnings or reopen unrelated PERF workstreams.
- Prior [PERF032-G6 structural evidence](PERF032_G6_ZERO_PARAMETER_CLOSURE_GRAPH_CONVERGENCE_AND_VALIDATION.md) and the [2026-10-08 pause record](PERF032_NODE_INVESTIGATION_PAUSED_FOR_LOCAL_READ_2026_10_08.md) are retained as dated history, not overwritten.

## AI assistance and provenance

The project owner authorized the NORMAL/DEBUG distinction and the maintainer performed the code publication and validation. This evidence record was prepared with AI assistance after reviewing the public product commit, relevant regression code, and existing PERF032 durable records. It contains no agent-executed test, compiler-graph or performance result.
