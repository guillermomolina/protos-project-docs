# PLAT064-B — Root lifecycle versus Protos invocation and unwind: KEEP

**Date:** 2026-10-17
**Formal owner:** [PLAT064-B / #914](https://github.com/guillermomolina/protos/issues/914); native child of [PLAT064 / #911](https://github.com/guillermomolina/protos/issues/911)
**Work type:** investigation; source-only
**Product revision:** [`guillermomolina/protos@e07340c2ea6ac26278a04495e145a08e3bb2cd4d`](https://github.com/guillermomolina/protos/tree/e07340c2ea6ac26278a04495e145a08e3bb2cd4d), `0.3.371-SNAPSHOT`
**Declared Truffle:** `25.4.4.1.1`; upstream 25.4 source snapshot [`oracle/graal@95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b`](https://github.com/oracle/graal/tree/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b)
**Verdict:** `KEEP / NO_ACTION` — research classification; not approval to implement and not a new ratified architecture.

## 1. Principal conclusion

No source-backed current defect requires adopting `@Prolog`, `@EpilogReturn` or `@EpilogExceptional`. Standard lifecycle hooks work at **physical generated-root** boundaries. Protos semantic activation, captured return homes, prepared calls, Task-local dynamic handlers and nested protected `ensure` scopes have different boundaries.

- A source root can enter with compact frame arguments and no rich `ProtosActivation`. `CurrentActivation` first materializes/publishes one precise activation **on demand**. Eager prolog creation would add unconditional work.
- `ReturnHomeOwningCall` completes a home at its **prepared invocation** boundary, including nested and non-local return paths. Normal return epilog observes its root's Return operation, not every prepared call completion.
- The generator's `interceptControlFlowException` **return-a-value** route stores the root result and returns directly, without executing the ordinary Return instruction. Therefore `@EpilogReturn` cannot be presumed to finish a home on that route. Protos currently bridges the CFE instead of returning from its interceptor.
- Handler selection via `interceptTruffleException` occurs **before** Bytecode DSL guest exception handlers and `TryFinally` regions. A selected handler must already be inactive when a crossed `ensure` cleanup signals a replacement Error. Exceptional epilog is too late and only covers escaping `AbstractTruffleException`.
- `Yield` and resumption do not constitute semantic scope exit. Nested `TryFinally` and Task cancellation-phase state own cleanup and shielding, including cleanup that itself suspends.

**Disposition:** retain both approved generated roots, explicit prepared-call and cleanup operations, both existing exception interceptors, CFE bridge and `Yield`. No Ixxx follows from this investigation.

## 2. Authority and source coverage ledger

Read root `AGENTS.md`, `AGENTS.work/DESIGN.md`, `REFERENCE.md`, `COORDINATION.md`, `src/AGENTS.md`, `spec/AGENTS.md`. Normative texts inspected: `spec/semantics/CALLABLES.md` (activation/default binding and non-local return), `EXECUTION_AND_CONTROL.md` (ensure/cleanup and suspension), `ERRORS.md` (one-shot handler selected before cleanup), `OBJECT_MODEL.md` (execution-context identity), relevant `spec/concurrency/FUTURES_AND_TASKS.md` and `ACTORS.md` (Task/Actor cancellation and isolation), and `spec/semantics/MODULES.md`. Source-only prior decisions preserved: PLAT036/#702, D179/#703, I071/#717, PLAT041/#759, PLAT042/#760, PLAT043/#763, PLAT045/#772, PLAT062/#907, PLAT063/#908. I110/#912 is independently authorized. [PLAT064-A KEEP](https://github.com/guillermomolina/protos-project-docs/blob/b4346d4071bcc7e9fa5b90941c44e45ea23c7fcc/docs/project/evidence/PLAT064/PLAT064_A_BCI_SCOPE_MATERIALIZED_LOCAL_KEEP_2026_10_17.md) and [parent triage](https://github.com/guillermomolina/protos-project-docs/blob/a63471424713682ef2eb18736781247de2dc43a8/docs/project/evidence/PLAT064/PLAT064_CROSS_IMPLEMENTATION_INFRASTRUCTURE_TRIAGE_2026_10_17.md) are prior evidence, not issues reopened here. UPSTREAM004/Native Tier-2 are unrelated acceptance criteria.

### Protos primary source

- **P1:** [`ProtosSemanticBytecodeRootNode.java`](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosSemanticBytecodeRootNode.java)
- **P2:** [`ProtosBytecodeRootNode.java`](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java)
- **P3:** [`CanonicalToBytecodeLowerer.java`](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java)
- **P4:** [`ProtosStructuredDispatchLowerer.java`](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/ProtosStructuredDispatchLowerer.java)
- **P5:** [`ProtosFrameArguments.java`](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosFrameArguments.java)
- **P6:** [`ProtosActivation.java`](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java)
- **P7:** [`ProtosReturnHome.java`](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/main/java/com/guillermomolina/protos/runtime/control/ProtosReturnHome.java)
- **P8:** [`ProtosSignalException.java`](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/main/java/com/guillermomolina/protos/runtime/ProtosSignalException.java)
- **P9:** [`ProtosCoreErrors.java`](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/main/java/com/guillermomolina/protos/runtime/ProtosCoreErrors.java)
- **P10:** [`ProtosDynamicControlState.java`](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/main/java/com/guillermomolina/protos/runtime/ProtosDynamicControlState.java)
- **P11:** [`ProtosBytecodeControlTransferException.java`](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeControlTransferException.java)
- **P12:** [`ProtosBytecodeTaskExecution.java`](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeTaskExecution.java)
- **P13:** [`ProtosTaskCPrimeEntryExecution.java`](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosTaskCPrimeEntryExecution.java)

Key method landmarks: P1 `CurrentActivation`, `interceptControlFlowException`, `interceptTruffleException`; P2 `ReturnHomeOwningCall`, `interceptGuestException`, `EnterClosureCall`, `FinishClosureCall`, `ResumeContinuation`; P3 `emitRootBody`, `emitCurrentActivation`, parameter/default binding, `emitPreparedInvocation`, `emitContinuationComposition`; P4 `emitPreparedInvocation` and nested `beginTryFinally`/`TryCatch` for `ensure` and `Error.handle`; P5 `materializeCompactActivation`; P9 `selectHandlerIfNeeded`; P10 `selectMatchingHandler`; P12 Task C-prime resume and cancellation terminalization; P13 `createPlan`.

The source trace covered these lifecycle methods and their immediate ownership boundaries, **not** every unrelated operation in multi-thousand-line roots and not every I/O-specific release branch. No contradiction was found in the inspected paths; this does not claim a universal runtime correctness proof.

### Exact upstream sources

- [`Prolog.java`](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/Prolog.java); [`EpilogReturn.java`](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/EpilogReturn.java); [`EpilogExceptional.java`](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/EpilogExceptional.java); `BytecodeRootNode.java`, `GenerateBytecode.java`, `Operation.java`, [UserGuide](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/docs/bytecode_dsl/UserGuide.md).
- [`BytecodeNodeElement.java`](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.dsl.processor/src/com/oracle/truffle/dsl/processor/bytecode/generator/BytecodeNodeElement.java) (`createHandleException`, `createHandleControlFlowException`).
- [`BytecodeInstructionHandler.java`](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.dsl.processor/src/com/oracle/truffle/dsl/processor/bytecode/generator/BytecodeInstructionHandler.java) (YIELD/RETURN instructions).
- [`BytecodeRootNodeElement.java`](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.dsl.processor/src/com/oracle/truffle/dsl/processor/bytecode/generator/BytecodeRootNodeElement.java) (`execute`, `continueAt`).
- [`ContinuationRootNodeImplElement.java`](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.dsl.processor/src/com/oracle/truffle/dsl/processor/bytecode/generator/ContinuationRootNodeImplElement.java) (saved-BCI resume).
- [`PrologEpilogTest.java`](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.api.bytecode.test/src/com/oracle/truffle/api/bytecode/test/PrologEpilogTest.java) (hook throwing, handler extent, tags, source, return).

**Version qualification:** exact byte-for-byte correspondence of upstream snapshot `95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b` with released `25.4.4.1.1` processor was not proven. Documented public contract is distinguished from generator-source observations; no exact-artifact tests were executed.

## 3. Framework lifecycle truth table

| Event | Prolog | EpilogReturn | EpilogExceptional | Interceptors / handler handling |
|---|---|---|---|---|
| Initial root entry | Before body, after RootTag enter and before RootBodyTag enter when enabled | — | — | Root-entry instrumentation applies |
| Explicit or implicit ordinary Return | Already executed | Yes; receives/replaces returned value | — | Normal root Return opcode |
| Body ATE handled inside root | Already executed | Not at catch | No for handled occurrence | `interceptTruffleException` before DSL guest handler |
| Uncaught ATE escapes | Already executed | No | Yes | Exceptional epilog is special EH exit handler |
| Raw CFE propagates unchanged | Already executed | No | No | `interceptControlFlowException` may rethrow |
| CFE interceptor **returns** a value | Already executed | **Not through Return opcode** | No | Generator encodes terminal state from returned object |
| CFE interceptor **throws converted ATE** | Already executed | No | If converted ATE eventually escapes | Guest EH/`TryFinally` now applies |
| Internal host exception | Already executed if thrown later | No | Not generic host finally | `interceptInternalException` / propagation |
| Prolog throws guest ATE | Throws | No | Sees escaping guest failure | Truffle interceptors apply |
| Return epilog throws guest ATE | Already executed | Throws | Sees escaping guest failure | Not caught by TryCatch/TryFinally protecting Return |
| Exceptional epilog itself throws | Already executed | — | Not recursively reinvoked | New guest/internal failure propagates |
| DSL Yield | Already executed | **No** normal return epilog | No ordinary exceptional epilog | Returns `ContinuationResult` with frame/BCI |
| Saved continuation resume | **Not rerun from initial prolog** | On later actual Return only | On escaping later guest exception only | Continuation calls source `continueAt` at saved BCI |
| Deopt/tier transition | No semantic new invocation | Only on actual Return | Only on qualifying failure | Generator reconciles bytecode/frame state |

Upstream `PrologEpilogTest` includes `testThrowInternalErrorInProlog`, `testThrowInternalErrorInReturnEpilog`, `testThrowInternalErrorInExceptionalEpilog`, guest counterparts, `testReturnEpilogExceptionNotHandledByTryFinally`, `testReturnEpilogExceptionObservedByRootTag`, `testReturnEpilogExceptionNotObservedByOtherTags` and source/local scope tests. The framework contracts distinguish RootTag and RootBodyTag events, and different exception categories. Hook executions are not a semantic once-per-Task guarantee.

Generator observations are source-proven at the comparison revision, with exact binary caveat above. In particular, `createHandleControlFlowException` does not jump into ordinary Return after receiving a root result; `ContinuationRootNodeImplElement.createExecute` restores the frame and reenters `continueAt` with the saved BCI. A combined runtime hook/Yield test was not executed.

## 4. Cross-implementation comparison (real mechanisms, not a vote)

| Peer and pinned source | Actual behavior | Applicability |
|---|---|---|
| [GraalPy root](https://github.com/oracle/graalpython/blob/7e7182b1036d56f8e89ecc9c7a86cee292180aa6/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java) | `@Prolog EnterCalleeContext` invokes `calleeContext.enter`; `@EpilogReturn` traces/profiles and exits context; `@EpilogExceptional` adds traceback, traces and exits context. `interceptTruffleException` independently records catch location/tracing/exception chaining; `interceptInternalException` conditionally maps host errors. | Hook and interceptors **coexist** because they own distinct stages. Python C-call context is not a Protos Task/home/ensure equivalent. |
| [TrufflePascal root](https://github.com/d45h71/TrufflePascal/blob/fcae9007500369dd194c7e09c932fd4807d64fa8/language/src/main/java/org/trufflepascal/bytecode/PascalBytecodeRoot.java), [activation](https://github.com/d45h71/TrufflePascal/blob/fcae9007500369dd194c7e09c932fd4807d64fa8/language/src/main/java/org/trufflepascal/bytecode/BytecodeActivation.java) | `@Prolog` acquires caller-provided activation; `@EpilogReturn`/`@EpilogExceptional` close it. Caller operations also close child activation in Java `finally`; `BytecodeActivation.close` checks a `closed` flag and is idempotent. | Valid root-resource lifetime at declared Truffle 25.4.4.1.1, unlike Protos captured/unobservable return-home and lazy-context lifetime. |
| [Oracle SimpleLanguage Bytecode root](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.sl/src/com/oracle/truffle/sl/bytecode/SLBytecodeRootNode.java) | No three-hook declarations; argument/builtin/invocation operations, generated root infrastructure. | Hook absence reflects absence of a demonstrated per-root resource requiring one. |
| [Bridje root](https://github.com/bridje/bridje/blob/189de1af103b5b7eb8cabd1bcbd64a137f493c1e/language/src/main/kotlin/brj/nodes/BridjeRootNode.kt) | No three hooks; explicit captured-variable and argument operations. | Different capture architecture; Truffle version 25.3.4.1. |
| [SWALox root](https://github.com/hpi-swa-lab/swalox/blob/38aa0428bc2471254da6d8ae594265c041a593bd/src/main/java/de/hpi/swa/lox/bytecode/LoxBytecodeRootNode.java) | No three hooks; standard operations and materialized-local access. | Different lexical/resource semantics; Truffle 24.2.0. |

## 5. Exhaustive lifecycle responsibility map

`EQUIVALENCE_ESTABLISHED=YES` denotes an existing Truffle mechanism appropriately used by Protos, **not** equivalence of a newly proposed root hook. Each row includes source owner, dynamic extent, ordinary/error/CFE/yield behavior, normative invariant, potential framework mechanism, and equivalence finding.

```yaml
- RESPONSIBILITY: "Invocation activation"
  CURRENT_OWNER: "Prepared call / compact ABI"
  CURRENT_SOURCE: "P2 P5 P6"
  TRIGGER_AND_DYNAMIC_EXTENT: "After supplied vector; one semantic call"
  NORMAL_EXIT_BEHAVIOR: "Context identity persists"
  ERROR_EXIT_BEHAVIOR: "Partial context can escape"
  CONTROL_TRANSFER_BEHAVIOR: "Owner/home distinguished"
  YIELD_RESUME_BEHAVIOR: "Same activation resumes"
  RELEVANT_NORMATIVE_INVARIANT: "CALLABLES"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "@Prolog"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "Physical root entry is not universal invocation entry"
- RESPONSIBILITY: "Defaults and parameters"
  CURRENT_OWNER: "Source lowerer"
  CURRENT_SOURCE: "P3 P5 P6"
  TRIGGER_AND_DYNAMIC_EXTENT: "Actual invocation before body"
  NORMAL_EXIT_BEHAVIOR: "Earlier slots established in order"
  ERROR_EXIT_BEHAVIOR: "Stops later binding"
  CONTROL_TRANSFER_BEHAVIOR: "Default ^ uses already valid home"
  YIELD_RESUME_BEHAVIOR: "Default may suspend without replay"
  RELEVANT_NORMATIVE_INVARIANT: "CALLABLES defaults"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "@Prolog + operations"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "Effects, operands and control exceed prolog contract"
- RESPONSIBILITY: "Receiver, this, methodHome"
  CURRENT_OWNER: "Invocation ABI"
  CURRENT_SOURCE: "P5 P6"
  TRIGGER_AND_DYNAMIC_EXTENT: "At selected dispatch"
  NORMAL_EXIT_BEHAVIOR: "Lexical provenance retained"
  ERROR_EXIT_BEHAVIOR: "Same guest identity"
  CONTROL_TRANSFER_BEHAVIOR: "Does not change home target"
  YIELD_RESUME_BEHAVIOR: "Persist in frame"
  RELEVANT_NORMATIVE_INVARIANT: "CALLABLES receiver"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "Frame arguments"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "Already carried in compact header"
- RESPONSIBILITY: "Creation of return home"
  CURRENT_OWNER: "Prepared call"
  CURRENT_SOURCE: "P2 P5 P7"
  TRIGGER_AND_DYNAMIC_EXTENT: "Before parameter binding"
  NORMAL_EXIT_BEHAVIOR: "Only owned materialized home completes"
  ERROR_EXIT_BEHAVIOR: "Completion during abnormal call exit"
  CONTROL_TRANSFER_BEHAVIOR: "Captured home is not reowned"
  YIELD_RESUME_BEHAVIOR: "Home remains active through Yield"
  RELEVANT_NORMATIVE_INVARIANT: "CALLABLES return"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "@Prolog"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "Captured/unobservable homes have different lifecycle"
- RESPONSIBILITY: "Capture of return home"
  CURRENT_OWNER: "Closure creation"
  CURRENT_SOURCE: "P6 P7"
  TRIGGER_AND_DYNAMIC_EXTENT: "Lexical nested Closure creation"
  NORMAL_EXIT_BEHAVIOR: "Reference may escape"
  ERROR_EXIT_BEHAVIOR: "Does not substitute home"
  CONTROL_TRANSFER_BEHAVIOR: "^ targets captured home"
  YIELD_RESUME_BEHAVIOR: "Capture remains stable"
  RELEVANT_NORMATIVE_INVARIANT: "CALLABLES ``13–14"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "None"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "Creation is not physical root entry"
- RESPONSIBILITY: "Activation publication"
  CURRENT_OWNER: "ProtosFrameArguments.activation"
  CURRENT_SOURCE: "P1 P5"
  TRIGGER_AND_DYNAMIC_EXTENT: "First materializing consumer"
  NORMAL_EXIT_BEHAVIOR: "One shared published activation"
  ERROR_EXIT_BEHAVIOR: "Can first happen on Error path"
  CONTROL_TRANSFER_BEHAVIOR: "Same activation used on unwind"
  YIELD_RESUME_BEHAVIOR: "No repeated publication"
  RELEVANT_NORMATIVE_INVARIANT: "PLAT036"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "@Prolog"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "Prolog would eager-publish unneeded state"
- RESPONSIBILITY: "Guest context materialization"
  CURRENT_OWNER: "ProtosActivation.context"
  CURRENT_SOURCE: "P5 P6"
  TRIGGER_AND_DYNAMIC_EXTENT: "First actual guest context observer"
  NORMAL_EXIT_BEHAVIOR: "Captured identity stable"
  ERROR_EXIT_BEHAVIOR: "No unrelated eager materialization"
  CONTROL_TRANSFER_BEHAVIOR: "Authority separate from NLR"
  YIELD_RESUME_BEHAVIOR: "No new context at resume"
  RELEVANT_NORMATIVE_INVARIANT: "PLAT036/D179"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "No equivalent eager hook"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "Frame authority stays lazy"
- RESPONSIBILITY: "Dynamic handler install"
  CURRENT_OWNER: "Prepared Error.handle"
  CURRENT_SOURCE: "P2 P4 P10"
  TRIGGER_AND_DYNAMIC_EXTENT: "Validated dynamic scope"
  NORMAL_EXIT_BEHAVIOR: "Frame removed on scope exit"
  ERROR_EXIT_BEHAVIOR: "Selected destination one-shot"
  CONTROL_TRANSFER_BEHAVIOR: "Leaving protected extent removes frame"
  YIELD_RESUME_BEHAVIOR: "Retained Task-locally"
  RELEVANT_NORMATIVE_INVARIANT: "ERRORS"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "TryCatch + explicit operation"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "Scope nested independently of root"
- RESPONSIBILITY: "Handler selection"
  CURRENT_OWNER: "Root Truffle interceptor / CoreErrors"
  CURRENT_SOURCE: "P1 P2 P8 P9 P10"
  TRIGGER_AND_DYNAMIC_EXTENT: "At signal/root crossing before DSL EH"
  NORMAL_EXIT_BEHAVIOR: "No overhead without Error"
  ERROR_EXIT_BEHAVIOR: "Innermost active frame deactivated"
  CONTROL_TRANSFER_BEHAVIOR: "Selected one-shot destination retained"
  YIELD_RESUME_BEHAVIOR: "Token survives continuation"
  RELEVANT_NORMATIVE_INVARIANT: "ERRORS before ensure"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "interceptTruffleException"
  EQUIVALENCE_ESTABLISHED: "YES"
  REASON: "Correct existing Truffle stage"
- RESPONSIBILITY: "Handler deactivation"
  CURRENT_OWNER: "DynamicControlState"
  CURRENT_SOURCE: "P2 P8 P10"
  TRIGGER_AND_DYNAMIC_EXTENT: "Selection or protected exit"
  NORMAL_EXIT_BEHAVIOR: "Frame removed"
  ERROR_EXIT_BEHAVIOR: "Consumed frame inactive before cleanup"
  CONTROL_TRANSFER_BEHAVIOR: "Replacement cannot reselect"
  YIELD_RESUME_BEHAVIOR: "Selection persists if handler suspends"
  RELEVANT_NORMATIVE_INVARIANT: "ERRORS"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "No epilog equivalent"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "Exceptional epilog is too late"
- RESPONSIBILITY: "Normal exact value"
  CURRENT_OWNER: "Return and prepared.finish"
  CURRENT_SOURCE: "P2 P3"
  TRIGGER_AND_DYNAMIC_EXTENT: "Semantic call completion"
  NORMAL_EXIT_BEHAVIOR: "Exact guest object preserved"
  ERROR_EXIT_BEHAVIOR: "No normal result"
  CONTROL_TRANSFER_BEHAVIOR: "Owned NLR may become result"
  YIELD_RESUME_BEHAVIOR: "Not on Yield"
  RELEVANT_NORMATIVE_INVARIANT: "CALLABLES"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "@EpilogReturn"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "A root Return is not the whole prepared call"
- RESPONSIBILITY: "Prepared-call completion"
  CURRENT_OWNER: "PreparedClosureCall + TryFinally"
  CURRENT_SOURCE: "P2 P3 P4"
  TRIGGER_AND_DYNAMIC_EXTENT: "Semantic call leaves its scope"
  NORMAL_EXIT_BEHAVIOR: "Complete home once, return value"
  ERROR_EXIT_BEHAVIOR: "Complete home on error unwind"
  CONTROL_TRANSFER_BEHAVIOR: "Correct NLR home consumed"
  YIELD_RESUME_BEHAVIOR: "Not completed on temporary suspension"
  RELEVANT_NORMATIVE_INVARIANT: "CALLABLES/ensure"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "@EpilogReturn/@EpilogExceptional"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "Caller/child owner, not callee root"
- RESPONSIBILITY: "Non-local return"
  CURRENT_OWNER: "Control-flow bridge and owning prepared call"
  CURRENT_SOURCE: "P2 P7 P11 P12"
  TRIGGER_AND_DYNAMIC_EXTENT: "Executed ^ targets lexical home"
  NORMAL_EXIT_BEHAVIOR: "Matching owner yields result"
  ERROR_EXIT_BEHAVIOR: "Bad home signals InvalidReturn"
  CONTROL_TRANSFER_BEHAVIOR: "Exact original CFE crosses cleanup"
  YIELD_RESUME_BEHAVIOR: "Task reachability preserved"
  RELEVANT_NORMATIVE_INVARIANT: "CALLABLES ``13–14"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "interceptControlFlowException + EH"
  EQUIVALENCE_ESTABLISHED: "YES"
  REASON: "Current bridge intentionally uses DSL EH"
- RESPONSIBILITY: "Escaped invalid home"
  CURRENT_OWNER: "Task and call boundary"
  CURRENT_SOURCE: "P2 P7 P12"
  TRIGGER_AND_DYNAMIC_EXTENT: "Return targets inactive/foreign Task home"
  NORMAL_EXIT_BEHAVIOR: "Never wrong-root return"
  ERROR_EXIT_BEHAVIOR: "Signals InvalidReturn"
  CONTROL_TRANSFER_BEHAVIOR: "Foreign Task transfer rejected"
  YIELD_RESUME_BEHAVIOR: "No remote resumption"
  RELEVANT_NORMATIVE_INVARIANT: "CALLABLES `14"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "No epilog equivalent"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "Requires lexical identity and Task ownership"
- RESPONSIBILITY: "Error occurrence identity"
  CURRENT_OWNER: "ProtosSignalException"
  CURRENT_SOURCE: "P1 P2 P8 P9"
  TRIGGER_AND_DYNAMIC_EXTENT: "Non-resumable Error signal"
  NORMAL_EXIT_BEHAVIOR: "Handler result may recover handle call"
  ERROR_EXIT_BEHAVIOR: "Same Error object delivered"
  CONTROL_TRANSFER_BEHAVIOR: "Cleanup Error can replace original"
  YIELD_RESUME_BEHAVIOR: "Pending transfer survives cleanup suspension"
  RELEVANT_NORMATIVE_INVARIANT: "ERRORS"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "Truffle intercept + EH"
  EQUIVALENCE_ESTABLISHED: "YES"
  REASON: "Framework already properly composed"
- RESPONSIBILITY: "Internal host exception"
  CURRENT_OWNER: "Default framework / outer boundary"
  CURRENT_SOURCE: "P1 P2 P12"
  TRIGGER_AND_DYNAMIC_EXTENT: "Non-guest host Throwable"
  NORMAL_EXIT_BEHAVIOR: "No silent conversion"
  ERROR_EXIT_BEHAVIOR: "Distinct host category"
  CONTROL_TRANSFER_BEHAVIOR: "Not confused with raw CFE"
  YIELD_RESUME_BEHAVIOR: "Retains category at resume"
  RELEVANT_NORMATIVE_INVARIANT: "Host/guest separation"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "interceptInternalException"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "No proven global conversion need"
- RESPONSIBILITY: "Cancellation observation"
  CURRENT_OWNER: "Task resume gate"
  CURRENT_SOURCE: "P2 P12"
  TRIGGER_AND_DYNAMIC_EXTENT: "Portable cancellation checkpoint"
  NORMAL_EXIT_BEHAVIOR: "No extra checkpoints"
  ERROR_EXIT_BEHAVIOR: "Cleanup Error may supersede"
  CONTROL_TRANSFER_BEHAVIOR: "Cancellation original CFE bridged"
  YIELD_RESUME_BEHAVIOR: "Injected at valid resume boundary"
  RELEVANT_NORMATIVE_INVARIANT: "FUTURES_AND_TASKS `23"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "Yield + EH"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "Root hook is not cancellation schedule"
- RESPONSIBILITY: "ensure cleanup"
  CURRENT_OWNER: "Nested generated TryFinally"
  CURRENT_SOURCE: "P3 P4 P10"
  TRIGGER_AND_DYNAMIC_EXTENT: "Semantic scope exit"
  NORMAL_EXIT_BEHAVIOR: "Exactly once, preserve prior result"
  ERROR_EXIT_BEHAVIOR: "Before Error crosses protected scope"
  CONTROL_TRANSFER_BEHAVIOR: "Also handles NLR/cancellation"
  YIELD_RESUME_BEHAVIOR: "No cleanup merely at Yield"
  RELEVANT_NORMATIVE_INVARIANT: "EXECUTION_AND_CONTROL"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "TryFinally"
  EQUIVALENCE_ESTABLISHED: "YES"
  REASON: "Appropriate structured DSL mechanism"
- RESPONSIBILITY: "Cleanup precedence"
  CURRENT_OWNER: "Structured dispatch + Task phase"
  CURRENT_SOURCE: "P4 P10 P12"
  TRIGGER_AND_DYNAMIC_EXTENT: "Crossed protected scope"
  NORMAL_EXIT_BEHAVIOR: "Original pending outcome restored"
  ERROR_EXIT_BEHAVIOR: "Escaping later Error supersedes"
  CONTROL_TRANSFER_BEHAVIOR: "Later valid ^/cancel can supersede"
  YIELD_RESUME_BEHAVIOR: "Pending transfer retained"
  RELEVANT_NORMATIVE_INVARIANT: "ERRORS/ensure"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "TryFinally + semantic operations"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "Not representable by ATE-only epilog"
- RESPONSIBILITY: "Cancellation shielding"
  CURRENT_OWNER: "Task UNWINDING phase"
  CURRENT_SOURCE: "P4 P12"
  TRIGGER_AND_DYNAMIC_EXTENT: "Cleanup of honored cancellation"
  NORMAL_EXIT_BEHAVIOR: "Cancellation continues after cleanup"
  ERROR_EXIT_BEHAVIOR: "Later cleanup Error can win"
  CONTROL_TRANSFER_BEHAVIOR: "Same cancellation not redelivered"
  YIELD_RESUME_BEHAVIOR: "Cleanup may suspend and resume"
  RELEVANT_NORMATIVE_INVARIANT: "FUTURES_AND_TASKS"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "No standalone hook"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "Needs Task state, not root exit"
- RESPONSIBILITY: "Nested child call completion"
  CURRENT_OWNER: "Each child's prepared call"
  CURRENT_SOURCE: "P2 P3 P4"
  TRIGGER_AND_DYNAMIC_EXTENT: "After nested/inline child runs"
  NORMAL_EXIT_BEHAVIOR: "Child complete before parent advances"
  ERROR_EXIT_BEHAVIOR: "Unwind child before parent"
  CONTROL_TRANSFER_BEHAVIOR: "Home ownership remains exact"
  YIELD_RESUME_BEHAVIOR: "Resume same child without replay"
  RELEVANT_NORMATIVE_INVARIANT: "Structured semantics"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "TryFinally + call ops"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "Inline child may have no separate root"
- RESPONSIBILITY: "Yield creation"
  CURRENT_OWNER: "DSL Yield and C-prime composition"
  CURRENT_SOURCE: "P2 P3 P4"
  TRIGGER_AND_DYNAMIC_EXTENT: "Explicit suspendible result"
  NORMAL_EXIT_BEHAVIOR: "Continuation is transport only"
  ERROR_EXIT_BEHAVIOR: "Pending transfer preserved"
  CONTROL_TRANSFER_BEHAVIOR: "Cancellation delivered on resume"
  YIELD_RESUME_BEHAVIOR: "Saved BCI and frame"
  RELEVANT_NORMATIVE_INVARIANT: "Suspension is not exit"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "Yield/ContinuationResult"
  EQUIVALENCE_ESTABLISHED: "YES"
  REASON: "Built-in framework support already used"
- RESPONSIBILITY: "Resume logical execution"
  CURRENT_OWNER: "ResumeContinuation/Task execution"
  CURRENT_SOURCE: "P2 P3 P12"
  TRIGGER_AND_DYNAMIC_EXTENT: "Saved continuation resumes"
  NORMAL_EXIT_BEHAVIOR: "Returns or yields again"
  ERROR_EXIT_BEHAVIOR: "Exception enters same protected scope"
  CONTROL_TRANSFER_BEHAVIOR: "Original CFE recovered"
  YIELD_RESUME_BEHAVIOR: "No new activation/home"
  RELEVANT_NORMATIVE_INVARIANT: "Continuation semantics"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "ContinuationRootNode"
  EQUIVALENCE_ESTABLISHED: "YES"
  REASON: "Resumes from saved location"
- RESPONSIBILITY: "Task and Actor isolation"
  CURRENT_OWNER: "ProtosTask and execution domain"
  CURRENT_SOURCE: "P5 P6 P10 P12 P13"
  TRIGGER_AND_DYNAMIC_EXTENT: "New Task/Actor/Context"
  NORMAL_EXIT_BEHAVIOR: "No inherited handler stack"
  ERROR_EXIT_BEHAVIOR: "Failure transports Error value"
  CONTROL_TRANSFER_BEHAVIOR: "Cross-domain return prohibited"
  YIELD_RESUME_BEHAVIOR: "Suspended control remains Task-local"
  RELEVANT_NORMATIVE_INVARIANT: "ACTORS/FUTURES_AND_TASKS"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "Root frame only"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "Cannot use global root state as owner"
- RESPONSIBILITY: "Source diagnostic origin"
  CURRENT_OWNER: "Semantic root intercept + Error trace"
  CURRENT_SOURCE: "P1 P8 P11"
  TRIGGER_AND_DYNAMIC_EXTENT: "First semantic failure crossing"
  NORMAL_EXIT_BEHAVIOR: "No normal-path trace allocation"
  ERROR_EXIT_BEHAVIOR: "Earliest guest origin captured"
  CONTROL_TRANSFER_BEHAVIOR: "Bridged transfer can retain origin"
  YIELD_RESUME_BEHAVIOR: "Semantic frames projected"
  RELEVANT_NORMATIVE_INVARIANT: "PLAT042/CLI"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "Tags/interceptors"
  EQUIVALENCE_ESTABLISHED: "NO"
  REASON: "Exceptional epilog may lose first crossing"
- RESPONSIBILITY: "Root instrumentation"
  CURRENT_OWNER: "Generated RootTag"
  CURRENT_SOURCE: "P1 P2"
  TRIGGER_AND_DYNAMIC_EXTENT: "Physical semantic root"
  NORMAL_EXIT_BEHAVIOR: "RootTag normal event"
  ERROR_EXIT_BEHAVIOR: "RootTag exceptional event"
  CONTROL_TRANSFER_BEHAVIOR: "Auxiliary root not tagged"
  YIELD_RESUME_BEHAVIOR: "Continuation framework contract"
  RELEVANT_NORMATIVE_INVARIANT: "PLAT042"
  STANDARD_TRUFFLE_FACILITY_AVAILABLE: "enableRootTagging"
  EQUIVALENCE_ESTABLISHED: "YES"
  REASON: "Already standard facility"
```

## 6. Falsification cases A–L

| Case | Required semantics and actual reachable path | Existing source evidence | Standard hook counterfactual / conclusion |
|---|---|---|---|
| **A. Trivial literal** | Compact argument vector → semantic root literal → caller finish; no rich activation unless required | P3/P5/P7, comments/tests for PERF025/PERF038 | Unconditional prolog allocates unnecessary state. **DISPROVEN** for eager adoption. |
| **B. Default with Error or ^** | Home/activation before binding; earlier slots persist; default Error/^ abort later parameter/body. P3 binding, P5 ABI and P6 activation | `ProtosPerf006B4BNonLocalReturnTest.defaultExpressionReturnUsesTheInvocationReturnHome` | Prolog cannot substitute effectful/possibly suspendible default and caller home. **DISPROVEN**. |
| **C. Normal exact result** | `Return` → `EnterClosureCall` result → `FinishClosureCall`/`prepared.finish`; preserves original guest object | P2, `ProtosPerf006B4CStructuredEnsureTest.normalCompletionPreservesExactBodyResultAndRunsCleanupOnce` | Return hook could pass value but cannot own all prepared completion routes. **DISPROVEN** for lifecycle relocation. |
| **D. Escaped NLR** | CFE bridge → crossed cleanup → owning call or invalid-home `InvalidReturn`, never wrong-root local return | P2/P7/P11/P12, `completedCapturedHomeStillSignalsInvalidReturn` | Root epilog lacks target/home/Task authority. **DISPROVEN**. |
| **E. Selected Error + ensure replacement** | At signal/root crossing deactivate selected frame BEFORE `TryFinally`. Cleanup Error supersedes and searches outer handler | P8/P9/P10; `ProtosPerf006B4DErrorHandlersTest.selectedInnerHandlerIsInactiveBeforeCrossedEnsureCleanup` | Exceptional epilog too late. **DISPROVEN**. |
| **F. Protected-body suspension** | `Yield` without cleanup; saved continuation; cleanup only on semantic exit | P3/P4; `bodySuspensionDoesNotRunCleanupAndCompletedPrefixDoesNotReplay` | Root epilog not protected-scope completion. **DISPROVEN**. |
| **G. Cleanup suspension** | Cleanup begins, saves pending transfer, yields/resumes, effects not repeated | P4/P12; `pendingBodyErrorSurvivesCleanupSuspensionWithExactIdentity` | Epilog cannot encode partially completed nested cleanup. **DISPROVEN**. |
| **H. Cancellation in cleanup** | Honored cancellation shielded; cleanup can suspend; cleanup Error may supersede | P4/P12; `cancellationCleanupMaySuspendWithoutRedeliveryOrReplay`, `cleanupErrorSupersedesCancellationBeforeOuterHandlerRuns` | ATE-only exceptional epilog lacks Task cancellation phase. **DISPROVEN**. |
| **I. Nested structured child** | Each child owns its prepared completion even when inline; parent advances after child | P2/P3/P4/P13 and inline while/each lowering | Physical root exit differs from semantic inline-child exit. **DISPROVEN**. |
| **J. Hook throws** | No three hooks in current Protos roots. Existing EH/operation behavior applies | Upstream `PrologEpilogTest` guest/internal failures and nonrecursive exceptional epilog | Hypothetical hook creates new exception extents. **INCONCLUSIVE** for unimplemented change; no current defect. |
| **K. CFE interceptor returns** | Protos throws bridge; generator can return encoded root state directly | P1/P2/P11; generator `createHandleControlFlowException` | `@EpilogReturn` cannot be assumed on direct interceptor return. **DISPROVEN** for equivalence. |
| **L. Multi-Context and Task isolation** | Frame argument ownership, Context-local C-prime plan, Task-local handlers; no cross-Task home/handler inheritance | P5/P6/P10/P12/P13, ACTORS/FUTURES_AND_TASKS | Root-global state would violate isolation; frame-local hook has no missing consumer. Current ownership supported; hypothetical relocation **INCONCLUSIVE**. |

## 7. Nine independent PLAT064 classifications

**Common:** `CURRENT_PROTOS_HEAD=e07340c2ea6ac26278a04495e145a08e3bb2cd4d`, declared `TRUFFLE_API_VERSION=25.4.4.1.1`, compared upstream `95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b`, owner `PLAT064-B` unless specified. `DISPOSITION` below preserves existing semantics; no implementation approval is inferred.

| Facility | Current Protos and peer behavior | DIVERGENCE | DISPOSITION / reason | EVIDENCE_CONFIDENCE |
|---|---|---|---|---|
| `@Prolog` | P1/P5 lazy compact entry vs GraalPy context enter and Pascal activation acquire | REQUIRED_SEMANTIC_CHOICE | **KEEP**: no missing unconditional entry action; pay-as-you-grow | HIGH |
| `@EpilogReturn` | P2 prepared/home completion vs peers' root context/resource exit | REQUIRED_SEMANTIC_CHOICE | **KEEP**: not equivalent to all parent, inline, NLR and direct-interceptor paths | HIGH |
| `@EpilogExceptional` | P2/P9 early Error selection vs peer escaping-ATE cleanup | REQUIRED_SEMANTIC_CHOICE | **KEEP**: too late and excludes raw CFE | HIGH |
| `interceptTruffleException` | P1/P2 selects handler before DSL EH; GraalPy catches/traces before EH | REQUIRED_SEMANTIC_CHOICE | **KEEP**: correct existing framework hook | VERY_HIGH |
| `interceptControlFlowException` | P1/P2/P11 CFE→ATE bridge vs generator can return or rethrow | REQUIRED_SEMANTIC_CHOICE | **KEEP**: preserve precise unwind and ownership | HIGH |
| `interceptInternalException` | P1/P2 framework default and boundary-specific host failures; GraalPy optional conversion | CONFIG_ONLY | **DEFER**: no proven universal host/guest conversion need | MODERATE |
| `TryFinally` | P3/P4 nested scoped completion; Pascal also has Java caller finally | REQUIRED_SEMANTIC_CHOICE | **KEEP**: precisely matching structured protected scope | VERY_HIGH |
| `Yield` / resume | P2/P3/P12 built-in continuation + Task phase; generator resumes saved BCI | REQUIRED_SEMANTIC_CHOICE | **KEEP**: suspension not semantic exit | HIGH, artifact caveat |
| Semantic vs auxiliary roots | P1 tagged, P2 auxiliary untagged, P13 entry; PLAT042 approved | REQUIRED_SEMANTIC_CHOICE | **KEEP**, owner PLAT042; no merging | VERY_HIGH |

No `FRAMEWORK_MISALIGNMENT` was established merely by the absence of annotations.

## 8. Adoption alternatives and GITHUB010/GITHUB021

| Candidate | Hard semantic assessment | Disposition |
|---|---|---|
| A. KEEP existing explicit operations/interceptors/`TryFinally` | Preserves activation, return-home, handler, cancellation, continuation and no-eager-work guarantees | **SELECT FOR RESEARCH CLOSURE** |
| B. Conditional prolog for one entry responsibility | No current redundant operation or real unmet consumer proved | NO ACTION |
| C. Return epilog owns prepared-call finish | Incorrect for nested/inline, abnormal and direct-interceptor routes | REJECT |
| D. Exceptional epilog owns handler/ensure | Violates selection-before-cleanup and CFE/Task/ensure distinctions | REJECT |
| E. All three hooks universally | Fails multiple hard invariants | REJECT |
| F. Small existing-mechanism correction | No concrete current mismatch found | NO ACTION |
| G. Future root-only telemetry | No present consumer or proportional cost of deferral | DEFER |

GITHUB010 proportional qualitative scores (1–5, higher better, not benchmarks). Hard semantic contradictions dominate totals:

| Criterion | KEEP | Conditional prolog | Broad hooks | Rationale |
|---|---:|---:|---:|---|
| Correctness | 5 | 3 | 1 | Root hooks miss semantic exits |
| Semantic fit | 5 | 3 | 1 | Task and prepared ownership differ |
| Present proportionality | 5 | 2 | 1 | No missing consumer |
| Incremental growth | 5 | 3 | 2 | Existing substrate composes scopes |
| Future resilience | 4 | 3 | 2 | Later real root resource can add a hook |
| Scalability | 4 | 3 | 2 | Avoid universal work |
| Simplicity | 4 | 2 | 2 | Shim would duplicate completion |
| Portability | 4 | 4 | 3 | DSL facilities remain supported |
| Cost/overhead | 5 | 2 | 1 | No eager activation/home |
| Failure behavior | 5 | 3 | 1 | Proper handler and cleanup order |
| Reversibility/deferral | 5 | 4 | 2 | No proved cost to defer |
| Evidence maturity | 5 | 2 | 1 | Current tested implementation vs hypothetical |
| **Total / 60** | **56** | **34** | **19** | No measured speed/runtime result |

GITHUB021 preserved: PLAT036 frame-backed lexical authority, D179 C0, CALLABLES fresh activation/default/return-home/Task rules, PLAT042 two roots, PLAT043 Boolean, PLAT062 invoked-call owners, selected handler inactive before ensure, and suspension not scope exit. No approved invariant is reopened.

## 9. Negative findings, existing tests, finite follow-up

Hooks demonstrably cannot replace effectful defaults, lexical home capture, lazy activation publication, early one-shot handler selection, nested ensure/cleanup, Task cancellation shielding or per-inline-child completion. An ATE bridge for a CFE is a **Bytecode EH transport** mechanism, not semantic reclassification of NLR/cancellation as a signaled Error. Hook exception extents and instrumentation are different from the explicit protected Return regions.

No Ixxx is proposed. A later correction would need a concrete code-path contradiction and smallest affected methods, plus preservation tests for: trivial entry; default Error/^; exact result identity; escaped/foreign-Task home; handler selection before crossed cleanup; cleanup Error supersession; suspend in body/cleanup; cancellation shielding; nested/inline completion; interceptor-return bypass; hook-thrown internal/guest errors; SourceTag/RootTag; multi-Context isolation.

Inspected **test sources only**, none executed:

- [B4A transfer substrate](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/test/java/com/guillermomolina/protos/execution/compiler/architecture/ProtosPerf006B4ATransferSubstrateTest.java)
- [B4B non-local return](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/test/java/com/guillermomolina/protos/execution/compiler/control/ProtosPerf006B4BNonLocalReturnTest.java)
- [B4C ensure](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/test/java/com/guillermomolina/protos/execution/compiler/architecture/ProtosPerf006B4CStructuredEnsureTest.java)
- [B4D Error handlers](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/test/java/com/guillermomolina/protos/execution/compiler/architecture/ProtosPerf006B4DErrorHandlersTest.java)
- [B4E cancellation](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/test/java/com/guillermomolina/protos/execution/compiler/control/ProtosPerf006B4ECancellationUnwindTest.java)
- [B4G cross-product](https://github.com/guillermomolina/protos/blob/e07340c2ea6ac26278a04495e145a08e3bb2cd4d/src/test/java/com/guillermomolina/protos/execution/compiler/closures/ProtosPerf006B4GControlUnwindClosureTest.java)

Limits: exact processor artifact identity unverified; I/O C-prime release branches and unrelated generated-root operations not independently exhaustively inspected; no build, test, runtime execution, benchmark, script, profiling, code mutation or architecture ratification. Confidence claims are source-relative.

## 10. Research closure recommendation

```yaml
ISSUE: PLAT064-B
ISSUE_NUMBER: 914
PARENT: PLAT064
PARENT_NUMBER: 911
TYPE: INVESTIGATION
PRODUCT_REVISION: e07340c2ea6ac26278a04495e145a08e3bb2cd4d
DECLARED_TRUFFLE_VERSION: 25.4.4.1.1
UPSTREAM_SOURCE_SNAPSHOT: 95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b
EXACT_PROCESSOR_ARTIFACT_IDENTITY: UNVERIFIED
RESEARCH_DISPOSITION: KEEP
NEW_IMPLEMENTATION_ISSUE: NONE
NEW_DESIGN_RATIFICATION: NONE
PROTOS_CODE_CHANGE: NONE
VALIDATION_EXECUTED: NONE
PERFORMANCE_CLAIM: NONE
OTHER_CHILDREN_C_D_E: INDEPENDENT
PARENT_PLAT064: REMAINS_OPEN
```

After publication/readback of this exact revision, link it in #914 and update parent #911 matrix facility **#4** to `KEEP / NO_ACTION`. Closing #914 is a bounded research-completion action, **not** an approval of any new semantics or implementation. PLAT064-C/D/E remain independently open.
