# PLAT070 — Truffle 25.5 direct unwind: published implementation evidence (0.3.444)

- **Evidence date:** 2026-11-05
- **Live work owner:** [PLAT070 / guillermomolina/protos#1049](https://github.com/guillermomolina/protos/issues/1049)
- **Exact product revision:** [`6787089691803824d7596e02c98ab69c4d3c92e3`](https://github.com/guillermomolina/protos/commit/6787089691803824d7596e02c98ab69c4d3c92e3)
- **Parent product revision:** [`a7aa54bc0eceabcc2258ebf0c5b52c1b6d6ea71b`](https://github.com/guillermomolina/protos/commit/a7aa54bc0eceabcc2258ebf0c5b52c1b6d6ea71b)
- **Product implementation series:** `0.3.444-SNAPSHOT` (`pom.xml` / `CHANGELOG.md`); this is a source commit, **not** a tagged GitHub Release.
- **Runtime prerequisite:** [DIST017 / #968](https://github.com/guillermomolina/protos/issues/968) completed adoption of the released GraalVM / Truffle `25.5.5.0.3` line.
- **Evidence class:** independently verified GitHub commit/source contents plus **maintainer-reported** local validation; not an independent benchmark, full real-DAP closure or a formal design-ratification record.

## Exact published change

The product commit is titled `PLAT070: adopt Truffle 25.5 direct unwind for internal control transfers (0.3.444)`. GitHub's commit API reports **41 changed files**, **4,441 additions** and **554 deletions**, including the implementation, tests, generated-Bytecode topology baseline, `pom.xml` and `CHANGELOG.md`.

1. [`ProtosBytecodeRootNode.java`](https://github.com/guillermomolina/protos/blob/6787089691803824d7596e02c98ab69c4d3c92e3/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java) sets `@GenerateBytecode(unwindExceptions = {ProtosNonLocalReturnException.class, ProtosTaskCancellationException.class})`. Guest `ProtosSignalException` remains outside direct unwind on the ordinary guest Error path.
2. The backend-private `ProtosBytecodeControlTransferException` bridge and its root interceptor and unwrapping catches are deleted. Prepared-call, Task, terminal continuation, host-entry and I/O boundaries operate on the original control-transfer object.
3. [`ProtosStructuredDispatchLowerer.java`](https://github.com/guillermomolina/protos/blob/6787089691803824d7596e02c98ab69c4d3c92e3/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/ProtosStructuredDispatchLowerer.java) uses nested `TryFinally` and a completion marker for `ensure` cancellation supersession while retaining suspend/resume behavior.
4. [`ProtosReturnHome.java`](https://github.com/guillermomolina/protos/blob/6787089691803824d7596e02c98ab69c4d3c92e3/src/main/java/com/guillermomolina/protos/runtime/control/ProtosReturnHome.java), `ProtosActivation` and `ProtosFrameArguments` register the owning Task only for invocation-owned materialized homes, including compact B5 calls. Completion clears the Task reference; captured homes are not rebound. `RaiseNonLocalReturn` checks ownership at the `^` instruction: valid transfers do not capture a live Truffle stack; invalid cross-Task or completed-home returns raise ordinary guest `InvalidReturn` at the source site.
5. [`ProtosIoReleaseCPrimeExecution.java`](https://github.com/guillermomolina/protos/blob/6787089691803824d7596e02c98ab69c4d3c92e3/src/main/java/com/guillermomolina/protos/execution/host/resources/ProtosIoReleaseCPrimeExecution.java) continues mandatory finalization after abrupt transfers, including multiple finalizers and suspension/resumption, before release terminalization. Four other operation-owned C-prime roots use direct-transfer accounting through Bytecode `TryFinally` and existing protocol failure paths.
6. The commit updates `tools/java_generated_bytecode_bci_pe_baseline.json` for the reviewed new handler topology and adds/revises tests for B4A–B4G, ReturnHome and cross-Task diagnostics, I/O finalizers and all four I/O operation roots, continuation/CLI and real-DAP observation.

The implementation deliberately changes internal exception transport, **not** the normative language specifications.

## Validation and provenance

**Maintainer report after publication:**

- `git diff --check`: clean.
- Affected focal tests: PASS, including repeated direct-unwind, ReturnHome, CLI008, I/O and DAP scenarios.
- `make test`: PASS locally.
- `make check`: PASS locally.
- Git push: completed; the exact commit above was independently observed on the repository's `main` branch.

The maintainer's success report is accepted as **reported local execution**, not misrepresented as a rerun by this documentation publisher. Complete local command output, CI logs, portable/Natively packaged debugger runs and per-test counts were not supplied for this checkpoint. The new generated-bytecode baseline is part of the **published commit**; `make check` PASS is maintainer-reported.

## Limits and independently owned follow-ups

- **Performance:** source-level removal of the valid-return `TruffleStackTrace` walk is verified. A measured pre/post latency, allocation, compiler graph and I115 hot-path comparison was **not reported**. No performance gain is asserted.
- **[BUG025 / #940](https://github.com/guillermomolina/protos/issues/940):** the real-DAP test `realDapObservedNoUncaughtStopForAValidNonLocalReturn` asserts no `uncaught` stops for the tested valid `Array.findIndex` return. Another test prints, but does not assert, the actual stop classification for an `InvalidReturn` produced in one Task and handled by a consumer. This is not full closure of BUG025's original full-program / Native/JVM / filter-matrix acceptance.
- **[PLAT069 / #1048](https://github.com/guillermomolina/protos/issues/1048):** the broader 25.5 capability inventory remains **owner-paused** until I115 is completed; adopting `unwindExceptions` does not resume or complete PLAT069.
- **Design governance:** this document records the **published implementation** and its validation. It does not invent the distinct GITHUB015/GITHUB021 exact-candidate owner-approval provenance or declare a PLAT070 decision formally ratified. The live Issue remains responsible for that gate and any closure.
- **Release:** `0.3.444-SNAPSHOT` source was pushed; no tag, asset publication, Native Image release or downstream distribution is claimed.

## Record boundaries

Normative authority remains in `guillermomolina/protos/spec/**`. Operational status, decision approval and closure belong to the linked GitHub Issues. This record is a non-normative, revision-bound publication and validation snapshot. Historical UPSTREAM001 experimental results are distinct from the current 25.5 production source publication.
