# D256-C1R — Explicit owner ratification of the SQLite public contract

**Date:** 2026-11-22  
**Decision issue:** [D256 #1118](https://github.com/guillermomolina/protos/issues/1118)  
**Implementation issue:** [LIB031 #1120](https://github.com/guillermomolina/protos/issues/1120)  
**Earlier architectural owner approval:** [D256-C1R owner selection and minimum-slice instruction](D256_C1R_OWNER_APPROVAL_MINIMAL_SLICE_2026_11_21.md)

## Ratification event and exact scope

The project owner was explicitly asked:

> ¿Apruebas la ratificación normativa de D256 con este contrato exacto?

The question followed a six-part description of the proposed public SQLite contract, reproduced without extension in substance below. The owner's explicit response on **2026-11-22** was:

> apruebo

Therefore **D256-C1R's enumerated exact public contract is OWNER-RATIFIED** as specified in this record. This closes the former *owner-decision* gate for that presented contract. It does **not** assert that the uncommitted product-side `spec/semantics/SQLITE.md` has been independently audited word for word, that its global spec revision has been committed, that LIB031 code has been published, or that a full `make test` suite has passed.

### Ratified API and semantic boundaries

1. **Acquisition:** `SQLite.openMemory()` and `SQLite.openFile(path, mode: "readWriteCreate")`; allowed modes `"readOnly"`, `"readWrite"`, `"readWriteCreate"` with ordinary Protos default-parameter semantics.
2. **Explicit transactions:** `db.begin()`, `db.commit()`, `db.rollback()` execute standard SQLite transaction-control statements through the existing prepared-statement mechanism.
3. **Transaction state:** `db.inTransaction()` observes native `sqlite3_get_autocommit` directly, without duplicate Protos-maintained transaction state. Failed COMMIT does not guarantee rollback; SQLite remains authoritative.
4. **Native lock waiting:** `db.busyTimeout(milliseconds)` delegates to native SQLite for qualifying lock waits; it is not an arbitrary query timeout or implicit cancellation.
5. **Execution and custody:** Connection/Statement and FFM sessions remain Actor-local; native calls run synchronously within the Actor without automatic suspension; lifecycle preserves explicit child-before-parent close and existing terminal-Future outcome semantics.
6. **Authority and persistence:** `sqlite3` requires explicit host authorization under approved D259; trusted in-process native SQLite executes with the OS process's own allowed effects and is *not* confined by guest Filesystem grants. SQLite itself owns WAL, journal, locking, atomic transactions and recovery. No Landlock, JDBC, SQLite-specific Java infrastructure, custom VFS, worker, IPC or duplicate Protos journal.

Retain the already approved LIB029/LIB030 behavior: complete single-statement admission, typed Integer i64/Float/TEXT/BLOB/null data, native copy/custody boundaries and standard resource error lifecycle. The ratification does not approve unrelated changes to Core semantics, Java runtime dispatch or Truffle execution, nor new APIs absent from the six enumerated points.

The existing implementation candidate additionally contains specific argument validation and error mappings documented in `spec/semantics/SQLITE.md`. Before publishing that normative document, reconcile its exact words against this explicitly ratified contract and its already approved predecessor invariants; material deviations or new substantive choices require the owner, not silent implementation authority. Do not reopen the approved six points for ceremonial re-approval.

## Observed validation evidence, distinct from publication

On 2026-11-22 the maintainer supplied the local validation output on a `0.4.189-SNAPSHOT` checkout:

- `git diff HEAD --check` clean.
- `ProtosLib029SqliteLifecycleTest`: 2 tests, zero failures/errors/skips, `BUILD SUCCESS` (reported in earlier focused run).
- `make test-protos-foreign`: `DIST_BUILD: PASS`, `foreign-provider: 84/84 PASS`.
- Individual `LIB031_LOCAL`, `LIB031_REOPEN`, `LIB031_READ_ONLY`, `LIB031_OPEN_MODES`, `LIB031_LOCKING`, `LIB031_RECOVERY`, `LIB031_EXAMPLE`: all `PASS`.
- The latest targeted edit concerned documentation: a table-name correction `sqlite_demo` to `durable_demo`, and labeling real validated cases correctly. Maintainer reported `git diff HEAD --check` clean after this correction.

These are **maintainer-supplied results, not agent-run tests**. A full final-candidate `make test` result was **not supplied** at this decision milestone. No publication SHA for the LIB031 product implementation exists in this evidence record; the snapshot version is not a released or committed version claim. Do not relabel any missing platform test `PASS`.

## GITHUB021 invariant reconciliation

| Previously owner-approved rule | Ratification treatment |
| --- | --- |
| Protos-first module and existing generic FFM; no new Java SQLite backend | PRESERVED |
| Explicit D259 native-library allowlist and trusted-in-process OS effects | PRESERVED |
| No native Landlock confinement guarantee (PLAT074 withdrawn/not planned) | PRESERVED |
| Actor-local, synchronous, nontransferable resources and terminal-Future close | PRESERVED |
| SQLite is the transaction-state and persistence authority | PRESERVED |
| Do not silently add retries, cancellation, auto-commit emulation, a second SQL engine or journal | PRESERVED |
| LIB029/LIB030 existing behavior and tests | PRESERVED |
| Previously unresolved API names/modes/transaction methods | **RATIFIED as enumerated above** |
| Owner rejected four product slices | PRESERVED: one integrated LIB031 implementation/finalization |

## Next required publication boundary

The owner decision is now complete. Finalization remains to be performed in the product repository from the actual locally validated candidate:

1. Verify that the pending `spec/semantics/SQLITE.md` agrees with the ratified six points and existing normative owners; publish the global spec revision in `spec/PROTOS_SPEC_CHANGELOG.md` in accordance with `spec/AGENTS.md`.
2. Following the maintainer's validation-green handoff, re-read the current `pom.xml`, root `CHANGELOG.md`, branch and publication metadata. Derive the *then-current* next `-SNAPSHOT` version once, add the corresponding English changelog entry, and preserve unrelated concurrent work. Do not reserve a version from this record.
3. Run/check the final applicable validation including the remaining full `make test` requirement and the exact published candidate, without claiming PASS early.
4. Publish product files only under the repository's applicable authorization/gates, then close D256 and LIB031 on actual normative/implementation publication and verified evidence. The owner's earlier permission to commit/push `guillermomolina/protos-project-docs` is **not** authorization to commit/push product code autonomously.

**Status at this event:** `D256_EXACT_PUBLIC_CONTRACT_OWNER_RATIFIED=YES`; `D256_NORMATIVE_TEXT_PUBLISHED=NO`; `LIB031_LOCAL_FFM_84_OF_84_PASS_REPORTED=YES`; `LIB031_PRODUCT_COMMIT_PUBLISHED=NO`; `FULL_MAKE_TEST_RESULT_SUPPLIED=NO`.
