# I121-A — Standard Closure loop polarities: verified product publication

**Work item:** [I121 #948](https://github.com/guillermomolina/protos/issues/948), first of two planned implementation slices.  
**Governing design:** [D212 #947](https://github.com/guillermomolina/protos/issues/947), [ratified decision](../../decisions/language/D212_POST_TEST_UNTIL_AND_WHILE_FALSE.md).  
**Publication date:** 2026-10-21.  
**Product revision (verified GitHub commit):** [`dafee2cb0c145fb6ed8a2a33ba8854526a448631`](https://github.com/guillermomolina/protos/commit/dafee2cb0c145fb6ed8a2a33ba8854526a448631).  
**Verified commit subject:** `I121-A: add standard Closure loops whileFalse, untilTrue and untilFalse (0.3.397)`.  
**Implementation version:** `0.3.397-SNAPSHOT`, verified in `pom.xml` and the root `CHANGELOG.md` at publication.  
**Global normative specification revision:** `0.1.467`, verified in `spec/PROTOS_SPEC_CHANGELOG.md`.  
**Status:** A PUBLISHED. I121 remains OPEN pending B contextual suffix syntax. No releases/tags or successful downstream B implementation are inferred.

## Delivered and observed file delta

The GitHub commit contains **23 changed paths**, covering the Java standard loop implementation and Bytecode DSL prepared/structured dispatch, its canonical selection and method-home guards, the Protos and JUnit test surfaces, normative specifications, Maven and changelog publication metadata, and generated-bytecode BCI baseline.

- **Standard Object protocols:** Add ordinary native `Object.whileFalse`, `Object.untilTrue`, and `Object.untilFalse` Closure-valued slots alongside existing `Object.whileTrue`. Use a single `StructuredLoopKind` describing phase and canonical terminal Boolean; preserve standard `Object.while` absence and ordinary dispatch/shadowing.
- **Execution:** `condition.whileFalse(body)` is pre-test and stops on canonical true; `body.untilTrue(condition)` and `body.untilFalse(condition)` are post-test and activate body before first condition, stopping on canonical true/false respectively. Exactly one semantic Closure argument; both operands validated before callback execution; strict canonical Boolean condition, fresh Error otherwise; exact last normally completed body result, canonical null only for zero-body pre-test; no implicit Future await or cancellation boundary.
- **Truffle/Bytecode DSL:** `PreparedWhileCall` carries immutable kind and maps receiver/argument to roles. Guarded standard selection keeps ordinary-method-home/stability requirements; direct generic and structured dispatch support all modes. Post-test path uses `StructuredWhilePostTest` and body-then-condition in the loop's condition region. Inline-literal admission extends to `whileFalse` but excludes post-test. `CanonicalToBytecodeLowerer` comments updated to describe both pre-test kinds. The BCI baseline change is included in the same product commit.
- **Conformance:** add `loop-standard-polarities.protos` and `ProtosD212StandardLoopPolaritiesTest.java`, register manifest entry, and update D210-era assertions prohibiting `whileFalse`. Covers strict validation, phase order, value identity, Future and cancellation/return-home, standard dispatch and local overrides.
- **Normative:** `spec/semantics/EXECUTION_AND_CONTROL.md`, `spec/semantics/CALLABLES.md`, and `spec/PROTOS_LANGUAGE_SPEC.md` specify the new protocols and preserve preexisting `whileTrue`. The global specification changelog explicitly states **no grammar change** in A.

## Validation and provenance boundary

The maintainer explicitly reported, after publishing A to GitHub:

> `I121-A: add standard Closure loops whileFalse, untilTrue and untilFalse (0.3.397), pushed.`  
> `el git diff check esta limpio.`  
> `Todos los tests han pasado en local.`

The root `CHANGELOG.md` also records `make test PASS`. These are **maintainer-reported validation results**, not independently executed by the coordinator in this session. No standalone Native-image execution, release package publication, detailed focal per-test totals, JIT throughput comparison or BGV evidence is inferred. The exact product commit and its 23-file delta were independently fetched from GitHub.

## Acceptance and remaining work

**I121-A:** published and reported locally green; its normal execution/protocol tests and publication metadata are present in the verified product commit.  
**I121-B:** NOT YET IMPLEMENTED. Add **only** the narrow contextual suffix source form `{ expression-sequence } until(oneConditionExpression)`, equivalent to `(() => { expression-sequence }).untilTrue(() => oneConditionExpression)`. Preserve `Object` construction outside the exact contextual form, ordinary dynamic dispatch of `untilTrue`, two independent lexical Closures and existing `while(condition) { body }` and `if/else` contextual forms. Own parser/AST/canonicalizer, grammar, static tooling, source layout/formatter, and conformance. Do not reimplement A or claim new semantics. I121 remains OPEN through B's validation, human product publication and later revision-specific evidence.

**Cross-reference policy:** all product coordinates use `guillermomolina/protos`; durable records use `guillermomolina/protos-project-docs`. Historical D210 decision remains historical and was explicitly superseded only on the `whileFalse` absence rule by D212.
