# D199 — Fluent Boolean syntax: falsification evidence (2026-10-13)

**Role:** Historical, non-normative design evidence; no executable implementation or test-run result.
**Issue:** [D199/#879](https://github.com/guillermomolina/protos/issues/879).
**Separate follow-up:** [D200/#880 — Io-style Boolean return contract](https://github.com/guillermomolina/protos/issues/880).
**Observed product revision:** [55be9274bf90677a2ac243c77450476150b1c8c6](https://github.com/guillermomolina/protos/commit/55be9274bf90677a2ac243c77450476150b1c8c6).
**Observed project-docs baseline:** [854ce1ee8d04e16f3dfa662bd6ba4d1a598db26c](https://github.com/guillermomolina/protos-project-docs/commit/854ce1ee8d04e16f3dfa662bd6ba4d1a598db26c).
**Owner's resolution:** «ok creo que lo vamos a cerrar como no aprobado», with an explicit request to create a separate Io-contract design issue. This resolves D199 as **no new syntax approved**, not a decision against Io returns.

## Source authority and existing forms

The checked [grammar](https://github.com/guillermomolina/protos/blob/55be9274bf90677a2ac243c77450476150b1c8c6/spec/PROTOS_GRAMMAR.md) §§4–5, 14–15, 18–21, 36 and [parser](https://github.com/guillermomolina/protos/blob/55be9274bf90677a2ac243c77450476150b1c8c6/src/main/java/com/guillermomolina/protos/parser/ProtosParser.java) (`parsePostfixFoundation`, `parseCallSuffix`, `parseObjectBody`) make these different expressions:

```protos
Animal { name: "Rex" }        // object construction; parent Animal
Animal() { work() }           // call with trailing Closure
condition.ifTrue() { A() }    // call with trailing Closure
```

The grammar currently does not admit a *general* point-free message send `receiver selector { body }`; normal member calls use `receiver.selector(arguments)`. The [object-expression parser tests](https://github.com/guillermomolina/protos/blob/55be9274bf90677a2ac243c77450476150b1c8c6/src/test/java/com/guillermomolina/protos/parser/ProtosParserObjectExpressionTest.java) include `animal {}`, `library.models.animal {}`, and `animal {}.name`.

## Falsification cases for proposed sugar

### F1 — Removing a branch can change the semantic category

```protos
condition.ifTrue {
    A()
}
```

Under today's grammar this is **object construction with `condition.ifTrue` as parent**, not conditional invocation. A special whole-pair reinterpretation of `condition.ifTrue { A() } ifFalse { B() }` makes removing the second part silently restore the object-construction meaning. An object-body expression may run unconditionally. A user should not have to know whether the second branch exists to determine what the first branch means.

### F2 — Postfix precedence on a full Boolean expression

```protos
x > 0 ifTrue { A() }
!ready ifTrue { A() }
```

If `identifier { body }` is added as another postfix suffix, then the selector binds to `0` or to `ready` before the comparison or negation, rather than receiving `x > 0` or `!ready`. Grouping `(x > 0) ifTrue { A() }` would avoid this but substantially reduces the savings. Giving the new suffix *lower* precedence instead creates two precedence classes for otherwise ordinary message sends. See [grammar §21](https://github.com/guillermomolina/protos/blob/55be9274bf90677a2ac243c77450476150b1c8c6/spec/PROTOS_GRAMMAR.md).

### F3 — Multiline chaining and existing expression separators

```protos
condition ifTrue { A() } ifFalse { B() }
```

The proposed syntax might remain one expression when on the same line. But:

```protos
condition ifTrue {
    A()
}
ifFalse {
    B()
}
```

ends the first expression at the newline after `}` under today's grammar. The standalone `ifFalse { ... }` already resembles parented object construction when `ifFalse` is an ordinary binding. A new automatic identifier-based continuation would conflict with the current strict newline separation and the one explicit next-line *leading dot* exception. See [grammar §§4–5](https://github.com/guillermomolina/protos/blob/55be9274bf90677a2ac243c77450476150b1c8c6/spec/PROTOS_GRAMMAR.md) and [continuation tests](https://github.com/guillermomolina/protos/blob/55be9274bf90677a2ac243c77450476150b1c8c6/src/test/java/com/guillermomolina/protos/parser/ProtosParserMemberContinuationTest.java).

### F4 — Mixed dotted and undotted spellings diverge

```protos
condition ifTrue { A() }.ifFalse { B() }
```

The proposed first operation would be a callback send; the second `.ifFalse { B() }` still matches the existing parented object form. Generalizing `.method { ... }` to be a call would instead reinterpret an already-valid object-construction expression, including `library.Animal { ... }`.

### F5 — Syntax cannot fix the return-value contract

The current [Boolean specification](https://github.com/guillermomolina/protos/blob/55be9274bf90677a2ac243c77450476150b1c8c6/spec/semantics/VALUES_AND_COLLECTIONS.md) returns the **selected callback result** for `true.ifTrue(A)` and `false.ifFalse(B)`, and `null` when either branch is skipped:

```protos
true.ifTrue(() => { 42 }).ifFalse(() => { 99 })
```

The second call receives `42`, not the original `true`. Changing only the skipped-branch return cannot make ordinary chains work for arbitrary callback results. Rewriting the chain in the parser to `ifTrueIfFalse` would change ordinary dispatch and evaluation semantics. Full Io-style receiver returns are a *separate public contract decision*, now D200.

Real observable dependency, not theoretical only:

- [`conditional-single-branch.protos`](https://github.com/guillermomolina/protos/blob/55be9274bf90677a2ac243c77450476150b1c8c6/protos/tests/conformance/boolean/conditional-single-branch.protos): exact `42`, `43`, and `null` conformance.
- [`callback-dispatch-and-control.protos`](https://github.com/guillermomolina/protos/blob/55be9274bf90677a2ac243c77450476150b1c8c6/protos/tests/conformance/boolean/callback-dispatch-and-control.protos): selected-result arithmetic, special callables, nonlocal return, capture.
- [`ProtosBytecodeRootNode.PreparedBooleanCall`](https://github.com/guillermomolina/protos/blob/55be9274bf90677a2ac243c77450476150b1c8c6/src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java): selected callback index and only-one-callback preparation; the unselected callback **body** is not executed. Inlining already exists independently of source sugar.

## Comparators and the exact lesson

- [Io `Object.io`](https://github.com/IoLanguage/io/blob/master/libs/iovm/io/Object.io) + [`IoObject.c`](https://github.com/IoLanguage/io/blob/master/libs/iovm/source/IoObject.c): sequential `ifTrue` / `ifFalse` messages work because their return contract preserves the Boolean receiver, **not because Smalltalk-style multipart selectors or special parser desugaring are used**. Io also has a different argument evaluation mechanism.
- Smalltalk `ifTrue:ifFalse:` is **one selector**, not ordinary message chaining.
- [Groovy's command chains](https://groovy-lang.org/dsls.html#_command_chains) use a different general grammar; this is not evidence they can be copied into Protos without the object-body, precedence and newline conflicts.

## Exact rejection boundary / preserved invariants

The project owner reviewed the concrete adverse examples and selected closure of D199 **without approval**. No grammar, object model, current trailing-Closure syntax, Boolean runtime return, `ifTrueIfFalse`, `caseOf` or `match` change follows. [D050](https://github.com/guillermomolina/protos/issues/208) and [D051](../../decisions/language/D051_CONDITIONAL_SURFACE_BOUNDARY.md) stay ratified and unchanged. New Io return-contract analysis is explicitly segregated into [D200/#880](https://github.com/guillermomolina/protos/issues/880).

```text
D199_OWNER_OUTCOME=REJECT_NEW_SUGAR_NOW
D199_NEW_SYNTAX_APPROVED=NO
D199_PRODUCT_OR_NORMATIVE_CHANGE=NO
D050_D051_PRESERVED=YES
D200_IO_RETURN_CONTRACT_APPROVED=NO
D200_INVESTIGATION=OPEN
AGENT_RAN_PRODUCT_TESTS=NO
```

**Evidence class:** read-only inspection of the above source files and direct owner instruction, not an independent execution, benchmarking or end-to-end parser test. The rejection does not prohibit later *separately approved* proposals, nor does it ratify any D200 candidate.

**AI provenance:** prepared with ChatGPT; no independent human verification of the record is claimed.
