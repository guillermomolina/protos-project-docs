# I068 Slice 6 — debugger/reflection projection

Status: **PUBLISHED / RETAINED EVIDENCE**

Owning Issue: `guillermomolina/protos#708`

## Exact publication identity

~~~text
PROTOS_REVISION=f5f94fdf95b45ba7fa77221069381014589095b2
PROTOS_VERSION=0.3.86-SNAPSHOT
COMMIT_MESSAGE=I068: project debugger reflection from frame-backed bindings

I068_SLICE_6=DEBUGGER_REFLECTION_PROJECTION
PLAT036_SELECTED_CANDIDATE=D
PRIOR_E1_RECOMMENDATION=SUPERSEDED
~~~

The exact Slice 6 publication is one commit ahead of its immediate predecessor:

~~~text
COMPARE_BASE=ceba64b3412a1b61de5d1d74568d42d9b6128d8a
COMPARE_HEAD=f5f94fdf95b45ba7fa77221069381014589095b2
COMPARE_STATUS=ahead
AHEAD_BY=1
BEHIND_BY=0
TOTAL_COMMITS=1
~~~

## Product delta

Changed paths in the exact Slice 6 commit:

~~~text
CHANGELOG.md
pom.xml
src/test/java/com/guillermomolina/protos/execution/ProtosI068Slice6DebuggerReflectionProjectionTest.java
~~~

No production Java source changed in Slice 6. The implementation audit established
that Slices 2–5 had already completed the required projection seam:

~~~text
ProtosBytecodeTagTreeNodeExports
  -> ProtosDebuggerScope(ProtosActivation)
  -> current/captured semantic contexts
  -> localSlotsSnapshot()/readLocalSlot()
  -> the context's single ProtosLexicalBindingAuthority
  -> PRESENT frame-backed bindings + dynamic overflow
~~~

Slice 6 therefore closes the missing focused evidence over real frame-backed
execution rather than introducing a second projection or another lexical store.

## Context reflection projection

Focused coverage executes a real frame-backed module binding, creates a
dynamic-only overflow binding through the same first-class execution context,
and observes both through Core `slotNames` / `slotValue`.

The frame-backed binding remains owned by
`ProtosFrameLexicalBindingAuthority`; the dynamic-only name remains outside the
static frame layout and is projected from the authority's dynamic overflow.

~~~text
CONTEXT_REFLECTION_PROJECTION=PASS
FRAME_BACKED_STATIC_BINDINGS_VISIBLE=PASS
DYNAMIC_OVERFLOW_VISIBLE=PASS
NO_DUAL_BINDING_AUTHORITY=PASS
~~~

## Semantic presence and backend-temporary hiding

Focused coverage constructs a Closure activation whose first parameter is
established before a later default fails. The later parameter has a physical
frame-local allocation but remains semantically ABSENT.

Both Core reflection and debugger scope hide that later binding while retaining
the established binding. The debugger member set also excludes known Bytecode
DSL implementation temporaries.

~~~text
UNESTABLISHED_STATIC_BINDINGS_HIDDEN=PASS
BACKEND_TEMPORARIES_HIDDEN=PASS
PRESENT_STATE_DERIVED_FROM_SEMANTIC_AUTHORITY=PASS
~~~

This directly preserves the Slice 4 rule that physical local allocation does
not imply semantic presence.

## Captured binding liveness and debugger precedence

Focused coverage projects a proven captured frame-backed binding through a
Closure activation, mutates the outer frame-backed binding after the debugger
scope is obtained, and then observes the replacement value through that same
debugger scope.

The debugger therefore remains a live semantic projection over
`ProtosActivation` lookup rather than a copied debugger-only binding table.

~~~text
DEBUGGER_SCOPE_PROJECTION=PASS
DEBUGGER_LOOKUP_PRECEDENCE_PRESERVED=PASS
CAPTURE_BY_REFERENCE=PASS
LATER_MUTATION_VISIBLE=PASS
NO_DEBUGGER_BINDING_COPY=PASS
~~~

## Preserved tooling and lexical boundaries

The affected regression set retained the established PLAT013 / PLAT015 and I026
debugger behavior, including the read-only scope baseline and fail-closed
Bytecode tag-tree bridge behavior.

The Slice 3–5 authority regressions also remain green, preserving receiver
fallback, captured lexical behavior, D179/C3, and the Object-construction
lexical boundary.

~~~text
READ_ONLY_TOOLING_BASELINE_PRESERVED=PASS
OBJECT_BODY_BOUNDARY_PRESERVED=PASS
DYNAMIC_FALLBACK_PRESERVED=PASS
D179_C3_PRESERVED=PASS
LAZY_CONTEXT_MATERIALIZATION=NO
SEMANTIC_CHANGE=NO
~~~

Direct Interop object-member projection was deliberately not enabled on
`ProtosExecutionContextValue`. PLAT013's ordinary-object Interop surface
remains unchanged; guest Core reflection and the synthetic debugger scope are
the semantic observation surfaces used by this slice.

## Validation

Maintainer-executed validation reported:

~~~text
GIT_DIFF_CHECK=PASS
I068_SLICE6_FOCAL_TESTS=PASS
AFFECTED_REGRESSION_SET=PASS
MAVEN_TEST_SUITE=PASS
FULL_TEST_SUITE=PASS
REMOTE_CI_PASS=NOT_CLAIMED
~~~

The affected regression set included:

~~~text
ProtosI026EScopeTest
ProtosPerf006B5ABytecodeDebuggerScopeTest
ProtosPlat036Slice3FrameBackedCurrentLocalTest
ProtosI068Slice4SequentialParameterLoweringTest
ProtosI068Slice5CapturedMaterializedLexicalLoweringTest
ProtosI068Slice6DebuggerReflectionProjectionTest
~~~

An initial focal-test failure was confined to the new test harness: its minimal
test prelude lacked the standard `SlotNotFound` error prototype required to
materialize the intended missing-lookup signal. Adding that prototype to the
test-only prelude resolved the harness defect; no product-code repair was
required.

## Slice conclusion

~~~text
I068_SLICE_6_STATUS=COMPLETE
CONTEXT_REFLECTION_PROJECTION=PASS
DEBUGGER_SCOPE_PROJECTION=PASS
FRAME_BACKED_STATIC_BINDINGS_VISIBLE=PASS
DYNAMIC_OVERFLOW_VISIBLE=PASS
UNESTABLISHED_STATIC_BINDINGS_HIDDEN=PASS
BACKEND_TEMPORARIES_HIDDEN=PASS
READ_ONLY_TOOLING_BASELINE_PRESERVED=PASS
DEBUGGER_LOOKUP_PRECEDENCE_PRESERVED=PASS
OBJECT_BODY_BOUNDARY_PRESERVED=PASS
NO_DUAL_BINDING_AUTHORITY=PASS

NEXT_SLICE=I068_SLICE_7_PROTOS_ACTIVATION_LEXICAL_DECOMPOSITION_FALLBACK_CLEANUP
~~~

I068 remains open. Slice 7 owns only the final
`ProtosActivation` lexical-decomposition / obsolete generic-fallback cleanup
after the authoritative current, parameter, captured, reflection and debugger
paths established by Slices 1–6.
