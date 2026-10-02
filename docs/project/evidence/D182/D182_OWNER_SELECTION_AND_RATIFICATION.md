# D182 — Owner selection and ratification evidence

Status: **FINAL DECISION EVIDENCE — Candidate A ratified**

Date: 2026-10-02

~~~text
DECISION=D182
DECISION_ISSUE=guillermomolina/protos#773
APPROVED_CANDIDATE=A
PARENT_DECISION=D181/guillermomolina/protos#770
IMPLEMENTATION_OWNER=DIST010/guillermomolina/protos#771
PROTOS_BASELINE=b91d6066eb41b420db4a2884ac4719b2468767fa
DIST010_A_PROJECT_RECORD_REVISION=61d53efdb79ef8453e8e645e3c2b4b82d6e3c5a2
~~~

## Investigation result carried into approval

D182-A investigated the durable remote publication boundary left intentionally
open by D181 after DIST010-A established the local exact-revision artifact set.

The decision compared:

~~~text
A = public GHCR / OCI artifact
B = immutable GitHub Release asset
C = GitHub Actions artifact
D = Maven-style GitHub Packages publication
E = dedicated static/Git exact-revision publication
F = S3-style immutable object publication with conditional writes/Object Lock
~~~

The decisive current-state constraints were:

~~~text
D181_BUILD_RELEASE_SEPARATION=KEEP
D181_WEBSITE_NODE_ONLY_D064_TARGET=KEEP
D064_SCHEMA_CHANGE=NO
EXACT_SHA_SOURCE_AUTHORITY=KEEP
SHORT_RETENTION_ACTIONS_ARTIFACT=INSUFFICIENT
PUBLIC_ANONYMOUS_READ=PREFERRED
CURRENT_CONSUMER=protos-website
CURRENT_REMOTE_NEED=D064(X)
~~~

GitHub Actions artifacts were eliminated as sole authority because their
retention is bounded. GitHub Release assets were retained as excellent
release-distribution machinery but rejected as the general development-SHA
D064 channel because arbitrary useful exact revisions must not become public
releases. Maven Packages were rejected because the documented GitHub Maven
consumption model requires authentication even for public packages and would
introduce unnecessary ecosystem coupling for the Node/Astro consumer.

A dedicated Git/static publication surface remained technically viable but
would require Protos to invent append-only artifact-registry semantics over Git
history. S3/Object Lock provided stronger WORM semantics but imposed a new
cloud/IAM/storage institution for one small JSON consumer.

Candidate A was recommended because it provides standard immutable digest
identity, public anonymous reads, repository-scoped producer authentication,
and bounded migration while reusing a provider/protocol family already present
in the project.

## Project-owner selection

On 2026-10-02 the project owner explicitly approved:

~~~text
APPROVED_CANDIDATE=A
~~~

The exact selected architecture is:

~~~text
BACKEND=GHCR_OCI
PUBLIC_PACKAGE=YES
ANONYMOUS_READ=YES

SOURCE_REVISION=X
D064_CONTENT_SHA256=H
OCI_MANIFEST_DIGEST=M

DISCOVERY_ALIAS=rev-X
DISCOVERY_ALIAS_IS_AUTHORITY=NO

CONSUMER_PERSISTS=X+H+M
NORMAL_CONSUMER_PULLS_BY=M

PUBLICATION_SOURCE=VERIFIED_DIST010_ARTIFACT_SET_ONLY
D064_REGENERATION_DURING_PUBLICATION=NO

PUBLICATION_TRIGGER=EXPLICITLY_REQUIRED_EXACT_REVISION
EVERY_MAIN_REVISION_PUBLICATION=NO
EVERY_ARTIFACT_SET_PUBLICATION=NO

D064_RETENTION=INDEFINITE_PROJECT_POLICY
AUTOMATIC_D064_CLEANUP=NO
PROVIDER_PERMANENCE_CLAIM=NO

REMOTE_PUBLICATION_GRANULARITY=D064_PLUS_OCI_CONTENT_METADATA

PUBLIC_RELEASE_SEPARATE=YES
RELEASE_MAY_REUSE_D064_H=YES
~~~

## Identity and failure evidence

The selected model deliberately separates first-time discovery from immutable
authority:

~~~text
X -> rev-X -> M
M -> D064 layer H
sha256(downloaded D064) == H
D064.provenance.repository == guillermomolina/protos
D064.provenance.revision == X
~~~

Once admitted, `X/H/M` is the consumer lock. `rev-X` is not consulted as
authority during normal reproducible acquisition.

Publication is idempotent only when an existing exact-revision alias resolves
to the same content/provenance. A different `H` for the same `X` fails closed.

Administrative tag mutation or package deletion remains possible at the
provider level. The approval therefore does not claim WORM tags or provider
permanence:

~~~text
CONTENT_IMMUTABILITY_BY_DIGEST=YES
REVISION_TAG_PROVIDER_WORM=NO
RETENTION=PROJECT_POLICY
PROVIDER_PERMANENCE_GUARANTEE=NO
~~~

## GITHUB021 preservation result

Candidate A preserves every D181 owner-approved invariant:

~~~text
OWNER_INVARIANT_1=PRESERVED
OWNER_INVARIANT_2=PRESERVED
OWNER_INVARIANT_3=PRESERVED
OWNER_INVARIANT_4=PRESERVED

EXACT_SHA_SOURCE_AUTHORITY=KEEP
PROTOS_OWNS_EXTRACTION=KEEP
WEBSITE_OWNS_RENDERING=KEEP
D064_SCHEMA_CHANGE=NO
PROTOS_SEMANTIC_CHANGE=NO

DECISION_INVARIANT_CONSISTENCY=PASS
OWNER_INVARIANT_REOPEN_REQUIRED=NO
~~~

No D181 invariant was reopened by the owner selection.

## Implementation routing

Ratification releases the next implementation slice:

~~~text
NEXT_SLICE=DIST010-B
TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
PUBLICATION_BACKEND=GHCR_OCI
PUBLIC_RELEASE=NO
WEBSITE_CHANGE=NO
~~~

DIST010-B owns the producer-side remote publication path. It must consume the
already-verified DIST010 artifact set and publish its exact D064 member rather
than rerunning the producer.

`guillermomolina/protos-website#5` remains the downstream consumer owner after
DIST010-B proves the public exact-revision acquisition contract.

## Decision authority

The selected architecture is maintained in:

`docs/project/decisions/tooling/D182_EXACT_REVISION_D064_GHCR_OCI_PUBLICATION_RETENTION_AND_DISCOVERY_BOUNDARY.md`

This evidence file records the owner selection, investigation result, invariant
review, and implementation routing. It does not replace the decision record.
