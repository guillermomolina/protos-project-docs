# D212 — Post-test Closure loops, contextual suffix until, and pre-test whileFalse

**Status:** RATIFIED AS LANGUAGE DESIGN — owner-approved 2026-10-21; product implementation remains separate.
**Issue:** [D212/#947](https://github.com/guillermomolina/protos/issues/947).
**Implementation:** [I121/#948](https://github.com/guillermomolina/protos/issues/948), not yet executed.
**Explicit approval:** Owner's active-chat statement: “Ok apruebo bodyClosure.untilFalse(conditionClosure) y untilTrue, y el sintactic sugar de untilTrue como {} until(condition). Y agregar el whileFalse”. Recorded without claiming the owner directly authored the [GitHub transcription](https://github.com/guillermomolina/protos/issues/947#issuecomment-6319758145).
**Previous decision, narrowly reopened:** [D210/#941](https://github.com/guillermomolina/protos/issues/941), [cross-reference](https://github.com/guillermomolina/protos/issues/941#issuecomment-6319782484).
**Evidence:** [D212 comparative evidence and invariant audit](../../evidence/D212/D212_LOOP_PROTOCOL_SYNTAX_RESEARCH_AND_INVARIANT_AUDIT_2026_10_21.md).
**Authority:** Non-normative ratification record; normative ownership stays in `guillermomolina/protos/spec/`. No build, test, product edit, implementation completion, or source-revision publication is asserted by this record.

## Selected public protocols

All four selector names are ordinary Object-local Closure-valued slots with standard behaviors defined only for semantic Closure original receivers; normal lookup, extraction, local shadowing, method-home, ordinary override, reflective publication, and zero-argument Closure activation semantics apply:

| Selector | Receiver | One Closure argument | Phase and normal exit |
| --- | --- | --- | --- |
| `whileTrue` (existing) | condition | body | pre-test; stop on canonical `false` |
| `whileFalse` (new) | condition | body | pre-test; stop on canonical `true` |
| `untilTrue` (new) | body | condition | post-test; stop on canonical `true` |
| `untilFalse` (new) | body | condition | post-test; stop on canonical `false` |

For a selected *standard* behavior, evaluate receiver/argument expressions first in the normal left-to-right manner; then validate the original receiver as a semantic Closure and exactly one supplied argument as a semantic Closure **before any callback activation**. Validation does not execute body/condition code or parameter defaults. Each reached activation receives zero caller-supplied positional arguments and follows the ordinary Closure-binding rules; declaration of parameters/defaults does not cause earlier evaluation.

For pre-test protocols: activate condition, test its exact canonical Boolean, and activate body only on the corresponding continue value. Zero body activations on normal termination return canonical `null`. For post-test protocols: activate body first, retain its exact normal result, then activate condition; a normal termination therefore requires at least one body activation. For all protocols, normal termination returns the exact value and identity of the last normally completed body, even when the terminal condition subsequently mutates that object. A condition result other than canonical `true` or `false` causes a fresh standard Error after any preceding effects; no coercion, truthiness, implicit await or Future adoption. A Future produced by the body is an ordinary exact body result; a Future produced by a condition is not a Boolean. Errors, valid/nonvalid non-local returns, cancellation/unwind, `ensure`, suspension/resumption and Futures retain the existing specifications; loops add no Task, Actor, scheduling, hidden polling/checkpoint or control-transfer handler. Do not silently translate `whileFalse` into `not()`: negation uses ordinary potentially overridden message dispatch and is not a universally equivalent protocol.

## Approved contextual expression

```protos
{
    work()
} until(done())
```

This is exactly the ordinary protocol selection and invocation represented by:

```protos
(() => { work() }).untilTrue(() => done())
```

The braced body and the predicate become **two different ordinary zero-parameter lexical Closures**. The body does not establish an enclosing lexical scope for the predicate: a binding created only in the body is invisible to the condition Closure; enclosing captured bindings may be shared by reference. The returned value is the protocol's last body result; it is an expression, not a new value kind.

Recognize only a standalone bare braced body containing a normal `expression-sequence`, followed **without a separating logical NEWLINE** by an ungrouped bare ordinary identifier `until` called with exactly one non-spread condition expression in parentheses. Preserve ordinary whitespace and comments without a logical newline. This exact contextual source form wins over ordinary object construction and an ordinary lexical binding called `until`; that collision is intentional in the owner's explicitly selected bare-brace source shape. Outside it, `{ ... }` still constructs an object, `until` remains unreserved and ordinary member/grouped calls remain unchanged. Do not admit object-composition-only `...item` syntax inside this Closure body. Do not add a new grammar keyword, a `repeat` prefix, `until(condition) { body }`, or sugar for `untilFalse`. Normal postfix chaining of the completed loop expression must still work.

## GITHUB021 approved-invariant delta

- **Preserve** D044's and D210's existing `whileTrue` strict pre-test behavior, normal-result policy, ordinary dispatch, return-home and effect propagation.
- **Explicitly reopen/supersede only** D180/D210's absence of standard `whileFalse`: the owner now approved the complementary pre-test selector. D210 historical records remain unchanged and still accurately describe their original ratification.
- **Preserve** D210's removal of standard `Object.while` and the contextual `while(condition) { body }` form.
- **Preserve** D211's Boolean conditional protocol and contextual `if/else`.
- **Explicitly narrow** the former unconditional bare-brace object-construction source interpretation only for the selected exact `{...} until(condition)` contextual shape. The condition and body still follow ordinary lexical capture rules.

No unrelated syntax, Boolean truthiness, shared lexical block, special scheduler/runtime class, or general no-dot fluent-call syntax is approved.

## Implementation and publication boundary

The D212 decision is independent of implementation. A subsequent `Ixxx` issue owns standard Object publication, native execution, prepared/structured/Bytecode DSL specialization, precise contextual parser/surface AST/canonical lowering, tooling, conformance and spec revision. The human executor performs build/test/check and Protos publication; do not claim green results before they are reported. Review new public slots, compatibility and source-location contracts together. The first implementation slice should establish protocol semantics, followed by contextual syntax and tooling; group adjacent edits within each meaningful publication unit.
