# D136 — Map populated construction model and source sugar

Status: **RATIFIED — Candidate E′ selected**

Explicit project-owner approval: **2026-09-16**
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
    key1: value1,
    key2: value2
}
```

The form is a Map-producing primary expression. It is semantic source sugar for
constructing one fresh result through the existing ordinary `Map()` factory and
then performing ordinary keyed insertion for each source entry from left to
right.

Conceptually:

```protos
%{
    k1: v1,
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
   `atPut` operations. D136 defines no literal-specific duplicate-key rule and
   does not change representative-key, recorded-hash, insertion-order, equality,
   or replacement semantics.

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

14. **Construction and object syntax remain distinct.** `{...}` is still object
    construction/body syntax operating in the slot domain. `%{...}` is keyed Map
    construction syntax. Their visual similarity is deliberate, but D136 does
    not unify object slots and Map entries.

15. **Construction and Map patterns may share surface shape without sharing
    semantics.** Expression-position `%{ key: value }` constructs. Pattern-position
    `%{ key: pattern }` remains owned by matching semantics. D136 does not change
    subset/exactness/remainder, snapshot, capture, or matching behavior.

16. **No new lexer context is required.** `%{` is structurally parsed from the
    existing `%` and `{` tokens in expression-start position; D136 does not require
    contextual tokenization or a new reserved word.

17. **Ordinary postfix composition applies.** Because the form is an expression,
    existing postfix operations can follow the completed construction, subject to
    normal grammar, for example indexed access or message/call suffixes.

18. **No spread/merge form is selected.** D136 does not add `%{...otherMap}`,
    generic iterable expansion, entry spread, merge precedence, or another
    collection-construction protocol. Such capability may be designed later if
    real use justifies it.

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

## Relationship to object construction

The visual relationship is intentional:

```protos
{
    name: value
}
```

operates in the object-slot domain, while:

```protos
%{
    key: value
}
```

operates in the keyed Map domain.

Both are container-construction forms that process source material in order, but
they do **not** share execution context semantics. Object construction evaluates
its body in the existing object-construction activation model. Map construction
must evaluate entry key/value expressions in the enclosing activation so that
`this`, `context`, lexical lookup, Closures, and non-local control behave exactly
as around the equivalent explicit `Map()` plus indexed assignments.

The `%` therefore acts as the domain discriminator between object/slot construction
and keyed construction without claiming that the underlying storage models are
the same.

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
  convenience APIs.
- **Lua**: table constructors demonstrate the strong conceptual similarity between
  record-like and keyed construction, though Lua deliberately unifies storage
  roles that Protos keeps separate.
- **Elixir**: `%{...}` provides a precedent for expression/pattern surface symmetry
  while keeping construction and matching semantics context-dependent.
- **Java**: `Map.Entry`/`ofEntries` demonstrates an explicit-entry approach, useful
  as evidence for the Association alternative but not sufficient to solve the
  Protos ordering problem caused by ordinary call argument evaluation.
- **Python**: dict displays provide direct keyed construction with deterministic
  source-order evaluation rather than requiring an alternating positional-argument
  factory.

The relevant architectural approaches were therefore:

1. ordinary empty construction plus sequential insertion;
2. one populated call with alternating key/value arguments;
3. one populated call consuming explicit association values;
4. intrinsic/dedicated keyed construction syntax with its own sequential
   construction semantics;
5. generalized object/table construction that merges record and map roles.

Candidate E′ keeps Protos's existing Map protocol and adopts only the bounded
source-construction sequencing needed for ergonomic keyed literals.

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

### Keep only explicit `Map()` plus insertion

Still semantically sufficient, but rejected as the final ergonomics outcome
because `%{...}` can provide concise construction without adding a new keyed
storage model, generic literal protocol, new keyword, or privileged constructor.

### Public `Association`

Deferred, not rejected. It may be useful for APIs, transformations, iteration, or
other collection work, but D136 does not need it and therefore does not charge the
current language with that additional abstraction.

### Spread / merge / comprehensions

Deferred. No current requirement justifies specifying merge precedence, generic
entry iteration, comprehensions, or spread semantics as part of the initial Map
construction surface.

## Incremental-design gate

**Smallest sufficient solution:** `%{ key: value, ... }` plus ordinary `Map()` and
ordinary sequential insertion. No new public entry type or generic collection
protocol is necessary.

**Pay for what is needed:** current users pay only for one punctuation form and
its precise sequencing semantics. They do not inherit tuple, pair, iterable,
merge, literal-conversion, or generic keyed-construction institutions.

**Grow as needed:** a future `Association`, merge API, or spread form can be added
independently without changing the selected fundamental model that a Map is
created first and populated through ordinary keyed semantics.

**Cost of deferral:** deferring Association/spread/merge requires adding new APIs
or grammar later but does not require changing the identity, equality, hashing,
ordering, or authority model selected here.

**Anti-overengineering result:** no speculative capability is required for the
motivating use case. The candidate preserves future options without
preimplementing them.

## Publication boundary

This record captures the approved D136 architectural contract only.

At ratification-record publication time:

- Protos specification changed: **NO**;
- Protos implementation changed: **NO**;
- Protos implementation version changed: **NO**;
- matching semantics changed: **NO**;
- public `Association` introduced: **NO**.

A subsequent specification/implementation work item must implement and test the
selected source form before D136 can be treated as delivered language surface.
