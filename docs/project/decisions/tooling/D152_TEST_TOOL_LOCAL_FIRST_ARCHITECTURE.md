# D152 — Test Tool architecture after local-first re-evaluation

Status: **RATIFIED — Candidate C selected**

Allocated: **2026-09-18**

Explicit project-owner approval: **2026-09-18**

Decision issue: `guillermomolina/protos#595`

Audit issue: `guillermomolina/protos#594` (AUD014)

Audit baseline: `guillermomolina/protos@f1818da1ad92fdcfeb428087235ea7b29929a190`

Ratification review baseline: `guillermomolina/protos@c98c38838a8b474b29ec71034f18d8615ed1e60e`

Nature: implementation-independent Test Tool architecture.

Normative language effect: **none**.

## Decision

D152 selects **Candidate C — LOCAL_FIRST_FLAT_CASE_PLAN**.

The replacement Test Tool architecture is centered on independently executable
logical cases, a small flat inert plan, one global bounded local scheduler and a
fresh semantic Protos Process for each logical case.

The selected high-level architecture is:

```text
                         protos test
                              |
                       case discovery
                              |
                      flat inert CasePlan
                              |
                     selection/filtering
                              |
                  global --jobs N scheduler
                     work-conserving
                              |
                        N useful lanes
                              |
                 test-neutral execution request
                              |
                  fresh semantic Process/case
                              |
                    private stdout/stderr
                              |
                         CaseResult
                              |
                deterministic presentation
```

This record selects the architecture and observable Tool boundaries, not one
particular scheduler lowering, object layout or host carrier implementation.

## Approval provenance

The project owner explicitly approved D152-C on 2026-09-18 after AUD014 had
published the complete comparative packet and D152 had presented the surviving
high-level alternatives.

Immediately after Candidate C was identified as the pending exact architecture
choice, the owner replied:

> ok , aprobada

The antecedent was D152-C / `LOCAL_FIRST_FLAT_CASE_PLAN`.

```text
DECISION_APPROVAL_PROVENANCE=PASS
```

## Owner invariants preserved

### Local scalability target

The initial meaning of Test Tool scalability is local useful concurrency.

```text
protos test --jobs N
    -> capacity for approximately N useful independent logical cases
    -> when enough CPU-bound work exists, the implementation should be capable
       of making effective use of approximately N CPUs
```

This is not a semantic promise of 100% physical CPU utilization.

The durable scheduler invariant is instead:

```text
READY_INDEPENDENT_CASE_EXISTS
&& RUNNING_CASES < JOBS
    => an unrelated running case must not impose an avoidable admission barrier
```

### Migration ownership

Production tests are not intentionally double-run through old and replacement
systems for parity.

```text
not migrated -> current Test Tool path
migrated     -> replacement path
```

### Current production continuity

The incumbent `protos test` remains operational for CI until the replacement
runner reaches its migration gate.

### No final migration scars

Temporary migration adapters may exist while the replacement is developed, but
the final architecture must not retain historical-only names or topology such as
`Runner2`, `CaseSpecV2`, `NewRunner`, `LegacyExecutor`, `old/`,
`new/` or compatibility-only branches.

## Exact selected contract

```text
D152_SELECTED_CANDIDATE=C_LOCAL_FIRST_FLAT_CASE_PLAN

PUBLIC_COMMAND=protos test

LOGICAL_UNIT=NAMED_INDEPENDENTLY_EXECUTABLE_CASE
FILE_IS_CASE_ID=NO
SOURCE_CAN_HAVE_N_CASES=YES

DISCOVERY_BEFORE_SCHEDULING=YES_INITIAL
PLAN_SHAPE=FLAT_INERT_CASE_PLAN
HIERARCHICAL_RUNNER_SUITEGRAPH=NO_INITIAL

JOBS=GLOBAL_LOGICAL_CASE_CAPACITY
SCHEDULER=WORK_CONSERVING
ROUND_BARRIER=NO
CORPUS_PHASE_BARRIER=NO
REPORT_ORDER=LOGICAL_PLAN_ORDER
COMPLETION_ORDER_IS_SEMANTIC=NO

ISOLATION_DEFAULT=FRESH_SEMANTIC_PROCESS_PER_CASE
STDOUT_STDERR=CASE_PRIVATE
EXECUTOR_KNOWS_TEST_SUITE=NO

CASE_SPECIFIC_ENVIRONMENT=YES_WHEN_DEMONSTRATED
GENERIC_RESOURCE_CATALOG=NO_INITIAL
REMOTE_HA_WORKER_MODEL=NO_INITIAL
RETRY_ATTEMPT_MODEL=NO_INITIAL

ASSERTIONS=ORDINARY_std:test_LIBRARY
LIBRARY_OWNED_RUNNER=NO

MIGRATION_DOUBLE_EXECUTION=NO
CURRENT_PATH_STAYS_GREEN=YES_UNTIL_CUTOVER
FINAL_MIGRATION_SCARS=NO
```

## Logical execution unit

The logical test Case is the scheduling, execution, result and focused-selection
unit.

A physical source file is only a source association / locator domain and may own
or declare zero, one or many logical Cases.

Therefore:

```text
one source file
    -> Case A
    -> Case B
    -> Case C
```

may consume up to three logical scheduler slots if those Cases are independent
and `--jobs` capacity permits.

Moving a Case between source files must not redefine its semantic isolation or
make physical file layout the Test Tool's durable identity model.

## Flat plan boundary

The initial replacement architecture materializes a finite inert flat CasePlan
before scheduling.

This initial choice is sufficient for the current repository scale and gives the
Tool one deterministic authority for:

- duplicate CaseId rejection;
- focal filtering;
- progress count;
- logical result order;
- scheduler admission input.

D152 does not require a recursive runner-owned SuiteGraph.

Suite/group concepts may exist in source authoring and discovery, but they do not
become the execution scheduler's mandatory topology.

A future scale requiring streaming or paged discovery may reopen plan
materialization without changing Case identity or fresh-Process isolation.

## Scheduler contract

`--jobs N` remains an outer logical-case capacity.

The replacement scheduler is work-conserving across all ready independent
selected Cases. Fixed waves/rounds that wait for every sibling before admitting
new unrelated work are not part of the selected architecture.

Deterministic output order is independent from physical completion order.

A possible implementation is N pull lanes built from ordinary `Future.then`
chains followed by one final `Future.all`, but this is feasibility evidence only.
D152 does not standardize that lowering and does not add a Core
`Future.race`/`Future.select` facility.

The existing D069 omitted-`--jobs` default is not changed by D152. Any future
change to that default requires its own explicit decision.

## Fresh Process isolation

D152 explicitly re-evaluates and re-selects one fresh semantic Protos Process per
logical Case as the default isolation unit.

The rationale is architectural rather than historical:

- mutable module/Process state must not leak across independent Cases;
- Process/RootActor semantics can be tested from a clean semantic root;
- arguments/environment/streams/default authority are naturally attempt-private;
- one fatal guest Process failure does not become ordinary shared state for other
  Cases;
- runtime implementation may still share RuntimeHost, immutable compiled
  artifacts and other unobservable machinery below the semantic boundary.

A shared-Process runner was explicitly compared and rejected because strengthening
isolation later would be an observable semantic migration.

## Test-neutral execution boundary

Lower execution machinery does not acquire Test/Suite semantics.

The Test Tool may hand one exact execution request to a host/runtime facility, but
the facility is not responsible for:

- test discovery;
- Suite semantics;
- CaseId policy;
- filtering;
- assertion interpretation;
- result-reporting policy;
- scheduler capacity policy.

The exact replacement execution-request representation remains a later bounded
decision.

## Assertions and result direction

`std:test/Assertions` remains ordinary reusable Standard Library test-authoring
behavior.

The runner is not a second assertion framework.

For suite-native self-asserting Cases, the architectural result distinction is
small:

```text
normal completion
assertion failure
other guest Error
tool/infrastructure inability to execute/classify the case
```

Exact public/internal result names and CLI exit-code mapping are not selected by
D152.

Historical manifest expectation kinds remain valid for the incumbent runner while
that runner owns unmigrated tests. They are not requirements on the final
suite-native replacement architecture.

## Case-specific environment boundary

The audit found a real present requirement for some tests to receive explicit
attempt-private environment/authority, notably current Package Tool project-tree
cases.

D152 therefore preserves:

```text
CASE_SPECIFIC_ENVIRONMENT=YES_WHEN_DEMONSTRATED
```

but does not carry forward the whole generic D076/D077 resource
requirement/catalog/provider/locality architecture as an initial requirement of
the replacement runner.

The distinction is:

```text
demonstrated now:
    explicit bounded per-case execution environment / authority

not demonstrated now:
    generic scarce-resource quantity/conflict/scope/locality scheduling
```

If a real scarce/shared-resource scheduling need appears, it may be added through
a later decision without changing Case identity or the basic execution boundary.

Historical D076/D077 records remain accurate records of the incumbent TOOL002
architecture. D152 does not rewrite history; it selects a different initial
architecture for the replacement Test Tool.

## D151 / TOOL008 compatibility

D151's durable identity distinction survives D152:

```text
--file FILE = invocation-local locator/filter
FILE_IS_CASE_ID=NO
```

TOOL008 was published between the AUD014 research baseline and this ratification
review. Its current incumbent implementation still filters authoritative current
TestPlans before the existing scheduler.

That is compatible with the migration invariant: the incumbent runner remains
production-authoritative until replacement cutover.

In the replacement architecture the durable behavior remains naturally:

```text
protos test --file FILE
    -> select every logical Case associated with that exact file
```

The current SuiteGraph/corpus routing used to implement TOOL008 is not thereby
made part of the replacement architecture.

## Comparative result

AUD014 compared complete architectures, not only API spelling.

The principal survivors were:

- A — current TOOL002 architecture;
- B — reconciled TOOL002;
- C — local-first flat CasePlan;
- D — shared-Process logical tests;
- E — Process-per-source-file.

The current-AGENTS 12-dimension informational scores were:

| Candidate | Score / 60 |
| --- | ---: |
| A — status quo TOOL002 | 39 |
| B — reconciled TOOL002 | 48 |
| **C — local-first flat CasePlan** | **56** |
| D — shared Process | 47 |
| E — Process per source file | 43 |

Candidate C's selection was not based on arithmetic alone.

Its decisive advantages are:

- present-need proportionality;
- logical-case scheduling granularity;
- work-conserving local parallelism;
- fresh semantic isolation;
- simpler identity/routing machinery;
- clean deferral of unused resource/distributed institutions;
- additive future growth without embedding Test/Suite semantics in Java.

The distributed/HA-first candidate was eliminated by the current AGENTS.md
non-compensating present-need/proportionality gate.

## Prior-art evidence

AUD014 surveyed materially different systems including:

- Pharo SUnit;
- Io UnitTest;
- Go `testing`;
- Rust/Cargo/libtest;
- pytest;
- JUnit Platform/Jupiter;
- ExUnit;
- Swift Testing;
- Node's built-in test runner.

The cross-system lesson used by D152 is narrow:

```text
durable useful core:
    stable logical test identity
    independent execution
    focused selection
    bounded concurrency
    clear failure attribution

framework growth begins when the runner also owns:
    lifecycle hierarchies
    fixture injection/scopes
    generic resource graphs
    tags/traits/plugins
    retry/remote-worker policy
```

D152 selects the first group and defers the second until concrete need appears.

## Pay-for-what-you-need and deferral analysis

The replacement architecture pays initially for:

- discovery;
- a flat CasePlan;
- stable Case identity;
- source association;
- exact test-neutral execution requests;
- global jobs capacity;
- work-conserving scheduling;
- fresh Process isolation;
- private output capture;
- small result classification;
- deterministic presentation.

It does not initially pay for:

- remote workers;
- worker-loss uncertainty;
- attempt/retry identity;
- CAS/artifact transport;
- resource locality/scope algebra;
- generic provider profiles;
- resource-catalog CLI;
- fixture-scope hierarchy;
- tags/traits/plugin registries;
- hard timeout/kill semantics.

The audit found zero production `resource-requirements.toml` declarations at the
reviewed Protos baseline, making deferral of the generic resource institution
proportionate to current use.

Future resource, hardened-worker or remote execution support remains possible
behind the case execution boundary; preserving an option does not require
pre-installing its full policy model.

## Migration contract

The selected migration is incremental and single-owner.

```text
M0
    all production tests -> incumbent TOOL002
    replacement -> synthetic/focal self-tests only

M1
    replacement reaches minimum complete runner

M2
    migrate tests gradually
    same bounded change removes each migrated test from old ownership

M3
    old ownership reaches zero
    delete historical runner/resource/expectation paths no longer used

M4
    final migration-scar audit
```

A temporary top-level mixed orchestrator is allowed only as transition
scaffolding. It is not part of the final architecture.

## Retrospective TOOL002 classification

D152 preserves the AUD014 classification.

### KEEP

- `protos test` user command;
- Tool policy primarily in Protos;
- test-neutral lower execution;
- stable logical test identity;
- case-private stdout/stderr;
- deterministic presentation independent of completion order;
- explicit `--jobs N` outer capacity;
- async exact execution as a mechanism;
- guest outcome versus Tool/infrastructure failure distinction;
- bounded explicit case-specific authority/environment where demonstrated.

### ADAPT

- TestPlan -> small flat CasePlan;
- current CaseSpec -> smaller logical Case descriptor;
- exact Source execution -> test-neutral exact execution request;
- progress/reporting -> independent from admission/completion timing;
- CaseAuthority -> bounded execution-environment mechanism for demonstrated
  integration cases;
- current execution-profile routing -> simpler request/environment data where
  possible.

### REMOVE FROM THE REPLACEMENT INITIAL ARCHITECTURE / RECONSIDER LATER

- generic resource requirements/catalog/reservation/provider stack;
- resource scope/locality algebra;
- fixed-H windows and round-wide admission barriers;
- sequential corpus-leaf scheduling boundaries;
- runner-owned explicit Repository SuiteGraph;
- independent SuiteId/CorpusId/ExecutionRequirementId institutions when used only
  to route the current repository;
- remote/hardened/distributed architecture as an initial obligation;
- attempt/variant/run identity not required by local execution;
- historical manifest expectation wrappers/inspectors after the last owning
  legacy test migrates.

### DO NOT INTRODUCE AS THE FUNDAMENTAL MODEL

- Java-owned Test/Suite semantics;
- a mandatory global mutable guest test registry;
- physical file path as logical test identity;
- a Standard Library `Suite.run()` mini-runner competing with `protos test`;
- completion-order semantics;
- permanent migration-only V2/legacy topology.

## Explicitly deferred decisions

D152 does not select:

- exact `std:test/Suite` / Test authoring API;
- exact discovery/rematerialization protocol;
- exact Case carrier fields/spelling;
- exact logical-entry execution mechanism;
- exact case-specific environment descriptor/provider;
- exact result enum names;
- CLI exit-code mapping;
- fixture lifecycle;
- parameterization API;
- timeout/retry;
- remote execution;
- new resource scheduling;
- a new `--jobs` default;
- a future `--case` spelling;
- platform carrier topology.

These remain independent decisions only when their owning work requires them.

## Invariant/delta consistency

Before durable ratification, Candidate C was checked against every explicit owner
invariant and against the repository delta from the AUD014 baseline to the
ratification review baseline.

```text
OI1_LOCAL_USEFUL_CPU_CONCURRENCY=PASS
OI2_NO_MIGRATION_DOUBLE_EXECUTION=PASS
OI3_CURRENT_PRODUCTION_PATH_CONTINUES=PASS
OI4_NO_FINAL_MIGRATION_SCARS=PASS

FRESH_PROCESS_REOPENED_AND_EXPLICITLY_SELECTED=PASS
FILE_IS_CASE_ID=NO=PASS
SOURCE_CAN_HAVE_N_CASES=PASS
D151_TOOL008_COMPATIBILITY=PASS
D069_OMITTED_JOBS_DEFAULT_NOT_CHANGED=PASS

NEW_UNSURFACED_ARCHITECTURAL_CONSEQUENCE=NO
DECISION_INVARIANT_CONSISTENCY=PASS
```

The intervening TOOL008 publication adds exact file-backed selection to the
incumbent runner but does not change any D152 invariant.

## Closure contract

```text
D152_STATUS=RATIFIED
D152_SELECTED_CANDIDATE=C_LOCAL_FIRST_FLAT_CASE_PLAN

PROTOS_REVISION=c98c38838a8b474b29ec71034f18d8615ed1e60e
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs

SPECIFICATION_CHANGED=NO
LANGUAGE_SEMANTICS_CHANGED=NO
STANDARD_LIBRARY_SEMANTICS_CHANGED=NO
EXECUTABLE_IMPLEMENTATION_CHANGED=NO
IMPLEMENTATION_VERSION_CHANGED=NO

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

D152 ratification is governance/documentation-only. Replacement implementation
and the exact discovery/execution-environment decisions remain separate work.
