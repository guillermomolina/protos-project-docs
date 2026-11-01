# D234-B2-S1 — Comparative evidence, attempted falsification, GITHUB021 and owner approval

**Date:** 2026-11-01. **D issue:** https://github.com/guillermomolina/protos/issues/1018 . **Exact source baseline:** `guillermomolina/protos@c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5`, product 0.3.431-SNAPSHOT, spec 0.1.486. **Durable ratification:** [D234-B2-S1](../../decisions/language/D234_RETURN_CONTEXTUAL_SUGAR_B2_S1.md). **No build/test/product edit has been run or claimed.**

## Research and approval provenance

The user requested exhaustive D234 research, selected B2, then expressly requested a GITHUB021 check **before** ratification. The assistant surfaced exact compatibility changes and newline rules in its D234-B2-S1 packet. The owner stated **“apruebo D234-B2-S1”** on 2026-11-01 and expressly authorized issue updates and commit/push of `guillermomolina/protos-project-docs` for this bounded task. Approval is *not* permission to execute or publish product implementation automatically.

## Normative source audit

- `spec/PROTOS_GRAMMAR.md` `2 reserves exactly six words, recognizes `return` as ordinary IDENTIFIER, `11 defines `^, expression`, `35 maps return home and `36 anticipates `return value` sugar; separators/newlines exclude ASI and require expression on both sides of `;`.
- `spec/semantics/CALLABLES.md` ``13–14 owns Smalltalk-style return home, lexical closure capture, completed/foreign-Task `InvalidReturn`.
- `spec/semantics/EXECUTION_AND_CONTROL.md` `8.2 owns normal last-expression result; `ensure` cleanup rules own precedence of non-local returns, errors and cancellation.
- `spec/concurrency/ACTORS.md` `24H, `FUTURES_AND_TASKS.md` and `PARALLEL_EXECUTION.md` prohibit cross-domain return and establish local P authority.
- `src/main/java/com/guillermomolina/protos/lexer/ProtosLexer.java` currently classifies `return` as IDENTIFIER; `ProtosParser.java` parses `^` with `parseExpressionFoundation` and builds `SurfaceNonLocalReturn`; `semantic/Canonicalizer.java` maps that to `CanonicalReturn`.
- `execution/bytecode/lowering/CanonicalReturnHomeAnalysis.java` recursively detects `CanonicalReturn`, including in nested closures and parameter defaults, and selects an optimized return-home representation without changing semantics. No second canonical AST needed.
- `analysis/ProtosStaticLint.java` warns on code unreachable after `SurfaceNonLocalReturn` and has a literal diagnostic string mentioning `^`; `analysis/ProtosSourceLayoutView.java` and Protos-first formatter `protos/tools/formatter/Structural.protos` need source-spelling/round-trip attention. `SurfaceNonLocalReturn` kind appears as `NON_LOCAL_RETURN` in formatting.
- D210 #941 / D211 #944 approve exact contextual recognition, D177 #683 owns foreign Task, D233 #1016 / spec 0.1.486 adds binary XOR `~` and unary NOT `~`: `return ~ x` is a **new** source collision. The prior D234 review at spec 0.1.485 was updated to actual 0.1.486 before approval.

## Comparative precedents: distinct return semantics

| Language / family | Mechanism and warning | Primary documentation |
| --- | --- | --- |
| C | `return` exits function, not arbitrary captured method home | https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf |
| C++ | `return` exits current function/lambda; not Protos lexical home | https://eel.is/c++draft/stmt.return |
| ECMAScript/JavaScript | `return` exits active function, including its callback | https://tc39.es/ecma262/#sec-return-statement |
| Python | `return` belongs to a function definition, not its nested function creator | https://docs.python.org/3/reference/simple_stmts.html#the-return-statement |
| Go | `return` terminates current function, no cross-goroutine return | https://go.dev/ref/spec#Return_statements |
| Rust | `return` leaves current function/closure | https://doc.rust-lang.org/reference/expressions/return-expr.html |
| Self | `^` return from lexical enclosing method; relevant non-local model | https://selflanguage.org/_static/published/self-lang.pdf |
| Smalltalk/GNU Smalltalk | `^` non-local exit in block context | https://www.gnu.org/software/smalltalk/manual/ |
| Ruby | non-lambda block/Proc `return` can escape defining method; lambda has own return behavior and escaped returns can fail | https://docs.ruby-lang.org/en/master/Proc.html |
| Kotlin | bare non-local lambda `return` possible under inline constraints, not general free closures | https://kotlinlang.org/docs/inline-functions.html#non-local-jump-expressions |

Evidence caution: comparative pages explain models and risk, they are not normative Protos authority and no external system tests were run.

## Twelve mandatory GITHUB010 dimensions

Scoring 1–5; H = high support from inspected Protos source, M = model/risk reasoning, L = empirical unknown. Alternatives A = status quo `^`; B1 = strict no-new-collision contextual alias; B2 = owner-selected full contextual alias; E = documentation-only adoption. Scores represent design judgement, not benchmarks.

| Dimension | A | B1 | B2-S1 | E |
| --- | --- | --- | --- | --- |
| 1. correctness / invariants | 5H | 4M | 4M | 5H |
| 2. Protos philosophy | 5H | 3M | 3M | 5H |
| 3. pay only for use | 4H | 3M | 3M | 5M |
| 4. incremental growth | 4M | 3M | 4M | 5M |
| 5. future options | 4M | 3M | 3M | 5M |
| 6. scalability | 5H | 5H | 5H | 5H |
| 7. conceptual simplicity | 5H | 2H | 3M | 4M |
| 8. portability | 5H | 5H | 5H | 5H |
| 9. runtime cost | 5H | 5M | 5M | 5H |
| 10. failure/diagnostics | 5H | 3M | 3M | 4M |
| 11. deferral/migration | 5M | 3M | 2M | 5M |
| 12. evidence maturity | 5H | 3M | 4M | 4M |

B2-S1 accepts a local source-compatibility loss for general ergonomic consistency, without paying for a second semantic engine. Highest adversarial argument: JavaScript/C++ newcomers may wrongly infer local callback return from a familiar word, making `^` pedagogically clearer; explicit docs must teach its non-local meaning.

## Falsification / mandatory regression matrix (not executed)

1. `return a + b * c` -> same `CanonicalReturn`/value as `^a + b * c`; `return -x`, `return ~x` and `return(x)` parse with approved semantics.
2. Nested `each` and contextual `if`/`while`, closure default arguments, module/method invocation retain exact captured return-home authority; no local callback fallback.
3. Escaped closure, completed and foreign Task home -> `InvalidReturn`; Actor and P boundaries and `ensure` cleanup precedence unchanged.
4. `return: x`, `return = x`, `return => x`, `object.return(x)`, `(return)(x)`, `return()`, `return(x,y)` stay ordinary.
5. `return[0] = x`, `return[0].name = x` and `return[0].name: x` preserve entire lvalue; do not swallow into return.
6. `return[0]` and `return { a:1 }` become return of Array/object; grouped ordinary `(return)[0]` and `(return) { a:1 }` remain available.
7. Bare `return` followed by logical NEWLINE -> ordinary separate expression; `return(expression multiline)` works; `return /*block*/ expr` works; `return // line` NEWLINE expr not joined.
8. No `return;` terminator, bare `return` implicit null, new lexer keyword, ASI, host exception, altered `^` or new Truffle opcode.
9. Formatter parse-format-parse preserves spans, comments, grouping and `return` source spelling; LSP/static lint warns unreachable code after either spelling; no wrong 'preceding ^' message after `return`.
10. Focal parser/lexer/runtime/conformance/formatter/LSP/REPL tests; final human-reported `make test`, `make check`, `git diff --check`; compare canonical lowering/return-home observability on both spellings.

## GITHUB021 decision reconciliation

- **Preserved:** one non-local return operation; same `CanonicalReturn`; return-home provenance and Task/Actor/P confinement; `ensure` precedence; Sequence normal completion; six reserved words; D210/D211 contextual patterns; unchanged global D233 bitwise semantics.
- **Explicit forward deltas approved:** contextual precedence for `return(x)`, `return - x`, `return ~ x`, `return[0]`, `return { a:1 }` over their previously valid ordinary meanings; grouped spellings offer an escape.
- **Additional exact boundary disclosed and approved:** no bare-`return` continuation over emitted logical NEWLINE; parenthesized multiline operand allowed; ordinary bare lookup and lvalue forms protected; no implicit null or statement terminator.
- **Approval:** owner first selected B2, then expressly approved exact D234-B2-S1 after complete delta disclosure. No approval for any further incompatible or semantically novel grammar branch.
- **Deferred:** exact source corpus statistics, implementation tests and performance measurement; no claims of PASS.

**DESIGN_APPROVAL=PASS; GITHUB021_INVARIANTS=PASS; DURABLE_DECISION_RECORD=LINKED; PROTOS_PRODUCT_IMPLEMENTATION=NOT_STARTED.**
