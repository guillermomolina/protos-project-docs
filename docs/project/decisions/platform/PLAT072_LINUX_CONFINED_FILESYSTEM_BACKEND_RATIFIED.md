# PLAT072 — Linux race-free confined Filesystem substrate

**Status: RATIFIED — Candidate E (2026-11-07), owner-approved with mandatory falsification gates.**
This is a **non-normative host/platform architecture** decision. It does **not** certify
an implemented, race-free Filesystem, completed I155/I156, passing tests or Native Image support.

- Decision: [PLAT072/#1082](https://github.com/guillermomolina/protos/issues/1082).
- Trigger/parent work: [I155/#1079](https://github.com/guillermomolina/protos/issues/1079);
  implementation/proof owner: [I156/#1083](https://github.com/guillermomolina/protos/issues/1083).
- Exact project-owner response following the Candidate E recommendation on 2026-11-07:
  **"ok apruebo la recomendación."** The approved recommendation included its conditional
  security gates, no silent fallback, and no Java 21 change.
- Full [research, candidate scores, source audit, invariants, and falsification gates](../../evidence/PLAT072/PLAT072_E_OWNER_APPROVAL_GITHUB010_GITHUB021_2026_11_07.md).
- Normative authority: [FILESYSTEM.md](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/spec/io/FILESYSTEM.md)
  §§18, 20.1–20.4 and [PROCESS_IO.md](https://github.com/guillermomolina/protos/blob/398e6cb7478b0d7735ffcb6787d43c723cc32937/spec/io/PROCESS_IO.md)
  §28, subordinate to the [D248-B ratification](https://github.com/guillermomolina/protos-project-docs/blob/afe0cb9d3304108de80a049c26e2475abac5d50c/docs/project/decisions/tooling/D248_EXPLICIT_CONFINED_FILESYSTEM_CLI_RATIFIED.md).
- Inspected **Protos product revision**:
  [`398e6cb7478b0d7735ffcb6787d43c723cc32937`](https://github.com/guillermomolina/protos/commit/398e6cb7478b0d7735ffcb6787d43c723cc32937);
  this is a **public research baseline**, **not** the uncommitted I155 patch or an I156 implementation.

## Decision: Candidate E (architecture, not proof)

For an explicitly provisioned **Linux standalone CLI Filesystem grant only**, implement
a narrow **kernel-backed host backend** with all these parts together:

1. **Safely pin the authorized root by an actual directory file descriptor.**
   The caller-provided path is input to a one-time selection. Reject a final-root
   symlink and unverifiable root/provider. The selected descriptor, not a later
   pathname lookup or a separately sampled `/proc/self/mountinfo` record, owns
   the authorization identity. The authorized root itself may be on a mounted
   filesystem; disallow **crossing mounts beneath it**, including same-device
   bind mounts. No pathname revalidation is permitted as a substitute for pinning.
2. **Select every guest-opened descendant through real `openat2` acquisitions**
   relative to the pinned root with `RESOLVE_BENEATH`,
   `RESOLVE_NO_SYMLINKS`, `RESOLVE_NO_MAGICLINKS`, and `RESOLVE_NO_XDEV`
   (the redundant-looking explicit magic-link bit records the intent).
   These flags apply to actual resource acquisitions, **never just to prechecks
   preceding unrestricted operations**. Bounded `EAGAIN` retries may fail closed;
   never relax resolution flags. Preserve the precise mode, creation, truncate
   and stable File-binding obligations in normative §18.
3. **Protect entry mutations by a mandatory Landlock policy** anchored to the
   exact selected root, applied to every native I/O executing thread/descendant
   that acts for that grant; a ruleset only on an unrelated thread is not an
   implementation. Required handled rights include the applicable read, write,
   create, remove, cross-directory `REFER`, and truncate controls; query the
   actual Landlock ABI/capabilities instead of trusting a kernel-version string.
   In particular, support for `REFER` and `TRUNCATE` must be adequate; do not
   degrade away either permission boundary for compatibility. Installation
   occurs **before guest work or privileged backend operations for the grant**.
4. **Use a contained JNI/C Linux shim**, retaining the project's
   `maven.compiler.release=21`, behind existing
   `ProtosStandardFilesystemProtocol.Backend` and host-neutral Open/Mutation/
   Entries flow contracts. The shim owns syscall ABI, native path representation,
   native descriptors, errno translation and cleanup; Java owns Protos Future,
   File capability, Actor/Process and cancellation integration. No JNI call
   from an unconstrained JVM carrier may bypass the grant-specific kernel policy.
   Provide explicit Linux x86-64/AArch64 deployment gates for both JVM and **real**
   Native Image builds; Native packaging/linking must be verified, not presumed.
5. **Mutations use genuine single-transition syscalls** (`renameat2`/equivalent
   atomic replacement and `unlinkat`/equivalent entry removal), with directory
   descriptors selected safely and **Landlock enforcing the effective policy at
   mutation time**. For `replace`/`remove`, the final symlink is an **entry**,
   never a referent to follow. Do not claim safety merely because the parent fd
   was confined at an earlier point: an external actor can move that parent.
   If concurrent reparenting can induce an out-of-authority effect in any
   supported operation, **Candidate E has failed its acceptance gate**.
6. **Enumerations and no-follow classification** must satisfy §20.4's
   name/kind single-observation requirement, including concurrent replacement,
   unknown `d_type` and exact name representation. No `readdir`/stat-by-path
   shortcut may silently combine two selected entry identities.
7. **Keep the public surface unchanged.** The existing one
   `--allow-filesystem=<directory>` opt-in, existing `Filesystem`/`Path`/`File`
   and standard operations, RootActor bootstrap-local slot, Context effective
   authority, Process lifetime and revocation, and Tool/import/Actor/P/Network/
   Resolver boundaries are not changed. No public constructor, OS-global
   filesystem sandbox, shell/process-spawn policy, per-operation prompts or
   blanket source/package-reading authority are introduced.

**Native operations and provider boundaries:** CLI code may use this Linux
mechanism only for the **exact host-granted CLI filesystem**. Never bypass
an embedding Context's custom/restricted Truffle file provider with a direct
native host-filesystem call. Parent and sibling runtimes/tools must not be
silently put under the irreversible Landlock policy. Workers need an explicit
per-grant isolation/custody discipline; no opportunistic generic worker pool
that reuses differently confined threads.

**Known scope limits of D248-B, unchanged:** this is not a general sandbox for
arbitrary native guest code, preexisting hardlinks/aliases that the approved
model expressly excludes, unprivileged-to-privileged attacker capabilities,
or privileged remounts *after* grant. These excluded threats do **not** excuse
ordinary ancestor substitution, rename/mount races, or backend bypass during
provisioning and subsequent namespace operations.

## Conditional implementation acceptance, not an optional refinement

`openat2` has resolution flags; `renameat2` and `unlinkat` do not. A parent
descriptor remains bound to the opened directory if the directory is later
moved. **Landlock plus kernel resolution is a selected architecture to
investigate and implement, not yet a security proof.** I156 must demonstrate
that its effective Landlock policy rules out an outside-root rename/remove
after the selected parent is moved and all related interleavings. Inability
to prove or falsification of this claim **disqualifies E**; do not patch over it
with repeated pathnames, mountinfo checks, `statx` observations or post-effect
verification. Return for a separate explicit PLAT decision on Candidate F
(isolated host broker / mount namespace subject to no new privileged requirement)
or fail-closed deferral.

Minimum regression/adversarial matrix: (T01) grant-root ancestor
swap-and-restore inside provisioning; (T02) same-device and different-device
bind/mount crossing; (T03) intermediate/final symlink; (T04) parent fd moved
outside between traversal and `unlinkat`; (T05) concurrent cross-directory
`renameat2`; (T06) two `createNew` calls; (T07) `create` with concurrent
create/delete; (T08) truncation, creation, cancel/commit and failures;
(T09) entry-kind combinations and symlink source/target replacement;
(T10) remove of regular/link/empty/nonempty directory; (T11) `entries`
name/kind identity amid swaps; (T12) open File stability after namespace
mutation; (T13) missing/insufficient `openat2`, Landlock, LSM, seccomp or
provider rejection **before guest execution**; (T14) multiple Process/Context
and Actor isolation; (T15) termination, service lifetime and in-flight cleanup;
(T16) actual JVM and **installed Native executable** persistence across restart,
security rejection, binary/dependency identity and architecture coverage.

The checks must be interleaving-controlled where possible; random stress is
supplemental. Cover `ENOENT`, `EEXIST`, `ELOOP`, `EXDEV`, `EACCES`,
`ENOSYS`, `EAGAIN`, `EINTR`, FD reuse, close/cancel races, unsupported
filesystem types, invalid/unrepresentable names, symlink entries and worker
lifetime. An ordinary failed operation must not publish an uncertain effect
as an uncommitted failure or a false success. Failure-atomic behavior and
observable outcome remain governed by the original normative flows.

**Platform availability:** a Linux backend is enabled only on architectures,
kernels, providers and distribution configurations for which this proof
succeeds. On Windows, macOS, unsupported Linux versions/providers or failing
Native deployments, refuse the *new CLI grant* before guest code: do not
silently switch to NIO/Truffle path fallback with weaker guarantees. This
does not revoke separately provisioned, otherwise conforming embedding or
Tool host capabilities.

## Explicitly rejected/deferred paths

- **A:** `SecureDirectoryStream` plus pathname/mountinfo/`fileKey`
  consistency checks; does not establish an atomic root-to-mount proof.
- **B alone:** `openat2` plus descriptor-relative `renameat2`/`unlinkat`;
  final-entry parent movement remains an unproved authority boundary.
- **C:** Panama FFM under current Java 21 release contract; adds compiler and
  Native deployment complexity without correcting B's missing security proof.
  FFM on a separately approved toolchain boundary may be revisited later.
- **D:** ordinary NIO/Truffle provider without kernel-backed confinement;
  cannot assert required mount and concurrent namespace guarantees.
- **F (fallback only after failed E proof):** isolated broker/namespace with
  a new bounded implementation/decision gate, not prebuilt now.
- **G:** no Linux CLI grant until an adequately proved backend exists; always
  the fail-closed operating state if E/F cannot be implemented correctly.

The selected minimum is **no new guest API and no Linux worker/native workload
without a grant**. Do not preimplement multiple roots, writable/read-only
permission families, `mkdir`, generic `mv`, shell sessions, OS subprocess
sandboxing, general JNI/FFM infrastructure, child Process inheritance or
new Container-style machinery. A future broker may replace the host backend
behind the same protocol if process density, privilege separation, portability
or Landlock limitations justify it. This is future-compatible, not
future-preimplemented.

## GITHUB021 invariant and approval boundary

Preserved: D248-B one explicit confined recursive full-protocol grant; D241-B
source-backed open mode and immutable option capture; `FILESYSTEM.md` atomic
selection/entry changes, stable File and `entries`; `PROCESS_IO.md`
bootstrap-local optional grant and §28 termination; D247 Network/service
orthogonality; D249 Resolver independence; D250 child Process authority remains
**unselected**; no imported-module, Actor, P or Tool implicit grant; no OS sandbox
claim, no invented crash-durability semantics. No existing approved invariant is
reopened. The implementation-specific delta is a Linux kernel-backed security
substrate with isolated I/O execution and JNI instead of pathname/mountinfo
authority checks; it introduces no Protos-visible semantics.

**Decision approval ≠ verified implementation.** I156 owns the proof,
falsification, packaging, testing and correction; I155 continues to own the
complete D248-B product acceptance. The design may be durably recorded as
RATIFIED while both implementation issues remain open. Do not close any
implementation issue, report security PASS, or release code before the human
reports required tests and a real Native run.

**Coordination remains separate:** GitHub-native PLAT072→I155 and I156→PLAT072
parent links and dependency edges must be verified under GITHUB006/GITHUB015.
A published decision document does not invent those relationships or establish
a software implementation.

## Primary sources

- [Linux `openat2(2)`](https://man7.org/linux/man-pages/man2/openat2.2.html):
  actual selector, `RESOLVE_NO_XDEV`, bounded retry for uncertain resolution.
- [Linux `landlock(7)`](https://man7.org/linux/man-pages/man7/landlock.7.html):
  per-thread inheritance, access rights, `REFER`/truncate ABI, preopened
  descriptor caveats, LSM availability and fail-closed requirements.
- [Linux `renameat2(2)`](https://man7.org/linux/man-pages/man2/rename.2.html)
  and [`unlinkat(2)`](https://man7.org/linux/man-pages/man2/unlink.2.html).
- [Java `SecureDirectoryStream`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/file/SecureDirectoryStream.html)
  and [GraalVM Native Image JNI](https://www.graalvm.org/latest/reference-manual/native-image/dynamic-features/JNI/).
