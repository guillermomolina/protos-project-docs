# PERF031-J — Final Java-test cost acceptance reconciliation and closure evidence

Date: 2026-10-08

## Authority and identity

- **Owning issue:** [PERF031 / #787](https://github.com/guillermomolina/protos/issues/787)
- **Investigation:** PERF031-J, read-only source/evidence review; no product edits, builds, tests, benchmarks, or specification decisions
- **Current product revision reviewed:** [`guillermomolina/protos@6dab6ecc08c9a2102a388e00908c710a15cf2a4c`](https://github.com/guillermomolina/protos/commit/6dab6ecc08c9a2102a388e00908c710a15cf2a4c)
- **Final PERF031 implementation:** [`d0f8ea1c6f917dd1e0c16d7529d8b94e5310b039`](https://github.com/guillermomolina/protos/commit/d0f8ea1c6f917dd1e0c16d7529d8b94e5310b039), 0.3.297-SNAPSHOT
- **Previously published documentation baseline:** `guillermomolina/protos-project-docs@68515e4eb35348b1d5a5ee95fc9ee70650d9499d`
- **Conclusion:** `CLOSE_PERF031` recommended, conditional on the normal GITHUB020 issue comment and closure-state verification
- **Observable semantics changed by this investigation:** none

The later product HEAD `6dab6ecc08c9a2102a388e00908c710a15cf2a4c` includes separate embedding/Filesystem work and does not modify the PERF031 Test Tool accumulators or fixture reuse paths. This review applies the actual #787 acceptance rule: each materially expensive class must have evidence of proportional integration cost, semantics-preserving reduction, appropriate stress routing, or an exact independently owned performance issue. No arbitrary speedup target is added.

## Evidence reviewed

- [PERF031_A_JAVA_INTEGRATION_TEST_COST_CAUSAL_AUDIT.md](https://github.com/guillermomolina/protos-project-docs/blob/73b4ff4ccbb10b5a1fcf9569dc43438bebc75933/docs/project/evidence/PERF031/PERF031_A_JAVA_INTEGRATION_TEST_COST_CAUSAL_AUDIT.md)
- [PERF031_B_REDUCED_TEST_CORE_BOOTSTRAPS_PUBLICATION.md](https://github.com/guillermomolina/protos-project-docs/blob/73b4ff4ccbb10b5a1fcf9569dc43438bebc75933/docs/project/evidence/PERF031/PERF031_B_REDUCED_TEST_CORE_BOOTSTRAPS_PUBLICATION.md)
- [PERF031_C_PACKAGE_LOGICAL_CASE_CORE_REUSE.md](https://github.com/guillermomolina/protos-project-docs/blob/73b4ff4ccbb10b5a1fcf9569dc43438bebc75933/docs/project/evidence/PERF031/PERF031_C_PACKAGE_LOGICAL_CASE_CORE_REUSE.md)
- [PERF031_D_FULL_JAVA_COHORT_AND_CAUSAL_TRIAGE_2026_10_08.md](https://github.com/guillermomolina/protos-project-docs/blob/73b4ff4ccbb10b5a1fcf9569dc43438bebc75933/docs/project/evidence/PERF031/PERF031_D_FULL_JAVA_COHORT_AND_CAUSAL_TRIAGE_2026_10_08.md)
- [PERF031_E_JFR_CAUSAL_AUDIT_AND_F_PUBLICATION_VERIFICATION_GATE_2026_10_08.md](https://github.com/guillermomolina/protos-project-docs/blob/73b4ff4ccbb10b5a1fcf9569dc43438bebc75933/docs/project/evidence/PERF031/PERF031_E_JFR_CAUSAL_AUDIT_AND_F_PUBLICATION_VERIFICATION_GATE_2026_10_08.md)
- [PERF031_F_G_PUBLICATION_AND_GLOBAL_RESIDUAL_HANDOFF_2026_10_08.md](https://github.com/guillermomolina/protos-project-docs/blob/73b4ff4ccbb10b5a1fcf9569dc43438bebc75933/docs/project/evidence/PERF031/PERF031_F_G_PUBLICATION_AND_GLOBAL_RESIDUAL_HANDOFF_2026_10_08.md)
- [PERF031_H_INDEPENDENT_CAUSAL_REVIEW_AND_I_GROUPED_HANDOFF_2026_10_08.md](https://github.com/guillermomolina/protos-project-docs/blob/73b4ff4ccbb10b5a1fcf9569dc43438bebc75933/docs/project/evidence/PERF031/PERF031_H_INDEPENDENT_CAUSAL_REVIEW_AND_I_GROUPED_HANDOFF_2026_10_08.md)
- [PERF031_I_GROUPED_CASE_SCALE_ARRAY_PUBLICATION_2026_10_08.md](https://github.com/guillermomolina/protos-project-docs/blob/73b4ff4ccbb10b5a1fcf9569dc43438bebc75933/docs/project/evidence/PERF031/PERF031_I_GROUPED_CASE_SCALE_ARRAY_PUBLICATION_2026_10_08.md)

Implemented revisions:
- B: `b7f7175c46134585c1b45d216b6adbcde0896220` — immutable shared Core Prelude in five test classes.
- C: `a463429f34d677ad896279861fa09d9dea251c1a` — Package logical Case fixture Core reuse.
- F: `e8678245192dc8f3e90b4b8a0ce10d092a3ef671` — grouped progress assignment and elimination of per-suite global Case refiltering.
- G: `c8a27d2a11afca4459d73de43de0f036e6857b21` — immutable Prelude reuse in external-capture and Run Driver digest fixtures.
- I: `d0f8ea1c6f917dd1e0c16d7529d8b94e5310b039` — internal binary-carry Case accumulation in Main and public CaseSelection listing.

## Final 23-class disposition

Historical PERF031-D run at `a463429f34d677ad896279861fa09d9dea251c1a`: 548 classes, 2,811 tests, zero failures, zero errors, one skipped, and 143.971 s cumulative Surefire class times. These times are **historical and pre-F/G/I**; they are not current costs, per-operation attribution or before/after evidence.

- **SE** = a specific structurally avoidable mechanism relevant to the class has been addressed; remaining independently required operations are retained.
- **PI** = independent ordinary/integration coverage is justified by source contracts and no other material avoidable work has been established.
- Confidence is in the source-level disposition, not a quantitative cost attribution.

| # | Java class (Protos prefix omitted) | Historical class s | Invariant proved / remaining real work | Completed work and final disposition |
|---:|---|---:|---|---|
| 1 | TestToolTool012TomlOfficialProgressGroupingTest | 19.216 | Complete TOML corpus group partition, exact owners, case counts and public list-only selection | F removes redundant grouping; I removes per-Case repeated-prefix copies; full corpus and CLI traversal retained. **SE / high** |
| 2 | PackageToolExecutionPlanInvalidIdentityGraphTest | 10.093 | Eight distinct invalid identity/registry graphs rejected through real verified capture | G shares immutable Package Prelude; fresh captures and error fixtures retained. **SE / high** |
| 3 | RegexSemanticTransferTest | 9.486 | Eight Regex/Actor/parallel transfer, destination-rebuild and authority scenarios | Real Futures and transfers retained; JFR busy-wait samples do not establish avoidable wall-time cost. **PI / medium-high** |
| 4 | TestToolCaseRejectionPublicIntegrationTest | 8.916 | Five public exact-ref fail-closed boundaries before scheduling | Separate CLI rejection cases are necessary; I benefits discovery where reached. **PI / high** |
| 5 | PackageRunDriverMixedApplicationTest | 7.322 | Three real application, verified capture, Network and termination contracts | G shares immutable digest Core preparation; live application/custody remain separate. **SE / high** |
| 6 | TestToolTool011ProgressGroupingTest | 6.464 | Group identity, descriptor order, complete paths, invalid paths and list-only isolation | F single-pass assignment; I bounded Case accumulation; representative assertions retained. **SE / high** |
| 7 | PackageToolExecutionPlanCaptureTest | 5.347 | Verified transitive registry/Git capture and missing/unlocked descriptor failures | G shares fixture Prelude, preserving separate capture and verification. **SE / high** |
| 8 | PackageRunDriverPlanningFailureCustodyTest | 4.653 | Failure cleanup, custody closure and no authority transfer | G reduces digest Prelude preparation only. **SE / high** |
| 9 | WorkspacePackageAuthorityIsolationIntegrationTest | 4.337 | Four Tool/Application Process isolation, Filesystem and Network cases | Separate Processes and authority contexts are the invariant; no reducible repeated work established. **PI / high** |
| 10 | PackageToolExecutionPlanInvalidGitAndEdgeGraphTest | 4.105 | Seven invalid Git/edge graph variants rejected | G immutable Prelude reuse; all vectors and real capture/rejection retained. **SE / high** |
| 11 | PackageRunDriverTest | 3.796 | Three workspace/default-network/explicit-grant execution paths | G shared immutable digest Prelude; distinct Processes and projects retained. **SE / high** |
| 12 | PackageRunDriverFailureCustodyTest | 3.582 | Two provider/materialization failures; cleanup before application start | G shared Prelude; real negative-path verification retained. **SE / high** |
| 13 | TestToolCaseSelectionPublicIntegrationTest | 3.538 | Three public list/select/out-of-scope rejection contracts | I bounds public Case listing construction, preserving D185 exact order and output. **SE / high** |
| 14 | WorkspaceRunCliTest | 3.356 | Seven public CLI/args/materialization/error and application execution scenarios | Real workspace execution/Process startup and provider checks are part of the contract. **PI / high** |
| 15 | TestToolCaseExecutionPublicIntegrationTest | 3.213 | Two selected-Case execution and reverse CLI argument-order cases | I benefits discovery; selected case bodies must still execute authentically. **PI / high** |
| 16 | PackageExecutionPlanAdapterTest | 2.712 | Immutable detached plan, malformed shape rejection and independence | Class-scoped Prelude already shared; separate hosted fixtures/plans retained. **PI / high** |
| 17 | PackageToolContentIdentityTest | 2.291 | Seven canonicalization, digest, vectors, symlink and captured-custody contracts | Real digest and verification retained; no additional material duplication established. **PI / medium-high** |
| 18 | TestToolFileSelectionPublicIntegrationTest | 2.123 | Public exact file selection, four authentic Cases and observable CLI output | I improves discovery; real CLI and Case execution cannot be replaced by mocks. **PI / high** |
| 19 | ExternalPackagePlanningPreflightTest | 1.936 | Two borrowed-custody, source-deletion, planning and termination contracts | B shares immutable assertion-helper Prelude while creating fresh activations. **SE / high** |
| 20 | ExactExternalRequirementsPreflightTest | 1.814 | Six exact requirements/lock/scope and fail-closed preflight scenarios | Distinct verification/Process states remain meaningful. **PI / high** |
| 21 | CliTest | 1.812 | Nineteen command-line, bootstrap, printing, errors and argument behaviors | Aggregate public-entry cost; no demonstrated material repeated work. **PI / high** |
| 22 | PackageToolManifestTest | 1.081 | Five valid/invalid/missing/UTF-8/chunked manifest cases | Authentic parsing and I/O are required; no demonstrated material duplicated setup. **PI / high** |
| 23 | FilesystemLibraryConformanceTest | 1.056 | Twelve bounded I/O, cancellation, Future completion, cleanup and error-precedence contracts | Class-scoped Core bootstrap; fresh per-test backend and lifecycle resources retained. **PI / high** |

## Final source-grounded checks

**Test Tool:** At the reviewed product revision, `protos/tools/test/Main.protos` imports the internal `ArrayAccumulator` and uses separate collectors for logical projections, invocation-wide Case references and metadata, and each suite's retained Case progress keys. `ArrayAccumulator.finish` produces ordinary Arrays in older-first insertion order; `Main` retains freezing at the prior boundaries. `protos/tools/test/CaseSelection.protos` uses the same collector before producing `protos.test.cases/v1` JSON. `--list-cases` finishes before grouping, progress state, scheduling, Case Processes or bodies. Published Java regressions exercise list ordering across projections, reverse exact selection, empty listing, collector carry/identity/independence, and fail-closed selection. Remaining spread constructions in the reviewed `Main` and `RepositorySuite` paths are predominantly per-suite, descriptor or subgroup; no remaining materially large per-retained-Case quadratic prefix accumulator is identified. The presence of `Array(...old, value)` alone is not proof of a performance defect.

**Package Tool / Run Driver:** `ProtosPackageToolProtosTestSupport.executeExternalCaptureFixture` uses `SharedExecutionPlanCore.PRELUDE`; `ProtosPackageRunDriverTestSupport.digest` uses `SharedDigestCore.PRELUDE`. Each fixture/digest continues real capture, verification, separate hosted Process/activation, authority lifetime and cleanup. It is not valid to cache success, error or authority outcomes simply to shorten tests.

**Regex / Actor / public CLI:** Existing Regex/parallel/Actor tests and CLI/workspace tests exercise actual transfer, destination-local materialization, Future completion, authoritative file/case selection, error paths and observable execution. The PERF031-E JFR diagnostic found busy-wait/interpreter samples but did not establish a material wall-time or CI effect caused by a specific spin loop. Sample shares and non-exclusive stack ancestry are not elapsed-time shares. There is no justified transfer of ordinary test cost to PERF032 merely because both touch Graal.

**Historical correction:** The external PERF031-H proposal incorrectly reported 11 structurally addressed / 12 proportional rows (its rows actually totaled 10 / 13), and prematurely asserted that the large per-Case accumulators were already optimized. The retained independent H review identified those remaining quadratic paths; published PERF031-I has now addressed them. This J verdict supersedes that premature closure argument, while retaining the valid earlier B/C/F/G evidence.

## Acceptance and uncertainty

1. **All 23 material historical classes dispositioned:** PASS (table and source contracts).
2. **Demonstrated avoidable material work addressed:** PASS (B/C/F/G/I), with no new material residual established by the reviewed current paths.
3. **Correctness/integration evidence retained:** PASS on available source/regression and maintainer gate provenance; no scope/fixture/vector removals are asserted.
4. **No unresolved material contradiction:** PASS, including the specific Case prefix-copy oversight identified in H.
5. **Measurement limitations explicitly retained:** PASS.

This is not a claim that every slow method is optimally implemented or that all observed time is intrinsic; proportionality is a bounded acceptance conclusion grounded in the actual correctness obligations and absence of another *demonstrated* material avoidable mechanism. If future evidence identifies such a mechanism, it merits a new concrete causal owner rather than speculative further PERF031 slices.

**Validation provenance:** PERF031-B focal 25 tests PASS reported by maintainer; PERF031-C focal-class PASS reported; PERF031-D full serial cohort 2,811 tests with zero failures/errors and one skipped executed by maintainer; PERF031-E forked JFR diagnostic 17/17 PASS executed by maintainer; F/G/I all-local-tests PASS and clean `git diff --check` reported by maintainer. No exact post-F/G/I gate counts or raw logs were provided to the evidence-writing agent, and this investigation did not run validation.

**Timing limitations:** D predates F/G/I and is a single reduced-contention serial run rather than multiple isolated class JVM runs. E uses distinct fork topology, instrumentation and compilation. There is no paired post-I elapsed-time measurement and no numerical speedup claim. The Issue's actual acceptance criteria do not make such a comparison mandatory once all demonstrated structural problems are dispositioned.

## Closure coordination (GITHUB020)

```text
DECISION=CLOSE_PERF031
CLOSURE_EVIDENCE_IDENTIFIED=PASS
DURABLE_RECORD_DECISION=REQUIRED
REQUIRED_DURABLE_PUBLICATION=PASS_AFTER_COMMIT_AND_REREAD
PRODUCT_REVISION=6dab6ecc08c9a2102a388e00908c710a15cf2a4c
FINAL_PERF031_PRODUCT_REVISION=d0f8ea1c6f917dd1e0c16d7529d8b94e5310b039
ISSUE_CLOSURE_COMMENT=REQUIRED_BEFORE_CLOSURE
PROJECT_RECORD_REVISION=THIS_RECORD_PUBLICATION_SHA
```

The coordinating agent must re-read the exact published record after commit, post a compact GitHub closure comment with exact product/document revisions, check the live issue state and only then close #787 as completed. No new PERF implementation, code/tests, design ratification, perf threshold or independent new Issue is implied. TEST008/#761 and TEST008-A/#785 are already closed; PLAT054/#838, I086/#840 and PERF032/#831 retain separate outstanding acceptance/ownership.

Prepared with ChatGPT assistance. No agent-executed product tests, benchmark or independent human code review is claimed.
