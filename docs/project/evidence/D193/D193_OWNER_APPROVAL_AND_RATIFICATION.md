# D193 — owner approval and ratification evidence

Status: **APPROVAL PROVENANCE RECORDED — A− SELECTED; DURABLE RATIFICATION**

Formal decision: `guillermomolina/protos#836` — D193

Parent library work: `guillermomolina/protos#835` — LIB021

Normative reconciliation owner: `guillermomolina/protos#837` — I084

Approval date: **2026-10-07**

## Approval provenance

D193 completed the owner-review packet for the first public `std:interop`
operation contract.

The final recommendation immediately presented to the project owner was:

~~~text
RECOMMENDED_CANDIDATE=A_MINUS_MINIMAL_OPERATION_ONLY_ESCAPE_HATCH
STD_INTEROP_MODULE=std:interop
PUBLIC_OPERATIONS=invoke, instantiate, readMember, writeMember
~~~

The packet explicitly included the material `invokeMember` correction:

~~~text
INVOKE_MEMBER_COMPOSITION_IS_NOT_UNIVERSAL=YES
INVOKE_MEMBER=DEFER
RATIONALE=NO_CURRENT_PROTOS_PROVIDER_OR_WORKLOAD_REQUIRES_IT
FUTURE_ADDITION=ADDITIVE_DISTINCT_OPERATION
~~~

Immediately after that exact recommendation, the project owner explicitly
answered:

> Apruebo esta recomendacion

The approval therefore selects the complete exact candidate, including the
surfaced `invokeMember` delta rather than the older LIB021-0 composition
rationale.

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
OWNER_APPROVAL=EXPLICIT
SELECTED_CANDIDATE=A_MINUS_MINIMAL_OPERATION_ONLY_ESCAPE_HATCH
~~~

## Selected surface

~~~text
STD_INTEROP_MODULE=std:interop

PUBLIC_OPERATIONS=
  invoke(target, ...arguments)
  instantiate(target, ...arguments)
  readMember(target, name)
  writeMember(target, name, value)

WRITE_MEMBER_NORMAL_RESULT=EXACT_ORIGINAL_PROTOS_VALUE_ARGUMENT
~~~

The selected API is operation-only. It publishes no capability predicates,
provider discovery, acquisition, conversion, metaobject/type surface,
explicit iteration, indexed/hash family, or retained callback facility.

## GITHUB021 consistency check

The final owner-review packet explicitly revalidated the already-ratified
foreign-interoperability foundations.

~~~text
D188_DELTA=NONE
D189_DELTA=NONE
PLAT052_DELTA=NONE
PLAT053_DELTA=NONE
D192_DELTA=NONE

NEW_DELTA=
  invokeMember composition is not universal;
  invocable-but-unreadable members are possible;
  no current Protos workload justifies v1 publication;
  future invokeMember remains additive

DECISION_INVARIANT_CONSISTENCY=PASS
~~~

The new delta does not change ordinary member invocation. D188's ordinary
`foreign.member(args...)` remains read plus ordinary Protos invocation.
It only corrects the reason why a distinct explicit `invokeMember` operation
is deferred.

## Product-state revalidation

Immediately before owner approval and again before ratification publication,
the live product revision remained:

~~~text
RATIFICATION_REVALIDATED_PROTOS_REVISION=c0ac98971df115d64b7bc9f146e8b02e11da30e6
RATIFICATION_REVALIDATED_PROTOS_VERSION=0.3.280-SNAPSHOT
RATIFICATION_SPECIFICATION_REVISION=0.1.447
HEAD_SUBJECT=I082-G: add restricted host Java provider baseline
~~~

This is exactly the revision audited by LIB021-0, so there is no intervening
foreign-interop, Standard Library, authority, callback, provider/session, or
lifetime drift to reconcile.

## Current implementation evidence

The existing I082 substrate already establishes:

~~~text
one provider-neutral D188 admission/projection/failure substrate
private EXECUTABLE / INSTANTIABLE / INDEXED_READ / INDEXED_WRITE / HASH_ENTRIES / ITERABLE classification
ordinary call projection only when execution is unambiguous
ordinary at/atPut projection only when indexed-vs-hash meaning is unambiguous
protected ordinary institution names
entered/not-entered ForeignError boundary
D189 operation-scoped callback lifetime
exact provider/session/generation binding
PLAT052 fail-closed authority
~~~

The current adapter has execution/member-read operations but no complete generic
public `instantiate` or `writeMember` path. That is implementation work after
normative reconciliation and is not a new semantic decision.

## Normative reconciliation allocation

Because D193 defines observable public Standard Library semantics, durable
ratification alone does not make the API normative.

Specification reconciliation is allocated to:

~~~text
I084 / guillermomolina/protos#837
TITLE=Reconcile D193 std:interop public operation contract into normative specification
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
IMPLEMENTATION_SCOPE=NORMATIVE_SPECIFICATION_RECONCILIATION_ONLY
~~~

Until I084 publishes:

~~~text
D193_DECISION=RATIFIED
D193_SPECIFICATION_RECONCILIATION=PENDING_I084
LIB021_IMPLEMENTATION=BLOCKED_ON_I084
~~~

After I084 publishes the normative reconciliation, LIB021 may implement the
approved A− operation surface without reopening D193 unless implementation
exposes a genuinely new substantive choice.

## Project-record revision coupling

This publication was prepared from live
`guillermomolina/protos-project-docs` default-branch base:

~~~text
PROJECT_RECORD_BASE_REVISION=6978fdfe71a791c9da3cb01c8cdc7c7458a27adc
~~~

The resulting project-record revision is recorded back on D193/#836,
I084/#837, and LIB021/#835 after publication.

## Validation class

This is governance/documentation-only publication. It changes no Protos source,
normative specification, product version, runtime code, or product tests.

The publication is re-read from the default branch after commit.

## Closure routing

After this durable publication is verified:

~~~text
D193_STATUS=COMPLETED
D193_CLOSE=YES

I084_STATUS=READY
I084_IMPLEMENTATION_AUTHORIZED=YES

LIB021_STATUS=BLOCKED_ON_I084
LIB021_RUNTIME_IMPLEMENTATION_AUTHORIZED=NO
~~~

## AI-assistance disclosure

This evidence record was materially prepared with AI assistance from ChatGPT
using the completed D193 investigation, current live repository state, the
project owner's explicit approval, and the established D192/I083 ratification
pattern. No independent human review is claimed.
