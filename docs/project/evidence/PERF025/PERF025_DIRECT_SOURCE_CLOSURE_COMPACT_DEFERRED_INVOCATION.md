# PERF025 — Direct source Closure compact/deferred invocation

Status: **PUBLISHED / MAINTAINER-REPORTED VALIDATION PASS**
Date: 2026-10-02
Formal owner: `PERF025 / guillermomolina/protos#758`

This is durable, non-normative implementation evidence for the PERF025 slice
published as:

`PERF025: compact deferred invocation for direct source Closure calls`.

It records the bounded convergence of direct **source-backed** Closure invocation
with the already-ratified PLAT040 F′ compact/deferred invocation architecture.

The change does not introduce new Protos semantics, change the specification,
reopen PLAT040, or claim a measured performance magnitude.

## Publication identity

```text
PROTOS_REPOSITORY=guillermomolina/protos

BASE_REVISION=aaa9130a9bd7190526f2c62a4317e8d90090c423
BASE_VERSION=0.3.149-SNAPSHOT

PERF025_DIRECT_SOURCE_REVISION=fe078828412aabfc48dc1835bf34dce784307b3e
PERF025_DIRECT_SOURCE_VERSION=0.3.150-SNAPSHOT
COMMIT_SUBJECT=PERF025: compact deferred invocation for direct source Closure calls

AHEAD_BY=1
BASE_IS_EXACT_PARENT=YES
```

The remote product HEAD and exact one-commit compare were inspected after the
maintainer reported that the slice had been tested and pushed.

## Changed product paths

The exact product delta is:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeTaskExecution.java
src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf025DirectSourceClosureCompactInvocationTest.java
```

Exact compare statistics:

```text
FILES_CHANGED=8
ADDITIONS=718
DELETIONS=65
```

The new focal regression class contributes 481 added lines.

## Governing architecture

PLAT040 F′ requires the ordinary hot source-call shape to prefer:

```text
stable selected callable/target
  -> DirectCallNode / inlineable target
  -> compact Truffle frame-argument invocation ABI
  -> conditional guest-visible rich-state materialization
  -> exact fallback where the optimized shape is not applicable
```

Its retained invariants include:

```text
FRAME_ARGUMENT_INVOCATION_ABI=REQUIRED
EAGER_SUPPLIED_GUEST_ARRAY_REQUIRED=NO
EAGER_GUEST_CONTEXT_PHYSICAL_OBJECT_REQUIRED=NO
UNIVERSAL_RICH_ACTIVATION_HOT_PATH=REJECTED
UNIVERSAL_PREPARED_CALL_CARRIER_HOT_PATH=REJECTED
```

I072 had already established this physical shape for ordinary selected source
method calls, while direct source Closure invocation remained on the rich
`ProtosActivation.forClosureInvocation(...)` path.

This PERF025 slice consumes that residual direct-Closure gap without redefining
the architecture.

## Previous direct source Closure shape

Before this product revision, stable direct source Closure calls and Task-owned
direct source Closure preparation could construct:

```text
ProtosActivation.forClosureInvocation(...)
  -> fresh guest execution Context
  -> frozen guest Array for supplied arguments
  -> ReturnHome relation
  -> rich ProtosActivation
  -> source target
```

The same rich factory therefore remained on the route used by reusable hosted
execution despite the earlier PERF014 direct-call selection work.

## Compact direct Closure ABI

`ProtosFrameArguments` now represents direct source Closure entry as a distinct
compact call kind.

Its documented runtime header is conceptually:

```text
closure
DIRECT_CLOSURE_CALL marker
explicit Task or null
caller activation/provenance
invocation ReturnHome
supplied values
```

The existing compact ordinary method-call ABI remains distinct.

For direct Closure invocation:

- receiver comes from the Closure capture;
- `methodHome` comes from the Closure capture;
- an explicit Task is transported for Task-owned entry;
- null Task means inherit caller Task/dynamic-control state;
- supplied positional values remain internal transport values rather than an
  eager guest Array.

`ProtosFrameArguments.activation(...)` detects the compact direct-Closure kind,
materializes exactly one callee activation after target entry, and publishes that
same activation back into frame argument 0.

## Deferred direct Closure activation

The new runtime factory is:

```text
ProtosActivation.forDirectClosureInvocationWithReturnHomeForRuntime(...)
```

It preserves the semantics of `forClosureInvocation(...)` while allowing the
guest execution Context and supplied guest Array to remain deferred.

The published shape is therefore:

```text
direct source Closure caller
  -> compact frame arguments
  -> source target
  -> root activation materialization
       context = deferred
       supplied guest Array = deferred
       captured lexical contexts = preserved
       captured receiver = preserved
       captured methodHome = preserved
       ReturnHome identity/ownership = preserved
       Task/dynamic-control provenance = preserved
```

This is representation-only optimization under the already-ratified PLAT040 F′
contract.

## Stable guest direct Closure calls

The existing PERF014 stable direct Closure selection remains in place.

The product changelog explicitly records that stable direct Closure-call hits in
`finishDirectClosureCall` keep their existing selection and `DirectCallNode`
target while replacing the pre-target rich activation with the compact
direct-Closure frame ABI.

The intended steady-state structure is now:

```text
stable direct Closure selection
  -> DirectCallNode
  -> compact direct-Closure frame arguments
  -> source target
  -> deferred callee activation
```

No general source-call lookup was reintroduced as part of this slice.

## Task-owned entry and PreparedTopLevel

`prepareTaskOwnedDirectClosureIfBytecode(...)` now prepares a compact direct
source Closure call rather than eagerly constructing a rich callee activation.

This is the source-backed route used by `PreparedTopLevel.invoke()`.

The compact ABI carries the exact owning Task when the creator activation is not
itself that Task's activation. The callee receives that Task when activation is
materialized after source-target entry; the creator is not mutated to simulate
child ownership.

Thus the published path is:

```text
PreparedTopLevel.invoke()
  -> root Task execution
  -> Task-owned direct source Closure preparation
  -> compact direct-Closure target arguments
  -> source target
  -> activation materialization with exact Task
```

The public hosted-session API and Process/Context/Engine lifecycle are unchanged.

## Arguments and rest parameters

Ordinary supplied positional values no longer require an eager guest
`ProtosArrayValue` merely for direct source-call transport.

The focal regression verifies that direct source activation initially retains
deferred supplied arguments with no guest Array.

Rest parameters retain their semantic Array requirement and materialize the
required guest Array when binding requires it.

Therefore:

```text
ORDINARY_POSITIONAL_GUEST_ARRAY_EAGER=NO
REST_ARGUMENT_GUEST_ARRAY_SEMANTICS=PRESERVED
```

## Context identity

A direct source Closure invocation still has a fresh logical execution Context.

The physical guest Context object may remain absent until observation. When it
is materialized, the same callee activation remains authoritative.

The focal regression covers successive direct invocations and verifies distinct
activation and Context identities.

```text
FRESH_LOGICAL_CONTEXT_SEMANTICS=PRESERVED
EAGER_GUEST_CONTEXT_REQUIRED=NO
```

## Capture, receiver and method-home semantics

The direct compact factory preserves the Closure's existing:

```text
captured lexical contexts
captured receiver
captured methodHome
captured ReturnHome where applicable
```

The published regression set covers capture-by-reference and `this` behavior.

No second lexical binding store or capture-by-value representation is
introduced.

## ReturnHome and non-local return

This slice intentionally does **not** redesign the physical ReturnHome
representation.

The compact call still establishes the invocation home required by existing
semantics, and the callee activation receives the exact same home identity.

The focal regression covers:

- owned-home completion;
- non-local return consumed by the owning home;
- nested/escaped behavior;
- InvalidReturn/failure semantics.

Thus:

```text
RETURN_HOME_REDESIGNED=NO
RETURN_HOME_SEMANTICS=PRESERVED=PASS
NON_LOCAL_RETURN_SEMANTICS=PRESERVED=PASS
```

Physical eager ReturnHome creation remains a separate PERF025 pay-as-you-grow
candidate.

## Frame materialization boundary

This slice does **not** change the BUG013-correct frame-lifetime rule and does not
attempt to remove the current eager `frame.materialize()` performed when
installing frame lexical authority.

In particular:

```text
FRAME_MATERIALIZATION_CHANGED=NO
RAW_VIRTUALFRAME_RETENTION_INTRODUCED=NO
BUG013_LIFETIME_INVARIANT_PRESERVED=YES
```

The separate PERF025 finding that a root with any frame-backed binding may
materialize its frame remains open.

## PublishFrameActivation boundary

The semantic root still publishes/materializes the exact
`ProtosActivation` at root entry.

This slice does not attempt to make the activation shell itself lazy.

The improvement is narrower:

```text
pre-target rich direct Closure activation = removed
guest Context = deferred
guest supplied Array = deferred
root activation identity = preserved
```

A future investigation may separately determine whether activation-shell
materialization itself can be made more conditional.

## Source/native boundary

The new compact ABI is for direct **source-backed** Closure invocation.

Native Closure execution, structured-control special paths, generic fallback,
and synchronous host Closure paths retain their existing shape unless they
already used another established optimized route.

No artificial source-frame ABI is imposed on Native execution.

## Published focal regression coverage

The new
`ProtosPerf025DirectSourceClosureCompactInvocationTest` contains these focal
tests:

```text
stableDirectHitEntersSourceTargetThroughCompactAbiWithDeferredState
successiveDirectInvocationsGetDistinctActivationsAndContexts
restParameterStillMaterializesItsGuestArray
taskOwnedDirectSourceClosureUsesCompactPathWithExactTask
rootTaskClosureExecutionKeepsNonLocalReturnAndFailureSemantics
guestDirectCallsKeepCaptureThisAndNonLocalReturnSemantics
compactDirectNonLocalReturnIsConsumedByTheOwnedHome
```

The published test emits the following explicit acceptance markers:

```text
PERF025_DIRECT_SOURCE_CLOSURE_COMPACT_ABI=PASS
PERF025_DIRECT_SOURCE_EAGER_GUEST_CONTEXT=NO
PERF025_DIRECT_SOURCE_EAGER_GUEST_ARGUMENT_ARRAY=NO
PERF025_FRESH_CONTEXT_SEMANTICS=PASS
PERF025_REST_ARGUMENT_SEMANTICS=PASS
PERF025_TASK_OWNED_DIRECT_SOURCE_COMPACT_PATH=PASS
PERF025_RETURN_HOME_SEMANTICS=PASS
PERF025_NON_LOCAL_RETURN=PASS
PERF025_CAPTURE_BY_REFERENCE=PASS
PERF025_THIS_SEMANTICS=PASS
```

These are published regression assertions/markers. Runtime validation execution
remains human-owned.

## Validation provenance

The maintainer reported in the active interaction:

```text
tested and pushed PERF025: compact deferred invocation for direct source Closure calls
```

The coordinating agent independently verified:

```text
REMOTE_PUBLICATION=PASS
REMOTE_HEAD=fe078828412aabfc48dc1835bf34dce784307b3e
COMMIT_SUBJECT_MATCH=PASS
EXACT_PARENT_MATCH=PASS
VERSION=0.3.150-SNAPSHOT
EXPECTED_CHANGED_PATHS=PASS
FOCAL_TEST_PUBLISHED=PASS
```

Validation execution itself remains human-owned under repository policy.

Therefore:

```text
MAINTAINER_REPORTED_VALIDATION=PASS
INDEPENDENT_REEXECUTION_BY_COORDINATING_AGENT=NO
```

No command-by-command PASS result beyond the maintainer's report is invented by
this evidence.

## Specification and architecture impact

```text
OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
NEW_PLAT_DECISION=NO

PLAT040_F_PRIME_DIRECT_SOURCE_CLOSURE_CONVERGENCE=PASS
PERF014_STABLE_DIRECT_SELECTION_PRESERVED=YES
I072_COMPACT_DEFERRED_MODEL_REUSED=YES
BUG013_FRAME_LIFETIME_RULE_PRESERVED=YES
```

No specification path changed in the exact product commit.

## License and publication metadata

The new Protos-owned test source carries the required APL-1.0 Part 5 notice.

The exact product delta does not modify `LICENSE.TXT` or licensing policy.

`pom.xml` advances the implementation version from
`0.3.149-SNAPSHOT` to `0.3.150-SNAPSHOT`, and `CHANGELOG.md` records the
implementation slice.

## Structural acceptance

The published product establishes:

```text
PERF025_DIRECT_SOURCE_F_PRIME_STATUS=PASS

DIRECT_SOURCE_CLOSURE_COMPACT_ABI=PASS

DIRECT_SOURCE_PRETARGET_RICH_ACTIVATION=NO
DIRECT_SOURCE_EAGER_GUEST_CONTEXT=NO
DIRECT_SOURCE_EAGER_GUEST_ARGUMENT_ARRAY=NO

TASK_OWNED_DIRECT_SOURCE_COMPACT_PATH=PASS
PREPARED_TOP_LEVEL_SOURCE_PATH=COMPACT_DEFERRED

DIRECT_CLOSURE_STABLE_SELECTION_PRESERVED=PASS
DIRECT_CLOSURE_DIRECT_CALL_NODE_PRESERVED=PASS

FRESH_CONTEXT_SEMANTICS=PASS
CAPTURE_BY_REFERENCE=PASS
THIS_SEMANTICS=PASS
ARGUMENT_BINDING=PASS
REST_ARGUMENT_SEMANTICS=PASS

RETURN_HOME_SEMANTICS=PASS
NON_LOCAL_RETURN=PASS

RAW_VIRTUALFRAME_RETENTION=NO
BUG013_INVARIANT_PRESERVED=PASS

RETURN_HOME_REDESIGNED=NO
FRAME_MATERIALIZATION_CHANGED=NO
PUBLISH_FRAME_ACTIVATION_LAZINESS_CHANGED=NO

MAINTAINER_REPORTED_VALIDATION=PASS

OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
```

## Performance-claim boundary

No benchmark result is retained by this record.

The source audit had identified rich direct Closure invocation as a P0 structural
pay-as-you-grow candidate. This product slice removes that structure from direct
source-backed calls, but no attributable nanosecond, allocation-survival, or
end-to-end speedup claim is made here.

```text
ATTRIBUTABLE_NS_PER_CALL=UNKNOWN
SURVIVING_HEAP_ALLOCATION_REDUCTION=NOT_MEASURED
END_TO_END_SPEEDUP_PERCENT=NOT_MEASURED
DOMINANT_RUNTIME_CAUSE=NOT_ESTABLISHED_BY_THIS_SLICE
```

## PERF025 remaining boundary

This slice completes the direct source Closure invocation part of the PLAT040 F′
convergence. It does **not** close PERF025 as a whole.

The retained post-F1 pay-as-you-grow audit still includes independent residual
surfaces, notably:

```text
EAGER_FRAME_MATERIALIZATION_FOR_ANY_LOCAL
UNIVERSAL_CLOSURE_CONTEXT_CAPTURE
RETURN_HOME_PHYSICALLY_CREATED_WHEN_UNOBSERVED
PARAMETER_BINDING_THROUGH_DYNAMIC_SLOT_AUTHORITY
EAGER_FRAME_AUTHORITY_OVERFLOW_AND_ORDER_STRUCTURES
```

Integer invocation and current lexical-write candidates were separately promoted
to PERF027 and PERF028 and are not re-owned by this slice.

```text
PERF025_STATUS=OPEN
DIRECT_SOURCE_CLOSURE_F_PRIME_SLICE=COMPLETE
NEXT_SLICE_NOT_SELECTED_BY_THIS_RECORD
```

## Materially inspected publication evidence

The coordinating publication review inspected:

- `guillermomolina/protos@fe078828412aabfc48dc1835bf34dce784307b3e`;
- the exact one-commit compare from
  `aaa9130a9bd7190526f2c62a4317e8d90090c423`;
- `CHANGELOG.md`;
- `pom.xml`;
- `ProtosFrameArguments.java`;
- `ProtosActivation.java`;
- `ProtosBytecodeRootNode.java`;
- `ProtosBytecodeTaskExecution.java`;
- `ProtosSemanticBytecodeRootNode.java`;
- `ProtosPerf025DirectSourceClosureCompactInvocationTest.java`;
- the retained PERF025 post-F1 pay-as-you-grow audit;
- repository PERF/implementation/coordination instructions.

This record is evidence only and does not replace GitHub Issue live
coordination.
