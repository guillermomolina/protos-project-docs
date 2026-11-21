# I166 / D229-B1 — External OS execution normative specification publication (2026-11-21)

**Role:** revision-bound evidence of published normative integration; **not** evidence that external OS execution is implemented.  
**Implementation issue:** [I166 #1111](https://github.com/guillermomolina/protos/issues/1111) — open; normative prerequisite satisfied, implementation not yet delivered.  
**Approved design:** [D229 #1008](https://github.com/guillermomolina/protos/issues/1008), [ratified D229-B1 record](../../decisions/language/D229_B1_EXTERNAL_OS_EXECUTION_CAPABILITY_RATIFIED.md) at exact project-record commit `29343718dbc0f8b4e36abc343a8e65a831e6da1a`.  
**Exact published normative product commit:** [`guillermomolina/protos@38789e6f3a9bc43bfcf38f94f3200a2159daa51e`](https://github.com/guillermomolina/protos/commit/38789e6f3a9bc43bfcf38f94f3200a2159daa51e).  
**Published commit subject:** `D229-B1: Publish explicit external OS execution normative specification`.  
**Global specification revision:** `0.1.505`, dated 2026-11-21; `0.1.504` is the previous entry.

## Publication verified

The exact product commit and `spec/PROTOS_SPEC_CHANGELOG.md` at that exact SHA were independently read from GitHub. The commit publishes the new primary normative owner `spec/io/EXTERNAL_PROCESS.md` and integrates it with `AGENTS.md` and the eleven other specification/change-log owners (12 paths overall):

- `AGENTS.md`
- `spec/PROTOS_SPEC_CHANGELOG.md`
- `spec/io/EXTERNAL_PROCESS.md`, `spec/io/PROCESS_IO.md`, `spec/io/IO_CORE.md`, `spec/io/BYTE_IO.md`
- `spec/concurrency/FUTURES_AND_TASKS.md`, `spec/concurrency/ACTORS.md`, `spec/concurrency/PARALLEL_EXECUTION.md`, `spec/concurrency/DISTRIBUTED_RUNTIME.md`
- `spec/semantics/MODULES.md`, `spec/semantics/ERRORS.md`

This is a **normative-only** publication: it does not implement CLI or embedding options, provide Java or Protos runtime behavior, add executable examples, or add I166 feature tests. The fact that grant selectors now exist in the specification must not be mistaken for executable grants.

## Published contract and owner normalization

The independently and explicitly granted, default-off `execution` capability has host selection `--allow-execution` for standalone CLI and `protos.AllowExecution` for Polyglot embedding. Neither existing Protos authority nor generic host permissions imply this grant; a Protos Filesystem is not an OS-child sandbox.

The published direct execution surface is `execution.start(executable, arguments, environment, workingDirectory) -> Future<ChildProcess>` with **exactly four arguments**: native `String` executable, `Array` of `String` argv elements, complete explicit `Map` of `String` to `String` environment, and a native cwd `String | null` (`null` selects the stable grant default). No implicit shell, subprocess `Process` capability, PID access, environment inheritance, or extra overload is standardized.

The child exposes `stdin()`, `stdout()`, `stderr()`, `wait()` and `terminate()`. Byte endpoints have finite buffering and ordinary byte-I/O lifecycle; wait returns a fresh Future per observation, preserves the terminal result under Process custody, and never requires stdout/stderr EOF. `ExitStatus` exposes `exited()`, `exitCode()`, and `signaled()`: normal exit vs signal termination is classified **only from reliable native evidence**, never exit-code heuristics. Indeterminate classification fails the wait Future with `IOError`; a nonzero verified normal exit is a resolved result.

The irreversible start commitment frontier is the earliest point at which cancellation can no longer guarantee zero children. Uncertain native creation is a **custody condition**, not a third terminal outcome. Children are registered before guest publication and remain under Process/host custody through Actor death or Process cutover; host-only cleanup retains best-effort termination, endpoint release and reap/equivalent-release responsibility. Execution grants are explicitly delegable as non-amplifying Actor-local views; `ChildProcess` and its live endpoints cannot cross Actors or isolated P. Cross-Process and cross-Context authority is not provided.

## Validation provenance and limits

After pushing the normative commit, the maintainer explicitly reported:

- `git diff --check` clean;
- **all local tests pass**;
- the commit was pushed.

These are **maintainer-reported local results**. No named test command, test counts, CI run, Native Image execution or post-publication feature test transcript was supplied. The published normative commit introduces no I166 executable feature or I166 implementation tests. Thus `ALL_LOCAL_TESTS=PASS_OWNER_REPORTED` does **not** mean external execution has passed acceptance.

## Coordination after publication

- [I166 #1111](https://github.com/guillermomolina/protos/issues/1111): **open, normative blocker satisfied; ready for implementation**. Work must implement the now-published contract and satisfy its own tests, host security, Actor/P/Context boundaries, cleanup, JVM and Native acceptance. Respect Protos-first implementation and verified Truffle-language precedents for any Java production change.
- [D229 #1008](https://github.com/guillermomolina/protos/issues/1008): design already ratified and closed; no reopening or reapproval is needed.
- [LIB027 #1009](https://github.com/guillermomolina/protos/issues/1009): **still blocked** until the external-execution implementation has been published and verified; normative publication alone does not establish a working command/pipeline library.

The ratification record's historical `SPECIFICATION_PUBLISHED=NO` referred to the state at its original 2026-11-10 design publication; this later exact-revision evidence records the change in state without overwriting that historical source.

## Compact evidence

```text
WORK_ITEM=I166
DECISION=D229-B1
SPECIFICATION_PUBLISHED=YES
PROTOS_REVISION=38789e6f3a9bc43bfcf38f94f3200a2159daa51e
SPEC_CHANGELOG_REVISION=0.1.505
PRODUCT_COMMIT_VERIFIED=PASS
NORMATIVE_OWNER=spec/io/EXTERNAL_PROCESS.md
NORMATIVE_CHANGED_PATHS=12
IMPLEMENTATION_PUBLISHED=NO
I166_IMPLEMENTATION_TESTS=NOT_RUN
ALL_LOCAL_TESTS=PASS_OWNER_REPORTED
GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
CI_VERIFICATION=NOT_PROVIDED
NATIVE_EXECUTION=NOT_CLAIMED
I166_STATUS=READY_FOR_IMPLEMENTATION
LIB027_STATUS=BLOCKED_PENDING_IMPLEMENTATION
```
