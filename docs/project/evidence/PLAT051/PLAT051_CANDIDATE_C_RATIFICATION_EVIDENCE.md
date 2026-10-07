# PLAT051 — Candidate C ratification evidence

Evidence date: **2026-10-07**

Decision Issue: `guillermomolina/protos#813`

Parent workstream: `LIB014 / guillermomolina/protos#431`

Governing semantic decision: `D187 / guillermomolina/protos#809`

Maintained decision record:

`docs/project/decisions/platform/PLAT051_STANDARD_LIBRARY_SEMANTIC_VALUE_TRANSFER_REMATERIALIZATION.md`

## Explicit project-owner selection

The project owner explicitly accepted the completed recommendation and its clarified pay-as-you-grow condition:

~~~text
ok pues acepto lo recomendado.
~~~

The selected candidate is exactly:

~~~text
Candidate C
=
privileged constrained Standard Library semantic-value transfer/rematerialization
+
inert portable semantic payload
+
destination-local implementation reconstruction
~~~

Approval includes the additional rule stated immediately before selection: PLAT051 does **not** grant portability merely because a value belongs to the Standard Library. A family must already have explicit semantic portability authority and must be explicitly authorized by trusted bootstrap.

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
SELECTED_CANDIDATE=C
~~~

## Product and project authority

Research completed against:

~~~text
PROTOS_RESEARCH_REVISION=5c1fd0dc1b0aa0bb3a7719ba6c0e3f5d0b759407
~~~

Immediately before ratification publication, product authority was revalidated at:

~~~text
PROTOS_REVISION=8bd5c37c7cca13599df39d18de2eadf01e4a86eb
PROJECT_DOCS_BASE_REVISION=750cfa4da83a9704ce69f54b9af2de02746e5417
~~~

The sole intervening Protos commit was:

~~~text
8bd5c37c7cca13599df39d18de2eadf01e4a86eb
TOOL012: bound toml-official progress groups to at most 100 Cases
~~~

Its parent is the research revision. TOOL012 changes Test Tool grouping only; it does not change the PLAT051 dependency surfaces: Actor snapshot transfer, isolated-parallel transfer, Standard Library bootstrap/module identity, Regex Pattern/Match representation, or isolation policy.

~~~text
MOVING_HEAD_REVIEW=PASS
PLAT051_REINVESTIGATION_REQUIRED=NO
~~~

## Trigger evidence retained

D187 already defines Regex Pattern and Match as portable semantic values across the relevant isolation boundary and forbids carrying a live host matcher.

The current runtime blocks that contract because Pattern/Match are ordinary Standard Library objects whose source-backed public operation slots reach `ProtosClosureValue`, while Actor transfer correctly rejects arbitrary Closures.

The trigger is therefore an existing generic runtime boundary, not a Regex matcher defect.

~~~text
CURRENT_REGEX_MATCHING_IMPLEMENTATION_VALID=YES
CURRENT_ACTOR_TRANSFERABILITY=BLOCKED_BY_EXISTING_RUNTIME
D187_REVISION_REQUIRED=NO
~~~

## Current transfer architecture findings

The investigation established:

- `ProtosActorValueTransfer` snapshots a detached whole graph before routing/admission;
- its transfer-wide identity memo preserves aliasing and cycles;
- ordinary Closures and Actor-local execution/resources remain rejected;
- `spawn`, `send` and `request` snapshot synchronously before delivery;
- request replies pass back through the same logical transfer boundary;
- existing runtime values/capabilities already demonstrate explicit transfer/rematerialization branches;
- `ProtosParallelRuntime.Transfer` has a separate graph copier and a distinct, narrow projection mechanism only for the explicitly executed P Closure;
- Standard Library bootstrap and exact `ProtosModuleKey` identity provide an existing non-forgeable authority seam without importing modules during transfer.

Existing examples such as Actor/Group/Process capabilities, standard streams, `ProtosEncodingValue`, and frozen standard IP representations demonstrate that transfer behavior is already selected by semantic/runtime category rather than by arbitrary guest serialization hooks.

## Selected trust boundary

Candidate C requires:

~~~text
STANDARD_LIBRARY_FAMILY
AND EXPLICIT_SEMANTIC_PORTABILITY_CONTRACT
AND BOOTSTRAP_AUTHORIZED_FAMILY_DESCRIPTOR
~~~

It rejects:

~~~text
GUEST_STRING_AS_FAMILY_AUTHORITY
GUEST_SLOT_AS_FAMILY_AUTHORITY
CLASS_NAME_AS_FAMILY_AUTHORITY
PUBLIC_SERIALIZATION_HOOK
GLOBAL_MUTABLE_STRING_TO_CONSTRUCTOR_REGISTRY
~~~

Trusted family recognition must be implementation-private and rooted in exact bootstrap authority.

## Payload and authority boundary

Only semantic inert data crosses through the privileged semantic-value node.

The source implementation graph is not payload authority and is not recursively transported when it contains destination-local callables or derived state.

Destination reconstruction establishes the local Standard Library implementation surface.

~~~text
SOURCE_CLOSURES_TRANSFERRED=NO
SOURCE_LEXICAL_CONTEXT_TRANSFERRED=NO
SOURCE_MODULE_CONTEXT_TRANSFERRED=NO
SOURCE_TASK_FUTURE_HANDLER_STATE_TRANSFERRED=NO
SOURCE_HOST_MATCHER_TRANSFERRED=NO
SOURCE_THREAD_STATE_TRANSFERRED=NO
~~~

Arbitrary user Closures remain non-transferable.

## Aliasing and identity result

The existing transfer-wide memo remains authoritative:

~~~text
SAME_SOURCE_IDENTITY_REPEATED_IN_ONE_GRAPH=ONE_DESTINATION_IDENTITY
DISTINCT_SOURCE_IDENTITIES=NOT_MERGED
SOURCE_IDENTITY_EQUALS_DESTINATION_IDENTITY=NO
IDENTITY_STABLE_ACROSS_SEPARATE_SENDS=NO_GUARANTEE
~~~

The rematerializable node must participate in allocation/population rather than bypass graph identity bookkeeping.

A family with cyclic semantic payload cannot opt in until it provides explicit safe two-phase construction semantics.

## Actor / P / Process scope

Actor transfer and P transfer must implement the same logical family/payload/rematerialization contract through their existing distinct graph-transfer machinery.

The narrow P computation-Closure projection remains separate and does not authorize arbitrary Closure transfer.

Future Process-to-Process transport should preserve the same logical contract, but PLAT051 deliberately does not define a wire format or cross-version negotiation protocol.

~~~text
ACTOR_LOGICAL_CONTRACT=IN_SCOPE
P_LOGICAL_CONTRACT=IN_SCOPE
PROCESS_FUTURE_LOGICAL_CONTRACT=PRESERVED
PROCESS_WIRE_FORMAT=DEFERRED
CROSS_VERSION_WIRE_PROMISE=NO
~~~

## Initial Regex proof mapping

Pattern can be represented by:

~~~text
source
canonical flags
~~~

while destination-local Regex recompiles/reconstructs its private Pike representation and callable surface.

Pattern does not carry the Pike program, source Closures, source lexical/module context or matcher execution state.

Match can be represented by:

~~~text
captured texts including group 0
scalar half-open bounds
capture count
capture-name -> group-number relation
~~~

Match does not carry the whole subject merely for convenience, Pattern implementation, Pike program, matcher state or source Closures. Destination reconstruction does not rerun the regex.

This is sufficient to show the selected generic mechanism can satisfy the motivating D187 family without becoming Regex-specific.

## Pay-as-you-grow ratification gate

The owner explicitly challenged whether the proposal penalizes ordinary programs. The recommendation was retained only with a hard localized-cost gate.

Mandatory implementation evidence:

~~~text
ORDINARY_OBJECT_LAYOUT_NEW_REQUIRED_FIELD=NO
GLOBAL_MUTABLE_REGISTRY=NO
ORDINARY_TRANSFER_STDLIB_SCAN=NO
ORDINARY_TRANSFER_MODULE_IMPORT=NO
ORDINARY_TRANSFER_SOURCE_EXECUTION=NO
ORDINARY_NON_REMATERIALIZABLE_TRANSFER_REGRESSION=NO_MATERIAL_REGRESSION

AUTHORIZED_VALUE_COST=LOCAL_TO_VALUE_THAT_CROSSES
REPEATED_ALIAS_ONE_TRANSFER=ONE_RECONSTRUCTION
MATCH_RERUNS_REGEX=NO
MATCH_RETAINS_WHOLE_SUBJECT=NO
NATIVE_IMAGE_GATE=REQUIRED
CLOSURE_NEGATIVE_GATE=PRESERVED
CAPABILITY_NEGATIVE_OR_SPECIAL_RULES=PRESERVED
~~~

A cheap generic recognition branch during ordinary graph visitation is acceptable. A required new field/tag on every ordinary `ProtosObjectValue`, repeated registry/module lookup, module import/source execution, or another transversal fixed cost is not.

If implementation cannot keep cost localized, the correct action is to stop and reopen PLAT051.

~~~text
PAY_AS_YOU_GROW=RATIFICATION_CONDITION
TRANSVERSAL_COST_DISCOVERED=REOPEN_DECISION
~~~

## Candidate disposition evidence

- **A**: inert payload is sound and incorporated, but a central family switch would accumulate Standard Library knowledge in generic transfer code.
- **B**: viable, but runtime-native representation per family scales poorly and unnecessarily moves Standard Library implementation into the platform.
- **C**: selected; generic privileged mechanism with family-local extraction/reconstruction.
- **D**: useful local technique where frozen canonical implementation state is enough, but insufficient for Regex and not a complete general boundary.
- **E**: incompatible with ratified D187 unless the semantic decision is reopened.
- **F**: no materially distinct superior architecture found; a generic semantic shell is an implementation of C.

The research scoring placed C highest for correctness, Protos alignment, future resilience, scalability, pay-for-use, portability and authority clarity. Implementation risk remains real and is handled through the explicit evidence gates rather than assumed away.

## External evidence retained

The investigation used materially different precedents:

- HTML structured clone/serialization — privileged supported families, target-realm reconstruction, graph memory preserving aliases/cycles, callable rejection;
- Dart isolates — explicit sendable universe and warning against treating captured Closures as ordinary data;
- Erlang external term format — function transport as a counterexample showing the origin/module/free-variable metadata that PLAT051 intentionally excludes;
- Python pickle — negative precedent for import/resolution/arbitrary code execution during deserialization;
- GraalVM Truffle context/shared-state guidance — support for keeping context-owned implementation local unless state is genuinely context-independent.

The common useful pattern is a closed privileged value universe with inert state and destination-local reconstruction, not arbitrary guest hooks.

## Strongest retained counterargument

Candidate C can evolve into a hidden second serialization/type system if every library family adds descriptors/schemas casually.

The ratified guard is that PLAT051 supplies mechanism only; semantic portability must be independently established, bootstrap authorization must be explicit, values remain ordinary observably, and no public/global reflective registry exists.

If the family catalogue or metadata grows into a parallel universal type system, that is a concrete reopen trigger.

## Implementation release

Ratification releases:

~~~text
NEXT=PLAT051-A
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
IMPLEMENTATION_AUTHORIZED=YES
REGEX_OPT_IN=NO
~~~

PLAT051-A is the generic privileged descriptor/rematerialization infrastructure plus Actor/P integration and its correctness/pay-as-you-grow gates.

After A passes, the bounded next slice is:

~~~text
NEXT_AFTER_A=PLAT051-B
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
GOAL=Regex Pattern/Match family opt-in and portability proof
~~~

PLAT051-B remains blocked by A. Final LIB014 portability closure remains blocked until B passes the relevant Actor/P/Native evidence.

## Result

~~~text
PLAT051_STATUS=RATIFIED
SELECTED_CANDIDATE=C_PRIVILEGED_STANDARD_LIBRARY_REMATERIALIZATION_PROTOCOL
EXPLICIT_SEMANTIC_PORTABILITY_CONTRACT_REQUIRED=YES
BOOTSTRAP_AUTHORIZED_FAMILY_DESCRIPTOR_REQUIRED=YES
PUBLIC_USER_SERIALIZATION_PROTOCOL=NO
ARBITRARY_CLOSURE_TRANSFER=NO
REGEX_SPECIFIC_GENERIC_COPIER_BRANCH=NO
PAY_AS_YOU_GROW=MANDATORY
D187_DELTA=NONE
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
NEXT=PLAT051-A
~~~
