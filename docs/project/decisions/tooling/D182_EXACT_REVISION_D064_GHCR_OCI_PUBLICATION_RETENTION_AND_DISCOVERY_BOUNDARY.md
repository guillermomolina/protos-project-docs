# D182 — Exact-revision D064 GHCR/OCI publication, retention, and discovery boundary

Status: **RATIFIED — Candidate A selected**

Explicit project-owner approval: **2026-10-02**

Decision issue: `guillermomolina/protos#773`

Implementation owner: `DIST010 / guillermomolina/protos#771`

Product baseline inspected for the decision:

~~~text
PROTOS_REVISION=b91d6066eb41b420db4a2884ac4719b2468767fa
DIST010_A_PROJECT_RECORD_REVISION=61d53efdb79ef8453e8e645e3c2b4b82d6e3c5a2
~~~

Parent architecture: `D181 / guillermomolina/protos#770`, Candidate D ratified.

Nature: durable non-normative tooling/distribution architecture.

Observable Protos language effect: **none**.

D064 schema effect: **none**.

## Decision

D182 selects **Candidate A — public GHCR OCI publication for exact-revision D064**.

The selected contract intentionally distinguishes source authority, immutable
content identity, immutable remote identity, and discovery naming:

~~~text
SOURCE_REVISION=X
D064_CONTENT_SHA256=H
OCI_MANIFEST_DIGEST=M
DISCOVERY_ALIAS=rev-X
~~~

`X` is the authoritative Protos source revision. `H` is the SHA-256 of the
exact D064 bytes admitted from the verified DIST010 artifact set. `M` is the
immutable OCI manifest digest for the remotely published object. `rev-X` is
only a discovery alias and is never artifact authority.

A consumer that has resolved and admitted a publication persists all three
authoritative facts:

~~~text
X + H + M
~~~

Normal reproducible consumption then acquires by `M`, verifies the downloaded
D064 bytes against `H`, and independently verifies internal D064 provenance
against `X`.

## Publication backend

The durable remote backend is the public GitHub Container Registry using the
OCI distribution model.

Conceptually:

~~~text
verified DIST010 artifact set for X
    -> select already-built D064 member
    -> H = sha256(D064 bytes)
    -> publish D064 as OCI content
    -> M = immutable OCI manifest digest
    -> publish rev-X only as discovery alias
~~~

The exact GHCR package name, OCI media-type strings, and client implementation
are implementation-local provided they preserve the D182 identity and failure
contract. A non-container OCI client such as ORAS may be used; D182 does not
require Docker CLI semantics for the JSON artifact.

## Public/anonymous read contract

The selected package must be public and must be anonymously readable by the
website consumer.

~~~text
WEBSITE_GHCR_CREDENTIAL_REQUIRED=NO
PRODUCER_PUBLICATION_AUTH=GITHUB_TOKEN_OR_EQUIVALENT_REPOSITORY_SCOPED_CREDENTIAL
PRODUCER_REQUIRED_PERMISSION=packages:write
~~~

The consumer must fail closed if the selected publication cannot be acquired
anonymously. D182 does not authorize storing a package-read credential in
`protos-website` merely to consume public D064.

## Publication admission

Remote publication may admit only the D064 member of an already-verified
canonical DIST010 exact-revision artifact set.

Required ordering:

~~~text
build artifact set for X
verify ARTIFACT-SET.json for X
select recorded D064 member
verify sha256(D064) == H
verify D064 provenance.repository == guillermomolina/protos
verify D064 provenance.revision == X
publish those exact bytes
~~~

Forbidden:

~~~text
remote publication
    -> rerun Maven
    -> rerun Java extractor
    -> regenerate a second D064 candidate
~~~

Publication owns transport. It does not become a second D064 producer.

## Discovery and consumer pinning

The revision-derived alias is:

~~~text
rev-X
~~~

or an implementation-equivalent exact-revision tag with the same semantics.

The alias exists only to perform first-time `X -> M` discovery. Because an OCI
tag can be administratively changed or deleted, it must not be treated as
immutable authority.

After discovery and verification, the downstream lock records conceptually:

~~~text
sourceRevision = X
d064ContentSha256 = H
d064ManifestDigest = M
~~~

A normal website build pulls by `M`, not by `rev-X`.

Therefore moving or deleting `rev-X` cannot silently change bytes consumed by
an already-admitted `X/H/M` lock.

## Publication retry and race model

Publication is retryable and fail closed.

Before creating new state, the publisher resolves any existing `rev-X`.

~~~text
rev-X absent
  -> publish D064 and bind rev-X to resulting M

rev-X present and resolves to the same H/X
  -> successful idempotent no-op

rev-X present and resolves to different bytes, different H, or inconsistent X
  -> fail closed
  -> do not move or replace the alias
~~~

Ordinary same-repository publication should serialize by exact source revision
so concurrent canonical publishers do not race for `X`.

Provider-level compare-and-create/WORM semantics for an OCI tag are **not**
assumed. Post-publication resolution and verification are mandatory.

Two different D064 byte sequences proposed for the same exact Protos revision
are an exact-revision identity conflict, not two acceptable package versions.

## Retention and deletion

D064 publication is demand-driven rather than automatic for every commit:

~~~text
EVERY_MAIN_REVISION_PUBLICATION=NO
EVERY_ARTIFACT_SET_BUILD_PUBLICATION=NO

PUBLISH_WHEN=
  explicitly required by a downstream exact source lock
  OR selected for public-release composition
  OR another explicit durable consumer requires it
~~~

Once successfully published:

~~~text
D064_RETENTION=INDEFINITE_PROJECT_POLICY
AUTOMATIC_D064_CLEANUP=NO
ORDINARY_D064_DELETION=NO
PROVIDER_PERMANENCE_GUARANTEE=NO
~~~

`indefinite` is a Protos project policy, not a claim that GitHub promises
permanent storage. Package administrators and provider/account lifecycle can
still remove state.

Deletion of the authoritative `M/H` object is availability- and
authority-breaking for historical locks. Cleanup therefore must never delete
an admitted D064 merely because it is old, is a SNAPSHOT revision, or is no
longer referenced by the current website lock.

A future backend migration may rehost the exact same D064 bytes `H` before
retiring the old backend. D064 itself does not change merely because the
transport changes.

## Remote publication granularity

D182 selects the smallest remote envelope required by the current consumer:

~~~text
REMOTE_PUBLICATION_GRANULARITY=
  D064 JSON + OCI immutable content/manifest metadata
~~~

D067 coverage, JVM artifacts, Native artifacts, `SHA256SUMS`, and the complete
`ARTIFACT-SET.json` remain part of the canonical verified local DIST010
construction envelope. They are not required to use the same remote
publication channel merely because they are built together.

OCI layer/manifest digests supply the transport/content metadata; D182 does not
require a redundant standalone `.sha256` sidecar for the public D064 object.

Additional layers such as D067 may be added later if a real downstream
consumer requires them, without changing the D064 schema or source identity.

## Public release composition

Artifact publication remains separate from public release publication:

~~~text
PUBLISH_D064_FOR_X != CREATE_PUBLIC_RELEASE_X
~~~

A later explicitly selected public release at revision `X` reuses the already
established D064 bytes `H`; it must not regenerate conflicting documentation
bytes for the same exact revision.

A GitHub Release may duplicate the exact D064 bytes as a user-convenience asset
provided its SHA-256 is exactly `H`. Such a duplicate does not automatically
replace the OCI D064 authority.

## Failure contract

~~~text
PUBLICATION_UNAVAILABLE=FAIL_CLOSED
PARTIAL_PUBLICATION=NOT_ADMITTED
UNKNOWN_RETRY_OUTCOME=RESOLVE_AND_VERIFY
SAME_X_SAME_H=IDEMPOTENT_SUCCESS
SAME_X_DIFFERENT_H=FAIL_CLOSED_IDENTITY_CONFLICT
DISCOVERY_ALIAS_MISSING=FAIL_CLOSED_FOR_X_ONLY_DISCOVERY
DISCOVERY_ALIAS_MUTATED=AUTHORITY_INCIDENT_FOR_NEW_DISCOVERY
CONTENT_OBJECT_DELETED=AUTHORITY_AND_AVAILABILITY_BREAK
PRODUCER_CREDENTIAL_MISSING=PUBLICATION_UNAVAILABLE
PUBLIC_CONSUMER_CREDENTIAL_REQUIRED=CONFIGURATION_FAILURE
PROVIDER_OUTAGE=AVAILABILITY_ONLY
CONSUMER_PINS_UNPUBLISHED_X=FAIL_CLOSED
~~~

An existing consumer already pinned to `X/H/M` remains independent of
`rev-X` mutation or deletion as long as the content object addressed by `M`
still exists.

## Strongest counterargument

The strongest argument against Candidate A is that GHCR provides immutable
digest addressing but does not make the revision tag write-once and does not
promise package permanence. Alias non-mutation and indefinite retention remain
project policy rather than provider-enforced WORM.

A dedicated object store with conditional-create and Object Lock could enforce
stronger first-write and deletion semantics. D182 does not select that larger
institution because the current requirement is one small public JSON consumer,
OCI already supplies immutable content identity, Protos already uses GHCR, and
the stronger WORM requirement has not been established.

~~~text
CONTENT_IMMUTABILITY_BY_DIGEST=PROVIDER_MECHANISM
TAG_NON_MUTATION=PROJECT_INVARIANT
RETENTION_INDEFINITE=PROJECT_POLICY
PROVIDER_PERMANENCE_GUARANTEE=NO
~~~

## D181 invariant/delta review

D182 Candidate A preserves all owner-approved D181 invariants:

~~~text
OWNER_INVARIANT_1=
  artifact generation remains reproducible repository machinery, not release creation

OWNER_INVARIANT_2=
  JVM + Native + D064 still share one exact-revision construction envelope

OWNER_INVARIANT_3=
  artifact construction/publication and public release selection remain separate

OWNER_INVARIANT_4=
  protos-website does not retain JDK/Maven merely to generate D064

D064_SCHEMA_CHANGE=NO
PROTOS_SEMANTIC_CHANGE=NO
EXACT_SHA_SOURCE_AUTHORITY=KEEP
PROTOS_OWNS_EXTRACTION=KEEP
WEBSITE_OWNS_RENDERING=KEEP
~~~

The new D182 delta is bounded to:

~~~text
PUBLICATION_BACKEND=GHCR_OCI
REMOTE_IMMUTABLE_ID=OCI_MANIFEST_DIGEST
DISCOVERY_ALIAS=EXACT_REVISION_TAG_ONLY
RETENTION_POLICY=INDEFINITE_NO_AUTOMATIC_CLEANUP
PUBLICATION_RETRY=IDEMPOTENT_AND_FAIL_CLOSED
REMOTE_GRANULARITY=D064_PLUS_OCI_CONTENT_METADATA
~~~

~~~text
DECISION_INVARIANT_CONSISTENCY=PASS
OWNER_INVARIANT_REOPEN_REQUIRED=NO
~~~

## Implementation routing

D182 releases the blocked implementation slice:

~~~text
NEXT_SLICE=DIST010-B
TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
GOAL=durable exact-revision D064 GHCR/OCI publication
PUBLIC_RELEASE=NO
WEBSITE_CHANGE=NO
~~~

DIST010-B must keep `make artifacts` as construction-only machinery and add a
separate explicit publication path which consumes the verified artifact set.

After DIST010-B supplies a verified public `X/H/M` publication contract,
`guillermomolina/protos-website#5` owns the consumer-side change from website
producer execution to anonymous D064 acquisition and `X/H/M` validation.

## Provider references used by D182-A

- GitHub Container registry: `https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry`.
- GitHub package deletion/restoration: `https://docs.github.com/en/packages/learn-github-packages/deleting-and-restoring-a-package`.
- GitHub Packages billing: `https://docs.github.com/en/billing/concepts/product-billing/github-packages`.
- OCI Distribution Specification: `https://github.com/opencontainers/distribution-spec/blob/main/spec.md`.
- ORAS documentation: `https://oras.land/docs/quickstart/`.

## References

- `guillermomolina/protos#773` — D182 decision issue.
- `guillermomolina/protos#771` — DIST010 implementation owner.
- `guillermomolina/protos#770` — D181 parent architecture.
- `guillermomolina/protos-website#5` — WEB009 downstream consumer.
- `docs/project/decisions/tooling/D181_EXACT_REVISION_D064_PUBLICATION_AND_DISTRIBUTION_BUILD_BOUNDARY.md`.
- `docs/project/evidence/DIST010/DIST010_A_CANONICAL_EXACT_REVISION_ARTIFACT_SET.md`.
- `docs/project/evidence/D182/D182_OWNER_SELECTION_AND_RATIFICATION.md`.
