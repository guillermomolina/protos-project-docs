# PERF028-A — Resolved current lexical write implementation

Status: **PUBLISHED / MAINTAINER-REPORTED VALIDATION PASS**
Date: 2026-10-02
Formal owner: `PERF028 / guillermomolina/protos#780`

This is durable, non-normative implementation evidence for PERF028-A.

It records the published product change that specializes statically proven
same-scope lexical writes through a constant Truffle Bytecode DSL
`LocalAccessor` while preserving the exact Protos bare-assignment semantics
owned by `spec/semantics/EXECUTION_AND_CONTROL.md`.

It does not introduce new language semantics, change the specification, or
claim a measured performance magnitude.

## Publication identity

```text
PROTOS_REPOSITORY=guillermomolina/protos

BASE_REVISION=d1aeea403cc7f7e5ffa7289006ceb072b025ca44
BASE_VERSION=0.3.148-SNAPSHOT

PERF028_A_REVISION=aaa9130a9bd7190526f2c62a4317e8d90090c423
PERF028_A_VERSION=0.3.149-SNAPSHOT
COMMIT_SUBJECT=PERF028-A: specialize proven current lexical writes through constant LocalAccessor

AHEAD_BY=1
BASE_IS_EXACT_PARENT=YES
```

The remote HEAD and exact one-commit compare were inspected after the
maintainer reported that PERF028-A had been pushed and passed validation.

## Changed product paths

The exact product delta is:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf028AResolvedCurrentLexicalWriteTest.java
```

Compare statistics:

```text
FILES_CHANGED=7
ADDITIONS=848
DELETIONS=1
```

The large majority of added lines are focused regression coverage in the new
PERF028-A test class.

## Implemented lowering

`CanonicalToBytecodeLowerer` now identifies an assignment-local
`BytecodeLocal` when the assignment:

```text
has no explicit member target
currentRootAnalysis is available
resolution is CanonicalBindingResolution.Resolved
resolved identity owner == currentRootTopScope
currentRootFrameLocals contains the identity name
```

The helper is:

```text
currentResolvedAssignmentBytecodeLocal(...)
```

Both body lowering and parameter-default lowering now route such assignments
through:

```text
ResolveCurrentFrameLocalWriteTarget(LocalAccessor constant)
evaluate RHS
AssignCurrentFrameLocal(LocalAccessor constant)
```

instead of emitting the ordinary same-site pair:

```text
ResolveWritableLexicalTarget
AssignResolvedLexicalTarget
```

for the proven current-local case.

The existing generic operations remain present for ineligible assignments and
for runtime fallback when static identity is known but current runtime presence
is absent.

## Pre-RHS destination selection

The new `ResolveCurrentFrameLocalWriteTarget` uses the assignment site's
constant `LocalAccessor`.

For a genuine current execution context whose frame local is PRESENT:

```text
activation.hasGenuineExecutionContextForRuntime()
&& !accessor.isCleared(bytecodeNode, frame)
```

it returns the stateless singleton marker:

```text
ResolvedLexicalWriteTarget.STATIC_CURRENT_FRAME_LOCAL
```

Thus the ordinary PRESENT path does not perform String-keyed lexical destination
search and does not create a fresh general destination object.

If the current local is cleared/ABSENT, or the current scope is not the genuine
execution-context case admitted by this optimization, the implementation calls
the unchanged:

```text
ResolveWritableLexicalTarget.perform(activation, name)
```

**before RHS evaluation**.

This preserves exact lexical/receiver fallback and `SlotNotFound` timing.

## Post-RHS exact-destination write

`AssignCurrentFrameLocal` distinguishes only the destination selected before
the RHS.

If that selection was not `STATIC_CURRENT_FRAME_LOCAL`, it delegates to the
unchanged:

```text
AssignResolvedLexicalTarget.perform(...)
```

with the exact retained fallback destination.

If the static current destination was selected, the operation:

1. checks current-context FROZEN state;
2. checks that the same accessor is still PRESENT;
3. signals a Protos Error if FROZEN or cleared;
4. otherwise writes with `accessor.setObject(...)`;
5. returns the exact RHS value.

It never re-runs lexical destination search after RHS evaluation.

Therefore the implementation preserves:

```text
DESTINATION_SELECTED_BEFORE_RHS=YES
POST_RHS_DESTINATION_RE_RESOLUTION=NO
```

## D179 C0 behavior

Static `Resolved` identity still does not imply permanent presence.

The implementation preserves the required D179 C0 cases:

### Current local ABSENT at selection

The specialized resolver immediately enters the unchanged generic destination
selection before evaluating the RHS.

A farther lexical or receiver-own local can therefore be selected exactly as
before.

### Selected current binding removed during RHS

If the current binding was selected while PRESENT and the RHS removes it, the
post-RHS accessor is cleared and the mutation signals Error.

The implementation does not retarget to another same-named binding.

### Selected current binding removed and recreated in the same context

The same frame-local identity becomes PRESENT again. The retained static current
selection writes the recreated same-owner binding.

### Current binding ABSENT, farther fallback selected, RHS recreates current

The exact farther target selected before RHS remains retained and receives the
write. The newly recreated current binding does not steal the assignment.

These cases are directly represented in the new focal regression class.

## Mutation-state seam

PERF028-A adds:

```text
ProtosActivation.currentContextIsFrozenForRuntime()
```

with:

```java
return context != null && context.isFrozen();
```

This checks FROZEN state without materializing a deferred guest execution
Context.

The invariant is:

```text
unmaterialized guest Context -> cannot have been guest-frozen
materialized guest Context   -> exact context.isFrozen()
```

The direct current-local write therefore preserves:

```text
CLOSED_EXISTING_WRITE=ALLOWED
FROZEN_EXISTING_WRITE=REJECTED
```

without introducing eager Context materialization.

## Single lexical authority

PERF028-A does not add another binding store.

The optimized current write uses the same frame-backed lexical authority already
selected by PLAT036/I068.

```text
ONE_SEMANTIC_BINDING_VALUE_AUTHORITY=PRESERVED
SECOND_LEXICAL_STORE=NO
RAW_STORELOCAL_FOR_ACCESSOR_OWNED_BINDING=NO
```

The actual write is performed through `LocalAccessor.setObject(...)`, matching
the established frame-backed accessor model.

## Unchanged assignment classes

The focal regression explicitly checks that ineligible forms retain their
existing paths:

```text
Candidate             -> generic Resolve/AssignResolvedLexicalTarget
Dynamic               -> generic Resolve/AssignResolvedLexicalTarget
explicit member write -> AssignLocalSlot path
CapturedResolved      -> existing PERF013 captured-write path
```

Therefore:

```text
CANDIDATE_FALLBACK_PRESERVED=YES
DYNAMIC_FALLBACK_PRESERVED=YES
EXPLICIT_MEMBER_ASSIGNMENT_UNCHANGED=YES
CAPTURED_WRITE_PATH_PRESERVED=YES
```

## Default-expression lowering

PERF028-A applies the same current-local accessor specialization in both:

```text
emitBodyAssign(...)
emitDefaultAssign(...)
```

The focal test includes an assignment inside a parameter default and verifies
that it uses the specialized current-local path.

## Suspension and Bytecode tier transition

The new focal test constructs a Bytecode root that:

1. creates the frame-backed current binding;
2. selects the current write destination;
3. suspends during the RHS;
4. externally mutates the binding while suspended;
5. resumes with the RHS value;
6. completes the write through the retained selection.

It verifies both:

```text
RHS_YIELD_RESUME_SELECTION_PRESERVED=PASS
CACHED_TRANSITION_ACCESSOR_COHERENT=PASS
```

and separately verifies that removal while suspended becomes the expected
post-resume mutation Error.

## Published focal coverage

`ProtosPerf028AResolvedCurrentLexicalWriteTest` covers the following published
behaviors:

```text
CURRENT_RESOLVED_WRITE_USES_CONSTANT_LOCAL_ACCESSOR
ASSIGNMENT_RESULT_IS_RHS
CURRENT_ABSENT_EXACT_FALLBACK
CURRENT_ABSENT_RECEIVER_FALLBACK
DESTINATION_SELECTED_BEFORE_RHS
POST_RHS_DESTINATION_RE_RESOLUTION=NO
RHS_REMOVE_SELECTED_CURRENT=EXPECTED_MUTATION_ERROR
RHS_REMOVE_RECREATE_SAME_OWNER
ABSENT_THEN_RHS_RECREATE_NO_RETARGET
CLOSED_WRITE_ALLOWED
FROZEN_WRITE_REJECTED
PRESENT_NULL_DISTINCT_FROM_ABSENT
RHS_CONTROL_TRANSFER_NO_WRITE
DEFAULT_ASSIGNMENT_PATH
RHS_YIELD_RESUME_SELECTION_PRESERVED
CACHED_TRANSITION_ACCESSOR_COHERENT
SUSPENDED_REMOVE_SELECTED_CURRENT=EXPECTED_MUTATION_ERROR
CANDIDATE_FALLBACK_PRESERVED
DYNAMIC_FALLBACK_PRESERVED
EXPLICIT_MEMBER_ASSIGNMENT_UNCHANGED
CAPTURED_WRITE_PATH_PRESERVED
```

The test also inspects generated Bytecode instruction names to establish that an
eligible current resolved write contains:

```text
ResolveCurrentFrameLocalWriteTarget
AssignCurrentFrameLocal
```

and does not contain the generic current-site pair:

```text
ResolveWritableLexicalTarget
AssignResolvedLexicalTarget
```

while ineligible writes retain their expected generic/specialized paths.

## Validation provenance

The maintainer reported in the active interaction:

```text
"PERF028-A: specialize proven current lexical writes through constant LocalAccessor pushed y pass"
```

The coordinating agent independently verified:

```text
REMOTE_PUBLICATION=PASS
REMOTE_HEAD=aaa9130a9bd7190526f2c62a4317e8d90090c423
COMMIT_SUBJECT_MATCH=PASS
EXACT_PARENT_MATCH=PASS
VERSION=0.3.149-SNAPSHOT
EXPECTED_CHANGED_PATHS=PASS
FOCAL_TEST_PUBLISHED=PASS
```

Validation execution itself remains human-owned under repository policy.

Therefore the retained validation claim is deliberately bounded to:

```text
MAINTAINER_REPORTED_VALIDATION=PASS
INDEPENDENT_REEXECUTION_BY_COORDINATING_AGENT=NO
```

No more specific command-by-command PASS claims are invented by this record.

## Specification / architecture impact

```text
OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
NEW_PLAT_DECISION=NO

PLAT036_FRAME_BACKED_AUTHORITY=PRESERVED
D179_C0_PRESENCE_SEMANTICS=PRESERVED
PERF013_CAPTURED_WRITE_ARCHITECTURE=PRESERVED
```

No specification file changed in the product commit.

## License / publication metadata

The new Protos-owned test source carries the required APL-1.0 Part 5 notice.

The modified existing source files retain their repository licensing model, and
the Maven package metadata continues to identify APL-1.0.

No licensing policy change is introduced.

## PERF028 structural acceptance

The published candidate establishes:

```text
PERF028_A=COMPLETE

CURRENT_RESOLVED_WRITE_USES_CONSTANT_LOCAL_ACCESSOR=YES

ORDINARY_PRESENT_CURRENT_WRITE_GENERIC_NAME_SEARCH=NO
ORDINARY_PRESENT_CURRENT_WRITE_FRESH_GENERAL_TARGET=NO

DESTINATION_SELECTED_BEFORE_RHS=PASS
POST_RHS_DESTINATION_RE_RESOLUTION=NO

CURRENT_ABSENT_EXACT_FALLBACK=PASS
ABSENT_THEN_RHS_RECREATE_NO_RETARGET=PASS

RHS_REMOVE_SELECTED_CURRENT=EXPECTED_MUTATION_ERROR
RHS_REMOVE_RECREATE_SAME_OWNER=PASS

PRESENT_NULL_DISTINCT_FROM_ABSENT=PASS
CLOSED_WRITE_ALLOWED=PASS
FROZEN_WRITE_REJECTED=PASS

RHS_CONTROL_TRANSFER_NO_WRITE=PASS
RHS_YIELD_RESUME_SELECTION_PRESERVED=PASS
DEFAULT_ASSIGNMENT_PATH=PASS

CANDIDATE_FALLBACK_PRESERVED=YES
DYNAMIC_FALLBACK_PRESERVED=YES
EXPLICIT_MEMBER_ASSIGNMENT_UNCHANGED=YES
CAPTURED_WRITE_PATH_PRESERVED=YES

ONE_SEMANTIC_BINDING_VALUE_AUTHORITY=PASS

MAINTAINER_REPORTED_VALIDATION=PASS

SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
NEW_PLAT_DECISION=NO
```

## Performance boundary

PERF028-A removes the previously audited general write-target shape from the
ordinary statically proven PRESENT current-local assignment path.

This record does **not** claim:

```text
ATTRIBUTABLE_NS_PER_ASSIGNMENT=UNKNOWN
HEAP_ALLOCATION_REDUCTION_COUNT=NOT_MEASURED
END_TO_END_SPEEDUP_PERCENT=NOT_MEASURED
DOMINANT_RUNTIME_CAUSE=NOT_ESTABLISHED
```

PERF028 defined a later causal timing measurement only as optional ("if needed"),
not as a closure prerequisite.

The bounded product objective is therefore complete at PERF028-A.

## Closure

```text
PERF028_STATUS=COMPLETE
FINAL_PRODUCT_REVISION=aaa9130a9bd7190526f2c62a4317e8d90090c423
FINAL_PRODUCT_VERSION=0.3.149-SNAPSHOT

REMAINING_REQUIRED_IMPLEMENTATION_SLICE=NONE
REQUIRED_TIMING_SLICE=NONE

ISSUE_READY_TO_CLOSE=YES
```

