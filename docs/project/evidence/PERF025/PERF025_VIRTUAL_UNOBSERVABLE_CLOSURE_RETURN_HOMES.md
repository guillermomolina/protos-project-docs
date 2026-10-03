# PERF025 — Virtual unobservable Closure return homes

## Status

Published product implementation evidence for PERF025 / `guillermomolina/protos#758`.

This record is non-normative. It retains the exact product publication identity,
the structural implementation result, and the maintainer-reported validation
outcome for the return-home virtualization slice.

## Exact product publication

```text
PROTOS_REVISION=10b5a6b29c81d142c3348484d0b3dd8132f78d0d
PROTOS_PARENT_REVISION=ce2db129102fd69b3bc442b647c8994e7e565d90
PROTOS_VERSION=0.3.155-SNAPSHOT
COMMIT_SUBJECT=PERF025: virtualize unobservable Closure return homes
OWNING_ISSUE=guillermomolina/protos#758
BASE_IS_EXACT_PARENT=YES
```

The published revision is exactly one commit ahead of the preceding PERF025
compact-callee publication.

## Bounded objective

The retained PERF025 post-F1 pay-as-you-grow audit identified:

```text
RETURN_HOME_PHYSICALLY_CREATED_WHEN_UNOBSERVED
```

Before this slice, an owning source-backed Closure invocation established a fresh
physical `ProtosReturnHome` even when the canonical source proved that neither
the Closure nor any lexical descendant could ever execute `^` against that
home.

The published implementation removes that physical lifecycle only for
statically proven-unobservable source Closure homes while preserving the
Smalltalk-style semantic home and capture provenance.

```text
SEMANTIC_HOME=PRESERVED
PHYSICAL_HOME_REQUIRED_ONLY_WHEN_OBSERVABLE=YES
```

No guest-visible language semantics are changed.

## Static return-home observability proof

The new
`src/main/java/com/guillermomolina/protos/execution/CanonicalReturnHomeAnalysis.java`
computes one immutable source-definition property per bytecode Closure execution
plan.

The analysis is lexical rather than call-graph based. It considers:

- direct `CanonicalReturn`;
- parameter default expressions;
- nested Closure literals transitively at arbitrary lexical depth;
- inline Object parent/body expressions; and
- all canonical expression containers that can contain one of those forms.

A dynamically obtained callee does not make the caller's home observable merely
because that callee may itself contain a non-local return: the callee keeps its
own captured home provenance.

The canonical expression switch is exhaustive over the sealed expression
family, so a newly introduced expression form cannot silently become an
unexamined fast-path case.

```text
RETURN_HOME_STATIC_ANALYSIS=YES
ANALYSIS_RUN_PER_INVOCATION=NO
DYNAMIC_CALL_GRAPH_ANALYSIS=NO
DEFAULTS_INCLUDED=YES
LEXICAL_DESCENDANTS_TRANSITIVE=YES
OBJECT_BODIES_INCLUDED=YES
```

## Non-materialized home representation

`ProtosReturnHome` now provides one shared internal marker:

```text
ProtosReturnHome.unobservable()
```

The marker preserves semantic/capture provenance but has no active/completed
lifecycle and is not a guest value.

An owning invocation of a prepared source-backed Closure whose execution plan
proves its home unobservable receives this shared marker instead of allocating a
fresh physical `ProtosReturnHome`.

The compact frame ABI remains fixed-width: its return-home slot still carries a
`ProtosReturnHome`, but the proven-unobservable case carries the marker.

```text
SOURCE_NLR_DEAD_INVOCATION_NEW_PROTOS_RETURN_HOME=NO
COMPACT_ABI_RETURN_HOME_SLOT=PRESERVED
HOME_PROVENANCE=PRESERVED
```

## Lifecycle elision

The owning-call paths distinguish physical/materialized homes from the
non-materialized marker.

For a proven-unobservable owned home, the runtime skips:

- `isActive()` lifecycle matching;
- non-local-return target matching for that owned home; and
- `complete()` on invocation completion.

The source-backed compact call, rich activation factories, ordinary prepared
invocation and deferred immediate-method preparation all select the invocation
home through `ProtosClosureValue.invocationReturnHomeForRuntime()`.

A defensive failure remains if a `CanonicalReturn` somehow reaches a home
that the static analysis proved unobservable.

```text
SOURCE_NLR_DEAD_COMPLETE_CALL=NO
SOURCE_NLR_DEAD_OWNED_NLR_MATCHING=NO
OBSERVABLE_HOME_PHYSICAL_LIFECYCLE=PRESERVED
```

## Provenance is separate from physical lifecycle

The slice does not collapse "has an enclosing semantic home" into "has a fresh
physical home".

A nested NLR-dead Closure created under a virtual home captures and shares the
same `ProtosReturnHome.unobservable()` marker. It therefore remains a captured
home rather than becoming a new owning invocation.

This preserves the provenance distinction used by PLAT044 inline literal
callbacks.

```text
NLR_DEAD_DESCENDANT_OWNER_HOME_MATERIALIZED=NO
NLR_DEAD_DESCENDANT_HOME_PROVENANCE=SHARED
PLAT044_INLINE_CALLBACK_ADMISSION=PRESERVED
```

## Observable non-local-return cases keep a physical home

Any source shape that may target the owning home retains the existing physical
home lifecycle.

The dedicated regression covers:

```text
DIRECT_CANONICAL_RETURN_RETAINS_PHYSICAL_HOME=YES
DEFAULT_CANONICAL_RETURN_RETAINS_PHYSICAL_HOME=YES
DESCENDANT_CANONICAL_RETURN_RETAINS_OWNER_HOME=YES
TRANSITIVE_DESCENDANT_CANONICAL_RETURN_RETAINS_OWNER_HOME=YES
OBJECT_BODY_DESCENDANT_RETURN_RETAINS_OWNER_HOME=YES
```

This includes shapes equivalent to:

```text
() => ^42
(x = ^42) => x
(x = () => ^42) => x
() => { () => ^42 }
() => { () => { () => ^42 } }
Object-body descendants containing ^
```

The transitive regression explicitly establishes:

```text
OUTER_DIRECT_CANONICAL_RETURN=NO
DESCENDANT_CANONICAL_RETURN=YES
OUTER_HOME_MATERIALIZED=YES
```

## NLR semantics retained

The dedicated product regression preserves valid same-Task non-local return,
including nested and multi-level descendant return, Object-body descendant
return, and escaped-Closure invalid-return behavior.

```text
SAME_TASK_NLR=PRESERVED
ESCAPED_NLR_INVALID_RETURN=PRESERVED
```

The product changelog records preservation of the broader existing NLR
boundaries under the full reported validation:

```text
FOREIGN_TASK_INVALID_RETURN=PRESERVED
ENSURE_PRECEDENCE=PRESERVED
SUSPENSION_PHYSICAL_HOME_LIFETIME=PRESERVED
```

This record does not claim an independent coordinating-agent rerun of those
tests; the validation provenance is recorded below.

## Native and unknown fallback

Native bodies are opaque to the canonical proof. Closures without a prepared
source execution plan also do not use the proof.

Both retain fresh physical homes.

```text
NATIVE_UNKNOWN_FALLBACK=PRESERVED
UNPREPARED_SOURCE_FALLBACK=PRESERVED
```

No native-Closure effect analysis or broader effect system was introduced.

## Dedicated structural regression

The product publication adds:

```text
src/test/java/com/guillermomolina/protos/execution/
  ProtosPerf025VirtualReturnHomeTest.java
```

Its published checks include:

```text
RETURN_HOME_STATIC_ANALYSIS=YES
ANALYSIS_RUN_PER_INVOCATION=NO
SOURCE_NLR_DEAD_INVOCATION_NEW_PROTOS_RETURN_HOME=NO
SOURCE_NLR_DEAD_COMPLETE_CALL=NO
NLR_DEAD_DESCENDANT_OWNER_HOME_MATERIALIZED=NO
NLR_DEAD_DESCENDANT_HOME_PROVENANCE=SHARED
DIRECT_CANONICAL_RETURN_RETAINS_PHYSICAL_HOME=YES
DEFAULT_CANONICAL_RETURN_RETAINS_PHYSICAL_HOME=YES
OUTER_DIRECT_CANONICAL_RETURN=NO
DESCENDANT_CANONICAL_RETURN=YES
OUTER_HOME_MATERIALIZED=YES
TRANSITIVE_DESCENDANT_CANONICAL_RETURN_RETAINS_OWNER_HOME=YES
SAME_TASK_NLR=PRESERVED
OBJECT_BODY_DESCENDANT_RETURN_RETAINS_OWNER_HOME=YES
ESCAPED_NLR_INVALID_RETURN=PRESERVED
PLAT044_INLINE_CALLBACK_ADMISSION=PRESERVED
NATIVE_UNKNOWN_FALLBACK=PRESERVED
```

Existing tests whose structural assertions expected every owned source call to
have a fresh physical home were updated to accept the proven-unobservable
representation.

## Exact product delta

The exact comparison from
`ce2db129102fd69b3bc442b647c8994e7e565d90` to
`10b5a6b29c81d142c3348484d0b3dd8132f78d0d` is one commit:

```text
FILES_CHANGED=21
ADDITIONS=654
DELETIONS=42
```

Changed paths:

```text
M CHANGELOG.md
M pom.xml
A src/main/java/com/guillermomolina/protos/execution/CanonicalReturnHomeAnalysis.java
M src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeClosureExecutionPlan.java
M src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
M src/main/java/com/guillermomolina/protos/execution/ProtosClosureExecutionPlan.java
M src/main/java/com/guillermomolina/protos/execution/ProtosClosureInvoker.java
M src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosClosureValue.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosReturnHome.java
M src/test/java/com/guillermomolina/protos/execution/ProtosClosureInvokerTest.java
M src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2BClosureContinuationCompositionTest.java
M src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2D1OrdinarySendCompositionTest.java
M src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2D3AOrdinaryObjectCallProtocolTest.java
M src/test/java/com/guillermomolina/protos/execution/ProtosPerf006B2D3BSelectedStandardObjectCallIntrinsicTest.java
M src/test/java/com/guillermomolina/protos/execution/ProtosPerf010APreparedTargetSpecializationTest.java
M src/test/java/com/guillermomolina/protos/execution/ProtosPerf025DirectSourceClosureCompactInvocationTest.java
M src/test/java/com/guillermomolina/protos/execution/ProtosPerf025H1LazyRootActivationTest.java
A src/test/java/com/guillermomolina/protos/execution/ProtosPerf025VirtualReturnHomeTest.java
M tools/java_slow_tests_allowlist.txt
```

Version publication:

```text
IMPLEMENTATION_VERSION=0.3.155-SNAPSHOT
ROOT_CHANGELOG_ENTRY=YES
SPECIFICATION_CHANGE=NO
```

The new production source file carries the repository's APL-1.0 Part 5 notice.

## Validation provenance

The maintainer reported in the active interaction after product publication:

```text
PERF025: virtualize unobservable Closure return homes pushed, test passed
```

The coordinating publication review independently verified:

```text
REMOTE_PUBLICATION=PASS
PRODUCT_REVISION=10b5a6b29c81d142c3348484d0b3dd8132f78d0d
COMMIT_SUBJECT_MATCH=PASS
EXACT_PARENT=ce2db129102fd69b3bc442b647c8994e7e565d90
BASE_IS_EXACT_PARENT=YES
PRODUCT_VERSION=0.3.155-SNAPSHOT
FILES_CHANGED=21
ADDITIONS=654
DELETIONS=42
FOCAL_TEST_PUBLISHED=PASS
CHANGELOG_RECORD=PASS
LICENSE_NOTICE_NEW_SOURCE=PASS
```

Therefore:

```text
MAINTAINER_REPORTED_VALIDATION=PASS
INDEPENDENT_TEST_REEXECUTION_BY_COORDINATING_AGENT=NO
```

No command-by-command result beyond the maintainer's report is invented.

## PERF025 audit consequence

Relative to the retained post-F1 pay-as-you-grow audit and the immediately
preceding compact-callee record:

```text
RETURN_HOME_PHYSICALLY_CREATED_WHEN_UNOBSERVED
    -> consumed for statically proven-unobservable prepared source Closures by 10b5a6b29...

DIRECT_OR_DEFAULT_CANONICAL_RETURN_REQUIRES_HOME
    -> preserved

LEXICAL_DESCENDANT_CANONICAL_RETURN_REQUIRES_OWNER_HOME
    -> preserved transitively

OBJECT_BODY_DESCENDANT_RETURN_REQUIRES_OWNER_HOME
    -> preserved

NATIVE_OR_UNKNOWN_RETURN_HOME
    -> conservative physical fallback preserved
```

This slice does not select or implement another PERF025 residual candidate.

## PLAT040 / pay-as-you-grow convergence

The implementation continues the selected pay-as-you-grow direction:

```text
stable source-backed Closure plan
  -> static home-observability proof once
  -> unobservable owner uses shared non-materialized home provenance
  -> observable owner keeps exact physical home lifecycle
  -> native/unknown fallback keeps exact physical machinery
```

The optimization changes representation only.

```text
OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
NEW_PLATFORM_DECISION_REQUIRED=NO
```

## Performance-claim boundary

No benchmark, timing campaign, JFR profile, allocation profile, or attributable
per-call magnitude is part of this publication.

```text
BENCHMARK_RUN_FOR_THIS_SLICE=NO
ATTRIBUTABLE_NS_PER_CALL=NOT_MEASURED
END_TO_END_SPEEDUP_PERCENT=NOT_MEASURED
PERFORMANCE_MAGNITUDE_CLAIMED=NO
```

The durable claim is structural: an owning prepared source Closure proven unable
to observe its home no longer creates a fresh physical return-home lifecycle.

## PERF025 status

This bounded implementation slice is complete. PERF025 itself remains open.

```text
PERF025_VIRTUAL_RETURN_HOME_SLICE=COMPLETE
PERF025_STATUS=OPEN

STATIC_FAIL_CLOSED_ANALYSIS=YES
SOURCE_NLR_DEAD_PHYSICAL_HOME=ELIDED
SEMANTIC_HOME_PROVENANCE=PRESERVED
OBSERVABLE_NLR_HOME_LIFECYCLE=PRESERVED
NATIVE_UNKNOWN_FALLBACK=PRESERVED
PLAT044_INLINE_CALLBACK_ADMISSION=PRESERVED

OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
BENCHMARK_RESULT_CLAIMED=NO
NEXT_SLICE_NOT_SELECTED_BY_THIS_RECORD=YES
```

## Materially inspected publication evidence

The coordinating publication review inspected:

- `guillermomolina/protos@10b5a6b29c81d142c3348484d0b3dd8132f78d0d`;
- the exact one-commit comparison against
  `ce2db129102fd69b3bc442b647c8994e7e565d90`;
- the `0.3.155-SNAPSHOT` Maven version and root CHANGELOG entry;
- `CanonicalReturnHomeAnalysis.java`;
- `ProtosBytecodeClosureExecutionPlan.java`;
- `ProtosClosureExecutionPlan.java`;
- `ProtosFrameArguments.java`;
- `ProtosActivation.java`;
- `ProtosClosureValue.java`;
- `ProtosReturnHome.java`;
- `ProtosBytecodeRootNode.java`;
- `ProtosClosureInvoker.java`;
- the new `ProtosPerf025VirtualReturnHomeTest.java`;
- the preceding compact-callee PERF025 durable evidence; and
- the repository performance, coordination, documentation, reference, and
  durable-record instructions.

This record is evidence only and does not replace live GitHub Issue
coordination.
