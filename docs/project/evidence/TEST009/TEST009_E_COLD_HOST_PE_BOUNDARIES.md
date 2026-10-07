# TEST009-E — cold host work moved out of partial evaluation

Evidence date: **2026-10-04**

Owning work item: `TEST009 / guillermomolina/protos#795`

Trigger: `PERF030 / guillermomolina/protos#784`

Nature: immutable publication evidence for the TEST009-E product cleanup that
moves demonstrated cold host-side work out of Truffle partial evaluation.

## Exact publication

~~~text
SLICE=TEST009-E
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

PROTOS_REVISION=176d76ffbf2974316249fa5eb0e47fb6c89e8769
COMMIT_SUBJECT=TEST009-E: move cold host work out of partial evaluation
PROTOS_VERSION=0.3.201-SNAPSHOT

MAINTAINER_REPORTED_LOCAL_TESTS=PASS

OBSERVABLE_PROTOS_LANGUAGE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

TEST009-E was published after real synchronous Truffle compilation of the Test
Tool corpus exposed host-side work expanded into compiled guest operations,
including `TooDeepInlining` through JDK reflection and HotSpot code-installation
failures due to oversized generated code.

## Published cleanup classes

The publication keeps hot guest fast paths visible to PE and moves only
demonstrated cold/generic host work behind `@TruffleBoundary`.

The affected production responsibilities are:

~~~text
ProtosModuleRuntime
  module import preparation
  module specifier resolution

ProtosFrameArguments
  compact-call activation materialization

ProtosLexicalFallback
  residual String-keyed lexical read

ProtosIntegerValue
  arbitrary-precision Integer arithmetic fallback
  long fast paths remain inline

ProtosCoreErrors / ProtosBytecodeRootNode
  Error handler selection on root crossing

ProtosActivation
  dynamic-control-state inheritance
~~~

The publication does not blanket-annotate hot guest semantics. The boundary
placement is limited to work demonstrated to have no useful partial-evaluation
value on the compiled guest path.

## Preserved semantics

~~~text
EVALUATION_ORDER=UNCHANGED
ERROR_PRECEDENCE=UNCHANGED
ACTIVATION_IDENTITY=UNCHANGED
HANDLER_SELECTION=UNCHANGED
LONG_INTEGER_FAST_PATHS=INLINE
ALREADY_MATERIALIZED_ACTIVATION_FAST_PATH=INLINE

OBSERVABLE_LANGUAGE_SEMANTICS=UNCHANGED
SPECIFICATION_CHANGE=NO
~~~

## Why this was needed

The dynamic TEST009 compilation gate demonstrated that static required-constant
guards alone are insufficient for all compilerability debt.

The E failures belong to a different class:

~~~text
GENERAL_DYNAMIC_COMPILATION_PATHOLOGY
  excessive host-code expansion
  TooDeepInlining
  oversized compiled graph/code
~~~

These failures require real Truffle compilation evidence; they are not safely
proven by a source-only PE-constant guard.

## Validation classification

The maintainer reported:

~~~text
Todos los tests han pasado en local
~~~

Retained classification:

~~~text
MAINTAINER_REPORTED_LOCAL_TESTS=PASS
EXACT_TEST_COMMANDS_NOT_REPORTED_IN_HANDOFF=YES
~~~

No additional test count or duration is inferred.

## Methodological checkpoint

TEST009-E closes the concrete cold-host-work cleanup that had already been
identified by the real compilation corpus.

It does **not** ratify an open-ended strategy of automatically trying
`@TruffleBoundary` on arbitrary methods and keeping benchmark winners.

The next TEST009 step must first reconcile the dynamic guard strategy with the
systematic compilerability procedures used by mature Truffle languages:

~~~text
NEXT_METHOD_REVIEW=SYSTEMATIC_TRUFFLE_COMPILERABILITY_PROCEDURE
AUTOMATIC_BOUNDARY_LOTTERY=NOT_AUTHORIZED_AS_DESIGN_AUTHORITY
EXISTING_E_PRODUCT_REPAIRS=PUBLISHED_AND_PRESERVED
NEW_BOUNDARY_SWEEP=PAUSED_PENDING_METHOD_REVIEW
~~~

The intended investigation must distinguish:

~~~text
hard compilation failures
performance warnings
unexpected PE method/node expansion
guest-language inlining
deoptimization/invalidations
native-image PE block-list violations
~~~

and map each class to the framework-provided diagnostic/test mechanism rather
than conflating all of them into boundary search.

## Coordination

~~~text
TEST009_A=COMPLETE
TEST009_B=COMPLETE
TEST009_C=COMPLETE
TEST009_D=COMPLETE
TEST009_E=COMPLETE

TEST009_COMPLETE=NO
CURRENT_STATE=METHOD_REVIEW_BEFORE_FURTHER_DYNAMIC_GUARD_EXPANSION
NEXT_WORK_TYPE=INVESTIGATION
PRODUCT_CHANGES_AUTHORIZED_BY_NEXT_WORK=NO
NEW_FORMAL_ISSUE_REQUIRED=NO
~~~

AI assistance: this durable record was drafted with ChatGPT from the exact
published TEST009-E commit and maintainer-reported validation.
