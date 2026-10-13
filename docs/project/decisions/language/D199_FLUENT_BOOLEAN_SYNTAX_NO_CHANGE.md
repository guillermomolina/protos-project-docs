# D199 — Fluent Boolean conditional syntax: no change approved

**Decision status:** **CLOSED / OWNER-REJECTED** on 2026-10-13: no D199 syntactic-sugar candidate approved, no normative language change.
**Owning live issue:** [D199/#879](https://github.com/guillermomolina/protos/issues/879).
**Evidence:** [D199 source-grounded falsification, 2026-10-13](../../evidence/D199/D199_FLUENT_BOOLEAN_SYNTAX_FALSIFICATION_2026_10_13.md).
**Distinct open decision:** [D200/#880 — consider Io-style Boolean returns](https://github.com/guillermomolina/protos/issues/880).
**Observed product revision:** [55be9274bf90677a2ac243c77450476150b1c8c6](https://github.com/guillermomolina/protos/commit/55be9274bf90677a2ac243c77450476150b1c8c6).
**Normative authority unchanged:** `spec/PROTOS_GRAMMAR.md`, `spec/semantics/VALUES_AND_COLLECTIONS.md`, `spec/semantics/CALLABLES.md`, `spec/semantics/EXECUTION_AND_CONTROL.md`.

This is a durable **non-normative decision/rejection record**, not a replacement for any ratified `spec/` contract or an assertion of product testing.

## Owner resolution and scope

After investigating point-free or parentheses-free chained Boolean syntax, testing proposed rules against existing source semantics and seeing counterexamples, the project owner explicitly instructed:

> ok creo que lo vamos a cerrar como no aprobado

The owner also requested a **new issue** to determine whether Protos should adopt **Io-style ordinary Boolean message returns**. The resulting outcome is:

- **Do not introduce** `condition ifTrue { A() } ifFalse { B() }`, a general `receiver selector { body }` message surface, or `condition.ifTrue { A() } ifFalse { B() }` as newly accepted sugar in D199.
- **Do not silently repurpose** currently valid object construction `foo { ... }` or `foo.member { ... }`.
- **Do not change** `ifTrue`, `ifFalse`, `ifTrueIfFalse`, `and`, `or`, `whileTrue`, `each`, `caseOf`, `match`, argument evaluation, or callback activation as a consequence of this rejection.
- **Do not infer rejection of Io-style return contracts.** D200/#880 now owns a fresh, explicitly independent substantive Dxxx investigation; no Io candidate is approved.
- This rejection is **for the current D199 proposal**, not a permanent language-wide prohibition on any future syntax design. New scope requires fresh evidence and explicit owner approval.

## What stays available

The currently ratified [D050](https://github.com/guillermomolina/protos/issues/208) Boolean behavior and [D051](D051_CONDITIONAL_SURFACE_BOUNDARY.md) source boundary remain:

```protos
condition.ifTrue() {
    A()
}

condition.ifFalse() {
    B()
}

condition.ifTrueIfFalse(
    () => { A() },
    () => { B() }
)
```

The two-message spelling:

```protos
condition.ifTrue(() => { A() }).ifFalse(() => { B() })
```

is **syntactically ordinary** in Protos but **not** presently equivalent to two-branch conditional selection: the second message receives the *result* of the first. D199 did not alter this. Similarly, the existing `condition.ifTrue() { ... }` and `.ifFalse() { ... }` trailing Closure sugar remains the only approved abbreviated invocation in this scope.

## Why the new syntax was not approved

The observed grammar and parser create non-trivial costs that outweigh the current ergonomic benefit:

1. `condition.ifTrue { A() }` already has an ordinary *object-construction* interpretation. A special paired rewrite changes meaning when a second branch is added/removed.
2. A generic undotted send of postfix precedence misbinds `x > 0 ifTrue { ... }`; lowering its precedence would introduce a second method-send precedence class.
3. The strict `NEWLINE` expression separator only exempts a leading structural `.`; automatically continuing bare `ifFalse` after `}` would introduce a further special continuation rule. A formatter could otherwise change the program.
4. Mixing `condition ifTrue { A() }` and `.ifFalse { B() }` reintroduces the object-construction ambiguity.
5. The syntax alone does not address the **separate return-value incompatibility**: currently executed `ifTrue`/`ifFalse` returns the callback's exact result, not necessarily the original Boolean. Current conformance tests explicitly rely on this.

Full primary-source paths and concrete counterexamples: [D199 evidence](../../evidence/D199/D199_FLUENT_BOOLEAN_SYNTAX_FALSIFICATION_2026_10_13.md).

## Philosophy, reversibility and GITHUB021 invariant check

The outcome retains the simplest existing mechanism and incurs **zero new grammar, runtime, parser, formatter, LSP or public syntax cost**. The strongest argument against the rejection is the readability of a Groovy-/Io-like command-chain syntax. That benefit does not eliminate the above precedence, object body, continuation and compatibility problems. The cost of deferring pure syntax is limited: a later explicitly approved design can add an unambiguous form without discarding current syntax.

| Ratified or owner-confirmed invariant | Effect of rejecting syntax |
| --- | --- |
| D050 result-producing selected Boolean callbacks | Preserved without reinterpretation |
| D051 ordinary method dispatch and no privileged `if`/`else` grammar | Preserved |
| `foo { ... }` vs `foo() { ... }` object/Closure distinction | Preserved |
| Ordinary evaluation, selected callback activation, NLR/errors/suspension | Preserved |
| Owner's optimization-neutral principle: Protos lowerer can aggressively inline ordinary control messages before Graal JIT | Preserved; source punctuation does not decide optimization eligibility |
| D131 ordinary extensible `matcher.match(subject)` and `caseOf` result ABI | Unchanged |
| D200 Io return-contract question | Explicitly unresolved; not implied by D199 closure |

```text
DECISION_APPROVAL_PROVENANCE=OWNER_EXPLICIT_REJECTION
D199_OUTCOME=NO_SYNTAX_CHANGE_APPROVED
D199_CLOSED_AS=NOT_PLANNED
D199_IMPLEMENTATION_REQUIRED=NO
CURRENT_NORMATIVE_SPECIFICATION_CHANGED=NO
CURRENT_PRODUCT_CHANGED=NO
D050_D051_REOPENED=NO
D200_IO_RETURN_POLICY_APPROVED=NO
D200_STATUS=OPEN_INVESTIGATION
GITHUB021_INVARIANT_DELTA=NONE
```

This is a deliberately bounded **negative design decision**. It closes D199 without pretending a previously rejected candidate needs further comparative scoring or a product implementation gate. D200 must independently follow the complete GITHUB010 process before requesting owner approval.

**Validation class:** documentation-only direct repository publication; no build, tests, specification change or product commit performed by the agent.
