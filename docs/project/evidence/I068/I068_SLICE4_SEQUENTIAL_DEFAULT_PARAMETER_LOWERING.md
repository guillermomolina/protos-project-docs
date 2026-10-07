# I068 Slice 4 — sequential/default parameter lowering

Status: **PUBLISHED / RETAINED EVIDENCE**

Owning Issue: `guillermomolina/protos#708`

## Exact publication identity

~~~text
PROTOS_REVISION=d6587c535ee83653c417d2bf780a9d7b83d7ac24
PROTOS_VERSION=0.3.84-SNAPSHOT
COMMIT_MESSAGE=I068: lower closure parameters to frame-backed locals

I068_SLICE_4=SEQUENTIAL_DEFAULT_PARAMETER_LOWERING
PLAT036_SELECTED_CANDIDATE=D
PRIOR_E1_RECOMMENDATION=SUPERSEDED
RUNTIME_AUTHORITY_CUTOVER=CURRENT_RESOLVED_PLUS_PARAMETERS
~~~

Immediate predecessor:

~~~text
I068_SLICE3_PRODUCT_REVISION=1756b3d3100d54ef1627bc618cae6b7ef8da3445
I068_SLICE3_EVIDENCE_REVISION=abc6fa288d177b9a3e8781b32f7aa0e1aeedfd6f
~~~

GitHub comparison establishes that the Slice 4 product publication is exactly
one commit ahead of Slice 3:

~~~text
COMPARE_STATUS=ahead
AHEAD_BY=1
BEHIND_BY=0
TOTAL_COMMITS=1
~~~

## Product delta

Changed paths:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalLexicalScope.java
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java
src/test/java/com/guillermomolina/protos/execution/ProtosI068Slice4SequentialParameterLoweringTest.java
~~~

## Parameter frame-local cutover

Slice 4 extends the genuine current execution-context root's stable Bytecode
local layout to Closure parameters.

A parameter's physical `BytecodeLocal` now exists from root construction, but
that physical identity does not establish a Protos binding. The local remains
cleared until the existing left-to-right parameter-binding operation
successfully creates the binding through the installed
`ProtosFrameLexicalBindingAuthority`.

The governing invariant is therefore retained directly in frame state:

~~~text
PHYSICAL_BYTECODE_LOCAL_EXISTS != SEMANTIC_PARAMETER_BINDING_PRESENT
STATIC_LOCAL_EXISTS != SEMANTIC_BINDING_PRESENT
PRESENT(null) != ABSENT
~~~

No placeholder guest value, hoisted semantic slot, TDZ object, second parameter
namespace or duplicate authoritative store is introduced.

## Sequential/default semantics retained

The existing parameter-binding bytecode sequence remains authoritative and was
not reordered by this slice.

Retained behavior includes:

- caller arguments are already evaluated before activation parameter binding;
- parameters bind strictly left-to-right;
- a supplied ordinary argument suppresses its default;
- an omitted default is evaluated once in the current invocation activation;
- semantic presence begins only after the default completes successfully and
  the binding operation creates the parameter;
- earlier bound parameters are visible to later defaults;
- the current parameter remains absent during its own default;
- later parameters remain absent during earlier defaults;
- self/future references therefore retain ordinary fallback while their
  physical frame locals are still cleared;
- required-parameter and excess-argument errors retain their existing ordering;
- earlier effects/bindings are not rolled back by a later parameter failure;
- rest receives a fresh frozen standard Array containing exactly the unconsumed
  supplied suffix;
- default-generated values are not included in rest;
- `args` remains an ordinary identifier; and
- receiver, method-home, return-home, suspension and dynamic-control behavior
  remain on the existing invocation path.

Once a parameter is semantically established, a statically `Resolved` read
owned by the current activation may use the Slice 3 direct frame-local read
path. `Candidate` and `Dynamic` references remain on exact generic fallback.

## First-class context and single authority

The invocation's `ProtosExecutionContextValue` projects the same frame-backed
parameter values used by direct current-root reads.

Focused coverage establishes that:

- required and defaulted parameters are visible through the first-class context
  after their binding point;
- a supplied Protos `null` is PRESENT rather than ABSENT;
- self-default and earlier-default/later-parameter lookup observe fallback while
  the relevant parameter local is cleared;
- rest is the expected fresh frozen suffix;
- an escaped invocation context still observes its frame-backed parameter after
  the activation has returned; and
- retained Bytecode parser/source-section materialization remains compatible
  with the parameter-local layout.

~~~text
PARAMETER_FRAME_LOCAL_MIGRATION=YES
SEQUENTIAL_DEFAULT_PARAMETER_SEMANTICS=PASS
PARAMETER_SEMANTIC_ABSENCE_UNTIL_BINDING_POINT=PASS
DIRECT_CURRENT_RESOLVED_PARAMETER_READ=YES
PARAMETER_CONTEXT_PROJECTION_SAME_AUTHORITY=YES
DUAL_AUTHORITATIVE_COPIES=NO
PRESENT_NULL_DISTINCT_FROM_ABSENT=YES
ESCAPED_CONTEXT_OBSERVES_FRAME_BACKED_PARAMETER=YES
BYTECODE_REPARSE_STATE_PRESERVED=YES
~~~

## Validation evidence

Maintainer-reported validation for the product candidate included:

~~~text
GIT_DIFF_CHECK=PASS
MAKE_COMPILE=PASS
I068_SLICE4_FOCAL_TESTS=PASS
PARAMETER_ARITY_DEFAULT_REST_REGRESSIONS=PASS
SLICE3_AUTHORITY_REGRESSIONS=PASS
MAKE_TEST_JAVA=PASS
MAKE_TEST_PROTOS=PASS

PUBLICATION_VALIDATION_IMPACT=FULL
PUBLICATION_VALIDATION_FULL_SUITE=REQUIRED
PUBLICATION_VALIDATION=PASS
FULL_TEST_SUITE=PASS
~~~

The final `scripts/publication_validation.py` run completed successfully on
the immutable Slice 4 commit. Its selector classified the delta as `FULL`,
with `AFFECTED_TEST_SET=ALL`, so the canonical integrated `make test` gate was
included in that successful publication validation.

No GitHub Actions workflow run was returned for the product SHA at evidence
record time, so this record does not claim a remote CI PASS.

## Remaining I068 boundary

Slice 4 does not complete I068.

~~~text
NEXT_SLICE=I068_SLICE_5_CAPTURED_MATERIALIZED_LEXICAL_LOWERING
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos

CAPTURED_MATERIALIZED_LEXICAL_LOWERING=NOT_YET_IMPLEMENTED
DEBUGGER_REFLECTION_PROJECTION_SLICE=NOT_YET_IMPLEMENTED
PROTOS_ACTIVATION_FALLBACK_CLEANUP=NOT_YET_IMPLEMENTED
LAZY_CONTEXT_MATERIALIZATION=NOT_IMPLEMENTED
~~~

Slice 5 owns proven captured lexical access through frame/materialized-local
mechanisms while preserving capture by reference, later mutation visibility,
legal nearer late creation/retargeting and exact dynamic fallback.
