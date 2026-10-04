# TEST009-A LocalRange operand PE guard closure evidence

Date: 2026-10-04

## Exact publication

```text
TEST009_A=COMPLETE
PROTOS_REVISION=2da6622eb4d6a570ab3e579183ae3b840af1f7b7
COMMIT=TEST009-A: guard LocalRange PE operands
VERSION=0.3.196-SNAPSHOT
PARENT_ISSUE=TEST009/#795
TRIGGER_PARENT=PERF030/#784
```

Published files:

```text
Makefile
pom.xml
CHANGELOG.md
src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java
tools/java_local_range_pe_guard_baseline.json
tools/java_local_range_pe_reachability_baseline.json
tools/java_local_range_operand_pe_guard.py
tools/test_java_local_range_operand_pe_guard.py
tools/java_local_range_operand_pe_baseline.json
```

## Family discovered and repaired

TEST009-A added a static guard for the two `LocalRangeAccessor` partial-evaluation constant dimensions not owned by the earlier index guard:

```text
LocalRangeAccessor receiver
BytecodeNode argument
```

The guard reuses the existing LocalRange index guard's lexer, sink inventory and conservative call graph while remaining a separate family-specific tool.

The old code contained five PE-reachable risky sinks across four methods of `ProtosFrameLexicalBindingAuthority`:

```text
hasFrameBackedBindingAt
readFrameBackedBindingAt
assignFrameBackedBindingAt
createFrameBackedBindingAt  (two sinks)
```

At these seams the accessor came from a field of a per-invocation authority object and the BytecodeNode was re-derived from `declaringRoot.getBytecodeNode()`. Those values were not structurally proven PE constants.

All four entry methods were moved behind `@TruffleBoundary`. This preserves the existing code, order, exceptions and PRESENT/ABSENT behavior while preventing the unsafe LocalRange accesses from participating in partial evaluation.

This is a compilerability repair, not proof that the boundary is the fastest possible implementation. A future performance optimization may choose to carry constant accessor/node operands explicitly through same-root indexed operations and retain an identity check, but that is not required to close this guard family.

## Guard results

```text
TOTAL_LOCAL_RANGE_SINKS=36

PE_REACHABLE_RECEIVER_PROVEN=8
PE_REACHABLE_RECEIVER_RISK=0

PE_REACHABLE_BYTECODE_NODE_PROVEN=8
PE_REACHABLE_BYTECODE_NODE_RISK=0

BOUNDARY_CUT=24
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0

OLD_CODE_FAMILY_DEBT_REMAINING=0
```

The topology baseline records only determinate PE-safe or cut/non-reachable sites. Risks are not accepted into the baseline.

## Make integration

Published targets:

```text
make check-local-range-index-pe
make check-local-range-operands-pe
make test-local-range-pe-guard   # compatibility alias
```

Measured runtimes reported by the maintainer:

```text
INDEX_GUARD_RUNTIME_SECONDS=1.97
OPERAND_GUARD_RUNTIME_SECONDS=1.81
```

Both deterministic static guards were admitted into aggregate `make check`.

## Validation

Maintainer-reported validation:

```text
LOCAL_RANGE_INDEX_GUARD=PASS
LOCAL_RANGE_OPERAND_GUARD=PASS
FOCUSED_TESTS=PASS
FULL_TESTS=PASS
REQUIRED_REPOSITORY_VALIDATION=PASS
MAKE_CHECK_EXIT=0

JAVA_TESTS_RUN=2504
JAVA_TESTS_FAILED=0
JAVA_TESTS_SKIPPED=1

PROTOS_TESTS_PASSED=1328
PROTOS_TESTS_FAILED=0
```

No tests were run after the final `pom.xml` and `CHANGELOG.md` edit, in accordance with repository publication discipline.

## Resulting state

```text
SEMANTIC_CHANGE=NO
PRODUCT_REPAIR_COUNT=4_methods_5_sinks
LOCAL_RANGE_RECEIVER_NODE_FAMILY=CLEAN
PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
```

PERF030 remains open because compilerability cleanup is not equivalent to proving final optimization quality. The newly added boundaries may remove a permanent bailout while also keeping some work out of partial evaluation.

## Next TEST009 family

The next identified family is:

```text
LocalAccessor / MaterializedLocalAccessor receiver + BytecodeNode PE constancy
```

This family is not covered by the LocalRange guards and should receive its own detector, whole-code old-debt cleanup, measured target, and eventual aggregate-`make check` admission decision under the same TEST009 model.
