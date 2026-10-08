# I086 / PLAT054-3E1 — Published host-entry suspension and authority specification

**Date:** 2026-10-08  
**Issue:** [I086 / #840](https://github.com/guillermomolina/protos/issues/840)  
**Design:** [PLAT054 / #838](https://github.com/guillermomolina/protos/issues/838)  
**Product commit:** [`guillermomolina/protos@3bb1278d91ee5cea98031462be2a5c4dd3c89019`](https://github.com/guillermomolina/protos/commit/3bb1278d91ee5cea98031462be2a5c4dd3c89019)  
**Prior product commit:** `e0bb880584908f65cb78e66896eba334f3d5e13b` (I086-3D)  
**Normative spec:** **0.1.450** (prior: 0.1.449)  
**Scope:** specification-only publication; no runtime/benchmark changes.

## Publication evidence, source-grounded

Read-only verification of the live `protos/main` ref and exact GitHub commit confirmed product HEAD `3bb1278d91ee5cea98031462be2a5c4dd3c89019`, commit message **“I086 PLAT054-3E1: publish embedding host-entry suspension and authority contracts (spec 0.1.450)”** (2026-10-08 07:08:35 UTC). The commit changes exactly three paths:

1. `spec/concurrency/FUTURES_AND_TASKS.md`
2. `spec/io/PROCESS_IO.md`
3. `spec/PROTOS_SPEC_CHANGELOG.md`

`spec/PROTOS_SPEC_CHANGELOG.md` HEAD starts with revision `0.1.450`, recording HOST-FUT-1, HOST-FS-1, HOST-FS-2, HOST-NET-1 and HOST-NET-2, and preserving PLAT054-2 invariants.

**Human validation report:** the owner explicitly reports **all tests PASS locally**, `git diff --check` clean, commit and push completed. These are owner-reported results; there is no claim that this coordinator ran tests, saw complete test logs, proved Native Image, or independently reproduced the results.

## Normative contract published

### HOST-FUT-1

`FUTURES_AND_TASKS.md` §29 adds **“Suspendible host-initiated RootActor entries”**. A standard Java host entry on an extracted Closure can suspend at an explicit pending `Future.value()` without a mandatory per-call Task, RootTask, scheduler or rich activation. `Value.execute()` stays synchronous from the host's viewpoint; no reexecution or duplicated guest effects; waiting is under Process/Context lifetime and terminates without post-close guest execution; unsafe concurrent RootActor entry remains disallowed, while other Actors/P can progress. Fatal unhandled Error rules continue to apply.

The rule for a **synchronous foreign callback** is expressly distinct: the nested callback cannot suspend across the synchronous foreign operation's dynamic extent. This is not weakened.

### HOST-FS-1 and HOST-FS-2

`PROCESS_IO.md` now specifies that the initial bootstrap-local `filesystem` slot is eligible only when file-I/O authority is **effectively granted** by the Polyglot host after restrictions, including any custom file provider. If there is no effective grant, the slot is absent, not `null`. No direct NIO or Core-resource authority bypass is permitted.

The confined base is the **effective Context working directory inside the authorized provider's namespace**; `Path.relative()` denotes that base. Unsafe/unrepresentable bases fail closed; no ambient host root, mutable Process-wide CWD, or broader fallback.

**Explicitly unresolved design:** when the host authorizes file I/O but no safe base can be represented, specification 0.1.450 does **not** choose between aborting bootstrap and completing without the `filesystem` slot. It says portable programs may depend on neither. **Do not claim B011's complete objective unblock condition has been met or silently select the outcome in implementation.**

### HOST-NET-1 and HOST-NET-2

`PROCESS_IO.md` specifies that effective Polyglot socket authorization grants the initial bootstrap-local `network` capability; without the grant it is absent. The scope remains host-bounded standard Protos TCP (`connectTcp`, `listenTcp`) with no new endpoint/protocol/policy surface or automatic Actor/P transfer.

Guest thread-creation permission is independent of socket authorization. The backend cannot create unauthorized guest-executing workers; host-only machinery is Process/Context-owned, retires on termination, and unused Network requires no poller, worker, or network resources.

## Blocker reevaluation and next work

- **HOST-FUT-1:** **READY FOR RUNTIME IMPLEMENTATION**, normative contract published; current direct Java `Value.execute()` path still lacks Task-free pending Future support. Recommended next bounded slice **PLAT054-3E2** in `guillermomolina/protos`.
- **B011 Filesystem:** **BLOCKED — narrowly unresolved observable failure/absence consequence for an unsafe Context base**. Most semantics are now normative, and mechanical TruffleFile-compatible backend planning can proceed independently without deciding that consequence. Do not mark globally READY or release a host authority path that would require selecting the missing behavior.
- **B012 Network:** **READY FOR RUNTIME IMPLEMENTATION**, published grant/thread/lifetime semantics have met the normative unblock gate. A Context-compliant host backend remains to be built and tested; READY does not mean implemented.
- **I087 / #841:** independent host-provided application module resolver; unaffected.
- **PERF032 / #831:** comparative graph/latency parity remains unmeasured and separate.
- **Native Image:** final integration/portable gate is unverified.

This classification is based on re-reading the exact specification, not merely on the commit message. None of these runtime acceptance obligations are discharged by a specification-only revision.

## Implementation guidance and ownership

The following product elements were inspected read-only at the accepted HEAD for the next slice:

- `src/main/java/com/guillermomolina/protos/execution/ProtosHostExecutableClosure.java`: direct/indirect/generic host Closure execution; admitted Java argument boundaries; enters/exits the RootActor; preserves compact non-Task fast path.
- `src/main/java/com/guillermomolina/protos/execution/ProtosEmbeddedProcess.java`: `evaluate()` currently enters via `ProtosRootTaskExecution`; `enterRootActor()`, `exitRootActor()`, `outermostEntryFailed()` and close/cancellation coordinate host-turn lifetime.
- `src/main/java/com/guillermomolina/protos/runtime/ProtosFutureValue.java`: pending `observeValue()` rejects synchronous observation; `observeValueForContinuationForRuntime()` currently requires `activation.task()` for waiter registration and resume.
- `src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeTaskExecution.java`: preexisting C-prime suspension/continuation machinery for Task-backed entries; reuse semantic guards, avoid building a second guest evaluator or replay model.
- `spec/concurrency/FUTURES_AND_TASKS.md` §29, `spec/io/PROCESS_IO.md`, `spec/concurrency/ACTORS.md` §24C, and adjacent Error/Closure control contracts remain authoritative.

**Next slice:** PLAT054-3E2, **TYPE=IMPLEMENTATION**, **REPOSITORY=`guillermomolina/protos`**, human-executor workflow. Agent edits runtime and tests, human performs test/build/validation/commit/push; no direct product Git mutation. Preserve `pom.xml` and product `CHANGELOG.md` until validation is green and versioning is justified, then no tests after their finalization.

```text
ISSUE=I086/#840
SLICE=PLAT054-3E1
PUBLICATION=VERIFIED_ON_MAIN
PRODUCT_REVISION=3bb1278d91ee5cea98031462be2a5c4dd3c89019
NORMATIVE_REVISION=0.1.450
FILES_CHANGED=spec/PROTOS_SPEC_CHANGELOG.md;spec/concurrency/FUTURES_AND_TASKS.md;spec/io/PROCESS_IO.md
VALIDATION=HUMAN_REPORTED_TESTS_PASS_AND_DIFF_CHECK_CLEAN
RUNTIME_IMPLEMENTED=NO
B011=BLOCKED_NARROW_FAILURE_ABSENCE_DECISION
B012=READY_FOR_IMPLEMENTATION
HOST_FUTURE=READY_FOR_IMPLEMENTATION
NATIVE_IMAGE=UNVERIFIED
NEXT_SLICE=PLAT054-3E2
WORKFLOW=HUMAN_EXECUTOR
```
