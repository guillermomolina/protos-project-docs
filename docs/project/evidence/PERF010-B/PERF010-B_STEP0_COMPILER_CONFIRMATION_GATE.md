# PERF010-B Step 0 — compiler confirmation gate result

Date: 2026-09-27

## Scope

This record closes only PERF010-B Step 0 from `guillermomolina/protos#722`.
It does not authorize Step 1, Step 2, Step 3A, or Step 3B and does not modify
Protos product code.

## Exact baseline

```text
PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=cf9b39b25dc9a3c4cd1c538749c3a363760ae45b
PROTOS_VERSION=0.3.96-SNAPSHOT

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
BENCHMARK_BASELINE_REVISION=06aa4f4af2476997b9eaf5c88c5df6ace02b8d40
BENCHMARK_DIAGNOSTIC_EXTENSION=UNCOMMITTED
```

The benchmark extension only parameterized the existing PERF010-A source-identity
diagnostic so that it could target `closure-call.protos`, retained the historical
method-call default, exposed observed source records on a failed exact-text match,
and added JVM warning-suppression options in the diagnostic image. The measured
product revision remained unchanged.

## Workload

Canonical `micro/closure-call` retained the shared driver:

```protos
repeat: (count, operation) => {
    (count > 0).ifTrue() {
        operation()
        repeat(count - 1, operation)
    }
}
```

The paired trivial control changed only:

```text
sink = identity(42)
```

to:

```text
sink = 42
```

Both paths retained the same `repeat`, `ifTrue`, recursion, callback,
operation count, CPU affinity, `-Xss128m`, warmup count 120, and steady count
100.

## Source identity

The source-identity instrument established:

```text
SOURCE=closure-call.protos

ROLE=ifTrue/repeat structured control
LINE=19
START_OFFSET=1058
END_OFFSET=1143

ROLE=operation-callback
LINE=20
START_OFFSET=1089
END_OFFSET=1100
TEXT=operation()

ROLE=recursive-repeat
LINE=21
START_OFFSET=1109
END_OFFSET=1137
```

## Compiler lifecycle

Exact source-correlated lifecycle:

```text
ROOT_ID=146
ROOT_CLASS=ProtosBytecodeRootNodeGen
SOURCE=closure-call.protos:20
TIER=1
OPT_DONE=0
OPT_FAILED=1
FAILURE=Partial evaluation did not reduce value to a constant
```

The complete failure stack places the concrete PE-constant failure at:

```text
CompilerAsserts.partialEvaluationConstant
  -> LocalRangeAccessor.isCleared
  -> ProtosFrameLexicalBindingAuthority.hasFrameBackedBindingAt
  -> ProtosBytecodeRootNode$ReadCapturedFrameLocal.perform
  -> ProtosBytecodeRootNodeGen...continueAt
  -> ProtosBytecodeRootNodeGen.execute
```

The paired semantic callback root compiled:

```text
ROOT_ID=147
ROOT_CLASS=ProtosSemanticBytecodeRootNodeGen
SOURCE=closure-call.protos:20
TIERS=1,2
OPT_DONE=3
OPT_FAILED=0
```

The source-correlated repeat semantic root also compiled:

```text
ROOT_ID=149
ROOT_CLASS=ProtosSemanticBytecodeRootNodeGen
SOURCE=closure-call.protos:19
TIERS=1,2
OPT_DONE=2
OPT_FAILED=0
```

Two associated Bytecode roots failed independently:

```text
ROOT_ID=148
ROOT_CLASS=ProtosBytecodeRootNodeGen
TIER=1
OPT_DONE=0
OPT_FAILED=1
FAILURE=PermanentBailoutException: Too deep inlining, probably caused by recursive inlining.

ROOT_ID=150
ROOT_CLASS=ProtosBytecodeRootNodeGen
TIER=1
OPT_DONE=0
OPT_FAILED=1
FAILURE=PermanentBailoutException: Too deep inlining, probably caused by recursive inlining.
```

The retained inlining evidence for these failures includes
`ProtosFrameLexicalBindingAuthority.<init>`,
`InstallFrameLexicalAuthority.perform`, `continueAt`, and Bytecode-root
execution, but the explicit bailout reason is `Too deep inlining`.

## JIT versus interpreter

Median of 100 steady samples:

```text
WORKLOAD_JIT_MEDIAN_NS=95572944
WORKLOAD_INTERPRETER_MEDIAN_NS=93588376
WORKLOAD_JIT_OVER_INTERPRETER=1.021205
WORKLOAD_JIT_REDUCTION_PERCENT=-2.121

CONTROL_JIT_MEDIAN_NS=63400718
CONTROL_INTERPRETER_MEDIAN_NS=88069532
CONTROL_JIT_OVER_INTERPRETER=0.719894
CONTROL_JIT_REDUCTION_PERCENT=28.011

JIT_WORKLOAD_OVER_CONTROL=1.507443
JIT_EXTRA_NS=32172226

INTERPRETER_WORKLOAD_OVER_CONTROL=1.062665
INTERPRETER_EXTRA_NS=5518844
```

The common driver therefore benefits materially from compilation in the trivial
control, while adding the guest call removes that benefit. The workload itself is
classified as `JIT ~= INTERPRETER`.

## Step-0 gate

```text
PERF010B_STEP0=PASS

HOT_ROOT_COMPILATION=OPT_FAILED

FAILURE_CAUSE=
  OTHER:
    PE_CONSTANT_FAILURE_IN_CAPTURED_FRAME_LOCAL_PATH
    TOO_DEEP_INLINING

JIT_VS_INTERPRETER=
  WORKLOAD: JIT ~= INTERPRETER
  TRIVIAL_CONTROL: JIT << INTERPRETER

STEP_1=NOT_APPLICABLE
```

Step 1 in #722 was conditional on an exact frame/VirtualFrame/materialization
failure. That discriminator was not observed. The callback Bytecode root instead
failed a PE-constant requirement in the captured-frame-local path, while other
Bytecode roots failed from recursive inlining depth.

## Causal interpretation

The experiment falsifies the broad explanation that the common
`repeat`/`ifTrue` driver simply cannot benefit from Truffle compilation: the
trivial control is 28.011% faster with normal compilation than with compilation
disabled.

The regression is localized more tightly to the guest-call-bearing path. Its
semantic roots compile through Tier 2, but required Bytecode roots do not all
converge to successful optimized code. This is a partial/asymmetric compilation
failure, not a global failure to compile the driver.

The evidence does not establish that frame materialization is the dominant cause,
nor that `Too deep inlining` alone explains the timing delta. It establishes the
two concrete compiler failures that the next investigation must discriminate.

## Next action

Do not execute the current Step 1.

Before entering Step 2 as a product implementation, perform a bounded
investigation of the two observed compiler failures against the exact current
source:

1. determine why `LocalRangeAccessor.isCleared()` cannot be PE-constant on the
   `ReadCapturedFrameLocal` path and whether this is caused by Protos'
   captured-binding representation/order or by an unavoidable dynamic property;
2. identify the exact recursive expansion responsible for roots 148/150 reaching
   `Too deep inlining`;
3. compare the relevant implementation shape with mature Truffle language
   implementations where equivalent closure/captured-local calls compile;
4. establish which failure lies on the measured guest-call increment and whether
   removing it is a prerequisite to, orthogonal to, or naturally solved by #722
   Step 2.

No product implementation is authorized by this record.
