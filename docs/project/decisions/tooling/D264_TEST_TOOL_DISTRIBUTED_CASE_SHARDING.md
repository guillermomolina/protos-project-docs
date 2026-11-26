# D264 — Deterministic distributed Test Tool Case sharding

Status: **Owner-selected and ratified tooling contract — Candidate B′**.

Decision issue: [guillermomolina/protos#1180](https://github.com/guillermomolina/protos/issues/1180).
Decision research baseline: `guillermomolina/protos@0561cf229766373af4f3d2dc6ce8a70a93c2904f`.
Ratification review baseline: [28d3c1df6531e31fd795ed9673b31e18bb13fc82](https://github.com/guillermomolina/protos/commit/28d3c1df6531e31fd795ed9673b31e18bb13fc82).
Upstream decision-record review baseline: `guillermomolina/protos-project-docs@908a7b60f8e6e7ba03531dbc9b8405420036e07a`.
Owner approval: **2026-11-26**, explicit active decision interaction: “apruebo B'”.
The antecedent is the presented D264 B′ deterministic global round-robin Case sharding contract, including option validation, selector composition, empty partitions, authority preservation and CI reconciliation boundaries. This is not approval of a new result schema or distributed resource manager.

Nature: implementation-independent bundled-tool policy. No change to normative language or Standard Library semantics; no concrete runtime architecture is selected.

## Decision and exact contract

```text
protos test --shard-count N --shard-run K [--jobs J]
protos test --list-cases --shard-count N --shard-run K
```

Both shard options are optional as a pair; when absent, existing unsharded behavior is unchanged. When present, each occurs exactly once, with a separate decimal-digit positive ordinary Integer value, without overflow or truncation. Require 1 <= K <= N. Missing values, one option without the other, duplicate occurrences, zero, negative, non-integer or out-of-domain values, and out-of-range indices fail before execution. Reject unsupported attached forms of these new options rather than silently running an unsharded suite. Preserve established literal consumption of selected option values and the compatibility policy for existing options/unknown arguments; no aliases, implicit environment fallback or auto-detection.

The universe P is the ordered authoritative logical CasePlan of the invocation **after** its existing source and exact CaseRef selectors. Resolve authoritative suite/corpus bindings and plans, apply existing file/directory union/dedup source scopes, discover all in-scope sources, and validate the entire explicit exact CaseRef set and duplicate discovered identities. Then retain case at zero-based global position i iff i mod N = K - 1. Global means never reset at a source, suite, corpus or ExecutionRequirement boundary. The implementation may fuse traversal but all validation must complete before listing, progress or scheduling; a case requested but assigned to another shard is valid, whereas an unknown/stale/malformed/duplicate/out-of-scope ref remains an error. Command argument order and physical completion order do not define plan order.

For a common ordered P, the remainder function assigns each position to one and only one partition. Hence pairwise intersections are empty, union equals P and sizes differ by at most one. Retain original order within each partition. A valid empty partition succeeds with zero cases and no test execution. It does not mask invalid source selectors, invalid catalogs, duplicate identities or discovery failures. N=K=1 is equivalent to the unsharded selected run.

Listing retains the exact D185 JSON schema `protos.test.cases/v1`, with the ordered retained `ref` and `display` entries (empty array for an empty shard); it never invokes Test bodies. Execution uses the same retained plan, preserves original CaseRef, source association, selector, declaration position, complete source discovery signature, CaseAuthority, ExecutionRequirement, expectations and existing resource semantics. Every retained Case runs through the existing fresh semantic Process and case-private output boundary. No new logical identity, discovery engine, scheduler, runtime/Java infrastructure or authority is introduced. `--jobs` remains D069 invocation-local logical admission capacity, absent means 1, and changing it never changes shard membership.

Keep existing CLI failure lanes: completed all-pass (including valid empty shard) 0; completed guest failure 1; selected-option/configuration/selection errors use the existing Test Tool Error diagnostic/1 lane, not a completed failure or new usage code; infrastructure-aborted 3; unexpected host/tool failure 70 where applicable. Preserve existing diagnostic distinctions and failure precedence. No new retries, fail-open behavior or outcome classifications.

## GITHUB021 invariant and delta review

The reviewed product delta from 0561cf229 to 28d3c1df changes only CHANGELOG, pom.xml, ProtosExternalChildValue.java and ProtosI166ExternalProcessCustodyTest.java (BUG041). No Test Tool, specification, or CI implementation change; the earlier source investigation remains applicable.

| Prior durable authority (in this tooling directory) | Preservation check |
|---|---|
| D069 | Local positive jobs capacity and default 1 are unchanged; sharding is selection, not a thread/worker count. |
| D122, D125, D126, D132 | Explicit suite graph and canonical plan order; corpus/source authority and execution requirement remain distinct and unchanged. |
| D151, D185 | Source selectors do not create tests; exact refs resolve before scheduling; opaque canonical CaseRef and listing schema unchanged. Sharding is an additional post-selection projection. |
| D152, D153, D133 | One independently executable logical Case, full signature, fresh semantic Process, case-private output and per-case authority preserved. |
| D155 | Terminal classification and existing report ordering unchanged. |
| D076, D077, D097, D098, D099, D101 | Inert requirements, environment-owned catalog, atomic reservations, provider/profile and placement/run accounting, explicit catalog acquisition and custody unchanged. |

The current suite-native path rejects nonempty resource requirements before discovery; D264 does not lift that restriction. Distinct invocations do not acquire shared reservation authority merely because they use the same catalog or shard count. For future resourceful execution, placement resources must genuinely be independent or coordinated by their existing authority; run-scope reservation within one invocation is not cross-run global exclusion. Catalog parsing/acquisition remains the existing policy, including for listing and valid empty shards.

D264 owner direction of 2026-11-25 (Protos-first, no new production Java, no duplicate discovery/execution engine, independence from initial GITHUB026 recovery) is preserved. No previously approved invariant is reopened. The new observable delta is only explicit opt-in Test Tool shard selection and its specified validation, listing and empty-shard behavior.

```text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
LANGUAGE_SEMANTIC_CHANGE=NO
PRODUCTION_JAVA_CHANGE=NO
```

## Comparative research and evidence

The implementation-independent research compared multiple families; source links identify primary evidence, not authorities over Protos.

- GraalPy [runner.py](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python.test/src/runner.py) (reviewed collect/collect_module, main_run, partition_tests_into_processes; blob edf03d096ea91aef223d547232fcebb9b02b045c), [mx_graalpython.py](https://github.com/oracle/graalpython/blob/master/mx.graalpython/mx_graalpython.py) (blob 97a0902caf88b5b7a52768088b8eba0537fbb67f): TAGGED_UNITTEST_PARTIAL converts one-based selected/total to zero-based slice; files are round-robin unless explicitly split per test, in which case collected tests are sliced. Local -n scheduling is separately timing-weighted where timing data exists. Protos already isolates individual logical Cases; no per-file split mode is needed.
- V8 [ShardProc](https://github.com/v8/v8/blob/main/tools/testrunner/testproc/shard.py) (blob e31a2ea2fbdb8c5337076e0c533a4319675560e8) uses radix hash modulo capacity over procid; [base_runner.py](https://chromium.googlesource.com/v8/v8/+/a7826ddf8f970818c12975194557f7d5a8005392/tools/testrunner/base_runner.py) establishes one-based shard-run, unlike zero-based GTest indices. Adopt the spelling, not the hash or environment fallback.
- [Cargo-nextest partitioning](https://nexte.st/docs/ci-features/partitioning/) provides sliced round-robin over the selected global tests and hash-based partitioning. Supports the separation of count balance and stable assignment.
- [GoogleTest](https://google.github.io/googletest/advanced.html) uses total/index environment variables; cooperating invocations require the same universe. [Bazel test encyclopedia](https://bazel.build/versions/8.6.0/reference/test-encyclopedia) separates external orchestration from runner partition policy and requires explicit runner support.
- [Playwright](https://playwright.dev/docs/test-sharding) distinguishes file and test granularity depending on isolation/parallelism configuration; [GraalJS TestSuite](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js.test.external/src/com/oracle/truffle/js/test/external/suite/TestSuite.java) (blob b6cbd433029120c7a2d3649e3897241f82bdde9f) uses external suite selection, and [TruffleRuby jt](https://github.com/truffleruby/truffleruby/blob/master/tool/jt.rb) (blob 9876978c6ed295cd0af4c16aff9b80161c4d0e32) delegates test routes; neither establishes a direct equivalent sharding guarantee in the inspected code. Espresso, Sulong, SimpleLanguage and Pkl are not required precedents for a guest-level public Test Tool filter; no host/runtime decision is being made.

Alternatives: A external D185 list/CaseRef partition; B global round-robin (B′ exact contract selected); C portable stable CaseRef hash; D contiguous equal-count blocks; E shared historical timing-weighted assignment; F dynamic distributed coordinator.

GITHUB010 comparative scores (1–5; each cell score and rationale; H high confidence in mathematical/design properties, M medium in inferred cost/feasibility, L low in unmeasured performance). Scores are review aids, not authority.

| Dimension | A | B | C | D | E | F |
|---|---|---|---|---|---|---|
| Correctness | 4/M caller dependent | 5/H exact partition | 5/H exact partition | 5/H exact partition | 4/M common historical plan | 3/M leases/failures |
| Protos alignment | 3/H duplicate policy | 5/H ordinary filter | 4/H hash contract | 4/H blocks group similar cases | 3/M history | 2/H coordinator |
| Present proportionality | 3/M scripts | 5/H minimal | 4/H extra hash | 5/H simple | 2/M unneeded history | 1/H new institution |
| Incremental growth | 3/M multiple callers | 5/H reusable filter | 5/H stable assignment | 4/H extensible | 3/M historical data | 3/M protocol dependency |
| Future resilience | 4/M flexible | 5/H leaves options open | 5/H stable identity | 4/M flexible | 4/M adapts costs | 4/M adaptive workers |
| Scalability | 3/M argv/coordination | 4/M replicated discovery | 4/M uneven counts | 3/L clustered costs | 5/L potential time balance | 5/L potential utilization |
| Simplicity | 3/H multiple owners | 5/H remainder | 4/H codec/hash | 5/H arithmetic | 2/H extra policy/state | 1/H protocols |
| Portability | 4/M caller tools | 5/H ordinary Integer | 5/H portable hash | 5/H arithmetic | 4/M timing portability | 3/M transport |
| Resource cost | 3/M list transport | 4/M repeated discovery | 4/M discovery/hash | 4/M repeated discovery | 3/L history | 2/M communication |
| Failure/operability | 3/M divergence | 5/H direct reproduction | 4/M hash stability | 4/M direct reproduction | 3/M plan validity | 2/M recovery |
| Reversibility | 5/H easy replace | 5/H additive policy | 4/H hash compatibility | 4/H assignment compatibility | 3/M persisted history | 3/M coordination coupling |
| Evidence maturity | 5/H D185 exists | 4/M external precedent | 4/M V8 precedent | 4/M known algorithm | 3/M no Protos measurements | 2/L unproven |
| Total | 43 | 57 | 52 | 51 | 39 | 31 |

A remains appropriate for specialized harnesses but duplicating a general policy across CI callers risks divergent partition semantics and argument-length limits. C has a genuine advantage when cases must keep their assignments across revisions; count imbalance and an extra stable hash contract are unnecessary today. D groups adjacent cases, potentially clustering expensive families. E and F have non-compensating overengineering red flags: no measured current necessity for persisted timing, remote queues, leases, retries or coordinator state. A has a tooling ownership/underengineering risk but remains an escape path, not prohibited. B′ is the smallest sufficient policy and preserves the existing D185 external selection capability.

## Falsification, scaling and deferral

- An insertion/reorder may move later cases across shards; the identity remains unchanged. Exact reproduction requires the same revision, selected ordered plan and N/K. Stable assignment across revisions may justify explicit hash policy later, not a silent switch.
- Equal counts do not imply equal times. A single very slow test dominates wall time. Timing-based balancing can later consume the same CasePlan via an explicitly approved policy; no runtime rewrite or new Case identity is needed.
- With M cases and N runners, each discovers the selected plan; filtering is O(M) per invocation with constant extra partition-counter state, retained projection size about M/N. Total discovery cost grows with N. Unsharded execution must not incur unnecessary new Case-scale work. If very large plans make discovery dominant, investigate revision-bound transportable inert plans and signature verification separately, not premature caching/coordinator machinery.
- Different plans on different runners invalidate the partition theorem. Compare ordered plan evidence and revision/configuration, never just counts or display labels. A lost, cancelled or failed shard leaves global validation incomplete; do not silently reassign, retry or accept it. Guarantee assignment, not exactly-once physical execution across arbitrary manual retries.
- Independent jobs preserve Process isolation, private streams, ordinary cancellation and host/resource custody. Native/JVM and future host/runtime portability follows from ordinary Tool policy, not a Java implementation. No new persisted state or remote protocol.
- Deferred: stable hash policy, timing weights, remote placement, resourceful logical Case provisioning and global resource reservations, dynamic scheduling, new structured result contract. These can be added only when justified without invalidating current identity/authority. Strongest argument against B′ is lack of duration balance; actual measurement, not speculative infrastructure, should determine the next step.

## Implementation and downstream validation boundaries

Keep all policy in bundled Protos. Reviewed sources: Options.protos (CommandLine and literal projection), CaseSelection.protos (retain, requireComplete, projection, listing), CaseRef.protos, LogicalCasePlan.protos, Main.protos (source validation, discovery, metadata/progress alignment), LogicalCaseRunner.protos, Progress.protos, docs/guide/tools/test-tool.md, Makefile, .github/workflows/tests.yml, tools/ci_validation_aggregate.py. Reuse standard facilities and keep the existing full declaration signature when filtering. No changes to normative spec files are required.

A separate I-family implementation work item consumes this decision. Required regressions: omitted options and 1/1 compatibility; invalid/missing/duplicate/overflow/attached forms, valid empty shards, order, global partition across files/suites, exact union/disjointness, selector composition and validation even for unassigned cases, same listed/executed plan, no body on listing, complete signature rematerialization, authority/resources preserved, fresh Process and unchanged jobs/report/exit lanes. Include focused and integrated human-executed validation with exact evidence. No tests for D264 have yet been executed; the owner reported clean git diff check and all local tests passing, not a D264 implementation revision or new regressions.

Makefile/CI is a downstream consumer: pass explicit count/index (proposed Make variables PROTOS_TEST_SHARD_COUNT and PROTOS_TEST_SHARD_INDEX), maintain existing unsharded default and mandatory coverage. Matrix only supplies index/count; no second guest partition algorithm. Reproduce a shard with the same selectors/revision and `--jobs 1`. Preserve unrelated Java, compilerability, native and external foreign-provider lanes; test-protos-foreign is separate. GITHUB026 initial CI recovery (#1176) remains independent and must not be blocked by D264.

Aggregation must fail closed on missing, duplicate, mismatched revision/configuration or failed shards; compare union of authoritative per-shard listed CaseRefs to complete listing and check terminal outcome/counts. Current output does not independently enumerate every successful Case identity. Listings plus counts establish planned coverage and completion quantity, not an independent per-Case observed identity proof. If strict CI acceptance requires that proof, expose the missing reporting contract as a decision boundary before implementing any new public reporting schema. Do not claim stronger evidence than available. No CI success, measured speedup or implemented feature is claimed by this record.
