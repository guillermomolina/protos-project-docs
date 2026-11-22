# D256-C1R — Ratified SQLite specification, durable learning material and closure evidence

**Date:** 2026-11-22  
**Formal design issue:** [D256 #1118](https://github.com/guillermomolina/protos/issues/1118)  
**Implemented dependency:** [LIB031 #1120](https://github.com/guillermomolina/protos/issues/1120), already closed completed  
**Published product revision:** [`c2bfd9b3fd4d3873a7530d1b0549cc6e4e9936e5`](https://github.com/guillermomolina/protos/commit/c2bfd9b3fd4d3873a7530d1b0549cc6e4e9936e5)  
**Product baseline:** [`b0bbcae2e69b888cbb3d64c471791a0251ef3b4b`](https://github.com/guillermomolina/protos/commit/b0bbcae2e69b888cbb3d64c471791a0251ef3b4b)  
**Existing durable owner ratification:** [exact D256-C1R approval evidence](D256_C1R_EXACT_PUBLIC_CONTRACT_OWNER_RATIFICATION_2026_11_22.md)  
**Earlier implementation evidence:** [published LIB031 revision](../LIB031/LIB031_SINGLE_SLICE_SQLITE_DURABLE_PUBLICATION_2026_11_22.md)

## Product publication verified, not merely a local candidate

The GitHub commit search and revision comparison verify that `c2bfd9b3fd4d3873a7530d1b0549cc6e4e9936e5` is published on the default branch. Compared directly with its predecessor `b0bbcae2e69b888cbb3d64c471791a0251ef3b4b`, GitHub reports **one commit ahead, zero behind, 17 changed paths**.

The published [`spec/semantics/SQLITE.md`](https://github.com/guillermomolina/protos/blob/c2bfd9b3fd4d3873a7530d1b0549cc6e4e9936e5/spec/semantics/SQLITE.md) explicitly records owner ratification on 2026-11-22, rather than the obsolete pending-owner wording. The [global specification changelog](https://github.com/guillermomolina/protos/blob/c2bfd9b3fd4d3873a7530d1b0549cc6e4e9936e5/spec/PROTOS_SPEC_CHANGELOG.md) publishes **revision `0.1.507`**, headed **D256-C1R — Ratified SQLite Standard Library contract**. [`spec/PROTOS_LANGUAGE_SPEC.md`](https://github.com/guillermomolina/protos/blob/c2bfd9b3fd4d3873a7530d1b0549cc6e4e9936e5/spec/PROTOS_LANGUAGE_SPEC.md) links to the normative SQLite owner.

This is a normative, educational and harness publication only. The 17 changed paths include **no Java, `src/`, `protos/lib/`, `pom.xml` or root `CHANGELOG.md`**. Implementation version remains **`0.4.190-SNAPSHOT`** from LIB031. A specification-only revision does not require a Maven version bump.

## Ratified public contract and preserved invariants

The owner's [six-point explicit ratification](D256_C1R_EXACT_PUBLIC_CONTRACT_OWNER_RATIFICATION_2026_11_22.md) and the exact published normative text agree on:

1. `SQLite.openMemory()` and `SQLite.openFile(path, mode: "readWriteCreate")`, with `"readOnly"`, `"readWrite"` and `"readWriteCreate"` modes, strict pre-entry argument admission and effective writable-mode validation.
2. `db.begin()`, `db.commit()` and `db.rollback()` as ordinary prepared SQLite SQL; after failure SQLite, not a guessed wrapper flag, governs transaction state.
3. `db.inTransaction()` obtaining the actual `sqlite3_get_autocommit` state.
4. `db.busyTimeout(milliseconds)` delegating qualifying lock waits to SQLite, without promising query cancellation, arbitrary deadlines or automatic interruption.
5. Actor-local, synchronous Connection/Statement resources; child-before-parent cleanup, at-most-once native release, primary-error preservation and the existing fresh terminal close-Future semantics.
6. D259 explicit host authorization of trusted in-process native SQLite, with the operating system process's authority. SQLite owns file I/O, journal/WAL, locking, transactions and recovery; guest Filesystem grants do not confine C code. There is no custom VFS, Landlock worker, JDBC, SQLite-specific Java implementation or invented alternate Protos journal.

GITHUB021 invariant/delta reconciliation: the exact owner-selected C1R boundaries and previous [LIB029/LIB030 and D259 approval invariants](D256_C1R_EXACT_PUBLIC_CONTRACT_OWNER_RATIFICATION_2026_11_22.md) remain **PRESERVED**. The product update ratifies the already selected public signatures and error/lifecycle rules rather than silently adding an independent design choice. It introduces no Core storage type or new public `std:ffi` capability. Portability and physical power-loss guarantees are not expanded.

## Published executable and user-learning material

- [`protos/tutorials/12-system-resources/07-sqlite-durable-transactions.protos`](https://github.com/guillermomolina/protos/blob/c2bfd9b3fd4d3873a7530d1b0549cc6e4e9936e5/protos/tutorials/12-system-resources/07-sqlite-durable-transactions.protos): persistent open, reusable prepared inserts, COMMIT, ROLLBACK, native in-transaction observation, terminal cleanup and independent read-only reopening with write-denial assertions.
- [`protos/examples/ffi/sqlite-wal.protos`](https://github.com/guillermomolina/protos/blob/c2bfd9b3fd4d3873a7530d1b0549cc6e4e9936e5/protos/examples/ffi/sqlite-wal.protos): database-wide WAL mode explicitly checked, `synchronous=FULL` checked on the writer connection, committed data and WAL mode verified after read-only reopening; durable-policy limitations are explained.
- [`dist/test_external_foreign_providers.py`](https://github.com/guillermomolina/protos/blob/c2bfd9b3fd4d3873a7530d1b0549cc6e4e9936e5/dist/test_external_foreign_providers.py): `LIB031_TUTORIAL` and `LIB031_WAL_EXAMPLE` added to the existing external-provider harness; two executions per program on separate disposable databases, demanding exit zero, no stdout/stderr and no timeout.
- User-facing SQLite, FFI and interoperability guides and example/tutorial indexes now describe the shipped APIs, commands, platform limits and trust boundary **without project-test counts, campaign history or issue-tracking metadata**. Results belong in this internal evidence record, not in the user guides.

## Validation, carefully attributed

The **maintainer**, not this GitHub-record-writing agent, supplied the following local transcript on 2026-11-22 for the D256 candidate:

| Gate | Evidence |
| --- | --- |
| `git diff HEAD --check` | clean, no output |
| `mvn -DskipTests package` | `BUILD SUCCESS`; tests explicitly skipped by this command |
| portable distribution | `DIST_BUILD: PASS`; layout, CRC, runtime projection and source identity checks PASS |
| `make test-protos-foreign` | **86/86 PASS** |
| `LIB031_TUTORIAL` | PASS |
| `LIB031_WAL_EXAMPLE` | PASS |
| prior seven `LIB031_*` scenarios | each PASS |
| full local tests after final editorial changes | **PASS reported explicitly by the maintainer**; no independent CI run or full final command transcript is asserted |

The previous **84/84** integration run remains historical evidence for the older LIB031 product publication `8b275264db144e50a7039eaeeebac6cd61a7919c`; it is not the current 86-case run. A `make test` full output was not attached to this D256 closure; the user expressly attests that all tests pass locally. No unsupported platform was promoted to PASS.

## Closure and independent work boundaries

**D256-C1R is ready to close `completed`** because exact owner approval, GITHUB021 consistency, product-side normative publication `0.1.507`, tested executable learning material and host-authority boundaries are all accounted for. `LIB031 #1120` was separately completed at its own published implementation revision. This decision does not declare [D255 #1107](https://github.com/guillermomolina/protos/issues/1107) or [I153 #1068](https://github.com/guillermomolina/protos/issues/1068) complete, nor claim a Native Image FFM provider, native sandbox or physical power-loss durability.

**Revision-bound reconciliation:**

`PROTOS_REVISION=c2bfd9b3fd4d3873a7530d1b0549cc6e4e9936e5`  
`SPEC_REVISION=0.1.507`  
`DECISION_APPROVAL_PROVENANCE=PASS`  
`DECISION_INVARIANT_CONSISTENCY=PASS`  
`PRODUCT_NORMATIVE_PUBLICATION=PASS`  
`LOCAL_FFM_86_OF_86_PASS=MAINTAINER_SUPPLIED`  
`FULL_LOCAL_TESTS=MAINTAINER_ATTESTED_PASS`  
`REQUIRED_DURABLE_PUBLICATION=PASS (once exact published revision is re-read)`  

The exact project-record commit SHA must be taken from GitHub after publication, re-read at that revision, and pinned in [D256 #1118](https://github.com/guillermomolina/protos/issues/1118). This source document is not its own commit authority.
