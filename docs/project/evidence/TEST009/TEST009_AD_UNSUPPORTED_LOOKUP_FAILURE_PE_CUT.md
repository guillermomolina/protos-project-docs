# TEST009-AD — unsupported lookup failure PE cut

## Scope

```text
WORK_ITEM=TEST009/#795
SLICE=TEST009-AD
WORK_TYPE=IMPLEMENTATION
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=92db72eb8b1e5d66f316a7e9f72a8c7023a28278
PROTOS_VERSION=0.3.261-SNAPSHOT
COMMIT_SUBJECT=TEST009-AD: keep unsupported lookup failure out of PE
```

TEST009-AD implements the single causal repair released after TEST009-AC: keep
the real, reachable unsupported-runtime-representation failure in
`ProtosValueLookup.delegationParent(...)` semantically unchanged while cutting
its exception-construction subtree out of Truffle partial evaluation.

## Published product change

The published product change is intentionally minimal:

```java
CompilerDirectives.transferToInterpreter();
throw new UnsupportedOperationException(
        "Standard delegation parent is not implemented for runtime value representation "
                + receiver.getClass().getName());
```

The existing exception is still constructed and thrown at the same semantic
position. AD does not:

- convert the failure into a lookup miss;
- change the exception class;
- change the exception message;
- use `transferToInterpreterAndInvalidate()`;
- add a new `@TruffleBoundary`;
- use a stackless/control-flow carrier; or
- modify any other TEST009 residual owner.

The published revision changes:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/runtime/ProtosValueLookup.java
```

## Maintainer-reported validation

The maintainer reported:

```text
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
FULL_SUITE=PASS
PUBLICATION=PUSHED
```

The change touches shared runtime Java, so the full integrated gate was required.

## Exact causal A/B result

The same logical Case and the same fixed compiler root were compared before and
after AD:

```text
ROOT_SPEC=protos/tools/test/Manifest.protos
ROOT_LABEL=protos-root:088d2ae81075aba8
```

| Metric | post-AC | post-AD | Delta |
|---|---:|---:|---:|
| `TARGET_COMPILATION` | `FAILED_CODE_TOO_LARGE` | `FAILED_CODE_TOO_LARGE` | unchanged |
| `Throwable.fillInStackTrace` size | 9867 / 138 frames | 8294 / 116 frames | **-1573 / -22 frames** |
| `UnsupportedOperationException.<init>` cumulative | 3108 / 42 frames | 1480 / 20 frames | **-1628 / -22 frames** |
| `rejectComposedInvocationProjection` | 403 / 1693 / 1592 | 403 / 1693 / 1592 | unchanged |
| `NoSuchElementException.<init>` | 4884 / 66 frames | 4884 / 66 frames | unchanged |
| `delegationParent` cumulative | 10068 | 374 | selected throw subtree removed |

The remaining UOE cumulative size of 1480 matches the untouched
`rejectComposedInvocationProjection` owner.

The selected `delegationParent` UOE subtree therefore disappears exactly as
intended:

```text
DELEGATION_PARENT_UOE_FRAMES_BEFORE=22
DELEGATION_PARENT_UOE_FRAMES_AFTER=0
DELEGATION_PARENT_UOE_CUMULATIVE_BEFORE=1628
DELEGATION_PARENT_UOE_CUMULATIVE_AFTER=0

THROWABLE_FILL_IN_STACK_TRACE_DELTA=-1573
```

The `-1573` `fillInStackTrace` reduction exactly matches the prior causal
estimate for this subtree.

A secondary graph-shape change was observed:

```text
PROTOS_VALUE_LOOKUP_SELF_SIZE_BEFORE=451
PROTOS_VALUE_LOOKUP_SELF_SIZE_AFTER=396
PROTOS_VALUE_LOOKUP_TREE_OCCURRENCES_BEFORE=192
PROTOS_VALUE_LOOKUP_TREE_OCCURRENCES_AFTER=165
```

AD did not edit `lookup()`. This is treated as graph reorganization caused by
removing the dominated `delegationParent` failure subtree, not as a second
repair.

## Contract preservation

AD preserves the contract established by AB and exercised by AC's Java
regression coverage:

```text
UNSUPPORTED_REPRESENTATION_EXCEPTION_CLASS_PRESERVED=YES
UNSUPPORTED_REPRESENTATION_EXCEPTION_MESSAGE_PRESERVED=YES
UNSUPPORTED_REPRESENTATION_FAILURE_BEHAVIOR_PRESERVED=YES
PUBLIC_API_CHANGE=NO
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
```

The local Optional repair from AC also remains intact:

```text
LOCAL_CHECKED_OPTIONAL_NSEE_SUBTREE_AFTER=0
```

The separately reachable composed-invocation UOE remains unchanged.

## Cumulative single-root causal progress

The current single-root campaign began at the W checkpoint with:

```text
THROWABLE_FILL_IN_STACK_TRACE_W=17303
```

Successive isolated repairs produced:

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
```

Therefore the same root has removed exactly:

```text
CUMULATIVE_THROWABLE_FILL_IN_STACK_TRACE_REDUCTION=9009
CURRENT_THROWABLE_FILL_IN_STACK_TRACE=8294
```

without changing observable Protos language semantics or weakening the runtime
failure contracts.

The root still fails `CodeTooLarge`, so TEST009 remains open.

## Current issue-level accomplishments

TEST009 has now delivered several durable layers of compilerability
infrastructure and methodology:

1. family-specific static PE guards rather than one generic bailout claim;
2. a dynamic generated-bytecode/BCI compilerability check;
3. validated cold-host PE cuts for demonstrated hot-path compiler expansion;
4. a strict Truffle compilation gate in synchronous and background modes;
5. separation of primary compiler failures from shutdown cascades;
6. a systematic diagnostic procedure based on real Truffle compilation,
   expansion/inlining evidence, and causal classification;
7. single-root selection/capture with stable diagnostic identity and BGV output;
8. a version-aligned headless BGV analyzer in `guillermomolina/protos-benchmarks`;
9. a durable single-root `CodeTooLarge` causal checkpoint;
10. rejection and rollback of non-maintainable/global speculative compiler
    policies when evidence falsified them; and
11. three consecutive, isolated, causally measured product repairs on the same
    selected root.

TEST009 is not complete. The selected root still fails `CodeTooLarge`, and
remaining independent expansion debt must continue to be selected one causal
owner at a time. Earlier evidence also identified `LOCAL_FRAME` as a separate
major axis; it remains independent from the host defensive exception reductions
recorded here and must not be conflated with them.

## Final state

```text
TEST009_AD=COMPLETE
PROTOS_REVISION=92db72eb8b1e5d66f316a7e9f72a8c7023a28278
PROTOS_VERSION=0.3.261-SNAPSHOT

SELECTED_MECHANISM=CompilerDirectives.transferToInterpreter
TRANSFER_AND_INVALIDATE_USED=NO
NEW_TRUFFLE_BOUNDARY_USED=NO
STACKLESS_EXCEPTION_USED=NO

TARGET_COMPILATION=FAILED_CODE_TOO_LARGE

THROWABLE_FILL_IN_STACK_TRACE_BEFORE=9867
THROWABLE_FILL_IN_STACK_TRACE_AFTER=8294
THROWABLE_FILL_IN_STACK_TRACE_DELTA=-1573

DELEGATION_PARENT_UOE_SUBTREE_REMOVED=YES
LOCAL_OPTIONAL_NSEE_SUBTREE_REMAINS_REMOVED=YES
REJECT_COMPOSED_INVOCATION_UOE_CHANGED=NO

GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
FULL_SUITE=PASS

SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PUBLIC_API_CHANGE=NO

TEST009_COMPLETE=NO
NEXT_CAUSAL_OWNER=NOT_YET_SELECTED
```
