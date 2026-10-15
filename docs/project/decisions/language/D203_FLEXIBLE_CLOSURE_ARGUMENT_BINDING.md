# D203 — Flexible ordinary Closure arity, null omission, colon initializers, and positional parameter order

**Role:** durable, non-normative owner-ratified language decision. **Owner approval:** 2026-10-15, in the live conversation immediately following the exhaustive/adversarial D203 discussion. **Live decision:** [D203/#901](https://github.com/guillermomolina/protos/issues/901). **Implementation:** [I105/#902](https://github.com/guillermomolina/protos/issues/902).
**Product HEAD inspected for this decision:** [`0c617058487be96ea05b26c04df417eda6985cbe`](https://github.com/guillermomolina/protos/commit/0c617058487be96ea05b26c04df417eda6985cbe). **This record does not itself change normative product specification or shipped behavior.**
**Supporting falsification:** [D203 research/evidence](../../evidence/D203/D203_ARGUMENT_BINDING_FALSIFICATION_2026_10_15.md).

## Exact owner-approved semantic and syntactic changes

The owner expressly approved the proposed flexible parameter policy and then expressly selected `:` (not `=` and not a new `:=`) as explicit parameter-initializer notation, acknowledging that initializer syntax inside a signature is not literally an ordinary source-level slot-creation expression.

1. **Flexible ordinary Closure positional binding.** A Closure's declared non-rest parameters describe *bindings*, not minimum/maximum runtime arity requirements. Any supplied positional argument is bound to the matching formal in declaration order. Missing arguments do **not** cause an automatic arity Error. Extra supplied arguments are evaluated in the ordinary eager call order but, absent a trailing `...rest`, ignored. No implicit guest `args` binding, global argument-count intrinsic, magic omitted-value object or second Closure category is introduced.
2. **Missing defaults to canonical `null`.** Each omitted non-rest parameter without an explicit default is established as a **PRESENT** new local activation-Context slot with canonical Protos `null`. It is never left ABSENT and never draws a value from outer lexical lookup or receiver fallback. A caller who explicitly supplies canonical `null` has supplied an argument, and must not trigger an explicit default.
3. **Explicit initializer syntax: `name: expression`.** Inside an ordinary Closure parameter list, `:` denotes an optional *fallback initializer* evaluated iff that positional argument is omitted; `=` is no longer a parameter-default declarator. This signature-specific use resembles creation/initialization semantically, but **does not** itself execute ordinary `name: value` slot creation or ordinary `name = value` mutation. Neither operator's meaning in regular program expressions is changed.
4. **Left-to-right activation binding unchanged.** Evaluate any explicit initializer exactly once at the parameter's turn in the **genuine invocation activation** with its correct `this`, `context`, captured lexical chain, dynamic return home and control state. Create that parameter's fresh local slot after obtaining its value. Later slots are ABSENT until reached; no eager prefill or hoisting; local creation conflicts, close/freeze/remove/recreate, effect ordering, nested closures, escaped contexts, suspension, NLR, cancellation and guest Error propagation retain their established rules. An earlier default that creates a later parameter name can still cause duplicate creation when that later parameter is bound. The presence of an omitted `null` parameter shadows any outer binding only *after* that parameter's turn.
5. **Arbitrary interleaving.** Any non-rest parameter is either bare `name` or `name: expression`; those forms may appear in any positional order, with unique names. The prior grammar restriction `required* defaulted* rest?` is eliminated. No named arguments, holes, positional skipping or keyword-argument system is added.
6. **Explicit rest unaffected.** A trailing `...rest` is optional and final; it establishes a fresh frozen standard Array containing *only* the genuinely supplied positional suffix after all preceding formals have consumed available positions, never default or implicit-`null` values. `(...args)` can explicitly inspect the complete caller-supplied vector without an ambient magic `args`.
7. **Call/ABI invariants.** The target and all caller-supplied expression/spread/trailing-Closure side effects remain evaluated in the ordinary order before ordinary dynamic `Object.call` dispatch and Closure activation. Preserve receiver, `methodHome`, captures, `super`, polyglot/host entry, actor/task/process control, DEBUG correctness, NORMAL optimization freedom and PLAT060 B5 universal physical CallTarget header: slots [0..4] followed by the real supplied arguments [5...]. No change to D202 or I104's approved physical contract is implied.
8. **Opt-in exact arity and existing public contracts.** Requiring an exact count is a **specific operation's explicit responsibility**, not a universal ordinary Closure requirement. A rest-only `(...args)` signature can validate `args.size()` and explicitly bind names, at its own cost. The original supplied count is not recoverable merely from a signature like `(x,y,...rest)` because `f()` and `f(null,null)` yield indistinguishable bound `x`, `y` and `rest`. Do **not** assume a fictitious automatic exact-arity guard. Existing independent standard-protocol exact-arity requirements (`Object.init`, `ifNull`/`ifNotNull`, `fail()`, standard Boolean/control, foreign/native methods and applicable P boundaries) remain obligations unless individually changed by another explicit owner decision. They require audit and a conformant explicit check/standard implementation path, not accidental relaxation through this language-level change.
9. **No general `:=` operator.** A third create-or-assign operation was considered and expressly rejected for D203. Bare `=` can locate a captured binding; bare `:` creates only in the current Context. A global `:=` introduces unresolved selection/evaluation/visibility behavior; it is not needed for a parameter fallback initializer and is not approved now.

## Owner-approved observable examples (illustrative new syntax)

```protos
identity: (x) => x
identity()                 // null
identity(1, 2)             // 1; extra value evaluated, then ignored

values: (a: 10, b, c: 30, d) => Array(a, b, c, d)
values()                  // Array(10, null, 30, null)
values(1, null)           // Array(1, null, 30, null)
values(1, 2, 3, 4, 5)    // Array(1, 2, 3, 4); fifth argument ignored

tail: (a: 10, ...rest) => rest
tail()                    // frozen Array()
tail(7, 8, 9)            // frozen Array(8, 9)
```

Examples express **approved prospective behavior**, not observed runtime results or tests. Explicit `null` suppresses the corresponding `:` initializer even though a missing argument with no initializer also produces `null`.

## GITHUB021 reconciliation of prior decisions

| Prior authority | Owner-approved D203 replacement | Surviving invariants |
| --- | --- | --- |
| [D003/#161](https://github.com/guillermomolina/protos/issues/161), previously ratified as amended | Removes required-then-defaulted grammar order; removes ordinary Closure missing/excess automatic arity Errors; changes signature initializer token from `=` to `:` | No name collisions, sequential Context binding and presence, defaults evaluated in real activation with all ordinary effects, supplied-only trailing rest, argument-evaluation order |
| [D202/#898](https://github.com/guillermomolina/protos/issues/898) and [I104/#899](https://github.com/guillermomolina/protos/issues/899) | D203 authorizes a **new guest semantic contract**, not retroactive misreporting of I104 | NORMAL-first pre-lowering, compliance acceptance, rich/DEBUG correctness and PERF042 measurements independently pending |
| [PLAT060-B5/#895](https://github.com/guillermomolina/protos/issues/895) | **No change** | Same physical CallTarget ABI header, argument array offsets, rich activation publication and dispatch semantics |
| D179 Context presence/removal | **No change** | Distinction between PRESENT `null` and ABSENT, captured lexical contexts by reference, only binding at its semantic turn |
| `OBJECT_MODEL.md` / `EXECUTION_AND_CONTROL.md` source `:`/`=` | **No change** | `:` only creates existing-conflict-checked slots; `=` only assigns to a valid existing destination; writes never delegate |

The owner made the exact replacement choice knowingly after adversarial counterexamples about duplicate binding, irreversible loss of caller-supplied count in nominal parameters, standard-library failures, source-level `:=` ambiguity and possible runtime node cost. Therefore D203 **amends only the named D003 rules**. It does not reopen completed D003, D202 or I104 historical Issues.

## Implementation, risks and migration

- **Normative product owners still pending implementation:** `spec/semantics/CALLABLES.md`, `spec/PROTOS_GRAMMAR.md`, relevant Core/control/error/concurrency specifications only where their inherited arity claims truly change, and spec changelog. Until a human-published I105 product commit makes the normative files current, the existing product specifications remain the operative shipped contract. A project-docs ratification is decision evidence, not the normative spec.
- **Source migration:** signature `a = expression` becomes `a: expression`; previously invalid interleaved signatures become valid; previously invalid calls with too few/many ordinary Closure arguments produce a normal binding/body outcome or independent guest error. Plain `:` and `=` expressions outside signatures do not change.
- **Audit surface:** lexer/parser/AST, canonical binding analysis, Bytecode DSL emission, optimized NORMAL unary/required-prefix binder, generic/DEBUG/inline callback/rich/host fallback, zero-parameter rejection root, Core prelude source Closure methods, standard native protocols, conformance suites, LSP/formatter/tutorials.
- **Performance not proven:** a bound-argument presence check remains necessary for missing `null`; bytecode upper-bound checks/rejection roots may disappear, but naïve default-style branches may add bytecodes elsewhere. No new Builder/Tier-2 metric was reported by this decision; [PERF042/#871](https://github.com/guillermomolina/protos/issues/871) stays independently open.
- **Human-executor:** the owner reported a clean `git diff --check` and locally passing tests for the pre-implementation workspace, **not** tests of D203 changes. The agent does not run product builds/tests/push, and must not claim I105 validation from that report. Product changelog/version bump occurs only after human green validation and immediately before commit, with no tests after that promotion. Scoped docs publication for this decision was explicitly authorized by owner.

```text
DECISION=D203
OWNER_APPROVAL=2026-10-15
DECISION_STATUS=RATIFIED_AFTER_DURABLE_PUBLICATION
SOURCE_BASELINE=0c617058487be96ea05b26c04df417eda6985cbe
IMPLEMENTATION=I105/#902
NORMATIVE_PRODUCT_AMENDMENT=NOT_YET_PUBLISHED
SELF_AS_POSITIONAL_PARAMETER=NO
FLEXIBLE_MISSING=null
FLEXIBLE_EXCESS=IGNORE_IF_NO_REST
EXPLICIT_PARAMETER_DEFAULT_SYNTAX=:
PLAIN_AND_DEFAULT_PARAMETER_ORDER=FREE
REST=FINAL_SUPPLIED_SUFFIX_ONLY
BINDING_ORDER=LEFT_TO_RIGHT
NEW_UPSERT_OPERATOR=NO
NEW_ARITY_INTRINSIC=NO
PREEXISTING_STANDARD_CONTRACTS=PROTECTED
PLAT060_B5_ABI_CHANGE=NO
PRODUCT_BUILDS_AND_TESTS_BY_AGENT=NO
```
