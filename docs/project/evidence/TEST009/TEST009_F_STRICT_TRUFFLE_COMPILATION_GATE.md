# TEST009-F — strict Truffle compilation gate publication

Evidence date: **2026-10-04**

Owning work item: `TEST009 / guillermomolina/protos#795`

Trigger: `PERF030 / guillermomolina/protos#784`

## Exact publication

~~~text
SLICE=TEST009-F
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

PROTOS_REVISION=45a6df118b83396f742667d05c1797ee2843e7e0
COMMIT_SUBJECT=TEST009-F: add strict Truffle compilation gate

PRODUCT_JAVA_CHANGE=NO
SPECIFICATION_CHANGE=NO
CHANGELOG_CHANGE=NO
POM_VERSION_CHANGE=NO
~~~

## Published surfaces

TEST009-F adds:

~~~text
tools/truffle_jvm_launch.py
tools/truffle_compilation_gate.py
tools/test_truffle_compilation_gate.py
~~~

and minimally refactors:

~~~text
tools/java_generated_bytecode_bci_compilation_check.py
~~~

to reuse the generic JVM launch/parsing machinery without changing the
family-specific TEST009-D ownership model.

The Makefile now exposes:

~~~text
make check-truffle-compilation
make diagnose-truffle-compilation
~~~

The strict gate runs the real Protos Test Tool corpus through the packaged JVM
runtime in two modes:

~~~text
TRUFFLE_COMPILATION_SYNC
TRUFFLE_COMPILATION_BACKGROUND
~~~

Both use immediate Truffle compilation, fatal compilation failures and
performance warnings as compilation errors. The diagnostic target is manual
escalation only and emits expansion/inlining/performance-warning traces without
mutating product code.

## Strict-gate validation

The maintainer executed the gate after publication preparation and reported that
both modes fail closed on a real product compilerability problem.

Observed SYNC:

~~~text
PRIMARY_COMPILATION_FAILURES=2
FAILURE_CLASS=PERFORMANCE_WARNING_TREATED_AS_COMPILATION_ERROR
ROOT_FAMILY=ProtosSemanticBytecodeRootNodeGen
SECONDS_TO_ABORT=10.6
RESULT=FAIL
~~~

Observed BACKGROUND:

~~~text
PRIMARY_COMPILATION_FAILURES=6
SHUTDOWN_CASCADE_FAILURES=1043
FAILURE_CLASS=PERFORMANCE_WARNING_TREATED_AS_COMPILATION_ERROR
ROOT_FAMILY=ProtosSemanticBytecodeRootNodeGen
SECONDS_TO_ABORT=15.7
RESULT=FAIL
~~~

The `DestroyedIsolateException` records are shutdown cascade after the
ExitVM-triggering primary failure. TEST009-F classifies those separately:

~~~text
SHUTDOWN_CASCADE_FAILURES
~~~

They remain visible in retained logs/reports, are not counted as independent
product compilation failures, and still fail closed if they ever occur without
a primary failure.

Across the reported strict runs:

~~~text
PE_CONSTANT_FAILURES=0
OTHER_PERMANENT_FAILURES=0
PRIMARY_WARNING_KIND=call
OPT_DONE=0
CORPUS_SUMMARY_REACHED=NO
~~~

No successful compilation or final Test Tool summary is expected after ExitVM
terminates the process on the first fatal warning.

## Dominant product-warning families

Reported unresolved-call warning families include:

~~~text
ProtosLexicalBindingAuthority.readBinding
ProtosLexicalBindingAuthority.containsBinding
ProtosLexicalBindingAuthority.putBinding
ProtosLexicalBindingAuthority.appendBindingsTo

ProtosBytecodeRootNode$PreparedClosureCall.*
  including isNative
  requiresStructuredDispatch
  finish
  complete
  bodyTarget
  and related operations

ProtosRepresentedValue.representedDelegationParent

AbstractPolyglotImpl.getCurrentContext
ConcurrentHashMap$Node.find
generic List / Collection / Map operations
ProtosBytecodeRootNode lambdas
~~~

This evidence does not determine one universal remedy. In particular, it does
not authorize automatic `@TruffleBoundary` placement.

## Dump-path correction

The first strict run exposed root-level Graal dump output. TEST009-F was corrected
before publication to use:

~~~text
-Djdk.graal.DumpPath=target/truffle-compilation/graal_dumps
~~~

for strict and diagnostic runs.

The maintainer reports:

~~~text
TARGET_GRAAL_DUMPS=YES
ROOT_LEVEL_GRAAL_DUMPS=ABSENT
~~~

## Validation

The maintainer reports the final TEST009-F self-test suite at:

~~~text
SELF_TESTS=26
SELF_TEST_RESULT=PASS
~~~

The strict gate itself is expected to remain FAIL on the current product because
it has discovered a real compilerability problem.

No clean full-corpus runtime can yet be measured: strict ExitVM aborts during
early compilation. Therefore TEST009-F remains an individually runnable target
and is not added to aggregate `make check` yet.

## Closure classification

TEST009-F is complete as detector infrastructure and blocked only in the sense
that the current product is not compiler-clean.

~~~text
TEST009_F=COMPLETE
TEST009_F_STATE=DETECTOR_COMPLETE_CLEANUP_BLOCKED

STRICT_GATE_IMPLEMENTED=YES
SYNC_MODE_IMPLEMENTED=YES
BACKGROUND_MODE_IMPLEMENTED=YES
PERFORMANCE_WARNINGS_AS_ERRORS=YES
COMPILATION_FAILURES_FATAL=YES
DIAGNOSTIC_TARGET=AVAILABLE
DUMP_PATH_UNDER_TARGET=YES

MAKE_CHECK_AGGREGATION=DEFERRED
CLEAN_FULL_CORPUS_RUNTIME_MEASUREMENT=DEFERRED

GATE_RELAXATION=NO
AUTOMATIC_BOUNDARY_MUTATION=NO
PRODUCT_REPAIR_INSIDE_F=NO
~~~

## Next work

The next TEST009 slice is investigation only:

~~~text
NEXT_SLICE=TEST009-G
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_SCOPE=CAUSAL_DIAGNOSIS_OF_STRICT_GATE_CALL_WARNINGS
IMPLEMENTATION_AUTHORIZED=NO
NEW_FORMAL_ISSUE_REQUIRED=NO
~~~

The investigation must use the strict-gate retained evidence plus the textual
diagnostic surfaces to determine the causal owners of the unresolved-call
warnings before any product repair is selected.

AI assistance: this durable record was drafted with ChatGPT from the exact
published TEST009-F commit and maintainer-reported validation evidence.
