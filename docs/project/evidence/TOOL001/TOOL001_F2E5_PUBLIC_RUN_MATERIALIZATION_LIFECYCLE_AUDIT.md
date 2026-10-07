# TOOL001-F2E5 public-run materialization/lifecycle audit

Date: 2026-10-04

Nature: immutable investigation/activation evidence; non-normative

Formal owner: `TOOL001-F2E5` / `guillermomolina/protos#93`

Triggered platform decision: `PLAT048` / `guillermomolina/protos#789`

## Exact product authority

Audited Protos revision:

`0ce6a30635cc1db8fa27b6834bb04aab47769ac2`

Commit:

`TOOL001-F2E4: add mixed PackageExecutionPlanV2 module resolver with lazy verified external source loading`

Implementation version at that publication:

`0.3.182-SNAPSHOT`

The maintainer reports that all local tests passed for the current product state used by this audit. This record makes no independent remote-CI claim.

## Investigation result

```text
RESULT=DECISION_REQUIRED

PUBLIC_RUN_CURRENTLY_V1_ONLY=YES
CURRENT_HEAD_CAN_DERIVE_EXACT_EXTERNAL_IDENTITIES=YES
CURRENT_HEAD_CAN_SELECT_EXACT_MATERIALIZED_EXTERNAL_ROOTS=NO
EXTERNAL_MATERIALIZATION_SELECTION_AUTHORITY=UNRESOLVED

F2E2_VERIFICATION_COMPOSABLE=YES
F2E3_PLANNING_COMPOSABLE=YES
F2E4_SCOPE_RESOLVER_COMPOSABLE=YES
FINAL_TEARDOWN_PLACEMENT_MECHANICAL=YES

F2E5_IMPLEMENTATION_READY=NO
NEW_DESIGN_DECISION_REQUIRED=YES
DECISION_FAMILY=PLATxxx
ALLOCATED_DECISION=PLAT048
```

## Current public-run lifecycle

Current public `protos run <entry> [args...]` remains the published workspace-only generation-1 path:

```text
ProtosCli
    |
    v
runWorkspaceApplication(...)
    |
    v
ProtosWorkspaceRunDriver.execute(...)
    |
    +--> one run-owned ProtosPolyglotRuntimeHost
    |
    +--> ProtosWorkspacePackagePreflight.build(...)
    |       |
    |       +--> read-only project Filesystem
    |       +--> Package Tool Process
    |       +--> generation-1 ExecutionPlan.build(...)
    |       +--> defensive detach
    |       `--> Process termination
    |
    v
ProtosWorkspacePackageApplicationExecution.execute(...)
        |
        +--> ProtosWorkspacePackageModuleResolver
        +--> application Process
        +--> canonical initial module execution
        `--> Process termination
```

`ProtosWorkspaceRunDriver.Request` currently carries the toolchain/project/application bootstrap inputs but no external selected-root provider, package-store authority, materialized-root map or verified custody set.

## Published F2E2/F2E3/F2E4 composition

The external execution machinery after physical materialization selection is already published:

```text
already-selected external root
    |
    v
F2E2 capture exactly once + verify locked ContentIdentity
    |
    v
VerifiedExternalPackage + run-owned custody
    |
    v
F2E3 buildV2FromVerifiedCaptures(...)
    |
    v
raw inert PackageExecutionPlanV2
    |
    v
F2E4 defensive detach
    |
    v
exact 1:1 plan/custody reconciliation
    |
    v
run-owned ProtosExternalPackageResourceScope
    |
    v
mixed PackageExecutionPlanV2 resolver
    |
    v
application Process
    |
    v
application termination
    |
    v
resource-scope close
```

The remaining lifecycle/ownership plumbing after materialization selection is mechanical under D053 and PLAT012.

## Exact blocker

Current HEAD has no production mechanism that performs:

```text
exact locked external node
    ->
already-selected exact local materialized package root
```

without inventing new policy.

F2E2 deliberately starts after this boundary. `ProtosPackageContentVerification.captureAndVerify(...)` receives a caller-selected root and performs no discovery, store scan, PackageId/version/revision selection, fetch, solve or lock mutation.

F2E3 can read the exact root-owned lock and validate complete registry/Git identity, provenance constraints and ContentIdentity against verified captures, but it requires the full verified-custody set as an input.

F2E4 deliberately removes physical external paths from the detached plan and resolver authority. The resource scope can reconcile only already-verified custody.

Repository tests that create temporary external roots and pass them directly are fixtures, not a production materialization architecture.

## Lock / identity boundary

Current HEAD can derive from the public-run project root:

- the current canonical non-stale lock;
- exact registry nodes identified by PackageId + exact ReleaseVersion + ContentIdentity;
- exact Git nodes identified by PackageId + exact revision + ContentIdentity;
- exact declaring-node + alias -> target-node edges.

It cannot locate those nodes' materialized roots without a new production authority/configuration choice.

Therefore:

```text
CAN_CURRENT_HEAD_DERIVE_EXTERNAL_NODE_IDENTITIES_FROM_PUBLIC_RUN_INPUT=YES
CAN_CURRENT_HEAD_LOCATE_THEIR_MATERIALIZED_ROOTS_WITHOUT_NEW_POLICY=NO
```

Logical package selection and physical materialization lookup must remain distinct.

## Authority alternatives exposed by the audit

The audit did not select an architecture.

The meaningful candidate families are:

1. caller-supplied exact identity -> materialized-root map/provider;
2. run-driver-private host provider/capability supplied by bootstrap;
3. explicit Package Tool store-read Filesystem/capability that returns exact selections;
4. host/CLI-owned canonical store location/layout and direct exact selection;
5. ambient/global cache lookup.

Classification at activation:

```text
A=REQUIRES_NEW_DECISION
B=REQUIRES_NEW_DECISION
C=REQUIRES_NEW_DECISION
D=REQUIRES_NEW_DECISION
E=CONFLICTS_WITH_EXISTING_AUTHORITY
```

The exact future package-store capability acquisition surface remains explicitly unfixed in current package architecture. Physical cache/store location is non-semantic and may be supplied by user/system/CI/vendor-style backends. Therefore selecting A/B/C/D changes durable host authority ownership/configuration rather than merely helper placement.

## Required invariants for PLAT048

Any selected architecture must preserve:

- complete exact external identity as lookup authority;
- no PackageId-only or version-only lookup;
- no mutable Git-ref selection;
- no ambient package/cache directory scanning;
- no locator, mirror or Git fetch URL used as executable authority;
- explicit host authority ownership;
- no store/root/custody Filesystem leakage into application Process authority;
- no solving, update or lock mutation during ordinary run;
- no remote fetch required for F2E5 closure;
- coexistence of several versions/revisions of one PackageId;
- one capture of each selected root;
- verification of the same immutable capture later used for planning and execution;
- no source/store Path reopen after verification;
- complete verification before application authority is created;
- no ownership transfer on failed resource-scope reconciliation;
- resource-scope ownership after successful reconciliation;
- custody live until all possible application imports have ended;
- scope close only after application Process termination;
- unchanged public `protos run <entry> [args...]` spelling and application argument semantics.

## RuntimeHost and teardown conclusion

RuntimeHost composition is not a blocker.

The current workspace driver demonstrates one host shared by package preflight and application. F2E2 and F2E3 separately demonstrate that verified immutable custody and inert planning results survive Package Tool Process/host termination and can be consumed by later execution domains.

No caller-supplied RuntimeHost overload is required for F2E5 closure.

The final teardown ordering is mechanical:

```text
application Process creation
    ->
mixed resolver may load external resources through borrowed scope
    ->
application execution ends/fails
    ->
application Process termination/cleanup completes
    ->
outer run owner closes resource scope
    ->
scope closes each owned custody exactly once
```

Before successful reconciliation, the run orchestrator owns each successfully verified custody individually and must close all of them on later failure. Failed reconciliation transfers no ownership.

## V1 compatibility conclusion

D053 does not require workspace-only public runs to converge immediately on V2.

F2E5 may preserve the current generation-1 workspace-only route and use V2 only when exact external nodes exist. Migrating all workspace-only runs to V2 is not required to close this blocker and should not be selected merely for code cleanup.

## Scope retained outside F2E5

F2E5 still does not require:

- remote registry discovery;
- networking/HTTP;
- credentials;
- automatic package download;
- package-store writes;
- fresh solving/update;
- lock mutation;
- archive transport;
- publication;
- package-store GC;
- overrides/patch/vendor semantics.

The blocker is narrower: read/select an already-present exact materialization through explicit production authority.

## Coordination result

`PLAT048 — Public-run external materialization authority boundary` was allocated as GitHub issue #789.

F2E5 / #93 is blocked pending explicit owner selection and durable ratification of PLAT048.

After PLAT048 ratification, the remaining F2E5 implementation can be bounded to:

```text
derive exact locked external identities
    ->
select each already-present materialized root through the ratified authority
    ->
verify/capture all externals
    ->
build + detach V2
    ->
reconcile scope
    ->
execute application with mixed resolver
    ->
terminate application
    ->
close scope
```

No provisional materialization policy is authorized by this evidence record.
