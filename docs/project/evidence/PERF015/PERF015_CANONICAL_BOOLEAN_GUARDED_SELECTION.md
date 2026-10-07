# PERF015 — Canonical Boolean guarded represented-selection closure evidence

Date: 2026-09-29

## Identity

```text
WORK_ITEM=PERF015/#726
PARENT=PERF010-B/#722
PRODUCT_REPOSITORY=guillermomolina/protos

BASE_REVISION=2e3f56fae3a500d3e4193e3345d8a82c35e4590e
PROTOS_REVISION=453f2b00ae2bd0af1cd2761474549e85ff1cb8b6
PROTOS_VERSION=0.3.118-SNAPSHOT
COMMIT=PERF015: admit canonical Boolean to guarded structured selection
```

This record retains the implementation, validation, and closure evidence for
PERF015. It is non-normative project evidence; D013 selection/delegation and the
existing Boolean protocol remain the semantic authority.

## Campaign routing

PERF014 closed with structural success and a falsified timing prediction.
PERF010-B then performed the required causal reconciliation and explicitly
authorized PERF015:

```text
POST_PERF014_RECONCILIATION=PASS
BOOLEAN_GAP=CONFIRMED
INTEGER_GAP=CONFIRMED
I072_E_BOOLEAN_MACHINERY=STILL_CORRECT_REUSE_TARGET
NEW_PRE_STEP3_BLOCKER=NONE
PERF015_AUTHORIZED=YES
```

PERF015 therefore owns only Step 3A: canonical Boolean represented-selection
admission. PERF016/#727 remains the separate Integer Step 3B and combined
post-Step-3 timing owner.

## Published implementation

Before PERF015, canonical `true` and `false` are represented values. The
ordinary I072-A guarded lookup rejects represented receiver steps, so canonical
Boolean control reaches the already-existing I072-E structured machinery only
after generic represented-value selection.

The published implementation adds a narrowly bounded guarded lookup:

```text
ProtosValueLookup.lookupGuardedCanonicalBoolean(...)
```

Admission is restricted to the two exact canonical identities:

```text
ProtosBooleanValue.TRUE
ProtosBooleanValue.FALSE
```

and requires their represented delegation parent to be the canonical root
Object. The lookup then runs the same D013 traversal as the generic path while
exempting only the represented receiver's own immutable canonical-Boolean step
from guarded-lookup invalidation. Ordinary objects visited after that step still
participate in the selector-specific lookup dependency.

`PrepareSendArguments.createGuardedStructuredSend(...)` selects this guarded
lookup only for canonical Boolean receivers. After lookup, admission additionally
requires the existing exact canonical Boolean behavior/home classifier:

```text
ProtosStandardBooleanProtocol.structuredCallbackKindForCanonicalSelection(...)
```

Only the existing canonical callbacks are admitted:

```text
ifTrue
ifFalse
ifTrueIfFalse
and
or
```

Any different selected behavior, noncanonical provenance, unsupported receiver,
or other selector returns to the exact generic path. The implementation does not
introduce primitive `if`, truthiness, a second Boolean-control mechanism, or
selector-name authority independent of D013 selection.

## Structural result

The valid-hit shape is now:

```text
canonical true / false
  -> representation-aware guarded D013 lookup
  -> exact canonical selected Boolean behavior/home
  -> existing I072-E structured Boolean machinery
  -> cached guarded hit while the lookup assumption remains valid
```

The generic represented-selection path is no longer required on a valid
canonical Boolean structured hit.

```text
REPRESENTED_BOOLEAN_FAST_PATH_COMPLETE=YES
CANONICAL_BOOLEAN_REPRESENTED_GUARD=PASS
BOOLEAN_STANDARD_BEHAVIOR_PROVENANCE=PASS
BOOLEAN_SELECTION_INVALIDATION=PASS
CANONICAL_BOOLEAN_VALID_HIT_GENERIC_SELECTION=NO
I072_E_STRUCTURED_BOOLEAN_RETAINED=YES
NO_STANDARD_PROTOCOL_PRIVILEGING=PASS
GENERIC_FALLBACK=PASS
SEMANTIC_CHANGE=NO
```

## Regression evidence

`ProtosI072PhaseEStructuredSendConvergenceTest` now directly proves the
PERF015 boundary.

`canonicalBooleanReceiversAreAdmittedToGuardedStructuredSelection`:

- exercises both canonical `TRUE` and `FALSE`;
- checks `ifTrue`, `ifFalse`, `ifTrueIfFalse`, `and`, and `or`;
- compares guarded selection with the authoritative generic D013 selection;
- requires identical selected Closure and method home;
- requires root Object as the canonical home;
- requires the expected existing I072-E Boolean structured kind;
- requires a valid lookup assumption; and
- verifies unrelated `not` and `ensure` selectors are not admitted as
  canonical Boolean structured selections.

`warmedCanonicalBooleanHitStaysValidOnFrozenStandardSelection` proves the
bootstrapped standard graph is frozen, repeated canonical Boolean control
continues through the warmed site, attempted mutation of the selected root
behavior is rejected, and the guarded selection remains valid.

## Specification and behavior

```text
D013_SELECTION_SEMANTICS=UNCHANGED
D051_BOOLEAN_SEMANTICS=UNCHANGED
I072_E_STRUCTURED_BOOLEAN_MACHINERY=REUSED
TRUTHINESS_INTRODUCED=NO
PRIMITIVE_IF_INTRODUCED=NO
PUBLIC_STANDARD_LIBRARY_CHANGE=NO
PUBLIC_TEST_TOOL_CHANGE=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
NORMATIVE_SPECIFICATION_CHANGE=NO
```

## Timing classification

PERF015 is required for structural implementation completeness independently of
its isolated timing contribution. The published changelog intentionally makes
no timing claim.

```text
PERF015_ISOLATED_TIMING_CLAIM=NONE
FINAL_STEP3_TIMING_OWNER=PERF016/#727
```

The combined Boolean + Integer shared-driver/common-workload timing checkpoint
must therefore be performed only after PERF016 has also established its
represented Integer selection fast path.

## Validation

The maintainer reported the implementation tests passing before publication.

The exact published revision then passed push CI:

```text
CI_RUN_ID=36611854640
CI_HEAD_SHA=453f2b00ae2bd0af1cd2761474549e85ff1cb8b6
CI_CONCLUSION=success

CI_COMMANDS:
  python3 tools/verify_toolchain.py --mode check --scope development
  make test JAVA_TEST_JOBS=4 PROTOS_TEST_JOBS=4

FINAL_REQUIRED_VALIDATION=PASS
```

## Changed product surface

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/runtime/ProtosValueLookup.java
src/test/java/com/guillermomolina/protos/execution/ProtosI072PhaseEStructuredSendConvergenceTest.java
```

Existing APL-1.0 notices are retained on modified Protos-owned source files.

## Closure and next routing

```text
REPRESENTED_BOOLEAN_FAST_PATH_COMPLETE=YES
CANONICAL_BOOLEAN_VALID_HIT_GENERIC_SELECTION=NO
I072_E_STRUCTURED_BOOLEAN_RETAINED=YES
NO_STANDARD_PROTOCOL_PRIVILEGING=PASS
GENERIC_FALLBACK=PASS
FINAL_REQUIRED_VALIDATION=PASS
SEMANTIC_CHANGE=NO

PERF015_STATUS=CLOSED_COMPLETE
NEXT_WORK_ITEM=PERF016/#727
PERF016_NEXT_STATE=READY
STEP4_REMAINS_CONDITIONAL=YES
```
