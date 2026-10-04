# TOOL001-F2E4 verified-custody resource scope

Date: 2026-10-04

Nature: immutable implementation/publication evidence; non-normative

Formal owner: `TOOL001-F2E4` / `guillermomolina/protos#92`

## Exact product revision

Published Protos revision:

`2586446b00fc43425d3fa84dd70b4af513a49643`

Commit:

`TOOL001-F2E4: add host-neutral immutable-resource reader and run-owned exact external custody scope`

Implementation version:

`0.3.181-SNAPSHOT`

## Published scope

This second bounded F2E4 slice implements the PLAT012 authority/lifetime layer
over the already-published generation-2 detach and exact external identity:

- `ProtosStandardFilesystemProtocol.CapturedBackend` exposes a narrow host-only
  immutable regular-resource read projection;
- `ProtosNioCapturedTreeFilesystemBackend` implements that projection over the
  same verified captured tree, blob backing and lease model;
- `ProtosCapturedFilesystemCustody.readResource(...)` reads without
  `ProtosActivation`, guest Filesystem, Process, Actor or Context;
- `ProtosPackageResourceName` represents package-relative host resource names
  without `java.nio.Path` authority;
- absolute, empty, `.`, `..`, NUL and otherwise malformed names fail closed;
- missing entries, directories, links and opaque/other entries are not accepted
  as regular resources;
- reads use independent channels and detached byte arrays, preserving concurrent
  immutable reads without a shared mutable cursor;
- `VerifiedExternalPackage.identity()` derives the complete exact external
  package identity without consulting custody;
- `ProtosExternalPackageResourceScope.reconcile(...)` builds the run-owned
  expected-O(1) exact-identity -> verified-custody index only after exact 1:1
  reconciliation against detached `PackageExecutionPlanV2` external nodes;
- missing, extra, duplicate, mismatched identities and reuse of one custody for
  multiple exact identities fail closed;
- successful reconciliation transfers execution ownership to the scope;
- failed reconciliation leaves caller ownership intact and returns no partial
  scope;
- scope close is deterministic/idempotent and releases each owned custody once;
- F2E3 planning remains a borrower.

No external module resolver, UTF-8 source decoding, source cache, public-run
integration, solving, fetching, lock mutation or source/store Path reopen is
introduced.

## Material changed paths

Product/publication metadata:

- `CHANGELOG.md`
- `pom.xml`

Implementation:

- `src/main/java/com/guillermomolina/protos/execution/ProtosCapturedFilesystemCustody.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosExternalPackagePlanningPreflight.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosExternalPackageResourceScope.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosNioCapturedTreeFilesystemBackend.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosPackageResourceName.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardFilesystemProtocol.java`

Tests:

- `src/test/java/com/guillermomolina/protos/execution/ProtosExternalPackagePlanningPreflightTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosExternalPackageResourceScopeTest.java`

## Validation evidence

The maintainer reports that all local tests, including the integrated test
suite, passed after publication of the exact product revision above.

At reconciliation time GitHub reported no remote status checks for
`2586446b00fc43425d3fa84dd70b4af513a49643`; this record therefore makes no
remote-CI claim.

## Authority preserved

This slice is mechanical implementation of already-ratified D053 and PLAT012:

- complete exact external package identity remains typed NodeRef +
  ContentIdentity;
- physical provenance/backing/custody remains outside PackageExecutionPlan and
  ModuleKey identity;
- equal content cannot collapse distinct logical package identities;
- no global mutable custody registry exists;
- no original source/store path is reopened after verification;
- no guest execution institution is introduced merely to read source bytes;
- backing representation remains implementation detail; and
- no normative Protos language/specification semantics change.

D056 and D057 remain preserved: immutable external packages do not gain ambient
path dependencies or external workspace expansion through this host layer.

## Remaining F2E4 boundary

F2E4 remains open. Its remaining technical boundary is the external-package
module resolver/source loader over the detached V2 graph plus this resource
scope:

1. resolve `self:`, `dep:` and delegated `std:` imports across exact
   workspace/registry/Git V2 package identities;
2. preserve canonical workspace and external ModuleKey domains;
3. map validated logical modules to exact package-relative `.protos` resources;
4. lazily read external source bytes from the exact verified custody;
5. decode UTF-8 strictly; and
6. return path-independent `ProtosModuleSource.fromCharacters(...)` for
   external modules.

Public `protos run` integration and final run-resource teardown placement remain
F2E5.
