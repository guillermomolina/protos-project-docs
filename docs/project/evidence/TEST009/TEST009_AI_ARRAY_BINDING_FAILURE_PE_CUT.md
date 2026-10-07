# TEST009-AI — missing Array binding failure PE cut

Status: **PUBLISHED — CAUSAL REPAIR CONFIRMED; TEST009 REMAINS OPEN**

Formal work: `guillermomolina/protos#795`

Publication date: **2026-10-07**

## Published product

```text
PROTOS_REVISION=84ff312ca6c29d2bfa9e6a4d69861c1dfdd07c47
PROTOS_VERSION=0.3.265-SNAPSHOT
COMMIT_SUBJECT=TEST009-AI: keep missing Array binding failure out of PE
```

The publication was made on top of:

```text
PUBLICATION_PARENT=b3d85c1deee91455c2777d0025a19ec3970530ac
PUBLICATION_PARENT_SUBJECT=PLAT051-B: opt std:regex/Regex Pattern and Match into semantic transfer
```

The concurrent PLAT051-B publication did not touch `ProtosPrelude` or the
TEST009 NSEE owner set selected by AI.

## AI acquisition and owner selection

TEST009-AI followed the executable gated workflow selected after AH:

```text
fresh fixed-root dynamic acquisition
  ->
exact residual NSEE decomposition
  ->
one owner/callsite selection
  ->
mechanism and semantic/timing/API/observability gates
  ->
one bounded repair
  ->
same-root causal A/B
```

The fixed root remained:

```text
ROOT_SPEC=protos/tools/test/Manifest.protos
ROOT_LABEL=protos-root:088d2ae81075aba8
ROOT_CHANGE=NO
CASE_REF=protos/corpus/conformance:boolean/lazy-binary.protos::and selected false
```

The fresh pre-repair expansion attributed one exact current NSEE subtree to:

```text
SELECTED_OWNER=ProtosPrelude.arrayPrototype()
SELECTED_CALLSITE=bindings.readLocalSlot("Array").orElseThrow()
SELECTED_SUBTREE=20 occurrences / cumulative size 1480
```

The missing `Array` binding failure is real and reachable through a Prelude
whose frozen bindings omit `Array`; the normal Core bootstrap separately
requires and validates the canonical Array binding.

AI classified the exact mechanism as:

```text
FAILURE_CLASS=COLD_REAL_FAILURE_CONSTRUCTION_IN_PE
FAILURE_EXCEPTION=NoSuchElementException("No value present")
BINDINGS_STABLE_AFTER_PRELUDE_CONSTRUCTION=YES
EAGER_ARRAY_VALIDATION_SELECTED=NO
INVALIDATION_REQUIRED=NO
```

The repair therefore preserves lazy validation and moves only the real missing
binding exception construction out of partial evaluation.

## Published repair

Before AI:

```java
Object binding = bindings.readLocalSlot("Array").orElseThrow();
```

Published AI shape:

```java
Optional<Object> found = bindings.readLocalSlot("Array");
if (found.isEmpty()) {
    CompilerDirectives.transferToInterpreter();
    throw new NoSuchElementException("No value present");
}
Object binding = found.orElse(null);
```

The existing ordinary-object validation after the binding read is unchanged.

The focal regression
`ProtosPreludeTest.lazilyRejectsMissingArrayBindingWithUnchangedFailure`
pins:

- lazy failure on `arrayPrototype()`;
- the exact `NoSuchElementException` class;
- exact message `No value present`;
- the same failure through `newArray(...)`.

The publication does not introduce `transferToInterpreterAndInvalidate()` and
does not add a `@TruffleBoundary`.

## Semantic and API result

```text
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PUBLIC_API_CHANGE=NO
VALIDATION_TIMING_CHANGE=NO
CONCURRENCY_OR_OBSERVABILITY_CHANGE=NO

MISSING_ARRAY_FAILURE_CLASS=UNCHANGED
MISSING_ARRAY_FAILURE_MESSAGE=UNCHANGED
MISSING_ARRAY_FAILURE_TIMING=UNCHANGED_LAZY
MULTI_OWNER_BATCH=NO
```

The frozen Prelude binding set makes the selected binding-presence observation
stable; no mutable publication/concurrency question is introduced by this
repair.

## Same-root causal A/B

The predeclared causal signature was:

```text
remove exactly the arrayPrototype -> Optional.orElseThrow NSEE subtree
keep independent standardErrorPrototype, Task unwrap and composed-call NSEE owners unchanged
```

Observed result:

| Metric | Pre-AI | Post-AI | Delta |
|---|---:|---:|---:|
| target compilation | FAILED_CODE_TOO_LARGE | FAILED_CODE_TOO_LARGE | unchanged |
| selected `arrayPrototype` NSEE subtree | 20 / 1480 | 0 | -20 / -1480 |
| all `NoSuchElementException.<init>` | 66 / 4884 | 46 / 3404 | -20 / -1480 |
| `Throwable.fillInStackTrace` | 96 / 6864 | 76 / 5434 | -20 / -1430 |

Independent controls were unchanged:

```text
standardErrorPrototype
  14 / 1036 -> 14 / 1036

attachTaskOrInheritDynamicControlState family
  10 / 740 unchanged
   8 / 592 unchanged
   2 / 148 unchanged

finishPreparingComposedCall family
  10 / 740 unchanged
   2 / 148 unchanged
```

Therefore:

```text
SELECTED_SUBTREE_REMOVED=YES
NSEE_DELTA_MATCHES_SELECTED_SUBTREE=YES
INDEPENDENT_CONTROL_OWNERS_CHANGED=NO
EQUIVALENT_NSEE_COST_DISPLACED=NO
CAUSAL_REPAIR_CONFIRMED=YES
```

The root still fails `CodeTooLarge`, so TEST009 remains open.

## Validation

The maintainer reports:

```text
GIT_DIFF_CHECK=PASS
FOCAL_TESTS=PASS
MAINTAINER_REPORTED_ALL_LOCAL_TESTS=PASS
PUBLICATION=PUSHED
```

The focal Java set covered 18 tests across:

```text
ProtosPreludeTest
ProtosStandardArrayFactoryTest
ProtosCoreBootstrapTest
ProtosSourceCompilerTest
```

During the AI validation sequence, one `make test` invocation observed a
transient failure in
`ProtosLm009DPublicDebugCliTest.realDebugThroughPublicCliSurvivesTransitiveClosureSpec`
during the parallel Java phase: the guest completed normally but the CLI returned
exit code 1. That test then passed three isolated runs, the TEST009-AI patch does
not reach the successful Core bootstrap path exercised there, and the maintainer
subsequently reports all local tests passing.

The transient observation is retained here as execution history, but it is not
classified as a TEST009 blocker and this publication does not allocate a formal
BUG from it. A recurring failure should be investigated independently with the
CLI diagnostic output included in the assertion before deciding whether its
execution lane should change.

No formatter or separate Java linter is defined for this source scope.

## Files published

```text
src/main/java/com/guillermomolina/protos/runtime/ProtosPrelude.java
src/test/java/com/guillermomolina/protos/runtime/ProtosPreludeTest.java
pom.xml
CHANGELOG.md
```

The modified Java files retain their existing Adaptive Public License Part 5
header.

## Current residual and next owner

Post-AI current NSEE residual:

```text
NoSuchElementException.<init>
  46 occurrences
  cumulative size 3404
```

The fresh AI expansion leaves three useful axes:

```text
standardErrorPrototype
  14 occurrences / 1036

attachTaskOrInheritDynamicControlState
  aggregate 20 occurrences / 1480 across three caller expansions

finishPreparingComposedCall
  aggregate 12 occurrences / 888, exact sub-callsite not yet separated
```

The next single repair can be released without another no-command investigation.

At the published product revision,
`ProtosPrelude.standardErrorPrototype(String)` has exactly one source-level
NSEE-producing callsite:

```java
Object binding = bindings.readLocalSlot(name).orElseThrow();
```

Its other explicit failure paths construct `IllegalStateException`, not NSEE.
The current 14 / 1036 attribution therefore identifies the exact NSEE callsite.

The missing named standard-Error binding is a real dynamic validation failure.
A repair may move only that exact missing-binding exception construction out of
PE while preserving:

```text
dynamic name lookup
lazy lookup timing
NoSuchElementException class/message
ordinary-object validation
Error-hierarchy validation
public behavior
```

It must not cache all standard Error prototypes or remove/advance hierarchy
validation.

Accordingly:

```text
TEST009_STATE=OPEN_IN_PROGRESS

NEXT_SLICE=TEST009-AJ
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos

SELECTED_OWNER=ProtosPrelude.standardErrorPrototype(String)
SELECTED_CALLSITE=bindings.readLocalSlot(name).orElseThrow()
CURRENT_CAUSAL_ATTRIBUTION=14 occurrences / 1036 cumulative
SELECTED_MECHANISM=real dynamic missing-standard-Error binding failure constructs NoSuchElementException in PE
SELECTED_REPAIR_CLASS=preserve dynamic lookup and lazy validation; transfer only the missing-binding exception construction to the interpreter without invalidation

ROOT_CHANGE=NO
MULTI_OWNER_BATCH=NO
NEW_FORMAL_ISSUE_REQUIRED=NO
```

AJ must revalidate the current HEAD and fresh same-root baseline before editing.
If concurrent movement invalidates the exact owner/callsite attribution, it must
stop without product edits rather than silently selecting a different owner.

## AI-assistance disclosure

This durable evidence was materially prepared with AI assistance from ChatGPT
using the published TEST009-AI commit, maintainer-reported local validation and
same-root diagnostic results, current live TEST009 coordination, and the current
public product source. No independent human review is claimed.
