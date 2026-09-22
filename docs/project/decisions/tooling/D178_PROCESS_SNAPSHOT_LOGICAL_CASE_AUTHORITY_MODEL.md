# D178 — Process Snapshot Logical Case authority model

Status: **RATIFIED — Candidate A selected**
Allocated: **2026-09-22**
Explicit project-owner approval: **2026-09-22**
Decision issue: `guillermomolina/protos#689`
Parent architecture: D152 / `guillermomolina/protos#595`
Discovery/rematerialization: D153 / `guillermomolina/protos#596`
Execution-requirement transport: TOOL009-D / `guillermomolina/protos#687`
Process Snapshot facility: TOOL009-E / `guillermomolina/protos#688`
Investigation evidence:
`guillermomolina/protos-project-docs:docs/project/evidence/D178/D178_INVESTIGATION.md`
Investigation evidence revision:
`36639ebe41c2b9310946ebd252023cefa84eadf4`
Nature: implementation-independent Test Tool decision.
Normative language effect: **none**.

## Decision

D178 selects **Candidate A — TEST_BODY_AUTHORITY**.

A suite-native Logical Case has one execution authority: the selected
`std:test/Test` invocation. `protos/test/process-snapshot` is a specialized
execution environment, not a specialized Case model.

```
Logical Case
    |
    v
select named Test
    |
    v
invoke Test.call()
    |
    v
Case completion
```

Process Snapshot authority remains underneath that boundary:

```
Logical Case
    |
    v
ExecutionRequirementId = protos/test/process-snapshot
    |
    v
fresh Process Snapshot bootstrap
    |
    v
selected Test.call()
    |
    v
Case completion
```

The existing D135 Process Snapshot bootstrap remains authoritative for
Process-local state, including `process.args()` and `process.environment()`.

## Exact selected contract

```
D178_SELECTED_CANDIDATE=A_TEST_BODY_AUTHORITY
CASE_AUTHORITY=SELECTED_TEST_CALL_COMPLETION
DISCOVERY_EXECUTES_TEST_BODY=NO
TEST_BODY_EXECUTES_EXACTLY_ONCE=YES
PROCESS_PER_CASE=FRESH
PROCESS_ARGS_SEMANTICS=UNCHANGED
PROCESS_ENVIRONMENT_SEMANTICS=UNCHANGED
D152=UNCHANGED
D153=UNCHANGED
TOOL009-D=UNCHANGED
CASE_AUTHORITY_EXTENSION=NO
FINAL_MIGRATION_SCARS=NO
```

## Candidate B rejected / disqualified

Candidate B would preserve whole-module Process Snapshot completion as Case
authority. That is incompatible with the lazy `std:test/Test` contract:
constructing/freezing a Test does not invoke its body. A failing Test body can
therefore remain unexecuted while the source module completes normally.

B would retain two Case semantics:

```
ordinary suite-native Case -> Test.call()
process-snapshot Case      -> whole-module completion
```

That is a final migration scar, directly contrary to D152
`FINAL_MIGRATION_SCARS=NO`. It also requires a second authority/expectation
mechanism to avoid false-positive Cases.

Therefore B is **disqualified**, not an equivalent alternative.

## D152 / D153 consistency

D152 invariants preserved:

```
LOGICAL_UNIT=NAMED_INDEPENDENTLY_EXECUTABLE_CASE
FILE_IS_CASE_ID=NO
SOURCE_CAN_HAVE_N_CASES=YES
ISOLATION_DEFAULT=FRESH_SEMANTIC_PROCESS_PER_CASE
ASSERTIONS=ORDINARY_std:test_LIBRARY
MIGRATION_DOUBLE_EXECUTION=NO
FINAL_MIGRATION_SCARS=NO
```

D153 execution contract preserved:

1. reconstruct the source declaration in a fresh semantic Process;
2. reproduce the discovery signature;
3. resolve exactly one selected local selector;
4. invoke exactly that selected body.

Discovery remains observational and no live Test body crosses the discovery or
CasePlan boundary.

## Process Snapshot semantics preserved

D178 does not alter the normative Process I/O contract. The existing bootstrap
continues to provide a fresh Process per Logical Case, stable
`process.args()` / `process.environment()` snapshot semantics, and the
existing Process-local bootstrap state used by the Process corpus.

D178 changes only which guest execution completion constitutes Logical Case
completion.

## Migration consequence

The 15 remaining Process corpus fixtures can migrate into the suite-native Test
model by moving their existing Process assertions/operations into selected Test
bodies while retaining the existing Process Snapshot bootstrap.

The migration must not execute legacy whole-file behavior in parallel with the
selected Test body.

## Approval provenance

The project owner explicitly approved Candidate A in the active decision
interaction on 2026-09-22.

Exact approval:

> `ok pues apruebo A, deja constancia durable y actualiza el caso`

```
DECISION_APPROVAL_PROVENANCE=PASS
```

## Invariant / delta consistency

```
D152_LOGICAL_CASE_UNIT=PASS
D152_FRESH_PROCESS_PER_CASE=PASS
D152_NO_INTENTIONAL_DOUBLE_EXECUTION=PASS
D152_FINAL_MIGRATION_SCARS=PASS
D153_AUTHORITY_FREE_DISCOVERY=PASS
D153_SIGNATURE_RECONSTRUCTION=PASS
D153_EXACTLY_ONE_SELECTED_BODY=PASS
D153_MISMATCH_DOES_NOT_INVOKE_BODY=PASS
TOOL009-D_EXECUTION_REQUIREMENT_SEPARATE_FROM_CASE_AUTHORITY=PASS
PROCESS_ARGS_SEMANTICS_UNCHANGED=PASS
PROCESS_ENVIRONMENT_SEMANTICS_UNCHANGED=PASS
NEW_CASE_AUTHORITY_ABSTRACTION=NO
LANGUAGE_SEMANTICS_CHANGED=NO
STANDARD_LIBRARY_SEMANTICS_CHANGED=NO
```

## Deliberately unchanged / deferred

No change to Actor, Group, CaseAuthority, CaseId serialization, fixtures/hooks,
timeout/retry, cancellation semantics, public TestResult/CaseResult APIs,
remote/distributed execution, Process Snapshot bootstrap, D152, or D153.

## Ratification state

```
D178_STATUS=RATIFIED
D178_SELECTED_CANDIDATE=A_TEST_BODY_AUTHORITY
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
REQUIRED_DURABLE_PUBLICATION=PASS
```
