# D233 — Ratified B1: fixed bitwise operator sugar

**Decision:** D233-B1. **Owner approval:** 2026-11-01, exact response: “Apruebo este contrato.”
**Issue:** https://github.com/guillermomolina/protos/issues/1016
**Product baseline:** guillermomolina/protos@fba47c40c4b79b6e580df1c1f78a3fcecbd067e2; product 0.3.429-SNAPSHOT; spec 0.1.484.
**Status:** design approved; normative product specification and implementation not yet published.

## Exact surface and lowering

| Syntax | Mandatory ordinary send |
|---|---|
| `a & b` | `a.bitAnd(b)` |
| `a \| b` | `a.bitOr(b)` |
| `a ~ b` | `a.bitXor(b)` |
| `~a` | `a.bitNot()` |
| `a << n` | `a.shiftLeft(n)` |
| `a >> n` | `a.shiftRight(n)` |

Exactly five newly valid fixed source tokens: `&`, `|`, `~`, `<<`, `>>`. Binary `~` means XOR; prefix `~` means NOT. No new method selectors, custom symbolic operators, user fixity, or special integer runtime route. Ordinary receiver lookup, shadowing, delegation, overriding, this/methodHome/extraction and error propagation are preserved. Every binary bitwise RHS is strict, evaluated once after the LHS, unlike lazy `&&`/`||`.

## Binding strength (highest to lowest)

postfix `. () []` > unary `! - ~` > `* / %` > `+ -` > `<< >>` > `&` > binary `~` > `|` > comparisons `< <= > >=` > equality `== != === !==` > `&&` > `||`. New binary levels left-associative. Existing non-local return `^` and slot writes `:`/`=` keep their enclosing current rules. Existing operator-to-operator relative precedence is unchanged.

Grammar: comparison receives bitwise-or; bitwise-or receives bitwise-xor and repeats `|`; bitwise-xor receives bitwise-and and repeats `~`; bitwise-and receives shift and repeats `&`; shift receives additive and repeats `<<`/`>>`; unary additionally admits prefix `~`.

## Lexical and observable constraints

Keep complete-symbolic-run maximal munch: valid `a&b`, `a&&b`, `a|b`, `a||b`, `a~b`, `~a`, `a ~ ~b`, `~(~a)`; lexical rejection of `a~~b`, `~~a`, `a&-b`, `a<<-1`, `a>>=b`, `a>>>b`, `a~=b`, `a|>b`, `^^a`, `!!a`. `a << -1` parses, then signals standard negative shift count Error. An incomplete infix expression continues after newline; a completed preceding line does not attach leading binary operators. `^` remains non-local return, never XOR.

All six D230-C1 standard methods retain unlimited sign-extended two's-complement, exact shifts, genuine semantic Integer validation, zero/large/negative count behavior, result normalization and resource failures. A custom non-Integer receiver can still implement its own ordinary named message; the syntax does not enforce an Integer family before lookup. No new Boolean coercion, host-width word, bit-vector, rotate, `>>>`, compound assignment, or named infix operator.

## GITHUB021 invariant reconciliation

D149 arbitrary custom operators, their declaration/fixity, and special precedence remain removed. The selected extension adds only five **fixed** tokens; old fixed operators keep their relative precedence. D230's exact six canonical named messages and semantics remain unchanged; D233 is an independently approved forward extension to D230's original no-syntax scope, not a retroactive amendment. Existing `^`, lazy Boolean and ordinary receiver-dispatch invariants are preserved.

**DECISION_APPROVAL_PROVENANCE=PASS; DECISION_INVARIANT_CONSISTENCY=PASS; D233_B1=RATIFIED; PRODUCT_IMPLEMENTATION=PENDING.**

## Publication boundary

This record is durable non-normative approval, not a normative Protos source change. A distinct I-family issue must implement all grammar, lexing, parser, canonicalizer, formatter, tests, tooling and documentation in **one** Protos product slice, human-executed validation and publication. No product build, test, benchmark, version increment or runtime implementation is claimed by this document.