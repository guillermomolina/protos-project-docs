# D251-A1 — Owner-approval provenance, comparison and GITHUB021 invariant audit

**Date:** 2026-11-07. **Formal issue:** [D251 #1088](https://github.com/guillermomolina/protos/issues/1088). **Approved candidate:** A1 — explicit pending-Future suspension of Actor initial MODULE/SOURCE/bootstrap without an implicit Task or Actor reentrancy.

**Exact published source baseline:** [`guillermomolina/protos@08c27760eb3f85105277e5b1b1b159abbca67ebe`](https://github.com/guillermomolina/protos/commit/08c27760eb3f85105277e5b1b1b159abbca67ebe).
**Decision record:** [D251-A1 ratification](../../decisions/language/D251_ACTOR_INITIALIZATION_PENDING_FUTURE_SUSPENSION_A1_RATIFIED.md).
**Evidence type:** design research and explicit conversation approval; **not** a human GitHub approval comment, execution log, passed test, verified I156 FFM implementation, security proof, or a published Protos normative-spec change.

## 1. Failure and evidence pedigree

A human implementation report during I156 observed that RootActor service startup failed to reach READY when its initial SOURCE evaluated `filesystem.open(path, write: true, create: true).value()`. I156's FFM/Linux confined-worker driver completes the filesystem operation asynchronously, unlike the earlier synchronously completed NIO path. Its Future can be pending at `value()`, and the bootstrap turn runs without a Task. An existing Future continuation observer requires a Task except for expressly eligible host entries. The reported `fs-service.protos` and `exact-service.protos` cases fail after rebuilding the JAR; `fs-service-late.protos` succeeds because its Filesystem access occurs after READY. Earlier green test runs using an outdated JAR do not establish acceptance.

The owner provided a large uncommitted/staged local I156 patch as evidence, including modifications to the confined filesystem worker/driver, targeted Future completion, service integration tests and the reproducer. **Those modifications are not present in the published product SHA above and are not claimed as a public commit.** The published baseline was inspected for normative contracts and existing synchronous initialization control, task/host suspension distinction and Actor scheduler admissibility.

The failure is independent of Linux as a *language* issue: pending Futures can come from Network/Resolver, other Actors and other asynchronous producers.

## 2. Exact owner approval provenance

After the comparative D251 investigation and an explicitly labelled recommendation of **A1** — including “no implicit Task”, retaining `INITIALIZING` and exclusive initialization, allowing only ownership-safe internal terminalization, preserving MODULE/SOURCE caching and READY, and warning of possible same-Actor Task-producer deadlock — the project owner replied:

> **“Apruebo tu recomendación A1.”**

The owner then requested updates to issues in `guillermomolina/protos` and evidence in `guillermomolina/protos-project-docs`, with a **bounded exception permitting commit and push in the project-docs repository**. There is no approval here for automatically running pending tasks to break deadlock, adding implicit await, changing Filesystem confinement, changing process delegation, or changing foreign callbacks. This is an assistant-authored evidence record of explicit approval in the conversation, **not** a claim that the project owner authored a GitHub comment.

## 3. Governing source materials inspected (GITHUB010 baseline)

| Repository file | Evidence and resulting constraint |
| --- | --- |
| `AGENTS.md`; `AGENTS.work/DESIGN.md` | Canonical spec authority, explicit owner gate; GITHUB010 5+ systems/3+ approaches and 12 axes, GITHUB021 invariant/delta recheck |
| `AGENTS.work/COORDINATION.md`; `AGENTS.work/REFERENCE.md` | Formal D/I separation; native parent/dependency truth; published durable records and exact revision evidence; DOC002 role-first paths |
| `spec/AGENTS.md` | Cross-domain semantic consistency, adversarial counterexamples |
| `spec/concurrency/FUTURES_AND_TASKS.md` `29 | Pending `value()` explicitly suspends with lost-wakeup exclusion; taskless host-initiated RootActor entry is existing eligible precedent; no same-Actor task progress promised under exclusivity |
| `spec/concurrency/ACTORS.md` ``8–11, 24C/32 | Creation-before-READY, bounded message admission without dispatch during initialization, exclusive behavior installation, fatality and Process termination |
| `spec/semantics/MODULES.md` | D242 MODULE/SOURCE identity, nested import/cache-before-execute/INITIALIZING, immediate partial cyclic import result, failed cache eviction |
| `spec/semantics/EXECUTION_AND_CONTROL.md` `ensure` | Suspension is not scope unwind or cleanup; preserve original dynamic extent and no-replay |
| `spec/io/PROCESS_IO.md`; `spec/io/IO_CORE.md` | RootActor authority/Process lifetime, async I/O Future shape, cancellation/committed effects |
| `spec/PROTOS_SPEC_CHANGELOG.md` | Normative spec revision ledger; future incorporation must create its own revision |
| `docs/project/decisions/language/D225_B1_OPT_IN_ROOT_ACTOR_MESSAGE_SERVICE_BOOTSTRAP.md` | Existing opt-in RootActor service bootstrap selected by owner |
| `docs/project/decisions/language/D242_MODULE_SOURCE_PUBLIC_EXPORTS_AND_PACKAGE_MANIFEST_RATIFIED.md` | Source vs public-module object invariants and private initializer slots |
| I156 local unpublished patch | Worker thread completes host work; runtime-owned Future completion; service startup reproducer; no published D251 fix inferred |

## 4. Prior-art comparison — distinct mechanisms, not transplanted semantics

| System / primary reference | Relevant model | Transferable lesson / mismatch |
| --- | --- | --- |
| Erlang/OTP `gen_server` — [OTP documentation](https://www.erlang.org/doc/apps/stdlib/gen_server.html) | Synchronous `init` and optional continuation processing; server readiness is tied to init | Preserves “no user messages before ready”; strict synchronous init is poorly suited to explicit awaited asynchronous Futures in Protos |
| Pekko/Akka Typed — [Actor stash](https://pekko.apache.org/docs/pekko/current/typed/stash.html) | Actor can defer messages while initializing behavior/state and later unstash | Demonstrates bounded early-message handling; Protos already has admission/backpressure and does not need a public stash or deferred behavior protocol |
| Kotlin coroutines — [Coroutine basics](https://kotlinlang.org/docs/coroutines-basics.html) | Suspension captures coroutine continuation without blocking an OS thread | Explicit await and continuation preservation work; creating an implicit Task/coroutine for every Actor bootstrap would change Protos ownership and cost |
| Python `asyncio` — [Tasks](https://docs.python.org/3/library/asyncio-task.html) | `await` inside cooperative Task and event-loop scheduling | Clear explicit suspension with cancellation, but an Actor bootstrap should not silently become a Python-style Task or surrender Actor exclusivity |
| ECMAScript modules / top-level await — [ECMA-262 Modules](https://tc39.es/ecma262/#sec-modules) | Async module evaluation, dependency cycles and evaluation state | Explicit suspension can compose with module initialization; adopting JS-style dependency-await or TDZ would contradict Protos immediate partially initialized MODULE/SOURCE objects |
| Truffle — [Continuations and yield](https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/ContinuationResult.html) | Bytecode continuation/yield and re-entry APIs | Potential implementation mechanism; no normative commitment to this API, Bytecode DSL representation, carrier thread or host callback model |
| GraalPy/foreign callback practice | Runtime distinguishes task/coroutine context and calls crossing into guest code | Warns against allowing arbitrary foreign/native worker guest execution while completing a Future; guest semantics must be independent of the runtime mechanism |

The three principal design families are: **strictly synchronous bootstrap**, **explicit continuation suspension with exclusive init**, and **task-backed/staged/reentrant initialization**. “Force synchronous filesystem completion” was considered as a separate falsification baseline because it is an attractive but backend-dependent workaround.

## 5. Candidate set and common 12-axis GITHUB010 scores

Legend: score **1** poor → **5** strong; H/M/L = evidence confidence; each cell gives the concise reason for that score. Scores are comparative aids; semantic violations cannot be compensated by high totals.

| Dimension | A1: explicit exclusive suspension | B: prohibit pending init await | C: implicit Task/staged init | D: force synchronous completion |
| --- | --- | --- | --- | --- |
| 1. Correctness / invariants | **5 H** existing Future semantics and Actor isolation compose | **3 H** preserves READY but time-sensitive program failure | **2 M** implicit Task and reentry risk | **1 H** backend timing changes guest validity |
| 2. Protos alignment | **5 H** ordinary explicit `value()`, no new object | **3 H** adds exception to ordinary `value()` | **2 M** hidden ownership/institution | **1 H** filesystem-specific magic |
| 3. Pay for present need | **5 M** cost at actual wait | **2 H** requires app-level workaround | **1 M** machinery for all bootstraps | **2 H** native backend constraints/cost |
| 4. Incremental growth | **5 M** later schedulers optional | **2 M** future removal changes accepted startup | **3 M** grow within task-based design | **1 H** future async backends break assumption |
| 5. Future-option resilience | **5 M** portable continuation semantics | **2 M** blocks async composition | **3 M** couples to task representation | **1 H** closes async I/O options |
| 6. Scalability | **4 M** independent Actors; exclusivity can stall own tasks | **3 M** no wait, but more startup phases | **4 M** concurrent Task progress at complexity cost | **2 M** blocked I/O carriers |
| 7. Conceptual simplicity | **5 M** one explicit suspension rule | **3 H** special init-only prohibition | **2 M** phase/Task/behavior interactions | **2 H** special backend completion promise |
| 8. Portability / freedom | **5 M** continuation mechanism-agnostic | **4 H** simple runtime restriction | **3 M** implicit scheduler/task model | **1 H** host I/O completion dependent |
| 9. Runtime/resource cost | **4 M** waiter/continuation only when pending | **5 H** minimal runtime but app ceremony | **2 M** Task allocation and coordination | **2 M** thread blocking/forced sync |
| 10. Failure/operability | **4 M** deterministic fatality, admits same-Actor deadlock | **2 H** surprising timing-sensitive startup failures | **3 M** Task cancellation/reentry hard | **1 H** portability and liveness confusion |
| 11. Deferral/reversibility | **5 M** no public new protocol | **2 M** future migration of startup semantics | **2 M** removing implicit Task is breaking | **1 H** async integration requires redesign |
| 12. Evidence maturity/risk | **4 M** `29 HOST-FUT-1 precedent; scheduler audit needed | **5 H** trivial restriction, but not desirable | **3 M** coroutine precedents; integration unproven | **2 H** sync NIO historic, FFM invalidates it |

**Overengineering red flag:** C pays for speculative general Task scheduling, phase ownership or reentrant bootstrap state even when startup does not suspend. **Underengineering red flags:** B prevents ordinary asynchronous initializers and makes equivalent source sensitive to completion timing; D forces a host-specific mechanism and hides the general language gap. **A1 drawback:** an awaited Future produced by an excluded same-Actor Task can deadlock; this is explicitly accepted as a non-progress guarantee, **not** secretly solved by the implementation.

### Pay-for-what-you-need and deferral

- **Smallest sufficient today:** extend existing taskless, explicitly suspendible continuation to initialization while preserving exclusive Actor ownership; no public primitive, generic new Task kind or bootstrap phase.
- **If deferred:** the asynchronous FFM Filesystem service remains unable to reach READY; any later explicit async start requires reworking module execution, bootstrap dispatch and continuation state or inventing an app-level readiness protocol. Other Future-returning I/O sees the same mismatch.
- **Incremental growth:** new schedulers/continuation mechanics can be added behind unchanged semantics. A future willingness to reenter the same Actor to run Task producers would change observable isolation and needs its own decision.
- **Most plausible regret scenario:** a legitimate same-Actor Task dependency at initialization becomes common; escape path is a separately designed explicit startup dependency handoff or owner-approved concurrency model, not hidden reentrancy now.
- **Present costs:** only pending bootstrap waits need suspension objects; ordinary startup remains direct. Long-stalled initializers retain continuations and module graphs until terminalization or lifecycle termination, requiring leak-safe cleanup.

## 6. Falsification / adversarial acceptance matrix

| Case | Required observation |
| --- | --- |
| T01 already-terminal vs pending Future in initial SOURCE | Same guest outcome, no extra await; exact-once prior effects |
| T02 initial MODULE and private/public bindings | Suspended body resumes correctly; only expressly published own slots visible |
| T03 nested imports and circular initial import | Same cached partial instance returned immediately; `import()` never waits for READY |
| T04 bootstrap binding awaits a pending Future | Exact behavior installed once before one READY cutover |
| T05 terminalization race, many waiters | No lost wakeups, double resume or retained completed waiters |
| T06 `Actor.spawn` before ready, bounded `send`/`request` | ActorRef immediately available; no pre-READY handler execution |
| T07 unrelated same-Actor Task or external message runnable | Remains excluded from guest dispatch until init finishes |
| T08 internal completion from Linux/host worker | Future may terminalize only through ownership-safe path; worker runs no guest code |
| T09 same-Actor Task is sole Future producer | No hidden reentry/progress promise; can remain suspended |
| T10 failed/cancelled observed Future, caught vs uncaught | Ordinary fresh Error observation; caught may recover; uncaught fatal Actor/RootActor |
| T11 Actor stop/Process termination/Context close during wait | Never resume guest, never READY; late completion and waiter cleanup safe |
| T12 `ensure`/structured child/committed I/O | No cleanup on mere suspension, no duplicate effects; existing unwind and committed-effect rules |
| T13 default RootActor host entry and synchronous foreign callback | Existing eligibility/restriction unchanged |
| T14 Native/JVM and non-Filesystem async producer | Same observable semantics, no dependence on prior synchronously terminal Future |
| T15 module failure/cache eviction + retry outside dead Actor | No cache poisoning or rollback; ordinary Actor failure still applies |

These are **required future regressions**, not executed tests. The I156 Linux T04/T05 confinement and T16 service tests remain independent security/lifecycle gates and are **not** discharged by this D decision.

## 7. Final GITHUB021 invariant/delta review

| Established rule | Preserved? | Delta |
| --- | --- | --- |
| Explicit `Future.value()` and atomic waiter terminality | **YES** | Allow initial bootstrap context as another eligible **non-Task** waiter |
| HOST-FUT-1 taskless RootActor host direct entry | **YES** | No change to host-entry identity or lifecycle |
| D225-B1 RootActor message-service opt-in | **YES** | Initial-source/bootstrap may explicitly suspend before READY |
| Actor creation/identity, no early message dispatch, stable behavior | **YES** | READY is deferred by explicit wait, never partially published |
| D242 SOURCE/MODULE separation, cached identity, partial cycles | **YES** | No implicit `import()` wait or new module representation |
| Process/Context termination, D112/D121 cleanup and `ensure` | **YES** | Init continuation becomes lifecycle-owned, not immortal |
| D248-B/PLAT072/PLAT073 security, File authority and I/O effects | **YES** | Async completion timing is not a reason to broaden grants or bypass confinement |
| Foreign synchronous callbacks | **YES** | Not made suspendible by bootstrap approval |

**DECISION_APPROVAL_PROVENANCE=PASS** (explicit current-conversation exact A1 approval); **DECISION_INVARIANT_CONSISTENCY=PASS** for the bounded approved A1 text. **IMPLEMENTATION_TESTS=NOT_RUN**; **NORMATIVE_PRODUCT_PUBLICATION=PENDING**; **I156_SECURITY_PROOF=NOT_CLAIMED**.

## 8. Relevant external primary sources

- [Erlang/OTP `gen_server`](https://www.erlang.org/doc/apps/stdlib/gen_server.html)
- [Apache Pekko typed actor stash](https://pekko.apache.org/docs/pekko/current/typed/stash.html)
- [Kotlin coroutines](https://kotlinlang.org/docs/coroutines-basics.html)
- [Python asyncio tasks](https://docs.python.org/3/library/asyncio-task.html)
- [ECMAScript module semantics](https://tc39.es/ecma262/#sec-modules)
- [GraalVM Truffle `ContinuationResult`](https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/ContinuationResult.html)

**Scope disclaimer:** comparisons describe design families, not a claim that external systems have identical Protos bootstrap, cancellation, module-cycle, or Actor exclusivity semantics.
