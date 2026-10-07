# WEB009 / D181 — D068 producer-boundary routing evidence

Date: 2026-10-02

## Coordination identity

~~~text
TRIGGER_WORK=WEB009
WEBSITE_ISSUE=guillermomolina/protos-website#5
NEW_DECISION=D181
DECISION_ISSUE=guillermomolina/protos#770
CURRENT_AUTHORITY=D068/#330
DECISION_STATE=OPEN_INVESTIGATION_REQUIRED
~~~

This record captures the routing evidence that caused WEB009 to stop before
changing the website producer/toolchain boundary. It is non-normative and does
not select a D181 candidate.

## Current ratified authority

D068 / #330 is closed and ratified with Candidate A-prime.

Its durable architecture is conceptually:

~~~text
website selects exact Protos SHA
  -> same exact Protos checkout owns source + extractor
  -> current producer toolchain compiles/runs the Protos-owned extractor
  -> D064 JSON
  -> website validates/renders
~~~

D068 deliberately made D064 the durable cross-repository model rather than the
current Java/Maven invocation details.

D068 also explicitly retained producer-side exact-revision D064 publication as
a future scaling path, but did not select it for the then-current single
consumer.

Therefore WEB009 cannot move producer execution responsibility to Protos
without an explicit follow-up decision.

## Current implementation pressure

At the WEB009 audit point, the website contains build responsibilities that are
implementation details of the Protos-owned producer:

~~~text
WEBSITE_DEVCONTAINER_BASE=maven:3.9.16-eclipse-temurin-21
WEBSITE_NODE_RUNTIME=22.x copied into Maven/JDK image
WEBSITE_KNOWS_EXTRACTOR_CLASS=YES
WEBSITE_KNOWS_MAVEN_COMPILE=YES
WEBSITE_KNOWS_TARGET_CLASSES=YES
WEBSITE_HAS_PINNED_MAVEN_JDK_FALLBACK=YES
~~~

The relevant website producer adapter is:

~~~text
scripts/materialize-protos-library-api.mjs
~~~

It currently:

- detects JDK >= 21 and Maven;
- compiles the exact Protos checkout with Maven;
- invokes
  `com.guillermomolina.protos.documentation.ProtosStandardLibraryDocumentationExtractor`;
- knows the `target/classes` layout;
- falls back to a pinned Maven/JDK container;
- validates the resulting D064 provenance/schema; and
- renders the D064 model.

The extractor implementation and its tests already live in
`guillermomolina/protos`.

This raised a concrete ownership question: whether producer compilation and
artifact generation should remain website build responsibilities or become a
Protos-owned build/publication responsibility, leaving the website as a D064
consumer/renderer.

## Sibling-repository trigger

The `guillermomolina/protos-benchmarks` development container already defines
a sibling Protos bind mount:

~~~text
source=${localWorkspaceFolder}/../protos
target=/workspaces/protos
type=bind
consistency=cached
~~~

The project owner identified that mount as an existing pattern for consuming
Protos-owned built state from a companion repository.

D181 must verify the exact benchmark consumer contract before treating it as
precedent. The mount itself is evidence of an existing sibling-repository
development topology; it is not automatic authority for website CI,
provenance, or artifact retention.

## Why D181 is independently necessary

The proposed change affects durable cross-repository architecture:

~~~text
producer ownership
producer execution ownership
artifact ownership
artifact publication/retention
local acquisition
isolated CI acquisition
exact-revision provenance
toolchain ownership
~~~

Those boundaries were explicitly selected by D068 and therefore must not be
changed inside WEB009 as an implementation convenience.

D181 / #770 was allocated as the next free top-level D identifier after D180.
Post-create search found exactly one D181 Issue.

## D181 investigation boundary

D181 must compare at least:

~~~text
A  retain D068 A-prime exactly
B  sibling Protos artifact/build consumption for local development only
C  Protos-owned documentation-artifact build target
D  producer-side exact-revision D064 publication
E  release/distribution-owned producer/artifact
F  any materially distinct supported minimal mechanism
~~~

The investigation must distinguish a local-development optimization from a
complete transfer of producer execution responsibility.

In particular, adding a sibling mount alone does not explain how an isolated
website CI/build obtains D064.

## Preserved invariants

Until D181 is explicitly approved and durably ratified:

~~~text
D068_AUTHORITY=KEEP
D064_SCHEMA=KEEP
PROTOS_OWNS_EXTRACTION=KEEP
WEBSITE_OWNS_RENDERING=KEEP
EXACT_SHA_SOURCE_AUTHORITY=KEEP
WEBSITE_PROTOS_PARSER=FORBIDDEN
PROTOS_SEMANTIC_CHANGE=NO
STANDARD_LIBRARY_SEMANTIC_CHANGE=NO
~~~

No repository implementation change is authorized by this routing record.

## WEB009 consequence

WEB009 already had an independent source-coherence blocker around I078.

D181 adds a second independent gate:

~~~text
WEB009_STATUS=BLOCKED

SOURCE_COHERENCE_GATE=I078/#764
PRODUCER_BOUNDARY_GATE=D181/#770

WEBSITE_SOURCE_REFRESH_IMPLEMENTATION=NOT_AUTHORIZED_YET
WEBSITE_TOOLCHAIN_REFACTOR=NOT_AUTHORIZED_YET
~~~

The two gates must not be conflated:

- I078 determines whether the selected Protos source revision is a coherent
  publication point.
- D181 determines how the website may obtain D064 from that exact revision.

## Next slice

~~~text
NEXT_SLICE=D181-A
TYPE=INVESTIGATION
SHELL_EXECUTION=NO
REPOSITORY_MUTATION=NO
IMPLEMENTATION_AUTHORIZED=NO
~~~

D181-A must build the complete current-state and candidate comparison packet
required by `AGENTS.work/DESIGN.md`, including the D068 invariant/delta check,
before requesting project-owner approval.
