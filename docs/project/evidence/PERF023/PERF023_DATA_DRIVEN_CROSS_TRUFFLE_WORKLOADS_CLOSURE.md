# PERF023 — data-driven cross-Truffle workloads and diagnostic micro-workloads

Date: 2026-10-01

Owning Issue: `guillermomolina/protos#754`

This record preserves the closure evidence for PERF023. It is non-normative:
live lifecycle state remains owned by the Protos Issue, and this record does not
define Protos language semantics.

## Published benchmark implementation and retained evidence

The measurement-producing PERF023 implementation was published in:

```text
repository=guillermomolina/protos-benchmarks
benchmark_base_revision=952a4b1f02cc2ef0315cf3c1eeebfd4654ae8f86
measurement_producer_revision=0d6a7727f507c3e3a164e5ef4cda84b4bccfa303
commit=PERF023: data-drive cross-Truffle workloads
semantic_change=NO
```

The raw-result retention repair was then published in:

```text
retention_revision=5cd9e0a492e9d00affc582012aeb95fed527ab4f
commit=PERF023: retain cross-Truffle raw results
retained_path=results/perf023/
```

The implementation keeps the PERF021 runner architecture and makes workload
discovery data-driven rather than adding a new benchmark runner. The later
retention commit does not rerun or replace the measurements: it promotes the
already-observed raw cache payloads byte-for-byte into versioned evidence and
records their original measurement-producing revision.

## Canonical workload catalog

The Truffle harness now owns one workload catalog:

```text
truffle/workloads/catalog.json
```

The maintained JVM matrix, Native matrix, Protos A/B runner, correctness runner,
and JVM diagnostic selector consume this catalog.

The catalog carries, for each workload:

- workload identifier;
- Protos source;
- GraalJS source;
- GraalPy source; and
- expected observable result.

The published corpus is:

| Workload | Expected result |
|---|---:|
| Fibonacci | 6765 |
| Factorial | 2432902008176640000 |
| method-call | 499500 |
| integer-loop | 499500 |

Adding a normal future workload now requires its three equivalent language
sources plus one catalog entry rather than edits to the runner machinery.

## Reproducible preparation

PERF023 added a single idempotent bootstrap entry point:

```text
make truffle-prepare
```

It prepares/verifies:

- exact Protos 0.3.128 and 0.3.129 A/B checkouts;
- the 0.3.128-SNAPSHOT Maven artifact required by the common JVM runner;
- equality of the current Protos Core tree and the pinned 0.3.128 Core tree;
- compiled A/B classes and dependency classpaths;
- the common Truffle JVM harness; and
- the pinned Protos/GraalJS/GraalPy Native runtimes.

Observed preparation:

```text
first prepare:
  real=38.906 s
  prepare=PASS

second prepare:
  real=0.682 s
  all managed checkouts/artifacts/classes reused
  prepare=PASS

post-repair verification:
  real=0.611 s
  prepare=PASS
```

Pinned identities verified by prepare:

```text
Protos JVM baseline:
  version=0.3.128-SNAPSHOT
  revision=f0791896c3c022a747b4d47af629227da58acfab

Protos JVM A/B candidate:
  version=0.3.129-SNAPSHOT
  revision=6e9dfd5aa2132adedeea43f75263f4cd264a6e8a

Core:
  sha256=cc5c6f797f45c64b54d5a5e087efbe43f3879b31b30722e9afe58da22c13fcca

Native Protos:
  version=0.3.116
  binary_sha256=8b3b451e3bf90c7af8a87c210d210e8136166ca2cfc36df7bfba6a3ca1d30e99

Native GraalJS:
  reported_version=GraalVM JavaScript (Oracle GraalVM Native 25.4.4.1.1)
  binary_sha256=5d3ec6dfdbee0da72d31b9358916194894c58ad73b2543d5e0e44e01ec62d75d

Native GraalPy:
  reported_version=GraalPy 3.13.14 (Oracle GraalVM Native 25.4.4.1.1)
  binary_sha256=0149d9d967e6bef020a33dea6b73d5f3764eb3b5f085f31e5a3b1811e78c354f
```

## Correctness

Correctness was established before accepting timing for both new workloads.

```text
method-call:
  protos=PASS result=499500
  js=PASS result=499500
  python=PASS result=499500

integer-loop:
  protos=PASS result=499500
  js=PASS result=499500
  python=PASS result=499500
```

The pre-existing catalog workloads were also revalidated:

```text
fibonacci:
  protos=PASS result=6765
  js=PASS result=6765
  python=PASS result=6765

factorial:
  protos=PASS result=2432902008176640000
  js=PASS result=2432902008176640000
  python=PASS result=2432902008176640000
```

## JVM benchmark observations

Normal JVM benchmark profile, one logical CPU, retained runtime:

| Implementation | method-call steady p50 | integer-loop steady p50 |
|---|---:|---:|
| Protos 0.3.128-SNAPSHOT | 234.167 ms | 170.594 ms |
| GraalJS | 3.041 ms | 0.710 ms |
| GraalPy | 0.056 ms | 0.027 ms |

The new pair is diagnostic rather than a whole-language ranking. Within the
observed Protos JVM run, `method-call` is about 37% slower than
`integer-loop`, so repeated call/dispatch work is materially visible, while
the substantial `integer-loop` baseline shows that the observed gap is not
isolated to method calls alone.

The GraalPy and GraalJS micro-workloads are sufficiently small that individual
steady samples show large relative variation; their values are retained as the
observed benchmark output, not generalized beyond these micro-workloads.

## Native benchmark observations

Pinned Native benchmark profile, five guest calls per sustained batch:

| Implementation | method-call sustained p50 | integer-loop sustained p50 |
|---|---:|---:|
| Protos 0.3.116 | 13.970 ms | 11.825 ms |
| GraalJS 25.4.4.1.1 | 1.476 ms | 1.143 ms |
| GraalPy 25.4.4 | 16.558 ms | 15.726 ms |

Within the pinned Protos Native plane, `method-call` is about 18% slower than
`integer-loop`. JVM and Native remain different measurement planes and these
numbers are not used to infer a direct JVM-vs-Native causal mechanism.

## Protos A/B proof

The catalog-driven A/B runner accepted both new workloads without new
workload-specific machinery.

| Workload | 0.3.128 steady p50 | 0.3.129 steady p50 | Arithmetic delta |
|---|---:|---:|---:|
| method-call | 245.021 ms | 241.808 ms | -1.31% |
| integer-loop | 176.898 ms | 171.149 ms | -3.25% |

These pairs prove A/B compatibility for the new catalog workloads. They are not
treated as evidence of a product optimization because the candidate revision is
the PERF022 Java test-suite change and the Core trees are identical.

After `truffle-prepare`, the A/B runner was also verified to reuse the prepared
classes/classpaths for both revisions rather than recompiling them.

## Diagnostic selection

Without executing JFR/IGV, command construction and identity generation were
validated for all six new workload/language selector pairs:

```text
method-call/protos=PASS
method-call/js=PASS
method-call/python=PASS
integer-loop/protos=PASS
integer-loop/js=PASS
integer-loop/python=PASS
```

Thus optional JVM diagnostics can select the new workloads without another
diagnostic-runner edit.

## Incremental cache and retained raw evidence

Routine incremental execution still uses the ignored local cache:

```text
results/local/truffle-cache/<identity-sha256>.json
```

JFR/IGV diagnostics remain separate under:

```text
results/local/truffle-diagnostics/
```

`results/local/` remains intentionally disposable and ignored by Git. PERF023
now additionally promotes the exact closure observations into versioned raw
evidence:

```text
results/perf023/manifest.json
results/perf023/raw/<identity-sha256>.json
```

The retained manifest published at
`guillermomolina/protos-benchmarks@5cd9e0a492e9d00affc582012aeb95fed527ab4f`
records:

```text
schema=1
work_item=PERF023
producer_revision=0d6a7727f507c3e3a164e5ef4cda84b4bccfa303
retention_policy=raw-cache-payloads-byte-for-byte
retained_observation_count=28

measurement classes:
  jvm-ab-reference=4
  jvm-reference=6
  jvm-smoke=6
  native-reference=6
  native-smoke=6
```

Every manifest entry records the cache key, raw path, raw SHA-256, measurement
class, mode, language, workload, source path/SHA-256, and observable result.
A/B entries also retain the exact Protos revision and version.

The owner-executed retention gate verified that all 28 retained files are
byte-for-byte identical to their original local cache payloads and that a fresh
clone can verify the versioned retained corpus without requiring
`results/local/`.

Repeated benchmark queries before retention demonstrated cache reuse:

```text
JVM method-call: cache hits, real about 0.396 s
JVM integer-loop: cache hits, real about 0.387 s
Native method-call: cache hits, real about 0.042 s
Native integer-loop: cache hits, real about 0.042 s
```

The local cache remains an execution convenience; `results/perf023/` is the
durable benchmark-repository evidence corpus for the PERF023 closure.

## Acceptance reconciliation

1. **PASS** — one canonical catalog replaces the maintained duplicated workload
   enumeration/result metadata in JVM, Native, A/B, correctness and diagnostic
   selection.
2. **PASS** — a future normal workload requires equivalent sources plus one
   catalog entry, with no benchmark-runner edit.
3. **PASS** — Fibonacci and Factorial remain available and correctness-valid.
4. **PASS** — `method-call` is equivalent across Protos/GraalJS/GraalPy and
   executes through JVM and Native machinery.
5. **PASS** — `integer-loop` is equivalent across Protos/GraalJS/GraalPy and
   executes through JVM and Native machinery.
6. **PASS** — the new workloads are cheap in Native and cache-backed; JVM first
   execution retains the existing runtime startup costs rather than adding a
   separate calibration campaign.
7. **PASS** — cache identity remains based on material benchmark/runtime
   identity; catalog discovery itself does not force unrelated observations to
   rerun.
8. **PASS** — JFR/IGV selection addresses the new workloads without another
   runner edit.
9. **PASS** — no Protos product/specification semantic change was introduced.
10. **PASS** — historical PERF020/PERF021 evidence was not rewritten.

### Reopened raw-evidence retention reconciliation

11. **PASS** — the exact already-observed PERF023 raw cache entries were
    promoted without rerunning benchmarks.
12. **PASS** — the retained manifest pins producer revision
    `0d6a7727f507c3e3a164e5ef4cda84b4bccfa303` and stores raw payload hashes.
13. **PASS** — a fresh clone can verify the retained PERF023 evidence from
    `results/perf023/` without `results/local/`.
14. **PASS** — `results/local/` remains disposable/ignored while retained
    closure evidence is versioned.
15. **PASS** — durable project evidence references retention revision
    `5cd9e0a492e9d00affc582012aeb95fed527ab4f`.

## Publication and validation

Owner-executed validation reported:

```text
Python syntax validation: PASS
catalog validation: PASS
duplicate maintained workload-table check: PASS
prepare first-run bootstrap: PASS
prepare idempotence/reuse: PASS
new-workload JVM correctness: PASS
new-workload JVM smoke: PASS
new-workload Native smoke: PASS
new-workload JVM benchmark: PASS
new-workload Native benchmark: PASS
new-workload Protos A/B: PASS
new-workload diagnostic selection: PASS
existing Fibonacci correctness: PASS
existing Factorial correctness: PASS
cache reuse: PASS
raw retention count: 28
raw byte-for-byte promotion: PASS
fresh-clone retained verification path: PASS
benchmark repository final working tree: clean
benchmark implementation push: PASS
benchmark raw-retention push: PASS
```

Published benchmark revisions:

```text
measurement producer:
  0d6a7727f507c3e3a164e5ef4cda84b4bccfa303

closure / retained raw evidence:
  5cd9e0a492e9d00affc582012aeb95fed527ab4f
```

## References

- `guillermomolina/protos#754` — PERF023
- `guillermomolina/protos#748` — PERF021
- `guillermomolina/protos-benchmarks@0d6a7727f507c3e3a164e5ef4cda84b4bccfa303` — measurement producer
- `guillermomolina/protos-benchmarks@5cd9e0a492e9d00affc582012aeb95fed527ab4f` — retained raw closure evidence
- PERF021 benchmark base
  `952a4b1f02cc2ef0315cf3c1eeebfd4654ae8f86`
- Protos I077 baseline
  `f0791896c3c022a747b4d47af629227da58acfab`
- Protos PERF022 candidate
  `6e9dfd5aa2132adedeea43f75263f4cd264a6e8a`
