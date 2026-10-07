# PLAT051 — Standard Library semantic-value transfer and rematerialization boundary

Status: **RATIFIED WITH IMPLEMENTATION AMENDMENT — IMPLEMENTED**

Selected architecture: **Candidate C — privileged constrained Standard Library semantic-value transfer/rematerialization protocol using inert portable payloads**.

Approval: explicit project-owner approval on 2026-10-07:

~~~text
ok pues acepto lo recomendado.
~~~

Decision Issue: `guillermomolina/protos#813`

Parent workstream: `LIB014 / guillermomolina/protos#431`

Governing public semantic authority: `D187 / guillermomolina/protos#809` for the initial Regex Pattern/Match portability requirement.

Research product revision:

~~~text
PROTOS_RESEARCH_REVISION=5c1fd0dc1b0aa0bb3a7719ba6c0e3f5d0b759407
PROTOS_RATIFICATION_REVALIDATION_REVISION=8bd5c37c7cca13599df39d18de2eadf01e4a86eb
~~~

The intervening product commit is `TOOL012: bound toml-official progress groups to at most 100 Cases`; it does not change the Actor transfer, P transfer, Standard Library bootstrap, module-key, Regex Pattern/Match, or isolation surfaces on which this decision depends.

Nature: durable non-normative platform/runtime architecture. PLAT051 does not create or revise observable Protos semantics. Portability authority remains with each owning semantic contract.

## Decision

Selected Standard Library semantic-value families may cross isolation boundaries through an **internal privileged rematerialization mechanism** that transports only an inert semantic payload and reconstructs destination-local implementation state and callable surface.

The mechanism is not a public serialization protocol and is not available merely because an object originates in the Standard Library.

Eligibility requires all three conditions:

~~~text
STANDARD_LIBRARY_FAMILY
AND EXPLICIT_SEMANTIC_PORTABILITY_CONTRACT
AND BOOTSTRAP_AUTHORIZED_FAMILY_DESCRIPTOR
~~~

Therefore:

~~~text
STANDARD_LIBRARY_MEMBERSHIP_ALONE_IMPLIES_PORTABILITY=NO
PUBLIC_USER_SERIALIZATION_PROTOCOL=NO
ARBITRARY_CLOSURE_TRANSFER=NO
REGEX_SPECIAL_CASE_IN_GENERIC_COPIER=NO
GLOBAL_MUTABLE_RECONSTRUCTION_REGISTRY=NO
~~~

A future Standard Library family may opt in only after its own semantic authority explicitly defines the value as portable and the runtime bootstrap explicitly authorizes the exact family.

## Mechanism boundary

Conceptually the privileged mechanism consists of:

~~~text
trusted family descriptor
+
source-local semantic extractor
+
inert portable payload
+
destination-local reconstructor
~~~

The descriptor is an implementation-only authority created by trusted bootstrap and scoped to an exact canonical Standard Library family identity, naturally aligned with the existing `ProtosModuleKey` / Standard Library bootstrap authority.

A guest-provided string, slot, object shape, class name, reflective name, serialized tag, or other forgeable public data is not sufficient authority to enter this path.

The source implementation graph is not the payload. In particular, transfer does not traverse destination-replaceable method Closures or private compiled artifacts merely because they are reachable from the source object.

Destination reconstruction uses destination-local Standard Library implementation authority.

## Fixed isolation invariants

PLAT051 preserves the existing Actor/Process isolation rules:

- ordinary Actor messaging remains pass-by-value;
- ordinary `ProtosClosureValue` remains non-transferable;
- creator lexical environment, `this`, module context, return home, handlers, Futures, Tasks, mutable execution graphs, host matcher/thread state and authority do not cross implicitly;
- existing capability transfer rules remain unchanged;
- source-local callable/prototype/implementation graphs are not smuggled through the semantic payload;
- malformed, forged or incompatible privileged transfer forms fail closed;
- there is no fallback from failed rematerialization to copying source Closures or source implementation state.

PLAT051 does not authorize a public `__serialize__` / `__deserialize__`, `readResolve`, pickle-style hook, arbitrary source execution during reconstruction, reflection-driven class loading, or a string-to-constructor global registry.

## Identity, aliasing and cycles

The existing transfer-wide identity memo remains authoritative.

For one transfer graph:

~~~text
same source rematerializable identity referenced repeatedly
    -> one destination identity

distinct source identities
    -> distinct destination identities

source identity
    != destination identity
~~~

No destination-identity stability is promised across separate sends.

Rematerializable nodes must participate in the same allocation/population discipline required to preserve graph aliasing and cycles. A future family whose **semantic payload itself** requires cyclic construction may not opt in until its reconstruction contract provides an explicit safe two-phase form.

The source delegation parent and source callable graph do not cross merely to preserve object shape. Destination reconstruction establishes the destination-local implementation surface.

## Actor boundary

`ProtosActorValueTransfer` already forms a detached logical graph before routing/admission and already owns transfer-wide alias/cycle preservation.

PLAT051 adds one generic privileged semantic-value recognition/rematerialization path to that architecture. The generic transfer layer must not contain Regex-specific branches.

For group/fan-out delivery, a logical inert transfer representation may be shared internally, but each receiving isolation domain obtains its own destination-local reconstructed value identity.

## P / isolated parallel boundary

`ProtosParallelRuntime.Transfer` has a distinct graph-copy path and must implement the same family authorization, inert payload and destination-local reconstruction contract.

Its existing special handling for the explicitly executed P computation Closure remains distinct. PLAT051 does not generalize that exception to arbitrary semantic values or Actor messages.

## Process / future transport boundary

Actor-to-Actor and Process-to-Process messaging share the same logical pass-by-value model, so the family/payload/rematerialization contract is intended to remain valid for a future Process transport.

PLAT051 deliberately does **not** select a wire format, cross-version protocol, serializer, retry policy, distributed family negotiation scheme, or network encoding.

A future transport may encode the same logical semantic-transfer node using its own implementation mechanism.

## Version and failure model

Initial implementation targets the same runtime/library image.

~~~text
SAME_RUNTIME_LIBRARY_IMAGE=SUPPORTED_TARGET
CROSS_VERSION_COMPATIBILITY_PROMISE=NO
NETWORK_WIRE_COMPATIBILITY_PROMISE=NO
~~~

Failures are fail closed:

- ordinary unsupported values keep the existing non-transferable failure;
- forged/untrusted family markers do not gain privileged reconstruction authority;
- malformed payloads are rejected;
- missing or incompatible destination family support is rejected;
- private-state reconstruction failure is reported as transfer/reconstruction failure;
- failed Pattern reconstruction never falls back to transferring a source Pike program or Closure graph.

PLAT051 does not require a new public Error hierarchy.

## Pay-as-you-grow requirement

Candidate C is ratified only with a strict localized-cost requirement.

Programs that never transfer an authorized rematerializable semantic value must not pre-pay a new object-layout, module-resolution, registry, import, source-execution or retained-metadata tax.

Mandatory implementation gates:

~~~text
ORDINARY_OBJECT_LAYOUT_NEW_REQUIRED_FIELD=NO
GLOBAL_MUTABLE_REGISTRY=NO
ORDINARY_TRANSFER_STDLIB_SCAN=NO
ORDINARY_TRANSFER_MODULE_IMPORT=NO
ORDINARY_TRANSFER_SOURCE_EXECUTION=NO

REPEATED_ALIAS_ONE_TRANSFER=ONE_RECONSTRUCTION
DISTINCT_SOURCE_IDENTITIES=NOT_MERGED

CLOSURE_NEGATIVE_TRANSFER_GATE=PRESERVED
CAPABILITY_TRANSFER_RULES=PRESERVED
NATIVE_IMAGE_BEHAVIOR=REQUIRED
~~~

The ordinary snapshot path may pay only a small recognition branch while visiting a value. The materialization cost belongs to the value that actually crosses the boundary.

If implementation evidence demonstrates that correctness requires a material per-object field/tag on every `ProtosObjectValue`, repeated Standard Library scans/imports, a required global mutable registry, or another non-local fixed cost, the implementation must stop and PLAT051 must be reopened rather than silently weakening this gate.

## Implementation amendment — explicit two-stage transfer

PLAT051-A initially implemented both semantic extraction and reconstruction during the source-side snapshot. The first PLAT051-B implementation preflight demonstrated that this was too early for guest-implemented Standard Library families: Actor and P snapshots run in the source domain, while source-backed callable surfaces such as Regex Pattern/Match must be constructed by executing the destination domain's own Standard Library module.

The project owner therefore retained Candidate C and amended its concrete runtime contract to make the source and destination phases explicit:

~~~text
SELECTED_ARCHITECTURE=
    CANDIDATE_C
    + EXPLICIT_TWO_STAGE_TRANSFER

SOURCE_STAGE=
    VALIDATE
    + EXTRACT_INERT_PAYLOAD
    + CREATE_NON_GUEST_TRANSFER_RECORD

DESTINATION_STAGE=
    MATERIALIZE_BEFORE_GUEST_OBSERVATION
    + DESTINATION_LOCAL_STANDARD_LIBRARY_IMPLEMENTATION

SOURCE_RECONSTRUCTION=NO
OPEN_GUEST_SHELL_IN_TRANSIT=NO
LAZY_FIRST_USE_RECONSTRUCTION=NO
~~~

The source stage remains synchronous and atomic with the existing Actor/P snapshot rules. It must reject an unauthorized family, non-transferable source or invalid payload before message acceptance / P submission. The detached graph carries only an implementation-internal semantic-transfer record plus inert payload.

The destination stage runs inside the destination execution domain before guest code can observe the value. It receives destination execution/module authority, loads the exact owning Standard Library module through the established module lifecycle, and materializes a fresh FROZEN semantic value whose callable/private implementation surface belongs to that destination.

The transfer representation in flight is not an OPEN guest value and has no guest slots/callability/public API. Aliasing is preserved across the two stages by source and destination identity memos:

~~~text
source identity -> one semantic-transfer record
one semantic-transfer record -> one destination identity per destination graph
distinct source identities -> distinct records -> distinct destination identities
~~~

For Group/fan-out, one inert logical record may be reused internally, but every receiving isolation domain materializes its own destination-local value identity.

Failure timing remains unchanged for source transferability. For the initially supported same runtime/library image, a record that passed source validation is required to materialize successfully in the compatible destination; a later materialization failure is an internal implementation inconsistency, not a new lazy user-visible transfer failure mode.

Pay-as-you-grow is preserved:

~~~text
ORDINARY_TRANSFER_SECOND_PASS=NO
ORDINARY_TRANSFER_STDLIB_SCAN=NO
ORDINARY_TRANSFER_MODULE_IMPORT=NO
ORDINARY_TRANSFER_SOURCE_EXECUTION=NO
SEMANTIC_DESTINATION_MATERIALIZATION_COST=ONLY_WHEN_RECORDS_EXIST
~~~

This amendment does not revise D187 or any observable Protos semantics. It rejects source-side destination execution, guest-visible OPEN shells, first-use lazy reconstruction, Java-side parallel Standard Library implementations and Regex-specific branches in generic transfer machinery.

### PLAT051-A2 implementation evidence

The amended contract was implemented and published as:

~~~text
PUBLISHED_SHA=e681dc3a09165e53e2a977afd9ab6b89b80b9583
COMMIT_MESSAGE=PLAT051-A2: two-stage Standard Library semantic-value transfer/materialization
IMPLEMENTATION_VERSION=0.3.262-SNAPSHOT
~~~

PLAT051-A2 introduces internal semantic-transfer records and destination materialization authority, integrates destination-stage materialization into Actor send/request/Group/spawn/reply and P worker/caller boundaries, and retains an O(1) ordinary-path record check rather than a mandatory second pass.

A hosted production-representative test proves that a test Standard Library family can create its destination callable surface by executing guest Protos module code in the destination domain. This closes the architectural gap that blocked the first PLAT051-B attempt.

## Initial consumer: Regex Pattern

D187 already requires Pattern to behave as portable semantic data.

The initial Regex mapping is:

~~~text
Pattern portable payload:
    source
    canonical flags

Pattern does NOT transfer:
    Pike program
    source Closures
    source lexical/module context
    matcher execution state
~~~

Destination-local Regex authority recompiles/reconstructs the Pattern from its semantic payload and attaches only destination-local implementation state/callable surface.

Capture count/names are observable but deterministic from compilation and need not be transmitted initially unless implementation evidence demonstrates a concrete requirement.

A Pattern may therefore pay destination recompilation cost. Invisible destination-local caching remains an optional optimization only; it is not correctness authority and must not change observable identity semantics.

## Initial consumer: Regex Match

The initial Match mapping is:

~~~text
Match portable payload:
    captured texts including group 0
    scalar half-open bounds
    capture count
    capture-name -> group-number relation

Match does NOT transfer:
    whole subject merely for convenience
    Pattern implementation graph
    Pike program
    matcher state
    source Closures
~~~

Destination reconstruction must not rerun the regular expression. The Match payload is sufficient to implement the public result operations locally.

This avoids retaining or transferring a potentially large subject string when only the immutable result semantics are required.

## Final implementation — PLAT051-B

The first production consumer of the amended mechanism is complete and published:

~~~text
PUBLISHED_SHA=b3d85c1deee91455c2777d0025a19ec3970530ac
COMMIT_MESSAGE=PLAT051-B: opt std:regex/Regex Pattern and Match into semantic transfer
IMPLEMENTATION_VERSION=0.3.264-SNAPSHOT
~~~

One exact `ProtosRegexSemanticTransferFamily`, owned by `std:regex/Regex`, serves both Pattern and Match.

Pattern transports only:

~~~text
source
canonical flags
~~~

and recompiles with the destination domain's own Regex module.

Match transports only the already-computed semantic result:

~~~text
capture count
per-group participation/text/scalar bounds
capture-name -> group-number relation
~~~

and is rebuilt without rerunning Regex. The whole subject, Pike program, matcher state and source Closures do not cross.

The exact Regex module mints its family values through a private bootstrap facility removed from public module surface. During destination-module initialization, it installs a guest factory in that Actor-local module record; materialization therefore creates the callable surface from destination-local Protos code rather than a Java-side parallel Regex implementation.

Retained production tests cover Actor spawn/request/reply, isolated-P inputs/results, alias preservation, distinct equal-content identities, payload validation, forgery rejection, public-surface preservation and ordinary transfers that do not load Regex.

The project owner reports:

~~~text
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=HUMAN_EXECUTOR_REPORTED
~~~

Durable implementation evidence is retained at:

`docs/project/evidence/PLAT051/PLAT051_B_IMPLEMENTATION_VALIDATION.md`

With B complete, the motivating D187 Actor/P portability requirement is satisfied. No further mandatory PLAT051 implementation slice remains.

~~~text
PLAT051_A=COMPLETE
PLAT051_A2=COMPLETE
PLAT051_B=COMPLETE
D187_ACTOR_P_PORTABILITY=COMPLETE
NEXT_REQUIRED_PLAT051_SLICE=NONE
~~~

## Native Image and alternate runtimes

The selected architecture is runtime-neutral at the semantic boundary.

It requires no Java serialization, reflective class loading, JNI object identity, dynamic Java classes or JVM-only public contract.

A JVM implementation may use internal Java classes to realize the descriptor/node mechanics, but the architecture is defined as trusted family authority + inert semantic payload + destination-local reconstruction and therefore remains meaningful for Native Image and alternate runtimes.

## Candidate disposition

### Candidate A — central inert-payload reconstruction

Technically viable, and its inert-payload principle is retained inside Candidate C.

Rejected as the complete architecture because a central copier switch would accumulate Standard Library family knowledge and tend toward family-specific runtime special cases.

### Candidate B — dedicated runtime semantic-value representation per family

Technically viable.

Rejected as the default because it moves Standard Library family semantics/representation into runtime-native classes, weakens Protos-owned library implementation freedom and scales poorly as more families appear.

### Candidate C — constrained privileged Standard Library transfer/rematerialization protocol

**Selected.**

It keeps family-specific extraction/reconstruction with the owning library authority while the runtime owns only the privileged generic isolation-boundary mechanism.

### Candidate D — shared immutable implementation object with destination-local callable surface

Useful as a family-specific implementation technique where the representation already fits it, and consistent with existing frozen standard objects.

Not sufficient as the general answer because Regex source-backed methods/private compiled state require an explicit destination reconstruction seam.

### Candidate E — remain non-transferable

Rejected for PLAT051 because it conflicts with the already-ratified D187 Pattern/Match portability contract. It would require reopening the owning semantic decision.

### Candidate F — another architecture discovered by research

No materially distinct better architecture was found. A generic semantic-value shell is an implementation form of Candidate C, not a separate semantic architecture.

## Strongest argument against Candidate C

Candidate C can drift into a second privileged serialization/type universe: hidden type metadata, special constructors, family schemas and an ever-growing privileged registry.

That risk is accepted only under these boundaries:

- values remain ordinary Protos values observably;
- descriptor metadata exists only for the isolation boundary and only for opted-in families;
- Standard Library implementation remains module-local;
- there is no public reflective type/serialization protocol;
- there is no global mutable registry required for correctness;
- a family cannot opt in merely because it is Standard Library;
- if many families begin adding ad-hoc transfer schemas without independently established portability semantics, the architecture must be reopened instead of normalizing that growth.

## External precedent retained

The investigation compared multiple distinct runtime families:

- HTML structured serialization/clone: closed privileged serializable families, target-realm reconstruction, memory-map alias/cycle preservation and callable rejection;
- Dart isolate messaging: explicit sendable-value universe and caution around Closure capture;
- Erlang external term format: transferable function forms demonstrate the module/free-variable/origin metadata that PLAT051 intentionally refuses to carry;
- Python pickle: negative precedent for deserialization that can import/resolve/execute arbitrary code;
- GraalVM Truffle context-local/shared-state guidance: shared implementation state must be context-independent, supporting destination-local reconstruction rather than accidental Context ownership leakage.

These precedents support a closed privileged value family with inert data and target-local reconstruction, while arguing against guest-extensible deserialization hooks and transferable Closures.

## Implementation release

Ratification releases two bounded mechanical slices in order:

~~~text
SLICE=PLAT051-A
TYPE=IMPLEMENTATION
STATUS=COMPLETE
REPOSITORY=guillermomolina/protos
GOAL=generic privileged semantic-value transfer/rematerialization infrastructure
REGEX_OPT_IN=NO

SLICE=PLAT051-A2
TYPE=IMPLEMENTATION
STATUS=COMPLETE
REPOSITORY=guillermomolina/protos
GOAL=split source validation/extraction from destination-domain materialization
REGEX_OPT_IN=NO

SLICE=PLAT051-B
TYPE=IMPLEMENTATION
STATUS=COMPLETE
REPOSITORY=guillermomolina/protos
GOAL=opt std:regex/Regex Pattern and Match into the amended mechanism
PUBLISHED_SHA=b3d85c1deee91455c2777d0025a19ec3970530ac
~~~

PLAT051-A established the generic trusted descriptor/value/payload mechanism. PLAT051-A2 corrected the implementation timing so source snapshots emit validated inert records and destination domains materialize them before guest observation, including support for guest-implemented Standard Library callable surfaces.

PLAT051-B completes the first production family opt-in: destination-local Regex reconstruction, Pattern/Match payloads and Actor/P portability proof. Its passing publication resolves the final required LIB014 portability blocker.

No Process wire-format slice is authorized by this decision.

## Ratified result

~~~text
PLAT051_STATUS=RATIFIED_WITH_IMPLEMENTATION_AMENDMENT
SELECTED_CANDIDATE=C_PRIVILEGED_STANDARD_LIBRARY_REMATERIALIZATION_PROTOCOL
TRANSFER_STAGING=EXPLICIT_SOURCE_RECORD_PLUS_DESTINATION_MATERIALIZATION
PLAT051_A=COMPLETE
PLAT051_A2=COMPLETE
PLAT051_B=COMPLETE
PORTABLE_PAYLOAD=INERT_SEMANTIC_DATA_ONLY
STANDARD_LIBRARY_MEMBERSHIP_ALONE_IMPLIES_PORTABILITY=NO
EXPLICIT_SEMANTIC_PORTABILITY_CONTRACT_REQUIRED=YES
BOOTSTRAP_AUTHORIZED_FAMILY_DESCRIPTOR_REQUIRED=YES
PUBLIC_USER_SERIALIZATION_PROTOCOL=NO
ARBITRARY_CLOSURE_TRANSFER=NO
REGEX_SPECIAL_CASE_IN_GENERIC_COPIER=NO
GLOBAL_MUTABLE_RECONSTRUCTION_REGISTRY=NO
PAY_AS_YOU_GROW=MANDATORY_GATE
D187_DELTA=NONE
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
NEXT_REQUIRED_PLAT051_SLICE=NONE
LIB014_FINAL_PORTABILITY_BLOCKER=RESOLVED
~~~
