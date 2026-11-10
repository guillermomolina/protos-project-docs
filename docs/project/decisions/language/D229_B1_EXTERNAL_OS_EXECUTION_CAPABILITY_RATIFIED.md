# D229-B1 — Explicit external OS execution capability and child lifecycle

**Decision state:** OWNER-APPROVED / DURABLY RECORDED (2026-11-10). **Normative specification integration and implementation: PENDING.** This design record is non-normative: current `spec/` remains the source of Protos language semantics until an independently published normative revision incorporates the decision.

- Owner decision: explicit **"Apruebo D229-B1"**, followed by **"Prepara la ratificación normativa ... dejando exactas las reglas de compromiso, custodia y limpieza"** (2026-11-10).
- Formal issue: [D229 #1008](https://github.com/guillermomolina/protos/issues/1008).
- Inspected product baseline: [`guillermomolina/protos@c9da7ff3e6c95f7a88948e875f61d67007fdeaa5`](https://github.com/guillermomolina/protos/commit/c9da7ff3e6c95f7a88948e875f61d67007fdeaa5) — **inspection identity only, not a D229 normative or implementation commit**.
- Prior scope split: [INC002 evidence](../../evidence/INC002/INC002_EXTERNAL_PROCESS_AND_COMMAND_LIBRARY_SPLIT_2026_10_31.md), [INC002 #1004](https://github.com/guillermomolina/protos/issues/1004), [LIB027 #1009](https://github.com/guillermomolina/protos/issues/1009).
- Related independent unselected decision: [D250 #1080](https://github.com/guillermomolina/protos/issues/1080), session-scoped grant and attenuation for **new Protos Processes**. D229 concerns **external operating-system children**; D250 is neither selected nor implicitly resolved.
- Governing procedures: `AGENTS.md`, `AGENTS.work/DESIGN.md` (GITHUB010/GITHUB021), `AGENTS.work/REFERENCE.md`, `AGENTS.work/COORDINATION.md` (GITHUB015/GITHUB020), `spec/AGENTS.md`.
- **Publication revision:** the exact commit containing this record is the `PROJECT_RECORD_REVISION`; it is not known within the source text before publication. Issue closure must use the actual returned commit SHA and re-read this exact file.

## Exact selected decision and limits

D229-B1 selects a small, optional, **host-provisioned external-execution capability**, with a bootstrap-local initial RootActor binding (conceptually `execution`) **absent when no grant exists**. It is independent of the Protos `Process` execution domain, `Filesystem`, `Network`, `Resolver`, `Entropy`, `Clock`, modules/imports, foreign providers and `Actor.spawn`. Possessing those values grants no right to launch OS children. No default grant, ambient `process.exec`, shell-command-string parsing, `sh -c`, generic POSIX interface, arbitrary-PID access or unguarded foreign bypass is selected. Host access is independently denied by default in standard CLI and embedding.

A grant may be **explicitly delegated between Actors** as a non-amplifying Actor-local proxy with shared logical authorization; `Actor.spawn` does not inherit it. No execution grant, live child handle or pipe capability crosses isolated parallel execution (P). Separate Protos Contexts/Processes receive independently authorized grants; a child created by one Context does not confer control to another. A grant may be narrower than the host's OS credentials. **Confinement of a Protos Filesystem is not confinement of an external OS child**. A host claiming OS-child isolation must prove it separately or refuse to grant execution.

The conceptual surface is `execution.start(executable, arguments, environment, workingDirectory) -> Future<ChildProcess>`, `child.stdin()/stdout()/stderr()`, `child.wait() -> Future<ExitStatus>`, and `child.terminate() -> Future`. Final selector, type and bootstrap-option spellings require normalization during specification integration; these conceptual names do not create an existing executable API.

### Launch request and validation

- `executable` is a single losslessly host-representable String naming an absolute native path admitted by the execution policy or a simple executable name resolved according to a **stable grant-specific host search policy**. Portable B1 does not admit relative path spellings containing separators. The grant's native path/search domain is *not* the Core `Path`/`Filesystem` namespace.
- `arguments` is an ordered finite vector of Strings. Passing it is direct executable invocation, without interpolation, expansion, shell quoting, command splitting or globbing.
- `environment` is a **complete explicit per-child environment**, captured as a finite mapping of native-representable String name/value pairs, with uniqueness under native environment-name equivalence. An empty map selects an empty environment; no implicit inheritance of Process bootstrap secrets or mutable host state. Applications may deliberately construct a complete child environment from authorized data.
- `workingDirectory` is native and grant-authorized; omission chooses a stable default specified by the grant, not a mutable Protos Process-global cwd. Nothing changes the immutable `process.environment()` or Core Filesystem base.
- Capture the complete invocation values before asynchronous backend execution can observe later caller mutations. Validate ordinary semantic invalidity (invalid types/shape, native NUL/representability, duplicate-equivalent environment names, invalid native option) before any attributable launch effect: successfully dispatched `start` returns a **fresh failed Future** with `InvalidIOArgument`. Actual denied/unresolvable/backend start failure uses `IOError`, barring a preexisting narrower normative category. Ordinary language lookup/arity errors remain ordinary language failures.
- Failure to start **does not** produce an exit code or a published `ChildProcess`. A successful start followed by nonzero exit is **not** an I/O failure.

### Exact start commitment and cancellation (D229-B1 invariant)

`COMMITTED` is an internal irreversible *operation frontier*, **not** a fifth Future state. It occurs at the **earliest instant at which the producer can no longer guarantee that cancelling the launch will produce zero OS children**. This may precede return from an OS/JVM process-creation call and precedes any guest-facing publication. The cancel/frontier race must have a consistent linearization/precedence.

- **Cancellation wins before the frontier:** the producer guarantees no child has been created and none will be created by that invocation; the fresh start Future becomes `cancelled`, with no later child, handle or launch effect.
- **Commitment wins:** cancellation cannot turn the start Future into a fictitious no-effect `cancelled` result. If a child exists, successful completion publishes one corresponding `ChildProcess` into a live originating Actor domain; the child is already under Process/host custody. If it is definitively proven that no child exists, the Future may fail with `IOError`.
- **Uncertain native creation:** the runtime must not publish a no-effect cancellation or a definitive failed launch while a possible OS child is unmanaged. It retains custody of the attempted launch, resolves whether creation occurred, and/or safely terminates/reaps a possible child before classifying the failure. The authoritative requirement is **never lose a possible child after the commitment frontier**.
- **Actor dies after commitment:** residual launch and resource cleanup remain under producer/Process custody. No guest code or callback executes inside a terminated Actor. A Process-termination cutover may prevent guest publication but does not erase the host effect or custody duty.
- `start` must not hide a potentially indefinite synchronous native launch wait inside an Actor turn. Its Future is an ordinary I/O-operation Future, not an implicit Protos Task.

### Child identity, wait and outcome

A successful start creates a distinct capability-backed child identity even when executable/arguments are identical or an OS PID is reused. A native PID is neither its semantic identity nor a transferable authorization token. `ExitStatus` is immutable and terminal; it distinguishes a normally exited child and native exit code from a host-observed signal termination when the backend genuinely supports that distinction. Do not infer a POSIX signal from a Windows numeric code. A nonzero exit is an ordinary **resolved** wait result, never an automatically failed wait.

Every distinct `wait()` invocation returns a **fresh** Future, and all successful waits for the same child observe its one stable terminal outcome. Cancelling a pending, reversible wait cancels only **that observation**, never the child or another waiter. Exit monitoring and native reaping remain under runtime custody; repeated waits cannot consume the one native exit status twice. Unknown/unrecoverable supervisory status fails observation with `IOError`, not a fabricated status. Observed process exit **does not imply** stdout/stderr EOF.

### Streams, ordering and bounded I/O

Each launched child has three separate standard byte endpoints, subject to their available/provisioned resource contracts: stdin `ByteWritable` + `Closable`; stdout and stderr each independently `ByteReadable` + `Closable`. They use the existing normative `IO_CORE.md`/`BYTE_IO.md` Future, error, commitment, closure, ordering, byte-sequence, capacity and cancellation contracts. Child stdin close prevents later writes and requests EOF to the child after preexisting committed-output aftermath; a closed child reader may produce broken-pipe write failure. A stdout/stderr read endpoint close releases that logical reader, **not** the child. Pipe EOF follows closure of relevant writers, not merely exit of the direct child; descendants retaining native writers may delay EOF.

No implicit UTF-8 transcoding, stdout/stderr merging, whole-output buffering, unbounded host queues, lost pre-read bytes, or automatic drain on wait is selected. A child blocked writing to an undrained pipe can legitimately prevent `wait()` completing. LIB027 may later build Protos-first concurrent draining/pipelines. Core B1 selects no general pipe-pair creation, pipe atomicity guarantee, or automatic inheritance of arbitrary Protos streams into OS children. On failed creation, every partially acquired endpoint remains in cleanup custody until released.

### Explicit termination versus wait cancellation

`child.terminate()` submits an **explicit best-effort host termination request** and returns a fresh operation Future; successful submission is not a promise of exit, instant force-kill, signal identity, successful shutdown of the descendant tree, or complete output drain. `wait()` remains the definitive exit observation. Failure to submit is `IOError`. Repeated requests do not restart the child. A request after already observed child exit is a harmless no-op success. Native signal/escalation/force semantics are host-adapter details until separately standardized. There is **no default portable `ChildProcess.close()` that kills the process**; pipe endpoints may be closed individually.

### Exact custody, Actor death and Process termination (D229-B1 invariant)

- **Custodian:** the Protos Process/runtime owns the native child, pipes and their supervision until completion, release or explicit independently authorized custody transfer. Actor-local guest objects hold delegated operation authority, not sole native cleanup responsibility. A child never becomes unmanaged merely because its last guest reference is lost.
- **Actor death:** triggers cancellation requests for its uncommitted pending I/O work under existing Future rules; it **does not kill an existing child**, undo committed start or invalidate separately delegated live-Actor views. If its start committed but its guest result can no longer be published, cleanup remains under Process/host custody, not a callback into the terminated Actor.
- **Process termination cutover:** immediately revokes all guest execution/child/pipe access from that Process, prohibits new launches, and transfers native children/endpoints still exclusively owned on its behalf to **host-only cleanup custody**. There is no synthetic successful guest close, flush, wait, EOF or task completion, and absolutely no guest callback after the terminal boundary.
- **Cleanup obligation:** the host must initiate supported **best-effort termination** for still-live exclusively owned children, safely close/revoke exclusively owned pipe handles, and retain tracking and native wait/reap/equivalent release duty until each known child is reaped or the backend establishes equivalent safe final release. Do **not** rely on guest reachability, garbage collection, terminated Actor Tasks, or Process reentry. Implementation/host aftermath may be asynchronous and native errors may be logged outside the dead guest domain.
- **Limits:** this is a positive **custody, revocation and attempt-to-reclaim** guarantee, **not** guaranteed termination time, successful kill, successful flush, descendant-tree cleanup, or a synchronous Process-termination wait. Independently provisioned host/other-Process principals are not destroyed merely because one Process loses its rights; initial B1 does not include a public detach/transfer-to-independent-custodian API.
- A backend unable to retain reliable per-child cleanup/reap custody must **not grant** B1. Multi-Context hosts cannot kill unrelated Contexts or global host services as a substitute for isolating one terminated Protos Process.

### Platform, runtime and deferrals

Linux/POSIX, macOS/BSD and Windows may implement creation, observation and termination differently. No POSIX signal/group/PTY/session/foreground-background-job semantics or Windows Job Object API is exported as portable Core. Java 25 `ProcessBuilder`/`ProcessHandle` is one possible implementation substrate, **not** semantic authority or sandbox proof. Host I/O completion workers never run ordinary guest Protos code; result publication occurs in the legitimate owning Actor domain. No child supervisors, native pipes or heavy synchronization are required for a Process without a granted/used capability (pay-as-you-grow).

Deferred: shell grammar/quoting, command-string `run`, glob, pipeline library, general pipes, raw native signals, PID enumeration, force-kill, process trees/groups, PTY/TTY, interactive jobs, daemon detach, cross-Process child authority transfer, distributed execution or OS sandbox promise. [LIB027 #1009](https://github.com/guillermomolina/protos/issues/1009) remains blocked until a separately verified implementation exists.

## GITHUB010 / GITHUB021 evidence and falsification

Research compared POSIX `posix_spawn/exec/waitpid`, Windows `CreateProcess`/Jobs, Java `ProcessBuilder/ProcessHandle`, Rust `std::process`/Tokio, Go `os/exec`, Python `subprocess/asyncio`, Node `child_process`. Candidates A (defer), B1 (explicit minimum), B+ (explicit sandbox policy), C (generic native privilege), D (ambient Process extension), E (foreign-only) were considered across the twelve GITHUB010 dimensions. B1 balances explicit authority, orthogonality, ordinary byte I/O/Futures, predictable custody and incremental growth. Strongest counterargument: even an explicit grant may allow a child to escape a Protos Filesystem sandbox; this is answered by truthful separate host authorization and fail-closed isolation promises, **not** by pretending a virtual Filesystem restricts the child.

GITHUB021 invariant/delta check: approved independence from Process/Filesystem and foreign bypass; no implicit shell; explicit grant/delegation/no P; snapshots; asynchronous start/wait; cancellation ≠ kill; Process custody; host cleanup; no global job-control/signals/PTY. The precise frontier, reaping obligation and cleanup limits above make those invariants implementable without changing them. No prior D250 design is preselected; its already-approved D248 shell-session constraint (reuse held Filesystem authority within a session without prompting per operation) remains intact, while new Protos Processes gain nothing automatically.

Adversarial implementation obligations: denied grants/import/foreign escape; Actor/P boundary; CLI/embedded separation; unrepresentable NUL/Unicode/casefold-equivalent env entries; mutable argv/env snapshot; absent executable vs exit=nonzero; start cancellation before/during/after irrevocable native creation; lost-launch-result Actor death; repeated wait and cancellation; natural exit vs termination race; undrained large stdout+stderr; EOF retained by descendants; broken pipe; failed-start leaked descriptor; Process/Context termination with live children; no guest-after-death or cross-Context kill; JVM/Native and operating-system-specific compatibility. **None of these tests has been run for D229.**

## Publication and next gate

This record publishes the **approved D229-B1 design**, not an executable implementation and not a global specification revision. Required next work: integrate a normative `spec/io/EXTERNAL_PROCESS.md`, adjust `spec/io/PROCESS_IO.md` §22/§28 and bootstrap, `IO_CORE.md`, `BYTE_IO.md`, Future/Actor/P/module references, and add the **next then-current** `spec/PROTOS_SPEC_CHANGELOG.md` revision in `guillermomolina/protos` under the human-executor workflow. That publication must be independently verified at its exact new `PROTOS_REVISION`. A separate `Ixxx` is for implementation, tests and CLI/embedding grants; do not start it from this record alone while normative integration is absent. This record does not approve an implementation strategy, undocumented platform claims or LIB027 library API.

```text
FORMAL_IDENTIFIER=D229
OWNER_APPROVAL=EXPLICIT_2026-11-10_B1
DECISION_SELECTED=B1
GITHUB021_INVARIANT_CHECK=PASS_FOR_DESIGN_RECORD
PROTOS_REVISION=c9da7ff3e6c95f7a88948e875f61d67007fdeaa5
PROTOS_REVISION_ROLE=INSPECTION_ONLY
SPECIFICATION_PUBLISHED=NO
IMPLEMENTATION_PUBLISHED=NO
TESTS_RUN=NO
PROJECT_RECORD_REVISION=THIS_RECORD_PUBLISHING_COMMIT
```
