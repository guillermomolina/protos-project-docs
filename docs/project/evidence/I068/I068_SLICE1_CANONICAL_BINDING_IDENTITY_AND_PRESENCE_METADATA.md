# I068 Slice 1 — canonical binding identity and presence metadata

Status: **PUBLISHED / RETAINED EVIDENCE**

Owning Issue: `guillermomolina/protos#708`

Implements the first bounded implementation slice under the ratified PLAT036
Candidate D architecture.

## Exact publication identity

~~~text
PROTOS_REVISION=04a243c863cf50feb1ab541e3a49dd4dcd317039
PROTOS_VERSION=0.3.81-SNAPSHOT
COMMIT_MESSAGE=I068: retain canonical binding identity and presence metadata

I068_SLICE_1=CANONICAL_BINDING_IDENTITY_AND_PRESENCE_METADATA
PLAT036_SELECTED_CANDIDATE=D
PRIOR_E1_RECOMMENDATION=SUPERSEDED
~~~

Baseline:

~~~text
PROTOS_BASELINE_REVISION=75231601e458930684d4b619dd5f4722377aa65d
PLAT036_PROJECT_RECORD_REVISION=96e093ab0e4a625c8d768a5a11c36d97d921b2c8
I068_ALLOCATION_RECORD_REVISION=f6f1ac90ab82e4f0cd47f61c3b0ccfbb3e5b0bb8
~~~

The product publication is exactly one commit ahead of the baseline.

## Product delta

Changed paths:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalBindingAnalysis.java
src/main/java/com/guillermomolina/protos/execution/CanonicalBindingAnalyzer.java
src/main/java/com/guillermomolina/protos/execution/CanonicalBindingIdentity.java
src/main/java/com/guillermomolina/protos/execution/CanonicalBindingResolution.java
src/main/java/com/guillermomolina/protos/execution/CanonicalLexicalScope.java
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/test/java/com/guillermomolina/protos/execution/CanonicalBindingAnalyzerTest.java
~~~

The implementation adds five backend-private lexical-analysis classes and one
focused test class. `CanonicalToBytecodeLowerer` computes/caches the analysis
for each lowering unit.

The `CHANGELOG.md` entry is owned by `I068`, not PLAT036.

## Retained classification model

The canonical analysis distinguishes three runtime-relevant categories while
leaving execution unchanged:

~~~text
Resolved
  = binding identity and owner are statically known
  + binding is guaranteed PRESENT in the current scope at this point

Candidate
  = binding identity/owner/depth are statically known
  + presence is not guaranteed here
  + a nearer legal future creation may still affect resolution

Dynamic
  = no lexical owner is statically known in the analyzed chain
  + preserve exact existing dynamic receiver/member fallback
~~~

This preserves the critical PLAT036/D179 distinction:

~~~text
STATIC_BINDING_IDENTITY != SEMANTIC_PRESENCE
PRESENT(null) != ABSENT
~~~

## Exact properties established

The focused product tests cover at least:

- same-name bindings in different lexical owners have distinct identities;
- repeated references to one binding retain the same identity;
- current versus outer lexical classification retains lexical depth;
- parameter identity may exist before semantic presence;
- legal nearer late creation prevents unconditional resolution before creation;
- unresolved names remain explicitly Dynamic;
- bare assignment destination metadata is retained independently of RHS;
- explicit member targets stay outside the lexical-binding model; and
- wiring the analysis into lowering does not change the observed runtime lexical result.

## Authority boundary after Slice 1

No runtime lexical authority moved in this slice.

~~~text
RUNTIME_AUTHORITY_CUTOVER=NO
BYTECODELOCAL_GUEST_AUTHORITY=NO
MATERIALIZED_LOCAL_MIGRATION=NO
CONTEXT_ADAPTER_IMPLEMENTED=NO

CURRENT_RUNTIME_LEXICAL_AUTHORITY=
  ProtosActivation + execution-context object + String-keyed lookup/write

ONE_SEMANTIC_BINDING_VALUE_AUTHORITY_REQUIRED=YES
~~~

Therefore Slice 1 is preparatory compiler metadata, not a partial E1-style
semantic store and not a partial frame-authority migration.

## Validation

The maintainer reported after publishing the product commit:

~~~text
ALL_TESTS_GREEN=YES
MAINTAINER_REPORTED_TESTS=ALL_GREEN
~~~

No exact test count or command transcript is asserted by this record because
that information was not supplied in the publication report.

The Git product revision itself is the stable publication identity for the code
and focused tests.

## Next boundary

The next implementation unit remains inside I068:

~~~text
NEXT_SLICE=I068_SLICE_2_FRAME_CONTEXT_SINGLE_AUTHORITY_SEAM
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
~~~

Slice 2 must establish the internal authority/adapter seam required by Candidate
D without reintroducing duplicate binding-value authority and without yet
claiming the later definitely-current, captured/materialized, debugger or
ProtosActivation cleanup slices as complete.
