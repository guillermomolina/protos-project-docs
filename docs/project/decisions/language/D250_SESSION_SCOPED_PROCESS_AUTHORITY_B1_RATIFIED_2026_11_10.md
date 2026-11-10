# D250 — B1: Host-derived, attenuated authority for new Protos Processes

**Status: RATIFIED — D250-B1 (owner approval, 2026-11-10).** This is a durable **non-normative** decision record about the semantics and hosting boundary of Protos execution domains. Observable semantics remain authoritative only when incorporated into the applicable `spec/` documents in `guillermomolina/protos`; this record is neither an implementing commit nor evidence of validated host-derived Filesystems.

- Decision owner: [D250 #1080](https://github.com/guillermomolina/protos/issues/1080).
- Separate implementation owner: [I165 #1110](https://github.com/guillermomolina/protos/issues/1110), **not yet implemented**. The native sub-issue association must be checked separately; the textual link alone does not certify it.
- [Exact B1 research, approval, counterexamples and GITHUB021 audit](../../evidence/D250/D250_B1_OWNER_APPROVAL_GITHUB010_GITHUB021_2026_11_10.md).
- **Product inspection baseline only:** [`c9da7ff3e6c95f7a88948e875f61d67007fdeaa5`](https://github.com/guillermomolina/protos/commit/c9da7ff3e6c95f7a88948e875f61d67007fdeaa5). No Protos product/specification change, validation or Native acceptance is attributed to that revision by this ratification.
- Prior approved shell-session invariant: [D248-B](../tooling/D248_EXPLICIT_CONFINED_FILESYSTEM_CLI_RATIFIED.md) and [D248 owner evidence](../../evidence/D248/D248_B_OWNER_APPROVAL_RESEARCH_AND_INVARIANT_AUDIT_2026_11_07.md).

## 1. Exact owner decision and reason

On **2026-11-10**, in direct response to the D250 comparative investigation and recommendation, the project owner stated **“Apruebo D250-B1”**, authorized updating the live Issues in `guillermomolina/protos`, and explicitly authorized documentation repository commit/push in `guillermomolina/protos-project-docs` for this task. Approval applies to **B1 as presented**: minimal host-only, non-amplifying derived provisioning for **new semantic Protos Processes**, with no immediate implementation and no guest launcher or public attenuation constructor.

D248-B already guarantees that a session's explicitly provisioned `Filesystem` can be retained and used for ordinary operations or functions inside the **same** Protos Process without reauthorization per `ls`/`cp` command. D250 does not reimplement or narrow that invariant. It answers what a future authorized host must preserve if it provisions a separate Process with derived authority, rather than making the parent guest execution itself a new authority source.

A Protos Process is a logical execution/isolation/failure domain, **not** an OS process. D229 separately owns external operating-system executable launch. No D250 choice makes D248's confined Filesystem an OS sandbox for native child executables.

## 2. Selected B1 authority model

1. **Authorizer:** The host owns effective authorization and must explicitly authorize every grant for a newly created Protos Process. A runtime session is a conceptual host-owned authorization/lifetime scope; B1 introduces **no privileged guest `Session` object**, global permission registry or Process manager.
2. **Non-amplification:** For a derivation requested in the scope of an existing guest-authorized session/initiator, the authority of the resulting grant is an effective subset of both. In abstract notation: `A_child ⊆ A_session ∩ A_requester`. This is an **effective resource/operation** relation, not a host-path string-prefix test. A host embedder may separately create an independently authorized Process with independent grants; that is a new host authorization, not guest-to-child derivation.
3. **New Process, new grant:** A recipient is a distinct semantic Protos Process with its own RootActor, Actor-local modules, Process capability, bootstrap data and lifecycle. A derived Filesystem is a **new independently custodied capability** over bounded authority, never a shallow copy or mutable alias of the originating guest wrapper, backend, context or worker. Contexts/Processes may share semantically invisible Engine/host machinery only when authority and mutable guest state remain isolated.
4. **Minimal optional Filesystem request:** the host may provision **no Filesystem**, a **fresh equivalent grant over the already-authorized root**, or a **fresh grant rooted at a safely selected, confined descendant** of that resource. These are host provisioning choices, not public guest selectors or new `Filesystem` methods. The granted capability exposes only the currently specified operations that its backend can faithfully provide; B1 does not select new operations or a public read/write policy vocabulary.
5. **RootActor placement:** an authorized grant is installed solely into the new RootActor's initial module context under `PROCESS_IO.md` / `MODULES.md`. Without authorization the `filesystem` slot is **absent**, not `null`. Imports, `Actor.spawn`, `ActorRef`, `process`, source loading, `package.toml`, bundled Tools and P do not acquire or inherit it.
6. **Orthogonal capabilities:** Network, Resolver, Entropy and Clock remain individually host-authorized under their existing specifications/decisions. Neither Filesystem nor Process/session/launch authority implies any of them. Likewise D247 service selection and D249's ratified independent CLI Resolver opt-in imply no derived Process grants.
7. **Confinement proof, not path reauthorization:** derive from the **selected, effective authority/resource identity**. A host must not concatenate/check string path prefixes and later reopen ambient paths, bypass a custom/restricted Polyglot `FileSystem` provider, or treat source/module resolution authority as guest filesystem authority. Symlink, mount, rename and concurrent ancestor/subtree substitution must not enlarge the grant. A platform/provider incapable of proving the required bounded derivation rejects provisioning **before the child executes guest code**; no partially reachable capability, unsafe fallback or silent grant widening.
8. **Independent lifecycle and revocation:** child Process termination revokes the child's grant/use and releases its custody without terminating or revoking independent peers. Termination of the requesting Process alone does not prove termination of a separate Process, and does not automatically imply global OS exit. Host session authorization revocation prevents further derivation and retires those dependent grants whose continuing authority exists **solely by that session**; independently authorized Process grants remain independent. Pending/committed I/O and host resource cleanup follow `IO_CORE.md` and `PROCESS_IO.md` `28. No late guest callback or authority resurrection is permitted.
9. **Pay as you grow:** no new worker, manager, scheduler, registry or guest allocation is required for executions that never create/request derived Processes. A private host adapter may be implemented only on demand and need not dictate one future API or OS mechanism.

## 3. Explicitly NOT selected

- No public `Process.spawn`, `ProcessLauncher`, `filesystem.restrict`, delegation constructor, guest provisioning service, Process management protocol, shell syntax, session CLI flag, automatic inheritance, Process-child supervision/communication API, or per-command prompt.
- No direct guest acquisition of host authority, and no new transferable live Filesystem category or implicit Actor/P proxy.
- No immediate read-only *live* filesystem derived mode, arbitrary rights/operation ACL DSL, multi-root/disjoint namespace model, mutable Process-global cwd, new File/Filesystem operation, or foreign-provider bypass.
- No automatic kill cascading from the requesting Process to other Processes, no distribution/lease/remote launch contract, no OS process sandbox claim, and no changes to D229.
- No mandatory implementation in the current D250 ratification transaction. [I165](https://github.com/guillermomolina/protos/issues/1110) independently owns future product/spec/test delivery; new public guest launch semantics require **another explicit owner design decision** if needed.

## 4. Integration with existing normative owners

- `spec/io/PROCESS_IO.md` ``22, 26, 28: host grants, bootstrap-local authority, Process custody and termination.
- `spec/io/FILESYSTEM.md` ``20–20.4: authority represented by Filesystem, Path without authority, rooted confinement and current operations.
- `spec/io/IO_CORE.md`: irreversible operation commitment, cancellation, close and lifecycle.
- `spec/concurrency/ACTORS.md` ``8–9, 16, 24, 25, 32: Actor creation, isolation, nontransferable resources, RootActor authority.
- `spec/concurrency/PARALLEL_EXECUTION.md` `71.11: isolated P cannot receive unapproved live authority.
- `spec/semantics/MODULES.md`: initial module versus import and per-Actor module ownership.
- `spec/concurrency/DISTRIBUTED_RUNTIME.md` ``37, 61, 65–70: Protos Process versus OS process, scoped Authority and no gratuitous coordination. B1's ordinary I/O-capability rights are **not** the exclusive distributed decision-Authority concept of ``66–70.
- Platform-specific implementation must preserve [PLAT072-E](../platform/PLAT072_LINUX_CONFINED_FILESYSTEM_BACKEND_RATIFIED.md) as amended by [PLAT073-C](../platform/PLAT073_JAVA_25_FFM_AMENDMENT_TO_PLAT072_E.md), not assume that the current Linux Java-25-FFM backend already implements derived grants.

## 5. Ratification audit and handoff

**GITHUB021:** The D248-B no-per-command-reauthorization invariant is fully preserved. B1 changes the proposed host-to-new-Process derived-provisioning model; it **does not** reopen or reduce the existing Process/Actor/module/IO or CLI grant invariants. No owner-approved invariant requires reinterpretation. All new consequences were part of the explicit B1 recommendation, not a hidden public API.

**Future implementation acceptance:** deterministic fail-closed same-root/descendant selection; symlink/mount/rename/TOCTOU protection; custom provider confinement; no cross-Context/cross-Process alias; nested derivation no amplification; session revocation versus independent grant survival; no import/Actor/P/Tool inheritance; independent Network/Resolver/Entropy/Clock; optional non-use overhead; JVM and real rebuilt installed Native Image verification when claiming support. [The research/audit record](../../evidence/D250/D250_B1_OWNER_APPROVAL_GITHUB010_GITHUB021_2026_11_10.md) lists specific falsification cases.

**Exact state at approval:** decision selected and ratifiable; product implementation **NOT STARTED**; no tests or builds executed for D250; Protos `spec/` **unchanged**. The formal decision may close while I165 remains open and D229/INC002 remain unselected.
