# D192 — projected foreign each normal-completion result

Status: **RATIFIED — Candidate A / exact receiver**

Formal decision: `guillermomolina/protos#833`

Parent implementation: `I082 / guillermomolina/protos#830`

Normative reconciliation owner: `I083 / guillermomolina/protos#834`

Decision date: **2026-10-07**

## Decision

D192 defines the one result contract that D188 left unspecified for projected
foreign pull iteration:

~~~text
foreign.each(block)
~~~

The selected contract is:

~~~text
SELECTED_CANDIDATE=A_EXACT_RECEIVER

FOREIGN_EACH_NORMAL_COMPLETION_RESULT=EXACT_ORIGINAL_PROTOS_FACING_RECEIVER

RAW_FOREIGN_RECEIVER_RESULT=SAME_RAW_PROTOS_FOREIGN_REFERENCE
FOREIGN_MODULE_FACADE_RESULT=SAME_ACTOR_LOCAL_PROTOS_FACADE
UNDERLYING_FOREIGN_TARGET_RESULT=NO
CALLBACK_RESULT_SELECTS_EACH_RESULT=NO
~~~

When projected foreign iteration exhausts normally after every reached callback
completes normally, `each` answers the exact original Protos-facing receiver.

A raw foreign reference therefore answers that same raw Protos foreign
reference. An attached foreign module facade answers that same Actor-local
Protos facade. The result is never replaced by the underlying foreign target,
the last element, the last callback result, the provider iterator, host null, a
foreign completion token, or an implementation-dependent value.

## Why this result

The current normative Protos surface has no single language-wide rule saying
that every selector named `each` must return its receiver. Instead, the
domain-specific contracts converge consistently:

~~~text
Array.each        -> original Array receiver
Bytes.each        -> receiver Bytes
Map.each          -> receiver Map
IdentityMap.each  -> same result semantics as Map
Environment.each  -> Environment receiver
~~~

`Environment.each` is particularly useful evidence because its normative
contract explicitly rejects `null`, the last callback result, a newly allocated
collection, and an implementation-selected result.

D188 also already separates projected foreign iteration from collection-family
membership. Returning the receiver therefore does not make a foreign value a
standard Array or Map and does not confer Array/Map snapshot, copying, identity,
or Actor/P transfer semantics.

## Alternatives considered

### Candidate A — exact receiver

Selected.

It preserves the observable convention of every current standard Protos
`each` contract while returning the Protos-facing identity actually used by
ordinary code.

### Candidate B — canonical null

Rejected.

Completion-only visitation has strong external precedent, but in current Protos
it would create the first known `each` result exception, including a direct
contrast with the explicit Environment contract. D188 selected the ordinary
Protos-facing `each` institution rather than a separate visitor API.

### Candidate C — other fixed result

Rejected.

No stronger semantic case was found for last element, last callback result,
iterator, host null, foreign completion token, or provider-dependent result.

## Comparative research summary

The GITHUB010 investigation covered materially different approaches:

~~~text
Ruby Array#each             -> receiver/self
Java Iterable.forEach       -> void
JavaScript Array.forEach    -> undefined
Kotlin Iterable.forEach     -> Unit
Rust Iterator::for_each     -> ()
Swift Sequence.forEach      -> Void
~~~

This prior art demonstrates a genuine split between fluent receiver-returning
protocols, completion-only visitor APIs, and consuming iterator combinators. It
is evidence about the design space, not authority over Protos.

Ruby is the closest receiver-oriented precedent. Rust is the clearest
architectural counterexample: its `for_each` is a terminal operation on the
Iterator abstraction itself, whereas D188 deliberately keeps the provider
iterator internal and exposes an ordinary Protos-facing receiver protocol.

## Chaining and composition

Candidate A permits:

~~~text
value.each(block).otherOperation()
~~~

against the exact same Protos-facing receiver.

No current repository dependence on post-`each` chaining was required to select
A, so chaining is a composability benefit rather than a compatibility argument.

## Provider and std:interop freedom

The selected result is owned entirely by the Protos projection layer.

Providers continue to supply only the foreign pull operations and never select
the Protos `each` result. A future `std:interop` API remains free to expose
explicit iterator/visitor operations with independently specified result
contracts.

~~~text
PROVIDER_FREEDOM_IMPACT=NONE
STD_INTEROP_IMPACT=NONE_MATERIAL
~~~

## GITHUB021 invariant / delta result

The selected candidate changes exactly one previously unspecified D188 point:

~~~text
D188_ITERATION_MECHANISM_DELTA=NONE

D188_NORMAL_COMPLETION_RESULT=
  previously unspecified
  ->
  exact original Protos-facing receiver

D189_DELTA=NONE
PLAT052_DELTA=NONE
PLAT053_DELTA=NONE

ACTOR_TRANSFER_DELTA=NONE
P_TRANSFER_DELTA=NONE
FOREIGN_ERROR_DELTA=NONE
PROVIDER_AUTHORITY_DELTA=NONE

ITERATION_ORDER_DELTA=NONE
SNAPSHOT_SEMANTICS_DELTA=NONE
PULLING_DELTA=NONE
CALLBACK_INVOCATION_DELTA=NONE

DECISION_INVARIANT_CONSISTENCY=PASS
~~~

## Strongest counterargument and regret path

The strongest argument against A is that many mainstream `forEach` APIs are
completion-only and that receiver-return could superficially suggest
collection-like semantics even though projected foreign iteration has no
Array/Map family membership or snapshot semantics.

The regret scenario is a future interoperability model that strongly prefers
completion-only visitors after user code has begun relying on fluent
`foreign.each(...)` chaining.

The preferred escape path is additive: keep ordinary projected `each` stable
and place explicit completion-only or lower-level iterator operations in a
future `std:interop` facility. Changing `foreign.each` later would remain
possible only as an explicit incompatible semantic revision.

## Normative reconciliation

This durable decision record is non-normative.

The selected observable language semantic is routed to:

~~~text
I083 / guillermomolina/protos#834
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
IMPLEMENTATION_SCOPE=NORMATIVE_SPECIFICATION_RECONCILIATION_ONLY
~~~

Expected primary owner at allocation time:

~~~text
spec/semantics/VALUES_AND_COLLECTIONS.md
  Foreign Values
    Indexed access, foreign hash containers, and iteration
~~~

The normative reconciliation must state the exact receiver result while
preserving all existing D188 iteration mechanics and must advance the global
specification revision.

## Implementation evidence is not decision authority

I082-D2 was already published at:

~~~text
PRODUCT_REVISION=271a27662ee600212b7163d73253c2060875b0e8
PRODUCT_VERSION=0.3.276-SNAPSHOT
COMMIT_SUBJECT=I082-D2: add D188 foreign pull iteration through ordinary each
CURRENT_IMPLEMENTATION_CHOICE=EXACT_RECEIVER
~~~

That implementation now agrees with the selected D192 result, but its prior
existence was not used as semantic authority.

At owner approval time the live Protos line had advanced independently to the
PERF033-A publication series; that unrelated movement does not change the D192
contract.

## AI-assistance disclosure

This durable decision record was materially prepared with AI assistance from
ChatGPT from the completed D192 investigation, current repository evidence, and
the project owner's explicit approval. No independent human review is claimed.
