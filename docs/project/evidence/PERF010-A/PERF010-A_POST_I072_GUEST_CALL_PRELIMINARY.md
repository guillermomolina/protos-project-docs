# PERF010-A — Post-I072 guest-call investigation preliminary checkpoint

Status: **SUPERSEDED BY FINAL CROSS-RUNTIME REPORT; RETAINED FOR TRACEABILITY**

This durable, non-normative checkpoint retains the preliminary independent
investigation supplied by the maintainer after the post-I072 F′ causal result.
It records the intermediate hypothesis before the seven-runtime comparison was
completed. Claims corrected by the final report are explicitly marked below.

## Evidence identity

```text
PERF_ITEM=PERF010-A
PARENT_PERF=PERF010
PROTOS_REVISION=cf9b39b25dc9a3c4cd1c538749c3a363760ae45b
PROTOS_VERSION=0.3.96-SNAPSHOT
GRAAL_TRUFFLE_SOURCE_COMMIT=7b025988a922

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
POST_I072_HARNESS_REVISION=113949a1aebc0eb769a16b35368a8738f38102a4
POST_I072_EVIDENCE_REVISION=06aa4f4af2476997b9eaf5c88c5df6ace02b8d40
POST_I072_EVIDENCE_ROOT=results/perf010a-post-i072-fprime

PRODUCT_MODIFICATION=NONE
BENCHMARK_MODIFICATION=NONE
NEW_EXPERIMENTS_EXECUTED=NO
```

## Preliminary finding

The preliminary investigation moved the search away from the four measured
operations and toward the common guest-call machinery in the shared recursive
`repeat` driver.

The retained post-I072 controls show that most of the elapsed time survives when
the measured leaf operation is trivialized. The preliminary arithmetic estimated
a guest-call increment of roughly three microseconds on the call-heavy workloads,
while the shared driver costs roughly 5.5–6.5 microseconds per iteration.

The common driver is:

```protos
repeat: (count, operation) => {
    (count > 0).ifTrue() {
        operation()
        repeat(count - 1, operation)
    }
}
```

This made the following cost model plausible:

```text
shared control iteration
  ~= Boolean callback invocation
   + recursive repeat invocation
   + smaller comparison/subtraction/binding costs

additional closure/method/dispatch workload cost
  ~= one additional guest call
```

The preliminary interpretation was therefore:

```text
PRIMARY_COST_UNIT=GUEST_CALL
MEASURED_LEAF_OPERATIONS=SECONDARY
```

## Hot-path structure observed in Protos

Static inspection at the fixed Protos revision found all of the following in
the normal call path:

1. direct Closure invocation reaches ordinary `call` selection through generic
   lookup on paths not admitted by an earlier specialization;
2. call preparation contains a large structured-call classifier and many
   Bytecode DSL locals/try-finally regions;
3. semantic execution uses a semantic shell root that invokes a helper target;
4. activation construction carries receiver, lexical, return-home, argument,
   actor/module/execution-domain and dynamic-control state;
5. root entry installs a frame-backed lexical authority;
6. represented receivers such as Integer and Boolean are not admitted by the
   ordinary-object guarded lookup introduced by I072-A.

At the fixed revision,
`CanonicalToBytecodeLowerer.emitPreparedInvocationForRuntime` contains 41
Bytecode locals and 19 nested structured-kind tests in the generated prepared
invocation path. This is compiler-visible machinery unless partial evaluation
successfully removes it.

## Preliminary cross-runtime comparison

The preliminary pass compared SimpleLanguage and GraalPy.

Both reduce admitted hot calls to the mature Truffle pattern:

```text
stable callee identity
+ stability assumption
+ DirectCallNode
+ flat argument representation
```

GraalPy is especially relevant because its Bytecode DSL root also enables yield,
showing that `enableYield=true` is not by itself a sufficient explanation.
Generator/coroutine machinery is localized to paths that require it.

## Preliminary candidate mechanisms

The preliminary report grouped the remaining mechanisms as:

```text
A. callee selection / call path does not become a PE constant
B. rich per-call activation and argument/context carriers survive
C. structured/continuation machinery remains compiler-visible
D. frame-backed lexical authority may force or inhibit frame virtualization
E. Integer/Boolean represented lookup remains generic in the repeat driver
```

It explicitly rejected the idea that one previously measured micro-cost such as
an Optional, duplicate map probe or List.copyOf could explain the full gap.

## Correction retained from the final investigation

The preliminary wording over-attributed the frame mechanism.

It suggested that frame materialization/escape could be the central explanation.
The completed comparison later established that materializing a frame for
escaping closures is normal in GraalJS, TruffleRuby, Pkl, TruffleSOM and
TruffleSqueak.

The anomaly is narrower:

- several mature runtimes materialize the frame at the closure-capture boundary
  when a frame must escape;
- Protos' rich-activation path can install a frame-backed authority into an
  already-existing execution context while retaining the `VirtualFrame`;
- that mechanism may explain a post-I068 regression or compiler failure, but it
  cannot explain the majority of the cost that already existed before I068.

Therefore:

```text
FRAME_MATERIALIZATION_AS_GENERAL_CAUSE=REJECTED
PROTOS_FRAME_ESCAPE_ORDER_AS_POST_I068_AMPLIFIER=PLAUSIBLE_NOT_PROVEN
```

## Preliminary confidence snapshot

These were investigative confidence estimates, not statistical confidence
intervals:

```text
COST_IS_COMMON_GUEST_CALL_PATH ~= 95%
CURRENT_HEAD_HAS_NO_EFFECTIVE_COMPILED_HOT_KERNEL ~= 75-80%
SINGLE_EXACT_MECHANISM_ESTABLISHED = NO
```

This checkpoint is retained only to show how the final investigation narrowed
and corrected the hypothesis. The final report below is authoritative for the
current post-I072 investigation state:

`PERF010-A_POST_I072_GUEST_CALL_CROSS_RUNTIME_FINAL.md`.
