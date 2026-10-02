# PERF026-B3 — standard IF_TRUE_IF_FALSE literal callback inline checkpoint

Status: **PUBLISHED IMPLEMENTATION EVIDENCE**

Date: 2026-10-02

## Identity

~~~text
WORK_ITEM=PERF026-B/#767
SLICE=PERF026-B3
PRODUCT_REPOSITORY=guillermomolina/protos

BASE_REVISION=7c16cec611c3cf5e504d9271c6964a32656ba32f
PROTOS_REVISION=75d52464eaddf2c1190602a9708d80237f0d7b76
PROTOS_VERSION=0.3.138-SNAPSHOT
COMMIT_SUBJECT=PERF026-B3: inline ifTrueIfFalse literal callbacks

PLAT044=#766 RATIFIED_B_PRIME
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

B3 is one commit ahead of the exact product base above. The intervening
0.3.137-SNAPSHOT product revision is BUG013-F; this record therefore does not
pretend B3 was published directly on top of the older B2 revision.

## Changed product paths

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf026B1BooleanInlineCallbackTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf026B2BooleanInlineCallbackTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf026B3BooleanInlineCallbackTest.java
~~~

The new B3 Java test carries the project APL-1.0 Part 5 notice. Modified Java
files retain their existing notice.

## Published B3 architecture

B3 generalizes the B1/B2 inline-callback representation from one supplied
literal candidate to positional literal candidates.

Each candidate retains:

~~~text
literal definition
supplied argument position
staged runtime literal value
literal plan cell
~~~

The send still evaluates callback-producing arguments through the ordinary send
evaluation path. IF_TRUE_IF_FALSE therefore preserves eager evaluation of both
arguments, exactly once and in source order.

After ordinary lookup, standard Boolean classification and selected-callback
preparation, the runtime-selected callback is associated with its ordinary
selected argument position. Inline execution is admitted only when that selected
position has a staged eligible literal and the prepared selected child exactly
matches that literal and plan under the existing PLAT044 B-prime activation
checks.

Conceptually:

~~~text
ordinary receiver evaluation
  -> eager ordered evaluation of true callback argument
  -> eager ordered evaluation of false callback argument
  -> ordinary lookup / selection
  -> PreparedBooleanCall
  -> ordinary selected callback preparation
  -> exact selected-position candidate association
  -> exact literal / plan / activation admission
  -> existing inline callback region
  -> existing Boolean callback completion

otherwise
  -> exact ordinary physical callback invocation
~~~

The selector spelling is not semantic authority for admission. The ordinary
PreparedBooleanCall selection remains authoritative.

## Completed standard Boolean family

At B3 the guarded PLAT044 B-prime literal path covers the complete standard
Boolean callback family owned by PERF026-B:

~~~text
IF_TRUE
IF_FALSE
IF_TRUE_IF_FALSE
AND
OR
~~~

B3 supersedes the B1/B2 temporary regression assertions that an otherwise
eligible IF_TRUE_IF_FALSE callback had to remain physical.

## Selected and unselected branch behavior

The exact B3 test source proves both selection directions structurally:

~~~text
PERF026_B3_TRUE_BRANCH_LITERAL_CALLBACK_ROOT_REMOVED=YES
PERF026_B3_UNSELECTED_FALSE_CALLBACK_ENTERED=NO

PERF026_B3_FALSE_BRANCH_LITERAL_CALLBACK_ROOT_REMOVED=YES
PERF026_B3_UNSELECTED_TRUE_CALLBACK_ENTERED=NO
~~~

Both candidate-producing arguments remain eagerly evaluated even though only
one callback body is invoked:

~~~text
PERF026_B3_EAGER_ORDERED_ARGUMENT_EVALUATION_PRESERVED=YES
~~~

IF_TRUE_IF_FALSE returns the selected callback result directly through the
existing Boolean completion authority; B3 does not introduce AND/OR-style
Boolean-result validation for this operation.

## Tooling and semantic preservation

The B3 regression source contains explicit assertions/markers for:

~~~text
PERF026_B3_INLINE_ROOTTAG=YES
PERF026_B3_NLR_PRESERVED=YES
PERF026_B3_CAPTURE_BY_REFERENCE_PRESERVED=YES
PERF026_B3_FRESH_ACTIVATION_PRESERVED=YES
PERF026_B3_ERROR_PROPAGATION_PRESERVED=YES
PERF026_B3_FUTURE_WAIT_PRESERVED=YES
PERF026_B3_INLINE_SCOPE_PROJECTION_PRESERVED=YES
~~~

The PLAT044 approved tooling boundary remains unchanged:

~~~text
SEMANTIC_CALLBACK_ACTIVATION=PRESERVED
INLINE_ROOTTAG=PRESERVED
DEBUGGER_SCOPE_PROJECTION=PRESERVED

DISTINCT_CALLBACK_FRAMEINSTANCE=NO
DISTINCT_CALLBACK_TRUFFLE_STACKTRACE_ELEMENT=NO
~~~

## Fallbacks preserved

The exact B3 regression source also records physical/ordinary fallback for:

~~~text
PERF026_B3_DYNAMIC_CALLBACK_ROOT_PRESERVED=YES
PERF026_B3_MISMATCHED_POSITION_FALLBACK_PRESERVED=YES
PERF026_B3_PARAMETERIZED_LITERAL_ROOT_PRESERVED=YES
PERF026_B3_NESTED_CLOSURE_LITERAL_ROOT_PRESERVED=YES
PERF026_B3_OWNED_RETURN_HOME_FALLBACK=YES
PERF026_B3_CUSTOM_BOOLEAN_SELECTOR_FALLBACK_PRESERVED=YES
PERF026_B3_NONCLOSURE_INVOKABLE_FALLBACK_PRESERVED=YES
~~~

A non-admitted shape remains a performance fallback, not a semantic error.

## Human-reported validation

The publication handoff from the human executor states:

~~~text
PERF026-B3: inline ifTrueIfFalse literal callbacks pushed
todos los tests en pass
~~~

Accordingly this durable checkpoint records:

~~~text
PRODUCT_PUBLICATION=YES
HUMAN_REPORTED_ALL_REQUESTED_TESTS=PASS
HUMAN_REPORTED_TEST_FAILURES=NONE
~~~

No more specific command-level result is invented beyond that report.

## PERF026-B closure consequence

B1 established the common B-prime inline literal mechanism for IF_TRUE.

B2 reused it for IF_FALSE, AND and OR.

B3 completes the remaining two-callback IF_TRUE_IF_FALSE shape while preserving
ordinary eager argument evaluation and exact selected-position authority.

Therefore:

~~~text
PERF026_B_STANDARD_BOOLEAN_FAMILY=COMPLETE
PERF026_B_IMPLEMENTATION=COMPLETE
PERF026_B_VALIDATION=PASS_HUMAN_REPORTED
PERF026_B_SEMANTIC_CHANGE=NO
PERF026_B_SPECIFICATION_CHANGE=NO
~~~

No new benchmark suite was introduced and B3 makes no standalone performance
claim.

## Next PERF026 family

PERF026-C / #768 is already READY. Its previous dependency on establishing the
common B-prime mechanism was satisfied by B1 and did not require full Boolean
completion.

The next implementation family is standard Closure.whileTrue local control,
reusing the same PLAT044 activation/tooling boundary rather than inventing
another inline-callback representation.

## Cross references

- guillermomolina/protos#767 — PERF026-B.
- guillermomolina/protos#768 — PERF026-C.
- guillermomolina/protos#769 — PERF026-D.
- guillermomolina/protos#766 — PLAT044.
- guillermomolina/protos#681 — BUG008 remains closed.
- docs/project/evidence/PERF026/PERF026_B1_PLAT044_IFTRUE_INLINE_CALLBACK.md.
- docs/project/evidence/PERF026/PERF026_B2_SINGLE_CALLBACK_BOOLEAN_INLINE.md.
- docs/project/decisions/platform/PLAT044_SEMANTIC_CLOSURE_PHYSICAL_ROOT_BOUNDARY.md.
