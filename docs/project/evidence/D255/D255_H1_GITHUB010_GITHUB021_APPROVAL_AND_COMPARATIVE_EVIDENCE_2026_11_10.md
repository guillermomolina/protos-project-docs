# D255-H1 — GITHUB010 comparative evidence, GITHUB021 reconciliation and exact owner approval

**Evidence date:** 2026-11-10. **Research/approval only; no product code, builds, tests, crash experiments or benchmarks performed.**

**Issue:** [D255 #1107](https://github.com/guillermomolina/protos/issues/1107). **Exact owner reply:** **“Apruebo D255-H1”**, in the active conversation after the decision packet recommended **H1 = E+A**, an app-owned single-writer bounded file journal in Protos over explicit confined Filesystem, with explicit `sync` durability, conservative recovery, no generic Core `Storage`, `std:storage` or ORM, and optional later SQLite/backend migration. The same owner message explicitly authorized GitHub Issue updates and commit/push of D255 evidence into **guillermomolina/protos-project-docs**. Approval does *not* mean implementation/test acceptance.

**Stable inspected source revisions:** Protos [`c9da7ff3e6c95f7a88948e875f61d67007fdeaa5`](https://github.com/guillermomolina/protos/commit/c9da7ff3e6c95f7a88948e875f61d67007fdeaa5); RealWorld [`56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c`](https://github.com/guillermomolina/protos-realworld-example-app/commit/56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c). Baseline alone is not implementation evidence. Ratification: [D255-H1](../../decisions/language/D255_PROTOS_FIRST_DURABLE_PERSISTENCE_H1_RATIFIED_2026_11_10.md).

## 1. Verified local state and exact decision need

At the fixed RealWorld revision `src/main.protos` creates `Store.create()` during RootActor service initialization; `src/app/MemoryStore.protos` delegates into `IdentityStore.protos`, `FollowStore.protos`, `ContentStore.protos` and `FavoriteStore.protos`. `src/app/AuthStore.protos` separately holds `id`, JWT `subject`, Argon2id `verifier` and `generation` in an Actor-local Array; its own comment says all state disappears on restart. RootActor owns the mutable store, security key and Clock/Entropy authority; Workers transfer bounded request snapshots only. App restart loses authentication and social/content relationships. See [I153 maintained status](../../work/I153/I153_REALWORLD_BACKEND_CURRENT_STATUS_2026_11_10.md) and [I153 evidence](../I153/I153_REALWORLD_PUBLISHED_APP_ACCEPTANCE_GAP_2026_11_10.md).

Protos normative constraints at the fixed revision:
- [FILESYSTEM.md](https://github.com/guillermomolina/protos/blob/c9da7ff3e6c95f7a88948e875f61d67007fdeaa5/spec/io/FILESYSTEM.md): `open` independent asynchronous acquisitions, stable File binding, only per-receiver ordering, no atomic cross-writer `seekToEnd`+write; `replace/remove` have live atomic visibility but **no promised namespace crash durability**. `File.sync` does not make directory entry durable.
- [BYTE_IO.md](https://github.com/guillermomolina/protos/blob/c9da7ff3e6c95f7a88948e875f61d67007fdeaa5/spec/io/BYTE_IO.md) §12: `sync()` is a **receiver-scoped durability frontier**; `close()` does not imply it; failed sync can leave partially persisted effects; write can produce a prefix.
- [IO_CORE.md](https://github.com/guillermomolina/protos/blob/c9da7ff3e6c95f7a88948e875f61d67007fdeaa5/spec/io/IO_CORE.md), [PROCESS_IO.md](https://github.com/guillermomolina/protos/blob/c9da7ff3e6c95f7a88948e875f61d67007fdeaa5/spec/io/PROCESS_IO.md): authority/commitment, fresh operation Futures, close/termination/cancellation; no automatic sync by Actor/Process termination.
- [ACTORS.md](https://github.com/guillermomolina/protos/blob/c9da7ff3e6c95f7a88948e875f61d67007fdeaa5/spec/concurrency/ACTORS.md), [FUTURES_AND_TASKS.md](https://github.com/guillermomolina/protos/blob/c9da7ff3e6c95f7a88948e875f61d67007fdeaa5/spec/concurrency/FUTURES_AND_TASKS.md) and [PARALLEL_EXECUTION.md](https://github.com/guillermomolina/protos/blob/c9da7ff3e6c95f7a88948e875f61d67007fdeaa5/spec/concurrency/PARALLEL_EXECUTION.md): Actor-local state, suspension interleavings, no raw capability transfer to P.
- [D238-B1-SQL-R1 #1047](https://github.com/guillermomolina/protos/issues/1047): optional in-process native FFI chosen, but [I144 #1050](https://github.com/guillermomolina/protos/issues/1050), [LIB028 #1051](https://github.com/guillermomolina/protos/issues/1051) and [LIB029 #1052](https://github.com/guillermomolina/protos/issues/1052) remain open at inspection. The LIB029 initial SQLite proof is `:memory:` and numeric binds; durable-path confinement, TEXT/BLOB input lifetime and Native Image packaging are not proven or preapproved.

**Question resolved:** What is the smallest useful persistence architecture *today* that preserves existing Protos semantics and lets RealWorld survive restarts without installing an implicit database subsystem?

## 2. Primary-source precedents: 6 systems, 4 approaches

| System and primary source | Locking/consistency/durability | Concrete Protos lesson |
| --- | --- | --- |
| [SQLite WAL](https://www.sqlite.org/wal.html), [synchronous PRAGMA](https://www.sqlite.org/pragma.html#pragma_synchronous), [C open](https://www.sqlite.org/c3ref/open.html), [bindings](https://www.sqlite.org/c3ref/bind_blob.html) | Embedded SQL, readers concurrent with one writer, WAL/checkpoint and sync-mode dependent durable commits; WAL depends on same-host coordination, TEXT/BLOB binding has pointer-lifetime rules | Most mature later embedded backend, but Protos FFI and confined path authority are **separate requirements**; SQL ACID cannot be inferred from File |
| [LMDB upstream header and API](https://github.com/LMDB/lmdb/blob/mdb.master/libraries/liblmdb/lmdb.h), [OpenLDAP LMDB source](https://www.openldap.org/software/repo.html) | MVCC/copy-on-write memory-mapped transactional KV, one writer, many readers, explicit sync configuration | KV protocol can be sufficient, but mmap/locks/native lifetime and portability must be verified |
| [RocksDB upstream Basic Operations](https://github.com/meta/rocksdb/wiki/Basic-Operations) | Embedded LSM, memtables/SST compaction, WriteBatch, WAL, configurable synced vs unsynced writes | Write scaling has material complexity and memory/background-work costs not justified by current RealWorld |
| [PostgreSQL WAL documentation](https://www.postgresql.org/docs/current/wal-intro.html) | External server, WAL and transactional recovery, concurrent DB sessions; server durability settings matter | Plausible future distributed/backend option; Network/service/auth/operational cost disqualify mandatory dependency now |
| [Python `sqlite3` DB-API](https://docs.python.org/3/library/sqlite3.html) | Embedded SQLite reached through a language library; transaction/autocommit behavior is library contract | Language-first facade need not elevate a database to Core; avoid inheriting another language's defaults |
| [Node `node:sqlite`](https://nodejs.org/api/sqlite.html) | In-runtime library exposes DB and statements with concrete lifecycle/parameter APIs; synchronous operations | Convenient library precedent but blocking database calls must not silently change Protos Actor/Future scheduling |

These systems cover journaled SQL, mmap/COW KV, LSM KV, remote transactional server and guest-runtime language wrappers. They are **comparative sources, not executed interoperability tests or performance evidence**. Licensing, version, platform packaging and native ABI compatibility remain implementation-specific due diligence (do not infer Protos portability from upstream availability). GraalPy, TruffleRuby and GraalJS examples do not grant additional Protos authority; D238's exact native contract and platform caveats govern optional future bindings.

## 3. Candidate set and elimination

| Candidate | Model | Assessment |
| --- | --- | --- |
| A+E **selected H1** | Ordinary application Protos journal/codec on explicit `Filesystem`, one writer | Smallest bounded durable-store profile, but must prove recovery and synchronization |
| B+E | App Protos repositories on SQLite, optional `std:ffi`/FFM backend | Robust transactional core, but FFI work and confinement are not finished; must not claim it is usable now |
| C+E | Host-provisioned `DurableStore` capability, concrete backends hidden | Good explicit privilege boundary, but extra Core/host mechanism without demonstrated need |
| D | Standard public `std:storage` + pluggable drivers | Portable contract looks appealing but generalized isolation/error semantics/driver duties currently unjustified |
| F+E | App Protos repository with external PostgreSQL | Legitimate scaling escape path, but server/network/deployment burden exceeds present need |
| E alone / defer | Keep memory or just a repository facade | Fails the actual restart/durability criterion, so not sufficient |
| Custom LMDB/RocksDB binding | Native KV engine as first mandatory dependency | Adds ABI, external-memory and packaging obligations while no proven scale requirement warrants them |

All options require explicit storage authority. A narrow *application interface* that allows a later export/migration is justified; a fully interchangeable public driver API is not.

## 4. Mandatory 12-dimension scoring (1 poor–5 strong)

Confidence suffixes: **H** high, **M** medium, **L** low. Each cell gives the rationale for its score. Scores compare *architecture suitability for the current Protos requirement*, not absolute database quality.

| # / dimension | A+E H1 | B+E | C+E | D | F+E |
| --- | --- | --- | --- | --- | --- |
| 1 Correctness/invariants | **3M** disciplined replay still unproven | **4M** SQLite ACID, FFI authority unproven | **4M** host can enforce narrow contracts | **3L** generic driver semantics unclear | **4H** mature transactional DB |
| 2 Protos alignment | **5H** ordinary modules + explicit FS | **4M** Protos wrappers, native boundary | **3M** extra privileged facility | **2H** speculative framework surface | **3H** guest policy with external service |
| 3 Present proportionality | **4M** minimal bounded app need | **2M** FFI prerequisite queue | **2H** new host surface | **1H** more abstraction than required | **2H** mandatory service unnecessary |
| 4 Incremental growth | **3M** compact/index later | **4M** SQL indexes/queries | **4M** future adapters possible | **4L** extensibility claimed not tested | **4H** growth techniques established |
| 5 Future options | **4H** private data boundary replaceable | **3M** SQL representation coupling | **4M** can swap host backend | **4M** multiple drivers, contracts premature | **3M** server deployment coupling |
| 6 Scalability | **2H** one writer/full replay | **4H** reader/writer transactional engine | **4M** dependent on backend | **4M** dependent on drivers | **5H** concurrent server workload |
| 7 Conceptual simplicity | **3M** journal correctness hard | **4M** accessible SQL library | **2H** new capability/lifecycle | **1H** generic error/transaction algebra | **3H** simple API, complex ops |
| 8 Portability/freedom | **4M** FS abstraction, host qualified | **3M** native ABI/packaging | **4M** platform adapter freedom | **5L** abstract but unproven | **4H** network DB clients |
| 9 Runtime/resource cost | **3M** replay/write/sync overhead | **4M** optimized embedded C engine | **4M** optional facility overhead | **3M** layers/resource policy | **2H** network/server costs |
| 10 Failure/operability | **2M** recovery must be built/proven | **5H** mature SQLite recovery | **4M** backend-managed recovery possible | **2L** backend-specific failures | **5H** mature operational tooling |
| 11 Deferral/reversibility | **4H** private format, explicit export | **3M** SQL schema migration | **3M** public capability hard to remove | **2H** public API lock-in | **3M** remote-state migration cost |
| 12 Evidence/implementation risk | **2M** no tested Protos journal | **4H** proven DB; Protos binding still pending | **3M** architectural precedent, no prototype | **2L** interoperability unproven | **5H** industrially mature DB |

**Qualitative gates:** H1's low recovery score cannot be hidden by high Protos alignment; H1 is accepted *only subject to fault-injected demonstration of its bounded durability*. D and C trigger overengineering/premature privilege concerns. H1 is not selected by summing scores. Its absence of multiprocess ACID is a deliberate limit, not a disguised guarantee.

## 5. Falsification, adversarial security and crash matrix

| Scenario / injector | Required acceptance or explicit limitation |
| --- | --- |
| Abrupt `SIGKILL` before any write | No transaction effect from that operation |
| Kill after partial record append | Never replay a partial transaction |
| Kill just before, during or after File `sync()` | Distinguish confirmed success from uncertain effects; validated consistent recovery |
| Complete checksum-valid record persisted but caller never received success | Possible **uncertain** outcome; replay cannot invent rollback; idempotent transaction ID policy required |
| Torn/truncated tail and corrupt checksum | Recover only a provably safe prefix or fail closed; no silent interior data loss |
| Older schema and unknown future format | Deterministic migration or fail closed; no guessed layouts |
| Recover 1M records / oversized length / full disk | Explicit configurable quota, bounded reading/allocation, no unbounded operation |
| Username/email conflict and concurrent registration through two Workers | One RootActor serialization point; durable uniqueness and stable IDs |
| Change password then crash/restart | Argon2id verifier + JWT subject/generation atomically persisted; revoked JWT stays revoked |
| Article/comment/favorite/follow transitions | Stable referential IDs, counters and derived indexes match replay |
| Two processes target same store | Cooperative exclusive lock `createNew`; reject second opener, no shared writer |
| Stale lock after hard crash | Administrative recovery, no automatic PID/time heuristic; not distributed fencing |
| Malicious `..`, final/intermediate symlink, mount/redirection | Remain inside explicit Filesystem authority or fail; tests of Linux backend confinement |
| Cancellation/Actor termination while write/sync commits | No pretend rollback of irreversible bytes, no HTTP success before confirmed sync |
| `close` during active journal transaction | No new admission, deterministic lifecycle/cleanup, no implicit sync claim |
| Revoked grant or unsupported `Syncable` | Stop/fail closed; never silently run an unsynced durable store |
| Untrusted SQL strings with future SQLite engine | Parameterized statements and explicit native authority; no accidental SQL injection |
| FFI loading via optional SQLite driver | Distinguish trusted-in-process native access from confined Filesystem; no sandbox claim |
| Native Image & Linux/macOS/Windows | Platform-specific tests before claiming parity; Java/JDK/Linux test is insufficient |
| Backup during active write | Consistent exported snapshot/copy protocol or reject; no raw arbitrary mid-write copy promise |

This is a test **plan**, not observed PASS results. Failure of acknowledged durability or inability to prove safe recovery falsifies H1's implementation and requires a renewed owner-controlled choice.

## 6. Pay-for-need, grow-as-need, deferral and escape route

**Minimum today:** one preprovisioned durable file, explicit existing filesystem grant, single cooperative writer, RootActor queue, framed versioned records, record checksum, conservative replay, `sync` before success and explicit quotas/error handling. No generic schema/SQL/ORM, multi-reader MVCC, async transaction object hierarchy, cluster replication or generalized storage privileges.

**Future-compatible, not future-preimplemented:** store domain stable identities and version data so an eventual SQLite/LMDB/PostgreSQL exporter/importer can be written. Separating domain operations from journal offsets is the only now-necessary migration seam. Concurrent database transactions, indexes, compaction and namespace syncing can be added only through later properly authorized work.

**Cost of deferral:** journal replay/size eventually become expensive; SQLite requires a domain-record-to-relational migration and separate FFI binding/confinement. However, no break to existing HTTP contract, JWT identity or Actor ownership is inherently required if these are kept independent. A premature `std:storage` API would lock in transactional and failure semantics not yet proven common.

**Largest regret scenario:** many independent Processes or machines need shared concurrent writes, large queries and online compaction. H1 would bottleneck and require a backend change. **Escape:** introduce an explicit engine-specific backend (SQLite first, PostgreSQL where distributed) and migrate versioned records; independently review any new capability/namespace-durability contract.

**Strongest argument against H1:** SQLite already provides far better tested recovery and locking; writing even a small safe Protos journal is expensive and can be dangerous. H1's acceptance is therefore conditional on fault-injection proof with conservative semantics. Do not weaken the durability requirement to preserve the selected label.

## 7. GITHUB021 reconciliation and owner checkpoint

| Settled boundary | Consequence | Finding |
| --- | --- | --- |
| D248, I155/I156 confined Filesystem grant | Only supplied `Filesystem` can open journal/lock; no symlink escape | PRESERVED |
| File `sync` per-resource, no cross-File ACID or namespace crash durability | One preprovisioned file and narrow journal; no rename-based durable checkpoint claim | PRESERVED |
| I/O irreversible prefix, cancellation, Actor/Process termination | Commit uncertain when bytes may have persisted; recovery rather than pretend rollback | PRESERVED |
| Actor/P isolation, D251 suspension | RootActor owns all live resources, Workers only data, queue protects interleaving | PRESERVED |
| D238 optional FFI/B1-SQL-R1 | No mandatory SQLite or new native authority; existing binding work independent | PRESERVED |
| Owner's exact H1 proposal | E+A app-owned, no `std:storage`/ORM/Core Store, bounded single writer | PRESERVED |

No contradicting previously approved invariant was identified; additional public/generalized semantics must return for approval. This is not authorization for unsafe lock stealing, rollback after partial writes, automatic compaction or SQL equivalence.

```text
D255_DESIGN=H1_E_PLUS_A_OWNER_RATIFIED
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
PROTOS_REVISION=c9da7ff3e6c95f7a88948e875f61d67007fdeaa5
REALWORLD_REVISION=56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c
BUILDS_TESTS_BENCHMARKS=NOT_RUN
JOURNAL_IMPLEMENTED=NO
REALWORLD_RESTART_ACCEPTANCE=NOT_RUN
UNMODIFIED_HURL_13_OF_13=NO
```

The decision's final publication SHA and Issue closure cross-reference must be verified after publishing all records. All official Hurl counts are retained as **previous maintainer-reported** results, not revalidated by this research.
