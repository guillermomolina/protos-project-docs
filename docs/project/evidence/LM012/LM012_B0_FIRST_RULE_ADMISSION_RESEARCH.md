# LM012-B0 — First useful lint-rule admission research

**TYPE=INVESTIGATION. Status: COMPLETE — RULE-SPECIFIC OWNER SELECTION PENDING. No implementation, tests or builds.**

Date: 2026-10-08. Parent: [LM012/#671](https://github.com/guillermomolina/protos/issues/671). Governing policy: [D194/#842](https://github.com/guillermomolina/protos/issues/842), explicit owner-approved modified B ratified at [the canonical tooling decision](../../decisions/tooling/D194_CANONICAL_LINT_DIAGNOSTICS_RULE_AND_FIX_POLICY.md), docs revision `2b12b4a59cbc6bce49356911ff1bb9e46f176bd9`.

Published source reference inspected: `guillermomolina/protos@3bb1278d91ee5cea98031462be2a5c4dd3c89019` during LM012-A0. This follow-up is **reasoned source inspection only**; any concurrent product revision must be rechecked immediately before implementation. No execution, benchmarks, local parser probes, automated checks, code changes or source revision bumps were performed.

## Decision question

Does a *specific* Protos source-only check demonstrably catch a valid but likely mistaken program, without duplicating parsing/formatting or making unsound inferences over open prototype lookup, mutable closures, effectful matcher evaluation or ordinary messages?

D194 admits correctness defaults only on both soundness and demonstrable usefulness **and** after separate rule-specific selection. This B0 packet proposes but **does not** select a rule ID, severity, default status or edit behavior.

## Audited semantic/source proof basis

- `spec/PROTOS_GRAMMAR.md`, expression-sequence (§7; lines near 546–570): semicolon/newline-separated expressions become ordered `Sequence(expressions)`; the syntax does not create general JS-style expression statements.
- `spec/semantics/EXECUTION_AND_CONTROL.md` §8.2, lines ~411–421: a normally completed Sequence evaluates left to right and returns only the final expression value. Earlier expression **values are not Sequence results**. Ordinary effects of earlier nonliteral expressions are **not** discarded.
- `src/main/java/com/guillermomolina/protos/parser/ast/SurfaceSequence.java`: exact ordered `expressions()` and `SourceSpan`.
- `src/main/java/com/guillermomolina/protos/parser/ast/SurfaceLiteral.java`: closed `Kind` set `NUMBER`, `STRING`, `TRUE`, `FALSE`, `NULL`; exact source span.
- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisCore.java`: real parser on an immutable snapshot, no guest execution, on-demand source-layout fallback.
- `spec/PROTOS_GRAMMAR.md` closure body (§15; lines ~1130–1145) and module program grammar: only use the proven Sequence boundary; do not extend findings to object composition/body, Map construction entries, argument lists, or every nested expression.
- `spec/semantics/VALUES_AND_COLLECTIONS.md`, Map construction initial associations (lines ~470–566): duplicate/equal *runtime* keys conflict, but ordinary `hash` and `==` are dispatched behavioral protocols, and creation does **not** perform compile-time duplicate checking. Do **not** infer that two identically spelled keys statically guarantee the runtime duplicate error without controlling the selected factory and key methods.
- `spec/semantics/VALUES_AND_COLLECTIONS.md`, default equality/hash (lines ~2640–2670): prototypes may override `==`/`hash`; this prevents an unconditional static duplicate-Map-key error. `D096` match redundancy likewise remains separate.

## Candidate comparisons

| Candidate | Distinct user value | Sound proof domain | Decision |
| --- | --- | --- | --- |
| A. **Discarded standalone literal value in a non-final explicit Sequence position** | Catches accidental no-op such as a forgotten assignment, stray numeric/boolean/string expression, or leftover debugging marker; not a formatting difference | Strictly `SurfaceLiteral` that is a *direct non-final child* of a proven `SurfaceSequence`; report only that its value does not become the Sequence normal result | **RECOMMEND A as a first low-severity rule candidate**, with exclusions below; final user value still requires owner approval |
| B. Identical literal keys inside `%{...}` | Potentially catches failure-prone Map construction | Same textual/value spelling is provable, but factory eligibility and ordinary key `hash`/`==` may be overridden; "always runtime Error" is not proven for arbitrary modules | **DO NOT admit as a correctness/error rule now**; at most later optional "suspected duplicate" phrasing after separate justification |
| C. Trailing whitespace or repeated blank lines | Mostly hygiene and generic source styling | Exact tokens/lexical payload might prove boundaries; triple-double Strings make blind regex unsafe | **Reject as LM012 launch rule**: D183/LM011 formatter already owns the meaningful operation |
| D. Undefined/unused names, unreachable match arms, generic after-return, guessed compiler warnings | Potentially higher value | Dynamic lookup, live lexical state, effects and D096 show absence of broad negative proofs | **Reject until specific stronger proof authority is ratified** |
| E. Build rule/config/plugin framework before first check | Infrastructure only | No value proof, adds policy and dormant implementation | **Reject under pay-as-you-grow** |

Precedents: [ESLint `no-unused-expressions`](https://eslint.org/docs/latest/rules/no-unused-expressions) warns on expressions known not to affect program state while exempting expressions that may have effects; [Ruff `B018`](https://docs.astral.sh/ruff/rules/useless-expression/) flags useless expressions and explicitly exempts last notebook-cell expressions because they can be presented interactively. Neither JavaScript's statement model nor Python's AST analysis may be copied into Protos; Protos's **own ordered Sequence** is the only proof boundary. This is a focused rule-specific complement to D194's already-completed broad GITHUB010 systems comparison, not a new design-space study.

## Exact proposed rule contract — **not selected**

```text
PROPOSED_RULE_ID=protos/discarded-sequence-literal
PROPOSED_CATEGORY=CORRECTNESS_HINT
PROPOSED_SEVERITY=HINT
PROPOSED_INITIAL_ENABLEMENT=DEFAULT_ONLY_WITH_EXPLICIT_OWNER_SELECTION
PROPOSED_AUTOFIX=NONE
PROPOSED_CODE_ACTION=NONE
DOCUMENT_SCOPE=EXACT_PARSE_SUCCESS_SNAPSHOT
SOURCE_FACT=IMMEDIATE_NONFINAL_SURFACE_LITERAL_OF_PROVEN_SEQUENCE
MESSAGE=Literal value is discarded before the sequence result.
```

**Truthful claim:** on normal Sequence completion, a direct non-final bare literal's computed value is **not** the result of that Sequence. This is a narrow source-level claim; it is not a guarantee about the programmer's intent, a claim that the whole preceding source span is unreachable, a statement about generic expression side effects, or a proof that its *removal* is semantics preserving in every larger source context.

Example positive cases (only where an actual `SurfaceSequence` owns both children):

```protos
process() {
    42
    doWork()
}

flag: () => {
    false
    compute()
}
```

Here the standalone literal in the body's non-final Sequence position has no result consumer. The **diagnostic range** must be the literal's exact `SurfaceLiteral.span()`, not the next expression or a guessed newline.

Mandatory negatives:

```protos
// Final expression is the meaningful closure result.
() => {
    compute()
    42
}

// A literal supplied as an argument is used, not a direct Sequence element.
() => {
    consume(42)
    done()
}

// A potentially effectful read/call cannot be classified as a no-op.
() => {
    object.member
    next()
}
```

Also exclude literals participating in binary expressions, slot creation/assignment, index operations, Map or Array entries, object bodies with different evaluation rules, matching arms without verified Sequence semantics, or any source whose parsing or snapshot validity is in doubt.

**Adversarial cases and proof limitations:**

1. A standalone early String could be an intentional pseudo-docstring/annotation even though Protos does not assign it JS directive semantics. A `Hint` is more honest than an `Error`; the owner may elect **opt-in** rather than default. Consider whether the initial rule should exclude leading String-only annotations, or reject the candidate for limited value. Do not invent a docstring standard implicitly.
2. A literal might intentionally be used in interactive/program-learning contexts. Top-level REPL presentation must be examined before applying the rule to REPL input. A first implementation should **restrict to braced Closure Sequences** unless source and test evidence proves module/REPL compatibility.
3. Literal source interpretation for numeric/Unicode values is governed by the parser; report only on a *successfully parsed* exact snapshot. No regex lexical scanning.
4. The detection is syntactic with no need to resolve a receiver, read live lexical slots, run user code, or load a Truffle Context. Do not generalize to arbitrary operator expressions or `Lookup`, where dispatch/lookup may be effectful.
5. The rule does not establish source deletion safety, because comments, attached trailing Closures, separator/newline grammar and other source-preservation constraints can make even an apparently isolated deletion unsafe. **No quick fix** in the first slice.
6. Source position mapping uses canonical `SourceSpan` and existing snapshot/LSP UTF-16 mapping. Superseded or closed buffers clear results; ignore out-of-date computations.
7. A warning may be visible for deliberately inert values; users can disagree about usefulness. Its correct classification is a **low-severity actionable hint**, not a semantic error or CI blocker.

## Go/no-go threshold

Before any implementation, the owner must approve or modify **this exact first rule**: category and diagnostic severity (`Hint` proposed), default versus explicit opt-in, treatment of leading String literals/pseudo-docstrings, and whether a minimal rule is useful enough to justify adding visible lint machinery.

If the owner does not consider the usefulness compelling, **choose no first rule now** and leave LM012 on hold rather than implementing a decorative lint framework. That is a successful evaluation result under D194, not permission to invent a stronger rule by analogy.

If approved with the conservative closure-only envelope, LM012-B is a *single coherent implementation slice* in `guillermomolina/protos`: extend the existing editor-neutral static core with one proof-based diagnostic (stable ID and span), project through current LSP diagnostics without a TypeScript analyzer, and add focused positive/negative/invalidation tests. Avoid `protos lint`, CI exits, new CLI/tool policy, code actions, formatter changes, runtime state, generic rule-engine plugin/config layers, extra D/PLAT issues, version bumps before human tests, and independent micro-slices. User runs tests/build/git for that repo.

No tests, builds or local `git diff --check` were executed in this investigation; the evidence is only source-level reasoning and future proof obligations.

```text
LM012_B0_INVESTIGATION=COMPLETE
FIRST_RULE=PROPOSED_NOT_APPROVED
RATIFIED_GENERAL_POLICY=D194_MODIFIED_B
SOURCE_LINT_IMPLEMENTATION=NOT_STARTED
PRODUCT_EDITS=NONE
TESTS_EXECUTED=NO
BUILDS_EXECUTED=NO
OWNER_FIRST_RULE_APPROVAL=REQUIRED
```
