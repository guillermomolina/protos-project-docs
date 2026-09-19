# D171 — Public Filesystem captureTree versus host/runtime immutable custody

Status: **RATIFIED — Candidate B (remove public captureTree; retain PLAT012 custody)**

Approval date: **2026-09-19**
Decision issue: `guillermomolina/protos#642`
Trigger: AUD009-D2 / `guillermomolina/protos#639`
Protos evidence revision: `3968473f9117fc9f4b4a5f2a347b7b2e15ce7969`
Project-record base: `314477a0146d3079fda0ae15982a906ac3797f23`

This is a durable non-normative decision record. Observable Protos semantics
remain authoritative only through the applicable ratified material under
`guillermomolina/protos:spec/**`.

## Decision

D171 selects **Candidate B**.

The general public Core operation:

```text
Filesystem.captureTree(path) -> Future<Filesystem>
```

is removed from the portable language surface under
`REMOVE_NOW_RECONSIDER_LATER`.

The project **does not** remove the verified immutable-tree architecture that
motivated D046 and is now required by PLAT012.

The selected boundary is:

```text
PUBLIC CORE

Filesystem.entries                KEEP
Filesystem.open                   KEEP
Filesystem.replace/remove         KEEP
Filesystem.captureTree            REMOVE_NOW_RECONSIDER_LATER

HOST / RUNTIME / TOOL ARCHITECTURE

PLAT012 exact immutable custody             KEEP
secure recursive no-follow capture          KEEP
immutable captured backing                  KEEP
read-only captured Filesystem views         KEEP
verify exact custody then use same custody  KEEP
original source/store Path never reopened   KEEP
run-owned custody/lifetime                  KEEP
```

D171 removes a **general guest-visible mint-new-Filesystem operation**. It does
not remove the internal capability to capture an already selected tree into
immutable runtime custody.

## Why this differs from deleting the capability

The current implementation has a shared capture engine.

Conceptually, public D046 capture uses:

```text
captureTree(path)
    -> captureSelectedDirectory(components)
    -> secure recursive capture
    -> ProtosNioCapturedTreeFilesystemBackend
    -> read-only Filesystem materialization
```

PLAT012 host custody uses:

```text
captureSelectedRoot(...)
    -> captureRootForHostCustody()
    -> captureSelectedDirectory(empty)
    -> secure recursive capture
    -> ProtosNioCapturedTreeFilesystemBackend
    -> run-owned verified custody
```

The expensive and security-sensitive machinery is therefore already shared.

Removing the public selector does **not** justify deleting:

- secure no-follow recursive traversal;
- stable selected-resource handling;
- captured regular/directory/link/other representation;
- immutable blob/tree backing;
- read-only captured Filesystem materialization;
- runtime-owned release/custody;
- package verification over the exact captured backing; or
- later source resolution from that same verified backing.

That machinery remains justified by a production Package Tool requirement and
PLAT012.

## PLAT012 fixed authority

D171 preserves all PLAT012 durable constraints relevant to immutable package
custody, including:

- one already-selected package root is captured into immutable custody;
- exact external package identity is distinct from physical backing;
- verification operates on the same custody later used for source resolution;
- the original selected/store Path is never reopened after verification;
- custody is run/session-owned rather than guest-owned;
- backing may evolve across NIO, memory, mmap, CAS, deduplicated blobs, brokered
  readers or remote immutable storage without changing semantic identity;
- source reading does not require guest Filesystem/Activation/Process/Actor
  machinery; and
- caches/reader projections borrow from the run owner and do not redefine
  lifetime.

D171 may simplify implementation seams that exist only to expose the guest
operation, but may not weaken those constraints.

## D046 relationship

D046 remains important prior art and historical authority for the semantics it
introduced.

D171 explicitly supersedes only the **public captureTree portion** of D046.

The following D046-derived concepts remain valid where required by internal
captured Filesystem views and PLAT012:

- exact no-follow directory observation/capture behavior;
- immutable captured backing;
- read-only captured Filesystem behavior;
- source-independence after successful capture;
- opaque treatment of link/other entries where represented;
- no caller-managed lifetime requirement for ordinary guest Filesystem views
  materialized over internally managed immutable backing; and
- verify-then-use of the same immutable material.

D171 does not require retaining every public D046 sentence, Future/cancellation
rule or guest materialization rule that exists solely because
`Filesystem.captureTree` is public.

## Why Candidate A is not selected

Candidate A keeps the complete public D046 operation.

It has genuine strengths:

- `Filesystem -> Filesystem` reuses an existing authority category;
- the result is authority attenuation, not ambient authority growth;
- no new public Directory/Snapshot/Lease family is required;
- the operation is pay-for-use at runtime;
- the implementation is mature and tested.

However, the only concrete production requirement that motivated immutable tree
capture has evolved toward a narrower host/runtime boundary.

Package Tool does not use guest:

```text
filesystem.captureTree(path)
```

as its production custody authority. It uses host-owned exact selected-root
capture and PLAT012 run custody because it must guarantee:

```text
capture exact selected root once
verify exact custody
reuse exact custody later
never reopen source/store Path
host owns lifetime
```

Retaining a general guest operation would therefore preserve a substantial
portable semantic commitment without a current guest/library consumer.

## Why Candidate C is not selected

A source-only Standard Library recursive copy cannot preserve the existing
strong contract from ordinary:

```text
Filesystem.entries
Filesystem.open
```

composition.

Between observing one child and reopening its name, a mutable namespace may
replace or retarget the entry. Source code lacks the backend-level stable
selection/no-follow machinery needed to guarantee equivalent race-safe capture.

A privileged Standard Library helper would simply re-expose the same general
public capability through another layer despite no current user requirement.

D171 therefore does not move captureTree mechanically from Core to Standard
Library.

## Why Candidate D is not selected

No smaller public privileged primitive has a demonstrated current consumer.

Introducing a new public:

```text
Snapshot
CaptureToken
TreeHandle
ImmutableTree
```

or another authority family would increase the universe merely to reconstruct a
capability already expressible as a read-only Filesystem.

If a future application needs immutable tree capture, D046's
`Filesystem -> Filesystem` shape remains strong prior art and may be
reintroduced substantially as-is if the then-current evidence supports it.

## Approval provenance

AUD009-D2 classified:

```text
public Filesystem.captureTree
    REMOVE_NOW_RECONSIDER_LATER

PLAT012 verified immutable package custody
    KEEP
```

D171 then separated public API necessity from retained internal capability.

The exact recommendation presented to the project owner was:

```text
D171 = Candidate B
       remove public Filesystem.captureTree;
       retain PLAT012 host/runtime immutable custody

KEEP:
    Filesystem.entries
    PLAT012 exact immutable package custody
    secure recursive capture machinery
    immutable captured backing
    read-only captured Filesystem materialization
    verify-same-custody-then-use
    no source reopen
    authority confinement
    TOCTOU protection

REMOVE_NOW_RECONSIDER_LATER:
    public Filesystem.captureTree(path)
    public arbitrary-subtree captured-Filesystem contract
    guest-facing capture cancellation/result contract

DO NOT REMOVE:
    ProtosCapturedFilesystemCustody
    host captureSelectedRoot
    shared captured backend merely because
    the public selector disappears
```

The project owner explicitly approved that exact pending candidate on
2026-09-19:

```text
aprobado
```

```text
SELECTED_CANDIDATE=B
DECISION_APPROVAL_PROVENANCE=PASS
```

## GITHUB021 invariant/delta consistency

Applicable fixed authority:

```text
PLAT012_EXACT_IMMUTABLE_CUSTODY                  PRESERVED
VERIFY_AND_USE_SAME_CUSTODY                      PRESERVED
NO_SOURCE_PATH_REOPEN                            PRESERVED
RUN_OWNED_CUSTODY                                PRESERVED
BACKING_NOT_LANGUAGE_IDENTITY                    PRESERVED
CAPTURED_READ_ONLY_FILESYSTEM_VIEWS              PRESERVED
FILESYSTEM_ENTRIES                               PRESERVED
AUTHORITY_CONFINEMENT                            PRESERVED
PACKAGE_TOCTOU_RESISTANCE                        PRESERVED
```

Observable semantic/API delta:

```text
Filesystem.captureTree(path)                     REMOVED
public arbitrary subtree immutable capture       REMOVED
guest Future/cancellation contract for capture   REMOVED

Filesystem.entries                               UNCHANGED
ordinary captured read-only Filesystem behavior
when host/runtime provisions such a capability   PRESERVED
```

No owner-approved PLAT012 invariant is contradicted.

```text
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Comparative evidence

The decision considered multiple mature approaches.

### Git object database

Git separates mutable working-tree observation from immutable tree/blob objects.

Contribution: validates immutable tree materialization as a valuable tool/storage
concept without requiring a general language filesystem API for arbitrary
snapshot minting.

### Nix store

Nix relies on immutable store objects and content identity.

Contribution: validates immutable verified material and physical backing
independence as infrastructure architecture.

### Bazel CAS / remote execution

Bazel models exact input roots and immutable content-addressed inputs for
execution.

Contribution: reinforces the separation between build/tool custody and ordinary
application filesystem API.

### OCI/container snapshotting

Container runtimes use snapshots, copy-on-write or materialized root filesystem
states behind runtime/storage boundaries.

Contribution: demonstrates that snapshot/capture capability may properly belong
to runtime infrastructure rather than the language-visible filesystem object.

### Java secure filesystem machinery

Secure directory-handle operations demonstrate why race-safe relative selection
requires backend/resource machinery that ordinary path reopening cannot
reproduce.

Contribution: rejects a naive source-only Standard Library replacement.

### Rust capability filesystems / WASI-style preopens

Capability-oriented filesystem systems place authority in explicit directory or
filesystem capabilities and keep traversal confined.

Contribution: validates the retained Protos Filesystem authority model without
requiring every capability holder to mint immutable snapshot authorities.

The comparison supports two independent conclusions:

```text
immutable captured trees are useful            YES
general public captureTree is currently needed NO
```

## Comparative scoring

Scores use 1-5; confidence H/M/L.

| Criterion | A Public | B Host/runtime | C stdlib source | D new primitive |
| --- | ---: | ---: | ---: | ---: |
| Correctness / invariant preservation | 5H | 5H | 2H | 5M |
| Protos alignment | 4H | 5H | 3H | 3M |
| Present-need proportionality | 2H | 5H | 3H | 2H |
| Incremental growth | 5H | 5H | 3M | 4M |
| Future-option resilience | 5M | 5H | 4M | 4M |
| Scalability | 4M | 5H | 3M | 4M |
| Conceptual simplicity | 3H | 5H | 3M | 3M |
| Portability / implementation freedom | 4H | 5H | 2H | 4M |
| Runtime / resource cost | 5H | 5H | 4M | 5H |
| Failure / operability | 5H | 5H | 2H | 4M |
| Deferral / reversibility / migration | 4M | 5H | 3M | 3M |
| Evidence maturity / implementation risk | 5H | 5H | 2M | 2M |

Arithmetic is only a comparison aid.

Candidate A carries an overengineering/public-surface red flag: it preserves a
large general guest contract after the motivating production requirement moved
to a narrower host/runtime boundary.

Candidate B avoids an underengineering red flag because the difficult capture
architecture is retained rather than discarded.

## Incremental-design analysis

### Smallest sufficient solution

Current requirements need:

```text
guest:
    Filesystem.entries/open/read

host/tool:
    secure exact selected-root capture
    immutable custody
    verification
    reuse of same custody
```

They do not need guest arbitrary-subtree capture.

### Pay for what you need

The project continues paying maintenance for secure capture because Package Tool
requires it.

Programs and the language model no longer pay the conceptual contract cost of a
general public capture operation that no production guest uses.

### Grow as you need

If a future compiler, IDE, plugin system, deployment tool, test facility or
ordinary application needs immutable subtree capture, the existing internal
engine and D046 design evidence provide a low-cost route to a new public
decision.

### Cost of deferral

Deferring the public API does not require rebuilding:

- recursive secure traversal;
- immutable backing;
- captured filesystem representation;
- host custody;
- verification plumbing.

A future reintroduction would primarily require:

- selecting the public placement/API;
- reinstating/adapting the guest operation bridge;
- defining any Future/cancellation/lifetime semantics then required;
- normative specification; and
- public conformance/tests.

This is bounded compared with deleting the underlying architecture.

### Reversibility

D046 remains available as a proven design if evidence later favors
reintroduction.

D171 intentionally does not classify the concept as
`REMOVE_PERMANENTLY`.

## Future-scenario stress test

A future requirement such as reproducible compiler workspaces, plugin package
verification, deployment bundles, static-site inputs, test fixture snapshots or
application integrity checks may justify guest immutable capture.

If that happens, Protos can evaluate whether:

```text
Filesystem.captureTree(path) -> Future<Filesystem>
```

is still the best surface.

The escape path is unusually strong because the underlying secure immutable
capture engine remains required independently.

## Strongest argument against Candidate B

The public D046 API is already implemented, tested, authority-safe, category
minimal and runtime pay-for-use. Removing a mature API solely because production
guest source does not call it risks unnecessary API churn.

D171 accepts that cost because the actual motivating production requirement now
uses a materially narrower host/runtime custody boundary and because the
expensive implementation is retained. The removal therefore reduces public
semantic commitment without throwing away the difficult architecture.

## Intentionally deferred

D171 does not decide:

- future reintroduction of public immutable capture;
- future Standard Library snapshot helpers;
- live subtree delegation;
- a public Directory family;
- incremental directory streams;
- persisted snapshot identities;
- CAS APIs;
- snapshot serialization/export;
- deterministic reclamation of large immutable guest datasets;
- remote snapshot services; or
- whether a future public facility should reuse the exact D046 spelling.

## Implementation consequences

D171 requires bounded implementation work.

```text
NORMATIVE_SPEC_CHANGE_REQUIRED=YES
PUBLIC_FILESYSTEM_API_CHANGE_REQUIRED=YES
PUBLIC_CAPTURE_FLOW_RECONCILIATION_REQUIRED=YES
PLAT012_RUNTIME_CUSTODY_CHANGE_REQUIRED=NO
SECURE_CAPTURE_ENGINE_REMOVAL_AUTHORIZED=NO
CAPTURED_BACKEND_REMOVAL_AUTHORIZED=NO
TEST_MIGRATION_REQUIRED=YES
IMPLEMENTATION_OWNER_REQUIRED=YES
```

I064 / `guillermomolina/protos#667` owns the bounded migration.

## Ratified result

```text
D171_STATUS=RATIFIED
SELECTED_CANDIDATE=B

FILESYSTEM_ENTRIES=KEEP
PUBLIC_FILESYSTEM_CAPTURE_TREE=REMOVE_NOW_RECONSIDER_LATER
PUBLIC_ARBITRARY_SUBTREE_CAPTURE_CONTRACT=REMOVE_NOW_RECONSIDER_LATER

PLAT012_VERIFIED_IMMUTABLE_PACKAGE_CUSTODY=KEEP
SECURE_RECURSIVE_CAPTURE_ENGINE=KEEP
IMMUTABLE_CAPTURED_BACKING=KEEP
READ_ONLY_CAPTURED_FILESYSTEM_MATERIALIZATION=KEEP
VERIFY_AND_USE_SAME_CUSTODY=KEEP
NO_SOURCE_REOPEN=KEEP
PACKAGE_TOCTOU_RESISTANCE=KEEP

D046_PUBLIC_CAPTURETREE=SUPERSEDED_BY_D171
D046_INTERNAL_CAPTURE_DESIGN_EVIDENCE=RETAINED

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
