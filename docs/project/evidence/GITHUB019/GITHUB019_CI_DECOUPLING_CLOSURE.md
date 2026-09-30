# GITHUB019 — CI execution / image-publication decoupling closure evidence

Date: 2026-09-30

## Identity

```text
WORK_ITEM=GITHUB019/#534
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs

FINAL_PRODUCT_REVISION=5c07d254415ac4e0e38dd363a010e5bd1c64b8ec
FINAL_CONTRACT_REVISION=11df887f5a498bb9bccbacf2464c37c4413ae8e0
CI_IMAGE_DIGEST=sha256:94c01739a95d6bbcb197180b86ef3aaa8c429483686d29d2b12d50ae35028fc1

FINAL_CI_RUN=36664624158
FINAL_CI_RUN_NUMBER=2037
FINAL_CI_RESULT=PASS
```

This record retains the architecture, publication, cache, validation, timing,
and closure evidence for GITHUB019 / guillermomolina/protos#534.

The work was an internal CI/tooling change. It did not change Protos language,
runtime, Standard Library, or public Test Tool semantics.

## Baseline

The pre-GITHUB019 routine CI path used `devcontainers/ci@v0.3` and entered the
repository test command through Dev Container lifecycle machinery.

Retained baseline:

```text
BASELINE_CI_RUN=36616064789
BASELINE_CI_RUN_NUMBER=2028
BASELINE_RESULT=PASS
DEVCONTAINERS_ACTION_START=2026-09-29T19:00:01Z
CANONICAL_TEST_START=2026-09-29T19:00:46Z
ENVIRONMENT_PREPARATION_APPROX=45s
PROTOS_TESTS_REPORTED=228s
```

Even with cached Docker layers, routine CI still executed the Dev Container
CLI/build/up path before repository validation.

## Published architecture

GITHUB019 separated CI-image publication from ordinary CI execution.

The producer path is:

```text
.devcontainer definition / toolchain definition / publisher workflow change
  -> CI image workflow
  -> build image
  -> verify repository toolchain against the image
  -> publish immutable SHA tag
  -> publish controlled :ci alias
```

The consumer path is:

```text
ordinary push / pull_request
  -> GitHub job-level container
  -> pinned ghcr.io/guillermomolina/protos-ci digest
  -> checkout current repository HEAD
  -> verify repository toolchain
  -> restore Maven dependency cache
  -> canonical make test
```

The normal CI job has package-read authority only. Image publication alone has
package-write authority.

The image consumed by normal CI is pinned by digest rather than relying on a
moving tag.

## Producer / consumer identity split

The final contract separates the definition that should cause a new image build
from the deployment lock selecting the already-published image:

```text
TOOLCHAIN_TRIGGER=toolchain.json
CONSUMED_IMAGE_LOCK=.github/ci-image-lock.json
CONSUMED_IMAGE_LOCK_SCHEMA=protos-ci-image-lock-v1
```

The CI-image workflow rebuild triggers include:

```text
.devcontainer/Dockerfile
.devcontainer/devcontainer.json
.devcontainer/devcontainer-lock.json
toolchain.json
.github/workflows/ci-image.yml
```

The consumed-image lock is intentionally not a producer trigger. This removes
the earlier circular behavior in which updating the consumed image identity
could itself trigger another image build, while preserving image rebuilds for
real toolchain/environment changes.

Final producer validation:

```text
PROTOS_REVISION=11df887f5a498bb9bccbacf2464c37c4413ae8e0
CI_IMAGE_RUN=36663518194
CI_IMAGE_RUN_NUMBER=4
CI_IMAGE_RESULT=PASS

IMMUTABLE_TAG=ghcr.io/guillermomolina/protos-ci:sha-11df887f5a498bb9bccbacf2464c37c4413ae8e0
CONTROLLED_ALIAS=ghcr.io/guillermomolina/protos-ci:ci
PUBLISHED_DIGEST=sha256:94c01739a95d6bbcb197180b86ef3aaa8c429483686d29d2b12d50ae35028fc1
LOCKED_DIGEST_MATCH=YES
```

The final contract-only producer rebuild produced the same registry digest as
the previously validated image, confirming that the contract split itself did
not change image contents.

## Direct-container validation

The first corrected direct job-container run completed successfully at:

```text
PROTOS_REVISION=7665ac7951200b68e672e6a63fbd201a5d4f6415
CI_RUN=36659415626
CI_RUN_NUMBER=2032
CONTAINER_INITIALIZATION=PASS
TOOLCHAIN_VERIFICATION=PASS
CI_REPOSITORY_TESTS=PASS
PROTOS_TESTS_REPORTED=216s
```

This established that `jobs.<job>.container.image` was sufficient for the
repository CI contract and that ordinary CI no longer required
`devcontainers/ci`.

The repository command was also returned to the canonical policy-owned form:

```text
make test
```

Normal CI no longer duplicates fixed `JAVA_TEST_JOBS` or
`PROTOS_TEST_JOBS` overrides.

## Maven dependency cache

The first Maven cache publication used the job-container `~/.m2/repository`
path. Live validation showed that the cache action had no path to save in that
location.

The repaired contract uses:

```text
CACHE_ACTION=actions/cache@v6
CACHE_PATH=/root/.m2/repository
CACHE_KEY=Linux-maven-${{ hashFiles('**/pom.xml') }}
```

Cold validation:

```text
PROTOS_REVISION=aa6b2e7985abba1d5d889628d8234fe453caa0c0
CI_RUN=36661927022
CI_RUN_ATTEMPT=1
CI_RESULT=PASS
CACHE_RESTORE=MISS
CACHE_SAVE=PASS
CACHE_KEY=Linux-maven-6412ae5fca351a695e1f95ff0885a93cb72f3855280d2bd064d541f51b12cdb0
PROTOS_TESTS_REPORTED=239s
```

A same-revision rerun then emitted:

```text
Cache hit for: Linux-maven-6412ae5fca351a695e1f95ff0885a93cb72f3855280d2bd064d541f51b12cdb0
Cache restored successfully
```

That rerun was intentionally cancelled after cache restoration had been proven.

The final live CI run also restored the same primary cache key and completed the
entire repository suite:

```text
PROTOS_REVISION=5c07d254415ac4e0e38dd363a010e5bd1c64b8ec
CI_RUN=36664624158
CI_RUN_NUMBER=2037
CI_RESULT=PASS
CACHE_RESTORE=HIT
CI_REPOSITORY_TESTS=PASS
PROTOS_TESTS_REPORTED=137s
MAKE_TEST_WALL_APPROX=302s
```

## PERF017 JFR test isolation discovered during closure

CI run #2036 / `36663518184` failed only in two methods of
`ProtosTestToolPerf017AdmissionTest`. Both expected the six-event PERF017
operation sequence but observed only:

```text
SUBMIT_ENTER
SUBMIT_RETURN
CARRIER_RUN_BEGIN
```

The GITHUB019 contract revision that exposed the failure changed no `src/`
files. The same test had passed in earlier complete CI runs. Five owner-executed
isolated local runs then passed.

The test owns JFR recording/instrumentation state and was therefore moved from
the class-parallel JUnit lane into the repository's existing serial Java lane.
It was not removed from `make test` and no coverage was dropped.

Published isolation:

```text
PROTOS_REVISION=5c07d254415ac4e0e38dd363a010e5bd1c64b8ec
COMMIT=TEST: isolate PERF017 JFR admission test
LOCAL_ISOLATED_RUNS=5/5 PASS
```

Final CI #2037 proved the intended arrangement:

```text
PARALLEL_LANE_EXCLUDES=ProtosTestToolPerf017AdmissionTest
SERIAL_LANE_INCLUDES=ProtosTestToolPerf017AdmissionTest
SERIAL_PERF017_TESTS=4/4 PASS
FULL_CANONICAL_MAKE_TEST=PASS
```

## Wall-time attribution

The architecture does remove routine environment-preparation work, but the
measured gain is modest relative to the total CI runtime.

Observed preparation:

```text
OLD_DEVCONTAINERS_PRE_TEST_APPROX=45s
DIRECT_CONTAINER_PRE_TEST_APPROX=20-25s
DEFENSIBLE_ENVIRONMENT_STARTUP_REDUCTION_APPROX=20s
```

The final warm run was materially faster overall than the slower cold run:

```text
COLD_SLOW_MAKE_TEST_APPROX=9m21s
COLD_SLOW_PROTOS_TESTS=239s

FINAL_WARM_MAKE_TEST_APPROX=5m02s
FINAL_WARM_PROTOS_TESTS=137s
```

That full wall-time difference is not attributed to GITHUB019. GitHub-hosted
runner variation materially changed the repository-test phase across otherwise
similar runs. The retained conclusion is therefore narrower:

- direct job-container execution removes the Dev Container lifecycle/build path
  from ordinary CI;
- Maven dependency persistence is effective;
- environment startup improves by roughly twenty seconds in the observed runs;
- total CI wall time remains dominated by `make test` and hosted-runner
  variability.

Further meaningful CI performance work should target repository test workload
rather than additional container-startup micro-optimization.

## Closure

```text
ROUTINE_CI_REBUILDS_DEVCONTAINER=NO
ENVIRONMENT_IDENTITY_PINNED_BY_DIGEST=YES
CURRENT_HEAD_TESTED=YES
CANONICAL_MAKE_TEST=PASS
MAVEN_CACHE_COLD_SAVE=PASS
MAVEN_CACHE_WARM_HIT=PASS
TOOLCHAIN_TRIGGER_REBUILDS_IMAGE=YES
DIGEST_LOCK_REBUILD_CYCLE=NO
CI_IMAGE_PUBLICATION=PASS
PERF017_JFR_TEST_RETAINED_IN_SERIAL_LANE=YES
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
GITHUB019_STATUS=CLOSED_COMPLETE
```

Authoritative live coordination was closed in guillermomolina/protos#534.
This document is retained evidence only and does not replace the Issue as the
project's live work state.
