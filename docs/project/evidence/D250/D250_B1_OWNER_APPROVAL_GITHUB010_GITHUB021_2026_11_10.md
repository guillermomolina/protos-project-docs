# D250-B1 — GITHUB010 research, source audit, owner approval and GITHUB021 consistency

**Recorded 2026-11-10.** Evidence supporting the explicit project-owner approval **“Apruebo D250-B1”**, provided after review of the exhaustive D250 recommendation. This document records a *decision*, not implemented behavior, tests, security certification or JVM/Native release evidence.

- Formal decision: [D250 #1080](https://github.com/guillermomolina/protos/issues/1080).
- [Ratified B1 contract](../../decisions/language/D250_SESSION_SCOPED_PROCESS_AUTHORITY_B1_RATIFIED_2026_11_10.md).
- Separate implementation tracking: [I165 #1110](https://github.com/guillermomolina/protos/issues/1110); not implemented or validated by this publication. Native parent/sub-issue state requires independent verification.
- **Inspected product revision:** [`c9da7ff3e6c95f7a88948e875f61d67007fdeaa5`](https://github.com/guillermomolina/protos/commit/c9da7ff3e6c95f7a88948e875f61d67007fdeaa5), dated 2026-11-10. This is a read-only inspection baseline, not a D250 product change.
- The earlier independent [D248-B source/evidence baseline](../D248/D248_B_OWNER_APPROVAL_RESEARCH_AND_INVARIANT_AUDIT_2026_11_07.md) was `3791394c46aff8ad7508f64e56cde4594e502a56` on 2026-11-07. Do not confuse that older baseline with the current one.
- Public CLI [D249-A1 Resolver](../../decisions/tooling/D249_EXPLICIT_RESOLVER_CLI_RATIFIED.md) was separately approved on 2026-11-10; [I164 #1109](https://github.com/guillermomolina/protos/issues/1109) is its independent implementation item. D249's previous status as unapproved in the historical D250 intake has been superseded.

## 1. Exact owner provenance and GITHUB021 protected history

On 2026-11-07, during D248-B, the owner asked whether a hypothetical interactive Protos shell would have to reauthorize every `cp`. The owner approved revised D248-B together with the invariant: **a session holds and reuses its host-granted Filesystem for ordinary in-Process operations without per-command permission requests**. It did **not** authorize a child Process protocol; D250 was allocated as an independent OPEN design.

On 2026-11-10, following the D250 comparison and precise B1 recommendation, the owner explicitly stated **“Apruebo D250-B1”** and separately authorized this task's live GitHub issue coordination and direct project-documentation commit/push. That **exact** selection endorses host-side non-amplifying derivation, independently custodied new Protos Process grant, no public guest launcher yet, and no immediate implementation. It does not approve a particular Java API spelling, host policy DSL, platform implementation, native subprocess grant, new filesystem method or distributed Process manager.

Decision/invariant preservation is therefore checked against both approved statements; the 2026-11-07 shell invariant is not broadened into child-process implicit inheritance, and the 2026-11-10 B1 choice does not narrow session reuse.

## 2. Materially inspected governing files and constraints

Read under `guillermomolina/protos` at the current source inspection revision except where historically noted:

| File / authority | Material result |
| --- | --- |
| `AGENTS.md`; `AGENTS.work/DESIGN.md` | Specification precedes implementation; GITHUB010 multi-approach research, 12 dimensions, future-regret/anti-overengineering and exact owner approval; GITHUB021 no hidden invariant redefinition. |
| `AGENTS.work/REFERENCE.md` | Small universe, no privileged pets, explicit local mechanisms, orthogonality, pay-as-you-grow, preservation of optional future paths. |
| `AGENTS.work/COORDINATION.md` | GitHub Issue is live authority; revision-bound non-normative records; independent I-family implementation; final closure comment and post-publication verification. |
| `spec/io/PROCESS_IO.md` ``22, 26, 28 | Process != OS process. RootActor bootstrap receives only explicit host-authorized facilities; Process authority does not imply Filesystem, Network or subprocess. Guest resource lifetime and termination revocation are Process-local; separately granted peers are not automatically revoked. |
| `spec/io/FILESYSTEM.md` ``18, 20–20.4 | Path is data, not authority; Filesystem is the capability; no standard public constructor; provider/base confinement; stable File binding; captured Filesystems are separate, host-provisioned immutable snapshots, **not** a live subtree attenuation constructor. |
| `spec/io/IO_CORE.md` | Effect commitment, cancellation, Future, explicit close and resource cleanup; no rollback after committed effects. |
| `spec/concurrency/ACTORS.md` ``8–9, 16, 24, 25, 32 | RootActor and Actor life/transfer boundaries; no automatic actor authority inheritance, no implicit proxy for nontransferable File/Network/Filesystem. |
| `spec/concurrency/PARALLEL_EXECUTION.md` `71.11 | No Core live-Filesystem authority through P capture/transfer. |
| `spec/semantics/MODULES.md` (initial module, imports and host entries) | Imported module contexts and non-root Actor initial modules are not granted RootActor bootstrap slots. |
| `spec/concurrency/DISTRIBUTED_RUNTIME.md` ``37, 61, 65–70 | New Process is ephemeral capacity, not OS child or reincarnation. Scoped exclusive distributed Authority is distinct from ordinary host File permissions. |
| `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosFreshProcessExecutor.java` | Existing host-owned mechanism creates an independent Process and distinct Context, optionally sharing an Engine; not a general guest Process launcher. No-host overload refuses Network/Resolver grants that require the provisioning host. |
| `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosStandaloneHostedExecution.java` | `FilesystemGrant.directory(Path)` starts from a host absolute directory; `FilesystemProvisioning` safely selects current standalone grant and binds per-Process operations; there is no proven derived-grant API from an already held root FD. |
| `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosStandaloneProcessBootstrap.java` | Each RootActor receives exactly optional grants via Process creation; not an inference from the Process prototype. |
| `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosPolyglotRuntimeHost.java` | Per-Process Filesystem backend registry rejects duplicate ownership; Process termination releases that backend; host may share Engine and lazily created Actor scheduling infrastructure. |
| `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosRootActorServiceSession.java` | Explicit opt-in long-lived service; Process grant lifetime differs from source evaluation lifetime. |
| `src/main/java/com/guillermomolina/protos/runtime/process/ProtosProcessRuntime.java` | Process-level RootActor and separate optional Root grants/retirement, Process own lifecycle. |
| `src/main/java/com/guillermomolina/protos/execution/host/io/ProtosLinuxConfinedReadWriteTreeFilesystemBackend.java` | Current Linux backend pins a directory FD, uses `openat2`, grant-specific Landlock worker and Process lifetime. Those primitives alone do not implement child grant derivation. |
| `src/main/java/com/guillermomolina/protos/execution/host/resources/ProtosEmbeddedFilesystemCustody.java` | Standard embedding can be subject to a custom Truffle FileSystem provider; a derived grant must not bypass it by native host path reopening. |
| `src/main/java/com/guillermomolina/protos/cli/ProtosCli.java` | `--allow-filesystem=<directory>` is explicit for an application invocation, independently from network and other grants; no public child Process mechanism. |
| `src/test/java/com/guillermomolina/protos/execution/integration/{filesystem/ProtosLinuxConfinedFilesystemSecurityTest.java,filesystem/ProtosLinuxConfinedReadWriteTreeFilesystemBackendTest.java,process/ProtosRootActorBootstrapAuthorityTest.java,process/ProtosRootActorServiceAuthorityIntegrationTest.java}` | Security and bootstrap test anchors exist for current host grants. Their presence is **not** proof of D250-derived-grant acceptance. |

Published and owner-approved antecedents: [D248-B](../../decisions/tooling/D248_EXPLICIT_CONFINED_FILESYSTEM_CLI_RATIFIED.md), [PLAT072-E](../../decisions/platform/PLAT072_LINUX_CONFINED_FILESYSTEM_BACKEND_RATIFIED.md), [PLAT073-C FFM amendment](../../decisions/platform/PLAT073_JAVA_25_FFM_AMENDMENT_TO_PLAT072_E.md), [D247-C](../../decisions/tooling/D247_EXPLICIT_NETWORK_ROOTACTOR_SERVICE_CLI_RATIFIED.md), [D243-C](../../decisions/language/D243_SECURE_ENTROPY_EXPLICIT_CAPABILITY_C_RATIFIED.md), [D253-C1](../../decisions/language/D253_EXPLICIT_CORE_WALL_CLOCK_NANOSECONDS_C1_RATIFIED_2026_11_09.md). [D229 #1008](https://github.com/guillermomolina/protos/issues/1008) (external OS executables) and [INC002 #1004](https://github.com/guillermomolina/protos/issues/1004) (future shell) remain **unselected**.

## 3. Cross-family external prior art — actual mechanisms and failure lessons

| System / primary reference | Real mechanism / relevant caveat | D250 lesson |
| --- | --- | --- |
| [WASI / Wasmtime preopened directories](https://docs.wasmtime.dev/api/wasmtime_wasi/struct.WasiCtxBuilder.html) | Host supplies directory handles and filesystem rights per sandbox; no guest directory access without preopens; guest path is independent of host location. | Capability acquisition belongs to host; pass a bounded resource not an ambient string path. |
| [Deno Worker permissions](https://docs.deno.com/api/web/workers/) and [permissions](https://docs.deno.com/api/deno/permissions/) | Workers can inherit, receive none or get restricted permission options; cannot expand beyond parent. **Default inheritance** is deliberately not copied to Protos. | Explicit attenuation is feasible, but non-inheritance is a required Protos invariant. |
| [Deno OS subprocess model](https://docs.deno.com/runtime/fundamentals/security/) | `--allow-run` OS children operate outside runtime sandbox, and can gain privileges if launcher poorly constrained. | Do not confuse Protos Process creation with native executable creation; keep D229 distinct. |
| [Rust cap-std Dir](https://docs.rs/cap-std/latest/cap_std/fs/struct.Dir.html) | Operations relative to directory capabilities; ambient directory open is an expressly different operation. | Derive from resource authority, not path-prefix checks and ambient reopens. |
| [GraalVM Context.Builder / IOAccess](https://www.graalvm.org/sdk/javadoc/org/graalvm/polyglot/Context.Builder.html) | Context has explicit IO policy and may use custom providers. | Host Context isolation is useful; bypassing an effective provider with native host IO would be an authority violation. |
| [Node.js Permission Model](https://github.com/nodejs/node/blob/main/doc/api/permissions.md) and [Node SECURITY boundary](https://github.com/nodejs/node/blob/main/SECURITY.md) | Permission checks are not a complete adversarial OS sandbox; worker execArgv and symlink behavior have important limits. | Do not call a language permission check OS isolation; deny hidden worker/child widening. |
| [Linux openat2(2)](https://man7.org/linux/man-pages/man2/openat2.2.html) | `RESOLVE_BENEATH`, `RESOLVE_NO_XDEV` and sibling flags constrain actual resource acquisitions. | Host-only derivation should use pinned authority and real confined operations, not a check-then-reopen. |
| [Linux Landlock kernel docs](https://www.kernel.org/doc/html/latest/userspace-api/landlock.html) | Restrictions attach to thread and descendants, **not to unrelated sibling threads**. | Distinct grant workers/custody matter; accidental reuse can silently defeat separation. |
| [OpenBSD unveil(2)](https://man.openbsd.org/unveil) | Path/permission-specific process restrictions with a separate OS-wide scope. | Cross-platform host policy possible, but OS sandbox and guest Filesystem capability are different domains. |

These precedents cover *host-capability preopens*, *permission/worker runtimes*, *Polyglot embedding*, and *OS capability/sandbox mechanisms*: well over the minimum five systems / three approaches. Precedent is evidence, **not** Protos semantic authority. In particular no source proves the not-yet-written B1 attenuator secure or Native-ready.

## 4. Complete candidate set and comparative GITHUB010 scores

A — **Independent host-only provisioning without derivation** (status quo).
B — **B1: explicit host-derived, non-amplifying independently custodied grants** (**selected**).
C — **Public host-granted guest ProcessLauncher plus attenuation and child lifecycle now** (credible future API, not selected).
D — **Privileged Process/session manager handling authority/control at scale** (not selected).
E — **Automatic inheritance from parent** (rejected outright: contravenes D248 owner-approved invariant and current authority isolation).
F — **Copy parent Filesystem or add only guest-visible `filesystem.restrict`** (rejected: does not establish host authorization or independent Process custody; risks unsafe backend aliasing).

Scores 1=weak to 5=strong, with H=HIGH, M=MEDIUM, L=LOW confidence. Each cell includes its rationale; **not** measured resource/Native results, and totals are deliberately not used as design authority.

| GITHUB010 dimension | A: independent host | **B1: derived host** | C: public launcher now | D: manager |
| --- | --- | --- | --- | --- |
| 1 Correctness/invariants | 5H — no new grant path | **5M — explicit subset proof** | 4M — proof plus public entry | 4L — policy interaction risks |
| 2 Protos alignment | 3H — limited composition | **5M — existing Process/capabilities** | 4M — ordinary capability possible | 2M — privileged institution |
| 3 Present-need proportionality | 5H — already sufficient for in-Process | **4M — only bounded host seam** | 2H — shell API still exploratory | 1H — speculative coordination |
| 4 Incremental growth | 2M — no derivation seam | **5M — guest frontend later** | 4M — broad API fixed early | 4M — extensible but heavy |
| 5 Future-option resilience | 3M — defers essential boundary | **5M — no launcher/backend lock-in** | 4M — compatibility commitments | 4M — service lock-in |
| 6 Scalability | 3M — independent Processes already | **4L — per-grant workers may cost** | 4L — same + many handles | 5L — central policy possible |
| 7 Conceptual simplicity | 5H — no new semantics | **4M — one explicit host derivation rule** | 2H — new public lifecycle | 1H — multiple new abstractions |
| 8 Portability/freedom | 5H — currently possible | **4L — secure provider-specific deriving** | 3M — launch semantics plus providers | 3L — cross-host control |
| 9 CPU/memory/resource | 5H — zero new cost | **4L — pay per derived grant** | 3L — handles/Futures overhead | 1H — continuous coordination |
| 10 Failure/operability | 4H — known bootstrap failures | **4M — fail closed before entry** | 3M — launch/cancel races | 3M — manager health/recovery |
| 11 Deferral/reversibility | 3M — later add host seam | **5M — can add public API later** | 3M — public API hard to retract | 2M — difficult to decentralize |
| 12 Evidence/risk | 5H — exists | **3L — mechanisms exist, derivation unproven** | 2L — public semantics unfinished | 2L — unjustified full system |

**Non-compensating red flags:** D is overengineered today regardless of its possible scale; C creates a durable guest API and supervision/lifecycle obligations ahead of actual need. A is the smallest current implementation and still valid for ordinary in-Process shell functions, but leaves no safely specified child grant derivation. B1 preserves the increment with only the boundary needed.

### Mandatory incremental-design questions

**Smallest sufficient solution today:** nothing additional for a command inside the same Process—reusing the D248-B host-granted Filesystem is already selected and implemented. For a *new* Protos Process, B1 specifies only host-authorized independent grants: none/same root/safely selected descendant, with effective authority intersection. No guest Process API is needed in this ratification.

**What must change when implementing later:** a grant-derivation adapter starting from a pinned, effectively authorized resource; per-Process grant ownership, session-dependent revocation and cleanup; correct provider-specific backing; normative Process/Filesystem cross-reference as needed; tests. The current `FilesystemGrant.directory(Path)` path-based host-authorization entry is **not** itself a safe derived-capability API. Existing Process/Context/RootActor identity need not change.

**What if guest Process launching becomes necessary?** Independently design an explicit host-provisioned guest launcher with spawn/result/cancel/child handle/supervision/message semantics; build it over B1's host seam. This is incremental extension, not a guaranteed existing API or immediate implementation.

**What premature machinery would we regret?** A Session global, permission DSL, ambient subprocess inheritance, global worker sharing across different Landlock domains, Process manager, multi-root ACLs, a public method constructor or mandatory per-command prompts.

**Strongest argument against B1:** host-only derivation is not directly usable by a future guest shell until a separate public launcher decision/implementation; B1 could also be expensive to prove for a complex filesystem provider, particularly when moving a pinned parent directory or crossing mount boundaries. If concrete guest-initiated child Process requirements become immediate, revisit C explicitly rather than smuggling C into an I-family implementation.

## 5. Adversarial falsification matrix: proposal, NOT executed

| ID | Attempt | Required outcome |
| --- | --- | --- |
| T01 | No effective session filesystem grant requests a child grant | Rejected; no new Filesystem |
| T02 | Child creation omits Filesystem request | No `filesystem` bootstrap slot |
| T03 | Parent root `/data`, child asks for `/etc` or a string prefix trick | Rejected without guest effects |
| T04 | Child asks for a genuine descendant | New bounded grant only for that subtree |
| T05 | Rename/symlink/ancestor substitution between select and acquisition | No widening or link escape |
| T06 | Same-device or cross-device bind mount beneath selected root | Never crosses forbidden mount boundary |
| T07 | Pinned parent directory moved outside root before `replace`/`remove` | No out-of-grant mutation |
| T08 | Imports, Actor.spawn, Process, ActorRef, package, Tool and P | No automatic derivative authority |
| T09 | Nontransferable Filesystem in Actor graph | Standard NonTransferableValue, no silent proxy |
| T10 | Two independent Process grants terminate in different order | No accidental cross-revocation |
| T11 | Session grant revoked while child Process remains alive | Dependent authority unusable; unrelated independent grant survives |
| T12 | RootActor/Process bootstrap fails after selecting a descendant | No partially reachable capability; cleaned custody |
| T13 | Child makes nested descendant/ancestor request | No grant amplification at any level |
| T14 | Custom Polyglot filesystem provider is virtual/restricted | No native host-FS bypass or fallback |
| T15 | Network-only, Resolver-only, Entropy/Clock, combined/no grants | All independent, no inferred slots |
| T16 | Native OS executable started by another authority | No false OS sandbox claim |
| T17 | Concurrent descendant creation/replacement, revocation and pending operation | Linearizable authority boundary, correct IO commitment |
| T18 | Unsupported Linux kernel, provider or OS feature | Provisioning fails closed; no unsafe substitute |
| T19 | Rebuilt JVM and installed Native Image | Exact authority/lifecycle equivalence where explicitly supported |
| T20 | Thousands of derivations; program never deriving any | Quantify per-grant cost; no mandatory unused machinery |

Existing I155/I156 tests and their reported completion do not establish that T01–T20 were executed or passed for B1. The Linux 25-FFM backend and precise worker/lifecycle constraints are necessary implementation inputs, not proof of this not-yet-written operation. Portability and binary authenticity are separate evidence obligations.

## 6. GITHUB021 owner-invariant/delta audit

| Approved owner invariant / normative boundary | Candidate B1 | New delta / reopening |
| --- | --- | --- |
| D248-B session reuse for multiple same-Process commands, **no per-command prompts** | PRESERVED | None |
| D248-B one explicit, confined default CLI Filesystem root | PRESERVED | Derived *new Process* grant must be independent, no new CLI flag |
| `PROCESS_IO` host authority, bootstrap-local slots, absent not null | PRESERVED | New host derived provisioning can choose none/same/descendant root |
| `FILESYSTEM` confinement and Path without authority | PRESERVED | Derived selection must prove subtree confinement, no string-prefix reauthorization |
| No import/Actor/P authority inheritance or silent proxy | PRESERVED | Child Process likewise no implicit inheritance |
| D247 Network/service, D249-A1 Resolver, D243 Entropy, D253 Clock | PRESERVED | Distinct independent grants only |
| `MODULES` initial RootActor context, Actor-local instance/cache | PRESERVED | New Process keeps independent module state |
| `ACTORS` lifecycle, Resource/Future commitment, Context termination | PRESERVED | Session revocation cuts dependent authority without proving child Process death |
| `DISTRIBUTED_RUNTIME` Process identity versus OS process, scoped decision Authority | PRESERVED | No new distributed control-plane authority or host OS sandbox |
| D229 native subprocess (open), INC002 shell (incubator) | NOT SELECTED by B1 | Both remain their respective future questions |
| PLAT072/073 Linux openat2/Landlock FFMed confined host backend | PRESERVED | Implementation proof still necessary; no unapproved native fallback |

**GITHUB021 result: PASS for exact B1 ratification.** No recorded owner-approved invariant is overridden. The distinct new architectural consequence is an explicitly host-authorized, independently custodied, non-amplifying Filesystem grant for another Protos Process; it is **not** a public guest Process API or immediate product change. A future owner would have to explicitly approve any such expansion.

## 7. Publication / validation boundaries

- **Owner decision:** approved exactly D250-B1 (2026-11-10).
- **Product baseline:** `c9da7ff3e6c95f7a88948e875f61d67007fdeaa5`, inspection-only.
- **Product source/specification:** no D250-B1 edits in this coordination publication.
- **Build / focal / integrated / Native tests for B1:** **NOT RUN**, no PASS claimed.
- **Live GitHub coordination:** after durable publication and re-reading, D250 may close with an exact final evidence comment; I165 remains open for implementation. Native parent relation for I165 must be separately verified and never inferred from this prose.
- **Durable repository:** this evidence and decision belong to `guillermomolina/protos-project-docs` under DOC002 role-first paths. Its published commit SHA is recorded in the final GitHub decision Issue after successful commit/update-ref verification, not invented here.
