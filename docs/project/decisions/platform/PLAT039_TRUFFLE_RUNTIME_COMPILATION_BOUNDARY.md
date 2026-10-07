# PLAT039 — Truffle runtime-compilation boundary for Native Image

Status: RATIFIED

Selected architecture: **Candidate C — evidence-gated PE-visible guest kernel with narrow host gateways**.

Approval: explicit project-owner approval on 2026-09-25:

~~~text
Approve Candidate C
~~~

Decision Issue: guillermomolina/protos#716

Implementation consumer: I069 / guillermomolina/protos#711

Performance consumer: PERF010-A / guillermomolina/protos#691

Product baseline:

~~~text
PROTOS_REVISION=d4ac1c7be600c00c785dc9742dc7aeaeab17eaec
PROTOS_VERSION=0.3.87-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.3.4.1
GRAALVM_SOURCE_TAG=vm-25.3.4.1
GRAALVM_SOURCE_REVISION=7b025988a922a73286d1326e1eddc1ca39d3f569
~~~

The product baseline remained identical to current `guillermomolina/protos`
`main` when PLAT039 was ratified.

Primary prior authority:

- PLAT038 — Native Image is a first-class artifact, Native Image runtime
  compilation is required, JVM development remains supported, and native
  conformance is required;
- I068 / PLAT036 — eligible lexical execution state is frame-backed while
  preserving first-class Protos execution-context semantics;
- the normative execution, callable, module, object, value/collection, I/O,
  filesystem, process, Future/Task and Actor specifications.

Nature: durable non-normative runtime architecture decision. Observable Protos
semantics remain owned by the normative specification.

## Decision

Protos adopts an explicit three-way runtime-compilation classification:

~~~text
KEEP_PE
  guest work that must remain visible to Truffle Partial Evaluation/runtime
  compilation

BOUNDARY
  opaque host/cold/runtime leaf work whose internals do not need guest PE

SPLIT
  an existing operation that mixes guest-hot semantics with host/cold mechanics;
  separate those responsibilities rather than hiding the whole operation
~~~

The canonical rule is:

> Guest computation that determines ordinary Protos execution remains visible to
> Truffle Partial Evaluation. Opaque host services and genuinely cold runtime work
> cross narrow explicit boundaries. When one method mixes those categories, split
> it instead of boundary-wrapping the whole operation.

A Java method does not become a Truffle boundary merely because it uses Java.

A Java representation is not replaced merely because one of its methods is in
GraalVM's runtime-compilation blocklist. Replacement requires current evidence
that the method is reachable from a relevant runtime-compilable guest root and
survives PE or otherwise prevents the required runtime compilation.

## GraalVM 25.3.4.1 boundary authority

The exact `vm-25.3.4.1` Graal source establishes:

~~~text
@TruffleBoundary -> runtimeCompilationForbidden

blocklisted implementation method
  -> not permitted for runtime compilation

reported blocklist violation
  -> blocklisted implementation method is an actual runtime-compilation
     candidate and its target is not in the temporary target allowlist
~~~

The exact source blocklists, among others, `BigInteger`, `BigDecimal`,
`Collection`, `List`, `Map`, `HashMap`, `ConcurrentHashMap`,
`IdentityHashMap`, `Iterable`, `Iterator`, `BiFunction`, `Function`,
`Predicate` and `Supplier`.

The same source temporarily allowlists selected target methods including
`Function.apply`, `Predicate.test`, selected `List` operations,
`Iterator.next/hasNext` and `Iterable.iterator`. That allowlist is migration
machinery, not Protos architecture authority.

Therefore:

~~~text
BLOCKLIST_MEMBERSHIP != REQUIRE_CALLER_BOUNDARY
BLOCKLIST_MEMBERSHIP != PROOF_OF_HOT_GRAPH_RESIDENCE
~~~

## Required PE-visible guest kernel

At minimum the following remain PE-visible:

- ordinary Bytecode send/call/lookup;
- parameter/default/rest binding;
- frame-backed lexical reads/writes;
- ordinary object lookup semantics;
- module cache-hit behavior and execution of an already materialized guest target;
- guest continuation and resumption decisions;
- Protos Future/Task/Actor/C-prime semantic state transitions; and
- guest callbacks and guest `CallTarget` execution.

In particular:

~~~text
BROAD_ENTER_NATIVE_BOUNDARY_ACCEPTABLE=NO
BROAD_RESUME_BOUNDARY_ACCEPTABLE=NO
BIND_PARAMETERS_MUST_REMAIN_PE=YES
~~~

A boundary must not enclose guest execution merely to hide a Java functional
carrier.

## Narrow boundary gateways

The following categories are canonical boundary work when reached from guest
execution:

- module/source cache-miss load, parse, canonicalization and Bytecode lowering;
- physical filesystem/network/stream operations;
- host environment/process acquisition;
- host wait, worker scheduling and resource-management leaves;
- operation-specific `BigInteger` leaves; and
- explicit post-lookup bound-Closure materialization where the existing
  `ProtosValueLookup.materializeMemberRead()` seam is used.

A boundary returns a value/state/guest target to PE-visible code before guest
execution continues.

## Mixed gateways must split

The following current surfaces are architectural SPLIT candidates:

- module cache hit versus cache miss/source compilation;
- `PreparedClosureCall.enterNative`;
- continuation/native suspension;
- generic `Supplier` carriers that cross guest-hot resumptions;
- argument-vector preparation when blocklisted collection machinery actually
  survives PE;
- Future/I/O/release suspension machinery;
- Actor scheduler host dispatch versus guest handler execution;
- portable file/open lifecycle state versus physical NIO calls; and
- ordinary object-slot representation only if current reachability/compiler
  evidence proves the current backing representation survives PE as a problem.

SPLIT does not authorize a broad rewrite. The smallest evidence-proven seam is
changed.

## Integer boundary

Protos unbounded Integer semantics do not change.

The selected direction is:

~~~text
INTEGER_GUEST_CHECKS_AND_RESULT=KEEP_PE
GENERIC_BIFUNCTION_HOT_DISPATCH=REMOVE_FROM_PROVEN_HOT_PATH
BIGINTEGER_OPERATION=NARROW_LEAF_BOUNDARY
SMALL_INT_REPRESENTATION_REQUIRED_BY_PLAT039=NO
NEW_INTEGER_SEMANTICS=NO
~~~

SimpleLanguage in the same Graal source family provides the direct precedent:
the guest specialization remains visible while individual `BigInteger`
operations use narrow `@TruffleBoundary(allowInlining = true)` helpers.

## Call argument representation

Call/send preparation remains guest-hot work.

~~~text
CALL_PREPARATION=KEEP_PE
BROAD_ARGUMENT_PREPARATION_BOUNDARY=NO
SEALED_HOT_VECTOR_PREFERRED_DIRECTION=Object[]
REPOSITORY_WIDE_LIST_REPLACEMENT=NO
~~~

`List`/`ArrayList` replacement is evidence-gated. If PE removes the relevant
machinery, no representation migration is required merely to satisfy this
decision.

## Ordinary object slots

Ordinary object-slot lookup remains PE-visible.

~~~text
PROTOS_VALUE_LOOKUP=KEEP_PE
BROAD_LOOKUP_BOUNDARY=NO
LINKED_HASH_MAP_REPLACEMENT_AUTOMATIC=NO
REPRESENTATION_REPLACEMENT=ONLY_ON_CURRENT_REACHABILITY_OR_COMPILER_EVIDENCE
~~~

This explicitly narrows the preliminary PLAT039 suspicion: the current map-backed
representation is not condemned merely by blocklist membership.

## Continuations and functional carriers

Host waiting/resource registration and guest resumption are separate concerns.

~~~text
HOST_WAIT_OR_RESOURCE_CALLBACK=BOUNDARY
EXPLICIT_SUSPENSION_STATE=KEEP_RUNTIME_VISIBLE
GUEST_RESUMPTION_DECISION=KEEP_PE
GENERIC_SUPPLIER_ACROSS_GUEST_HOT_SEAM=SPLIT_REMOVE
GUEST_CALLBACK_OR_CALLTARGET_EXECUTION=OUTSIDE_OPAQUE_BOUNDARY
~~~

`Runnable` is not automatically classified by analogy with `Supplier`; each
use must be traced according to what it executes.

## Source compilation and I/O boundaries

~~~text
SOURCE_COMPILATION_RUNTIME_BOUNDARY=
  CACHE HIT AND RESULTING GUEST TARGET KEEP_PE;
  CACHE MISS LOAD/PARSE/CANONICALIZE/LOWER IS COLD BOUNDARY;
  GUEST TARGET EXECUTION RETURNS OUTSIDE THE BOUNDARY

IO_ENVIRONMENT_RUNTIME_BOUNDARY=
  PHYSICAL HOST IO/ENVIRONMENT/WAIT/SCHEDULING IS BOUNDARY;
  PROTOS LIFECYCLE/FUTURE/TASK/ACTOR/CONTINUATION STATE AND GUEST REENTRY
  REMAIN OUTSIDE THE OPAQUE BOUNDARY
~~~

## I068 interaction

PLAT039 preserves I068.

I068 solved frame-backed lexical authority for eligible current, parameter and
captured lexical state plus debugger/reflection projection. It did not claim to
solve call argument carriers, ordinary object-slot backing representation,
`BigInteger`, parser/lowerer reachability, generic continuation carriers or
physical I/O.

~~~text
I068_REWORK_REQUIRED=NO
I068_ARCHITECTURE_REOPEN_REQUIRED=NO
~~~

## PERF010 interaction

PLAT039 identifies causal experiment targets but does not establish a performance
percentage.

~~~text
PLAT039_PROVES_PERF010_DOMINANT_CAUSE=NO
PLAT039_PROVES_ATTRIBUTABLE_FRACTION=NO
~~~

PERF010 should interpret one isolated PLAT039 implementation change only after
compiler-graph/lifecycle evidence and paired timing establish causality.

## Candidate result

PLAT039 considered:

- A — status quo / ad-hoc `@TruffleBoundary` repair;
- B — narrow host/cold leaf boundaries only, current hot representations retained;
- C — explicit PE-visible guest kernel + narrow host gateways + evidence-gated
  splitting/replacement of blocklisted hot carriers;
- D — broad native/continuation/`enterNative` boundary;
- E — disable Native Image runtime compilation;
- defer / do nothing.

A is rejected because a changing reachability graph would drive ad-hoc annotation
placement without a semantic architecture rule.

B remains the strongest conservative alternative but is insufficient as the
durable rule because existing mixed guest/host gateways require structural
splitting rather than either exposing everything or hiding everything.

D is rejected because it can hide parameter binding, continuation composition
and guest re-entry.

E is inadmissible without reopening PLAT038.

Deferral is rejected because I069 is already blocked on the boundary decision and
PLAT038 requires an optimizing Native Image.

Candidate C is selected in its evidence-gated form. It does **not** authorize a
repository-wide collection rewrite, ordinary-slot redesign, small-int redesign or
custom collection framework.

## Owner-approved invariant/delta consistency check

The selected Candidate C preserves every applicable ratified PLAT038 invariant:

~~~text
TRUFFLE_RUNTIME_COMPILER_IN_NATIVE_REQUIRED=PASS
INTERPRETER_ONLY_NATIVE_REJECTED=PASS
JVM_DEVELOPMENT_RETAINED=PASS
NATIVE_CONFORMANCE_REQUIRED=PASS
CANONICAL_GRAAL_TRUFFLE_AUTHORITY_PRESERVED=PASS
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
~~~

It also preserves I068's frame-backed lexical authority and all normative Protos
lookup/call/control semantics.

No owner-approved invariant is reopened.

Material refinement from the preliminary evidence is explicit:

~~~text
HOT_COLLECTION_REPLACEMENT_AUTOMATIC=NO
ORDINARY_SLOT_MAP_REPLACEMENT_AUTOMATIC=NO
REPRESENTATION_CHANGE_REQUIRES_CURRENT_PE_REACHABILITY_EVIDENCE=YES
~~~

## Ratification summary

~~~text
PLAT039_STATUS=RATIFIED

TRUFFLE_RUNTIME_COMPILER_IN_NATIVE=REQUIRED

GUEST_HOT_PATH_BOUNDARIES_IDENTIFIED=YES
HOST_RUNTIME_BOUNDARIES_IDENTIFIED=YES

BROAD_ENTER_NATIVE_BOUNDARY_ACCEPTABLE=NO
BROAD_RESUME_BOUNDARY_ACCEPTABLE=NO
BIND_PARAMETERS_MUST_REMAIN_PE=YES

OBSERVABLE_PROTOS_SEMANTIC_CHANGE_REQUIRED=NO

PLAT039_SELECTED_CANDIDATE=C
PLAT039_SELECTED_CANDIDATE_NAME=
  EVIDENCE_GATED_PE_VISIBLE_GUEST_KERNEL_WITH_NARROW_HOST_GATEWAYS

IMPLEMENTATION_SCOPE=
  BOUNDED I069 GATEWAY SLICES;
  NARROW HOST/COLD BOUNDARIES;
  SPLIT MIXED GUEST/HOST GATEWAYS;
  REPRESENTATION CHANGES ONLY WHEN CURRENT COMPILER/REACHABILITY EVIDENCE
  REQUIRES THEM

IMPLEMENTATION_REPOSITORY=guillermomolina/protos
EXISTING_I069_CAN_OWN_IMPLEMENTATION=YES
NEW_IMPLEMENTATION_ISSUE_REQUIRED=NO
IMPLEMENTATION_READY=YES
~~~
