# D186 — Object/Map construction symmetry research and ratification evidence

## Status

~~~text
FORMAL_IDENTIFIER=D186
ISSUE=guillermomolina/protos#802
SELECTED_CANDIDATE=A
OWNER_APPROVAL_DATE=2026-10-05
OWNER_APPROVAL_COMMENT=5999841611
PROTOS_REVISION=00526bdeffaec4360f2ed2deed3d59e16ae3a431
PROJECT_RECORD_BASE=b14cf085daa7788d4990622e6923fea5b266c4b1
RELATED_PRIOR_DECISION=D136/#542
OBSERVABLE_SEMANTIC_CHANGE=NO
IMPLEMENTATION_AUTHORIZED=NOT_REQUIRED
~~~

This file retains the investigation and ratification evidence for D186. It is
non-normative. Observable Protos semantics remain authoritative under
guillermomolina/protos:spec/**.

## Research question

D186 tested whether the visual symmetry:

~~~protos
{
    name: value
}

%{
    key: value
}
~~~

can be taken further by making Map a more direct special case of Object,
removing the percent discriminator, or allowing arbitrary Objects to acquire
keyed/indexed state.

The investigation was required to distinguish syntax cleanup from object-model
change and to falsify every serious candidate against current Protos semantics.

## Authority and current-state reconciliation

The final research pass was reconciled against the then-current Protos main
revision:

~~~text
00526bdeffaec4360f2ed2deed3d59e16ae3a431
I054: reconcile publication metadata
~~~

The immediately preceding substantive source revision used during the detailed
semantic/runtime audit was:

~~~text
9bed87430f99f3df8c2c3112f5a51b429a6e3176
I054: retire generic GraalVM dynamic-LSP test evidence (#654)
~~~

The delta to 00526bde modifies only pom.xml and CHANGELOG.md and explicitly
contains no language/specification/runtime semantic change. The investigation
therefore remains valid against the ratification-time Protos HEAD.

The project-record repository was re-established before publication at:

~~~text
b14cf085daa7788d4990622e6923fea5b266c4b1
~~~

## Documentation and policy audit

The investigation systematically audited the complete current maintained
Protos documentation surface required by #802.

Material authority included:

- root AGENTS.md;
- AGENTS.work/DESIGN.md;
- AGENTS.work/REFERENCE.md;
- AGENTS.work/COORDINATION.md;
- AGENTS.work/DOCUMENTATION.md;
- spec/AGENTS.md;
- src/AGENTS.md;
- protos/AGENTS.md;
- all current maintained files under spec/;
- all current maintained Markdown documentation under docs/;
- current root README.md, CONTRIBUTING.md, CHANGELOG.md and ROADMAP.md;
- material durable project records including D130, D131, D136, D140, D142,
  D143, D147, D154, D160, D168, D179 and D183; and
- PERF025 pay-as-you-grow object/collection evidence.

No current normative or owner-ratified record requires arbitrary Objects to
acquire keyed state.

No authority conflict was found that requires choosing implementation over
specification.

## D136 reconstruction

The D136 decision history was re-read because D186 directly re-evaluates the
boundary that made percent a keyed-domain discriminator.

The retained history shows that percent was not selected as arbitrary
punctuation.

D136 considered and falsified alternatives including:

- flat alternating Map(k1, v1, k2, v2, ...) construction;
- public Association/Pair-like construction items;
- alternative association punctuation;
- one-call Map(Association(...), ...) lowering;
- repeated post-construction atPut as semantic authority; and
- arbitrary custom Map.call participation.

Candidate F-prime was finally ratified with construction-time initial
association definition, normal Map hash/equality search, duplicate initial
definition failure, standard factory eligibility, and no public Association
institution.

D186 Candidate A preserves that complete boundary.

## Current semantic model found

The investigation found that the useful conceptual unification is already
present:

~~~text
Object
    identity
    immutable delegation parent
    local named-slot state
    OPEN / CLOSED / FROZEN mutation state

standard Map
    Object
    + normal equality-keyed association state

standard IdentityMap
    Object
    + semantic-identity-keyed association state

standard Array
    Object
    + dense positional indexed state
~~~

Current Protos therefore already allows standard collection values to retain
ordinary Object behavior and ordinary named slots while separately owning their
collection state.

What current Protos does not provide is generic state-domain acquisition by an
arbitrary Object.

## Key falsification: name versus expression

The decisive source distinction is not merely the result container.

Examples:

~~~text
{ a: 1 }
    a is a slot name

%{ a: 1 }
    a is evaluated as an expression producing the key

{ "a": 1 }
    syntax error as an ordinary slot-creation target

%{ "a": 1 }
    valid String key expression

{ foo.bar: 1 }
    creates member slot bar on the evaluated receiver foo

%{ foo.bar: 1 }
    reads/evaluates foo.bar and uses that result as the key

{ foo(): 1 }
    syntax error as a slot-creation target

%{ foo(): 1 }
    invokes foo() and uses its result as the key
~~~

A one-brace syntax cannot erase this distinction. It must represent the same
information through another local marker, contextual expected-kind machinery,
or a change to colon/member/index semantics.

## Interaction falsification summary

Candidate families were attacked against the full required interaction set.

### Object model

Generalized keyed state needs explicit rules for acquisition, semantic-family
membership, delegation, local reflection, composition, close/freeze and removal.

Delegation to Map cannot grant Map state without contradicting the existing
distinction between delegation topology and semantic-family membership.

### Keyed/indexed collections

Normal Map, IdentityMap and Array do not differ only by storage representation.

Normal Map owns observable hash/directed-equality behavior, representative keys,
recorded hashes and insertion order.

IdentityMap uses primitive semantic identity authority rather than ordinary
hash/equals dispatch.

Array owns dense bounded positional semantics rather than arbitrary-key
association semantics.

A generalized public state mechanism would therefore have to retain those
different laws as policies.

### Parsing and syntax

The existing parent-expression form:

~~~protos
Map {
    name: "Rex"
}
~~~

already means ordinary Object construction with Map as delegation parent.

Factory-directed brace reinterpretation cannot reuse that spelling without a
compatibility break.

Expected-kind inference cannot classify ordinary dynamic calls and returns
without installing a new language mechanism.

### Evaluation and control

D136 construction is not equivalent to ordinary repeated atPut.

Each reached initial association evaluates key then value exactly once and then
performs normal Map search/initial definition. Hash/equality callbacks may
execute guest code, signal Error, suspend, or transfer control. Duplicate
initial definitions remain Error rather than value replacement.

Any new syntax must preserve those observable boundaries.

### Matching

D131 structural Map and Array matching is protocol-first but relies on the
semantic state of standard collection receivers.

Allowing arbitrary Objects to acquire keyed state would require an explicit
answer to whether they become eligible standard Map matchers or merely expose an
unrelated indexing protocol.

### Concurrency and transfer

Ordinary Object transfer is defined from the Object graph of local slots and
parent edges, while standard collection transfer preserves the applicable
collection state and key/index law.

Generalized keyed state would therefore require transfer/copy semantics to
become part of the general Object model or to remain a separately tagged state
domain. Either way, the distinction reappears.

### Modules, Standard Library and data models

Repository usage confirms intentional use of both state models.

Ordinary record/configuration/specification values commonly use named Object
slots.

Dynamic key domains use Map or IdentityMap in JSON, TOML, CLI, package and Test
Tool code.

D168 independently demonstrates the same design discipline: Process arguments
were reduced to ordinary frozen Array where semantics truly coincide, while
Environment-to-Map substitution was explicitly rejected because its native-name
semantics differ.

### Tooling

The current parser, Surface AST, canonical AST, source-layout projection,
formatter/static analysis and execution pipeline represent Map construction
explicitly.

This is implementation evidence rather than semantic authority, but it proves
that a syntax change has real compatibility/tooling cost and is not a
zero-cost spelling alias.

## Implementation reality map

At the audited revision, implementation evidence includes:

- ProtosObjectValue;
- ProtosMapValue;
- ProtosIdentityMapValue;
- ProtosArrayValue;
- ProtosStandardMapProtocol;
- ProtosStandardIdentityMapProtocol;
- ProtosStandardArrayProtocol;
- SurfaceMapConstruction;
- CanonicalMapConstruction;
- Canonicalizer Map-construction lowering;
- CanonicalToBytecodeLowerer Map-construction execution;
- parser source facts/static definitions/static references/document symbols;
- source-layout projection and formatter Map nodes; and
- focused Map/IdentityMap/Array/matching/transfer construction tests.

The separate Java classes are not language authority and D186 does not freeze
that representation.

A future implementation may internally factor lazily allocated state components
while preserving the selected observable semantics.

## Pay-as-you-grow evidence

PERF025 demonstrates that empty ordinary Object slot storage can remain
storage-free and promote only on first use, and that collection snapshot/index
representations can likewise be specialized lazily.

Therefore Candidate C was not rejected on the simplistic basis that every
Object must eagerly allocate a HashMap.

It was rejected because semantic acquisition creates a permanent rules surface
even if physical allocation is lazy.

## Prior-art comparison result

The required survey included Self, Smalltalk/Pharo, Io, JavaScript, Lua, Python,
Ruby, and Elixir/Erlang.

The evidence showed:

- Lua demonstrates that deep object/table unification is coherent when named
  member access is fundamentally table-key access;
- JavaScript demonstrates that syntactic convergence/computed property names can
  coexist with a separate arbitrary-key Map;
- Self and Io show prototype/object systems do not require every ordinary object
  to expose arbitrary keyed collection semantics;
- Smalltalk/Pharo, Python and Ruby preserve strong object-state versus
  dictionary/mapping distinctions; and
- Elixir/Erlang show that representation sharing or map-centric data does not
  erase stronger semantic domains automatically.

The comparison therefore supplies precedent for multiple coherent models rather
than authority for one punctuation style.

## Candidate matrix

~~~text
A  STATUS QUO / CLARIFY CURRENT MODEL
   SURVIVES
   SELECTED

B  SYNTAX CONVERGENCE ONLY
   REJECTED FOR NOW
   reason: discriminator moves elsewhere; empty braces/context remain unresolved

C  OPTIONAL KEYED STATE ON ARBITRARY OBJECTS
   REJECTED FOR NOW
   reason: acquisition/family/reflection/composition/matching/transfer rules added

D  UNIFIED GENERALIZED OBJECT STATE
   REJECTED
   reason: existing distinctions become a large policy product rather than disappear

E  FACTORY/CONTEXT-DIRECTED BRACES
   REJECTED
   reason: Map { ... } already has incompatible parent-object meaning

F  EXPECTED-KIND / CONTEXTUAL INFERENCE
   REJECTED
   reason: new non-local classification mechanism disproportionate to the problem

G  EXPLICIT MIXED OBJECT STATE
   REJECTED AS DISTINCT NEED
   reason: standard Map already mixes ordinary named slots and keyed state;
           arbitrary-Object acquisition reduces to C

DEFER
   NOT_SELECTED
   reason: evidence is sufficient to affirm the current boundary
~~~

## GITHUB010 result

The complete investigation scorecard favored A across correctness,
Protos-alignment, present-need proportionality, incremental growth,
future-option resilience, scalability, conceptual simplicity, portability,
runtime cost, operability, reversibility, evidence maturity, source locality,
semantic transparency, parser/tooling simplicity, compatibility and
pay-as-you-grow quality.

Candidate C scored best only on maximal surface object-model uniformity, but
that uniformity is obtained by expanding the semantic definition of Object
rather than deleting enough rules to justify the expansion.

No arithmetic average was allowed to hide the severe acquisition/family-state
red flags.

## Strongest counterargument and regret scenario

The strongest counterargument remains Candidate C:

one arbitrary Object identity could lazily grow true keyed state, potentially
making some future programs and internal representations more uniform.

The regret scenario is a future body of real Protos code that repeatedly needs
that exact capability and finds both a separate Map value and a Map-valued slot
materially awkward.

The escape path remains clean: allocate a future Dxxx and define the acquisition
operation plus all affected state/family/transfer/reflection/matching rules
explicitly. Candidate A does not reserve or foreclose that future syntax.

## Owner approval

The project owner explicitly approved Candidate A in the active D186
interaction on 2026-10-05 with:

~~~text
Apruebo A.
~~~

The approval was mirrored to authoritative issue #802 as comment 5999841611.

Approved invariant result:

~~~text
D136_INVARIANTS_PRESERVED=ALL
OTHER_RATIFIED_INVARIANTS_REOPENED=NONE
OBSERVABLE_SEMANTIC_DELTA=NONE
SYNTAX_DELTA=NONE
SPECIFICATION_DELTA=NONE
IMPLEMENTATION_DELTA=NONE
~~~

## Closure result

Because the selected candidate is the existing semantic model, no product
change, normative specification reconciliation, version bump, test execution or
implementation work item is required by D186.

The durable publication consists of:

- the D186 language decision record; and
- this research/ratification evidence.

The authoritative GitHub issue is closed only after these files and their
navigation entries are published, re-read at the exact project-record revision,
and the cross-references are verified.

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
SPECIFICATION_RECONCILIATION=NOT_REQUIRED
IMPLEMENTATION_RECONCILIATION=NOT_REQUIRED
FOLLOW_UP_IMPLEMENTATION=NONE
NEXT_TECHNICAL_SLICE=NONE
~~~
