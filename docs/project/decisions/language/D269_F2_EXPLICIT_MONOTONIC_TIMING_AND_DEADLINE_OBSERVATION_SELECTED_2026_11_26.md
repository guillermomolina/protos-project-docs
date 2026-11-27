# D269-F2 — Explicit monotonic timing and deadline observation

Date: 2026-11-26. Owner-selected recommendation; final ratification and normative publication remain pending. This non-normative record does not amend the product specification.

- Design: [D269 #1197](https://github.com/guillermomolina/protos/issues/1197).
- Independent implementation: [I181 #1200](https://github.com/guillermomolina/protos/issues/1200).
- Independent transport-abort investigation: [D270 #1201](https://github.com/guillermomolina/protos/issues/1201).
- Motivation: [WEB016 #12](https://github.com/guillermomolina/protos-website/issues/12).
- PROTOS_REVISION: [1afc4ceeb4b6df9878fb2d2f168235277f8e7e70](https://github.com/guillermomolina/protos/tree/1afc4ceeb4b6df9878fb2d2f168235277f8e7e70).
- [Comparative research, invariant and approval evidence](../../evidence/D269/D269_F2_GITHUB010_GITHUB021_OWNER_APPROVAL_2026_11_26.md).

## Approval provenance

In the active D269 investigation on 2026-11-26, the maintainer explicitly responded to the complete presented F2 recommendation: “ok apruebo la recomendación.” This is owner selection of the presented F2 design, not approval of any unpresented boundary rule or of D270. The same message authorizes direct publication of these records to this repository and necessary Issue allocation. No independent product repository publication is authorized.

## Selected mechanism

Timing is an independent, authentic, explicitly host-provisioned capability. The proposed private initial RootActor slot is `timing`, CLI opt-in `--allow-timing`, embedding option `protos.AllowTiming`. Absent without authorization. Import, Process, Network and D253-C1 Clock grant no Timing authority. No ambient clock, new syntax, primitive type, special public scheduler, implicit Task or automatic authority inheritance.

Ordinary message surface selected:

```text
timing.monotonicNanoseconds() -> Integer
timing.atNanoseconds(deadline) -> Future
timing.observeUntil(source, deadline) -> Future
```

The observation is synchronous, exact nondecreasing nonnegative Integer nanoseconds relative to a non-civil origin. Repeated samples are valid, nanoseconds are a unit rather than an accuracy guarantee, no signed-64 overflow or Float coercion. No portable guarantee that host suspend time is included. Each grant belongs to a Process-local temporal domain; derived Actor-local representations share that domain. A plain Integer is data, interpreted in the supplied capability's coordinates: provenance is not encoded or automatically detectable. No global distributed deadline or public Deadline/Duration type. Source-level code composes relative delays and policies.

One-shot timer invocation produces a fresh ordinary non-task-backed Future resolving to null no earlier than the deadline. Elapsed deadlines may resolve immediately; invalid semantic arguments fail without temporal registration. Cancellation removes or invalidates registration; no late resurrection. Admission/retention must be bounded and provider failures explicit. No hard real-time progress guarantee.

Observation invocation returns a fresh ordinary non-task-backed Future, with no transfer of source ownership, no source cancellation or resource release. Validate source and local domain before registration. The strict F2 ordering is:
- admission at or beyond D: DeadlineExceeded, even if source is already terminal;
- admission before D and source already terminal: mirror its outcome;
- admitted pending source publishes its terminal outcome before D: mirror that outcome; at or after D: DeadlineExceeded;
- observation cancellation winning before D: cancelled observation only; provider failure before decision uses the corresponding explicit failure.

The temporal comparison is at the semantic publication frontier, not guest continuation execution, kernel packet arrival or delayed callback. Equality favors expiry. Preserve the resolved value and recorded Error identities, or mirror cancelled state under existing Future rules. DeadlineExceeded is a proposed standard Error prototype directly beneath Error; each expired observation stores its own fresh occurrence. Four existing Future states remain sufficient. Ordinary consumer cancellation at resume still wins for that consumer without altering either observed Future.

Timing is explicitly delegable only in same-Process Actor.spawn initialization, following the narrowly provisioned local-capability pattern of D217. Destination representations are Actor-local, non-amplifying, with no shared mutable slots or transferred pending Futures. Atomic transfer validation; no ordinary send/request/reply transfer, cross-Process transfer or isolated P transfer. Worker termination retires local registrations, not other Workers' or RootActor's grants; Process/Context termination retires the entire grant domain. No implicit RootActor broker.

## Composition and boundaries

Ordinary Protos code derives sleep, relative timeouts, fixed-rate/fixed-delay repetition, backoff and shared absolute budgets. No automatic producer cancellation: an owner explicitly cancels and awaits mandatory aftermath, handles late acquisition and release, and preserves existing cleanup precedence. Borrowed resources are not implicitly closed. Pre-check before starting another stage does not make the producer admission atomically deadline-bound. Actor cooperative non-preemption and foreign callback restrictions remain unchanged. Internal terminal publication must work during authorized suspended initialization and host entries without running unrelated guest Tasks.

DNS/TCP/TLS/HTTP may share one budget across stages. HTTP Worker policy is explicit and opt-in; deadline/inactivity policy and any public HTTP extension require the applicable library contract. A timer does not guarantee completion of required cleanup. In particular a partially committed TCP write may remain pending under backpressure; ordinary close awaits committed aftermath. D270 investigates independent transport-abort semantics; F2 neither selects abort nor weakens close, TLS integrity, I/O commitment or custody. Do not remove the hostile-ingress protection caveat merely because Timing exists.

## Implementation constraints and outstanding gate

Reuse source-backed capability architecture, existing ordinary Future/Actor registration and targeted completion, and actual Truffle precedents (GraalPy System.nanoTime and lazily allocated ScheduledExecutorService). Private host events cannot execute guest callbacks on host threads. No parallel guest scheduler, timestamps on every ordinary Future or timing polls in normal bytecode. Maintain finite retained state; release timer and source registrations on all outcomes. Trusted deterministic fake time for tests; actual host observations at Native Image runtime, not build time.

The presented research expressly did not certify complete line-by-line normative coverage or experimental Native feasibility. Before final ratification/release of I181, complete the required whole-document read audit and reconcile exact frontiers for timer expiry versus cancellation/publication, pending registrations on host revocation, provider failure precedence, admission bounds/error categories, integer normalization and representability, and atomic registration/publication/retirement. Preserve the selected semantics; any new material semantic choice must return to the owner rather than be inferred. No tests or implementation have been run. Normative publication is separate and must precede or coherently accompany authorized implementation.

Proposed normative owner: spec/io/TIMING.md, with references in FUTURES_AND_TASKS, ACTORS, PARALLEL_EXECUTION, PROCESS_IO, MODULES, ERRORS, IO_CORE and CLOCK as applicable. Existing D253-C1 remains unchanged. No grammar change.

## D269-F2.1-B — Owner-approved boundary refinement (2026-11-27)

The maintainer explicitly approved the exact recommendation in the active interaction: “apruebo D269-F2.1-B.” See [revision-bound research and approval evidence](../../evidence/D269/D269_F2_1_B_ACTOR_TERMINATION_AND_OWNER_APPROVAL_2026_11_27.md). Current product research baseline: [52591e7ce5c7e068f98101db94e77e68a64320da](https://github.com/guillermomolina/protos/commit/52591e7ce5c7e068f98101db94e77e68a64320da), specification 0.1.518. The F2 selection above remains authoritative except for the explicit boundary refinement below.

At Actor termination, retire local Timing registrations and cancel still-pending temporal Futures, including an observation with internally selected but not yet terminally published outcome. This is a narrow exception to the ordinary no-rewrite-after-decision rule, not a rewrite of any already-terminal Future or of the observed source. Do not retain the Actor for Timing publication, create an additional Task, execute guest code on provider threads, or revoke other Actor-local grants. Reject direct external publication and preserve the existing committed lifecycle-close exception without extension. Process/Context retirement remains governed by existing host custody.

During normal Actor operation, decide against the original Future terminal publication frontier, not the later callback. At admission at or beyond D, expiry wins; before D mirror an already-terminal source. For pending sources, publication before D wins, equality and later publication lose. Ordinary observation cancellation wins only before the irreversible decision and before D. Active-Actor host revocation preserves previously decided outcomes, otherwise fails the remaining observation with IOLifecycleError unless expiry can already be established. Provider failure is IOError; invalid temporal argument or source outside the local domain InvalidIOArgument; registration capacity refusal IOCapacityExhausted. Validate semantic arguments before sampling. Nonnegative Integer deadlines exceeding signed long range are valid, with no truncation or Float conversion.

D270 and I184 are now closed. The published TCP abort operation is available to an authorized connection owner; it neither supplies Timing authority nor changes observation semantics. It initiates retirement without waiting for a peer to read outstanding output, preserves actual admitted effects and reconciliation, and does not guarantee a real-time release bound. The earlier D270 investigation reference above is historical. No implicit close/abort, change to TLS/HTTP ownership or hostile-ingress safety guarantee follows from F2.1-B.

The existing Future monitor and Actor targeted completion/termination machinery must be reused, with no mandatory clock reads/timestamps for ordinary Future transitions, no new continuation architecture, per-timer Task/thread or bytecode polling, and lazy host service. GraalPy and GraalJS concrete precedents are recorded in the accompanying evidence; every proposed production Java change still requires verified equivalent method, annotation and behavior in an existing Truffle language.

I181 remains the sole implementation work item, with proposed internal slices A–E (grant/sampling, timers, observation, Actor delegation/lifetime, integrated validation and overhead). D269 remains open for the outstanding whole-document audit, any remaining exact boundary reconciliation and final normative/ratification gate; I181 remains blocked. No D269 product change, test execution or Native feasibility is claimed.
