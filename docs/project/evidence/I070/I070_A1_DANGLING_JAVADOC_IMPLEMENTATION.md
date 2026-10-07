# I070-A1 dangling-Javadoc implementation evidence

I070-A1 is the first implementation slice of I070-A /
`guillermomolina/protos#713`, child of I070 / #712. It removes the three stale
documentation comments in `ProtosTask` that were no longer attached to any
declaration.

## Exact product publication

```text
PROTOS_REVISION=a5d3fe00c941c29f0a4e80eb262be610ff01443b
PROTOS_VERSION=0.3.220-SNAPSHOT
COMMIT=I070-A1: retire stale dangling Javadocs in ProtosTask
ISSUE=#713
PARENT=#712
```

The exact remote commit changes only:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/runtime/ProtosTask.java
```

The implementation delta in `ProtosTask.java` deletes the three consecutive
stale Javadocs describing legacy evaluator replay state, non-creating B6B replay
inspection, and Task-owned C-prime host execution. The existing Javadoc attached
to `dynamicControlState()` is preserved.

No executable statement, method signature, runtime control-flow behavior,
language semantics, or specification text is changed by this slice.

The publication metadata advances:

```text
0.3.219-SNAPSHOT -> 0.3.220-SNAPSHOT
```

and records the bounded I070-A1 change in the root implementation changelog.

## Validation evidence

The maintainer reports for the published candidate:

```text
GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
PRODUCT_PUSH=PASS
```

The pushed range reported by the maintainer is:

```text
00526bdeffaec4360f2ed2deed3d59e16ae3a431
    -> a5d3fe00c941c29f0a4e80eb262be610ff01443b
```

The exact GitHub commit currently exposes no combined status checks and no
pull-request workflow runs. This record therefore does not manufacture CI
evidence.

The exact compiler-warning baseline totals used during implementation were not
repeated in the publication handoff recorded here, so this evidence does not
invent a post-A1 warning count. The durable claim is bounded to the published
removal of the three previously identified dangling documentation comments and
the maintainer-reported local validation.

## Resulting I070-A state

```text
I070_A1=COMPLETE
DANGLING_DOC_COMMENT_SLICE=PUBLISHED
EXECUTABLE_CHANGE=NO
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
I070_A=OPEN
I070=OPEN
```

Current static source reconciliation keeps the next bounded family separate:

```text
NEXT_SLICE=I070-A2
NEXT_SCOPE=UID-only serial warnings
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
```

At revision `a5d3fe00c941c29f0a4e80eb262be610ff01443b`, the following six
serializable-by-inheritance exception classes have no instance state of their
own and are therefore the UID-only candidates established by source inspection:

- `ProtosEncodingValue.ConversionFailure`
- `ProtosTestResourceProviderRegistry.UnknownProviderException`
- `ProtosTestResourceProviderCoordinator.InvalidLeaseCoverageException`
- `ProtosTestResourceProviderCoordinator.DuplicateResourceKeyException`
- `ProtosTaskCancellationException`
- `ProtosParallelRuntime.NonParallel`

The remaining serial-warning family is deliberately excluded from A2:
`ParseError`, `ProtosNonLocalReturnException`, `ProtosSignalException`, and
the already-UID-bearing `ProtosBytecodeControlTransferException` contain
runtime state that must be treated in the later accidental-serialization slice
rather than by a mechanical UID-only patch.

This is non-normative project evidence. Live lifecycle state remains owned by
GitHub Issues #713 and #712.
