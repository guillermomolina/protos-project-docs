# PERF025 — Residual inline-callback Activation consumer specialization

## Status

```text
STATUS=COMPLETE
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=cfc0fb433e82f0478c9fff9cc965c3fc506fabc9
PRODUCT_PARENT_REVISION=bd8617157d9c8cc6e1ce0edc0502b7a361e1e891
PRODUCT_VERSION=0.3.169-SNAPSHOT
PRODUCT_COMMIT=PERF025: specialize residual inline callback Activation consumers

PLAT044_DECISION=B_PRIME_RATIFIED_UNCHANGED
NEW_PLATFORM_DECISION_REQUIRED=NO
OBSERVABLE_LANGUAGE_SEMANTIC_CHANGE=NO
NEW_CALLBACK_ABI=NO
NEW_SEMANTIC_CARRIER=NO
NEW_ROOTCALLTARGET=NO

MAKE_TEST_REPORTED_BY_MAINTAINER=PASS
FINAL_STATIC_REVIEW=PASS
REVALIDATION_REQUIRED=NO
PRODUCT_PUSH=PASS

BENCHMARKS_RUN_FOR_THIS_SLICE=NO
PERFORMANCE_MAGNITUDE_CLAIMED=NO
```

## Purpose

Close the optional third inline-callback residual slice released by the
post-lazy-Activation audit.

The preceding PERF025 publications established:

```text
CALLBACK_STATIC_FRAME_LOCAL_AUTHORITY=COMPLETE
CALLBACK_LAZY_SEMANTIC_ACTIVATION=COMPLETE
```

The bounded post-Slice-2 audit then established that several common successful
PLAT044 B′ callback operations still forced semantic callback
`ProtosActivation` materialization only because their existing Bytecode
operations accepted a rich Activation operand.

This publication specializes those residual consumers against the existing
`PreparedInlineLiteralCall` carrier without changing B′ admission, callback
identity, the physical-root boundary, or semantic/tooling observation rules.

## Published carrier-native consumers

The successful path is now carrier-native for:

```text
statically resolved captured reads
statically resolved captured writes
canonical Integer sends
guarded ordinary source sends
fast ordinary source sends
direct source-Closure calls
THIS
successful member reads
successful inline multiple creation
```

The lowerer selects carrier-operand Bytecode operations for an admitted
frame-native callback. If the operation can complete from the existing carrier
projection, no callback Activation is constructed.

Otherwise the operation uses the existing durable materialization boundary and
runs the previous semantic operation unchanged.

No second callback ABI, aggregate, surrogate Activation, global registry,
synthetic RootCallTarget, FrameInstance, or Truffle stack element was added.

## Provenance projection

Child invocation preparation reuses the callback invocation's compact caller
only when the existing compact authority proves it equivalent to materializing
the callback first.

The implementation's projection is deliberately conservative. In particular,
the direct compact provenance caller is used only when the child is a direct
Closure call, the compact Task slot does not introduce a distinct task value,
and the callback Closure prelude is absent or identical to the compact caller's
prelude.

Where exact inheritance cannot be established from the existing carrier, the
callback Activation is materialized and the existing path remains authoritative.

This preserves:

```text
Task ownership
dynamic-control inheritance
actorModuleState
currentModuleKey
executionDomain
prelude
receiver
methodHome
ReturnHome
multi-Context ownership
```

## Captured lexical access

Static captured reads now use the callback Closure's captured lexical
environment while preserving D179 current-block and captured-chain membership
checks.

Static captured write destination selection is likewise carrier-native where
the destination is proven before RHS evaluation.

PERF028-A remains unchanged:

```text
DESTINATION_SELECTED_BEFORE_RHS=YES
DESTINATION_RETAINED_ACROSS_RHS=YES
POST_RHS_RETARGET=NO
```

ABSENT, nearer-binding, FROZEN, dynamic, invalid-mutation and Error paths retain
the existing materializing fallback.

## Canonical sends and direct Closure calls

Canonical Integer successful operations preserve the existing D013/canonical
guard authority while no longer materializing the callback merely to supply the
caller Activation.

Guarded/fast ordinary source sends and direct source-Closure calls similarly use
the compact equivalent caller only when provenance equivalence is exact.

Noncanonical selection, override/shadowing, foreign Context, structured send,
generic lookup and Error paths remain on the existing materializing path.

## THIS and member reads

`THIS` now obtains the direct Closure invocation's semantic receiver from the
Closure's captured receiver when the callback remains unmaterialized.

`CONTEXT` is unchanged: it remains a genuine semantic observer and the
existing persistent-authority boundary still prevents it from becoming a
frame-native no-Activation operation.

Successful member reads can use receiver/name/prelude and the existing PIC
authority without materializing the callback. Missing-member and unsupported
representation Error paths materialize as before.

Fresh receiver-bound Closure extraction, exact-receiver PIC behavior, shared
inherited-member PIC behavior and D013 invalidation remain unchanged.

## Multiple creation

The inline callback multiple-create success path no longer performs
`durableActivation(...)` before observing a valid fixed Array prefix.

It first observes the complete required prefix and then performs creation in
order, preserving the existing D143 contract:

```text
FIXED_PREFIX_OBSERVED_BEFORE_FIRST_CREATE=YES
CREATE_LEFT_TO_RIGHT=YES
ROLLBACK_AFTER_LATER_FAILURE=NO
```

Invalid/short source and duplicate/mutation Error paths retain materialization
where needed for the exact guest Error.

## Intentional materialization boundaries

The following remain intentional observers or fallbacks:

```text
explicit context observation
debugger/tooling scope observation
non-local return / control transfer
D179 ABSENT or dynamic fallback
arity Errors
duplicate creation / mutation Errors
unsupported representation / lookup failure
noncanonical/generic send or call fallback where required
already-materialized invocation
non-frame-native callback
non-admitted PLAT044 fallback
```

Suspension alone remains non-observing:

```text
MATERIALIZE_ON_SUSPENSION_ALONE=NO
```

## Deliberate residuals

The implementation does not attempt a global elimination of
`emitCurrentActivation(...)`.

The following remain materializing or otherwise outside this bounded slice:

```text
PrepareClosureCallVector
PrepareSendVector
PrepareDefaultClosureCallArguments
super sends
non-local return
ComplementEqualityResult / !=
Closure literal materialization
create/assign with explicit target
generic Lookup
structured send execution
```

These residuals do not justify another PERF025 product slice on the current
evidence.

## fastOrdinarySend miss review

The final static review explicitly examined the inline `fastOrdinarySend`
lookup-miss behavior.

The carrier-native tier binds a miss as no selected value and falls through to
the generic path, which materializes and raises the same guest lookup Error
through the existing ordinary lookup implementation.

This can change only the post-error internal specialization state of that site:
after a miss, the generic specialization may replace the PIC tiers.

The final review classified this as:

```text
FAST_ORDINARY_SEND_MISS_DIFFERENCE=ACCEPTABLE_IMPLEMENTATION_ONLY
OBSERVABLE_ERROR_SEMANTICS_CHANGED=NO
D013_INVALIDATION_CHANGED=NO
MULTI_CONTEXT_CHANGED=NO
```

Any effect is limited to possible performance at a site that has raised a
lookup error.

## Validation and publication

The maintainer reported the product validation green before metadata closure:

```text
MAKE_TEST_REPORTED_BY_MAINTAINER=PASS
```

After that green validation, only version/CHANGELOG metadata was changed:

```text
VERSION_BUMP=0.3.168-SNAPSHOT -> 0.3.169-SNAPSHOT
CHANGELOG_UPDATED=YES
JAVA_CHANGED_AFTER_GREEN_TEST=NO
TESTS_CHANGED_AFTER_GREEN_TEST=NO
REVALIDATION_REQUIRED=NO
FINAL_STATIC_REVIEW=PASS
PUSH_READY=YES
```

Published product files:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java
src/main/java/com/guillermomolina/protos/execution/ProtosInlineCallbackFrameBindings.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf025CallbackConsumerSpecializationTest.java
```

GitHub publication verifies:

```text
PRODUCT_REVISION=cfc0fb433e82f0478c9fff9cc965c3fc506fabc9
PRODUCT_PARENT_REVISION=bd8617157d9c8cc6e1ce0edc0502b7a361e1e891
PRODUCT_VERSION=0.3.169-SNAPSHOT
PRODUCT_PUSH=PASS
```

## PERF025 residual after this publication

This completes the callback implementation sequence:

```text
CALLBACK_STATIC_FRAME_LOCAL_AUTHORITY=COMPLETE
CALLBACK_LAZY_SEMANTIC_ACTIVATION=COMPLETE
RESIDUAL_ACTIVATION_CONSUMER_SPECIALIZATION=COMPLETE
INLINE_CALLBACK_TECHNICAL_LINE=COMPLETE
```

The previously reconciled non-benchmark residuals are also closed or explicitly
not justified for further PERF025 implementation:

```text
D179_CURRENT_RESOLVED_READ_MEMBERSHIP=COMPLETE
D179_CAPTURED_MEMBERSHIP_SPECIALIZATION=NOT_JUSTIFIED

COLLECTION_SNAPSHOT_PHYSICAL_REPRESENTATION=COMPLETE_FOR_PERF025
ADDITIONAL_MAP_BYTES_SNAPSHOT_SLICE=NOT_JUSTIFIED

SHARED_SHAPE_MEMBER_LOOKUP=H3A_COMPLETE
GENERAL_SHAPE_REQUIRED_FOR_PERF025=NO

ROOT_TASK_TASK_ACTOR_LINE=COMPLETE
```

Therefore:

```text
REMAINING_NON_BENCHMARK_PRODUCT_WORK=NO
FINAL_BENCHMARK_PENDING=YES
PERF025_STATUS=OPEN_PENDING_FINAL_BENCHMARK
```

The historical PERF025-D3 reference remains valid historical evidence for the
then-final product revision `19d7426a...` / 0.3.143-SNAPSHOT. PERF025
subsequently continued through additional product optimization publications up
to this 0.3.169-SNAPSHOT revision, so D3 cannot serve as the final closure
measurement for the current product state.

A new exact-revision final benchmark/reference is therefore the remaining
PERF025 step before issue closure.
