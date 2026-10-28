# D219 — Private runtime capabilities: ratification and decision evidence

**Status:** RATIFIED — Option B (bounded local reuse; defer generic infrastructure)  
**Owner approval:** Guillermo Adrián Molina explicitly approved the D219 research recommendation on 2026-10-28 in the active project-owner interaction: “ok apruebo la recomendación”.  
**Owning live issue:** [D219 #973](https://github.com/guillermomolina/protos/issues/973).  
**Exact product baseline:** [`guillermomolina/protos@4f45348c39f218fd18503ef52a40bd20b71b8cf7`](https://github.com/guillermomolina/protos/commit/4f45348c39f218fd18503ef52a40bd20b71b8cf7), `0.3.409-SNAPSHOT`.  
**Previous completion evidence:** [I126 HTTP Host / D121 completion](https://github.com/guillermomolina/protos-project-docs/blob/0a28d8e6404d5078b64ebce60d0a0c4fc46bf543/docs/project/evidence/I126/I126_HTTP_HOST_0_3_409_COMPLETION_2026_10_28.md).  
**Authority:** This is non-normative architecture/decision rationale. The canonical observable contracts remain in `spec/`; no normative amendment, runtime change, implementation validation, or new public API is asserted by this ratification.

## Exact approved decision

1. **Keep** `ProtosHttpAsyncFacility` a deferred, frozen, private, exact-module-local capability captured by the HTTP modules. Retain its existing Actor-local provisioning, without an actor-wide or Context-wide service registry, new Core API, or shared global capability.
2. **Remove one strictly duplicated operation in the next independently tracked Ixxx implementation:** replace `facility.invokable(handler)` at its two Protos call sites with `facility.provides(handler, "call")`, then remove Java registration and implementation of `invokable` and stale descriptions. Preserve the other **eight** operations and their contracts. This is an internal implementation only; it is not performed by this record.
3. **Do not attempt to reimplement irreducible primitives with guest sends.** Terminal non-Task Futures, exact host-representation tests, non-invoking lookup, effective closability that excludes inherited `Object.close`, and structured service Tasks retain their current private runtime implementation.
4. **Defer** a reusable runtime-capability layer or generic registry until a **second independently identified consumer** requires equivalent operations with matching authority, lifetime, isolation, failure, and cancellation contracts, or a concrete high cost of later extraction is established. A future candidate must pass the ordinary Dxxx/PLATxxx owner gate if it selects new durable architecture or observable semantics.
5. **Preserve pay-as-you-grow:** no new import-free allocations, initialization, Task or Actor state, synchronization, authority, scheduler dependency, or Cross-Context sharing. The existing `ProtosTask` has a fixed pre-start-cancellation flag from earlier work; D219 does not claim that this baseline field is zero bytes or change it.

**Non-goals:** public Future/Task/inspection primitives, broad Java-to-Protos migration, replacing the scheduler, refactoring HTTP request/response policy, I/O ownership/custody changes, speculative generic runtime services, and specification edits.

## Exact HEAD source audit and nine operations

The following evidence was examined at the exact baseline. The Java source is `src/main/java/com/guillermomolina/protos/execution/facilities/ProtosHttpAsyncFacility.java` (its Java package is `com.guillermomolina.protos.execution` despite the source-directory name). Standard-module registration is in `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosCoreBootstrap.java`; the runtime installs a fresh frozen facility only when each relevant Actor-local module instance initializes, not globally at Context start.

| Operation | Concrete service, users and retention rationale |
| --- | --- |
| `failed(error)` | Creates a fresh, already-failed non-Task Future in the caller's Actor; invalid HTTP admission must create no Task. Used by Server (1) and Session (3). The Java implementation currently checks `ProtosObjectValue`, not a stronger semantic Error-family predicate; this observed detail is **not** authorization to change failure semantics during the dedup slice. |
| `resolved(value)` | Creates a fresh, already-resolved non-Task Future; explicitly rejects Future values to avoid adoption. Used by Session (1). |
| `provides(value, selector)` | Checks whether delegated selector lookup ends at a Closure, without invoking candidate guest code. Used by Server (2), Session (1). **Retain; use with "call" in place of `invokable`.** |
| `invokable(value)` | Exact same `hasCallable(value, "call", activation)` predicate as `provides(value, "call")`. Used by Server (1) and HostWorker (1). **Only operation approved for removal.** |
| `closable(value)` | Read-only check for effective callable `close`, excluding inherited root `Object.close`. Used by Server (1); not equivalent to ordinary `provides(value, "close")`. |
| `ordinaryObject(value)` | Exact Java class identity `value.getClass() == ProtosObjectValue.class`, excluding specialized value/resource families and unforgeable by guest delegation. Used by Session (1), HostConfig (3), Limits (1). |
| `isBytes(value)` | Native `ProtosBytesValue` representation guard, not guest-protocol spoofing. Used by Session (1). |
| `startService(body)` | Creates the single structured service Task with a fresh associated Future and opt-in pre-start-cancellation entry. Used by Server (1) and HostLifecycle (1). |
| `enterService()` | Called as the service body enters its owning `ensure`; delivers pre-start cancellation as that scope's normal unwind without bypassing D121 authorized cleanup. Used by Session (1), HostLifecycle (1). |

`Host.protos` and `HostConnection.protos` do not directly call these facility operations. The Java facility has nine operations initially and will have eight only **after** implementation and tests; this ratification does not claim that change already happened.

Other facilities inspected to test actual reuse: `ProtosSealedFamilyFacility` already performs a comparable exact ordinary-object Java predicate, which supports possible **low-level Java helper** reuse someday, but it is not evidence that another module needs the same guest-visible nine-operation capability object. `ProtosLoggingFacility` checks LogEvent/Instant shapes under its own authority; `ProtosInspectFacility` is explicitly gated by `protos.Inspect`; `ProtosRuntimeMetadataFacility` has Context-specific data; `ProtosMathFacility` is another private native boundary. Their semantics do not justify a generic shared facility now.

## Semantic, security, and resource invariants

- **Future identity and adoption:** a terminal Future is fresh and Actor-local; the first terminal outcome wins. `failed` preserves the supplied failure object; `resolved` must not adopt another Future or turn an invalid admission into a newly scheduled Task.
- **Callability inspection:** `provides(x, "call")` and the removed `invokable(x)` currently use the identical non-invoking Java lookup. Merely sending `x.call`, `x.hasSlot`, or `x.slotValue` would not be equivalent: own-slot reflection differs from delegated selection and guest sends can be overridden or effectful.
- **Closable:** inherited `Object.close` is structurally present but does not confer the effective I/O `Closable` capability. Preserve the selected-slot home check.
- **Representation:** exact ordinary-object and Byte checks cannot be replaced by user-controlled tags, prototypes, or similar public reflection without semantic change.
- **Structured cleanup:** cancellation before the service Task's first instruction must still enter the owning `ensure`; post-termination close authority (D121) is limited to already-authorized cleanup in the applicable dynamic extent, not inherited by new Tasks/callbacks.
- **Isolation and reachability:** facility modules `std:http/Server`, `std:http/Host`, and `std:http/internal/HostWorker` are installed lazily per Actor-local module instance. The temporary `_httpAsyncFacility` module slot is removed after lexical capture; this is ordinary source-level privacy, **not** a promise that deliberately exported bound closures are universally revocable. Do not enlarge the exposure or introduce shared mutable Context/Actor state.
- **Pay-as-you-grow:** no HTTP import must mean no facility initialization; no invalid-admission Task; one service Task per admitted server/host service, not per HTTP body read. The selected local removal does not add background work or per-program infrastructure.

Normative owners reviewed: `spec/io/HTTP.md` §§1, 2, 6, 8–10, 15; `spec/concurrency/FUTURES_AND_TASKS.md` §§23–24, 26, 28, 24G; `spec/concurrency/ACTORS.md` including D121 §24A; `spec/semantics/CALLABLES.md`, `OBJECT_MODEL.md`, `MODULES.md`, `ERRORS.md`, `VALUES_AND_COLLECTIONS.md`; and `spec/io/IO_CORE.md`, `BYTE_IO.md`, `NETWORK.md`. D215, [D216 #964](https://github.com/guillermomolina/protos/issues/964), D217, and [I126 #967](https://github.com/guillermomolina/protos/issues/967) are invariants, not re-opened work.

## Comparative research, alternatives, and cost of delay

Examined multiple implementation families, not only Truffle:

1. **GraalPy (Truffle):** Java builtin nodes coexist with Python-level library implementation; sharing a host implementation helper does not require one universal guest object. [Primary repository](https://github.com/oracle/graalpython).
2. **GraalJS (Truffle):** runtime and builtin installation are organized around JavaScript realm/context state, showing why language-guest-visible capability exposure should not be conflated with shared internal host machinery. [Primary repository](https://github.com/oracle/graaljs).
3. **TruffleRuby (Truffle):** internal primitive management and language-level libraries are separated; generic primitives carry registration, lifecycle and native-image concerns. [Primary repository](https://github.com/oracle/truffleruby).
4. **Apple Pkl (Truffle):** VM Context and standard-module evaluation isolate context-specific capabilities; direct analogy is architectural, not a claim of identical Actor semantics. [Primary repository](https://github.com/apple/pkl).
5. **Espresso / Truffle Context API:** per-Context boundaries and resource ownership restrict cross-context sharing; an unconditional registry would add lifecycle and isolation decisions absent from HTTP's current bridge. [Truffle APIs](https://www.graalvm.org/truffle/javadoc/).
6. **CPython extension modules:** per-module interpreter state offers a counterexample to process-wide mutable extension singleton state; allocation can remain local to need. [Official module-state guidance](https://docs.python.org/3/c-api/module.html#module-state-and-lifecycle).
7. **Go `internal` packages:** source/implementation reuse can avoid inventing a new public capability surface. [Official module layout reference](https://go.dev/doc/modules/layout).
8. **Erlang/OTP NIF resources:** native resources demonstrate strict lifetime/ownership boundaries, rather than free general sharing. [Official NIF documentation](https://www.erlang.org/doc/system/nif.html).
9. **Rust Tokio `JoinSet`:** structured task sets are instructive for cancellation/lifetime but cannot replace Protos's exact pre-start cancellation and D121 cleanup contract. [Official API](https://docs.rs/tokio/latest/tokio/task/struct.JoinSet.html).

These precedents illustrate **at least three distinct approaches**: runtime-specific builtin/primitive frameworks, per-context or per-module state, and source/private-host-helper reuse. No precedent overrules Protos semantics. No actual second consumer requiring the same fully equivalent nine-operation guest bridge was found at baseline.

**A — retain all nine operations unchanged.** Simplest zero-delta option; retains one redundant surface and diverging-callability risk.

**B — approved: deduplicate one operation locally, keep eight private, defer generic architecture.** Smallest evidence-backed source change; preserves all established semantics and leaves a later internal extraction path.

**C — introduce stateless shared Java helpers under module-private façades.** Credible when two facilities demonstrably duplicate the exact algorithm and constraints; at this baseline the comparable predicates have non-identical surrounding policy and the complexity yields little current benefit.

**D — introduce a generic Actor-local/context-level runtime capability registry or service.** Speculative access, revocation, lifecycle, initialization, cancellation and portability complexity for a single real HTTP consumer; rejected now.

### GITHUB010 twelve-dimension decision matrix

Scores run **1 (weak) to 5 (strong)**. Confidence is `H` high, `M` medium, `L` low; each cell gives the reason. These are architectural judgments grounded in the exact baseline, not benchmarks.

| Dimension | A unchanged | B local (approved) | C Java helpers | D registry |
| --- | --- | --- | --- | --- |
| 1. Correctness/invariants | 5/H existing semantics | 5/H exact shared predicate | 4/M helper extraction risks policy conflation | 2/M authority/cleanup redesign risk |
| 2. Protos philosophy | 4/H minimal but duplicate | 5/H mechanism stays local | 4/M host-only reuse | 2/H new privileged runtime institution |
| 3. Present proportionality | 5/H no work | 5/H one actual duplicate | 3/H no demonstrated second consumer | 1/H speculative infrastructure |
| 4. Incremental growth | 4/M later refactor viable | 5/M preserves extraction path | 4/M prepares common code prematurely | 3/L generalization may entrench wrong API |
| 5. Future options | 4/M no public commitment | 5/H no public or VM commitment | 4/M couples helper shape | 2/M registry constrains host backends |
| 6. Scaling | 4/M module-local lazy facility | 4/M same topology | 4/M stateless helpers scale | 3/L registry adds contention/lifetime risks |
| 7. Conceptual simplicity | 4/H 9 private names | 5/H 8 private names | 3/M extra layer | 1/H extra registry semantics |
| 8. Portability | 4/M private host boundary only | 4/M unchanged boundary | 3/M Java-helper coupling | 2/M host-wide management coupling |
| 9. Runtime/resource cost | 5/M lazy baseline; tiny duplicate operation slot | 5/M one fewer closure/slot per facility | 4/L extra indirection or none if optimized | 2/L potential fixed state/lookup cost |
| 10. Failure/operability | 4/H known paths | 5/M fewer entry points, same failure | 3/M helper error semantics must be proven | 2/L wider failure/authority surface |
| 11. Deferral/reversibility | 4/M bounded later extraction | 5/H internal-only migration later | 4/M reversible internal helper | 2/M early architecture hard to retract |
| 12. Evidence/implementation risk | 5/H no patch | 5/H exact predicate audited | 3/M limited second-use evidence | 2/L no concrete consumer/cost model |

**Non-compensating overengineering gate:** D scores poorly on present proportionality and simplicity; hypothetical scaling scores cannot compensate. C is still viable later, but current semantic duplication is only one provable operation, not a justified new infrastructure layer.

### Adversarial and future-scenario checks

- An object may *declare* `call` in an inherited slot, not in its own slots. Guest reflection replacing host lookup could falsely reject it; a shadowing non-Closure must reject without invoking anything.
- A value inheriting `Object.close` must not be accepted as an owned duplex solely for that reason.
- A Future supplied as `resolved` argument must not be adopted. A cancellation requested before a service starts must still run its exact owning `ensure`, including D121-bounded close.
- Two Actors import the same HTTP module independently: neither may share a facility, Task or runtime inspection authority through a new global registry.
- Context count, Actor count, idle programs, Native Image, alternate scheduler/backend, and a future independent module all make fixed facility and registry costs material. B adds no new singleton, lifecycle protocol, allocation or synchronization.
- If a second consumer appears, determine its **exact** required operation and privacy/authority semantics first. At worst B requires a bounded internal refactor of Java helpers and tests, with no published API to migrate, no persisted data, and no changed wire format. Thus the concrete **cost of deferral is low** at this baseline; this must be re-evaluated if evidence changes.

**Strongest objection to B:** duplication of host-only predicates across module-specific facilities can gradually diverge, and some existing low-level predicates already resemble each other. Response: the exact semantics and authority differ (`closable` vs general selector, sealed-family type validation vs HTTP admission). The first real cross-module equality should justify a narrowly scoped stateless Java helper while keeping private guest façades, **without** retroactively creating a generic Actor-wide service today.

## Implementation handoff and evidence limits

**One bounded implementation Issue/slice** is appropriate: two guest call sites and one Java facility (plus focused regression tests only if missing; stale comments/Javadocs in affected file). `I128 #972` remains **closed as not planned**, a historical scope-correction record with no code implemented; do not reopen or reuse its identifier.

**Implementation agent:** inspect the actual current `protos` HEAD and path instructions, inspect all patch-owned files, then edit only the required source/tests; do **not** compile, test, run a program, change Git state, commit/push, or pre-edit `pom.xml`/`CHANGELOG.md`. Preserve licensing and unrelated changes. No external research is needed.

**Human executor:** run the cheap diff/static gates, focused HTTP and Actor/NIO regressions, then the required `make test` and `make check` for executable top-level closure. After all tests are green, and **immediately before publication**, request the exact current version/changelog update from the agent on real HEAD; do **not run tests after editing `pom.xml` and `CHANGELOG.md`**. Human synchronizes with `origin/main`, stages explicit paths, and alone performs product commit/push. Only after the exact product SHA exists, publish Ixxx completion evidence in `guillermomolina/protos-project-docs` and close the implementation Issue.

**Validation on ratification date:** read-only repository/source and comparative documentation inspection only; **no D219 builds, test commands, benchmark or product changes executed**. Earlier I126 completion reports `make test` and `make check` PASS for I126, not for D219's unimplemented cleanup.

**Files materially inspected for the design:** `AGENTS.md`; `AGENTS.work/DESIGN.md`, `COORDINATION.md`, `REFERENCE.md`; `src/AGENTS.md`, `protos/AGENTS.md`, `spec/AGENTS.md`; the Java facility, Core bootstrap, Task/dynamic-cleanup/runtime facilities above; `protos/lib/http/{Server,Host}.protos` and its `internal/{Session,HostLifecycle,HostWorker,HostConfig,Limits,HostConnection}.protos`; normative owners enumerated above; guest HTTP regression suites and integration/architecture tests. The inspection establishes source-level decision evidence, not test execution or new performance measurements.
