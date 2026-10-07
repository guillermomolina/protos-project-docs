# I084 — D193 specification-reconciliation allocation evidence

Status: **ALLOCATED — BLOCKED UNTIL D193 DURABLE RATIFICATION, THEN READY**

Implementation issue: `guillermomolina/protos#837`

Decision authority: `D193 / guillermomolina/protos#836`

Parent library work: `LIB021 / guillermomolina/protos#835`

Implementation repository: `guillermomolina/protos`

Allocation date: **2026-10-07**

## Allocation

I084 owns only the normative specification reconciliation required after the
project owner selected D193 Candidate A−.

~~~text
FORMAL_IDENTIFIER=I084
IMPLEMENTATION_SCOPE=NORMATIVE_SPECIFICATION_RECONCILIATION_ONLY
RUNTIME_IMPLEMENTATION_CHANGE=NO
~~~

Expected primary owner at allocation time:

~~~text
spec/semantics/VALUES_AND_COLLECTIONS.md
  Foreign Values
    Relation to explicit interoperability
~~~

The current normative text intentionally leaves the public `std:interop` API
undefined. I084 replaces that placeholder with the exact D193 contract and
advances the global specification revision.

## Approved semantic input

~~~text
std:interop

invoke(target, ...arguments)
instantiate(target, ...arguments)
readMember(target, name)
writeMember(target, name, value)
~~~

The reconciliation must preserve the exact D193 error, callback, authority,
Actor/P, session-generation and deferral rules, including the explicit
`invokeMember` delta:

~~~text
invokeMember is not universally equivalent to readMember + invoke
invokeMember remains deferred because no current workload requires it
a future invokeMember operation is additive
~~~

## Non-goals

I084 does not implement runtime/library code and does not add any deferred
interop family.

It must not change ordinary call, member lookup/invocation, assignment,
indexing, iteration, D189 callbacks, ForeignError taxonomy, Actor/P transfer,
PLAT052 authority, PLAT053 provider topology, D192 `each`, provider discovery,
or dependency acquisition.

## Release sequencing

~~~text
D193 durable ratification publication
    -> I084 READY

I084 normative specification publication
    -> LIB021 implementation released
~~~

Repository-content implementation remains human-executed in
`guillermomolina/protos`.

## AI-assistance disclosure

This allocation evidence was materially prepared with AI assistance from
ChatGPT from the owner-approved D193 contract and current repository
coordination. No independent human review is claimed.
