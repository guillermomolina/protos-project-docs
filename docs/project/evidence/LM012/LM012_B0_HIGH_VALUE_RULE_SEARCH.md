# LM012-B0 — Higher-value lint rule search and admission ranking

**TYPE=INVESTIGATION — COMPLETE. No new rule is selected or implemented.**

**Date:** 2026-10-08. Owner: [LM012/#671](https://github.com/guillermomolina/protos/issues/671). Governing policy: [ratified D194 modified B](../../decisions/tooling/D194_CANONICAL_LINT_DIAGNOSTICS_RULE_AND_FIX_POLICY.md), originally published at exact docs revision `2b12b4a59cbc6bce49356911ff1bb9e46f176bd9`.

**Reference:** inspected published Protos sources at `guillermomolina/protos@3bb1278d91ee5cea98031462be2a5c4dd3c89019`. Before implementation, re-evaluate affected paths at the implementing checkout's actual HEAD, preserving concurrent changes. The previous [LM012-B0 candidate research](LM012_B0_FIRST_RULE_ADMISSION_RESEARCH.md) proposed a discarded non-final literal. This follow-up was explicitly requested by the owner to find a **more useful rule**, not to approve another rule or to execute an implementation.

**Evidence level:** normative and implementation-source inspection plus official prior-art rule documentation; no source code modifications, builds, tests, runtime programs, Git validation, or benchmarks. Rule utility rankings are qualitative, not measured bug incidence.

## Governing proof and cost boundaries

- `spec/PROTOS_GRAMMAR.md`, expression sequence, object expressions, closure bodies, operator precedence and §22 Equality Lowering: real `Sequence` AST, new object construction, `===` primitive non-overridable identity and `!==` its negation. The grammar itself explicitly exemplifies two distinct object literals comparing false under `===`.
- `spec/semantics/EXECUTION_AND_CONTROL.md`, §8.1 / §8.2: strict left-to-right evaluation including binary operands, no normal continuation from a transferred Sequence, precise normal Sequence result.
- `spec/semantics/CALLABLES.md`, §13–14: direct `^value` performs a non-local return to its function/method home; invalid or cross-Task homes signal `InvalidReturn`, **never resume after the ^**.
- `spec/semantics/ERRORS.md`: signaling is non-resumable and abandons the signaled continuation; `ensure` cleanup cannot reactivate a superseded control transfer.
- `spec/semantics/OBJECT_MODEL.md`, §§2, 20: `{ ... }` and `parent { ... }` create a **new ordinary identity-bearing object** with a fixed parent; it is not the parent's semantic identity.
- `spec/semantics/VALUES_AND_COLLECTIONS.md`, §21 and numeric equality: `===` compares semantic identity without overridable ordinary `==`; numeric `1 == 1.0` is true but `1 === 1.0` is false because numeric **families** contribute to identity; `0.0 === -0.0` is false by signed-zero identity. `Map` construction, `Array` construction and ordinary equality have different effect/factory authorities; do not generalize these proofs by analogy.
- Existing AST fields: `parser/ast/SurfaceSequence.expressions/span`; `SurfaceNonLocalReturn.expression/span`; `SurfaceBinary.left/operator/right/span`; `SurfaceObject`; `SurfaceGroup`; `SurfaceLiteral.Kind`. They are exposed from the real editor-neutral parser under `ProtosStaticAnalysisCore` and immutable snapshots; there is no need for runtime introspection, a type inferencer or a new always-on index.
- D194 has **not** selected particular rule IDs, severities, default activation, edits or CLI/CI outcome. LM009-G owns parser errors, D183/LM011 owns formatting, D096 owns matching and D110/D124 prohibit guessed negative name proofs.

## Candidate ranking

Scores: 1 (low) to 5 (high); `soundness` evaluates only the *exact admissible scope*, not tempting expansions. Confidence H/M/L. Implementation burden is qualitative, not measured.

| Priority | Rule proposal | User-value | Soundness | Scope/noise | Cost and constraint |
| --- | --- | --- | --- | --- | --- |
| **1** | **Unreachable Sequence suffix after a direct unconditional `^`** | **5/H** — detects code that can never execute, often a real control-flow mistake | **5/H** | **5/H** — strict same-Sequence direct-child proof; no guessed branch CFG | Very low; AST visit and diagnostic, no edit |
| **2** | **Fresh-object RHS of primitive `===` or `!==`** | **4/M** — detects mistaken identity comparisons against an object constructed only on the RHS | **5/H** | **4/H** — exact `SurfaceObject` as RHS; conditional on normal completion | Very low; direct AST shape, no semantic name proof |
| **3** | **Provably constant cross-family literal identity comparison** | **3/M** — teaches Protos-specific numeric-family identity, catches impossible comparisons | **5/H** for explicit literal pairs | **4/M** — only exactly classified literal values/families, not variables | Low; exact numeric parsing can complicate scope |
| **4** | **Discarded non-final bare literal** (earlier candidate) | **2/M** — plausible typo, but also deliberate inert marker | **5/H** as result-discard claim | **3/M** — false-positive-of-intent risk, especially docstring-like strings | Low; already researched |
| Rejected | `undefined`/`unused` names and broad dead code | Potentially high | **1/H** without proof | Poor — open lookup, mutable contexts, effects | Unacceptable guessed claims |
| Rejected | Duplicate literal `Map` key is guaranteed Error | Moderate | **1/H** as a universal error | Poor — normal `hash` and `==` may be overridden | Runtime check; do not assert static identity |
| Rejected | Blanket constant `&&`/`||` branches | Moderate | **1/H** unless canonical Boolean receiver is proved | Poor — ordinary overridable `and`/`or` | Do not import JavaScript truthiness |
| Rejected | Whitespace/formatting-only warnings | Low distinct value | Sometimes sound | Overlaps D183 formatter | Not a first lint rule |

### A. Recommended first: unreachable after direct `^`

**Proposed identity:** `protos/unreachable-after-nonlocal-return`. **Suggested initial level:** `Warning`, enabled by default **only if owner selects this exact rule**. **Autofix:** none.

A direct `SurfaceNonLocalReturn` in the ordered `expressions()` of a `SurfaceSequence` cannot complete normally to its successor. Evaluating the `^` operand may have effects, fail, suspend or diverge; on normal operand completion the return transfers out, while `InvalidReturn`/Error/cancellation also fail to resume the abandoned Sequence. Therefore **each later direct Sequence child is unreachable**, independently of dynamic delegation or user-defined messages.

Positive, within one braced Closure Sequence:

```protos
run: () => {
    prepare()
    ^42
    cleanup()   // unreachable
    audit()     // unreachable
}
```

Negative/boundary cases:

```protos
// Conditional invocation does not prove the outer Sequence is exited:
run: () => {
    condition.ifTrue(() => { ^42 })
    cleanup()       // must NOT warn: condition may be false, or behavior overridden
}

// A nested Closure is only created here; its ^ does not execute now:
run: () => {
    f: () => { ^42 }
    cleanup()       // must NOT warn
}

// A final ^ has no unreachable suffix:
run: () => {
    prepare()
    ^42
}
```

The smallest first domain is **successfully parsed braced Closure bodies**. Do not infer that a nested `^` in an argument, `ifTrue` callback, matcher, outer object body or arbitrary call executes unconditionally; no whole-program CFG. Do not claim that the body containing the code is itself always entered. Emit **one** diagnostic for the first unreachable following expression or the exact complete suffix range (choose one deterministic representation after inspecting available `SourceSpan` machinery), not one warning per token. This is *unreachable* code, not a parser error. An invalid/stale parse must fail closed. Do not automatically delete anything: source separators, trailing closures and comments can make apparent text deletion change the grammar.

### B. Recommended second: newly created object on the RIGHT of `===`

**Proposed identity:** `protos/always-different-fresh-object`. **Suggested initial level:** `Warning` or `Hint` depending on owner choice. **Autofix:** none.

```protos
different: (candidate) => candidate === {}
alsoDifferent: (candidate) => candidate !== { marker: true }
```

When evaluated normally, the right operand constructs a **fresh ordinary object** *after* the left operand was evaluated; it therefore cannot be the already-computed left object, even when the parent delegates to the same prototype and even if the two contain equal-looking slots. Primitive `===` is then always `false`, and `!==` always `true`. **Source proof:** direct `SurfaceBinary` operator `===`/`!==` with an unambiguously parsed `SurfaceObject` RHS (optionally transparent exact `SurfaceGroup` parenthesis wrapper). Evaluate and report only on parse-success snapshots. Describe the outcome **conditional on normal completion**, not as a guarantee of no errors, effects, or suspension.

**Crucial unsound generalization to reject:** do *not* blindly mark a new object on the **left** as always different from an arbitrary RHS. The RHS evaluates later and may read a reference to that same object if it escaped during LHS evaluation (including constructions/writes wrapped inside an expression). The initial policy's directionality is essential; no generic "new object anywhere" rule. Likewise do not include `==`/`!=` because user equality can be customized. Do not extend to `[]`, `%{}` or user factories without a separate exact normal-result/freshness proof and admission test.

Unlike raw literal-vs-literal constant comparisons, this rule can catch a bug in a real variable-versus-literal comparison without needing runtime type inference.

### C. Third: strict identity between provably distinct immutable value families

**Proposed identity:** `protos/constant-identity-comparison`, **not admitted yet**. Examples:

```protos
1 === 1.0      // always false: Integer vs Float semantic numeric family
42 === "42"    // always false: Number vs String
false !== null // always true
```

The Protos-specific `1 === 1.0` consequence is particularly easy for programmers accustomed to JavaScript to miss. **Do not warn** for `1 == 1.0` (true numeric equality), unknown bindings such as `a === 1.0`, or newly created objects as if they were numeric values. Numeric spelling/family must be checked against the exact grammar. Unary minus and signed zero require deliberate handling; do not infer all Float comparisons from token text or import JavaScript numeric rules. Prefer source-local exact families before any constant-folding engine.

### D. Earlier literal-discard candidate

Keep the existing `protos/discarded-sequence-literal` `Hint` in the candidate backlog; its source fact is sound, but often intentional and normally lower value than A or B. Do not silently treat the owner's conversational "me parece bien" as a finalized set of ID, default profile, severity, exception policy and specific implementation permission while the owner explicitly requested this comparative search.

## Prior art and portability cautions

- [ESLint `no-unreachable`](https://archive.eslint.org/docs/rules/no-unreachable) / [gopls unreachable analyzer](https://go.dev/gopls/analyzers): production static warnings after guaranteed control transfer; Go's full type/CFG machinery is **not** needed for Protos's exact direct `^` case.
- [ESLint `no-constant-binary-expression`](https://eslint.org/docs/latest/rules/no-constant-binary-expression) and [real-world bugs found](https://eslint.org/blog/2022/07/interesting-bugs-caught-by-no-constant-binary-expression/): comparisons impossible on semantic grounds often uncover real mistakes; only Protos's non-overridable `===` and object freshness transfer, not JavaScript coercion.
- [Ruff `comparison-of-constant`](https://docs.astral.sh/ruff/rules/comparison-of-constant/): narrow constant-comparison lint; Python literal/value semantics are not Protos's numeric family identity.
- [Ruff `true-false-comparison`](https://docs.astral.sh/ruff/rules/true-false-comparison/): unsafe auto-fix due overridable equality; reinforces D194's no-unproven-fixes boundary.

These are implementation-independent precedents; the normative Protos specifications above exclusively justify soundness.

## Recommendation and smallest coherent next slice

**Recommendation:** prefer **A first**, then **B** if exact RHS freshness proof is confirmed against implementation HEAD. Together they are a small useful initial **correctness-only** set with common AST traversal and diagnostic projection. The first slice could implement both in one cohesive patch **after explicit individual owner approval**, with a stable rule code and level for each, focused proof/adversarial tests, exact invalidation/UTF-16 LSP diagnostics, and no CLI, configuration framework, code actions, runtime costs or per-rule micro-issues. C and D remain unimplemented pending value evidence.

**No owner approval is presumed by this research.** The owner's explicit choice is needed for A and B: adopt/reject each, their exact identifier, `Warning` vs `Hint`, default-enabled vs opt-in, the stated narrow coverage and no edits. In particular, the earlier discarded-literal candidate remains optional; the search recommends postponing it to avoid adding low-value noise.

**Validation left to a future human-executed implementation:** inspect actual product HEAD and source spans, write focused parser/static-unit and real LSP tests for each rule and negative case; run affected tests and the validation impact policy; only then human-managed version/changelog/commit/push. This investigation ran no commands, tests or builds.

```text
LM012_B0_HIGH_VALUE_SEARCH=COMPLETE
GENERAL_POLICY=D194_MODIFIED_B_RATIFIED
RECOMMENDED_RULE_1=UNREACHABLE_AFTER_DIRECT_NONLOCAL_RETURN
RECOMMENDED_RULE_2=FRESH_OBJECT_ON_RHS_STRICT_IDENTITY
RULE_3=CONSTANT_CROSS_FAMILY_IDENTITY_DEFERRED
OLD_DISCARDED_LITERAL=DEFERRED_LOWER_VALUE
FIRST_RULE_OWNER_APPROVAL=REQUIRED
LM012_IMPLEMENTATION=NOT_AUTHORIZED_BY_THIS_RESEARCH
PRODUCT_EDITS=NONE
TESTS=NOT_RUN
```
