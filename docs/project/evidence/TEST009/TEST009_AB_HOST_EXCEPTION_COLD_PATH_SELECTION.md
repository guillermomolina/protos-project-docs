# TEST009-AB — host exception cold-path selection

## Scope

```text
WORK_ITEM=TEST009/#795
SLICE=TEST009-AB
WORK_TYPE=INVESTIGATION
PRODUCT_REPOSITORY=guillermomolina/protos
EXAMINED_PROTOS_REVISION=ddf59b1b3f5c97758354688088845d641f7a4e04
PRODUCT_CHANGES=NONE
COMMAND_EXECUTION=NONE
```

TEST009-AB investigated the remaining host-exception construction visible in the
fixed TEST009 root after TEST009-AA.

The fixed causal root remained:

```text
ROOT_SPEC=protos/tools/test/Manifest.protos
ROOT_LABEL=protos-root:088d2ae81075aba8
TARGET_COMPILATION=FAILED_CODE_TOO_LARGE
THROWABLE_FILL_IN_STACK_TRACE=11440
```

AA had removed one Optional parent-navigation subtree without changing the root
total. The remaining dominant ValueLookup evidence contained two independent
host exception paths:

```text
UnsupportedOperationException subtree ~= 1573
NoSuchElementException subtree          ~= 1573
```

## Exact contract classification

### Unsupported runtime representation

The public Java helper:

```java
throw new UnsupportedOperationException(
        "Standard delegation parent is not implemented for runtime value representation "
                + receiver.getClass().getName());
```

is reachable for a non-Protos runtime representation passed to
`ProtosValueLookup.lookup(...)` / `delegationParent(...)`.

Therefore:

```text
UNSUPPORTED_REPRESENTATION_CLASSIFICATION=REAL_REACHABLE_HOST_FAILURE
PRESERVE_EXCEPTION_CLASS=YES
PRESERVE_MESSAGE=YES
PRESERVE_FAILURE_BEHAVIOR=YES
```

It is not an ordinary lookup miss and must not be converted to
`Optional.empty()`, a guest Error, or a stackless/control-flow carrier.

### Checked local Optional unwrap

The generic lookup loop used the same immutable Optional instance:

```java
Optional<Object> local = ordinary.readLocalSlot(name);
if (local.isPresent()) {
    return Optional.of(new ProtosSlotLookupResult(local.orElseThrow(), ordinary));
}
```

The `NoSuchElementException` path of that exact `orElseThrow()` is logically
impossible: the same Optional has already proved present and Java Optional cannot
contain null.

Therefore:

```text
LOCAL_OPTIONAL_CLASSIFICATION=IMPOSSIBLE_EXCEPTION_PATH
SELECTED_REPAIR=NONTHROWING_EXTRACTION_AFTER_SAME_INSTANCE_PRESENCE_CHECK
SEMANTIC_CHANGE=NO
PUBLIC_API_CHANGE=NO
```

## Cross-Truffle evidence

AB compared current public implementations and framework contracts from:

```text
GraalJS      oracle/graaljs@95a0db94efeb033e841648ecd68e25cf95a2f065
GraalPy      oracle/graalpython@154205b7511c9a51772ab510821b51aaf9216366
TruffleRuby  Shopify/truffleruby@c734f26543003fefd4519a29adbca62d0c711a2d
Graal/Truffle current public framework documentation/source
```

The common result was:

- normal guest/user failures are kept off hot compiled expansion with focused
  cold-path construction, profiling/cutoffs, or framework-supported slow paths;
- explicit invalidation is reserved for cases where the occurrence changes a
  specialization/state assumption or represents an internal unexpected state;
- stackless `ControlFlowException`, `SlowPathException`, and
  `InteropException` contracts are specialized framework mechanisms and are
  not generic replacements for an existing Java `UnsupportedOperationException`;
- a `@TruffleBoundary` that itself throws has exception-transfer semantics, so
  it must not be introduced mechanically merely to hide exception construction.

## Strategy result

The investigated strategies were:

```text
A = cold boundary / slow-path construction
B = transfer to interpreter + invalidation
C = eliminate logically impossible exception construction
D = stackless exception
E = combine multiple owners
```

AB selected exactly C for the next slice.

```text
TEST009_AB_RESULT=SELECT_C_REMOVE_IMPOSSIBLE_CHECKED_LOCAL_OPTIONAL_UNWRAP
TRUFFLE_BOUNDARY_REQUIRED=NO
TRANSFER_TO_INTERPRETER_REQUIRED=NO
INVALIDATION_REQUIRED=NO
STACKLESS_EXCEPTION_REQUIRED=NO
MULTI_OWNER_BATCH=NO
IMPLEMENTATION_READY=YES
```

## Residual owner classification

The remaining owners were intentionally not batched:

| Owner | Host exception construction confirmed | Same generic fillInStackTrace mechanism | Same exact repair as local Optional |
|---|---|---|---|
| `ProtosPrelude.arrayPrototype` | YES | YES | NO |
| `attachTaskOrInheritDynamicControlState` | YES | YES | UNKNOWN |
| `rejectComposedInvocationProjection` | YES | YES | NO |
| `PreparedBooleanCall.hasCallback` | UNKNOWN | UNKNOWN | NO |
| `ProtosPrelude.standardErrorPrototype(String)` | YES | YES | NO |
| `finishPreparingComposedCall` | UNKNOWN | UNKNOWN | NO |

`ProtosActivation.inheritDynamicControlState` was already behind a
`@TruffleBoundary` and was not selected.

## Release

```text
NEXT_SLICE=TEST009-AC
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos
NEXT_SCOPE=REMOVE_ONLY_THE_IMPOSSIBLE_CHECKED_LOCAL_OPTIONAL_EXCEPTION_PATH
NEW_FORMAL_ISSUE_REQUIRED=NO
```

The unsupported-representation UOE remained deliberately untouched until AC
could establish the exact causal delta from removing only the impossible local
Optional path.
