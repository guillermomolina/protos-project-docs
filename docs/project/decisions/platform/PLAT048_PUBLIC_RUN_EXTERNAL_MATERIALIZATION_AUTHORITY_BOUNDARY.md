# PLAT048 — Public-run exact external materialization authority boundary

Status: **RATIFIED**

Selected architecture: **Candidate B′ — requirements-first Package Tool preflight plus a public-run-bootstrap-owned exact materialization provider**.

Approval: explicit project-owner approval on 2026-10-04:

~~~text
Apruebo PLAT048 Candidate B′
~~~

Decision Issue: `guillermomolina/protos#789`

Primary implementation consumer: `TOOL001-F2E5 / guillermomolina/protos#93`

Product authority inspected for the decision packet:

~~~text
CURRENT_HEAD=e7b2ae2cc6688d4ec647306ab3fec270a5687976
AUDITED_F2E_REVISION=0ce6a30635cc1db8fa27b6834bb04aab47769ac2
INTERVENING_F2E_SURFACE_CHANGE=NO
~~~

Nature: durable non-normative package/runtime host architecture. Observable Protos
language, PackageId, lockfile, ModuleKey and PackageExecutionPlan semantics remain
unchanged.

## Decision

Public `protos run <entry> [args...]` obtains external materialization authority
from an explicit host-owned provider constructed by the public-run bootstrap.

The operation is deliberately narrow:

~~~text
complete exact external identity
    ->
one already-present local materialized package root
~~~

The complete lookup key is:

~~~text
kind
PackageId
exact ReleaseVersion or exact Git revision
ContentIdentity
~~~

No PackageId-only, version-only, revision-only or ContentIdentity-only lookup is
authorized.

The bundled Package Tool remains the owner of package policy. It derives and
validates the exact external requirements from the root project/lock and hands
the host inert exact identities. It does not select physical store paths.

The host provider owns only irreducible physical authority:

~~~text
exact already-selected identity
    ->
already-present local materialization
~~~

It does not solve, select versions, interpret dependency constraints, mutate the
lock, fetch, use credentials, write the package store or perform GC.

## Selected B′ topology

~~~text
public protos run bootstrap
        |
        v
Package Tool requirements preflight
        |
        | inert exact external identities only
        v
run-scoped host ExactExternalMaterializationProvider
        |
        | lookup(complete exact identity)
        v
already-present selected local root
        |
        v
F2E2 capture exactly once + ContentIdentity verification
        |
        v
verified run-owned custody
        |
        v
F2E3 generation-2 planning
        |
        v
F2E4 defensive detach + exact scope reconciliation
        |
        v
mixed application execution
        |
        v
application Process termination
        |
        v
resource scope close
~~~

Workspace-only graphs retain the existing generation-1 public-run path. PLAT048
does not select V2-for-all merely for implementation uniformity.

## Public bootstrap boundary

The real public launcher, not only tests or embedders, must construct the
provider without changing:

~~~text
protos run <entry> [args...]
~~~

F2E5 requires no new CLI flag, Protos-specific environment variable, public
configuration file, ambient directory scan or global mutable singleton.

The initial provider may use one implementation-private read-only local
materialization backend. Its concrete path, directory layout and index encoding
are not package semantics, lock ABI, PackageExecutionPlan ABI or public
configuration.

In particular PLAT048 does **not** ratify any public equation such as:

~~~text
ContentIdentity -> ~/.cache/protos/<digest>
PackageId       -> directory name
version         -> directory name
Git revision    -> directory name
~~~

The physical cache/store location remains non-semantic.

## Package Tool versus host authority

Package Tool owns:

- manifest and lock interpretation;
- stale-state validation;
- exact registry/Git node semantics;
- PackageId and exact version/revision identity;
- ContentIdentity;
- dependency edges and aliases;
- the set of exact external requirements;
- generation-2 planning over already-verified captures.

The host owns:

- provisioning the run-scoped materialization provider;
- physical lookup of an already-decided exact identity;
- handing one selected root to F2E2;
- custody cleanup and final lifecycle composition.

The application Process receives neither store/provider authority nor source
Paths, credentials, locators or Package Tool Filesystems.

## Ownership and failure

Before successful F2E4 reconciliation, the run orchestrator owns every custody
returned by successful F2E2 verification.

If the Nth lookup or verification fails:

- the newly failing F2E2 operation closes any custody it created unsuccessfully;
- the run orchestrator closes all previously successful custodies;
- no application authority is created.

If V2 planning or defensive detach fails, the orchestrator closes all verified
custodies.

Failed resource-scope reconciliation transfers no ownership. The caller therefore
closes all custodies.

After successful reconciliation, `ProtosExternalPackageResourceScope` owns the
complete reconciled custody set. It remains open while any application
module/resource load can occur and closes only after application Process
termination/cleanup.

## Existing decisions preserved

PLAT048 does not reopen or amend:

- D053 — PackageExecutionPlan ABI generation 2;
- D056 — immutable external path dependency policy;
- D057 — immutable external workspace policy;
- PLAT012 — verified external package custody/source resolution.

Consequently:

~~~text
F2E2_COMPOSITION_CHANGED=NO
F2E3_COMPOSITION_CHANGED=NO
F2E4_COMPOSITION_CHANGED=NO
PLAT012_CHANGED=NO
D053_CHANGED=NO
D056_CHANGED=NO
D057_CHANGED=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
~~~

## Rejected candidates

### A — caller-supplied provider/map as the complete public solution

Useful as an embedding/test seam, but incomplete for public run because it moves
the bootstrap question to the caller. Once the public bootstrap constructs it,
the architecture is Candidate B′.

### C — Package Tool receives a general store Filesystem

Rejected for F2E5. A general Filesystem does not by itself define
`exact identity -> materialized root`; it would require store layout/index
policy inside the Package Tool and would grant broader physical authority than
the current requirement needs.

### D — canonical host/CLI store location and layout

Rejected because it prematurely freezes physical store topology and raises the
later cost of user/system/CI/vendor/CAS backends even though location is
deliberately non-semantic.

### E — ambient/global cache lookup

Rejected by existing authority. Package presence must never create importability
or change the exact graph. Ambient name/path scanning is forbidden.

## Incremental-growth boundary

F2E5 implements only read/select authority for already-present exact material.

Plausible later capabilities remain additive:

~~~text
multiple stores       -> provider/backend composition
vendor/offline        -> another explicitly authorized provider
system/CI caches      -> read-only provider implementations
remote acquisition    -> separate acquisition layer on exact miss
store writes          -> separate materialization/writer authority
GC                    -> store lifecycle machinery
CAS/distributed       -> alternate provider/backing
~~~

Those capabilities are not preimplemented by PLAT048.

If future GC requires pinning while F2E2 captures a selected root, the host-only
provider result may evolve internally from a Path to a selected-materialization
lease/handle without changing lock identity, plan ABI, public CLI or application
authority.

## Implementation authority

Ratification releases the bounded TOOL001-F2E5 implementation in
`guillermomolina/protos`.

The implementation may:

1. expose a Package Tool preflight that returns inert exact external requirements;
2. add a host exact-materialization provider keyed only by the complete exact
   external identity;
3. provide one initial private read-only local backend sufficient for real public
   `protos run`;
4. compose external runs through existing F2E2 -> F2E3 -> F2E4 machinery;
5. preserve generation-1 public execution for workspace-only graphs;
6. prove cleanup/failure ownership and public-CLI mixed execution.

It must not add fetch, network, credentials, store writes, GC, vendor policy,
multi-store public configuration, a canonical public store layout, solving,
lock mutation or application-visible store authority.

## Ratified result

~~~text
PLAT048_STATUS=RATIFIED
SELECTED_CANDIDATE=B_PRIME_RUN_BOOTSTRAP_OWNED_EXACT_MATERIALIZATION_PROVIDER

PUBLIC_CLI_SPELLING_CHANGES=NO
NEW_CLI_FLAG_REQUIRED=NO
NEW_ENV_VAR_REQUIRED=NO
NEW_CONFIG_SURFACE_REQUIRED=NO
CANONICAL_STORE_LAYOUT_REQUIRED=NO
AMBIENT_GLOBAL_LOOKUP_REQUIRED=NO

HOST_AUTHORITY_OWNER=PUBLIC_RUN_HOST_BOOTSTRAP
EXACT_ROOT_SELECTOR=RUN_SCOPED_EXACT_MATERIALIZATION_PROVIDER
LOOKUP_KEY=COMPLETE_EXACT_EXTERNAL_PACKAGE_IDENTITY
MATERIALIZATION_RESULT=ONE_ALREADY_PRESENT_LOCAL_SELECTED_ROOT

PACKAGE_TOOL_POLICY_BOUNDARY=EXACT_REQUIREMENTS_AND_V2_PLANNING
HOST_MECHANISM_BOUNDARY=EXACT_PHYSICAL_LOOKUP_AND_CUSTODY_COMPOSITION
APPLICATION_OBSERVES_STORE_AUTHORITY=NO

V1_WORKSPACE_ONLY_PATH=PRESERVED
V2_EXTERNAL_PATH=AUTHORIZED

IMPLEMENTATION_AUTHORIZED=YES
IMPLEMENTATION_SLICE=TOOL001-F2E5
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
~~~

## Evidence and references

- `guillermomolina/protos#789` — PLAT048 decision Issue.
- `guillermomolina/protos#93` — TOOL001-F2E5 implementation owner.
- `docs/project/evidence/PLAT048/PLAT048_PUBLIC_RUN_EXTERNAL_MATERIALIZATION_DECISION_EVIDENCE.md`.
- `docs/project/evidence/TOOL001/TOOL001_F2E5_PUBLIC_RUN_MATERIALIZATION_LIFECYCLE_AUDIT.md`.
- `docs/project/decisions/platform/PLAT012_VERIFIED_EXTERNAL_PACKAGE_CUSTODY_SOURCE_RESOLUTION.md`.
