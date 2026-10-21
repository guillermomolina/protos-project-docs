# D211 — Contextual result-producing if/else and removal of redundant Boolean action messages

**Status:** RATIFIED — owner approved the inseparable A+B+C bundle, 2026-10-21.  
**Owning design issue:** [D211/#944](https://github.com/guillermomolina/protos/issues/944).  
**Approval provenance:** [explicit owner-approval checkpoint](https://github.com/guillermomolina/protos/issues/944#issuecomment-6314980398), following [falsification](https://github.com/guillermomolina/protos/issues/944#issuecomment-6314847624) and [GITHUB010 matrix](https://github.com/guillermomolina/protos/issues/944#issuecomment-6314883087).  
**Evidence:** [D211 falsification and approval](../../evidence/D211/D211_CONTEXTUAL_IF_FALSIFICATION_AND_RATIFICATION_2026_10_21.md).  
**Observed product baseline:** [`7a122cdeeed91ca31690e2d39412dadad3518f72`](https://github.com/guillermomolina/protos/commit/7a122cdeeed91ca31690e2d39412dadad3518f72) (before implementation, NOT a D211 implementation revision).  
**Normative owners requiring implementation publication:** `spec/PROTOS_GRAMMAR.md`, `spec/PROTOS_LANGUAGE_SPEC.md`, `spec/semantics/VALUES_AND_COLLECTIONS.md`, with ordinary lookup, invocation and control governed by `OBJECT_MODEL.md`, `CALLABLES.md`, and `EXECUTION_AND_CONTROL.md`.

## Exact indivisible owner decision: A + B + C, or none

### A — remove standard receiver-preserving action messages

Remove the standard **Object** slots `if(block)` and `else(block)`, their implementation/structured dispatch, and normative status as standard Boolean selectors. They are **not aliases** to any other selector. Preserve the ordinary `if` and `else` identifier/member-name spellings for user-defined slots and lexical bindings outside the exact contextual grammar. Preserve `ifTrue`, `ifFalse`, `ifTrueIfFalse`, `not`, `and`, `or`, all of their independently specified semantics, and other root slots.

This is a deliberate public API and reflection compatibility break: `Object.slotValue("if")`, the standard extracted `.if`/`.else` methods, and Boolean-preserving chained action selection are no longer available as **standard** methods.

### B — one-arm contextual if

```protos
if(condition) { trueBody }
```

is observably the ordinary result-producing send

```protos
condition.ifTrue(() => { trueBody })
```

The `condition` expression is evaluated **exactly once in the caller's current execution context**, before the invocation. Do **not** defer the condition in a zero-argument Closure: that wrapping is required for D210's loop predicate but would be wrong for this conditional. The true branch is a new ordinary parameterless lexical Closure; the standard `ifTrue` executes it on canonical true and returns its exact normal result; canonical false returns `null`, without executing the callback. No implicit truthiness or Future-awaiting.

### C — two-arm contextual if/else

```protos
if(condition) { trueBody } else { falseBody }
```

is observably the ordinary result-producing send

```protos
condition.ifTrueIfFalse(
    () => { trueBody },
    () => { falseBody }
)
```

Evaluate the condition exactly once; then produce both branch Closure arguments in ordinary left-to-right call evaluation; only the selected branch is invoked and validated for callability under the standard behavior. Return the selected branch's exact normal result, including object identity and Future identity. No implicit Future adoption, coercion, or result policy shared with former D200 `if`/`else`.

Both new surface forms are **expressions**, not distinct statements/keywords/runtime primitives. Preserve ordinary lookup, overrides/shadowing, receiver-domain checks of the selected standard behavior, Closure lexical capture, `this`, `super`, `context`, home activation of `^`, errors, nested Tasks, suspension/replay, cancellation and cleanup. The two forms deliberately use **different ordinary protocol selectors**, even when customized.

## Exact syntax boundary

Use D210's narrow contextual interpretation, **not** generic undotted message syntax or new reserved words:

- Bare **ungrouped** identifier `if`, followed by exactly one ordinary, non-spread expression in `(...)`, then an attached parameterless braced trailing Closure after the closing `)`, with **no logical NEWLINE** token between `)` and `{`.
- Optional `else { falseBody }` contextual suffix only **immediately after the first branch's closing `}` within the same continuing logical line**; `else` remains an ordinary lexer identifier globally, followed by a braced parameterless body Closure. No `elseIf`/`elif`, no generic statement grammar and no new newline continuation exception.
- The exact `if(...){...}` shape takes precedence over lexical binding named `if`, including custom callables. This source incompatibility is explicitly accepted, as with D210's `while`. `(if)(condition) { body }`, `object.if(condition) { body }`, `if(condition)` without trailing body, spread/multiple-argument variants, and `if(condition)\n{ body }` remain ordinary grammar forms. User-defined `.if`/`.else` members remain available; removing the standard slots does not ban names.
- Standalone `else { ... }` remains ordinary object construction from the `else` identifier; it is not a new top-level conditional. A separating newline after `}` ends the contextual `if` expression; a subsequent `else` must be parsed under ordinary rules.
- Match the optional `else` before subsequent postfix suffixes. Both conditional expression results support existing member/call/index suffixes and compose within assignments, argument lists, parentheses, nested Closures and `^`. Nested ifs pair structurally by mandatory braces; reject/retain malformed source according to the existing parser.
- Comments, source ranges, formatting and precise logical newline/continuation rules remain normative, and must round-trip unambiguously. Block comments with internal line breaks do not create logical NEWLINE tokens; `//` does.

The core language **does not** acquire global `if`/`else` reserved lexical tokens.

## Reconciliation of prior approved decisions (GITHUB021)

| Previous authority / invariant | D211 delta and explicit status |
|---|---|
| D050: standard `ifTrue`, `ifFalse`, `ifTrueIfFalse` selected-result contracts | **PRESERVED** without weakening |
| D051: `if(condition){body}` is ordinary lexical invocation, no conditional interpretation | **EXPLICITLY REOPENED AND SUPERSEDED for this exact contextual source shape** |
| D199: non-approval of broad undotted fluent Boolean syntax | **PRESERVED for general fluent/undotted sends**; narrow D211 contextual if is a new independently approved decision |
| D200: additive standard `Object.if`/`Object.else` preserving Boolean and ignoring callback normal results | **EXPLICITLY REOPENED AND REMOVED**; no compatibility aliases |
| D200: `ifTrue`/`ifTrueIfFalse`, `not`/`and`/`or`, no standard `elseIf` | **PRESERVED** |
| D210: narrow contextual `while`, no reserved token, priority over exactly shaped lexical `while` call | **PRESERVED**; new if uses the same grammar design but condition is **not** deferred |
| Ordinary member names `if` and `else`, custom method overrides, source object model and Closure invocation | **PRESERVED except deliberately reinterpreted exact bare `if(...){...}` spelling** |

## Adversarial compatibility result

Three **proven** breaks in the published product baseline must not be represented as new implementation bugs: lexical `if: (...) => {...}; if(123){42}` formerly invokes user callable; D200 Boolean result/reflection/extraction of the removed standard slots; and effect-only `.if(...).or(...).else(...)` multiway chains. The latter **cannot** be transparently rewritten to `ifTrue`/nested `if` without reviewing effects and return values. D211 intentionally selects a simpler, result-producing surface instead of preserving D200 action combinators.

The targeted owner review found no fundamental inconsistency in *new* B/C lowering, but source compatibility is intentionally broken. Further runtime, parser and tooling assertions belong to the implementation phase; no implementation tests have been run for D211.

## Publication and implementation gate

This record ratifies the **language-design decision only**. Normative `spec/` changes, parser/semantic AST, Object installation/structured fast paths, conformance/source tools and metadata remain for the separate implementation work item. The owner reports pre-existing local `git diff --check` clean and tests green; that is **not** D211 implementation validation. Preserve this record and older D199/D200/D210 records as accurate historical decisions; do not rewrite them.

Implementation should deliver **A+B+C in one atomic product publication**, with editing batches permitted but no intermediate product release exposing A without B/C or B/C without A. Human runs all product tests and publication; direct agent repository publication is authorized **only for bounded D211/I119 project-docs records**, not `guillermomolina/protos` product code.
