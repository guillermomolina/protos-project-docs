# DIST010-B — Exact-revision D064 GHCR/OCI publication implementation

Status: **PUBLISHED IMPLEMENTATION — OBSERVED REMOTE PUBLICATION PASS**

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
publication. The observed exact X/H/M publication and anonymous acquisition are
now recorded below.

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


## Observed remote publication closure — 2026-10-02

The human executor completed the real DIST010-B publication against a clean
exact Protos checkout and reported every required gate PASS.

### Exact published identity

~~~text
PRODUCT_HEAD=14558fd9ea6758698ae4a8d4d38a273438212176
WORKTREE_CLEAN=YES

PACKAGE_REFERENCE=ghcr.io/guillermomolina/protos-stdlib-documentation

SOURCE_REVISION=14558fd9ea6758698ae4a8d4d38a273438212176
D064_CONTENT_SHA256=236597164f28f3d735ea329a9b20fbe348f7628347cc1e3e498d1650a12e80e9
OCI_MANIFEST_DIGEST=sha256:19b27bb39e2321333ec570631a821e8d43c2d4c4c98c6cfd9c8150879418bc4e
DISCOVERY_ALIAS=rev-14558fd9ea6758698ae4a8d4d38a273438212176
DISCOVERY_ALIAS_IS_AUTHORITY=NO
~~~

The published product revision is one unrelated PERF026-D2 commit after the
DIST010-B implementation revision. The human explicitly inspected history and
confirmed that the DIST010-B implementation surfaces were unchanged at that
HEAD. No DIST010 source edits were required for observed publication.

### Canonical build and verification

~~~text
CANONICAL_ARTIFACT_SET_BUILD=PASS
CANONICAL_ARTIFACT_SET_VERIFY=PASS
~~~

The exact D064 bytes admitted by the canonical artifact set therefore define H,
and the OCI publication for that exact source revision defines M.

### First publication

~~~text
FIRST_PUBLICATION_EXISTING=NO
FIRST_PUBLICATION_IDEMPOTENT=NO
FIRST_AUTHENTICATED_VERIFICATION=PASS
FIRST_ANONYMOUS_VERIFICATION=FAIL_PACKAGE_NOT_PUBLIC
FIRST_PUBLICATION_STATE=AWAITING_PUBLIC_VISIBILITY
~~~

The anonymous failure was the expected one-time first-package visibility state.
The package was made Public manually in the GitHub package UI. No source change
or second package was required.

An earlier attempt had failed with HTTP 403 before uploading anything because
the current GitHub CLI OAuth token did not include the required
`write:packages` scope. The human refreshed that credential scope. This was a
credential/configuration failure, not a publisher defect and not a partial
publication.

### Idempotent retry and anonymous acquisition

After public visibility was established, the canonical publisher was rerun
idempotently. The human reported the complete retry result:

~~~text
RETRY_SOURCE_REVISION=14558fd9ea6758698ae4a8d4d38a273438212176
RETRY_D064_CONTENT_SHA256=236597164f28f3d735ea329a9b20fbe348f7628347cc1e3e498d1650a12e80e9
RETRY_OCI_MANIFEST_DIGEST=sha256:19b27bb39e2321333ec570631a821e8d43c2d4c4c98c6cfd9c8150879418bc4e

RETRY_EXISTING_PUBLICATION=YES
RETRY_IDEMPOTENT=YES
RETRY_AUTHENTICATED_VERIFICATION=PASS
RETRY_ANONYMOUS_VERIFICATION=PASS
RETRY_PROVENANCE_VERIFICATION=PASS

X_STABLE_ACROSS_RETRY=YES
H_STABLE_ACROSS_RETRY=YES
M_STABLE_ACROSS_RETRY=YES
~~~

The publisher was then run idempotently again after public visibility; the
reported complete retry output matched exactly. Thus the observed remote state
satisfies D182's same-X/same-H retry contract and public anonymous-read
requirement.

### Closure result

~~~text
DIST010_B_OBSERVED_PUBLICATION_STATUS=PASS

PUBLIC_RELEASE_CREATED=NO
D064_SCHEMA_CHANGE=NO
PROTOS_SEMANTIC_CHANGE=NO

SOURCE_CHANGES_REQUIRED=NO
FILES_CHANGED=none

DIST010_A=PASS
DIST010_B_IMPLEMENTATION=PASS
DIST010_B_REMOTE_PUBLICATION=PASS
DIST010_B_ANONYMOUS_ACQUISITION=PASS
DIST010_B_IDEMPOTENT_RETRY=PASS

DIST010_PRODUCER_SIDE_SCOPE=COMPLETE
DIST010_READY_TO_CLOSE=YES
~~~

Package visibility remains a one-time provider configuration requirement when an
equivalent package is created from scratch. The selected Protos package is now
public. D064 retention remains indefinite project policy; provider permanence is
not claimed.

The credential used by the human now has `write:packages` scope. Credential
lifecycle is an operator concern and does not alter the published X/H/M
identity.

## Downstream routing after DIST010

DIST010 completes the producer-side D181/D182 path. The downstream website
consumer remains owned by:

~~~text
guillermomolina/protos-website#5
WEB009
~~~

That issue still has an independent source-coherence blocker:

~~~text
guillermomolina/protos#764
I078
~~~

Therefore completion of DIST010 removes the producer/publication blocker but
does not itself authorize the website refresh until I078's publication
reconciliation is closed.
