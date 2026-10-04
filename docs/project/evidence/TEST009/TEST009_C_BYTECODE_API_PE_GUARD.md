# TEST009-C Bytecode API PE guard closure evidence

Date: 2026-10-04

## Exact publications

TEST009-C was completed in two publications because review of the first publication exposed a same-family coverage gap.

```text
INITIAL_REVISION=bdeed4534beeba74949c2c65d74d7eb89d95b589
INITIAL_COMMIT=TEST009-C: guard Bytecode API PE arguments

FINAL_REVISION=bb3a15b2c529368879eeed527356bac0bb9e7d6e
FINAL_COMMIT=TEST009-C: complete Bytecode local API PE coverage

TEST009_C=COMPLETE
PARENT_ISSUE=TEST009/#795
TRIGGER_PARENT=PERF030/#784
SEMANTIC_CHANGE=NO
```

## Initial product repair

The first TEST009-C publication introduced a static Bytecode API PE-argument guard and found two real old-code risks in `ProtosBytecodeTagTreeNodeExports`.

Both were debugger/tooling paths reachable from `@ExportMessage` entry points. They called `BytecodeNode.getLocalNames(bytecodeIndex)` using a runtime value from `TagTreeNode.getEnterBytecodeIndex()`.

No PE-constant bytecode index exists naturally for those tooling queries, so the two helper seams were moved behind `@TruffleBoundary`:

```text
inlineCallbackFrameBindings
inlineCallbackCall
```

Their callers now pass a materialized frame. Debugger-visible values, names, frame contents, error behavior and Protos semantics are unchanged.

The unrepaired source was verified to produce exactly those two risks under the new guard.

## Same-family coverage correction

The first guard version covered the originally enumerated Bytecode API methods but omitted generated local-slot methods from the same upstream contract.

Review against:

```text
oracle/graal@95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b
```

showed that the generated implementations of:

```text
BytecodeNode.getLocalValue(int, Frame, int)
BytecodeNode.setLocalValue(int, Frame, int, Object)
BytecodeNode.getLocalName(int, int)
BytecodeNode.getLocalInfo(int, int)
BytecodeNode.getLocalCount(int)
```

also require PE-constant arguments. The get/set/name/info implementations use the generated `buildVerifyLocalsIndex` path, which asserts `bci` and `localOffset`; `getLocalCount` asserts `bci`.

TEST009-C therefore remained open and the same slice was extended instead of creating a new slice.

The final publication adds those methods to the same guard and self-test surface.

## Final production coverage

The final baseline contains seven production sinks.

The two production `getLocalValue(bytecodeIndex, frame, offset)` calls are now explicitly inventoried and classified as `BOUNDARY_CUT` inside the two debugger helpers repaired by the initial TEST009-C publication.

Final result:

```text
BYTECODE_API_PE_GUARD=PASS
PE_REACHABLE_BYTECODE_INDEX_RISK=0
PE_REACHABLE_LOCAL_OFFSET_RISK=0
PE_REACHABLE_LOCAL_COUNT_RISK=0
PE_REACHABLE_NODE_ARGUMENT_RISK=0
PE_REACHABLE_BYTECODE_CONFIG_RISK=0
PE_REACHABILITY_UNKNOWN=0
OLD_CODE_FAMILY_DEBT_REMAINING=0
PRODUCT_REPAIR_COUNT=2
```

The guard now covers:

```text
BytecodeNode.getLocalValues
BytecodeNode.getLocalNames
BytecodeNode.getLocalInfos
BytecodeNode.setLocalValues
BytecodeNode.copyLocalValues
BytecodeNode.getLocalValue
BytecodeNode.setLocalValue
BytecodeNode.getLocalName
BytecodeNode.getLocalInfo
BytecodeNode.getLocalCount
BytecodeNode.get(Node)
BytecodeRootNodes.update(BytecodeConfig)
BytecodeLocation.get(...)
```

Risks and unknowns are never baselineable.

## Make integration

The family remains exposed as:

```text
make check-bytecode-api-pe
```

and is included in aggregate `make check`.

The previously reported measured runtime for this deterministic standard-library guard is approximately 1.9 seconds.

## Validation

The maintainer reports that all local tests passed after the final TEST009-C continuation.

The final continuation changed only:

```text
tools/java_bytecode_api_pe_guard.py
tools/test_java_bytecode_api_pe_guard.py
tools/java_bytecode_api_pe_baseline.json
```

The earlier product repair had already passed the required validation before its version/changelog publication, and no tests were run after that publication's final `pom.xml` / `CHANGELOG.md` edit.

This record treats the final all-tests-pass statement as maintainer-reported evidence; it does not claim assistant execution.

## Resulting state

```text
TEST009_C=COMPLETE
BYTECODE_API_FAMILY=CLEAN
SEMANTIC_CHANGE=NO
PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
```

The next known family is the Bytecode DSL generated cached-dispatch `bci` invariant. Unlike A-C, the assertion lives in annotation-processor-generated interpreter code and therefore requires inspection of generated Java topology rather than only handwritten `src/main/java`.
