# TEST010 — Final logical-Case reachability inventory and closure evidence

**Date:** 2026-10-30. **Owner:** [TEST010 #991](https://github.com/guillermomolina/protos/issues/991). **Role:** additional immutable, non-normative acceptance evidence for the [original TEST010 publication record](TEST010_0_3_425_REPOSITORY_REGISTRATION_EVIDENCE_2026_10_30.md). This supplement supersedes the earlier record's *pending numerical inventory* qualification, without altering its historical observation.

## Exact source and method

```text
WORK_ITEM=TEST010
ISSUE=guillermomolina/protos#991
PROTOS_REVISION=e5a55588362340aba3faa39e007b0228db77f543
IMPLEMENTATION_VERSION=0.3.425-SNAPSHOT
VALIDATION_PROVENANCE=HUMAN_EXECUTOR_REPORTED
CORPUS_REGISTERED_SOURCE_ASSOCIATIONS=381
PHYSICAL_PROTOS_SOURCES=596
REGISTERED_PHYSICAL_SOURCES=381
SUITE_NATIVE_PRODUCTION_SOURCES=381
SOURCES_OUTSIDE_REGISTRATION=215
OFFICIAL_CORPORA=28
DEFAULT_LOGICAL_CASES=3442
UNIQUE_CASE_REFS=3442
DUPLICATE_CASE_REFS=0
INTEGRITY=PASS
TEST_BODIES_EXECUTED=NO
REPOSITORY_MODIFIED=NO
```

Human executor performed the audit on **the exact product commit** [`e5a55588362340aba3faa39e007b0228db77f543`](https://github.com/guillermomolina/protos/commit/e5a55588362340aba3faa39e007b0228db77f543), not a floating `main`, with no source edits, recompilation, or Test-body execution. `bin/protos test --list-cases` with explicit `--file` / `--directory` selections builds each corpus's observational `protos.test.cases/v1` JSON CasePlan, including D133 source associations. Python read-only scripts decoded each `v1.` hexadecimal JSON identity `[sourceAssociation, selector]`, collected all identities, and independently reconstructed manifest/explicit-plan registrations. The scripts compared 28 `RepositorySuite.root` corpus leaves against the configured plans, then compared every observed source association against the 381 registered associations. Physical `protos/tests/**/*.protos` inventory and suite-native declarations were also compared with registration paths.

The enumeration was divided into 13 named non-library partitions (12 selected corpus runs plus one verified empty `content-identity` manifest), alongside the already saved complete library selection. Per-partition case lists were filtered by the exact expected corpus identity where directory selectors physically overlapped. The 28 corpus identities stayed separate. No dynamic filesystem auto-enrollment or source/Case identity conflation was introduced.

## Reconciled case totals

| Group | Logical Cases | Source-plan entries |
| --- | ---: | ---: |
| Standard Library (15 corpora) | **984** | **86** |
| Conformance, main | 2193 | 235 |
| Process snapshot | 15 | 3 |
| Actor | 11 | 4 |
| Group | 10 | 3 |
| Package TOML | 44 | 5 |
| Package version | 78 | 10 |
| Package lock | 71 | 10 |
| Package resolution input | 12 | 3 |
| Package content identity (empty manifest) | 0 | 0 |
| Package resolution input lock | 2 | 2 |
| Package resolution root | 8 | 8 |
| Package execution plan | 10 | 9 |
| Package project projection | 4 | 3 |
| **Total** | **3442** | **381** |

All 984 standard-library Logical Cases, including the 16 Cases declared in the one registered `library/logging/colored-formatter.protos` source, were listed from their authorized plans. The other groups contribute 2458 cases. The source-plan total of 381 is independently corroborated by manifest rows and `RepositoryCorpusPlans.protos`; these are **source associations**, not Case counts.

## Final integrity output

The maintainer reported the following final read-only reconciliation outputs:

| Signal | Result |
| --- | ---: |
| `PHYSICAL_PROTOS_SOURCES` | 596 |
| `REGISTERED_SOURCE_ASSOCIATIONS` | 381 |
| `REGISTERED_PHYSICAL_SOURCES` | 381 |
| `SUITE_NATIVE_PRODUCTION_SOURCES` | 381 |
| `SOURCES_OUTSIDE_REGISTRATION` | 215 |
| `OFFICIAL_CORPORA` | 28 |
| `LISTING_PARTITIONS` | 13 |
| `LOGICAL_CASES` | 3442 |
| `UNIQUE_CASE_REFS` | 3442 |
| `DUPLICATE_CASE_REFS` | 0 |
| `INVALID_CASES` | 0 |
| `REGISTRATION_DUPLICATES` | 0 |
| `MISSING_SOURCES` | 0 |
| `UNEXPECTED_SOURCES` | 0 |
| `DANGLING_PHYSICAL` | 0 |
| `UNREGISTERED_NATIVE` | 0 |
| `MISSING_CORPORA` | 0 |
| `EXTRA_CORPORA` | 0 |
| `TEST010_FINAL_RECONCILIATION` | **PASS** |

The remaining 215 `.protos` files are *outside* the default suite's source registrations, not 215 missing tests. Their deliberate JUnit fixture, tooling, auxiliary or project-data ownership is guarded by `ProtosTestToolRepositoryCompletenessTest`, whose final integrated validation passed. No unregistered production suite-native sources were found.

The observed 3442 unique CaseRefs are exactly the identities reachable through the individual authoritative corpus plans of `RepositorySuite.root`; no reference occurs in more than one corpus enumeration. This is an exhaustive partitioned plan-level reachability proof. It is **not** a successful unfiltered `--list-cases` output or independent assertion that all 3442 Test bodies were run within this audit. Existing `make test` PASS and `make check` PASS were separately maintainer-reported before version/changelog finalization.

## Separately observed full-listing limitation

An earlier **unfiltered** `bin/protos test --list-cases` invocation returned code 1, `Test tool error: Object {}` at `protos/tools/test/Main.protos:562` while producing the JSON listing. Its host displayed Java **27.0.1.0.3** with a Truffle **25.4.4.1.1** compatibility warning and fallback interpreter. No evidence establishes whether Java incompatibility, listing size or another cause triggered the failure. Subsequent *partitioned* listings in a separate environment reporting Java **25.0.4.1.1** worked: the complete library partition listed 984 unique cases, and all 12 non-empty non-library partitions completed. The **unfiltered** invocation was **not reproduced under that Java 25 runtime** in the reported evidence; avoid claiming a general Protos defect or a confirmed unsupported-runtime bug.

A separate `--directory protos/tests/package-tool` selector also failed at `Main.protos:253`, because this broad path did not match an explicit corpus root. Each actual Package Tool corpus directory could be listed separately. These two observations are retained here for further diagnostics if they recur; neither is hidden or represented as a clean global listing.

## TEST010 acceptance and closure boundary

With the published fail-closed guard, human-reported 21/21 focused JUnit PASS, 16/16 `colored-formatter` body execution PASS, `make test` PASS, `make check` PASS, `git diff --check` clean, and the source/Logical-Case reconciliation above, the two previously pending inventory and default-suite Case-reachability acceptance criteria are now supported. No tests were run after the product `pom.xml` / `CHANGELOG.md` update.

```text
CORPUS_REGISTRATION_GUARD=PASS
PRODUCTION_TEST_SOURCES=381
PRODUCTION_LOGICAL_CASES=3442
DEFAULT_SUITE_REACHABLE_CASES=3442
INTENTIONALLY_EXCLUDED_SOURCES=215
MISSING_REGISTRATIONS=0
DANGLING_SUITE_REFERENCES=0
DUPLICATE_EXECUTION_IDENTITIES=0
AUTHORITY_MISMATCHES=0
COMPLETENESS_GUARD=PASS
```

The exact product revision, this durable Git publication, and the final [TEST010 #991](https://github.com/guillermomolina/protos/issues/991) closure comment together provide the stable cross-referenced record. This document does not modify Protos source, normative semantics, or live Issue/Project status.
