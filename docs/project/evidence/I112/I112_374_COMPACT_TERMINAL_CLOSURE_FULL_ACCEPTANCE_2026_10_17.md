# I112 — Compact terminal Closure invocation: published acceptance (0.3.374-SNAPSHOT)

**Date:** 2026-10-17  
**Implementation owner:** [I112/#920](https://github.com/guillermomolina/protos/issues/920)  
**Approved design:** [PLAT062-T/#907](https://github.com/guillermomolina/protos/issues/907), [canonical platform decision](../../decisions/platform/PLAT062_SPECIALIZED_CLOSURE_INVOCATION_OPERATION_BOUNDARY.md)  
**Exact product revision:** [`d63430375c06ac159bae23b918c633c7cd6cd6c6`](https://github.com/guillermomolina/protos/commit/d63430375c06ac159bae23b918c633c7cd6cd6c6)  
**Product commit subject:** `I112: implement compact terminal Closure invocation (0.3.374)`  
**Product version:** `0.3.374-SNAPSHOT`; this is a source implementation checkpoint, **not** a tagged binary release.

## Verified publication and validation provenance

GitHub `main` was independently read on 2026-10-17 and resolves to the exact product commit above. The published change includes the source lowerer, semantic and structured Bytecode interpreters, terminal continuation and runtime bridges, CLI008 diagnostic projection, I112 tests and affected regression tests, the TEST009-D generated BCI baseline, `pom.xml`, and `CHANGELOG.md`. The changelog records the I112 implementation and human-reported `make test` / `make check` PASS.

The maintainer **reports**: the product commit was pushed; `git diff --check` is clean; all local tests pass. Earlier in the same acceptance session the focal test set was reported **112 tests, 0 failures**, and the physical census **`I112_TOTAL_FULL=16`**. These are **human-executed results**, not independently executed or log-replayed by the coordinating agent. The independently verified evidence is the published Git revision and the assertions/implementations available in that revision.

The acceptance implementation is `src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI112TerminalCallBudgetTest.java`, which enumerates `getBytecodeNode().getInstructionsAsList()` unfiltered for the identity case; checks two source roots, no extra group roots, target **<=16** (with a separately declared 17-instruction design ceiling), and checks the compact operation topology. It reports independent NORMAL and DEBUG censuses; DEBUG is **not** improperly compared with the NORMAL physical budget. Related semantic regressions are in `ProtosI112TailContinuationTest.java` and affected CLI008, PERF006, I101, I107, I108 and PERF042 tests.

## Physical acceptance: 16 FULL across two source roots

Program (NORMAL, default Bytecode DSL configuration):

```protos
identity: (value) => { value }
identity(1)
```

| Source root | Full Bytecode DSL instructions | Result |
|---|---:|---|
| Module | **11** | `EnterRoot`; 2 constant loads; `MaterializeCurrentClosure`; `CreateCurrentFrameLocal`; discarded-declaration `pop`; `ReadRootFrameLocal`; literal load; `InvokeClosureOneTail`; `EpilogForReturn`; `return` |
| `identity` | **5** | `EnterRoot`; `BindUnaryFrameParameter`; `ReadRootFrameLocal`; `EpilogForReturn`; `return` |
| **TOTAL FULL (unfiltered)** | **16** | Two source roots; no new mandatory helper root for generic fallback |

The complete instruction list is reproducible from the published acceptance test; it was not pasted into this record as an independently captured run log. The earlier 15-instruction architectural ledger was **provisional**; the real emitted module includes the declaration-result `pop`. Historical Protos 420/434/~460 figures were taken in other worktree/revision circumstances and are **not** asserted to be an exact controlled before/after benchmark. The published GraalPy reference had 12 module + 5 identity instructions; language/runtime semantics differ.

The output is a physical pre-Graal/Bytecode DSL result only: **no measured speedup, final-tier graph improvement, latency parity, allocation benchmark or Native Image benefit is claimed**. [PERF042/#871](https://github.com/guillermomolina/protos/issues/871) retains independent performance ownership.

## Implementation and semantic preservation

- `CanonicalToBytecodeLowerer` emits a true terminal `InvokeClosureZeroTail`, `InvokeClosureOneTail` or `InvokeClosureArgumentsTail` and retains direct operands without mechanically embedding the fallback/`Yield` tree in the caller. The terminal operation owns D013 late-selected `Object.call`, guarded/PIC and generic dispatch, B5 ABI construction, and prepared-call completion. Nonterminal suspension retains its existing resumable structure.
- Fresh Closure creation in the module uses `MaterializeCurrentClosure` and `CreateCurrentFrameLocal`. The admitted module retains **persistent lexical authority**, installed by `EnterRoot` rather than an extra body instruction; `ReadRootFrameLocal` preserves D179 presence/continuity. Ordinary roots outside the narrow admission use the previous installation path.
- `ProtosTailCallContinuation` is a backend-private **only-on-real-suspension** transport for live callee continuation/suspension plus deferred completion ownership. No extra root or second ABI is created. Task, C-prime, host-entry and I/O drivers resume the retained child without replay and finish/unwind exactly once, innermost first; the synchronous path does not allocate a transport.
- A terminated terminal caller is not fabricated into a physical Truffle frame. Instead its source-owned `BytecodeLocation` is retained on actual suspension and its **logical guest diagnostic frame** is projected when a resumed Error crosses the transport. `ProtosDiagnosticTraceCapture` preserves innermost-first CLI008 diagnostic ordering, bounding, isolation and source projection, while `ProtosPerf006B5BSourceInstrumentationLocationTest` asserts caller/callee logical locations and source stability across reparse. **Limitation:** physical Truffle frame iteration/debugger frame enumeration cannot see a caller root that already returned its transport.
- Regression coverage includes post-argument override, generic/direct transitions, PIC invalidation, native/structured/Boolean suspension, continuation chains, cancellation and cleanup, ReturnHome/NLR, cross-Context isolation, D179 remove/recreate, and diagnostic location/source reporting. The source assertions and human-reported local suite completion support implementation acceptance; this record makes no new guest-language semantic decision.

## Publication reconciliation

```text
WORK_ITEM=I112/#920
DECISION=PLAT062-T/#907 (owner approved 2026-10-17)
PROTOS_REVISION=d63430375c06ac159bae23b918c633c7cd6cd6c6
PRODUCT_VERSION=0.3.374-SNAPSHOT
NORMAL_SOURCE_ROOTS=2
NORMAL_MODULE_FULL=11
NORMAL_IDENTITY_FULL=5
I112_TOTAL_FULL=16 (maintainer-reported test output)
TARGET_FULL_LE_16=PASS (maintainer-reported)
HUMAN_LOCAL_TESTS=PASS (maintainer-reported)
HUMAN_GIT_DIFF_CHECK=CLEAN (maintainer-reported)
REPOSITORY_COMMIT_PUBLICATION=VERIFIED
PERFORMANCE_OR_NATIVE_CLAIM=NONE
```

**Scope boundary:** This evidence closes the I112 compact-terminal implementation and PLAT062-T structural acceptance; it does not close PERF042, replace I111/PLAT065, certify debugger physical stack preservation after terminal delegation, or assert release/tag publication.
