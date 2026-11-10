# DOC023-S1 — Revision-bound public `std:ffi` teaching publication and local validation

**Date:** 2026-11-10
**Owning Issue:** [DOC023 #1114](https://github.com/guillermomolina/protos/issues/1114)
**PROTOS_REVISION:** `fcae8c544347d53841b35e4cdebf69d168ca9f05`
**Product commit:** https://github.com/guillermomolina/protos/commit/fcae8c544347d53841b35e4cdebf69d168ca9f05
**Maintained work record:** [DOC023 completion](../../work/DOC023/DOC023_PUBLIC_STD_FFI_LEARNING_PATH_COMPLETION_2026_11_10.md)

## Independently observed product publication

Published GitHub commit `fcae8c544347d53841b35e4cdebf69d168ca9f05` has subject `DOC023-S1: Complete the public std:ffi learning path` and changes **18 files**:

```text
dist/test_external_foreign_providers.py
docs/guide/06-modules-and-imports.md
docs/guide/11-process-io-filesystems-and-authority.md
docs/guide/README.md
docs/guide/interop/README.md
docs/guide/interop/ffm-provider.md
docs/guide/library/README.md
docs/guide/library/ffi.md
docs/guide/library/interop.md
protos/examples/README.md
protos/examples/ffi/README.md
protos/examples/ffi/sqlite-complete-public.protos
protos/examples/ffi/sqlite-numeric-query-public.protos
protos/tutorials/08-language-interactions/08-public-ffi-descriptors.protos
protos/tutorials/08-language-interactions/09-public-ffi-bindings.protos
protos/tutorials/12-system-resources/05-public-ffi-owned-out.protos
protos/tutorials/README.md
providers/ffm/README.md
```

One public guide, three progressive Protos tutorials and two cookbook examples are added. Existing DOC022 backend lessons remain. The Python external-provider runner registers four new cases: `I144_DOC023_PUBLIC_BINDINGS`, `I144_DOC023_PUBLIC_OWNERSHIP`, `I144_DOC023_PUBLIC_COMPLETE` and `I144_DOC023_PUBLIC_NUMERIC_QUERY`. The source inventory confirms 27 total result keys, including the 23 prior ones. The provider-free descriptor tutorial is intentionally served by the ordinary CLI tutorial sweep rather than the FFM provider harness.

No changed path belongs to the normative specification, `protos/lib/ffi/`, the Java FFM provider or Java runtime. Published `pom.xml` is still `0.3.461-SNAPSHOT` with Java 25; there is no new public release or ABI specification revision.

## Human-executed validation — provenance faithfully scoped

| Gate | Reported/observed outcome |
| --- | --- |
| Exact source commit and 18 file paths | **VERIFIED** via live GitHub |
| New guide and five runnable Protos source files | **VERIFIED** as published source, not independently run |
| Four new FFM harness keys; total 27 registered | **VERIFIED** from published Python source |
| `git diff --check` | **CLEAN**, maintainer reported |
| Local tests | **PASS**, maintainer explicitly reported all tests green |
| Individual 27 FFM statuses / unsupported breakdown | **NOT SUPPLIED** in this completion report; not independently asserted |
| `make test` / `make check` exact separate exit statuses and counts | **NOT INDEPENDENTLY CAPTURED** in this completion report |
| CI and independently rerun binary/SQLite tests | **NOT CLAIMED** |
| Native Image external-provider loading | **OUT OF SCOPE** |
| This documentation-repository `git diff --check` | **NOT EXECUTED** through connector publication |

The agent confirmed publication paths and source semantics; the maintainer operated the local test suite and reports that all tests passed. No automated CI or copied console test log is invented.

## Accepted scope

The examples exercise public `FFI.bind`, exact scalar admission, output Arrays, native status codes as ordinary data, `FFI.owned`/`FFI.release` and SQLite numeric data. They do not introduce a public borrowed-memory API, SQLite Connection/Statement abstraction, callbacks, generic C pointers, TEXT/BLOB bindings or Native Image plugin support. I144's private provider controls and lifetime lessons remain deliberately separate from the public R1 learning path.

```text
FORMAL_IDENTIFIER=DOC023
SLICE=DOC023-S1
PROTOS_REVISION=fcae8c544347d53841b35e4cdebf69d168ca9f05
PRODUCT_COMMIT=PASS_GITHUB_VERIFIED
CHANGED_PATHS=18
NEW_TUTORIALS=3
NEW_EXAMPLES=2
NEW_DOC023_FFM_HARNESS_KEYS=4
TOTAL_FFM_HARNESS_KEYS=27
GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
ALL_LOCAL_TESTS=PASS_OWNER_REPORTED
DETAILED_HARNESS_RESULTS=NOT_SUPPLIED
CI_INDEPENDENT_RUN=NOT_VERIFIED
```
