# LIB028-S1 — Protos-first `std:ffi` R1 implementation, publication and validation

**Date:** 2026-11-10
**Formal owner:** [LIB028 #1051](https://github.com/guillermomolina/protos/issues/1051)
**Approved design:** [D238 #1047](https://github.com/guillermomolina/protos/issues/1047), [ratified D238-B1-SQL-R1](../../decisions/language/D238_PROTOS_FIRST_NATIVE_FFI_B1_SQL_R1_RATIFIED.md)
**Provider prerequisite:** [I144 #1050](https://github.com/guillermomolina/protos/issues/1050), [I144 evidence](../I144/I144_EXTERNAL_FFM_PROVIDER_IMPLEMENTATION_PUBLICATION_2026_11_10.md)
**PROTOS_REVISION:** `4cfec564959db9be86486e332234b5c27ec9aad6`
**Published source:** https://github.com/guillermomolina/protos/commit/4cfec564959db9be86486e332234b5c27ec9aad6
**Implementation version:** `0.3.461-SNAPSHOT`
**Normative specification revision:** `0.1.502`
**Next independent consumer:** [LIB029 #1052](https://github.com/guillermomolina/protos/issues/1052)

## Published scope — verified against GitHub

Commit [`4cfec56495`](https://github.com/guillermomolina/protos/commit/4cfec564959db9be86486e332234b5c27ec9aad6) has subject `LIB028-S1: Implement the Protos-first std:ffi R1 API` and publishes the ordinary Protos Standard Library implementation, normative specification, tests, examples, documentation, and version/changelog finalization. This is a source commit, not a public release.

- `protos/lib/ffi/module.protos` implements `std:ffi` using ordinary Protos objects: frozen `void`, `i32`, `i64`, `f64`, `sizeT`, `utf8z` descriptors; `opaque`, `nullable`, `owned`, `out`, `bind`, and `release`. Parameter Arrays are snapshotted; each binding yields a new ordinary Closure; `out` results become fresh ordinary Arrays.
- Private Actor-local `IdentityMap` metadata identifies descriptor origin and routes owned-resource release. Native custody is retained by the I144 provider, including resources acquired before failed result conversion. Release retires the public association before attempting the destructor.
- `import("std:ffi")` grants no FFM authority, imports no native library, and initializes no native session. Native libraries are separately host-authorized through the optional I085/I144 `ffi:` provider.
- `spec/semantics/FFI.md`, reconciled `MODULES.md`, `VALUES_AND_COLLECTIONS.md`, `CALLABLES.md`, `ERRORS.md`, `ACTORS.md`, `PARALLEL_EXECUTION.md`, and the specification changelog define the public R1 contract at revision `0.1.502`.
- `protos/tests/library/ffi/declarations-and-denial.protos` checks ordinary import, descriptor identities/immutability, surface and denied authority without the FFM provider. `protos/tests/foreign-provider/ffm-public-api.protos` and `ProtosI144FfmFixtureAbiTest` exercise the actual public ABI against the test-only native fixture, including scalars, width/range checks, UTF-8/NUL, arity, `out`, owned resources, errors and cleanup.
- Test Tool corpus/SuiteGraph/CLI registrations and Java wiring were updated. `dist/test_external_foreign_providers.py` adds `I144_LIB028_PUBLIC_VERSION` to exercise `protos/examples/ffi/sqlite-version-public.protos` through the portable distribution.

## Validation and provenance

| Gate | Observed basis |
| --- | --- |
| Product publication | **VERIFIED** through GitHub at exact `PROTOS_REVISION` |
| Version / specification | **VERIFIED** published `CHANGELOG.md`/`pom.xml` (`0.3.461-SNAPSHOT` / Java 25), spec changelog (`0.1.502`) |
| Focal Java tests | **PASS**, maintainer supplied Maven output: 67 run, 0 failures, 0 errors, 0 skipped, BUILD SUCCESS |
| Portable JVM distribution | **PASS**, maintainer supplied `DIST_BUILD: PASS`, classfile major 69, archive CRC/layout/runtime/launcher checks PASS |
| Pre-commit distribution artifact | `protos-0.3.460-SNAPSHOT-posix-jvm.zip`, built **before** the final version bump; not evidence of a 0.3.461 distribution build |
| External FFM provider harness | **PASS**, maintainer reports all tests green; intermediate harness progress reported 23 cases, final per-case transcript not supplied |
| Integrated local tests | **PASS**, explicit maintainer statement: `todos los tests pasan en local`; exact final suite aggregate not supplied |
| `git diff --check` | **CLEAN**, maintainer reported |
| Separate `make check` exit status | **NOT INDEPENDENTLY EVIDENCED** by the final message |
| CI / independent agent test run | **NOT CLAIMED** |
| Native Image dynamic external FFM plugin | **OUT OF SCOPE** |
| Documentation-repository `git diff --check` | **NOT EXECUTED** via connector; newly authored text inspected for whitespace |

The user/maintainer supplied the focal Maven and distribution logs and later reported the implementation pushed with a clean diff and all tests passing locally. GitHub verifies the product source commit and files, **not** the independent execution of those tests.

## Boundaries and follow-up

R1 remains Protos-first: the optional Java 25 FFM provider is generic I144 machinery; the public declaration layer and final Arrays are implemented in Protos. No new runtime callable family, Core type, implicit native authority, C callbacks, arbitrary native addresses, arbitrary shared-library paths, or Native Image plugin loading is introduced. `FFI.copy`, `FFI.borrowed` and `FFI.preserving` are not public R1 operations. Borrowed-memory copying remains provider-private.

SQLite connections, statements, numeric bindings, borrowed text/blob copying, high-level `Closable` lifecycle and full SQLite integration remain owned by independent [LIB029 #1052](https://github.com/guillermomolina/protos/issues/1052). Its implementation or tests are not claimed by this evidence.

```text
FORMAL_IDENTIFIER=LIB028
SLICE=LIB028-S1
PROTOS_REVISION=4cfec564959db9be86486e332234b5c27ec9aad6
IMPLEMENTATION_VERSION=0.3.461-SNAPSHOT
SPEC_REVISION=0.1.502
PRODUCT_PUBLICATION=PASS_GITHUB_VERIFIED
FOCAL_JAVA_TESTS=67_PASS_OWNER_OUTPUT
DISTRIBUTION_BUILD=PASS_OWNER_OUTPUT_PRE_VERSION_BUMP
EXTERNAL_FFM_HARNESS=PASS_OWNER_REPORTED
INTEGRATED_LOCAL_TESTS=PASS_OWNER_REPORTED
GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
MAKE_CHECK=NOT_SEPARATELY_VERIFIED
LIB029=INDEPENDENT_NOT_IMPLEMENTED
```
