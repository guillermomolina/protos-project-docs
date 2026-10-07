# PERF030 — external IGV/JFR acceptance failure

## Scope

This record preserves the external PERF030-B acceptance result for the published
PERF030-A product repair.

The owning live work item is
`guillermomolina/protos#784` (**PERF030 — Make frame-local creation ordinal
PE-constant**). PERF024 / #756 remains paused for cross-runtime physical graph
interpretation until the Protos permanent compiler blocker is actually cleared.

This is non-normative diagnostic evidence. It does not change Protos semantics
and it does not claim a performance improvement.

## Authorities

```text
SLICE=PERF030-B
WORK_TYPE=INVESTIGATION
IMPLEMENTATION=NO
COMMANDS_EXECUTED_BY_AGENT=NO

PRODUCT_REVISION=383ffc025e82918336e3bc2ab350f1275886a576
PRODUCT_VERSION=0.3.177-SNAPSHOT
PRODUCT_RECORD_REVISION=542c7dee07a0a799815abc0179bfe225665e2b49
PRODUCT_RECORD_PATH=docs/project/evidence/PERF030/PERF030_PE_CONSTANT_FRAME_LOCAL_CREATION_REPAIR.md

BENCHMARK_REVISION=dfc2a34dacc40e57e2e625789c5d3300615bc979
WORKLOAD=primitive-closure-call
DIAGNOSTIC_WARMUP=60
DIAGNOSTIC_STEADY=10
DIAGNOSTIC_SAMPLE_CALLS=100000

MAINTAINER_REPORTED_TESTS=PASS
```

The benchmark checkout and Protos checkout were both reported clean before the
diagnostics.

## Diagnostic identities

```text
IGV_IDENTITY=/workspaces/protos-benchmarks/results/local/truffle-diagnostics/igv/10d83aa5e6fae48276f80475e15939a7c5e77290d9725b90acc59c6d36781fb9/identity.json
IGV_ARTIFACT_DIR=/workspaces/protos-benchmarks/results/local/truffle-diagnostics/igv/10d83aa5e6fae48276f80475e15939a7c5e77290d9725b90acc59c6d36781fb9
IGV_DIAGNOSTIC_SHA256=4b053c5c27fde4280be08b746b4651f727c8cd0022ee3b5e958f0526361cd611

JFR_IDENTITY=/workspaces/protos-benchmarks/results/local/truffle-diagnostics/jfr/2b1102560d551ec04cddcf6c427f2ad6feb9e0b05a64b4d30825c42d0e8a0b89/identity.json
JFR_ARTIFACT_DIR=/workspaces/protos-benchmarks/results/local/truffle-diagnostics/jfr/2b1102560d551ec04cddcf6c427f2ad6feb9e0b05a64b4d30825c42d0e8a0b89
JFR_RECORDING=/workspaces/protos-benchmarks/results/local/truffle-diagnostics/jfr/2b1102560d551ec04cddcf6c427f2ad6feb9e0b05a64b4d30825c42d0e8a0b89/recording.jfr
JFR_DIAGNOSTIC_SHA256=4b053c5c27fde4280be08b746b4651f727c8cd0022ee3b5e958f0526361cd611
```

Both identity files report:

- `harness_revision=dfc2a34dacc40e57e2e625789c5d3300615bc979`;
- `harness_dirty=false`;
- `protos.revision=383ffc025e82918336e3bc2ab350f1275886a576`;
- `protos.version=0.3.177-SNAPSHOT`;
- GraalVM CE `25.4.4.1.1`;
- `sample_calls=100000`;
- `steady_iterations=10`;
- `warmup_iterations=60`; and
- `workload=primitive-closure-call`.

## Correctness and IGV admission

Both unchanged generic diagnostics preserved workload correctness:

```text
IGV_CORRECTNESS=PASS
JFR_CORRECTNESS=PASS
RESULT=1
```

IGV produced two BGV files:

```text
/workspaces/protos-benchmarks/results/local/truffle-diagnostics/igv/10d83aa5e6fae48276f80475e15939a7c5e77290d9725b90acc59c6d36781fb9/graal_dumps/TruffleHotSpotCompilation-2256[ProtosSemanticBytecodeRootNodeGen@1d73d12a].bgv | 33546005 bytes
/workspaces/protos-benchmarks/results/local/truffle-diagnostics/igv/10d83aa5e6fae48276f80475e15939a7c5e77290d9725b90acc59c6d36781fb9/graal_dumps/TruffleHotSpotCompilation-2805[ProtosSemanticBytecodeRootNodeGen@1d73d12a].bgv | 11873532 bytes
```

No BGV graph-content interpretation is made by this record.

## JFR compiler result

The JFR recording contains three
`jdk.graal.compiler.truffle.Compilation` events:

```text
JFR_COMPILATION_EVENTS=3
JFR_SUCCESSFUL_COMPILATIONS=2
JFR_UNSUCCESSFUL_COMPILATIONS=1
JFR_PERMANENT_FAILURES=1
```

The unsuccessful compilation has the same permanent bailout token that PERF030-A
was intended to remove:

```text
rootFunction=ProtosSemanticBytecodeRootNodeGen@67601481
truffleTier=1
permanentFailure=true
failureReason=jdk.graal.compiler.code.SourceStackTraceBailoutException$1:
  Partial evaluation did not reduce value to a constant,
  is a regular compiler node: 8231|Pi
```

The two other compilation events for
`ProtosSemanticBytecodeRootNodeGen@74eda1bc` succeed at Truffle tiers 1 and 2.

The bounded known-token searches of the IGV and JFR `run.log` files returned
no matches. That does not override the JFR recording: the
`CompilationFailure` event above is the primary evidence for the acceptance
failure.

The previously repaired PERF029 tokens did not recur:

```text
PERF029_BAILOUT_7799=ABSENT
PERF029_BAILOUT_5866=ABSENT
```

## Classification

```text
PERF030_BAILOUT_8231=STILL_PRESENT
NEW_PERMANENT_COMPILER_BLOCKER=NO

PERF030_B_EXTERNAL_ACCEPTANCE=FAIL
PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
```

The published source repair remains a valid historical product change and the
maintainer-reported tests remain PASS, but PERF030's external acceptance
criterion is not satisfied.

PERF030 therefore remains open. PERF024 must not resume the deferred
Protos/GraalJS/GraalPy physical graph interpretation from this evidence.

## Follow-up boundary

The next bounded step is diagnostic re-attribution of the surviving
`8231|Pi` against current Protos source and the exact failing root.

No further product patch is justified merely by the token recurrence. The next
slice must determine which operation/value remains non-constant at partial
evaluation time, whether the failing root exercises the repaired
`CreateCurrentFrameLocal` operation at all, and whether the surviving token is
the same source-level cause or a different path that happens to produce the same
compiler-node token.

If that investigation identifies a bounded semantics-preserving source repair,
implementation can follow as another PERF030 slice. If it exposes an independent
promotion trigger under project coordination policy, it must be promoted before
implementation.

AI assistance: this durable evidence record was drafted with ChatGPT from the
maintainer-returned PERF030-B diagnostic output, the published PERF030-A record,
and live issue/source state.
