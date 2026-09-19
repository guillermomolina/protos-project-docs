# D176 — Test Tool stalled in-flight Case presentation policy

Status: **RATIFIED — Candidate G**

Approval date: **2026-09-19**
Decision issue: `guillermomolina/protos#674`
Parent work: TOOL009 / `guillermomolina/protos#600`
Lifecycle authority: D174 / `guillermomolina/protos#673`
Progress authority: D120 / `guillermomolina/protos#455`
Protos revision at ratification: `15b667ce21c1825ffbd7b09c9e3527f21599933e`
D174 implementation evidence: local commit `9ada407669e3ed09526c327170b7501f8aef726c`
Project-record base: `08ae3350a0c439dc4dcd4dcfcf14301260bed93a`

This is a durable non-normative Test Tool presentation decision. It does not
define observable Protos language or Standard Library semantics.

## Decision

D176 selects **Candidate G — one-shot terminal-inactivity snapshot**.

The Test Tool may diagnose a stalled invocation from the D174 lifecycle state
without converting lifecycle observation into mandatory per-Case output.

The selected policy is:

```text
arm:
    at least one Case is in-flight

trigger:
    no CaseTerminal occurs for 30 seconds while at least one Case is in-flight

action:
    emit one diagnostic snapshot to Test Tool stderr

snapshot:
    show at most 8 current in-flight Case display references
    if more Cases are in-flight, collapse the remainder as "+N more"

repeat:
    do not repeat while the same no-terminal-progress episode continues

re-arm:
    only after a later CaseTerminal
    if Cases remain in-flight, a new 30-second episode begins
    if no Cases remain in-flight, the watchdog is disarmed

semantics:
    diagnostic only
    no failure
    no cancellation
    no timeout
    no slow-test classification
    no retry
    no scheduling change
    no result or exit-code change
```

The 30-second interval is a **Test Tool liveness-diagnostic threshold**. It is
not a semantic timeout and does not mean that a Case running for 30 seconds is
incorrect or "slow".

## Exact inactivity clock semantics

To avoid ambiguity when no Case has yet become terminal:

- if the invocation has in-flight Cases and no prior `CaseTerminal` in the
  current episode, the 30-second interval starts when the first `CaseStarted`
  makes the in-flight set non-empty;
- every later `CaseTerminal` ends the current no-terminal-progress episode;
- if at least one Case remains in-flight after that terminal transition, a new
  30-second interval starts from that terminal transition;
- if the in-flight set becomes empty, the watchdog is disarmed;
- additional `CaseStarted` events during an already armed episode do not reset
  the inactivity interval.

Therefore the diagnostic measures **absence of terminal progress for the
invocation**, not age of an individual Case.

## Presentation boundary

The selected diagnostic:

- uses plain UTF-8 line-oriented Test Tool stderr;
- requires no ANSI cursor control;
- requires no spinner;
- requires no TTY detection;
- does not make its exact prose a stable machine-readable protocol;
- does not require one output line per Case;
- emits at most one stall snapshot per no-terminal-progress episode.

The exact rendering of a Case display reference is replaceable presentation.
It must nevertheless be sufficient for a human to identify the in-flight Case
from the current Test Tool invocation.

The bounded list size is **8**. When more than 8 Cases are in-flight, the
reporter must expose the omitted count instead of expanding without bound.

## Relationship to D120

D120 remains authoritative for ordinary Test Tool progress:

- deterministic completion milestones;
- immediate failure reporting;
- phase summaries;
- final aggregate summary;
- bounded line-oriented stderr;
- no TTY/ANSI dependency.

D176 does not replace those rules.

The one-shot stall diagnostic is exceptional liveness reporting layered on top
of D120. It is intentionally wall-clock-triggered because a true hang can
produce no later terminal event from which deterministic completion milestones
could reveal the active Case.

## Relationship to D174

D174 remains authoritative for execution lifecycle:

```text
CaseStarted(caseRef)
CaseTerminal(caseRef, result)
```

D176 consumes the reporter-derived in-flight state from that boundary.

The scheduler remains presentation-neutral:

```text
SCHEDULER_PROGRESS_IO=NO
MANDATORY_PER_CASE_TERMINAL_WRITE=NO
REPORTER_OWNS_PRESENTATION=YES
```

D176 does not add a public event protocol, persistent Case identity or general
listener institution.

## Host watchdog boundary

Current Protos guest code does not expose a general Test Tool clock/timer
facility merely for progress reporting.

D176 therefore authorizes a **Test-Tool-internal host watchdog/reporter
mechanism** to observe elapsed host time and request the bounded diagnostic
snapshot.

That mechanism:

- is internal Test Tool machinery;
- must not expose a new Clock/Timer capability to Protos language code;
- must not own Case scheduling;
- must not complete, cancel, interrupt or otherwise alter Case execution;
- must stop with the Test Tool invocation;
- must consume D174-derived reporter state rather than infer activity from guest
  stdout/stderr.

A later general clock/timer facility remains a separate design question.

## Why this decision was required

D174 solved the architectural observability defect exposed by TOOL009: a Case can
now be known as in-flight before it hangs.

That alone is insufficient operationally. If a Case never reaches
`CaseTerminal`, the Test Tool can hold the correct in-flight identity forever
without displaying it.

D176 therefore owns only the bounded presentation policy needed to answer:

> Which Case is still running when the invocation has stopped making terminal
> progress?

## Prior-art evidence

D176 compared mature test/build runners across different reporting families.

### cargo-nextest

Official documentation:
<https://nexte.st/docs/reporting/>
<https://nexte.st/docs/running/>
<https://nexte.st/docs/configuration/slow-tests/>

nextest separates execution from reporting, can display currently running tests
in bounded interactive progress, and has separately configurable slow-test and
timeout behavior.

Relevant lessons:

- active-work identity can be bounded rather than printed once per test;
- slow-test policy is distinct from general progress/liveness presentation;
- a richer future UI can consume execution state without redefining test
  semantics.

### Go test

Official documentation:
<https://pkg.go.dev/cmd/go#hdr-Test_packages>

Verbose test output exposes individual tests while they execute.

Relevant lesson: per-test start visibility is useful diagnostically, but making
it the default would impose output proportional to Case count. D176 therefore
does not select that model for ordinary Test Tool runs.

### Gradle

Official documentation:
<https://docs.gradle.org/current/userguide/java_testing.html>

Gradle exposes configurable test logging events, including started-test events,
while ordinary output can remain substantially quieter.

Relevant lesson: per-test start logging is appropriately optional/configurable;
it is not required as the default architecture for diagnosing stalls.

### pytest

Official documentation:
<https://docs.pytest.org/en/latest/how-to/output.html>
<https://docs.pytest.org/en/latest/reference/reference.html>

pytest supports different verbosity levels and lifecycle/reporting hooks such as
runtest log-start/report/finish.

Relevant lesson: execution lifecycle and terminal presentation are separate, and
per-test identity can be a presentation choice rather than scheduler behavior.

### Bazel

Official documentation:
<https://bazel.build/reference/command-line-reference>

Bazel applies progress-reporting limits and rate controls rather than expanding
active-work presentation without bound.

Relevant lesson: large concurrent work sets need bounded progress surfaces.

## Candidate result

### Status quo

Rejected.

It retains D174 internal observability but does not solve the motivating
operator problem because a hung Case can remain invisible to the user.

### Candidate A — print every Case start

Rejected.

It gives excellent immediate identity but makes default progress output
proportional to Case count, contradicting the owner-established bounded-output
invariant.

### Candidate B — periodic heartbeat with current in-flight Cases

Rejected as the durable default.

It eventually reveals a hang, but a permanently stalled invocation can emit
unbounded repeated output over wall-clock time.

### Candidate C — per-Case slow/stall threshold

Rejected for this decision.

It couples the diagnostic to individual Case age and therefore introduces a
slow-test policy boundary. D176 needs liveness diagnosis, not a classification
of long-running tests.

### Candidate D — explicit verbose/diagnostic mode only

Not selected as the default solution.

It has excellent proportionality when disabled and remains a plausible future
addition, but it fails the motivating requirement unless the user predicts the
hang and enables the mode before the run.

### Candidate E — bounded snapshots only on existing progress events

Rejected.

It preserves deterministic event-driven output, but if no later terminal event
occurs, the actual hung Case can remain invisible indefinitely.

### Candidate F — manual snapshot trigger only

Rejected as the sole default solution.

It can be useful later, but requires user interaction/anticipation and does not
reliably help non-interactive CI runs.

### Candidate G — one-shot terminal-inactivity snapshot

**Selected.**

It reveals a real no-progress episode automatically while bounding output by
episode rather than by Case count or hang duration.

## Twelve-dimension comparison

Scores are 1–5 and advisory. Selection follows the qualitative trade-off, not
an arithmetic winner.

| Dimension | B — heartbeat | C — per-Case threshold | D — verbose only | G — one-shot inactivity |
| --- | --- | --- | --- | --- |
| Correctness / invariants | 5 HIGH — reveals active work, but repeats | 5 HIGH — identifies long-running Cases | 2 HIGH — default run can still be opaque | 5 HIGH — reveals in-flight work without execution changes |
| Protos alignment | 3 HIGH — presentation driven continuously by time | 3 HIGH — introduces per-Case duration policy | 5 HIGH — simple optional mechanism | 5 HIGH — consumes D174 state and keeps scheduler neutral |
| Present-need proportionality | 3 HIGH — repeated writes during long stalls | 3 HIGH — timing state per Case | 5 HIGH — no cost unless enabled | 5 HIGH — exceptional output only when progress disappears |
| Incremental growth | 5 HIGH — richer heartbeat UI can evolve | 4 HIGH — grows toward timeout/slow infrastructure | 5 HIGH — can add richer modes later | 5 HIGH — richer UI/manual/verbose modes layer on same lifecycle |
| Future-option resilience | 5 HIGH — preserves lifecycle consumers | 4 HIGH — biases toward per-Case timing semantics | 5 HIGH — leaves reporting choices open | 5 HIGH — no public protocol or timeout commitment |
| Scalability | 3 HIGH — output grows with stall duration | 4 HIGH — state scales with in-flight count | 5 HIGH — user opts into O(cases) output | 5 MEDIUM — one bounded snapshot per episode |
| Conceptual simplicity | 3 HIGH — cadence and repeated rendering policy | 3 HIGH — per-Case age/threshold state | 5 HIGH — very simple switch | 4 HIGH — requires one small watchdog state machine |
| Portability / freedom | 4 MEDIUM — requires host timing | 4 HIGH — requires host timing | 5 HIGH — no timing mechanism needed | 5 HIGH — host watchdog only; no TTY or language timer |
| Runtime / resource cost | 3 HIGH — recurring wakeups and possible writes | 4 HIGH — per-Case timing bookkeeping | 5 HIGH — zero when disabled | 5 HIGH — bounded watchdog and exceptional write |
| Failure / operability | 5 HIGH — strong liveness evidence | 5 HIGH — strong Case-age evidence | 3 HIGH — weak unless anticipated | 5 HIGH — automatic diagnosis of the motivating hang |
| Deferral / reversibility | 4 HIGH — easy to replace but noisy meanwhile | 3 HIGH — later changes can affect slow-test expectations | 5 HIGH — easy to add/remove | 5 HIGH — diagnostic-only and replaceable without semantic migration |
| Evidence / implementation risk | 5 HIGH — common reporting pattern | 5 HIGH — mature slow-test precedent | 5 HIGH — ubiquitous verbose pattern | 4 MEDIUM — hybrid policy, composed from mature bounded-active-work and rate-control precedents |

Candidate G is selected because it is the only surviving default policy that
simultaneously:

- solves the hang-diagnosis problem without anticipation;
- remains bounded during an indefinitely hung run;
- does not classify individual tests as slow;
- preserves D120 ordinary progress;
- preserves D174 scheduler/reporting separation.

## Adversarial incremental-design gate

### Pay for what is needed

The present need is automatic identification of in-flight work after the Test
Tool stops making terminal progress.

Candidate G pays only for:

- the already-required D174 lifecycle state;
- one internal watchdog;
- at most one bounded diagnostic per stall episode.

It does not charge every successful Case a terminal write.

### Grow as needed

Later work may independently add:

- a verbose per-Case mode;
- manual current-status dump;
- richer TTY display;
- structured machine reporting;
- true slow-test diagnostics;
- timeout/cancellation policy.

Those features can consume the same D174 lifecycle state. Candidate G does not
pre-build them.

### Cost of deferral

Deferring all in-flight presentation would leave the current TOOL009 hang class
operationally opaque even though D174 already knows the active Case.

Deferring richer output modes has low cost because Candidate G establishes no
public protocol that they must replace.

### Speculation burden

The candidate adds no semantic timer, timeout, retry, history, persistent
identity, JSON schema or terminal framework.

It preserves those future options without installing them.

## Deliberately deferred

D176 does not select:

```text
per-Case timeout
slow-test classification
automatic cancellation
retry
verbose per-Case default
manual signal/input handler
TTY-rich progress UI
ANSI cursor control
JSON/event stream
JUnit/XML reporting
persistent lifecycle history
distributed/remote reporting
public reporter/listener API
global/persistent Case identity
general Protos Clock/Timer capability
```

## Invariant / delta consistency

Candidate G preserves the applicable owner-approved constraints from D120 and
D174.

```text
D120_BOUNDED_STDERR_OUTPUT=KEEP
D120_COMPLETION_MILESTONES=KEEP
D120_IMMEDIATE_FAILURES=KEEP
D120_PHASE_AND_FINAL_SUMMARIES=KEEP
D120_NO_TTY_ANSI_DEPENDENCY=KEEP

D174_CASE_STARTED_TERMINAL_BOUNDARY=KEEP
D174_CASE_REF_INVOCATION_LOCAL_INERT=KEEP
D174_SCHEDULER_PRESENTATION_NEUTRAL=KEEP
D174_REPORTER_OWNS_PRESENTATION=KEEP
D174_MANDATORY_PER_CASE_WRITE=NO

TIMEOUT_POLICY=NOT_ADDED
SLOW_TEST_POLICY=NOT_ADDED
RETRY_POLICY=NOT_ADDED
PUBLIC_EVENT_PROTOCOL=NOT_ADDED
GLOBAL_CASE_ID=NOT_ADDED

DECISION_INVARIANT_CONSISTENCY=PASS
```

## Implementation consequence

D176 authorizes a bounded implementation owner to:

1. add a Test-Tool-internal host watchdog consuming D174-derived reporter state;
2. arm it when the invocation acquires in-flight work;
3. emit one stderr diagnostic after 30 seconds without a `CaseTerminal`;
4. render at most 8 current in-flight Case display references plus an omitted
   count;
5. suppress repeat output until a later `CaseTerminal` re-arms the policy;
6. disarm when no Cases remain in-flight;
7. add focal tests for no-output healthy runs, first-stall emission, bounded
   active list, no repeated emission within one episode, re-arm after terminal
   progress, and unchanged result/scheduling semantics.

Implementation must not silently add any deliberately deferred facility.

## Approval provenance

The exact Candidate G contract was presented to the project owner in the active
interaction on 2026-09-19, including:

- 30 seconds of terminal inactivity;
- one-shot snapshot rather than repeated heartbeat;
- at most 8 in-flight Cases plus omitted count;
- re-arm only after a later `CaseTerminal`;
- diagnostic-only semantics;
- internal host watchdog rather than a language timer facility.

The project owner explicitly approved it:

```text
aprobada
```

```text
D176_STATUS=RATIFIED
SELECTED_CANDIDATE=G
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
DURABLE_RECORD_DECISION=REQUIRED

STALL_TRIGGER=NO_CASE_TERMINAL_FOR_30_SECONDS_WITH_IN_FLIGHT_WORK
STALL_OUTPUT=ONE_SHOT_PER_NO_TERMINAL_PROGRESS_EPISODE
IN_FLIGHT_DISPLAY_LIMIT=8
OMITTED_COUNT=REQUIRED_WHEN_OVER_LIMIT
REARM=AFTER_LATER_CASE_TERMINAL
DISARM=WHEN_IN_FLIGHT_EMPTY

DIAGNOSTIC_ONLY=YES
CASE_FAILURE=NO
CASE_CANCELLATION=NO
TIMEOUT=NO
SLOW_TEST_CLASSIFICATION=NO
SCHEDULING_CHANGE=NO
RESULT_EXIT_CHANGE=NO

OUTPUT_CHANNEL=STDERR
OUTPUT_TEXT=PLAIN_UTF8_LINE_ORIENTED
TTY_ANSI_DEPENDENCY=NO

HOST_INTERNAL_WATCHDOG=AUTHORIZED
LANGUAGE_CLOCK_TIMER_CAPABILITY=NOT_ADDED
```
