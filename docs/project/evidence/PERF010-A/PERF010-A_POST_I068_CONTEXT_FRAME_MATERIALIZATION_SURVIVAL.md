# PERF010-A — Post-I068 context/frame materialization survival checkpoint

Status: MEASUREMENT CHECKPOINT COMPLETE — FURTHER MICRODECOMPOSITION DEFERRED PENDING PLAT039

This is durable, non-normative performance-investigation evidence for
`PERF010-A / guillermomolina/protos#691`. It does not define Protos semantics,
does not select a production optimization, and does not authorize PLAT037
Candidate B.

## Exact identities

```text
PERF_ITEM=PERF010-A
PRODUCT_REVISION=f1cee2d85858804ad3775adf43a9fab97664da2a
PRODUCT_VERSION=0.3.87-SNAPSHOT

POST_I068_BASELINE_HARNESS_REVISION=d6d96f22775a743f604c1645731fa7ebf0ac7ca4
POST_I068_BASELINE_RESULT_REVISION=bdb69e2e5135f95a0b41fd258f8afd8d8ca3d7df
POST_I068_BASELINE_RESULT=guillermomolina/protos-benchmarks:results/perf010a-post-i068-baseline

PLAT037_DECISION_REVISION=08fdc813295743c5dbb9cf862a264bcb1259e4ce
PLAT039_PRELIMINARY_PRODUCT_REVISION=d4ac1c7be600c00c785dc9742dc7aeaeab17eaec
PLAT039_PRELIMINARY_PROJECT_RECORD_REVISION=d04c82e0f7e56cdca262c5885db90153a194ccdb
```

The PLAT039 preliminary audit is one product commit after the PERF010-A pinned
product revision. Comparing
`f1cee2d85858804ad3775adf43a9fab97664da2a..d4ac1c7be600c00c785dc9742dc7aeaeab17eaec`
shows only `bin/protos` changed. None of the measured activation, execution
context, Bytecode root, lexical-authority, or frame-materialization implementation
surfaces changed between those two revisions.

## Question

After I068 moved statically admitted lexical bindings toward Bytecode DSL frame
storage, what eager physical activation/context/frame state still survives on
ordinary execution paths when the guest does not observe or escape the execution
context, and is the surviving cost large enough to justify PLAT037 Candidate B
before other PE/runtime-representation work?

The required cases were:

1. minimal invocation with no current-scope static binding;
2. static parameter/local present but context not observed;
3. captured lexical read;
4. dynamic nearer-binding creation;
5. explicit context observation/escape.

The investigation deliberately separated:

- structural allocation evidence;
- compiler / partial-evaluation evidence; and
- timing evidence.

## Source-level physical path at the pinned revision

A closure invocation eagerly creates a fresh execution context through:

```text
ProtosActivation.forClosureInvocation(...)
  -> prelude.newExecutionContext()
  -> new ProtosExecutionContextValue(contextPrototype)
  -> new ProtosMapBackedLexicalBindingAuthority()
  -> LinkedHashMap-backed authority
```

For roots with at least one eligible current-scope frame binding,
`InstallFrameLexicalAuthority` then executes as the first root operation and
installs:

```text
new ProtosFrameLexicalBindingAuthority(
    frameBackedNames,
    frameBackedLocals,
    bytecodeNode,
    frame.materialize())
```

The frame-backed authority itself owns JDK collection state including a
`LinkedHashMap` dynamic overflow, `LinkedHashSet` establishment order, and
name/offset collection materialization.

Therefore two distinct eager physical dimensions exist:

1. fresh execution-context wrapper + initial map-backed lexical authority;
2. frame-backed authority + materialized frame for roots with eligible static
   bindings.

PLAT037 Candidate B can address the first dimension. It does not automatically
remove the second.

## Exact current-thread allocation decomposition

Using `com.sun.management.ThreadMXBean` with normal TLAB behavior established
the following deterministic host-side allocation costs:

```text
execution-context-only          ~= 104 B/op
ordinary-object-same-parent     ~= 104 B/op
return-home-only                ~=  16 B/op
frozen-empty-array-only         ~= 136 B/op
activation-factory-full         ~= 344 B/op
```

The fresh execution-context package is therefore real physical allocation, but
it is only one component of activation creation.

A later paired direct-root probe converged after warmup to:

```text
minimal invocation              ~=  432 B/op
unused-static-local invocation  ~= 1312 B/op
increment                       ~=  880 B/op
```

The large increment is associated with the presence of a static binding shape,
not with reading the binding: unused/read parameter and unused/read local cases
had essentially the same steady allocation shape.

## Exact structural allocation evidence

For exact class counts the diagnostic used JFR
`jdk.ObjectAllocationOutsideTLAB` with `-XX:-UseTLAB`. JFR total allocation
bytes are not interpreted as reference bytes/op because the recording and
disabled-TLAB mode add substantial diagnostic overhead. Deterministic
product-class counts and pairwise class deltas are the retained structural
signal.

### Minimal invocation

Per invocation:

```text
ProtosActivation                       1
ProtosArrayValue                       1
ProtosExecutionContextValue            1
ProtosMapBackedLexicalBindingAuthority 2
ProtosReturnHome                       1
ProtosFrameLexicalBindingAuthority     0
FrameWithoutBoxing                     0
```

The execution-context wrapper therefore survives physically even when the
closure body has no current-scope static binding and does not observe
`context`.

### Static parameter/local present

Both unused and read static parameter/local cases add exactly:

```text
ProtosFrameLexicalBindingAuthority  +1/op   +40 B/op
FrameWithoutBoxing                  +1/op   +40 B/op
LinkedHashMap                       +3/op  +192 B/op
LinkedHashMap$Entry                 +2/op   +80 B/op
LinkedHashMap$LinkedEntrySet        +1/op   +24 B/op
```

The fuller unused-local minus minimal diagnostic also showed deterministic
collection/carrier additions including one `LinkedHashSet`, one immutable
single-entry Map carrier, one Map entry array, and further backing arrays. Some
small JVM/JFR/class-generation deltas are diagnostic noise and are not assigned
to Protos runtime cost.

The key structural conclusion is independent of those noisy classes:

```text
STATIC_BINDING_PRESENT
  -> eager ProtosFrameLexicalBindingAuthority = YES
  -> eager physical FrameWithoutBoxing        = YES
```

This occurs even when the static binding is never read.

### Captured read

The tested captured-read invocation retained a physical execution-context wrapper
and one physical frame object but did not create a new
`ProtosFrameLexicalBindingAuthority` for the child invocation. The proven
captured owner remains outside that invocation.

### Dynamic nearer binding

The dynamic-nearer case retained the execution-context wrapper and map-backed
dynamic state. The test shape did not add a new frame lexical authority for the
invocation; the dynamic nearer slot was created on the invocation context and
added the expected map-entry activity.

### Context escaped

The explicit `context`-escape case retained exactly one physical
`ProtosExecutionContextValue` per invocation, as required. This case is a
positive control for semantically required materialization.

## Guest-level timing control

To bound materiality on a real guest path rather than only a host-side direct
driver, two sources used the same recursive `repeat(10000, operation)` shape:

- control: callback returns `42` with no callback-local binding;
- treatment: callback creates an unused `local: 42` and returns `42`.

Measurement used the existing JFR-free `Perf010aTimingDriver`, one pinned CPU,
`-Xss128m`, five forks, 120 warmup iterations and 100 steady samples per fork.

Retained steady summaries:

```text
no-binding:
  fork medians ns =
    53358658.0,54703499.0,55156989.5,52361523.5,60788461.0
  steady median ns = 57689361.5
  MAD ns           = 7648441.0

unused-local:
  fork medians ns =
    62161742.5,75714393.0,74461740.0,72987057.0,73129786.5
  steady median ns = 73691918.0
  MAD ns           = 19562963.5

pairwise descriptive delta:
  delta / 10000 callbacks = 16002556.5 ns
  delta / callback        = 1600.256 ns
  ratio                   = 1.277392
  percent                 = +27.739%
```

Relative to the retained post-I068 `micro/closure-call` median
(`88019301.5 ns`), that descriptive delta is `18.181%`.

This is **not** an attributable frame-materialization fraction. The treatment
changes the complete static-binding runtime shape, which currently includes
frame materialization, frame-backed lexical-authority construction, and multiple
JDK collection/carrier allocations. The timing is also noisy, especially for the
treatment. The retained interpretation is only:

```text
STATIC_BINDING_RUNTIME_SHAPE_COST_MATERIAL=YES
```

## Compiler / PE evidence and limitation

Compiler diagnostics produced two materially different observations.

Successful semantic helper roots showed:

- no surviving context-class allocations in the inspected low-tier graph;
- one `FrameWithoutBoxing` represented as a virtual instance;
- no explicit new-instance nodes for the context package.

However, relevant ordinary `ProtosBytecodeRootNodeGen` compilations also
reported permanent/partial-evaluation failures, including:

```text
Too deep inlining, probably caused by recursive inlining
Partial evaluation did not reduce value to a constant ... Pi
```

Therefore source/JFR physical allocation survival must not be interpreted as
proof that the same objects necessarily survive optimized machine code on a
successfully compiled hot root. Conversely, successful semantic-helper PE does
not prove elimination on the ordinary roots that fail compilation.

```text
ORDINARY_HOT_ROOT_PE_SURVIVAL=INCONCLUSIVE
```

## Interaction with PLAT039

PLAT039 preliminary evidence independently identifies hot runtime-representation
and runtime-compilation boundary problems that materially overlap the physical
shape observed here:

- call/send preparation still exposes `ArrayList` / `List` carriers on hot
  guest paths;
- ordinary object/local-slot machinery still exposes `LinkedHashMap`-backed
  representation;
- broad boundaries around guest call/slot paths are rejected preliminarily;
- hot Java collection/carrier machinery should be split or replaced where
  required rather than hidden from PE;
- I068 remains directionally correct but did not complete the PE-friendly
  runtime representation.

The PERF010-A structural delta itself contains multiple `LinkedHashMap`,
`LinkedHashSet`, Map/List-copy and backing-array allocations around the
frame-backed lexical authority. This means the measured "static binding cost"
cannot be cleanly equated with `frame.materialize()` or with the lazy execution
context wrapper proposed by PLAT037 Candidate B.

Because the runtime implementation surfaces measured at
`f1cee2d858...` are unchanged at the PLAT039 preliminary revision
`d4ac1c7b...`, the two evidence sets are directly relevant to the same
implementation shape while retaining their different formal owners and
questions.

## Result

```text
EAGER_CONTEXT_WRAPPER_PHYSICAL_SURVIVAL=YES
STATIC_FRAME_AUTHORITY_PHYSICAL_SURVIVAL=YES
STATIC_FRAME_OBJECT_PHYSICAL_SURVIVAL=YES

STATIC_BINDING_RUNTIME_SHAPE_COST_MATERIAL=YES

FRAME_MATERIALIZE_ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
LAZY_CONTEXT_ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED

POST_I068_EAGER_CONTEXT_MATERIALIZATION_BIG_COST=INCONCLUSIVE
PLAT037_B_PERFORMANCE_JUSTIFICATION=INCONCLUSIVE
PLAT037_B_IMPLEMENTATION_SELECTED=NO

ORDINARY_HOT_ROOT_PE_SURVIVAL=INCONCLUSIVE
COMPILATION_FAILURE_CONFOUND=YES

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

## Scheduling consequence

Further decomposition of the current `~1.6 us/callback` static-binding delta
into `frame.materialize`, authority construction, individual JDK carriers, and
context-wrapper sub-costs is deferred.

The next higher-value step is to complete PLAT039's architecture decision. After
PLAT039 is ratified:

- if the selected implementation changes the measured hot carriers or PE
  boundary, implement and re-run this same discriminator against the new exact
  product revision before selecting PLAT037 Candidate B;
- if the selected architecture leaves these surfaces unchanged, PERF010-A may
  resume decomposition from this checkpoint.

No further production optimization is selected here.

## Maintained harness publication

The reusable subset of this investigation is now maintained in
`guillermomolina/protos-benchmarks` at exact revision:

```text
CONTEXT_MATERIALIZATION_HARNESS_REVISION=
  454e3abe764dcb47b3da4cdb1d332cc7c6e7fd9e
```

Published files/entry points:

```text
docker/protos-perf010a/Perf010aContextMaterializationProbe.java
runner/perf010a_context_materialization.py

make perf010a-context-materialization-validate
make perf010a-context-materialization-smoke
make perf010a-context-materialization-measure
```

The maintained runner deliberately preserves the current evidence boundary:
allocation decomposition uses `ThreadMXBean` with normal TLAB behavior and
guest timing uses the JFR-free `Perf010aTimingDriver`. It does not encode or
select a PLAT037/PLAT039 production optimization.

Human-executed publication validation reported:

```text
PERF010A_CONTEXT_MATERIALIZATION_VALIDATE=PASS
SMOKE=PASS
GIT_DIFF_CHECK=PASS
```

The smoke allocation output reproduced the expected shape:

```text
execution-context = 104.0272 B/op
ordinary-object   = 104.0 B/op
return-home       = 16.0072 B/op
```

The smoke timing is not retained as reference performance evidence because the
smoke contract intentionally uses only one fork and five steady samples.
