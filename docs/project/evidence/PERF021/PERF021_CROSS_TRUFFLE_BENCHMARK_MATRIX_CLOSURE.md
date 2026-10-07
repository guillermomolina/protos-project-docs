# PERF021 — cross-Truffle benchmark matrix closure evidence

Date: 2026-10-01

Owning Issue: `guillermomolina/protos#748`

This record preserves the closure evidence for PERF021. It is non-normative:
live lifecycle state remains owned by the Protos Issue, and this record does not
define Protos language semantics.

## Published benchmark implementation

The reusable benchmark-space implementation was published in:

```text
repository=guillermomolina/protos-benchmarks
benchmark_revision=952a4b1f02cc2ef0315cf3c1eeebfd4654ae8f86
commit=PERF021: add cross-Truffle benchmark matrix
semantic_change=NO
```

The implementation establishes `truffle/` as the durable cross-Truffle harness
rather than a PERF021-specific one-off directory. The root Makefile delegates to
that harness while retaining the pre-existing Docker/historical infrastructure.

The initial benchmark corpus contains equivalent sources for:

```text
fibonacci(20) = 6765
factorial(20) = 2432902008176640000
```

for Protos, GraalJS, and GraalPy.

## JVM comparison plane

JVM execution uses one Java/Polyglot comparison plane:

- GraalJS and GraalPy sources are evaluated once and expose a retained
  `truffleRun` callable;
- Protos uses the public reusable I077 embedding authority,
  `ProtosStandaloneHostedSession`;
- each Protos session retains one Process, Polyglot Context, Engine/JIT state,
  and guest carrier across repeated `run` invocation;
- timing separates setup, first explicit/cold invocation, warmup iterations, and
  steady-state iterations; and
- benchmark result values are checked on every repeated invocation.

The normal retained JVM profile uses 5 warmup iterations and 10 steady-state
iterations. The measured process is pinned to logical CPU 0 on the observed
host; topology reported SMT sibling 8.

The common JVM matrix produced correct Fibonacci and Factorial results for all
three implementations.

## Protos revision A/B proof

PERF021 demonstrated A/B comparison without constructing a workload-specific
harness by using two exact Protos checkouts:

```text
baseline:
  version=0.3.128-SNAPSHOT
  revision=f0791896c3c022a747b4d47af629227da58acfab

candidate:
  version=0.3.129-SNAPSHOT
  revision=6e9dfd5aa2132adedeea43f75263f4cd264a6e8a
```

The candidate is exactly one commit ahead of the baseline. That commit is
`PERF022: reduce pathological Java test-suite runtime`.

The Core trees were independently rehashed with relative paths and proved
identical:

```text
core_sha256=cc5c6f797f45c64b54d5a5e087efbe43f3879b31b30722e9afe58da22c13fcca
```

The exact A/B observations were:

| Workload | 0.3.128 steady p50 | 0.3.129 steady p50 | Arithmetic delta |
|---|---:|---:|---:|
| Fibonacci | 187.071 ms | 189.628 ms | +1.37% |
| Factorial | 9.324 ms | 8.497 ms | -8.87% |

These values prove the A/B mechanism and cache identities. PERF021 makes no
causal performance claim from this single observation pair: the compared commit
was test-suite work, and the measurements were not a dedicated statistical
campaign for that commit.

## Incremental timing cache

Timing observations are stored outside versioned repository content under the
local result cache. Cache identity includes the benchmark-space dimensions and
relevant runtime/host identity.

The implementation demonstrated all of the intended incremental behaviors:

- repeating the full JVM matrix reused its valid observations and returned in
  about 0.7 s;
- repeating the full Native matrix reused its valid observations and returned
  in about 0.046 s;
- repeating the Protos A/B matrix reused its four valid observations and
  returned in about 0.258 s;
- selecting JVM Fibonacci after the local Protos revision changed created only
  the missing Protos observation while the existing GraalJS and GraalPy cells
  remained cache hits; and
- ordinary cache population/reuse required no benchmark-definition commit.

The final local timing-cache count observed during closure was 35. The increase
from 34 occurred before diagnostics when the selected-workload JVM command
encountered a genuinely missing Protos identity; diagnostics did not add timing
observations.

## Native comparison plane

Because the current Protos HEAD Native Image path was under investigation for a
GraalVM Native Image problem, PERF021 did not treat a fresh HEAD build as
authoritative. Instead it pinned the latest published Protos Native distribution
known at the time of closure.

Pinned Native runtimes:

```text
Protos:
  release=0.3.116
  candidate_revision=0336ae20216bf2eec17854bea0f6435e4e1e9b19
  archive=protos-0.3.116-native-linux-x86_64.zip
  archive_sha256=61fd90b39a43c574900b3c61d4fe2e65e100166ced66e495281b336f934883e8
  binary_sha256=8b3b451e3bf90c7af8a87c210d210e8136166ca2cfc36df7bfba6a3ca1d30e99

GraalJS:
  reported_version=GraalVM JavaScript (Oracle GraalVM Native 25.4.4.1.1)
  archive_release=25.4.4.1.1
  binary_sha256=5d3ec6dfdbee0da72d31b9358916194894c58ad73b2543d5e0e44e01ec62d75d

GraalPy:
  reported_version=GraalPy 3.13.14 (Oracle GraalVM Native 25.4.4.1.1)
  archive_release=25.4.4
  binary_sha256=0149d9d967e6bef020a33dea6b73d5f3764eb3b5f085f31e5a3b1811e78c354f
```

The Native measurement definition records whole-process cold time separately
from sustained batches. The final `native-process-v2` form also prints and
checks the last Protos result, matching the semantic verification already
performed for GraalJS and GraalPy.

Observed Native benchmark profile (5 guest calls per sustained batch, 3
batches):

| Implementation | Workload | Cold process | Sustained amortized p50 | Result |
|---|---|---:|---:|---:|
| Protos 0.3.116 | Fibonacci | 181.138 ms | 147.568 ms | 6765 |
| GraalJS 25.4.4.1.1 | Fibonacci | 25.599 ms | 6.997 ms | 6765 |
| GraalPy 25.4.4 | Fibonacci | 82.508 ms | 24.029 ms | 6765 |
| Protos 0.3.116 | Factorial | 15.049 ms | 3.168 ms | 2432902008176640000 |
| GraalJS 25.4.4.1.1 | Factorial | 7.677 ms | 1.592 ms | 2432902008176640000 |
| GraalPy 25.4.4 | Factorial | 64.382 ms | 13.720 ms | 2432902008176640000 |

JVM and Native remain separate comparison planes; these values are retained
observations, not a general cross-language performance ranking.

## Workload selection

The published Makefile/runner surface supports:

```text
all workloads
one selected workload
JVM timing
Native timing
Protos revision A/B
JVM JFR diagnostic
JVM IGV/BGV diagnostic
Native diagnostic availability reporting
```

Selected-workload closure checks demonstrated:

- JVM Fibonacci dispatch over Protos, GraalJS, and GraalPy;
- Native Factorial dispatch over Protos, GraalJS, and GraalPy; and
- Protos A/B Fibonacci dispatch over the two exact revisions above.

Existing observations were reused whenever their identity remained valid.

## Diagnostics are independent of timing

JFR and IGV/BGV are separate diagnostic runs with a separate local diagnostic
cache. They are explicitly marked non-primary timing evidence.

PERF021 produced:

```text
JFR:
  status=PASS
  artifact=recording.jfr
  subsequent_request=cache_hit

IGV/BGV:
  status=PASS
  bgv_files=2
  roots:
    ProtosSemanticBytecodeRootNodeGen
    ProtosBytecodeRootNodeGen
```

The IGV path uses Graal's Truffle dump option without forcing experimental
Polyglot engine options. An initial attempt that enabled experimental
`engine.BackgroundCompilation` / `engine.CompileImmediately` was rejected by
the embedding context and was not retained as a successful diagnostic.

For the pinned Native distributions, JFR and IGV are explicitly reported as
`N/A` because those published binaries were not built/validated for the
respective diagnostic capability. PERF021 does not rebuild external languages
or the temporarily non-authoritative Protos HEAD Native Image merely to fill
those cells.

The timing cache remained at 35 after successful JFR/IGV collection, proving
that diagnostic requests did not regenerate primary timing observations.

## Acceptance reconciliation

PERF021 acceptance is satisfied as follows:

1. **PASS** — Fibonacci executes through the JVM benchmark machinery for Protos,
   GraalJS, and GraalPy.
2. **PASS** — Fibonacci executes through pinned Native Image launchers for all
   three available implementations.
3. **PASS** — exact Protos revisions `f0791896...` and `6e9dfd5...` are
   compared through the generic A/B runner.
4. **PASS** — changing/adding only the Protos identity leaves valid GraalJS and
   GraalPy observations reusable.
5. **PASS** — JFR and IGV/BGV are independently requested JVM diagnostics and
   do not populate the primary timing cache; unsupported pinned-Native
   diagnostics report `N/A`.
6. **PASS** — Factorial is present as the second equivalent workload across the
   three language sources and uses the same generic runner machinery.
7. **PASS** — repeated measurements of existing dimensions are cache operations,
   not repository-content publications.
8. **PASS** — historical PERF evidence was not rewritten.

## Validation and publication

Owner-executed closure validation reported:

```text
Python syntax checks for the maintained matrix/diagnostic scripts: PASS
JVM correctness and timing execution: PASS
Native setup/smoke/timing execution: PASS
Protos A/B execution: PASS
single-workload dispatch: PASS
JFR diagnostic: PASS
IGV/BGV diagnostic: PASS
benchmark repository push: PASS
benchmark final working-tree status: clean
```

Published benchmark revision:

```text
952a4b1f02cc2ef0315cf3c1eeebfd4654ae8f86
```

No Protos product or specification source was changed by PERF021 itself.
I076/I077 supplied the reusable public embedding surface in the product
repository; PERF021 consumes that surface from the benchmark repository.

## References

- `guillermomolina/protos#748` — PERF021
- `guillermomolina/protos#750` — I076
- `guillermomolina/protos#751` — I077
- `guillermomolina/protos-benchmarks@952a4b1f02cc2ef0315cf3c1eeebfd4654ae8f86`
- Protos I077 revision `f0791896c3c022a747b4d47af629227da58acfab`
- Protos PERF022 revision `6e9dfd5aa2132adedeea43f75263f4cd264a6e8a`
- Protos Native v0.3.116 candidate
  `0336ae20216bf2eec17854bea0f6435e4e1e9b19`
