# D178 — Process Snapshot Logical Case authority investigation

Evidence owner: D178 — Process Snapshot Logical Case authority model
Decision issue: GitHub #689
Evidence date: 2026-09-22
PROTOS_REVISION: 529ab58c2cf57a2e4170dd5ffa972651e89ac92e

This is immutable, non-normative investigation evidence. It does not ratify or
select D178.

## Executive finding

The repository contains a concrete execution-authority mismatch.

Ordinary suite-native execution discovers an inert tests declaration, rematerializes
the source in a fresh Process, resolves the selected Test, and invokes selected().
The Test invocation therefore supplies Case execution authority.

The current Process Snapshot suite-native facility validates a one-element signature
and matching selector, then delegates the complete raw source to
ProtosProcessSnapshotExecution.execute(...). It does not resolve or invoke a
selected Test. Its Case completion is therefore whole-source completion.

std:test/Test is lazy: Test(name, body) stores a call Closure whose body executes only
when that call Closure is invoked. Constructing or freezing the Test does not invoke
the body.

Therefore this source shape can complete without executing its assertion body under
the current Process Snapshot path:

    tests: [
        Test("case", () => {
            Assertions.require(...)
        })
    ]
    tests.freeze()

This establishes the decisive Candidate-B risk: zero Test-body execution can still
produce normal module completion and therefore a false PASS if module completion is
the Case authority.

## Direct repository evidence

Normative and governance sources inspected:

    AGENTS.md
    AGENTS.work/DESIGN.md
    AGENTS.work/REFERENCE.md
    AGENTS.work/TEST.md
    AGENTS.work/COORDINATION.md
    protos/AGENTS.md
    src/AGENTS.md
    spec/AGENTS.md
    spec/PROTOS_LANGUAGE_SPEC.md
    spec/semantics/CALLABLES.md
    spec/semantics/MODULES.md
    spec/semantics/EXECUTION_AND_CONTROL.md
    spec/io/PROCESS_IO.md

Test Tool / implementation sources inspected:

    protos/lib/test/Test.protos
    protos/tools/test/Discovery.protos
    protos/tools/test/LogicalCaseMigration.protos
    protos/tools/test/Main.protos
    protos/tools/test/RepositoryCorpusPlans.protos
    src/main/java/com/guillermomolina/protos/execution/ProtosTestLogicalCaseAttemptBridge.java
    src/main/java/com/guillermomolina/protos/execution/ProtosTestLogicalCaseExecutionFacility.java
    src/main/java/com/guillermomolina/protos/execution/ProtosTestLogicalCaseDiscoveryFacility.java
    src/main/java/com/guillermomolina/protos/execution/ProtosProcessSnapshotLogicalCaseExecutionFacility.java
    src/main/java/com/guillermomolina/protos/execution/ProtosProcessSnapshotExecution.java
    src/test/java/com/guillermomolina/protos/execution/ProtosProcessSnapshotLogicalCaseExecutionFacilityTest.java
    protos/tests/conformance/process/manifest.tsv
    all 15 current Process corpus fixtures

Key code facts:

    Discovery.declarationSignatureFromModule(module)
        requires module.hasSlot("tests")
        reads Test.name
        does not invoke Test.call()

    Ordinary Logical Case execution
        rematerializes source
        resolves selected Test
        executes selected()

    Process Snapshot Logical Case execution
        validates a one-Test signature/selector shape
        delegates to ProtosProcessSnapshotExecution.execute(source, ...)
        does not invoke Test.call()

    ProtosProcessSnapshotExecution
        creates fresh Process state
        establishes args/environment snapshots
        executes the complete raw source

The existing TOOL009-E focal test proves that Logical Case execution reaches the
D135 Process Snapshot bootstrap and preserves process.args()/process.environment()
semantics. It does not prove selected Test.call() execution.

## Process bootstrap evidence

spec/io/PROCESS_IO.md is the normative owner of Process bootstrap, arguments and
environment.

The specification defines one Protos Process execution domain and stable Process-local
args/environment snapshots. Those semantics are independent of which code inside the
Process constitutes the Test Tool Case authority.

The current D135 implementation creates fresh Process instances and establishes
argument/environment snapshots before guest source execution.

Thus the relevant architectural separation is:

    Process Snapshot = execution environment / host authority
    Test.call()      = suite-native Logical Case execution authority

Preserving the existing Process Snapshot bootstrap does not require preserving
whole-module completion as Case authority.

## Candidate A — Test-body authority

Conceptual path:

    Logical Case
      -> fresh Process Snapshot bootstrap
      -> rematerialize source
      -> validate declaration/signature
      -> resolve selected Test
      -> invoke Test.call()
      -> Case completion derives from Test invocation

Required invariants:

    DISCOVERY_EXECUTES_TEST_BODY = NO
    TEST_BODY_EXECUTES_EXACTLY_ONCE = YES
    CASE_AUTHORITY = SELECTED_TEST_CALL_COMPLETION
    PROCESS_PER_CASE = FRESH
    PROCESS_ARGS_SEMANTICS = UNCHANGED
    PROCESS_ENVIRONMENT_SEMANTICS = UNCHANGED
    D152 = UNCHANGED
    D153 = UNCHANGED
    TOOL009-D = UNCHANGED

The implementation must not execute the whole source as the Case authority and then
invoke the Test again.

## Candidate B — Whole-module authority

Current conceptual path:

    Logical Case
      -> whole Process Snapshot source execution
      -> module completion = Case authority

The direct counterexample is a lazy Test whose body signals failure while the module
only constructs and freezes the Test. The body is never called, so module completion
can incorrectly become Case success.

Possible repairs were examined:

    1. Make Test construction eager.
       This conflicts with the established Test/Closure semantics and discovery purity.

    2. Execute the Test body after module completion.
       This makes Test invocation the real Case authority and collapses toward A.

    3. Duplicate assertions outside Test bodies.
       This creates a second expectation/authority mechanism.

A Process-specific metadata-only Test contract would be a new design, not an
implementation interpretation of D152/D153/LIB018.

## Migration impact

The Process corpus contains:

    PROCESS_MANIFEST_ENTRIES_TOTAL = 15
    PROCESS_ALREADY_SUITE_NATIVE = 0
    PROCESS_MIGRATED = 0
    PROCESS_REMAINING_LEGACY = 15

The 15 fixtures cover arguments, Environment, Process receiver validation, iteration,
and same-Process/different-Process snapshot identity.

Inspection found no fixture that legitimately requires whole-module completion itself
to be the semantic Test result. The existing Process behavior can be moved into a
selected Test body while retaining the established Process bootstrap.

Candidate A therefore preserves the existing Process evidence while aligning Case
authority with the suite-native model.

Candidate B requires a second authority/expectation model to avoid false PASS.

## Comparative prior art

The investigation compared at least five systems spanning three material design
families:

    JUnit Platform
        discovery/TestDescriptor/TestPlan is distinct from later execution

    pytest
        collection is distinct from execution; --collect-only does not execute tests

    Go testing
        named TestXxx functions are executable test units invoked by the harness

    Rust libtest / Cargo test
        #[test] functions are identified by the harness and executed later

    RSpec
        ExampleGroup declaration is distinct from individual example block execution

The transferable invariant is:

    discovery / registration / descriptor construction
        !=
    test-body execution

No compared system provides evidence that constructing a lazy test descriptor is
equivalent to executing its body.

## Comparative conclusion

Candidate A:

    preserves Test/Closure semantics
    preserves D152/D153 discovery purity
    preserves fresh Process isolation
    preserves D135 Process args/environment bootstrap
    uses one Case authority model
    supports clean migration of all 15 Process cases

Candidate B:

    preserves current TOOL009-E source execution
    but leaves selector/Test invocation without execution authority
    has a direct zero-execution / false-PASS counterexample
    requires a second Process-specific test contract to become coherent
    makes future multi-Test-per-source semantics harder to express

## Adversarial findings

False PASS:

    Candidate A: avoidable if selected Test.call() is the sole authority
    Candidate B: YES under the current lazy Test contract

Double execution:

    Candidate A: avoidable; Test body must execute exactly once
    Candidate B: fixing the false PASS by adding Test execution creates a second
                 execution stage and converges toward A

Future pressure against A:

    a future requirement for multiple Cases sharing one Process would conflict with
    D152's fresh-Process-per-Case invariant and would require a new decision.

Future pressure against B:

    multiple named Tests in one source cannot naturally produce independent Case
    results if one whole-module completion is authoritative.

## Recommendation

    RECOMMENDATION = A_TEST_BODY_AUTHORITY
    OWNER_APPROVAL_REQUIRED = YES

The exact approval gate is:

    D178_CANDIDATE_SELECTED = A_TEST_BODY_AUTHORITY

This investigation does not itself ratify the decision.

## Final status

    D178_INVESTIGATION_STATUS = COMPLETE
    D178_OUTCOME = A_TEST_BODY_AUTHORITY
    RECOMMENDATION = A_TEST_BODY_AUTHORITY
    OWNER_APPROVAL_REQUIRED = YES
