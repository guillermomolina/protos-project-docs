# TEST009-U — failed causal aggregation and upstream single-root diagnostic redirection

## Status

```text
WORK_ITEM=TEST009/#795
SLICE=TEST009-U
PROTOS_DIAGNOSTIC_REVISION=62f3f5710210f247aad8574d3d0d56d3254bfd70
PROTOS_DIAGNOSTIC_VERSION=0.3.230-SNAPSHOT
TEST009_T_REVISION=b6f62a1f52a4d26cd026beb69d3c28a6730c83b7
CURRENT_COORDINATION_HEAD=ef8d43e86902b1edec99365bfd7952b29563c598
CURRENT_COORDINATION_VERSION=0.3.231-SNAPSHOT

TEST009_U=COMPLETE
DIAGNOSTIC_EXECUTION_COMPLETED=YES
CAUSAL_ACQUISITION=INCOMPLETE
SOURCE_REPAIR_AUTHORIZED=NO
TEST009_COMPLETE=NO
```

This record supersedes the prior assumption that TEST009-U should classify the
post-Q CodeTooLarge failures from the TEST009-T aggregate causal correlator. The
completed U acquisition demonstrated that the correlator's completeness model
does not match the contracts of the underlying Graal/Truffle diagnostic
surfaces. The correction is methodological, not a Protos semantic decision.

## Completed U acquisition

The maintainer completed the retained one-Case diagnostic for:

```text
CASE=protos/corpus/conformance:call/closure-call-and-return.protos::plain closure call
SHARD_WORKERS=1
COMPLETED_EXPENSIVE_DIAGNOSTIC_RUNS=1
ABORTED_DIAGNOSTIC_ATTEMPTS=1
PARTIAL_ABORTED_EVIDENCE_USED=NO
```

The completed run reproduced the residual compilerability set:

```text
SEMANTIC_CORPUS=PASS
COMPILATION_FAILURES=23
CODE_TOO_LARGE=22
SEMANTIC_CODE_TOO_LARGE=21
TOO_DEEP=1
PE_CONSTANT_FAILURES=0

CAUSAL_COMPILATIONS=3067
CODE_TOO_LARGE_CAUSAL_RECORDS=22
SEMANTIC_CODE_TOO_LARGE_CAUSAL_RECORDS=21
CAUSAL_ACQUISITION=INCOMPLETE
```

The acquisition failed closed because the current correlator reported, among
other completeness problems:

```text
MISSING_METHOD_EXPANSION
MISSING_NODE_EXPANSION
DURABLE_KEY_COLLISION
```

The retained output reported 235 records missing method/node expansion evidence
and 124 durable-key collisions. Those observations are evidence about the
TEST009-T acquisition model; they are not a causal classification of the
CodeTooLarge product failures.

## Why TEST009-T did not produce the intended "magic API" result

A focused upstream review established that the Graal/Truffle facilities
themselves did run. The mismatch is in the aggregation layer added by TEST009-T.

### Per-compilation expansion trees

Graal's current Truffle optimization guidance defines:

```text
compiler.TraceMethodExpansion=truffleTier
compiler.TraceNodeExpansion=truffleTier
```

as per-compilation expansion-tree views.

Both are emitted through the same expansion-tree formatter and use an
`Expansion tree for <target> after <tier>` heading. The TEST009-T parser instead
looked for headings matching `method expansion` / `node expansion`. Therefore
the parser's required per-record method/node sections cannot be satisfied by the
actual upstream trace format as implemented.

### Aggregate expansion statistics

```text
compiler.MethodExpansionStatistics=truffleTier
compiler.NodeExpansionStatistics=truffleTier
```

are aggregate statistics accumulated across compilations and printed on
shutdown. They are not per-compilation records and must not be required as
though they belonged to every listener lifecycle window.

### Runtime listener API

`OptimizedTruffleRuntimeListener` is still useful and worked as designed. Its
compiler graph surface provides summary `GraphInfo` such as node counts/types
at Truffle/Graal tier boundaries, plus lifecycle success/failure information and
successful compilation-result summaries. It does not expose the textual
method/node expansion tree as structured per-compilation data.

The completed U run therefore proves:

```text
TRUFFLE_LISTENER_API_WORKS=YES
GRAAL_EXPANSION_TOOLING_WORKS=YES
TEST009_T_GLOBAL_CORRELATION_MODEL_WORKS=NO
```

## Upstream standard workflow

The upstream Truffle workflow is targeted rather than global:

1. identify the responsible guest compilation unit;
2. isolate that compilation unit with `engine.CompileOnly=<name>`;
3. use `engine.CompileImmediately=true`;
4. use `engine.BackgroundCompilation=false` for deterministic inspection;
5. enable one of the per-compilation expansion views when useful;
6. enable Graal graph dumping with `-Djdk.graal.Dump=Truffle:1`;
7. inspect the responsible compilation's `After TruffleTier` graph in Ideal
   Graph Visualizer (IGV);
8. use `Dump=Truffle:2` only when phase-by-phase compiler detail is required.

The upstream implementation of `CompileOnly` selects call targets by substring
matching their target name. Mature language roots provide meaningful target
names; for example Espresso's root name is derived from the guest method name
and signature.

## Concrete Protos mismatch

Current Protos semantic Bytecode roots do not provide a stable tooling name.
Their trace identity appears in the form:

```text
ProtosSemanticBytecodeRootNodeGen@<run-local-hash>
```

Current Truffle `OptimizedCallTarget.getName()` derives its cached name from the
root's textual representation. The default Truffle node textual representation
contains the class plus a run-local hash.

Consequently:

```text
COMPILE_ONLY_CAN_SELECT_SEMANTIC_ROOT_FAMILY=YES
COMPILE_ONLY_CAN_SELECT_ONE_DURABLE_SEMANTIC_ROOT=NO
```

This is the immediate blocker to using the standard targeted workflow.

The current Protos diagnostic gate already sets:

```text
-Djdk.graal.DumpPath=target/truffle-compilation/graal_dumps
```

but does not enable `jdk.graal.Dump=Truffle:1`. The repository already
documents `graal_dumps/*.bgv` as a useful retained artifact. IGV itself does not
need to become a Protos product dependency: Protos only needs to emit the BGV
file, which can be opened by the existing IGV installation used by the benchmark
workspace.

## Coordination decision

No new Issue or sub-issue is created.

The work remains a TEST009 slice because it has the same closure target, blocker,
owner and schedule as #795 and does not have independent closure, blockage,
dependency, decision or multi-publication scope under the Issue/slice boundary.

PERF024/#756 and PERF030/#784 are not reopened. Their benchmark/performance IGV
work is historical/independent; TEST009 owns the current compilerability
diagnostic continuation.

```text
NEW_FORMAL_ISSUE_REQUIRED=NO
NEW_SUB_ISSUE_REQUIRED=NO
REOPEN_PERF024=NO
REOPEN_PERF030=NO

NEXT_SLICE=TEST009-V
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos
NEXT_SCOPE=UPSTREAM_STYLE_SINGLE_ROOT_COMPILER_DIAGNOSTIC
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PRODUCT_REPAIR_AUTHORIZED=NO
```

TEST009-V should implement only the missing diagnostic usability needed by the
upstream workflow: a stable diagnostic identity suitable for selecting one
semantic compilation unit and a bounded single-root expansion/BGV diagnostic
surface. It must not attempt another CodeTooLarge product repair and must not
recreate the TEST009-T global stdout/listener correlation model.

## Upstream and repository surfaces reviewed

- `oracle/graal:truffle/docs/Optimizing.md`
- `oracle/graal:truffle/docs/Options.md`
- `oracle/graal:compiler/.../truffle/ExpansionStatistics.java`
- `oracle/graal:compiler/.../truffle/TruffleCompilerImpl.java`
- `oracle/graal:truffle/.../OptimizedTruffleRuntimeListener.java`
- `oracle/graal:truffle/.../runtime/EngineData.java`
- `oracle/graal:truffle/.../nodes/RootNode.java`
- `oracle/graal:truffle/.../nodes/Node.java`
- `oracle/graal:espresso/.../nodes/EspressoRootNode.java`
- `guillermomolina/protos:tools/truffle_compilation_gate.py`
- `guillermomolina/protos:tools/truffle_compilerability_causal.py`
- `guillermomolina/protos:src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java`
- `guillermomolina/protos:docs/design/TRUFFLE_GRAAL_OPTIMIZATION_INVESTIGATION_REFERENCE.md`

## Methodological conclusion

```text
SYSTEMATIC_METHOD=TARGET_ONE_COMPILATION_UNIT
GLOBAL_CORRELATION_REQUIRED=NO
NEXT_EVIDENCE=ONE_NAMED_ROOT_EXPANSION_AND_AFTER_TRUFFLE_TIER_GRAPH
TRIAL_AND_ERROR_REPAIR=FORBIDDEN
```
