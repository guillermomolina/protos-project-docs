# PLAT046 — Candidate B ratification evidence

Date: 2026-10-02

## Authority

~~~text
WORK_ITEM=PLAT046
ISSUE=guillermomolina/protos#778
SELECTED_CANDIDATE=B
OWNER_APPROVAL="Ok apruebo candidate B."
APPROVAL_DATE=2026-10-02
~~~

The approval applies to the exact Candidate B produced by the exhaustive
PLAT046-A research retained at:

~~~text
RESEARCH_REVISION=2763f36308c4ed967f923a66c6124607dfeef36d
RESEARCH_RECORD=
  docs/project/evidence/PLAT046/
  PLAT046_A_EXHAUSTIVE_MINIMAL_SINGLE_ACTOR_EXECUTION_RESEARCH.md
~~~

## Exact approved architecture

~~~text
ORDINARY_SINGLE_ACTOR_EXECUTION=CALLER_THREAD
DEDICATED_SECOND_GUEST_EXECUTION_THREAD_FOR_MINIMAL_PATH=NO

SESSION_SERIALIZATION=EXPLICIT_LOCAL_GATE
MULTI_CALLER_SAFETY=REQUIRED
CALL_VS_CLOSE_EXCLUSION=REQUIRED
PROCESS_CONTEXT_ENTRY=PRESERVED
ROOT_TASK_AND_ACTOR_SEMANTICS=PRESERVED

CHILD_ACTOR_CARRIERS=
  EXISTING_LAZY_RUNTIMEHOST_OWNED_SUBSTRATE

PREBUILT_STRONG_STACK_MECHANISM=NO
DEEP_RECURSION_10000_REQUIREMENT=RETAINED
STACK_GATE_BLOCKS_BASE_SELECTION=NO
~~~

## GITHUB021 invariant/delta check

The owner invariant recorded before final selection was:

~~~text
MINIMAL_ONE_CALLER_ONE_PROCESS_ONE_ROOTACTOR_SYNCHRONOUS_EXECUTION
  MUST_NOT_REQUIRE_A_SECOND_GUEST_EXECUTION_THREAD
  MERELY_FOR_UNUSED_CAPABILITY
~~~

Candidate B implements that invariant directly.

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
~~~

No previously ratified semantic invariant is contradicted.

Preserved:

- PLAT001: Java thread identity is not Actor/Process identity.
- PLAT010: Actors do not own dedicated physical threads by semantic identity.
- PLAT011: normal Actor carrier capacity remains RuntimeHost-owned and shared.
- PLAT014/016: optional/suspension machinery remains paid only when required.
- PLAT019/021/027/028: host/native stack identity is not semantic continuation
  identity.
- PLAT040: optional stronger physical machinery does not become a universal
  ordinary-call carrier.
- PLAT043/044: semantic behavior and physical execution structure remain
  separable when observable behavior is preserved.
- PLAT045: Native Image interpreter-only guest-JIT policy is untouched.

The direct delta is implementation placement:

~~~text
BEFORE=
  ProtosStandaloneHostedSession ordinary guest work
  -> dedicated protos-embedded-guest platform thread
  -> fixed requested 16 MiB stack
  -> LinkedBlockingQueue
  -> caller park/unpark

AFTER_SELECTED_ARCHITECTURE=
  ordinary guest work
  -> current caller
  -> explicit local session serialization
  -> existing Process Context / RootTask / Actor machinery
~~~

Historical BUG008/PERF025 stack evidence remains valid evidence. The universal
carrier consequence is superseded for the minimum execution path.

## Product and project-record identity at ratification

~~~text
PROTOS_REVISION=ff618f0dd8ef680f7884c9145552f77912f040a2
PROTOS_VERSION=0.3.145-SNAPSHOT

RESEARCH_PROJECT_RECORD_REVISION=
  2763f36308c4ed967f923a66c6124607dfeef36d

RATIFICATION_DECISION_PUBLICATION_REVISION=
  fabb1000e0e0f7369ade4ec073cfa014cf7c2e65

DECISION_RECORD=
  docs/project/decisions/platform/
  PLAT046_ORDINARY_HOSTED_SINGLE_ACTOR_CALLER_EXECUTION_BOUNDARY.md
~~~

## Coordination consequence

~~~text
PLAT046=RATIFIED
PERF025=#758 RELEASED_FROM_PLAT046_PAUSE

NEXT_SLICE=PERF025-F1
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
NEW_ISSUE_REQUIRED=NO

GOAL=
  cut over ProtosStandaloneHostedSession ordinary guest operations
  from ProtosGuestCarrier to direct caller-thread execution
  while preserving explicit session serialization, multi-caller safety,
  Context entry, RootTask/Actor semantics and close ordering

BENCHMARK_CHANGE=NO
MEASUREMENT_DURING_PRODUCT_SLICE=NO
REMEASURE_AFTER_VALIDATED_PRODUCT_PUBLICATION=YES
~~~

PERF025-F1 is intentionally bounded to the reusable hosted-session ownership
surface first. It must not simultaneously redesign Actor scheduling, Test Tool
parallel execution, generic guest recursion, stack fallback policy or benchmark
methodology.

## Implementation-risk assessment

The product change is not expected to be a large execution-engine rewrite.

The main current ownership is concentrated in
`ProtosStandaloneHostedSession`:

- one `ProtosGuestCarrier` field;
- carrier creation in `open()`;
- `carrier.call(...)` around open, dynamic invocation, preparation, prepared
  invocation and close;
- carrier-provided serialization as an implementation side effect.

The selected replacement needs an explicit session gate and exact close
cutover ordering.

Expected directly affected product/test surfaces include at least:

~~~text
src/main/java/.../execution/ProtosStandaloneHostedSession.java
src/main/java/.../execution/ProtosGuestCarrier.java   (possible retirement)
src/test/java/.../execution/ProtosGuestCarrierTest.java
hosted-session lifecycle/embedding tests
CHANGELOG/version metadata as required by repository policy
~~~

The shared `GUEST_CALL_STACK_SIZE_BYTES` constant cannot be mechanically
deleted in the same step because CLI/Test Tool surfaces still reference it and
are outside the bounded F1 ownership unless current-HEAD implementation proves
otherwise.

Therefore:

~~~text
IMPLEMENTATION_SIZE=SMALL_TO_MODERATE
TWO_LINE_CHANGE=NO
EXECUTION_ENGINE_REWRITE=NO
PRIMARY_RISK=SERIALIZATION_AND_CLOSE_LIFECYCLE_CORRECTNESS
~~~

## Cross references

- `guillermomolina/protos#778`
- `guillermomolina/protos#758`
- `guillermomolina/protos#681`
- `docs/project/evidence/PLAT046/PLAT046_A_EXHAUSTIVE_MINIMAL_SINGLE_ACTOR_EXECUTION_RESEARCH.md`
- `docs/project/decisions/platform/PLAT046_ORDINARY_HOSTED_SINGLE_ACTOR_CALLER_EXECUTION_BOUNDARY.md`
