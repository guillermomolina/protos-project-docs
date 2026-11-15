# I167 — Protos-first module import production cutover: source and validation evidence

**Date:** 2026-11-15 (Europe/Madrid).
**Issue:** [I167 — production implementation, #1125](https://github.com/guillermomolina/protos/issues/1125).
**Architecture:** [PLAT075 #1124](https://github.com/guillermomolina/protos/issues/1124); [PLAT025 #368](https://github.com/guillermomolina/protos/issues/368) continues to own the Bytecode child/C-prime composition.
**Exact published implementation revision:** [`guillermomolina/protos@86f705237c15f51717dbcadeb6a6fa49d318b3ed`](https://github.com/guillermomolina/protos/commit/86f705237c15f51717dbcadeb6a6fa49d318b3ed).
**Published commit subject:** `I167: Promote Protos-first module import to production`.
**Parent baseline:** [`5b5580ff49e2e3a379c8eb0a63d2906d98a43800`](https://github.com/guillermomolina/protos/commit/5b5580ff49e2e3a379c8eb0a63d2906d98a43800).
**Declared product version in the published commit:** `0.4.126-SNAPSHOT` (`pom.xml` and `CHANGELOG.md` updated in the publication commit).

## Provenance and result boundaries

The published Git commit and its changed files were inspected through the connected GitHub repository. This establishes the exact source revision and that the production cutover was committed and pushed to `main`; it is **not**, by itself, CI or runtime test evidence.

The project owner expressly reported on 2026-11-15: **"I167: Promote Protos-first module import to production" pushed; `git diff --check` clean; all tests passing locally**. The 101-test Maven focal run was supplied as console output in the conversation; the 16/16 Test Tool conformance output was supplied separately. The complete integrated-suite success and whitespace check are **owner-reported**; a full integrated-suite transcript, CI run ID and Native Image evidence were not supplied here.

The targeted Maven output identifies `0.4.125-SNAPSHOT` at its invocation. The final product commit advances `pom.xml` to `0.4.126-SNAPSHOT`. Do not claim that the captured Maven command ran *after* the version change: it validates the feature candidate before the version-only publication step. The owner confirms the final local validation/publication process, but no full independently captured post-version-bump test log is asserted.

## Source-backed implementation confirmed at the exact product SHA

1. **Guest policy:** `protos/lib/core/import.protos` defines the ordinary source-backed `import.call` closure. It uses `String.recognizes`, then private `resolve`, `lookup`, and on a null cache miss `begin` followed by `execute`. The returned result is the MODULE object, SOURCE Context or FOREIGN facade. Cyclic INITIALIZING hits reuse the existing module instance.
2. **Bootstrapping:** `ProtosCoreBootstrap.java` constructs the owning module runtime before loading source `import.protos`, installs the private lexical bridge, keeps that lexical anchor private, validates/freezes the source-defined facility without replacing `call`, and binds the runtime to the Prelude once.
3. **Private bridge:** `ProtosStandardImportProtocol.java` contains four private native primitive closures, `resolve`, `lookup`, `begin`, `execute`, not a native public `import.call`. It preserves scoped single-use resolved/pending tokens, authority before cache observation, and an inert `begin` that inserts no INITIALIZING record.
4. **Structured execution:** `ProtosPreparedCalls.java` admits the exact private `ImportExecuteBody` implementation after ordinary selection; `claimImportAttempt` revalidates and delegates to existing `prepareDecidedCacheMiss` and `ModuleInitializationCall`. The existing `RunModuleInitialization`/`FailModuleInitializationUnlessCompleted` C-prime machinery remains the physical continuation owner.
5. **Module runtime/Prelude:** `ProtosModuleRuntime.java` removes obsolete native-public-import entry points, retaining shared source/MODULE/FOREIGN canonical loading, RootActor bootstrap and PLAT051 semantic transfer consumers. `ProtosPrelude.java` retains the private per-instance, one-time module-runtime association; it is never a public binding.
6. **Runtime machinery left intact:** `ProtosStructuredDispatchLowerer.java` and `ProtosActorModuleState.java` were not modified by the production commit. C-prime completion/failure, per-Actor cache and module states are reused rather than replaced.

The exact committed delta contains **13 files**: `protos/lib/core/import.protos`, `protos/tests/conformance/modules/module-source-separation.protos`, six production Java files (`ProtosPreparedCalls.java`, `ProtosCoreBootstrap.java`, `ProtosModuleRuntime.java`, `ProtosStandardImportProtocol.java`, `ProtosPrelude.java`), four JUnit test files (`ProtosPerf010APreparedTargetSpecializationTest.java`, `ProtosPlat075ProtosFirstImportFeasibilityTest.java`, `ProtosForeignModuleImportTest.java`, `ProtosCoreNativeBoundaryArchitectureTest.java`, `ProtosModuleRuntimeTest.java`), and the two publication metadata files `pom.xml` and `CHANGELOG.md`.

## Locally supplied test evidence

| Gate | Provenance | Result |
| --- | --- | --- |
| `mvn compile test-compile` | Owner-supplied console, 2026-11-15 05:40:35Z | **BUILD SUCCESS**; main/test Java compiled |
| Nine-class focused `mvn test -Dtest=...` | Owner-supplied console, 2026-11-15 05:41:22Z | **101 tests, 0 failures, 0 errors, 0 skipped; BUILD SUCCESS** |
| `bin/protos test --file protos/tests/conformance/modules/module-source-separation.protos` | Owner-supplied console after package, 2026-11-15 05:47:11Z | **`conformance/object-model` 16/16 passed**, 0 failed; includes four new cases |
| Integrated tests over final local source | Explicit owner statement, 2026-11-15 | **All tests pass locally**; full command/output not archived with this record |
| `git diff --check` | Explicit owner statement, 2026-11-15 | **Clean**; not independently executed here |
| Public revision/push | GitHub commit and `main` inspection | **VERIFIED at exact SHA `86f705237c15f51717dbcadeb6a6fa49d318b3ed`** |

The focal Maven command selected these nine classes:

```text
ProtosPlat075ProtosFirstImportFeasibilityTest
ProtosModuleRuntimeTest
ProtosCoreNativeBoundaryArchitectureTest
ProtosPerf010APreparedTargetSpecializationTest
ProtosForeignModuleImportTest
ProtosPerf006B6A5ModuleInitializationCPrimeBridgeTest
ProtosRootActorServiceBootstrapIntegrationTest
ProtosCanonicalInitialModuleExecutionTest
ProtosCoreBootstrapTest
```

The adversarial group remained **23/23 PASS** within the final 101-test run. The retained coverage includes PLAT051 Prelude lookup, FOREIGN, MODULE/SOURCE separation, alias/cycle identity, authorization, token isolation, failure eviction/retry, suspended module continuation and cancellation.

The Protos conformance source adds four tests for non-String values, a String-delegating non-semantic impostor, ordinary shadowing of the `import` binding and repeated exact instance identity. The Test Tool groups this source in `conformance/object-model`.

## GITHUB021 ratification implementation reconciliation

| Approved design invariant | Implementation disposition |
| --- | --- |
| Public standard `import.call` policy Protos-first | Implemented by the source-backed Core closure in the published revision |
| One normal production path, no native public `StandardImportBody` | Old native body and experiment flag/alternate bootstrap removed |
| Private host resolution and authorization before Actor-cache hit | Kept in bridge and shared module runtime |
| Actor-local canonical identity and cache-before-execute cycles | Kept; `begin` inert and claimed `execute` inserts record before body/acquisition |
| MODULE/SOURCE and FOREIGN independently authorized results | Shared runtime frames and foreign facade lifecycle retained |
| Post-selection exact structured implementation recognition | Private `execute` body only, not selector text |
| PLAT025 sole child-root/C-prime continuation authority | Retained; no new ContinuationResult/Task/replay ABI |
| PLAT051, RootActor and host-only loaders | Private Prelude binding and existing internal canonical loading retained |
| Native-selector audit | Updated from 1 public import native Closure to 4 private bridge primitives; focal audit passed |
| Performance parity | Explicitly not an owner-required gate; no benchmark claimed |

**Closure assessment:** I167 production source is published; focal compilation/JUnit and Protos conformance are demonstrably successful in owner-supplied output; owner attests full local tests and clean diff. No Native Image or CI pass, performance equivalence or independent reproduction is claimed.

**Status:** `I167=IMPLEMENTED_AND_PUBLISHED`; `SOURCE_SHA=86f705237c15f51717dbcadeb6a6fa49d318b3ed`; `FOCAL_JUNIT=101/101_PASS`; `PROTOS_MODULE_CONFORMANCE=16/16_PASS`; `INTEGRATED_LOCAL=OWNER_ATTESTED_PASS`; `DIFF_CHECK=OWNER_ATTESTED_CLEAN`; `NATIVE_IMAGE=NOT_EVIDENCED`.
