# D148 — Equality and inequality protocol relationship

## Decision

D148 ratifies **Candidate A-prime — derived inequality with retained `!=` source syntax**.

```text
D148_STATUS=RATIFIED
SELECTED_CANDIDATE=A_PRIME
PROTOS_REVISION=b1b5c91b365a57ed65797b78ab9a6466e7df16f5
PROJECT_RECORD_BASE=4c057a3761ea43bceb773d8b9b60ce5080bb7922
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

This record is durable non-normative decision/rationale evidence. Observable
Protos semantics remain authoritative under `guillermomolina/protos:spec/**`.

## Owner approval

The project owner explicitly approved the exact Candidate A-prime in the active
D148 interaction:

```text
ok, aqui tengo que creerte, no tengo una opinion al respecto, apruebo A'
```

The approved candidate is the exact A-prime proposal in D148/#578 and its
research packet comment `5726198362`.

## Ratified semantics

The selected model is:

```text
EQUALITY_CUSTOMIZATION_POINT="=="
!=_SOURCE_SURFACE=KEEP
!=_SEMANTICS=LOGICAL_COMPLEMENT_OF_VALIDATED_==
!=_ORDINARY_PROTOCOL_SELECTOR=NO
OBJECT_!=_STANDARD_SLOT=REMOVE
CUSTOM_!=_DIVERGENCE=REJECT

===_IDENTITY=UNCHANGED
!==_NONIDENTITY=UNCHANGED
MAP_HASH_EQUALITY=UNCHANGED
TRUTHINESS=NO
```

For source expressions:

```protos
a != b
```

the observable semantic contract is:

1. evaluate `a` exactly once;
2. evaluate `b` exactly once, after `a`;
3. invoke the receiver's ordinary customizable `==` behavior exactly once;
4. require the `==` result to be canonical `true` or canonical `false`, or
   propagate its Error/non-normal control result according to existing semantics;
5. return the opposite canonical Boolean.

At the source-semantic level this is the exact relationship:

```text
a != b  means  !(a == b)
```

with the existing strict Boolean-result contract for `==`.

The `!=` source operator therefore does not perform ordinary lookup or dispatch
of a selector named `"!="`.

## Equality customization boundary

`==` remains the sole ordinary customization point for Core semantic equality.

D148 does not strengthen custom `==` into a universal equivalence relation.
Existing user-defined equality may remain asymmetric, non-reflexive, or otherwise
domain-specific where the current language permits that behavior.

D148 establishes only this law:

```text
for every completed valid comparison:
    a != b  is exactly the logical complement of  a == b
```

A separate domain relation remains expressible as an ordinary meaningfully named
message such as `approximatelyEquals`, `compatibleWith`, or another domain-owned
selector. Such a relation is not Core `!=`.

## Removed protocol capability

The independently customizable ordinary `!=` relation is removed from the Core
equality protocol.

Accordingly:

- Core no longer requires a standard `Object.!=` slot;
- a user-defined ordinary slot literally named `"!="`, if another structural
  facility can create such a name, does not affect source `a != b`;
- no Core contract reserves that slot as an inequality customization hook;
- implementation optimization does not justify preserving a second public
  semantic relation.

This does not remove the `!=` source spelling.

## Identity remains separate

D148 does not change semantic identity.

```text
===  = existing non-overridable semantic identity operation
!==  = existing non-overridable logical complement of ===
```

Overriding `==` still cannot change `===` or `!==`.

## Map and hash remain unchanged

Map/Set-style equality-sensitive behavior continues to depend on the existing
`hash` plus directed `==` contracts.

Independent `!=` has no Map/hash responsibility to preserve.

D148 therefore introduces no new equality/hash coherence rule and does not change
the existing obligation that custom equality used as associative-key equality be
coherent with the applicable hash behavior.

## Comparative evidence

The D148 packet compared materially different design families:

- Kotlin and ECMAScript — inequality is structurally derived from equality;
- Python, Ruby and Smalltalk/Pharo — a separate inequality hook/message exists,
  with default behavior derived from equality;
- Self — binary selectors are ordinary messages, providing the strongest
  message-uniformity argument for an independent selector;
- Rust, Swift and C# — separate or pairable inequality customization is coupled
  to an explicit coherence obligation.

The strongest alternative was the message-oriented model represented by
Self/Smalltalk/Ruby.

The strongest hybrid was a separate `!=` hook constrained by a mandatory
complement law, as seen in Rust/Swift/C#-style designs.

That hybrid was rejected for Protos because the law has no satisfactory dynamic
enforcement boundary: leaving it unchecked collapses to two potentially
contradictory public relations, while dynamically validating both `==` and `!=`
would duplicate user-observable effects, suspension, errors, and control
behavior. Protos has no nominal type/trait declaration system that can prove the
behavioral complement law statically.

## Rationale

Current repository evidence shows real use of `!=` source syntax but no
production requirement for independently customizing an ordinary `"!="`
selector.

`==` already has the meaningful systemic responsibility: it participates in
semantic equality and in equality/hash-sensitive collection behavior. No Core
subsystem requires a second independent inequality relation.

Candidate A-prime is therefore the smallest sufficient design:

- equality remains ordinary customizable behavior;
- concise inequality syntax remains available;
- contradictory `==`/`!=` user-defined states are structurally impossible
  through the source operator;
- no second equality-like protocol slot must be learned, documented, tested, or
  kept coherent;
- alternate domain relations remain ordinary named messages;
- implementations retain freedom to optimize the complement when observable
  behavior is preserved.

## GITHUB021 invariant consistency

D148 recorded the following existing invariants before selection:

```text
== = ordinary customizable semantic equality
=== = non-overridable semantic identity
!== = non-overridable complement of ===
EQUALITY_RESULT = canonical Boolean or Error
TRUTHINESS = absent
OVERRIDING_EQUALITY_CHANGES_IDENTITY = NO
MAP_EQUALITY_HASH_RULES = unchanged unless explicitly reopened
IMPLEMENTATION_CONVENIENCE_IS_AUTHORITY = NO
```

Candidate A-prime preserves every invariant.

The only changed consequence is the exact question D148 was opened to decide:

```text
OLD: != may dispatch an independently overridable ordinary selector
NEW: != is the mandatory complement of validated ==
```

That consequence was explicitly surfaced in the decision packet and explicitly
approved by the owner.

```text
PRESERVES_RECORDED_INVARIANTS=YES
HIDDEN_REOPENING=NO
MATERIALLY_NEW_UNSURFACED_CONSEQUENCE=NO
REOPENED_INVARIANT=NONE
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Reconsideration boundary

If a future real requirement needs another standardized comparison relation,
design that relation from the then-current evidence and give it semantics and
naming that reflect its actual meaning.

D148 does not reserve an independently dispatchable `!=` hook for that future.
Reintroducing such a hook would require a new semantic decision because it would
again permit the spelling "not equal" to diverge from semantic equality.

## Deferred normative and implementation publication

This ratification record selects semantics only.

Separate implementation work must reconcile at least:

- `spec/PROTOS_GRAMMAR.md`;
- `spec/semantics/VALUES_AND_COLLECTIONS.md`;
- `spec/runtime/ABSTRACT_RUNTIME.md` where it describes inequality evaluation;
- `spec/PROTOS_SPEC_CHANGELOG.md`;
- canonical lowering and equality tests;
- source-backed Core `Object.!=` behavior and bootstrap installation;
- conformance tests that currently prove the independent protocol;
- user-facing documentation where it describes custom inequality.

That work must also prove the existing strict Boolean-result contract for custom
`==` on the `!=` execution path. An arbitrary non-Boolean result from custom
`==` must not become a valid inequality result merely because the returned
object responds to `not`.

No normative or executable change to `guillermomolina/protos` is made by this
decision-record publication.
