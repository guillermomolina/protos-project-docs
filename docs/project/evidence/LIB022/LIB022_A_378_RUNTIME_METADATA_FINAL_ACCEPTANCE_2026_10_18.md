# LIB022-A — `std:runtime` lazy metadata: implementation and acceptance (2026-10-18)

**Owner:** [LIB022-A / #923](https://github.com/guillermomolina/protos/issues/923); parent [LIB022 / #921](https://github.com/guillermomolina/protos/issues/921).  
**Approved public contract:** [D205 / #922](../../decisions/language/D205_LIB022_RUNTIME_INTROSPECTION_B2_PLUS_PUBLIC_CONTRACT.md), owner-ratified Candidate B / B2+.  
**Final product revision:** [`74c4d2b41102a3dfed5e382e4b62f4b2df7fa427`](https://github.com/guillermomolina/protos/commit/74c4d2b41102a3dfed5e382e4b62f4b2df7fa427) (`0.3.378-SNAPSHOT`; normative specification revision `0.1.461`).  
**Status:** LIB022-A implementation and publication **COMPLETE**. LIB022 parent **OPEN** pending the distinct `std:inspect` implementation slice.  
**Role:** pinned implementation/acceptance evidence, **not** a normative Standard Library contract or a performance benchmark.

## Published artifact and verified revision

The exact product commit contains all 21 modified/created paths, including `pom.xml` and `CHANGELOG.md`, correcting the incomplete earlier publication checkpoint at [`0298ee8755cf71dc8338193cd10d65998f03df1b`](https://github.com/guillermomolina/protos/commit/0298ee8755cf71dc8338193cd10d65998f03df1b). The current published `pom.xml` identifies `0.3.378-SNAPSHOT`, and the root implementation changelog attributes this version to LIB022-A/#923, with maintainer-reported `make test`, `make check` and `make -C build/native test` PASS. GitHub commit metadata and published source files were independently inspected; no tests were independently run by this documentation agent.

**Runtime and public surface:**

- [`protos/lib/runtime.protos`](https://github.com/guillermomolina/protos/blob/74c4d2b41102a3dfed5e382e4b62f4b2df7fa427/protos/lib/runtime.protos) exposes precisely `std:runtime.info/0`, with its private facility slot removed after module initialization.
- [`ProtosRuntimeMetadataFacility.java`](https://github.com/guillermomolina/protos/blob/74c4d2b41102a3dfed5e382e4b62f4b2df7fa427/src/main/java/com/guillermomolina/protos/execution/facilities/ProtosRuntimeMetadataFacility.java) returns a **fresh frozen ordinary Object** per call, with exactly five own String fields: `implementation`, `implementationVersion`, `engineVersion`, `platform`, `executionMode`. Context and Engine observations are made at invocation; invalid arity or unavailable metadata uses ordinary Error behavior.
- [`ProtosVersionIdentity.java`](https://github.com/guillermomolina/protos/blob/74c4d2b41102a3dfed5e382e4b62f4b2df7fa427/src/main/java/com/guillermomolina/protos/execution/facilities/ProtosVersionIdentity.java) centralizes the version fallback shared with CLI `--version`.
- [`ProtosPrelude.java`](https://github.com/guillermomolina/protos/blob/74c4d2b41102a3dfed5e382e4b62f4b2df7fa427/src/main/java/com/guillermomolina/protos/runtime/ProtosPrelude.java) and [`ProtosCoreBootstrap.java`](https://github.com/guillermomolina/protos/blob/74c4d2b41102a3dfed5e382e4b62f4b2df7fa427/src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosCoreBootstrap.java) register a **typed deferred standard member supplier**, not an eager facility. The supplier is invoked at Actor-local module initialization when that module is first loaded. Reimport uses the Actor's existing module cache; newly created private facilities are not shared frozen Prelude transfer anchors.
- [`spec/semantics/MODULES.md`](https://github.com/guillermomolina/protos/blob/74c4d2b41102a3dfed5e382e4b62f4b2df7fa427/spec/semantics/MODULES.md), [`spec/PROTOS_SPEC_CHANGELOG.md`](https://github.com/guillermomolina/protos/blob/74c4d2b41102a3dfed5e382e4b62f4b2df7fa427/spec/PROTOS_SPEC_CHANGELOG.md) and [`docs/guide/library/runtime.md`](https://github.com/guillermomolina/protos/blob/74c4d2b41102a3dfed5e382e4b62f4b2df7fa427/docs/guide/library/runtime.md) record the normative and user-facing contract.

**Regression and conformance assets:** `ProtosRuntimeMetadataFacilityTest`, `ProtosRuntimeMetadataEmbeddingTest`, `ProtosLazyStandardModuleMembersTest`, `ProtosCliTest`, the Core native-boundary regression, the TOOL011 progress-group regression, and [`protos/tests/conformance/library/runtime/info.protos`](https://github.com/guillermomolina/protos/blob/74c4d2b41102a3dfed5e382e4b62f4b2df7fa427/protos/tests/conformance/library/runtime/info.protos), wired into the conformance manifest and repository Test Tool group.

## Validation provenance and limitations

The maintainer explicitly reported that all local tests passed, `git diff --check` was clean, and the implementation was pushed. The published root changelog additionally records `make test`, `make check`, and `make -C build/native test` as PASS. These are **maintainer-reported** outcomes, not locally reproduced or independent performance measurements.

The source-level PAYG argument is bounded: the Core bootstrap stores a supplier rather than constructing the native facility on startup; no permanent Closure/Task/root registries or instrumentation listeners were introduced by this slice; module-initialization work is demand-driven. This is not evidence of zero overhead in measured nanoseconds, allocations or compiler graphs; no such benchmarks are claimed. The newly introduced Native Image platform marker is covered by the implementation and reported native gate, but no separately archived Native `info.protos` transcript or per-platform observed version string is asserted here.

## Closure boundary and next phase

**LIB022-A completed:** owner-ratified portable runtime-metadata module, lazy construction, shared CLI/distribution version policy, implementation/spec/docs/tests and required `0.3.378-SNAPSHOT` publication metadata.

**LIB022 remains open:** the approved B2+ second module `std:inspect` is not implemented by this commit. Remaining work is `protos.Inspect` authorization (default false, independent of DEBUG) and exactly `representation/1`, `callableInfo/1`, `bytecode/1`, `truffleTree/1`, following D205's on-demand preparation/materialization, ordinary Error, Context ownership, no leaked host handles and PAYG/NORMAL/DEBUG guarantees. No owner-approved new public operations, compiler-state predicates or format-stability promises are inferred.

```text
WORK_ITEM=LIB022-A
FAMILY=LIB
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=74c4d2b41102a3dfed5e382e4b62f4b2df7fa427
IMPLEMENTATION_VERSION=0.3.378-SNAPSHOT
SPEC_REVISION=0.1.461
PRODUCT_PUBLICATION=VERIFIED
GIT_DIFF_CHECK=MAINTAINER_REPORTED_PASS
MAKE_TEST=MAINTAINER_REPORTED_PASS
MAKE_CHECK=MAINTAINER_REPORTED_PASS
NATIVE_TEST=MAINTAINER_CHANGELOG_REPORTED_PASS
IMPLEMENTATION_STATUS=COMPLETE
PARENT_LIB022=OPEN
NEXT=LIB022-B_STD_INSPECT
```
