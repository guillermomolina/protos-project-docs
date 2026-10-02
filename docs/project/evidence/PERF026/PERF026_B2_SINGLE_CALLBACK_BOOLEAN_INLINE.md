# PERF026-B2 — remaining single-callback Boolean literal inline checkpoint

Status: **PUBLISHED IMPLEMENTATION EVIDENCE**

Date: 2026-10-02

## Identity

~~~text
WORK_ITEM=PERF026-B/#767
SLICE=PERF026-B2
PRODUCT_REPOSITORY=guillermomolina/protos

BASE_REVISION=5b5dedd7a36b4aba0684a072a4f1a86a52ec9923
PROTOS_REVISION=18805ee47eb73c0817e3f4ee2b28a11571615de0
PROTOS_VERSION=0.3.136-SNAPSHOT
COMMIT_SUBJECT=PERF026-B2: inline remaining single-callback Boolean literals

PLAT044=#766 RATIFIED_B_PRIME
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

This checkpoint records facts observable from the exact published product
revision plus the publication handoff from the human executor. It does not infer
an unreported focal or integrated validation result merely from commit existence.

## Changed product paths

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf026B1BooleanInlineCallbackTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf026B2BooleanInlineCallbackTest.java
~~~

The new B2 Java test carries the APL-1.0 Part 5 notice. Modified Java files
retain the existing notice.

## Published B2 architecture

PERF026-B2 reuses the exact PLAT044 B-prime representation established by B1.
It does not introduce a second inline-callback mechanism.

The existing prepared standard Boolean capability remains the semantic
authority after ordinary lookup/selection.

At the published revision, the one-callback Boolean kinds admitted by the
common inline path are:

~~~text
IF_TRUE
IF_FALSE
AND
OR
~~~

The two-callback kind remains excluded:

~~~text
IF_TRUE_IF_FALSE=PHYSICAL_CALLBACK_PATH
~~~

For the admitted family, the existing selected callback is prepared through
ordinary callable authority first. Inline execution is admitted only when the
selected runtime callback is the exact staged immediate literal, the prepared
child is an ordinary source call with a rich activation, that activation does
not own its return home, and the prepared body target matches the literal
composition target.

Therefore the B1 semantic boundary remains intact:

~~~text
ordinary lookup/selection
  -> PreparedBooleanCall classification
  -> selected callback prepared normally
  -> exact B-prime eligibility check
  -> fresh callback activation
  -> inline callback body region
  -> normal Boolean callback completion

otherwise
  -> ordinary physical callback invocation
~~~

For AND and OR, the inline result still passes through the existing standard
Boolean-result validation. B2 therefore does not replace or weaken the Error
raised for a non-Boolean callback result.

## Short-circuit behavior

The inline path is reached only when the standard operation selects its callback.

~~~text
true.ifFalse(callback) -> callback not entered
false.and(callback)    -> callback not entered
true.or(callback)      -> callback not entered
~~~

The immediate standard results remain owned by the existing Boolean state
machine.

## Fallbacks preserved

B2 retains the B1 physical callback path for non-admitted shapes, including:

- dynamic callback values;
- non-Closure ordinarily invokable callback objects;
- custom same-name Boolean methods;
- literals whose activation owns its return home;
- literals with parameters;
- literals containing nested Closure literals;
- prepared runtime values or targets that do not exactly match the staged
  literal plan; and
- the two-callback IF_TRUE_IF_FALSE family reserved for PERF026-B3.

A non-admitted shape remains a performance fallback, not a semantic error.

## Structural regression surface present in the exact revision

The published B2 test source contains explicit checks/markers for:

~~~text
PERF026_B2_IF_FALSE_LITERAL_CALLBACK_ROOT_REMOVED
PERF026_B2_AND_LITERAL_CALLBACK_ROOT_REMOVED
PERF026_B2_OR_LITERAL_CALLBACK_ROOT_REMOVED

PERF026_B2_TRUE_IF_FALSE_CALLBACK_ENTERED=NO
PERF026_B2_FALSE_AND_CALLBACK_ENTERED=NO
PERF026_B2_TRUE_OR_CALLBACK_ENTERED=NO

PERF026_B2_AND_BOOLEAN_RESULT_VALIDATION_PRESERVED
PERF026_B2_OR_BOOLEAN_RESULT_VALIDATION_PRESERVED

PERF026_B2_IF_TRUE_IF_FALSE_ROOT_PRESERVED
PERF026_B2_DYNAMIC_CALLBACK_ROOT_PRESERVED
PERF026_B2_CUSTOM_BOOLEAN_SELECTOR_FALLBACK_PRESERVED
PERF026_B2_NONCLOSURE_INVOKABLE_FALLBACK_PRESERVED

PERF026_B2_NLR_PRESERVED
PERF026_B2_CAPTURE_BY_REFERENCE_PRESERVED
PERF026_B2_FRESH_ACTIVATION_PRESERVED
PERF026_B2_ERROR_PROPAGATION_PRESERVED
PERF026_B2_FUTURE_WAIT_PRESERVED
PERF026_B2_INLINE_SCOPE_PROJECTION_PRESERVED
~~~

These are structural test assertions present in the product revision. This
record does not claim they were executed in this coordination step.

## Validation provenance for this checkpoint

The handoff received for this coordination step is:

~~~text
PERF026-B2: inline remaining single-callback Boolean literals pushed
~~~

Accordingly:

~~~text
PRODUCT_PUBLICATION=YES
PROTOS_REVISION=18805ee47eb73c0817e3f4ee2b28a11571615de0

FOCAL_VALIDATION_RESULT=NOT_REPORTED_TO_THIS_COORDINATION_STEP
FULL_INTEGRATED_GATE_RESULT=NOT_REPORTED_TO_THIS_COORDINATION_STEP
GIT_DIFF_CHECK_RESULT=NOT_REPORTED_TO_THIS_COORDINATION_STEP
CI_STATUS_CHECKS=NONE_REPORTED_BY_GITHUB
~~~

No PASS result is fabricated from publication alone.

## Boolean decomposition after B2

B1 and B2 now cover every one-callback standard Boolean kind selected through
the existing prepared Boolean authority.

The remaining Boolean child slice is:

~~~text
PERF026-B3:
  IF_TRUE_IF_FALSE
  two eagerly evaluated callback arguments
  one selected callback invocation
  two candidate literal identities/plans
  exact selected-plan association
  same PLAT044 B-prime activation/tooling/fallback semantics
~~~

B3 must extend the common representation rather than add a selector-specific
parallel implementation.

The central additional requirement is that both callback-producing argument
expressions retain ordinary eager ordered evaluation while only the callback
selected by the canonical Boolean receiver is invoked. Inline admission must
associate the prepared selected child with the correct staged literal and plan;
if that association cannot be proven exactly, execution must retain the
physical callback path.

## Coordination consequence

~~~text
PERF026_B1_PRODUCT_REVISION=5b5dedd7a36b4aba0684a072a4f1a86a52ec9923
PERF026_B2_PRODUCT_REVISION=18805ee47eb73c0817e3f4ee2b28a11571615de0

PERF026_B=#767 REMAINS_OPEN
NEXT_SLICE=PERF026-B3

PERF026_C=#768 READY_INDEPENDENTLY
PERF026_C_DEPENDS_ON_FULL_BOOLEAN_COMPLETION=NO

GUEST_CALL_STACK_SIZE_BYTES=64_MIB
BUG008=#681 CLOSED_DO_NOT_REOPEN
~~~

## Cross references

- guillermomolina/protos#767 — PERF026-B.
- guillermomolina/protos#766 — PLAT044.
- guillermomolina/protos#768 — PERF026-C.
- guillermomolina/protos#681 — BUG008 remains closed.
- docs/project/evidence/PERF026/PERF026_B1_PLAT044_IFTRUE_INLINE_CALLBACK.md.
- docs/project/decisions/platform/PLAT044_SEMANTIC_CLOSURE_PHYSICAL_ROOT_BOUNDARY.md.
