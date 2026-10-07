# PERF032-E — cross-Truffle graph-parity research

Date: 2026-10-07

## Work identity

~~~text
WORK_ITEM=PERF032
ISSUE=guillermomolina/protos#831
WORK_TYPE=INVESTIGATION
PRODUCT_REVISION=26a844c6f825822e73a419e44635aad4558ebdc4
PRODUCT_VERSION=0.3.279-SNAPSHOT
BENCHMARK_REVISION=1afa23af712988db1137e1528af25661ee530704
WORKLOAD_OWNER=primitive-return-literal
PRODUCT_CHANGE=NO
BENCHMARK_CHANGE=NO
COMMAND_EXECUTION=NONE
~~~

This is durable, non-normative performance-investigation evidence. It does not
change observable Protos semantics, the normative specification, product code,
or benchmark code.

## Starting point

PERF033 completed the pay-as-you-grow canonical callable repair selected by
PERF032 and retained the compatible current cross-Truffle reference:

~~~text
primitive-return-literal
  Protos canonical = 84.7862255 ns/call
  GraalJS          = 40.5446410 ns/call
  GraalPy          = 70.4909510 ns/call

sample_calls = 1,000,000
warmup       = 60
steady       = 10
~~~

All three measurements use the canonical Polyglot executable surface and passed
the retained correctness and steady-state admission policy. The remaining
PERF032 question is therefore no longer the removed fixed host envelope. It is
whether current Protos exposes materially more compiled Truffle/Graal structure
than its peers for the smallest guest mechanisms.

The investigation deliberately follows this order:

~~~text
semantics / pay-as-you-grow
  -> compiled graph structure
  -> cross-Truffle structural parity
  -> only then microoptimization
~~~

The timing ratio by itself authorizes no new optimization.

## Selected graph phase

The comparison phase is:

~~~text
SELECTED_GRAPH_PHASE=After TruffleTier
~~~

This is the Graal graph after Truffle partial evaluation and Truffle-tier
cleanup, before later generic Graal high/mid/low-tier optimization obscures the
language-owned structure.

The current benchmark repository already contains a GraalVM 25.4.4.1.1
IgvUtility-based analyzer that verifies the presence of After TruffleTier.
The normal graph matrix therefore reuses that path rather than adding another
IGV stack or using the historical Graal 24 tooling.

Primary upstream reference:

- GraalVM 25 optimization guide:
  https://www.graalvm.org/jdk25/graalvm-as-a-platform/language-implementation-framework/Optimizing/

## Primary and secondary graph metrics

The primary metric is:

~~~text
relevant_graph_nodes_total_after_truffle_tier
~~~

It is the sum of the exact Graal IR node counts in the selected
After TruffleTier graphs for every relevant language-owned compiled unit
required by one steady prepared Value.execute() operation.

Also retain separately:

~~~text
primary_guest_graph_nodes
graph_count
additional_language_owned_compiled_units
~~~

An inlined helper is already represented by the caller graph and is not counted
twice. A separately compiled language-owned CallTarget that remains on the
steady execution path is counted once in the total. Superseded tiers, retries,
setup/module-evaluation roots, inactive continuation roots and invalidated
compilations are not summed as steady-state structure.

TraceCompilation is discovery and lifecycle evidence. Its AST count is not
the graph-size metric. Its IR counts are useful cross-checks, while the selected
phase-specific BGV graph remains authoritative.

Secondary retained structure should include, where the current tooling exposes
it:

~~~text
TraceNodeExpansion truffleTier:
  Count
  Size
  Cycles
  Ifs
  Loops
  Invokes
  Allocs

selected graph:
  node-class histogram
  surviving Invoke nodes
  control-flow splits
  allocation nodes
  guards / deoptimization checks
  loads / surviving indirections
  language/runtime method expansion attribution
~~~

## Compilation-unit matching

Each language must be matched to the callable actually executed by the retained
benchmark surface:

~~~text
Protos   = prepared.executable().execute()
GraalJS  = truffleRun Value.execute()
GraalPy  = truffleRun Value.execute()
~~~

Select the final stable optimized guest root for the workload source/function,
not the hottest graph and not the numerically smallest graph.

For Protos, the existing diagnostic stable root identity
protos-root:<16 lowercase hex> may be reused where useful. No benchmark-only
product identity mechanism is required.

For GraalJS and GraalPy, use the current TraceCompilation target/source
metadata plus BGV source/call-target identity to identify the workload's
truffleRun function.

A framework-owned HostToGuest/Polyglot wrapper common to the three
Value.execute() surfaces is not charged as language-owned graph cost.
Language-specific Truffle roots/helpers are charged.

## Peer convergence and Protos excess

GraalJS and GraalPy form a usable peer reference only when their relevant
compilation-unit accounting is compatible and their selected graphs show the
same qualitative structure band.

Retain their exact node counts and absolute spread rather than inventing a
fixed percentage tolerance.

If the peers diverge materially on a rung:

~~~text
PEER_REFERENCE=UNRESOLVED
~~~

for that rung.

Once the peers converge, presume a Protos structural defect when Protos lies
outside that empirical peer envelope and the excess is supported by concrete
structure, including any of:

- an additional Protos-owned compiled unit;
- a node-count excess larger than the peer spread;
- a distinct block of invokes, control flow, allocations, guards, frame work or
  runtime expansion absent from both peers.

Extra Protos structure is justified only when it maps to active observable
Protos semantics genuinely absent from both peers.

If Protos lies within the peer band/spread and has no unique structural family:

~~~text
STRUCTURALLY_CONVERGED
~~~

and the investigation advances to the next rung.

## Initial workload ladder

The primitive ladder is intentionally ordered by graph mechanism rather than
source-program size alone:

~~~text
1. primitive-return-literal
   baseline callable entry/return of one primitive literal
   EXISTING=YES

2. primitive-local-read
   one local/lexical binding definition and read
   EXISTING=YES

3. primitive-local-write
   mutation of an established local/lexical binding
   EXISTING=YES

4. primitive-integer-add
   primitive integer arithmetic after local-binding machinery
   EXISTING=YES

5. primitive-object-slot-read
   monomorphic read of an existing object slot/property/attribute
   EXISTING=NO
   ADD_REQUIRED=YES

6. primitive-object-slot-write
   monomorphic mutation of an existing object slot/property/attribute
   EXISTING=NO
   ADD_REQUIRED=YES

7. primitive-closure-call
   one ordinary direct identity Closure/function invocation
   EXISTING=YES

8. primitive-method-call
   one receiver lookup/dispatch plus invocation
   EXISTING=YES
~~~

The object-slot rungs must create or acquire the holder outside the measured
callable so the rung isolates access rather than object allocation.

Do not insert an N-call closure loop into this initial matrix. Repeated calls
add loop/hotness/OSR/inlining interactions before the single-call structural
cost is understood.

Larger recursive or loop workloads, including Fibonacci, remain deferred until
the primitive ladder has identified the first structural divergence.

## PERF008 / continueAt reconciliation

Historical PERF008/#496 established that generated Bytecode
CachedBytecodeNode.continueAt was nearly ubiquitous in the then-current
steady-state stacks and that nested semantic/helper Bytecode execution existed
in that older architecture.

That evidence is attribution context only.

Current Protos HEAD no longer contains the old general
InvokeSemanticHelper wrapper pattern in the semantic source interpreter.
The explicit secondary helper CallTarget is now scoped to structured/C-prime
dispatch. Ordinary primitive roots therefore must not be presumed to pay the
historic double-Bytecode dispatch.

The new graph matrix must answer this empirically:

~~~text
if current primitive graphs contain an extra Protos Bytecode compilation unit
or duplicated semantic/helper structure absent in both peers:
    classify it as a priority structural owner
else:
    do not reopen continueAt from historical sampling alone
~~~

No continueAt optimization is authorized by this research.

## Graph-capture execution policy

Graph diagnostics are separate from timing.

For each language/workload use the same prepared executable surface as the
retained timing harness and require correctness before accepting graph
evidence.

Normal matrix capture uses:

~~~text
engine.TraceCompilation=true
engine.BackgroundCompilation=false
compiler.TraceNodeExpansion=truffleTier
jdk.graal.Dump=Truffle:1
~~~

Do not use CompileImmediately for the parity reference because it changes the
normal warmed/profiled compilation conditions. It remains an escalation tool,
not the reference graph.

Use bounded natural warmup with a geometric call budget:

~~~text
1k -> 4k -> 16k -> 64k -> 256k
~~~

Stop at the first two consecutive budgets that retain the same:

~~~text
highest stable compilation tier
relevant graph_count
After-TruffleTier structural summary
no later invalidation/deoptimization of the selected target
~~~

If the cap is reached without stabilization, retain:

~~~text
GRAPH_NOT_STABLE
~~~

rather than choosing a graph arbitrarily.

Use Dump=Truffle:2 only for a later anomalous-rung causal investigation that
needs phase-by-phase evidence.

## Harness ownership and reuse

The implementation belongs in:

~~~text
IMPLEMENTATION_REPOSITORY=guillermomolina/protos-benchmarks
~~~

Reuse:

- truffle/workloads/catalog.json as workload/data authority;
- truffle/measure/cases.json and the existing canonical/executable-value
  surface adapters;
- the current MeasurementEngine.Invocation seam;
- scripts/igv_analyzer.sh and its GraalVM 25.4.4.1.1 analyzer image;
- current reproducibility identity/retention machinery.

Add one generic cross-language graph-capture orchestrator and a compact
After TruffleTier summary path. Language-specific code should supply only the
root/callable identity needed for matching; phase extraction, node accounting
and metrics must remain generic.

Expected node counts must never be hard-coded as benchmark expectations.
Retained graph results are evidence, not source constants.

## Routing

~~~text
PERF032_E_RESULT=DESIGN_COMPLETE
NEXT_SLICE=PERF032-F
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos-benchmarks

NEXT_IMPLEMENTATION_SCOPE=
  generic cross-language graph capture
  + After-TruffleTier compact summaries
  + primitive-object-slot-read
  + primitive-object-slot-write
  + focused parser/unit-matching/reuse tests
  + first 8-rung x 3-language graph matrix
  + retained compact evidence

NEW_ISSUE_REQUIRED=NO
PRODUCT_CHANGE_AUTHORIZED=NO
MICROOPTIMIZATION_AUTHORIZED=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

This remains one slice inside PERF032 because it has no independently meaningful
closure, blockage, scheduling, dependency or decision checkpoint. The first
divergent primitive rung, if any, determines the next causal owner.

## Sources inspected

Current exact repository state:

- guillermomolina/protos@26a844c6f825822e73a419e44635aad4558ebdc4
- guillermomolina/protos-benchmarks@1afa23af712988db1137e1528af25661ee530704

Relevant current project records and code included:

- PERF032/#831 and its PERF033-compatible retained baseline;
- PERF008/#496 historical continueAt characterization;
- ProtosSemanticBytecodeRootNode;
- ProtosHostExecutableClosure;
- ProtosInvocation;
- Protos stable-root diagnostic tooling/reference;
- benchmark primitive workload catalog;
- benchmark measurement surfaces and MeasurementEngine;
- current GraalVM 25.4.4.1.1 IGV analyzer integration.

External upstream reference:

- GraalVM 25 optimization guide:
  https://www.graalvm.org/jdk25/graalvm-as-a-platform/language-implementation-framework/Optimizing/

## AI-assistance disclosure

This research record was materially prepared with AI assistance from ChatGPT
using the exact repository revisions and upstream references listed above.
