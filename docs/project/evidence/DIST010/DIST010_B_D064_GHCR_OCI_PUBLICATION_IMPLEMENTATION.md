# DIST010-B — Exact-revision D064 GHCR/OCI publication implementation

Status: **PUBLISHED IMPLEMENTATION — REMOTE PUBLICATION PENDING**

Date: 2026-10-02

## Identity

~~~text
WORK=DIST010-B
OWNING_ISSUE=guillermomolina/protos#771
PROTOS_REVISION=f366ae819291a3ad9a6590c8cc4a8cd2b74d1e93
COMMIT_SUBJECT=DIST010-B: explicit exact-revision D064 GHCR/OCI publication
RATIFIED_ARCHITECTURE=D182/#773 Candidate A
DIST010_A_ORIGINAL_REVISION=b91d6066eb41b420db4a2884ac4719b2468767fa
~~~

The human executor reported that the DIST010-B implementation was pushed and all
requested tests were green.

This record distinguishes source implementation from the first real GHCR
publication. No remote X/H/M publication is claimed here.

## Published product delta

~~~text
Makefile
dist/README.md
dist/publish_d064_oci.py
dist/test_publish_d064_oci.py
~~~

The published repository entry point is:

~~~text
make artifacts-publish-d064
~~~

which delegates to:

~~~text
python3 dist/publish_d064_oci.py --artifact-set target/artifact-set
~~~

The existing DIST010-A entry points remain:

~~~text
make artifacts
make artifacts-verify
~~~

Artifact construction and D064 publication remain separate explicit operations.

## Selected publication identity

The implementation realizes the ratified D182 identity model:

~~~text
SOURCE_REVISION=X
D064_CONTENT_SHA256=H
OCI_MANIFEST_DIGEST=M
DISCOVERY_ALIAS=rev-X
DISCOVERY_ALIAS_IS_AUTHORITY=NO
CONSUMER_LOCK=X+H+M
~~~

The selected package and media types are:

~~~text
PACKAGE_REFERENCE=ghcr.io/guillermomolina/protos-stdlib-documentation
OCI_ARTIFACT_TYPE=application/vnd.protos.stdlib-documentation.v1
D064_LAYER_MEDIA_TYPE=application/vnd.protos.stdlib-documentation.v1+json
OCI_MANIFEST_MEDIA_TYPE=application/vnd.oci.image.manifest.v1+json
OCI_CONFIG_MEDIA_TYPE=application/vnd.oci.empty.v1+json
~~~

The publisher speaks the OCI Distribution API directly through the Python
standard library. It does not require Docker, ORAS, Maven, or a second D064
producer.

## Publication admission

The implementation first invokes the existing DIST010 artifact-set verifier and
selects exactly one manifest member of kind:

~~~text
stdlib-documentation
~~~

It then re-reads the exact bytes that will be published, recomputes H, and
requires internal provenance:

~~~text
kind=repositoryRevision
repository=guillermomolina/protos
revision=X
~~~

The publication path therefore consumes only an already-admitted D064 member.
It does not call:

~~~text
Maven
the D064 extractor
make artifacts
release preparation
Git tag creation
GitHub Release creation
git push
package cleanup/deletion
~~~

## Deterministic OCI representation

The OCI manifest contains no timestamp. The exact D064 bytes are one layer and
the exact source revision is carried as OCI revision metadata. The same admitted
X/H therefore renders the same manifest bytes and the same content-addressed M.

The alias remains only:

~~~text
rev-X
~~~

and is never treated as immutable authority.

## Retry and conflict contract

Before writing, the publisher resolves rev-X.

~~~text
rev-X absent
  -> upload required blobs
  -> publish deterministic manifest

rev-X present with same admitted X/H
  -> verify by immutable digest
  -> idempotent success

rev-X present with different H or wrong provenance
  -> fail closed
  -> never overwrite/move the alias

uncertain manifest PUT outcome
  -> resolve and verify
  -> never blindly repush
~~~

A local lock also prevents concurrent local publication attempts against the same
artifact-set directory.

## Post-publication verification

After publication the implementation requires:

~~~text
rev-X resolves to expected M
authenticated pull by M = PASS
downloaded D064 sha256 = H
downloaded D064 provenance revision = X
~~~

It then performs a second verification through a distinct credential-free
Registry instance. Publisher credentials are not exposed to that consumer path.

A private first-created GHCR package therefore does not count as completion.
The publisher returns the explicit state:

~~~text
ANONYMOUS_VERIFICATION=FAIL_PACKAGE_NOT_PUBLIC
D064_PUBLICATION=AWAITING_PUBLIC_VISIBILITY
~~~

and exits with status 3. After the package is made public, rerunning the same
publisher must take the same-X/same-H idempotent path and require anonymous
verification to pass.

## Machine-readable result

A successful real publication emits:

~~~text
SOURCE_REVISION=<X>
D064_CONTENT_SHA256=<H>
OCI_MANIFEST_DIGEST=<M>
DISCOVERY_ALIAS=rev-<X>
PACKAGE_REFERENCE=ghcr.io/guillermomolina/protos-stdlib-documentation
EXISTING_PUBLICATION=YES|NO
IDEMPOTENT_RETRY=YES|NO
AUTHENTICATED_VERIFICATION=PASS
ANONYMOUS_VERIFICATION=PASS
D064_PROVENANCE_VERIFICATION=PASS
PUBLIC_RELEASE_CREATED=NO
D064_PUBLICATION=PASS
~~~

No concrete X/H/M values are recorded yet because the human report supplied
implementation/test publication evidence, not a successful real GHCR
publication transcript.

## Focused test coverage

The new focused test suite uses an in-memory GHCR model and covers the bounded
publication policy, including artifact-set admission, D064 identity/provenance,
OCI blob/manifest behavior, alias discovery, idempotent same-X/same-H retry,
conflicting identity rejection, uncertain-write recovery, post-push digest
verification, content/provenance tampering, anonymous-read failure, and
credential separation.

The human executor reported:

~~~text
PRODUCT_PUBLICATION=YES
ALL_REQUESTED_TESTS=GREEN
TEST_FAILURES=NONE_REPORTED
~~~

No command transcript is fabricated beyond that report.

## Release and retention boundaries

The implementation preserves:

~~~text
EVERY_MAIN_REVISION_PUBLICATION=NO
BUILD_IMPLIES_PUBLICATION=NO
PUBLICATION_IMPLIES_PUBLIC_RELEASE=NO
PUBLIC_RELEASE_CREATED=NO

D064_RETENTION=INDEFINITE_PROJECT_POLICY
AUTOMATIC_D064_CLEANUP=NO
ORDINARY_D064_DELETION=NO

D064_SCHEMA_CHANGE=NO
PROTOS_SEMANTIC_CHANGE=NO
~~~

No workflow was added that publishes every main revision.

## Remaining closure gate

DIST010-B implementation is published and tested, but D182 requires observed
remote evidence before the publication boundary can be called complete.

The remaining gate is:

~~~text
1. from clean exact Protos HEAD, build canonical artifact set
2. verify the exact artifact set
3. explicitly publish D064 through the canonical publisher
4. if GHCR created the package private, make that package public
5. rerun publication idempotently
6. retain exact X/H/M output
7. require credential-free acquisition by M to PASS
~~~

Therefore:

~~~text
DIST010_B_IMPLEMENTATION=PUBLISHED
DIST010_B_IMPLEMENTATION_VALIDATION=PASS_HUMAN_REPORTED
REMOTE_PUBLICATION_EXECUTED=NOT_REPORTED
REMOTE_X_H_M=NOT_YET_RETAINED
ANONYMOUS_GHCR_ACQUISITION=NOT_YET_RETAINED
DIST010_COMPLETE=NO
~~~

The next slice is the observed remote publication/anonymous-acquisition closure
for DIST010-B itself. It is not a new architecture decision and does not require
a new Issue.

## Cross references

- guillermomolina/protos#771 — DIST010.
- guillermomolina/protos#773 — D182, Candidate A ratified.
- docs/project/evidence/DIST010/DIST010_A_CANONICAL_EXACT_REVISION_ARTIFACT_SET.md.
- docs/project/decisions/tooling/D182_EXACT_REVISION_D064_GHCR_OCI_PUBLICATION_RETENTION_AND_DISCOVERY_BOUNDARY.md.
- docs/project/evidence/D182/D182_OWNER_SELECTION_AND_RATIFICATION.md.
