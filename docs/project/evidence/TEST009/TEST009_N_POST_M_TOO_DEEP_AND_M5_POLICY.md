# TEST009-N — post-M TooDeep classification and M5 policy

Date: 2026-10-05

Owning work item: `TEST009 / guillermomolina/protos#795`

Triggering product revision:

~~~text
PROTOS_REVISION=9c28dee1199840e6fcaa4fc914d115c6c90a37bd
COMMIT_SUBJECT=TEST009-M: amortized post-K compiler-expansion cleanup batch
IMPLEMENTATION_VERSION=0.3.216-SNAPSHOT
~~~

## Purpose

TEST009-N was a read-only causal investigation. It did not modify product code,
run another Truffle diagnostic, or authorize a new permanent-failure family.

The post-M single-Case acquisition was:

~~~text
CASE_DISPLAY=protos/corpus/conformance:call/closure-call-and-return.protos::plain closure call
SHARD_WORKERS=1

COMPILATIONS_DONE=194
COMPILATION_FAILURES=36
PERFORMANCE_WARNINGS=4877
PE_CONSTANT_FAILURES=0

CODE_INSTALLATION_TOO_LARGE=21
TOO_DEEP_INLINING=15
COMPILER_OOM=0
~~~

Compared with the normalized post-K acquisition:

~~~text
PERFORMANCE_WARNINGS~=19988 -> 4877
CODE_INSTALLATION_TOO_LARGE~=23 -> 21
COMPILER_OOM=1 -> 0
TOO_DEEP_INLINING=1 -> 15
PE_CONSTANT_FAILURES=0 -> 0
~~~

The 21 code-installation-too-large findings remained the residual of an existing
family. The new regression requiring policy work was the increase in
TooDeepInlining.

## Causal classification

TEST009-N retained TEST009-M M1-M4. No source-level mechanism was found that
required reopening those cuts.

The M5 exact-native-body PIC was classified as the proximate cause of the newly
visible host/JDK graphs: an exact `ProtosNativeClosureBody` becomes PE-constant,
so partial evaluation can descend through host-heavy work that the prior generic
native dispatch did not expose as deeply.

The 15 post-M TooDeep findings were grouped into three observed shared seams:

1. host String/Charset/JDK leaves reached from specialized native bodies;
2. Context-local C-prime plan cache-miss construction reaching Bytecode DSL
   `ConstantsBuffer` machinery;
3. physical NIO resource close reaching JDK file-lock / map machinery.

## Policy selected for TEST009-O

The source evidence supported Candidate A as the next falsifiable experiment:

~~~text
M5_POLICY=KEEP_GLOBAL_PIC
M1_M4=KEEP
CODE_TOO_LARGE_ACTION=DEFER_UNTIL_M5_STABLE
~~~

The selected repair shape was deliberately narrow:

- keep guest/runtime control and validation PE-visible;
- bound only demonstrated host transformation leaves;
- retain the visible fast path for already-built Context-local C-prime plans and
  bound only lazy cache-miss construction;
- retain Protos I/O lifecycle state and bound only the physical host close.

No native-body eligibility marker or global native-body boundary was authorized.

## Falsification condition

Candidate A was explicitly provisional. The next final diagnostic had to satisfy
monotonic compilerability:

~~~text
NEW_PERMANENT_FAILURE_FAMILY_ALLOWED=NO
PE_CONSTANT_FAILURES=0
COMPILER_OOM=0
TOO_DEEP_INLINING<=1
CODE_INSTALLATION_TOO_LARGE=NO_MATERIAL_REGRESSION
~~~

Performance-warning count remained secondary telemetry.

If TEST009-O exposed additional unrelated M5-opened permanent families, the
global-PIC policy was to be reconsidered instead of continuing an unbounded
boundary-by-boundary patch loop.

## Next slice selected at N

~~~text
NEXT_SLICE=TEST009-O
NEXT_SLICE_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
NEXT_SCOPE=BOUND_THREE_SHARED_POST_M_HOST_EXPANSION_SEAMS
EXPENSIVE_DIAGNOSTICS_PER_SLICE_MAX=1
DIAGNOSTIC_POINT=END_OF_BATCH_ONLY
~~~

No observable Protos semantic change and no specification change were
authorized.

AI assistance: this durable record was drafted with ChatGPT from the published
TEST009-M revision, its retained compiler diagnostic evidence, the current
source architecture reviewed during TEST009-N, and the owner-selected TEST009-O
execution that followed.
