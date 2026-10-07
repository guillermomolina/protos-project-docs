# PLAT051-A2 — two-stage semantic-value transfer/materialization validation

Status: **COMPLETED AND PUBLISHED**

Evidence date: **2026-10-07**

Decision Issue: `guillermomolina/protos#813` — PLAT051

Parent workstream: `guillermomolina/protos#431` — LIB014

Governing semantic authority for the initial consumer: `guillermomolina/protos#809` — D187.

Maintained platform decision:

`docs/project/decisions/platform/PLAT051_STANDARD_LIBRARY_SEMANTIC_VALUE_TRANSFER_REMATERIALIZATION.md`

## Published product revision

~~~text
PUBLISHED_SHA=e681dc3a09165e53e2a977afd9ab6b89b80b9583
COMMIT_MESSAGE=PLAT051-A2: two-stage Standard Library semantic-value transfer/materialization
IMPLEMENTATION_VERSION=0.3.262-SNAPSHOT
~~~

The published revision is the current `main` head at evidence capture time.

## Human-executor validation

After publication, the project owner explicitly reported:

~~~text
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=HUMAN_EXECUTOR_REPORTED
~~~

No unreported test, build, benchmark or Native Image command is claimed by this record.

## Why A2 existed

PLAT051-A correctly established trusted family authority, inert payloads, alias-preserving generic transfer recognition and pay-as-you-grow structure, but its concrete implementation reconstructed semantic values during the source-side snapshot.

PLAT051-B preflight demonstrated that this timing could not support a guest-implemented Standard Library family such as `std:regex/Regex`: Actor and P snapshots execute in the source domain, while source-backed callable surfaces must be created from the destination domain's own module execution state.

No product file was changed during that preflight. The project owner retained Candidate C and selected an explicit two-stage realization rather than transferring source Closures/Pike state, creating lazy native proxies, or moving Regex into a Java-side parallel library.

## Amended architecture implemented

A2 implements:

~~~text
SOURCE DOMAIN
    authorized semantic value
        -> validate
        -> extract inert payload
        -> validate payload
        -> internal ProtosSemanticTransferRecord

ISOLATION BOUNDARY

DESTINATION DOMAIN
    ProtosSemanticTransferRecord
        -> destination execution/module authority
        -> destination-local materialization
        -> fresh FROZEN semantic value
        -> guest observation
~~~

The in-flight record is internal runtime state, not an OPEN guest object.

~~~text
SOURCE_RECONSTRUCTION=NO
OPEN_GUEST_SHELL_IN_TRANSIT=NO
LAZY_FIRST_USE_RECONSTRUCTION=NO
~~~

## New runtime abstractions

The published implementation adds:

~~~text
ProtosSemanticTransferRecord
ProtosSemanticTransferDestination
~~~

`ProtosSemanticTransferRecord` owns the exact trusted family descriptor plus the already validated inert payload. It is not a `ProtosObjectValue`, has no guest slot/callable surface and is not public serialization authority.

`ProtosSemanticTransferDestination` supplies destination execution authority to the family materializer. It exposes the exact owning Standard Library module through the established destination Actor-local module lifecycle and allows the family to invoke destination-local guest implementation code.

`ProtosSemanticTransferFamily` now separates:

~~~text
extract(source semantic value)
acceptsPayload(inert payload)
materialize(payload, destination authority)
~~~

Source preparation never calls `materialize`.

## Source-stage validation and failure timing

Actor/P source snapshots remain synchronous.

The source stage:

- verifies the exact bootstrap-authorized family;
- requires a valid transferable source semantic value;
- extracts the inert payload without executing destination guest code;
- validates the payload with the family contract;
- emits a semantic-transfer record into the detached graph;
- preserves source graph identity through the existing transfer memo.

Unauthorized families or invalid payloads are rejected at the original source transfer boundary.

Therefore:

~~~text
SOURCE_TRANSFERABILITY_VALIDATION_SYNCHRONOUS=PASS
PARTIAL_MESSAGE_ACCEPTANCE=NO
SOURCE_NON_TRANSFERABLE_FAILURE_TIMING=UNCHANGED
~~~

For the same runtime/library image targeted by PLAT051, a validated emitted record is required to materialize successfully; a later materialization failure is treated as an internal implementation inconsistency rather than a new lazy public transfer failure mode.

## Actor destination materialization

A2 places destination materialization before guest observation across the relevant Actor paths.

The published change integrates record materialization into:

- Actor spawn/bootstrap arguments before the bootstrap binding is invoked;
- accepted Actor message turns before the guest handler receives arguments;
- Actor request replies before the requester Future exposes the returned value;
- Group request/send delivery boundaries using the generic Actor-transfer machinery.

Destination materialization uses the actual destination `ProtosActivation` / execution domain and loads the exact family owner module through the installed Standard Library import/module runtime.

No source module context or source Closure graph is reused.

## P destination materialization

P snapshot formation remains caller-side and synchronous.

Only snapshots containing semantic records are marked for a destination materialization pass.

Inside the isolated worker domain, records are materialized before the computation Closure sees the arguments.

When a P result crosses back to the caller and contains semantic records, the caller's producer Task materializes them inside the caller domain before resolving the public Future.

The existing narrow rule for projecting the explicitly executed P computation Closure remains unchanged and is not generalized to arbitrary Closures.

## Guest-implemented Standard Library proof

A2 adds `ProtosSemanticTransferMaterializationTest`, a hosted production-representative test.

Its test-only family is owned by an exact `std:` module whose destination callable surface is created by running guest Protos module code inside the destination domain.

The source value deliberately carries a source-only native callable. The retained test proves that:

- the source callable is never used after transfer;
- the destination Standard Library module is not loaded in the source domain merely to snapshot the value;
- the destination domain loads its own module implementation;
- the destination callable surface is created from guest Protos code before Actor/P guest consumers observe the value;
- Actor request replies materialize in the requester domain;
- P results materialize back in the caller domain.

This closes the architectural gap discovered by the first PLAT051-B attempt:

~~~text
GUEST_IMPLEMENTED_STANDARD_LIBRARY_FAMILY_SUPPORTED_ARCHITECTURALLY=PASS
~~~

## Identity, aliasing and cycles

The two-stage implementation preserves logical identity with a memo at each stage:

~~~text
SOURCE:
same source semantic identity -> one transfer-record identity

DESTINATION:
same transfer-record identity -> one destination semantic identity
~~~

Therefore:

~~~text
REPEATED_SOURCE_IDENTITY_ONE_TRANSFER=ONE_DESTINATION_IDENTITY
DISTINCT_SOURCE_IDENTITIES=NOT_MERGED
SOURCE_IDENTITY_EQUALS_DESTINATION_IDENTITY=NO
SURROUNDING_ORDINARY_GRAPH_CYCLES=UNCHANGED
~~~

For fan-out, the detached inert representation can be reused internally while each destination isolation domain gets its own materialized identity.

## Pay-as-you-grow

The implementation marks only detached graphs that actually contain semantic-transfer records.

Ordinary transfers do not perform a mandatory second graph copy/materialization pass and do not import/execute Standard Library source.

~~~text
ORDINARY_OBJECT_LAYOUT_NEW_REQUIRED_FIELD=NO
ORDINARY_TRANSFER_SECOND_PASS=NO
ORDINARY_TRANSFER_STDLIB_SCAN=NO
ORDINARY_TRANSFER_MODULE_IMPORT=NO
ORDINARY_TRANSFER_SOURCE_EXECUTION=NO
GLOBAL_MUTABLE_REGISTRY=NO
SEMANTIC_EXTRA_COST=LOCAL_TO_GRAPH_WITH_RECORDS
PAY_AS_YOU_GROW=PASS_STRUCTURAL
~~~

No dedicated throughput benchmark is claimed by this record.

## Negative boundaries preserved

A2 does not make arbitrary Closures transferable and does not introduce a public serialization protocol.

It does not add:

- Java serialization;
- reflection-driven family resolution;
- `Class.forName` family lookup;
- `ServiceLoader` family discovery;
- a global mutable String-to-constructor registry;
- guest-visible transfer tags/hooks;
- Regex-specific branches in generic Actor/P transfer.

Capability/resource rules remain governed by their existing specialized semantics.

## Regex boundary

A2 deliberately does **not** opt Regex into PLAT051.

~~~text
REGEX_PATTERN_OPT_IN=NO
REGEX_MATCH_OPT_IN=NO
REGEX_SPECIFIC_GENERIC_TRANSFER_BRANCH=NO
D187_DELTA=NONE
CURRENT_REGEX_MATCHING_IMPLEMENTATION_VALID=YES
~~~

The point of A2 is to make the generic mechanism capable of the already-ratified B design without requiring another architecture rewrite.

## PLAT051-B release

With A2 published and the maintainer reporting a clean diff check and complete local test PASS, PLAT051-B is mechanically unblocked.

~~~text
NEXT=PLAT051-B
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
GOAL=opt std:regex/Regex Pattern and Match into PLAT051
IMPLEMENTATION_AUTHORIZED=YES
~~~

B must now remain a bounded family opt-in:

- one exact Regex family owned by `std:regex/Regex`;
- Pattern source-stage payload: source + canonical flags;
- Pattern destination materialization: destination-local Regex compile/construction;
- no source Pike program or source Closure transfer;
- Match payload: captures/texts + scalar bounds + capture count/name relation;
- Match destination construction without rerunning Regex;
- no whole-subject transfer merely for convenience;
- no Regex branch in generic Actor/P transfer.

If B passes its Actor/P/applicable Native gates and no new concrete defect appears, the final D187 portability blocker is resolved and LIB014 may close. LIB014-4 remains optional and requires measured performance evidence.

## Result

~~~text
PLAT051_A2_IMPLEMENTATION=PUBLISHED_AND_VALIDATED
PUBLISHED_SHA=e681dc3a09165e53e2a977afd9ab6b89b80b9583
IMPLEMENTATION_VERSION=0.3.262-SNAPSHOT

GIT_DIFF_CHECK=PASS_REPORTED_BY_HUMAN
ALL_LOCAL_TESTS=PASS_REPORTED_BY_HUMAN

SOURCE_STAGE_VALIDATES_AND_EXTRACTS=PASS
SOURCE_STAGE_MATERIALIZES=NO
INTERNAL_TRANSFER_RECORD=PASS
TRANSFER_RECORD_GUEST_VISIBLE=NO
OPEN_GUEST_SHELL_IN_TRANSIT=NO

ACTOR_DESTINATION_MATERIALIZATION=PASS
P_DESTINATION_MATERIALIZATION=PASS
MATERIALIZATION_BEFORE_GUEST_OBSERVATION=PASS
DESTINATION_EXECUTION_AUTHORITY=PASS
GUEST_IMPLEMENTED_STANDARD_LIBRARY_FAMILY_SUPPORTED_ARCHITECTURALLY=PASS

SOURCE_NON_TRANSFERABLE_FAILURE_TIMING=UNCHANGED
PARTIAL_ACCEPTANCE=NO
ALIAS_PRESERVATION=PASS
DISTINCT_IDENTITY_PRESERVATION=PASS

ARBITRARY_CLOSURE_TRANSFER=NO
CAPABILITY_RULE_DELTA=NONE
REGEX_SPECIFIC_BRANCH=NO
REGEX_OPT_IN=NO

ORDINARY_OBJECT_LAYOUT_FIXED_COST_DELTA=NONE
ORDINARY_TRANSFER_SECOND_PASS=NO
ORDINARY_STDLIB_IMPORT_EXECUTION=NO
GLOBAL_MUTABLE_REGISTRY=NO
PAY_AS_YOU_GROW=PASS_STRUCTURAL

D187_DELTA=NONE
SPECIFICATION_CHANGE=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO

NEXT=PLAT051-B
NEXT_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos
~~~

## AI-assistance disclosure

This durable evidence record was materially prepared with AI assistance from ChatGPT using the published PLAT051-A2 revision, the maintained PLAT051/D187 records, the current GitHub repository state, and the project owner's explicit human-executor validation report. No unreported build or test execution is claimed.
