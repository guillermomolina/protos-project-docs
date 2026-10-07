# TOOL001-F2E4 first bounded implementation slice

Date: 2026-10-03

Nature: immutable implementation/publication evidence; non-normative

Formal owner: `TOOL001-F2E4` / `guillermomolina/protos#92`

## Exact product revision

Published Protos revision:

`39d351699b06ab8d1293e315694b8a66f13f53c8`

Commit:

`TOOL001-F2E4: detach PackageExecutionPlanV2 and add exact external identity/ModuleKey`

Implementation version:

`0.3.180-SNAPSHOT`

## Published scope

This first bounded F2E4 slice implements the host-side inert identity boundary
already fixed by D053 and PLAT012:

- `ProtosPackageExecutionPlanV2Adapter` defensively detaches ordinary-Protos
  generation-2 plan data into immutable host `ProtosPackageExecutionPlanV2`;
- workspace, registry and Git refs retain exact typed identity;
- external packages carry detached `ProtosPackageContentIdentity`;
- exact external package identity is represented by
  `ProtosExactExternalPackageIdentity`;
- several exact versions/revisions of one PackageId remain distinct;
- equal ContentIdentity does not collapse distinct logical package identities;
- `ProtosExternalPackageModuleKey` canonically encodes exact external package
  identity plus internal logical module in a domain disjoint from workspace
  package keys;
- dependency aliases, exports, registry locator, Git fetch URL, source/store/cache
  paths and custody handles remain outside external ModuleKey identity; and
- the existing generation-1 workspace detach remains unchanged.

The publication does not add an external package resolver, custody index,
host-neutral source reader, source loading, public-run wiring, acquisition,
solving, fetch or lock mutation.

## Material changed paths

Product/publication metadata:

- `CHANGELOG.md`
- `pom.xml`

Implementation:

- `src/main/java/com/guillermomolina/protos/execution/ProtosExactExternalPackageIdentity.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosExternalPackageModuleKey.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosPackageContentIdentity.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosPackageExecutionPlanAdapter.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosPackageExecutionPlanV2.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosPackageExecutionPlanV2Adapter.java`

Tests:

- `src/test/java/com/guillermomolina/protos/execution/ProtosExactExternalPackageIdentityTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosExternalPackageModuleKeyTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosExternalPackagePlanningPreflightTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPackageExecutionPlanV2AdapterTest.java`

## Validation evidence

The maintainer reports that all local tests, including the integrated test
suite, passed after publication of the exact product revision above.

At the time this evidence was reconciled, GitHub reported no remote status
checks for `39d351699b06ab8d1293e315694b8a66f13f53c8`. Therefore this record makes
no remote-CI claim.

## Authority preservation

This slice is mechanical implementation of already-ratified authority:

- D053 keeps generation 1 frozen as workspace-only and generation 2 as one inert
  mixed workspace/registry/Git graph;
- PLAT012 defines complete exact external package identity as the run-time
  authority key and canonical external ModuleKey as exact immutable package
  identity plus internal logical module;
- no physical provenance, custody handle or ambient/global mutable authority is
  introduced; and
- no normative Protos specification change is made by this slice.

## Remaining F2E4 boundary

F2E4 remains open. The next bounded mechanical implementation step is:

1. expose a narrow host-neutral immutable-resource read projection over the
   already-verified captured custody, independent of guest Activation/Filesystem;
2. create the run-owned exact external package identity -> verified-custody
   scope; and
3. reconcile that scope 1:1 against the detached V2 external package set,
   failing closed on missing, extra, duplicate or mismatched identity/custody.

External module routing and lazy `ProtosModuleSource.fromCharacters(...)`
construction remain a later F2E4 slice. Public run integration and final
run-resource teardown remain F2E5.
