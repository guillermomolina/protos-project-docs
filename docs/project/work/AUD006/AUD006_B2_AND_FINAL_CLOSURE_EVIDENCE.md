# AUD006-B2 and final closure evidence

## Status

```text
WORK_ITEM=AUD006
ISSUE=guillermomolina/protos#453
STATUS=CLOSED
CLOSURE_DATE=2026-10-06

FINAL_PRODUCT_REVISION=698f90c3a7785fc9f51d2685ae6024b67b6ea1a2
FINAL_PRODUCT_REVISION_SUBJECT=AUD006-B2: eliminate CommandLine canonicalization execution-stack depth
IMPLEMENTATION_VERSION=0.3.231-SNAPSHOT

AUD006_A_STATUS=COMPLETE
AUD006_F1_STATUS=RESOLVED
D115_F1_LINEAR_ACCUMULATION_TARGET=RESTORED

AUD006_B1_STATUS=COMPLETE
AUD006_B2_STATUS=COMPLETE
AUD006_B_STATUS=CLOSED_WITH_ACCEPTED_RESIDUAL_DEBT

CANONICALIZATION_RECURSION=REMOVED
CANONICALIZATION_EXECUTION_STACK_WRT_DEPTH=O1
PARSE_SELECTED_CHILD_RECURSION=REMAINS_ACCEPTED_RESIDUAL

AUD006_C_STATUS=NOT_EXECUTED_ACCEPTED_NON_BLOCKING_DEBT
AUD006_D_STATUS=SUPERSEDED_BY_OWNER_DIRECTED_CLOSURE_RECONCILIATION

LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED

SPECIFICATION_CHANGED=NO
PUBLIC_API_CHANGED=NO
D111=KEEP
D115=KEEP
D118=KEEP
D119=KEEP
LIB011_ISSUE_428=KEEP_CLOSED

NEW_SEMANTIC_DECISION=NO
NEW_PLATFORM_DECISION=NO
PUBLIC_DEPTH_LIMIT=NO
FOLLOWUP_AUD006_SLICE=NONE
```

This is durable non-normative closure evidence. It records the published B2
implementation and the project owner's explicit instruction to close AUD006
without creating further micro-slices. It does not claim that work which was
not performed was performed.

## Published B2 implementation

The final product revision is:

```text
698f90c3a7785fc9f51d2685ae6024b67b6ea1a2
AUD006-B2: eliminate CommandLine canonicalization execution-stack depth
```

The commit changes:

```text
CHANGELOG.md
pom.xml
protos/lib/cli/CommandLine.protos
src/test/java/com/guillermomolina/protos/execution/ProtosCommandLineSpecModuleTest.java
```

The implementation version is `0.3.231-SNAPSHOT`.

B2 removes recursive `canonicalizeCommand` traversal from
`std:cli/CommandLine.command`. The replacement is an explicit linked heap-frame
depth-first traversal in Protos using `enterCommand`, `exitCommand`, a current
frame and an iterative loop.

The implementation preserves the B1-required ordering: a child is fully
canonicalized before the parent performs duplicate child-name detection,
registration and storage.

## Active-path identity semantics

The B2 implementation retains `visiting` as an active-path `IdentityMap`.

Therefore:

- self cycles remain rejected;
- indirect ancestor cycles remain rejected;
- a descriptor reused after a completed branch remains valid;
- there is no descriptor-to-result memoization; and
- each valid occurrence produces a fresh canonical result.

The retained tests prove indirect ancestor-cycle rejection and successful reuse
of the same descriptor across two completed branches with distinct canonical
result identities.

## Retained depth evidence

B2 adds a deterministic 4096-level single-child canonicalization regression.

The commit explicitly records that 4096 is a test scale, not a public maximum or
safe-depth claim. The same case overflowed the previous recursive
canonicalization implementation and passes with the linked-frame traversal.

Therefore:

```text
CANONICALIZATION_DEPTH_REGRESSION=4096_LEVELS
PREVIOUS_RECURSIVE_IMPLEMENTATION=STACK_OVERFLOW_AT_RETAINED_CASE
ITERATIVE_IMPLEMENTATION=PASS
PUBLIC_DEPTH_LIMIT=NONE
```

No timing benchmark, Java agent or new runtime facility is introduced.

## Complexity result

For command-tree depth `D` and expanded canonicalization surface `S`:

Before B2:

```text
time:             O(S)
heap:             O(result + active logical state)
execution stack:  O(D)
```

After B2:

```text
time:             O(S)
heap:             O(result + explicit linked frames)
execution stack:  O(1) with respect to D
```

The repair is entirely guest-side Protos machinery.

## Validation

After publication, the maintainer reported:

```text
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

No additional tests are invented by this record.

## Owner-directed closure disposition

The earlier B1 decomposition proposed separate B2 and B3 implementation slices
and the original audit plan also contained separate C and D stages.

After B2 publication, the project owner explicitly rejected further
single-file/micro-slice decomposition and directed AUD006 to close rather than
continue producing artificial slices.

The closure therefore supersedes that internal execution decomposition. This is
a scheduling/governance disposition, not a claim that every originally proposed
subtask was implemented.

### Residual selected-child parse recursion

`CommandLine.parse` still contains the selected-child `parseScope` recursion
identified by B1.

It remains classified exactly as the original audit classified F2:
robustness/scalability hardening debt, with no demonstrated ordinary-use
functional defect.

At closure it is explicitly accepted as residual debt:

```text
PARSE_SELECTED_CHILD_RECURSION=KNOWN
PARSE_SELECTED_CHILD_FUNCTIONAL_DEFECT_DEMONSTRATED=NO
PUBLIC_DEPTH_LIMIT=NO
FOLLOWUP_B3_ALLOCATED=NO
DISPOSITION=ACCEPTED_RESIDUAL_NON_BLOCKING
```

This record does not invent a safe numeric parse depth.

### Documentation/governance debt

The originally proposed AUD006-C documentation/governance pass was not executed
as a separate slice.

F3/F4/F5 are therefore not marked COMPLETE. They are accepted as non-blocking
documentation/governance debt at owner-directed AUD006 closure.

F6 remains a documented future integration risk rather than a current defect:
TOOL002 projected argument indices must not later be mistaken for original argv
indices if future diagnostics require original provenance.

No successor AUD006 slice is allocated by this closure.

## Final architecture disposition

The material conformance defect that motivated AUD006, F1, was resolved by A4.
B2 also eliminates canonicalization command-depth execution-stack growth with
retained discriminating evidence.

The closed LIB011 architecture remains in force:

```text
LIB011_ARCHITECTURE=KEEP
LIB011_PUBLIC_API=KEEP
D111=KEEP
D115=KEEP
D118=KEEP
D119=KEEP
LIB011_ISSUE_428=KEEP_CLOSED

COMMANDLINE_POLICY_OWNER=PROTOS
NEW_PUBLIC_ARRAY_API=NO
NEW_COMMANDLINE_PUBLIC_API=NO
NEW_DEPTH_POLICY=NO
NEW_NATIVE_BOUNDARY_FROM_B2=NO
```

## Final closure

```text
AUD006_STATUS=CLOSED
AUD006_RESULT=F1_RESOLVED_CANONICALIZATION_DEPTH_HARDENED_WITH_EXPLICIT_ACCEPTED_RESIDUAL_DEBT
AUD006_FINAL_PRODUCT_REVISION=698f90c3a7785fc9f51d2685ae6024b67b6ea1a2
FOLLOWUP_AUD006_SLICE=NONE
LIB011_ISSUE_428=KEEP_CLOSED
```
