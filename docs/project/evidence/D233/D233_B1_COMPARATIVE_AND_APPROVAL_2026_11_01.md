# D233-B1 — Research, approval and falsification (2026-11-01)

**Approved by owner:** “Apruebo este contrato” after the exact B1 packet. **Issue:** https://github.com/guillermomolina/protos/issues/1016. **Product baseline:** `guillermomolina/protos@fba47c40c4b79b6e580df1c1f78a3fcecbd067e2`, version `0.3.429-SNAPSHOT`, spec `0.1.484`. **Selected language decision:** [D233-B1](../../decisions/language/D233_FIXED_BITWISE_OPERATOR_SUGAR_B1.md). No production edit or test is claimed.

## Verified source facts

`spec/PROTOS_GRAMMAR.md` defines maximal-munch exact token classification, fixed precedence, `^` return, `&&`/`||` lazy and `-` unary/binary. `spec/semantics/VALUES_AND_COLLECTIONS.md` ratifies the six D230 integer messages. `EXECUTION_AND_CONTROL.md`, `CALLABLES.md`, `OBJECT_MODEL.md` preserve strict order, override/home and original-receiver validation. `ProtosLexer.java` already treats `&`, `|`, `~` as candidate characters but rejects them as complete nonstandard spellings. `ProtosParser.java` has recursive unary and precedence-oriented `parseLeftAssociative`; `SurfaceBinary` and `SurfaceUnary` already suffice. `Canonicalizer.lowerBinary/lowerUnary` builds `CanonicalSend` and separate lazy Boolean Closures. `protos/tools/formatter/Structural.protos` concatenates unary operator+operand: `~ -a` could become lexically invalid `~-a` without a fix. SHA256 has 16 direct bitwise calls; the integer conformance file has 129 direct calls (not timing results).

## Comparative precedents

| Language/runtime | Contribution (not authority over Protos) |
|---|---|
| Lua 5.4 | Exact five glyphs incl. dual binary/unary `~`; ignore Lua's finite-width coercion rules. https://www.lua.org/manual/5.4/manual.html#3.4.2 |
| Python/GraalPy | Infinite-bit mathematical model, bitwise precedence and shift floor; `^` conflicts with Protos return. https://docs.python.org/3/reference/expressions.html |
| Smalltalk/Self | Ordinary bitwise messages and operator-mixing constraints; named messages remain a genuine alternative. https://www.gnu.org/software/smalltalk/manual/ |
| Ruby/TruffleRuby | Operator semantics with overridable methods; preserve Protos exact receiver binding, not Ruby behavior. https://www.truffleruby.org/ |
| Java BigInteger | Named unbounded bitwise API distinct from primitive finite arithmetic. https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/math/BigInteger.html |
| JavaScript/GraalJS | Number 32-bit versus BigInt warns against implicit widths/coercions. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_AND |
| Rust/Go | Closed operators but different precedence and word semantics; no universally mandatory order. https://go.dev/ref/spec#Arithmetic_operators |
| Kotlin | Named infix functions need a distinct global grammar rule. https://kotlinlang.org/docs/numbers.html#bitwise-operations |
| Scala/Haskell/Swift | User fixity and operator declarations are bigger institutions, not justified here. https://www.haskell.org/onlinereport/haskell2010/haskellch4.html#x10-830004.4.2 |
| Truffle SimpleLanguage/Espresso/Sulong/Pkl | No Truffle-mandated bitwise grammar; backend is not language authority. https://www.graalvm.org/latest/graalvm-as-a-platform/language-implementation-framework/ |

## Alternatives

A: only ordinary messages (zero grammar cost). B1: five fixed glyphs (`&`, `|`, binary/unary `~`, `<<`, `>>`) to six existing sends (selected). B2: `^^` as XOR (re-admits D149 rejected glyph, weaker prior art). B3: `^` as XOR (conflicts with return). B4: named infix `xor` (new grammar category). C: separate symbolic selectors (divergent overrides/extraction). E: fully custom operators (reopens D149). F: subset excluding XOR (irregular).

## 12-factor comparison: score/confidence/reason

| Criterion | A current | B1 selected | B2 ^^ |
|---|---|---|---|
| Correctness | 5/H current | 5/M same canonical sends | 5/M same |
| Protos philosophy | 5/H ordinary | 5/M sugar only | 4/M awkward glyph |
| Present proportionality | 5/H no cost | 3/M one real consumer | 3/M same |
| Incremental growth | 5/H extensible | 4/H closed scope | 4/H closed scope |
| Future resilience | 5/H open | 4/M five tokens fixed | 3/M ^^ obligation |
| Scalability | 5/H existing | 5/M same runtime dispatch | 5/M same |
| Conceptual simplicity | 5/H one spelling | 4/H five tokens | 3/H six glyphs |
| Portability | 5/H current | 5/H no host width | 5/H same |
| Resource cost | 5/H zero | 4/M frontend only | 4/M frontend only |
| Operability | 5/H known | 4/M lex/format risks | 3/M caret adjacency |
| Reversibility | 5/H add later | 3/H removal breaks source | 2/H removal breaks |
| Evidence maturity | 5/H production | 4/M Lua + AST | 3/M weaker precedent |

## Adversarial and incremental gates

Lex valid: `a~b`, `~a`, `a ~ ~b`, `~(~a)`, `a&&b`, `a & -b`, `a << -1` (later Error). Lex invalid: `a~~b`, `~~a`, `a&-b`, `a<<-1`, `a>>=b`, `a>>>b`, `a~=b`, `a|>b`, `^^a`, `!!a`. Incomplete `a ~` then newline continues; complete `a` then newline `~b` separates. `a ~ b & c` is XOR of `a` and `(b & c)`. Strict operand evaluation, ordinary overrides, `^`, Boolean laziness and integer normalization must not change.

**Pay now:** fixed global grammar+formatter/tool maintenance; no new numeric runtime. **Grow later:** explicit width/rotates and user fixity remain separate. **Deferral cost:** adding sugar later is source-compatible; removing after publication breaks callers. **Overengineering risk:** one direct production consumer. **Strongest argument against B1:** binary `~` unfamiliar outside Lua. **GITHUB021:** D149 no arbitrary operators/fixity, D230 six methods, `^` return and Boolean laziness preserved; D233 is an independently approved extension to fixed source tokens.

**Approval provenance PASS; owner-invariant consistency PASS.** No tests, performance benchmarks, product change, version/spec bump, or runtime/NATIVE verification performed.