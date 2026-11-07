# PLAT072-E — Source-backed decision research, scoring, falsification and owner approval

**Date:** 2026-11-07. **Nature:** GITHUB010/GITHUB021 architectural evidence,
not executed test evidence. [Canonical PLAT072 decision](../../decisions/platform/PLAT072_LINUX_CONFINED_FILESYSTEM_BACKEND_RATIFIED.md).
**Exact product source baseline:** [`guillermomolina/protos@398e6cb7478b0d7735ffcb6787d43c723cc32937`](https://github.com/guillermomolina/protos/commit/398e6cb7478b0d7735ffcb6787d43c723cc32937).
**Durable D248-B authority:** [`guillermomolina/protos-project-docs@afe0cb9d3304108de80a049c26e2475abac5d50c`](https://github.com/guillermomolina/protos-project-docs/commit/afe0cb9d3304108de80a049c26e2475abac5d50c).
This record is published in its own exact docs commit; references to this
record after publication must name that revision rather than treating the
product revision as an implementation.

## 1. Approved candidate and scope of approval

[PLAT072/#1082](https://github.com/guillermomolina/protos/issues/1082)
was created by the root-provisioning race found in I155/#1079. The project
owner's next-turn reply to the **exact conditional recommendation E** was:
**"ok apruebo la recomendación."** Candidate E was:
**actual openat2 acquisition + mandatory grant-specific Landlock restrictions +
narrow JNI/C Linux shim + isolated workers, keeping Java 21 and existing
host-neutral Protos backend interfaces**. It includes falsification of the
moved-parent rename/unlink case. Candidate E is not a promise that Landlock
solves that interleaving: failing its kernel proof mandates rejection/further
architectural choice, not fallback to insecure path checks.

Approval preserves the approved D248-B semantic boundary, including one
existing CLI grant, one initial RootActor capability and its full standard
recursive namespace semantics. No new `Dxxx` semantic amendment was approved
or required by this architecture; any newly discovered semantic conflict must
stop implementation and be explicitly routed to a distinct D decision.

## 2. Exact product-source audit and existing tests

The published `main` baseline inspected was `398e6cb7478b0d7735ffcb6787d43c723cc32937`:
- [`AGENTS.md`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/AGENTS.md),
  [`AGENTS.work/DESIGN.md`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/AGENTS.work/DESIGN.md)
  (GITHUB010/GITHUB021), [`COORDINATION.md`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/AGENTS.work/COORDINATION.md)
  (GITHUB006/GITHUB015/GITHUB020), [`REFERENCE.md`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/AGENTS.work/REFERENCE.md),
  [`IMPLEMENTATION.md`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/AGENTS.work/IMPLEMENTATION.md),
  [`src/AGENTS.md`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/src/AGENTS.md),
  [`spec/AGENTS.md`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/spec/AGENTS.md).
- [`spec/io/FILESYSTEM.md`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/spec/io/FILESYSTEM.md)
  §§18.0–18.5, 20.1–20.4: open tuple and commitment, race-free selection,
  final-entry semantics, `entries` name/kind consistency and no provider escape.
  [`spec/io/PROCESS_IO.md`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/spec/io/PROCESS_IO.md)
  §§22, 28: provider restrictions, bootstrap fail-closed, Process termination.
- [`pom.xml`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/pom.xml)
  (`maven.compiler.release=21`, Graal 25.5.5.0.3, Native profile) and
  [`toolchain.json`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/toolchain.json)
  (GraalVM Community JDK 25.0.5.0.3 selected).
- [`ProtosNioReadOnlyTreeFilesystemBackend.java`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/src/main/java/com/guillermomolina/protos/execution/host/io/ProtosNioReadOnlyTreeFilesystemBackend.java):
  secure descendant directory handles, final no-follow, **read-only**.
- [`ProtosNioConfinedFilesystemBackend.java`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/src/main/java/com/guillermomolina/protos/execution/host/io/ProtosNioConfinedFilesystemBackend.java):
  host-pinned `SecureDirectoryStream` but writable createNew and
  mutation allowlists limited to **direct children**.
- [`ProtosEmbeddedFilesystemCustody.java`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/src/main/java/com/guillermomolina/protos/execution/host/resources/ProtosEmbeddedFilesystemCustody.java):
  distinct effective Truffle `FileSystem` provider, no native host bypass.
- [`ProtosStandardFilesystemProtocol.java`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/src/main/java/com/guillermomolina/protos/execution/protocols/io/ProtosStandardFilesystemProtocol.java),
  [`ProtosFilesystemOpenFlow.java`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/src/main/java/com/guillermomolina/protos/runtime/io/ProtosFilesystemOpenFlow.java),
  [`ProtosFilesystemOpenOptions.java`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/src/main/java/com/guillermomolina/protos/runtime/io/ProtosFilesystemOpenOptions.java),
  [`ProtosFilesystemNamespaceMutationFlow.java`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/src/main/java/com/guillermomolina/protos/runtime/io/ProtosFilesystemNamespaceMutationFlow.java):
  existing host-neutral adapters and success/cancel cutover.
- [`ProtosCli.java`](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/src/main/java/com/guillermomolina/protos/cli/ProtosCli.java),
  [NIO confined backend tests](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/src/test/java/com/guillermomolina/protos/execution/integration/filesystem/ProtosNioConfinedFilesystemBackendTest.java),
  [NIO read-tree tests](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/src/test/java/com/guillermomolina/protos/execution/integration/filesystem/ProtosNioReadOnlyTreeFilesystemBackendTest.java):
  cover legacy surfaces; not a race-proof Linux recursive read/write grant.

The I155 **uncommitted** patch was not accessible in the published `main`
revision and was **not** audited as if committed. The exact reported defect is
recorded in [PLAT072/#1082](https://github.com/guillermomolina/protos/issues/1082):
`/proc/self/mountinfo` is sampled separately from the pinned root, with
another pathname acquisition and later `fileKey` confirmation. An ordinary
ancestor swap-and-restore can pair one pinned root with another root's mount
assessment. A post-hoc identity equality cannot make the earlier mount check
atomic. No source inspection can certify that unpublished patch has been fixed.

The owner reported **`mvn test-compile` BUILD SUCCESS** and **257 focused
tests / 0 failures / 0 errors / 0 skips** in I155 on 2026-11-07. Neither
`make test` nor `make check` nor a fresh installed Native binary acceptance
was reported for that final slice. These are owner-reported validations, **not
tests executed by this decision transaction** and not a security proof.

## 3. Cross-system investigation: mechanisms and limits

| System or mechanism | Relevant precedent | Non-equivalence / constraint |
| --- | --- | --- |
| Linux `openat2(2)` | `BENEATH`, `NO_SYMLINKS`, `NO_XDEV`, bounded `EAGAIN` retry | Applies to the selected *open* operation, not later `unlinkat` on a relocated parent fd |
| Linux Landlock LSM | Rules anchored in directory hierarchy, `REFER` since ABI 2, `TRUNCATE` since ABI 3, per-thread enforcement | Does not retroactively regulate all pre-opened FDs; moved-parent mutation proof remains a required gate |
| POSIX `renameat2` / `unlinkat` | Kernel entry operations and stable directory descriptors | No `RESOLVE_*` parameters; parent fd may outlive a directory move |
| OpenJDK `SecureDirectoryStream` | No-follow relative directory operations on supported providers | No portable kernel `NO_XDEV` contract; Java NIO provider is optional |
| GraalPy | Distinct native POSIX and Java/Truffle provider boundaries | Native POSIX access cannot be presumed bounded by Truffle host provider |
| TruffleRuby | Native-host interface evolution (including Panama) | Native interoperability is not confinement; use as cost/ABI precedent |
| GraalJS | Host/polyglot filesystem access integration | Provider selection alone cannot prove Linux mount and rename boundaries |
| Espresso | Native/JVM execution boundary separation | Truffle hosting is not a host OS sandbox |
| Sulong/LLVM | Mediated native/LLVM execution alternative | Additional execution substrate does not itself supply VFS confinement |
| SimpleLanguage | Minimal Truffle/Native integration sample | No comparable production filesystem authority substrate |
| Apple Pkl | Host-provisioned permissions and IO isolation | Configuration/model is not proof of recursive writable Linux entry atomicity |
| WASI preopens | Directory capability and relative handles | Does not itself decide Linux mount/rename behavior for this host |
| FreeBSD Capsicum | Capability-mode handle-mediated host resource restrictions | Valuable OS design reference, unavailable as Linux syscall ABI |
| Windows / macOS | Reparse/junction and Darwin vnode/mount differences | Linux kernel mechanism not portable by pretending to use Java NIO alone |

Representative external anchors:
[openat2(2)](https://man7.org/linux/man-pages/man2/openat2.2.html),
[landlock(7)](https://man7.org/linux/man-pages/man7/landlock.7.html),
[rename(2)](https://man7.org/linux/man-pages/man2/rename.2.html),
[unlink(2)](https://man7.org/linux/man-pages/man2/unlink.2.html),
[Java 21 SecureDirectoryStream](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/file/SecureDirectoryStream.html),
[GraalVM Native JNI](https://www.graalvm.org/latest/reference-manual/native-image/dynamic-features/JNI/),
[WASI filesystem](https://wa.dev/wasi%3Afilesystem),
[Capsicum](https://manpages.debian.org/trixie/freebsd-manpages/capsicum.4freebsd.en.html),
[Pkl threat model](https://pkl-lang.org/threat-model.html).
The surveyed languages are precedent, not normative authority or evidence that
Candidate E's kernel race cases pass.

## 4. Full meaningful candidate set and elimination

A: existing NIO SecureDirectoryStream + mountinfo/fileKey checks.
**Eliminate:** a pathname lookup and mount-table assessment do not share
an atomic selection; repeated checks do not create a security proof.

B: Linux `openat2` actual selector, followed by `renameat2`/`unlinkat`
relative to earlier selected parent FDs.
**Eliminate as complete design:** a concurrent rename may move an opened parent
outside the root before final entry mutation.

C: B through Panama FFM.
**Eliminate for current source/toolchain:** keeps B's missing kernel proof and
requires Java preview/release compatibility work that Java 21 policy did not
approve. FFM in current runtime JDK 25 does not imply `--release 21` can
compile a stable JDK 22+ API.

D: ordinary secure Truffle or Java NIO provider with no kernel-enforced
mount/mutation boundary.
**Eliminate for this Linux grant:** no comparable `NO_XDEV` contract or
comprehensive relocatable-parent mutation proof.

E: actual `openat2` + grant-scoped Landlock + narrow JNI Linux shim
+ isolated workers.
**SELECTED (owner-approved), with mandatory falsification**. Prevents
the known cross-root mountinfo check if correctly implemented, leaves guest
contract untouched; correctness of entry mutations remains to be proven.

F: process/broker isolation with restricted mount namespace and an IPC
Filesystem adapter.
**Credible contingent alternative** if E's kernel proof fails; requires a
separate exact architecture approval, packaging and zero-privilege feasibility
proof before adopting. It is *not* a secretly selected E implementation.

G: fail closed/no CLI grant on hosts where no mechanism can prove confinement.
**Mandatory safe operating behavior**, not a sufficient implementation of
the existing user-facing D248-B adoption goal.

## 5. GITHUB010 12-axis comparative scored packet

Scale 1–5 (larger is better); confidence `H` high, `M` medium, `L` low.
Scores compare alternatives, **not demonstrated implementation quality**.
Disqualified A–D do not survive the hard correctness gate, so the detailed
survivor comparison covers E/F/G. Per-cell reasons are included; no total score
may override a correctness failure.

| Required axis | E: openat2 + Landlock + JNI | F: broker + host namespace | G: fail-closed deferral |
| --- | --- | --- | --- |
| 1 Correctness / invariants | **4 L**: mechanism promising; rename/mount proof unverified | **4 L**: isolation credible, broker/namespace correctness unverified | **1 H**: refuses unsafe grant but does not implement ratified availability |
| 2 Protos alignment | **5 M**: guest capabilities ordinary, mechanism stays host-only | **4 M**: same guest protocol, heavier host institution | **3 H**: no new public concepts but cannot deliver selected grant |
| 3 Present-need proportionality | **4 M**: costs charged only when grant is used | **2 H**: process and IPC expense for simple CLI grant | **5 H**: no implementation cost today, misses functionality |
| 4 Grow as needed | **4 M**: backend-neutral seam allows more providers later | **4 M**: broker can add isolation providers | **3 M**: postpones concrete architecture and execution tests |
| 5 Future options | **4 M**: retains guest API and broker escape path | **4 M**: runtime neutral beyond IPC interface | **5 H**: keeps options open but resolves nothing |
| 6 Scalability | **3 M**: per-grant policy/thread resources require scaling proof | **3 M**: broker/process count and IPC contention | **5 H**: no workers, but no Filesystem workload |
| 7 Conceptual simplicity | **3 M**: two kernel mechanisms, bounded host shim | **2 H**: additional process/protocol/monitoring semantics | **5 H**: rejection is simple |
| 8 Portability | **2 H**: Linux only; unsupported platforms explicitly reject | **2 H**: namespace/broker policy remains OS-specific | **5 H**: portable rejection rather than pretend support |
| 9 Machine costs | **3 M**: JNI/syscalls/FDs/workers only on grant | **2 M**: IPC and copies, additional process lifecycle | **5 H**: absent grant imposes no native runtime work |
| 10 Failure/operability | **4 M**: errno and explicit kernel refusal localized | **3 M**: broker death, fencing, IPC and partial failures | **4 H**: early deterministic denial, missing feature |
| 11 Deferral/reversibility | **4 M**: swap backend to broker if required | **3 M**: larger substrate to remove/replace | **2 M**: no portable grant until backend later supplied |
| 12 Evidence/risk | **3 L**: kernel APIs mature, exact E integration not tested | **3 L**: established isolation, no Protos integration proof | **5 H**: safe denial requires no novel kernel proof |

**Non-compensating flags:** A–D have a correctness disqualifier (A/B/D)
or fail to solve it while charging migration complexity (C). E carries
a **low-confidence security red flag** until direct falsification: no
score offsets it. F has an **overengineering red flag** for a single
grant: broker, IPC and isolation would be paid now without evidence
that E cannot work. G has an **underengineering red flag** because the
ratified D248-B CLI use case is not delivered at all. Scores do not
authorize weakening a hard invariant.

## 6. Pay-for-use / growth / exact cost of deferral

**Smallest sufficient solution:** one Linux backend under the existing
`ProtosStandardFilesystemProtocol.Backend`, configured only by an explicit
grant; one pinned root, one Landlock policy per granted authority scope,
isolated native worker ownership, JNI access to only the necessary syscalls.
No unused-grant setup, no public language change and no framework
anticipating child Processes, arbitrary roots or system-wide sandboxing.

**Grow as needed:** an independently designed provider backend, broker or
attenuated child authority can be added without changing Path/File/Filesystem
if the current root/grant abstraction remains host-only and its resources
have explicit custody. Cross-platform support must prove equivalent
contracts separately; a Java wrapper is not equivalent evidence.

**Cost of deferral:** deferring F avoids a new process/IPC lifecycle and
failure mode now; adopting it later rewrites the private backend, native
deployment and possibly scheduler wiring, **not** normative guest API.
Deferring E entirely leaves the D248-B CLI unusable for real persistent
data and blocks I155 acceptance. Deferring optional narrower rights or
multiple roots has no present compatibility cost because they were
explicitly not selected by D248; inventing them now would be speculative.

**Future regret/escape:** Many multi-Context Processes, restricted embedding,
alternate OS or failure of moved-parent Landlock proof could make E
inappropriate. Escape: keep backend contract, stop and explicitly approve F
or a different kernel-backed proof; do not reinterpret D248 or weaken it.
Long-lived services, shutdown/cancellation, Truffle carrier reuse and
Native fallback runtime are part of acceptance even for the first grant.

## 7. Deterministic falsification and deployment gates

| ID | Adversarial case | Required result |
| --- | --- | --- |
| T01 | Rename/symlink root ancestor swap-and-restore between mount check and root pin | The exact pinned descriptor determines authority; never pair root with another mount assessment |
| T02 | Bind-mount and normal mount crossing under root, including same-device | `NO_XDEV`/effective kernel policy denies traversal |
| T03 | Intermediate/final symbolic link, including magic links | Acquisitions never follow outside targets; entry-only remove/replace may act on symlink itself |
| T04 | Open descendant parent fd; another host thread moves it outside root; attempt unlink | No entry mutation outside authorized namespace |
| T05 | Cross-directory rename/move while source/destination parents are reparented concurrently | No namespace effect outside; atomic selected transition or clean failure |
| T06 | Concurrent `createNew` same target | Exactly one exclusive creation can commit; new content is empty |
| T07 | `create` vs external concurrent create/remove | Race-free open-or-create selection, no spurious probe failure |
| T08 | `truncate`/create/cancel and IO failure at each commit boundary | Result reflects actual irreversible effect, never false failed/no-effect |
| T09 | `replace` of symlink source/target and cross-kind entry combinations | Entry replacement atomic if supported; reject unsupported transitions safely |
| T10 | `remove` regular, symlink, empty/nonempty directory | Never follows target and never recursively removes |
| T11 | `entries` amid name/type swaps, `DT_UNKNOWN`, unrepresentable names | Complete nonduplicate observation; no mismatched name+kind |
| T12 | File acquired, then rename/replace/remove | Selected File descriptor and logical cursor unchanged |
| T13 | Missing Landlock, inadequate ABI, blocked syscalls, unsupported fs/provider | Provisioning fails **before** guest executes; no NIO fallback |
| T14 | Multiple Processes, Contexts, Actors, Tasks and worker recycling | No authority bleed, no unconstrained native operation |
| T15 | Service lifetime, failed bootstrap, cancellation and Process termination | Revoke resources without premature service teardown or implied sync |
| T16 | JVM and real installed Native for each supported Linux ABI; persistence across restart | Same guest contract, real data, verified binary/linked native dependency and negative security cases |

Use controlled interleaving barriers inside provisioner, directory
selection, and mutation paths where possible, not only random stress.
Test native errno/error translation (`EAGAIN`, `EINTR`, `EXDEV`,
`ENOSYS`, `ELOOP`, `EACCES`), fd reuse/close races, failure atomicity
and the actual Landlock ABI. For Native, record executable identity,
architecture, linked library identity and execution output; neither
compilation nor JVM PASS is Native parity.

**Proof dependency:** T04/T05 are mandatory falsification checks of
Landlock's handling of directories moved after fd acquisition. Openat2
does not supply those syscall restrictions itself. If a counterexample
survives, report `E_REJECTED`, leave unsafe grant disabled and return to
PLAT design approval for F; do not relabel a passing subset as closure.

## 8. GITHUB021 explicit invariant/delta review

| Approved invariant | Candidate E outcome | Result |
| --- | --- | --- |
| D248-B one CLI grant, none by default | Unchanged; backend only when grant exists | PRESERVED architecturally |
| D248-B pinned host root, relative Path, full recursive authority | Root fd + `openat2` + Landlock; proof T01–T05 outstanding | CONDITIONED |
| D248-B no link/mount/rename escape | Requires actual kernel proof, no pathname precheck | CONDITIONED, NOT SECURITY PASS |
| D241-B existing/create/createNew/truncate tuple and cursor | Same Flow contract, atomic native acquisitions | CONDITIONED on T06–T08/T12 |
| FILESYSTEM §20.3 entries/remove/replace | No new operation; final-entry semantics | CONDITIONED on T09–T11 |
| Process §28 custody, RootActor-only bootstrap | Same Process owner, worker isolation/revocation | PRESERVED; T14–T15 pending |
| D247 Network/service and independent D249 Resolver | No grants or isolation through other capabilities | PRESERVED |
| D250 shell-child/new Process design still unselected | No new child Process feature or inheritance | PRESERVED |
| Truffle restricted embedding provider | Direct Linux shim **not used** as embedding bypass | PRESERVED boundary |
| Java 21 and GraalVM 25.5.5.0.3 | JNI shim; no FFM/Java source-release migration | PRESERVED boundary |
| No general OS-process/native-code sandbox claim | Limited to exact provisioned capability and threats D248 approved | PRESERVED |

**No known prior owner invariant is reopened.** Architecture-only delta:
Linux kernel-backed direct selector plus mandatory LSM and JNI worker
isolation replace the unsafe pathname/mountinfo claim. Any new semantic
restriction requires a separate D decision and approval. This audit
certifies **approval consistency**, not test or security success.

## 9. Publication and live coordination nonclaims

The active owner expressly approved E after the complete recommendation
on 2026-11-07. The decision requires a platform ratification document
and this evidence record before dependent I156 can pass its design gate.
The GitHub native parent/subissue and blocked-by edges require separate
live reconciliation and verification; textual mentions do **not** establish
native relationships. Product implementation, `git diff --check`,
Maven tests, `make test`, `make check` and Native Image runs were **not**
executed in this research/decision transaction. Product code/spec and
tests were **not changed** by this project-docs-only publication.
