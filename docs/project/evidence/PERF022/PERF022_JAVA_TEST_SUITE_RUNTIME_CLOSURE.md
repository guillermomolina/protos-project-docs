# PERF022 — Java test-suite runtime closure evidence

Status: **CLOSED**

Owning Issue: `guillermomolina/protos#752`

Exact Protos publication:

```text
PROTOS_REVISION=6e9dfd5aa2132adedeea43f75263f4cd264a6e8a
IMPLEMENTATION_VERSION=0.3.129-SNAPSHOT
```

This record retains the runtime attribution, accepted test-structure changes,
rejected experiments, and final validation evidence for PERF022. It is
non-normative and does not define Protos language semantics.

## Trigger and baseline

PERF022 was opened after a small set of ordinary Java/Surefire classes again
dominated routine validation time despite low test counts.

The intake hotspot set on 2026-10-01 was:

| Test class | Tests | Baseline elapsed |
|---|---:|---:|
| `ProtosWorkspaceRunDriverTest` | 1 | 25.53 s |
| `ProtosFilesystemLibraryConformanceTest` | 12 | 31.50 s |
| `ProtosWorkspacePackagePreflightTest` | 2 | 12.60 s |
| `ProtosWorkspaceRunCliTest` | 3 | 42.89 s |
| `ProtosPackageExecutionPlanAdapterTest` | 2 | 11.20 s |
| `ProtosWorkspacePackageAuthorityIsolationIntegrationTest` | 2 | 13.00 s |
| `ProtosTestToolFileSelectionPublicIntegrationTest` | 4 | 56.69 s |
| `ProtosExternalPackagePlanningPreflightTest` | 2 | 35.43 s |
| `ProtosTestToolSourceLoaderTest` | 1 | 10.81 s |
| `ProtosStandaloneHostedExecutionEmbeddingTest` | 3 | 91.91 s |
| `ProtosStandaloneHostedSessionEmbeddingTest` | 5 | 183.70 s |

The complete intake list summed to **515.26 s** of Surefire-reported class
elapsed time. That sum was used only as a hotspot diagnostic; it is not a
wall-clock suite total when classes execute concurrently.

The two standalone embedding classes alone accounted for **275.61 s**.

## Attribution and retained changes

### Standalone embedding tests

The I076/I077 ordinary JUnit tests had inherited PERF021 benchmark-scale
workloads: recursive Fibonacci(30), factorial(20), and repeated same-session
invocation at a benchmark-oriented repeat count.

Those sizes were not required for the correctness properties owned by ordinary
JUnit. PERF021 remains the owner of benchmark-scale workloads.

The published PERF022 change retains proportional correctness evidence:

- recursive Fibonacci uses input 10 with result 55;
- factorial uses input 8 with result 40320;
- reusable-session repetition remains present with a proportional repeat count;
- the dedicated guest-carrier and hosted execution/session lifecycle remain
  exercised.

Measured aggregate:

```text
before=275.61 s
after=1.021 s
reduction=274.589 s
reduction_percent=99.63
```

### Public Test Tool file selection

Four full public CLI executions duplicated relative/absolute/fail-closed
selection semantics already owned by lower-level facility, plan, and
main-adoption tests.

PERF022 retains one real public relative-URI success integration while the
focused tests retain the other selection dimensions.

```text
before=56.69 s
after=8.241 s
reduction_percent=85.5
```

### Workspace run CLI

The original test paid for repeated full workspace executions to prove both a
real successful public run and outcome/error translation.

The published shape retains one real successful workspace execution and tests
FAILED, CANCELLED, and host-I/O translation through the production policy seams
used by the real path.

```text
before=42.89 s
after=5.593 s
reduction_percent=87.0
```

### Test Tool source loader

The original regression used an approximately 1.1 MiB NIO file only to cross
`Runner.readSource`'s 16-read batch boundary.

PERF022 replaces that scale with a deterministic backend that returns at most
two bytes per read. A 38-byte payload still requires 19 data reads, therefore
crossing the same 16-read batch boundary, and the 31-byte ASCII prefix splits
the following UTF-8 pi scalar across two reads.

```text
before=10.81 s
after=1.466 s
reduction_percent=86.4
```

### Filesystem library conformance

Repeated identical Core/Standard-Library bootstrap was amortized at class
scope. Every test still obtains fresh activations/resources needed by its
contract.

```text
before=31.50 s
after=5.567 s
reduction_percent=82.3
```

### Package execution-plan adapter

The immutable Package Tool Prelude is bootstrapped once for the class while
each test retains fresh hosted fixture/backend/plan state.

```text
before=11.20 s
after=7.465 s
reduction_percent=33.3
```

### Workspace package preflight

The class reuses only the `ProtosPolyglotRuntimeHost`/Engine. Each test still
creates fresh Prelude, Process, activation, and filesystem state.

```text
before=12.60 s
after=6.888 s
reduction_percent=45.3
```

### Workspace run driver duplicate integration

`ProtosWorkspaceRunDriverTest` was removed after ownership review showed no
unique retained invariant:

- the real public driver path remains exercised by
  `ProtosWorkspaceRunCliTest`;
- Tool/Application Process separation and dependency execution remain owned by
  `ProtosWorkspacePackageAuthorityIsolationIntegrationTest`;
- read-only metadata non-mutation remains owned by
  `ProtosWorkspacePackagePreflightTest`.

Its baseline cost was 25.53 s.

## Retained-target result

For the surfaces actually changed and retained by PERF022:

```text
diagnostic_before=466.830 s
diagnostic_after=36.241 s
diagnostic_reduction=430.589 s
diagnostic_reduction_percent=92.2
```

These are focal/isolated class measurements used for before/after comparison.

Per-class `Time elapsed` values observed during the final class-parallel Maven
run are intentionally not substituted for these focal measurements because
concurrent classes contend for CPU, Engine/JIT, filesystem, and other runtime
resources.

## Rejected experiments

PERF022 also retained negative evidence instead of publishing changes whose
benefit was not demonstrated.

### Authority-isolation RuntimeHost reuse

A shared RuntimeHost experiment for
`ProtosWorkspacePackageAuthorityIsolationIntegrationTest` first measured
8.084 s but a second measurement was 17.16 s against a 13.00 s baseline.

The improvement was therefore not stable enough to claim. The experiment was
reverted and no AuthorityIsolation optimization was published.

### External package planning

A RuntimeHost-sharing experiment reached 13.97 s from a 35.43 s baseline, but
subsequent fixture simplification using direct captured custody measured
15.65 s and reduced integration value.

The experimental ExternalPackagePlanning changes were ultimately reverted
before publication. The published PERF022 revision leaves
`ProtosExternalPackagePlanningPreflightTest` at its pre-PERF022 repository
shape.

## Final publication and validation

The final product commit is:

```text
6e9dfd5aa2132adedeea43f75263f4cd264a6e8a
```

Publication metadata:

```text
MAVEN_VERSION=0.3.129-SNAPSHOT
SPECIFICATION_CHANGE=NO
```

Final checks reported by the project owner:

```text
git diff --check: PASS

Java class-parallel Maven lane:
  BUILD SUCCESS
  Total time: 01:26 min

Java serial Maven lane:
  BUILD SUCCESS
  Total time: 10.036 s

Package-before-Protos Maven lane:
  BUILD SUCCESS
  Total time: 9.004 s

Protos Test Tool:
  1264 passed, 0 failed
  Protos tests total time: 86 s

Final ProtosTestToolSourceLoaderTest focal rerun after fixture reconciliation:
  PASS
```

The final working tree was clean after publication and `origin/main` accepted
the exact product revision above.

## Closure

PERF022 introduced no Protos language semantic change, no generic skip, no
timeout-based pass, and no hidden global mutable runtime state.

The published result removes or amortizes demonstrated pathological ordinary
test cost while preserving focused owners for the required integration,
authority-isolation, source-loading, embedding, and workspace/package
correctness properties.
