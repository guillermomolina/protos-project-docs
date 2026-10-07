# TOOL001-F2E — External Immutable-Package Execution

Status: **CLOSED — F2E1/F2E2/F2E3/F2E4/F2E5 complete; public exact external run closure published**
Nature: non-normative Package Tool / host-integration project record
Allocated after: `TOOL001-F2D` workspace-only execution closure

## Purpose

`TOOL001-F2D` closed a complete workspace-only normal-execution subset. Registry
and Git nodes remain present in canonical lock-format 1 but are still rejected by
execution preflight.

F2E owns the next bounded continuation: make an **already selected and already
materialized immutable external node** executable only after the exact locked
content has been verified and converted into inert execution-plan data.

F2E does not own version discovery, fresh selection, update, registry protocol,
network transport, credentials or package publication.

## Mandatory inherited invariant

External source must never become executable merely because a host path, cache
directory, registry locator, Git URL, revision or matching PackageId was found.

Before an external source root enters `PackageExecutionPlan`, all of these must
hold:

```text
exact locked node identity
exact materialized source root
package-store confinement
recorded ContentIdentity verification
no ambient cache/store visibility
no version solving
no lock mutation
```

Registry and Git nodes therefore remain fail-closed until F2E reaches its final
integration slice.

## Fresh audit findings

### ContentIdentity tree canonicalization was open; F2E1 now closes it

`docs/design/PACKAGE_IDENTITY_VERSIONING.md` defines ContentIdentity as the
algorithm-tagged digest of a versioned canonical logical package tree and gives
the illustrative method name `protos-package-tree-v1`.

That record explicitly leaves the exact tree algorithm to a focused
artifact-format audit. The future contract must define included files/manifest,
canonical relative path encoding, separator/case rules, symlink and special-entry
treatment, semantic/excluded metadata, deterministic bytes, resources and
machine-local exclusions.

Therefore no external store directory can yet be truthfully checked against a
locked ContentIdentity. An ad-hoc recursive host hash would silently invent
package semantics.

### Current Core Filesystem does not expose portable tree observation

The current standard Filesystem surface is intentionally narrow:

```text
open
replace
remove
```

It does not standardize broad tree observation such as directory enumeration,
stat/type queries, directory creation, symlink inspection or recursive traversal.

F2E must not bypass this with package-only host calls such as
`PackageNative.walkTree`, `hashDirectory` or ambient cache scanning.

After F2E1 freezes the exact observations required by
`protos-package-tree-v1`, F2E2 must re-audit the then-current general capability
surface. If portable semantics are still missing, record the proper general
Filesystem prerequisite rather than hiding it in Package Tool Java.

### Fetching is not normal execution

Normal execution consumes an already-selected exact graph. Acquisition is a
different authority boundary.

F2E scopes first to **already materialized** external content. Missing exact
content fails. Normal run must not silently resolve, query a registry, follow a
mutable Git ref, download, mutate the lock or write the store.

A later explicit `fetch` path may consume the exact lock under separately
designed store-write/network authority.

## Cost-aware decomposition

```text
F2E   external immutable-package execution                         CLOSED
F2E0  prerequisite audit + decomposition                           CLOSED
F2E1  protos-package-tree-v1 ContentIdentity contract              CLOSED
F2E1A logical-tree domain + portable path/entry-kind contract       CLOSED
F2E1B canonical byte stream + method/hash contract                  CLOSED
F2E1C independent conformance vectors + F2E1 closure                CLOSED
F2E2  verified read-only package-store binding                     CLOSED
F2E2A  captured-Filesystem ContentIdentity canonicalizer/verifier  CLOSED
F2E2B  exact selected-root capture + verified-capture host custody CLOSED
F2E2C same-capture integration + F2E2 closure                      CLOSED
F2E3  external-node execution-plan construction                    CLOSED — F2E3A/B/C; D053/D056/D057 RATIFIED
F2E4  external canonical ModuleKey + source resolver               CLOSED — PLAT012 RATIFIED
F2E5  public run integration + F2 external-execution closure       CLOSED — private exact local backend + public CLI cutover published
```

### F2E1 — canonical logical package tree

Freeze the exact `protos-package-tree-v1` contract before implementing
verification. Independent implementations must agree on the same digest for the
same logical package tree.

The audit must explicitly cover regular files, directories/empty directories if
semantic, symlinks/special entries, relative-path encoding/order, file bytes,
manifest/resources, ignored/generated/VCS content and concurrent mutation.

It must also freeze the persisted method/algorithm/digest relationship and
compatibility behavior for future methods/algorithms.

### F2E2 — verified read-only store binding

After E1 fixes required observations, design/implement the minimum general
capability path that can bind one **already-selected** store entry, confine
traversal to store authority and verify exactly the locked ContentIdentity.

No ambient scan by PackageId/name/version is allowed.

### F2E3 — external execution-plan construction

D053 (`docs/project/decisions/tooling/D053_PACKAGE_EXECUTION_PLAN_ABI_EVOLUTION.md`) is now **RATIFIED**. F2E3 is READY and must consume that decision mechanically: generation 1 remains the exact workspace-only ABI; external-capable planning uses generation 2 as one mixed inert graph with typed compact refs, one ContentIdentity per immutable external package node and uniform dependency edges. Provenance, paths, custody, resolver authority and host handles remain outside the plan. F2E3 must not reopen those ratified choices.

Extend Protos-owned preflight only after E2 returns verified immutable material.

Registry identity must preserve at least:

```text
PackageId
exact ReleaseVersion
ContentIdentity
```

Git identity must preserve at least:

```text
PackageId
exact revision
ContentIdentity
```

Retrieval locator, registry locator, authority endpoint, cache/store path and
dependency alias remain provenance/lookup data.

### F2E4 — host resolver extension

Defensively detach the extended plan and mechanically bind verified external
source roots.

Canonical immutable external module identity is the exact immutable package
instance plus internal logical module. Mirrors/store paths/aliases never enter
ModuleKey identity.

### F2E5 — public run integration and F2 closure

Extend the published `protos run <entry> [args...]` path to mixed
workspace/external exact graphs without changing the CLI entry convention.

F2E5 may close F2 only when missing/corrupt/mismatched external material fails
before application authority begins, normal execution performs no solving/fetch
or lock mutation, and store verification authority does not leak into the
application Process.

PLAT048 is now RATIFIED as Candidate B′. F2E5 is implementation-ready with the
following fixed host/package boundary:

~~~text
Package Tool
    -> derive inert complete exact external requirements

public-run host bootstrap
    -> own a run-scoped exact materialization provider
    -> lookup only by
       (kind, PackageId, exact version/revision, ContentIdentity)
    -> obtain one already-present local root

selected root
    -> unchanged F2E2 capture/verify
    -> unchanged F2E3 V2 planning
    -> unchanged F2E4 detach/reconcile/resolution
~~~

Workspace-only graphs remain on the existing generation-1 public-run path.
External graphs use generation 2. The implementation must not add a CLI flag,
Protos-specific environment variable, public configuration surface, ambient
store scan, canonical public store layout, fetch, store writes, network,
credentials, GC, vendor policy, solving or lock mutation.

The ratified decision is
`docs/project/decisions/platform/PLAT048_PUBLIC_RUN_EXTERNAL_MATERIALIZATION_AUTHORITY_BOUNDARY.md`;
its investigation/approval evidence is
`docs/project/evidence/PLAT048/PLAT048_PUBLIC_RUN_EXTERNAL_MATERIALIZATION_DECISION_EVIDENCE.md`.

#### F2E5 exact-requirements publication checkpoint

The first bounded F2E5 implementation is published at
`guillermomolina/protos@eff751f7cde2009feb9b91bc5d2cd31509f3ef9e`
(`0.3.183-SNAPSHOT`), with maintainer-reported local tests PASS.

It closes the requirements-first half of PLAT048 B′:

~~~text
project/lock
    ->
bundled Package Tool exactExternalRequirements(...)
    ->
detached List<ProtosExactExternalPackageIdentity>
~~~

The result is complete exact identity only and retains no Path, locator,
Filesystem, custody, guest object, store/cache location or host handle. The
Package Tool Process terminates before the call returns.

This publication deliberately does not yet add the materialization provider,
store backend, public-run mixed orchestration or CLI behavior change. F2E5
therefore remains IN_PROGRESS. The retained evidence is
`docs/project/evidence/TOOL001/TOOL001_F2E5_EXACT_EXTERNAL_REQUIREMENTS_PREFLIGHT.md`.

The next boundary is the ratified host-owned exact materialization provider plus
public-run composition through unchanged F2E2/F2E3/F2E4 ownership/lifecycle
rules.

## Explicit exclusions

F2E does not by itself implement registry discovery, networking/HTTP,
credentials, fresh resolution/update, fetch/store-write acquisition, archive
transport, ArtifactDigest download validation, publication or store GC.

## F2E0 closure

Direct executable continuation is premature for two independent reasons:

1. the exact ContentIdentity logical tree contract remains open;
2. the current general Filesystem surface does not yet expose portable tree
   observation sufficient to implement an unknown future contract.

The correct next work is F2E1, not an external resolver shortcut.

After publication:

```text
TOOL001-F2D   CLOSED
TOOL001-F2E   IN_PROGRESS
TOOL001-F2E0  CLOSED
TOOL001-F2E1  READY
TOOL001-F2E2  BLOCKED_BY_DEPENDENCIES: TOOL001-F2E1
TOOL001-F2E3  BLOCKED_BY_DEPENDENCIES: TOOL001-F2E2
TOOL001-F2E4  BLOCKED_BY_DEPENDENCIES: TOOL001-F2E3
TOOL001-F2E5  BLOCKED_BY_DEPENDENCIES: TOOL001-F2E4
TOOL001-F2    CLOSED: NO
TOOL001-F     CLOSED: NO
TOOL001       CLOSED: NO
```

## F2E1 decomposition refinement and F2E1A closure

The ContentIdentity contract is decomposed because three boundaries can be
reviewed independently without publishing temporary semantics:

```text
F2E1A  logical-tree domain + portable path/entry-kind contract     CLOSED
F2E1B  canonical byte stream + method/hash contract                READY
F2E1C  independent conformance vectors + F2E1 closure              BLOCKED_BY_DEPENDENCIES
```

E1A is owned in `docs/design/PACKAGE_CONTENT_IDENTITY.md`.

It freezes the abstract logical package tree before selecting serialization:
ContentIdentity observes an already-materialized package payload, not an arbitrary
source checkout. Every valid regular-file leaf in that payload is semantic.
There is no hidden `.gitignore`, VCS configuration, build-directory heuristic,
timestamp/permission rule or file-extension allowlist.

The root must contain exact regular file `protos.toml`. Directories are structural
only; empty directories are non-semantic. Symbolic links and other traversal/
special-entry forms are rejected in v1. File content is exact bytes and all host
metadata is excluded. Package payload paths use the conservative portable ASCII
artifact-path domain selected by E1A, including case-fold collision and Windows
reserved-name rejection.

E1A does not yet define the canonical byte stream or digest. E1B owns that
serialization/hash boundary. E1C then publishes independent fixed vectors and
closes parent E1. F2E2 remains blocked on the whole E1 parent, not merely E1A.

## F2E1B closure — canonical stream + digest support

E1B freezes one binary serialization over the E1A map. Entries sort by exact
unsigned ASCII path bytes. The stream begins with the method-domain separator
`protos-package-tree-v1` plus NUL, then zero or more tagged FILE records and one
END tag. Each path/content field is length-prefixed with the canonical minimal
arbitrary-precision unsigned base-128 varuint; file bytes are fed directly and no
directory/metadata/per-file-hash records are introduced.

Current ContentIdentity support hashes that complete canonical stream with
standard SHA-256 and emits 64 lowercase hexadecimal digits under the separately
persisted `sha256` algorithm token. Method and digest algorithm remain orthogonal:
tree/serialization changes require a new method token, while a future hash
transition over exactly the same stream may use a new algorithm token.

E1C is now READY to publish independently computed fixed vectors and close F2E1.
F2E2 remains blocked on the parent F2E1 closure.

## F2E1 final closure and F2E2 prerequisite result

F2E1 is CLOSED by A/B/C. The exact package-tree identity is owned by
`PACKAGE_CONTENT_IDENTITY.md` and `PACKAGE_CONTENT_IDENTITY_VECTORS.md`.

The required E2 post-E1 capability audit now has a concrete answer. Verification
needs confined directory enumeration, exact stored child names, entry-kind
observation without following links/special entries, regular-file acquisition,
and one stable logical snapshot or fail-closed mutation detection.

Current standard Filesystem exposes `open`, `replace` and `remove`, but not that
tree-observation boundary. LIB004 already records that directory
enumeration/stat/symlink inspection cannot be manufactured from ambient host
APIs. A package-only Java/NIO recursive verifier would violate F2E0, so B009
records the missing general semantic owner and F2E2 is BLOCKED.

## D046 / B009 normative resolution

D046 / specification revision `0.1.383` closes the semantic gap identified
after F2E1 without moving Package Tool policy into Java.

The general Core surface is:

```text
Filesystem.entries(path)
Filesystem.captureTree(path)
```

The critical pattern is **capture then verify then use the same captured
Filesystem**. `captureTree` need not freeze a hostile mutable source at one
physical instant. It produces a fresh immutable logical tree from exact
race-safe selections; F2E2 will compute the closed `protos-package-tree-v1`
identity over that captured capability and only admit it when the digest matches
the lock. Execution must then bind the same verified capture, never re-open the
original store Paths.

B009 therefore moves to READY. F2E2 remains dependency-blocked until I024
publishes the general D046 implementation.


## Post-I024 dependency closure

I024-D closes the general D046 implementation/conformance prerequisite and B009.
`TOOL001-F2E2` is now READY. Its next implementation must capture the exact
already-selected package-store root through the provisioned Filesystem, validate
`protos-package-tree-v1` against that immutable captured Filesystem, and hand the
same captured authority to subsequent execution planning. It must not re-read the
mutable/source store tree after verification and must not introduce package-only
Java/NIO traversal.

F2E3/F2E4/F2E5 remain dependency-gated in order behind F2E2.

## F2E2 decomposition refinement

The post-I024 audit confirms that the missing work is no longer one indivisible
implementation step. Three responsibilities have different semantic and host
failure surfaces and must remain separate:

```text
F2E2   verified read-only package-store binding                    IN_PROGRESS
F2E2A  captured-Filesystem ContentIdentity canonicalizer/verifier  CLOSED
F2E2B  exact selected-root capture + verified-capture host custody READY
F2E2C  same-capture integration + F2E2 closure                     BLOCKED_BY_DEPENDENCIES
```

### F2E2A — captured-Filesystem ContentIdentity policy

A owns package policy and is implemented in bundled Protos.

Its input is already the immutable read-only Filesystem produced by D046. A
must implement the closed `protos-package-tree-v1` contract without acquiring,
discovering or reopening a store root:

- enumerate recursively only through `Filesystem.entries`;
- reject link/other entries rather than following or silently ignoring them;
- validate the exact canonical portable path domain, including sibling ASCII
  case-fold collisions;
- require exact root regular `protos.toml`;
- include every valid regular-file leaf and treat directories only as structure;
- read exact regular bytes through ordinary Filesystem/File facilities;
- serialize the frozen E1B path/content map exactly, including arbitrary-size
  minimal base-128 varuint framing and canonical unsigned ASCII path order;
- support the mandatory `protos-package-tree-v1` + `sha256` pair through the
  existing standard SHA256 library;
- compare exact lowercase digest identity fail-closed; and
- on successful verification, return/pass through the **same supplied captured
  Filesystem** rather than another path, re-capture or reconstructed authority.

`std:io/Files.readAllBytes` may provide the already-published complete binary
File-read/cleanup mechanism. A must not duplicate host traversal in Java, decide
package-store layout, fetch content, scan by PackageId/version, mutate the lock,
extend PackageExecutionPlan, install a resolver, or choose how captured backing
crosses the later host boundary.

A is independently executable because ContentIdentity membership,
canonicalization, method/algorithm support and verification semantics are already
closed by F2E1 and D046.

### F2E2B — exact selected-root capture and host custody

B owns the irreducible host-mechanical boundary deliberately excluded from A.

It starts from **one exact store entry already selected by the caller/upper
mechanism**. It must not infer that entry from PackageId, version, locator,
registry authority, Git URL, directory basename or ambient store contents.

B must provision/confine the general read-only Filesystem authority for that
selected root, obtain one D046 immutable capture, expose that capture to A for
verification, and preserve custody of that exact verified captured backing for
the later F2E3/F2E4 handoff. The concrete custody representation is intentionally
not selected by this decomposition: B must re-audit current I024 captured-backend
and Package Tool Process-lifetime mechanics before choosing it.

The important invariant is fixed now:

```text
selected mutable/materialized root M
        |
        | capture exactly once for this binding
        v
immutable capture C
        |
        | A verifies ContentIdentity(C)
        v
verified C
        |
        +----> later planning/resolver handoff uses C
```

No step after verification may re-open `M` to obtain executable bytes.

B remains host mechanism only. It must not reimplement canonical path/hash policy,
parse locks/manifests, solve versions, scan/fetch the store, or make an external
node executable.

### F2E2C — integrated same-capture closure

C composes A and B and publishes final F2E2 evidence.

It must exercise the frozen F2E1 fixed vectors through the production
ContentIdentity implementation and prove at the integration boundary that:

- valid exact content verifies;
- mismatched content/method/algorithm and invalid tree shapes fail closed;
- source/store mutation after capture cannot alter the verified bytes;
- the authority handed forward is the same immutable capture that was verified;
- Package Tool verification receives no ambient store visibility; and
- no package-specific Java/NIO tree walker appears.

C closes F2E2 only after those properties and the full applicable test suite pass.
At that point `TOOL001-F2E3` becomes READY. F2E4/F2E5 remain dependency-gated.

This decomposition does not change F2E1, D046, I024, lock format, ContentIdentity
bytes, package-store physical layout, acquisition/fetch policy or public run
semantics.

## F2E2A closure — captured-Filesystem ContentIdentity policy

F2E2A is CLOSED.

Published bundled-Protos implementation:

```text
self:ContentIdentity

digest(capturedFilesystem)
    -> { method, algorithm, hex }

verify(capturedFilesystem, expectedContentIdentity)
    -> same capturedFilesystem on exact match
```

The implementation consumes the already-captured Filesystem supplied by its
caller. It never calls `captureTree`, inspects a host/store path, searches by
PackageId/version, performs registry/Git acquisition, or introduces Java/NIO tree
walking.

`digest` implements the closed `protos-package-tree-v1` policy directly in
Protos:

- iterative directory traversal through `Filesystem.entries`, so physical tree
  depth does not create one recursive Protos frame per directory;
- exact E1A portable-ASCII segment validation plus Windows-reserved basename
  rejection and per-directory ASCII-case-fold collision rejection;
- exact root regular `protos.toml` requirement;
- link/other rejection without following;
- directories as structure only and every valid regular leaf as identity input;
- exact regular bytes through the existing `std:io/Files.readAllBytes` helper;
- canonical complete-path unsigned ASCII ordering through
  `std:collections/Array.sort`;
- E1B FILE/END framing and minimal arbitrary-precision base-128 varuint;
- standard `std:crypto/SHA256` over the canonical stream; and
- exactly 64 lowercase hexadecimal digest digits under method
  `protos-package-tree-v1` and algorithm `sha256`.

`verify` validates the recorded method/algorithm/digest shape fail-closed before
tree hashing, compares the exact resulting digest, and returns the same supplied
captured Filesystem object only on a match. It does not recapture or reconstruct
authority.

The F2E2A behavioral fixtures remain Protos-owned under
`protos/tests/package-tool/content-identity/**`. Their host materialization and
string-binding mechanics are executed by the already-published single
`ProtosPackageToolProtosTest` Java bridge. No `ProtosPackageToolContentIdentityTest`
or other per-corpus Java wrapper is introduced. Fixture execution is RootActor-
local through `ProtosRootTaskExecution`, so D046 Future observation occurs in the
same execution model used by the consolidated TOOL001 runner.

The production implementation is checked against the frozen E1 vectors for
minimal content, canonical ordering/empty-file framing, binary content, 128/300
varuint boundaries and both exact-case identities. A pure Protos fixture checks
the standalone frozen varuint values through `2^70`. Representative negative
evidence covers missing root manifest, invalid artifact character, reserved
basename, sibling ASCII-case-fold collision, symbolic-link rejection,
unsupported method/algorithm and malformed digest spelling.

F2E2C intentionally remains the owner of the complete cross-boundary fixed-vector
and equivalence matrix together with the same-capture custody proof. F2E2A does
not claim that final integration closure.

After this slice:

```text
TOOL001-F2E2   IN_PROGRESS
TOOL001-F2E2A  CLOSED
TOOL001-F2E2B  READY
TOOL001-F2E2C  BLOCKED_BY_DEPENDENCIES: TOOL001-F2E2B
TOOL001-F2E3   BLOCKED_BY_DEPENDENCIES: TOOL001-F2E2
```

No normative Protos specification, lock format, ContentIdentity bytes,
package-store layout, acquisition/fetch policy, captured-backend custody design,
PackageExecutionPlan shape, resolver identity or public-run semantics change in
F2E2A.

Implementation note: exact file bytes are awaited into ordinary local values before
the canonical record object is constructed. Record construction is therefore
suspension-free while preserving the frozen path/content map and the same
ContentIdentity bytes.


## F2E2B closure — exact selected-root capture and host custody

F2E2B is CLOSED after explicit project-owner approval of the ownership/lifetime
architecture on 2026-09-08 following alternative, scalability and future-evolution review.

The selected boundary is one run-scoped host custody object for one exact immutable capture:

```text
selected materialized root M
        |
        | exact D046 capture, once
        v
immutable captured authority C
        |
        +--> fresh Filesystem(C, PackageToolDomain)
        |         |
        |         +--> F2E2A verifies ContentIdentity(C)
        |
        +--> retain SAME C after Package Tool Process termination
                  |
                  +--> later fresh Filesystem(C, application/resolver domain)
```

The production host machinery is `ProtosCapturedFilesystemCustody`. It owns the exact
`CapturedBackend` and its release callback independently of any Protos Actor-domain wrapper.
`materialize(activation)` creates a fresh standard structurally read-only `Filesystem` bound to
that activation's execution domain while delegating to the same captured backend. The source NIO
backend is closed immediately after the one selected-root capture, so later materialization never
reopens the original store `Path`. `close()` releases the run-owned captured backing exactly once.

The standard Filesystem bridge now has one host-internal captured-capability rematerialization
entry that reuses the existing D046 read-only adapter; no second Filesystem semantics or Actor
transfer exception is introduced. The current NIO implementation factors its already-published
secure capture machinery so host custody can capture the selected authority root without inventing
a second traversal/copy path.

The durable contract is intentionally backing-opaque. Today's implementation-managed temporary
blob backing is not part of F2E2B identity or policy; a future in-memory, CAS/deduplicated,
copy-on-write/versioned or remote immutable representation may implement the same custody without
changing `Filesystem`, F2E2A verification or the later execution boundary. There is no global
capture registry and no cross-run mutable coordination point.

F2E2B deliberately does not associate custody with `PackageId`, registry/Git node identity or a
`PackageExecutionPlan` record. That mapping remains F2E3 work. The detached execution plan stays
inert and contains no Filesystem/backend/Process/resolver/store authority. F2E2B also does not
select CAS persistence, cache GC, fetch, acquisition, store layout or distributed transport.

Focused evidence proves that source deletion after capture cannot change bytes observed through
either the Package Tool-domain or a separately bootstrapped application-domain view; those views
are fresh non-identical `ProtosFilesystemValue` objects backed by the same captured backend. It
also proves deterministic idempotent host-custody release and rejection of rematerialization after
close. Existing I024 capture/materialization focals plus the complete suite remain required by the
publication launcher.

After this slice:

```text
TOOL001-F2E2   IN_PROGRESS
TOOL001-F2E2A  CLOSED
TOOL001-F2E2B  CLOSED
TOOL001-F2E2C  READY
TOOL001-F2E3   BLOCKED_BY_DEPENDENCIES: TOOL001-F2E2
```

No normative Protos specification, lock format, ContentIdentity bytes, package-store physical
layout, acquisition/fetch policy, `PackageExecutionPlan` authority model, resolver identity or
public-run semantics change in F2E2B. Implementation version becomes `0.2.262-SNAPSHOT`.

## F2E2C closure — same-capture verification integration

F2E2C is CLOSED and closes parent F2E2.

The production composition boundary is `ProtosPackageContentVerification`. It accepts one exact
already-selected materialized package root plus the inert expected ContentIdentity fields. It does
not discover a root by PackageId/version/locator, scan a store, fetch content, parse a lock, or
perform package-specific tree traversal.

The host path is deliberately one-way:

```text
selected materialized root M
        |
        | ProtosCapturedFilesystemCustody.captureSelectedRoot(M)
        | capture exactly once; source authority closes
        v
run-owned immutable capture C
        |
        | fresh Filesystem(C, PackageToolDomain)
        v
self:ContentIdentity.verify(view(C), expected)
        |
        | integration requires result === supplied view(C)
        v
return custody(C) after Package Tool Process TERMINATED
        |
        +--> later fresh Filesystem(C, application/resolver domain)
```

The Package Tool verification Process receives no ambient/root Filesystem for the selected store.
Its only package-content authority is the captured Filesystem materialized from custody; method,
algorithm and hex are inert String inputs. Java does not duplicate ContentIdentity validation:
unsupported method/algorithm/digest spelling, digest mismatch and invalid captured-tree shape all
fail through the already-published bundled-Protos `self:ContentIdentity` policy.

On every unsuccessful verification path, host custody is closed rather than handed forward. On
success the verifier must return exactly the supplied Package Tool-domain Filesystem object, after
which that Process is terminated before the still-open custody is returned. A later domain receives
a fresh non-transferable Filesystem wrapper backed by the same immutable capture C; neither the
source Path nor the Package Tool Filesystem crosses the Actor/Process boundary.

Focused integration evidence composes the frozen F2E1 vectors already exercised through the single
`ProtosPackageToolProtosTest` with the new host gate. It proves a frozen binary-tree vector verifies,
the verification Process terminates with no root Filesystem, deleting/replacing the source after
verification cannot change bytes visible through a later independently bootstrapped domain,
post-capture source additions remain invisible, digest mismatch fails closed, unsupported
method/algorithm fail in Protos policy, and an invalid captured tree is not admitted. The production
host gate contains no Java/NIO tree walker and never reopens source content after verification.

F2E2 therefore now provides the required verified immutable-material boundary for the next slice.
F2E3 may decide how exact registry/Git nodes map to their already-verified run-local custody, but C
does not pre-select that PackageId/registry/Git association, extend `ProtosPackageExecutionPlan`,
install a resolver or change public run behavior.

After this slice:

```text
TOOL001-F2E2   CLOSED
TOOL001-F2E2A  CLOSED
TOOL001-F2E2B  CLOSED
TOOL001-F2E2C  CLOSED
TOOL001-F2E3   READY
TOOL001-F2E4   BLOCKED_BY_DEPENDENCIES: TOOL001-F2E3
TOOL001-F2E5   BLOCKED_BY_DEPENDENCIES: TOOL001-F2E4
```

No normative Protos specification, lock format, ContentIdentity bytes, captured-Filesystem
semantics, package-store physical layout, acquisition/fetch policy, PackageExecutionPlan authority,
ModuleKey identity, resolver behavior, public-run semantics or capture-backing policy changes in
F2E2C.

## F2E3 implementation progress — generation-2 external-leaf construction

F2E3 has started under ratified D053 without modifying generation 1.

The first bounded executable slice adds a separate bundled-Protos constructor:

```text
ExecutionPlan.buildV2FromVerifiedLeaves(
    projectTreeFilesystem,
    verifiedExternalPackages
)
```

It is intentionally not wired to public run or the Java host adapter yet.

This constructor preserves the existing V1 `build(projectTreeFilesystem)` path
unchanged and constructs the D053 generation-2 single mixed graph only from:

- the current non-stale workspace resolution state;
- canonical lock-format-1 workspace/registry/Git refs;
- exact lock `ContentIdentity`; and
- caller-supplied inert external descriptors whose ref/content exactly match
  locked external nodes and whose exports pass the existing runtime-name rules.

Workspace dependency reconciliation remains fail-closed. Path dependencies retain
the exact workspace target check. Registry dependencies additionally reconcile
the locked target against the declared authority/locator and selected version
constraint; Git dependencies reconcile fetch provenance and exact revision.
Those provenance fields are validation inputs only and do not enter the emitted
V2 plan.

This first slice deliberately admits only **external leaf packages**: any lock
edge declared by registry/Git is rejected. That is not a generation-2 semantic
restriction; it is the bounded implementation frontier. The remaining F2E3 work
must derive/validate each external package's manifest/exports/dependency
declarations from the same F2E2 verified custody, then allow the uniform mixed
edge relation already ratified by D053.

The slice also keeps the crucial authority boundary explicit: the V2 constructor
accepts only inert descriptors and emits only inert plan data. It accepts no
Filesystem, captured backend, custody object, host path, resolver, locator,
credentials or host handle.

F2E3 therefore remains **IN_PROGRESS** after this slice. F2E4 and F2E5 remain
dependency-gated.

## D056 ratification — external immutable packages cannot operationalize `path`

D056 is RATIFIED by explicit project-owner approval on 2026-09-09 after an
expanded cross-ecosystem review covering Cargo, npm/Yarn/pnpm, Go, Bazel,
Gradle, Python/uv, Ruby, Dart, Hex/Rebar3, Composer, Cabal, opam, Conan, vcpkg,
NuGet, SwiftPM and Nix.

The selected A+ rule is:

```text
mutable workspace/local-development package
    path dependency
        -> permitted under workspace policy

immutable registry/Git package
    path dependency
        -> fail closed

future override/vendor/patch
    -> may be designed explicitly as resolution-root-owned policy
       that produces an ordinary exact graph
```

The same F2E2 verified capture may now be used to read the external ManifestV1.
F2E3 must reject external `path` declarations, validate registry/Git declarations
against the exact root-owned lock and emit only the D053 inert typed graph.

D056 does not add publish-time manifest rewriting, override syntax, nested
multi-package immutable bundle identity or any PackageExecutionPlan path/host
authority. Those remain future decisions if real use cases justify them.

F2E3 remains **IN_PROGRESS** and is no longer decision-blocked.

## D057 ratification — workspace is not active immutable-package semantics

D057 is RATIFIED by explicit project-owner approval on 2026-09-09 after an
expanded cross-ecosystem review of workspace, monorepo, package-publication and
remote multi-package-source models.

The selected A-strict/refined rule is:

```text
mutable resolution-root / development package
    [workspace] -> permitted

immutable registry/Git package
    [workspace] -> fail closed
```

The presence of the table is rejected during immutable consumption even when
`members = []`.

The refinement deliberately preserves future monorepo/source-container support:
a later acquisition design may select explicit canonical package roots/subroots
inside one immutable source and produce ordinary exact PackageNodes. A later
publication design may also project a development workspace into independently
consumable package payloads. Neither future capability is inferred from current
`workspace.members`.

F2E3B / GitHub #236 is released to continue mechanically under D053, D056 and
D057. It may read ManifestV1 through the same F2E2 verified capture, reject
external workspace/path semantics, validate registry/Git manifest identity and
transitive dependency declarations against the exact root-owned lock, and emit
only inert generation-2 plan data.

F2E3 remains **IN_PROGRESS**. F2E4/F2E5 remain dependency-gated.

## F2E3B closure — verified external manifest + transitive graph construction

F2E3B is CLOSED by this executable slice under D053, D056 and D057.

The bundled-Protos execution planner now has a separate
`buildV2FromVerifiedCaptures(projectTreeFilesystem, verifiedExternalPackages)`
path. Each supplied external item carries only the exact locked ref,
ContentIdentity evidence and the **same already-verified captured Filesystem**
owned by F2E2. F2E3B reads exact `protos.toml` through
`ManifestCommand.load(capturedFilesystem)`, derives exports/dependencies from
that manifest and never accepts caller-authored export/dependency policy.

Preflight now enforces exact PackageId, ReleaseVersion validity, registry
release equality, exact active LanguageCompatibilityId when declared, D056
external-path rejection, D057 external-workspace rejection, exact
registry/Git provenance and constraint/revision reconciliation, complete
external descriptor coverage and exact dependency-edge accounting.

The resulting generation-2 plan still contains only typed NodeRefs,
ContentIdentity, exports and dependency edges. Captured Filesystem/custody,
physical/source/store paths, registry locator/authority, Git fetch provenance,
resolver authority and host handles do not cross into PackageExecutionPlan.

Focused conformance includes one mixed workspace -> registry ->
{registry, exact-Git} graph plus fail-closed identity/version/compatibility,
D056/D057, provenance, edge-accounting, descriptor, duplicate-edge and
dangling-target evidence. Git manifest `package.version` is validated as
ReleaseVersion metadata but does not replace exact revision identity.

F2E3 remains **IN_PROGRESS**. The remaining parent work is the final
verified-custody/composition boundary that will connect F2E2 host custody to
this Protos-owned V2 construction before F2E4 host detach/resolver work can
begin.

## F2E3C closure — borrowed verified-custody composition + F2E3 closure

F2E3C is CLOSED by this executable host-composition slice, closing parent F2E3.

`ProtosExternalPackagePlanningPreflight` now composes the already-published
boundaries without adding a new semantic layer:

```text
root-owned exact lock + workspace Filesystem
        |
borrowed F2E2 verified external custodies
        |
        | materialize each exact custody into one fresh Package Tool activation
        | no source/store reopen, no second capture
        v
temporary ordinary verifiedExternalInputs
        |
        v
ExecutionPlan.buildV2FromVerifiedCaptures(...)
        |
        v
raw inert PackageExecutionPlanV2
```

Incoming `ProtosCapturedFilesystemCustody` values are borrowed run-owned
authority. F2E3C neither consumes nor closes them on success or failure.
F2E4 remains the owner of the future concrete exact-package-identity ->
verified-custody resolver/lifetime mapping.

The composition Process owns only its confined workspace Filesystem view,
temporary domain-bound external Filesystem views and Package Tool Process
lifetime. It terminates before the raw inert V2 value is returned. No
Filesystem/custody/path/store/locator/fetch/authority/host handle is retained in
the plan.

Integration evidence verifies that three independently captured and
ContentIdentity-verified external packages (registry A, registry B, exact-Git C)
still produce the mixed V2 graph after all three original selected source
directories are deleted. This proves planning consumes the exact same immutable
captured backing verified by F2E2 rather than reopening mutable store paths.
Both successful and failed planning leave all borrowed custodies open for later
F2E4/F2E5 use, while the Package Tool Process terminates in both cases.

F2E3 is therefore CLOSED. F2E4 is READY to implement the already-deferred
generation-2 defensive detach plus canonical external ModuleKey/source resolver
and its concrete run-local custody mapping. F2E5 remains dependency-gated on
F2E4.

## PLAT012 ratification — F2E4 architecture gate cleared

PLAT012 is RATIFIED by explicit project-owner approval on 2026-09-09 after the
expanded loader/runtime/package-store review and scalability analysis.

F2E4 must now consume A+ mechanically: detached exact V2 identities are
reconciled 1:1 with a run-owned verified package-resource scope; canonical
external ModuleKeys use exact immutable package identity + internal logical
module; source bytes are read lazily from the same verified backing through a
host-neutral resource reader and returned without physical source-path identity.

Equal content may be physically deduplicated without collapsing logical package
identity. Cache policy remains tuning. F2E5 retains public run lifecycle and
final teardown integration.

F2E4 remains **READY** and is no longer architecture-blocked.

## F2E5 activation audit — PLAT048 materialization-authority gate

F2E4 closed at product revision
`0ce6a30635cc1db8fa27b6834bb04aab47769ac2` and released F2E5 for a
public-run integration audit.

That audit found that the already-published F2E2/F2E3/F2E4 machinery is
mechanically composable after one exact external package root has already been
selected:

```text
selected root
    -> same-capture ContentIdentity verification
    -> borrowed verified custody planning
    -> generation-2 defensive detach
    -> exact 1:1 resource-scope reconciliation
    -> mixed V2 module resolution
    -> application execution
    -> application Process termination
    -> resource-scope close
```

Current public `protos run <entry> [args...]` nevertheless remains
workspace-only because current HEAD has no production authority that maps every
exact locked external identity to an already-present local materialized package
root before F2E2 capture.

The missing boundary is:

```text
exact locked registry/Git identity
    ->
explicit production materialization-selection authority
    ->
already-selected exact local root
```

The lock/Package Tool can already derive the complete exact external identity
set from the public-run project input. It cannot select physical roots without
new host authority/configuration policy. Tests that pass temporary roots
directly are fixtures and do not close this production boundary.

This is a durable platform/host-authority choice rather than helper placement.
`PLAT048 — Public-run external materialization authority boundary` is allocated
as `guillermomolina/protos#789`.

Until PLAT048 is explicitly selected and durably ratified:

```text
TOOL001-F2E5   BLOCKED_BY_DECISION: PLAT048/#789
TOOL001-F2E    IN_PROGRESS
TOOL001       IN_PROGRESS
```

F2E5 must not bypass the gate by inventing a canonical store directory, ambient
cache scan, environment variable, new CLI/config input, hidden global, fetch,
solve or lock mutation.

The immutable activation evidence is retained at:

`docs/project/evidence/TOOL001/TOOL001_F2E5_PUBLIC_RUN_MATERIALIZATION_LIFECYCLE_AUDIT.md`.

The maintainer reports all local tests PASS for the audited current product
state. No new product implementation or normative semantics are claimed by this
investigation record.

## F2E5 provider-backed mixed-run composition checkpoint

Published product revision:

~~~text
PROTOS_REVISION=a38470bc6e2f68e770ddc8054053995bb2477b19
SUBJECT=TOOL001-F2E5: add provider-backed mixed package run composition
VERSION=0.3.185-SNAPSHOT
LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

This publication adds the CLI-neutral `ProtosPackageRunDriver` and the
run-scoped `ProtosExactPackageMaterializationProvider` seam selected by
PLAT048 Candidate B′.

The provider key is the complete exact external identity and each external
requirement receives exactly one provider selection before unchanged F2E2
capture plus ContentIdentity verification. The composed external path is:

~~~text
exact external requirements
    -> exact host provider selection
    -> F2E2 capture + verification
    -> F2E3 raw V2 planning
    -> F2E4 defensive detach
    -> F2E4 exact resource-scope reconciliation
    -> mixed V2 application Process
    -> application Process TERMINATED
    -> resource scope close
~~~

Workspace-only graphs preserve the generation-1 `ProtosWorkspaceRunDriver`
route and do not consult the provider.

Before successful reconciliation the driver owns all verified custodies.
Provider miss, Nth verification failure, planning failure, detach failure or
reconciliation failure closes every previously verified custody and starts no
application Process. After successful reconciliation the resource scope owns
the complete custody set and remains live until application Process
termination.

Published tests cover the workspace fast path, exact lookup multiplicity and
identity separation, real mixed registry/Git execution, provider miss, wrong
materialization / Nth verification failure, planning failure, reconciliation
failure, and successful termination-before-scope-close lifecycle ordering.

This checkpoint intentionally does **not** add:

- a default physical materialization backend;
- public `ProtosCli` mixed-run wiring;
- a new CLI flag, environment variable or public configuration surface;
- a canonical public package-store/cache layout;
- solving, fetch, network, credentials, store writes, GC or lock mutation.

F2E5 is **CLOSED** by product revision
`cd710a0cff2768691c6b654b1ab85fbb96d9d6ec` / `0.3.187-SNAPSHOT`.

The public-run bootstrap now constructs the implementation-private read-only
local exact materialization backend and routes public
`protos run <entry> [args...]` through the existing provider-backed
`ProtosPackageRunDriver`. The private backend computes one deterministic opaque
location from the complete typed exact identity; it performs no directory
enumeration, partial lookup, fallback, solve, fetch, network access, lock
mutation, store write, repair or GC. A missing exact already-present root fails
closed before application execution.

The selected root still crosses unchanged F2E2 capture + ContentIdentity
verification, F2E3 V2 planning and F2E4 detach/reconciliation/resolution.
Workspace-only public runs remain on generation 1 and do not consult the
materialization provider. Focused public-run coverage proves an imported external
module executes through that complete path; missing materialization fails with
the existing host-failure family and does not create or mutate the store/lock.

Maintainer-reported local validation is PASS, including the focused F2E5/CLI
tests and the integrated `make test` suite. This project-docs repository did
not independently execute that validation.

F2E, F2 and F are therefore closed for the current bounded Package Tool scope.
Remote acquisition/fetch, credentials, store-write/repair/GC, publication,
vendor/system stores and multi-store composition remain separately scoped future
capabilities rather than residual F2E work.

Final retained evidence:
`docs/project/evidence/TOOL001/TOOL001_F2E5_FINAL_PUBLIC_RUN_CLOSURE.md`.

