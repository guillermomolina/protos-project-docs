# D248 — Explicit confined Filesystem capability for standalone Protos CLI

**Status: RATIFIED — owner-approved Candidate B + shell-session future constraint, 2026-11-07.** This record selects a CLI/host authorization contract. It is **not** evidence that the Filesystem grant, recursive writable backend or JVM/Native support has been implemented or tested.

- Decision: [D248/#1074](https://github.com/guillermomolina/protos/issues/1074).
- Implementation: [I155/#1079](https://github.com/guillermomolina/protos/issues/1079), a native sub-issue of D248; no implementation or validation claimed here.
- Future independent design gate: [D250/#1080](https://github.com/guillermomolina/protos/issues/1080), **unselected**.
- Motivation/adoption: [I153/#1068](https://github.com/guillermomolina/protos/issues/1068) and future [INC002/#1004](https://github.com/guillermomolina/protos/issues/1004).
- Independent ratified peer: [D247/#1073](https://github.com/guillermomolina/protos/issues/1073) (Network and service CLI). Independent **unselected** peer: [D249/#1075](https://github.com/guillermomolina/protos/issues/1075) (Resolver CLI).
- Full comparative investigation, GITHUB010 scoring, falsification and GITHUB021 audit: [D248 ratification evidence](../../evidence/D248/D248_B_OWNER_APPROVAL_RESEARCH_AND_INVARIANT_AUDIT_2026_11_07.md).
- Inspected Protos product revision: [`3791394c46aff8ad7508f64e56cde4594e502a56`](https://github.com/guillermomolina/protos/commit/3791394c46aff8ad7508f64e56cde4594e502a56) (2026-11-07). **Evidence baseline, not a revision implementing D248.**

## Selected minimal CLI form and eligibility

```text
protos --allow-filesystem=/home/user/appdata app.protos
protos --allow-filesystem=./data -e '...'
protos --allow-filesystem=./data                  # REPL
protos run --allow-filesystem=./data app
protos debug --allow-filesystem=./data app.protos
protos --allow-filesystem=./data --service=serve server.protos
protos run --allow-filesystem=./data --service=serve app
```

- **Exactly one** `--allow-filesystem=<directory>` is a standalone-CLI application opt-in, not a general `--allow-all` or ambient permission. The value is a nonempty host-native directory path, absolute or relative; a relative value is resolved from **invocation-time host working directory** before guest execution, not from source/package root or package metadata. The root must already exist; no implicit creation.
- The selected host root is securely opened/pinned as the authorized directory. Reject a final symlink as the grant root, and reject any root/provider/platform for which the intended pinning or confinement cannot be proved. A startup failure must occur **before** guest source expressions run. A root path spelling is an input to host provisioning, not a guest `Path`.
- The initial RootActor receives one standard Core `Filesystem` capability at its bootstrap-local initial `moduleContext.filesystem` slot. Without the flag, the slot is **absent**, not present with `null`. A flag grants nothing to imported modules, Actor children, P workers, unrelated contexts, or host OS subprocesses; ordinary explicit authorized use/delegation remains constrained by the existing normative protocols.
- The opt-in applies only to ordinary **application** entry: direct SOURCE, `-e`, REPL, `protos run` MODULE, application `debug`, and a D247-selected RootActor message-service entry. `--service` **alone** grants nothing. Language server and bundled Tool commands (`test`, `format`, `lint`, `package`, etc.) reject it and keep their separate, narrowly provisioned host filesystem custody.
- Parse only in the application's pre-entry option region (or after `run`/`debug` but before the target); subsequent tokens are application arguments with the existing exact `process.args()` snapshot. Retain independent `--inspect`, `--foreign-provider-path`, `--allow-network` and `--service` parsing. Duplicate, empty, malformed, or ineligible usage is exit **2**. Provisioning failure such as unavailable/unsecure root is an application bootstrap failure before execution (proposed guest-failure exit **1**); internal faults preserve the ordinary CLI error convention. No unapproved automatic fallback to no grant or a broader host directory.

## Authorized Filesystem namespace and operations

The capability represents **one recursive directory namespace** confined to the securely provisioned host root. The guest uses **existing** `Path.relative()` and `child(name)` component semantics: no absolute host path strings, parent traversal, or host paths inferred from the package or source file.

Within the authorized namespace, preserve the complete existing standard `Filesystem` protocol rather than substituting a direct-child staged-file subset:

| Existing operation | Authorized behavior and boundary |
| --- | --- |
| `filesystem.open(path, read:, write:, create:, createNew:, truncate:)` | Valid normative combinations for existing reads, writes, creation/exclusive creation, truncation; each open preserves race-free selection, stable resource binding, cancellation and effect-commitment semantics |
| `filesystem.entries(path)` | Direct children of the **selected in-root directory**; may be used recursively by selecting descendant directory Paths, without automatic recursive listing |
| `filesystem.replace(source, target)` | Existing atomic namespace-replace semantics under the confined root; no expanded generic rename protocol |
| `filesystem.remove(path)` | Existing entry-only removal semantics; not recursive directory deletion |

**No** new `mkdir`, general `mv`/rename, host-global current directory, multiple mounts, per-operation prompt, implicit File durability guarantee, directory-sync guarantee, rollback, or transaction facility is selected by D248. These would require their own design if needed; the future shell's illustrative `mv` is not proof that Protos currently has a general rename operation.

Confinement is an **invariant**, not a best-effort path normalization rule:

1. Every intermediate directory traversal and final open/observation must remain relative to secure pinned directory handles without symlink following through the target; escaping via links, aliases, mount transitions, provider virtualizations or rename races must be denied rather than silently accepted. For `replace`/`remove`, the selected final entry itself may be a symlink, but the operation must act on that **entry**, never the outside referent.
2. Preserve existing atomic `create`/`createNew`, `truncate` and `replace` commitments; a `Files.exists` check followed by an unrestricted `Files.newByteChannel` is not an equivalent implementation. A concurrency race must not change the grant's authority.
3. The implementation must actually prove any claimed security property on the supported JVM/Native platforms and providers. If a host filesystem, platform or runtime cannot enforce the contract, **fail closed** at provision/use rather than loosening standard `FILESYSTEM.md` guarantees. Directory grants are not represented as a universal OS sandbox against arbitrary native code, hardlinks, privileged host mutation, or unrelated OS children.
4. Process bootstrap/session ownership retains custody until real termination. End of a one-shot source is not equivalent to a D247 service termination; do not close its backend immediately after service initialization. Cleanup/revocation follows existing `PROCESS_IO.md` §28 and never implies automatic flush, sync, durable directory entry or graceful HTTP drain.

## Explicit future-shell invariant — approval is deliberately narrow

The owner asked whether a future Protos shell replacing Bash would need repeated authorization for every `cp`. The owner approved the resulting **modified recommendation** on 2026-11-07:

> A host-authorized Protos shell should hold capabilities for the shell session and reuse them for ordinary in-Process commands. Authorization is attached to the capability/hosting scope, **not** requested independently for every filesystem operation or command.

This is compatible with D248-B: the shell can pass its already-held Core `Filesystem` explicitly to in-Process modules/functions. It does not require a new grant for each ordinary `cp` or `ls`-style function call. A future shell may track a command-local working directory as ordinary shell state, without silently creating a mutable `Process`-global CWD.

The separate question of creating **new Protos Processes** and provisionally offering a narrowed child view is **D250**, **OPEN/UNSELECTED**. The design must never amplify beyond the parent/session's effective host authorization and must keep provisioning explicit; **no guest spawning API, automatic inheritance, attenuation constructor or OS subprocess confinement is selected now**. This avoids forcing session/child architecture into the simple D248 application CLI. External OS executables remain governed by unselected [D229/#1008](https://github.com/guillermomolina/protos/issues/1008).

## Selected/rejected alternatives

- A — no standalone Filesystem grant; embedding only: fully conservative but cannot support ordinary installed Protos applications needing persistent data without a custom host.
- **B — one confined recursive full-protocol `--allow-filesystem=<directory>` grant: SELECTED**; directly meets persistent application data needs while preserving existing guest capability semantics and authorization locality.
- C — separate read-only/write-only CLI permissions now: finer least privilege, but commits current CLI users and backends to an unrequired policy matrix; a later explicit narrower grant is not ruled out.
- D — implicit app-managed data directory selected by package/manifest or runtime policy: convenient, but risks unauthorized ambient filesystem scope and conflating package resolution with host grants.
- E — external named/mounted capability only: good for managed embedding, insufficient as a simple user-facing installed CLI path. May coexist in later hosting designs.

The strongest objection is that a **recursive writable confined filesystem** meeting all race/mount/alias/Native guarantees is materially harder than parsing a CLI flag; the existing direct-child writable and recursive read-only NIO backends are not an implementation. D248 retains correctness and fail-closed behavior over permissive, unproven platform support. Implementation and genuine Native verification belong to I155.

## Authority, reversibility, and publication limits

Governing norm: `spec/io/FILESYSTEM.md` (all `Path`, `File`, open, entries, replace/remove and confinement semantics); `spec/io/PROCESS_IO.md` (initial RootActor slot, Process custody/absence), `spec/io/IO_CORE.md` (errors/lifecycle), `spec/concurrency/ACTORS.md` (no automatic Actor transfer), `spec/concurrency/PARALLEL_EXECUTION.md` (P authority boundary), `spec/semantics/MODULES.md` (SOURCE/MODULE separation). This decision does **not** edit those normative owners, weaken them or assert an implemented backend.

**GITHUB021 invariant check:** passes for the approved surface: D247 network/service orthogonality, independent D249 Resolver, D241-B1 `Filesystem.open` semantics, Process-local grant/Actor and import isolation, separate Tool custody, no implicit host effects, and the explicit new shell session reuse constraint. No known owner-approved invariant is reopened or displaced; the new public host CLI option and recursive capability are the owner-approved delta. See the linked evidence document for full checks and uncertainty.

**Completion of D248 is decision closure only.** I155 implementation remains independently open, with no test, build, Native Image, or Protos product commit claimed by this document.
