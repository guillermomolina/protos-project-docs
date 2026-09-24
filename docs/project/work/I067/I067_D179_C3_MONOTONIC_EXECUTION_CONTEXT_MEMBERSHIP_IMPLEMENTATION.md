# I067 — D179 C3 monotonic execution-context membership implementation

Status: **COMPLETE**

This record is durable, non-normative implementation evidence for
`guillermomolina/protos#707`. Observable Protos semantics remain owned by the
normative specification in `guillermomolina/protos`.

## Authority and input decision

I067 implements the ratified D179 execution-context boundary:

```text
D179_SELECTED_CANDIDATE=C3
D179_SELECTED_CANDIDATE_NAME=MONOTONIC_CONTEXT_MEMBERSHIP
C0_STATUS=DEFER_RECONSIDER_LATER
D179_PROJECT_RECORD_REVISION=9a7648181b70733fc0bb97940cd1e25482b38fd6
```

Ratified transition set:

```text
EXECUTION_CONTEXT_ABSENT_TO_PRESENT=ALLOWED_WHILE_OPEN
EXECUTION_CONTEXT_PRESENT_TO_PRESENT=ALLOWED_WHILE_WRITABLE
EXECUTION_CONTEXT_PRESENT_TO_ABSENT=REJECTED
ORDINARY_OBJECT_REMOVE_SLOT=UNCHANGED
```

## Published product revision

```text
PRODUCT_REVISION=75231601e458930684d4b619dd5f4722377aa65d
COMMIT_MESSAGE=Implement D179 C3 monotonic execution-context membership
MAVEN_VERSION=0.3.80-SNAPSHOT
SPEC_CHANGELOG_VERSION=0.1.433
```

The published delta adds
`ProtosExecutionContextValue extends ProtosObjectValue` and routes
`ProtosPrelude.newExecutionContext()` through that runtime family.
`removeLocalSlot(name)` rejects removal when the named local slot is PRESENT.
Missing-slot removal continues through the ordinary base-object path.
Ordinary non-execution-context `ProtosObjectValue.removeLocalSlot` behavior is
unchanged.

The implementation deliberately does not introduce lexical-versus-dynamic slot
provenance, hidden UNBOUND/tombstone state, backend/profile-dependent semantics,
PLAT036 lexical representation changes, or C0 removal support.

## Specification convergence

Product revision `75231601e458930684d4b619dd5f4722377aa65d` updates:

- `spec/semantics/EXECUTION_AND_CONTROL.md` with monotonic local-slot
  membership for genuine execution contexts;
- `spec/semantics/OBJECT_MODEL.md` with the execution-context specialization
  of `removeSlot(name)`; and
- `spec/PROTOS_SPEC_CHANGELOG.md` at `0.1.433`.

Preserved behavior:

```text
LATE_ABSENT_TO_PRESENT_CREATION=YES
PRESENT_TO_PRESENT_VALUE_MUTATION=YES
LATE_NEARER_CREATION_RETARGETING=YES
CAPTURE_BY_REFERENCE=YES
CONTEXT_ESCAPE=YES
CLOSE_FREEZE_EXISTING_SEMANTICS=YES
PRESENT_NULL_DISTINCT_FROM_ABSENT=YES
ORDINARY_OBJECT_REMOVE_SLOT=YES
```

## Error semantics

No new public error kind, payload, selector, or failure timing was introduced.

The existing `ProtosStandardObjectProtocol.removeSlot` path converts
`IllegalStateException` from `removeLocalSlot(...)` into the ordinary
structural-mutation guest `Error`. The execution-context specialization uses
that same existing path.

```text
ERROR_SEMANTICS=EXISTING_RULE_REUSED
NEW_ERROR_FAMILY=NO
NEW_PUBLIC_SELECTOR=NO
```

## Conformance evidence

The product revision adds focused Java and guest-Protos coverage for:

- rejection of removal of a PRESENT execution-context slot;
- preservation of that slot and value after rejection;
- unchanged ordinary-object `removeSlot`;
- `ABSENT -> PRESENT` creation while OPEN;
- `PRESENT -> PRESENT` mutation while writable;
- captured-context observation of later slot creation;
- late-nearer creation retargeting subsequent lexical lookup;
- escaped-context reflection;
- close/freeze behavior;
- PRESENT `null` remaining distinct from ABSENT; and
- execution-context factory wiring.

No pre-existing repository test was found that depended on successful
`context.removeSlot(existingName)`.

## Validation evidence

The maintainer executed the requested validation and reported all requested
checks/tests green.

Requested sequence:

```text
make compile

mvn -q -Dtest=ProtosExecutionContextValueTest,ProtosPreludeTest,ProtosObjectValueTest test

mvn -q package -DskipTests
bin/protos test --jobs 4 protos/tests/conformance/execution-context

make test
```

Durable result:

```text
VALIDATION_EXECUTOR=HUMAN_MAINTAINER
COMPILE_CHECK=PASS_REPORTED
FOCUSED_JAVA_TESTS=PASS_REPORTED
FOCUSED_PROTOS_CONFORMANCE=PASS_REPORTED
FULL_INTEGRATED_SUITE=PASS_REPORTED
FINAL_REQUIRED_VALIDATION=PASS
```

No raw validation log is stored here; PASS status records the maintainer's
reported execution result.

## Compatibility consequence

Guest code that previously removed a PRESENT local slot from a genuine execution
context and relied on subsequent lookup exposing an outer or receiver binding now
receives the ordinary structural-mutation `Error` instead. The slot remains
present with its prior value.

Repository inspection during I067 found no current product, library, tool, or
pre-existing conformance use relying on that superseded behavior.

Ordinary object structural removal remains unchanged.

## PLAT036 handoff

I067 does not implement PLAT036.

```text
PLAT036_PRODUCT_BASELINE=75231601e458930684d4b619dd5f4722377aa65d
D179_SELECTED_CANDIDATE=C3
C0_STATUS=DEFER_RECONSIDER_LATER
```

PLAT036 must rebuild or explicitly revalidate its candidate architecture against
this implemented C3 baseline rather than automatically restoring the pre-D179
recommendation.

The investigation must distinguish what C3 actually removes from the runtime
architecture from what remains necessary because `ABSENT -> PRESENT` late
creation is still valid.

## Closure state

```text
I067_RESULT=COMPLETE
PRODUCT_REVISION=75231601e458930684d4b619dd5f4722377aa65d

SPEC_C3_MONOTONIC_CONTEXT_MEMBERSHIP=PASS
RUNTIME_C3_MONOTONIC_CONTEXT_MEMBERSHIP=PASS

EXECUTION_CONTEXT_ABSENT_TO_PRESENT=PASS
EXECUTION_CONTEXT_PRESENT_TO_PRESENT=PASS
EXECUTION_CONTEXT_PRESENT_TO_ABSENT_REJECTED=PASS

ORDINARY_OBJECT_REMOVE_SLOT_PRESERVED=PASS
LATE_CONTEXT_CREATION_PRESERVED=PASS
CAPTURE_BY_REFERENCE_PRESERVED=PASS
ESCAPED_CONTEXT_REFLECTION_PRESERVED=PASS
CLOSE_FREEZE_PRESERVED=PASS
PRESENT_NULL_DISTINCTION_PRESERVED=PASS

ERROR_SEMANTICS=EXISTING_RULE_REUSED

FOCAL_VALIDATION=PASS_REPORTED
FINAL_REQUIRED_VALIDATION=PASS_REPORTED

D179_SELECTED_CANDIDATE=C3
C0_STATUS=DEFER_RECONSIDER_LATER

PLAT036_IMPLEMENTED_BY_I067=NO
PLAT036_READY_AFTER_I067=YES
```
