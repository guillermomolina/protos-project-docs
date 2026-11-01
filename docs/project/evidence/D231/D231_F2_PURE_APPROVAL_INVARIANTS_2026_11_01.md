# D231-F2-PURE — Owner approval, GITHUB010 research and GITHUB021 invariant record

**Date:** 2026-11-01. **Formal design:** [D231 #1013](https://github.com/guillermomolina/protos/issues/1013). **Selected:** F2-PURE, BOTH generic and numeric contextual forms, all expressed via ordinary each/eachStep. **Product implementation of for:** NOT YET DONE by this decision publication. **Source baseline:** [protos@2108e5ab2eacd1ebd0f64041b49d7245a414481e](https://github.com/guillermomolina/protos/commit/2108e5ab2eacd1ebd0f64041b49d7245a414481e), with shipped I140 0.3.434. Language semantics will be normative only upon a future implemented spec revision under protos/spec.

## Approval chronology and scope

1. Initial [INC003 #1007](https://github.com/guillermomolina/protos/issues/1007) accepted DEFER for exploratory counted-for sugar at that time; this did **not** forbid later newly authorized D231 research.
2. [D231 #1013](https://github.com/guillermomolina/protos/issues/1013) was allocated to research contextual bounded iteration; the owner subsequently requested considering **both** for(item in expression) and numeric for(i,start,stop[,step]) as complementary forms, not choosing one.
3. D232 ratified first-class Range/InvertedRange with signed step; D235 promoted them to frozen shared Core/Prelude. Owner required the dynamic-step selection and return semantics resolved **before** implementation, rejecting implementation-agent choice.
4. [D236 #1028](https://github.com/guillermomolina/protos/issues/1028) ratified canonical source-backed opposite-direction range construction; [I140 #1029](https://github.com/guillermomolina/protos/issues/1029) published that contract at product SHA 2108e5ab2eacd1ebd0f64041b49d7245a414481e, with separate [human-reported PASS evidence](https://github.com/guillermomolina/protos-project-docs/blob/451574e4cc9f4771d6b5cee027d3fb5bc3527ba2/docs/project/evidence/I140/I140_0_3_434_BIDIRECTIONAL_RANGE_CLOSURE_2026_11_01.md). The design of numeric for no longer needs any custom sign selection.
5. The owner clarified the goal: **"no estamos implementando un nuevo bucle, estamos agregando una syntactic sugar que ahorra escribir un monton de cosas"**, and **"todo el mundo sable lo que for(i in expression) significa"**. The design must be familiar without replicating JavaScript's distinct for-in semantics.
6. [D231 final research packet](https://github.com/guillermomolina/protos/issues/1013#issuecomment-6556299117) investigated nine external languages, mandatory 12-dimension scores, source parser/semantic boundaries, every approved predecessor, the literal for call collision, detailed evaluation order, Map arity, source tooling and adversarial failure cases. It recommended the exact **D231-F2-PURE** option, with explicit collision and recovery.
7. **Exact final owner approval, 2026-11-01:** **"Si apruebo D231-F2-PURE, Actualiza issues en guillermomolina/protos y agrega evidencias en guillermomolina/protos-project-docs."** This is deliberate, express approval of the **entire named exact packet**, including the three equivalent ordinary calls, contextual identifier grammar, the reinterpreted bare for call patterns, grouped/member escape routes, normal lexical Range lookup and evaluation order, Closure scope/return semantics, and absence of new loop/runtime machinery. Approval also directs a **single implementation slice**, self-contained agent and human executor, and authorized docs repository commit/push.
8. The design approval is **not** permission for the AI editing agent to run code/tests or publish Protos source, and does not claim the for syntax already runs.

## GITHUB010 comparative evidence

Full alternatives, nine precedents, hard red flags and 12 explicitly justified score dimensions with confidence: [research comment](https://github.com/guillermomolina/protos/issues/1013#issuecomment-6556299117).

- Selected D: BOTH contextual families; one-param Closure expansion; ordinary lexical Range, each, eachStep; no new runtime loop. Proportional to the user-facing ergonomics requirement, incremental and backend-neutral.
- Rejected A: no syntax, fails established ergonomic goal; B/C: only one family, fails requested coexistence; E: canonical Range eligibility guard/hidden staging, unnecessary extra semantic privilege and new error precedence.
- Eliminated overengineering: generic Iterator, specialized Java range loop, JS for-in enumeration, generic parameterized trailing-Closure grammar, special break/continue, new control operation. These are distinct possible future decisions.
- Primary comparator families: Pharo/Smalltalk ordinary message iteration, Ruby each-based for, Python/Kotlin/Rust/Swift generic iterators and ranges, Lua/Go dedicated numeric for, JavaScript property-name vs value enumeration. See original issue packet for primary documentation URLs.
- Strongest case against selected D: compatibility impact on an exact ordinary named for invocation, plus ordinary programmer expectations about break/continue. Both were disclosed before exact approval. There is no evidence-based numeric performance promise.

## GITHUB021 approved-invariant and delta matrix

| Authority or invariant | F2-PURE status | Evidence/consequence |
| --- | --- | --- |
| D210/D211/D212 exact contextual while/if/until, without globally reserved identifiers | PRESERVED | for and in remain ordinary identifiers elsewhere, exact parser recognition only |
| D221 disallows general parameterized trailing Closures | PRESERVED | only a contextual for grants one binder; arbitrary foo(args){body} still appends parameterless Closure |
| CALLABLES ordinary Closure, flexible supplied arity, receiver/methodHome, return-home and InvalidReturn | PRESERVED | body is one-param Closure; Map each(key,value) causes key-only binder, surplus ignored; return is non-local as before |
| EXECUTION_AND_CONTROL contextual object, lexical lookup/slot visibility, ensure, errors | PRESERVED | body runs ordinary captured lexical context and fresh activation; no loop-local state or hidden context |
| D232 directed Range, half-open, exact Integer, strict signed eachStep, invert/reverse | PRESERVED | step sign checked against constructed range; equal Range empty rejects negative step |
| D235-B1-F frozen shared Core/Prelude canonical objects and ordinary module exports | PRESERVED | bare Range ordinary lexical lookup; no implicit import/new Core object |
| D236-B1 direction-normalizing constructors, equal retains invoked prototype | PRESERVED | Range(a,b) gives correct ascending/descending result without a for-specific sign test |
| Ordinary Array, Map, Range each contracts | PRESERVED | return result, order, snapshot/callback behavior delegated to ordinary methods |
| Owner explicitly rejects new loop mechanism | PRESERVED | no Iterator, native loop, new break/continue or sign-stage node |
| Owner requires both for families | PRESERVED | generic one form and numeric two arities |
| Existing ordinary ungrouped for(i,a,b){body} and for(i,a,b,s){body} bindings | **INTENTIONALLY REINTERPRETED — EXPLICITLY APPROVED** | exact structural collision; ordinary (for)(args){body} and receiver.for(args){body} remain available |
| Historical INC003 DEFER | SUPERSEDED ONLY FOR NEW D231 SCOPE | owner explicitly reopened D231 research and selected F2-PURE; historical DEFER record preserved |
| Earlier tentative D231 M1 staging, sign selection and canonical guard | NOT APPROVED INVARIANTS; REJECTED | new F2-PURE preserves ordinary Range and call order; no user-authorized M1 change being erased |

**DECISION_APPROVAL_PROVENANCE=PASS** — exact named candidate and disclosed collision expressly approved. **DECISION_INVARIANT_CONSISTENCY=PASS** — the only deliberate public change to prior behavior is the explicitly accepted contextual name/call precedence; D232/D235/D236 and earlier Closure contracts preserved. **REQUIRED_DURABLE_PUBLICATION** is established only after verification of the resulting docs commit and exact readback; do not prefill a fictitious SHA here.

## Concrete acceptance checks for the future one-slice Ixxx

- All three equivalences at source parser/canonical level; normal lexical Range shadows; same evaluation order (Range construction before step), runtime dynamic values; ascending/descending/equal, step sign/zero/type, huge integers, constructor errors before step.
- Generic Array snapshot+receiver result; Map key-only+map result; custom each invocation/result and Errors, Range via for-in; user-supplied Closure invocation count/arity.
- One binder as ordinary Closure param with correct this/context/methodHome/return home; nested loops; nonlocal caret and contextual return, escaped InvalidReturn, ensure, callback error, suspension, Actor/Task/P/Native where testable.
- Exact call collision with a user binding for, plus grouped/member/explicit-Closure escape; no special interpretation without attached same-line braced body; invalid for-in header, spread and wrong arity, comments/newline/postfix/nesting.
- Preserve lexical source spans, binder declaration, trivia/layout, static lexical binding/formatter/LSP, REPL, source/coverage diagnostics and existing contextual if/while/until behavior.
- No new Java host range iteration, exposed runtime loop protocol, new Core Range selectors or unapproved semantics; no unmeasured speedups asserted.

## Traceable work ownership

Decision approval belongs to D231; implementation, normative updates, tests and product release belong to a separate top-level Ixxx in guillermomolina/protos after design close. The one-slice agent/human handoff belongs under docs/project/work/Ixxx, and any eventual product PASS evidence belongs under docs/project/evidence/Ixxx.

This record is a **snapshot of design authority**, not a copy of the current GitHub scheduling state; it intentionally does not infer priority, assignee, milestone, blocked-by or native parent relationship.
