# CLI008-C guest stack capture re-audit

**Work item:** CLI008-C / guillermomolina/protos#416
**Parent:** CLI008 / guillermomolina/protos#312
**Governing presentation decision:** D063 / guillermomolina/protos#314 — Candidate B + S3
**Platform gate opened by this re-audit:** PLAT049 / guillermomolina/protos#790
**Audited Protos revision:** `6ca7cee5c3a09112268b7a04ed6922086a994f35`
**Current Protos revision reconciled:** `e7b2ae2cc6688d4ec647306ab3fec270a5687976`
**Checkpoint date:** 2026-10-04

## Result

```text
CLI008_C_REAUDIT=PLAT_DECISION_REQUIRED
MECHANICAL_IMPLEMENTATION_READY=NO
PLATFORM_GATE=PLAT049/#790
MISSING_AUTHORITY=GUEST_FRAME_SEQUENCE_AUTHORITY_AND_FAILURE_TIME_CAPTURE_POLICY
MAINTAINER_REPORTED_LOCAL_TESTS=PASS
```

CLI008-C remains open and is blocked on PLAT049. This record is investigation
evidence only; it does not select or ratify a platform candidate and does not
authorize product implementation.

The audit itself was performed against Protos revision
`6ca7cee5c3a09112268b7a04ed6922086a994f35`. Current `main` at coordination
time is five commits ahead at
`e7b2ae2cc6688d4ec647306ab3fec270a5687976`. The intervening delta changes
package-execution and Java-test-policy surfaces only; it does not touch the
Error, Task, Future, Bytecode continuation, debugger/source or CLI
stack-diagnostic surfaces inspected by this re-audit. The routing result
therefore remains applicable to current HEAD.

The maintainer subsequently reported that all local tests pass. That report is
recorded as current product validation evidence; no tests or project commands
were executed by the investigation.

## Transfer and terminal-failure path

The current unhandled Task failure path reduces an Error transfer to the Error
value before terminal CLI presentation:

```text
ProtosSignalException
    -> ProtosBytecodeTaskExecution
    -> task.fail(signalled.error())
    -> ProtosTask.failure = Error
    -> ProtosExecutionOutcome.failed(error)
    -> CLI
```

Relevant owners include:

- `src/main/java/com/guillermomolina/protos/runtime/ProtosSignalException.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosCoreErrors.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeTaskExecution.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosTask.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosExecutionOutcome.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosRootTaskExecution.java`
- `src/main/java/com/guillermomolina/protos/cli/ProtosCli.java`

The exact guest Error identity is preserved, but the transfer occurrence is not.
`ProtosSignalException` currently carries the exact Error and selected-handler
state, while Task and `ProtosExecutionOutcome` retain only the terminal Error
value. Some Task failures also originate directly as `task.fail(error)`, so a
future diagnostic carrier cannot assume that every terminal failure still owns
a live `ProtosSignalException`.

CLI paths that convert a failed `ProtosExecutionOutcome` back into a new
`ProtosSignalException` cannot reconstruct the original occurrence after that
information has been discarded.

## Source and logical-location authority already available

PERF006-B5 established sufficient authority to locate an already-identified
guest execution point:

```text
Source / SourceSection
ContinuationResult -> BytecodeLocation -> SourceSection
```

`ProtosPerf006B5BSourceInstrumentationLocationTest` freezes exact Source
identity and source ranges for semantic roots and demonstrates stable caller and
callee `BytecodeLocation` values across C-prime suspension/resume without
completed-prefix replay.

`ProtosModuleSource` preserves physical source identity when available and a
canonical virtual identity otherwise.

Therefore the unresolved CLI008-C question is not how to obtain line/column
information for one known frame. The missing authority is the ordered guest
frame sequence and the failure-time snapshot/capture mechanism.

## Suspension and Future boundary

C-prime suspension retains the complete top-level continuation in the suspended
Task, and nested `ContinuationResult` state preserves the logical execution
needed to resume. This proves that same-Task execution state survives suspension,
but current terminal failure does not materialize that state into an inert
diagnostic trace before reducing the failure to an Error value.

The normative Future contract removes a separate async-causality ambiguity:
a failed Future stores the exact domain-local Error, while `Future.value()`
re-signals that Error as a new consumer-side signaling occurrence and explicitly
does not preserve producer activation frames, continuations, handlers, return
homes or other producer control state.

CLI008-C must therefore not concatenate a producer stack with the consumer
`Future.value()` stack.

Relevant normative authority:

- `spec/semantics/ERRORS.md`
- `spec/concurrency/FUTURES_AND_TASKS.md`

## PLAT044 inline semantic activation constraint

PLAT044 Candidate B-prime deliberately permits an eligible immediate literal
Closure callback to remain a real semantic activation/scope while executing
inside its caller's physical semantic Bytecode root. The callback no longer has
a separate physical `FrameInstance` / `TruffleStackTraceElement`.

Consequently:

```text
semantic guest activation != always one physical Truffle frame
```

A raw physical Truffle stack cannot be adopted mechanically as the complete
D063 S3 guest-stack authority without deciding how these inline semantic
activations map to diagnostic frames.

Relevant current implementation/tooling surfaces include:

- `src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java`
- `src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeTagTreeNodeExports.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosI026EDebuggerIntegrationTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosI026FDapBehaviorTest.java`

The debugger/DAP work supplies reusable source/scope projection principles, but
there is no existing Protos-neutral guest-stack DTO or stack-sequence authority
that CLI008-C can simply consume.

## Actor and Process boundaries

Current normative semantics prohibit manufacturing a cross-domain causal stack:

- an unhandled Actor Error is local failure state of that Actor incarnation;
- accepted request failure does not implicitly transfer the destination Actor's
  internal Error or stack to the requester;
- Process and Actor isolation/failure boundaries do not imply a synchronous
  caller chain.

Relevant authority:

- `spec/semantics/ERRORS.md`
- `spec/concurrency/ACTORS.md`
- `spec/concurrency/DISTRIBUTED_RUNTIME.md`

CLI008-C baseline therefore needs only the truthful local Task guest stack for
the actual signaling occurrence. Optional async-origin segments remain outside
the baseline unless independently retained provenance is later approved.

## Cost and retention consequence

D063 requires successful ordinary execution not to pay a large stack-diagnostic
tax. Current PERF025 work also deliberately avoids materializing rich
`ProtosActivation` state unless it becomes observable.

A compatible final architecture should therefore be capable of this shape:

```text
successful ordinary execution
    -> no diagnostic stack snapshot

escaping Error / terminal failure
    -> bounded immutable guest-frame snapshot
    -> no retained live Frame/Activation/Continuation graph
```

The audit does not choose which supported Truffle/Bytecode/runtime mechanism
produces that sequence. Selecting that mechanism is the PLAT049 decision.

## Why implementation is not mechanical

D063 selected the public S3 presentation contract but intentionally deferred the
physical Truffle/JVM stack-capture representation.

PERF006-B4/B5 subsequently closed:

```text
Error/unwind semantics                         YES
C-prime suspension/resume                     YES
exact source ownership                        YES
stable BytecodeLocation after resume           YES
debugger/source projection                    YES
```

but did not close:

```text
durable diagnostic caller-chain authority      NO
terminal-failure frame snapshot                NO
PLAT044 inline-activation frame policy         NO
selected failure-time capture mechanism        NO
```

Choosing among physical Truffle capture, explicit logical-frame snapshot,
hybrid projection or retained task-local provenance would therefore create a
durable runtime/tooling architecture choice inside CLI implementation. That is
outside CLI008-C's mechanical authority.

## Routing

PLAT049 / guillermomolina/protos#790 now owns the exact missing decision:

> What runtime-owned mechanism is the authoritative source for materializing,
> only when an Error transfer escapes or becomes a terminal failure, the bounded
> guest-only D063 S3 frame sequence for the current Task, including semantic
> source roots, composed/suspended Bytecode continuations and PLAT044 inline
> semantic callback activations, without successful-path per-call capture or
> retention of live frame graphs?

CLI008-C/#416 remains blocked until PLAT049 receives explicit project-owner
selection and any required durable ratification.

The next executable work unit is therefore investigation only:

```text
NEXT_SLICE=PLAT049-A
TYPE=INVESTIGATION
COMMAND_EXECUTION=NONE
IMPLEMENTATION_AUTHORIZED=NO
```

## Coordination limitation

Project policy requires native Parent/Sub-issue and native blocked-by edges for
formal relationships. The available GitHub connector can create and update
Issues but does not expose those relationship mutations. PLAT049 therefore
records textual `Parent: #416` / `Blocking: #416` bootstrap evidence, but
this record does not claim that the native hierarchy/dependency graph has been
reconciled.
