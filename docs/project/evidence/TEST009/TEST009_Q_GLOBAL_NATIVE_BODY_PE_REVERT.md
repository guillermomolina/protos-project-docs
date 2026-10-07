# TEST009-Q — global native-body PE specialization revert

Date: 2026-10-06

Owning work item: `TEST009 / guillermomolina/protos#795`

Implementation repository: `guillermomolina/protos`

This record captures the published TEST009-Q implementation selected by the
TEST009-P architecture investigation and the maintainer-supplied post-Q Truffle
diagnostic. It is non-normative project evidence and does not replace live
GitHub coordination.

## Exact published product identity

~~~text
PROTOS_REVISION=206bbde5593e4eea414b695047d07b894e201fbc
COMMIT_SUBJECT=TEST009-Q: revert global native-body PE specialization
PROTOS_VERSION=0.3.224-SNAPSHOT
~~~

Changed product paths:

- `CHANGELOG.md`;
- `pom.xml`;
- `src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java`;
- `src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java`;
- `src/test/java/com/guillermomolina/protos/execution/ProtosI072PhaseDPreparedCallSeparationTest.java`.

## Implemented policy

TEST009-P established that the heterogeneous `ProtosNativeClosureBody`
population has no maintainable diagnostic-independent structural authority that
means “this native body is intended to be visible to guest partial evaluation”.
P therefore selected a full causal revert of TEST009-M M5 rather than a new
manual marker/whitelist or another host-boundary chase.

Q implements that selection exactly.

~~~text
GLOBAL_NATIVE_BODY_PIC_AS_DEFAULT_POLICY=REVERTED
ARCHITECTURAL_PE_ELIGIBILITY_MARKER_ADDED=NO
WHITELIST_ADDED=NO
BOUNDARY_CHASING_ADDED=NO
~~~

Specifically:

- `EnterClosureCall.nativeDirect` is removed from both
  `ProtosBytecodeRootNode` and `ProtosSemanticBytecodeRootNode`;
- both generated-interpreter authorities again expose one generic
  `nativeCall(NativeCall)` specialization for native entry;
- M5-only `NativeCall.nativeBody()` and
  `NativeCall.enterNativeBody(body)` helpers are removed;
- the ordinary-versus-suspension-capable rule again has one owner in
  `NativeCall.enterNative()`;
- structured-dispatch rejection, Task/deferred-C-prime admission and
  `ProtosSuspensionCapableNativeClosureBody` continuation entry are retained;
- TEST009-M M1-M4 are retained;
- TEST009-O Encoding, C-prime plan-cache-miss and physical-close host
  boundaries are retained;
- `ProtosI072PhaseDPreparedCallSeparationTest` now requires exactly one
  `NativeCall` entry specialization with parameter list `[NativeCall]`.

No observable Protos language semantics are intentionally changed and no
specification change is required.

## Maintainer-reported validation

The maintainer reports after publication:

~~~text
MAINTAINER_REPORTED_GIT_DIFF_CHECK=PASS
MAINTAINER_REPORTED_ALL_LOCAL_TESTS=PASS
PUBLICATION=PUSHED
~~~

These are maintainer-reported validation results, not independently re-executed
by the agent.

## Post-Q Truffle diagnostic provenance

The supplied diagnostic command selected one logical Case:

~~~text
CORPUS=protos/corpus/conformance
FILE=call/closure-call-and-return.protos
CASE=plain closure call
SHARD_WORKERS=1
~~~

The diagnostic log reports Maven build version `0.3.222-SNAPSHOT`, while the
published Q commit carries the final `0.3.224-SNAPSHOT` publication metadata.
Accordingly, this record treats the diagnostic as evidence from the post-Q
implementation code before the final version/changelog publication edits; it
does **not** claim that the diagnostic executed the literal final Q SHA after
those metadata-only edits.

The semantic workload itself passed:

~~~text
SEMANTIC_CORPUS=PASS
CORPUS_PASSED=1
CORPUS_FAILED=0
~~~

The compilerability diagnostic remained red:

~~~text
COMPILATIONS_DONE=217
COMPILATION_FAILURES=23
SHUTDOWN_CASCADE_FAILURES=0
PERFORMANCE_WARNINGS=5775
PE_CONSTANT_FAILURES=0
OTHER_PERMANENT_FAILURES=23
DIAGNOSTIC_WALL_SECONDS=927.9
TRUFFLE_COMPILATION_DIAGNOSE=FAIL
~~~

All 23 permanent failures are accounted for by the supplied output:

~~~text
CODE_INSTALLATION_TOO_LARGE=22
TOO_DEEP_INLINING=1
OTHER_PERMANENT_FAILURE_CLASS=0
~~~

The single remaining TooDeep failure is a
`ProtosBytecodeRootNodeGen` compilation; the other 22 are code-installation
“code is too large” failures across Bytecode/Semantic Bytecode roots.

## Comparison with TEST009-O

The retained TEST009-O diagnostic was:

| Metric | TEST009-O | Post-Q |
| --- | ---: | ---: |
| Compilations done | 211 | 217 |
| Compilation failures | 40 | 23 |
| Performance warnings | 5758 | 5775 |
| PE-constant failures | 0 | 0 |
| Code installation too large | 30 | 22 |
| TooDeep inlining | 8 | 1 |
| Compilation exceeded 100 seconds | 2 | 0 permanent failures of that class |
| Diagnostic wall seconds | 1500.6 | 927.9 |

The Q result is therefore consistent with the causal policy selected by P:
removing the global exact-native-body PIC materially retracts compiler expansion
debt. TooDeep falls from 8 to 1, code-too-large falls from 30 to 22, total
permanent failures fall from 40 to 23, and diagnostic wall time falls from
1500.6 s to 927.9 s.

This does **not** make the strict compilerability gate green. Performance
warnings remain essentially unchanged/slightly higher, and 23 permanent
compiler failures remain. Q therefore closes the M5 policy correction, not
TEST009 as a whole.

## Systematic-method consequence

The post-Q failures are evidence to classify, not automatic authorization for
another boundary batch.

The TEST009 systematic invariant remains:

~~~text
DIAGNOSTIC_FAILURE != AUTOMATIC_BOUNDARY_PRESCRIPTION
BOUNDARY_CHASING_ALLOWED=NO
CODE_TOO_LARGE_MICRO_REPAIR_BATCH_ALLOWED=NO
~~~

No new `@TruffleBoundary` batch, native-body marker, exact-body cache,
whitelist, or code-size micro-repair is selected by this record.

## State and next slice

TEST009 remains open and in progress.

The next slice is the read-only analysis already anticipated by the Q
CHANGELOG entry:

~~~text
TEST009_Q=COMPLETE
M5_GLOBAL_NATIVE_BODY_PIC=REVERTED
M1_M4=RETAINED
O_ENCODING_BOUNDARY=RETAINED
O_CPRIME_PLAN_BOUNDARY=RETAINED
O_PHYSICAL_CLOSE_BOUNDARY=RETAINED

SEMANTIC_CORPUS=PASS
STRICT_COMPILERABILITY=STILL_RED

TEST009_COMPLETE=NO

NEXT_SLICE=TEST009-R
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_SCOPE=POST_Q_RESIDUAL_COMPILERABILITY_CLASSIFICATION
IMPLEMENTATION_AUTHORIZED=NO
EXPENSIVE_DIAGNOSTIC_RERUN_REQUIRED=NO
NEW_FORMAL_ISSUE_REQUIRED=NO
~~~

R should consume the already-acquired post-Q evidence and determine the causal
classification of the remaining 22 code-size failures, one TooDeep failure and
the performance-warning population under the established systematic procedure.
It must not begin by adding boundaries or rerunning the expensive diagnostic.

AI assistance: this durable record was drafted with ChatGPT from the exact Q
product commit, the previously published TEST009-P/O evidence, the
maintainer-reported validation state, and the supplied post-Q diagnostic output.
