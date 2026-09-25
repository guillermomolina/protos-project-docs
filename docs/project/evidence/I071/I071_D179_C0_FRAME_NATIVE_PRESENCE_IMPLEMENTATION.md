# I071 — D179 C0 frame-native presence implementation evidence

FORMAL_IDENTIFIER=I071
GITHUB_ISSUE=https://github.com/guillermomolina/protos/issues/717
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
BASE_PROTOS_REVISION=d4ac1c7be600c00c785dc9742dc7aeaeab17eaec
PROTOS_REVISION=0c9307240796260882ddca27611c91b7ffa4ce3b
PROTOS_VERSION=0.3.88-SNAPSHOT
SPEC_REVISION=0.1.434
COMMIT_MESSAGE=I071: restore execution-context removal semantics
DECISION_AUTHORITY=D179_C0_WITH_IMPLEMENTATION_CANDIDATE_E
IMPLEMENTATION_STATE=COMPLETE

## Result

I071 restores the owner-approved D179 C0 execution-context structural-removal
semantics on top of the retained PLAT036 Candidate D / I068 frame-backed lexical
architecture.

The published implementation establishes:

```text
D179_C0_RESTORED=PASS
FRAME_NATIVE_CLEARED_PRESENCE=PASS
CURRENT_RESOLVED_READ_PRESENCE_GUARD=PASS
OUTER_LEXICAL_FALLBACK_AFTER_REMOVAL=PASS
RECEIVER_FALLBACK_AFTER_REMOVAL=PASS
CAPTURE_BY_REFERENCE_AFTER_REMOVAL=PASS
REMOVE_RECREATE=PASS
PRESENT_NULL_DISTINCT_FROM_ABSENT=PASS
ASSIGNMENT_DESTINATION_BEFORE_RHS=PASS
CLOSED_FROZEN_RULES=PASS
ORDINARY_OBJECT_REMOVE_SLOT=PASS
CONTEXT_REFLECTION_PROJECTION=PASS
DEBUGGER_SCOPE_PROJECTION=PASS
FRAME_BACKED_SINGLE_AUTHORITY=PASS
NO_DUAL_BINDING_AUTHORITY=PASS
PLAT039_COMPATIBILITY=PASS
FINAL_REQUIRED_VALIDATION=PASS
```

No PLAT036 or I068 architecture decision is reopened.

## Product implementation

The C3-specific execution-context removal override was removed from
`ProtosExecutionContextValue`, so genuine execution contexts once again use the
ordinary open/closed/frozen structural rules inherited from
`ProtosObjectValue`.

For statically admitted frame-backed bindings, the existing Bytecode DSL
frame-local cleared state is the semantic presence mechanism:

```text
PRESENT -> ABSENT
  existing frame-local clear()

ABSENT -> PRESENT
  existing frame-local setObject()/putBinding()

binding identity / owner / lexical depth / ordinal
  unchanged
```

The current-scope `ReadFrameLocal` direct path now checks frame-local presence
before reading. A cleared binding takes the exact existing lexical/receiver
fallback path.

Captured frame-backed reads retain their existing nearer-presence and
owner-presence checks. No duplicate authoritative store was introduced.

## Semantic coverage

Published conformance coverage now includes the execution-context fixtures in
the suite-native manifest and covers:

- current local removal revealing an outer lexical binding;
- current local removal revealing receiver fallback after lexical exhaustion;
- captured/escaped removal observed by later Closure reads;
- removal followed by legal re-creation in the same open execution context;
- exact removed value, including PRESENT `null` remaining distinct from ABSENT;
- CLOSED structural-removal rejection;
- FROZEN creation, assignment and removal rejection;
- ordinary non-execution-context `Object.removeSlot` behavior.

Focused Java evidence additionally establishes:

- clear/re-create retains the same statically allocated frame ordinal rather
  than migrating the binding to dynamic overflow;
- debugger and Core reflection omit a cleared static binding while it is
  semantically absent and expose it again after re-creation;
- a captured assignment destination selected before RHS evaluation does not
  retarget to a newly visible nearer binding if the RHS removes the originally
  selected destination; the pinned write fails against the removed destination.

## Normative convergence

Specification revision `0.1.434` supersedes the active C3 monotonic-membership
rule introduced by historical revision `0.1.433` without rewriting that
historical entry.

`EXECUTION_AND_CONTROL.md` now states that genuine execution contexts follow
the ordinary structural Object contract, including PRESENT-to-ABSENT removal
while OPEN and the resulting lexical/receiver fallback behavior.

`OBJECT_MODEL.md` removes the former C3 execution-context specialization and
points to the lexical consequences owned by `EXECUTION_AND_CONTROL.md`.

## Validation evidence

Human-executed validation reported during the implementation session:

```text
git diff --check=PASS

FOCUSED_JAVA=
  ProtosExecutionContextValueTest
  ProtosLexicalBindingAuthoritySeamTest
  ProtosI068Slice5CapturedMaterializedLexicalLoweringTest
  ProtosI068Slice6DebuggerReflectionProjectionTest
  ProtosI068Slice7ActivationLexicalDecompositionTest
  ProtosPreludeTest
RESULT=PASS

PROTOS_CONFORMANCE=1263 passed, 0 failed
PROTOS_TEST_TOOL_BOOTSTRAP=PASS
PROTOS_TESTS_TOTAL_TIME=122 s

PUBLICATION_VALIDATION_TOP_LEVEL_CLOSURE=PASS
PUBLICATION_VALIDATION_PROVENANCE=HUMAN_REPORTED_GREEN
REMOTE_CI=NOT_CLAIMED
```

The immutable publication candidate used the exact range:

```text
BASE=d4ac1c7be600c00c785dc9742dc7aeaeab17eaec
HEAD=0c9307240796260882ddca27611c91b7ffa4ce3b
```

## Architectural exclusions retained

The implementation introduced none of the excluded representation changes:

```text
UNBOUND_SENTINEL=NO
SECOND_BINDING_AUTHORITY=NO
SEPARATE_PRESENCE_BITMAP=NO
STATIC_BINDING_MAP_MIGRATION=NO
BROAD_TRUFFLE_BOUNDARY=NO
PLAT036_REWRITE=NO
I068_REWRITE=NO
PLAT039_REOPEN=NO
```

The frame-local presence guard remains in the PE-visible guest kernel, consistent
with PLAT039 Candidate C.

## Closure

I071 is complete at Protos revision
`0c9307240796260882ddca27611c91b7ffa4ce3b`.

D179's current physical product/specification state is now C0. The prior C3
implementation and specification revisions remain historical evidence only.
