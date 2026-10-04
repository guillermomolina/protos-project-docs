# TEST009-B LocalAccessor / MaterializedLocalAccessor PE guard closure evidence

Date: 2026-10-04

## Exact publication

```text
TEST009_B=COMPLETE
PROTOS_REVISION=38abecd1697993ddba5c0bc549aff662c98af535
COMMIT=TEST009-B: guard LocalAccessor/MaterializedLocalAccessor PE operands
PARENT_ISSUE=TEST009/#795
TRIGGER_PARENT=PERF030/#784
SEMANTIC_CHANGE=NO
```

Published files:

```text
Makefile
tools/java_local_accessor_pe_guard.py
tools/test_java_local_accessor_pe_guard.py
tools/java_local_accessor_pe_baseline.json
```

No production Java or Protos source changed.

## Family covered

TEST009-B adds a dedicated static guard for:

```text
LocalAccessor receiver PE constancy
LocalAccessor supplied BytecodeNode PE constancy
MaterializedLocalAccessor receiver PE constancy
MaterializedLocalAccessor supplied BytecodeNode PE constancy
MaterializedLocalAccessor derived declaring BytecodeNode PE constancy
```

The exact Graal authority used for this contract is:

```text
oracle/graal@95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b
```

For LocalAccessor, the relevant accessors assert that both `this` and the supplied `BytecodeNode` are partial-evaluation constants.

For MaterializedLocalAccessor, the same two assertions apply and the accessor additionally derives the declaring node through the shared BytecodeRootNodes group and asserts that derived `declaringBytecodeNode` is also PE-constant.

## Static proof model

The guard discovers sinks from the inferred static type of the receiver rather than a hand-maintained list of Protos methods.

It reuses the TEST009-A parser, conservative call graph and provenance helpers, while remaining a separate family-specific tool.

A PE path is proven only when the accessor comes from the matching Bytecode DSL `@ConstantOperand` and the supplied node comes from `@Bind("$bytecodeNode")`, directly or through helpers that preserve both values unchanged.

For MaterializedLocalAccessor the declaring-node proof additionally requires that:

- accessor and node belong to the same reached Bytecode DSL root path;
- the operation is declared inside a `@GenerateBytecode` root class; and
- every `begin<Operation>` / `emit<Operation>` builder emission supplies a `BytecodeLocal` at the accessor operand position.

Risks and unknowns are never baselineable. The baseline records only exact proven topology.

During development a self-test exposed one analyzer bug: a receiver produced by an inherited external-class method had been classified as not-an-accessor. The guard was corrected to classify that case as unresolved receiver type and fail closed.

## Final inventory

```text
TOTAL_LOCAL_ACCESSOR_SINKS=14
TOTAL_MATERIALIZED_ACCESSOR_SINKS=5

PE_REACHABLE_LOCAL_ACCESSOR_RECEIVER_RISK=0
PE_REACHABLE_LOCAL_ACCESSOR_BYTECODE_NODE_RISK=0

PE_REACHABLE_MATERIALIZED_ACCESSOR_RECEIVER_RISK=0
PE_REACHABLE_MATERIALIZED_BYTECODE_NODE_RISK=0
PE_REACHABLE_MATERIALIZED_DECLARING_NODE_RISK=0

PE_REACHABILITY_UNKNOWN=0
OLD_CODE_FAMILY_DEBT_REMAINING=0
```

All 19 production sinks are statically proven on every required dimension.

No product repair was required:

```text
PRODUCT_REPAIR_COUNT=0
TRUFFLE_BOUNDARY_ADDED=0
```

## Make integration and runtime

Published target:

```text
make check-local-accessor-pe
```

The target runs the 21 guard self-tests followed by the whole-`src/main/java` analysis.

Measured runtime reported by the maintainer:

```text
LOCAL_ACCESSOR_GUARD_RUNTIME_SECONDS=1.87
```

The published Makefile adds the new target as a prerequisite of aggregate `make check`, after the two previously admitted TEST009-A guards.

## Validation

Maintainer-reported validation:

```text
git diff --check=PASS
NEW_SELF_TESTS=21/21_PASS
LOCAL_ACCESSOR_GUARD=PASS
LOCAL_RANGE_INDEX_GUARD=PASS
LOCAL_RANGE_OPERAND_GUARD=PASS
PRODUCTION_UNRESOLVED_SINKS=0
```

Maven, Java and Protos suites were not rerun because no Java or Protos source changed.

The aggregate `make check` command itself was not rerun after the Makefile-only admission. The published Makefile was verified to compose the already-passing `check-local-accessor-pe` target with the existing toolchain, TEST009-A guards and `test` prerequisites. This record does not claim a fresh aggregate-suite run.

## Resulting state

```text
TEST009_B=COMPLETE
OLD_CODE_FAMILY_DEBT_REMAINING=0
DESIGN_DECISION_REQUIRED=NO
SEMANTIC_CHANGE=NO
PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
```

The next family should be selected from an uncovered explicit Graal PE-constant contract rather than from a synthetic slice boundary.
