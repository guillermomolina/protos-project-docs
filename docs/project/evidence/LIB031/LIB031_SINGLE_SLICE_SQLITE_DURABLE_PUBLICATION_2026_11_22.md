# LIB031 — Single-slice Protos-first durable SQLite publication and maintainer validation

**Publication date:** 2026-11-22  
**Formal work:** [LIB031 #1120](https://github.com/guillermomolina/protos/issues/1120)  
**Approved design:** [D256 #1118](https://github.com/guillermomolina/protos/issues/1118)  
**Product revision:** [`8b275264db144e50a7039eaeeebac6cd61a7919c`](https://github.com/guillermomolina/protos/commit/8b275264db144e50a7039eaeeebac6cd61a7919c)  
**Previous product revision:** [`356fbffbff502c7d76253c07ffd6533a5196676d`](https://github.com/guillermomolina/protos/commit/356fbffbff502c7d76253c07ffd6533a5196676d)  
**Owner's exact contract ratification:** [revision-pinned D256-C1R record](../D256/D256_C1R_EXACT_PUBLIC_CONTRACT_OWNER_RATIFICATION_2026_11_22.md)  
**Prior architecture and one-slice direction:** [revision-pinned D256-C1R record](../D256/D256_C1R_OWNER_APPROVAL_MINIMAL_SLICE_2026_11_21.md)

## GitHub publication verified

GitHub's default-branch history lists `8b275264db144e50a7039eaeeebac6cd61a7919c` after `356fbffbff502c7d76253c07ffd6533a5196676d`. The compare endpoint reports **one commit ahead**, zero behind, and 16 changed files. The commit is published, not merely a local working-tree proposal.

The product `pom.xml` at `8b275264db144e50a7039eaeeebac6cd61a7919c` declares **`0.4.190-SNAPSHOT`**. The root `CHANGELOG.md` includes the corresponding `0.4.190-SNAPSHOT` LIB031/D256-C1R entry. The new `spec/semantics/SQLITE.md` is included in the published commit; see the distinct normative status caveat below.

### Published implementation and tests

- `protos/lib/sqlite/module.protos`: ordinary, lazy-import `std:sqlite` entry points; `openMemory()` retained, `openFile(path, mode: "readWriteCreate")` added.
- `protos/lib/sqlite/abi.protos`: FFI bindings for native `sqlite3_get_autocommit`, `sqlite3_busy_timeout`, and `sqlite3_db_readonly` in addition to the existing SQLite C API.
- `protos/lib/sqlite/internal/Resources.protos`: common acquisition for in-memory and file-backed Connections, effective writable-mode admission, transaction control through ordinary prepared SQLite SQL, native transaction-state observation, lock-wait configuration, existing Actor-local custody and first-error/once-only close semantics.
- `protos/tests/foreign-provider/sqlite-failure-model.protos`: deterministic failure, acquisition, transaction-state and cleanup coverage through Protos-owned ABI models.
- `protos/tests/foreign-provider/sqlite-durable.protos` plus `dist/test_external_foreign_providers.py`: real SQLite file modes, typed data and reopening across processes, locking, WAL, synchronized kill/recovery and realistic failure classification.
- `protos/tests/foreign-provider/sqlite-missing.protos` and `ffm-allowlist-denied.protos`: no implicit FFM or native-library authority for the new entry point.
- `protos/examples/ffi/sqlite-durable.protos`: reusable prepared inserts, commit/rollback, read-only reopen and assertions; `durable_demo` table.
- User-facing library/FFM guides and README navigation; new draft contract `spec/semantics/SQLITE.md`.

The connector compare confirms the exact 16 paths above plus `pom.xml`, `CHANGELOG.md`, and guide files. **No `src/main/java` or `src/test/java` paths were changed in this product commit**. This is a one-commit, one-implementation-slice outcome, not S1–S4.

## Validation evidence — maintainer-reported, not agent-run

On 2026-11-22, the maintainer supplied these local results for the LIB031 candidate:

| Gate | Reported result | Precision |
| --- | --- | --- |
| `git diff HEAD --check` | PASS / clean | User-supplied, including after documentation corrections |
| `mvn -Dtest=ProtosLib029SqliteLifecycleTest test` | 2 tests, zero failures/errors/skips; BUILD SUCCESS | User-supplied focal output |
| `make test-protos-foreign` | **84/84 PASS**; `DIST_BUILD: PASS` | User-supplied full integration output |
| `LIB031_LOCAL` | PASS | Observed integration result |
| `LIB031_REOPEN` | PASS | Observed integration result |
| `LIB031_READ_ONLY` | PASS | Observed integration result |
| `LIB031_OPEN_MODES` | PASS | Observed integration result |
| `LIB031_LOCKING` | PASS | Observed integration result |
| `LIB031_RECOVERY` | PASS | Observed integration result; process termination, not physical power failure |
| `LIB031_EXAMPLE` | PASS | Observed integration result |
| `make check`, `make test`, `make test-native` | **PASS reported by maintainer** | Reported in published product `CHANGELOG.md` / `docs/guide/library/sqlite.md` and user's current statement that all tests pass locally; raw logs for all three are not independently attached here |

The independent tool calls for this record verified the commit, changed-path inventory, product version and checked-in claims. They did **not** execute builds, native tests or suites. Do not upgrade a maintainer assertion into CI certification. No unsupported platforms were counted as PASS.

## Approved boundaries preserved

- Owner-approved Protos-first implementation; ordinary `std:sqlite` objects/Closures, no Java SQLite service, no JDBC, no generic database API.
- Existing general `std:ffi` and host-granted `--allow-library sqlite3=libsqlite3.so.0`; importing `std:sqlite` does not authorize native code.
- Native SQLite itself owns file I/O, journal/WAL, transaction commit/rollback, locking and recovery; C code executes trusted in-process with OS-process authority, **without native filesystem confinement**. PLAT074 is not planned.
- Actor-local Connections/Statements with synchronous calls and unchanged close/Future semantics. SQLite remains the only authority for transaction state after failure.
- Existing LIB029/LIB030 input, output, prepare, resource lifecycle and provider-negative behavior remains covered by the 84/84 integration campaign.
- No claims of full Native Image FFM-provider support, cross-platform guarantees, physical power-loss resilience, actor-independent native library state or automatic interruption.

## Separate unresolved normative-publication discrepancy — D256

The 2026-11-22 product commit `8b275264db144e50a7039eaeeebac6cd61a7919c` **does include** `spec/semantics/SQLITE.md`, but its header still states **“PENDING OWNER RATIFICATION AND NORMATIVE PUBLICATION.”** This statement was correct when drafted but is now stale: the owner explicitly answered **“apruebo”** to the exact six-point public contract before this product commit. The global `spec/PROTOS_SPEC_CHANGELOG.md` at `8b275264db144e50a7039eaeeebac6cd61a7919c` still has latest revision **`0.1.506`**, with **no new SQLite normative entry**.

Therefore the code/tests/version/changelog were published, while **the product-side normative text has not yet been advanced to a valid ratified specification revision**. This discrepancy must be resolved as a bounded normative-publication correction under [D256 #1118](https://github.com/guillermomolina/protos/issues/1118). It does not authorize a new LIB031 implementation slice or reinterpret the owner's accepted API.

**LIB031 closure scope:** the implementation and validated delivery in this published commit may close as completed.  
**D256 closure scope:** keep open until the stale pending header and global specification changelog are reconciled and actually published in `guillermomolina/protos`; the exact owner decision itself is already ratified in the durable record.

## Revision-bound handoff flags

`PROTOS_REVISION=8b275264db144e50a7039eaeeebac6cd61a7919c`  
`LIB031_PRODUCT_PUBLICATION=PASS`  
`LIB031_MAINTAINER_LOCAL_TESTS_REPORTED_PASS=YES`  
`LIB031_REAL_SQLITE_FFM_84_OF_84_PASS=YES`  
`LIB031_NEW_JAVA_PRODUCTION_PATHS=0`  
`D256_EXACT_API_OWNER_RATIFIED=YES`  
`D256_PRODUCT_NORMATIVE_REVISION_PUBLISHED=NO`  
`SPEC_GLOBAL_REVISION_AT_PRODUCT_COMMIT=0.1.506`
