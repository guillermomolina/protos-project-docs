# PERF026-B1 — PLAT044 standard IF_TRUE literal callback inline checkpoint

Status: **PUBLISHED IMPLEMENTATION EVIDENCE**

Date: 2026-10-02

## Identity

~~~text
WORK_ITEM=PERF026-B/#767
SLICE=PERF026-B1
PRODUCT_REPOSITORY=guillermomolina/protos

BASE_REVISION=8ea87fb1794599247f0e95fd5570d7dd7e08a52f
PROTOS_REVISION=5b5dedd7a36b4aba0684a072a4f1a86a52ec9923
PROTOS_VERSION=0.3.135-SNAPSHOT
COMMIT_SUBJECT=PERF026-B1: inline standard ifTrue literal callback

PLAT044=#766 RATIFIED_B_PRIME
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

This checkpoint records the exact published product slice and validation reported
by the human executor. It does not infer unreported CI or validation from commit
existence.

## Changed product paths

~~~text
CHANGELOG.md
pom.xml
protos/tests/conformance/boolean/callback-dispatch-and-control.protos
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeTagTreeNodeExports.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedExecution.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf026B1BooleanInlineCallbackTest.java
~~~

The new Java test carries the APL-1.0 Part 5 notice. Modified files retain their
existing notices.

## Published B1 architecture

The existing PLAT043 PreparedBooleanCall state machine remains authoritative.

At the published revision, runtime admission is deliberately narrow:

~~~text
BOOLEAN_KIND=IF_TRUE
CALLBACK_SELECTED=YES
SELECTED_RUNTIME_CALLBACK_IS_STAGED_LITERAL=YES
PREPARED_CHILD=OrdinarySourceCall
PREPARED_CHILD_HAS_RICH_ACTIVATION=YES
CALLBACK_OWNS_RETURN_HOME=NO
PREPARED_CHILD_TARGET_MATCHES_LITERAL_PLAN=YES
~~~

The compile-time candidate is:

~~~text
one supplied send argument
+ immediate CanonicalClosure literal
+ zero declared parameters
+ callback body contains no nested Closure literal
~~~

The compile-time candidate detector does not use selector spelling as semantic
authority. Runtime admission still occurs only after ordinary lookup/selection
has produced the existing standard Boolean capability.

The literal remains an ordinary send argument and is evaluated/materialized
exactly once. The send site stages it in an argument local so the prepared
selected callback can be compared against that exact runtime value.

For an admitted callback:

~~~text
PreparedBooleanCall
  -> prepare selected callback through existing ordinary callback authority
  -> obtain the prepared child's fresh ProtosActivation
  -> store that activation in an inline-callback BytecodeLocal
  -> execute callback body in an inline semantic region
  -> FinishClosureCall using the prepared child
  -> FinishStructuredBooleanCallback
~~~

The callback remains a fresh semantic Closure activation even though the eligible
path no longer enters its RootCallTarget.

## RootTag and debugger projection

The inline callback body carries a custom StandardTags.RootTag over its exact body
source span.

ProtosBytecodeTagTreeNodeExports detects the inline callback activation local and
projects debugger scope from that callback activation instead of blindly using
physical frame argument zero.

~~~text
INLINE_CALLBACK_ROOTTAG=YES
CURRENT_CALLBACK_SCOPE_PROJECTION=YES
FRESH_SEMANTIC_ACTIVATION=YES

DISTINCT_CALLBACK_FRAMEINSTANCE=NO
DISTINCT_CALLBACK_TRUFFLE_STACKTRACE_ELEMENT=NO
~~~

No custom DAP, virtual stack, or synthetic generic Truffle frame was introduced.

## Fallbacks preserved

Ordinary physical callback invocation remains for at least:

- a literal at module top level that owns its return home;
- a literal whose body contains another Closure literal;
- a literal that declares parameters;
- a callback stored in a variable and passed dynamically;
- a non-Closure ordinarily invokable object;
- a custom same-name ifTrue implementation;
- a prepared child whose runtime value or target does not exactly match the
  staged literal; and
- every other non-admitted shape.

A non-eligible shape is a performance fallback, not a semantic error.

## Boolean scope of B1

B1 changes only eligible IF_TRUE.

~~~text
IF_FALSE=UNCHANGED
IF_TRUE_IF_FALSE=UNCHANGED
AND=UNCHANGED
OR=UNCHANGED
~~~

## Published structural markers

~~~text
PERF026_B1_IFTRUE_LITERAL_CALLBACK_ROOT_REMOVED=YES
PERF026_B1_IFTRUE_INLINE_ROOTTAG=YES
PERF026_B1_DYNAMIC_CALLBACK_ROOT_PRESERVED=YES
PERF026_B1_OWNED_RETURN_HOME_FALLBACK=YES
PERF026_B1_IF_FALSE_UNCHANGED=YES
PERF026_B1_IF_TRUE_IF_FALSE_UNCHANGED=YES
PERF026_B1_AND_UNCHANGED=YES
PERF026_B1_OR_UNCHANGED=YES
PERF026_B1_CUSTOM_IFTRUE_ORDINARY_DISPATCH_PRESERVED=YES
PERF026_B1_NONCLOSURE_INVOKABLE_FALLBACK_PRESERVED=YES
PERF026_B1_FALSE_IFTRUE_CALLBACK_ENTERED=NO
PERF026_B1_NLR_PRESERVED=YES
PERF026_B1_CAPTURE_BY_REFERENCE_PRESERVED=YES
PERF026_B1_IFTRUE_LITERAL_FRESH_ACTIVATION_PRESERVED=YES
PERF026_B1_ERROR_PROPAGATION_PRESERVED=YES
PERF026_B1_FUTURE_WAIT_PRESERVED=YES
PERF026_B1_IFTRUE_INLINE_SCOPE_PROJECTION=YES
~~~

## Human-reported validation

~~~text
FOCAL_TEST_CLASSES=6
FOCAL_TEST_CLASSES_RESULT=PASS
B1_MARKERS_RESULT=PASS

GIT_DIFF_CHECK=CLEAN
FULL_INTEGRATED_GATE=make test
FULL_INTEGRATED_GATE_RESULT=PASS

SEPARATE_COMPILE_ONLY_STATIC_CHECK=NOT_RUN
COMPILE_EXECUTED_AS_PART_OF_FOCAL_AND_FULL_TEST_GATES=YES
~~~

No separate compile-only static result is claimed.

The Boolean conformance file gained four B1-oriented cases. The full integrated
gate was reported PASS on the exact candidate later published.

## Carrier consequence

~~~text
GUEST_CALL_STACK_SIZE_BYTES=64_MIB
DEDICATED_GUEST_CARRIER=STILL_PRESENT
BUG008=#681 CLOSED_DO_NOT_REOPEN
~~~

For an eligible recursive IF_TRUE literal callback, the physical shape can now
be:

~~~text
before B1:
  R_i -> B_i -> R_i+1

B1 eligible path:
  R_i -> [inline B_i semantic region] -> R_i+1
~~~

Therefore PERF025 post-B-prime topology/stack measurement is now actionable.
No carrier change is inferred before measurement.

## Known limitations and follow-up evidence

### General lexical lookup inside inline callback

B1 deliberately disables the enclosing root's direct/captured-local lowering
state while emitting the callback region. Variable accesses inside the inline
body therefore use the general activation authority rather than the
captured-local fast path.

~~~text
CORRECTNESS=VALIDATED
PERFORMANCE_EFFECT=UNMEASURED
~~~

### Compile-time inline-copy growth

At B1, inlineLiteralCallbackCandidate accepts any send with exactly one
zero-parameter Closure literal argument whose body has no nested Closure.

Therefore an inline copy is emitted even for a send such as:

~~~text
whileTrue(() => ...)
~~~

or another one-argument send whose current runtime capability cannot consume the
B1 inline branch.

~~~text
BYTECODE_GROWTH_RISK=KNOWN
MEASURED_COST=NOT_RECORDED
~~~

Do not silently make selector spelling semantic authority merely to remove this
copy. Any later narrowing is a performance implementation choice and must retain
ordinary-selection/fallback semantics.

### Other deliberate B1 exclusions

~~~text
LITERAL_WITH_PARAMETERS=FALLBACK
LITERAL_BODY_WITH_NESTED_CLOSURE=FALLBACK
OWNED_RETURN_HOME_LITERAL=FALLBACK
~~~

## Next Boolean decomposition

The published compile-time mechanism carries one candidate literal.

The smallest next reuse slice is therefore:

~~~text
PERF026-B2:
  extend the existing one-callback mechanism to
    IF_FALSE
    AND
    OR

PERF026-B3:
  add two-candidate selected-branch support for
    IF_TRUE_IF_FALSE
~~~

B2 must reuse B1 rather than introduce a second representation.

B3 is separate because ifTrueIfFalse eagerly evaluates two callback-producing
arguments but selects and invokes exactly one, so the lowering must retain two
candidate identities/plans and associate the prepared selected child with the
correct one.

## Coordination consequence

~~~text
PERF026_B1_CODE=YES
PERF026_B1_FOCAL_VALIDATION=YES
PERF026_B1_FULL_VALIDATION=YES
PERF026_B1_PUBLICATION=YES

PROTOS_REVISION=5b5dedd7a36b4aba0684a072a4f1a86a52ec9923

PERF026_B=#767 REMAINS_OPEN
NEXT_SLICE=PERF026-B2

PERF026_C=#768 COMMON_B_PRIME_MECHANISM_ESTABLISHED
PERF026_D=#769 COMMON_B_PRIME_MECHANISM_ESTABLISHED

PERF025=#758 POST_B1_STACK_MEASUREMENT_ACTIONABLE
~~~

## Cross references

- guillermomolina/protos#767 — PERF026-B.
- guillermomolina/protos#766 — PLAT044.
- guillermomolina/protos#768 — PERF026-C.
- guillermomolina/protos#769 — PERF026-D.
- guillermomolina/protos#758 — PERF025.
- guillermomolina/protos#681 — BUG008 remains closed.
- docs/project/decisions/platform/PLAT044_SEMANTIC_CLOSURE_PHYSICAL_ROOT_BOUNDARY.md.
