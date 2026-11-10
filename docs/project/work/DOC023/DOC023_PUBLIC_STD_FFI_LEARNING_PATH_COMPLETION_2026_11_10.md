# DOC023 — Public Protos-first `std:ffi` learning-path completion

**Date:** 2026-11-10
**Formal owner:** [DOC023 #1114](https://github.com/guillermomolina/protos/issues/1114)
**Source prerequisite:** [LIB028 #1051](https://github.com/guillermomolina/protos/issues/1051), the implemented public R1 API over [I144 #1050](https://github.com/guillermomolina/protos/issues/1050)
**Approved design:** [D238-B1-SQL-R1](../../decisions/language/D238_PROTOS_FIRST_NATIVE_FFI_B1_SQL_R1_RATIFIED.md)
**PROTOS_REVISION:** `fcae8c544347d53841b35e4cdebf69d168ca9f05`
**Published source:** https://github.com/guillermomolina/protos/commit/fcae8c544347d53841b35e4cdebf69d168ca9f05
**Implementation version:** `0.3.461-SNAPSHOT` (development source; no new release implied)
**Normative reference:** `spec/semantics/FFI.md` at existing spec revision `0.1.502` (unchanged by this work)
**Publication/validation snapshot:** [DOC023-S1 evidence](../../evidence/DOC023/DOC023_S1_STD_FFI_LEARNING_PUBLICATION_VALIDATION_2026_11_10.md)

## Distinct ownership and purpose

DOC023 completes the public-language teaching route opened after LIB028-S1. It adds no FFI semantics or runtime support: the API, declaration composition, ordinary Closures and Arrays already exist in source. The independent [DOC022 #1112](https://github.com/guillermomolina/protos/issues/1112) deliberately remains an explanation of the provider-private I144 operations, such as `signature`, `param`, `borrowed`, `preserving` and `copy`; DOC023 starts with the public `FFI.bind` and never treats those provider members as public `std:ffi` operations.

## Published learning deliverables

- [Public FFI guide](https://github.com/guillermomolina/protos/blob/fcae8c544347d53841b35e4cdebf69d168ca9f05/docs/guide/library/ffi.md): complete progression through the three layers, native host authorization, 12 public members, exact scalar types, declaration snapshots, `out` Arrays, irreversible release, Errors, borrowed-memory exclusions, Actor/P boundaries, platform limits and troubleshooting.
- [Authority-free descriptors tutorial](https://github.com/guillermomolina/protos/blob/fcae8c544347d53841b35e4cdebf69d168ca9f05/protos/tutorials/08-language-interactions/08-public-ffi-descriptors.protos): ordinary import and repeat identity, frozen scalar descriptors, invalid outputs and lookalikes; executes without a provider and prints a single success marker after assertions.
- [Public binding tutorial](https://github.com/guillermomolina/protos/blob/fcae8c544347d53841b35e4cdebf69d168ca9f05/protos/tutorials/08-language-interactions/09-public-ffi-bindings.protos): actual authorized SQLite symbols via `FFI.bind`, Closure identity, parameter snapshot, strict admission and the pre-entry Error/post-entry ForeignError distinction.
- [Public ownership/out tutorial](https://github.com/guillermomolina/protos/blob/fcae8c544347d53841b35e4cdebf69d168ca9f05/protos/tutorials/12-system-resources/05-public-ffi-owned-out.protos): `FFI.opaque`, `FFI.owned`, `FFI.out`, real `sqlite3_open_v2`/`sqlite3_prepare_v2`, numeric row inspection and ordered release.
- [SQL-completeness cookbook example](https://github.com/guillermomolina/protos/blob/fcae8c544347d53841b35e4cdebf69d168ca9f05/protos/examples/ffi/sqlite-complete-public.protos): complete/incomplete SQL, UTF-8 and input errors through the public API.
- [Numeric parameter-binding cookbook example](https://github.com/guillermomolina/protos/blob/fcae8c544347d53841b35e4cdebf69d168ca9f05/protos/examples/ffi/sqlite-numeric-query-public.protos): `sqlite3_bind_int64` and `sqlite3_bind_double` into an in-memory query, numeric row results and explicit statement/connection cleanup.
- [External-provider harness](https://github.com/guillermomolina/protos/blob/fcae8c544347d53841b35e4cdebf69d168ca9f05/dist/test_external_foreign_providers.py): four stable DOC023-specific tests reuse the original I144 provider build, shared SQLite support test, silent assertion runner and existing reporting. The 23 previous result keys remain registered; the harness now has 27.

Documentation/navigation was reconciled in `docs/guide/library/interop.md`, both library and interop READMEs, the provider guide, the general guide and modules/authority chapters, both examples indexes, the tutorial index and `providers/ffm/README.md`. Public application code is now directed to `std:ffi`; the retained private-provider documentation remains linked for backend/lifetime concepts.

## Verification, remaining boundary, and downstream

GitHub independently confirms that commit `fcae8c544347d53841b35e4cdebf69d168ca9f05` has the expected title `DOC023-S1: Complete the public std:ffi learning path` and changes 18 paths (the five new `.protos` files plus the new guide, documentation navigation and harness). The published `pom.xml` retains `0.3.461-SNAPSHOT` and JDK release 25. No files under `spec/`, `protos/lib/ffi/`, Java runtime or Java FFM provider are in the changed-file list.

The maintainer explicitly reported the DOC023-S1 source pushed, `git diff --check` clean and **all local tests passing**. The maintainer did not supply the final external-provider 27-case transcript or the exact final aggregate of `make test` / `make check` in the completion message; the report is recorded as owner-reported PASS rather than independent re-execution. The four DOC023 harness cases are registered, but individual per-case statuses are not invented in this record.

Public R1 intentionally has no `FFI.copy`, `FFI.borrowed`, `FFI.preserving`, TEXT/BLOB bindings, arbitrary library authority, native callbacks, numeric pointers or Native Image dynamic external provider. The C library runs trusted in-process and is not sandboxed. Session-level custody remains the backup for resources not explicitly released on exception, without promising a language-level `finally` or automated public `Closable` lifecycle.

[LIB029 #1052](https://github.com/guillermomolina/protos/issues/1052) independently owns the higher-level SQLite Connection/Statement library. DOC023 does not implement or validate LIB029.

```text
FORMAL_IDENTIFIER=DOC023
SLICE=DOC023-S1
PROTOS_REVISION=fcae8c544347d53841b35e4cdebf69d168ca9f05
IMPLEMENTATION_VERSION=0.3.461-SNAPSHOT
SPEC_REVISION=0.1.502_UNCHANGED
PRODUCT_SOURCE_PUBLICATION=PASS_GITHUB_VERIFIED
PRODUCT_FILES_CHANGED=18
NEW_PUBLIC_PROTOS_PROGRAMS=5
NEW_GUIDE=1
EXTERNAL_FFM_CASE_KEYS=27_REGISTERED
NEW_DOC023_FFM_CASE_KEYS=4_REGISTERED
GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
ALL_LOCAL_TESTS=PASS_OWNER_REPORTED
EXTERNAL_HARNESS_DETAILED_TRANSCRIPT=NOT_SUPPLIED
CI_OR_INDEPENDENT_REEXECUTION=NOT_CLAIMED
LIB029=INDEPENDENT_OPEN
```
