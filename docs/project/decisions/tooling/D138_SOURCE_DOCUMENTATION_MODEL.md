# D138 — Source documentation model

Status: **RATIFIED — Candidate A′ selected**

Allocated: **2026-09-17**

Explicit project-owner approval: **2026-09-18**

Decision issue: `guillermomolina/protos#548`

Nature: durable implementation-independent source-documentation/tooling contract

Research baseline: `guillermomolina/protos@b3faad11322c5b436235cdfd0134c85f44d391a1`

Normative Core language effect: **none**.

Runtime/Standard Library semantic effect: **none**.

## Problem

Protos needs documentation authored beside source that tools can associate
deterministically with meaningful program entities without turning documentation
into runtime state, a nominal type system, a second parser, an API-publication
policy, or a universal symbol graph.

The existing D061/D062/D064/D067 work solved a narrower Standard Library API
publication problem. D138 generalizes source documentation while preserving the
parts of those decisions that remain valid.

The design must distinguish:

```text
ordinary source comment
    !=
source documentation owned by a source entity
    !=
mechanically/provably derived source structure
    !=
downstream generated/presented documentation
```

## Selected candidate — A′ source-local documentation

D138 selects a deliberately small source-local model.

```text
documentation occurrence
        +
authoritative Surface AST
        |
        v
deterministic source-local association
        |
        v
module source documentation
or named slot-creation source documentation
```

No runtime documentation object, global documentation registry, second parser,
durable local-symbol identity system, background index, type language, or
documentation-specific semantic graph is introduced.

## Documentable owners

The initial independent source-documentation owners are exactly:

```text
module source unit

explicit named SurfaceSlotCreation occurrence
    bare target or member target
    top-level or nested
    independent of initial value shape
```

Examples:

```protos
//! JSON construction and parsing utilities.

/// Parses one complete JSON document from `text`.
parse: (text) => {
    /// Converts an ASCII hexadecimal digit to its numeric value.
    hexDigit: (octet) => {
        ...
    }
}
```

`parse` and `hexDigit` are distinct source-documentation owners. The fact that
their initial values are Closure literals does not recategorize them as
Function/Method entities.

### Not independent baseline owners

The following do not initially acquire independent authored-documentation
ownership:

- Closure parameters;
- anonymous Closures;
- anonymous objects;
- assignments;
- delegated/inherited exposures;
- composition-contributed exposures; or
- runtime objects/values.

Parameters and other mechanically known structure may still be described as
facets inside the owning slot's authored documentation.

## Source ownership, not runtime ownership

Documentation belongs to the exact source occurrence that creates the named
slot. It does not automatically follow runtime value identity.

```protos
primary: service
secondary: primary
```

`primary` and `secondary` may have different documentation even when both slots
contain the same runtime object because the source bindings can represent
different roles/contracts.

Likewise:

```protos
findTarget().timeout: 30
```

has one source-documentation owner even if repeated execution creates `timeout`
slots on different runtime receiver objects.

Authored documentation does not automatically transfer through:

```text
assignment
value aliasing
delegation
composition
Closure extraction
repeated execution
same-name coincidence
runtime object identity
```

A downstream tool may present existing documentation through an independently
proven relation, but that relation does not manufacture a new authored owner.

## Structure versus authored prose

D138 distinguishes three information classes:

```text
SOURCE FACT
    directly observable in the current source snapshot

PROVEN FACT
    established exactly by an applicable static-analysis authority

AUTHORED CONTRACT
    human meaning/intention not expressed by structure alone
```

Tooling owns mechanically derivable structure such as, when applicable:

- slot name;
- source span/location;
- source containment/nesting;
- module/source identity when independently known;
- initializer source shape;
- Closure parameter/default/rest shape when directly observable; and
- exact origin/reference/callable relations when another static authority proves
  them.

Authors may describe semantic information such as:

- purpose and role;
- parameter meaning;
- accepted semantic domain/protocol expectations;
- result meaning;
- observable effects and state changes;
- failures/errors;
- authority/capability requirements;
- lifetime/ownership/concurrency constraints;
- invariants and special cases;
- relevant complexity; and
- examples.

Mechanically observable structure is not automatically a durable contract. In
particular, a slot whose initial value is a Closure may later be assigned another
value under ordinary Protos semantics.

Authored prose may state dynamic expectations such as "`text` must be a semantic
String", but baseline documentation is not interpreted as a type/specification
language. D138 introduces no mandatory `@param`, `@return`, `@type`, `@throws`,
`@effects` or equivalent structured contract vocabulary.

## Authoring markers

D138 retains the D062 marker spellings:

```text
//!  module documentation
///  named slot-creation documentation
```

Both remain ordinary Protos line comments lexically and semantically. Their
documentation meaning exists only for documentation/source tooling.

Removing documentation therefore leaves ordinary Protos execution semantics
unchanged.

### Module documentation

A contiguous `//!` block in the module preamble documents the module source
unit. It must occur before the first executable/source construct. At most one
module-documentation block is accepted.

Ordinary preamble comments may precede `//!`. No blank line is semantically
required between ordinary preamble comments and the module documentation block:

```protos
// ordinary/legal preamble
//! Module documentation.
```

and:

```protos
// ordinary/legal preamble

//! Module documentation.
```

have the same documentation meaning. A formatter/linter may prefer visual
separation, but presentation whitespace must not create additional module-doc
semantics.

### Slot documentation

A contiguous `///` block documents the exact following documentable named
`SurfaceSlotCreation` occurrence.

The association is local, preceding, deterministic and fail-closed. A blank
logical line, an unrelated ordinary comment between the doc block and target, or
an unrelated source construct breaks association. Documentation tooling reports
invalid placement rather than searching by name or guessing another target.

Pure grouping syntax around the documented slot-creation expression may be
transparent for association, preserving the already-supported shape:

```protos
/// Calls with `value`.
(call: (value) => value)
```

but grouping does not become a documentation owner.

A `///` marker before an assignment, anonymous Closure/object or other
non-documentable construct is a documentation-validation error when
documentation tooling is invoked. It remains an ordinary comment for Protos
execution.

D138 selects only preceding documentation. It does not add trailing-documentation
syntax.

## Documentation payload

The portable authored payload uses **CommonMark 0.31.2**.

This selects a common textual baseline for source documentation consumers. It
does not select website styling, HTML sanitization policy, renderer layout, or
other presentation behavior.

Ordinary CommonMark links and code spans are allowed.

```protos
/// See the [format specification](https://example.org/spec).
/// Uses `TOML.parse`.
```

`TOML.parse` in the second example is initially authored code text, not a
semantic cross-entity reference.

## Cross-entity references

D138 deliberately defers Protos-symbol-aware documentation references.

A later facility may resolve documentation references against authoritative
module/source/symbol identity, but it must not silently inherit runtime
`import()` resolution semantics merely because a documentation spelling resembles
a module specifier.

This keeps D138 compatible with D137/CLI009 local-source import resolution,
package resolution and Standard Library resolution without prematurely coupling
documentation links to one runtime resolver domain.

## Parser/source/tooling boundary

The smallest implementation model reuses existing authorities:

```text
ProtosLexer
    -> opt-in line-comment occurrences

ProtosParser
    -> authoritative Surface AST + SourceSpan

small source-documentation layer
    -> documentable occurrences
    -> deterministic association
    -> authored CommonMark payload
```

`ProtosDocumentSymbols` / D079 provides useful traversal evidence but is an LSP
projection, not documentation authority.

The baseline does not require:

- Truffle/runtime execution;
- runtime object identity;
- delegation resolution;
- D110/D124 analysis;
- package/release identity;
- Git revision identity;
- global indexing;
- D064 durable IDs for locals; or
- a documentation graph/registry.

## Source occurrence identity and downstream projections

The fundamental source-documentation owner is an occurrence in the current
source snapshot.

`SourceSpan` locates the occurrence in that snapshot; it is not an eternal
documentation ID.

Different consumers may project the same source information only when they need
stronger identity:

```text
current-source editor
    -> Surface AST occurrence / SourceSpan

durable published Standard Library API
    -> D064 SymbolIdentity / SymbolOccurrenceKey

static navigation/presentation
    -> D110/D124 proof where applicable
```

This is a pay-for-what-you-need boundary: local source documentation does not
pre-pay for cross-release identity.

## Documentable is not publishable

Source documentability and API publication are independent concerns.

A nested/local helper may have useful source documentation without becoming a
published Standard Library API entry. D067 remains the authority for the
Standard Library documentation coverage/publication domain it selected.

## Comparative research

The decision followed a broad comparison spanning materially different
documentation models.

### Self

Self attaches tool-facing annotations/comments to objects and slots. This is
highly relevant to Protos' prototype/slot model, but Self's live-image annotation
institution is larger than required for file/source-local documentation.

### Io

Io distinguishes documentation comments such as `//doc`/`//metadoc` from
ordinary comments and associates them with protos/slots. It is the closest
prototype-language precedent for an explicit tooling-oriented documentation
marker while keeping slot-centric ownership.

### Smalltalk / Pharo

Smalltalk-family environments keep documentation close to class/method/object
entities and emphasize explaining use, result and side effects rather than
paraphrasing implementation. Their image/browser storage model is not adopted.

### Go

Go demonstrates that source-adjacent documentation plus mechanically known
declarations can serve CLI, web and language-server consumers. Ordinary `//`
proximity is less suitable for Protos because Protos source already contains
legal/implementation comments and lacks Go's exported-declaration distinction.

### Rust and Zig

Rust and Zig provide strong precedent for `///` following-item documentation and
`//!` containing/module documentation. Rust internally lowers docs to attributes;
D138 borrows the compact marker distinction without adopting an attribute
institution.

### Java, C#, Swift and Dart

These ecosystems demonstrate declaration-associated source documentation at
large scale, with compiler/analyzer structure supplying names/signatures and
documentation tooling supplying presentation. Swift/DocC additionally
demonstrates the value of a richer downstream symbol/publication model without
requiring that model to be the source-authoring primitive.

### Python and Julia

Docstrings/runtime documentation provide excellent interactive discovery but
make documentation a runtime/language metadata institution. D138 rejects that
cost for the current source-tooling requirement. Python's guidance that prose
should explain arguments, results, side effects and errors without redundantly
restating introspectable signatures remains useful content evidence.

### Elixir / Erlang

Elixir's `@doc`/`@moduledoc` and Erlang's compiled documentation facilities
demonstrate a scalable structured/compiled metadata model. They also show the
qualitative institution cost D138 avoids initially.

### JavaScript / TypeScript and JSDoc

JSDoc demonstrates both source-local documentation and the risk of documentation
becoming a parallel static type authority through `@type`, `@param`, `@returns`
and related tags. D138 deliberately keeps baseline prose non-authoritative for
static typing/contracts.

### Ruby / RDoc

RDoc shows that a dynamic language can associate source comments with
definitions without executing the program, supporting source-oriented rather
than runtime-oriented extraction.

### Racket / Scribble

Racket demonstrates powerful external binding-aware documentation and
co-located source-doc options. Its explicit binding/linkage machinery is strong
evidence for a future sidecar/cross-reference layer, but would be speculative
overhead for D138's baseline.

## Candidate space

The final comparison retained five materially different architectures:

### S — D062/API-oriented status quo

Module + top-level slot documentation only, backed directly by the durable
Standard Library API model.

Rejected as the general source-documentation model because the top-level
restriction reflects publication scope, not a fundamental Protos source-entity
boundary.

### A′ — source-local module + named slot occurrences

Selected.

Documentation ownership is source-local; `//!`/`///` are tooling-only markers;
the existing Surface AST supplies structure; stronger identities are downstream
projections only when required.

### M — structured metadata/annotation model

Strong deterministic association and future extensibility, but introduces a new
metadata institution and user/tooling surface without a current requirement.

### R — runtime/value-attached documentation

Powerful for reflection/REPL discovery, but creates aliasing, rebinding,
composition, lifetime, storage/stripping and Actor/runtime questions unrelated
to the motivating source-documentation requirement.

### X — external/model-first documentation

Sidecar/binding-aware or graph-first documentation provides excellent long-form
and cross-link capability but requires identity/linkage machinery before the
baseline source-local need demonstrates it.

## Comparative scoring

Scores are 1–5. `H` = high confidence, `M` = medium confidence. Each cell
contains the score/confidence and the principal justification. Arithmetic totals
are not decision authority.

| AGENTS criterion | S status quo | **A′ source-local** | M metadata | R runtime | X external/model-first |
| --- | --- | --- | --- | --- | --- |
| Correctness / invariants | 4/H — safe but artificially top-level | **5/H — exact AST/source ownership** | 5/H — explicit association | 3/M — source/runtime ownership conflicts | 4/H — exact with linkage |
| Protos alignment | 4/H — simple but API-shaped | **5/H — module/slot, no new semantic entity** | 3/H — new institution | 2/H — runtime pollution | 3/H — external identity machinery |
| Present-need proportionality | 5/H — cheap | **5/H — only current source need** | 2/H — metadata not needed | 1/H — runtime facility not needed | 2/H — IDs/linkage anticipated |
| Incremental growth | 3/M — nested growth changes model | **5/H — owner kinds/features add locally** | 5/H — extensible | 3/M — runtime model constrains growth | 4/H — extensible but heavier |
| Future-option resilience | 3/M — API identity overcommitted | **5/H — preserves runtime/types/links choices** | 5/H — broad metadata | 4/M — runtime choice constrains storage | 5/H — flexible |
| Scalability | 4/H — good for public API | **5/H — source-local linear traversal** | 5/H — proven | 3/M — runtime/lifecycle cost | 5/H — proven |
| Conceptual simplicity | 5/H — very small | **5/H — module + slot + prose** | 3/H — metadata ontology | 2/H — lifetime/copy rules | 2/H — identity/link layer |
| Portability / implementation freedom | 5/H — source-based | **5/H — parser/source based** | 4/H — compiler integration | 3/M — runtime-dependent | 5/H — implementation-neutral |
| Runtime / resource cost | 5/H — zero runtime | **5/H — zero runtime when unused** | 5/H — tooling/compiler time | 2/H — storage/runtime cost | 5/H — tooling time |
| Failure / operability | 4/H — deterministic top-level | **5/H — local fail-closed association** | 5/H — explicit metadata validation | 3/M — lifecycle ambiguity | 4/H — broken IDs/links diagnosable |
| Deferral / reversibility / migration | 3/M — nested support needs scope change | **5/H — richer layers add without ownership migration** | 3/M — difficult to retract public metadata | 2/M — runtime contract hard to retract | 3/M — durable identity migration cost |
| Evidence maturity / implementation risk | 5/H — already implemented | **5/H — broad mature precedent + existing lexer/AST** | 5/H — mature ecosystems | 5/H — mature ecosystems | 5/H — mature ecosystems |

### Non-compensating red flags

`M`, `R` and `X` carry overengineering red flags because their additional
institutions solve plausible future problems rather than demonstrated baseline
requirements.

`S` carries an underengineering red flag: its top-level restriction conflates
Standard Library API publication with source-documentation ownership and blocks
useful nested source documentation even though Protos already recognizes nested
named slot creations structurally.

A′ avoids both red flags.

## Incremental-design / anti-overengineering gate

### Pay for what you need

Current users/project code pay only for documentation-comment observation, the
already-authoritative Surface AST, and deterministic association when
documentation tooling is invoked. Ordinary runtime execution pays no
documentation cost.

### Grow as you need

Future parameter owners, structured metadata, semantic documentation links,
runtime reflection or additional documentable source kinds can be added
separately if evidence requires them. Their absence today does not invalidate
the source-occurrence ownership model.

### Cost of deferral / reversibility

Deferring those capabilities has bounded cost:

- parameters already have source structure and `SourceSpan`;
- cross-entity links can later consume independently authoritative identity;
- D064 already exists for durable public/API identity;
- a runtime documentation facility can later project source docs if a real
  reflection/REPL requirement appears; and
- publication policy remains independent.

No deferred capability requires changing runtime identity, persistence, package
format, source execution semantics or the fundamental documentation owner model.

### Smallest sufficient solution

The smallest sufficient design is:

```text
module source unit
+
named SurfaceSlotCreation occurrence
+
authored CommonMark prose
+
existing lexer/parser/AST
+
deterministic source-local association
```

### Speculation burden

D138 preserves options rather than preimplementing them.

```text
future-compatible       YES
future-preimplemented   NO
```

## Failure modes and counterexamples

### Duplicate names / shadowing

Two same-named slot creations in different or identical lexical regions remain
distinct source occurrences. Documentation is never associated by same-name
search.

### Nested helper

```protos
outer: () => {
    /// Local cache.
    cache: Map()
}
```

`cache` is documentable even though it need never appear in a published API
reference.

### Assignment

```protos
strategy: normal
strategy = emergency
```

Documentation belongs to `strategy:`. The assignment is an operation/occurrence,
not a new documentation owner. An ordinary comment can explain the assignment.

### Delegation

If `dog.speak` resolves to a defining `speak:` occurrence through delegation, a
tool may display that source documentation only when the relation is independently
proven. D138 does not copy authored documentation onto `dog`.

### Composition / aliases

Composition or alias operations may expose a slot name without an explicit new
`name:` creation occurrence at the exposure site. D138 does not create authored
owners for such derived exposures. Future explicit exposure-documentation is
additive if a real need appears.

### Source movement

Moving a documented local slot changes its current `SourceSpan`; the comment
moves with the source occurrence. No global DocumentationId update is required.

### Parameter prose

A parameter may need explanation without becoming a documentation owner. The
owning slot documentation can explain the parameter by its mechanically known
name.

## Strongest counterarguments and regret paths

The strongest counterargument is that parameters, derived aliases/exposures or
cross-symbol links may eventually need machine-readable identity and structured
metadata.

If that need appears, A′ may look initially too small. The recovery path is
bounded, however: those facilities can add a target kind, relation or downstream
identity layer without changing the meaning of existing module/slot
documentation.

Another risk is that allowing nested documentation could create too much
editor/publication noise. The recovery is policy-level rather than ownership
migration: editors and publishers may choose which documentable occurrences to
surface. D067 remains separate evidence that publication and documentation
coverage are independent dimensions.

## Existing-decision reconciliation

### D061 — partially narrowed

Retained:

- mechanical facts are separate from authored semantic prose;
- generated presentation is downstream;
- canonical source/documentation authority remains in `guillermomolina/protos`;
  and
- durable neutral artifacts remain appropriate where actual consumers require
  them.

Narrowed by D138:

- one versioned neutral model is not a mandatory universal internal
  representation for every current-source consumer.

A source-local editor/tool may use AST occurrence + prose directly; D064
projection is required only when its durable publication/identity properties are
actually needed.

### D062 — partially superseded

Retained:

- `//!` and `///` spellings;
- ordinary-comment/documentation distinction;
- tooling-only semantics;
- preceding deterministic association;
- fail-closed invalid placement; and
- no mandatory structural documentation tags.

Superseded:

- `///` is no longer top-level only; and
- nested `///` is no longer invalid solely because it is nested.

### D064 — retained with clarified scope

D064 durable identity remains valid for API/publication artifacts.

Its top-level `SymbolIdentity` is not generalized to every nested/local D138
source owner. D064 `Callable` is mechanical source-shape data, not a durable
nominal callable/type contract.

### D066 / D067 — retained

D066 earned-generality/documentation topology remains compatible.

D067 Standard Library coverage/publication policy remains intact. A source entity
being documentable does not make it a published API entry.

### D079 — retained

D079 remains the tooling decision demonstrating that explicit named
`SurfaceSlotCreation` occurrences are uniform source structural units. Its LSP
`DocumentSymbol` projection is not documentation authority.

### D110 / D124 — unchanged

Static definition/reference proof may later support exact documentation
presentation/navigation. D138 does not weaken their no-guessing boundary.

### D137 / CLI009 — unchanged

D138 selects no semantic cross-reference spelling and no documentation import
resolver. A future semantic documentation-reference mechanism must not silently
inherit direct-file/runtime `import()` resolution.

## Invariant consistency

```text
documentation does not change program semantics                 PASS
ordinary comments and documentation remain distinct            PASS
known structure need not be redundantly authored               PASS
documentation association is deterministic                     PASS
syntax follows ownership/model                                 PASS
module/slot small-universe model                               PASS
slot identity not value-shape Function/Method taxonomy         PASS
documentable != published API                                  PASS
source ownership != runtime object identity                    PASS
D064 durable publication identity preserved                    PASS
D067 publication policy preserved                              PASS
D110/D124 no-guessing static boundaries preserved              PASS
D137 runtime import-resolution boundary preserved              PASS
```

```text
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Exact selected contract

```text
D138_SELECTED_CANDIDATE=A_PRIME

DOCUMENTATION_OWNERS=MODULE_SOURCE_UNIT,NAMED_SURFACE_SLOT_CREATION
NAMED_SLOT_TARGETS=BARE_OR_MEMBER
NESTED_NAMED_SLOT_DOCUMENTATION=YES
INITIAL_VALUE_SHAPE_CHANGES_OWNER_KIND=NO

PARAMETER_INDEPENDENT_OWNER=NO
ANONYMOUS_CLOSURE_INDEPENDENT_OWNER=NO
ANONYMOUS_OBJECT_INDEPENDENT_OWNER=NO
ASSIGNMENT_INDEPENDENT_OWNER=NO
DELEGATED_EXPOSURE_INDEPENDENT_OWNER=NO
COMPOSED_EXPOSURE_INDEPENDENT_OWNER=NO
RUNTIME_VALUE_INDEPENDENT_OWNER=NO

OWNERSHIP=SOURCE_OCCURRENCE
RUNTIME_DOC_OWNERSHIP=NO
AUTOMATIC_DOC_TRANSFER_THROUGH_ALIASING=NO
AUTOMATIC_DOC_TRANSFER_THROUGH_DELEGATION=NO
AUTOMATIC_DOC_TRANSFER_THROUGH_COMPOSITION=NO

MODULE_DOC_MARKER=//!
SLOT_DOC_MARKER=///
MARKERS_HAVE_RUNTIME_SEMANTICS=NO
ORDINARY_COMMENTS_ARE_DOCS=NO
ASSOCIATION=PRECEDING_LOCAL_DETERMINISTIC_FAIL_CLOSED
TRAILING_DOCS=NO
GROUPING_MAY_BE_ASSOCIATION_TRANSPARENT=YES

MODULE_DOC_LOCATION=MODULE_PREAMBLE
MODULE_DOC_MAX_BLOCKS=1
BLANK_LINE_BEFORE_MODULE_DOC_REQUIRED=NO

AUTHORED_PAYLOAD=COMMONMARK_0_31_2
MANDATORY_STRUCTURED_TAGS=NO
DOCUMENTATION_AS_STATIC_TYPE_AUTHORITY=NO
SEMANTIC_PROTOS_DOC_REFERENCES=DEFERRED
RUNTIME_IMPORT_RULES_DEFINE_DOC_LINKS=NO

BASIC_TOOLING=EXISTING_LEXER_PLUS_SURFACE_AST_PLUS_SMALL_ASSOCIATION_LAYER
SECOND_PARSER=NO
GLOBAL_DOC_GRAPH=NO
DURABLE_ID_FOR_EVERY_LOCAL_OWNER=NO
RUNTIME_DOC_OBJECT=NO

DOCUMENTABLE_EQUALS_PUBLISHED_API=NO
D064_DURABLE_API_PROJECTION=RETAINED_WHEN_NEEDED
```

## Intentionally deferred

D138 does not select:

- parameters as independent documentation owners;
- machine-readable type/effect/error/authority contract tags;
- deprecation/stability metadata vocabulary;
- doctest/executable-example semantics;
- semantic Protos symbol-reference syntax inside documentation;
- documentation-reference resolution rules;
- formal sidecar Markdown-to-symbol linkage;
- runtime reflection/documentation APIs;
- documentation inheritance/copy through delegation/composition;
- publication/visibility policy beyond existing decisions;
- website/IDE rendering/layout;
- formatter/linter style beyond the semantic whitespace boundaries above; or
- new durable identity for nested/local source owners.

These remain additive future decisions if real requirements justify them.

## Approval provenance

The project owner explicitly approved **D138 Candidate A′** on **2026-09-18**
after the exhaustive source-entity, ownership, content, tooling and authoring
comparison.

The exact owner statement was:

> Aprobada A′

GitHub Issue `guillermomolina/protos#548` records the decision packet, follow-up
clarifications and approval event.

```text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Ratification closure contract

```text
D138_STATUS=RATIFIED
D138_SELECTED_CANDIDATE=A_PRIME

PROTOS_REVISION=b3faad11322c5b436235cdfd0134c85f44d391a1
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs

SPECIFICATION_CHANGED=NO
IMPLEMENTATION_CHANGED=NO
IMPLEMENTATION_VERSION_CHANGED=NO
STANDARD_LIBRARY_CHANGED=NO
RUNTIME_SEMANTICS_CHANGED=NO

D061_SCOPE=PARTIALLY_NARROWED
D062_SCOPE=PARTIALLY_SUPERSEDED
D064_SCOPE=RETAINED_AND_CLARIFIED
D066_SCOPE=RETAINED
D067_SCOPE=RETAINED

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

D138 ratification is governance/documentation-only.
