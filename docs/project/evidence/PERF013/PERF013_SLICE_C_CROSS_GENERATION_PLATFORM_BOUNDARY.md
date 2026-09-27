# PERF013 Slice C — cross-generation materialized-local platform boundary

Date: 2026-09-27

## Scope

```text
WORK_ITEM=PERF013/#724
PARENT=PERF010-B/#722

PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=0af8960363a557dad1b87968cf8a632e4716ee8a
PRODUCT_VERSION=0.3.102-SNAPSHOT

SLICE=PERF013-C_CONTEXT_LOCAL_GROUP_REMATERIALIZATION
RESULT=IMPLEMENTATION_PREMISE_FALSIFIED
PRODUCT_CHANGE=NONE
SEMANTIC_CHANGE=NO
```

Slice C was activated after the B2 compiler gate removed the captured-read and
captured-write `CompilerAsserts.partialEvaluationConstant` failures. Its proposed
implementation premise was that an isolated/context-local Closure rebuild could
reconstruct a structurally equivalent lexical `BytecodeRootNodes` group and then
use newly generated `MaterializedLocalAccessor` operands against the
`MaterializedFrame` already captured by the semantic Closure.

That premise is not valid for the current Truffle Bytecode DSL identity model.

## Platform fact

Pinned upstream evidence:

```text
GRAAL_REPOSITORY=oracle/graal
GRAAL_REVISION=758b75e32ba6fc493da91459ca95782bc65b093c
```

At that revision,
`com.oracle.truffle.api.bytecode.MaterializedLocalAccessor` stores logical
`rootIndex/localOffset/localIndex` identity. At execution time it resolves the
declaring `BytecodeNode` through the current bytecode root's
`BytecodeRootNodes` group.

The generated local-access implementation validates the supplied materialized
frame against the declaring root by exact `FrameDescriptor` identity:

```java
getRoot().getFrameDescriptor() == frame.getFrameDescriptor()
```

The validation is not structural equivalence. A fresh `create()` invocation
produces a new generated root group and fresh frame-descriptor identities even
when the reconstructed lexical names, nesting, depth, and local ordering are
structurally identical.

Therefore a `MaterializedLocalAccessor` generated for group G2 cannot safely
operate on a retained materialized frame produced by the corresponding owner
root in group G1.

Relevant upstream files:

- `truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/MaterializedLocalAccessor.java`
- `truffle/src/com.oracle.truffle.dsl.processor/src/com/oracle/truffle/dsl/processor/bytecode/generator/AbstractBytecodeNodeElement.java`
- `truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/GenerateBytecode.java`
- `truffle/docs/bytecode_dsl/UserGuide.md`

## Protos evidence already present at B2

The published B1/B2 implementation already encodes this distinction.

At product revision
`0af8960363a557dad1b87968cf8a632e4716ee8a`:

- same-generation proven captures whose lexical owner is available in the
  current shared `BytecodeRootNodes` group use the
  `MaterializedLocalAccessor` fast path;
- isolated Closure rebuild/rematerialization where the lexical owner is not in
  that same generated group retains the runtime-authority fallback;
- the B1 and B2 focal tests explicitly retain that isolated-rebuild behavior;
- the product changelog explicitly states that the runtime-authority path
  remains necessary for the isolated/rematerialized case.

Relevant product files:

- `src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf013SliceB1MaterializedCapturedReadTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf013SliceB2MaterializedCapturedWriteTest.java`
- `CHANGELOG.md`

## Slice-C implementation attempt

The active Slice-C implementation attempt reconstructed the whole lexical module
group, matched the projected Closure back to the reconstructed group, and wired
the context-local rebuild through that coherent group. The source compiled, but
runtime captured reads/writes against the already-captured frame failed with the
Bytecode DSL descriptor-identity assertion:

```text
java.lang.AssertionError: Invalid frame with invalid descriptor passed.
  at ...AbstractBytecodeNode.isLocalClearedInternal
  at MaterializedLocalAccessor.isCleared
  at ...ReadCapturedMaterializedLocal.perform
  / ResolveCapturedMaterializedWritableLexicalTarget.perform
```

The same root cause affected existing reparse coverage. The attempted changes
were reverted; no Slice-C product commit was published and
`guillermomolina/protos` remained at
`0af8960363a557dad1b87968cf8a632e4716ee8a`.

This runtime attempt is retained as maintainer/implementation-session evidence;
the platform conclusion does not depend on the attempt because the exact
descriptor-identity requirement is independently established by upstream
source.

## Mature Truffle-language comparison

The surrounding Truffle ecosystem reinforces the same lifetime model rather
than providing a precedent for cross-generation accessor/frame mixing.

### GraalJS

```text
REPOSITORY=oracle/graaljs
REVISION=fc826cc4d2886c94d8f84e045bb09ca7618e6c1f
```

A closure-producing `JSFunctionExpressionNode` materializes the active frame
and creates a function object carrying that frame. `JSFunctionObject` stores
both its `JSFunctionData` and its enclosing `MaterializedFrame`. Lexical
scope traversal uses the retained enclosing-frame chain; it does not construct
a new accessor generation and apply it to an old captured frame.

Relevant files:

- `graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/function/JSFunctionExpressionNode.java`
- `graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/runtime/builtins/JSFunctionObject.java`

### TruffleRuby

```text
REPOSITORY=truffleruby/truffleruby
REVISION=c734f26543003fefd4519a29adbca62d0c711a2d
```

`RubyProc` retains the executable and environment together:

```text
RootCallTarget callTarget
MaterializedFrame declarationFrame
```

The declaration frame is passed through Ruby's internal frame arguments and is
the runtime lexical environment associated with the proc.

Relevant files:

- `src/main/java/org/truffleruby/core/proc/RubyProc.java`
- `src/main/java/org/truffleruby/language/arguments/RubyArguments.java`

### Apple Pkl

```text
REPOSITORY=apple/pkl
REVISION=e83d8fa14b82dfe1925536dd304a9c21cbb3238d
```

`FunctionLiteralNode` constructs a `VmFunction` from
`frame.materialize()` and the corresponding `PklRootNode`. `VmFunction`
retains that root, while its `VmObjectLike` base retains the enclosing
materialized frame that was active when the object was instantiated.

Relevant files:

- `pkl-core/src/main/java/org/pkl/core/ast/expression/literal/FunctionLiteralNode.java`
- `pkl-core/src/main/java/org/pkl/core/runtime/VmFunction.java`
- `pkl-core/src/main/java/org/pkl/core/runtime/VmObjectLike.java`

## Corrected PERF013 boundary

The supported boundary is:

```text
same BytecodeRootNodes generation
+ owner BytecodeLocal available in that generated group
+ captured frame has the matching FrameDescriptor identity
    -> MaterializedLocalAccessor fast path

cross-generation / isolated rematerialization
+ semantic Closure retains an already-existing captured frame
    -> runtime lexical-authority fallback
```

This is a backend identity/lifetime constraint, not a Protos semantic change.

Executing a newly reconstructed lexical owner does not solve the original
problem by itself: it would create a new frame compatible with the new
descriptor, but the semantic Closure's authoritative captured state is still in
the old frame. Migrating state and changing the authoritative captured
environment would be a different architecture and is outside PERF013.

## Slice-C resolution

```text
PERF013_C_IMPLEMENTATION=NOT_APPLICABLE
PERF013_C_ORIGINAL_ACCEPTANCE=UNACHIEVABLE_WITH_CURRENT_BYTECODE_DSL_IDENTITY_MODEL

SAME_GENERATION_PROVEN_READ_MATERIALIZED_LOCAL_ACCESSOR=YES
SAME_GENERATION_PROVEN_WRITE_MATERIALIZED_LOCAL_ACCESSOR=YES

CROSS_GENERATION_REMATERIALIZED_READ_RUNTIME_AUTHORITY_FALLBACK=RETAINED
CROSS_GENERATION_REMATERIALIZED_WRITE_RUNTIME_AUTHORITY_FALLBACK=RETAINED

CROSS_GENERATION_ACCESSOR_FRAME_MIXING=FORBIDDEN
COPY_CAPTURED_VALUES_TO_SECOND_AUTHORITY=NO
STORE_TRUFFLE_FRAME_IN_SEMANTIC_CLOSURE=NO
GUEST_VISIBLE_CELL_UPVALUE_CHANGE=NO

SEMANTIC_CHANGE=NO
NEW_PLAT_DECISION_REQUIRED=NO
PRODUCT_PATCH_REQUIRED=NO
```

The B1/B2 same-generation fast path remains accepted. Slice C is closed as an
invalid extension of that mechanism, not as an unfinished implementation.

## Remaining PERF013 work

PERF013 remains open only for its retained B1/B2 controlled timing checkpoint
and final coordination/closure reconciliation. That timing may quantify the
effect of removing the captured-local PE-constant obstacle, but it cannot make
cross-generation `MaterializedLocalAccessor` valid.

The independent `Too deep inlining, probably caused by recursive inlining`
bailouts remain outside this Slice-C resolution.
