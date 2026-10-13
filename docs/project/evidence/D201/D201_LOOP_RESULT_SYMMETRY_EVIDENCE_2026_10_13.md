# D201 — Comparative research, matrix and adversarial falsification (2026-10-13)

**Formal decision:** [D201/#882](https://github.com/guillermomolina/protos/issues/882).
**Selected:** Candidate **D** only, following explicit owner approval *after* a stronger falsification pass.
**Decision:** [D201 ratification](../../decisions/language/D201_CLOSURE_WHILE_RESULT_SYMMETRY.md).
**Research baseline:** \`guillermomolina/protos@8f7e5ea5b7ad75f8052adf7ce51945539348b4a1\`. Prior investigation also observed \`57a43b6d94697f9a6256e36c7a775dfa4a1218d7\`; current implementers must use actual HEAD.
**Evidence class:** source inspection and semantic reasoning; no code edits, builds, test runs or performance measurements by the agent in this decision.

## Sources checked in Protos

- \`AGENTS.md\`, \`AGENTS.work/DESIGN.md\` (GITHUB010/GITHUB021), \`AGENTS.work/REFERENCE.md\` (small universe, ordinary messages, pay-as-you-grow), \`AGENTS.work/COORDINATION.md\` (owner approval, durable record, live issue transaction).
- \`spec/semantics/EXECUTION_AND_CONTROL.md\` §17: \`whileTrue\` validates receiver and body semantic Closures up front; exact canonical Boolean predicate, zero-arg activations, result historically \`null\`; same effects/control and suspension invariants. \`spec/semantics/CALLABLES.md\`: ordinary bound method, lookup/extraction and Closure activation. \`spec/semantics/VALUES_AND_COLLECTIONS.md\` §16: result-returning Boolean \`ifTrue\`/\`ifFalse\`; D200 separately approves effect-oriented receiver-returning \`if\`/\`else\`. \`spec/PROTOS_GRAMMAR.md\`: ordinary trailing-Closure call syntax. \`spec/concurrency/FUTURES_AND_TASKS.md\` §§26–29: a returned Future can be adopted by **outer** Future resolution, not by a loop.
- \`src/main/java/com/guillermomolina/protos/execution/ProtosStandardObjectProtocol.java\`: direct standard \`whileTrue\` ignores body result and returns \`ProtosNullValue.INSTANCE\`.
- \`src/main/java/com/guillermomolina/protos/execution/ProtosStructuredDispatchLowerer.java\`: separate structured while continuation with final constant \`null\`.
- \`src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java\`: inline literal while with final constant \`null\`; admission/fallback separate. \`ProtosBytecodeRootNode.java\`: \`PreparedWhileCall\` and guarded canonical standard while selection.
- \`protos/tests/conformance/control/while-basic-and-validation.protos\`: zero \`null\`, multiple activations, errors, \`!Object.hasSlot("while")\`, overrides.
- \`protos/tests/conformance/control/while-binding-and-transfer.protos\`: NLR/Error identity and validation timing; \`while-future-ownership.protos\`: discarded Future/result and task ownership; \`while-suspension-and-cancellation.protos\`: exact-once suspension/cancellation.
- D044/#202, D180/#762 (old while result/no \`while\`), D050/#208, D051/#209, D200/#880 (new Boolean pair) and I097/#885.

## External primary-source models: at least 5, three approaches

| Approach | System/source | What the precedent tests; what it does NOT imply for Protos |
| --- | --- | --- |
| Prototype/message | [Self Handbook](https://selflanguage.org/_static/pdf/SelfHandbook.pdf) | Message-level control and explicit loop-exit result options; no requirement to copy its loop-result policy |
| Object/block messages | [GNU Smalltalk Boolean and block control](https://www.gnu.org/software/smalltalk/manual/html_node/Boolean.html) | Message-sent callback control; precise dialect-specific \`whileTrue:\` return differences are not authority for Protos |
| Prototype/message | [Io implementation](https://github.com/IoLanguage/io/blob/master/libs/iovm/source/IoObject.c), [Io guide](https://iolanguage.com/guide/guide.html) | Ordinary messages and control families, but Io's unevaluated message argument model differs fundamentally; no unverified claim about exact \`while\` result |
| Dynamic imperative | [Ruby control expressions](https://docs.ruby-lang.org/en/master/syntax/control_expressions_rdoc.html) | Normally \`while\` yields \`nil\`, explicit \`break\` can carry a value; values are deliberately opt-in |
| Expression-oriented typed | [Rust loop expression reference](https://doc.rust-lang.org/reference/expressions/loop-expr.html) | Predicate \`while\` uses unit result; \`loop\` and labeled blocks can take explicit \`break\` values; counters accidental retention |
| Expression/statement distinction | [Kotlin control flow](https://kotlinlang.org/docs/control-flow.html) | \`if\` as value expression, \`while\` as loop statement: symmetry between branches and loops is not inevitable |
| Imperative statement | [Python compound while specification](https://docs.python.org/3/reference/compound_stmts.html#the-while-statement) | \`while\` and optional \`else\` are statements; loop-else controls normal-completion flow rather than returning last body value |

These are comparative examples, not a normative appeal to popularity or a requirement to reproduce other implementations.

## Decision-space comparison and distinguishing behavior

| Candidate | \`whileTrue\` normal result | Additional standard selector | Strongest objection |
| --- | --- | --- | --- |
| A status quo | \`null\` for all counts | none | Inconsistent work-vs-control return family after D200 |
| B changed existing | last body/\`null\` if zero | none | Breaks existing return; no short control-return counterpart |
| C | \`null\` | \`while\` returns last body/\`null\` | Short selector now means work return, opposite D200 naming model |
| **D selected** | **last body/\`null\`** | **\`while\` returns final predicate \`false\`** | Breaking old value; possible retention and Future outcome changes |
| E | \`null\` | \`while\` returns final predicate \`false\` | Adds control selector without the result-oriented counterpart |
| F | \`null\` | opt-in differently named last-body selector | More vocabulary, but non-breaking; serious migration alternative |
| G | \`null\` | executed-at-least-once Boolean | Different information from final predicate, fails claimed symmetry |
| H defer | unchanged | none | User-level accumulation, leaves result/control convention incomplete |

Zero iterations: D \`whileTrue -> null\`, \`while -> false\`. One: D \`whileTrue -> B1\`, \`while -> false\`. Many: D \`whileTrue -> Bn\`, \`while -> false\`. \`Bn\` may itself be \`null\` or either Boolean, any identity-bearing object, a Future, or large graph. Thus last-body \`null\` is not a reliable indicator of zero iterations. \`while\` returns \`false\` on all normal exits; it cannot indicate whether body ran. Both properties were considered, not hidden.

## GITHUB010 comparison matrix

1–5 higher is better. Each cell is **score/confidence: bounded justification** (H=HIGH, M=MEDIUM, L=LOW). These are reasoned rankings, **not measured benchmarks**.

| Criterion | A | B | C | D | E | F | G | H |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 correctness/invariants | 5/H current | 3/H breaks old return | 4/M additive only | 3/H explicit reopening | 4/H additive | 5/M old intact | 4/M alternate meaning | 5/H current |
| 2 Protos alignment | 3/M asymmetric | 4/M work result | 3/M naming inversion | 5/M systematic messages | 4/M control chain | 4/M opt-in work | 3/M special count Boolean | 3/M no uniformity |
| 3 pay for need | 5/H no cost | 3/M retention for all | 3/M extra slot | 2/M value retained broadly | 4/M cheap selector | 4/M opt-in retention | 3/M extra flag | 5/H no cost |
| 4 incremental growth | 4/M can add later | 4/M later while | 3/M naming debt | 4/M two policies now | 4/M later work selector | 5/M can add further | 3/M other selector needed | 4/M still flexible |
| 5 future resilience | 4/M open | 4/M data loops | 3/M pair ambiguity | 5/M coherent protocol | 4/M cheap control | 5/M opt-in | 3/M conflates purpose | 4/M open |
| 6 scalability | 5/H discard | 3/L possible retention | 5/M old fast path | 3/L possible retained graphs | 5/M no body retention | 5/M opt-in | 4/M Boolean state | 5/H discard |
| 7 conceptual simplicity | 3/M extra asymmetry | 4/M single rule | 2/M counterintuitive | 5/M work vs control | 4/M control-only | 3/M selector vocabulary | 3/M Boolean meaning | 3/M work-around |
| 8 portability | 5/H ordinary | 5/M ordinary | 5/M ordinary | 5/M ordinary | 5/M ordinary | 5/M ordinary | 5/M ordinary | 5/H current |
| 9 runtime/resource | 5/H discarded | 2/L retained value | 4/M additive | 2/L value-lifetime cost | 4/M control-only | 4/M opt-in overhead | 4/M tracking | 5/H current |
| 10 failure/operability | 5/H known | 3/M downstream Future | 4/M additive | 3/M changed propagation | 5/M old intact | 5/M old intact | 4/M ambiguous count | 5/H current |
| 11 deferral/migration | 3/M later break | 2/H breaking now | 4/M additive | 2/H breaking now | 4/M additive | 5/M additive | 4/M additive | 3/M potential later break |
| 12 evidence/risk | 5/H code exists | 3/M new semantics | 3/M name policy | 3/M three paths | 4/M small extension | 4/M incremental | 3/M unused model | 5/H code exists |

**Non-compensating concerns:** D has demonstrably weaker present-resource proportionality and compatibility than F/H. D's benefit is one coherent conceptual return convention across current Boolean and loop families; this is a design preference **expressly selected by owner**, not a measured speed win. F remains a credible alternative when absolute backward compatibility is a requirement. The owner accepted the specific break and its operational risk rather than treating numerical scores as decisive.

## Stronger falsification of D and disposition

1. **Not fully symmetric with D200:** Boolean skipped callback is not callability-validated; loop body must be Closure-validated before first condition, even if condition is false. Keep distinction. D's symmetry is confined to the *normal-result policy*.
2. **Not same receiver preservation:** \`if\`/\`else\` returns original Boolean; \`while\` returns terminal predicate \`false\`, never Closure receiver. Chaining \`condition.while(body).else(done)\` invokes \`done\` after every normal completion (including zero iterations), but not upon Error/NLR/cancellation. It is not zero-iteration detection or \`ensure\`.
3. **Last result retention:** the final body value must remain observable through the final predicate, potentially through explicit suspension. Large retained graphs can increase live memory. When result unused, opt-out may be optimized if semantics preserved, not guaranteed. Measure rather than assert negligible cost.
4. **Future adoption spillover:** direct loop does not await/adopt Future returned by body; \`whileTrue\` now returns it. If an **outer asynchronous Closure** returns that Future, existing outer resolution adoption can change result/error/cancel and expose cycles, whereas current \`null\` would not. Owner explicitly accepted this breaking consequence.
5. **Terminal predicate effects:** body B is returned only after the terminal condition has completed normally. It may mutate B; identity remains B. If it errors/NLRs/cancels, no normal body result can escape.
6. **Zero ambiguity:** body returning \`null\` makes positive and zero counts indistinguishable; \`while\` final \`false\` cannot report whether anything ran. No promise of count/flag.
7. **Existing API and reflection:** D044's constant \`null\` and D180's absence of standard \`while\` both intentionally reversed. Object frozen-slot enumeration and inherited lookup change; local overridden \`while\` still dispatches ordinarily.
8. **Three compiler paths:** direct native method and structured/inline paths currently hardcode null; exact result capture and failure/suspension semantics require synchronized updates. Guard/canonical-selection correctness and fallback parity cannot be assumed.
9. **User-space alternative:** last body could be captured by a local variable, so D is not strictly essential expressive power. Selected value is a consistent message-level result protocol and less per-use boilerplate, knowingly trading off retention/migration.
10. **Zero new grammar/authority:** no hidden \`while\` parser token or changes to \`if\`, \`else\`, \`whileFalse\`, \`ensure\`, Task/Actor/Process semantics. Ordinary shadowing and extraction are required.

## Adversarial incremental-design gate

- **Smallest current solution:** A/H costs nothing; F offers safe opt-in result capability. D is **not** the minimum binary/runtime change, but targets a current explicitly owner-valued cross-family control protocol.
- **Can add later:** An additive selector F/E can be added later without breaking \`whileTrue\`; D's chosen change to existing \`whileTrue\` cannot be made fully backward-compatible whenever introduced. Deferral moves, rather than eliminates, the break.
- **Actual rewrite if deferred:** the method's normal return, normative specification, conformance, direct native implementation, structured dispatch and literal inline path must all change; there is no evidence for a total engine rewrite.
- **Pay-as-you-grow:** current use of \`whileTrue\` acquires a last-result obligation, even when caller discards it, unless proven safe to optimize away. \`while\` permits a pure control-return path that need not retain body results. Benchmarks/profiling required to distinguish implementation cost from obligation.
- **Disqualifying red flag?** no semantic impossibility found, but compatibility/resource trade-offs are documented, accepted and assigned to implementation regression proof.

## Test/evidence protocol for follow-up I-family

Semantic predictions to convert into conformance (not executed here): zero/one/many; exact identity; last body \`null\`, Booleans, primitive, object, Future; terminal predicate mutates body; final predicate fails; body fails/NLR; default-argument side effects; first false skips *activation* but still validates body Closure domain; non-Closure condition/body/receiver; Object standard slot visibility, override/shadow/extraction; Future returned but not awaited; outer Future adoption; suspension within final predicate and resumption exact once; cancellation with cleanup exact once; direct vs structured vs inline and fallback semantics; compare allocations/retained graph when discarding \`whileTrue\` vs \`while\`.

**Publication statement:** Owner explicitly approved exact **D** after these caveats. No product implementation, test execution, spec change, or measurements were performed by the ratification agent.
