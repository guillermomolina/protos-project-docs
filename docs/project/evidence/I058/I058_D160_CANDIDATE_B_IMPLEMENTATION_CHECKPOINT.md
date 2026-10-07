# I058 — D160 Candidate B implementation checkpoint

Date: 2026-10-03

This snapshot records the published implementation state for I058 / `guillermomolina/protos#661`. It is durable non-normative project evidence; observable Protos semantics remain owned by `guillermomolina/protos:spec/**`.

## Stable identities

```text
PROTOS_REVISION=1ad6c5b5566a7e2c639a27ae95a8a545b03fd402
IMPLEMENTATION_VERSION=0.3.178-SNAPSHOT
SPECIFICATION_REVISION=0.1.440
D160_RATIFICATION_REVISION=ef02556fc3f1f0b2d8fc10de7312c78b76ab389e
GITHUB_ISSUE=guillermomolina/protos#661
PUSH_CI_RUN=2111
```

## Published result

The published revision implements D160 Candidate B:

- `std:collections/Array` owns `parallelMap`, `parallelFilter`, `parallelFindIndex`, `parallelReduce`, and `parallelSort` as Protos-source module functions;
- the five former Core `Array.parallel*` selectors and their compatibility aliases are absent;
- the operation-specific privileged Array paths are removed from `ProtosParallelRuntime` while the retained `Closure.parallel` / Future / isolated-P substrate remains;
- the Standard Library implementation composes the retained public substrate rather than introducing a privileged batching/chunking helper, public worker/executor/task object, `Array.parallelEach`, or writable collection partitions;
- current specification, guide/example material, and the canonical `parallel-array-map` benchmark source are migrated to the Standard Library placement.

The publication also records deterministic source-order/failure handling, Future-returning structured ownership/cancellation, adjacent-pair `parallelReduce`, and stable sequential-equivalent `parallelSort` behavior in the new library-owned contract.

## Validation evidence available at this checkpoint

The maintainer reported after publishing the exact revision above:

```text
PUSH_TO_MAIN=PASS
ALL_TESTS=PASS
```

At the coordination observation immediately after publication, GitHub Actions CI run `#2111` for the exact revision was still `in_progress`; therefore remote-CI closure evidence was not yet established by this snapshot.

The canonical `parallel-array-map` benchmark was migrated in the product revision, but no benchmark execution result or methodologically comparable before/after measurement was supplied with the current closure handoff. Because #661 explicitly requires performance evidence, this checkpoint does not claim that gate as satisfied.

## Closure-gate state

```text
I058_IMPLEMENTATION_PUBLISHED=YES
D160_CANDIDATE_B_REALIZED=YES
CORE_FIVE_SELECTORS_REMOVED=YES
STDLIB_FIVE_ALGORITHMS_AVAILABLE=YES
P_FUTURE_SUBSTRATE_PRESERVED=YES
PRIVILEGED_HELPER_ADDED=NO
MAINTAINER_REPORTED_TESTS=PASS
REMOTE_CI=IN_PROGRESS
BENCHMARK_SOURCE_MIGRATED=YES
BENCHMARK_EXECUTION_EVIDENCE=NOT_RECORDED
ISSUE_CLOSURE_READY=NO
```

This record intentionally separates the completed product implementation from the still-pending closure evidence. Later closure should cite the exact successful CI identity and the required benchmark result rather than rewriting this observation.
