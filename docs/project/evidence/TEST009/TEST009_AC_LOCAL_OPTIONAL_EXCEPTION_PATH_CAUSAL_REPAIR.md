# TEST009-AC — local Optional exception-path causal repair

## Scope

```text
WORK_ITEM=TEST009/#795
SLICE=TEST009-AC
WORK_TYPE=IMPLEMENTATION
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=08cfc69477ad45e2fe91a95140e5c71dffb9cdc0
PROTOS_VERSION=0.3.259-SNAPSHOT
COMMIT_SUBJECT=TEST009-AC: remove impossible local lookup exception path
```

TEST009-AC implements exactly the repair selected by TEST009-AB: remove the
logically impossible `NoSuchElementException` construction path from the
checked local-slot Optional unwrap in the generic authoritative lookup loop.

## Published product change

The published revision changes:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/runtime/ProtosValueLookup.java
src/test/java/com/guillermomolina/protos/runtime/ProtosRepresentedValueLookupTest.java
```

The production change is intentionally one line of behavior:

```java
Optional<Object> local = ordinary.readLocalSlot(name);
if (local.isPresent()) {
    return Optional.of(new ProtosSlotLookupResult(local.orElse(null), ordinary));
}
```

The same immutable Optional has already proved present; `orElse(null)` therefore
returns the same non-null value while presenting no `NoSuchElementException`
construction path to partial evaluation.

No parent/delegation Optional handling was changed. The real unsupported runtime
representation failure remains:

```java
throw new UnsupportedOperationException(
        "Standard delegation parent is not implemented for runtime value representation "
                + receiver.getClass().getName());
```

Focused Java regression coverage preserves:

- generic own-slot value identity;
- exact slot home identity;
- normal lookup miss;
- unsupported-representation exception class; and
- unsupported-representation exception message.

## Maintainer-reported validation

The maintainer reported:

```text
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
FULL_SUITE=PASS
PUBLICATION=PUSHED
```

The full suite was required because the slice changes shared runtime production
code.

No tests were run after the late `pom.xml` / `CHANGELOG.md` finalization step.

## Exact causal A/B result

The same logical Case and the same fixed root were compared before and after AC.
The A/B candidate differed only in the selected production line.

```text
ROOT_SPEC=protos/tools/test/Manifest.protos
ROOT_LABEL=protos-root:088d2ae81075aba8
```

| Metric | Before | After |
|---|---:|---:|
| `TARGET_COMPILATION` | `FAILED_CODE_TOO_LARGE` | `FAILED_CODE_TOO_LARGE` |
| `Throwable.fillInStackTrace` self size | 11440 | 9867 |
| delta | — | **-1573** |
| local NSEE subtree | 22 frames / cumulative 1628 | **0** |
| delegation-parent UOE subtree | 22 frames / cumulative 1628 | 22 frames / cumulative 1628 |
| `rejectComposedInvocationProjection` UOE subtree | 1480 | 1480 |
| `ProtosValueLookup.lookup` self size | 451 | 451 |

The measured `fillInStackTrace` reduction is exactly `1573`, matching the AB
estimate for the selected local Optional exception-construction subtree.

The checked-local NSEE subtree disappears completely while the real
delegation-parent UOE and the untouched residual owners remain unchanged.

Therefore:

```text
TEST009_AC_CAUSAL_REPAIR_CONFIRMED=YES
LOCAL_OPTIONAL_EXCEPTION_PATH_REMOVED=YES
LOCAL_NSEE_SUBTREE_AFTER=0
THROWABLE_FILL_IN_STACK_TRACE_BEFORE=11440
THROWABLE_FILL_IN_STACK_TRACE_AFTER=9867
THROWABLE_FILL_IN_STACK_TRACE_DELTA=-1573
UNSUPPORTED_REPRESENTATION_FAILURE_PRESERVED=YES
OTHER_MEASURED_RESIDUAL_OWNERS_CHANGED=NO
TARGET_COMPILATION=FAILED_CODE_TOO_LARGE
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PUBLIC_API_CHANGE=NO
```

The root remaining `CodeTooLarge` is an expected parent-work result and does not
invalidate AC.

## Next causal repair selection

AC leaves one directly adjacent, independently attributable ValueLookup subtree
unchanged: the real unsupported-representation `UnsupportedOperationException`
constructed in `ProtosValueLookup.delegationParent(...)`.

TEST009-AB already established that this exception is reachable and its class,
message, and failure behavior must be preserved. AC now establishes that it is
independent from the removed local Optional path.

Current Truffle 25.4 framework guidance supplies the narrow implementation
mechanism:

- `CompilerDirectives.transferToInterpreter()` discontinues compilation at the
  marked position and transfers execution to the interpreter;
- unlike `transferToInterpreterAndInvalidate()`, it does not invalidate the
  currently executing machine code;
- Truffle 25.4 host-optimization guidance states that compilation boundaries are
  inferred from directives such as `transferToInterpreter()` and
  `@TruffleBoundary`;
- a throwing `@TruffleBoundary` has additional exception-transfer/invalidation
  semantics by default, so it is not selected for this real Java failure.

The next bounded repair is therefore to keep the existing UOE construction and
throw in the same method and same semantic position, but precede that exact rare
failure branch with `CompilerDirectives.transferToInterpreter()`.

This avoids changing the exception class/message into a specialized stackless
carrier and avoids unnecessary invalidation.

```text
NEXT_SLICE=TEST009-AD
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos
NEXT_SCOPE=CUT_ONLY_THE_REAL_UNSUPPORTED_REPRESENTATION_UOE_BRANCH_OUT_OF_PE
SELECTED_MECHANISM=CompilerDirectives.transferToInterpreter
TRANSFER_TO_INTERPRETER_AND_INVALIDATE=NO
TRUFFLE_BOUNDARY_SELECTED=NO
STACKLESS_EXCEPTION_SELECTED=NO
MULTI_OWNER_BATCH=NO
NEW_FORMAL_ISSUE_REQUIRED=NO
```

AD must remeasure the same fixed root before any other owner is considered.
