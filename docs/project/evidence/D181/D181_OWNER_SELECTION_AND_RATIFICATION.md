# D181 — Owner selection and ratification evidence

Status: **FINAL DECISION EVIDENCE — Candidate D ratified**

Date: 2026-10-02

~~~text
DECISION=D181
DECISION_ISSUE=guillermomolina/protos#770
APPROVED_CANDIDATE=D
IMPLEMENTATION_OWNER=DIST010/guillermomolina/protos#771
PROTOS_BASELINE=8ea87fb1794599247f0e95fd5570d7dd7e08a52f
~~~

## Investigation result carried into approval

D181-A compared the status quo and the producer-boundary alternatives after
WEB009 exposed that the website currently knows and executes Protos producer
implementation details.

The important current-state facts were:

~~~text
PROTOS_OWNS_EXTRACTOR=YES
WEBSITE_EXECUTES_EXTRACTOR=YES
WEBSITE_KNOWS_MAVEN_COMPILE=YES
WEBSITE_KNOWS_JAVA_EXTRACTOR_CLASS=YES
WEBSITE_KNOWS_TARGET_CLASSES=YES
WEBSITE_BUILD_CARRIES_JDK_MAVEN_FOR_D064=YES
~~~

D068 had deliberately made D064, not the Java/Maven invocation, the durable
cross-repository model and had preserved producer-side immutable D064
publication as its explicit future scaling path.

The project owner rejected the practical consequence of leaving Java/Maven in
the website merely to keep executing the producer there.

## Project-owner selection

The project owner explicitly approved Candidate D and refined the candidate with
a construction invariant:

~~~text
APPROVED=D

ARTIFACT_GENERATION=
  one reproducible, dependable Protos-owned command/entry point

ARTIFACT_SET=
  JVM artifact(s)
  + Native Image artifact(s)
  + D064 documentation artifact
  + coherent provenance/checksum envelope

PUBLIC_RELEASE_CREATION=
  separate from artifact generation
~~~

The decisive correction to the earlier Candidate C recommendation was:

~~~text
CLEAN_INTERFACE_BUT_WEBSITE_STILL_BUILDS_PROTOS=INSUFFICIENT
WEBSITE_JDK_MAVEN_FOR_D064=NOT_ACCEPTED_END_STATE
~~~

The desired boundary is instead:

~~~text
protos@X
  -> builds/publishes D064(X)

protos-website
  -> acquires D064(X)
  -> validates provenance X
  -> renders with Node/Astro
~~~

## Approved exact-revision artifact model

The owner-approved refinement joins D064 generation to the normal Protos
artifact-construction plane without joining it to public release selection.

~~~text
BUILD_ARTIFACTS_FOR_X
  -> JVM
  -> Native
  -> D064
  -> manifest/checksums/provenance

CREATE_PUBLIC_RELEASE
  -> optional later milestone operation
  -> explicitly authorized separately
~~~

This preserves the existing release rule that successful `main` /
`-SNAPSHOT` work does not automatically become a public release.

## GITHUB021 preservation result

Preserved from D061/D064/D066/D067/D068:

~~~text
PROTOS_SOURCE_AUTHORITY=KEEP
EXACT_SHA_SOURCE_SELECTION=KEEP
PROTOS_EXTRACTION_AUTHORITY=KEEP
D064_RENDERER_INDEPENDENT_MODEL=KEEP
D064_EXACT_REVISION_PROVENANCE=KEEP
WEBSITE_RENDERER_OWNERSHIP=KEEP
WEBSITE_PROTOS_PARSER=FORBIDDEN
WEBSITE_WRITE_AUTHORITY_TO_PROTOS=NO
D064_SCHEMA_CHANGE=NO
PROTOS_SEMANTIC_CHANGE=NO
~~~

Explicitly superseded from D068 A-prime:

~~~text
WEBSITE_EXECUTES_PRODUCER=NO
D064_EPHEMERAL_ONLY=NO
WEBSITE_REQUIRES_JDK_MAVEN_FOR_D064=NO
~~~

Replacement:

~~~text
PROTOS_EXECUTES_PRODUCER=YES
D064_EXACT_REVISION_PUBLICATION=YES
WEBSITE_CONSUMES_D064_X=YES
~~~

## Implementation routing

A new implementation owner was allocated after the decision:

~~~text
DIST010=guillermomolina/protos#771
NEXT_SLICE=DIST010-A
TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
~~~

DIST010-A owns only the canonical exact-revision artifact-build envelope. It
does not publish a GitHub Release and does not change the website.

The durable publication mechanism follows after that construction boundary is
proven. Any newly exposed material architecture choice around publication
retention/discovery must fail closed into the design process rather than being
silently embedded in implementation.

## Decision authority

The selected architecture is maintained in:

`docs/project/decisions/tooling/D181_EXACT_REVISION_D064_PUBLICATION_AND_DISTRIBUTION_BUILD_BOUNDARY.md`

This evidence file records the approval path and implementation routing; it does
not replace the decision record.
