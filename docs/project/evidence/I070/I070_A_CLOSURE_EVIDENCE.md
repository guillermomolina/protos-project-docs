# I070-A handwritten-main warning cleanup closure evidence

I070-A / `guillermomolina/protos#713` is the handwritten-main child of
I070 / #712. This record binds the completed child to the exact published
product revision and the maintainer-reported validation.

## Exact product publication

```text
PROTOS_REVISION=7f398f623a210b2bc3837d37dd3e5980dd726e75
PROTOS_VERSION=0.3.222-SNAPSHOT
COMMIT=I070-A: eliminate handwritten main compilation warnings
ISSUE=#713
PARENT=#712
NATIVE_PARENT_VERIFIED=PASS
```

The native GitHub parent relation was re-verified by querying the children of
`guillermomolina/protos#712`; #713 is present together with sibling children
#714 and #715.

The publication supersedes the earlier bounded A1 publication:

```text
I070_A1_REVISION=a5d3fe00c941c29f0a4e80eb262be610ff01443b
I070_A1_VERSION=0.3.220-SNAPSHOT
```

Concurrent product work advanced the repository to 0.3.221-SNAPSHOT before the
final I070-A publication, so the final I070-A metadata correctly advances the
actual then-current version to 0.3.222-SNAPSHOT.

## Published outcome

The exact root changelog at the published revision records:

```text
HANDWRITTEN_MAIN_WARNINGS_BEFORE=179
HANDWRITTEN_MAIN_WARNINGS_AFTER=0
XLINT=all
TRUFFLE_DSL_PROCESSOR=enabled
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
```

The completed cleanup covers all handwritten-main warning families owned by
#713:

- stale dangling documentation comments were removed by the preceding A1
  publication;
- stateless serializable-by-inheritance exceptions now declare
  `serialVersionUID = 1L`;
- exceptions whose Java serializability is accidental and whose state is
  runtime-only use narrow class-scoped `@SuppressWarnings("serial")` rather
  than invented Java-serialization semantics;
- Bytecode-root context helpers are classified `@NonIdempotent`, matching the
  Truffle `ContextReference.get` contract;
- diagnosed redundant `@Bind("$bytecodeNode")` /
  `@Bind("$frame")` expressions use the canonical type-inferred `@Bind`
  form while retaining the bound parameters;
- LocalRangeAccessor operand, LocalAccessor, and Bytecode API PE guards accept
  canonical bare `@Bind BytecodeNode` as the same structural proof as the
  former explicit expression, with corresponding self-test coverage.

The published revision changes:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeControlTransferException.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosParallelRuntime.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosTaskCancellationException.java
src/main/java/com/guillermomolina/protos/execution/ProtosTestResourceProviderCoordinator.java
src/main/java/com/guillermomolina/protos/execution/ProtosTestResourceProviderRegistry.java
src/main/java/com/guillermomolina/protos/parser/ParseError.java
src/main/java/com/guillermomolina/protos/runtime/ProtosEncodingValue.java
src/main/java/com/guillermomolina/protos/runtime/ProtosNonLocalReturnException.java
src/main/java/com/guillermomolina/protos/runtime/ProtosSignalException.java
tools/java_bytecode_api_pe_guard.py
tools/java_local_accessor_pe_guard.py
tools/java_local_range_operand_pe_guard.py
tools/test_java_bytecode_api_pe_guard.py
tools/test_java_local_range_operand_pe_guard.py
```

No generated source is edited directly.

## Validation and publication evidence

The maintainer reports for the published candidate:

```text
GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
PRODUCT_PUSH=PASS
```

The exact GitHub commit currently exposes no combined status checks and no
pull-request workflow runs. No CI result is claimed beyond the maintainer's
reported local validation.

The publication includes the required version bump and root changelog entry:

```text
VERSION_INCREMENT=PASS
CHANGELOG_ENTRY=PASS
FINAL_VERSION=0.3.222-SNAPSHOT
```

## Closure

```text
I070_A_ACCEPTANCE=PASS
ACTIONABLE_HANDWRITTEN_MAIN_WARNINGS=0
FOCUSED_STATIC_COMPILATION_EVIDENCE=PASS
PRODUCT_PUBLICATION=PASS
LOCAL_REQUIRED_TESTS=PASS
GIT_DIFF_CHECK=PASS
NATIVE_PARENT=#712
NEXT_I070_A_SLICE=NONE
READY_TO_CLOSE_713=YES
```

I070 itself remains open because sibling children #714 (Java test-source
warnings) and #715 (generated-source / annotation-processing warning
disposition) remain unresolved.

This is non-normative project evidence. Live lifecycle state remains owned by
GitHub Issues.
