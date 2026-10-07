# TEST009-J — prepared structured-dispatch specialization

Date: 2026-10-05

Owning work item: `TEST009 / guillermomolina/protos#795`

Trigger/consumer: `PERF030 / guillermomolina/protos#784`

## Exact publication

~~~text
PROTOS_REVISION=a153b4198da1c5d74be857cb7e24a11b1f7c53c6
PARENT_REVISION=28d057d61f40be32a215b15e2ee463700709028c
COMMIT_SUBJECT=TEST009-J: specialize prepared structured dispatch
IMPLEMENTATION_VERSION=0.3.205-SNAPSHOT
~~~

## Scope

TEST009-J removes the remaining generic `PreparedClosureCall` receiver from
the structured-dispatch selection/preparation family in both generated
Bytecode interpreters.

The closed prepared-call leaf set remains:

~~~text
OrdinarySourceCall
NativeCall
ImmediateResultCall
ModuleInitializationCall
~~~

`NativeCall` retains the real structured capabilities. The other three
representations retain the same absent capability answer and the same
`IllegalStateException` preparation behavior.

The repaired family includes:

~~~text
RequiresStructuredDispatch
IsStructured*
PrepareStructured*
~~~

The semantic interpreter also specializes the inline-vs-structured admissions:

~~~text
AdmitsInlineLiteralWhile
AdmitsInlineLiteralIndexedEach
AdmitsInlineLiteralTwoParameterEach
~~~

Those operations are part of the same causal family: leaving them generic
would preserve `PreparedClosureCall` type erasure at the semantic
inline/structured selection point.

`EnterNestedStructuredDispatch` is now specialized to `NativeCall` only.
Both lowerers reach that operation only after a positive
`RequiresStructuredDispatch` decision, and only `NativeCall` owns that
capability.

`ResumeContinuation` is deliberately outside TEST009-J.

## Structural regression protection

`ProtosI072PhaseDPreparedCallSeparationTest` now guards:

- no J-owned DSL specialization receives generic `PreparedClosureCall`;
- the closed set of four concrete prepared-call representations;
- structured capability ownership remains exclusive to `NativeCall`;
- non-structured representations acquire no structured fields/state;
- their false / `IllegalStateException` behavior is preserved;
- nested structured dispatch receives `NativeCall` only;
- `ResumeContinuation` remains the explicit pending generic boundary.

## Validation

Maintainer-reported validation before metadata publication:

~~~text
MAVEN_TEST_COMPILE=PASS
TRUFFLE_DSL_ANNOTATION_PROCESSOR=PASS
JAVA_FOCAL_TESTS=PASS
PROTOS_TESTS=1328 passed, 0 failed
PROTOS_TESTS_TOTAL_TIME=31 s
~~~

The post-J focal Truffle textual diagnostic ran the selected
`closure-call-and-return.protos` corpus with `--jobs 8`:

~~~text
TRUFFLE_COMPILATION_DIAGNOSE=FAIL
TRUFFLE_COMPILATION_DIAGNOSE_SECONDS=721.7
SEMANTIC_CORPUS=PASS
CORPUS_PASSED=3
CORPUS_FAILED=0
COMPILATIONS_DONE=171
COMPILATION_FAILURES=22
SHUTDOWN_CASCADE_FAILURES=0
PERFORMANCE_WARNINGS=19031
PE_CONSTANT_FAILURES=0
OTHER_PERMANENT_FAILURES=22
~~~

The J-owned generic structured-dispatch warnings are absent from the retained
diagnostic:

~~~text
PreparedClosureCall.requiresStructuredDispatch=0
PreparedClosureCall.isStructured*=0
PreparedClosureCall.prepareStructured*=0

PreparedClosureCall.activation=0
PreparedClosureCall.handleControlTransfer=0
PreparedClosureCall.failIfModuleInitialization=0
PreparedClosureCall.mapRuntimeFailure=0
~~~

The remaining `PreparedClosureCall` warning observed in this diagnostic is
`taskForRuntime`, attributable to the deliberately deferred
`ResumeContinuation` path rather than TEST009-J structured dispatch.

## Global compilerability state

TEST009-J closes its causal family but does not claim the global gate is green.

The post-J focal diagnostic reports:

~~~text
CODE_INSTALLATION_TOO_LARGE=21
TOO_DEEP_INLINING=1
GLOBAL_COMPILATION_FAILURES=22
GLOBAL_CHECK_STATE=RED_ON_OTHER_DEBT
~~~

The pre-J run used a different Test Tool parallelism setting, so the change in
raw bailout count is not treated as a controlled A/B performance measurement.

No additional expensive diagnosis is required to close J.

## Semantics

~~~text
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
NEW_TRUFFLE_BOUNDARY=NO
NEW_COMPILATION_FINAL=NO
NEW_ASSUMPTION=NO
TEST009_J_CAUSAL_RESULT=PASS
~~~

The metadata bump to `0.3.205-SNAPSHOT` occurred only after validation. Under
the repository publication rule, no tests were rerun after the
`pom.xml` / `CHANGELOG.md` update.

## Follow-up

`ResumeContinuation / PreparedClosureCall.taskForRuntime` remains a small
separate TEST009 residue.

Separately, the TEST009-J diagnostic exposed that increasing Test Tool
`--jobs` from 1 to 8 did not reduce the approximately twelve-minute focal
diagnostic wall time. That observation is not folded into TEST009-J; it is
routed to the Test Tool parallel-execution owner/defect follow-up.

AI assistance: this durable record was drafted with ChatGPT from the exact
published revision and maintainer-reported validation evidence.
