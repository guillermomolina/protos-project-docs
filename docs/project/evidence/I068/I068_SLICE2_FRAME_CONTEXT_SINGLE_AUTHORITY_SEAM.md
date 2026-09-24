# I068 Slice 2 — frame/context single-authority seam

Status: **PUBLISHED / RETAINED EVIDENCE**

Owning Issue: `guillermomolina/protos#708`

## Exact publication identity

~~~text
PROTOS_REVISION=1ccbb717fa824acd7aef71bd89c876e3bf8f6fe7
PROTOS_VERSION=0.3.82-SNAPSHOT
COMMIT_MESSAGE=I068: establish frame-context single-authority seam

I068_SLICE_2=FRAME_CONTEXT_SINGLE_AUTHORITY_SEAM
PLAT036_SELECTED_CANDIDATE=D
PRIOR_E1_RECOMMENDATION=SUPERSEDED
~~~

Immediate predecessor:

~~~text
I068_SLICE1_PRODUCT_REVISION=04a243c863cf50feb1ab541e3a49dd4dcd317039
I068_SLICE1_RECORD_REVISION=f37dc53a58efce10734497bd708dee31f36ed2ae
~~~

The product publication is exactly one commit ahead of Slice 1.

## Product delta

Changed paths:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/runtime/ProtosExecutionContextValue.java
src/main/java/com/guillermomolina/protos/runtime/ProtosLexicalBindingAuthority.java
src/main/java/com/guillermomolina/protos/runtime/ProtosMapBackedLexicalBindingAuthority.java
src/main/java/com/guillermomolina/protos/runtime/ProtosObjectValue.java
src/test/java/com/guillermomolina/protos/runtime/ProtosLexicalBindingAuthoritySeamTest.java
~~~

The CHANGELOG entry is owned by `I068`; PLAT036 Candidate D is cited only as
the governing decision.

## Authority seam established

`ProtosObjectValue` now owns exactly one
`ProtosLexicalBindingAuthority` installed at construction, and the affected
local-slot operations route through that authority instead of reading or
writing a private map directly.

The current concrete authority is:

~~~text
ProtosMapBackedLexicalBindingAuthority
  -> one insertion-ordered map
  -> one authoritative binding-value store per object
~~~

`ProtosExecutionContextValue` explicitly installs one authority through the
new authority-attachment constructor.

The active execution-context authority remains map-backed in this slice.
Therefore no Truffle frame/local value authority cutover has occurred yet.

## Single-authority invariant

~~~text
ONE_SEMANTIC_BINDING_VALUE_AUTHORITY_REQUIRED=YES

AUTHORITY_SEAM_IMPLEMENTED=YES
AUTHORITY_SEAM_ACTUALLY_USED=YES
EXECUTION_CONTEXT_SINGLE_AUTHORITY=YES
DUAL_AUTHORITATIVE_COPIES=NO

RUNTIME_AUTHORITY_CUTOVER=NO
BYTECODELOCAL_GUEST_AUTHORITY=NO
MATERIALIZED_LOCAL_MIGRATION=NO
LAZY_CONTEXT_MATERIALIZATION=NO
~~~

There is no mirror map, dual-write path or synchronization protocol keeping two
authoritative copies of the same binding.

## Preserved behavior represented by the retained tests

The focused product test class covers:

- create/read/assign through the seam;
- PRESENT canonical `null` distinct from ABSENT;
- D179/C3 execution-context removal rejection;
- ordinary-object removal remaining available;
- close/freeze behavior;
- escaped/captured references observing the same context instance;
- snapshots being observations rather than a second authority; and
- delegated lookup through an execution context using the same live authority.

The product diff also routes composition, structural-copy and ordinary lookup
paths through the authority rather than the former private map.

## Validation status

At the time this record was published, GitHub Actions had not completed:

~~~text
GITHUB_ACTIONS_WORKFLOW=CI
GITHUB_ACTIONS_RUN=35992940177
GITHUB_ACTIONS_CHECK=test
GITHUB_ACTIONS_STATUS=IN_PROGRESS
GITHUB_ACTIONS_CONCLUSION=NOT_YET_AVAILABLE
~~~

No explicit manual all-green result for Slice 2 was supplied in the interaction
that requested this record, so this evidence does not manufacture a validation
PASS.

## Next boundary

~~~text
NEXT_SLICE=I068_SLICE_3_DEFINITELY_CURRENT_LOCAL_LOWERING
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
~~~

Slice 3 is the first planned authority cutover for statically proven
definitely-current lexical bindings. It must consume Slice 1's
`CanonicalBindingResolution.Resolved` metadata and the Slice 2 authority seam
without migrating Candidate/Dynamic accesses, captured/materialized bindings,
sequential/default parameter semantics, debugger projection or final
`ProtosActivation` cleanup ahead of their owning slices.
