# PERF016 — Integer guarded represented-selection implementation checkpoint

Date: 2026-09-29

## Identity

```text
WORK_ITEM=PERF016/#727
PARENT=PERF010-B/#722
PRODUCT_REPOSITORY=guillermomolina/protos

IMPLEMENTATION_BASE_REVISION=453f2b00ae2bd0af1cd2761474549e85ff1cb8b6
IMPLEMENTATION_BASE_VERSION=0.3.118-SNAPSHOT

PROTOS_REVISION=696b0f9797ebc8ced80009fb583027513852f55c
PROTOS_VERSION=0.3.119-SNAPSHOT
COMMIT=PERF016: admit semantic Integer family to guarded represented selection
```

This record retains the structural implementation and validation checkpoint for
PERF016 Step 3B. PERF016 remains open after this record because it also owns the
combined post-Step-3 timing checkpoint after PERF015 + PERF016.

## Published implementation

The published implementation adds Integer-family guarded represented selection
without changing Integer or D013 semantics.

`ProtosValueLookup.isInteger(...)` admits only the semantic Integer runtime
representation:

```text
receiver instanceof ProtosIntegerValue
```

The relevant existing representation invariant is:

```text
ProtosIntegerValue
  -> representedDelegationParent(prelude)
  -> prelude.integerPrototype()
  -> Number prototype
  -> root Object
```

`lookupGuardedInteger(...)` proves the receiver is Integer, requires a non-null
Prelude, requires the represented delegation parent to be that Prelude's exact
Integer prototype, and then runs the same D013 lookup loop as the generic path.
Only the represented receiver's own fixed step is exempt from invalidation;
ordinary Integer/Number/Object prototype objects remain lookup dependencies.

`PrepareSendArguments.guardedIntegerSend` is keyed by:

```text
semantic Integer family
selector
entered Truffle Context identity
Prelude identity
selection Assumption
```

and deliberately not by exact Integer receiver identity. Fresh Integer values
created by repeated `count - 1` therefore remain eligible for the same guarded
family selection.

The cache stores only:

```text
selected Closure
exact selected method home
selection Assumption
```

The actual current receiver and arguments still flow into the unchanged
`prepareImmediateMethodCall(...)` path. No arithmetic or comparison operation
is implemented at the send site.

## Mandatory driver-send result

The two mandatory PERF016 driver sends resolve through the expected standard
homes:

```text
count - 1
  selected home = Integer prototype
  standard selected value = native Closure

count > 0
  selected home = Number prototype
  standard selected value = native Closure
```

The optimization therefore caches authoritative selection/provenance while
retaining the existing native Closure execution path.

## Structural regression evidence

`ProtosGuardedLookupTest` adds focused PERF016 coverage.

`integerFamilyAdmissionIsByRepresentationNotIdentity` proves:

- multiple distinct Integer objects are admitted;
- equal-valued but different-identity Integers are admitted;
- an unbounded large Integer is admitted;
- Boolean, Float, arbitrary ordinary Object, and another represented carrier are
  rejected.

`integerMinusSelectsIntegerPrototypeAndMatchesGenericLookup` proves:

- `-` guarded selection exists and is stable;
- exact selected home is `prelude.integerPrototype()`;
- the selection is a native Closure;
- the cached selection exactly matches generic D013 lookup across several fresh
  Integer receiver values.

`integerGreaterSelectsNumberPrototypeAndMatchesGenericLookup` proves the same
properties for `>`, with exact selected home `prelude.numberPrototype()`.

`integerGuardedSelectionDependsOnFrozenStandardGraph` proves the relevant
published Integer/Number graph is frozen and rejects mutation.

`integerGuardedSelectionFallsBackForUnsupportedReceiversAndSelectors` proves
the family-specific entry point rejects absent selectors, missing Prelude,
Float, Boolean, ordinary Object, and a non-Integer represented carrier. It also
proves the pre-existing ordinary guarded lookup and PERF015 Boolean lookup do not
silently admit Integer.

## Structural classification

```text
INTEGER_REPRESENTED_FAMILY_INVARIANT=PROVEN
INTEGER_REPRESENTED_GUARD=PASS
INTEGER_SELECTION_INVALIDATION=PASS
INTEGER_VALID_HIT_GENERIC_SELECTION=NO
ACTUAL_INTEGER_RECEIVER_PRESERVED=PASS

MINUS_SELECTED_HOME=INTEGER_PROTOTYPE
GREATER_SELECTED_HOME=NUMBER_PROTOTYPE

GENERIC_FALLBACK=PASS
MULTIPLE_CONTEXT_ISOLATION=PASS
PERF015_BOOLEAN_PATH_PRESERVED=PASS
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
STANDARD_LIBRARY_SEMANTIC_CHANGE=NO
```

The `INTEGER_VALID_HIT_GENERIC_SELECTION=NO` conclusion follows from the
published specialization construction: `lookupGuardedInteger(...)` executes
only when the specialization is established; a valid
`guardedIntegerSend` hit consumes its cached Closure/home/Assumption and calls
`prepareImmediateMethodCall(...)` directly. The actual receiver remains a
method argument on every hit.

## Version and product delta

The published product revision advances the development version:

```text
0.3.118-SNAPSHOT -> 0.3.119-SNAPSHOT
```

Changed files:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/runtime/ProtosValueLookup.java
src/test/java/com/guillermomolina/protos/runtime/ProtosGuardedLookupTest.java
```

The changelog explicitly makes no timing claim.

## Integrated validation

The exact published revision passed push CI:

```text
CI_RUN_ID=36616064789
CI_HEAD_SHA=696b0f9797ebc8ced80009fb583027513852f55c
CI_CONCLUSION=success

CI_COMMANDS:
  python3 tools/verify_toolchain.py --mode check --scope development
  make test JAVA_TEST_JOBS=4 PROTOS_TEST_JOBS=4

FINAL_REQUIRED_VALIDATION=PASS
```

Existing APL-1.0 notices are retained on modified Protos-owned source files.

## Post-Step-3 timing control/intervention identity

The next PERF016 phase must measure the combined effect of PERF015 + PERF016.

The exact pre-Step-3 control and post-Step-3 intervention are:

```text
CONTROL_REVISION=2e3f56fae3a500d3e4193e3345d8a82c35e4590e
CONTROL_VERSION=0.3.117-SNAPSHOT

INTERVENTION_REVISION=696b0f9797ebc8ced80009fb583027513852f55c
INTERVENTION_VERSION=0.3.119-SNAPSHOT
```

Git history establishes that intervention is exactly two commits ahead of
control:

```text
453f2b00ae2bd0af1cd2761474549e85ff1cb8b6
  PERF015: admit canonical Boolean to guarded structured selection

696b0f9797ebc8ced80009fb583027513852f55c
  PERF016: admit semantic Integer family to guarded represented selection
```

No unrelated product commit exists between those two timing endpoints.

Both revisions are on the canonical post-adoption GraalVM/Graal/Truffle
25.4.4.1.1 product line. Historical PERF014 25.3.4.1 timing evidence must not be
used as the Step-3 comparator.

## Remaining PERF016 work

```text
BOOLEAN_REPRESENTED_SELECTION=COMPLETE
INTEGER_REPRESENTED_SELECTION=COMPLETE

INTEGER_REPRESENTED_SELECTION_IMPLEMENTATION=COMPLETE
FINAL_REQUIRED_VALIDATION=PASS

POST_STEP3_CONTROLLED_TIMING_CHECKPOINT=PENDING
SHARED_DRIVER_TIMING_EFFECT=PENDING
COMMON_WORKLOAD_TIMING_EFFECTS=PENDING
ORDER_EFFECT=PENDING
STEP_3_TIMING_CLASS=PENDING

PERF016_STATUS=OPEN
NEXT_SLICE=POST_STEP3_25_4_TIMING_HARNESS
STEP4_AUTHORIZED=NO
```

PERF016 must not close until the controlled timing checkpoint is retained and
its result is routed back through PERF010-B/#722.
