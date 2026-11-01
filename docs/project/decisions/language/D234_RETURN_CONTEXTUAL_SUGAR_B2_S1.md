# D234 — Ratified B2-S1: contextual `return` is non-local `^` sugar

**Decision:** D234-B2-S1. **Owner approval:** 2026-11-01, exact statement: “apruebo D234-B2-S1”, following a detailed GITHUB021 invariant/delta review and explicit incompatibility and newline disclosure.
**Issue:** https://github.com/guillermomolina/protos/issues/1018
**Exact inspected Protos revision:** `guillermomolina/protos@c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5` (product 0.3.431-SNAPSHOT; spec 0.1.486). This is a design-only baseline, not an implementation claim.
**Status:** selected and approved, durably ratified in this project record; *normative specification and product implementation remain pending*.

## 1. Positive language contract

`return expression` is a **contextual expression**, not a statement and not a reserved keyword. The token `return` stays an ordinary `IDENTIFIER`. When the structural contextual shape is recognized in expression-initial position, `return expression` has exactly the same **`CanonicalReturn`** semantics, return-home authority, operand evaluation once, result identity, error/unwind/cancellation behavior and scope-exit cleanup as `^expression`. No second return mechanism, local callback return, runtime opcode, Closure kind, method type, dynamic send or new return home. `^` remains valid with unchanged meaning.

Non-local return from lexically nested closures exits their captured active method/function home, not merely the callback invocation. Escaped, completed, or foreign-Task home signals `InvalidReturn` (D177); Actors cannot transfer return-home authority; isolated P execution has fresh P-local return-home rules; `ensure` cleanup precedence and Future/Task semantics are unchanged. Normal last-expression Sequence result remains distinct; no implicit return-`null` from a bare keyword. `return;` does not become a valid terminating statement; Protos semicolon is an inline **separator** and has no ASI.

The contextual alias has the same expression-wide operand binding as `^`: `return a + b` -> `^(a + b)`, and it may appear in legal expression positions (including nested arguments, assignment RHS, parameter defaults and closure bodies), subject to ordinary surrounding grammar and exact contextual recognition. The runtime does not distinguish source spellings after canonicalization, but source spans and formatter retain the spelling.

## 2. Approved recognition and structural precedence

- Only bare, ungrouped `return` in expression-initial position may begin the alias. `obj.return`, `obj.return(x)`, `(return)(x)` and non-initial ordinary member dispatch keep their old meaning.
- `return(x)` **with exactly one ordinary non-spread expression** and `return (x)` mean the contextual return; `return()` and `return(x,y)` are ordinary calls. Spacing is insignificant; a parseable one-expression parenthesized operand may be multiline. A grouped call `(return)(x)` escapes the reinterpretation.
- `return - x`, `return ~ x`, `return[0]` and `return { a: 1 }` are contextual returns of unary negative, unary bitwise NOT, an Array construction, and a bare object construction, respectively. This deliberately changes some formerly valid ordinary expressions. `(return) - x`, `(return) ~ x`, `(return)[0]` and `(return) { a: 1 }` remain ordinary.
- Bare `return` without an operand stays an ordinary lookup, not `^null`. `return: v`, `return = v`, `return => v` and `object.return(v)` keep ordinary syntax/semantics.
- Protect complete ordinary `return`-rooted **assignment/creation target** shapes, including `return[0] = x`, `return[0].name = x` and `return[0].name: x`, when syntactically valid under current target grammar. In these cases the outer assignment/creation is ordinary; contextual return must not swallow part of a target, change the lvalue or silently reject the assignment. `return(x,y)` and `return()` stay ordinary calls. No new generalized pattern-matching operator or whitespace sensitivity.
- Contextual recognition takes syntactic precedence over lexical lookup in approved shapes, as with ratified contextual `if` and `while`; there is no runtime conditional heuristic based on the value of the variable `return`.
- A logical `NEWLINE` immediately after bare `return` does **not** continue its operand. Thus `return` newline `calculate()` is two ordinary expressions, whereas `return(calculate())` and a multiline parenthesized operand work. A line comment followed by its newline also prevents pairing; a block comment with no emitted `NEWLINE` token does not. `^` keeps its independently existing continuation-newline rule. No new ASI rule; preservation of complete-before-newline behavior remains mandatory.
- For operator precedence, valid operands and postfix continuation use exactly the already-defined expression grammar. The parser must preserve its usual token maximal-munch, source spans, literals, Unicode, member-lookup and contextual-if/while/until boundaries.

## 3. Deliberate compatibility deltas (owner approved)

| Former valid source | Former behavior | B2-S1 behavior | Ordinary spelling preserving old intent |
| --- | --- | --- | --- |
| `return(42)` | Invoke lexical `return` with one argument | Non-local return 42 | `(return)(42)` |
| `return - x` | Binary subtraction from lexical `return` | Non-local return unary `-x` | `(return) - x` |
| `return ~ x` | D233 binary XOR from lexical `return` | Non-local return unary bitwise NOT `~x` | `(return) ~ x` |
| `return[0]` | Indexed lookup on lexical `return` | Return constructed Array `[0]` | `(return)[0]` |
| `return { a: 1 }` | Object construction using `return` as parent | Return object literal `{ a: 1 }` | `(return) { a: 1 }` |

All five were explicitly identified in the selected B2-S1 packet and the owner subsequently approved B2-S1. No claim that all possible third-party source usages have been audited. Do not silently extend contextual precedence to ordinary method names, parameter binders, assignments or additional syntactic collisions.

## 4. GITHUB021 decision-invariant/delta reconciliation

| Prior authority / invariant | Reconciliation |
| --- | --- |
| `CALLABLES.md` ``13–14, D177 #683: return home, completed/foreign-Task `InvalidReturn` | Preserved, same `CanonicalReturn` |
| `EXECUTION_AND_CONTROL.md` `8.2: final Sequence result and cleanup precedence | Preserved; no statement terminators, local returns or new `null` |
| `ACTORS.md` `24H, `PARALLEL_EXECUTION.md`: Actor boundary and fresh P-local home | Preserved |
| D210 #941 / D211 #944: contextual exact patterns without reserving ordinary identifiers | Preserved; approved new bounded contextual exception |
| D233 #1016: `~` unary NOT and binary XOR | Preserved globally; new approved contextual ambiguity `return ~ x` |
| Exact reserved-word set `this context super true false null` | Preserved |
| Ordinary lookup and assignment | Explicitly narrowed for five listed patterns; complete assignment/slot creation targets preserved |
| Current `^` and source-level span / lexer behavior | Preserved; novel `return` newline rule explicit |

**GITHUB021:** Every applicable existing invariant is preserved unless the five syntactic source-compatibility changes expressly approved by the owner are counted as the selected forward extension. There is no claim of permission to change other observable behavior.

**DECISION_APPROVAL_PROVENANCE=PASS** (owner selected B2 then explicitly approved D234-B2-S1);
**DECISION_INVARIANT_CONSISTENCY=PASS** for the reviewed bounded contract;
**D234_B2_S1=RATIFIED_DESIGN**;
**PRODUCT_NORMATIVE_PUBLICATION=PENDING**;
**PRODUCT_IMPLEMENTATION=PENDING**.

## 5. Design versus product publication

This record is durable **non-normative** decision evidence, not an update to `guillermomolina/protos/spec`. The distinct I138 implementation issue owns **one** integrated human-validated product patch: normative grammar and owners; parser and source tooling; formatter Protos-first; tests and guides; one version/spec changelog update derived from actual HEAD; one product commit/push by the human only. If the precise parser mechanics reveal a new material conflict with B2-S1, stop and return it for owner approval, do not quietly revise the selected semantics.

Source references: https://github.com/guillermomolina/protos/blob/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5/spec/PROTOS_GRAMMAR.md ; https://github.com/guillermomolina/protos/blob/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5/spec/semantics/CALLABLES.md ; https://github.com/guillermomolina/protos/blob/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5/src/main/java/com/guillermomolina/protos/parser/ProtosParser.java ; https://github.com/guillermomolina/protos/blob/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalReturnHomeAnalysis.java .
