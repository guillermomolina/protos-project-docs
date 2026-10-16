# D204-A — Suspension capability and proof-boundary investigation / falsification

**Record type:** durable, non-normative decision evidence; not an executable or benchmarking result.  
**Issue:** [D204/#905](https://github.com/guillermomolina/protos/issues/905).  
**Owner's exact selection, 2026-10-16:** “entonces apruebo A, para este ISSUE, pero quiero implementar esta falencia, y que el lowerer lo utilice, (no solo el PLAT), que sea un caso general.”  
**Selected conclusion:** `NO_NEW_SEMANTIC_DECISION_REQUIRED`.  
**Canonical decision record:** [D204-A](../../decisions/language/D204_SUSPENSION_CAPABILITY_AND_PROOF_BOUNDARIES.md).  
**Inspected product checkpoint:** [`guillermomolina/protos@d1697153256dc3952008cb40e507f09aea0f7101`](https://github.com/guillermomolina/protos/commit/d1697153256dc3952008cb40e507f09aea0f7101).  
**Scope:** semantics/proof analysis and read-only source inspection. No Java tests, builds, native tests, benchmarks or independently executed Protos examples are claimed.

## Question, terms and hypotheses

The question was whether the source language needed a new guarantee/annotation to allow a compiler to conclude that a particular Closure body, physically emitted root or complete invocation cannot suspend. The originating PERF042 witness compared the full program `identity: (value) => { value }\nidentity(1)` with a handwritten Builder oracle. Static instruction-count differences are not evidence of dynamic cost or legality of deleting a fallback.

Definitions:
- **Semantic suspension:** the executing Protos continuation pauses at a normatively defined suspension point and may resume there. This is not OS-thread blocking, async work creation, a Future return value, Error, non-local return, cancellation unwind or any Bytecode instruction by itself.
- **Observed no suspension:** one run did not suspend. This does *not* prove an all-executions property.
- **Canonical-body proof:** a closed claim over operations admissible in an exact canonical expression/definition, lexical analysis and scope. A body is smaller than its invocation.
- **Emitted-root proof:** covers the actual physical body *and its entry* under a specified compiler profile and previously established transformations.
- **Complete-invocation proof:** additionally covers evaluation order and dynamic target selection, arguments/spread, defaults, callbacks, cleanup and transitive operations.
- **Guarded site:** a proof restricted to specific target/Context/identity/prelude/profile assumptions; the guard's miss requires sound fallback.
- **UNKNOWN:** the compiler has no sound proof; not a new language-level fact, and not proof that suspension *will* occur.

Normative anchors in the product checkpoint:
1. [FUTURES_AND_TASKS.md](https://github.com/guillermomolina/protos/blob/d1697153256dc3952008cb40e507f09aea0f7101/spec/concurrency/FUTURES_AND_TASKS.md) (pending versus completed Future observation, cancellation wake-up and resume precedence, §29 suspendible host entries).
2. [CALLABLES.md](https://github.com/guillermomolina/protos/blob/d1697153256dc3952008cb40e507f09aea0f7101/spec/semantics/CALLABLES.md) (ordinary `Object.call`, D203 sequential defaults/rest/spread, lexical/ReturnHome).
3. [EXECUTION_AND_CONTROL.md](https://github.com/guillermomolina/protos/blob/d1697153256dc3952008cb40e507f09aea0f7101/spec/semantics/EXECUTION_AND_CONTROL.md) (structured loops and `ensure`, explicit suspension without replay or early cleanup).
4. [ERRORS.md](https://github.com/guillermomolina/protos/blob/d1697153256dc3952008cb40e507f09aea0f7101/spec/semantics/ERRORS.md) (non-resumable Error; cleanup unwinding).
5. [OBJECT_MODEL.md](https://github.com/guillermomolina/protos/blob/d1697153256dc3952008cb40e507f09aea0f7101/spec/semantics/OBJECT_MODEL.md), [ACTORS.md](https://github.com/guillermomolina/protos/blob/d1697153256dc3952008cb40e507f09aea0f7101/spec/concurrency/ACTORS.md) and applicable host/foreign rules (dynamic selection/ownership).

## Adversarial proof-boundary matrix

These are **semantic counterexamples, not claimed test executions**.

| Scenario | Sound boundary; falsification of an over-broad proof |
| --- | --- |
| `(value) => { value }` | Exact own-parameter read may prove the *body* non-suspending. It says nothing about the dynamic selection of `call` at an arbitrary caller. |
| `identity(1)` after a lexical binding | The binding can carry a candidate, but dynamic `call` overrides and D179 changes defeat an unconditional site proof. A guarded fast path must retain fallback. |
| Long literal/local sequences and many nested safe Boolean paths | A compositional proof should establish all reachable children without arbitrary branch-count limits; only genuinely proven selection can erase a dynamic alternative. |
| `a + b` | Ordinary message/operator dispatch may be overridden. Numeric syntax alone does not establish a non-suspending invocation; any frozen-Core fast path needs its real guard. |
| `f.call: ...` or inherited override | A Closure's own body proof does not cover a newly selected `call` implementation. |
| Captured binding, remove/recreate, shadowing | Lexical identity is not guaranteed semantic presence, nor constant selected value; a reference remains live by D179. |
| Missing argument with `(x: someCall())` | Body might be literal; selected default executes before it and can suspend or fail. Explicitly supplied argument can avoid the default for that invocation, but only under a proven call shape. |
| Spread argument | Eager argument/spread evaluation and expansion may invoke ordinary behavior; not covered by a proof of callee body. |
| Recursive/mutually recursive Closure | A local syntactic check cannot infer a bounded transitive no-suspension guarantee if calls or their targets are dynamically selected. |
| `Future` constructed or returned | Producing/returning a Future alone is not the pending `value()` observation suspension. |
| `future.value()` pending / already completed | Pending observation suspends; completed returns immediately. A previously completed observed run is not a universal static no-suspension proof. |
| Canonical ifTrue/while/each callback | Selection, validation, invocation and possible callback suspension must be accounted for, including fallback from an inline candidate. Non-selected callbacks must not execute spuriously. |
| `ensure` body and cleanup | Suspending a protected body does not trigger cleanup. Cleanup may itself suspend on unwind; neither can be omitted from a complete-invocation proof. |
| `ReturnHome`, Error or cancellation | These are independent transfer/lifetime/unwind semantics; `non-suspending` does not authorize eliminating their necessary machinery. |
| Host/native/foreign callbacks | External selection/entry and behavior need explicit authority; absent a strong proof, `UNKNOWN`. |
| RootActor host entry, cross-Task and cross-Context projection | Entry may be suspendible without forcing runtime machinery for non-suspending runs. Proofs scoped to another destination Context/profile cannot be imported as physical truth. |
| Reparse and DEBUG | Same retained canonical identities may permit reusing a canonical proof; emitted decisions and Bytecode-local artifacts must be independently valid per parse/profile. |

### Actual source-level gap identified

At the inspected product checkpoint:
- `CanonicalCompilationFacts.java` stores isolated lexical analysis, persistent-frame authority, ReturnHome observability, and binding call candidates. It does not yet own a general compositional suspension-proof lifecycle.
- `CanonicalToBytecodeLowerer.java` evaluates `provesNonSuspendingBody` after `builder.endRoot()`, with narrow literal/own-parameter cases and effective-body exceptions.
- `ProtosSemanticBytecodeRootNode.java` publishes `provablyNonSuspendingBody` and uses it in `TryDirectCallZero/One`, `AdmitsStraightLineSourceCall` and `IsContinuationForCall`.
- `emitContinuationComposition` physically emits `IsContinuationForCall`, `Yield` and `ResumeContinuation`. A fact calculated *after* emission cannot retroactively remove them.
- The generic dynamically selected paths remain semantically necessary on guard misses.

**Conclusion:** this is a bounded compiler-proof/emission-lifecycle gap under already ratified PLAT061-B, not an unspecified guest semantics. [I107](https://github.com/guillermomolina/protos/issues/906) owns the implementation. No implementation or performance result is claimed here.

## Comparative language/runtime precedent (primary references)

1. **Kotlin** — `suspend` is part of callable typing and authorizes suspension points; suspending calls need a permitted coroutine context. Declaration/type effect and physical suspension are separate. [Kotlin language coroutines documentation](https://kotlinlang.org/docs/coroutines-basics.html), [Kotlin language spec](https://kotlinlang.org/spec/asynchronous-programming-with-coroutines.html).
2. **C#** — `async` functions can suspend at `await`, with a generated state-machine/builder; an already completed awaiter does not suspend. Syntax and lowered machinery are distinct. [C# language specification: async functions](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/classes#1514-async-functions), [await operator](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/await).
3. **JavaScript** — async functions and `await` delimit asynchronous suspension in the language, with Promise-based semantics distinct from the creation or return of a Promise. [ECMAScript specification: AsyncFunction](https://tc39.es/ecma262/#sec-async-function-definitions), [Await](https://tc39.es/ecma262/#sec-await).
4. **Python** — `async def`/coroutines and `await` distinguish creation of a coroutine object from execution/observation via awaiting; scheduling a Task is yet another operation. [Python asyncio tasks/coroutines reference](https://docs.python.org/3/library/asyncio-task.html).
5. **Lua 5.4** — dynamic coroutines yield explicitly, including through nested Lua calls; C boundaries require continuation-aware APIs or prohibit crossing them. It illustrates why transitive runtime behavior and host boundaries defeat a blanket lexical-function proof. [Lua 5.4 manual §§2.6, 4.5](https://www.lua.org/manual/5.4/manual.html).

**Precedent conclusion:** declarations and syntax delimit permitted suspension in several languages, but those mechanisms are neither necessary nor automatically appropriate for Protos's existing runtime-visible suspension boundaries. No peer's implementation is itself Protos normative authority. Adding a public effect discipline only to support a compiler optimization would expand the language without solving dynamic-target/guard correctness.

## Candidate comparison and GITHUB010 decision axes

**A — selected:** no new semantic contract; implement scoped sound compiler facts and guarded emission under PLAT061-B/I107.

**B — rejected as unnecessary:** add a clarifying normative definition of transitive suspension/observation even though existing clauses already distinguish the relevant operations.

**C — rejected:** a user-visible `suspend`/`async` modifier and effect-type category with static or dynamic enforcement.

**D — rejected:** public callable suspension-capability/guard contract, requiring user-visible authority and rules for overridden dynamic calls, host/foreign values and preservation across mutation.

**E — rejected for the current goal:** defer the proof capability; keep conservative continuations forever in this workstream. It is semantics-safe but fails the owner's expressly requested general implementation.

Scores are a **comparative qualitative assessment** (1 = poor, 5 = strong). `H/M/L` indicates high/medium/low confidence, **not** a test outcome. Each cell supplies its rationale. The scores are decision-support, not a substitute for normative safety gates.

| Required axis | A | B | C | D | E |
| --- | --- | --- | --- | --- | --- |
| 1 Correctness / invariant preservation | **5H** no semantic change, proof only | **5M** may restate existing rules faithfully | **2M** enforcement may constrain old calls | **3L** public capability must track dynamic overrides | **5H** general fallback safe |
| 2 Protos alignment | **5H** mechanisms remain internal | **4H** minor extra normative text | **1H** privileged effect callable kind | **2M** extra public capability category | **4H** no new entities but misses requested capability |
| 3 Pay for present need | **5H** analysis only where consumed | **4H** low documentation cost | **1H** pervasive syntax/type/tool cost | **2L** ongoing public bookkeeping | **5H** no immediate new cost |
| 4 Grow as you need | **5H** composable scoped facts | **4H** clarification permits growth | **3M** effects may extend future typing | **3L** expandable but public lock-in | **3M** growth blocked until work resumes |
| 5 Future options | **5H** syntax/backend independent | **5H** can preserve no-change semantics | **2M** commits public function contract | **3L** future implementations must support public flags | **4H** retains options while doing nothing |
| 6 Scalability | **5M** near-linear demand-driven proofs possible | **5M** implementation stays free | **3M** declarations scale but retrofitting costly | **3L** transitive invalidation/guards grow | **3M** universal generic overhead remains |
| 7 Conceptual simplicity | **5H** existing language model | **4H** extra restatement | **1H** new distinctions everywhere | **2M** new category plus mismatch rules | **5H** no new machinery |
| 8 Portability / implementation freedom | **5H** fact storage backend-private | **5H** normative explanation portable | **4M** portable syntax but demands support | **3L** implementation detail exposed | **5H** generic semantics remain portable |
| 9 Runtime / resource cost | **5M** only proven paths change topology | **5M** negligible direct effect | **3L** state-machine and metadata costs unknown | **3L** capability guards/metadata risk | **3M** retains unneeded generic machinery |
| 10 Failure / operability | **5M** UNKNOWN and contradiction fail closed | **5M** rules aid readers | **3M** misuse and migration failures | **3L** stale capability claims are hazardous | **4M** correct but opaque inefficiency |
| 11 Deferral / reversibility / migration | **5H** future effects remain possible | **5H** simple wording reversible | **1H** breaking existing call model | **2M** public contract hard to retract | **4H** no breaking change but implementation gap remains |
| 12 Evidence / implementation risk | **5M** supported by existing semantic/runtime boundaries | **4M** low-risk but no demonstrated need | **3M** peers exist, Protos migration unproved | **2L** public capability coherence unproved | **5H** known fallback works |

### Strongest falsifying objections and residual unknowns

- **Objection to A:** the existing clauses do not spell a single universal no-suspension theorem. **Answer:** this is neither required nor sufficient; the compiler must prove each separate subject using existing effects and dynamically admissible behavior. Never promote UNKNOWN to false. A genuine uncovered observable semantic choice would require a new owner decision rather than fabrication in I107.
- **Objection to A:** demanding complete-invocation proof may leave many call sites UNKNOWN, yielding little instruction-count reduction. **Answer:** correct. This is not a mandate to optimize unsafe dynamic dispatch; I107 must demonstrate at least one physically removable path, or report the limitation without pretending proof.
- **Objection to B:** minimal clarification could improve readability. **Disposition:** non-normative decision evidence can explain existing clauses now without creating a normative amendment. A later normative contradiction would require its own gate.
- **Objection to C:** explicit effects may unlock aggressive optimization. **Disposition:** costs and source compatibility are unjustified for the motivating requirement; it would not eliminate mutated/foreign dynamic-target proof obligations.
- **Objection to D:** public capability flags could help tools. **Disposition:** no approved public semantic meaning or mutation invalidation discipline; do not make compiler metadata visible.
- **Objection to E:** doing nothing is safe. **Disposition:** fails the owner's requirement that lowerer consume and contribute general compositional facts.
- **Residual unknowns:** the exact set of physically eradicable prepared-call continuations is an I107 source-level correctness result, not something D204 may assume; code-size, performance, cold-start and compiled-graph benefit remain unproven and belong to PERF042. No guarantee of success is recorded.

## GITHUB021, authority and closure implications

A changes **no** guest syntax, lexical binding, call, default, error, cleanup, ReturnHome, host, Task/Actor/Process or concurrency semantics. It preserves D013, D179, D202/D203, PLAT014, PLAT057, PLAT058, PLAT059, PLAT060-B5 and ratified PLAT061-B. It delegates bounded implementation to I107 and independent performance acceptance to PERF042.

**DECISION_INVARIANT_CONSISTENCY=PASS** based on those unchanged invariant owners. **DECISION_APPROVAL_PROVENANCE=PASS** from the exact explicit owner choice already recorded at D204/#905. This record by itself does not close the GitHub Issue, validate source, or claim implementation.

`PROJECT_RECORD_REVISION` must be the actual published project-docs commit containing this file, verified by readback before GITHUB020 closure.
