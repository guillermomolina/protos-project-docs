# D136 — Map populated construction model and source sugar

Status: **RATIFIED — Candidate E′ selected, sequence-layout amendment ratified**

Initial project-owner approval: **2026-09-16**
Sequence-layout amendment approval: **2026-09-16**
Decision issue: `guillermomolina/protos#542`
Source audit: `AUD011` / `guillermomolina/protos#540`
Protos baseline reviewed: `e8a74f29bcbb321c0308c932446ae3950824ed8f`

Nature: non-normative decision/rationale record for future normative grammar and
Map-construction semantics under `guillermomolina/protos/spec/`.

## Decision

Select **Candidate E′ — sequential keyed construction**.

The selected source form is:

```protos
%{}

%{
    key1: value1
    key2: value2
}
```

On one physical line, entries use the ordinary Protos sequence separator:

```protos
%{ key1: value1; key2: value2 }
```

A comma is **not** an entry separator in Map construction.

The form is a Map-producing primary expression. It is semantic source sugar for
constructing one fresh result through the existing ordinary `Map()` factory and
then performing ordinary keyed insertion for each source entry from left to
right.

Conceptually:

```protos
%{
    k1: v1
    k2: v2
}
```

has the same observable construction behavior as:

```protos
m: Map()
m[k1] = v1
m[k2] = v2
m
```

except that the source form creates no user-visible binding `m`, no Closure, no
hidden lexical scope, and no construction activation.

The selected design is intentionally **not** a lowering to one populated
`Map(...)` call.

## Ratified contract

1. **Ordinary `Map` lookup.** At the evaluation point of the construction
   expression, `Map` is resolved by ordinary identifier lookup. The syntax does
   not capture or bypass the standard-prelude binding.

2. **Exactly one factory invocation.** The selected `Map` value is invoked once
   with zero positional arguments before any source entry key or value expression
   is evaluated. If lookup or invocation fails, no entry expression is evaluated.

3. **No hidden construction context.** Entry expressions are evaluated in the
   same enclosing activation in which the `%{...}` expression appears. The new
   Map does not become `this`, `context`, a lexical parent, a return home, or a
   construction activation merely because it is being populated.

4. **Entry order is source order.** Entries are processed strictly left to right.
   For each entry, the key expression is evaluated exactly once, then the value
   expression exactly once, then ordinary keyed insertion is performed before
   evaluation proceeds to the next entry.

5. **Insertion uses the existing protocol.** Conceptually each insertion is the
   existing indexed-assignment operation:

   ```protos
   result[key] = value
   ```

   and therefore uses ordinary `atPut` selection/invocation. The implementation
   must not bypass custom receiver behavior merely because the operation
   originated from `%{...}`.

6. **Per-entry dispatch remains ordinary.** Each entry performs its own ordinary
   insertion dispatch. An implementation may optimize this only when observable
   behavior remains equivalent, including cases where receiver behavior changes
   between entries.

7. **Failure is fail-fast and non-transactional.** Any Error, non-local control
   transfer, lookup failure, invocation failure, `hash`/`==` failure, or insertion
   failure stops construction immediately. Later entry expressions are not
   evaluated. Effects and insertions already completed are not rolled back.

8. **Existing Map duplicate/equality semantics are authoritative.** Equal or
   duplicate keys have exactly the same behavior as sequential ordinary
   `atPut` operations. D136 defines no construction-specific duplicate-key rule
   and does not change representative-key, recorded-hash, insertion-order,
   equality, or replacement semantics.

9. **Empty construction.** `%{}` performs ordinary `Map` lookup and one zero-arg
   invocation and returns that result without any insertion.

10. **Shadowing is intentional.** A local or lexical binding named `Map` affects
    `%{...}` exactly as it affects the corresponding explicit `Map()` plus
    insertion sequence. There is no privileged canonical-Map constructor.

11. **Custom factory results compose through ordinary protocol.** If shadowed
    `Map()` returns an object that supports the ordinary indexed-assignment
    protocol, entries are applied to that object normally. If it does not, the
    construction fails at the same point the equivalent explicit operation would.

12. **`:` is contextual entry punctuation inside `%{...}`.** It separates the
    entry key expression from its value expression. It is not ordinary slot
    creation at that location and does not alter the meaning of `:` elsewhere.

13. **Key expressions are expressions, not implicit names or Strings.** For
    example:

    ```protos
    %{
        name: value
    }
    ```

    uses the value of the ordinary expression `name` as the key; it does not mean
    the String key `"name"`. A String key is written explicitly as `"name"`.

14. **Entry separation follows ordinary sequence layout.** Across physical lines,
    newline separates entries. On one physical line, `;` separates entries. The
    grammar does not use comma as a Map-construction entry separator.

15. **No trailing sequence separator.** A trailing `;` before `}` is not admitted,
    matching the ordinary object-body/sequence treatment rather than argument-list
    trailing-separator rules.

16. **Continuation after `:` follows ordinary continuation behavior.** A value may
    continue after the entry colon using the same continuation-newline rules that
    apply to ordinary Protos expression syntax. The continuation newline does not
    terminate the entry.

17. **Construction and object syntax remain distinct but intentionally related.**
    `{...}` is object construction/body syntax operating in the slot domain.
    `%{...}` is keyed Map construction syntax. Both are sequential container
    construction forms and therefore share newline/`;` layout, but D136 does not
    unify object slots and Map entries or their execution-context semantics.

18. **Construction and Map patterns may share surface shape without sharing
    grammar or semantics.** Expression-position `%{ key: value }` constructs.
    Pattern-position `%{ key: pattern }` remains owned by matching semantics.
    Existing Map-pattern comma separation, subset/exactness/remainder, snapshot,
    capture, and matching behavior are unchanged by D136.

19. **No new lexer context is required.** `%{` is structurally parsed from the
    existing `%` and `{` tokens in expression-start position; D136 does not require
    contextual tokenization or a new reserved word.

20. **Ordinary postfix composition applies.** Because the form is an expression,
    existing postfix operations can follow the completed construction, subject to
    normal grammar, for example indexed access or message/call suffixes.

21. **No spread/merge form is selected.** D136 does not add `%{...otherMap}`,
    generic iterable expansion, entry spread, merge precedence, or another
    collection-construction protocol. Such capability may be designed later if
    real use justifies it.

22. **Normal `Map` owns the shorthand.** `%{...}` constructs through the ordinary
    binding named `Map`. D136 does not add corresponding construction syntax for
    `IdentityMap` or another keyed collection merely because it exposes the same
    indexing protocol.

23. **Parameterized keyed construction remains an available future extension, not
    a current feature.** If real usage later justifies a generalized spelling such
    as `%IdentityMap{...}` or `%Factory{...}`, that must be decided independently.
    D136 intentionally leaves that currently-invalid surface available for future
    compatible extension; it does not define `%{...}` as a shorthand for such a
    generalized grammar today.

## Why the construction is sequential

An initially attractive alternative was to make each mapping a first-class
`Association(key, value)` argument and lower the source form to one call:

```protos
Map(
    Association(k1, v1),
    Association(k2, v2)
)
```

This fails the desired observable ordering contract. Ordinary invocation evaluates
all argument expressions before the callee behavior begins. As a result, later
key/value expressions would run before hashing, equality comparison, or insertion
for earlier entries. Because Protos `hash` and `==` are ordinary observable guest
behavior, the difference is semantically visible.

It also changes fail-fast behavior: if insertion of the first association fails,
a one-call model may already have evaluated later association arguments.

Candidate E′ instead preserves the behavior programmers already get from
`Map()` followed by ordinary insertion: each key/value pair is evaluated and
inserted before the next source entry begins.

## Why construction uses newline / `;`, not comma

The initial ratification draft used comma-separated entries, influenced by
argument-list and existing Map-pattern surface forms. Subsequent analysis exposed
that this was inconsistent with the semantic model actually selected by E′.

Map construction is not an argument vector and does not first materialize a list
of Association values. It is a sequential container-construction form:

```text
create Map
entry operation
entry operation
...
return Map
```

The closest existing Protos surface analogue is therefore object construction:

```protos
{
    name: value
    age: other
}
```

Object bodies already use newline across lines and `;` on one line. Applying the
same layout family to `%{...}` makes the visual relationship reflect the selected
semantics:

```protos
{
    name: value
    age: other
}

%{
    key1: value
    key2: other
}
```

The `%` is the domain discriminator: slot-oriented object construction versus
keyed Map construction. The two forms still differ in what `:` means and in the
activation/context rules described below.

The owner explicitly approved this sequence-layout amendment before specification
or implementation work began.

## Relationship to object construction

The visual and layout relationship is intentional:

```protos
{
    name: value
    age: other
}
```

operates in the object-slot domain, while:

```protos
%{
    key: value
    otherKey: other
}
```

operates in the keyed Map domain.

Both are container-construction forms that process source material in order and
both use ordinary Protos sequence separators. They do **not** share execution
context semantics. Object construction evaluates its body in the existing
object-construction activation model. Map construction must evaluate entry
key/value expressions in the enclosing activation so that `this`, `context`,
lexical lookup, Closures, and non-local control behave exactly as around the
equivalent explicit `Map()` plus indexed assignments.

The `%` therefore acts as the domain discriminator between object/slot construction
and keyed construction without claiming that the underlying storage models are
the same.

## IdentityMap boundary

`IdentityMap` remains a distinct Core keyed collection whose key law is identity
rather than normal Map equality/hash semantics. D136 does not infer that every
keyed collection exposing `atPut` deserves parallel literal syntax.

The ordinary use case remains:

```protos
m: IdentityMap()
m[key] = value
```

This is intentional. Repository evidence shows IdentityMap is commonly used for
specialized implementation concerns such as visited/active object sets, cycle
tracking, graph traversal, and identity-sensitive internal state, often beginning
empty and populated dynamically. That does not establish present need for a
populated-construction shorthand.

If future real usage justifies concise populated IdentityMap construction, a
parameterized keyed-construction surface such as `%IdentityMap{...}` can be
considered without changing the ratified `%{...}` meaning. No such extension is
approved by D136.

## Association / Entry boundary

D136 does not require a public `Association` abstraction in order to define Map
construction.

Research found that a first-class detached `Association(key, value)` may still be
useful independently. Protos already has association-like concepts internally and
other languages expose similar first-class values. That question is deliberately
deferred rather than bundled into the syntax decision.

The current internal `ProtosMapValue.Entry` is not reinterpreted as such a public
Association. It represents Map-owned storage and carries Map-specific state such
as the recorded hash and representative-key lifecycle. A future public
Association, if independently approved, should not automatically be the backing
entry object of a Map.

## Comparative evidence

The investigation covered multiple design families rather than selecting by
surface familiarity alone.

- **Self**: prototype/message-oriented collections emphasize ordinary construction
  and mutation mechanisms rather than a dedicated dictionary literal institution.
- **Io**: similarly uses ordinary Map cloning/insertion while providing separate
  conveniences for sequence-like collections.
- **Smalltalk**: basic Dictionary construction remains ordinary `new` plus
  `at:put:`; some implementations also expose first-class Association-based
  convenience APIs. Identity-oriented dictionaries need not receive a parallel
  literal merely because their protocol is similar.
- **Lua**: table constructors demonstrate the strong conceptual similarity between
  record-like and keyed construction, though Lua deliberately unifies storage
  roles that Protos keeps separate.
- **Elixir**: `%{...}` provides a precedent for expression/pattern surface symmetry
  while keeping construction and matching semantics context-dependent.
- **Java**: `Map.Entry`/`ofEntries` demonstrates an explicit-entry approach, useful
  as evidence for the Association alternative but not sufficient to solve the
  Protos ordering problem caused by ordinary call argument evaluation.
  `IdentityHashMap` also demonstrates that identity-keyed Maps can remain an
  explicit specialized facility rather than receiving parallel literal syntax.
- **Python**: dict displays provide direct keyed construction with deterministic
  source-order evaluation rather than requiring an alternating positional-argument
  factory.

The relevant architectural approaches were therefore:

1. ordinary empty construction plus sequential insertion;
2. one populated call with alternating key/value arguments;
3. one populated call consuming explicit association values;
4. dedicated keyed construction syntax with its own sequential construction
   semantics;
5. generalized object/table construction that merges record and map roles;
6. generalized `%Factory{...}` keyed construction, deferred until real need.

Candidate E′ keeps Protos's existing Map protocol and adopts only the bounded
source-construction sequencing needed for ergonomic normal-Map construction.

## Alternatives rejected or deferred

### Alternating populated invocation

```protos
Map(k1, v1, k2, v2)
```

Rejected. A Map element is conceptually one key/value association, not one scalar
argument. Flattening entries into alternating positional arguments erases that
unit, introduces an odd-arity rule, and scales poorly visually and semantically.

### Association-argument lowering

```protos
Map(Association(k1, v1), Association(k2, v2))
```

Rejected as the mandatory lowering for `%{...}` because ordinary call evaluation
moves all entry-expression evaluation ahead of insertion, changing observable
ordering and failure behavior. It also introduces a second shadow-sensitive name
that the keyed syntax does not need.

### Comma-separated construction entries

```protos
%{
    k1: v1,
    k2: v2
}
```

Superseded by the owner-approved sequence-layout amendment. Comma separation
models an argument/list-like structure that E′ explicitly does not use. Newline
and `;` better expose the sequential container-construction model and align with
ordinary object-body layout.

### Keep only explicit `Map()` plus insertion

Still semantically sufficient, but rejected as the final ergonomics outcome
because `%{...}` can provide concise construction without adding a new keyed
storage model, generic literal protocol, new keyword, or privileged constructor.

### IdentityMap construction sugar

Deferred. `IdentityMap` is a specialized identity-keyed collection and does not
automatically inherit every syntax convenience of normal `Map`. Future evidence
may justify `%IdentityMap{...}` or a generalized `%Factory{...}` mechanism, but
D136 does not pay for that surface now.

### Public `Association`

Deferred, not rejected. It may be useful for APIs, transformations, iteration, or
other collection work, but D136 does not need it and therefore does not charge the
current language with that additional abstraction.

### Spread / merge / comprehensions

Deferred. No current requirement justifies specifying merge precedence, generic
entry iteration, comprehensions, or spread semantics as part of the initial Map
construction surface.

## Incremental-design gate

**Smallest sufficient solution:** `%{ key: value }` with newline/`;` entry
separation, ordinary `Map()` lookup/invocation, and ordinary sequential insertion.
No new public entry type, generic keyed-construction protocol, or IdentityMap
surface is necessary.

**Pay for what is needed:** current users pay only for one bounded keyed
construction form and its precise sequencing semantics. They do not inherit tuple,
pair, iterable, merge, literal-conversion, generalized `%Factory{...}`, or
identity-keyed literal institutions.

**Grow as needed:** a future `Association`, merge API, spread form, or parameterized
`%Factory{...}` construction can be added independently without changing the
selected fundamental model that a keyed result is created first and populated
through ordinary keyed semantics.

**Cost of deferral:** deferring Association/spread/merge/IdentityMap sugar requires
adding new APIs or grammar later but does not require changing normal Map identity,
equality, hashing, ordering, or authority. Currently invalid `%IdentityMap{...}`
syntax remains available for a compatible future decision.

**Anti-overengineering result:** no speculative capability is required for the
motivating use case. The candidate preserves future options without
preimplementing them.

## Publication boundary

This record captures the approved D136 architectural contract only.

At amended-ratification publication time:

- Protos specification changed: **NO**;
- Protos implementation changed: **NO**;
- Protos implementation version changed: **NO**;
- matching semantics changed: **NO**;
- `IdentityMap` construction syntax introduced: **NO**;
- generalized `%Factory{...}` introduced: **NO**;
- public `Association` introduced: **NO**.

A subsequent specification/implementation work item must implement and test the
selected source form before D136 can be treated as delivered language surface.
