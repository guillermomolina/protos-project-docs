# PERF018 — Shared Core Prelude implementation and closure evidence

Date: 2026-09-29

## Identity

```text
WORK_ITEM=PERF018/#731
PRODUCT_REPOSITORY=guillermomolina/protos

BEFORE_REVISION=1637fe514ee9327241e49964031d981423b16198
BEFORE_VERSION=0.3.116-SNAPSHOT

PROTOS_REVISION=2d33ef9a6167067a0af04840af2bd62ff9bee563
PROTOS_VERSION=0.3.117-SNAPSHOT
COMMIT=PERF018: reuse Core Prelude across logical Case attempts

HOST_LOGICAL_CPUS=16
HOST_CPU_AFFINITY=0-15
```

This record retains the implementation, measurement, validation, and closure
evidence for PERF018 / guillermomolina/protos#731.

PERF018 is independent of the already-closed PERF017/PERF019 Test Tool scaling
recovery. PERF019 explicitly left the Core-bootstrap optimization out of scope.
The purpose here is narrower: remove the redundant per-logical-Case Core
bootstrap and its temporary Polyglot Engine/Context while preserving fresh Case
Process isolation and existing Test Tool behavior.

## Published implementation

The published change replaces the per-Case:

```text
logical Case
  -> ProtosCoreBootstrap
       -> temporary Polyglot Context
       -> implicit temporary Engine
       -> rebuild Core/Prelude
  -> fresh semantic Process
  -> RuntimeHost Process Context
```

with bridge-local lazy bootstrap reuse:

```text
logical Case bridge
  -> one lazily bootstrapped shared Core Prelude
       -> ProtosContextBoundModuleResolver
  -> each logical Case
       -> fresh ProtosDirectFileModuleResolver
       -> fresh semantic Process
       -> fresh RuntimeHost Process Context
       -> bind that Process's resolver to its language Context
       -> freshly rematerialize suite declaration
       -> validate discovery signature
       -> resolve exact selected Test
       -> execute selected body
```

The implementation surface is:

```text
src/main/java/com/guillermomolina/protos/execution/ProtosContextBoundModuleResolver.java
src/main/java/com/guillermomolina/protos/execution/ProtosLanguageContext.java
src/main/java/com/guillermomolina/protos/execution/ProtosPolyglotProcessContext.java
src/main/java/com/guillermomolina/protos/execution/ProtosTestLogicalCaseAttemptBridge.java
src/test/java/com/guillermomolina/protos/execution/ProtosTestLogicalCaseAttemptBridgeTest.java
pom.xml
CHANGELOG.md
```

`ProtosTestLogicalCaseAttemptBridge` now owns one lazily initialized
`ProtosPrelude`. `ProtosContextBoundModuleResolver` selects the
Process-private module resolver from the currently entered
`ProtosLanguageContext`, falling back to the bootstrap resolver when no Process
resolver is bound. Each hosted Process binds its resolver before guest module
execution.

The regression test executes two Cases through one bridge using same-named local
module paths with different contents. It requires the shared Prelude identity to
be the same while proving that resolver/module state does not leak from one Case
to the other.

## Bootstrap-count effect

No permanent diagnostic instrumentation was added. The count follows directly
from the construction boundary.

Before PERF018:

```text
CORE_BOOTSTRAP_FREQUENCY=ONE_PER_LOGICAL_CASE_ATTEMPT
TEMPORARY_BOOTSTRAP_ENGINE_CONTEXT_FREQUENCY=ONE_PER_LOGICAL_CASE_ATTEMPT
```

After PERF018:

```text
CORE_BOOTSTRAP_FREQUENCY=ONE_LAZY_BOOTSTRAP_PER_LOGICAL_CASE_BRIDGE
TEMPORARY_BOOTSTRAP_ENGINE_CONTEXT_FREQUENCY=ONE_LAZY_BOOTSTRAP_PER_BRIDGE
```

The four Test Tool facility flavours — ordinary, Actor, Group, and Package —
hold independent bridge instances, so a complete run can perform up to four
shared Prelude bootstraps. The important PERF018 result is that bootstrap and
temporary Engine/Context construction are no longer proportional to logical
Case count.

```text
PER_CASE_REDUNDANT_CORE_BOOTSTRAP=REMOVED
PER_CASE_TEMPORARY_ENGINE=REMOVED
GLOBAL_BOOTSTRAP_COUNT_ZERO=NO
BRIDGE_LOCAL_LAZY_BOOTSTRAP=YES
```

## Before/after measurements

All retained full Protos Test Tool runs completed:

```text
LOGICAL_CASES=1263
PASSED=1263
FAILED=0
```

Each timing cell below is one sample.

| Workload | Metric | Before `1637fe51` | After candidate | Change |
| --- | --- | ---: | ---: | ---: |
| default jobs | wall | 166.83 s | 170.81 s | +2.4% |
| default jobs | user | 783.99 s | 805.13 s | +2.7% |
| default jobs | user/real | 4.699x | 4.714x | approximately flat |
| `--jobs 16` | wall | 80.30 s | 73.51 s | -8.5% |
| `--jobs 16` | user | 956.99 s | 865.08 s | -9.6% |
| `--jobs 16` | system | 22.29 s | 16.49 s | -26% |
| `--jobs 16` | user/real | 11.918x | 11.768x | -1.3% |

The default-jobs movement is only about 2-3% from one sample and is not treated
as a demonstrated regression or improvement.

The `--jobs 16` sample is consistent with the intended mechanism: wall time
falls while user and system CPU time also fall, indicating less bootstrap work.
Because each side is a single sample, the 8.5% wall-time reduction is retained
as observed evidence rather than promoted to a statistically established
performance guarantee.

The primary PERF018 success criterion is structural: redundant per-Case Core
bootstrap and temporary Engine/Context construction are removed while the
required Case semantics remain intact. No minimum speedup was required.

## Preserved semantics and Test Tool invariants

The published implementation and focused regression coverage preserve:

```text
FRESH_SEMANTIC_PROCESS_PER_CASE=PASS
FRESH_PROCESS_CONTEXT_PER_CASE=PASS
SUITE_REMATERIALIZATION=PASS
DISCOVERY_SIGNATURE_VALIDATION=PASS
EXACT_SELECTED_TEST_RESOLUTION=PASS
NO_LIVE_VALUE_CROSS_PROCESS=PASS
CASE_PRIVATE_OUTPUT=PASS
JOBS_SCHEDULING_SEMANTICS_UNCHANGED=PASS
PERF019_RECOVERY_MACHINERY_PRESERVED=PASS
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
STANDARD_LIBRARY_SEMANTIC_CHANGE=NO
PUBLIC_TEST_TOOL_BEHAVIOR_CHANGE=NO
```

The specification is unaffected. Reusing the frozen/shared Core Prelude does not
reuse the semantic Process, Process Context, suite declaration, selected Test
Closure, or Process-private module resolver.

## Validation

Candidate-level focused validation completed successfully before publication,
and every retained complete Protos Test Tool run reported 1263 passed, 0 failed.

The exact published revision then passed GitHub Actions CI:

```text
CI_RUN_ID=36604272824
CI_WORKFLOW=CI
CI_EVENT=push
CI_HEAD_SHA=2d33ef9a6167067a0af04840af2bd62ff9bee563
CI_CONCLUSION=success

CI_COMMANDS:
  python3 tools/verify_toolchain.py --mode check --scope development
  make test JAVA_TEST_JOBS=4 PROTOS_TEST_JOBS=4

FULL_REQUIRED_VALIDATION=PASS
```

The new `ProtosContextBoundModuleResolver.java` carries the repository APL-1.0
notice and the modified Protos-owned source files retain theirs.

The initial pristine-HEAD `mvn package` anomaly observed during implementation
was cleared by the normal build path and was not reproduced or established as a
PERF018 defect. It is therefore retained only as an implementation-session
observation, not as a PERF018 closure blocker or causal claim.

## Closure

```text
PER_CASE_REDUNDANT_CORE_BOOTSTRAP=REMOVED
PER_CASE_TEMPORARY_ENGINE=REMOVED
FRESH_SEMANTIC_PROCESS_PER_CASE=PASS
FRESH_PROCESS_CONTEXT_PER_CASE=PASS
SUITE_REMATERIALIZATION=PASS
DISCOVERY_SIGNATURE_VALIDATION=PASS
EXACT_SELECTED_TEST_RESOLUTION=PASS
NO_LIVE_VALUE_CROSS_PROCESS=PASS
CASE_PRIVATE_OUTPUT=PASS
JOBS_SCHEDULING_SEMANTICS_UNCHANGED=PASS
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
BEFORE_AFTER_PERFORMANCE_EVIDENCE=RETAINED
FULL_REQUIRED_VALIDATION=PASS
LICENSE_COMPLIANCE=PASS

PERF018_STATUS=CLOSED_COMPLETE
NEXT_ACTION=NONE
```
