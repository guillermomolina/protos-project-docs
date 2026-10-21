# D212 — Comparative research, failure cases and exact owner-decision audit

**Status:** DESIGN RESEARCH / RATIFICATION EVIDENCE, 2026-10-21. No source implementation, compilation or test run is claimed.  
**Decision:** [D212/#947](https://github.com/guillermomolina/protos/issues/947), [approved exact candidate](https://github.com/guillermomolina/protos/issues/947#issuecomment-6319758145).  
**Durable decision:** [D212 post-test and whileFalse](../../decisions/language/D212_POST_TEST_UNTIL_AND_WHILE_FALSE.md).  
**Earlier normative baseline:** [D210/#941](https://github.com/guillermomolina/protos/issues/941), [D210 ratification](../../decisions/language/D210_CLOSURE_WHILE_SELECTOR_AND_CONTEXTUAL_SYNTAX.md).  
**Ownership:** All language rules in this document are evidence of the owner-selected intended spec delta, **not** a substitute for future normative edits under `guillermomolina/protos/spec/`.

## Problem and origin

The motivating Bubble Sort pseudocode from Wikipedia uses `repeat ... until`: perform at least one complete pass, then decide whether another pass is required. Existing Protos `condition.whileTrue(body)` and contextual `while(condition) {body}` check before the first body. Writing `body(); while(...) { body() }` is expressive but duplicates source and can alter ordinary message/override effects. A distinct post-test phase is justified. The owner also challenged the asymmetric omission of `whileFalse`: forcing `not()` is not generally observationally equivalent because negation is an ordinary potentially overridable Boolean message; compare standard `ifTrue` and `ifFalse`.

## Prior art: distinct language approaches (GITHUB010 breadth)

1. **Smalltalk/Squeak:** receiver/argument block messages `[condition] whileTrue: [body]`, `whileFalse:`; `doWhileTrue:`/`doWhileFalse:` variants use the body as receiver. This demonstrates a natural *body-receiver* post-test design without new statement categories. [GNU Smalltalk BlockClosure manual](https://www.gnu.org/software/smalltalk/manual-base/gst-base.html).
2. **GemStone Smalltalk:** unary `[body; finalBoolean] untilTrue` and `untilFalse` demonstrate post-test control, but require the receiver's final result to control termination; this prevents retaining arbitrary body results independently.
3. **Lua:** `repeat {body} until predicate` is post-test and permits predicate references to local bindings in the repeat block, whereas two independent Protos lexical Closures intentionally do **not** share body-local scope. [Lua 5.4 manual, §3.3.4](https://www.lua.org/manual/5.4/manual.html#3.3.4).
4. **JavaScript:** `do {body} while(predicate)` is post-test, contrasting with `while` pre-test. [MDN do...while](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/do...while).
5. **Swift:** `repeat {body} while(predicate)` explicitly advertises the later test. [Swift control flow](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/controlflow/).
6. **Ruby:** ordinary `until condition` is typically **pre-test**, so bare `until` is not cross-language proof of post-test semantics. [Ruby control expressions](https://docs.ruby-lang.org/en/master/syntax/control_expressions_rdoc.html).
7. **Rust:** general `loop` plus `break` can express later-tested exits and optional loop values without dedicated `until` spelling. Protos currently lacks the same separately ratified `break` result mechanism; do not invent one. [Rust loop expressions](https://doc.rust-lang.org/reference/expressions/loop-expr.html).
8. **Io and other prototype/message languages:** control constructs based on ordinary sends show that a language can expose loop protocols rather than privileged statements; specifics are not automatically transferable to Protos's semantic Closure domain and receiver binding.

Distinct families examined: message/Closure protocols; privileged structured post-test syntax; general-loop composition. The owner's selection composes standard Closure messages with **narrow, context-sensitive** source sugar.

## Candidate set and owner selection

- **A—Status quo:** retain `whileTrue` only and manual composition, no suffix syntax. Minimal present surface, but loses direct post-test expression and forces Boolean `not()` when inverting continuation.
- **B—Unary single-block post-test:** `body.untilTrue()` / `untilFalse()`; compact but body must produce the Boolean itself and cannot independently provide an arbitrary result as Protos loops already do.
- **C—Two independent Closure post-test protocols:** `body.untilTrue(condition)` / `untilFalse(condition)`, retain `whileTrue` alone; clean phase separation but omits source ergonomics and pre-test symmetry.
- **D—Selected package:** C, plus `condition.whileFalse(body)` and contextual `{ body } until(condition)` for `untilTrue` only. The four protocols are uniform ordinary Closure selectors with strict results, while one narrow contextual construct matches algorithm pseudocode.
- **Rejected extra mechanisms:** `condition.untilTrue(body)` (opposite receiver orientation); single-block `untilTrue()` semantics; `repeat {body} until(condition)` mandatory prefix; `until(condition) {body}` (front-loaded post-test predicate); generic postfix modifier on any expression; implicit truthiness, await, break/continue or scope sharing. They either conflict with selected source form, increase semantic categories, or weaken strict protocol identity.

## GITHUB010 12-dimensional scores

Scores 1–5 (5 best). Every cell includes short evidence and confidence H/M/L; these are comparative judgments, not performance measurements.

| Dimension | A status quo | B unary | C two-Closure | D owner-selected |
| --- | --- | --- | --- | --- |
| 1 Correctness / invariants | 5H existing | 2M conflates Boolean/body result | 5M distinct results/phase | 4M extra contextual collision to test |
| 2 Protos alignment | 4H minimal | 3M body forced Boolean | 5M ordinary Closure messages | 4M orthogonal messages, one contextual exception |
| 3 Pay for present need | 5H no new API | 3M adds protocols with result limitation | 4M adds two standard methods | 3M adds third method, parser and tools |
| 4 Incremental growth | 4H extension remains possible | 3M hard result migration | 5M syntax can grow later | 5M all selected needs solved now |
| 5 Future-option resilience | 5H no commitment | 3M one-block commitment | 5M flexible syntax | 4M deliberate suffix interpretation |
| 6 Scalability | 4H existing Closures | 4M no scheduler needed | 4M local callback state | 4M same + small static parser cost |
| 7 Conceptual simplicity | 5H one loop family | 4M compact but overloads Boolean result | 4M two phase contracts | 4M symmetric four-way protocol, source exception |
| 8 Portability / freedom | 5H baseline | 5M VM-independent | 5M VM-independent | 5M VM-independent |
| 9 Runtime / resources | 3M composition overhead at use | 4L no measurements | 4L two-Closure cost unmeasured | 4L no proof of speedup; no hidden state |
| 10 Failure / operability | 4H known model | 3M Boolean/body ambiguity | 4M strict errors | 4M strict errors plus source tooling |
| 11 Deferral / migration | 5H adding later bounded | 2M public result contract costly to reverse | 5M additive post-test | 3M newly privileged suffix + standard slots public |
| 12 Evidence / implementation risk | 5H existing tests | 4M GemStone precedent | 4M Smalltalk precedent | 4M established D210/D211 contextual tooling; novel braces |

**Non-compensating gates:** D's new global `Object` slots increase reflection/API/maintenance surface for all contexts; the suffix reinterpretation of bare `{...}` is a real 0-to-1 source-grammar exception. No arithmetic score erases these costs. The owner explicitly accepted the four-protocol symmetry and terse algorithmic suffix. B fails exact body-result preservation as normally expected in Protos. C is the smallest post-test primitive solution; the selected D adds two independently motivated ergonomic capabilities, not speculative scheduling machinery. Adding omitted capabilities later is bounded, so we do not claim impossibility of deferral.

## Adversarial observable cases

- **Pre-test vs post-test:** `whileFalse` terminates normally without a body if the first test is `true`; `untilTrue` always activates body before its first `true` test.
- **Strict Boolean:** `null`, user object, or Future as a condition result signals a fresh Error after any already completed body effects. No `not()` normalization, truthiness or implicit `Future.value()`.
- **Result identity:** preserve the exact last normal body object, including Future identity; terminal condition may mutate that same object but cannot substitute a new result. If terminal condition errors, there is no normal loop result.
- **Validation/evaluation:** evaluate all receiver and argument expressions before invoking selected behavior; validate semantic Closure domain and exactly one Closure argument before any callback activation. Zero-arg activation applies defaults only when that particular callback runs.
- **Override:** `condition.whileFalse` and `body.untilTrue` must use ordinary method selection and local shadowing; native/compiled optimizations may recognize only the exact standard behavior at its legitimate home.
- **Transfer:** a valid `^` nonlocal return exits with preserved return home; `InvalidReturn`, Error and cancellation unwind propagate unchanged. No extra `ensure` or structured child scope.
- **Replay:** suspended body or condition resumes exactly once; no reactivation or repeated effects on deoptimization/carrier change. The protocol adds no hidden cancellation poll.
- **Lexical scope:** `n: 0; { n = n + 1 } until(n >= 3)` shares outer `n` by capture, but `{ internal: 1 } until(internal == 1)` cannot see the body-local slot.
- **Syntax collision:** `{ f() } until(done())` has special Closure body meaning; `{ f() }` still constructs an object; lexical `until: someClosure` loses precedence **only** in the exact contextual shape; grouped/member `until` remains ordinary.
- **Separators:** a logical NEWLINE between `}` and `until` must not join the expression. Commas/spreads/multiple arguments are excluded. Parenthesized object constructions and object-only spread/composition items must not be silently captured.
- **Compilation:** same observable behavior in interpreter, Tier 1, Tier 2 and Native, including source tagging and debugger stepping; no claim of validation until human executor reports it.

## GITHUB021 invariant check

| Existing selected invariant | D212 disposition |
| --- | --- |
| D044 strict semantic Closure loops, NLR and Boolean | PRESERVE |
| D180/D210 exclusive `whileTrue`, no standard `whileFalse` | REOPEN only absence of `whileFalse`; explicit owner approval in D212 |
| D201/D210 last-body result, zero `null`, identity | PRESERVE and generalize |
| D210 removed `Object.while` | PRESERVE; no alias reinstated |
| D210 contextual `while(cond) {body}` | PRESERVE |
| D211 contextual `if/else` | PRESERVE |
| Bare-brace object expression | NARROW explicit owner-selected `{...} until(cond)` exception; ordinary objects otherwise unchanged |
| Independent Closure lexical capture and external return home | PRESERVE; no Lua-style shared local |
| Cancellation/Future/Actor/Task and ordinary overrides | PRESERVE |

Approval provenance is the owner's exact interactive utterance transcribed in D212, not a manufactured owner-authored comment. The prior owner's “leave `whileFalse` for later” was explicitly superseded by the subsequent approval including `whileFalse`. No other D210 or D211 invariant is reopened.

## Future stress, costs and escape paths

No shared mutable loop mechanism is introduced per process/Task/Actor; only normal lexical closures and scalar local last-body value. Native or alternative Truffle/Bytecode DSL representations remain free to specialize without bypassing selected Object slots. No serialization format, scheduler policy, implicit await, task cancellation boundary or new value kind is added. If future users demand a Lua-style shared lexical scope, that cannot be silently retrofitted into two independent Closures; it requires a distinct approved construct. If `whileFalse` proves little-used, the public standard slot cannot be removed without a compatibility decision. The selected suffix sugar is semantically defined by the underlying ordinary method message, not by Java lowering details.

**Strongest argument against D:** Protos adds three standard slots and a parser exception to ordinary object-body semantics for a capability expressible by a library or by composing existing `whileTrue`; the source collision is irreversible for previously valid uses of same-shape lexical `until`. The owner's expressed preference for symmetric polarity, exact post-test semantics and Bubble Sort pseudocode justifies that tradeoff. No concrete numerical performance or compiled instruction-count win is claimed.

## External Truffle evidence for subsequent Ixxx handoff (not language authority)

- [GraalVM Bytecode DSL introduction](https://github.com/oracle/graal/blob/master/truffle/docs/bytecode_dsl/BytecodeDSL.md): language semantics remain guest-language-owned; bytecode encoding, tiers, continuation support and optimizations are implementation details.
- [Truffle Bytecode DSL UserGuide](https://github.com/oracle/graal/blob/master/truffle/docs/bytecode_dsl/UserGuide.md): structured `While` and its condition/body ordering; no need to create a new guest-language primitive merely because the backend uses a loop operation.
- [Truffle `LoopNode` Java API](https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/nodes/LoopNode.html): repeating nodes, loop OSR and condition profiles; not permission to introduce a Protos-visible polling or scheduling boundary.
- [GraalJS/Truffle optimization examples](https://github.com/oracle/graal/blob/master/truffle/docs/Optimizing.md): `WhileNode.WhileDoRepeatingNode` and `OptimizedOSRLoopNode` show specialized loop machinery. Protos may reuse its existing selected-standard-behavior fast path only with correct lookup guards.
- GraalPy, TruffleRuby, Espresso, Sulong, Pkl and SimpleLanguage are comparative runtime families, not direct semantic authorities for Protos; the feature changes only the guest control protocol, not platform ownership, host interop, packaging or system ABI. No external language repository checkout is a prerequisite for the implementation slice.

## Publication boundary

This is a ratified *design*, not a Protos product implementation. Future changes to `spec/`, Java and tests must be performed in the product repository by a separately tracked Ixxx implementation slice and verified by the human executor. No build or performance evidence has been manufactured.
