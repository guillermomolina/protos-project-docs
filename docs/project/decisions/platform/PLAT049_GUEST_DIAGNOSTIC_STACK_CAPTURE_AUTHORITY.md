# PLAT049 — Guest diagnostic stack capture authority

Status: **RATIFIED**

Selected architecture: **Candidate C — failure-only physical Truffle stack base
plus deterministic Protos semantic projection and augmentation**.

Approval: explicit project-owner approval on 2026-10-04:

~~~text
Apruebo C .
~~~

Decision Issue: `guillermomolina/protos#790`

Primary implementation consumer: `CLI008-C / guillermomolina/protos#416`

Product authority inspected for the decision packet:

~~~text
PROTOS_REVISION=ee066755219dfbe6ac7dd69a68b3ac76492bf7ee
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
~~~

Nature: durable non-normative Truffle/runtime architecture. Observable Protos
language, Error identity, Future/Task semantics, Actor/Process isolation and
ordinary program output remain unchanged.

## Decision

The durable runtime authority for D063 S3 uncaught guest diagnostics is an
immutable Protos diagnostic occurrence snapshot captured only when an Error
transfer escapes or becomes a terminal Task failure.

The current Truffle backend acquires the physical failure-time stack only on that
exceptional path. Protos immediately projects it into bounded immutable guest
facts, augments it with semantic frames that intentionally have no separate
physical Truffle frame, and then discards every live runtime object used during
capture.

After capture, the immutable Protos snapshot is authoritative. A
`TruffleStackTrace`, `FrameInstance`, `ContinuationResult`,
`BytecodeLocation`, `RootNode`, `Source`, `ProtosActivation` or other live
runtime graph is never retained merely so a later CLI formatter can print a
trace.

Conceptually:

~~~text
terminal Error occurrence while execution is live
    |
    +-- failure-only Truffle physical stack / current frame walk
    |
    +-- normalize generated continuation roots
    |      -> semantic source root
    |      -> BytecodeLocation / SourceSection
    |
    +-- augment active PLAT044 B-prime nested semantic RootTag regions
    |
    +-- filter structured/C-prime/runtime/host helper roots
    |
    v
bounded immutable Protos DiagnosticTrace
    |
    v
Task terminal failure / ProtosExecutionOutcome / CLI
~~~

## Why raw physical stack authority is insufficient

PLAT044 B′ intentionally permits an eligible semantic Closure callback to execute
inside its caller's physical semantic Bytecode root while retaining a distinct
semantic activation, scope and semantic `RootTag`, but **without** a distinct
`FrameInstance` or `TruffleStackTraceElement`.

Therefore:

~~~text
SEMANTIC_GUEST_ACTIVATION != ALWAYS_ONE_PHYSICAL_TRUFFLE_FRAME
PHYSICAL_TRUFFLE_STACK_SUFFICIENT=NO
~~~

Candidate A, which treats the filtered physical stack itself as the complete
guest trace, is rejected by this already-ratified counterexample.

## Failure occurrence ownership

D063 remains authoritative:

- diagnostic trace metadata belongs to an Error transfer/failure occurrence;
- it is not a guest-visible slot or mutation on the Error object;
- re-signalling the same Error is a distinct occurrence and may produce a
  different trace;
- only genuine guest/tooling facts may be printed;
- host/JVM/Truffle/scheduler/internal helper frames are excluded;
- async, Actor and Process causal stacks are never fabricated.

The exact Error identity continues to flow through existing semantics.

The diagnostic snapshot must be established at the first terminal-failure commit
while the original occurrence/current execution stack is still available. This
is **before** later structured-child drain can defer final Task publication.

A direct Task failure that manufactures an Error without a live
`ProtosSignalException` may perform a failure-only current Truffle frame walk at
that exact failure site. Every `FrameInstance` is consumed inside the walk and
does not escape it.

## Immutable DTO boundary

The smallest sufficient retained shape is implementation-private and inert:

~~~text
DiagnosticTrace
    frames: immutable sequence<DiagnosticFrame>
    truncated: boolean

DiagnosticFrame
    executableLabel: optional String
    sourceOrModuleIdentity: String
    displayPath: optional String
    line: int
    column: int
~~~

A label is present only when current semantic metadata genuinely establishes it.
Anonymous Closure frames remain anonymous.

The retained DTO must not hold strong references to:

~~~text
Frame
MaterializedFrame
FrameInstance
ProtosActivation
ContinuationResult
ContinuationRootNode
RootNode
CallTarget
BytecodeNode
BytecodeLocation
Source
Context
~~~

No frame-kind field is required merely to preserve the physical mechanism used
during capture.

## Bound and truncation

The retained D063 S3 baseline is:

~~~text
TRACE_BOUND=64_SEMANTIC_GUEST_FRAMES
TRUNCATION=KEEP_INNERMOST_64_AND_SET_TRACE_TRUNCATED
~~~

No fabricated "... frame" is part of the semantic DTO, and no exact omitted
frame count is required.

The implementation must not pre-filter the underlying Truffle trace with a
smaller arbitrary physical limit that can be consumed by helper or continuation
roots before the 64 semantic frames are recovered.

A later performance-only refinement may bound the physical substrate more tightly
only after proving that semantic frames cannot be under-captured.

## Ordinary semantic roots

A physical `ProtosSemanticBytecodeRootNode` is projected from its current
Bytecode location to the truthful `SourceSection`.

Untagged `ProtosBytecodeRootNode` structured/C-prime roots are implementation
machinery. They do not become guest diagnostic frames merely because they appear
physically.

Host, JVM, scheduler and unrelated Truffle infrastructure are likewise excluded.

## Same-Task suspension and continuation roots

PERF006-B5 remains source/location authority for composed C-prime suspension.

A resumed generated continuation is not presented as a separate artificial
"continuation" guest frame. The capture adapter normalizes it through the
generated continuation's semantic source root and `BytecodeLocation`.

Nested composed continuations are projected in actual unwind order and
de-duplicated by semantic frame identity, not Java class/object identity.

This keeps the current Task's real logical stack without inventing pre-suspension
history that no longer exists.

## PLAT044 inline semantic frames

For a physical semantic Bytecode frame at its current BCI, the capture adapter
consults the existing Bytecode tag tree.

The enclosing automatic root tag corresponds to the physical semantic activation
already represented by the physical frame.

Any active nested PLAT044 B′ semantic `RootTag` represents an inline callback
activation that intentionally has no separate physical Truffle frame. Those
nested regions are emitted as diagnostic semantic frames in innermost-to-outer
order before the enclosing physical semantic frame.

This is a diagnostic projection only. It must **not** fabricate a Truffle
`FrameInstance`, `RootCallTarget` or debugger frame and must not materialize a
`ProtosActivation` merely for stack presentation.

## Failed Future observation

A failed Future continues storing only the exact Error required by existing
semantics.

A later consumer `Future.value()` observation is a new signal in the
consumer's current dynamic context:

~~~text
producer Task failure
    -> Future stores exact Error only

consumer Future.value()
    -> same Error identity
    -> new consumer-side signal occurrence
    -> consumer Task diagnostic trace if terminal
~~~

Producer stack/control state is not copied or concatenated.

## Actor and Process boundaries

Actor-local fatal Errors remain local to the Actor.

Accepted request failure and Process/Actor communication continue using their
existing semantic outcomes. The destination's internal diagnostic trace is not
implicitly transferred to the caller.

No implicit cross-Actor or cross-Process causal stack is authorized.

## Successful-path cost

Candidate C adds no per-call diagnostic caller chain, stack snapshot, Task-local
provenance list, activation retention or extra synchronization.

~~~text
SUCCESS_PATH_STACK_CAPTURE_COST=ZERO_ADDITIONAL
TASK_LOCAL_ALWAYS_ON_PROVENANCE=NO
RETAINED_LIVE_FRAME_GRAPH=NO
~~~

Handled Errors use the ordinary existing Truffle exception/unwind and handler
machinery; no terminal diagnostic DTO needs to survive a handled occurrence.

Only an escaping/terminal failure pays for trace projection and the bounded
immutable DTO.

## Candidate disposition

### A — failure-only Truffle stack only

Rejected.

Cost is excellent but the physical stack is semantically incomplete for
PLAT044 B′ inline callbacks.

### B — explicit Protos logical-frame snapshot

Viable but not selected.

It would require a new logical caller-chain authority or accumulation protocol
duplicating information already available from physical failure-time stack,
continuation metadata and semantic tags.

### C — physical base plus semantic projection/augmentation

Selected.

It reuses current runtime authority, adds no success-path bookkeeping, handles
the known PLAT044 mismatch explicitly and leaves the retained DTO independent of
Truffle after capture.

### D — durable Task-local logical caller provenance

Rejected as overengineering for the present requirement.

Every successful call/return/suspend/resume would pay for diagnostic provenance
whose only current consumer is a terminal Error path. This conflicts with the
pay-for-what-you-need/pay-as-you-grow direction reinforced by PERF025.

### E — another supported mechanism

No materially distinct public mechanism was found that provides the same truth
with a smaller cost/authority surface.

Capturing actual Bytecode frames, debugger suspended frames or private generated
continuation machinery is either heavier, debugger-lifetime-bound or an
implementation-internal variant rather than a distinct architecture.

## GITHUB010 comparative scorecard

Scores are 1-5, higher is better.

| Dimension | A | B | C | D |
| --- | ---: | ---: | ---: | ---: |
| Correctness / invariant preservation | 2 | 4 | 5 | 4 |
| Protos alignment | 3 | 4 | 5 | 2 |
| Present-need proportionality | 5 | 3 | 5 | 1 |
| Incremental growth | 2 | 4 | 5 | 3 |
| Future-option resilience | 2 | 5 | 4 | 4 |
| Scalability | 4 | 4 | 5 | 2 |
| Conceptual simplicity | 5 | 3 | 4 | 2 |
| Portability / implementation freedom | 1 | 5 | 4 | 4 |
| Runtime / resource cost | 5 | 4 | 5 | 1 |
| Failure / operability | 2 | 4 | 5 | 4 |
| Deferral / reversibility / migration | 2 | 4 | 5 | 2 |
| Evidence maturity / implementation risk | 4 | 2 | 4 | 3 |
| **Total / 60** | **37** | **46** | **56** | **32** |

The score does not override hard constraints. Candidate A fails the PLAT044
semantic-frame counterexample. Candidate D carries an explicit overengineering
red flag because all successful execution would pay for unused diagnostic
provenance.

Recommendation confidence is **MEDIUM-HIGH**. The residual implementation risk
is generated continuation-root ordering/de-duplication under the pinned Bytecode
DSL, which must be frozen by focal semantic-output tests.

## GITHUB021 invariant/delta result

Candidate C preserves the applicable previously ratified invariants.

~~~text
D063_DELTA=NONE
PLAT014_DELTA=NONE
PERF006_B4_B5_DELTA=NONE
PLAT026_DELTA=NONE
PLAT034_DELTA=NONE
PLAT041_DELTA=NONE
PLAT042_DELTA=NONE
PLAT044_DELTA=NONE
PERF025_DELTA=NONE

DECISION_INVARIANT_CONSISTENCY=PASS
OBSERVABLE_PROTOS_LANGUAGE_SEMANTIC_CHANGE_REQUIRED=NO
SPECIFICATION_CHANGE_REQUIRED=NO
~~~

In particular, a semantic diagnostic frame for a PLAT044 inline callback does
not reverse the approved absence of a distinct physical callback
`FrameInstance`/`TruffleStackTraceElement`.

## Portability and future migration

The occurrence DTO and terminal-failure ownership boundary are backend-neutral.

The current Truffle capture adapter is intentionally backend-specific. If a
future backend cannot reconstruct a failure-time physical stack faithfully, only
that adapter must be replaced with an equivalent producer of immutable semantic
frames.

No guest semantics, Error representation, Task scheduling contract or persisted
state format must migrate.

The strongest argument against Candidate C is this deliberate split of capture
authority:

~~~text
Truffle physical call order
+
Protos continuation/source normalization
+
PLAT044 semantic RootTag augmentation
~~~

A future Truffle Bytecode DSL topology change may therefore require adapter
maintenance and regression-test updates. This does not justify pre-paying for a
durable Task-local caller stack today.

## Released implementation

Ratification releases one bounded implementation slice in
`guillermomolina/protos`:

~~~text
SLICE=CLI008-C1
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
GOAL=Occurrence-carried bounded guest trace substrate + Candidate-C projection + CLI rendering
~~~

The implementation may add:

1. immutable internal `DiagnosticFrame` / `DiagnosticTrace`;
2. one failure-only Truffle capture/projection adapter;
3. direct-failure current-stack projection when no live signal exception remains;
4. separate Task retention of the trace while preserving exact
   `task.failure()` Error identity;
5. `ProtosExecutionOutcome` carriage of the occurrence trace;
6. D063 S3 CLI rendering;
7. focal JVM and current Native Image regression coverage.

It must not:

- enable always-on frame capture merely for diagnostics;
- add a Task-local logical caller stack;
- retain live frames, activations or continuations;
- mutate Error guest-visible structure;
- concatenate producer Future/Actor/Process traces;
- expose host/JVM/Truffle/helper frames;
- alter PLAT044 physical topology;
- change ordinary `print(...)` output.

## Ratified result

~~~text
PLAT049_STATUS=RATIFIED
SELECTED_CANDIDATE=C_FAILURE_ONLY_TRUFFLE_BASE_PLUS_PROTOS_SEMANTIC_AUGMENTATION

CURRENT_PRODUCT_REVISION=ee066755219dfbe6ac7dd69a68b3ac76492bf7ee
PHYSICAL_TRUFFLE_STACK_SUFFICIENT=NO
TRACE_BOUND=64_SEMANTIC_GUEST_FRAMES

PLAT044_INLINE_FRAME_POLICY=ACTIVE_NESTED_SEMANTIC_ROOTTAG_AUGMENTATION
SAME_TASK_SUSPENSION_RESUME_POLICY=NORMALIZE_CONTINUATION_ROOTS_TO_SOURCE_BYTECODE_LOCATIONS
FAILED_FUTURE_VALUE_POLICY=CONSUMER_LOCAL_SIGNAL_ONLY
ACTOR_PROCESS_CAUSAL_STACK=NO_IMPLICIT_CONCATENATION

SUCCESS_PATH_STACK_CAPTURE_COST=ZERO_ADDITIONAL
RETAINED_LIVE_FRAME_GRAPH=NO
TASK_LOCAL_ALWAYS_ON_PROVENANCE=NO

GITHUB021_INVARIANT_DELTA_CHECK=PASS
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO

IMPLEMENTATION_AUTHORIZED=YES
IMPLEMENTATION_SLICE=CLI008-C1
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
~~~

## References

- `guillermomolina/protos#790` — PLAT049 decision Issue.
- `guillermomolina/protos#416` — CLI008-C implementation owner.
- `guillermomolina/protos#314` — D063 Candidate B + S3.
- `docs/project/evidence/PLAT049/PLAT049_CANDIDATE_C_RATIFICATION_EVIDENCE.md`.
- `docs/project/evidence/CLI008/CLI008_C_GUEST_STACK_CAPTURE_REAUDIT.md`.
- `docs/project/evidence/CLI008/CLI008_C_PLAT049_RELEASE.md`.
- `docs/project/decisions/platform/PLAT044_SEMANTIC_CLOSURE_PHYSICAL_ROOT_BOUNDARY.md`.
