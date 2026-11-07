# D251 — A1: Explicit pending-Future suspension during Actor initialization

**Decision:** D251-A1, **owner approved 2026-11-07; durable ratification record**. This record is non-normative: observable language semantics are owned by the corresponding published `spec/` documents, not by this project record.

**Issue:** [D251 #1088](https://github.com/guillermomolina/protos/issues/1088).
**Trigger / consumer:** [I156 #1083](https://github.com/guillermomolina/protos/issues/1083), originating from the asynchronous Linux confined Filesystem backend; also applicable to ordinary `Actor.spawn`, MODULE/SOURCE and other Future-producing I/O.
**Comparative and invariant audit:** [D251-A1 research and owner-approval evidence](../../evidence/D251/D251_A1_OWNER_APPROVAL_GITHUB010_GITHUB021_2026_11_07.md).
**Exact product source inspection baseline:** [`08c27760eb3f85105277e5b1b1b159abbca67ebe`](https://github.com/guillermomolina/protos/commit/08c27760eb3f85105277e5b1b1b159abbca67ebe). This is **not** an implementation or normative-spec publication for D251. The motivating I156 code and tests were supplied as uncommitted/staged local diff; they are not attributed to that product revision.

## 1. Exact owner selection

The project owner expressly stated **“Apruebo tu recomendación A1”** on 2026-11-07 following the exhaustive D251 investigation, and authorized coordinated Issue updates and publication of durable project evidence in `guillermomolina/protos-project-docs`. This is approval **only of the A1 contract presented for D251**, not approval of an unexamined change to scheduling, Task ownership, the Future protocol, process provisioning, Actor reentrancy, Filesystem security, or a specific implementation mechanism.

### Selected normative contract (for incorporation into the canonical specifications)

> An Actor's initialization is eligible to suspend at an **explicit** `Future.value()` on a pending Future, including its initial MODULE or SOURCE body, nested imports and bootstrap binding. No Task is implicitly created. The Actor remains `INITIALIZING`, and its initialization continuation retains exclusive Actor-local execution authority until it either resumes and finishes or is terminated. Only internal terminalization/completion work which does not require unrelated guest Task/message execution may make progress toward settling the waited-on Future; suspension does not admit arbitrary same-Actor Tasks or external message dispatch. On successful resumption, initialization continues at the exact suspension point without replay, installing the behavior once and crossing to `READY` once. Existing module cache/identity/cycle, errors, cancellation, Process/Context termination, resource authority and cleanup semantics remain in force. No progress is guaranteed where the waited-on Future requires another, otherwise excluded, Task of that same Actor to execute.

This applies to both the opt-in message-service **RootActor** bootstrap and the ordinary destination-Actor bootstrap of `Actor.spawn`. Ordinary I/O operation calls still **return a Future without implicitly awaiting it**. Merely returning a Future from a module or bootstrap closure does not make the runtime await or flatten that Future.

## 2. Observable boundaries

1. **Explicit suspension only.** `value()` on an already-terminal Future observes its outcome immediately. `value()` on a pending Future suspends the **current initialization continuation** using the existing atomic waiter/terminalization guarantee. Ordinary `import()`, I/O dispatch, module resolution, returning a Future, and bootstrap invocation do not each add a hidden await.
2. **No hidden Task.** Bootstrapping a RootActor or newly spawned Actor is not converted into an ordinary Task, a synthetic RootTask, or a separate asynchronous guest function. A non-suspending initializer pays no semantic cost for possible suspension.
3. **Exclusive INITIALIZING.** Suspension retains the Actor's initialization ownership and its lifecycle state. No ordinary same-Actor Task, unrelated guest operation, or externally delivered mailbox turn may run during the suspended initialization merely to supply progress. Host/backend completion notifications can terminalize Futures through **existing ownership-safe internal paths** only; they do not grant native threads guest execution rights. Other Actors retain their ordinary independent progress.
4. **Waiter correctness.** Registration and first terminalization are atomic in the semantic sense of `FUTURES_AND_TASKS.md` `29. No lost wakeup, duplicate semantic resume, repeated earlier effects, skipped live waiter, or indefinite waiter retention after terminalization. A waiter becoming runnable is not a new global scheduling order.
5. **Spawn and READY remain distinct.** `Actor.spawn` returns its `ActorRef` after the existing creation/transfer cutover without waiting for module init, bootstrap, or READY. Accepted/preaccepted messages keep their present bounded backpressure and failure semantics. No guest message handler executes before the required behavior is installed and the Actor reaches READY.
6. **Stable behavior.** The bootstrap binding remains an exact own slot of the correct SOURCE context or public MODULE object; the result must still be a valid behavior object. That precise object is installed once, immediately before the unique `INITIALIZING -> READY` transition. There is no second behavior-setting mechanism.
7. **MODULE/SOURCE semantics unchanged.** A canonical Actor-local initial module is cache-inserted as `INITIALIZING` before body execution. A nested or cyclic import of that key returns the same partially initialized public MODULE object or SOURCE context **immediately**, not a Future and not after waiting for READY. Private initializer bindings stay private. The module's own successful/failing cache cutover remains separate from the Actor's READY cutover; a failed initialization evicts the cache entry and never rolls back committed effects.
8. **Failures and termination.** A failed or cancelled observed Future produces its ordinary `value()` Error observation. Handled Errors can continue initialization; an Error escaping bootstrap prevents READY and follows ordinary Actor/RootActor fatality. Actor stop, fatal failure, Process termination or Context close during suspension prevents any later continuation revival or late READY publication. The observed Future's outcome and committed I/O effects remain governed by their existing contracts; no automatic rollback or implicit resource close is introduced.
9. **Cleanup.** A suspension is not an `ensure` scope exit and cannot duplicate or prematurely execute cleanup. Ordinary handler/`ensure`/structured-scope rules still govern real unwind; introducing a taskless initialization wait is **not** an authorization to invent Task-cancellation semantics, late guest execution or lifecycle privileges.
10. **No same-Actor Task progress promise.** Waiting from exclusive initialization on a Future produced by an otherwise excluded Task in the **same Actor** may not make progress. The runtime does not solve that by running the Task inside the bootstrap, surrendering Actor exclusivity, spawning a helper Task or inventing a deadlock Error. Future enhancements may require a separate owner-approved design.
11. **No foreign callback expansion.** A synchronous callback into Protos in the dynamic extent of a foreign call does not acquire bootstrap suspension eligibility. Existing host-initiated RootActor entry rules (HOST-FUT-1), unsafe-concurrent-entry rules and RootActor failure semantics remain distinct and unchanged.
12. **Portable guest semantics.** No requirement to park an OS thread, use a specific Truffle continuation primitive, synchronously complete native I/O, or adopt one scheduler. JVM/Native, alternate backend, and non-Filesystem Future sources must be equivalent at the guest boundary.

## 3. GITHUB021: approved invariants carried forward

| Prior owner-approved contract | A1 consistency / explicit delta |
| --- | --- |
| HOST-FUT-1: explicitly suspendible host-initiated RootActor entries without implicit Task, exclusive Actor-local entry | **Preserved**; A1 extends eligibility to the previously undefined initialization turn, not to arbitrary foreign callbacks |
| ACTORS ``8–9: `spawn` returns before READY, no external message dispatch until READY | **Preserved**; READY can now occur after an explicit pending wait |
| ACTORS `11: exact behavior installed once, stable after READY | **Preserved**; no provisional behavior or post-READY replacement |
| D225-B1: opt-in RootActor message service, no implicit service Actor | **Preserved**; its existing bootstrap can suspend without changing its opt-in entry or identity |
| D242 MODULE/SOURCE: distinct instance/context identity, cache-before-execute, immediate cyclic partial visibility | **Preserved**; the same initializer may resume later but `import()` itself never waits for an initializing cached instance |
| FUTURES_AND_TASKS `29: one atomic wait/terminalization and ordinary `value()` Error/cancellation observation | **Preserved**; extends non-Task suspendibility to this lifecycle |
| PROCESS_IO/IO_CORE/D248-B: Process-scoped authority and I/O commit/cancel cutovers | **Preserved**; pending async completion does not grant authority, extend Process lifetime or force sync I/O |
| Foreign synchronous callback restrictions; `ensure` not triggered by suspension | **Preserved**, not implicitly broadened |

**Owner-approved invariant reopening:** none identified for the exact approved A1 formulation. **New scoped semantic delta:** eligibility of the initial Actor module/bootstrap continuation to suspend on explicit pending `value()` while preserving initialization exclusivity. The distinction between internal completion notifications and running arbitrary same-Actor guest work is an essential scope boundary, not an implementation loophole.

## 4. Canonical specification incorporation and implementation ownership

The approved choice has to be inserted consistently into:

- `spec/concurrency/FUTURES_AND_TASKS.md` `29: eligibility, atomic waiter, lifecycle of taskless initialization, exclusivity and same-Actor producer caveat;
- `spec/concurrency/ACTORS.md` ``8–9, bootstrap representation/`11/`32 and RootActor failure/termination owners: INITIALIZING, behavior, READY, no dispatch, termination;
- `spec/semantics/MODULES.md`: initial MODULE/SOURCE, nested imports, partial cached identities and failure semantics on suspension;
- `spec/semantics/EXECUTION_AND_CONTROL.md` `ensure` only to the extent that existing scope-exit distinctions require an explicit cross-reference;
- `spec/io/PROCESS_IO.md` and `spec/io/IO_CORE.md` only for genuinely needed cross-references; do **not** change I/O authority, grant lifetime or commitment rules; and
- `spec/PROTOS_SPEC_CHANGELOG.md` for the resulting published normative change.

An independently tracked **Ixxx implementation issue** must own the coordinated spec/runtime/test changes, including RootActor and ordinary Actor bootstrap, general Future completion, module loader, all applicable JVM and Native pathways, no-replay continuations and cancellation/termination. The decision itself is not evidence that those product files are already changed.

## 5. Non-goals and deliberately deferred questions

No `Future.await` addition, new Actor API, implicit automatic awaiting, new public Task kind, child Process/CLI authority, implicit Process grant, user-level deadlock detection, arbitrary same-Actor Task reentry, general foreign-callback suspension, scheduler requirement, IO implementation change to force synchrony, or guaranteed same-Actor task-producer progress.

Open problems requiring implementation validation rather than additional invented semantics: distinguishing internal terminalization from reentrant guest Task execution; continuation-safe entry/module bootstrap execution; late terminalization and cleanup under Process/Context closing; exact guest authority on host-completion threads; and Native Image correctness.

**Ratification state:** A1 owner-approved and recorded here; **product specification incorporation and runtime/test acceptance are separate future publications**. Never describe the `08c27760` baseline or the owner's unpublished I156 patch as implementing D251.
