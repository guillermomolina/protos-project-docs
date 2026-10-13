# D200 — Additive Boolean `if` / `else` control messages

**Status:** RATIFIED — owner's explicit approval of Candidate A (2026-10-13).  
**Authority:** [D200/#880](https://github.com/guillermomolina/protos/issues/880).  
**Evidence:** [D200 comparative research and falsification](../../evidence/D200/D200_BOOLEAN_RETURN_AND_ELSEIF_EVIDENCE_2026_10_13.md).  
**Observed product baseline:** [`57a43b6d94697f9a6256e36c7a775dfa4a1218d7`](https://github.com/guillermomolina/protos/commit/57a43b6d94697f9a6256e36c7a775dfa4a1218d7) (read 2026-10-13, before implementation).  
**Implementation:** not yet performed under this decision; a separate `Ixxx` issue must apply it in `guillermomolina/protos`.  
**Normative ownership:** `guillermomolina/protos/spec/semantics/VALUES_AND_COLLECTIONS.md` §16, with ordinary call/lookup/grammar responsibilities in `CALLABLES.md`, `EXECUTION_AND_CONTROL.md`, `OBJECT_MODEL.md` and `PROTOS_GRAMMAR.md`.

This is a durable **non-normative ratification record**, not a product change or substitute for the normative `spec/` contract. All new behavior remains prospective until implementation/specification publication and human-reported tests.

## Approval provenance and candidate identity

The project owner explicitly wrote **“OK apruebo A”** in the direct exchange following the comparison of `elseIf` alternatives, where the recommended **A** was:

> Add `if` and `else` as **distinct** Boolean-receiver-returning ordinary standard messages; **do not add `elseIf`**. Express ordered `else if`-style alternatives by short-circuiting `or`/`||` over the additive `if` messages.

The earlier discussion established the **specific additive `if`/`else` contract** as the working prerequisite of those alternatives, and recommended that pair explicitly. Approval is **not** Io's full replacement of `ifTrue`/`ifFalse` returns. The selected A applies only to the bounded additive pair and to withholding `elseIf`. It does not approve an unrelated grammar, loop, matching or architectural change.

## Ratified ordinary-message contract

Add exactly two canonical-Boolean-family standard selectors as ordinary local Closure-valued `Object` slots during normal root construction, before publication/freeze:

```text
if(block)      exactly one already-evaluated argument
else(block)    exactly one already-evaluated argument
```

Each standard behavior requires the **original receiver** to be exactly canonical `true` or canonical `false`; non-Boolean receivers signal the standard Error. Lookup, shadowing, extraction, methodHome and custom overrides remain ordinary object/Closure semantics. Merely inheriting or copying a Boolean method does not grant canonical-Boolean membership.

Results after **normal** completion:

| Invocation | Action | Exact result |
| --- | --- | --- |
| `true.if(block)` | Invoke `block()` once with zero arguments | Exact original canonical `true` |
| `false.if(block)` | Do not invoke or prevalidate `block` | Exact original canonical `false` |
| `true.else(block)` | Do not invoke or prevalidate `block` | Exact original canonical `true` |
| `false.else(block)` | Invoke `block()` once with zero arguments | Exact original canonical `false` |

For selected invocations, invoke `block` through ordinary polymorphic `call` behavior. It need not be a Closure; validate invokability only when selected, and allow ordinary zero-argument arity/default errors at activation. Discard its normal result (including `null`, Boolean, numeric/object with exact identity, Future); return the original canonical Boolean. If selected callback instead signals Error, valid/nonvalid non-local return, cancellation, or explicitly suspends, preserve the ordinary transfer/resume behavior; do **not** synthesize a normal Boolean result or repeat an effect. No implicit Future adoption, flattening, awaiting, or cancellation; no hidden Task, Actor, scheduling, or truthiness.

Argument-producing expressions are evaluated exactly once **before the message**, left-to-right under existing call semantics, even when the resulting callback is unselected. A Closure literal is created but its body not run unless selected. The two sends in `condition.if(A).else(B)` evaluate B's argument-producing expression **after** A and the first send complete normally, whereas `ifTrueIfFalse(A, B)` evaluates both argument producers before selecting a branch.

Illustrative result policy:

```protos
true.if(() => 42).else(() => 99)    // true; runs only first body
false.if(() => 42).else(() => 99)   // false; runs only second body
false.else(() => 1).else(() => 2)  // false; runs both (no syntactic pairing)
```

All such operations are ordinary sequential sends; `else` is **not** grammatically paired to the preceding `if`. Calling the standard `else` twice on `false` intentionally invokes twice. Any custom `if` or `else` nearer in lookup may return something else, exactly as for other overridable ordinary messages.

## Unchanged and explicitly rejected

- **KEEP** current `ifTrue(block)` and `ifFalse(block)`: selected callback exact normal result, otherwise `null`.
- **KEEP** `ifTrueIfFalse(trueBlock, falseBlock)`: selected callback exact result, both producers eagerly evaluated in order before selection.
- **KEEP** existing strict `and`/`or` callback result validation, short circuit and `!`/`&&`/`||` mandatory ordinary-protocol lowering. `||` is not intrinsically a special `elseIf`.
- **NO `elseIf` or `elif` standard selector** in this candidate. Comparable source selection uses `condition1.if(A).or(() => condition2.if(B)).else(C)` or `(condition1.if(A) || condition2.if(B)).else(C)` with the ordinary lazy `or` protocol. This expresses **effect-oriented** conditional actions, not an equivalent result-producing `caseOf` or `ifTrueIfFalse`.
- **NO new `if`/`else` language keywords, bare undotted sends, message syntax/grammar lowering, paired syntactic continuation, or callback-evaluation rule.** D051 and owner-rejected D199/#879 remain binding.
- **NO changes** to `whileTrue`, `while`, `whileFalse` (independent D201/#882), `each`, `caseOf`, `pattern.match` (including D131 and independent PERF044/#883), or the public matching/capture ABI.
- **NO new privileged intrinsic**: any bytecode/Truffle fast path must be guarded by the exact ordinary standard selection, original receiver and preserved override/lookup/activation semantics, with fallback.

## GITHUB021 approved-invariant/delta reconciliation

| Existing authority or owner boundary | Delta | Outcome |
| --- | --- | --- |
| D050: canonical Boolean-family membership, strict dispatch | Adds two standard methods to the family, does not relax membership | PRESERVED |
| D050: `ifTrue`/`ifFalse` result-producing operations | Existing messages untouched | PRESERVED |
| D050: `ifTrueIfFalse`, `and`, `or`, `not` | Existing contracts untouched | PRESERVED |
| D051: ordinary messages; no dedicated `if`/`else` syntax | Two ordinary slots only | PRESERVED |
| D199: reject fluent/undotted sugar and `foo.member { }` reinterpretation | No grammar changes | PRESERVED |
| D180/D044: `whileTrue` semantics/selector | No loop change | PRESERVED |
| D131: matching authority and `caseOf` result | No matching change | PRESERVED |
| Root `Object` publication/frozen topology | **Observable additive slot names `if`, `else`** during bootstrap | EXPLICITLY APPROVED bounded new API |
| No `elseIf` yet | `or` composition used instead | EXPLICITLY APPROVED Candidate A |

```text
D200_CANDIDATE=A_ADDITIVE_IF_ELSE_NO_ELSEIF
OWNER_APPROVAL=2026-10-13_EXPLICIT
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
RATIFICATION_SCOPE=PUBLIC_BOOLEAN_PROTOCOL_ONLY
NORMATIVE_SPEC_CHANGED_BY_THIS_PUBLICATION=NO
PRODUCT_IMPLEMENTED_BY_THIS_PUBLICATION=NO
TESTS_RUN_BY_AGENT=NO
HUMAN_REPORTED_EXISTING_LOCAL_TESTS=PASS
HUMAN_REPORTED_EXISTING_GIT_DIFF_CHECK=CLEAN
FUTURE_TESTS_AFTER_IMPLEMENTATION=REQUIRED
D201_LOOP_RETURNS=INDEPENDENT_OPEN
PERF044_MATCHING_INLINE=INDEPENDENT_OPEN
```

## Why / incremental design

The older result-producing `ifTrue`/`ifFalse` cannot safely chain all callback outcomes because the next send receives the previous callback's arbitrary result. The new pair provides the complementary, receiver-Boolean-preserving behavior without breaking existing result expressions or introducing an intermediate fluent object, message-pair grammar, new type, hidden Task, implicit truthiness or new matching authority.

The stronger alternative of changing existing `ifTrue`/`ifFalse` to Io-like receiver return breaks existing conformance and consumers requiring `42`/`43`/chosen `Future`. Skipped-only return changes also fail when a selected callback returns a non-Boolean. A distinct intermediate builder adds state, identity, allocation and a new control institution when the canonical Boolean is sufficient.

The strongest argument against the selected pair is expanding the standard frozen `Object` slot surface for behavior that can be expressed with existing `ifTrueIfFalse` and locals. The concrete current benefit is transparent two-way action chaining that retains its control Boolean, plus a composition path to multiple alternatives with standard lazy `or`. Runtime/performance benefits are **not claimed**: tests and compiled graphs must determine the cost.

Add `elseIf` later only if demonstrated real-source readability needs justify another selector and its unusual eager argument-producing-expression timing. The owner approved **not adding it now**, not a permanent language-wide ban.

## Implementation handoff — I097

The approved product implementation is tracked separately in [I097/#885](https://github.com/guillermomolina/protos/issues/885), created **after** this decision and its evidence were durably published. Its initial state is READY, not implemented or validated. It must consume the exact contract in this record and publish code, normative spec and conformance through the human-executor gate. No new `elseIf`, loop or matching capability is delegated.

## Next step

After verification of this durable record, update D200's live issue and allocate one I-family implementation item for the bounded spec + runtime + conformance + guide reconciliation in `guillermomolina/protos`. Agent edits; human runs tests/checks/commits/push under root `AGENTS.md`. Product version/changelog only after all tests green and immediately before human commit, with **no tests after version metadata changes**. No implementation claims until published and verified.
