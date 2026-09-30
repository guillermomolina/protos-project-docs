# I075-D — int boxing elimination with current-node lexical authority

Status: **PUBLISHED / VALIDATED**

This durable, non-normative record retains the implementation and closure
evidence for I075-D / guillermomolina/protos#746.

## Identity

```text
DATE=2026-09-30
WORK_ITEM=I075-D
PARENT_WORK_ITEM=I075
GITHUB_ISSUE=guillermomolina/protos#746
PARENT=PERF011 / guillermomolina/protos#693

TYPE=IMPLEMENTATION

PROTOS_REVISION=898eb8b2bafe4be99a33032ab0cf6436ce6f3e72
PROTOS_VERSION=0.3.125-SNAPSHOT
BASE_REVISION=a89a8897ea20b10344785ee9573ed329d188ef28
GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1
```

Published commit:

```text
898eb8b2bafe4be99a33032ab0cf6436ce6f3e72
I075-D: enable int boxing elimination with current-node lexical authority
```

The commit is exactly one revision ahead of the I075-C/I074 baseline.

## Published delta

Changed paths:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java
src/test/java/com/guillermomolina/protos/execution/ProtosI075DLexicalAuthorityCurrentBytecodeNodeTest.java
```

Diff statistics from the published revision:

```text
CHANGELOG.md                                                        +16
pom.xml                                                              +1/-1
ProtosBytecodeRootNode.java                                          +1
ProtosFrameLexicalBindingAuthority.java                              +47/-18
ProtosI075DLexicalAuthorityCurrentBytecodeNodeTest.java             +167
```

## Boxing-elimination adoption

`ProtosBytecodeRootNode` now enables exactly the I075-A-selected carrier:

```java
boxingEliminationTypes = {int.class}
```

No guest primitive Integer, Float or Boolean representation was introduced.

The selected `int` values remain implementation-internal metadata, including
indices/counts, lexical depths and frame ordinals.

```text
FIRST_PRIMITIVE_CARRIER=IMPLEMENTED
BOXING_ELIMINATION_TYPES={int.class}
NEW_GUEST_PRIMITIVE_REPRESENTATION=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
```

## Current-node lexical-authority repair

I075-C established that `ProtosFrameLexicalBindingAuthority` incorrectly kept
the tier-specific `BytecodeNode` that existed when the authority was installed.

I075-D implements the bounded repair.

The authority now retains:

```text
stable declaring BytecodeRootNode
```

rather than:

```text
installation-time BytecodeNode
```

and resolves:

```java
declaringRoot.getBytecodeNode()
```

for frame-backed runtime-authority access.

The implementation deliberately obtains one current node for each logical
multi-step operation where presence and value access must remain coherent.
This includes captured runtime-authority checks/reads/writes and ordinary
context-authority reads, snapshots and removals.

The published implementation therefore preserves the selected I075-C model:

```text
ACCESSOR_BYTECODE_NODE=MUST_BE_CURRENT

PLAT036_DELTA=NONE
D179_DELTA=NONE
SECOND_LOCAL_STORAGE_AUTHORITY=NO
GENERIC_BYTECODE_LOCAL_ACCESS_MIGRATION=NO
CAPTURE_OWNERSHIP_CHANGE=NO
MATERIALIZATION_LIFETIME_CHANGE=NO
CONTINUATION_MODEL_CHANGE=NO
```

Presence remains derived from the physical frame's cleared/non-cleared state.

## Focused regression

The new test:

`ProtosI075DLexicalAuthorityCurrentBytecodeNodeTest`

constructs the exact lifetime that exposed I075-B:

```text
1. create a statically allocated frame-backed local
2. install its lexical authority while the root is UNCACHED
3. leave the local ABSENT
4. suspend
5. resume and deterministically transition the root to CACHED
6. create the binding through the retained execution-context authority
7. read it through that authority
8. remove it
9. re-create it using the same static binding
10. mutate it
11. resume execution and read through direct current-node ReadFrameLocal
```

The regression therefore covers the material failure boundary rather than only
asserting implementation structure.

It also exercises:

```text
ABSENT -> PRESENT
PRESENT -> ABSENT
PRESENT -> ABSENT -> PRESENT
PRESENT -> PRESENT mutation
uncached -> cached continuation transition
runtime-authority access after transition
direct current-node frame-local read after transition
```

Existing integrated tests continue to cover the unaffected direct
`MaterializedLocalAccessor` captured paths and the broader PLAT036/D179
semantics.

## Validation

The maintainer reported after publication:

```text
make test = PASS
```

This is the repository integrated validation gate and includes the newly
published I075-D regression together with the existing semantic/runtime test
suite.

The agent did not independently execute the suite; the validation result is
retained from the maintainer's execution report.

```text
FOCUSED_VALIDATION=PASS
REQUIRED_INTEGRATED_VALIDATION=PASS
PUBLICATION=PASS
```

No dedicated benchmark was required by the I075 lightweight validation policy,
and no performance claim is made from this implementation.

```text
PERF020_REQUIRED_FIRST=NO
PROTOS_BENCHMARKS_REQUIRED_FIRST=NO
PERFORMANCE_CLAIM=NONE
CLEAR_PERFORMANCE_REGRESSION=NO_CLEAR_REGRESSION_EVIDENCE
```

## Closure result

I075's implementation outcome is now complete:

```text
I075-A=COMPLETE_HISTORICAL
I075-B=STOPPED_NOT_PUBLISHED
I075-C=COMPLETE
I075-D=PUBLISHED_VALIDATED

FIRST_PRIMITIVE_CARRIER=IMPLEMENTED
BOXING_ELIMINATION=ENABLED_FOR_SELECTED_TYPE_SET
BOXING_ELIMINATION_TYPES={int.class}

GUEST_SEMANTICS=UNCHANGED
BOUNDARY_MATERIALIZATION=UNCHANGED_AND_VALIDATED_BY_INTEGRATED_SUITE
FOCUSED_VALIDATION=PASS
REQUIRED_INTEGRATED_VALIDATION=PASS
PUBLICATION=PASS

PLAT036_DELTA=NONE
D179_DELTA=NONE
ARCHITECTURE_DECISION_REQUIRED=NO
UPSTREAM_COORDINATION_REQUIRED=NO

I075_STATUS=COMPLETE
NEXT_I075_SLICE=NONE
```

I075 introduced no benchmark claim and does not by itself close PERF011.
PERF011 retains its broader runtime-representation-fit work independently of
this completed adoption item.
