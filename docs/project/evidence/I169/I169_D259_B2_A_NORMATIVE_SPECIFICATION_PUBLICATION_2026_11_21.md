# I169 — D259-B2+A normative FFM library authorization published (2026-11-21)

**Evidence role:** revision-bound normative publication and implementation-gate handoff, **not** evidence that the native-library allowlist has been implemented.

**Product repository:** [`guillermomolina/protos`](https://github.com/guillermomolina/protos).
**Published Protos revision:** [`ee8be92c482cc2451e888d444d74b382b319f4f1`](https://github.com/guillermomolina/protos/commit/ee8be92c482cc2451e888d444d74b382b319f4f1) on `main`, commit message `D259-B2+A: Publish trusted native FFM library allowlist normative specification` (2026-11-21 05:01:49 UTC).
**Global normative revision:** `0.1.506` following `0.1.505` (D229-B1 / I166).
**Live implementation owner:** [I169 #1140](https://github.com/guillermomolina/protos/issues/1140), **open, implementation not yet completed**.
**Design authority:** closed [D259 #1139](https://github.com/guillermomolina/protos/issues/1139) and the [ratified decision](../../decisions/language/D259_TRUSTED_NATIVE_FFM_ALLOWLIST_B2_A_RATIFIED_2026_11_21.md); see also the [earlier owner-approval/source audit](../D259/D259_B2_A_OWNER_APPROVAL_SOURCE_AND_HANDOFF_2026_11_21.md). Those earlier records are historical snapshots of the pre-publication state, not current normative assertions.

## 1. Verified product publication

GitHub's immutable commit endpoint confirms **exactly four changed paths**, with no production Java, CLI implementation, tests, runtime sources or packaged examples in this publication:

| Normative owner | Exact published source | Verified contract |
| --- | --- | --- |
| FFI | [`spec/semantics/FFI.md`](https://github.com/guillermomolina/protos/blob/ee8be92c482cc2451e888d444d74b382b319f4f1/spec/semantics/FFI.md) | Host-only empty-by-default logical-alias → exact native target map; case-sensitive C-style aliases; no implicit SQLite; `ForeignError` categories `unauthorized-library` / `library-unavailable`; FFM acquisition authorization before that library's `Linker`/`Arena`/load; exact load names or absolute paths without Protos search fallback; loader risks explicitly accepted; `TRUSTED_IN_PROCESS`; immutable `library.<alias>` embedding options |
| Actor isolation | [`spec/concurrency/ACTORS.md`](https://github.com/guillermomolina/protos/blob/ee8be92c482cc2451e888d444d74b382b319f4f1/spec/concurrency/ACTORS.md) | §24K.1 limits the shared-native-global-state exception to explicitly host-authorized in-process libraries; Protos object/slot isolation, Actor-local modules, handle non-transferability and §24J remain |
| Host CLI and grants | [`spec/io/PROCESS_IO.md`](https://github.com/guillermomolina/protos/blob/ee8be92c482cc2451e888d444d74b382b319f4f1/spec/io/PROCESS_IO.md) | Repeatable pre-entry `--allow-library alias=target` for file, `-e`, REPL, `run` and `debug`, not bundled Tools; duplicate/malformed/relative entries rejected; FFM JAR trust separately required; guest capabilities independent |
| Changelog | [`spec/PROTOS_SPEC_CHANGELOG.md`](https://github.com/guillermomolina/protos/blob/ee8be92c482cc2451e888d444d74b382b319f4f1/spec/PROTOS_SPEC_CHANGELOG.md) | New `0.1.506` entry records the three owners and D259-B2+A; preceding `0.1.505` retained |

The host-selected load name is passed unchanged to the OS library loader. The OS loader may perform its own dependency search, symbolic-link resolution and library initialization; Protos neither searches an additional set of directories nor pins an immutable file identity. Native-side operations may use OS-process permissions outside the corresponding Protos guest grants. A denied import may still have created a logical provider session, but `protos-ffm` must not create its native acquisition resources for that denied library.

## 2. Ratification invariants and implementation boundary

The published normative text reconciles the D259 owner-approved **B2 plus configuration A** choices with the pre-existing FFI and Actor rules, rather than silently disabling Protos capabilities or conflating native globals with Protos-managed state. No C++ ABI, callbacks/upcalls, Native Image dynamic plugin loading, Landlock, worker/IPC sandbox or per-library confinement is added. The permitted ABI remains the established R1.

Read-only verification at the exact Protos revision confirms that the actual executable implementation is still **pre-I169**:
- [`FfmLibraries.java`](https://github.com/guillermomolina/protos/blob/ee8be92c482cc2451e888d444d74b382b319f4f1/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmLibraries.java) still contains the hardcoded `sqlite3` default, `libsqlite3.so.0` and legacy `sqlite3.path` policy.
- [`FfmForeignProvider.java`](https://github.com/guillermomolina/protos/blob/ee8be92c482cc2451e888d444d74b382b319f4f1/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmForeignProvider.java) still restricts canonical targets according to that old policy.
- [`FfmSession.java`](https://github.com/guillermomolina/protos/blob/ee8be92c482cc2451e888d444d74b382b319f4f1/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmSession.java) has not yet moved the authorization ahead of its initial native `Linker`/`Arena` allocation.
- [`ProtosCli.java`](https://github.com/guillermomolina/protos/blob/ee8be92c482cc2451e888d444d74b382b319f4f1/src/main/java/com/guillermomolina/protos/cli/ProtosCli.java) does not yet accept `--allow-library`.

This is an expected normative-first checkpoint, **not** an implementation PASS and not a reason to close I169. It also does not satisfy unrelated D256 / PLAT074 / LIB031 SQLite durable-runtime gates.

## 3. Validation provenance

The maintainer explicitly reported, on 2026-11-21 after pushing the normative commit, **`git diff check esta limpio`** and **`todos los tests pasan en local`**. Those are maintainer-reported local results; no exact executed command list or test logs were supplied to this evidence record, and the coordinating agent ran no validation. The exact GitHub publication and changed paths were independently read via GitHub. This publication changed only normative Markdown, so no new I169 Java/CLI/FFM behavior or issue-specific tests could have been validated by this commit.

| Gate | Evidence status |
| --- | --- |
| D259 owner approval and ratified project design | PASS (previously published design authority) |
| Normative Protos specification published | **PASS** — Protos `ee8be92c482cc2451e888d444d74b382b319f4f1`, spec `0.1.506` |
| Normative trust/Actor delta explicitly delimited | **PASS** — FFI, ACTORS §24K.1, PROCESS_IO |
| Maintainer diff check | **PASS (reported)** — no raw transcript |
| Maintainer local test suite | **PASS (reported)** — no per-test log or command supplied |
| I169 executable implementation | **NOT STARTED / NOT PUBLISHED** |
| I169-specific executable tests | **NOT RUN / NOT PUBLISHED** |
| Independent implementation version / CHANGELOG | **NOT APPLICABLE TO SPEC-ONLY COMMIT; pending implemented slice and validation** |

## 4. Next-action handoff

The unique **normative publication blocker** originally recorded on I169 is resolved. The actionable Issue state should move from `status:blocked` to `status:ready`, while retaining `family:I`, open state, unassigned ownership and unset Priority. The existing Issue continues to own the entire implementation, with no new formal implementation identifier required.

The next implementation slice should modify the existing FFM provider, CLI, embedding configuration usage, Protos-first fixtures and distribution tests/guides under the ratified contract, requiring verifiable Java-method/annotation/behavior equivalents in GraalPy or another Truffle language. No retired guest execution path or per-library Java wrapper may be reintroduced. The human-executor model owns Protos code validation, commit and push; only after those are verified should implementation completion be claimed.

```text
EVIDENCE_OWNER=I169
EVIDENCE_KIND=D259_B2_A_NORMATIVE_SPEC_PUBLICATION_AND_UNBLOCK
PROTOS_REVISION=ee8be92c482cc2451e888d444d74b382b319f4f1
SPEC_REVISION=0.1.506
DESIGN_AUTHORITY=D259/#1139
IMPLEMENTATION_ISSUE=I169/#1140
NORMATIVE_SPEC_PUBLICATION=PASS
GITHUB021_INVARIANT_RECONCILIATION=PASS
MAINTAINER_DIFF_CHECK=PASS_REPORTED
MAINTAINER_LOCAL_TESTS=PASS_REPORTED_UNSPECIFIED
FFM_IMPLEMENTATION=NOT_PUBLISHED
FFM_IMPLEMENTATION_TESTS=NOT_RUN
NEXT_ISSUE_STATUS=READY
```
