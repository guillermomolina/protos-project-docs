# PLAT049 Candidate C ratification evidence

## Purpose

This record preserves the investigation result, project-owner approval,
GITHUB021 consistency result and implementation handoff for PLAT049
(`guillermomolina/protos#790`).

It is durable non-normative project evidence. The maintained platform decision is:

`docs/project/decisions/platform/PLAT049_GUEST_DIAGNOSTIC_STACK_CAPTURE_AUTHORITY.md`

## Product authority

~~~text
PROTOS_REVISION=ee066755219dfbe6ac7dd69a68b3ac76492bf7ee
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
TRIGGER=CLI008-C/#416
GOVERNING_PRESENTATION_DECISION=D063/#314_CANDIDATE_B_PLUS_S3
~~~

The preceding product checkpoint was
`e7b2ae2cc6688d4ec647306ab3fec270a5687976`. The intervening PERF030-G commit
changes only LocalRangeAccessor PE guard tooling/baselines and does not alter the
audited Error/Task/Future/Bytecode/debugger/CLI diagnostic-stack surfaces.

## Investigation result

The current terminal path preserves exact Error identity but drops the
failure-occurrence stack before terminal CLI presentation:

~~~text
ProtosSignalException
    -> task.fail(signalled.error())
    -> ProtosTask.failure = Error
    -> ProtosExecutionOutcome.failed(error)
    -> CLI
~~~

Some Task failures are manufactured directly, so a solution cannot depend on
retaining a live `ProtosSignalException` until CLI execution.

Existing authority already supplies truthful location inputs:

~~~text
Source / SourceSection
ContinuationResult / ContinuationRootNode
BytecodeLocation
semantic automatic RootTag
PLAT044 nested inline semantic RootTag
~~~

The decisive counterexample to a raw physical-stack solution is PLAT044 B′:
an inline callback can be a distinct semantic activation/RootTag without a
distinct physical Truffle frame.

## Candidate result

~~~text
CANDIDATE_A=REJECTED_PHYSICAL_STACK_INCOMPLETE_FOR_PLAT044
CANDIDATE_B=VIABLE_BUT_DUPLICATES_LOGICAL_CALLER_AUTHORITY
CANDIDATE_C=RECOMMENDED
CANDIDATE_D=REJECTED_ALWAYS_ON_PROVENANCE_OVERENGINEERING
CANDIDATE_E=NO_DISTINCT_SMALLER_SUPPORTED_MECHANISM
~~~

Selected Candidate C is:

~~~text
failure-only physical Truffle stack/current frame walk
    +
continuation/source normalization
    +
PLAT044 active nested semantic RootTag augmentation
    ->
bounded immutable Protos diagnostic occurrence
~~~

After projection, live Truffle/Bytecode/runtime objects are discarded.

## Owner approval

Project-owner approval in the active 2026-10-04 interaction:

~~~text
Apruebo C .
~~~

The approval is exact: Candidate C from the completed PLAT049 decision packet is
selected. It does not approve a Task-local always-on caller stack, async causal
concatenation, live frame retention, Error mutation, extra language semantics or
another candidate hidden inside implementation details.

## Ratified invariants

~~~text
TRACE_OWNER=ERROR_TRANSFER_OR_TERMINAL_FAILURE_OCCURRENCE
ERROR_GUEST_VISIBLE_MUTATION=NO
TRACE_BOUND=64_SEMANTIC_GUEST_FRAMES
SUCCESS_PATH_CAPTURE=NO
TASK_LOCAL_ALWAYS_ON_PROVENANCE=NO
RETAINED_LIVE_FRAME_GRAPH=NO

PHYSICAL_TRUFFLE_STACK_SUFFICIENT=NO
PLAT044_INLINE_FRAME_POLICY=ACTIVE_NESTED_SEMANTIC_ROOTTAG_AUGMENTATION
CONTINUATION_POLICY=NORMALIZE_TO_SEMANTIC_SOURCE_ROOT_AND_BYTECODE_LOCATION

FAILED_FUTURE_VALUE_POLICY=CONSUMER_LOCAL_SIGNAL_ONLY
ACTOR_PROCESS_CAUSAL_STACK=NO_IMPLICIT_CONCATENATION
HOST_JVM_TRUFFLE_HELPER_FRAMES=EXCLUDED
~~~

## GITHUB021 check

~~~text
D063_COMPATIBILITY=PASS_NO_DELTA
PLAT014_COMPATIBILITY=PASS_NO_DELTA
PERF006_B4_B5_COMPATIBILITY=PASS_NO_DELTA
PLAT026_COMPATIBILITY=PASS_NO_DELTA
PLAT034_COMPATIBILITY=PASS_NO_DELTA
PLAT041_COMPATIBILITY=PASS_NO_DELTA
PLAT042_COMPATIBILITY=PASS_NO_DELTA
PLAT044_COMPATIBILITY=PASS_NO_DELTA
PERF025_COMPATIBILITY=PASS_NO_DELTA

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

## Implementation release

The ratified architecture releases CLI008-C/#416.

Smallest mechanical implementation slice:

~~~text
SLICE=CLI008-C1
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
IMPLEMENTATION_AUTHORIZED=YES
~~~

Required implementation coverage includes ordinary nested calls, same Error
signalled at distinct sites, handled Error non-retention, ensure supersession,
direct Task failure, same-Task suspension/resume, consumer-local failed
`Future.value()`, nested composed continuation ordering, PLAT044 inline callback
projection, physical callback fallback, bounded truncation, helper-frame
filtering, concurrent Task/Context isolation, JVM behavior and current Native
Image behavior.

The continuation ordering/de-duplication test is a mechanical acceptance gate
under Candidate C. It becomes a new design gate only if supported pinned APIs
cannot realize the ratified projection.

## Non-authorized work

PLAT049 does not authorize:

- `captureFramesForTrace` or equivalent always-on frame capture merely for CLI
  diagnostics;
- always-stored BCI/frame provenance solely for terminal printing;
- a durable Task logical caller stack;
- producer-to-consumer Future trace transfer;
- Actor/Process trace concatenation;
- live `FrameInstance`, Activation or Continuation retention;
- guest-visible traceback objects;
- source snippets, colors or new CLI formatting policy beyond D063/CLI008-C;
- specification changes.

## Result

~~~text
PLAT049_STATUS=RATIFIED
SELECTED_CANDIDATE=C_FAILURE_ONLY_TRUFFLE_BASE_PLUS_PROTOS_SEMANTIC_AUGMENTATION
IMPLEMENTATION_AUTHORIZED=YES
IMPLEMENTATION_SLICE=CLI008-C1
CLI008_C_RELEASED=YES
~~~
