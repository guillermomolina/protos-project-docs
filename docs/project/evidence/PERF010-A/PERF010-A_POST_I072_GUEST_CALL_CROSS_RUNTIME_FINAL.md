# PERF010-A — Post-I072 guest-call cross-runtime final investigation

Status: **FINAL STATIC/CROSS-RUNTIME INVESTIGATION; COMPILER CONFIRMATION STILL REQUIRED**

This durable, non-normative record retains the completed post-I072 investigation
supplied by the maintainer and reconciled against Protos
`cf9b39b25dc9a3c4cd1c538749c3a363760ae45b`, the retained
`protos-benchmarks` evidence, and the installed Graal/Truffle source line.

It extends, rather than replaces, the retained post-I072 F′ causal result:
F′ remains closed as not the dominant big cost. This record identifies the
strongest remaining architectural family and the comparison basis for the next
causal campaign.

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

COMPARISON_IMPLEMENTATIONS=
  SimpleLanguage,
  GraalPy,
  GraalJS,
  TruffleRuby,
  Apple_Pkl,
  TruffleSqueak,
  TruffleSOM

PRODUCT_MODIFICATION=NONE
BENCHMARK_MODIFICATION=NONE
NEW_EXPERIMENTS_EXECUTED=NO
```

## Final result

The strongest common explanation is no longer one leaf operation. It is the
complete hot **guest-call path** failing to reduce to the normal mature-Truffle
steady-state form under partial evaluation.

The post-I072 controls and canonical workloads are consistent with an additional
cost on the order of roughly three microseconds for one extra guest call, while
the shared recursive driver retains roughly 5.5–6.5 microseconds per iteration.

The exact value is a descriptive model over retained evidence rather than a new
controlled measurement, but the scaling is strong enough to identify the unit
of cost:

```text
DOMINANT_COST_UNIT=GUEST_CALL_PATH
MEASURED_LEAF_OPERATION_COST=MINOR_TO_SECONDARY
```

The dominant-cause confidence is still below formal PERF010-A closure because no
current-HEAD compiler trace has yet proved exactly whether the hot roots fail
compilation or compile while retaining the same work.

## Shared-driver interpretation

All four diagnostic workloads share the same recursive shape:

```protos
repeat: (count, operation) => {
    (count > 0).ifTrue() {
        operation()
        repeat(count - 1, operation)
    }
}
```

The retained post-I072 control medians remain around the same absolute
tens-of-milliseconds scale as the canonical workloads. The three call-heavy
canonical workloads add a further similar per-call increment.

This produces the descriptive model:

```text
driver iteration:
  Boolean control callback
  + recursive repeat call
  + smaller Integer/control/binding work

call workload:
  driver iteration
  + one measured guest call
```

A roughly three-microsecond guest-call tax predicts the observed absolute scale
surprisingly closely. This materially reduces the priority of isolated
`BigInteger`, comparison, subtraction or leaf slot-read costs as the first
explanation, while not proving that they are zero.

## Cross-runtime comparison

| implementation | callee selection | arguments | closure environment | captured access |
|---|---|---|---|---|
| SL Bytecode DSL | function identity + stable-target assumption + DirectCallNode | flat Object[] | no general closures | frame slots |
| GraalPy Bytecode DSL | cached function + code-stability assumption + DirectCallNode | flat PArguments | PCell cells; frame not captured as lexical environment | cell in slot |
| GraalJS | instance cache, then code identity while retaining DirectCallNode | flat JSArguments | materialized frame/block frame only when capture requires it | constant depth/slot |
| TruffleRuby | method metaclass/lookup assumptions; blocks by call target + DirectCallNode | flat RubyArguments.pack | frame materialized when block capture requires it | constant depth/slot |
| Pkl | callTarget or VmClass + DirectCallNode | flat receiver/function/args shape | frame materialized for lambda capture | constant levelsUp/slot |
| TruffleSqueak | receiver class + hierarchy/method-dictionary assumptions + stable call target | flat | first-class context lazily attached to materialized frame | copied/block values and fixed frame layout |
| TruffleSOM | class/layout + assumptions; canonical Boolean/block paths specialize aggressively | flat | frame materialized only when block capture requires it | constant contextLevel/slot |
| Protos | direct Closure paths can repeat D013 `call` lookup; represented Integer/Boolean fall through generic selection; ordinary guarded path is narrower | rich activation or compact ABI depending path | frame-backed authority installed per root; rich context path remains | dynamic lexical/context machinery plus frame-backed local paths |

The exact implementation details differ, but the common pattern across all
seven comparison runtimes is:

```text
stable semantic identity
  -> guard / assumption
  -> stable DirectCallNode
  -> compact or flat arguments
  -> captured state represented so PE can see stable depth/slot/cell structure
  -> exact generic fallback when assumptions fail
```

Protos reaches a `DirectCallNode` in several paths, but too much generic
selection, activation/control and lexical machinery can remain ahead of that
stable target.

## Protos deviations that remain relevant

### 1. Direct Closure invocation

Direct `operation()` / recursive Closure calls can still enter a path that
performs ordinary `call` selection and generic preparation rather than making
the Closure definition/call target a stable early call-site fact.

This is a major contrast with the comparison runtimes.

### 2. Represented Integer and Boolean receivers

I072-A's guarded lookup is deliberately limited to ordinary
`ProtosObjectValue` chains. A represented value encountered while building the
guard invalidates that guarded selection.

Therefore the `repeat` driver operations:

```text
count > 0
count - 1
(...).ifTrue(...)
```

are precisely on receiver families not covered by the ordinary-object guarded
selection substrate.

I072-E already contains structured Boolean machinery for canonical
`ifTrue`/`ifFalse`/related selections, but canonical Boolean represented
receivers cannot fully benefit from that machinery through I072-A's ordinary
guard. This is an implementation-completeness gap, not a reason to reopen the
ratified Boolean semantics.

### 3. Structured prepared-invocation cascade

The current prepared-invocation lowering contains a large structured-kind
classifier and associated Bytecode locals/try-finally regions. Once a root is
successfully partially evaluated, branch profiling and constant selection may
remove most of it. If the root does not compile or target selection remains
non-constant, this machinery becomes real repeated work.

It is therefore an amplifier of the PE failure family, not yet independently
established as the dominant cause.

### 4. Semantic shell plus helper root

Protos uses a semantic shell root that calls a helper target containing the
semantic body. Prior causal ablation measured only a small percentage-scale
effect, so this remains secondary by itself.

It can still increase the compiler graph or interact with other mechanisms, but
the retained evidence does not support treating it as the primary target.

### 5. Activation and invocation carriers

The rich Closure path constructs fresh invocation state including execution
context, supplied guest Array, return-home state and copied captured-context
lists. I072's compact ordinary method ABI defers some of this work on admitted
paths, but direct Closure invocation remains materially richer.

The final investigation does not claim that one allocation here is the sole
cost. The issue is whether the full carrier remains physical because PE never
reduces the call.

### 6. Frame escape / lexical environment

The completed cross-runtime comparison weakens the generic claim that
"materializing a frame is wrong": GraalJS, TruffleRuby, Pkl, TruffleSOM and
TruffleSqueak all materialize frames or context state when closures need it.

The Protos-specific concern is the representation/order on the rich path:
a frame-backed lexical authority can retain the `VirtualFrame` in an
already-existing execution context. On that route, the deferred
`prepareForContextObservation() -> frame.materialize()` seam is not the normal
installation path.

This is a strong static candidate for a post-I068 compiler regression, but it
cannot explain the majority of the cost that predates I068. It must therefore
remain a conditional prerequisite candidate rather than the universal root
cause.

## Boolean `ifTrue` reconciliation

The canonical Boolean protocol is already ratified and should not be reopened.

```text
D051:
  ordinary strict-Boolean conditional protocol
  no truthiness
  no second primitive-if semantic universe

I072-E:
  structured Boolean prepared-call machinery exists

remaining implementation gap:
  represented canonical Boolean receivers are not admitted by I072-A's
  ordinary-object guarded lookup
```

Therefore making canonical Boolean represented dispatch reach a stable guarded
selection is required implementation completeness even if its isolated timing
contribution later proves small.

A separate question remains whether a literal trailing Closure used only by a
canonical conditional can remain virtual/elided in compiled code. That is a
later architecture question and must preserve Closure identity, capture,
reflection, non-local return and ordinary override semantics.

## Confidence

These values are investigative confidence estimates, not statistical confidence
intervals:

```text
COST_IS_GUEST_CALL_PATH_NOT_MEASURED_LEAF ~= 95%
HOT_CALL_PATH_NOT_EFFECTIVELY_PE_REDUCIBLE ~= 85%
THIS_FAMILY_IS_THE_ONLY_MATERIAL_CAUSE ~= 55-60%
ONE_SINGLE_MECHANISM_EXPLAINS_EVERYTHING < 10%
FRAME_ESCAPE_ORDER_AS_SOLE_CAUSE <= 5%
```

The formal PERF010-A fields remain:

```text
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
```

until current-HEAD compiler lifecycle and causal implementation evidence close
the remaining gap.

## Consequence

The next campaign should be ordered by dependency:

```text
compiler confirmation
  -> restore compilability only if the exact failure requires it
  -> constant direct Closure-call selection
  -> represented Boolean/Integer guarded selection
  -> measure residual
  -> only then reopen captured-environment / literal-control-Closure architecture
```

The companion plan record is:

`PERF010-A_POST_I072_RECOMMENDED_ACTION_PLAN.md`.
