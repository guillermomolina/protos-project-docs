# TEST009-L — post-K compiler-expansion investigation

Date: 2026-10-05

Owning work item: `TEST009 / guillermomolina/protos#795`

Trigger/consumer: `PERF030 / guillermomolina/protos#784`

## Exact analyzed product state

~~~text
PROTOS_REVISION=3c9738f5835cc8ed2e43d50fb8edcc7eb956ddc9
COMMIT_SUBJECT=TEST009-K: cut remaining compiler expansion debt
IMPLEMENTATION_VERSION=0.3.213-SNAPSHOT
~~~

TEST009-L was a read-only investigation. It executed no project/runtime command
and made no product change. Afterward, the maintainer reported the current local
test suite PASS.

~~~text
MAINTAINER_REPORTED_LOCAL_TESTS=PASS
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

## Input evidence and normalization

TEST009-K's final diagnostic selected the three logical Cases from
`protos/tests/conformance/call/closure-call-and-return.protos`.

Each Case produced essentially the same result:

~~~text
SEMANTIC_CORPUS=PASS
CORPUS_PASSED=1
CORPUS_FAILED=0
COMPILATIONS_DONE=214
COMPILATION_FAILURES=25
PERFORMANCE_WARNINGS=19988
PE_CONSTANT_FAILURES=0
OTHER_PERMANENT_FAILURES=25
~~~

The aggregate 75 permanent failures are therefore approximately three copies of
the same 25 findings, not 75 independent defect families.

Stable per-Case residual:

~~~text
CODE_INSTALLATION_TOO_LARGE~=23
COMPILER_OUT_OF_MEMORY_ERROR=1
NEW_TOO_DEEP_INLINING=1
~~~

TEST009-K's owned repairs remain closed:

~~~text
K1 original TextReader recursive queue handoff = FIXED
K2 residual lexical write fallback expansion = FIXED
K3 Polyglot Context.getCurrent expansion = FIXED
K4 Context-local Bytecode plan CHM lookup expansion = FIXED
K1_K4_REOPEN_REQUIRED=NO
~~~

## Causal map

### Lexical authority

The dominant retained interface calls are
`ProtosLexicalBindingAuthority.readBinding`, `containsBinding`,
`putBinding` and `appendBindingsTo`.

The frame-backed implementation already cuts PE-unsafe LocalRangeAccessor work
with narrow boundaries. The remaining expansion opportunity is primarily
caller-side interface dispatch in the generic/local-slot facades:

~~~text
ProtosObjectValue local-slot facade
ProtosActivation residual current-context lexical facade
ProtosActivation context observation / authority handoff
LEXICAL_AUTHORITY_CAUSAL_CLASS=B+D
~~~

The next implementation must leave ordinal/frame-native paths intact and must
not reopen TEST009-K's already bounded residual lexical fallback.

### Represented values

`ProtosValueLookup.delegationParent` contains a generic
`ProtosRepresentedValue.representedDelegationParent` branch.

Production currently has thirteen represented-value implementations with
heterogeneous parent rules. Boolean and Integer already have family-specific
guarded fast paths. A thirteen-type `instanceof` ladder is therefore rejected.

Selected repair shape:

~~~text
retain Boolean/Integer guarded paths
isolate only the generic represented-value parent projection
REPRESENTED_VALUE_CAUSAL_CLASS=B+D
~~~

### Native-body dispatch

`ProtosBytecodeRootNode.NativeCall` stores a `ProtosNativeClosureBody` and
ultimately calls it through `NativeCall.enterNative()`.

TEST009-J already separated structured dispatch. A global boundary around every
native body would discard useful monomorphic specialization. The selected
repair is instead a small body-identity inline cache at the `EnterClosureCall`
native entry, with one generic fallback after cache miss/megamorphism, while
preserving suspension-capable dispatch.

~~~text
NATIVE_BODY_CAUSAL_CLASS=A+B
~~~

### Host collections

Retained warnings for `List.size`, `List.get`, `List.isEmpty` and
`Collection.toArray` come from several independent carriers, including
`NativeCall.supplied`, `PreparedArgumentVector`, deferred supplied arguments
and structured-call snapshots/callback carriers.

No single representation rewrite is sufficiently source-proven now.

~~~text
HOST_COLLECTION_CAUSAL_CLASS=D_MULTI_SOURCE
HOST_COLLECTION_REPAIR=DEFER
~~~

## Compiler OOM

The OOM occurs in the same generated semantic-root family that owns almost all
code-too-large failures:

~~~text
java.lang.OutOfMemoryError:
Required array length 1342177280 + 1342177280 is too large

COMPILER_OOM_RELATION=SAME_GRAPH_EXPLOSION
HEAP_TUNING_REPAIR=REJECTED
~~~

The next batch should first reduce the shared graph expansion. Surviving OOM
after those repairs would be new evidence; it is not a reason to increase heap
or relax the compilerability gate now.

## New TooDeepInlining

The new post-K TooDeepInlining is not the retired K1 queue recursion.

The retained chain reaches `ProtosTextReader.scanLine` once and then repeatedly
expands a JDK String bounds/error path through StringBuilder,
AbstractStringBuilder, Preconditions, Formatter, Locale, reflection and
String.substring.

The narrow source seam is:

~~~text
line.append(unit.text())
~~~

Selected repair:

~~~text
tiny @TruffleBoundary helper -> StringBuilder.append(text)
~~~

Only the append is cut. TextReader framing, CR/LF handling, byte limits, decoder
state and terminal-result control remain PE-visible.

## Selected implementation batch

~~~text
NEXT_SLICE=TEST009-M
NEXT_SLICE_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

M1=TextReader StringBuilder cold-path cut
M2=ProtosObjectValue lexical-authority caller cut
M3=ProtosActivation residual lexical/observation caller cut
M4=ProtosValueLookup represented-value generic fallback cut
M5=NativeCall native-body identity specialization plus generic fallback
~~~

Expected primary production files:

~~~text
src/main/java/com/guillermomolina/protos/runtime/ProtosTextReader.java
src/main/java/com/guillermomolina/protos/runtime/ProtosObjectValue.java
src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java
src/main/java/com/guillermomolina/protos/runtime/ProtosValueLookup.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
~~~

Explicit deferrals/rejections:

~~~text
HOST_COLLECTION_REPRESENTATION_REWRITE=DEFER
REPRESENTED_VALUE_13_TYPE_LADDER=REJECT
NATIVE_BODY_PER_METHOD_SPECIALIZATION=REJECT
GLOBAL_NATIVE_BODY_BOUNDARY=REJECT
COMPILER_HEAP_INCREASE=REJECT
CODE_SIZE_LIMIT_RELAXATION=REJECT
STRICT_GATE_RELAXATION=REJECT
K1_K4_REOPEN=REJECT
BUG016_REOPEN=REJECT
~~~

## Validation policy for TEST009-M

Cheap focal/static validation is allowed throughout implementation. The ordinary
full functional suite runs after the complete M1-M5 batch is integrated.

No expensive Truffle diagnosis may run between repairs.

~~~text
INTERMEDIATE_EXPENSIVE_DIAGNOSTIC=FORBIDDEN
EXPENSIVE_DIAGNOSTICS_PER_SLICE_MAX=1
DIAGNOSTIC_POINT=END_OF_BATCH_ONLY
~~~

Suggested focal owners:

~~~text
ProtosTextReaderLineProtocolTest
ProtosLexicalBindingAuthoritySeamTest
ProtosI068Slice7ActivationLexicalDecompositionTest
ProtosI075DLexicalAuthorityCurrentBytecodeNodeTest
ProtosRepresentedValueLookupTest
ProtosPerf006B6A4OrdinaryNativeFastPathTest
ProtosPerf006B3CSuspensionCapableNativeLeafTest
existing LocalRange/Bytecode PE guards
~~~

## Sole expensive diagnostic

The final diagnostic should select one exact logical Case:

~~~text
FILE=protos/tests/conformance/call/closure-call-and-return.protos
CASE_DISPLAY=protos/corpus/conformance:call/closure-call-and-return.protos::plain closure call
SHARD_WORKERS=1
CASES_EXECUTED=1
DIAGNOSTIC_OPTIONS=UNCHANGED
~~~

The D185 CaseRef must be obtained from current HEAD's `protos test --list-cases`
output immediately before the diagnosis. The CaseRef is opaque and must not be
synthesized or decoded.

One Case is sufficient because TEST009-K showed essentially the same findings in
all three Cases, while BUG016 measured that the dominant fixed compilation
chain occurs before the selected Case body.

## Closure

~~~text
TEST009_L_INVESTIGATION=COMPLETE
PRODUCT_CHANGE_IN_L=NONE
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
TEST009_COMPLETE=NO
NEXT_SLICE=TEST009-M
NEXT_SLICE_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
NEW_FORMAL_ISSUE_REQUIRED=NO
~~~

AI assistance: this durable record was drafted with ChatGPT from the exact
published product revision, retained TEST009-K diagnostic evidence, current
GitHub source inspection and maintainer-reported local validation.
