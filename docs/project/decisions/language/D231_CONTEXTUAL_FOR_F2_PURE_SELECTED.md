# D231 — Contextual for as pure ordinary-iteration sugar (F2-PURE selected)

**Status: RATIFIED / OWNER-APPROVED (2026-11-01).** This is a durable non-normative design decision; the Protos language specification remains normative only under guillermomolina/protos/spec. Approval of the design is **not** an assertion that the proposed for syntax has been implemented or tested.

- Formal owner: [D231 #1013](https://github.com/guillermomolina/protos/issues/1013).
- Owner approval: **"Si apruebo D231-F2-PURE, Actualiza issues en guillermomolina/protos y agrega evidencias en guillermomolina/protos-project-docs."** This explicitly accepts the exact named F2-PURE contract following the [full GITHUB010/GITHUB021 packet](https://github.com/guillermomolina/protos/issues/1013#issuecomment-6556299117). Approval includes the separately disclosed contextual numeric-call collision and the ordinary-call escape routes.
- Evidence: [D231-F2-PURE approval chronology and GITHUB021 reconciliation](../../evidence/D231/D231_F2_PURE_APPROVAL_INVARIANTS_2026_11_01.md).
- Product revision inspected: [guillermomolina/protos@2108e5ab2eacd1ebd0f64041b49d7245a414481e](https://github.com/guillermomolina/protos/commit/2108e5ab2eacd1ebd0f64041b49d7245a414481e), implementation version 0.3.434, including D236-B1 cross-direction Core Range.
- Governing foundations: [D232](https://github.com/guillermomolina/protos/issues/1014) half-open directed ranges and signed steps, [D235](https://github.com/guillermomolina/protos/issues/1019) shared frozen Core/Prelude Range publication, [D236](https://github.com/guillermomolina/protos/issues/1028) direction-normalizing canonical construction, and [completed I140](https://github.com/guillermomolina/protos/issues/1029).

## Exact public syntactic forms and semantic equivalence

Protos adds **two complementary contextual for families** containing three exact forms:

~~~protos
for(item in expression) { body }          // (expression).each(item => { body })
for(i, start, stop) { body }               // Range(start, stop).each(i => { body })
for(i, start, stop, step) { body }         // Range(start, stop).eachStep(step, i => { body })
~~~

The right-hand sides specify observable equivalence to ordinary Protos message sends and an ordinary one-parameter lexical Closure. They are **not** a requirement to create a new runtime/Truffle numeric-loop node, Iterator/Iterable protocol, eagerly materialized Array, implicit import, helper function or extra return/exit mode. No separate public special built-in for is defined.

### Generic iteration

- Evaluate expression once in its surrounding activation. Invoke its normally selected, polymorphic each message with a freshly created ordinary one-parameter Closure. The method retains full authority for receiver eligibility, callback validation and invocation count, snapshotting, ordering, errors, suspension, cancellation, cleanup and normal result.
- Standard Array.each captures its established shallow indexed snapshot and returns the Array receiver; standard Range.each returns null; Map/IdentityMap.each uses its established shallow association snapshot and returns the map receiver; a custom each retains its own result and effects. No global assumption that for returns null.
- Standard Map.each passes two positional arguments (key,value) to the callback. The single-parameter Closure binds key and ignores the extra value according to the existing flexible Protos argument rule. Existing map.each((key,value)=>{...}) remains how code obtains both; no implicit pair, Map special casing, multi-binder or destructuring is added.

### Numeric iteration

- Resolve the ordinary lexical identifier Range at the calling site, exactly as in normal source Range(start,stop); the name is **not** privileged or identity-checked by for. Local shadowing, ordinary factory invocation, method overrides and errors remain observable. A custom Range binding may behave differently, exactly as if the developer wrote the equivalent ordinary expression. No hidden canonical Range guard.
- Three header positions construct Range(start,stop), then send each. Four header positions construct Range(start,stop), then send eachStep(step,bodyClosure). Exactly these ordinary messages are used, including when step equals 1 or -1.
- With the shipped canonical D236 constructor, start < stop selects Range (ascending), start > stop selects InvertedRange (descending), and equal endpoints retain Range (ascending empty). Original endpoints are unmodified: first inclusive, second exclusive. Bounds are genuine arbitrary-precision Integers subject to existing source-backed constructor validation.
- EachStep preserves its existing strict signed-step rule based on the **actual constructed prototype**: positive on Range, negative on InvertedRange; zero, non-Integer or wrong sign signals Error, even if empty. It does not flip the sign or select the constructor based on step. Thus for(i,0,10,2) visits 0,2,4,6,8; for(i,10,0,-2) visits 10,8,6,4,2; for(i,5,5,-1) signals Error despite an empty range.
- Normal numeric completion returns the existing Range each/eachStep result, null. Callback result is ignored as already specified by Range; no new implicit early-exit semantics.

### Ordering, scopes and control

- Strict equivalence to the right-hand side requires ordinary receiver-first evaluation. For numeric for: resolve Range, evaluate start and stop **once in that order**, complete construction (including errors), resolve the resulting each/eachStep send; only then evaluate step if present, followed by construction of the user body Closure argument. If construction errors, step is not evaluated. There is no eager preflight of all three numeric expressions before construction. For generic for, evaluate receiver before constructing the body Closure.
- The binder is a real declaration of exactly one formal parameter of an ordinary Closure created lexically in the surrounding activation. It is **not** an outer-name read, implicit mutable loop-counter slot, new semantic variable class, or extra helper-Closure activation. Each callback activation creates its ordinary parameter context; capture by reference, outer-name shadowing, this, super, methodHome, context and return-home identity follow standard CALLABLES and EXECUTION_AND_CONTROL contracts.
- Existing Smalltalk-like nonlocal caret return and contextual return syntax keep the same return home and InvalidReturn behavior, including escaped and cross-Task Closures. Error, ensure, ordinary cancellation/suspension and Future semantics propagate through existing messages. No break, continue, loop else, automatic await, synthetic Task or scheduler.
- No re-evaluation of header expressions during iteration: the Range instance is constructed first; a mutation of an outer bound inside the callback does not change its already captured endpoints. Other ordinary side effects remain subject to the selected each implementation.

## Contextual grammar, namespace and source compatibility

- The words for and in remain ordinary identifier tokens globally. The word for is interpreted contextually only when it is a **bare, ungrouped primary name** immediately used as a call receiver with one of the exact headers and with an attached braced body on the same continuing logical token sequence. The word in is a contextual header separator only in the generic form, never an added general infix operator.
- Generic header: exactly one simple ordinary identifier binder, followed by identifier in and one ordinary nonspread expression; numeric header: exactly one simple ordinary identifier binder, then two or three **comma-separated**, nonspread ordinary expressions. No other binders, default/rest parameters, trailing comma, destructuring, implicit second parameter or arbitrary parameterized trailing Closure is admitted.
- Body braces are parsed as an ordinary Closure **body**, never as object-construction syntax, with the one bound parameter. The construct remains an expression; valid postfix member/call/index suffixes are allowed. Nested for forms scope normally via their respective Closures. Ordinary newline rules apply inside the parentheses; an actual logical NEWLINE between ) and { prevents attached-body contextual recognition. Normal non-newline block comments and spaces do not.
- **Explicit accepted compatibility change:** previously valid bare for(i,a,b){body} and for(i,a,b,step){body} calls to a user binding named for, with a bare name in the first position, become contextual numeric for whenever the exact grammar matches. Recognition is structural, **not** runtime type-dependent or dependent on whether lexical for exists. Use (for)(i,a,b){body}, holder.for(i,a,b){body}, or explicitly supply the Closure argument to invoke an ordinary user-defined for function. The ordinary for(i,a,b) call without attached body remains ordinary. Parenthesized/member for receivers, spread or wrong-arity calls remain ordinary or retain previous syntax errors.
- for(i in expr) without the attached body is invalid grammar, since in is not a general infix operator. Malformed partial generic forms are syntax errors with original source locations, not fallback iterations.
- The original binder/in separator/comma/delimiter/body source spans and comment-layout facts remain visible to parser, formatting, static lexical analysis, LSP, diagnostics, tooling, source section tracing and round-trip rendering. A syntax-specific **Surface** AST is permissible; a new **canonical/runtime loop node** is unnecessary. Lower to existing ordinary canonical Call/Send/Closure forms preserving all above observables.

## Rejected alternatives and deliberately deferred scope

The full decision matrix and comparative scores are in the research [packet](https://github.com/guillermomolina/protos/issues/1013#issuecomment-6556299117): A no change; B generic only; C numeric only; D both pure ordinary message sugar (**selected**); E both with extra canonical Range guard/header staging (**rejected as unnecessary semantic exception**). A new native loop, generic Iterator, generalized parameterized trailing Closure, hidden canonical Range binding, loop-control exits, comprehensions, arbitrary step normalization, two-binder Map syntax and range operators are not part of D231.

The selected minimum covers the explicitly requested ergonomic surface without copying JavaScript property enumeration or adding an independent iteration substrate. Source-level numeric for conflicts with exact preexisting calls to ordinary for; that impact was fully disclosed and specifically accepted in the owner's F2-PURE approval.

## Handoff and authority

The design is settled. **Do not reopen choices in the implementation agent.** One independent, atomic Ixxx implementation in guillermomolina/protos must update the normative grammar/spec changelog, parser/Surface AST/source facts, canonical lowering, formatter/LSP/static analyses, conformance/Java tests and directly affected guide. The editor does not run commands, tests, Git or web; the human executor validates and commits/pushes the Protos product after PASS. Implementation must not create a Java loop kernel, altered Core Range API or new canonical-loop runtime protocol.

Until the product implementation is published and verified, these source forms are **approved but not yet executable**. Only the foundation D236/I140 is shipped in product 0.3.434.
