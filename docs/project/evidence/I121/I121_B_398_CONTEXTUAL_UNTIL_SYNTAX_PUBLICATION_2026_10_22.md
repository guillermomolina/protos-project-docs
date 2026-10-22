# I121-B — Contextual until suffix: verified product publication

**Formal owner:** [I121 #948](https://github.com/guillermomolina/protos/issues/948), second of two implementation slices.
**Governing decision:** [D212 #947](https://github.com/guillermomolina/protos/issues/947); [durable decision](../../decisions/language/D212_POST_TEST_UNTIL_AND_WHILE_FALSE.md).
**Exact product commit:** [`7af62b8394b04790b678029501031a12d9d13c4d`](https://github.com/guillermomolina/protos/commit/7af62b8394b04790b678029501031a12d9d13c4d), subject `I121-B: add contextual suffix until syntax lowering to untilTrue (0.3.398)`.
**Publication date:** 2026-10-22 (per product specification changelog).
**Maven version:** `0.3.398-SNAPSHOT`, verified in the committed `pom.xml`.
**Normative revision:** `0.1.468`, verified in the committed `spec/PROTOS_SPEC_CHANGELOG.md`.
**Earlier slice:** [I121-A evidence](I121_A_397_STANDARD_CLOSURE_LOOPS_PUBLICATION_2026_10_21.md), exact [product commit `dafee2cb0c145fb6ed8a2a33ba8854526a448631`](https://github.com/guillermomolina/protos/commit/dafee2cb0c145fb6ed8a2a33ba8854526a448631), Maven `0.3.397-SNAPSHOT`, specification `0.1.467`.
**Status:** A and B product publications verified. I121 formal closure is a separate live GitHub transaction; this evidence file alone does not close the Issue.

## Observed delta at the exact revision

The verified I121-B commit contains **29 changed paths** covering source parser, source AST, canonicalization, static tooling, formatter, tests, normative grammar and publication metadata.

- **Contextual recognition:** `TokenCursor.bracedBodyFollowedByContextualUntil()` uses non-consuming, balanced-delimiter lookahead for parentless `{ ... }` immediately followed (without logical `NEWLINE`) by the ungrouped bare identifier `until` and exactly one non-spread parenthesized argument. Otherwise the ordinary object parser remains in control.
- **AST and lowering:** `ProtosParser.parseContextualUntil()` parses a parameterless braced Closure body; `SurfaceContextualUntil` preserves the body, condition, identifier span and full source span. `Canonicalizer.lowerContextualUntil()` emits a normal `CanonicalSend` to the already standard `untilTrue`, with the braced body Closure as receiver and a separate, zero-parameter condition Closure as sole argument. No canonical Until opcode, new runtime selector or interpreter is introduced.
- **Source facts and static tooling:** `ContextualUntilOrigin`, source-layout paths `body`/`condition`, and the three distinct comment boundaries `CONTEXTUAL_UNTIL_BEFORE_KEYWORD`, `CONTEXTUAL_UNTIL_BEFORE_LPAREN` and `CONTEXTUAL_UNTIL_AFTER_LPAREN`. `ProtosSourceLayoutToolBridge` exports `CONTEXTUAL_UNTIL`; `protos/tools/formatter/Structural.protos` keeps the suffix on the body-closing line while preserving contextual comments. Hover, lint, definitions, references and document symbols recognize separate deferred Closure scopes.
- **Tests:** new suite-native `protos/tests/conformance/control/until-contextual-syntax.protos` plus manifest entry; Java `ProtosI121ContextualUntilCanonicalizationTest.java`; additional parser/source-facts/layout/formatter and static-tool cases for scoped names, correct comment counts and placement, stable reformatting, nested delimiters and ordinary object-fallback cases.
- **Normative and release bookkeeping:** `spec/PROTOS_GRAMMAR.md` adds D212 `contextual-until-expression`; `spec/PROTOS_SPEC_CHANGELOG.md` records global revision `0.1.468`; root `CHANGELOG.md` and `pom.xml` record `0.3.398-SNAPSHOT`.

## Observable contract and selected exception

```protos
{
    work()
} until(done())
```

desugars to:

```protos
(() => { work() }).untilTrue(() => done())
```

The `untilTrue` protocol's preexisting I121-A semantics provide post-test order, exact Boolean condition checks, exact last-body result, normal/abnormal transfer and ordinary selected method behavior. The braced body and condition are independent zero-parameter lexical Closures, sharing only legitimately captured outer contexts. The body executes at least once on normal loop completion. Body-local slots are not in the condition Closure's activation scope.

The deliberate, owner-approved D212 **source-grammar exception** reinterprets parentless bare `{ ... }` only for that immediate suffix shape; all other braces remain their normal object construction or Closure roles. The lexical name `until` stays unreserved. No `repeat`, contextual `untilFalse`, new `Object.until`, truthiness, implicit Future awaiting, shared lexical block or additional scheduling boundary.

## Validation evidence and limitations

The maintainer reported in this conversation:

> I121-B: add contextual suffix until syntax lowering to untilTrue (0.3.398), pushed.
>
> el git diff check esta limpio.
>
> Todos los tests han pasado en local.

The verified root `CHANGELOG.md` at I121-B's product commit also explicitly records **`make check PASS`** and **`make test PASS`**. These are **maintainer-reported validation outcomes**, not checks independently run by the documenting assistant. Exact per-test counts, an independent Native-image run, Tier-1/2 graph numbers and measured performance are not supplied and are not claimed. No standalone tagged release is inferred from the Snapshot version.

## Completion boundary

I121-A introduced the standard `whileFalse`, `untilTrue` and `untilFalse` protocols (with unchanged `whileTrue`); I121-B provided the approved contextual suffix syntax and tooling. Together they cover the D212 design. Final coordination must link **both exact product revisions** and the **exact project documentation commit**, and verify live Issue completion status. There is no need for a third implementation issue or new owner decision.
