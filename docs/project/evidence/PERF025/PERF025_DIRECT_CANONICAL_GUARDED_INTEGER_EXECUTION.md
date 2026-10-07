# PERF025 — Direct canonical guarded Integer execution

## Status

Published product implementation evidence for PERF025 / `guillermomolina/protos#758`.

This record is non-normative. It retains the exact product publication identity,
the structural result, validation provenance, and the coordination boundary with
PERF027 / `guillermomolina/protos#779`.

## Exact product publication

```text
PROTOS_REVISION=51bf02af1fe0fd6247ee17c8a1a764ced77d2732
PROTOS_PARENT_REVISION=bf08d8b4e106e299d3250ac16079767feb3e48a3
PROTOS_VERSION=0.3.157-SNAPSHOT
COMMIT_SUBJECT=PERF025: execute canonical guarded Integer operations directly
OWNING_ISSUE=guillermomolina/protos#758
RELATED_ISSUE=guillermomolina/protos#779
BASE_IS_EXACT_PARENT=YES
```

The product publication is exactly one commit ahead of the preceding PERF025
static indexed Closure-parameter publication.

## Ownership and PERF027 coordination

This is a PERF025 post-F1 pay-as-you-grow structural implementation slice.

It consumes a residual invocation tax identified by the Integer-specific
PERF027 investigation, and it reuses PERF016 guarded Integer selection plus the
PERF027-A deferred native path as its semantic fallback. It does not rename,
replace, or satisfy PERF027-B.

```text
FORMAL_OWNER=PERF025/#758
PERF027_B_STATUS=NOT_EXECUTED
PERF027_B_MEASUREMENT_SATISFIED=NO
PERFORMANCE_MEASUREMENT_IN_THIS_SLICE=NO
INTEGER_REPRESENTATION_EXPERIMENT_AUTHORIZED_BY_THIS_SLICE=NO
```

PERF027-A remains the historical checkpoint where admitted guarded Integer
native sends first stopped eagerly materializing guest Context and guest
argument Array while still retaining a semantic native method Activation.

## Bounded objective

Before this publication, an exact stable standard Integer selection still
entered the ordinary prepared native-call machinery:

```text
guarded Integer D013 selection
  -> selected native Closure + methodHome + Assumption
  -> deferred immediate native-method preparation
  -> ProtosActivation / ReturnHome
  -> PreparedClosureCall.NativeCall
  -> nativeBody.execute(...)
```

This slice removes that residual general native-Closure invocation scaffold only
for successful exact canonical local Integer arithmetic whose complete semantic
selection and operand domain have already been proved.

## Canonical operation authority

`ProtosStandardIntegerProtocol` now installs the six local canonical native
arithmetic operations through one explicit operation family:

```text
+
binary -
*
/
div
mod
```

Each installed operation owns private provenance connecting:

- the exact bootstrapped native Closure;
- the private standard Integer native body;
- the exact Integer prototype method home;
- the selector;
- the current Prelude; and
- the canonical operation kind.

The direct execution classifier therefore does not treat selector spelling as
semantic authority.

```text
EXACT_INSTALLED_CLOSURE_REQUIRED=YES
EXACT_PRIVATE_BODY_PROVENANCE_REQUIRED=YES
EXACT_INTEGER_METHOD_HOME_REQUIRED=YES
CURRENT_PRELUDE_REQUIRED=YES
SELECTOR_MATCH_REQUIRED=YES
D013_GUARDED_SELECTION_REMAINS_AUTHORITY=YES
```

A different Closure wrapping the same native body, a different method home, an
alias spelling, or another unsupported selection does not acquire the direct
capability.

## Guarded direct execution shape

`PrepareSendArguments.GuardedIntegerSend` now retains the proven canonical
operation kind in addition to the selected Closure, method home, and existing
lookup-stability `Assumption`.

For a valid canonical hit with a successful Integer operand domain:

```text
D013 guarded selection
  -> exact canonical standard implementation proof
  -> existing lookup Assumption
  -> canonical arithmetic helper
  -> PreparedClosureCall.ImmediateResultCall
  -> EnterClosureCall immediate result
```

The successful direct shape owns no rich method invocation state:

```text
RICH_METHOD_ACTIVATION=NO
RETURN_HOME=NO
GUEST_ARGUMENT_ARRAY=NO
PREPARED_NATIVE_CALL=NO
NATIVE_BODY_EXECUTION=NO
SOURCE_CALL_TARGET=NO
```

`ImmediateResultCall` is a minimal fourth prepared-call leaf whose only
payload is the already-computed result. It is non-structured, non-native, has no
Task/lifecycle state, enters through the existing `isImmediate()` /
`enterImmediate()` operation, and finishes by identity.

## One arithmetic semantic authority

The fast path does not duplicate Integer arithmetic rules in the Bytecode root.

Both the ordinary canonical native body and the successful direct path delegate
to the same standard Integer arithmetic implementation. This preserves:

- exact unbounded BigInteger arithmetic;
- exact binary64 conversion for `/`;
- quotient/remainder semantics for `div` and `mod`; and
- one implementation authority for successful canonical arithmetic.

```text
DUPLICATE_ARITHMETIC_SEMANTICS=NO
BIGINTEGER_ARITHMETIC_CHANGED=NO
FLOAT_DIVISION_ROUNDING_CHANGED=NO
```

## Error and generic fallback preservation

Direct execution is attempted only when the selected operation and supplied
operands are already inside the successful standard domain.

The following cases deliberately retain the exact PERF027-A deferred native
path:

- wrong arity;
- wrong argument type/domain;
- division or remainder by zero;
- inherited Number operations such as ordering;
- non-canonical selected native Closures; and
- any unsupported guarded selection.

That path still uses the selected Closure and method home and materializes the
rich semantic invocation when Error/control provenance requires it.

```text
WRONG_DOMAIN_DEFERRED_NATIVE_FALLBACK=YES
ZERO_DIVISOR_DEFERRED_NATIVE_FALLBACK=YES
INHERITED_NUMBER_OPERATION_FALLBACK=YES
NONCANONICAL_SELECTION_FALLBACK=YES
ERROR_ACTIVATION_PROVENANCE=PRESERVED
GENERIC_FALLBACK=PRESERVED
```

This bounded slice does not include inherited Number ordering
`< <= > >=`.

## Integer representation unchanged

The product still represents semantic Integer values as:

```text
ProtosIntegerValue(BigInteger)
```

No SmallInteger, tagged Integer, primitive `long` guest carrier, transparent
promotion representation, or boxing-elimination expansion is introduced here.

```text
INTEGER_REPRESENTATION_CHANGED=NO
SMALL_INTEGER_REPRESENTATION_INTRODUCED=NO
BOXING_ELIMINATION_TYPES_CHANGED=NO
```

## Native-boundary contraction

Centralizing the six canonical arithmetic selectors under the audited standard
installer reduces lexical occurrences of
`ProtosClosureValue.nativeClosure(...)` in
`ProtosStandardIntegerProtocol.java` from four to two.

The architecture guard was updated accordingly:

```text
INTEGER_NATIVE_PROVIDER_LEXICAL_COUNT_BEFORE=4
INTEGER_NATIVE_PROVIDER_LEXICAL_COUNT_AFTER=2
TOTAL_CORE_NATIVE_PROVIDER_LEXICAL_COUNT_BEFORE=141
TOTAL_CORE_NATIVE_PROVIDER_LEXICAL_COUNT_AFTER=139
NATIVE_SEMANTIC_SURFACE_EXPANDED=NO
```

This is a contraction of construction sites, not removal of the six standard
native Integer behaviors.

## PLAT040 alignment

The implementation was checked against ratified PLAT040 Candidate F′.

PLAT040 requires D013-authoritative guarded stable selection, exact invalidation
and generic fallback, conditional guest-state materialization, separation of
optional invocation state, and allows standard protocol specialization only
after ordinary semantic selection establishes the expected standard behavior
under an equivalent validity contract.

The same decision explicitly rejects a universal prepared-call carrier on the
hot path.

```text
PLAT040_COMPATIBLE=YES
D013_AUTHORITY_PRESERVED=YES
EXISTING_LOOKUP_ASSUMPTION_REUSED=YES
STANDARD_SELECTOR_PRIVILEGING=NO
UNIVERSAL_RICH_ACTIVATION_HOT_PATH_FOR_CANONICAL_INTEGER=REMOVED
NEW_PLATFORM_DECISION_REQUIRED=NO
```

## Exact product delta

Exact comparison:

```text
BASE=bf08d8b4e106e299d3250ac16079767feb3e48a3
HEAD=51bf02af1fe0fd6247ee17c8a1a764ced77d2732
COMMITS=1
FILES_CHANGED=7
ADDITIONS=553
DELETIONS=131
```

Changed paths:

```text
M CHANGELOG.md
M pom.xml
M src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
M src/main/java/com/guillermomolina/protos/execution/ProtosStandardIntegerProtocol.java
M src/test/java/com/guillermomolina/protos/execution/ProtosCoreNativeBoundaryArchitectureTest.java
M src/test/java/com/guillermomolina/protos/execution/ProtosPerf027AGuardedIntegerDeferredActivationTest.java
M src/test/java/com/guillermomolina/protos/execution/ProtosStandardIntegerArithmeticTest.java
```

Version publication:

```text
IMPLEMENTATION_VERSION=0.3.157-SNAPSHOT
ROOT_CHANGELOG_ENTRY=YES
SPECIFICATION_CHANGE=NO
```

## Regression coverage

The publication extends focused coverage for:

- exact installed Closure/body/home/selector canonical provenance;
- rejection of copied Closure identity and wrong method home;
- exact values beyond machine-width assumptions;
- direct successful canonical local Integer operations;
- absence of rich Activation on a successful direct hit;
- wrong-domain and zero-divisor deferred native Error paths;
- inherited Number ordering remaining outside the bounded direct slice;
- separate Truffle Context fallback Activation isolation; and
- the audited Java native-provider boundary.

Relevant tests:

```text
ProtosStandardIntegerArithmeticTest
ProtosPerf027AGuardedIntegerDeferredActivationTest
ProtosGuardedLookupTest
ProtosCoreNativeBoundaryArchitectureTest
```

## Validation provenance

The maintainer reported the semantic-authority focal test PASS, followed by the
combined canonical/direct-execution focal tests PASS.

The affected regression set initially exposed one expected architecture-guard
mismatch after the lexical native-construction contraction:

```text
EXPECTED_INTEGER_NATIVE_PROVIDER_COUNT=4
ACTUAL_INTEGER_NATIVE_PROVIDER_COUNT=2
```

The guard was updated to retain exact auditing at the new lower count, then
`ProtosCoreNativeBoundaryArchitectureTest` passed.

For the final publication bytes, the first integrated `make test` attempt
completed its Java test execution but the TEST008 slow-test guard observed:

```text
ProtosPackageExecutionPlanAdapterTest=17.12s
ALLOWLIST_BUDGET=15s
JAVA_SLOW_TEST_GUARD=FAIL
```

No implementation or allowlist change was made in response. That Package Tool
test is unrelated to the changed Integer/runtime surfaces.

The maintainer then reran the canonical complete Java lane and the Protos lane
on the unchanged final candidate and reported both PASS:

```text
make test-java=PASS
make test-protos=PASS
FINAL_FULL_INTEGRATED_LANES=PASS
```

The publication preflight also reported:

```text
LOCAL_BASE=bf08d8b4e106e299d3250ac16079767feb3e48a3
REMOTE_BASE=bf08d8b4e106e299d3250ac16079767feb3e48a3
GIT_DIFF_CHECK=PASS
FINAL_CHANGED_PATHS=7
VERSION=0.3.157-SNAPSHOT
```

Therefore:

```text
MAINTAINER_REPORTED_FOCAL_VALIDATION=PASS
MAINTAINER_REPORTED_AFFECTED_VALIDATION=PASS
MAINTAINER_REPORTED_FINAL_JAVA_LANE=PASS
MAINTAINER_REPORTED_FINAL_PROTOS_LANE=PASS
FULL_VALIDATION=PASS
INDEPENDENT_TEST_REEXECUTION_BY_COORDINATING_AGENT=NO
```

No command output beyond the maintainer-reported interaction is invented.

## Performance-claim boundary

No benchmark, timing comparison, JFR campaign, allocation profile, or
exact-revision performance attribution was performed for this slice.

```text
BENCHMARK_RUN_FOR_THIS_SLICE=NO
ATTRIBUTABLE_INVOCATION_TAX_REDUCTION=NOT_MEASURED
END_TO_END_SPEEDUP_PERCENT=NOT_MEASURED
PERFORMANCE_MAGNITUDE_CLAIMED=NO
```

The durable claim is structural: successful exact canonical local Integer
operations admitted by the guarded selection no longer construct or enter the
general rich native-Closure invocation scaffold.

## PERF025 and PERF027 status

This bounded PERF025 implementation slice is complete. PERF025 itself remains
open.

PERF027 remains separately open. Its previously published PERF027-B
exact-revision measurement step was not executed by this work. The later
PERF025 product revision must not be mistaken for that measurement, and no
machine-width Integer representation experiment is authorized by this record.

```text
PERF025_DIRECT_CANONICAL_INTEGER_EXECUTION=COMPLETE
PERF025_STATUS=OPEN

PERF027_A_HISTORICAL_CHECKPOINT=PRESERVED
PERF027_B_EXECUTED=NO
PERF027_STATUS=OPEN

INTEGER_REPRESENTATION_CHANGED=NO
OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
BENCHMARK_RESULT_CLAIMED=NO
```

## Materially inspected publication evidence

The coordinating publication review inspected:

- `guillermomolina/protos@51bf02af1fe0fd6247ee17c8a1a764ced77d2732`;
- the exact comparison against
  `bf08d8b4e106e299d3250ac16079767feb3e48a3`;
- the published `0.3.157-SNAPSHOT` changelog/version delta;
- the ratified PLAT040 Candidate F′ record;
- the implementation blocker ledger;
- PERF027-A durable evidence;
- PERF025/#758 and PERF027/#779 live coordination state; and
- the maintainer-reported focal, affected and final integrated validation
  outcomes.

This record is evidence only and does not replace live GitHub Issue
coordination.
