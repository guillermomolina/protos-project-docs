# AUD009-F1 — Package Tool architecture complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#652`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence baseline:
`ecf563ed01275929d5b85330e8e6259cc85d73d8`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Checkpoint proposal: `guillermomolina/protos#652`, issue comment
`5739979538`.

Owner approval provenance: `guillermomolina/protos#652`, issue comment
`5740007820`, 2026-09-19.

Derived implementation routes: **NONE**

## Explicit scope boundary

By direct project-owner instruction, this slice did **not** audit the Test Tool.

Out of scope:

```text
TOOL002 / Test Tool
TOOL005 test-corpus architecture
AUD014
D152 / D153 and successor Test Tool decisions
protos/tools/test/**
std:test/**
the active Test Tool rework
```

F1 must not be cited as approval, rejection, or evidence for changing any of
those mechanisms.

## Current Package Tool state

At the evidence baseline:

```text
TOOL001=#47 IN_PROGRESS
F2E3=CLOSED
F2E4=READY
F2E5=DEPENDENCY_GATED_BEHIND_F2E4
```

F1 audits durable architecture and continuing cost. It does not treat unfinished
TOOL001 implementation as evidence of unnecessary complexity and does not alter
TOOL001 sequencing.

## Final classification

```text
BUNDLED_PACKAGE_TOOL_BOOTSTRAP=KEEP
PACKAGE_POLICY_IN_PROTOS=KEEP
EXPLICIT_LEAST_AUTHORITY=KEEP

PROTOS_TOML_MANIFEST=KEEP
PRIVATE_BOOTSTRAP_TOML10=KEEP
MANIFEST_SCHEMA_GENERATION=KEEP

PACKAGE_RELEASE_VERSION_POLICY=KEEP
DEPENDENCY_CONSTRAINT_V1=KEEP
FRESH_RETAINED_SELECTION=KEEP
EDGE_LOCAL_RESOLUTION_MODEL=KEEP

MANIFEST_LOCK_SEPARATION=KEEP
CANONICAL_LOCKFILE=KEEP
RESOLUTION_INPUT_STALE_DETECTION=KEEP

INERT_PACKAGE_EXECUTION_PLAN=KEEP
TOOL_APPLICATION_PROCESS_SEPARATION=KEEP
PACKAGE_BACKED_RESOLVER=KEEP

CONTENT_IDENTITY=KEEP
CAPTURE_VERIFY_SAME_CAPTURE=KEEP
RUN_LOCAL_VERIFIED_CUSTODY=KEEP
STORE_LOCATION_NOT_IDENTITY=KEEP

F2E_EXTERNAL_EXECUTION_DIRECTION=KEEP
```

No F1 mechanism is classified `REMOVE_NOW_RECONSIDER_LATER` or
`REMOVE_PERMANENTLY`.

## Bundled acquisition and Protos-owned policy

The Package Tool remains an exact component of the selected toolchain rather
than a package discovered through the user's project resolution graph.

This avoids a bootstrap cycle:

```text
need package manager
    -> resolve package manager through project graph
        -> need package manager
```

Package policy remains primarily in bundled Protos code. The host retains only
irreducible bootstrap, capability and mechanical resolver responsibilities.

Moving manifest, lock, version, graph and package policy into Java would create
host-owned package semantics and reduce implementation freedom.

Classification: **KEEP**.

## Explicit least authority

Package operations continue to receive explicit bounded authority rather than one
ambient package/store/network capability.

The current architecture separates:

- confined project metadata access;
- narrow staging/publication access;
- application authority;
- immutable captured external package authority.

The complexity is primarily internal composition and buys authority isolation
without broadening ordinary program semantics.

Classification: **KEEP**.

## Manifest and bootstrap TOML

`protos.toml` remains a non-executable human-edited manifest with an explicit
schema generation.

The private shared TOML 1.0 bootstrap authority remains justified:

- Package Tool bootstrap cannot depend on project package resolution;
- the package manifest generation is explicitly pinned;
- TOML syntax/document mechanics are separate from package schema policy;
- the existence of a public TOML library does not erase this bootstrap/dialect
  boundary.

This does not require permanent implementation duplication. Future factoring may
share mechanism only if bootstrap independence and the pinned contract remain
intact.

Classification: **KEEP**.

## Package version and selection policy

Package-specific ReleaseVersion and DependencyConstraint policy remains separate
from reusable general SemVer semantics.

The Package Tool continues to own:

- its release-identity restrictions;
- dependency-constraint syntax;
- prerelease admission;
- fresh and retained selection policy;
- edge-local resolution policy.

The edge-local model permits multiple exact versions in one graph when different
dependency edges require them, while exact-equal resolved package instances
canonicalize to the same node.

A graph-wide singleton-version solver remains unjustified until a real package
semantic requires stronger coordination.

Classification: **KEEP**.

## Manifest / resolution / lock separation

The selected separation remains:

```text
protos.toml
    human intent / constraints

explicit resolution operation
    graph selection policy

protos.lock
    exact machine-owned graph + provenance/content identity
```

Normal execution does not silently resolve or rewrite dependency selection.
Canonical resolution-input identity detects stale lock state fail-closed.

Collapsing manifest and lock, or replacing semantic stale detection with
path/timestamp heuristics, would reduce reproducibility and clarity.

Classification: **KEEP**.

## Inert PackageExecutionPlan and authority boundary

The PackageExecutionPlan remains inert detached data.

It does not carry:

```text
Filesystem
captured custody
host Path
resolver handle
store authority
Process authority
```

Package Tool preflight and application execution remain separate Process/authority
domains. Application code does not inherit Package Tool authority.

The package-backed resolver mechanically realizes already-validated exact plan
data through the selected `self:`, `dep:`, and `std:` routing model.

Classification: **KEEP**.

## External immutable package verification and custody

The external package lifecycle retains:

```text
exact selected materialized root
    -> capture once
    -> immutable captured backing
    -> verify ContentIdentity over that exact backing
    -> retain run-local custody of the verified backing
    -> materialize fresh domain-local Filesystem views as needed
```

Execution does not reopen a mutable source/store path after verification.

This closes a real TOCTOU gap and preserves supply-chain integrity. The
implementation complexity is internal and does not create a new user-facing
language institution.

Exact package identity remains distinct from ContentIdentity and physical
store/cache location. Equal content may be deduplicated physically without
collapsing logical package identity.

Classification: **KEEP**.

## F2E continuation

The current external immutable-package execution direction remains justified.

```text
F2E3 verified graph construction                     KEEP
F2E4 exact identity -> verified custody/resolver     KEEP DIRECTION
F2E5 public run lifecycle completion                 KEEP DIRECTION
```

F1 does not implement, redesign or reorder F2E4/F2E5. Current TOOL001 ownership
and dependency sequencing remain authoritative.

## Deliberate absences remain absent

```text
AMBIENT_PACKAGE_SEARCH_PATH=ABSENT_RETAIN_ABSENCE
NORMAL_RUN_IMPLICIT_RESOLUTION=ABSENT_RETAIN_ABSENCE
GLOBAL_ONE_VERSION_SOLVER=ABSENT_RETAIN_ABSENCE
LIVE_AUTHORITY_IN_EXECUTION_PLAN=ABSENT_RETAIN_ABSENCE
APPLICATION_INHERITS_TOOL_AUTHORITY=ABSENT_RETAIN_ABSENCE

REMOTE_REGISTRY_PROTOCOL=ABSENT_RETAIN_ABSENCE
REGISTRY_NETWORK_ACQUISITION=ABSENT_RETAIN_ABSENCE
PACKAGE_CREDENTIAL_MODEL=ABSENT_RETAIN_ABSENCE
PUBLISH_PROTOCOL=ABSENT_RETAIN_ABSENCE
GLOBAL_STORE_SEARCH_BY_PACKAGE_NAME=ABSENT_RETAIN_ABSENCE
THIRD_PARTY_TOOL_PLUGIN_SYSTEM=ABSENT_RETAIN_ABSENCE
INSTALL_TIME_PACKAGE_SCRIPTS=ABSENT_RETAIN_ABSENCE
```

These remain deferred until concrete package operations justify them. F1 keeps
future-compatible boundaries without preimplementing speculative breadth.

## Audit lesson applied

F1 deliberately applies the E2/E4 correction:

> current narrowness, incomplete consumer rollout, or unfinished implementation
> is not sufficient removal evidence when a mechanism is a deliberate,
> low-ambient-cost growth boundary for a concrete project direction.

At the same time, this principle does not justify implementing speculative
registry, credential, publishing or plugin systems before they are needed.

## Required routing

No F1 removal/redesign route is required.

```text
REMOVAL_ROUTES=NONE
DERIVED_TOOLXXX=NONE
DERIVED_DXXX=NONE
DERIVED_PLATXXX=NONE
```

Existing TOOL001 work continues independently.

## Closure checklist

```text
OWNER_APPROVAL_PROVENANCE=PASS
EVIDENCE_BASELINE=ecf563ed01275929d5b85330e8e6259cc85d73d8

PACKAGE_TOOL_ARCHITECTURE=KEEP
F2E_EXTERNAL_EXECUTION_DIRECTION=KEEP
TEST_TOOL_AUDITED=NO
TEST_TOOL_EXCLUDED_BY_OWNER=YES

REMOVAL_ROUTES=NONE
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_F1_CLASSIFICATION=COMPLETE
```

AUD009-F1 is complete.
