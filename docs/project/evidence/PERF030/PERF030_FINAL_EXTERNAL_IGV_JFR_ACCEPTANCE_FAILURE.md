# PERF030-O — final external IGV/JFR acceptance failure

## Scope

This record preserves the final external compilerability acceptance performed
after all six repair families selected by PERF030-H were published.

The owning live work item is
`guillermomolina/protos#784` (**PERF030 — Make frame-local creation ordinal
PE-constant**). PERF024 / #756 remains blocked from physical cross-runtime graph
interpretation because this acceptance still observes a permanent Truffle
partial-evaluation compiler failure on the selected workload.

This is non-normative diagnostic evidence. It does not change Protos semantics,
specification authority, benchmark semantics, or product source.

## Authorities

```text
SLICE=PERF030-O
WORK_TYPE=INVESTIGATION
IMPLEMENTATION=NO

PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=cf6eb4c9aeaef4049fa5b56089d2dd1d2654d44b
PRODUCT_VERSION=0.3.193-SNAPSHOT
PRODUCT_COMMIT_SUBJECT=PERF030-N: boundary generic frame name lookup

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
BENCHMARK_REVISION=dfc2a34dacc40e57e2e625789c5d3300615bc979

WORKLOAD=primitive-closure-call
DIAGNOSTIC_WARMUP=60
DIAGNOSTIC_STEADY=10
DIAGNOSTIC_SAMPLE_CALLS=100000
PROTOS_RUN_MODE=auto
ACTUAL_RUN_MODE=prepared

MAINTAINER_REPORTED_LOCAL_TESTS=PASS
```

Before diagnostics, the maintainer verified that both exact revisions were
checked out and both worktrees were clean.

The product revision is the final PERF030-N authority. Its checked-in static
guard reports:

```text
TOTAL_LOCAL_RANGE_SINKS=36
PE_REACHABLE_PROVEN_CONSTANT=13
PE_REACHABLE_RISK=0
BOUNDARY_CUT=19
NOT_PE_REACHABLE=4
PE_REACHABILITY_UNKNOWN=0
local-range-pe-guard=PASS
```

The external acceptance below therefore tests whether that static model is
sufficient to eliminate the real compiler blocker on the unchanged workload.

## IGV acceptance

The unchanged generic IGV diagnostic completed with workload correctness PASS:

```text
IGV_DIAGNOSTIC=PASS
IGV_CORRECTNESS=PASS
IGV_RESULT=1
IGV_PRODUCT_REVISION=cf6eb4c9aeaef4049fa5b56089d2dd1d2654d44b
IGV_PRODUCT_VERSION=0.3.193-SNAPSHOT
IGV_BGV_FILES=2
IGV_IDENTITY=/workspaces/protos-benchmarks/results/local/truffle-diagnostics/igv/e4fd8ac763bf8df8fa85dfa57a6609e91dfccc920267e08421bccce641da0f32/identity.json
IGV_ARTIFACT_DIR=/workspaces/protos-benchmarks/results/local/truffle-diagnostics/igv/e4fd8ac763bf8df8fa85dfa57a6609e91dfccc920267e08421bccce641da0f32
```

Exact BGV identities:

```text
/workspaces/protos-benchmarks/results/local/truffle-diagnostics/igv/e4fd8ac763bf8df8fa85dfa57a6609e91dfccc920267e08421bccce641da0f32/graal_dumps/TruffleHotSpotCompilation-2436[ProtosSemanticBytecodeRootNodeGen@679a13b6].bgv | 16218983 bytes
/workspaces/protos-benchmarks/results/local/truffle-diagnostics/igv/e4fd8ac763bf8df8fa85dfa57a6609e91dfccc920267e08421bccce641da0f32/graal_dumps/TruffleHotSpotCompilation-2626[ProtosSemanticBytecodeRootNodeGen@679a13b6].bgv | 10690672 bytes
```

No BGV size is interpreted as performance evidence.

A bounded signature inspection of those BGVs shows the compiled root contains
`BindClosureFrameParameter`, `LocalRangeAccessor[0...0]`,
`LocalRangeAccessor.setObject`, and
`ProtosBytecodeRootNode.createCurrentFrameBinding`.

For the exact workload:

```protos
run: () => {
    identity: (value) => { value }
    identity(1)
}
```

that identifies the root represented by the successful BGVs as the nested
`identity(value)` Closure.

## JFR acceptance

The unchanged generic JFR diagnostic also preserved correctness:

```text
JFR_DIAGNOSTIC=PASS
JFR_CORRECTNESS=PASS
JFR_RESULT=1
JFR_PRODUCT_REVISION=cf6eb4c9aeaef4049fa5b56089d2dd1d2654d44b
JFR_PRODUCT_VERSION=0.3.193-SNAPSHOT
JFR_IDENTITY=/workspaces/protos-benchmarks/results/local/truffle-diagnostics/jfr/467e36044aa8c0ccc8f5307f5ff173e288dff9b4deff1ce4478af7271b4408ef/identity.json
JFR_RECORDING=/workspaces/protos-benchmarks/results/local/truffle-diagnostics/jfr/467e36044aa8c0ccc8f5307f5ff173e288dff9b4deff1ce4478af7271b4408ef/recording.jfr
```

Primary `jdk.graal.compiler.truffle.Compilation` evidence:

```text
JFR_COMPILATION_EVENTS=3
JFR_SUCCESSFUL_COMPILATIONS=2
JFR_UNSUCCESSFUL_COMPILATIONS=1
JFR_PERMANENT_FAILURES=1
```

The only unsuccessful compilation has:

```text
rootFunction=ProtosSemanticBytecodeRootNodeGen@4ac373e9
truffleTier=1
permanentFailure=true
failureReason=jdk.graal.compiler.code.SourceStackTraceBailoutException$1:
  Partial evaluation did not reduce value to a constant,
  is a regular compiler node: 36664|Pi
```

The numeric `36664|Pi` compiler-node identity is not treated as a durable
source identifier and is not equated numerically with historical
`8231|Pi`, `7799|Pi`, or `5866|AnyNarrow`.

The recording contains two successful compilation events for the other guest
root, at Truffle tiers 1 and 2.

The previously observed PERF029 numeric tokens do not recur in the current
failure inventory:

```text
PERF029_BAILOUT_7799=ABSENT
PERF029_BAILOUT_5866=ABSENT
```

## Root attribution

The IGV BGV signature identifies the successfully compiled root as the nested
`identity(value)` Closure. The single permanently failing JFR root is therefore
the outer `run` Closure.

```text
SUCCESSFUL_ROOT_SEMANTIC_ROLE=INNER_IDENTITY
FAILING_ROOT_SEMANTIC_ROLE=OUTER_RUN
```

At the accepted product revision, the old PERF030-C runtime-name creation path is
no longer the ordinary outer-root path. Lowering emits:

```text
CreateCurrentIndexedLocalSlot
 -> ProtosBytecodeRootNode.createIndexedCurrentLocalSlot(...)
 -> ProtosFrameLexicalBindingAuthority.createFrameBackedBindingAt(...)
 -> LocalRangeAccessor.isCleared(..., ordinal)
 -> LocalRangeAccessor.setObject(..., ordinal, value)
```

`CreateCurrentIndexedLocalSlot.ordinal` is a Bytecode DSL `int`
`@ConstantOperand`, and the static guard classifies the relevant indexed
LocalRange sinks as PE-reachable proven-constant.

Nevertheless, the unchanged real compiler acceptance still fails in this outer
root with the same required-constant partial-evaluation failure class.

The evidence therefore establishes the surviving source corridor but does not
yet decompose the exact point where a value expected to remain PE-constant
ceases to satisfy the compiler. In particular, this record does not claim that
the new numeric `Pi` node is the historical node, nor does it justify a product
patch merely from numeric-token recurrence.

```text
OLD_F3_RUNTIME_NAME_PATH=REPAIRED
CURRENT_SURVIVING_ROOT=OUTER_RUN
CURRENT_SURVIVING_CORRIDOR=CreateCurrentIndexedLocalSlot -> createFrameBackedBindingAt -> LocalRangeAccessor
EXACT_CONSTANT_LOSS_POINT=UNRESOLVED
```

## Acceptance classification

PERF030-O requires zero permanent Truffle compiler failures. That condition is
not met.

The surviving failure remains a required-constant compilerability failure on the
outer-root indexed frame-local creation corridor whose terminal accesses are
`LocalRangeAccessor` operations. It is therefore retained under the same
PERF030 compilerability class while the exact loss-of-constancy point is
investigated next.

```text
CLASSIFICATION=FAIL — SAME COMPILERABILITY CLASS

PERF030_O_EXTERNAL_ACCEPTANCE=FAIL
PERF030_COMPILERABILITY_CLASS_SURVIVES=YES
NEW_PERMANENT_COMPILER_BLOCKER=NO

PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
```

PERF030 remains open. PERF024 physical cross-runtime graph interpretation must
remain paused.

## Next bounded work

The next slice is investigation only:

```text
NEXT_SLICE=PERF030-P
WORK_TYPE=INVESTIGATION
IMPLEMENTATION=NO
PURPOSE=SOURCE_ATTRIBUTE_OUTER_RUN_CONSTANT_LOSS
```

PERF030-P must determine, from exact source/generated-code/compiler-contract
evidence, where the outer `CreateCurrentIndexedLocalSlot` path loses the
required PE constancy despite its `@ConstantOperand` provenance.

It must not implement a repair. It must distinguish at least:

- loss before or at generated Bytecode DSL constant-operand handling;
- loss while crossing ordinary Java helper parameters;
- loss at `createFrameBackedBindingAt` / `LocalRangeAccessor`;
- a different required-constant assertion in the same outer root.

Only after that source attribution may a bounded implementation slice be
authorized.

AI assistance: this durable evidence record was drafted with ChatGPT from the
maintainer-executed exact-revision PERF030-O preflight, IGV and JFR diagnostics,
bounded BGV signature inspection, current exact-revision Protos source, prior
PERF030 durable evidence, and maintainer-reported local validation.
