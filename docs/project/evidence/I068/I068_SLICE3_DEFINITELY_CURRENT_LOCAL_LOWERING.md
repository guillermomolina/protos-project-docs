# I068 Slice 3 — definitely-current local lowering

Status: **PUBLISHED / RETAINED EVIDENCE**

Owning Issue: `guillermomolina/protos#708`

## Exact publication identity

~~~text
PROTOS_REVISION=1756b3d3100d54ef1627bc618cae6b7ef8da3445
PROTOS_VERSION=0.3.83-SNAPSHOT
COMMIT_MESSAGE=I068: lower definitely-current lexicals to frame-backed locals

I068_SLICE_3=DEFINITELY_CURRENT_LOCAL_LOWERING
PLAT036_SELECTED_CANDIDATE=D
PRIOR_E1_RECOMMENDATION=SUPERSEDED
RUNTIME_AUTHORITY_CUTOVER=CURRENT_RESOLVED_ONLY
~~~

Immediate predecessor:

~~~text
I068_SLICE2_PRODUCT_REVISION=1ccbb717fa824acd7aef71bd89c876e3bf8f6fe7
I068_SLICE2_RECORD_REVISION=6a0f8cbf294316e49c73ea604e39029e8acff7a7
~~~

GitHub comparison establishes that the Slice 3 product publication is exactly
one commit ahead of Slice 2:

~~~text
COMPARE_STATUS=ahead
AHEAD_BY=1
BEHIND_BY=0
TOTAL_COMMITS=1
~~~

## Product delta

Changed paths:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalLexicalScope.java
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java
src/main/java/com/guillermomolina/protos/runtime/ProtosExecutionContextValue.java
src/main/java/com/guillermomolina/protos/runtime/ProtosObjectValue.java
src/test/java/com/guillermomolina/protos/execution/ProtosPlat036Slice3FrameBackedCurrentLocalTest.java
~~~

The CHANGELOG entry is owned by `I068`; PLAT036 Candidate D is cited only as
the governing ratified architecture.

## Authority cutover implemented

Slice 3 performs the first runtime value-authority cutover under PLAT036
Candidate D.

For a statically `Resolved` binding owned by the genuine current execution
context scope (`ROOT` / `CLOSURE`), the lowering creates a stable Truffle
Bytecode DSL `BytecodeLocal`. Eligible bare reads use a generated
`LocalAccessor` path rather than `Lookup -> ProtosActivation.lookup(String)`.

The corresponding execution-context object installs
`ProtosFrameLexicalBindingAuthority`, which projects the same frame-backed
storage through the ordinary first-class context API. Writes performed through
the context seam therefore update the same authoritative local storage rather
than a mirror map.

The implementation uses generated `LocalRangeAccessor` metadata for the
frame-backed authority rather than manually constructing LocalAccessor
instances.

## Single-authority and handoff behavior

Some genuine execution contexts are already populated by bootstrap/tooling
before the lowered root begins execution. Slice 3 therefore performs a
preserving authority handoff:

~~~text
old map-backed authority remains visible
    -> populate replacement frame-backed authority
    -> switch the object's single authority reference
    -> old authority is no longer reachable through the context
~~~

This preserves already-established bindings without introducing a guest-visible
dual-authority period.

~~~text
ONE_SEMANTIC_BINDING_VALUE_AUTHORITY_REQUIRED=YES
FRAME_BACKED_CURRENT_BINDING_AUTHORITY=YES
DUAL_AUTHORITATIVE_COPIES=NO
PREEXISTING_CONTEXT_BINDINGS_PRESERVED=YES
~~~

## Admission boundary preserved

Slice 3 deliberately does not broaden the direct-local admission boundary:

- `CanonicalBindingResolution.Resolved` owned by the current genuine execution
  context is eligible;
- `Candidate` references remain on the exact dynamic presence/topology path;
- `Dynamic` references remain on the String-keyed receiver/member fallback;
- object-construction bodies (`OBJECT_BODY`) remain ordinary object state;
- closure parameters/default establishment remain outside this slice;
- captured/materialized outer lexical bindings remain outside this slice; and
- legacy/internal activations whose current lexical context is an ordinary
  `ProtosObjectValue` retain the historical generic lookup path.

No lazy physical execution-context materialization is introduced.

## Presence, mutation and escape invariants

Focused Slice 3 coverage retains evidence for:

- bare create/read through frame-backed authority;
- assignment observing the same authoritative value;
- `PRESENT(null) != ABSENT`;
- nearer not-yet-present `Candidate` lookup retaining dynamic fallback;
- fully dynamic lookup retaining String-keyed receiver/member behavior;
- object-body slots remaining ordinary object storage;
- OPEN/CLOSED/FROZEN mutation-state behavior;
- D179/C3 monotonic execution-context membership; and
- an escaped execution-context object observing its frame-backed binding after
  the activation that created it has returned.

## Bytecode DSL retained-parser compatibility

Truffle `BytecodeRootNodes` retains its parser and may invoke it again when
source or instrumentation metadata is materialized. Slice 3 therefore restores
the root-specific canonical analysis/top-scope/local-layout lowering state on
every parser invocation.

The integrated regression
`ProtosPerf006B2AClosureActivationTest.bytecodeClosureRootRetainsBodySourceSection`
reported:

~~~text
PERF006_B2A_CLOSURE_BODY_SOURCE_SECTION=PASS
~~~

This specifically retains lazy source-section materialization after the Slice 3
lowering-state cutover.

## Validation evidence

The maintainer reported the following validation on the final product state:

~~~text
GIT_DIFF_CHECK=PASS
MAKE_COMPILE=PASS
I068_SLICE3_FOCAL_TESTS=PASS
MAKE_TEST_JAVA=PASS

MAKE_TEST_PROTOS=PASS
PROTOS_TESTS_PASSED=1250
PROTOS_TESTS_FAILED=0
PROTOS_TESTS_TOTAL_TIME_SECONDS=113
~~~

The native Protos suite completed with:

~~~text
[package-tool-project-projection] 4/4 passed
1250 passed, 0 failed
Protos test tool bootstrap
test
Protos tests total time: 113 s
~~~

No GitHub combined status or associated pull-request workflow run was returned
for the product revision at record time. This record therefore does not claim a
remote CI PASS.

## Remaining I068 boundary

Slice 3 does not complete I068.

~~~text
NEXT_SLICE=I068_SLICE_4_SEQUENTIAL_DEFAULT_PARAMETER_LOWERING
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos

PARAMETER_FRAME_LOCAL_MIGRATION=NOT_YET_IMPLEMENTED
CAPTURED_MATERIALIZED_LEXICAL_LOWERING=NOT_YET_IMPLEMENTED
DEBUGGER_REFLECTION_PROJECTION_SLICE=NOT_YET_IMPLEMENTED
PROTOS_ACTIVATION_FALLBACK_CLEANUP=NOT_YET_IMPLEMENTED
LAZY_CONTEXT_MATERIALIZATION=NOT_IMPLEMENTED
~~~

Slice 4 owns sequential/default parameter lowering and must preserve semantic
absence until each parameter binding point. It must not pull captured/materialized
outer lexical migration, debugger projection or final generic fallback cleanup
forward from their allocated slices.
