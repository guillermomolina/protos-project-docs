# BUG017 — P carrier host-failure containment and closure evidence

Date: 2026-10-05

This snapshot records the diagnosis, repair, and validation evidence for
`guillermomolina/protos#800` (BUG017). It is durable non-normative project
evidence; observable Protos semantics remain owned by
`guillermomolina/protos:spec/**`.

## Stable identities

```text
FORMAL_WORK_ITEM=BUG017
GITHUB_ISSUE=guillermomolina/protos#800

ORIGINAL_I058_TRIGGER_REVISION=564dc97aacb593826011a8876554d69dd6529faa
DIAGNOSTIC_REVISION=c1b8a3f87d8c90a654e19069139191dba4d922b6
REPAIR_REVISION=a85d9ca846d7408f869916a59b72f02ee12b9222
REPAIR_VERSION=0.3.210-SNAPSHOT

RELATED_I058=guillermomolina/protos#661
RELATED_BUG015=guillermomolina/protos#794
RELATED_BUG018=guillermomolina/protos#801
SEMANTIC_CHANGE=NO
```

## Trigger and reproduction

I058's canonical `parallel-array-map` benchmark first exposed one width-2
execution that remained live for more than 300 seconds. The event was initially
non-reproducible.

BUG017 diagnostic work later reproduced the failure on Phase-D run 6 at exact
Protos revision `c1b8a3f87d8c90a654e19069139191dba4d922b6`:

```text
CPUSET=0-1
DIAGNOSTIC_RUNS=6
HARD_STALLS=1
THREAD_DUMP_CAPTURED=YES
TWO_DUMP_PROGRESS_COMPARISON=NO_PROGRESS
```

The failing P carrier terminated with an uncaught host
`com.oracle.truffle.api.frame.FrameSlotTypeException`. The captured stack passed
through:

```text
MaterializedLocalAccessor.getObject
 -> ProtosBytecodeRootNode.readCapturedMaterializedBindingOrNull
 -> ProtosInlineCallbackFrameBindings.readCapturedMaterialized
 -> guest execution
 -> ProtosParallelRuntime.runInlinePlaced
 -> ProtosParallelRuntime.run
 -> ThreadPoolExecutor.runWorker
```

After the carrier died, the P Completion had no outcome, its producer Task
remained suspended, the result Future remained pending, and the root execution
waited indefinitely in `ProtosActorExecutionDomain.dispatchUntilTerminal()`.
Two thread dumps taken roughly five seconds apart showed no progress and no
remaining runnable P work.

## Classification

This is not a recurrence of BUG015.

BUG015 repaired a ready-before-suspend race in which the Completion had already
become ready. BUG017 instead had a Completion that never became ready because an
unexpected host exception escaped the P carrier before result publication.

The concrete `FrameSlotTypeException` is independently owned by BUG018/#801.
BUG017 owns only the liveness/terminalization boundary: an unexpected host
execution failure must not abandon an admitted P result Future forever.

The existing specification was sufficient for the repair. In particular,
`spec/concurrency/PARALLEL_EXECUTION.md` requires isolated-parallel failure to
fail its result Future and requires admitted P work to retain progress. No Dxxx,
PLATxxx, or Protos-visible semantic addition was required.

## Published repair

Protos revision
`a85d9ca846d7408f869916a59b72f02ee12b9222`
(`0.3.210-SNAPSHOT`) adds one settlement boundary around P carrier execution.

The repair:

- keeps normal P success and ordinary guest failure behavior unchanged;
- converts unexpected host `RuntimeException` failures into a fresh generic
  standard `Error` occurrence and settles the Completion;
- does not expose the host Throwable, host class, or host message to Protos;
- fails the Completion before rethrowing JVM `Error` conditions to the carrier,
  so the Future does not remain pending while fatal host conditions are still
  not silently converted into ordinary guest failures;
- preserves exactly-once Completion terminalization;
- does not override a cancellation that already won;
- preserves BUG015's ready-before-suspend and cancellation-winning handling; and
- does not modify the captured-materialized frame/accessor machinery owned by
  BUG018.

The deterministic regression
`ProtosParallelExecutionTest.hostExecutionFailureSettlesFutureAsGenericError`
injects an unexpected host failure through the real P carrier settlement
boundary and requires the Future and producer Task to become terminal rather
than remain pending.

## Validation

The maintainer reported after publication:

```text
PUSH_TO_MAIN=PASS
MAINTAINER_REPORTED_LOCAL_TESTS=PASS
PRODUCT_FILES_CHANGED_AS_PUBLISHED=
  CHANGELOG.md
  pom.xml
  src/main/java/com/guillermomolina/protos/execution/ProtosParallelRuntime.java
  src/test/java/com/guillermomolina/protos/execution/ProtosParallelExecutionTest.java
```

The handoff did not include raw local test logs or an exact post-fix stress-run
count, so this record does not fabricate those details. The deterministic
regression and the published source change are revision-bound above.

No GitHub Actions run was yet associated with the repair revision when this
record was prepared; BUG017's Issue contract requires repository validation but
does not independently require a remote-CI run for closure.

## Result

```text
BUG017_ROOT_CAUSE=CONFIRMED
HOST_FAILURE_CONTAINMENT=PUBLISHED
COMPLETION_ABANDONMENT_REPAIRED=YES
BUG015_RECURRENCE=NO
BUG018_FIXED=NO
SEMANTIC_CHANGE=NO
DXXX_REQUIRED=NO
PLATXXX_REQUIRED=NO
MAINTAINER_REPORTED_LOCAL_TESTS=PASS
```

BUG018/#801 remains the independent owner of the captured-materialized
`FrameSlotTypeException`. Repairing BUG017 prevents that class of unexpected
host failure from turning into an indefinitely pending P Future; it deliberately
does not claim that the underlying guest-execution defect is fixed.

AI assistance: this evidence record was drafted with ChatGPT from the
maintainer-provided BUG017 diagnostic capture and validation report, the live
BUG017/BUG018/I058 Issues, and exact inspection of the published repair commit.
