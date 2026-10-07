# PERF033-E — final validation addendum

Date: 2026-10-07

## Scope

This addendum records the maintainer's final local validation of the already
published PERF033 compatible cross-Truffle harness/evidence revision.

It does not change the measurement interpretation or reopen PERF033.

## Published identities

~~~text
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=26a844c6f825822e73a419e44635aad4558ebdc4
PROTOS_VERSION=0.3.279-SNAPSHOT

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
BENCHMARK_REVISION=1afa23af712988db1137e1528af25661ee530704

PRIOR_PROJECT_RECORD_REVISION=f92977e9174dedfe24d7a288c7f9de72202f00c4
~~~

## Maintainer-reported validation

After publication, the maintainer explicitly reported:

~~~text
GIT_DIFF_CHECK=PASS
LOCAL_TESTS=PASS
~~~

This validation applies to the published reusable benchmark-policy/harness
state associated with PERF033-E.

The retained compatible reference remains:

~~~text
primitive-return-literal
warmup=60
steady=10
sample_calls=1000000
admission_scope=steady-only

Protos canonical = 84.7862255 ns/call
GraalJS          = 40.544641 ns/call
GraalPy          = 70.490951 ns/call

Protos/JS = 2.09118x
Protos/Py = 1.20280x
~~~

No additional Protos product change is implied by this addendum.

## Lifecycle consequence

~~~text
PERF033_FINAL_VALIDATION=PASS
PERF033_STATE=COMPLETED
PERF033_REOPEN_REQUIRED=NO

PERF032_STATE=READY
NEXT_OWNER=PERF032
NEXT_QUESTION=causal attribution of the residual current primitive-return-literal cost
~~~

Further performance work must follow PERF032's profiling-first funnel. The
remaining numerical ratio alone is not authority for another optimization.

## AI-assistance disclosure

This durable addendum was materially prepared with AI assistance from ChatGPT
from the maintainer's explicit final validation report and the already
published PERF033 evidence identities.
