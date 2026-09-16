# D130 — Array construction syntax

Status: **RATIFIED — Candidate A′ selected**

Explicit project-owner approval: **2026-09-16**
Decision issue: `guillermomolina/protos#502`
Protos baseline reviewed: `e99d0baba547ac41b3894f32ddca450172ee1f8b`
AUD009 sequencing dependency: `AUD009-A1` / `guillermomolina/protos#535`
Related matching semantic decision: `D131` / `guillermomolina/protos#503`

Nature: non-normative decision/rationale record for future normative grammar and
Array-construction semantics under `guillermomolina/protos/spec/`.

## Decision

Select **Candidate A′ — Array construction syntax is pure ordinary-call sugar**.

The selected source form is:

```protos
[]
[a]
[a, b]
[a, f(), object.x]
```

with mandatory lowering to the already-existing ordinary invocation form:

```protos
Array()
Array(a)
Array(a, b)
Array(a, f(), object.x)
```

The lowering is semantic source sugar over the ordinary expression `Array(...)`.
It does not denote a privileged runtime Array constructor, an intrinsic Core
Array allocation opcode, a hidden prelude handle, or a separate collection
literal protocol.

## Ratified contract

1. **Primary-expression form.** A bracketed Array-construction form may begin an
   ordinary expression. The opening `[` at expression start is distinct from the
   existing postfix index suffix in `receiver[index]`.

2. **Ordinary `Array` lookup is preserved.** The generated `Array(...)` receiver
   is resolved by the same ordinary lexical/prelude lookup rules as if the user
   had written `Array(...)` directly.

3. **Shadowing is observable and intentional.** If an ordinary local or lexical
   binding named `Array` shadows the prelude binding, `[a, b]` invokes that
   resolved object exactly as `Array(a, b)` would. If the resolved object is not
   invokable, the bracket form fails exactly as the equivalent ordinary call
   fails. D130 does not bypass lookup to recover the standard Core `Array`.

4. **No new construction semantics.** When ordinary lookup resolves the standard
   prelude `Array`, the bracket form inherits the existing standard Array factory
   semantics: each successful invocation creates a fresh open standard Array,
   with dense receiver-owned indexed state containing exactly the supplied
   argument objects in order.

5. **Existing argument evaluation is authoritative.** Element expressions use
   the ordinary call-argument evaluation rules, including existing left-to-right
   evaluation, exact-once evaluation, error/control propagation, and side-effect
   behavior.

6. **Existing call spread is inherited.** A spread element in Array construction,
   for example:

   ```protos
   [a, ...items, b]
   ```

   lowers to:

   ```protos
   Array(a, ...items, b)
   ```

   and therefore uses the already-defined call-spread semantics. D130 introduces
   no separate Array-spread protocol and no new iterable abstraction.

7. **Existing call-list punctuation/layout is inherited.** Array-construction
   items follow the same separator and layout policy as the equivalent ordinary
   argument list. D130 does not create a second list punctuation dialect. In
   particular, if a trailing comma is invalid in the ordinary call grammar, it
   remains invalid in the bracket form.

8. **Postfix composition remains ordinary.** Because the bracket form is an
   expression, ordinary postfix syntax may follow it. Conceptually:

   ```protos
   [a][0]
   ```

   lowers first to `Array(a)[0]`, after which existing indexed-access lowering
   applies normally.

9. **Nested construction is ordinary composition.** For example:

   ```protos
   [[a], b]
   ```

   lowers compositionally to `Array(Array(a), b)` under the same rules.

10. **No keyword or reserved-word change.** D130 adds no reserved word and no
    contextual word spelling. It uses punctuation only.

11. **No new Array family or subtype rule.** D130 does not define a special
    bracket syntax for `MyArray`, typed Arrays, fixed-length Arrays, immutable
    Arrays, or other collection families. Existing ordinary invocation remains
    the mechanism for those forms.

12. **No implementation privilege.** An implementation may optimize the lowered
    standard-Array case only when observable behavior remains identical to the
    ordinary `Array(...)` expression selected by source semantics, including
    lookup/shadowing behavior.

## Explicit exclusions

D130 does **not** select or define:

- Array holes such as `[a, , b]`;
- comprehensions;
- fill/repetition syntax such as `[value; count]`;
- implicit capacity or length constructors;
- a generic collection-literal protocol;
- expected-type-directed literal conversion;
- implicit iterable expansion beyond existing call spread;
- dedicated syntax for `MyArray` or another Array-derived factory;
- Array pattern syntax;
- wildcard, binder, alias, rest/remainder, or capture syntax for matching;
- match arms;
- `match`, `case`, or `when` spellings;
- matcher extraction semantics;
- any redesign of the existing matching grammar or matching runtime.

In particular, `[a, b]` under D130 is an ordinary Array-producing expression
whose meaning is determined by the `Array(a, b)` lowering. Any use of bracket
syntax inside a matching-pattern grammar remains separately owned by matching
work and gains no semantics from D130.

## Why ordinary lookup is part of the decision

The main semantic fork in D130 is not the punctuation. It is whether `[a, b]`
means exactly the ordinary Protos expression `Array(a, b)`, or whether the syntax
captures an unshadowable built-in Array authority.

Candidate A′ deliberately chooses the former.

The standard prelude name `Array` is an ordinary prelude binding rather than a
reserved intrinsic spelling. Preserving ordinary lookup keeps the bracket form
honest as syntactic sugar and preserves Protos's object/message model. A hidden
canonical-Array constructor would make the source form semantically stronger than
`Array(...)` and would create a new privileged language institution merely for
convenience syntax.

The selected contract therefore favors transparent desugaring over conventional
built-in-literal behavior.

## Grammar compatibility

At the reviewed Protos baseline, indexed access is a postfix construct:

```text
receiver[index]
```

whose index suffix is structurally attached after an already-formed expression.
The ordinary primary-expression set does not currently begin with `[`. D130 can
therefore add a bracketed primary-expression form without making expression-start
Array construction ambiguous with postfix indexed access.

This also makes ordinary compositions natural:

```protos
[a, b][0]
factory()[index]
[[a], [b]][1][0]
```

D130 does not rely on matching-pattern parsing to establish this distinction.
Matching has a separate contextual grammar and remains outside this decision.

## Falsification cases used during selection

The candidate was checked against cases intended to reveal hidden semantics:

```protos
[]
[a]
[[a], b]
[a][0]
[a, ...items]
```

Each has a direct ordinary-call lowering without inventing another runtime
operation.

The decisive shadowing case is:

```protos
Array: customFactory
[a, b]
```

Candidate A′ requires the same observable behavior as:

```protos
Array: customFactory
Array(a, b)
```

A design in which the first form still created a canonical Core Array would fail
the selected pure-sugar criterion.

## Comparative evidence

The decision research covered more than the minimum five systems and compared
multiple architectural approaches rather than treating square brackets as a
universal convention.

### JavaScript / ECMAScript

ECMAScript uses dedicated Array initializer syntax and specifies built-in Array
creation behavior for it. This is a useful example of **intrinsic literal
semantics**: the literal does not mean an ordinary lookup of a variable named
`Array`.

Reference: <https://tc39.es/ecma262/>

### Python

Python list displays use dedicated bracket syntax that creates a new list and
evaluates element expressions in source order. This is another example of an
**intrinsic built-in collection display** rather than source lowering through a
shadowable name.

Reference: <https://docs.python.org/3/reference/expressions.html#list-displays>

### Rust

Rust has dedicated array expressions, including element-list and repetition
forms. The result is part of the language's fixed-size typed array model rather
than an ordinary dynamically-resolved constructor call.

Reference: <https://doc.rust-lang.org/reference/expressions/array-expr.html>

### Swift

Swift supports Array literal syntax through `ExpressibleByArrayLiteral`, showing
a materially different **protocol/type-directed literal** design in which the
expected target type may own literal initialization behavior.

Reference: <https://developer.apple.com/documentation/swift/expressiblebyarrayliteral>

### Kotlin

Kotlin deliberately uses ordinary APIs such as `arrayOf(...)`, `Array(...)`, and
collection factory functions rather than introducing a general bracketed Array
literal. This represents the **ordinary factory/no-sugar** approach.

Reference: <https://kotlinlang.org/docs/arrays.html>

### Lua

Lua generalizes collection construction through table constructors using `{...}`
rather than separating Array and Map literals as distinct nominal constructs.
This is evidence for a broader **general collection-constructor syntax** option,
which D130 intentionally rejects as too broad for the bounded Array question.

Reference: <https://www.lua.org/manual/5.4/manual.html#3.4.9>

### GNU Smalltalk

GNU Smalltalk distinguishes literal Array forms from dynamic Array construction
syntax, demonstrating that a language can make constant/literal collection forms
and evaluated collection construction separate semantic concepts.

Reference: <https://www.gnu.org/software/smalltalk/manual/html_node/The-syntax.html>

### Self

Self provides useful counter-evidence from a prototype/message-oriented language:
the core language favors a small syntax and ordinary object/message mechanisms,
while square brackets are already used for blocks. The important lesson for
Protos is architectural rather than syntactic: convenience syntax should not
silently create an independent semantic mechanism when ordinary objects/messages
already provide the operation.

Reference: <https://handbook.selflanguage.org/>

## Approaches compared

The research therefore covered at least these distinct approaches:

1. **Intrinsic built-in literal/display** — dedicated syntax directly creates the
   language's built-in collection family, as in JavaScript/Python and typed Array
   expressions in Rust.
2. **Protocol/type-directed literal** — literal syntax delegates construction
   through a language protocol or expected type, as in Swift.
3. **Ordinary factory with no bracket sugar** — use ordinary callable/factory APIs,
   as in Kotlin and message-oriented minimalist designs.
4. **Pure source sugar over an ordinary factory expression** — the selected Protos
   design: keep the ordinary object lookup/invocation semantics and provide only
   compact punctuation.
5. **Generalized collection constructor** — one broader constructor surface for
   sequential and keyed collection roles, represented by Lua tables.

## Alternatives rejected

### Candidate B — intrinsic canonical Array literal

`[a, b]` would always construct the canonical Core Array family regardless of a
shadowing `Array` binding.

Rejected because it would introduce a privileged semantic construction path and
would not be honest sugar for the existing `Array(a, b)` expression.

### Candidate C — literal construction protocol

Bracket syntax would invoke a new literal-construction protocol or select a
receiver through expected-type/context rules.

Rejected because Protos currently has no need for a new context-sensitive
literal-dispatch institution merely to abbreviate an already-complete ordinary
Array factory.

### Candidate D — retain only `Array(...)`

No new syntax would be added.

Rejected because the compact form materially improves ordinary collection-heavy
source while Candidate A′ can provide that ergonomics without adding independent
semantics.

### Broader generalized collection syntax

A Lua-style generalized collection constructor could eventually be coherent, but
it expands D130 from Array convenience syntax into a collection-model redesign.
It is outside the approved scope.

## Downstream relationship to AUD009 / D131

AUD009-A1 is paused pending D130 because its protocol-first matching comparison is
expected to use ordered collections of matcher/case objects. The purpose of the
sequencing dependency is ergonomic fairness, not semantic dependence.

D130 now establishes the intended ordinary source baseline:

```protos
[matcherA, matcherB, matcherC]
```

is compact syntax whose complete semantic explanation remains the ordinary
`Array(matcherA, matcherB, matcherC)` call.

This decision does **not** approve any matching API, selector, pattern surface,
arm grammar, or `match`/`case` sugar. D131/AUD009-A1 must resume from its existing
protocol-first checkpoint and make those decisions independently.

## Publication boundary

This record captures the approved D130 architectural contract only.

At ratification-record publication time:

- Protos specification changed: **NO**;
- Protos implementation changed: **NO**;
- Protos implementation version changed: **NO**;
- matching semantics changed: **NO**.

A subsequent Protos implementation/specification slice must implement and test
the selected lowering before D130 can be treated as delivered language surface.
