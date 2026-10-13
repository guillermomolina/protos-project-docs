# D200 — Boolean result and `elseIf` comparative evidence (2026-10-13)

**Decision authority:** [D200/#880](https://github.com/guillermomolina/protos/issues/880).  
**Ratification:** [D200 additive selector decision](../../decisions/language/D200_BOOLEAN_ADDITIVE_CHAINED_MESSAGES.md).  
**Product baseline at research/ratification:** `guillermomolina/protos@57a43b6d94697f9a6256e36c7a775dfa4a1218d7` (`0.3.343-SNAPSHOT`).  
**Scope:** additive messages `if` and `else`, no `elseIf`; strictly separate from D201/#882 and PERF044/#883.  
**Research:** source inspection only. No product code was modified and no new tests or measurements were run by this agent.

## Research reconstruction

Current [Boolean §16](https://github.com/guillermomolina/protos/blob/main/spec/semantics/VALUES_AND_COLLECTIONS.md) and [ProtosStandardBooleanProtocol.java](https://github.com/guillermomolina/protos/blob/main/src/main/java/com/guillermomolina/protos/execution/ProtosStandardBooleanProtocol.java) agree:

```text
true.ifTrue(A)  -> A() exact result
false.ifTrue(A) -> null
true.ifFalse(B) -> null
false.ifFalse(B)-> B() exact result
true.ifTrueIfFalse(A,B) -> A() exact result
false.ifTrueIfFalse(A,B)-> B() exact result
```

The selected callback need only be ordinarily invokable through `call`. Its producing argument expression is evaluated before selector invocation even if the callback is skipped. Selected callback control transfers propagate; `and`/`or` require canonical Boolean returned by the selected callback. D051 says `if` and `else` are ordinary identifiers, not special grammar. D199 owner-rejected undotted/fluent syntax; D131 fixed matching as a distinct ordinary protocol.

Initial D200 originated in an Io-like idea: change the old `ifTrue`/`ifFalse` returns to enable `condition.ifTrue(A).ifFalse(B)`. That is not observationally compatible with existing result-producing Protos. The research isolated a safer **additive selector pair** `condition.if(A).else(B)`: both selected and skipped normal branches return the original canonical Boolean, while the selected action still runs for its effects.

## Falsification cases

| Case | Result or consequence |
| --- | --- |
| `true.if(() => 42).else(() => 99)` | Runs first action, final canonical `true` |
| `false.if(() => 42).else(() => 99)` | Runs second action, final canonical `false` |
| `true.if(() => false).else(B)` | B skipped despite action result `false` |
| `false.if(A).else(() => true)` | Final `false`, not selected callback's `true` |
| `false.if(123)` / `true.else(123)` | Skipped callback neither invoked nor callability-validated |
| `true.if(123)` / `false.else(123)` | Selected callback invocation signals Error |
| `false.else(A).else(B)` | Both actions invoked; messages are not grammatically paired |
| `condition.if(makeA()).else(makeB())` | `makeA` evaluated first; `makeB` when second send reached, even if B not invoked |
| Selected action returns `Future` | Result ignored; Future not implicitly awaited/adopted |
| Selected action signals Error or NLR | No normal Boolean returned; transfer propagates |
| Custom nonstandard `if`/ `else` shadow | Ordinary dispatch, not a privileged Boolean intrinsic |
| Noncanonical receiver inheriting standard selector | Error; inheritance does not convert it to a semantic Boolean |
| Previous `true.ifTrue(() => 42)` | Still returns `42`; no migration for existing code |

Cases are semantic predictions to protect with implementation conformance, **not claims of executed tests for the new messages**.

## `elseIf` alternatives and decisive composition

The owner's selected **A** is `if`/`else` only; to model ordered action selection:

```protos
condition1.if(() => A())
    .or(() => condition2.if(() => B()))
    .or(() => condition3.if(() => C()))
    .else(() => D())
```

This uses the existing [`or` strict Boolean protocol](https://github.com/guillermomolina/protos/blob/main/spec/semantics/VALUES_AND_COLLECTIONS.md) and existing [`||` lowering `a.or(() => b)`](https://github.com/guillermomolina/protos/blob/main/spec/PROTOS_GRAMMAR.md). Each reached `if` yields a canonical Boolean indicating **prior selection**, independently of what A/B/C returned. `or` will not evaluate later callback *producer expressions* when it short-circuits, unlike a hypothetical ordinary two-argument `elseIf(conditionThunk, action)`, which evaluates argument-producing expressions before sending the message. Deferring the condition in a thunk is mandatory for proper control selection; passing an eager condition value breaks that behavior. A future selector can be added after evidence of recurring readability problems without redesigning Boolean semantics.

The chain is effect-oriented: it does **not** return A/B/C/D's values. Result-oriented selection retains `ifTrueIfFalse` or `caseOf`; public `matcher.match(subject)` and `caseOf` are unchanged. PERF044 may lower matching into conditional **physical** control flow, not Protos language `if`/`elseIf` syntax.

## Cross-system primary-source precedents / boundaries

| System | Relevant observed model and limit for Protos |
| --- | --- |
| [Io guide](https://iolanguage.com/guide/guide.html), [Io Object.io](https://github.com/IoLanguage/io/blob/master/libs/iovm/io/Object.io) | Prototype/message model, receiver-oriented control chains; Io's unevaluated message arguments and return details cannot be imported into Protos ordinary eager argument-expression semantics |
| [Smalltalk-80 control messages (GNU Smalltalk manual)](https://www.gnu.org/software/smalltalk/manual/html_node/Boolean.html) | Boolean `ifTrue:`/`ifFalse:` and blocks; receiver/message model distinct from a new grammar |
| [Self handbook](https://selflanguage.org/_static/pdf/SelfHandbook.pdf) | Object-message Boolean selection and block control; demonstrates message-level control without Protos `Object` topology |
| [Ruby `tap` / `then`](https://docs.ruby-lang.org/en/master/Object.html) | Existing named APIs distinguish returning the original receiver from returning callback result, supporting distinct Protos selector names |
| [Groovy `tap` / `with`](https://docs.groovy-lang.org/latest/html/groovy-jdk/java/lang/Object.html) | Receiver-return and closure-result APIs are distinguishable even when both execute callbacks; Groovy closure delegation differs |
| [JavaScript `if...else`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else) | Chained `else if` is a syntax composition; not evidence for ordinary Protos `elseIf` |
| [Python `if/elif`](https://docs.python.org/3/reference/compound_stmts.html) | Explicit ordered condition selection with later conditions skipped, but no ordinary selector protocol |
| [Kotlin conditionals](https://kotlinlang.org/docs/control-flow.html) / [Rust `if` expressions](https://doc.rust-lang.org/reference/expressions/if-expr.html) | Expression-producing branch values are important; Protos already retains result-producing `ifTrueIfFalse` |
| [Clojure `cond`](https://clojure.org/guides/learn/flow) | Ordered predicate alternatives need no special per-alternative matcher authority, but macro evaluation differs from Protos |

These span prototype/message, separate receiver/result callback APIs and grammar/macro approaches. They inform design, not normative Protos semantics.

## Candidate comparison and decision

Main D200 candidates:

- **S — current status quo / defer:** no new selector; users can write independent `ifTrue`/`ifFalse`, `ifTrueIfFalse` and locals. Minimal cost but no concise ordinary chaining with Boolean preservation.
- **I — full Io-style replacement:** modify both current `ifTrue`/`ifFalse` selected and skipped returns to canonical receiver. Chaining works but **breaks result-producing callers and conformance**.
- **K — skipped-only change:** change only skipped returns. **Falsified:** a selected callback returns `42`, `null`, `false`, or custom object, and the following send sees the wrong receiver.
- **A — additive `if`/`else`, no `elseIf`:** distinct return policy, no break in old messages, ordinary chaining and lazy composition with existing `or`; **owner-approved**.
- **E — explicit fluent adapter object:** preserves old messages but creates object identity/state/extra lifetime and semantics; unjustified today.
- **G — new grammar pair / desugaring:** conflicts with owner-rejected D199 and D051's ordinary syntax; not authorized.
- **L — additive pair plus standard `elseIf`:** readable flat chains, but adds public root slot and condition-thunk/producer-evaluation subtleties when `or` already composes selection. **Not selected**, can reconsider later.

### GITHUB010 12-dimensional comparison

Scores 1–5 (higher is better), each shown with HIGH/MEDIUM/LOW confidence and short reason. Scores are comparative *design judgments*, not benchmark outputs.

| # / criterion | S — status quo | I — replace old messages | A — additive approved | E — adapter | L — plus `elseIf` |
| --- | --- | --- | --- | --- | --- |
| 1 Correctness/invariants | 5/H current | 1/H breaks selected values | 5/H retains old + exact new | 4/M wrapper semantics | 4/M extra timing |
| 2 Protos alignment | 5/H minimal | 3/M ordinary but breaking | 5/H ordinary methods | 2/M new state carrier | 4/M extra selector |
| 3 Pay for what needed | 4/H no API but verbose chaining | 2/H migration expense | 4/M two frozen slots | 2/M object/lifetime | 3/M third slot |
| 4 Grow as needed | 5/H easy add | 2/H compat migration | 5/H can add later | 3/M extra model | 4/M can extend |
| 5 Future resilience | 4/M no direct chain | 2/H loses result API | 5/M both return policies | 3/M identity commitments | 4/M flat but eager args |
| 6 Scalability | 5/H established | 4/M call path similar | 5/M no state per branch | 3/M intermediates | 5/M no holder |
| 7 Conceptual simplicity | 4/H minimal API | 3/M changed mental model | 4/M two complementary policies | 2/M new builder concept | 3/M new thunk meaning |
| 8 Portability/freedom | 5/H existing | 5/M ordinary | 5/M ordinary | 4/M more machinery | 5/M ordinary |
| 9 Runtime/resource cost | 5/H known | 4/LOW unmeasured | 4/LOW extra selectors but optional | 2/LOW allocation hypothesis | 4/LOW unmeasured |
| 10 Failure/operability | 5/H known | 2/H break surprises | 5/M inherited errors | 3/M state/error paths | 4/M thunk errors |
| 11 Deferral/migration | 5/M cheap to add later | 1/H breaking rewrite | 5/M additive/no migration | 3/M complex to retire | 4/M optional addition |
| 12 Evidence/implementation | 5/H exists | 3/H precedent but breaks | 4/M grounded simple contract | 2/M no need proof | 3/M ergonomics unproven |

K and G are excluded from the numerical survivor matrix by direct disqualifying falsification / prior owner rejection, not hidden by score totals.

### Specific non-compensating red flags and alternatives

The highest-risk alternative is I: changing existing producer messages is a hard backward-compatibility break. K does not even meet its claimed fluent behavior. G contradicts ratified grammar and D199 rejection. E introduces a new public identity-bearing institution, with upfront cost and no demonstrated need. L is plausible, but its unique benefit is flat readability; no verified frequent need justifies the extra public selector.

The strongest objection to A is API proliferation: `Object` is frozen, reflective and ubiquitous, so two new globally visible slots impose a permanent vocabulary/maintenance obligation. The countervailing bounded requirement is preserving both result-producing and receiver-retaining Boolean actions without semantic exceptions. The selected pair preserves the user-space escape hatch and does not preimplement `elseIf`.

### Future stress and recovery

- Many Actors/Tasks/Contexts: add constant stable standard-root slots, **no new per-instance mutable state**, allocation or scheduling contract. No performance measurement claim.
- Explicit suspension and cancellation: no extra replay or handler boundary; reuse ordinary callback activation and preserve exactly-once effects.
- Native Image, alternate backends, Truffle PE: ordinary contract leaves representation/inline implementation free, but any guard must preserve dynamic lookup and fallback; cost of re-optimizing stays implementation-only.
- Eager argument producers: an eventually needed true `elseIf` thunk/short circuit can be added without breaking `if`, `else` or `or`.
- Result-producing source code: unchanged; existing `ifTrueIfFalse` and `caseOf` remain available.
- Potential regret: future code may need a human-readable flat many-branch chain; recover by an explicitly approved ordinary `elseIf` or separate library helper without reversing A. Opposite regret: redundant standard selector becomes difficult to retract; avoid it now.

## GITHUB021 / publication handoff

Owner exact wording **“OK apruebo A”** following the explicit A/B/C `elseIf` alternatives. In context A means **additive `if`/`else` and no `elseIf`**; it does not approve loops, matching, syntax, Io replacement or compiler architecture. D050/D051/D199/D180/D131 invariants preserved; adding two observable root slots is bounded newly approved surface. Proposed normative tests must prove all falsifiers; baseline human report `git diff --check` clean and local tests PASS is **not** evidence that the future implementation was tested.

```text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
D200_DECISION=A_ADDITIVE_IF_ELSE_NO_ELSEIF
SPECIFICATION_CHANGED_BY_DOCS_COMMIT=NO
PRODUCT_CHANGED_BY_DOCS_COMMIT=NO
EVIDENCE_CLASS=SOURCE_RESEARCH_AND_EXPLICIT_OWNER_SELECTION
IMPLEMENTATION_FUTURE_I_FAMILY=REQUIRED
```
