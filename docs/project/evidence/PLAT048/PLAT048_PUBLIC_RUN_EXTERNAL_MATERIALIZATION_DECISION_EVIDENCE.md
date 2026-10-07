# PLAT048 — public-run external materialization decision evidence

Date: 2026-10-04

Nature: immutable decision-investigation and approval evidence; non-normative

Formal owner: `PLAT048 / guillermomolina/protos#789`

Implementation consumer: `TOOL001-F2E5 / guillermomolina/protos#93`

## Approval provenance

The project owner explicitly approved the exact recommendation after the
comparative investigation:

~~~text
Apruebo PLAT048 Candidate B′
~~~

Result:

~~~text
PLAT048_STATUS=RATIFIED
SELECTED_CANDIDATE=B_PRIME_RUN_BOOTSTRAP_OWNED_EXACT_MATERIALIZATION_PROVIDER
IMPLEMENTATION_AUTHORIZED=YES
STOP_FOR_EXPLICIT_OWNER_APPROVAL=SATISFIED
~~~

## Product authority

~~~text
CURRENT_HEAD=e7b2ae2cc6688d4ec647306ab3fec270a5687976
CURRENT_HEAD_SUBJECT=TEST008-B: portable Java slow-test admission (PLAT047)

AUDITED_F2E_REVISION=0ce6a30635cc1db8fa27b6834bb04aab47769ac2
AUDITED_F2E_SUBJECT=TOOL001-F2E4: add mixed PackageExecutionPlanV2 module resolver with lazy verified external source loading

INTERVENING_F2E_SURFACE_CHANGE=NO
~~~

The only commit between the F2E4 audit revision and current HEAD is TEST008-B.
Its changed paths are Makefile, pom.xml and Java slow-test admission policy files;
no F2E/package-execution source changed.

## Production-boundary audit

Current production mechanisms were classified as follows:

| mechanism | classification | PLAT048 role |
| --- | --- | --- |
| `ProtosWorkspaceRunDriver` | PRODUCTION_MECHANISM | current public workspace-only run composition; no external provider |
| `ProtosWorkspacePackagePreflight` | PRODUCTION_MECHANISM | project Filesystem + generation-1 Package Tool preflight |
| `ProtosPackageContentVerification` | PRODUCTION_MECHANISM | starts from an already-selected root; capture + verify only |
| `ProtosCapturedFilesystemCustody` | PRODUCTION_MECHANISM | owns immutable captured backing after selection |
| `ProtosExternalPackagePlanningPreflight` | PRODUCTION_MECHANISM | V2 planning over already-verified borrowed custodies |
| `ProtosExternalPackageResourceScope` | PRODUCTION_MECHANISM | exact identity -> verified custody after F2E2/F2E3 |
| `ProtosPackageExecutionPlanV2ModuleResolver` | PRODUCTION_MECHANISM | mixed runtime resolution over the reconciled scope |
| `ProtosStandardFilesystemProtocol` | PRODUCTION_MECHANISM | general Filesystem mechanism, not an exact package-store index |
| temporary roots passed by tests | TEST_FIXTURE | evidence of composability, not public materialization architecture |
| package-store/vendor/multi-cache prose | DESIGN_PROSE | future-compatible design, not current production authority |
| exact public-run materialization provider | ABSENT | blocker resolved by PLAT048 |

Exact audit conclusion:

~~~text
CURRENT_HEAD_CAN_DERIVE_EXACT_EXTERNAL_IDENTITIES=YES
CURRENT_HEAD_CAN_SELECT_EXACT_MATERIALIZED_EXTERNAL_ROOTS_WITHOUT_NEW_POLICY=NO
~~~

A general Filesystem does not close the gap by itself. The missing operation
requires an identity-to-root index/layout/provider policy that did not exist.

## Exact decision question

For public `protos run <entry> [args...]`, which explicit production host
authority maps each locked exact external identity:

~~~text
(kind, PackageId, exact version/revision, ContentIdentity)
~~~

to one already-selected local materialized package root without ambient/global
lookup, name scanning, solving, fetch, lock mutation or CLI-contract change?

## Candidate results

### A — caller-supplied exact provider/map

Correct and explicit for embedders/tests, but incomplete for the public launcher:
some public bootstrap still has to construct the provider. Completing that
bootstrap converges on B or D.

### B — run-bootstrap-owned host provider

Selected after refinement to B′. It preserves the clean split:

~~~text
Package Tool = exact package policy and requirements
host         = irreducible physical authority
plan         = no physical Path
application  = no store authority
~~~

### C — Package Tool store-read capability

A generic Filesystem alone cannot perform exact store lookup without new layout
or index policy. Giving the Tool that physical convention is broader than F2E5
needs. Replacing the general Filesystem with an exact lookup capability makes the
architecture materially equivalent to B plus an unnecessary guest/host bridge.

### D — canonical host/CLI store location/layout

Mechanically simple but prematurely freezes physical topology and increases
migration cost for multiple stores, vendor/offline, system caches, CI caches and
CAS/distributed backends.

### E — ambient/global lookup

Disqualified. It conflicts with the existing rule that machine presence never
creates dependency visibility and with the exact root lock as graph authority.

No genuinely distinct sixth authority topology was found.

## Comparative scoring summary

Scale 1–5. Confidence H/M/L.

| criterion | A | B′ | C | D |
| --- | ---: | ---: | ---: | ---: |
| correctness / invariants | 5/H | 5/H | 5/M | 5/H |
| Protos alignment | 4/H | 5/H | 4/M | 3/H |
| authority discipline | 5/H | 5/H | 4/H | 4/H |
| failure/concurrency correctness | 4/M | 5/H | 4/M | 4/H |
| present-need proportionality | 5/H | 5/H | 3/H | 5/H |
| incremental growth | 4/M | 5/H | 4/M | 2/H |
| future-option resilience | 4/M | 5/H | 3/M | 2/H |
| scalability | 4/H | 5/H | 4/H | 4/H |
| implementation simplicity | 5/H | 4/M | 3/M | 5/H |
| API/interaction simplicity | 3/H | 4/H | 3/M | 5/H |
| conceptual simplicity | 4/M | 5/H | 3/M | 4/H |
| portability / freedom | 5/H | 5/H | 4/M | 2/M |
| runtime/resource cost | 5/H | 5/H | 4/H | 5/H |
| failure/operability | 3/M | 5/H | 4/M | 4/H |
| deferral/reversibility | 4/M | 5/H | 3/M | 2/H |
| evidence maturity / risk | 5/H | 4/H | 4/H | 5/H |

The recommendation was not selected by arithmetic total. B′ uniquely establishes
the required authority boundary now while keeping physical store topology and
future acquisition policy outside current package semantics.

## Incremental-design result

Smallest sufficient capability:

~~~text
already-derived complete exact identity
    ->
read/select one already-present local root
~~~

F2E5 does not need:

~~~text
fetch
network
credentials
store writes
GC
multi-store public configuration
vendor semantics
registry discovery
fresh solving
lock mutation
~~~

Those are deferred because each can be added behind or adjacent to the selected
provider boundary without changing PackageId, ContentIdentity, lock format,
PackageExecutionPlanV2, ModuleKey, public CLI or application authority.

## Prior-art evidence

The investigation compared materially different package/store designs:

- Cargo: lock identity separated from CARGO_HOME cache/source replacement/vendor;
- Go modules: module-path+version identity separated from GOMODCACHE/vendor;
- Nix: explicit store abstraction and alternate store backends, while noting that
  Nix store paths themselves are more semantic than Protos wants;
- Bazel: repository cache, distdir and vendor as separate materialization sources;
- Maven: canonical local-repository layout as evidence that Candidate D is viable
  but creates a durable ecosystem commitment;
- Gradle: dependency metadata separated from checksum-keyed cache and shared
  read-only cache;
- OCI image layout: digest-addressed blobs as evidence for replaceable physical
  backing without making ContentIdentity alone the complete Protos package key.

The convergent lesson was separation of logical dependency identity from physical
local storage, with mature systems commonly needing more than one materialization
source over time.

## Failure and ownership result

~~~text
provider unavailable/miss
    -> fail before application; close prior verified custodies

selected root unavailable
    -> fail before application

ContentIdentity mismatch / Nth verification failure
    -> F2E2 closes unsuccessful new custody
    -> orchestrator closes all prior successful custodies

V2 planning/detach failure
    -> orchestrator closes all verified custodies

scope reconciliation failure
    -> no ownership transfer
    -> orchestrator closes all verified custodies

successful reconciliation
    -> scope owns all custodies
    -> application executes
    -> application Process terminates
    -> scope closes
~~~

The application Process never observes package-store/provider authority.

## Future stress result

B′ survives:

- one or thousands of exact external nodes;
- several versions of one PackageId;
- identical ContentIdentity under distinct logical identities;
- offline CI;
- vendored material through a future explicit provider;
- per-user and read-only system caches;
- future multiple stores;
- concurrent runs;
- Native Image;
- non-JVM hosts;
- future CAS/distributed backing;
- later remote acquisition and store GC.

If future GC requires pinning during capture, the host-only selected-root result
can become a selected-materialization lease/handle without changing durable
package identity or application authority.

## Implementation handoff

Ratification releases exactly:

~~~text
Package Tool exact-requirements preflight
    ->
host exact materialization provider
    ->
one initial private read-only local backend
    ->
F2E2 verify/capture all
    ->
F2E3 plan
    ->
V2 defensive detach
    ->
F2E4 scope reconciliation
    ->
mixed application execution
    ->
application termination
    ->
scope close
~~~

Workspace-only graphs remain on the generation-1 public-run route.

## Final decision packet

~~~text
PLAT048_STATUS=RATIFIED
TYPE=PLATFORM_DECISION
REPOSITORY=guillermomolina/protos

CURRENT_HEAD=e7b2ae2cc6688d4ec647306ab3fec270a5687976
AUDITED_F2E_REVISION=0ce6a30635cc1db8fa27b6834bb04aab47769ac2
INTERVENING_F2E_SURFACE_CHANGE=NO

SELECTED_CANDIDATE=B_PRIME_RUN_BOOTSTRAP_OWNED_EXACT_MATERIALIZATION_PROVIDER
RECOMMENDATION_CONFIDENCE=HIGH

PUBLIC_CLI_SPELLING_CHANGES=NO
NEW_CLI_FLAG_REQUIRED=NO
NEW_ENV_VAR_REQUIRED=NO
NEW_CONFIG_SURFACE_REQUIRED=NO
CANONICAL_STORE_LAYOUT_REQUIRED=NO
AMBIENT_GLOBAL_LOOKUP_REQUIRED=NO

HOST_AUTHORITY_OWNER=PUBLIC_RUN_HOST_BOOTSTRAP
EXACT_ROOT_SELECTOR=RUN_SCOPED_EXACT_MATERIALIZATION_PROVIDER
LOOKUP_KEY=(kind,PackageId,exact-version-or-revision,ContentIdentity)
MATERIALIZATION_RESULT=ONE_ALREADY_PRESENT_LOCAL_SELECTED_ROOT

PACKAGE_TOOL_POLICY_BOUNDARY=EXACT_REQUIREMENTS_AND_V2_PLANNING
HOST_MECHANISM_BOUNDARY=EXACT_PHYSICAL_LOOKUP_AND_CUSTODY_COMPOSITION

F2E2_COMPOSITION_CHANGED=NO
F2E3_COMPOSITION_CHANGED=NO
F2E4_COMPOSITION_CHANGED=NO
PLAT012_CHANGED=NO
D053_CHANGED=NO
D056_CHANGED=NO
D057_CHANGED=NO

POST_RATIFICATION_IMPLEMENTATION_READY=YES
POST_RATIFICATION_IMPLEMENTATION_SLICE=TOOL001-F2E5
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
IMPLEMENTATION_AUTHORIZED=YES
~~~

## References

- `guillermomolina/protos#789`
- `guillermomolina/protos#93`
- `docs/project/decisions/platform/PLAT048_PUBLIC_RUN_EXTERNAL_MATERIALIZATION_AUTHORITY_BOUNDARY.md`
- `docs/project/evidence/TOOL001/TOOL001_F2E5_PUBLIC_RUN_MATERIALIZATION_LIFECYCLE_AUDIT.md`
