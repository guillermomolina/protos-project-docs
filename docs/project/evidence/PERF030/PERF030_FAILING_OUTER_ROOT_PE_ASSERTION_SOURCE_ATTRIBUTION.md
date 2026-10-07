# PERF030-Q — failing outer-root PE assertion source attribution

## Scope

This record preserves the PERF030-Q investigation of the surviving permanent
Truffle partial-evaluation failure observed by PERF030-O on the outer
`primitive-closure-call` root.

The owning live work item is `guillermomolina/protos#784`
(**PERF030 — Make frame-local creation ordinal PE-constant**).

PERF024 / `guillermomolina/protos#756` remains blocked from physical
cross-runtime graph interpretation until this compilerability failure is
source-attributed and repaired or otherwise resolved.

This is non-normative diagnostic evidence. PERF030-Q changes no Protos source,
tests, specification, benchmark semantics, or guard implementation.

## Authorities

```text
SLICE=PERF030-Q
WORK_TYPE=INVESTIGATION
IMPLEMENTATION=NO

PRODUCT_FAILURE_AUTHORITY=cf6eb4c9aeaef4049fa5b56089d2dd1d2654d44b
PRODUCT_FAILURE_VERSION=0.3.193-SNAPSHOT
PRODUCT_FAILURE_COMMIT=PERF030-N: boundary generic frame name lookup

CURRENT_PROTOS_MAIN=a5b00b7c1bb72bacd6ef36bc714a3c8e0972dac4
CURRENT_PROTOS_MAIN_COMMIT=CLI008-D: reconcile diagnostic presentation surfaces
RELEVANT_CORRIDOR_DRIFT=NO

GRAAL_25_4_SOURCE_REVISION=95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b
FAILING_ROOT_SEMANTIC_ROLE=OUTER_RUN
FAILURE_COMPILER_NODE=36664|Pi
```

Current Protos main is two commits ahead of the exact failure authority. The
intervening changes affect BUG015 / parallel runtime and CLI008-D diagnostic
presentation surfaces, plus their tests, `pom.xml` and `CHANGELOG.md`.
They do not modify the PERF030-Q source corridor:

```text
ProtosSemanticBytecodeRootNode
ProtosBytecodeRootNode
ProtosFrameLexicalBindingAuthority
ProtosActivation
ProtosFrameLexicalLayout
```

The failure is therefore interpreted against the exact PERF030-O/N product
authority rather than against moving main.

## Surviving Protos corridor

At the failure authority, the outer `run` root reaches frame-backed current
binding creation through:

```text
CreateCurrentIndexedLocalSlot.perform
 -> ProtosBytecodeRootNode.createIndexedCurrentLocalSlot
 -> ProtosFrameLexicalBindingAuthority.createFrameBackedBindingAt
 -> LocalRangeAccessor.isCleared
 -> LocalRangeAccessor.setObject
```

The scalar `ordinal` originates as a Bytecode DSL
`@ConstantOperand(type = int.class, name = "ordinal")` and is forwarded
unchanged through the two Java helpers before being supplied as the
`LocalRangeAccessor` offset.

## Required-constant assertion inventory

GraalVM 25.4 `LocalRangeAccessor` asserts PE constancy separately for three
values on each relevant access.

For `isCleared(bytecodeNode, frame, ordinal)`:

```text
CompilerAsserts.partialEvaluationConstant(this)
CompilerAsserts.partialEvaluationConstant(bytecodeNode)
CompilerAsserts.partialEvaluationConstant(offset)
```

For `setObject(bytecodeNode, frame, ordinal, value)`:

```text
CompilerAsserts.partialEvaluationConstant(this)
CompilerAsserts.partialEvaluationConstant(bytecodeNode)
CompilerAsserts.partialEvaluationConstant(offset)
```

The exact upstream source authority is:

```text
truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/LocalRangeAccessor.java
blob=07addc390d2c6a755ab3d30e5494e08cb99fc2e1
```

No additional `partialEvaluationConstant` assertion is introduced by
`LocalRangeAccessor.checkBounds`, `ProtosFrameLexicalLayout.nameAt`,
generated `isLocalClearedInternal`, or the directly relevant generated
`setLocalValueInternal` implementation.

There is, however, another required-constant assertion in the generated
Bytecode DSL interpreter dispatch loop:

```text
CompilerAsserts.partialEvaluationConstant(bci)
```

Therefore the permanent bailout must not be assumed to originate from one of
the three `LocalRangeAccessor` values merely because the root reaches those
calls.

## Offset provenance

The integer offset path remains structurally clean:

```text
CreateCurrentIndexedLocalSlot.perform
  int ordinal @ConstantOperand
 -> createIndexedCurrentLocalSlot(... ordinal ...)
 -> createFrameBackedBindingAt(ordinal, value)
 -> LocalRangeAccessor.isCleared(... ordinal)
 -> LocalRangeAccessor.setObject(... ordinal, value)
```

Upstream Bytecode DSL source establishes that `@ConstantOperand` has
compilation-final semantics. Primitive constant operands are read as generated
instruction immediates rather than reconstructed from a dynamic guest stack
operand.

Along the surviving Protos path, no source evidence shows:

- boxing/unboxing of the ordinal;
- storage of the ordinal into a runtime object;
- reassignment of the ordinal;
- name-to-ordinal re-resolution;
- a source-level phi/merge replacing the ordinal; or
- an array-derived index replacing it.

`frameBackedLayout.nameAt(ordinal)` consumes the scalar as an array index but
does not mutate or replace the scalar passed onward.

Therefore:

```text
SOURCE_LEVEL_OFFSET_LOSS=NONE_FOUND
OFFSET_PE_PROVENANCE=PE_CONSTANT_FROM_OPERATION
```

This remains consistent with the checked-in PERF030 guard classification for
the integer index.

## LocalRangeAccessor receiver provenance

The `LocalRangeAccessor` begins as a Bytecode DSL constant operand in the
frame-authority installation operation.

That constant operand is then stored into a Java `final` field of a newly
constructed per-invocation `ProtosFrameLexicalBindingAuthority`.

The authority is installed into runtime activation/context state and later
recovered through:

```text
ProtosActivation.currentAuthorityAdmittingLocalCreationForRuntime()
 -> instanceof ProtosFrameLexicalBindingAuthority authority
 -> authority.createFrameBackedBindingAt(...)
 -> authority.frameBackedLocals
```

The activation field holding the authority is runtime state and may be
replaced during authority handoff. A Java `final` field inside the recovered
authority is not, by itself, an upstream Truffle/Graal guarantee that the
loaded object value is a PE constant.

Therefore:

```text
LOCAL_RANGE_ACCESSOR_ORIGINAL_SOURCE=BYTECODE_DSL_CONSTANT_OPERAND
LOCAL_RANGE_ACCESSOR_RECEIVER_PE_CONSTANT_AT_FAILING_CALL=NOT_PROVEN
```

## BytecodeNode provenance

The authority does not retain the installation-time `BytecodeNode` directly.
It retains:

```text
declaringRoot = bytecodeNode.getBytecodeRootNode()
```

Each frame-backed access later obtains:

```text
declaringRoot.getBytecodeNode()
```

Upstream generated Bytecode DSL code stores the current bytecode node in a
private volatile `@Child` field. Truffle `@Child` explicitly implies
`@CompilationFinal` semantics.

The generated replacement protocol is compatible with that contract:

- uncached-to-cached transition transfers to the interpreter and invalidates
  before replacement;
- bytecode replacement uses the generated atomic updater;
- general bytecode update is outside compiled execution and invalidates the old
  bytecode node; and
- execution detects bytecode/tier changes, deoptimizes and reloads the current
  child.

Thus the generated current-bytecode child itself has the expected
CompilationFinal/invalidation discipline despite the field also being
`volatile`.

However, PERF030-Q does not establish that the recovered
`ProtosFrameLexicalBindingAuthority` is a PE-constant object. Consequently,
its `declaringRoot` field load is not proven PE constant either.

Therefore:

```text
CURRENT_BYTECODE_CHILD_STABILITY=PROVEN
DECLARING_ROOT_PE_CONSTANT=NOT_PROVEN
CURRENT_BYTECODE_NODE_PE_CONSTANT_AT_LOCAL_RANGE_CALL=NOT_PROVEN
```

## Meaning of 36664|Pi

Graal's Truffle PE constant plugin bails out when the value passed to
`CompilerAsserts.partialEvaluationConstant` is not a constant. Its failure
message prints the failing compiler `ValueNode`.

`36664|Pi` therefore identifies a Graal `PiNode`, but a Pi node is a generic
value-refinement node. It does not identify one Java source value category.

A Pi can represent refinement of:

- an object value, including an `instanceof`-refined authority receiver;
- another object such as a `BytecodeNode`;
- an integer value refined by a guard/range fact; or
- another required constant in the same partial-evaluation graph.

Accordingly:

```text
36664|Pi -> LocalRangeAccessor receiver = NOT_PROVEN
36664|Pi -> BytecodeNode               = NOT_PROVEN
36664|Pi -> ordinal/offset             = NOT_PROVEN
36664|Pi -> generated dispatch bci     = NOT_EXCLUDED
36664|Pi -> other required constant    = NOT_EXCLUDED
```

The numeric Graal node id is not a durable source identity.

## Source identity exists upstream but was not retained in JFR

The missing distinction is available earlier in Graal's bailout path.

`TruffleGraphBuilderPlugins.failPEConstant` invokes
`GraphBuilderContext.bailout` at the exact failing
`partialEvaluationConstant` invocation.

The ordinary Graal `BytecodeParser.bailout` path:

1. constructs a `FrameState` at the current BCI;
2. converts that state to approximate Java-source stack elements; and
3. wraps the failure in `SourceStackTraceBailoutException`.

`SourceStackTraceBailoutException` installs those Java-source frames as its
stack trace.

The PE constant plugin also requests a verbose graph dump immediately before
the bailout.

Therefore the original compiler failure can carry enough source position to
distinguish the exact assertion line/value. The retained PERF030-O JFR
`failureReason` preserved only the exception class/message and compiler node
text, not the source stack that would make this attribution decisive.

## Static guard result

The checked-in PERF030 static guard is sound for the dimension it actually
models: interprocedural provenance of the integer LocalRange index.

It is not a complete proof of the full runtime compilation contract required by
`LocalRangeAccessor`.

```text
STATIC_GUARD_INDEX_MODEL_SOUND=YES
STATIC_GUARD_COMPLETE_FOR_LOCAL_RANGE_ACCESS=NO
GUARD_REPAIR_REQUIRED=YES
```

A future guard repair should preserve the current offset provenance analysis
and add distinct proof obligations for:

1. the `LocalRangeAccessor` receiver;
2. the supplied current `BytecodeNode`; and
3. the existing integer offset.

Alternatively, the current guard must be explicitly framed as index-only and a
separate guard must own receiver/current-node constancy.

No guard implementation is authorized by PERF030-Q.

## Result

The exact failing assertion still cannot be identified from the retained
PERF030-O JFR evidence.

```text
PERF030_Q=INCOMPLETE

FAILURE_ASSERTION=SOURCE_ATTRIBUTION_INCOMPLETE
FIRST_LOSS_POINT=UNRESOLVED
WHY_36664_PI_MAPS_TO_THIS_VALUE=NOT_PROVEN

OFFSET_PE_PROVENANCE=PE_CONSTANT_FROM_OPERATION
LOCAL_RANGE_ACCESSOR_RECEIVER_PE_CONSTANT=NOT_PROVEN
CURRENT_BYTECODE_NODE_PE_CONSTANT=NOT_PROVEN
OTHER_REQUIRED_CONSTANT=CANNOT_BE_EXCLUDED

PRODUCT_REPAIR_BOUNDED=NO
DESIGN_DECISION_REQUIRED=NO
GUARD_REPAIR_REQUIRED=YES

PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
NEW_FORMAL_ISSUE_REQUIRED=NO
```

The work remains one bounded diagnostic continuation of PERF030/#784 and does
not cross the Issue/Sub-issue promotion boundary.

## Minimum missing evidence

The minimum decisive evidence is one of:

- the complete Java-source stack/BCI attached to the
  `SourceStackTraceBailoutException` for the same outer-root permanent
  failure at `cf6eb4c9...`; or
- equivalent compiler graph/source-position metadata at the exact failing PE
  constant invocation.

That evidence must distinguish the concrete
`CompilerAsserts.partialEvaluationConstant` source call, including at least:

```text
LocalRangeAccessor.isCleared: this
LocalRangeAccessor.isCleared: bytecodeNode
LocalRangeAccessor.isCleared: offset
LocalRangeAccessor.setObject equivalents
generated interpreter: bci
other exact assertion if present
```

No product repair should be selected from the numeric `Pi` token alone.

## Next bounded work

```text
NEXT_SLICE=PERF030-R
WORK_TYPE=INVESTIGATION
IMPLEMENTATION=NO
PURPOSE=RECOVER_OR_BOUND_THE_MISSING_BAILOUT_SOURCE_IDENTITY
```

PERF030-R must use only repository HEAD/current published GitHub state and
upstream Graal source/web documentation. It must execute no build, benchmark,
runtime, JFR, IGV, shell, Maven, Java, Python, Git, or other local command.

Its job is to determine whether the exact source stack/BCI for the existing
failure can be recovered from already-published artifacts or metadata and, if
not, identify the smallest mechanically sufficient future capture surface that
would preserve it. It must not implement or run that capture.

AI assistance: this durable evidence record was drafted with ChatGPT from the
exact PERF030-O/N Protos revision, current Protos main, checked-in PERF030 guard
and baseline, live PERF030/PERF024 coordination, prior durable PERF030 evidence,
and exact GraalVM 25.4 source at
`95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b`.
