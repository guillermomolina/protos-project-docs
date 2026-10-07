# TEST009-Y — validated Error identity causal repair

## Scope

```text
WORK_ITEM=TEST009/#795
SLICE=TEST009-Y
WORK_TYPE=IMPLEMENTATION
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=5c1fd0dc1b0aa0bb3a7719ba6c0e3f5d0b759407
PROTOS_VERSION=0.3.255-SNAPSHOT
COMMIT_SUBJECT=TEST009-Y: retain validated Error identity in ProtosPrelude
```

TEST009-Y implements the first structural causal repair selected by TEST009-X
for the single-root `CodeTooLarge` failure isolated in TEST009-W.

The change is intentionally narrow: `ProtosPrelude` retains the exact canonical
`Error` object that its constructor already reads and validates from frozen
prelude bindings, and `errorPrototype()` returns that stable identity directly
instead of repeating `bindings.readLocalSlot("Error").orElseThrow()`.

No other Prelude binding is made eager. Existing constructor validation and its
failure timing are preserved.

## Published product change

The published revision changes:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/runtime/ProtosPrelude.java
src/test/java/com/guillermomolina/protos/runtime/ProtosPreludeTest.java
```

The implementation adds one final retained `ProtosObjectValue errorPrototype`,
assigns it only after the existing `Error` binding validation has succeeded, and
returns it directly from `errorPrototype()`.

Focused regression coverage proves:

- the validated `Error` identity is retained exactly;
- repeated `errorPrototype()` calls return that same identity;
- `newError()` still delegates to that identity;
- missing or misparented `Error` bindings still fail with the existing
  constructor diagnostic; and
- absent unrelated Prelude bindings remain accepted at construction time, so
  TEST009-Y does not convert their lazy validation into eager requirements.

## Maintainer-reported validation

The maintainer reported:

```text
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
```

The publication commit bumps the development version to:

```text
PROTOS_VERSION=0.3.255-SNAPSHOT
```

No tests were run after the `pom.xml` / `CHANGELOG.md` finalization step.

## Fresh causal remeasurement

The same TEST009-W root was compiled fresh on current product bytes plus the Y
repair:

```text
SELECTOR=protos-root:088d2ae81075aba8
SOURCE_FAMILY=protos/tools/test/Manifest.protos
TARGET_ROOT_FOUND=YES
TARGET_COMPILATION=FAILED_CODE_TOO_LARGE
CODE_TOO_LARGE=YES
AFTER_TRUFFLE_TIER_PRESENT=PASS
```

The diagnostic driver printed `TOOL_ACQUISITION_FAILED` because its supplied
`path/to/case.protos::case name` text was only placeholder Case-selection input,
not a real Test Tool Case. The Test Tool rejected that selector and therefore did
not emit a normal corpus summary.

That later semantic-selection failure does not invalidate the compiler evidence:
before rejecting the placeholder Case, ordinary Case selection traversed
`CaseSelection.protos` -> `CaseRef.protos`, executed the target root, compiled it
fresh on the Y candidate, emitted the full post-Truffle-tier method expansion
tree, and recorded:

```text
Code installation failed: code is too large
```

The captured tree was retained locally at:

```text
target/truffle-compilation/root-diagnostic/088d2ae81075aba8/diagnostic.log
```

with the relevant target compilation result/tree beginning around local line
4732. The local path is evidence provenance only, not a durable repository
contract.

## W -> Y comparison

The Y evidence was parsed using the same method-expansion-tree view as W:

| Metric | TEST009-W | TEST009-Y | Delta |
|---|---:|---:|---:|
| parsed rows | 24604 | 23521 | -1083 |
| unique method names | 524 | 524 | 0 |
| `Throwable.fillInStackTrace()` self size | 17303 | 11440 | **-5863** |
| `fillInStackTrace` attributed to `ProtosPrelude.errorPrototype()` | 5863 | **0** | **eliminated** |
| entire `errorPrototype()` subtree | not separately recorded | 41 | residual frame/call only |
| `OptimizedCallTarget.profiledPERoot(Object)` | 14469 | 13813 | -656 |
| `StringLatin1.inflate(...)` | 5731 | 5731 | 0 |
| `ProtosFrameArguments.hasCompactHeader(Object)` | 3687 | 3687 | 0 |

The `Throwable.fillInStackTrace()` reduction is exactly `5863`, which is exactly
the W attribution to `ProtosPrelude.errorPrototype()`. Other directly compared
owners remain unchanged. This is the causal signature TEST009-X required:
TEST009-Y removed the defensive exception-expansion axis it targeted rather than
merely moving cost elsewhere.

Therefore:

```text
TEST009_Y_CAUSAL_REPAIR_CONFIRMED=YES
CAUSAL_ACCEPTANCE_CRITERION=B
PROTOS_PRELUDE_ERROR_PROTOTYPE_EXPANSION=ELIMINATED
TARGET_ROOT_CODE_TOO_LARGE_AFTER_REPAIR=YES
```

## Residual host defensive exception ranking

After Y, `Throwable.fillInStackTrace()` totals `11440`. The largest remaining
Protos-attributed owners are:

```text
ProtosValueLookup.lookup -> delegationParent                     1573 / 1573
ProtosPrelude.arrayPrototype()                                    1430
ProtosBytecodeRootNode.attachTaskOrInheritDynamicControlState     1430
ProtosBytecodeRootNode.rejectComposedInvocationProjection         1430
PreparedBooleanCall.hasCallback()                                 1287
ProtosPrelude.standardErrorPrototype(String)                      1001
ProtosBytecodeRootNode.finishPreparingComposedCall                 858
```

This ranking is evidence for the next selection investigation, not authorization
to batch those owners into one implementation.

## LOCAL_FRAME status

TEST009-W recorded:

```text
LOCAL_FRAME=14230
```

The exact W classifier is not present in the current repository, so Y does not
claim a directly comparable replacement value. A different frame-primitives /
accessors proxy measured `8773`, but that number uses different classification
rules and must not be compared numerically with W's `14230`.

Directly comparable frame-related axes remain unchanged:

```text
ProtosFrameArguments.hasCompactHeader(Object): W=3687  Y=3687
CachedBytecodeNode.handleLoadLocal$generic:       W=1764  Y=1764
```

Accordingly:

```text
LOCAL_FRAME_W=14230
LOCAL_FRAME_CURRENT=NOT_COMPARABLE
LOCAL_FRAME_PROXY_CURRENT=8773
LOCAL_FRAME_DIRECTLY_COMPARABLE_AXES_CHANGED=NO
LOCAL_FRAME_REPAIR_AUTHORIZED=NO
```

## Closure and continuation

TEST009-Y is complete: the selected structural repair is published, functional
validation passes, and its expected causal effect is directly confirmed.

TEST009 as a parent issue is not complete because the selected root still fails
with `CodeTooLarge`.

The next step must not immediately implement the largest residual. The residual
owners mix different semantic reachability and validation-timing properties:

- `ProtosValueLookup.lookup` / `delegationParent` contains duplicated
  Optional-based parent consumption and remains the strongest cumulative
  candidate from TEST009-X plus the Y residual;
- `ProtosPrelude.arrayPrototype()` cannot simply copy the Y technique because
  TEST009-X established that its validation is currently lazy rather than
  constructor-certified;
- task attachment contains a repeated Optional access but is a separate owner;
- `rejectComposedInvocationProjection(...)` protects a real reachable invalid
  state and must not be removed merely because it appears in the expansion;
- `PreparedBooleanCall` / generated exhaustive-switch defensive paths require
  their own reachability analysis; and
- `LOCAL_FRAME` remains a major axis but is still not authorized for repair.

Therefore the next slice is read-only selection work:

```text
NEXT_SLICE=TEST009-Z
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_SCOPE=Select the next single causal structural repair from the post-Y CodeTooLarge residual, starting with ProtosValueLookup lookup/delegationParent and comparing all major residual owners without implementing them
PRODUCT_CHANGES_AUTHORIZED=NO
COMMAND_EXECUTION_AUTHORIZED=NO
NEW_FORMAL_ISSUE_REQUIRED=NO
```

## Final state

```text
TEST009_Y_RESULT=COMPLETE
PROTOS_REVISION=5c1fd0dc1b0aa0bb3a7719ba6c0e3f5d0b759407
PROTOS_VERSION=0.3.255-SNAPSHOT
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
TARGET_ROOT_FOUND=YES
TARGET_COMPILATION=FAILED_CODE_TOO_LARGE
CODE_TOO_LARGE=YES
PROTOS_PRELUDE_ERROR_PROTOTYPE_EXPANSION=ELIMINATED
THROWABLE_FILL_IN_STACK_TRACE_W=17303
THROWABLE_FILL_IN_STACK_TRACE_Y=11440
THROWABLE_FILL_IN_STACK_TRACE_DELTA=-5863
CAUSAL_REPAIR_CONFIRMED=YES
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
BOUNDARY_CHASING=NO
LOCAL_FRAME_INCLUDED=NO
TEST009_COMPLETE=NO
NEXT_SLICE=TEST009-Z
NEXT_SLICE_TYPE=INVESTIGATION
```
