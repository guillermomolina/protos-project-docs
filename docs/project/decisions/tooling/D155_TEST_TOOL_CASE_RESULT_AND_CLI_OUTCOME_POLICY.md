# D155 — Suite-native Test Tool CaseResult and CLI outcome policy

Status: **RATIFIED — Candidate A′ selected**

Allocated: **2026-09-19**

Explicit project-owner approval: **2026-09-19**

Decision issue: `guillermomolina/protos#615`

Owning implementation work: TOOL009 / `guillermomolina/protos#600`

Upstream architecture: D152 / `guillermomolina/protos#595`

Discovery/rematerialization: D153 / `guillermomolina/protos#596`

Ratification review baseline:
`guillermomolina/protos@fcd3d8f9d489ae5210ef2076037dfaa074f7654b`

Nature: implementation-independent Test Tool result and CLI outcome policy.

Normative language effect: **none**.

## Decision

D155 selects **Candidate A′ — TERMINAL_STATE_SELF_ASSERTING**.

A suite-native logical Case is self-asserting. Its body may return any ordinary
Protos value; that value is retained as execution observation but is not itself
a pass/fail verdict.

The Test Tool classifies one already-executed logical Case from the execution
phase plus the terminal execution state:

```text
CASE_EXECUTION + COMPLETED
    => PASS

CASE_EXECUTION + FAILED
    => FAIL

CASE_EXECUTION + CANCELLED
    => TOOL_ERROR

REMATERIALIZATION_ERROR
    => TOOL_ERROR
```

An uncaught guest Error during the selected body is therefore an ordinary failed
Case. The runner does not give `AssertionFailure` or another assertion
prototype privileged result-classification semantics.

A test that deliberately catches/observes an expected Error and then completes
normally passes. This is important because ordinary `std:test/Assertions`
operations may return an observed Error as an ordinary value.

## Exact selected contract

```text
D155_SELECTED_CANDIDATE=
    A_PRIME_TERMINAL_STATE_SELF_ASSERTING

CASE_EXECUTION_COMPLETED=PASS
CASE_EXECUTION_FAILED=FAIL
CASE_EXECUTION_CANCELLED=TOOL_ERROR
REMATERIALIZATION_ERROR=TOOL_ERROR

COMPLETED_BODY_VALUE_IS_VERDICT=NO
COMPLETED_BODY_VALUE_RETAINED_AS_OBSERVATION=YES

ASSERTION_FAILURE_SPECIAL_RUNNER_CLASSIFICATION=NO
UNCAUGHT_GUEST_ERROR=FAIL
EXPECTED_ERROR_CAUGHT_BY_CASE=ORDINARY_COMPLETION

TOOL_ERROR_BECOMES_GUEST_FAILURE=NO
TOOL_ERROR_FAIL_FAST=NO_INITIAL
UNRELATED_ALREADY_PLANNED_CASES_CONTINUE=YES

INVOCATION_NO_TOOL_ERROR_ALL_PASS_EXIT=0
INVOCATION_NO_TOOL_ERROR_SOME_FAIL_EXIT=1
INVOCATION_ANY_TOOL_ERROR_EXIT=3

INITIAL_OUTER_STATUS_FOR_EXIT_0_OR_1=completed
INITIAL_OUTER_STATUS_FOR_EXIT_3=infrastructure-aborted
```

The outer status compatibility above does not require the replacement path to
retain D108's historical internal resource/cutover payload or scheduling model.

## Approval provenance

The project owner explicitly approved D155-A′ in the active decision interaction
on 2026-09-19.

The exact approval was:

> **`apruebo D155-A′`**

The antecedent was explicit and singular: D155-A′ —
`TERMINAL_STATE_SELF_ASSERTING`.

```text
DECISION_APPROVAL_PROVENANCE=PASS
```

## Why body return values are not verdicts

LIB018's canonical Test value preserves ordinary body invocation. It did not
select a public TestResult, Boolean verdict, assertion-status object, or runner
protocol encoded in the returned value.

That matters for self-asserting tests. A successful Case may legitimately return:

- `null`;
- an Integer, String, Array or arbitrary object;
- the value produced by the operation being tested;
- an Error value that the test intentionally caught and inspected.

Interpreting one of those ordinary values as a Test Tool verdict would add an
implicit testing protocol that neither LIB018 nor D152 selected.

D155 therefore uses terminal execution behavior as the minimal result authority.

## Guest failure versus Tool error

D155 preserves a strict distinction:

```text
selected body begins execution
    + uncaught guest Error
        => FAIL

Tool cannot faithfully rematerialize/select/execute the authoritative Case
    => TOOL_ERROR
```

A guest failure is evidence about the program/test body.

A Tool error is evidence that the Test Tool could not faithfully perform the
planned Case execution.

D153's rematerialization mismatch remains a Tool error and does not invoke the
selected body.

Cancellation is initially a Tool error rather than a failed assertion because
D155 does not select timeout, retry, user-cancellation or another semantic reason
for cancellation.

## No assertion-framework knowledge in the runner

`std:test/Assertions` remains an ordinary Standard Library authoring facility.

The runner does not inspect whether the Error was produced by
`Assertions.require`, `Assertions.signals`, a user-defined assertion helper,
or ordinary program code.

This preserves D152's test-neutral lower execution boundary and avoids making one
Error prototype a hidden runner ABI.

## Invocation aggregation

Invocation outcome is deterministic over the logical Case results, not physical
completion timing.

Initially:

```text
no Tool error and every selected Case passes
    => status "completed"
    => exit 0

no Tool error and at least one selected Case fails
    => status "completed"
    => exit 1

one or more Tool errors
    => status "infrastructure-aborted"
    => exit 3
```

Tool errors do **not** initially create a fail-fast barrier. Unrelated
already-planned Cases may continue and produce evidence.

The final invocation still reports in logical plan order as required by D152;
physical completion order remains non-semantic.

## Relationship to current TOOL002 outcome policy

The incumbent path currently distinguishes ordinary completed test runs from
`infrastructure-aborted` and maps healthy/failed completed invocations to exit
0/1 and infrastructure abort to exit 3.

D155 retains those outer categories during migration so the public command does
not acquire a gratuitous new exit/status vocabulary.

This is compatibility at the outer command boundary only.

The replacement runner is not required to retain:

- D108 run envelopes;
- legacy manifest expectation kinds;
- historical resource reservation evidence;
- cutover-not-admitted payloads;
- historical SuiteGraph/corpus scheduling topology.

Those remain incumbent-path machinery until their owning tests migrate.

## Migration ownership

D152 remains authoritative:

```text
not migrated -> incumbent path
migrated     -> replacement path
MIGRATION_DOUBLE_EXECUTION=NO
```

When a source becomes suite-native production-owned, the same bounded change
must remove that source from incumbent ownership.

D155 does not authorize shadow execution for parity.

## Comparative evidence

The decision packet compared materially different testing models including:

- JUnit/Jupiter;
- Go `testing`;
- ExUnit;
- Swift Testing;
- Rust/libtest;
- current Protos `std:test` / LIB018.

The relevant transferable distinction is narrow: ordinary body return values are
not generally used as an implicit universal verdict, while an uncaught
test-execution failure is observable failure evidence.

Rust's result-capable test termination was considered as a materially different
model, but adopting such a protocol in Protos would require selecting a public
or implicit TestResult/Termination-style contract that current Protos does not
need.

## Rejected candidates

### B — AssertionFailure-only

Rejected because it would make the runner depend on one assertion Error
prototype and would classify another uncaught guest Error as Tool failure rather
than as failure of the selected Case.

### C — returned verdict

Rejected because it would implicitly select a Boolean/TestResult-style body
contract absent from LIB018 and would make ordinary successful returned values
semantically significant to the runner.

### D — legacy expectation adapter

Rejected because it would retain the external expectation/sidecar ownership
model that suite-native self-asserting Cases are intended to replace.

## Deliberately deferred

D155 does not select:

- fixtures or hooks;
- skip/todo semantics;
- soft assertions;
- timeout or retry;
- user-requested cancellation semantics;
- CaseId serialization;
- Case execution-environment protocol;
- a public CaseResult value/API;
- richer result enum names beyond the contract needed by the Tool;
- remote execution;
- detailed presentation/report formatting.

A later demonstrated need may add those without redefining the selected
completion/failure distinction unless it explicitly reopens D155.

## Invariant/delta consistency

D155-A′ was checked against the explicit D152/D153 invariants, LIB018 behavior
and the current Protos ratification-review baseline.

The public-main delta from
`82cc94664be79a3aac121b0babb70cf82b1c4284` to
`fcd3d8f9d489ae5210ef2076037dfaa074f7654b` contains only:

- `AGENTS.md` governance reconciliation;
- `CONTRIBUTING.md` governance reconciliation;
- `spec/PROTOS_SPEC_CHANGELOG.md` metadata/governance clarification.

No Test Tool implementation, D152/D153 contract, LIB018 Test semantics, or
current Test Tool result projection changed in that delta.

```text
D152_LOGICAL_CASE_RESULT_UNIT=PASS
D152_REPORT_ORDER=PASS
D152_COMPLETION_ORDER_NON_SEMANTIC=PASS
D152_TEST_NEUTRAL_EXECUTOR=PASS
D152_MIGRATION_DOUBLE_EXECUTION_NO=PASS
D152_CURRENT_PATH_STAYS_GREEN=PASS

D153_REMATERIALIZATION_ERROR_DISTINCT=PASS
D153_MISMATCH_INVOKES_BODY_NO=PASS
D153_SILENT_REDISCOVERY_NO=PASS

LIB018_ORDINARY_BODY_RESULT_NOT_VERDICT=PASS
STD_TEST_ASSERTIONS_REMAIN_ORDINARY_LIBRARY=PASS

CURRENT_OUTER_EXIT_0_1_3_COMPATIBILITY=PASS
NEW_CASE_ENVIRONMENT_SEMANTICS=NO
NEW_FIXTURE_RETRY_TIMEOUT_SEMANTICS=NO
NEW_CASEID_POLICY=NO

NEW_UNSURFACED_ARCHITECTURAL_CONSEQUENCE=NO
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Closure contract

```text
D155_STATUS=RATIFIED
D155_SELECTED_CANDIDATE=
    A_PRIME_TERMINAL_STATE_SELF_ASSERTING

PROTOS_REVISION=fcd3d8f9d489ae5210ef2076037dfaa074f7654b
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs

SPECIFICATION_CHANGED=NO
LANGUAGE_SEMANTICS_CHANGED=NO
STANDARD_LIBRARY_SEMANTICS_CHANGED=NO
EXECUTABLE_IMPLEMENTATION_CHANGED=NO
IMPLEMENTATION_VERSION_CHANGED=NO

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

D155 ratification is governance/documentation-only. TOOL009 may resume its
production migration slices against this result policy after the durable
publication is verified.
