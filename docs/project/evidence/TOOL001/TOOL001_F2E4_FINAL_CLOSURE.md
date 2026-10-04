# TOOL001-F2E4 final closure — mixed V2 resolver and lazy verified external source loading

Date: 2026-10-04

Nature: immutable implementation/closure evidence; non-normative

Formal owner: `TOOL001-F2E4` / `guillermomolina/protos#92`

## Exact product revision

Published Protos revision:

`0ce6a30635cc1db8fa27b6834bb04aab47769ac2`

Commit:

`TOOL001-F2E4: add mixed PackageExecutionPlanV2 module resolver with lazy verified external source loading`

Implementation version:

`0.3.182-SNAPSHOT`

## Final published slice

The final bounded F2E4 slice adds
`ProtosPackageExecutionPlanV2ModuleResolver`.

It closes the remaining D053/PLAT012 host resolver boundary:

- one detached mixed workspace/registry/Git generation-2 graph is indexed by
  exact typed NodeRef;
- importers are recovered only from canonical workspace or external ModuleKeys;
- `self:` remains within the exact importing package and does not consult exports;
- `dep:<alias>/<publicExport>` routes through the exact
  `(declaring NodeRef, alias) -> target NodeRef` edge and target exports;
- `std:` remains delegated to the supplied standard-library resolver;
- distinct registry versions, Git revisions, registry/Git kinds and distinct
  logical package identities sharing ContentIdentity remain distinct;
- workspace sources retain the pre-existing confined physical workspace rules;
- external logical modules map to exact package-relative `.protos` resources;
- external source is loaded lazily from
  `ProtosExternalPackageResourceScope`, never from an original source/store
  path;
- external bytes are decoded using strict UTF-8 reporting malformed/unmappable
  input;
- external source is returned with
  `ProtosModuleSource.fromCharacters(...)`, with no physical source path;
- the resolver borrows but never closes the run-owned external resource scope;
- no source cache, global custody registry, guest loader Process/Actor/Activation,
  solving, fetching, lock mutation or public-run integration is introduced.

Focused tests include mixed workspace/registry/Git routing, exact-version and
registry/Git distinction, same-content distinct identities, workspace/external
`self:`, exact external dependency routing, standard-library delegation,
external source immutability after source mutation/removal, strict UTF-8
rejection, missing/directory failures, scope-close failure, workspace physical
source behavior and real module-runtime integration.

## Complete F2E4 publication chain

F2E4 was intentionally implemented in three bounded product slices:

1. `39d351699b06ab8d1293e315694b8a66f13f53c8`
   (`0.3.180-SNAPSHOT`) — defensive generation-2 detach, exact external package
   identity and canonical external ModuleKey codec.
2. `2586446b00fc43425d3fa84dd70b4af513a49643`
   (`0.3.181-SNAPSHOT`) — host-neutral immutable-resource reader, run-owned
   exact external identity -> verified-custody scope and exact 1:1
   plan/custody reconciliation.
3. `0ce6a30635cc1db8fa27b6834bb04aab47769ac2`
   (`0.3.182-SNAPSHOT`) — mixed V2 module resolver and lazy strict external
   source loading.

Together these implement the F2E4 closure boundary recorded by #92.

## Validation evidence

The maintainer reports that all local tests, including the integrated test
suite, passed after publication of the exact final product revision above.

At reconciliation time GitHub reported no remote combined status checks for
`0ce6a30635cc1db8fa27b6834bb04aab47769ac2`; this record therefore makes no
remote-CI claim.

## Authority preservation

The closure preserves the already-ratified boundaries:

- D053: generation 2 remains one inert mixed exact graph; generation 1 remains
  workspace-only;
- D056: external immutable packages gain no operational ambient `path`
  dependency semantics;
- D057: no immutable external workspace/source-bundle semantics are introduced;
- PLAT012: exact external identity is the authority key; external ModuleKey is
  exact immutable package identity + internal logical module; verified custody
  is run-owned; external reads are path-independent and host-neutral; no global
  mutable authority is introduced.

No normative Protos language/specification change is introduced by F2E4.

## Closure result

`TOOL001-F2E4` is complete at
`0ce6a30635cc1db8fa27b6834bb04aab47769ac2`.

Its closure criteria are satisfied:

- generation-2 host detach is published;
- canonical exact external package/module identity is published;
- verified external custody is reconciled exactly 1:1 and retained in a run-owned
  scope;
- external package resources can be read lazily from the verified immutable
  backing;
- mixed workspace/external `self:` / `dep:` / `std:` module resolution is
  published;
- external source construction is strict, lazy and path-independent.

The remaining work belongs to `TOOL001-F2E5` / #93:

- wire already-selected/materialized/verified external packages into the public
  `protos run <entry> [args...]` path;
- create/retain the F2E4 resource scope across application execution;
- install/use the mixed V2 resolver for the application Process;
- fail before application authority begins when required external material is
  missing, corrupt or mismatched;
- close run-owned external custody only after all application execution that may
  require module/resource loading has ended; and
- publish final TOOL001-F2E closure evidence.

F2E5 must not introduce solving, fetching, lock mutation, ambient store
discovery or leakage of verification/store authority into the application
Process.
