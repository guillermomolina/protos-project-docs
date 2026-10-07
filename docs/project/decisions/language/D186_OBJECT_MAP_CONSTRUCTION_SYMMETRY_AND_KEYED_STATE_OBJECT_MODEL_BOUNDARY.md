# D186 — Object/Map construction symmetry and keyed-state object-model boundary

Status: **RATIFIED — Candidate A**

Approval date: **2026-10-05**
Decision issue: guillermomolina/protos#802
Audited Protos revision: 00526bdeffaec4360f2ed2deed3d59e16ae3a431
Project-record base: b14cf085daa7788d4990622e6923fea5b266c4b1
Related prior decision: D136 / guillermomolina/protos#542

This is a durable non-normative decision record. Observable Protos semantics
remain authoritative under guillermomolina/protos:spec/**.

## Decision

D186 selects **Candidate A — retain the current Object/Map construction and
state model**.

The selected boundary is:

~~~text
OBJECT_CONSTRUCTION={}                         KEEP
NORMAL_MAP_CONSTRUCTION=%{}                    KEEP
OBJECT_COLON=NAMED_SLOT_CREATION               KEEP
MAP_KEY=EVALUATED_EXPRESSION                   KEEP
MEMBER_AND_INDEXED_STATE=ORTHOGONAL            KEEP
MAP_INITIAL_DEFINITION_VS_ATPUT                 KEEP
NORMAL_MAP_HASH_DIRECTED_EQUALITY_LAW           KEEP
IDENTITYMAP_DISTINCT                           KEEP
ARRAY_POSITIONAL_STATE_DISTINCT                KEEP

ARBITRARY_OBJECT_KEYED_STATE_ACQUISITION       NOT_ADDED
GENERIC_KEYED_LITERAL_PROTOCOL                  NOT_ADDED
EXPECTED_KIND_CONTEXTUAL_INFERENCE              NOT_ADDED
ASSOCIATION_PAIR_TUPLE_INSTITUTION              NOT_ADDED

OBSERVABLE_SEMANTIC_DELTA                       NONE
SYNTAX_DELTA                                    NONE
IMPLEMENTATION_DELTA                            NONE
~~~

D186 therefore does not alter current Protos syntax, semantics, specification,
runtime behavior, tooling, Standard Library surface, or compatibility.

## Clarified conceptual model

The investigation established that the existing model is already more unified
than the surface distinction can initially suggest.

A standard collection value remains an ordinary Protos Object while owning an
additional receiver-local semantic state domain:

~~~text
standard Map
    = ordinary Protos Object
    + normal equality-keyed association state

standard IdentityMap
    = ordinary Protos Object
    + identity-keyed association state

standard Array
    = ordinary Protos Object
    + dense positional indexed state
~~~

This description is explanatory only. D186 does **not** introduce a public
StateDomain abstraction, a capability object, a new protocol, or a new family.

An arbitrary Object does not thereby own or generically acquire normal Map,
IdentityMap, or Array state.

Delegating to a standard collection prototype, inheriting at/atPut behavior, or
defining similarly named messages does not confer the corresponding standard
collection-family state.

## Why the percent discriminator remains

The current forms are visually similar:

~~~protos
{
    name: value
}

%{
    key: value
}
~~~

but the left side of the colon has different semantics.

Inside an ordinary Object body, the left side denotes a slot-creation target and
therefore a **name**.

Inside Map construction, the left side is an **expression**. It is evaluated to
an arbitrary Protos key before the construction-time normal-Map search and
initial-association definition.

Consequently these examples are intentionally not equivalent:

~~~protos
{ a: 1 }
%{ a: 1 }

{ "a": 1 }
%{ "a": 1 }

{ foo.bar: 1 }
%{ foo.bar: 1 }
~~~

Removing the percent sign without another local discriminator would not remove
this semantic distinction. It would move the distinction into contextual
typing, body-item heuristics, a new computed-key marker, or runtime intent
guessing.

D186 finds no smaller current mechanism than the existing explicit percent
domain bit.

## D136 invariant reconciliation

Candidate A preserves the complete ratified D136 boundary.

~~~text
%{...} construction surface                         PRESERVED
colon-delimited initial association definitions     PRESERVED
ordinary lexical lookup of Map                      PRESERVED
standard Map factory eligibility                    PRESERVED
fresh standard normal Map result                    PRESERVED
left-to-right reached entry processing               PRESERVED
key then value evaluation exactly once              PRESERVED
normal Map hash / directed equality search          PRESERVED
duplicate/equal initial definition -> Error          PRESERVED
representative-key and recorded-hash semantics      PRESERVED
construction distinct from later atPut mutation     PRESERVED
arbitrary custom Map.call ineligible                PRESERVED
IdentityMap distinct                                PRESERVED
Association/Pair/Tuple absent                       PRESERVED
generic keyed-literal protocol absent               PRESERVED
expected-kind contextual inference absent           PRESERVED
~~~

No D136 invariant is reopened.

## Other preserved boundaries

D186 also preserves the applicable outcomes of later decisions and current
specification architecture, including:

- D131 protocol-first matching and standard structural Map/Array matching;
- D140 local-slot-based Object composition;
- D142 normal Map versus IdentityMap key-law distinction;
- D147 semantic-family membership not being granted by accidental
  delegation/protocol shape;
- D154 primitive semantic identity authority used by IdentityMap;
- D168 reuse of ordinary Array where semantics coincide while retaining
  specialized Environment semantics instead of replacing it with Map;
- D179 ordinary Object/execution-context boundaries;
- D183 source-form preservation and formatter authority.

No current decision must be reopened by Candidate A.

## Named state and keyed/indexed state remain orthogonal

Current Protos already permits a standard Map value to carry ordinary named
slots and keyed associations simultaneously. Those states remain separate.

Conceptually:

~~~protos
m.description: "users"
m["description"] = someUser
~~~

The member operation and indexed operation are not aliases.

Named slots own member/delegation/method-binding/reflection/composition
semantics. Normal Map associations own arbitrary-key hash/equality,
representative-key, recorded-hash, insertion-order and keyed-matching semantics.

IdentityMap and Array have different additional state laws again.

A hypothetical universal state abstraction would therefore need to parameterize
key domain, lookup, delegation, equality, hashing, create/replace/remove,
iteration, reflection, composition, matching, transfer and mutation-state
behavior. D186 finds no present requirement that justifies turning that policy
product into a new public Protos institution.

## Pay-as-you-grow result

The investigation explicitly considered lazy keyed state on arbitrary Objects.

Current implementation evidence demonstrates that lazy physical state is
feasible: ordinary slot storage and several collection representations already
use pay-as-you-grow techniques.

That removes one weak objection to generalized keyed state, but it does not
remove the semantic cost.

Making arbitrary Objects eligible for keyed state would require permanent rules
for:

- acquisition of the state;
- selected key law;
- standard-family membership;
- reflection;
- composition;
- matching;
- close/freeze;
- Actor and P transfer;
- display/tool projection; and
- interaction with custom at/atPut behavior.

D186 therefore distinguishes physical pay-as-you-grow from semantic
pay-as-you-grow. Candidate A keeps both small.

## Prior-art conclusion

The research compared prototype/object and dictionary models across Self,
Smalltalk/Pharo, Io, JavaScript, Lua, Python, Ruby, and Elixir/Erlang.

The comparison showed two coherent broad families:

1. systems such as Lua that achieve deep unification by making named access a
   projection of keyed table access; and
2. systems that preserve a language-level distinction between named
   object/property/slot state and arbitrary-key dictionary/map state even when
   implementation representations can share machinery.

Protos currently depends on member/index, delegation, write, reflection,
composition and method-binding distinctions that align with the second family.
Adopting Lua-style unification would therefore be an object-model redesign, not
a punctuation simplification.

## Candidate result

### Candidate A — current model

**Selected.**

It preserves every current invariant, requires no migration, leaves future
internal representation convergence open, and keeps the name-versus-expression
distinction locally visible.

### Candidate B — syntactic convergence only

Rejected for now.

Any one-brace family still needs a local discriminator for named slots versus
evaluated keys, and empty braces remain unable to infer a collection family
without another rule. Retaining both old and new spellings would create
duplicate institutions.

### Candidate C — optional keyed-state capability on arbitrary Objects

Rejected for now.

Lazy storage is feasible, but no existing operation defines how an arbitrary
Object acquires keyed state, which key law it receives, or whether that grants
standard Map-family semantics. Every plausible acquisition mechanism adds a new
public institution or changes existing indexing/delegation behavior.

### Candidate D — generalized object-state model

Rejected.

A public abstraction broad enough to cover named slots, normal Maps,
IdentityMaps and Arrays must encode most of their existing differences as
policies. That relocates complexity rather than deleting it.

### Candidate E — factory/context-directed braces

Rejected.

Existing parent-expression syntax such as Map { ... } already constructs an
ordinary child Object delegating to Map. Repurposing it would break current
delegation/object-construction semantics; another explicit marker merely moves
the present discriminator.

### Candidate F — expected-kind/contextual inference

Rejected.

Protos has no expected-container type flow that can locally classify
foo({ ... }), return { ... }, or other dynamically consumed expressions.
Adding such a mechanism only to remove one punctuation marker is
disproportionate.

### Candidate G — explicitly mixed Object state

Rejected as a distinct need.

Standard Map values can already carry ordinary named Object slots together with
keyed state. The remaining proposal — giving arbitrary Objects keyed state —
reduces to Candidate C.

### Defer / no decision

Not selected.

The evidence is sufficient to affirm the current boundary rather than leave the
question unresolved.

## Strongest argument against Candidate A

Candidate C could make arbitrary Objects lazily acquire keyed state and might
enable a more uniform internal representation.

That becomes compelling only if real Protos programs need one arbitrary
ordinary Object identity to gain arbitrary-key state and using a Map or a
Map-valued slot is materially inadequate.

No such present requirement was found.

## Reconsideration trigger and escape path

A future Dxxx may reconsider generalized keyed state if concrete programs
demonstrate that requirement.

Any future proposal must still define, explicitly and independently:

- the acquisition operation;
- key policy;
- family membership;
- reflection and composition;
- matching;
- transfer/copy semantics;
- close/freeze behavior;
- tooling projection; and
- migration/source compatibility.

Candidate A does not foreclose that future work. It merely refuses to
pre-implement it.

## Ratification and follow-up

The project owner explicitly approved Candidate A on 2026-10-05. Approval was
mirrored to guillermomolina/protos#802 in issue comment 5999841611.

Because Candidate A has no observable semantic, syntax, specification or
implementation delta, D186 requires no implementation Ixxx and no normative
reconciliation slice.

The only durable effect of D186 is this rationale/decision record and its
research/ratification evidence.

~~~text
D186_STATUS=RATIFIED
SELECTED_CANDIDATE=A
OBSERVABLE_SEMANTIC_CHANGE=NO
SYNTAX_CHANGE=NO
SPECIFICATION_CHANGE=NO
IMPLEMENTATION_CHANGE=NO
D136_REOPENED=NO
FOLLOW_UP_IMPLEMENTATION=NONE
~~~
