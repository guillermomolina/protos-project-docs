# TEST009-W — single-root CodeTooLarge causal checkpoint

## Scope and handoff state

```text
WORK_ITEM=TEST009/#795
SLICE=TEST009-W
WORK_TYPE=DIAGNOSIS_THEN_CAUSAL_PRODUCT_REPAIR_IF_SUFFICIENT
PRODUCT_REPOSITORY=guillermomolina/protos

BASE_PUBLISHED_PROTOS_REVISION=e1904bb01b61c6ee0da5fd279da0f7feb143229e
BASE_PUBLISHED_PROTOS_VERSION=0.3.241-SNAPSHOT
DIAGNOSTIC_WORKSPACE=/tmp/test009-w-088d2
SELECTOR=protos-root:088d2ae81075aba8

CURRENT_CHECKPOINT_PRODUCT_PATCH=PUBLISHED
CURRENT_CHECKPOINT_PRODUCT_COMMIT=3c00089193bef2669b4fe9ccc36707c284920b3a
CURRENT_CHECKPOINT_PRODUCT_VERSION=0.3.244-SNAPSHOT
CURRENT_CHECKPOINT_PRODUCT_PUSH=YES
```

The retained TEST009-W worktree was created from the exact published base above but contained
local TEST009-W diagnostic/lifecycle changes while the single-root evidence was acquired. This
checkpoint therefore does not claim that the dirty worktree itself is a published Protos revision.
No product repair is published by this record.

At checkpoint time Protos `main` had already advanced beyond that base. Any product repair must
therefore be prepared against the then-current real HEAD, preserving concurrent changes, rather
than publishing the old temporary worktree.

## Single-root capture

The V/V2/V3 workflow successfully isolated one current semantic root and produced a fresh BGV
whose After TruffleTier graph retained the target root.

```text
SELECTOR=protos-root:088d2ae81075aba8
SOURCE_FAMILY=protos/tools/test/Manifest.protos
TARGET_COMPILATION=FAILED_CODE_TOO_LARGE
TRUFFLE_ROOT_CAPTURE_READY=YES
TRUFFLE_ROOT_CAPTURE_BOUNDARY=BGV
AFTER_TRUFFLE_TIER_PRESENT=PASS
```

The temporary BGV and `diagnostic.log` were acquisition artifacts. The durable evidence below
records the causal measurements needed to continue from another machine; the binary BGV is not
a product artifact and is not required to be committed to Protos.

## Exact method aggregation

The selected expansion tree contained:

```text
PARSED_ROWS=24604
UNIQUE_METHOD_NAMES=524

Throwable.fillInStackTrace()                                  self_size=17303  occurrences=242
OptimizedCallTarget.profiledPERoot(Object)                    self_size=14469  occurrences=1
<root>                                                        self_size= 7959  occurrences=1
StringLatin1.inflate([B, I, [B, I, I)                        self_size= 5731  occurrences=22
ProtosFrameArguments.hasCompactHeader(Object)                 self_size= 3687  occurrences=85
Unsafe.putByte(Object, J, B)                                  self_size= 2604  occurrences=1672
System.arraycopy(Object, I, Object, I, I)                     self_size= 2020  occurrences=27
CachedBytecodeNode.handleLoadLocal$generic(...)                self_size= 1764  occurrences=23
FrameWithoutBoxing.unsafePutObject(...)                        self_size= 1726  occurrences=830
ProtosFrameArguments.activation(Object)                       self_size= 1718  occurrences=102
```

The important family aggregates were:

```text
GENERATED_DISPATCH  self_size=  540  rows=   6
SEND_PREPARATION    self_size=  593  rows= 596
INLINE_CALLBACK     self_size=  293  rows= 112
LOCAL_FRAME         self_size=14230  rows=8680
LEXICAL_CAPTURE     self_size= 2791  rows= 601
EXCEPTION_CONTROL   self_size= 4417  rows= 377
```

This falsified the initial idea that send preparation or inline callbacks were the dominant
self-cost. `LOCAL_FRAME` is large, but the largest exact method name was
`Throwable.fillInStackTrace()`.

## `Throwable.fillInStackTrace()` attribution

The 242 occurrences were then attributed through their immediate parents and six nearest
ancestors. The totals reconcile exactly:

```text
TARGET=Throwable.fillInStackTrace()
HITS=242
SELF_COUNT=1452
SELF_SIZE=17303

IMMEDIATE_PARENT Throwable.<init>(String)             self_size=15158 hits=212
IMMEDIATE_PARENT Throwable.<init>(String, Throwable)  self_size= 2145 hits= 30
SUM=17303
```

The dominant nearest Protos owners were:

```text
ProtosPrelude.errorPrototype()                                      5863
ProtosValueLookup.lookup -> delegationParent                         1573
ProtosValueLookup.lookup -> Optional.orElseThrow                     1573
ProtosPrelude.arrayPrototype()                                      1430
ProtosBytecodeRootNode.attachTaskOrInheritDynamicControlState(...)  1430
ProtosPrelude.standardErrorPrototype(String)                         1001
ProtosBytecodeRootNode.finishPreparingComposedCall(...)               858
ProtosBytecodeRootNode.prepareImmediateMethodCall ->
  rejectComposedInvocationProjection                                  715
PrepareSendArguments.guardedStructuredSend ->
  rejectComposedInvocationProjection                                  572
PreparedBooleanCall / generated exhaustive-switch defensive paths    residual
```

All 17,303 self-size units are attributable to host/JDK defensive exception construction paths.
The dominant pattern is repeated expansion of invariant-failure machinery such as
`Optional.orElseThrow()`, `UnsupportedOperationException`, and exhaustive-switch
`MatchException` paths.

## Stack-trace interpretation correction

The method-name ranking must not be interpreted as proof that Protos guest exceptions are
materializing Java stack traces.

The exact Graal revision pinned by the TEST009-V3 analyzer is:

```text
GRAAL_REV=95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b
```

At that revision `AbstractTruffleException.fillInStackTrace()` is final and returns `this`.
Protos has two `AbstractTruffleException` subclasses in this area (`ProtosSignalException` and
`ProtosBytecodeControlTransferException`), so the aggregated `Throwable.fillInStackTrace()` leaf
is evidence of host/JDK exception-constructor expansion from multiple callers, not evidence of
17,303 units of Protos guest-stack capture.

Therefore:

```text
GUEST_ERROR_STACK_TRACE_CAUSE=FALSIFIED
THROWABLE_METHOD_NAME_IS_SHARED_SINK=YES
HOST_DEFENSIVE_EXCEPTION_EXPANSION=CONFIRMED
LOCAL_FRAME_SECOND_MAJOR_AXIS=YES
LOCAL_FRAME_REPAIR_NOT_YET_AUTHORIZED=YES
```

## Methodological consequence

TEST009 already rejected boundary chasing and CodeTooLarge micro-repair batches. The W evidence
does not authorize placing `@TruffleBoundary` on the listed callers merely because they appear
in the ranking.

The next product change must be one causally justified repair that follows from a structural
invariant or representation decision. It must be implemented in `guillermomolina/protos` from
the real current HEAD, validated there, and published there. The temporary diagnostic worktree
is not the publication vehicle.

## Safe machine handoff

This checkpoint is sufficient to abandon the temporary W acquisition directory after the
maintainer no longer needs its raw BGV/log for ad-hoc inspection:

```text
DURABLE_ROOT_IDENTITY=RECORDED
DURABLE_CODE_TOO_LARGE_OUTCOME=RECORDED
DURABLE_METHOD_AGGREGATES=RECORDED
DURABLE_FAMILY_AGGREGATES=RECORDED
DURABLE_THROWABLE_ATTRIBUTION=RECORDED
DURABLE_INTERPRETATION_CORRECTION=RECORDED

SUPPORTING_PRODUCT_REPAIR=PUBLISHED
SUPPORTING_PRODUCT_PATCH_SCOPE=CLI_SESSION_TERMINATION_CANCELLATION_DRAIN
SUPPORTING_PRODUCT_FILES=3
PATCH_VALIDATION=MAINTAINER_REPORTED_PASS
PATCH_PUBLICATION=3c00089193bef2669b4fe9ccc36707c284920b3a
NEXT_REPOSITORY=guillermomolina/protos
NEXT_ACTION=FUTURE_CAUSAL_CODE_TOO_LARGE_REPAIR_FROM_CURRENT_HEAD
```

TEST009 remains open because the selected root still has a real CodeTooLarge residual. W itself
has reached a durable diagnostic and handoff checkpoint; its supporting CLI termination repair
is now validated and published in Protos.


## Handoff correction — local Protos patch exists

A subsequent maintainer worktree inspection exposed a real tracked TEST009-W product diff that
was present in the temporary Protos checkout and had not yet been published. The previous
`UNPUBLISHED_PRODUCT_REPAIR=NONE` classification was therefore incorrect and is superseded by
this section.

The local patch changes exactly these product/test files:

```text
src/main/java/com/guillermomolina/protos/cli/ProtosCli.java
src/test/java/com/guillermomolina/protos/cli/ProtosCliPolyglotRoutingArchitectureTest.java
src/test/java/com/guillermomolina/protos/cli/ProtosCliTest.java
```

Its causal purpose is separate from the CodeTooLarge graph interpretation: `Session.terminate()`
must request Process termination and drain cooperative cancellation work while the owning
Polyglot Process Context is entered, before blocking on Process terminality. The regression
creates a suspended root-domain Task, requests session termination on another thread, and fails
if an external test-only drain is required.

At reconciliation time, current Protos `main` was:

```text
RECONCILED_PROTOS_HEAD=41a06e07d5d74053911f291ac27f911678c61ff5
RECONCILED_PROTOS_COMMIT=LIB013-B: add temporal amounts and calendar arithmetic
```

The touched source regions in those three files remained unchanged on that HEAD, so there was no
semantic/content conflict with the concurrent LIB013 work. The maintainer then validated the
rebased/current-HEAD tree and published the repair.

```text
PUBLISHED_PROTOS_REVISION=3c00089193bef2669b4fe9ccc36707c284920b3a
PUBLISHED_PROTOS_VERSION=0.3.244-SNAPSHOT
COMMIT_SUBJECT=TEST009-W: drain cancellation during CLI termination
MAINTAINER_REPORTED_FUNCTIONAL_VALIDATION=PASS
MAINTAINER_REPORTED_GIT_DIFF_CHECK=PASS
PUBLICATION=PUSHED

SAFE_TO_DELETE_TEMP_WORKTREE=YES
PRODUCT_PUBLICATION_REPOSITORY=guillermomolina/protos
```

The published repair is supporting W infrastructure/lifecycle correctness. It does not claim to
remove the selected root's CodeTooLarge residual. That causal compilerability repair remains
future TEST009 work and must start from the real current Protos HEAD.


## Final W publication state

```text
TEST009_W_DIAGNOSTIC_CHECKPOINT=DURABLE
TEST009_W_SUPPORTING_PRODUCT_REPAIR=PUBLISHED
TEST009_W_PRODUCT_REVISION=3c00089193bef2669b4fe9ccc36707c284920b3a
TEST009_W_PRODUCT_VERSION=0.3.244-SNAPSHOT
TEST009_W_MACHINE_HANDOFF=COMPLETE

SELECTED_ROOT_CODE_TOO_LARGE=STILL_OPEN
TEST009_STATE=OPEN_IN_PROGRESS
NEXT_PRODUCT_WORK=CAUSALLY_JUSTIFIED_CODE_TOO_LARGE_REPAIR
NEXT_PRODUCT_REPOSITORY=guillermomolina/protos
```

The temporary TEST009-W acquisition directory and the local checkout no longer contain unique
unpublished project state required for continuation.
