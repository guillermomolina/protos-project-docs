# D248 — Candidate B research, owner approval, falsification and GITHUB021 audit

**2026-11-07 — RATIFICATION EVIDENCE**, not product implementation, independent platform certification, build/test result or approval of shell process-launch semantics.

- Decision: [D248/#1074](https://github.com/guillermomolina/protos/issues/1074).
- [Ratified D248-B contract](../../decisions/tooling/D248_EXPLICIT_CONFINED_FILESYSTEM_CLI_RATIFIED.md).
- Implementation: [I155/#1079](https://github.com/guillermomolina/protos/issues/1079); native child relation to #1074 was independently verified after allocation.
- Separate future design investigation [D250/#1080](https://github.com/guillermomolina/protos/issues/1080), **unselected**; [INC002/#1004](https://github.com/guillermomolina/protos/issues/1004) and [D229/#1008](https://github.com/guillermomolina/protos/issues/1008) remain distinct.
- **Exact inspected `guillermomolina/protos` baseline:** [`3791394c46aff8ad7508f64e56cde4594e502a56`](https://github.com/guillermomolina/protos/commit/3791394c46aff8ad7508f64e56cde4594e502a56), committed 2026-11-07 07:32:45Z. This is a source-inspection identity, not a D248 implementation commit. Current `protos-project-docs` baseline before writing D248 records was `e250bc9392ad75b9a11f143a5580519b5ea710e1`.

## 1. Exact owner approval and the revised recommendation

The initial [D248/#1074](https://github.com/guillermomolina/protos/issues/1074) issue was explicitly **investigation-only**. After examining Protos spec, current host and CLI code, and other systems, the assistant recommended **Candidate B**: a single explicit `--allow-filesystem=<directory>` for an ordinary, recursively confined, read-write Core Filesystem capability, with no ambient authority, using the current normative `Filesystem` operations and failure semantics. This was expressly a proposal, not ratification.

The owner then tested the recommendation against a future Protos shell: **«imagina que un dia tienes un shell protos en lugar de bash, como lo harías? tendrías que dar permisos con cada cp que quieras ejecutar?»**. The answer distinguished **in-Process commands/functions**, which may use the shell's already-granted Filesystem without further host authorization, from **new Protos Processes**, whose host-authorized explicit and potentially attenuated capability provisioning is future work. It explicitly retained B for CLI, avoided per-command permission prompts, and separated a future provisioner from D248.

The owner then stated **«ok apruebo esta nueva recomendación»** and explicitly directed updates to Issues and durable evidence in `guillermomolina/protos-project-docs`, with a **bounded commit/push exception for that repository only**. This is **exact candidate approval** of B plus the future-shell **constraint/separation**, not approval of an implemented or specified API for child Process creation, attenuated authority, shell syntax, OS process launching or broad permissions.

The issue/evidence records are agent-written documentation of approval **in this active conversation**; they are not represented as an owner-authored GitHub comment.

## 2. Current Protos evidence and normative owners

The following files were inspected under `guillermomolina/protos`, including current product SHA rechecks for principal normative and implementation paths:

| File | Observed constraint / fact |
| --- | --- |
| `AGENTS.md` | Normative specs own semantics, explicit D owner approval, human-executor for product code/tests, bounded GitHub coordination exception |
| `AGENTS.work/DESIGN.md` | GITHUB010 exhaustive multi-system/candidate research and 12-axis scoring; GITHUB021 invariant/delta preservation |
| `AGENTS.work/COORDINATION.md` | GITHUB006 native hierarchy, GITHUB015 closure requires exact durable publication, family/status and collision-safe allocation |
| `AGENTS.work/REFERENCE.md` | DOC002 role-first durable decisions under `protos-project-docs/docs/project/decisions/tooling/`, evidence under `docs/project/evidence/D248/` |
| `spec/io/FILESYSTEM.md` | Core Filesystem is explicit authority; Path is a component-based value, not host path authority; `open`, `entries`, `replace`, `remove` and confinement/atomicity semantics already normative; no general `mkdir` or general rename |
| `spec/io/PROCESS_IO.md` | Initial RootActor receives `filesystem` only when host explicitly grants it; absent otherwise; module imports, `Actor.spawn` and Process capability do not confer filesystem authority; Process custody/termination owns revocation |
| `spec/io/IO_CORE.md` | Cross-capability, ordinary I/O failures, Future/cancellation/commitment and resource ownership; no implicit global authority or OS child execution |
| `spec/concurrency/ACTORS.md`, `spec/concurrency/PARALLEL_EXECUTION.md` | Nontransferability/no automatic Actor/proxy authority and no automatic P-safe host I/O |
| `spec/semantics/MODULES.md` | SOURCE/MODULE distinction and import resolution are separate from initial RootActor bootstrap grants |
| `src/main/java/com/guillermomolina/protos/cli/ProtosCli.java` | Distinct source, eval, REPL, package run, tool, debugger routes; current CLI has no D248 grant path; internal tool scopes read source/data with separately captured authority |
| `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosStandaloneHostedExecution.java` | Host session/bootstrap seams already convey Network and Resolver selections independently; Filesystem is not automatically injected in CLI application bootstrap |
| `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosStandaloneProcessBootstrap.java` | Default optional `ProtosFilesystemValue` parameter exists; using it requires distinct provisioning plus ownership |
| `src/main/java/com/guillermomolina/protos/execution/workspace/ProtosWorkspacePackageApplicationExecution.java` | Package app bootstrap and ownership must be wired without leaking package-tool filesystem grants |
| `src/main/java/com/guillermomolina/protos/execution/host/io/ProtosNioConfinedFilesystemBackend.java` | Existing `SecureDirectoryStream` confined **direct-child**, staged-write behavior and atomic entry mutation, not a general recursive writable tree |
| `src/main/java/com/guillermomolina/protos/execution/host/io/ProtosNioReadOnlyTreeFilesystemBackend.java` | Recursive `SecureDirectoryStream` no-follow walk, but **read-only** |
| `src/main/java/com/guillermomolina/protos/execution/host/io/ProtosNioReadOnlyFilesystemBackend.java`, `ProtosEmbeddedFilesystemCustody.java` | Existing direct-file and provider-supplied embedding authority is not a CLI-visible unrestricted guest filesystem |
| `src/test/java/.../ProtosNioConfinedFilesystemBackendTest.java`, `ProtosNioReadOnlyTreeFilesystemBackendTest.java`, `ProtosEmbeddedFilesystemTest.java`, `ProtosCliTest.java` | Relevant existing security/embedding/CLI regression anchors; their existence is **not** evidence that I155 tests passed |
| `bin/protos` | Installs/launches Native when available; no JVM-only implementation can be presumed equivalent |

Further directly relevant published authority: [D247-C ratification](../D247/D247_C_OWNER_APPROVAL_RESEARCH_AND_INVARIANT_AUDIT_2026_11_07.md), [D229/#1008](https://github.com/guillermomolina/protos/issues/1008), [INC002/#1004](https://github.com/guillermomolina/protos/issues/1004), and [D249/#1075](https://github.com/guillermomolina/protos/issues/1075).

## 3. Cross-system prior art — mechanisms rather than borrowed syntax

At least **nine** distinct implementations/specifications across **permission-oriented runtime**, **capability-hosting** and **OS/provider containment** approaches inform the choice:

| System / authoritative reference | Observed mechanism or instructive limit | Protos-specific lesson |
| --- | --- | --- |
| [Deno permissions](https://docs.deno.com/runtime/fundamentals/security/) | Runtime permission grants for file/network effects, with permission scopes; symlink and host-path interpretation must be considered rather than treating spelling as authority | Explicit CLI opt-in is ergonomic; Protos uses a concrete `Filesystem` capability, not a global path-permission check |
| [Node.js permission model](https://nodejs.org/api/permissions.html) | Permission flags are not a complete adversarial OS sandbox; limitations on path/links and other execution mechanisms matter | Do not overclaim confinement by calling a check a grant |
| [WASI preopened directory model](https://github.com/WebAssembly/WASI) | Host preopens directory capability and hands guest handles/rights, rather than offering unrestricted process-wide paths | Confined root and capability custody fit Protos first |
| [Rust cap-std](https://docs.rs/cap-std/latest/cap_std/) | Directory-relative authority and operations, avoiding accidental ambient filesystem access | Host directory-handle-relative operations are safer than checking absolute strings |
| [OpenBSD unveil(2)](https://man.openbsd.org/unveil.2) | Process-level host path restrictions by operation class; different granularity than guest object capabilities | Split read/write authorization is possible but not needed to make first ordinary data app usable |
| [GraalVM Polyglot IOAccess / FileSystem](https://www.graalvm.org/latest/reference-manual/embed-languages/) | Host decides filesystem access and may provide a custom provider; context access need not equal host-global Filesystem | Preserve explicit host authorization, per-Context ownership and guest-visible capability absence |
| [Java SecureDirectoryStream](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/nio/file/SecureDirectoryStream.html) | Secure directory-handle-relative operations supported conditionally by provider, no-follow selection | Reuse secure handles and fail closed when provider/platform cannot prove semantics |
| [Linux openat2(2)](https://man7.org/linux/man-pages/man2/openat2.2.html) | `RESOLVE_BENEATH`, `RESOLVE_NO_SYMLINKS`, `RESOLVE_NO_XDEV` can enforce selected path resolution policy where available | Strong native substrate is possible, but Linux-specific flags cannot silently become portable JVM/Native promises |
| [Docker bind mounts](https://docs.docker.com/engine/storage/bind-mounts/) | An operator grants a directory into another namespace, with distinct containment/symlink/mount concerns | Directory root is a useful operator grant; do not conflate mount admission with Core Filesystem security |

Windows junction/reparse points, hardlinks, bind mounts, nested mounts and provider-specific path behavior are material counterexamples; a host implementation must verify its actual substrate on each supported platform. These sources provide **prior art**, never normative Protos semantics. No external system proves I155's Native viability.

## 4. Complete candidate set and GITHUB010 12-axis evaluation

A — **No CLI grant** (embedding-only); B — **one explicit recursively confined full-protocol read-write grant**; C — **separate read/write flags**; D — **managed/automatic application-data directory**; E — **external named host-capability only**. Variants rejected as unauthorized: `--allow-all`, package manifest autoauthorization, source-file directory inferred authority, one grant per filesystem command, and bypassing the Core guest API through unrestricted Java/native interop.

Scores **1 (poor)–5 (strong)**; suffix **H/M/L** = HIGH/MEDIUM/LOW confidence. Scores express architecture and available evidence, **not measured performance or pass rates**.

| GITHUB010 dimension | A | **B selected** | C | D | E |
| --- | ---: | ---: | ---: | ---: | ---: |
| 1. Correctness / invariant preservation | 5H | 4M | 4M | 2H | 4M |
| 2. Protos alignment | 3M | 5M | 4M | 2H | 4M |
| 3. Present-need proportionality | 1H | 5M | 3M | 3M | 2M |
| 4. Incremental growth | 2M | 5M | 4M | 2M | 4M |
| 5. Future-option resilience | 3M | 5M | 4M | 2M | 5M |
| 6. Scalability | 2M | 4L | 4L | 3L | 4L |
| 7. Conceptual simplicity | 5H | 4M | 2M | 3M | 2M |
| 8. Portability / implementation freedom | 5M | 3L | 3L | 3L | 4M |
| 9. Runtime / resource cost | 5H | 4L | 3L | 3L | 3M |
| 10. Failure / operability | 3M | 4L | 4L | 2M | 3M |
| 11. Deferral / reversibility | 2H | 5M | 3M | 2M | 4M |
| 12. Evidence maturity / implementation risk | 5H | 2L | 2L | 2L | 3M |

**Per-axis short rationale for all candidates:**

1. **Correctness:** A grants nothing (safe but insufficient); B/C can honor normative rooted authority only with a new proof-grade backend; D risks implicit authority; E preserves a host-selected source but lacks a usable CLI grant.
2. **Protos alignment:** B preserves one existing ordinary Core `Filesystem` with explicit host scope; C adds user-visible permission modes now; A prevents standalone use; D makes packages/runtime policy an authority source; E elevates host capability names into the first consumer path.
3. **Present need:** A and E leave installed persistent apps without a simple launch; B needs just one requested directory; C policy machinery is extra work; D creates additional data-location conventions.
4. **Growth:** B admits later optional narrower grants, multiple roots or shell hosting without redefining Core semantics; C already owns policy expansion; E can compose provider policies; A/D defer or tie growth to extra host conventions.
5. **Future choices:** B/E keep host provisioning decoupled from guest behavior; C prematurely hardens a public read/write permission language; A has fewer real use cases; D makes a default application directory or manifest meaning harder to change.
6. **Scalability:** B/C/E are handle-local across many Processes in principle but must solve concurrent name mutation; A cannot scale CLI filesystem apps; D conflates data lifetime with package/launcher evolution.
7. **Simplicity:** A has no new code; B one explicit flag and existing guest protocol; C read/write interactions, E names/provisioner, D package/working-dir convention add competing concepts.
8. **Portability:** A works everywhere by doing nothing; B/C need real recursive secure R/W provider and actual Native proof, so LOW; E delegates substrate selection to host; D does not avoid host path/security portability.
9. **Cost:** A has no runtime work; B is pay-for-use with root/handle work on opt-in only; C more policy checks, D more provisioning/canonicalization, E external host management; platform cost uncertainty is LOW confidence for B–D.
10. **Operability:** B/C can cleanly reject unauthorized roots and absent flags if backend fail-closes; E depends on named host deployment; D has hidden default path errors; A has clear denial but no application path.
11. **Regret/rewrites:** Without B, application data needs continued custom embedding; B can acquire scoped variants independently; C and D create public policy/data-location compatibility; E remains a later option without current CLI usability.
12. **Maturity:** A exists as current behavior; E has strong host-capability precedents; B/C/D's complete recursive writable JVM+Native confinement is **unproven**, LOW; no scores are a substitute for I155 tests.

**Selection is not arithmetic.** B is the smallest adequate **user-facing** policy that meets an existing persistent-app requirement while preserving normative authority locality. Its main weakness is implementation feasibility, which becomes an explicit fail-closed implementation gate rather than a silently weakened contract.

## 5. Adversarial falsification and hard blockers for I155

| Attack / failure attempt | Required observation, not a claimed test result |
| --- | --- |
| No `--allow-filesystem`, including service entry and package `run` | Root `moduleContext` has no `filesystem` slot; Tool source access does not leak to guest |
| Relative grant rooted at source/package rather than invocation CWD | Must **not** happen; resolve from invocation CWD once before guest launch |
| Final-root symlink or root replaced between validation and open | Secure pinning or denial; no check-then-follow race |
| `Path` parent/absolute/host-native bytes or malicious child | Cannot escape capability namespace; reject semantic violations |
| Intermediate/final symlink inside recursive tree | Never follow target to outside; symbolic-link entry mutation differs from following target |
| Bind mount, junction, alias, hardlink, untrusted provider | Do not assert general OS sandboxing; enforce approved namespace semantics or reject uncertain backend/platform; document residual OS limitations |
| Concurrent `createNew` of same name | Exactly normative exclusive selection outcomes, not probe-then-create both succeed |
| Concurrent rename/symlink swap during nested open | Stable selected file resource; no host directory escape |
| `replace` between directories / symlink final entry | Existing atomic semantics; act on final entry with no referent escape |
| `truncate` cancellation or failure | Existing effect/commit boundary; invalid combos fail preflight without effects |
| `Filesystem.entries` of descendant dir | Direct children of that descendant only; don't dereference an external symlink |
| Combined Network-only / Filesystem-only / Resolver absent | No implicit cross-grants; D247's `--service` remains grant-neutral |
| Import, `Actor.spawn`, P, bundled Tool | No accidental bootstrap capability inheritance |
| One-shot vs persistent RootActor message service | File resource and backend custody live through genuine Process lifetime; no early `finally` teardown |
| Restart with created content | Persistence of actual data across separate JVM and Native Process runs, not merely in-memory backend |
| Genuine installed Native executable | Real R/W, nested operations, errors and security negatives; do not equate Java source presence or image build with passed Native behavior |

**Strongest objection:** a recursively writable, adversarially confined directory on all JVM and Native platforms may require secure directory handles, provider restrictions, unsupported-filesystem rejection, and OS-specific secure primitives. Hardlinks/mount aliases and unrelated privileged OS processes can invalidate a naive “safe pathname” story. Merely reusing the existing direct-child writable backend would **not** satisfy D248-B. If the supported substrate cannot meet the approved guest contract, I155 must remain unverified/blocked and surface the precise feasibility gap for design review; it must **not** silently broaden access.

## 6. Pay for what you need, incremental extension and future stress

- **Current user need:** a normal installed Protos app keeps ordinary data under one user-selected directory. B grants the full already-normative Filesystem API without requiring Java embedding; no grant means no extra guest capability/host work.
- **What is intentionally deferred:** read-only flags; per-operation rights ACL; disjoint roots; dynamic attenuation or Process child spawning; shell syntax, `mkdir`, generic `mv`, OS subprocess sandboxing, full file-system virtual mounts, ambient CWD, journaling/transactions/durable namespace sync. These are not needed to authorize the existing Core operations inside a single app-data directory.
- **Explicit future-shell stress:** a session-scoped host Filesystem supports repeated in-Process `cp`/listing calls via ordinary capability possession, without prompts. Shell commands executed as *separate Protos Processes* would need a future host-authorized child-provisioning contract; **D250** investigates that independently, while **D229** investigates *OS executables*. Do not claim a current implementation or leak a session capability by Actor inheritance.
- **Regret:** a later shell may need attenuated child roots, multiple mounts, and separate read/write rights. **Escape:** introduce an independently approved child provisioner and optional narrower host grant without redefining the already-stable Core `Filesystem` object or weakening `--allow-filesystem` meaning. An overbroad initial root is an operator decision, not grounds to inject per-command prompts.
- **Defer-cost:** if B were omitted, each application needing persistence would still require host embedding or a custom launcher; if the whole shell/process manager were built now, every current Protos CLI user would pay speculative semantics/maintenance before that workload exists.
- **Portability:** Native Image and JVM must implement identical guest contract **where supported**, but inability to prove secure confinement on a given provider is a controlled denial, not an implicit wider grant. Platform-specific implementations remain permitted behind a stable host boundary.

## 7. GITHUB021 invariant/delta consistency review

| Prior approved/normative constraint | B + approved future-shell consequence | Result |
| --- | --- | --- |
| Existing Filesystem Core protocol and D241-B named `open` | No new guest Filesystem API or change to capture/preflight/create/truncate/replace/remove semantics | PASS |
| Filesystem `Path` namespace confinement | Root selected by host; no raw guest host-native path syntax; secure no-follow path selection and fail-closed provider | PASS (implementation unverified) |
| `PROCESS_IO.md` explicit bootstrap-local `filesystem` | New CLI option selects grant only for initial RootActor; absent otherwise | PASS |
| D247-C independent `--allow-network` and `--service=<binding>` | Filesystem option is orthogonal; service selection alone grants no filesystem | PASS |
| D223-A1 and D249 separation of Resolver | Filesystem permission does not supply DNS or Network | PASS |
| SOURCE/MODULE D242 and package tool custody | No automatic path grant from source/package; no private module export or Tool authority leak | PASS |
| Actor/P and imported module isolation | No implicit propagation; in-Process module/function can receive existing capability explicitly | PASS |
| Process termination and I/O ownership | No new implicit flush, namespace durability, Actor transfer or service drain | PASS |
| D229 external OS execution remains unapproved | Filesystem grant does not spawn or sandbox OS subprocesses | PASS |
| Owner's explicit future shell requirement | Session-scoped grant reused for ordinary in-Process commands; no per-command reauthorization | PASS |
| Separate future child Process provisioning | D250 exists only for investigation; no Process constructor, grant attenuation API or inheritance selected | PASS |

**Delta surfaced to and approved by owner:** one opt-in confined CLI Filesystem grant plus a future-shell lifetime/separation constraint. **No** already approved invariant is knowingly displaced. Any newly discovered inability to provide recursive R/W confinement, hardlink/mount isolation, or Native parity must be reported as implementation/design evidence rather than a claim that approval permits semantic degradation.

## 8. Publication / validation claims and next gates

- **Research/coordination only:** no Protos source edits, normative spec edits, builds, tests, Native runs, releases or product commits were performed in D248 ratification.
- I155 must publish source and validation results later, record the exact tested product commit and actual Native binary identity; then publish any required implementation evidence separately.
- D248 decision closure needs exact revision-bound publication of this document and the decision record in `guillermomolina/protos-project-docs`, native child I155 linkage, Issue family/status/priority checks, and final closure comment.
- D250 remains an **open, unselected** semantic checkpoint; its existence is not evidence that child Process provisioning or shell command syntax is implemented.
