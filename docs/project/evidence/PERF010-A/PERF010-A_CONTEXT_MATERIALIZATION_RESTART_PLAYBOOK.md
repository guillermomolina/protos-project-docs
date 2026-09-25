# PERF010-A — Context/materialization investigation restart playbook

This is an operational restart aid for the PERF010-A context/frame materialization
investigation. It exists to preserve the debugging effort already spent building
working probes and rejecting misleading ones. It is not normative, does not
select a performance optimization, and does not replace the owning Issue.

## Scope and exact baseline

```text
PRODUCT_REVISION=f1cee2d85858804ad3775adf43a9fab97664da2a
PRODUCT_VERSION=0.3.87-SNAPSHOT

BASELINE_IMAGE=
  protos-benchmarks-perf010a-post-i068-protos:f1cee2d85858

POST_I068_BASELINE_HARNESS_REVISION=
  d6d96f22775a743f604c1645731fa7ebf0ac7ca4

POST_I068_BASELINE_RESULT_REVISION=
  bdb69e2e5135f95a0b41fd258f8afd8d8ca3d7df
```

Before reusing any numeric result against a later product revision, compare that
revision to the pinned product revision and identify whether the activation,
execution-context, Bytecode-root, lexical-authority, frame, call-preparation, or
compiler-boundary surfaces changed.

## Working discipline that avoided wasted loops

Repository/source inspection belongs on the agent side. Use the GitHub connector
or repository source directly. Do not ask the human executor to run `cat`,
`grep`, `sed`, or `git show` merely to expose repository content that the
agent can read itself.

The human executor is used for local/runtime-only evidence:

- Docker execution;
- compilation of temporary diagnostic drivers;
- JFR recording;
- Graal compiler/IGV diagnostics;
- timing;
- allocation measurement.

When shell execution is required, prefer several small independent command
blocks in one interaction over one deeply nested command. Keep the product
repository read-only for investigation slices unless an explicitly authorized
implementation experiment requires otherwise.

## Proven measurement methods

### 1. Exact current-thread aggregate allocation

Use:

```text
com.sun.management.ThreadMXBean
getThreadAllocatedBytes(threadId)
```

The pinned JDK reports:

```text
SUPPORTED=true
THREAD_ALLOCATED_MEMORY_ENABLED=true
```

This method worked reliably with normal TLAB behavior and is the preferred
aggregate bytes/op diagnostic.

A simple calibration allocated 10,000 `byte[128]` arrays and reported exactly
1,440,000 bytes, confirming useful exactness for the current thread.

Use this for aggregate per-operation allocation. Do not use JFR total allocation
bytes as a replacement.

### 2. Exact per-class structural allocation counts

The working JFR method was:

```text
jdk.ObjectAllocationOutsideTLAB = enabled
jdk.ObjectAllocationInNewTLAB   = disabled
jdk.ObjectAllocationSample      = disabled

JVM:
  -XX:-UseTLAB
```

Use a small steady count such as 1000 operations. Print the raw events with the
JFR CLI and aggregate the text on the host.

Interpret:

- deterministic product-class count/op;
- deterministic class byte/op;
- pairwise structural deltas.

Do **not** interpret JFR `TOTAL_BYTES_PER_OP` as real program allocation cost.
JFR recording plus disabled TLAB introduces substantial additional allocation.

### 3. Guest-level reference timing

Use the existing:

```text
Perf010aTimingDriver
```

Properties of this driver that must be preserved:

- persistent process/source context;
- fresh module activation per outer iteration;
- timer begins after fresh module activation;
- no JFR in timing;
- no compiler tracing in timing;
- correctness checked for each iteration.

Reference-like measurement used:

```text
CPU              = one explicit allowed cpuset CPU
network          = none
stack            = -Xss128m
forks            = 5
warmup/fork      = 120
steady/fork      = 100
primary statistic= median
```

The recursive 10,000-step benchmark sources require the large stack.

### 4. Compiler / partial-evaluation diagnostic

The working Graal diagnostics used Truffle graph dumping and compilation
tracing, including:

```text
-Djdk.graal.Dump=Truffle:2
-Djdk.graal.PrintGraph=File
-Djdk.graal.DumpPath=<dir>
-Dpolyglot.engine.TraceCompilation=true
```

Distinguish root identities carefully.

Successful `ProtosSemanticBytecodeRootNodeGen` compilation does not establish
the fate of ordinary `ProtosBytecodeRootNodeGen` roots. In the retained
investigation, successful semantic roots virtualized/eliminated much state while
relevant ordinary roots failed compilation.

## Minimal required case set

When resuming, do not immediately expand the matrix. Re-establish these five
semantic shapes first:

```text
1. minimal invocation / no static current-scope binding / context not evaluated
2. static parameter or local present / context not evaluated
3. captured lexical read / context not evaluated
4. dynamic nearer local creation / context not evaluated
5. context evaluated and escaped
```

Useful controls that were already established:

```text
unused parameter vs read parameter
unused local     vs read local
```

The allocation result was that binding **presence**, not binding read, drove the
large shape change.

## Known-good guest timing discriminator

The simplest guest-level treatment/control pair was:

Control:

```protos
repeat: (count, operation) => {
    (count > 0).ifTrue() {
        operation()
        repeat(count - 1, operation)
    }
}

repeat(10000, () => { 42 })
42
```

Treatment:

```protos
repeat: (count, operation) => {
    (count > 0).ifTrue() {
        operation()
        repeat(count - 1, operation)
    }
}

repeat(10000, () => {
    local: 42
    42
})
42
```

This pair established a material static-binding runtime-shape delta without
mixing in a semantic read requirement.

At the pinned revision the retained reference-like result was:

```text
control median     = 57,689,361.5 ns / 10,000 callbacks
treatment median   = 73,691,918.0 ns / 10,000 callbacks
descriptive delta  = 16,002,556.5 ns / 10,000 callbacks
delta/callback     = 1,600.256 ns
ratio              = 1.277392
percent            = +27.739%
```

The treatment MAD was high. Treat this as evidence that the complete static
binding shape is material, not as an attributable `frame.materialize()`
fraction.

## Known-good structural discriminator

The most important exact pairwise class delta, static binding versus minimal,
was:

```text
ProtosFrameLexicalBindingAuthority  +1/op  +40 B/op
FrameWithoutBoxing                  +1/op  +40 B/op
LinkedHashMap                       +3/op +192 B/op
LinkedHashMap$Entry                 +2/op  +80 B/op
LinkedHashMap$LinkedEntrySet        +1/op  +24 B/op
```

The full unused-local delta also contained one `LinkedHashSet`, immutable
Map/List carriers, entry arrays and backing arrays. Small JFR/JDK
instrumentation-class differences are not causal evidence.

## Known-good host-side decomposition

A simple fresh-object factory loop established:

```text
ProtosExecutionContextValue package ~= 104 B/op
ordinary object same parent         ~= 104 B/op
ProtosReturnHome                    ~=  16 B/op
frozen empty ProtosArrayValue       ~= 136 B/op
full activation factory             ~= 344 B/op
```

This is useful as a lower-level decomposition aid only. Host Java loop timing is
not reference guest timing.

## Failed or misleading approaches — do not repeat

### Fake ordinary-context activation as a frame-materialization ablation

Attempting to execute a root with an activation whose `context()` is an
ordinary `ProtosObjectValue` was not a valid causal ablation.

It produced semantic failures (`ProtosSignalException`) on local-binding
execution because the current Bytecode implementation deliberately falls back
to generic lexical lookup for non-genuine execution contexts. That fallback
does not reproduce the genuine frame-local establishment path.

Do not use:

```text
genuine ProtosExecutionContextValue -> ordinary ProtosObjectValue
```

as a proxy for "same semantics without frame.materialize()".

A valid ablation would require a product/harness intervention that preserves the
same binding semantics and changes exactly the materialization mechanism.

### Reusing one activation across repeated root executions

Do not reuse one invocation activation for repeated measurements of a root that
installs frame authority. `InstallFrameLexicalAuthority` is root-entry state
for a fresh invocation. Reusing the activation can cause repeated authority
replacement/migration and measures a different lifecycle.

Use a fresh invocation activation per measured invocation.

### Constructing a partial activation without invocation arguments

A closure root may execute argument-count checks even with zero explicit
parameters. A hand-built activation without the invocation argument array can
fail with:

```text
parameter binding requires an invocation activation
```

Use `ProtosActivation.forClosureInvocation(...)` or reproduce all invocation
state exactly.

### Direct-driver timing as reference performance

Early direct-driver cases produced surprising orderings such as the explicit
context-escape case timing below the minimal case. That is evidence of
warmup/compiler/shape sensitivity, not a semantic speedup.

Use direct timing diagnostically. Use `Perf010aTimingDriver` for retained
guest-level timing.

### JFR allocation sampling for exact counts

`ObjectAllocationSample` is useful for stack presence but not exact
count/op. The working exact structural method is
`ObjectAllocationOutsideTLAB` with `-XX:-UseTLAB`.

### Huge JFR JSON aggregation

Do not emit and parse one huge `jfr print --json` allocation stream. One
earlier aggregation was killed after the first case.

Prefer:

1. small steady operation count;
2. `jfr print --events jdk.ObjectAllocationOutsideTLAB`;
3. host-side streaming/text aggregation.

### Python inside the runtime image

The final PERF010-A runtime image does not provide a usable `python3` command
for these ad-hoc analyses. Run aggregation Python on the host.

### JFR Configuration API assumption

Do not assume `jdk.jfr.Configuration.parse(Path)` exists on the pinned JDK.
Prefer the JFR CLI or `Configuration.create(Reader)` when Java-side custom
configuration is necessary.

## Source-inspection facts worth rechecking only after code movement

At the pinned revision:

```text
ProtosActivation.forClosureInvocation
  -> prelude.newExecutionContext()

ProtosExecutionContextValue
  -> ProtosMapBackedLexicalBindingAuthority

InstallFrameLexicalAuthority
  -> ProtosFrameLexicalBindingAuthority(...)
  -> frame.materialize()
```

`InstallFrameLexicalAuthority` is only emitted for roots where the lowerer
found at least one eligible current-scope frame-backed binding.

`ProtosFrameLexicalBindingAuthority` owns:

- `frameBackedNames`;
- `frameBackedOffsets`;
- `LocalRangeAccessor`;
- `BytecodeNode`;
- retained materialized frame;
- `LinkedHashMap` dynamic overflow;
- `LinkedHashSet` establishment order.

Do not ask the human executor to re-prove these by shell inspection. Re-read the
current repository source directly if the product revision changes.

## Relationship to PLAT039 on restart

Before running more fine-grained context/frame experiments, read the final
PLAT039 decision and the implementation that consumes it.

The preliminary PLAT039 evidence identified hot `List/ArrayList`,
`LinkedHashMap`, generic functional carriers and runtime-compilation boundary
leaks as representation/PE work. Several of those same JDK collection shapes are
present in the measured frame-authority construction.

Therefore the next restart question is:

```text
DID_PLAT039_CONSUMING_IMPLEMENTATION_CHANGE_THE_MEASURED_HOT_SHAPE?
```

If YES:

1. establish exact new product revision;
2. rerun the no-binding vs unused-local guest timing pair;
3. rerun the small exact JFR structural pair only if the timing/shape delta
   remains;
4. repeat compiler diagnostics on the exact ordinary roots;
5. only then reconsider PLAT037 Candidate B.

If NO, PERF010-A can continue decomposition directly from the retained checkpoint.

## Stop conditions

Stop rather than expanding the matrix when any of these is true:

- the candidate requires an unresolved platform/runtime architecture decision;
- the proposed ablation changes lexical semantics rather than only physical
  representation;
- the ordinary root still fails compilation such that optimized survival cannot
  be established;
- a PLAT039-selected implementation is about to change the same hot carriers;
- further measurement would only split one noisy descriptive delta without
  changing the next project decision.

## Current restart state

```text
EAGER_CONTEXT_WRAPPER_PHYSICAL_SURVIVAL=YES
STATIC_FRAME_AUTHORITY_PHYSICAL_SURVIVAL=YES
STATIC_FRAME_OBJECT_PHYSICAL_SURVIVAL=YES
STATIC_BINDING_RUNTIME_SHAPE_COST_MATERIAL=YES

FRAME_MATERIALIZE_ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
LAZY_CONTEXT_ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED

PLAT037_B_PERFORMANCE_JUSTIFICATION=INCONCLUSIVE
PLAT037_B_IMPLEMENTATION_SELECTED=NO

ORDINARY_HOT_ROOT_PE_SURVIVAL=INCONCLUSIVE
COMPILATION_FAILURE_CONFOUND=YES

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO

NEXT_HIGH_VALUE_ACTION=COMPLETE_PLAT039_BEFORE_MORE_CONTEXT_FRAME_MICRODECOMPOSITION
```
