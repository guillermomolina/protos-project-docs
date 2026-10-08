# PERF031-E — JFR causal audit and PERF031-F publication-verification gate

Date: 2026-10-08

## Authority and revisions

```text
PARENT_ISSUE=guillermomolina/protos#787
EVIDENCE_SLICE=PERF031-E
IMPLEMENTATION_SLICE=PERF031-F
PRODUCT_REPOSITORY=guillermomolina/protos
MEASURED_PRODUCT_REVISION=a463429f34d677ad896279861fa09d9dea251c1a
MEASURED_PRODUCT_VERSION=0.3.288-SNAPSHOT
LAST_VERIFIED_REMOTE_MAIN=3bb1278d91ee5cea98031462be2a5c4dd3c89019
PERF031_F_OWNER_REPORTED_PUSH=YES
PERF031_F_COMMIT_SHA=NOT_VERIFIED
PERF031_F_REMOTE_MAIN_PUBLICATION=NOT_VERIFIED
PERF031_F_AGENT_EXECUTED_TESTS=NO
PERF031_F_OWNER_REPORTED_LOCAL_GATES=PASS
PERF031_F_OWNER_REPORTED_DIFF_CHECK=CLEAN
PERF031_F_MEASURED_SPEEDUP=NOT_ESTABLISHED
PERF031_ISSUE_CLOSE=NO
```

The product revision used by the human-executed diagnostics was `a463429f`. The maintainer subsequently reported that `PERF031-F: single-pass Test Tool progress grouping` had been pushed, `git diff --check` was clean, and all required local tests had passed. **Those PASS statements are maintainer reports**, not tests performed by the documenting agent. The conversation did not supply exact post-implementation reports, a timing delta, or a commit SHA. During the 2026-10-08 verification, GitHub's `main` branch and commit-history/commit-search endpoints still exposed `3bb1278d` (the earlier PLAT054-3E1 specification publication), with no discoverable PERF031-F commit. Therefore this record is **not** evidence that PERF031-F is published on `main`; an exact commit/changed-path verification remains a required follow-up before a definitive F publication checkpoint.

## Pre-F source audit and measured cohort

The independently published [PERF031-D full Java cohort](PERF031_D_FULL_JAVA_COHORT_AND_CAUSAL_TRIAGE_2026_10_08.md) measured 548 classes / 2,811 tests / 0 failures / 0 errors / 1 skipped, with 143.971 s sum of class-reported timing on `a463429f`. The most material grouped owners were:

| Group | Classes / scope | Prior measured class-time sum |
|---|---|---:|
| A | TOOL011 and TOOL012 Test Tool grouping/selection | 25.680 s |
| B | Invalid identity / invalid Git and edge graphs / capture fixtures | 19.545 s |
| C | Four Package Run Driver classes | 19.353 s |

PERF031-E source inspection found the following mechanical repetition in the then-published Test Tool implementation: `protos/tools/test/RepositorySuite.protos` repeatedly scanned descriptor/Case arrays, performed `Arrays.findIndex` over progressively constructed names and rebuilt Arrays with spread. `protos/tools/test/Main.protos` re-filtered all final progress Case keys for each retained suite and then recomputed per-Case grouping indices. The `--list-cases` branch is before group/progress construction and **does not execute** those grouping operations; its corpus-discovery cost must not be charged to pure grouping.

Group B's `ProtosPackageToolProtosTestSupport.executeExternalCaptureFixture` opens a fresh Package Prelude, hosted fixture and independently captured project/external filesystems for each fixture vector; the latter per-case authority/custody must not be shared. Group C's `ProtosPackageRunDriverTestSupport.digestExternalTemplates` is class-level `@BeforeAll` setup across four concrete classes, with three fresh digest calls, each constructing a new bootstrap/hosted context and selected-root custody. The class-only versus summed-method timing gap of 4.454 s across the four classes is **not** a direct measurement of just setup.

## Human-executed JFR diagnostic

The maintainer executed one `make test-java-confirm` over four representative Java classes on `a463429f`, with JFR `settings=profile`, fork count one, `reuseForks=false`, and parallel JUnit disabled. Maven recompiled 562 test sources. One Maven JVM and four independent Surefire JVMs recorded profiles.

| Test class | Tests | Failures | JUnit class time, JFR run | Surefire PID | ExecutionSample |
|---|---:|---:|---:|---:|---:|
| ProtosRegexSemanticTransferTest | 8 | 0 | 11.87 s | 2053297 | 1,419 |
| ProtosPackageRunDriverMixedApplicationTest | 3 | 0 | 18.05 s | 2053493 | 1,635 |
| ProtosTestToolTool012TomlOfficialProgressGroupingTest | 5 | 0 | 31.27 s | 2053747 | 2,827 |
| ProtosPackageToolExecutionPlanInvalidIdentityGraphTest | 1 | 0 | 13.64 s | 2054072 | 1,221 |
| **Total** | **17** | **0** | | | |

All 17 passed, with zero errors and zero skips; the Maven command reported BUILD SUCCESS. PID 2053153 is Maven / `javac`, not a fifth test-class profile. The JFR test-run times must **not** be treated as paired before/after timings: fork topology, startup, profiling and compilation differ from the original serial-cohort baseline.

The top Java execution samples in `ProtosSemanticBytecodeRootNodeGen$CachedBytecodeNode.continueAt` were: Regex 472/1,419 (33.26%); Run Driver 1,019/1,635 (62.32%); TOOL012 1,109/2,827 (39.23%); Package Tool 651/1,221 (53.32%). Those are **fractions of sampled Java method activity**, not shares of elapsed wall time, and they conflate many guest operations within the generated interpreter.

An additional human-executed extraction examined the full stack traces of all Java ExecutionSamples:

| Route (non-exclusive stack membership) | Regex | Run Driver | TOOL012 | Package Tool |
|---|---:|---:|---:|---:|
| Regex await | 397 | — | — | — |
| Hosted fixture | 359 | 7 | — | 1 |
| Polyglot explicit context enter | 92 | — | — | — |
| Actor dispatch | 53 | 5 | 57 | 18 |
| Core bootstrap | 13 | 18 | 19 | 7 |
| Package Run Driver | — | 20 | — | — |
| Package digest | — | 12 | — | — |
| Public CLI | — | — | 85 | — |
| Package fixture | — | — | — | 35 |

Every `jdk.ExecutionSample` contained a stack. Stack-route counts are non-exclusive and **do not** establish proportional wall-time attribution. No `Contention by Site` events were seen in the extracted report; this does not exclude all forms of waiting. The profiles suggest real guest execution plus bootstrap/fixture overhead, but cannot isolate pure progress-group cost, repeated Package Prelude cost, or digest cost precisely.

## PERF031-F — maintainer-reported outcome, verification still required

The maintainer reported that an implementation agent edited five existing files, without executing the tests itself:

- `protos/tools/test/RepositorySuite.protos`: new `progressGroupAssignment(leaf, caseKeys)` computing frozen group `names` and per-Case `caseGroupIndexes`; previous public lookup/descriptor functions retained.
- `protos/tools/test/Main.protos`: collect per-suite progress Case keys during discovery; call the combined assignment once per suite, avoiding re-filtering the global Case-key list.
- Three existing Java test files: `ProtosTestToolTool011ProgressGroupingTest.java`, `ProtosTestToolTool004CProgressPresentationTest.java` and `ProtosTestToolTool012TomlOfficialProgressGroupingTest.java`, covering source-layout assertions and complete-corpus assignment equivalence.

This implementation description is **agent-reported, not yet checked against a published F product diff**. The maintainer reports that required local tests passed, `git diff --check` was clean and the work was pushed; GitHub source verification did not yet corroborate that publication. No measured speedup is claimed. The residual array appends by spread remain potentially quadratic because current ordinary Array has no append/growth API; this is not authority to invent one or weaken array semantics.

The implementation's intended invariants are stable group order, exact descriptor matching, fail-closed invalid paths/selectors, first-seen subdivided group order, one group index per Case, and no progress grouping in `--list-cases`. These are reported goals and regression assertions, not independently re-executed here.

## Next verification and work routing

1. Confirm the actual `PERF031-F` product commit SHA is reachable from `guillermomolina/protos/main`; read the exact diff and verify scope and license/metadata. Only then publish a **revision-bound F completion checkpoint** and amend the live #787 coordination with that SHA.
2. Preserve original human-reported PASS attribution. Do not replay tests without changed-code or validation-policy cause.
3. Keep PERF031 **OPEN / READY**, no inferred assignee or priority. TEST008/PLAT047 admission and PERF032's different runtime graph investigation remain separate.
4. Next candidate is a **grouped Java-test fixture-cost implementation** for groups B and C, gated by current HEAD and explicit proof that any shared state is immutable or test-only. Per-Case filesystem capture/custody, activations, Process/Actor/Context/Task/Future state, scope lifetime and failure assertion vectors remain independent. A later proportionality/same-host comparison may quantify any gain but is not a prerequisite to proving an algorithmic reduction.

This record does not allocate a new formal Issue, ratify a design, assert an unobserved performance improvement, or close PERF031.
