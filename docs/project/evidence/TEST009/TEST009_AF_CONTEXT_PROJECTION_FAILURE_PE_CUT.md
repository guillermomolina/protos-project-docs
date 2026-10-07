# TEST009-AF — context projection failure PE cut

## Scope

```text
WORK_ITEM=TEST009/#795
SLICE=TEST009-AF
WORK_TYPE=IMPLEMENTATION
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=6a807bb64d44e86a232803d66da9fb4e46953219
PROTOS_VERSION=0.3.263-SNAPSHOT
COMMIT_SUBJECT=TEST009-AF: keep context projection failure out of PE
```

TEST009-AF implements the single causal repair selected and owner-approved by
TEST009-AE: keep the real, reachable Context-local Closure projection failure
semantically unchanged while cutting its Java exception-construction subtree out
of Truffle partial evaluation.

The product revision was published after a concurrent unrelated product advance
had already moved the repository from the AE baseline version
`0.3.261-SNAPSHOT` to `0.3.262-SNAPSHOT`. AF therefore correctly publishes as
`0.3.263-SNAPSHOT`; it does not overwrite that concurrent version advance.

## Published product change

The production edit is intentionally one line inside
`ProtosBytecodeRootNode.rejectComposedInvocationProjection(...)`:

```java
CompilerDirectives.transferToInterpreter();
throw new UnsupportedOperationException(
        "Context-local Closure projection requires an entered Protos Context");
```

The implementation uses the fully qualified
`com.oracle.truffle.api.CompilerDirectives.transferToInterpreter()` spelling
at the call site.

AF does not:

- use `transferToInterpreterAndInvalidate()`;
- add a new `@TruffleBoundary`;
- change the failure condition;
- change the exception class or message;
- convert the failure to a guest Error, miss, sentinel, fallback or control-flow
  carrier;
- modify another TEST009 residual owner; or
- change observable Protos semantics or specification.

A focal Java regression was added in
`ProtosAPlusExecutionProjectionTest` to pin the unentered Context-local
composed-call failure class and exact message.

The published revision changes exactly:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosAPlusExecutionProjectionTest.java
```

## Maintainer-reported validation

The maintainer reported after publication:

```text
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
FULL_SUITE=PASS
PUBLICATION=PUSHED
```

The product commit itself records the focal contract coverage and causal
diagnostic result in `CHANGELOG.md`.

## Exact same-root causal A/B

AF reuses exactly the fixed TEST009 root:

```text
ROOT_SPEC=protos/tools/test/Manifest.protos
ROOT_LABEL=protos-root:088d2ae81075aba8
```

The saved post-AD log reproduced the already-published AD values exactly, so the
comparison method was validated before comparing AF.

| Metric | post-AD | post-AF | Delta |
|---|---:|---:|---:|
| target compilation | `FAILED_CODE_TOO_LARGE` | `FAILED_CODE_TOO_LARGE` | unchanged / allowed |
| `rejectComposedInvocationProjection` | 49 frames, `403 / 1693 / 1592` | 10 frames, `182 / 212 / 182` | selected failure expansion removed; helper remains |
| `UnsupportedOperationException.<init>` | 20 frames, cumulative 1480 | absent | **-20 frames; selected constructor subtree removed** |
| `Throwable.fillInStackTrace` | 116 frames, 8294 | 96 frames, 6864 | **-20 frames; -1430 cumulative** |
| `NoSuchElementException.<init>` control | 66 frames, 4884 | 66 frames, 4884 | unchanged |

AF deliberately did not predict an exact `Throwable.fillInStackTrace` delta
from the UOE cumulative value. The measured result is a fresh A/B observation.

The selected helper remains in the expansion because its valid-path work still
exists. What remains is the
`requiresContextLocalExecutionProjectionForRuntime()` predicate plus the
already-boundaried entered-Context lookup. The UOE construction itself no
longer appears in PE.

## No displacement

The maintainer compared every `*Exception.<init>` and `*Error.<init>` frame in
the AD and AF logs.

The only exception-construction differences are:

```text
UnsupportedOperationException.<init>  20 frames -> 0
RuntimeException.<init>                matching 20 frames -> 0
```

No other exception type appeared or grew.

Therefore:

```text
SELECTED_UOE_SUBTREE_REMOVED=YES
EQUIVALENT_EXCEPTION_COST_DISPLACED=NO
NSEE_CONTROL_CHANGED=NO
```

This is the causal signature requested by AE.

## Contract preservation

The focal regression and the unchanged source contract establish:

```text
UNENTERED_CONTEXT_FAILURE_PRESERVED=YES
EXCEPTION_CLASS=java.lang.UnsupportedOperationException
EXCEPTION_MESSAGE=Context-local Closure projection requires an entered Protos Context
FAILURE_CONDITION_CHANGED=NO

TRANSFER_TO_INTERPRETER_AND_INVALIDATE_USED=NO
NEW_TRUFFLE_BOUNDARY_USED=NO

SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PUBLIC_API_CHANGE=NO
VALIDATION_TIMING_CHANGE=NO
```

## Cumulative single-root causal progress

The W checkpoint began with:

```text
Throwable.fillInStackTrace=17303
```

The isolated same-root repairs now measure:

```text
TEST009-Y:
  17303 -> 11440
  DELTA=-5863
  OWNER=ProtosPrelude.errorPrototype

TEST009-AC:
  11440 -> 9867
  DELTA=-1573
  OWNER=checked local Optional NSEE path

TEST009-AD:
  9867 -> 8294
  DELTA=-1573
  OWNER=delegationParent unsupported-representation UOE path

TEST009-AF:
  8294 -> 6864
  DELTA=-1430
  OWNER=rejectComposedInvocationProjection UOE path
```

Therefore the same selected root has removed:

```text
CUMULATIVE_THROWABLE_FILL_IN_STACK_TRACE_REDUCTION=10439
CURRENT_THROWABLE_FILL_IN_STACK_TRACE=6864
```

without changing observable Protos semantics or weakening the corresponding
runtime failure contracts.

The root still fails `CodeTooLarge`, so TEST009 remains open.

## Remaining selection state

AF closes only the AE-selected owner.

The post-AF evidence does not by itself authorize another implementation.
Previously known residual families still require a fresh causal selection
against current HEAD and post-AF state. In particular:

- `NoSuchElementException.<init>` remains at 4884 / 66 frames, but the post-AF
  aggregate is not yet decomposed in durable evidence to one exact remaining
  owner and mechanism;
- `attachTaskOrInheritDynamicControlState(...)` remains a plausible checked
  then reread Optional candidate, but requires current attribution plus a
  read-stability/observability proof before implementation;
- `ProtosPrelude.arrayPrototype()` must not be turned into the earlier Error
  retention repair if doing so advances lazy validation;
- `PreparedBooleanCall` / generated exhaustive-switch and
  `finishPreparingComposedCall(...)` families still require exact current
  mechanism attribution; and
- `LOCAL_FRAME` was historically large but remains a multi-owner axis unless a
  current post-AF diagnostic isolates one bounded owner/mechanism.

Therefore the next slice is investigation, not implementation.

## Final state

```text
TEST009_AF=COMPLETE

PROTOS_REVISION=6a807bb64d44e86a232803d66da9fb4e46953219
PROTOS_VERSION=0.3.263-SNAPSHOT

SELECTED_OWNER=ProtosBytecodeRootNode.rejectComposedInvocationProjection
SELECTED_REPAIR=CompilerDirectives.transferToInterpreter before existing UOE
TRANSFER_AND_INVALIDATE_USED=NO
NEW_TRUFFLE_BOUNDARY_USED=NO

TARGET_COMPILATION_AFTER=FAILED_CODE_TOO_LARGE
SELECTED_UOE_SUBTREE_AFTER=ABSENT
THROWABLE_FILL_IN_STACK_TRACE_AFTER=6864 / 96_FRAMES
NSEE_CONTROL_AFTER=4884 / 66_FRAMES

CAUSAL_REPAIR_CONFIRMED=YES
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PUBLIC_API_CHANGE=NO

TEST009_COMPLETE=NO
NEXT_SLICE=TEST009-AG
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_SCOPE=select exactly one next post-AF causal owner/mechanism from current evidence; no implementation
```

AI assistance: this durable record was drafted with ChatGPT from the published
AF product commit, the maintainer-reported validation and A/B results, the
existing TEST009 durable evidence, and live TEST009/#795 coordination state.
