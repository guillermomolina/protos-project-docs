# D174 — Test Tool case lifecycle observability and scalable progress reporting

Status: **RATIFIED — Candidate C′**

Approval date: **2026-09-19**  
Decision issue: `guillermomolina/protos#673`  
Parent work: TOOL009 / `guillermomolina/protos#600`  
Protos revision at ratification: `9473ee45566c6f75aa95165917ab38159fd5459b`  
TOOL009 runner evidence baseline: `ca7f4e6633c356edf5423e6691eeeda869532be6`  
Project-record base: `08b351e83f9eb7c66f386a55b5237d97dd313e21`

This is a durable non-normative tooling decision record. It does not define
observable Protos language or Standard Library semantics.

## Decision

D174 selects **Candidate C′ — internal presentation-neutral Case lifecycle
observations feeding a separate reporter**.

The durable boundary is:

```text
CaseStarted(caseRef)
CaseTerminal(caseRef, result)
```

The names above describe the lifecycle contract, not a required concrete type,
class hierarchy, queue, object representation or public API.

### Case lifecycle

For every logical Case admitted for execution:

```text
CaseStarted
    occurs exactly once
    after admission/resolution
    immediately before Case execution begins

CaseTerminal
    occurs exactly once
    only when the Case reaches the existing terminal/reconciliation boundary
```

Therefore a hung Case is observably in-flight:

```text
CaseStarted exists
CaseTerminal absent
```

Cross-Case ordering is intentionally unspecified. Per-Case ordering is:

```text
Started -> Terminal
```

### caseRef boundary

`caseRef` is invocation-local, inert and presentation-neutral.

It is not:

- a new global CaseId;
- a public persisted identity;
- a live `Test` value;
- a live body Closure;
- a new distributed identity contract.

The selected boundary preserves D152/D153 local-first logical Case authority.

## Scheduler versus reporter

The scheduler/execution path:

- emits or otherwise publishes the internal lifecycle observations;
- performs no progress-terminal formatting;
- performs no mandatory progress write per Case;
- owns no progress UI;
- owns no TTY policy;
- remains presentation-neutral.

A reporter consumes lifecycle observations and may derive:

```text
total
completed
failed
currentlyRunning
```

The reporter, not the scheduler, decides if or when those states are rendered.

Lifecycle observation is therefore **not equivalent to terminal I/O**.

## Performance invariant

The project owner explicitly rejected a design whose durable architecture is
merely one `print("RUN ...")` per Case.

The selected boundary preserves this invariant:

> Progress I/O is not required to scale linearly with Case execution count.
> A presentation mode may explicitly choose per-Case output, but the execution
> architecture must not require that cost.

A reporter may coalesce, rate-limit, snapshot or suppress presentation while
still receiving enough lifecycle information to identify in-flight Cases.

D174 does not select a concrete coalescing interval, buffering algorithm or TTY
rendering policy.

## Legacy and suite-native execution

D174 selects one conceptual lifecycle boundary for both:

- incumbent/legacy Test Tool execution; and
- TOOL009 suite-native logical Case execution.

The current legacy path already has a terminal observer after terminal
reconciliation. The suite-native path currently projects logical Case results
only after `LogicalCaseRunner.run(...)` completes. D174 requires both paths to
be able to report Case start and terminal transitions without making
presentation part of scheduling.

Implementation may adapt the two existing paths differently as long as they
preserve the same selected lifecycle semantics.

## Why this decision was required

During TOOL009 migration of the `control` conformance domain, a selected
suite-native Case hung while directory execution exposed aggregate progress but
not the identity of the currently running Case.

The existing suite-native flow had enough Case identity inside
`LogicalCaseRunner`, but `Progress` only saw results after execution
completed. A Case that never reached completion was therefore invisible to the
progress reporter.

The incident demonstrated two distinct requirements:

1. a Case must become observable before it can hang; and
2. satisfying that requirement must not turn every Case transition into
   mandatory terminal I/O.

## Comparative evidence

D174 compared at least five mature systems spanning materially different
reporting designs.

### cargo-nextest

Official documentation:
<https://nexte.st/docs/reporting/>
<https://nexte.st/docs/running/>

nextest separates test execution from human-readable reporting. Interactive
progress can display a bounded number of currently running tests; excess running
tests are collapsed. Non-interactive and interactive output may use different
presentation modes.

Relevant lesson for D174: retaining in-flight state does not require printing one
line for every test.

### Go test2json

Official documentation:
<https://pkg.go.dev/cmd/test2json>

Go exposes a live stream of structured test events including `run`, `pass`,
`fail`, `pause`, `cont` and output events. Events from parallel tests may
interleave while retaining test identity.

Relevant lesson: execution facts can be represented independently from their
human presentation.

### JUnit Platform

Official API:
<https://docs.junit.org/6.0.0/api/org.junit.platform.launcher/org/junit/platform/launcher/TestExecutionListener.html>

JUnit exposes lifecycle callbacks including `executionStarted` and
`executionFinished` around test identifiers.

Relevant lesson: explicit start/finish lifecycle is a mature boundary. D174 does
not copy JUnit's public listener institution; it keeps the Protos boundary
internal and presentation-neutral.

### pytest

Official reference:
<https://docs.pytest.org/en/latest/reference/reference.html>

pytest exposes hooks including `pytest_runtest_logstart`,
`pytest_runtest_logreport` and `pytest_runtest_logfinish`, while terminal
reporting is a separate concern.

Relevant lesson: execution lifecycle and terminal representation need not be the
same mechanism.

### Bazel Build Event Protocol

Official documentation:
<https://bazel.build/remote/bep>
<https://bazel.build/versions/8.5.0/remote/bep-glossary>

Bazel exposes a rich structured event protocol for programmatic consumers,
including test result and summary events.

Relevant lesson: structured execution/reporting separation scales to large
systems. D174 deliberately does **not** pre-build Bazel-scale protocol machinery
because Protos has no present requirement for a public event service, persistent
event graph or remote reporting protocol.

## Candidate result

### Status quo / defer

Rejected.

The motivating hang demonstrated a current operability failure: a Case can begin
execution and then disappear from progress observability indefinitely.

### Candidate A — direct per-Case scheduler printing

Rejected.

It couples scheduling to presentation and makes terminal I/O proportional to
Case execution unless additional special cases are added later.

### Candidate B — presentation callbacks executed directly by scheduling lanes

Rejected as the durable architecture.

It separates formatting names from the scheduler, but still permits arbitrary
presentation work to execute on the scheduling path and does not establish the
clean presentation-neutral boundary required for growth.

### Candidate C′ — internal lifecycle observations + separate reporter

**Selected.**

It adds only the execution facts required today while keeping representation,
TTY policy, buffering, rate limiting and richer reporting independently
evolvable.

### Candidate D — public/general event bus or durable event protocol

Rejected for now.

It can support future IDE, remote, recording and machine-consumer scenarios, but
those requirements do not exist today. Queue semantics, delivery policy,
backpressure, persistence, versioning and public compatibility would be
speculative cost.

### Timeout-centric diagnosis

Not selected as a substitute.

Timeouts may later be useful, but they are execution policy rather than the
observability boundary required by D174.

## Twelve-dimension comparison result

The decision packet evaluated every surviving candidate on the required common
dimensions:

1. correctness / invariant preservation;
2. Protos alignment;
3. present-need proportionality;
4. incremental growth;
5. future-option resilience;
6. scalability;
7. conceptual simplicity;
8. portability / implementation freedom;
9. runtime / resource cost;
10. failure / operability;
11. deferral / reversibility / migration;
12. evidence maturity / implementation risk.

Candidate C′ was selected qualitatively rather than by arithmetic total.

Its decisive advantages were:

- it directly represents the missing pre-execution fact;
- it keeps the scheduler presentation-neutral;
- it permits bounded or zero terminal I/O despite arbitrarily many Cases;
- it introduces no public protocol or persistent identity;
- richer reporting can be added as consumers/policies rather than by replacing
  the scheduler boundary.

A full public event bus scored well on future-option resilience but carried an
overengineering red flag. Direct printing scored well on immediate
implementation size but violated the owner-approved performance/architecture
invariant.

## Adversarial incremental-design gate

### Smallest sufficient solution

The smallest durable solution is two internal lifecycle transitions plus a
reporter-derived in-flight set.

No queue, public listener API, timestamps, history, event persistence or output
format is necessary to satisfy the current requirement.

### Pay for what is needed

Every Case pays only the minimal internal lifecycle-observation cost required to
make starts and terminalization visible.

Users do not automatically pay one terminal write per Case.

### Grow as needed

A later reporter may add:

- richer TTY display;
- bounded running-Case lists;
- machine-readable output;
- recording;
- slow-test diagnostics;
- IDE integration.

Those are additional consumers/policies over the same lifecycle boundary rather
than changes to Case execution semantics.

### Cost of deferral

Deferring the lifecycle boundary itself would require reopening both legacy and
suite-native scheduler paths later and would leave current hangs
undiagnosable.

Deferring richer presentation has low cost because it can be layered over the
selected lifecycle observations.

### Speculation burden

D174 preserves escape paths for richer reporting but does not implement them.

It is intentionally future-compatible rather than future-preimplemented.

## Deliberately deferred

The following remain unselected:

```text
per-Case timeout policy
slow-test thresholds
retries
public JSON/event-stream contract
JUnit/XML report artifact policy
persistent event history
terminal UI framework
interactive input handler
distributed/remote reporting
public plugin listener API
global/persistent Case identity
```

Any of these may be evaluated independently if concrete requirements arise.

## Invariant / delta consistency

The owner-approved D174 invariant recorded before candidate selection was:

```text
A simple print per Case is not sufficient durable architecture.
Progress I/O must not be inherently proportional to Case execution count.
```

Candidate C′ preserves it.

D174 also preserves the relevant existing TOOL009/D152/D153 boundaries:

```text
ONE_TEST_TO_ONE_LOGICAL_CASE=KEEP
ONE_SOURCE_TO_ZERO_ONE_MANY_CASES=KEEP
FILE_PATH_NOT_CASE_IDENTITY=KEEP
LIVE_BODY_CLOSURES_OUTSIDE_CASE_PLAN=KEEP
FRESH_SEMANTIC_PROCESS_PER_CASE=KEEP
CASE_STDOUT_STDERR_PRIVATE=KEEP
EXECUTOR_PRESENTATION_NEUTRAL=KEEP
GLOBAL_CASE_ID=NOT_ADDED
PUBLIC_EVENT_PROTOCOL=NOT_ADDED
```

No owner-approved invariant is reopened or narrowed.

```text
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Implementation consequence

D174 authorizes a bounded Test Tool implementation owner to:

1. introduce presentation-neutral Case start/terminal lifecycle observations;
2. carry invocation-local inert Case reference data sufficient for reporting;
3. expose lifecycle transitions from both legacy and suite-native execution
   paths;
4. maintain in-flight state in reporting/progress machinery;
5. preserve current compact reporting without mandatory per-Case writes;
6. add focused regression coverage proving a hung/incomplete Case becomes
   identifiable as in-flight.

The implementation owner must not silently add any deliberately deferred
facility.

The exact internal representation — closures, small immutable values, callbacks,
a private observer protocol or another equivalent mechanism — remains an
implementation choice so long as it preserves the selected boundary and does not
create a public/general event institution.

## Approval provenance

The exact Candidate C′ boundary was presented to the project owner in the active
interaction on 2026-09-19.

The project owner explicitly approved it:

```text
aprobada
```

```text
D174_STATUS=RATIFIED
SELECTED_CANDIDATE=C_PRIME
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
DURABLE_RECORD_DECISION=REQUIRED

CASE_STARTED_LIFECYCLE=SELECTED
CASE_TERMINAL_LIFECYCLE=SELECTED
CASE_REF=INVOCATION_LOCAL_INERT
GLOBAL_CASE_ID=NOT_ADDED

SCHEDULER_PROGRESS_IO=NO
REPORTER_OWNS_PRESENTATION=YES
MANDATORY_PER_CASE_TERMINAL_WRITE=NO
COALESCING_RATE_LIMITING_POLICY=DEFERRED

LEGACY_AND_SUITE_NATIVE_COMMON_CONCEPTUAL_BOUNDARY=YES

TIMEOUT_POLICY=DEFERRED
SLOW_TEST_POLICY=DEFERRED
RETRIES=DEFERRED
PUBLIC_EVENT_PROTOCOL=DEFERRED
PERSISTENT_EVENT_HISTORY=DEFERRED
TERMINAL_UI_FRAMEWORK=DEFERRED
REMOTE_REPORTING=DEFERRED
```
