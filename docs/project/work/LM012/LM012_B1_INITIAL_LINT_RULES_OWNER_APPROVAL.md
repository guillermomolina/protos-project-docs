# LM012-B1 — Owner-approved initial correctness lint rules

**Status: OWNER-APPROVED — IMPLEMENTATION READY; not implemented or tested.**

**Decision date:** 2026-10-08. **Owner approval provenance:** in the active conversation, immediately after the explicit proposed package *“reglas 1 y 2, con severidad Warning, activadas por defecto y sin autofix, como primer slice de implementación”*, the project owner replied **“aprobado”**. This approval selects the **exact two narrow rules** below; it does not select more rules, new language semantics, CLI/CI outcomes or automatic edits.

- Live work: [LM012/#671](https://github.com/guillermomolina/protos/issues/671).
- Governing public tooling policy: [D194/#842](https://github.com/guillermomolina/protos/issues/842), [ratified modified-B record](../../decisions/tooling/D194_CANONICAL_LINT_DIAGNOSTICS_RULE_AND_FIX_POLICY.md); exact first publication revision `2b12b4a59cbc6bce49356911ff1bb9e46f176bd9`.
- Comparative evidence: [LM012-B0 higher-value search](../../evidence/LM012/LM012_B0_HIGH_VALUE_RULE_SEARCH.md), first published in docs at `650639857656dcb0fbb8bf2a54929f6c8f107681`, index revision `ad9fea5c5ba36517405fb9c70a3ddfa4a308a9b3`.
- Earlier baseline: `guillermomolina/protos@3bb1278d91ee5cea98031462be2a5c4dd3c89019`. **Implementation starts from the human's actual current local HEAD**, so re-audit relevant changed source if baseline drifted.
- **Nature:** durable non-normative LM012 owner selection and implementation handoff. This document does not define Protos language semantics and does not supersede `spec/`.

## Exactly approved initial rule 1

```text
RULE_ID=protos/unreachable-after-nonlocal-return
CATEGORY=CORRECTNESS
SEVERITY=WARNING
ENABLED_BY_DEFAULT=YES
AUTOFIX=NO
CODE_ACTION=NO
```

When a successfully parsed *braced Closure body* contains a real `SurfaceSequence` and a **direct child** of `SurfaceSequence.expressions()` is `SurfaceNonLocalReturn` (`^value`), every subsequent direct child of **that exact same Sequence** is unreachable. The return transfers out; an invalid return home yields non-resumable `InvalidReturn` instead of normal fallthrough.

- **Report:** preferably one deterministic diagnostic for the unreachable suffix, anchored to the first subsequent expression's exact span (or exact well-defined contiguous suffix span if canonical span utilities support it); do not emit arbitrary repetitive warnings for each token.
- **Positive:** `() => { prepare(); ^42; cleanup(); audit() }` — `cleanup()` and `audit()` cannot be reached.
- **Negative:** `() => { condition.ifTrue(() => { ^42 }); cleanup() }` — nested `^` in a callback does not guarantee calling it. A nested Closure merely created, a `^` in an argument that might not execute, or a final `^` without successors do not trigger.
- **Scope limits:** do not infer whole-program control flow, conditions, match coverage, method dispatch, or unreachable subexpressions. The only initial source owner is an exact *braced Closure* `SurfaceSequence`, not an object-body construction, Map entry list or lexical text scan.
- **Severity wording:** report unreachable expressions in the local sequence, not that the entire program must execute or always terminate. No automatic removal/rewrite.

## Exactly approved initial rule 2

```text
RULE_ID=protos/always-different-fresh-object
CATEGORY=CORRECTNESS
SEVERITY=WARNING
ENABLED_BY_DEFAULT=YES
AUTOFIX=NO
CODE_ACTION=NO
```

When successfully parsed `SurfaceBinary` uses the **primitive non-overridable** operator `===` or `!==`, and the **right operand** is an exact source `SurfaceObject` newly constructed in the expression (allow only demonstrably transparent `SurfaceGroup` parentheses), on normal completion:

- `left === newRightObject` is **always false**;
- `left !== newRightObject` is **always true**.

The left expression evaluates before the right, and a normally completed right `SurfaceObject` construction returns a fresh identity-bearing object distinct from the already-evaluated left value. This says nothing about whether evaluating either operand has effects, suspends, fails, or transfers control. Its proof uses ordinary Protos object freshness and primitive semantic identity, **not** operator `==` and not inferred object typing.

- **Positive:** `candidate === {}`, `candidate !== { marker: true }`, and `candidate === (parent { ... })` only when exact AST shape and normal-result freshness are established.
- **Negative:** `candidate == {}`, arbitrary right-hand `makeObject()`, ordinary `[]` or `%{}` constructions absent an independently proven result model, and `{ ... } === arbitraryRight` must not trigger under the approved directional scope.
- **Report:** exact `SurfaceBinary.span()` (or more precise existing approved span projection), stable code and a message indicating the normal-completion constant result; never say that neither operand is evaluated.
- **No quick fix:** do not offer `==` replacement because it may dispatch user code and change semantics; do not replace the expression with `true` or `false` because that could suppress operand effects or errors.

## Mandatory shared behavior

1. These **two and only these two** correctness diagnostics are default-enabled on successfully parsed, current, in-domain source snapshots. No opt-in switch, config/plugin institution or project-wide analyzer is required.
2. Reuse existing editor-neutral parser/snapshot/semantic AST authorities (`ProtosStaticAnalysisCore`, `ProtosStaticParseResult`, `SurfaceSequence`, `SurfaceNonLocalReturn`, `SurfaceBinary`, `SurfaceObject`, `SourceSpan`). Do not execute guest code, run a Truffle Context or create new always-on runtime state. D194/PLAT024 pay-as-you-grow applies.
3. Existing LM009-G1 parser errors retain ownership; do not duplicate parse diagnostics as lint. Publish a deterministic combined result through the existing LSP server diagnostics path, with correct immutable snapshot/version and UTF-16 source conversion; clear on valid reparse with no findings and on close. The standalone VS Code extension stays thin; no TypeScript parser/linter.
4. Handle nested Closure bodies without confusing the parent Sequence with a Closure that has not been called. Inspect actual AST ownership and child traversal before emitting any result; fail closed when the exact grammar/AST proof does not hold.
5. No new `protos lint`/`protos check` command, CI exit semantics, profile/configuration format, diagnostics suppression syntax, auto-fix, CodeAction advertisement, formatting policy, generic framework, matching coverage or new PLAT architecture is admitted.
6. Produce focused unit and LSP integration tests for positive, negative, nested, stale-source, close, UTF-16/CRLF and parser-error cases; the human executes validation/build/tests and product Git operations. Do not change `pom.xml` or `CHANGELOG.md` until the test gate has passed and immediately before user-executed commit/push; do not rerun tests after such metadata edits.

## Exact approval scope and invariant/delta review

This approved selection **satisfies**, rather than reopens, the ratified D194 modified-B criteria: useful distinct correctness findings; bounded source proof; stable rule IDs; explicit severity; individual default-enable approval; explicit zero-edit policy; snapshot/source identity; no ordinary runtime lint cost; no speculative negative dynamic-lookup or matching assertions.

Earlier LM012-B0 candidates for a discarded literal and a constant cross-family literal identity warning remain **UNSELECTED**, not silently included. A broader "general unreachable" data-flow checker, fresh-object-on-left detection, user-`==` inference, warning-to-CLI error gate or automatic fix are **NOT APPROVED**. Any future substantive expansion must follow its own decision authority.

```text
LM012_B1_OWNER_APPROVAL=EXPLICIT_2026-10-08
APPROVED_RULE_COUNT=2
RULE_1=protos/unreachable-after-nonlocal-return
RULE_1_SEVERITY=WARNING
RULE_1_DEFAULT=ON
RULE_2=protos/always-different-fresh-object
RULE_2_SEVERITY=WARNING
RULE_2_DEFAULT=ON
AUTOFIX=NO
CLI_CI_POLICY=UNSELECTED
D194_GENERAL_POLICY=PRESERVED
SPEC_CHANGE=NO
IMPLEMENTATION_STARTED=NO
TESTS_RUN=NO
BUILDS_RUN=NO
```
