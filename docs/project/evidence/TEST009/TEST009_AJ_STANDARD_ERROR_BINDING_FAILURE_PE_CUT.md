# TEST009-AJ — missing standard Error binding failure PE cut

Status: **PUBLISHED — SELECTED ROOT NOW COMPILES; TEST009 CLOSURE REVALIDATION PENDING**

Formal work: `guillermomolina/protos#795`

Publication date: **2026-10-07**

## Published product

```text
PROTOS_REVISION=5e2e459b952583a48b870c84321a9796bf81794f
PROTOS_VERSION=0.3.266-SNAPSHOT
COMMIT_SUBJECT=TEST009-AJ: keep missing standard Error binding failure out of PE
```

AJ is published directly on top of TEST009-AI:

```text
BASE_PROTOS_REVISION=84ff312ca6c29d2bfa9e6a4d69861c1dfdd07c47
```

## Selected owner and mechanism

The fixed root and Case remained:

```text
ROOT_SPEC=protos/tools/test/Manifest.protos
ROOT_LABEL=protos-root:088d2ae81075aba8
CASE_REF=protos/corpus/conformance:boolean/lazy-binary.protos::and selected false
ROOT_CHANGE=NO
```

AI had already attributed the next exact current NSEE owner:

```text
SELECTED_OWNER=ProtosPrelude.standardErrorPrototype(String)
SELECTED_CALLSITE=bindings.readLocalSlot(name).orElseThrow()
PRE_REPAIR_SELECTED_OCCURRENCES=14
```

At the AJ base revision this method had one NSEE-producing source callsite.
Its other explicit failure paths were `IllegalStateException` validations for
ordinary-object type and Error-hierarchy membership.

The selected mechanism was therefore:

```text
dynamic missing standard-Error binding
  ->
Optional.orElseThrow()
  ->
NoSuchElementException.<init>
  ->
Throwable.fillInStackTrace
  ->
partial-evaluation expansion
```

The missing binding remains a real, reachable, lazy host-side validation failure.
No semantic or specification change was required.

## Published repair

AJ changes only the missing-binding exception construction path.

Conceptually:

```java
Optional<Object> found = bindings.readLocalSlot(name);
if (found.isEmpty()) {
    CompilerDirectives.transferToInterpreter();
    throw new NoSuchElementException("No value present");
}
Object binding = found.orElse(null);
```

All following validation remains unchanged:

- dynamic name lookup;
- ordinary-object validation;
- Error-hierarchy membership validation;
- canonical Error identity checks.

No `transferToInterpreterAndInvalidate()`, `@TruffleBoundary`, eager
standard-Error caching, or hierarchy shortcut was introduced.

The focal regression
`ProtosPreludeTest.lazilyRejectsMissingStandardErrorBindingWithUnchangedFailure`
pins lazy lookup plus the exact `NoSuchElementException("No value present")`
failure contract.

## Semantic/API result

```text
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PUBLIC_API_CHANGE=NO
VALIDATION_TIMING_CHANGE=NO
CONCURRENCY_OR_OBSERVABILITY_CHANGE=NO
MULTI_OWNER_BATCH=NO
```

## Same-root causal A/B

The predeclared primary signature was satisfied exactly on occurrence counts:

```text
standardErrorPrototype -> orElseThrow NSEE
  PRE=14
  POST=0

NoSuchElementException aggregate
  PRE=46
  POST=32
  DELTA=-14
```

Independent controls remained unchanged:

```text
arrayPrototype
  0 -> 0

attachTaskOrInheritDynamicControlState family
  20 -> 20

finishPreparingComposedCall family
  12 -> 12
```

No equivalent exception cost was found under the changed ProtosPrelude path.

A row-count parser used during AJ reported the selected pre-repair subtree as
119 cumulative row Count units. The earlier AI record used a different subtree
metric and reported 1036. These cumulative values are therefore not compared
across the two tools. The exact owner occurrence count is common to both
measurements and matches at 14; AJ's own pre/post comparison uses one metric
consistently.

The larger-than-required result is the key TEST009 milestone:

```text
PRE_REPAIR_TARGET=FAILED_CODE_TOO_LARGE
POST_REPAIR_TARGET=COMPILED
POST_REPAIR_OPT_DONE=YES
POST_REPAIR_CODE_SIZE=609716
```

This is the first published state in the W→AJ causal sequence where the selected
fixed root completes optimizing compilation instead of failing with
`CodeTooLarge`.

`Throwable.fillInStackTrace` occurrence count also falls:

```text
PRE=76
POST=62
DELTA=-14
```

Therefore:

```text
SELECTED_NSEE_SUBTREE_REMOVED=YES
NSEE_AGGREGATE_DELTA_MATCHES_SELECTED_OCCURRENCES=YES
INDEPENDENT_CONTROLS_CHANGED=NO
COST_DISPLACEMENT_DETECTED=NO
CAUSAL_REPAIR_CONFIRMED=YES
SELECTED_ROOT_COMPILERABILITY=PASS
```

## Diagnostic acquisition note

Both AJ single-root runs ended after the relevant compilation evidence with:

```text
TOOL_ACQUISITION_FAILED
TRUFFLE_ROOT_CAPTURE_READY=NO
```

because the Test Tool later reported:

```text
Object {} at CaseRef.protos:40
```

The target compilation trace and requested expansion tree had already completed,
so the same-root compiler A/B remains usable and internally consistent.

This post-compilation Test Tool diagnostic failure is not attributed to AJ.
No separate formal work is allocated by this record. If the final repository-wide
strict gate reproduces a real semantic/tool failure, it must be handled from
that current closure evidence rather than inferred from this local diagnostic
tail.

## Validation

Maintainer-reported AJ validation:

```text
FOCAL_TESTS=PASS
FOCAL_COUNT=27
FOCAL_FAILED=0

FULL_SUITE=PASS
FULL_SUITE_COMMAND=make test

FINAL_GIT_DIFF_CHECK=PASS
PUBLICATION=PUSHED
```

The focal classes included:

```text
ProtosPreludeTest
ProtosIoLifecycleTest
ProtosActorRequestTest
```

The shared-runtime delta therefore passed both focal and integrated functional
validation before `pom.xml` / `CHANGELOG.md` finalization.

No tests were run after the version/changelog update.

## Published files

```text
src/main/java/com/guillermomolina/protos/runtime/ProtosPrelude.java
src/test/java/com/guillermomolina/protos/runtime/ProtosPreludeTest.java
pom.xml
CHANGELOG.md
```

The modified Java files retain their existing Adaptive Public License Part 5
header.

## Cumulative TEST009 single-root progress

The W→AJ sequence now has the following compilerability trajectory:

```text
W
  Throwable.fillInStackTrace=17303
  target=FAILED_CODE_TOO_LARGE

Y
  17303 -> 11440
  owner=ProtosPrelude.errorPrototype

AC
  11440 -> 9867
  owner=checked local Optional NSEE

AD
  9867 -> 8294
  owner=delegationParent unsupported-representation UOE

AF
  8294 -> 6864
  owner=rejectComposedInvocationProjection UOE

AI
  6864 -> 5434
  owner=ProtosPrelude.arrayPrototype missing binding
  target=FAILED_CODE_TOO_LARGE

AJ
  selected standardErrorPrototype NSEE 14 -> 0
  Throwable.fillInStackTrace occurrences 76 -> 62
  target=COMPILED
  CodeSize=609716
```

The remaining current single-root NSEE occurrence families include:

```text
attachTaskOrInheritDynamicControlState=20
finishPreparingComposedCall=12
```

and AJ's post log also contains MatchException rows attributed outside
`ProtosPrelude`, including `PreparedBooleanCall` record-accessor expansion.

These are **not automatically further TEST009 repair work**. Once the root
compiles, residual expansion alone is not compilerability debt.

## Closure boundary

TEST009's repository-level compilerability authority is already implemented in
the product:

```text
make check
  ->
toolchain
check-local-range-index-pe
check-local-range-operands-pe
check-local-accessor-pe
check-bytecode-api-pe
check-generated-bytecode-bci-pe
check-truffle-compilation
```

The final `check-truffle-compilation` gate runs the real Test Tool corpus in:

```text
TRUFFLE_COMPILATION_SYNC
TRUFFLE_COMPILATION_BACKGROUND
```

with immediate compilation, fatal compilation failures and performance warnings
as errors.

Therefore the next TEST009 work is closure revalidation, not automatic residual
NSEE cleanup:

```text
NEXT_SCOPE=run current repository-wide make check on the published AJ HEAD and reconcile every TEST009 acceptance criterion
```

If current `make check` is green, remaining expansion rows are not by
themselves grounds for another repair slice and TEST009 is technically
closable subject to final durable/live coordination reconciliation.

If `make check` is red, only the exact current failing compilerability owner
released by that gate becomes further TEST009 repair work.

```text
TEST009_COMPLETE=NO
NEXT_SLICE=TEST009-AK
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos
NEXT_SCOPE=CLOSURE_REVALIDATION_ONLY_UNLESS_CURRENT_MAKE_CHECK_EXPOSES_REAL_COMPILERABILITY_DEBT
NEW_FORMAL_ISSUE_REQUIRED=NO
```

## AI-assistance disclosure

This evidence record was materially prepared with AI assistance from ChatGPT
using the published AJ product commit, maintainer-reported focal/full-suite and
same-root A/B results, the current TEST009 Issue contract, and current product
compilerability/check surfaces. No independent human review is claimed.
