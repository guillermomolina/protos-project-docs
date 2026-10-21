# D211 — Contextual if/else bundle: falsification, scorecard and ratification evidence

**Date:** 2026-10-21  
**Issue:** [D211/#944](https://github.com/guillermomolina/protos/issues/944)  
**Decision:** [ratified D211 bundle](../../decisions/language/D211_CONTEXTUAL_IF_BUNDLE.md)  
**Scope correction:** [owner's indivisible A+B+C invariant](https://github.com/guillermomolina/protos/issues/944#issuecomment-6314689951).  
**Owner approval:** [explicit "apruebo todo A+B+C" checkpoint](https://github.com/guillermomolina/protos/issues/944#issuecomment-6314980398).  
**Detailed evidence:** [read-only falsification](https://github.com/guillermomolina/protos/issues/944#issuecomment-6314847624) and [scored comparison and source constraints](https://github.com/guillermomolina/protos/issues/944#issuecomment-6314883087).  
**Product baseline studied:** `7a122cdeeed91ca31690e2d39412dadad3518f72`; **not** an implementation SHA.

## Evidence provenance and important constraints

This source-based investigation was performed read-only using current public project repository files and the directly relevant prior ratifications in `protos-project-docs`. **No** build, run, benchmark, local working-tree read or integration test was executed by the agent. The owner's statement that the local tree passes `git diff --check` and local tests is recorded as a **pre-implementation** report, not proof the not-yet-edited D211 bundle works.

The owner expressly required **one indivisible candidate**: remove standard `Object.if` and `Object.else`; contextual `if(condition) { ... }` → ordinary `ifTrue`; contextual `if(condition) { ... } else { ... }` → ordinary `ifTrueIfFalse`. The only selectable comparator is the status quo. This narrower language-specific question reuses D199/D200/D210 earlier multi-language research; it does not need another speculative Truffle-runtime model.

## Observed factual falsifications

1. `protos/tests/conformance/control/control-spelling-is-ordinary.protos` declares lexical `if: (condition, branch) => {...}` and calls `if(123) {42}`. Approved syntax now interprets this **exact source form** contextually, breaking former lexical-call meaning. `(if)(123) {42}` retains ordinary invocation.
2. `protos/tests/conformance/boolean/receiver-preserving-if-else.protos` asserts selected-body result is ignored, the original Boolean returned, `Object.slotValue("if")`/`Object.slotValue("else")` work, and arbitrary values/Futures do not change result. These assertions **must change** when standard messages disappear. Ordinary custom member names remain.
3. D200 Boolean chains `c1.if(A).or(() => c2.if(B)).else(D)`, reversed/repeated `.else` and lazy `or` composition cannot be replaced mechanically by result-producing conditional expressions. They are a real compatibility cost, not runtime lowering bugs.
4. `spec/PROTOS_GRAMMAR.md` §§5, 19, 36 explicitly defines both ordinary call-plus-trailing Closure and newline separation. Exact narrow contextual recognizers work for the *new* B/C shape; allowing `else` after logical NEWLINE would add a grammar exception and collide with independent `else { ... }` object construction. Owner-selected D211 design uses same continuing logical line.
5. `src/main/java/com/guillermomolina/protos/parser/ProtosParser.java` already constructs `SurfaceContextualWhile` from `parseCallSuffix` and `semantic/Canonicalizer.java` lowers it to ordinary `CanonicalSend`. The same **surface→canonical** architecture suffices for D211 without new runtime op; unlike while, if's condition is **evaluated now**, not wrapped in a deferred Closure.
6. `ProtosStandardBooleanProtocol.java`, `ProtosCoreBootstrap.java`, `ProtosStructuredControlCalls.java` and related prepared/guarded send paths represent standard IF/ELSE selectors; remove all their specific cases/slots, retain IF_TRUE/IF_TRUE_IF_FALSE and the correct generic fallback.
7. `ProtosParserSourceFacts.java`, `ProtosSourceLayoutView.java`, `ProtosSourceLayoutToolBridge.java`, `ProtosStaticDefinitions.java`, `ProtosStaticReferences.java`, `ProtosStaticHover.java`, `ProtosStaticLint.java`, `ProtosDocumentSymbols.java` and `protos/tools/formatter/Structural.protos` contain dedicated handling of contextual while and need corresponding treatment of both if branches. Source-to-Closure association and LSP/DAP/formatter behavior cannot be assumed free.

## GITHUB010 scorecard, 1–5

Confidence H/M/L follows each numeric score; qualitative hard gates take precedence.

| Dimension | Keep D200/D051 | Adopt A+B+C | Reason |
|---|---|---|---|
| Correctness & invariants | 5 H | 3 M | New sugar coherent; explicit old-invariant reopening and source breaks |
| Protos philosophy | 4 H | 5 M | Remove redundant action-combinator protocols; reuses existing messages |
| Current-need proportionality | 3 M | 4 M | Simple user syntax; one-time nontrivial parsing/tooling work |
| Incremental growth | 4 M | 4 M | Both retain ordinary user-defined slots/Closures |
| Future-option resilience | 4 M | 4 M | No generic keyword or hidden custom dispatch boundary |
| Scalability | 5 H | 5 M | No new Task/Actor/Process semantic mechanism |
| Conceptual simplicity | 3 M | 4 M | Fewer published selectors; narrow contextual pairing rule |
| Portability | 5 H | 5 M | Ordinary message lowering, VM neutral |
| Runtime/resource cost | 4 L | 4 L | No compiled graph/performance verification; no new protocol machinery |
| Failure/operability | 5 H | 4 M | Existing error control paths preserved, source migration needed |
| Deferral/reversibility | 5 H | 2 H | Real lexical call/API return/reflection incompatibilities |
| Evidence maturity | 5 H | 3 M | D210 parser precedent, but D211 two-arm grammar/tools untested |

### Strongest objection and disposition

**Objection:** D200 was explicitly approved, implemented and published, and D051 deliberately protected `if(condition){...}` as an ordinary lexical call. A+B+C removes public methods and changes valid source meaning. **Disposition:** the owner expressly approves these known breaking deltas *together*, with the exact narrow contextual grammar. No artificial compatibility aliases, implicit old return behavior, or automatic preservation of D200 lazy `or` chains are authorized. If new collisions arise outside this documented boundary, return them for owner decision rather than widen sugar.

## Primary materials actually reviewed

- Project contracts: `AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, `AGENTS.work/COORDINATION.md`, `spec/AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `src/AGENTS.md`, `protos/AGENTS.md`.
- Normative: `spec/PROTOS_LANGUAGE_SPEC.md`, `spec/PROTOS_GRAMMAR.md`, `spec/semantics/VALUES_AND_COLLECTIONS.md`, `spec/semantics/CALLABLES.md`, `spec/semantics/EXECUTION_AND_CONTROL.md`, `spec/semantics/OBJECT_MODEL.md`.
- Product: `ProtosParser.java`, `SurfaceContextualWhile.java`, `SurfaceExpression.java`, `Canonicalizer.java`, `ProtosStandardBooleanProtocol.java`, `ProtosCoreBootstrap.java`, `ProtosStructuredControlCalls.java`; source facts, tooling references, formatter, parser/conformance tests listed above.
- Prior owner decisions and historical evidence: [D199](https://github.com/guillermomolina/protos/issues/879), [D200](https://github.com/guillermomolina/protos/issues/880), [D210](https://github.com/guillermomolina/protos/issues/941), and their evidence in this docs repository. Those precedents already compare Io, Smalltalk, Groovy, Ruby, Rust, Kotlin and Python, which do **not** determine Protos's independent execution model.

**GITHUB021 status:** D050 semantics preserved; D051 exact no-contextual-if rule and D200 standard message availability intentionally superseded by explicit owner A+B+C approval; D210 unchanged. **DESIGN_APPROVAL=EXPLICIT / BUNDLE=INDIVISIBLE / D211_PRODUCT_IMPLEMENTATION=NOT_YET_PUBLISHED.**
