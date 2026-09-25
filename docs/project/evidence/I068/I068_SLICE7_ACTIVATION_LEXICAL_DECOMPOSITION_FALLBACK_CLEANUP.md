# I068 Slice 7 — activation lexical decomposition / fallback cleanup

Status: **PUBLISHED / RETAINED EVIDENCE**

Owning Issue: `guillermomolina/protos#708`

## Exact publication identity

~~~text
PROTOS_REVISION=f1cee2d85858804ad3775adf43a9fab97664da2a
PROTOS_VERSION=0.3.87-SNAPSHOT
COMMIT_MESSAGE=I068: decompose activation lexical fallback machinery

I068_SLICE_7=PROTOS_ACTIVATION_LEXICAL_DECOMPOSITION_FALLBACK_CLEANUP
PLAT036_SELECTED_CANDIDATE=D
PLAT036_SELECTED_CANDIDATE_NAME=FRAME_BACKED_SEMANTIC_CONTEXT_ADAPTER
PRIOR_E1_RECOMMENDATION=SUPERSEDED

IMMEDIATE_PREDECESSOR_REVISION=fa487e51882c402149699f9024b2ec5aa11dfd76
PRODUCT_COMMIT_FILES=25
PRODUCT_COMMIT_ADDITIONS=485
PRODUCT_COMMIT_DELETIONS=95
~~~

The product commit is one ordinary commit on top of its immediate predecessor.
It advances the implementation version from `0.3.86-SNAPSHOT` to
`0.3.87-SNAPSHOT`.

## Architectural decomposition

Before this slice, `ProtosActivation` still exposed generic String-keyed
`lookup(String)` and `writableLexicalContext(String)` operations even though
Slices 3–6 had already established direct frame-backed authority for statically
proven current/captured bindings and semantic projection for reflection/debugger
surfaces.

Slice 7 removes those generic resolvers from `ProtosActivation` and introduces
the explicit residual helper:

~~~text
ProtosLexicalFallback.readByName(ProtosActivation, String)
ProtosLexicalFallback.writableContextByName(ProtosActivation, String)
~~~

The helper owns no lexical values and creates no second store. It traverses the
existing semantic activation topology and delegates storage operations to the
same authoritative execution-context objects/lexical-binding authorities.

~~~text
PROTOS_ACTIVATION_LEXICAL_DECOMPOSITION=PASS
NEW_LEXICAL_VALUE_STORE=NO
NO_DUAL_BINDING_AUTHORITY=PASS
~~~

## Static paths remain direct

Focused structural coverage proves that a statically proven current
`Resolved` read retains `ReadFrameLocal` and a proven
`CapturedResolved` read retains `ReadCapturedFrameLocal`, without lowering
through the generic `Lookup` operation for those sites.

~~~text
STATIC_CURRENT_FALLBACK_BYPASS=PASS
STATIC_CAPTURED_FALLBACK_BYPASS=PASS
FRAME_BACKED_AUTHORITY_PRESERVED=PASS
~~~

The captured write path also retains the Slice 5 destination-selection and
single-authority behavior.

## Required residual fallback remains exact

Candidate and Dynamic reads continue to use the exact name-based residual
fallback. The same helper is also used for compatibility paths where a direct
frame-backed proof is not applicable, captured late-presence/retargeting
fallback, non-statically-proven bare assignment destination resolution and
debugger semantic reads.

Read precedence remains:

~~~text
current execution-context local
-> captured lexical contexts, nearest first
-> ordinary receiver/member fallback
~~~

Bare-assignment destination resolution remains distinct:

~~~text
current execution-context own local
-> captured lexical contexts, nearest first
-> receiver own local only
-> missing
~~~

Receiver delegation is therefore preserved for reads and remains forbidden for
writes.

~~~text
CANDIDATE_DYNAMIC_FALLBACK_PRESERVED=PASS
RECEIVER_FALLBACK_PRESERVED=PASS
WRITES_NEVER_DELEGATE=PASS
ASSIGNMENT_DESTINATION_BEFORE_RHS=PASS
~~~

## Capture topology remains an activation responsibility

The slice deliberately retains:

~~~text
ProtosActivation.capturedLexicalContexts()
ProtosActivation.lexicalContextsForClosureCapture()
~~~

These methods describe semantic invocation/capture topology rather than generic
lexical name resolution. Removing them would have changed Closure capture and
Object-construction lexical-boundary behavior and was therefore outside the
cleanup.

~~~text
CAPTURE_TOPOLOGY_RETAINED=PASS
CAPTURE_BY_REFERENCE=PASS
OBJECT_BODY_BOUNDARY_PRESERVED=PASS
~~~

## Tooling and dynamic semantics

`ProtosDebuggerScope` now reads through the explicit residual fallback helper
rather than through a generic resolver method on `ProtosActivation`. The
semantic debugger projection remains live and read-only, with the same
current/captured/receiver precedence established before this slice.

The earlier I068 regression set remains green for:

- frame-backed current locals;
- sequential/default parameters;
- captured/materialized lexical access;
- late nearer creation and retargeting;
- assignment destination before RHS;
- debugger/reflection projection;
- Closure capture/object-construction boundaries;
- non-local return;
- Task/suspension/control paths.

~~~text
DEBUGGER_SEMANTIC_LOOKUP_PRESERVED=PASS
LATE_CREATION_RETARGETING=PASS
PRESENT_NULL_DISTINCT_FROM_ABSENT=PASS
D179_C3_PRESERVED=PASS
LAZY_CONTEXT_MATERIALIZATION=NO
SEMANTIC_CHANGE=NO
~~~

## Test migration

Existing Java tests that directly inspected `ProtosActivation.lookup` or
`writableLexicalContext` were migrated to the explicit
`ProtosLexicalFallback` boundary. This is an API-boundary update to the tests,
not a semantic relaxation.

A new focused test,
`ProtosI068Slice7ActivationLexicalDecompositionTest`, establishes that:

1. `ProtosActivation` no longer owns the two generic lexical resolver methods;
2. proven current reads retain `ReadFrameLocal` and bypass generic lookup;
3. proven captured reads retain `ReadCapturedFrameLocal` and bypass generic lookup;
4. Candidate reads retain generic fallback;
5. Dynamic reads retain generic fallback.

## Validation

Maintainer-executed validation reported:

~~~text
PRODUCTION_COMPILE=PASS
I068_SLICE7_FOCAL_TESTS=PASS
MIGRATED_PERF006_REGRESSION_SET=PASS
I068_SLICES_3_6_SEMANTIC_REGRESSION=PASS
MAVEN_TEST_SUITE=PASS
FULL_TEST_SUITE=PASS
GIT_DIFF_CHECK=PASS
REMOTE_CI_PASS=NOT_CLAIMED
~~~

The migrated PERF006 regression set retained parameter/default/rest binding,
suspension/no-replay, ordinary call/send behavior, receiver/home behavior,
composed targets/receivers and spread/default behavior.

The broader I068 regression set retained the Slice 3–6 authority and semantic
invariants plus Closure/NLR/Task coverage.

## I068 completion

~~~text
CANONICAL_BINDING_IDENTITY_PRESERVED=PASS
FRAME_CONTEXT_SINGLE_AUTHORITY=PASS
DEFINITELY_CURRENT_LOCAL_LOWERING=PASS
SEQUENTIAL_DEFAULT_PARAMETER_SEMANTICS=PASS
CAPTURED_MATERIALIZED_LEXICAL_LOWERING=PASS
LATE_CREATION_RETARGETING=PASS
DYNAMIC_FALLBACK=PASS
CONTEXT_REFLECTION_PROJECTION=PASS
DEBUGGER_SCOPE_PROJECTION=PASS
PROTOS_ACTIVATION_LEXICAL_DECOMPOSITION=PASS
NO_DUAL_BINDING_AUTHORITY=PASS
LAZY_CONTEXT_MATERIALIZATION=NOT_IMPLEMENTED
FOCAL_CONFORMANCE=PASS
FINAL_REQUIRED_VALIDATION=PASS

PLAT036_SELECTED_CANDIDATE=D
PRIOR_E1_RECOMMENDATION=SUPERSEDED

I068_SLICE_7_STATUS=COMPLETE
I068_STATUS=COMPLETE
NEXT_SLICE=NONE
~~~

This is the final planned I068 implementation slice. No Slice 8 is implied or
allocated by this completion record.
