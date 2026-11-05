# BUG025 — Real-DAP closure regression publication (0.3.445)

## Source identity and authority

- **Date:** 2026-11-05.
- **Owning issue:** [BUG025 / #940](https://github.com/guillermomolina/protos/issues/940).
- **Exact published Protos revision:** [`5fc18249ea425d99fcfcfe387be950678b7ac5c5`](https://github.com/guillermomolina/protos/commit/5fc18249ea425d99fcfcfe387be950678b7ac5c5); commit subject: `BUG025: add real DAP exception-classification matrix and full-example closure tests (0.3.445)`. Confirmed on `main`, parent `6787089691803824d7596e02c98ab69c4d3c92e3`.
- **Runtime implementation fix:** [PLAT070, `6787089691803824d7596e02c98ab69c4d3c92e3`](https://github.com/guillermomolina/protos/commit/6787089691803824d7596e02c98ab69c4d3c92e3) adopted Truffle 25.5 direct `unwindExceptions` and removed the private control-transfer bridge; this BUG025 revision contains no production-code fix.
- **Version metadata:** `pom.xml` updated to `0.3.445-SNAPSHOT`; matching root `CHANGELOG.md` section was included in the same commit.
- **Observed validation:** maintainer reports `git diff --check` clean; `mvn test -Dtest=ProtosBug025RealDapClosureTest` PASS; the combined `ProtosBug025RealDapClosureTest,ProtosI026FDapBehaviorTest,ProtosCli008C1GuestStackCliTest` focused tests PASS; `make test` PASS; `make check` PASS; product commit pushed. These are maintainer-reported local executions, **not** reruns by the evidence publisher. Exact final Surefire `BUG025_*` output and CI/native logs were not supplied.
- **Change identity:** GitHub reports exactly six paths, 886 additions and 43 deletions: four Java test paths, `pom.xml`, and `CHANGELOG.md`. No product `src/main`, `protos/lib`, normative `spec`, original example, Truffle dependency or production DAP code is changed.

## Published regression and test infrastructure

1. [`ProtosI026FDapBehaviorTest.java`](https://github.com/guillermomolina/protos/blob/5fc18249ea425d99fcfcfe387be950678b7ac5c5/src/test/java/com/guillermomolina/protos/execution/integration/interop/ProtosI026FDapBehaviorTest.java) adds a real DAP matrix with exactly three exception filters `none`, `all`, `uncaught` and guest cases A–E. A (valid `Array.findIndex` non-local return) has a completed value `1` and no stops under `uncaught`; B is a truly unhandled `Error().signal()` and must produce exception stops under `all` and `uncaught`. All cases have zero stops under `none`. Guest outcomes C/D/E remain strictly checked while their DAP catch classification is explicitly observational for PLAT071.
2. [`ProtosBug025RealDapClosureTest.java`](https://github.com/guillermomolina/protos/blob/5fc18249ea425d99fcfcfe387be950678b7ac5c5/src/test/java/com/guillermomolina/protos/execution/integration/interop/ProtosBug025RealDapClosureTest.java) starts the **real** `ProtosCli debug` entry on the **unchanged** [`protos/examples/concurrency/parallel-execution.protos`](https://github.com/guillermomolina/protos/blob/5fc18249ea425d99fcfcfe387be950678b7ac5c5/protos/examples/concurrency/parallel-execution.protos) in fresh DAP sessions for all three filters. It gets the loopback host/port from `PROTOS_DEBUG_READY`; records and resumes stops; reconstructs guest stdout from DAP `output` events; excludes only the adapter's exact `Debugger attached.` message from guest stderr; and handles adapter-initiated transport close after `terminated`.
3. The full-program test strictly requires exit code 0, byte-for-byte full stdout, empty guest stderr, and **zero stops at the actual valid `^index` source line** under `none`, `all` and `uncaught`. With `none`, it also asserts zero total stops. Other guest Error stops under `all`/`uncaught` are recorded and not silently suppressed or asserted away; their classification is outside the original valid-NLR bug.
4. [`ProtosCli008C1GuestStackCliTest.java`](https://github.com/guillermomolina/protos/blob/5fc18249ea425d99fcfcfe387be950678b7ac5c5/src/test/java/com/guillermomolina/protos/cli/ProtosCli008C1GuestStackCliTest.java) now executes the exact same complete example using ordinary CLI without DAP, asserting exit 0, exact stdout and empty stderr.
5. [`ProtosDapTestSupport.java`](https://github.com/guillermomolina/protos/blob/5fc18249ea425d99fcfcfe387be950678b7ac5c5/src/test/java/com/guillermomolina/protos/execution/integration/interop/ProtosDapTestSupport.java) adds a validated `DapClient.connect(DebugEndpoint, Duration)` overload, preserves loopback-only connecting, prevents short poll windows from truncating partially received DAP messages, and distinguishes a clean adapter socket close *between* complete messages from malformed/incomplete messages.
6. [`pom.xml`](https://github.com/guillermomolina/protos/blob/5fc18249ea425d99fcfcfe387be950678b7ac5c5/pom.xml) and [`CHANGELOG.md`](https://github.com/guillermomolina/protos/blob/5fc18249ea425d99fcfcfe387be950678b7ac5c5/CHANGELOG.md) publish 0.3.445-SNAPSHOT together with the test-only scope and the PLAT071 distinction.

The full example must produce eight lines, with a final newline:

```text
42
[9, 1, 16, 1, 25, 81, 4, 36]
[4, 2, 6]
2
null
31
null
[1, 1, 2, 3, 4, 5, 6, 9]
```

The `2` comes from `parallelFindIndex`, which internally uses `findIndex(children, ...)` and the library's valid `^index`. The DAP test also checks the source location/line contents to distinguish that transfer from unrelated guest Error stops. The published matrix retains the earlier breakpoint/step/scope test and positive unhandled-Error checks.

## Previously observed classification matrix and separate PLAT071

The maintainer previously provided the following **actual real-DAP log** for the five scenarios (6 test methods PASS). Those counts are an earlier local run; the published final source preserves the corresponding tests, and the maintainer reports green validation, but the final run's complete Surefire log has not been provided. Do not misrepresent these counts as a separately independently measured post-publication run.

| Scenario | `none` | `all` | `uncaught` | Guest outcome |
| --- | ---: | ---: | ---: | --- |
| A valid non-local `^` | 0 | 0 | 0 | completed 1 |
| B unhandled ordinary Error | 0 | 1 | 1 | failed Error |
| C handled Error, same Task | 0 | 1 | **1, incorrect classification** | completed 42 |
| D producer InvalidReturn, consumer handles Future observation | 0 | 2 | 2 | completed, returns InvalidReturn Error |
| E producer ordinary Error, consumer handles Future observation | 0 | 2 | 2 | completed, returns ordinary Error |

The handled guest Error problem C/D/E belongs to independent [PLAT071 / #1054](https://github.com/guillermomolina/protos/issues/1054), documented by [PLAT071 pre-publication research](../PLAT071/PLAT071_DYNAMIC_ERROR_HANDLER_DAP_CLASSIFICATION_INVESTIGATION_2026_11_05.md). The product commit above now makes the matrix source durable; the prior PLAT071 research record remains an historical snapshot, not a statement that the tests remain unpublished. PLAT071 stays open and not yet ratified. A producer failure is one exception occurrence; `Future.value()` re-signals the *same guest Error object* as a separate consumer occurrence, so the producer's legitimate uncaught exception cannot be ignored or globally filtered.

## Limits and closure boundary

- **JVM with real GraalVM/Truffle 25.5 DAP:** the full unchanged example was validated under `none`/`all`/`uncaught`, plus ordinary CLI. Maintainer reports both focal Maven validations, `make test`, `make check` green.
- **Native distributed binary and installed VS Code extension:** **NOT VERIFIED** against the current `0.3.445` source. The original report concerned an older installed Native Image `0.3.392-SNAPSHOT`; no Native Image acceptance run, distribution asset/release or Marketplace test is claimed. These are explicit platform parity follow-ups, not fictitious PASS evidence.
- **Generic DAP guest `Error.handle` classification:** **NOT FIXED**; PLAT071 remains an independent architecture question. The BUG025 change does not add `TryBlockTag`, change task or Future semantics, suppress guest Errors, or modify the runtime.
- **Source vs release:** `0.3.445-SNAPSHOT` is a published commit, not a tag, release or new Native distribution.
- **Scope of BUG025 closure:** the originally reported false *uncaught guest Error* for **valid internal non-local return** is fixed by PLAT070 and now protected by published real-CLI and real-DAP regression tests. Closing BUG025 does not claim Native 0.3.445 parity or resolution of PLAT071.

## GITHUB020 closure evidence fields

```text
WORK_ITEM=BUG025
FAMILY=BUG
PROTOS_REVISION=5fc18249ea425d99fcfcfe387be950678b7ac5c5
SOURCE_FIX_REVISION=6787089691803824d7596e02c98ab69c4d3c92e3
VERSION=0.3.445-SNAPSHOT
MAINTAINER_GIT_DIFF_CHECK=CLEAN
MAINTAINER_FOCAL_MAVEN=PASS
MAINTAINER_MAKE_TEST=PASS
MAINTAINER_MAKE_CHECK=PASS
FULL_EXAMPLE_REAL_DAP_FILTERS=NONE_ALL_UNCAUGHT
VALID_NLR_GUEST_ERROR_STOPS=ZERO_AT_CARET_INDEX
UNHANDLED_ERROR_POSITIVE_CONTROL=PRESERVED
NATIVE_DISTRIBUTION_AND_EXTENSION=NOT_VERIFIED
PLAT071_GUEST_HANDLED_ERROR_CLASSIFICATION=UNRESOLVED
DURABLE_RECORD_DECISION=REQUIRED
```

This non-normative evidence record must be cited by an exact docs commit revision in the final BUG025 issue closure comment.
