# PERF033-E — compatible retained cross-Truffle reference

Date: 2026-10-07

## Work identity

~~~text
WORK_ITEM=PERF033
ISSUE=guillermomolina/protos#832

PRODUCT_REPOSITORY=guillermomolina/protos
MEASURED_PROTOS_REVISION=26a844c6f825822e73a419e44635aad4558ebdc4
MEASURED_PROTOS_VERSION=0.3.279-SNAPSHOT

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
STARTING_BENCHMARK_HEAD=6c27328eda9ad65565eee0ce45de28e2fc624c7b
BENCHMARK_PUBLICATION_REVISION=1afa23af712988db1137e1528af25661ee530704
SUBJECT=PERF033: stabilize primitive reference sampling policy
~~~

## Final generic policy

The published reusable workload policy for
`primitive-return-literal/reference` is:

~~~text
warmup_iterations=60
steady_iterations=10
sample_calls=1000000
admission_scope=steady-only
~~~

The policy is data-owned by `truffle/measure/cases.json` and is shared across
Protos, GraalJS and GraalPy. It is not keyed to a Protos revision, language or
PERF033-specific execution mode.

The harness remains revision-independent: an ordinary future Protos revision is
selected by checkout directory and does not require source changes.

## Why the policy changed

The earlier 10,000-call peer attempts exposed two timing populations because
the sample unit was too short. A 100,000-call follow-up removed that peer
bimodality, but Protos was still completing tiering in early steady samples and
GraalJS narrowly failed the steady drift gate.

The final 1,000,000-call policy lengthens the sample unit and warmup work
without changing admission thresholds or adding runtime-specific exceptions.

Historical retained and invalid observations remain preserved and are not
rewritten.

## Retained compatible reference

All three final reference runs use the same workload and timing policy and were
published under:

~~~text
results/perf033-cross-truffle-current-1m/
~~~

### Protos canonical

~~~text
PRODUCT_REVISION=26a844c6f825822e73a419e44635aad4558ebdc4
PRODUCT_VERSION=0.3.279-SNAPSHOT
PRODUCT_CLEAN=YES
LANGUAGE=protos
SURFACE=canonical
WORKLOAD=primitive-return-literal
CORRECTNESS=PASS
STEADY_STATE_ADMISSION=PASS
MEASUREMENT_VALID=YES
REFERENCE_ELIGIBLE=YES
STEADY_AMORTIZED_P50_NS_PER_CALL=84.7862255
~~~

Retained path:

~~~text
results/perf033-cross-truffle-current-1m/
  protos-canonical--primitive-return-literal--reference-none/
    20261007T142139395333Z/
~~~

### GraalJS

~~~text
LANGUAGE=js
SURFACE=executable-value
WORKLOAD=primitive-return-literal
CORRECTNESS=PASS
STEADY_STATE_ADMISSION=PASS
MEASUREMENT_VALID=YES
REFERENCE_ELIGIBLE=YES
STEADY_AMORTIZED_P50_NS_PER_CALL=40.544641
~~~

Retained path:

~~~text
results/perf033-cross-truffle-current-1m/
  js-executable-value--primitive-return-literal--reference-none/
    20261007T142227617551Z/
~~~

### GraalPy

~~~text
LANGUAGE=python
SURFACE=executable-value
WORKLOAD=primitive-return-literal
CORRECTNESS=PASS
STEADY_STATE_ADMISSION=PASS
MEASUREMENT_VALID=YES
REFERENCE_ELIGIBLE=YES
STEADY_AMORTIZED_P50_NS_PER_CALL=70.490951
~~~

Retained path:

~~~text
results/perf033-cross-truffle-current-1m/
  python-executable-value--primitive-return-literal--reference-none/
    20261007T142236809230Z/
~~~

## Compatible ratios

Using only the final compatible 1M-policy retained references:

~~~text
PROTOS_VS_JS_RATIO=2.09118
PROTOS_VS_PY_RATIO=1.20280
~~~

These ratios describe this single microbenchmark and are not whole-language
performance claims.

## PERF033 closure interpretation

PERF033's acceptance condition is structural convergence plus compatible
measurement, not a predetermined numeric speedup.

The product implementation already established:

- executable Protos callable support through the public Polyglot/Truffle
  executable-value surface;
- no fresh RootTask or ProtosTask on the minimum ordinary callable path;
- no Actor task-registration/dispatch/cancellation/terminal-publication
  machinery on that path;
- compact/deferred Closure invocation without universal rich activation;
- standard Truffle host-to-guest Context ownership rather than a duplicate
  Protos host envelope;
- no private/internal Graal API dependency; and
- preservation of stronger Task/Actor/Process/session behavior when those
  capabilities are actually used.

The compatible retained cross-Truffle reference now satisfies the remaining
measurement gate.

~~~text
PERF033_STRUCTURAL_GATE=PASS
PERF033_MEASUREMENT_GATE=PASS
PERF033_CLOSEABLE=YES
NEW_PRODUCT_CHANGE_REQUIRED_FOR_PERF033=NO
~~~

The remaining numerical difference versus peers is not, by itself, evidence of
another unjustified fixed layer. Any further causal investigation belongs to
PERF032 and must follow its profiling-first funnel rather than reopening
PERF033 architecture without evidence.

## AI-assistance disclosure

This durable record was materially prepared with AI assistance from ChatGPT
using the exact published benchmark commit and retained result metadata.
