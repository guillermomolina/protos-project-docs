# PLAT054-3E1 — Exact owner approval of host suspension and capability grants

Date: **2026-10-08**  
Design: [PLAT054 / #838](https://github.com/guillermomolina/protos/issues/838)  
Implementation: [I086 / #840](https://github.com/guillermomolina/protos/issues/840)  
Adjacent independent child: [I087 / #841](https://github.com/guillermomolina/protos/issues/841)  
Status: **OWNER-APPROVED; DURABLE DECISION PUBLISHED HERE; NORMATIVE PRODUCT PUBLICATION PENDING**.

## Provenance and exact scope

In the active ChatGPT conversation immediately following the **I086 / PLAT054-3E0** investigation and its explicit five-decision approval table, the project owner stated: **“aprobadas las recomendaciones”** and instructed publication in `guillermomolina/protos-project-docs` and coordination of `guillermomolina/protos` Issues. This is the owner's approval of the five expressly presented recommendations below, **not** of unspecified additional API, fallback, namespace, implementation or semantic choices.

Read-only public product snapshot verified at approval publication preparation:
`guillermomolina/protos@e0bb880584908f65cb78e66896eba334f3d5e13b`, product `0.3.289-SNAPSHOT`, normative spec **0.1.449**. Existing implementation acceptance is human-reported local PASS for PLAT054-3D; no new test/build/Native Image result is claimed. Evidence derives from read-only source/spec/issue inspection, not from executing a program.

## Five expressly approved choices

### HOST-FUT-1 — Pending Future in a Java host entry

An extracted Protos Closure invoked with standard Java `Value.execute()` is a synchronous host-initiated RootActor entry **eligible for explicit semantic suspension**, including `Future.value()` when the Future is pending. Java `Value.execute()` remains synchronous and returns the terminal value or propagates the terminal failure. The implementation may preserve/schedule a resumable guest continuation internally, but must not capture/re-execute previous guest effects, invent a new guest Future API, unconditionally materialize a `RootTask`/`ProtosTask`, or add scheduler/activation overhead to trivial non-suspending calls. Suspension is owned by the enclosing embedded Context/Process lifecycle: cancelling/closing prevents guest execution after the terminal boundary. Ordinary Future monitor/waiter atomicity, cancellation precedence, RootActor exclusivity and fatal unhandled Error remain governing contracts. Do not confuse an initial Java host entry with a *synchronous foreign callback* nested inside guest execution, where the existing rule restricting cross-foreign-stack suspension still applies.

The existing path divergence is concrete: `ProtosEmbeddedProcess.evaluate()` invokes `ProtosRootTaskExecution.execute()` with a Task-backed C-prime continuation, while `ProtosHostExecutableClosure.Execute` normally invokes compact direct/generic Closure paths with no Task. Pending observation in `ProtosFutureValue.observeValueForContinuationForRuntime()` currently insists on `activation.task()`; the ordinary synchronous `observeValue()` path also rejects pending values. A new host-compatible suspension bridge is **implementation work**, not evidence that one already exists. No blanket `Future.get()`/thread-blocking workaround is approved.

### HOST-FS-1 — Filesystem grant

A host's **effective Polyglot file-I/O authorization** (including an effective `IOAccess` grant, whether configured as `IOAccess.ALL`, file access, or a host-provided `FileSystem`) may grant the initial bootstrap-local Protos `filesystem` capability. Broad `allowAllAccess(true)` grants this capability only insofar as file-I/O authorization is **effectively enabled after applicable restrictions/overrides**. Absence of effective file-I/O authorization leaves `filesystem` **absent**, never `null`. Granting Protos authority must never bypass the Context's configured file provider or enlarge the host grant. Core internal packaged-resource access does not confer guest Filesystem authority.

### HOST-FS-2 — Filesystem base

The initial Filesystem uses the **effective Context working directory**, interpreted within the host-authorized provider/namespace, as its confined base. `Path.relative()` denotes that base; no implicit Process-global CWD, host physical root, arbitrary traversal or public `protos.FilesystemRoot` option is approved. If the provider cannot represent or confine that base, provisioning **fails closed**: do not expose a broader capability or silently fall back to unrestricted NIO.

**Deliberately unselected subdecision:** the approved recommendation did not select whether inability to provision a safe base aborts the first embedding bootstrap or instead leaves the bootstrap without a `filesystem` slot. That observable failure/absence policy must not be guessed. The normative-publication author should express only the decided fail-closed guarantee; if an exact choice is indispensable to write a complete unambiguous rule, return this narrowly scoped choice to the owner before publishing that part.

### HOST-NET-1 — Network grant and scope

The Context's **effective Polyglot host-socket authorization** is an explicit host grant for the initial bootstrap-local Protos `network` capability, subject to applicable restrictions/overrides; without socket authorization, that slot remains absent. `allowAllAccess(true)` is interpreted through effective permissions, not as a separately invented network privilege. The provisioned capability exposes only the already-standardized **TCP** acquisition surface and remains bounded by host-authorized socket authority. No new public `NetworkPolicy`, CIDR/port API, endpoint/protocol option, DNS, general UDP or externally exposed poller API is authorized. Existing Protos authority/non-transfer rules remain in force.

### HOST-NET-2 — Network and thread permissions remain independent

Network grant does **not** intrinsically require `allowCreateThread(true)`. Socket access and guest-thread creation are distinct host policies. The implementation must preserve both: when the embedding Context does not authorize guest threads, it cannot silently create unauthorized guest workers merely to service Network. A backend capable of servicing the approved socket authority within the effective Context restrictions is required. The current `ProtosNioHostIoPoller` uses a raw Java `new Thread(...)`, so it cannot be inserted into the embedded Process without an explicit lifecycle/permission audit. A host-only poller thread does not by itself prove a guest thread-permission violation, but it still requires correct Process custody, close, cleanup, and no guest callback after termination.

## Approved implementation constraints, not extra public choices

- **FS backend:** a `TruffleFile`-respecting or proven-equivalent confined adapter is necessary for custom host file providers. The current direct `java.nio.file` backends may not bypass a virtual/restricted host filesystem. Preserve `FILESYSTEM.md` atomic open/replace/remove, link/race, cancellation and resource-custody guarantees; fail unsupported operations safely.
- **NET backend:** lazy initialization, explicit Process/Context custody of pollers/sockets/listeners, Context-compatible thread policy and guaranteed shutdown. No worker/poller/provider/session is created on the no-Network path. NET-4 lifecycle obligations follow previously ratified PLAT054 and do not select a new public API.
- **Future:** preserve normal Error/Cancelled identities, lost-wakeup exclusion, cancellation-first and terminal-before-resume ordering; prove races and context close, never demand RootTask for `return 1`.
- No change to I087 module resolution/Actor.spawn identities or to PERF032/PERF033 benchmark interpretation.
- Native Image validation remains pending, not inferred from JVM tests.
- Product work stays in the user-operated `guillermomolina/protos` checkout (human executor for builds/tests/version/commit/push); direct agent publication here is the specific owner-authorized docs exception.

## GITHUB021 — exact earlier-invariant reconciliation

| Earlier PLAT054 invariant | Impact of approved 3E1 choices |
| --- | --- |
| One lazy Process per Context | Preserved; no new Process per host call |
| Core location fallback/override | Preserved; packaged Core never confers filesystem authority |
| Env-controlled I/O and restricted authority; Network denied by default | Preserved; effective **explicit host** grant is the new exact meaning of default slot eligibility |
| Persistent Process and ordinary module instance semantics | Preserved; initial bootstrap-local slot only, not auto-injected on later evals |
| Host-read-only bindings; guest mutation | Preserved |
| Reject unsafe simultaneous RootActor entry, no universal serialization | Preserved; suspension needs explicit safe re-entry/ownership handling |
| Retained extracted Closure identity | Preserved across host suspension/resumption |
| PAY AS YOU GROW without universal RootTask/scheduler/activation | Preserved; true suspension may pay for continuation, trivial calls may not |

The approval does not repeal the prohibition on suspending across a synchronous *foreign callback*; only the initiating Java host-call boundary is covered. Existing Actor and P independence remains binding.

## Normative-publication gate and work decomposition

This record is **non-normative**. The effective public/guest-visible contract must be published by the human executor in `guillermomolina/protos/spec`, with a new global specification revision after the current HEAD's changelog entry, before implementing any widened guest authority or host suspension:

- `spec/concurrency/FUTURES_AND_TASKS.md`: host-origin, non-Task explicit suspension and lifecycle distinction from synchronous foreign callbacks.
- `spec/io/PROCESS_IO.md`: effective I/O and socket grants for the initial slot, Context working-directory base and Context/thread/lifecycle restrictions.
- `spec/io/FILESYSTEM.md` and `spec/io/NETWORK.md` only for genuinely necessary cross-references or host-configuration precision; avoid duplicating owner rules.
- `spec/PROTOS_SPEC_CHANGELOG.md`: single next global revision documenting exactly changed normative owners.

B011/B012 remain **BLOCKED — DESIGN APPROVED, NORMATIVE PUBLICATION PENDING**. Do not mark READY merely because the decision record was committed. After exact product specification publication and review, reassess readiness before backend slices; release gate includes human validation. The suggested grouped slices are 3E1 normative publication, 3E2 host Future bridge, 3E3 Filesystem projection/backend, 3E4 Network projection/backend; slice suffixes are cost-only tracking and do not require new formal Issues.

## Evidence and non-claims

Inspected public sources include `AGENTS.md`, `AGENTS.work/{DESIGN,COORDINATION,IMPLEMENTATION,REFERENCE}.md`, `spec/{concurrency/FUTURES_AND_TASKS.md,io/PROCESS_IO.md,io/FILESYSTEM.md,io/NETWORK.md}`, `ProtosHostExecutableClosure`, `ProtosEmbeddedProcess`, `ProtosFutureValue`, `ProtosRootTaskExecution`, `ProtosBytecodeTaskExecution`, `ProtosNioConfinedFilesystemBackend`, `ProtosNioNetworkHost`, `ProtosNioHostIoPoller` and relevant PLAT054/I086 records.

```text
ISSUE=I086/#840
DESIGN=PLAT054/#838
SLICE=PLAT054-3E1
OWNER_APPROVAL=EXACT_FIVE_CHOICES_APPROVED_2026-10-08
DECISION_INVARIANT_CONSISTENCY=PASS
PRODUCT_REVISION=e0bb880584908f65cb78e66896eba334f3d5e13b
NORMATIVE_SPEC_REVISION=0.1.449
NORMATIVE_3E1_PUBLICATION=PENDING
FILESYSTEM_BLOCKER=B011_BLOCKED_SPEC_GATE
NETWORK_BLOCKER=B012_BLOCKED_SPEC_GATE
GUEST_RUNTIME_MODIFIED=NO
HUMAN_TEST_EXECUTION=NONE_FOR_THIS_DECISION_RECORD
NATIVE_IMAGE=UNVERIFIED
```
