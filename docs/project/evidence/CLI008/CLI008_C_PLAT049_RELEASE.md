# CLI008-C — PLAT049 release and implementation handoff

## Checkpoint

CLI008-C / `guillermomolina/protos#416` was previously blocked because the
completed PERF006-B4/B5 machinery did not define the complete D063 S3 guest-frame
sequence or failure-time snapshot authority.

PLAT049 / `guillermomolina/protos#790` has now resolved that missing platform
choice.

Project-owner approval on 2026-10-04:

~~~text
Apruebo C .
~~~

## Released architecture

~~~text
PLAT049_STATUS=RATIFIED
SELECTED_CANDIDATE=C

CAPTURE=FAILURE_ONLY
PHYSICAL_BASE=TRUFFLE_STACK_OR_CURRENT_FRAME_WALK
SEMANTIC_NORMALIZATION=CONTINUATION_ROOT_TO_SOURCE_BYTECODE_LOCATION
INLINE_AUGMENTATION=ACTIVE_PLAT044_NESTED_ROOTTAG
RETAINED_FORM=BOUNDED_IMMUTABLE_PROTOS_DIAGNOSTIC_TRACE
TRACE_BOUND=64
~~~

The trace belongs to the failure occurrence, not to guest-visible Error state.

A failed `Future.value()` observation is a consumer-local signal only.
Actor/Process causal stack concatenation remains forbidden.

## CLI008-C state

~~~text
PREVIOUS_STATE=BLOCKED_BY_PLAT049
CURRENT_STATE=READY_FOR_IMPLEMENTATION
NEXT_SLICE=CLI008-C1
NEXT_SLICE_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

The prior re-audit evidence remains historical and must not be rewritten:

`docs/project/evidence/CLI008/CLI008_C_GUEST_STACK_CAPTURE_REAUDIT.md`

The ratified platform authority is:

`docs/project/decisions/platform/PLAT049_GUEST_DIAGNOSTIC_STACK_CAPTURE_AUTHORITY.md`

Ratification evidence is:

`docs/project/evidence/PLAT049/PLAT049_CANDIDATE_C_RATIFICATION_EVIDENCE.md`
