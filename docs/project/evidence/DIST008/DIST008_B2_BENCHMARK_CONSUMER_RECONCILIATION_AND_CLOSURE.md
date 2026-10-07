# DIST008-B2 — benchmark consumer reconciliation and DIST008 closure

Status: PASS
Date: 2026-09-29
Owning Issue: `guillermomolina/protos#739`
Authoritative Protos B1 revision:
`6411d39bf33014c958ba4dad60a6e3fe44760ebf`
Baseline benchmark revision:
`a2a8eafe74a45cce987a0918d1023be061ed17f3`
Published benchmark revision:
`e8a1f1735e0c2751de99459aeeb689a8d46e5f0d`

## Implementation boundary

DIST008-B2 reconciled the current benchmark consumer of the canonical Protos
toolchain contract after DIST008-B1 published `protos-toolchain-v2`.

The published B2 delta is exactly these seven paths:

```text
config/dist006d-baseline.json
docker/protos-dist006d/Dockerfile
runner/dist006d_baseline.py
runner/perf009a.py
runner/toolchain.py
tests/test_dist006d_baseline.py
tests/test_toolchain.py
```

No historical result directory, retained timing evidence, reference evidence,
PERF010-A path, language implementation, or Protos specification path was
changed.

## Consumed Maven contract

The live benchmark consumer now recognizes the B1 contract:

```json
{
  "schema": "protos-toolchain-v2",
  "maven": {
    "minimum_version": "3.9.9",
    "supported_major": 3
  }
}
```

The current supported Maven line is therefore stable Maven 3.x >= 3.9.9.
Exact Maven patch identity is not part of current Protos runtime identity.

The GraalVM/JDK coordinates remain exact:

```text
GRAALVM_RELEASE=25.4.4.1.1
JDK_VERSION=25.0.4.1.1
CONTAINER_CHANNEL=25i4
CONTAINER_IMAGE=ghcr.io/graalvm/graalvm-community:25i4-25.0.4.1.1-ol10@sha256:a7b4810d7c755e9627feaa1459eb5a93338643b16d745d4f3fc86db71e5da7f5
```

## Historical benchmark boundary

`runner/toolchain.py` now separates the current v2 reader from an explicit
historical v1 reader.

`runner/perf009a.py` uses the historical reader for its pinned historical
toolchain evidence. This preserves replay of the old exact-Maven contract
without allowing v1 to remain a valid current contract.

Historical benchmark Dockerfiles and retained measurements were not bulk
rewritten.

## Maven provisioning and Java authority

The live DIST006-D Docker build now provisions:

```text
Oracle Linux 10 Maven
maven-unbound
--enablerepo=ol10_codeready_builder
```

The CodeReady Builder repository is scoped to the Maven transaction.
`JAVA_HOME` remains the canonical GraalVM Java authority.

The build rejects a redundant OpenJDK RPM and validates the actual Maven
runtime against the compatibility floor instead of an exact patch coordinate.

The Maven identity parser also tolerates distribution preamble/ANSI output
while still requiring the extracted Maven coordinate itself to be a stable
`x.y.z` release in supported major 3 at or above 3.9.9.

## Validation evidence

### Focal validation

Owner-executed syntax and focal validation passed.

The initial focal set completed:

```text
33 tests
OK
DIST008_B2_FOCAL=PASS
```

After the Maven runtime-identity parser correction, the focused DIST006-D set
completed:

```text
17 tests
OK
DIST006D_D1_CONFIG=PASS
DIST006D_D1_TOOLCHAIN_CONTRACT=PASS
DIST006D_D1_WORKLOAD_SET=PASS
DIST006D_D1_DOCKER_STRUCTURE=PASS
DIST006D_D2_BUILD_TOOLCHAIN_SELECTION=PASS
DIST006D_D1_EXACT_SHA_POLICY=PASS
DIST006D_D1_HISTORICAL_UPSTREAM003_MUTATION=NO
DIST006D_D1_SMOKE_RETAINED_PERFORMANCE_EVIDENCE=NO
```

`git diff --check` also passed.

### Full benchmark-suite boundary

`make test` and `make validate` each reached the existing benchmark suite and
reported one failure out of 237 tests:

```text
test_perf010a.Perf010aContractTest.test_smoke_and_reference_are_top_level_commands
```

The failure is outside B2. The exact B2 baseline
`a2a8eafe74a45cce987a0918d1023be061ed17f3` already contains both:

- the stale test expectation ending at `source-identity-smoke`; and
- the additional live `stable-identity-phase1-*` and `phase2-*` command
  choices in `runner/perf010a.py`.

B2 changes none of those PERF010-A paths. This is retained as a pre-existing
repository-suite limitation rather than misreported as a B2 regression or
silently repaired inside DIST008.

### Non-retained DIST006-D smoke

The final smoke ran against exact Protos B1 revision:

```text
6411d39bf33014c958ba4dad60a6e3fe44760ebf
```

Result:

```text
DIST006D_SMOKE_CORRECTNESS=PASS
DIST006D_SMOKE=PASS
RETAINED_PERFORMANCE_EVIDENCE=NO
TIMING_EVIDENCE=NO
REFERENCE_EVIDENCE=NO
```

The smoke identity proved:

- GraalVM CE 25.4.4.1.1;
- Java 25.0.4.1.1 from GraalVM Community;
- `protos-toolchain-v2`;
- Maven minimum 3.9.9 / supported major 3;
- exact Protos B1 revision;
- all four DIST006-D correctness workloads observed their expected value;
- no retained performance, timing or reference evidence was produced.

## Publication audit

Immediately before B2 publication:

```text
LOCAL_HEAD=a2a8eafe74a45cce987a0918d1023be061ed17f3
REMOTE_HEAD=a2a8eafe74a45cce987a0918d1023be061ed17f3
DIST008_B2_FINAL_ADMISSION=PASS
ANSI_HELPER_COUNT=1
MAVEN_EXTRACTOR_COUNT=1
MAVEN_VALIDATOR_COUNT=1
```

Published commit:

```text
e8a1f1735e0c2751de99459aeeb689a8d46e5f0d
DIST008-B2: reconcile benchmark Maven compatibility
```

The push advanced benchmark `main` from `a2a8eaf` to `e8a1f17`.

## DIST008 closure

The complete DIST008 chain is:

```text
DIST008-A   PASS — requirement/provisioning investigation
DIST008-B1  PASS — canonical Protos toolchain/provisioning implementation
DIST008-B2  PASS — live benchmark consumer reconciliation
```

Top-level result packet:

```text
DIST008_RESULT=PASS
PROTOS_REVISION=6411d39bf33014c958ba4dad60a6e3fe44760ebf
BENCHMARK_REVISION=e8a1f1735e0c2751de99459aeeb689a8d46e5f0d

ACTUAL_MAVEN_REQUIREMENT=ESTABLISHED
EXACT_MAVEN_REQUIRED=NO
MAVEN_MINIMUM_VERSION=3.9.9
MAVEN_SUPPORTED_MAJOR=3
REDUNDANT_JDK_PROVISIONING=ELIMINATED
PROVISIONING_MODEL=OL10_MAVEN_PLUS_MAVEN_UNBOUND
PROVISIONING_COST_EVIDENCE=RETAINED
TOOLCHAIN_CONTRACT=COHERENT
DEVCONTAINER_MAVEN=PASS
NATIVE_BUILDER_MAVEN=PASS
BUILD_AND_TEST_VALIDATION=PASS_WITH_PREEXISTING_BENCHMARK_SUITE_LIMITATION
DISTRIBUTION_NATIVE_IMPACT=VALIDATED
GRAALVM_JDK_TRUFFLE_COORDINATES_UNCHANGED=YES

PREEXISTING_BENCHMARK_SUITE_LIMITATION=PERF010A_HELP_ASSERTION
DIST008_REGRESSION=NO
BLOCKER=NONE
```

No new Dxxx/PLATxxx decision was required.
