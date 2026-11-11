# DOC024-S1 — Protos-first SQLite learning-path completion

**Date:** 2026-11-11  
**Formal work:** [DOC024 #1116](https://github.com/guillermomolina/protos/issues/1116)  
**Design origin:** [D238 #1047](https://github.com/guillermomolina/protos/issues/1047), approved B1-SQL-R1  
**Implemented prerequisites:** [I144 #1050](https://github.com/guillermomolina/protos/issues/1050), [LIB028 #1051](https://github.com/guillermomolina/protos/issues/1051), [LIB029 #1052](https://github.com/guillermomolina/protos/issues/1052)  
**Prior education:** [DOC022 #1112](https://github.com/guillermomolina/protos/issues/1112), [DOC023 #1114](https://github.com/guillermomolina/protos/issues/1114)  
**PROTOS_REVISION:** `7ae162258f67553a9bbfc02bb859d3ae328c0c0d`  
**Product commit:** https://github.com/guillermomolina/protos/commit/7ae162258f67553a9bbfc02bb859d3ae328c0c0d  
**Implementation baseline:** `0.3.462-SNAPSHOT` (no new implementation-version bump or public release)  
**Evidence:** [DOC024-S1 revision-bound validation](../../evidence/DOC024/DOC024_S1_SQLITE_LEARNING_PUBLICATION_VALIDATION_2026_11_11.md)

## Distinct documentary purpose

DOC024 completes a public, task-oriented path from the implemented Protos-first `std:sqlite` library to ordinary application usage, beyond LIB029's initial explanatory guide, one-table example, and introductory tutorial. It is documentation and runnable educational material, **not** a second SQLite implementation or a new API/ABI decision.

The chain is:

1. D238: ratified optional typed Protos-first FFI over host-authorized native authority.
2. I144: shipped a generic Java 25 FFM external provider with the explicitly authorized `ffi:sqlite3` logical alias, private native handling, lifetime and admission checks.
3. LIB028: implemented public ordinary-Protos `std:ffi` descriptors, binding Closures, output Arrays and owned release.
4. LIB029: implemented `std:sqlite` in Protos source with in-memory Connection/Statement, numeric/NULL input binds, copied TEXT/BLOB output and deterministic Future-returning close.
5. DOC022/DOC023: preserved the private provider learning path and added the public `std:ffi` learning path.
6. DOC024: added practical library-level SQLite learning programs and reconciled documentation, without turning private FFI operations into public APIs.

## Published DOC024-S1 deliverables

GitHub confirms 16 changed paths: thirteen existing paths and three new `.protos` learning programs.

**New educational programs**

- `protos/tutorials/12-system-resources/06-sqlite-query-lifecycle.protos`: `ensure`-guarded Connection/Statement scope, table creation, three fresh prepared numeric/NULL inserts, deterministic ordered `each` traversal, error boundaries, idempotent close Futures, child retirement on parent close.
- `protos/examples/ffi/sqlite-prepared-numeric.protos`: five numeric inserts, two-bound predicate with ordering and independent NULL count, including Float/NULL assertions and explicit statement cleanup.
- `protos/examples/ffi/sqlite-copied-values.protos`: SQL-literal TEXT/BLOB output (including multibyte UTF-8, embedded NUL, empty BLOB versus NULL), mutable independent Bytes copies retained through step, statement close and connection close.

**Documentation and integration**

- Expanded `docs/guide/library/sqlite.md` into an end-to-end getting-started, query, error, custody and lifetime guide, with commands, supported platform and security boundaries.
- Updated `docs/guide/library/ffi.md` and `docs/guide/interop/ffm-provider.md` to distinguish the now-published `std:sqlite`, public `std:ffi` and private I144 provider; removed obsolete `future LIB029` claims in pre-existing tutorial/example comments.
- Reconciled guide, interop, library, tutorial and examples navigation, including `protos/examples/ffi/README.md`.
- `dist/test_external_foreign_providers.py` registers three opt-in silent assertion cases: `DOC024_TUTORIAL_SQLITE_LIFECYCLE`, `DOC024_EXAMPLE_NUMERIC`, `DOC024_EXAMPLE_COPIED_VALUES`, using the existing optional provider setup and preserving earlier I144/DOC022/DOC023/LIB029 results and fail-closed UNSUPPORTED distinctions.

No code under `protos/lib/` or `src/` or normative `spec/` path was changed; the implementation `pom.xml` stays at `0.3.462-SNAPSHOT`.

## Validation and scope limits

The maintainer shared the complete `make test-protos-foreign` result, including **37/37 PASS**, all three DOC024 keys PASS, existing LIB029 cases PASS, distribution projection/layout/source identity checks PASS, and classfile major 69. The maintainer subsequently reported clean `git diff --check` and all local tests PASS. This last report includes `make check` and `make test` from the agreed final validation sequence but individual final console transcripts were not independently captured by this publication action. The product commit and file inventory are verified directly through GitHub.

These tests were executed by the maintainer, not by the documentation-coordination agent. No CI run, Native Image plugin capability, SQLite upstream version, or additional OS/ABI acceptance is inferred.

The library remains intentionally bounded: only `:memory:`; no arbitrary file-backed DB, generic SQL scripting with semicolons, TEXT/BLOB input binding, SQL transaction abstraction, C callbacks, native pointers as Protos values, general ORM, or in-process native sandbox. Trusted native execution requires explicit external host authorization, and only the stated Linux x86-64 LP64/JDK25 portable-JVM profile is positively exercised.

```text
FORMAL_IDENTIFIER=DOC024
SLICE=DOC024-S1
PROTOS_REVISION=7ae162258f67553a9bbfc02bb859d3ae328c0c0d
PRODUCT_CHANGED_PATHS=16
NEW_PROTOS_LEARNING_PROGRAMS=3
NEW_FFM_HARNESS_KEYS=3
FFM_TOTAL_RESULTS=37_OF_37_PASS_MAINTAINER_TRANSCRIPT
DOC024_CASES=3_OF_3_PASS_MAINTAINER_TRANSCRIPT
MAKE_CHECK=PASS_MAINTAINER_REPORTED
MAKE_TEST=PASS_MAINTAINER_REPORTED
DIFF_CHECK=CLEAN_MAINTAINER_REPORTED
IMPLEMENTATION_VERSION=0.3.462-SNAPSHOT_UNCHANGED
INDEPENDENT_CI=NOT_CLAIMED
```
