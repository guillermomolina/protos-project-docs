# D255-SQL1 — Exact owner approval, comparative evidence and GITHUB021 reconciliation

**Date:** 2026-11-22  
**Live decision:** [D255 #1107](https://github.com/guillermomolina/protos/issues/1107)  
**Ratified record:** [D255-SQL1](../../decisions/language/D255_SQL1_REALWORLD_SQLITE_SINGLE_AUTHORITY_RATIFIED_2026_11_22.md)  
**Implementation target:** [I153 #1068](https://github.com/guillermomolina/protos/issues/1068) / [RealWorld application](https://github.com/guillermomolina/protos-realworld-example-app)

## 1. Exact approval and revisions

After being presented with **D255-SQL1** (SQLite as the single authoritative RealWorld store, RootActor-owned Connection, private Protos domain repositories, explicit schema/transactions/security/host authority, WAL/FULL, migration/restart failure policy, independent I153 testing and no app journal), the owner was asked:

> ¿Apruebas **D255-SQL1** con estas decisiones?

The owner answered in the immediately subsequent turn:

> si la apruebo

This is direct approval of **the named presented design**, including the express rejection of memory-as-authority plus mirrored SQLite and the historic journal as the current backend. It does not imply I153 has been implemented, that any new tests have passed, or that unrelated language semantics were approved.

**Pinned inspected sources:**

- Protos [`c2bfd9b3fd4d3873a7530d1b0549cc6e4e9936e5`](https://github.com/guillermomolina/protos/commit/c2bfd9b3fd4d3873a7530d1b0549cc6e4e9936e5), SQLite spec `0.1.507`, with [D256-C1R normative publication evidence](../D256/D256_C1R_SQLITE_NORMATIVE_PUBLICATION_CLOSURE_2026_11_22.md).
- RealWorld [`56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c`](https://github.com/guillermomolina/protos-realworld-example-app/commit/56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c): `src/main.protos`, `src/app/Api.protos`, `Auth.protos`, `AuthStore.protos`, `MemoryStore.protos`, `IdentityStore.protos`, `ContentStore.protos`, `FollowStore.protos`, `FavoriteStore.protos`, `HttpWorker.protos`, README/Makefile/TODO.
- Historical owner-ratified [D255-H1 contract](../../decisions/language/D255_PROTOS_FIRST_DURABLE_PERSISTENCE_H1_RATIFIED_2026_11_10.md), its [six-system comparison](D255_H1_GITHUB010_GITHUB021_APPROVAL_AND_COMPARATIVE_EVIDENCE_2026_11_10.md) and [I153 acceptance snapshot](../I153/I153_REALWORLD_PUBLISHED_APP_ACCEPTANCE_GAP_2026_11_10.md).

At the inspected RealWorld revision, `main.protos` initializes `Store.create()` as memory state, while `AuthStore.protos` keeps credentials in an Actor-local Array. `AuthStore.register` hashes asynchronously, generates a subject, obtains a JWT before mutating both identity stores, and rechecks uniqueness after yielding. `changePassword` and `updateAccount` also contain suspension-sensitive generation checks. Replacing this with SQLite must preserve these ordering/security invariants, not merely serialize a post-hoc snapshot. No new running application/CI evidence was obtained in this decision step.

## 2. Comparative technical precedents (GITHUB010)

This comparison builds on the [previous six-system H1 primary-source survey](D255_H1_GITHUB010_GITHUB021_APPROVAL_AND_COMPARATIVE_EVIDENCE_2026_11_10.md) and the now-published `std:sqlite` D256-C1R contract. It covers native embedded SQL, language-level DB interfaces, embedded transactional KV/LSM and external SQL service models.

| System | Concrete precedent and relevance | Why it does / does not become the RealWorld design |
| --- | --- | --- |
| [SQLite WAL](https://www.sqlite.org/wal.html), [foreign keys and PRAGMAs](https://www.sqlite.org/pragma.html), [transaction language](https://www.sqlite.org/lang_transaction.html) | Mature native embedded relational transactions; persistent journal mode, per-connection foreign key/synchronous policy, failed COMMIT can leave active state | **Selected backend** through existing authorized `std:sqlite`, without new database infrastructure |
| [Python `sqlite3`](https://docs.python.org/3/library/sqlite3.html) | Ordinary language-level access to SQLite cursors/connections, with host-language transaction policies | Reuse only the architectural idea of ordinary application policy above SQLite; **do not import Python-specific default transaction semantics** |
| [Node `node:sqlite`](https://nodejs.org/api/sqlite.html) | Synchronous SQLite API with prepared statements | Shows a synchronous library surface can be practical; native blocking remains governed by Protos Actor semantics, not Node's |
| [Rust `rusqlite`](https://docs.rs/rusqlite/latest/rusqlite/) | Typed Rust library around SQLite Connection, statements and transactions | Language-integrated repositories are viable; do not require Rust-style static ownership or expose native handles in Protos |
| [Go `database/sql`](https://pkg.go.dev/database/sql) | Transaction interfaces and pooled DB connections around driver boundaries | Informs transactional unit-of-work discipline, but generic driver/pooling abstraction is extra cost for a single RootActor today |
| [LMDB](https://github.com/LMDB/lmdb/blob/mdb.master/libraries/liblmdb/lmdb.h) | Copy-on-write embedded transactional KV | Another viable storage engine but would require new native binding/data model without current necessity |
| [RocksDB](https://github.com/facebook/rocksdb/wiki/Basic-Operations) | Embedded KV with write batching, logging and compaction | Its compaction and KV mapping obligations are disproportionate to this small relational example |
| [PostgreSQL WAL](https://www.postgresql.org/docs/current/wal-intro.html) | Transactional external server supporting larger deployment footprints | Preserves a plausible future option; mandatory network/server operation now is not justified by I153 |

No source in this table authorizes silently copying another runtime's dispatch/VM architecture. **No Java or Protos Standard Library production changes are part of SQL1.**

## 3. Meaningful design candidates

- **SQL1 (selected):** SQLite is the only authoritative application data store, with thin *ordinary Protos* private domain repositories, one RootActor Connection and native SQLite transactions.
- **SQL+memory mirror (rejected):** keep the current `MemoryStore` as authoritative and mirror changes into SQL; synchronization after failures/COMMIT uncertainty makes it impossible to state a single correct post-crash state without introducing a second transaction/reconciliation system.
- **Historical H1 journal (superseded for I153):** application-owned append-only File journal/codec/replay; a previously valid approved design packet but the owner corrected I153's target to **actual SQLite**, and it duplicates a mature database's persistence logic.
- **Generic storage/driver/ORM (rejected now):** public `std:storage` or pluggable engine layer with a universal transaction/error/identity contract; premature universalization for one application and needless public semantic lock-in.
- **Mandatory remote PostgreSQL (deferred):** valid scaling escape route, but additional service, credential, network and deployment burden today; adding a remote backend later is application migration work, not a requirement to preimplement one.
- **LMDB/RocksDB first (deferred):** credible engines, but would require new bindings and relational mapping with no demonstrated benefit over the now-available SQLite library.
- **Leave application in memory (rejected):** fails mandatory restart durability.

## 4. Twelve-dimension scored comparison

Scores are **1 (poor) through 5 (strong)** for today's I153. Every cell includes a qualitative rationale and confidence **H** (high), **M** (medium) or **L** (low). These are **design assessments**, not measured performance, and totals do not override disqualifying safety/design constraints.

| Dimension | SQL1 single authority | Memory + SQLite mirror | H1 journal | Generic storage layer |
| --- | --- | --- | --- | --- |
| 1. Correctness/invariants | **5H** one SQL transaction authority | **2H** independent state split | **2M** separate journal recovery proof | **3L** cross-driver semantics unresolved |
| 2. Protos alignment | **5H** ordinary Protos repositories | **4M** guest code but duplicated authority | **4H** ordinary Protos but wrong backend target | **2H** speculative public abstraction |
| 3. Present proportionality | **5H** only known SQLite/RealWorld need | **2H** dual maintenance | **2H** own storage engine unnecessary now | **1H** prebuilds unused options |
| 4. Incremental growth | **4M** versioned SQL schema grows | **3L** extra cache consistency rules | **2M** journal/index upgrades costly | **4L** adaptable but unproven |
| 5. Future options | **4M** private repos preserve migration seam | **3L** coupled cache and SQL | **3M** export possible but custom format | **4M** broad engines, high current cost |
| 6. Scalability | **3M** one RootActor/write custodian | **3L** same custodian plus mirror | **2H** one writer and replay bound | **4L** depends on driver, no proof |
| 7. Conceptual simplicity | **5H** engine does storage work | **2H** two truths and reconciliation | **2H** own checksums/replay/locks | **1H** generic contract complexity |
| 8. Portability/freedom | **3H** verified Linux/JDK25 FFM envelope | **3H** same native envelope | **4M** File API wider but unproven crash guarantee | **4L** abstract but adapters absent |
| 9. Runtime/resource cost | **4M** one Connection and SQL | **2M** duplicate in-memory state/writes | **3L** journal/replay and compaction cost | **2L** mandatory layers and drivers |
| 10. Failure/operability | **5H** native WAL/transactions with known limits | **2H** ambiguity across two stores | **2M** bespoke recovery fault space | **3L** driver-specific failure model |
| 11. Deferral/reversibility | **4M** private SQL repos can migrate later | **2H** two persisted semantics | **2H** migrating custom journal required | **3L** public contract difficult to change |
| 12. Evidence maturity/risk | **4H** LIB031 + 86/86 FFM, app untested | **2L** mirror recovery unproved | **3M** prior journal tests, not SQLite acceptance | **2L** no public implementation |

**Non-compensating qualitative gates:** the mirror is disqualified by duplicate authority and uncertain-commit divergence; H1 is disqualified as the owner's corrected RealWorld objective (not because its prior approval was false); the generic layer is disqualified by unjustified scope and foundational public API lock-in. SQL1's native platform limitation is real but explicit, not silently made portable. PostgreSQL, LMDB and RocksDB are deferred as viable later alternatives, not rejected as universally inferior.

**Pay only for present need:** SQL1 reuses the published SQLite Stack and leaves private application repositories simple; no Java, future backend driver, ORM, worker pool or global schema service is installed.

**Grow only as needed:** SQLite indexes, additional tables and explicit versioned migrations can be added to the application at future schema revisions without changing Core semantics. Multi-RootActor writes, remote databases or other host profiles require later independent evidence and design, not speculative support now.

**Cost of deferral:** deferring generic storage means future application queries/repositories may need rewriting for a different engine; this is limited to the private RealWorld boundary, not a public Protos compatibility break. Deferring stable IDs, credential generations or transactional identity integrity **would** force a persisted-data/security migration and can revoke the wrong users; those invariants are included now.

**Smallest sufficient path:** one configured SQLite database, one RootActor Connection, private Protos queries, atomic transactions, versioned schema and restart acceptance; anything more must satisfy a demonstrated present need.

## 5. Adversarial scenario and acceptance matrix

| Failure / change | SQL1 requirement or explicit limit |
| --- | --- |
| Provider missing, library unauthorized, no DB path or malformed key | Fail startup **before listener/readiness**, sanitized diagnostics; no memory fallback |
| Invalid/newer `user_version`, incompatible schema, corruption | Reject known-bad state, no destructive rewrite or silent empty-store creation |
| Read-only file/directory, missing WAL sidecar permissions | Fail closed; guest `Filesystem` grant does not confine or authorize native SQLite |
| Concurrent registration via two HTTP Workers | RootActor revalidates after password-hash suspension; SQL unique constraints are final arbiter |
| JWT password-change generation after restart | Same subject/id/verifier/generation; revoked tokens remain revoked |
| Rename user after articles/follows/favorites exist | All references use stable numeric IDs, not mutable username as authority |
| Repeated favorite/follow/unfollow or deleted article | Unique edges and foreign keys; idempotence and dependent deletion policy consistent |
| Crash before COMMIT | No acknowledged success; engine-owned rollback/recovery on reopen |
| Crash after COMMIT before HTTP reply | Possible uncertain client outcome; do not promise exactly-once delivery or guessed rollback |
| `SQLITE_BUSY`, commit failure | Do not treat error as rollback; inspect native transaction state, prevent unsafe additional writes |
| Disk full, native error, process SIGKILL | No response success without confirmed commit; restart consistency, independent OS process |
| Unknown H1 journal data | Preserve historical evidence; no silent journal replay or automatic destructive import |
| Unmodified official Hurl vs patched suite | Keep independent 12/13 vs 13/13 (157 requests) prior evidence, require new pinned verification |

SQLite primary-source constraints were verified against [SQLite transactions](https://www.sqlite.org/lang_transaction.html), [PRAGMA documentation](https://www.sqlite.org/pragma.html) and [persistent WAL mode](https://www.sqlite.org/wal.html). Foreign-key enforcement is per-connection and must be changed outside a transaction; WAL mode is persistent but synchronous policy is connection-specific. Failed COMMIT may leave an active transaction. These are already approved choices within the selected D256 `std:sqlite` contract, not new Protos runtime semantics.

## 6. GITHUB021 — Explicit prior-owner invariant/delta reconciliation

| Prior authoritative decision or invariant | SQL1 treatment |
| --- | --- |
| RootActor owns mutable state and secrets; Workers only bounded snapshots | **PRESERVED** |
| RealWorld must use actual SQLite durability, not a hand-built journal substitute | **FULFILLED AS DESIGN** (implementation still pending) |
| D256-C1R: `std:sqlite` Protos-first, no Java SQLite infrastructure or new Core abstraction | **PRESERVED** |
| D259: explicit native-library authorization; no guest Filesystem confinement of trusted C | **PRESERVED** |
| Actor-local, synchronous Connection/Statement, explicit release and uncertainty after failed commit | **PRESERVED** |
| Stable numeric ID, immutable JWT subject, Argon2id verifier, revocation generation, authorization recheck after suspension | **PRESERVED AND NOW MADE PERSISTENT IN DESIGN** |
| The real 2026-11-10 owner approval of D255-H1 (ordinary Protos journal/File.sync/lock/replay) | **EXPLICITLY SUPERSEDED FOR I153 BY THE LATER CORRECTED OBJECTIVE AND THE OWNER'S 2026-11-22 SQL1 APPROVAL**; historical record unchanged |
| No unsupported universal durability, cross-platform/Native Image, native sandbox guarantees | **PRESERVED** |
| Distinct official RealWorld Hurl gap and application acceptance | **PRESERVED**; neither merged into design closure nor marked PASS |

**Material delta made explicit before approval:** the current RealWorld persisted-data authority changes from H1's app-owned journal to the newly available owner-approved SQLite library. The original owner rejected a journal as replacement for SQLite and was presented the exact SQL1 alternative including sole authority, mode, schema families, transaction boundaries, FK/WAL/FULL and failure/validation. The affirmative reply selects that revised architecture. Neither a silent override nor retroactive falsification of the historic H1 event occurs.

**No new observable Protos language or Standard Library API is selected**, and no Java production changes, source edits, tests, commits in the product repository, spec revision or release occurred in this decision step.

## 7. Formal publication/release boundary

`DECISION_APPROVAL_PROVENANCE=PASS` — immediate explicit owner response to the named D255-SQL1 approval request.  
`DECISION_INVARIANT_CONSISTENCY=PASS` — explicit H1 replacement and all remaining constraints reconciled.  
`PROTOS_REVISION=c2bfd9b3fd4d3873a7530d1b0549cc6e4e9936e5`  
`REALWORLD_REVISION=56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c`  
`I153_IMPLEMENTATION_COMPLETE=NO`  
`SQL1_APPLICATION_TESTS_EXECUTED=NO`  
`REQUIRED_DURABLE_PUBLICATION=REQUIRES_PUBLISHED_REVISION_RECHECK`

When this evidence and the canonical decision are published, pin their exact `PROJECT_RECORD_REVISION` to [D255 #1107](https://github.com/guillermomolina/protos/issues/1107) and re-read both records before closing D255 `completed`. I153 remains separately open. A direct GitHub documentation publication creates its own revision; the source file cannot self-certify its future commit hash.
