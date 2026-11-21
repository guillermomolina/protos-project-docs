# PLAT074 — SQLite confinement requirement withdrawn after D259 (2026-11-21)

**Scope:** durable evidence and GITHUB021 reconciliation for [PLAT074 #1122](https://github.com/guillermomolina/protos/issues/1122), [D256 #1118](https://github.com/guillermomolina/protos/issues/1118), [LIB031 #1120](https://github.com/guillermomolina/protos/issues/1120), and [D259 #1139](https://github.com/guillermomolina/protos/issues/1139).

## Owner decision and correction

The owner explicitly corrected the proposed distinction between a trusted SQLite profile and a separate confined SQLite profile: authorizing a library means accepting **all consequences of its native execution**, including filesystem access outside any directory associated with Protos guest Filesystem grants. Such native effects are **not a breach of the Protos guest Filesystem guarantee**. The owner then requested the downstream Issue coordination and this evidence publication following the recommendation to close PLAT074 as **not planned**.

This does **not** assert that Protos can make arbitrary native libraries safe, or that native code ignores operating-system-level permissions. It says that the host consciously accepts whatever native behavior the OS permits. The distinction is between guest API authority, which Protos continues to mediate, and separately authorized trusted native code, which it does not sandbox.

## Published authority and provenance

- [D259-B2+A ratified design](../../decisions/language/D259_TRUSTED_NATIVE_FFM_ALLOWLIST_B2_A_RATIFIED_2026_11_21.md): explicit host-selected `alias=target`, no implicit native library authorization, acceptance of all in-process native risks, no invented Landlock, per-library VFS, worker or IPC.
- [Protos normative publication at `ee8be92c482cc2451e888d444d74b382b319f4f1`](https://github.com/guillermomolina/protos/commit/ee8be92c482cc2451e888d444d74b382b319f4f1), specification `0.1.506`, including `spec/semantics/FFI.md` and `spec/concurrency/ACTORS.md` §24K.1.
- [I169 #1140](https://github.com/guillermomolina/protos/issues/1140) owns **unimplemented** generic provider/CLI/embedding support for D259, with explicit `--allow-library sqlite3=libsqlite3.so.0` and no implicit `ffi:sqlite3` grant.
- [D256-C1 approval](../../decisions/language/D256_C1_PROTOS_FIRST_SQLITE_ARCHITECTURE_APPROVED_2026_11_11.md): SQLite-specific Protos-first architecture remains selected; its historical confinement prerequisite is superseded only as described below.

## GITHUB021 invariant/delta reconciliation

| Previously recorded invariant / concern | Reconciled consequence |
| --- | --- |
| `std:ffi` import grants nothing; guest does not acquire arbitrary native targets | Preserved. The host must explicitly authorize the native library; until I169 implements D259, the new CLI/provider option is not usable as a completed feature. |
| Guest `Filesystem` grants are explicitly scoped and confined within their own API/worker | Preserved **for guest Filesystem operations**. A trusted native library's own system calls are outside that mediation and may access other OS-permitted paths. |
| D256-C1 proposed effective native SQLite file-I/O confinement | **Withdrawn as a requirement** for the explicitly authorized trusted-in-process SQLite facility, per owner correction. Do not market `std:sqlite` as a per-directory kernel-confined database capability. |
| PLAT074 Landlock worker / custom SQLite VFS / isolated helper-process investigation | **Not planned** for the trusted-native SQLite contract, rather than implemented or proved unnecessary for every future threat model. No replacement confinement scheme is implied. |
| Actor-local Connection/Statement/FFM handle custody and synchronous call semantics | Preserved. Native C globals can be shared in one OS process under D259 §24K.1; Protos-managed nontransferability, resource retirement and session checks remain. |
| SQLite durability, WAL/journal, locking, recovery and correct close/commit behavior | **Not discharged by D259.** Implementation and reproducible failure/concurrency tests remain necessary in LIB031. |
| D256 exact public API, Actor blocking/cancellation/timeout and uncertain commit | **Still pending** owner-approved reconciliation and normative/public contract work; do not claim D256 complete. |
| `--allow-library` execution and tests | **Still pending** I169 implementation and maintainer validation; a ratified specification is not a completed implementation. |

## Consequences for coordination

1. Close [PLAT074 #1122](https://github.com/guillermomolina/protos/issues/1122) as `not_planned`: its original confinement design question is no longer a prerequisite for selected host-trusted SQLite.
2. Update [D256 #1118](https://github.com/guillermomolina/protos/issues/1118) to remove the effective native confinement gate, **without** treating the remaining Actor/public-contract gate as solved.
3. Update [LIB031 #1120](https://github.com/guillermomolina/protos/issues/1120) to remove the PLAT074 *conceptual* blocker while retaining D256's unresolved public/Actor contract, I169 provider implementation and SQLite durability acceptance gates. Existing native GitHub blocker or hierarchy relations must be verified before claiming they were changed.
4. Do not alter [PLAT076 #1138](https://github.com/guillermomolina/protos/issues/1138): an independently proposed opt-in Landlock capability is a separate design matter and cannot be inferred from or required by trusted SQLite.
5. No product source, implementation, normative spec or tests were modified or executed by this coordination record.

**Verification status:** documentary reconciliation only; no SQLite or I169 implementation tests claimed. Historical issue text remains intact below its current-status updates as provenance.
