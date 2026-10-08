# PERF031-F/G — Exact publication verification and global residual-work handoff

Date: 2026-10-08

## Publication identities and provenance

```text
ISSUE=guillermomolina/protos#787
PRODUCT_REPOSITORY=guillermomolina/protos
RECORD_REPOSITORY=guillermomolina/protos-project-docs
PERF031_F_PRODUCT_SHA=e8678245192dc8f3e90b4b8a0ce10d092a3ef671
PERF031_F_VERSION=0.3.290-SNAPSHOT
PERF031_G_PRODUCT_SHA=c8a27d2a11afca4459d73de43de0f036e6857b21
PERF031_G_VERSION=0.3.292-SNAPSHOT
PRODUCT_MAIN_AT_VERIFICATION=c8a27d2a11afca4459d73de43de0f036e6857b21
OWNER_REPORTED_GIT_DIFF_CHECK=CLEAN
OWNER_REPORTED_LOCAL_TESTS=PASS
TESTS_RUN_BY_RECORDING_AGENT=NO
MEASURED_SPEEDUP=NOT_ESTABLISHED
SPECIFICATION_CHANGE=NO
PERF031_STATE=OPEN_READY
PERF031_PRIORITY=INTENTIONALLY_UNSET
```

Both published revisions were fetched and their changed-file patches reviewed from current GitHub history. The earlier [PERF031-E JFR record and F publication verification gate](PERF031_E_JFR_CAUSAL_AUDIT_AND_F_PUBLICATION_VERIFICATION_GATE_2026_10_08.md) correctly recorded the temporary absence of an externally visible F commit **as of that earlier check**. This checkpoint resolves the historical *publication verification* gate without rewriting the contemporaneous E evidence.

## PERF031-F — verified grouped Test Tool progress optimization

[Exact F commit](https://github.com/guillermomolina/protos/commit/e8678245192dc8f3e90b4b8a0ce10d092a3ef671).

The product commit changes exactly seven paths: `protos/tools/test/Main.protos`, `protos/tools/test/RepositorySuite.protos`, three Java test classes, `pom.xml`, and `CHANGELOG.md`.

- `RepositorySuite.progressGroupAssignment(leaf, caseKeys)` generates names and aligned Case-group indexes together; it validates conformance descriptors once per Case, collects first-appearance subgroups without per-Case linear name searches and indexes names through local Maps without depending on iteration order.
- `Main.protos` retains per-suite Case progress keys during discovery instead of re-filtering the global Case-key set once per suite. The assignment is consumed once per suite. The `--list-cases` branch still avoids progress state and logical execution.
- Structural verification tests updated: `ProtosTestToolTool011ProgressGroupingTest.java`, `ProtosTestToolTool004CProgressPresentationTest.java`, `ProtosTestToolTool012TomlOfficialProgressGroupingTest.java`. TOOL012 additionally asserts full-corpus assignment equivalence with existing name/index projections.
- Array spread appends can remain quadratic for large suites because ordinary Protos Array has no append/grow API; that residual is explicitly retained and is not an authorization to invent collection semantics.

The maintainer earlier reported all required local tests PASS and `git diff --check` clean. Exact class counts and any post-F comparable timing measurement were not supplied. The recording agent did not execute those gates; no timing improvement is quantified.

## PERF031-G — verified grouped Package Tool/Run Driver fixture reuse

[Exact G commit](https://github.com/guillermomolina/protos/commit/c8a27d2a11afca4459d73de43de0f036e6857b21).

The product commit changes exactly four paths:

1. `src/test/java/com/guillermomolina/protos/execution/ProtosPackageToolProtosTestSupport.java`
2. `src/test/java/com/guillermomolina/protos/execution/ProtosPackageRunDriverTestSupport.java`
3. `pom.xml`
4. `CHANGELOG.md`

In the first helper, `executeExternalCaptureFixture` now opens its fresh hosted fixture with `SharedExecutionPlanCore.PRELUDE`, initialized lazily by `newPackagePrelude()`. The other callers of `newPackagePrelude()` were intentionally not modified. In the second helper, each real `digest(Path)` invocation still captures the exact package tree and evaluates bundled `ContentIdentity.digest` inside its own hosted Process, but uses one lazily prepared `SharedDigestCore.PRELUDE` instead of bootstrapping Core again for every digest.

The source-level **claimed structural counts**, not stopwatch measurements, are: one avoided bootstrap per repeated external-capture fixture call; for the original four Run Driver concrete classes, three `@BeforeAll` digests each meant 12 calls to the Core bootstrap path, now one holder initialization per test JVM if exercised. Verify actual execution and concurrency when making future performance claims. Shared holders retain no project-tree backend, custody, hosted fixture, Process, activation, or Actor-local module state. Every invocation continues to open independent resource custody, create a fresh hosted Process and activation, and close owned resources. No invalid graph or digest fixture was removed in the published diff.

The maintainer explicitly reported after push: `git diff --check` clean and all local tests PASS. Exact run outputs and test counts were not supplied to the recording agent, and no tests or builds were executed by it. The owner report does not establish a measured speedup.

## Baseline and evidence interpretation

The prior [PERF031-D complete Java cohort](PERF031_D_FULL_JAVA_COHORT_AND_CAUSAL_TRIAGE_2026_10_08.md), measured at `guillermomolina/protos@a463429f34d677ad896279861fa09d9dea251c1a`, observed 548 classes, 2,811 tests (one skipped), 143.971 s accumulated class timing and a list of 23 classes above 1.0 s; this is a **historical pre-F/G cohort**.

The [PERF031-E JFR diagnostic](PERF031_E_JFR_CAUSAL_AUDIT_AND_F_PUBLICATION_VERIFICATION_GATE_2026_10_08.md) recorded 17/17 passing diagnostic tests in four independently forked Surefire JVMs. Sampling found substantial generated interpreter activity, but neither sampled Java method percentages nor non-exclusive stack ancestry isolate one Protos operation's wall-time contribution. In particular, `--list-cases` does not execute the progress grouping optimized in F. Do not compare profiled per-class times with the ordinary serial cohort as a paired speedup estimate.

The original historically material groups were:

| Historical group | Class-time sum before F/G | Current technical disposition |
|---|---:|---|
| A: Test Tool progress and corpus traversal | 25.680 s | F removes specific repeated grouping calculations; true post-F impact unmeasured, discovery/listing costs may remain |
| B: Package Tool invalid graph and capture fixtures | 19.545 s | G reduces redundant fixture Prelude initialization; real graph/custody checks preserved; impact unmeasured |
| C: Package Run Driver | 19.353 s | G reduces redundant digest Prelude initialization; real digest and runtime/custody checks preserved; impact unmeasured |
| D: Public integration, Process/Actor and Regex | Separate class timings in PERF031-D | Retain legitimate end-to-end correctness unless further causal evidence demonstrates safe reducible work |

No narrow static source refactoring, JFR stack-count result or owner-reported PASS alone establishes that all materially expensive classes are now proportional or that PERF031 can close.

## Next work: PERF031-H — one global residual-cost reconciliation, investigation only

**Do not initiate another one- or two-file optimization as the default next step.**

Investigate the complete *current* surviving Java cost cohort, grounded in exact published source and PERF031-D/E history. For every materially expensive class, assign a disposition: (a) already addressed structurally by B/C/F/G; (b) still a justified integration/authority/concurrency/CLI workload; (c) demonstrably avoidable repeated cost with a bounded, meaningful common owner; (d) genuine insufficient evidence requiring at most one grouped, human-executed measurement; or (e) residual correctly routed to an existing independently owned issue.

Pay particular attention to F's untouched full-corpus discovery and spread-based retained-Case projections, G's untouched real digests and per-case custody, legitimate Regex Future/Actor/Polyglot transitions, and the correctness obligations behind public Test Tool and workspace/Package CLI tests. Do not attribute all historical class time to any one suboperation.

**Deliverable:** one evidence-based final reconciliation table covering the 23 historical >1 s classes, the remaining same-host measurement gaps, closure criteria per class, and at most one substantive *grouped* implementation proposal if material avoidable cost survives. If all survivors are proportional or belong to explicit other owners, propose evidence-backed PERF031 closure instead. No mandatory new test run unless necessary for a specific falsifiable question; do not rerun already-passing tests without changed code.

Investigations read GitHub and repository evidence only; the investigation agent executes no code/commands/builds/tests, makes no local Git changes and publishes no product commit. Only the human executes validation and product publication when a later implementation is actually authorized.

## Scope, issue state, and stop conditions

- `guillermomolina/protos#787` remains **OPEN / READY**, unassigned and without invented priority.
- F and G are completed **implementation publications**, not proved wall-time performance wins.
- No new Issue is needed solely to name the bounded H investigation.
- No language/spec semantics, TEST008/PLAT047 guard/admission budgets, suite correctness, corpus extent, Process/Actor authority, lifecycle or filesystem-custody boundaries may be relaxed.
- PERF032's compiler-graph parity investigation is a separate owner and must not be folded into PERF031 for convenience.
- The project owner explicitly requests fewer micro-slices. Group only genuinely causal and validation-coherent implementation changes; when the remaining benefit is marginal, close/document rather than manufacture work.

## Files and project authority inspected

- `guillermomolina/protos/AGENTS.md`, `AGENTS.work/PERFORMANCE.md`, `AGENTS.work/COORDINATION.md`, and the applicable shared-implementation discipline.
- Exact published F/G commit patches, `pom.xml`/root `CHANGELOG.md`, and current repository state.
- Prior PERF031-D/E durable evidence.
- `guillermomolina/protos-project-docs/AGENTS.md` and role-first evidence path contract.

Publication of this record is documentation/evidence only and does not approve an unratified design.
