# PLAT046 — callable-entry pay-as-you-grow amendment ratification evidence

Date: 2026-10-07

## Decision identity

~~~text
DECISION=PLAT046
ISSUE=guillermomolina/protos#778
ORIGINAL_RATIFICATION_DATE=2026-10-02
AMENDMENT_APPROVAL_DATE=2026-10-07

TRIGGER=PERF032/guillermomolina/protos#831
IMPLEMENTATION_OWNER=PERF033/guillermomolina/protos#832

PROTOS_REVISION=c21ff278e6dca1219429cc7f08568c54af2bd951
PROTOS_VERSION=0.3.274-SNAPSHOT
~~~

This record ratifies an amendment to the physical minimum callable boundary. It
does not change observable Protos language or Standard Library semantics.

## Approval provenance

The active PERF032 discussion presented the revised architecture as:

~~~text
Value.execute()
  -> HostToGuestRootNode
  -> InteropLibrary.execute
  -> Protos callable
  -> compact ordinary Closure call
  -> cached/direct guest CallTarget
~~~

with complete removal or justification of the Protos-owned fixed minimum-call
envelope rather than preservation of eager RootTask machinery.

The project owner explicitly approved that exact direction:

~~~text
ok aprobado
~~~

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
~~~

## Evidence requiring the amendment

PERF032 retained evidence established that the dominant cost of
`primitive-return-literal` lies before the first compiled guest CallTarget.

Current Protos source and existing ratified architecture also establish:

~~~text
CLOSURE_INVOCATION_INTRINSICALLY_REQUIRES_TASK=NO
ACTIVATION_TASK_OWNERSHIP=OPTIONAL
COMPACT_ZERO_ARG_CLOSURE_WITHOUT_RICH_ACTIVATION=PROVEN
TASK_OWNED_COMPACT_PATH_RETAINED_WHEN_TASK_EXISTS=YES
PLAT040_UNIVERSAL_RICH_INVOCATION_STATE=REJECTED
D189_SYNCHRONOUS_CALLBACK_CREATES_TASK=NO
~~~

The retained peer harness uses `Value.execute()` for GraalJS/GraalPy while the
Protos prepared runner uses `PreparedTopLevel.invoke()`, which currently
routes through a stronger Protos-owned host/session/RootTask envelope.

## Amendment

The original PLAT046 ratification remains authoritative for:

~~~text
ORDINARY_PHYSICAL_CARRIER=CALLER_THREAD
DEDICATED_SECOND_GUEST_EXECUTION_THREAD_FOR_MINIMAL_PATH=NO
CHILD_ACTOR_CARRIERS=LAZY_RUNTIMEHOST_OWNED_SUBSTRATE
DEEP_RECURSION_10000_REQUIREMENT=RETAINED
NO_PREBUILT_STRONG_STACK_MECHANISM_WITHOUT_CONCRETE_REQUIREMENT
PAY_ONLY_FOR_USE=YES
PAY_AS_YOU_GROW=YES
LOWER_LAYERS_PROTECTED=YES
~~~

The amendment clarifies/supersedes only the interpretation that the minimum
ordinary callable must eagerly traverse RootTask/Task/Actor/Process machinery.

~~~text
TASK_IDENTITY_AND_LIFECYCLE_WHEN_TASK_EXISTS=UNCHANGED
ROOTACTOR_TASK_SEMANTICS_WHEN_PRESENT=UNCHANGED

UNIVERSAL_ROOT_TASK_MATERIALIZATION=NO
UNIVERSAL_TASK_ALLOCATION=NO
UNIVERSAL_ACTOR_TASK_BOOKKEEPING=NO
UNIVERSAL_RICH_CALLEE_ACTIVATION=NO
UNIVERSAL_STRONGER_CAPABILITY_MACHINERY=NO

CANONICAL_MINIMUM_HOST_CALLABLE_BOUNDARY=
  STANDARD_POLYGLOT_VALUE_EXECUTE

CANONICAL_LANGUAGE_ENTRY=
  INTEROP_LIBRARY_EXECUTE
  -> COMPACT_ORDINARY_PROTOS_CLOSURE_INVOCATION
~~~

The original `ROOT_TASK_EXECUTION=UNCHANGED` requirement is classified as a
bounded F1 implementation constraint: F1 changed carrier topology without
simultaneously changing the inner RootTask route. It is not a permanent
requirement for every future canonical callable entry.

The stronger reusable-session contract remains separately supported:

~~~text
PREPARED_TOP_LEVEL_MULTI_CALLER_SAFETY=PRESERVED
CALL_VS_CLOSE_EXCLUSION=PRESERVED
STRONGER_SESSION_CONTRACT_MAY_HAVE_ITS_OWN_GATE=YES
STRONGER_SESSION_GATE_DEFINES_CANONICAL_MINIMUM_VALUE_EXECUTE_FLOOR=NO
~~~

## GITHUB021 invariant/delta consistency

The original owner invariant was:

~~~text
MINIMAL_ONE_CALLER_ONE_PROCESS_ONE_ROOTACTOR_SYNCHRONOUS_EXECUTION
  MUST_NOT_REQUIRE_A_SECOND_GUEST_EXECUTION_THREAD
  MERELY_FOR_UNUSED_CAPABILITY
~~~

The amendment applies the same principle to inner execution machinery:

~~~text
UNUSED_STRONGER_CAPABILITY
  MUST_NOT_REQUIRE_EAGER_PHYSICAL_MACHINERY
  ON_THE_WEAKER_ORDINARY_CALLABLE_PATH
~~~

This is consistent with PLAT040 conditional materialization and D189 ordinary
callback semantics.

~~~text
PLAT046_ORIGINAL_OWNER_INVARIANT=PRESERVED
PLAT040_CONSISTENCY=PASS
D189_CONSISTENCY=PASS
PERF025_H1_CONSISTENCY=PASS

DECISION_INVARIANT_CONSISTENCY=PASS
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
BUG008_REOPEN_REQUIRED=NO
~~~

## Implementation release

The amendment releases PERF033/#832 to implement the canonical minimum callable
boundary and continue until every Protos-specific fixed layer is removed or
justified.

It does not authorize unrelated changes to child-Actor scheduling, distributed
Process/Node/Cluster semantics, deep-recursion representation, benchmark
workloads or private/internal Graal APIs.

~~~text
PLAT046_STATUS=RATIFIED_WITH_OWNER_AMENDMENT
IMPLEMENTATION_AUTHORIZED=YES
IMPLEMENTATION_OWNER=PERF033/#832
~~~

## AI-assistance disclosure

This ratification evidence was materially prepared with AI assistance from
ChatGPT using the original PLAT046 record, current Protos source, retained
PERF025/PERF032 evidence, current normative D189 callback semantics, and explicit
project-owner approval.
