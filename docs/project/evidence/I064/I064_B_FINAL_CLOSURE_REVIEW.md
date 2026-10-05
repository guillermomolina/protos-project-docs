# I064-B — final closure review

Date: 2026-10-05

Owning work item: `I064 / guillermomolina/protos#667`

Decision authority: `D171 / guillermomolina/protos#642`, Candidate B.

Implementation checkpoint:
`docs/project/evidence/I064/I064_A_PUBLIC_CAPTURETREE_REMOVAL_IMPLEMENTATION.md`.

## Exact product state

~~~text
I064_A_REVISION=812b3f0b29dcba59562e3d03930c163db8529410
CURRENT_PROTOS_HEAD=812b3f0b29dcba59562e3d03930c163db8529410
I064_A_IS_CURRENT_HEAD=YES
POST_I064_COMMITS=NONE
IMPLEMENTATION_VERSION=0.3.214-SNAPSHOT
SPECIFICATION_REVISION=0.1.443
~~~

Because I064-A is still the current product HEAD, there is no post-I064 product
delta to audit for regression.

## Falsifying implementation review

The final published product state satisfies D171 Candidate B.

The standard Filesystem construction path exposes `open`, `replace`,
`remove` and `entries`; it no longer installs `captureTree`.
Captured Filesystem materialization remains host-internal and produces the same
ordinary read-only Filesystem surface without a capture selector.

The former guest-facing capture flow and backend methods are absent from the
runtime protocol. The remaining source-backend custody path is explicitly
host-only:

~~~text
ProtosCapturedFilesystemCustody.captureSelectedRoot(...)
  -> ProtosNioReadOnlyTreeFilesystemBackend.captureRootForHostCustody()
  -> captureSelectedDirectory(...)
  -> secure recursive no-follow traversal
  -> ProtosNioCapturedTreeFilesystemBackend
~~~

The traversal retains `SecureDirectoryStream` confinement and
`LinkOption.NOFOLLOW_LINKS` selection/classification. The captured backend
retains immutable backing, `entries`, read-only `open`, independent
host-only `readRegularResource(...)`, lease/release handling and
Actor-domain rematerialization.

The package verification path continues to consume one already-selected root,
verify a read-only Filesystem over that exact immutable custody, and retain the
same custody for later reads rather than reopening the original mutable source
or store Path.

## Normative review

Specification revision `0.1.443` states that the standard Filesystem protocol
has no `captureTree` operation and defines no guest snapshot/clone/freeze
equivalent for arbitrary subtrees.

It retains `Filesystem.entries`, Filesystem authority/confinement and the
host-provisioned captured-Filesystem contract. A captured Filesystem remains an
ordinary read-only Filesystem consumed through retained operations and cannot
mint another Filesystem.

The guide/design reconciliation preserves D046/I024 as historical evidence while
making D171 supersession explicit.

## Validation and publication evidence

Maintainer-reported validation for the exact published candidate:

~~~text
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_FULL_VALIDATION=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
PRODUCT_PUBLICATION=PUSHED
~~~

Exact-sha GitHub Actions publication validation:

~~~text
CI_RUN_ID=37322294900
CI_RUN_NUMBER=2160
CI_HEAD=812b3f0b29dcba59562e3d03930c163db8529410
CI_WORKFLOW=CI
CI_EVENT=push
CI_STATUS=completed
CI_CONCLUSION=success
CI_JOB=test
CI_JOB_CONCLUSION=success
~~~

## Closure matrix

~~~text
D171_PUBLIC_CAPTURETREE_REMOVAL=PASS
FILESYSTEM_ENTRIES_PRESERVED=PASS
PLAT012_CUSTODY_PRESERVED=PASS
SECURE_CAPTURE_ENGINE_PRESERVED=PASS
PACKAGE_TOCTOU_GUARANTEE_PRESERVED=PASS
NORMATIVE_SPEC_RECONCILIATION=PASS
FOCUSED_VALIDATION=PASS
REQUIRED_FULL_VALIDATION=PASS
PUBLICATION_VALIDATION=PASS
~~~

Additional preservation checks:

~~~text
PUBLIC_ARBITRARY_SUBTREE_CAPTURE_REMOVED=PASS
IMMUTABLE_CAPTURED_BACKING_PRESERVED=PASS
READ_ONLY_CAPTURED_FILESYSTEM_MATERIALIZATION=PASS
VERIFY_AND_USE_SAME_CUSTODY=PASS
NO_SOURCE_REOPEN=PASS
~~~

## GitHub coordination closure gate

The native hierarchy was re-read from GitHub and is established:

~~~text
FORMAL_IDENTIFIER_UNIQUE=PASS
FAMILY=family:I PASS
CANONICAL_STATUS=PASS_ON_CLOSURE
ASSIGNEE_INVARIANT=PASS
NATIVE_PARENT=#642 PASS
PROJECT_ROUTING=PASS_VIA_ISSUE_AUTOMATION
DECISION_APPROVAL_PROVENANCE=D171_ALREADY_RATIFIED
DECISION_INVARIANT_CONSISTENCY=PASS
~~~

Durable closure evidence is required and is this I064 evidence set.

~~~text
ISSUE_CLOSURE_COMMENT=REQUIRED_BEFORE_CLOSE
CLOSURE_EVIDENCE_IDENTIFIED=PASS
DURABLE_RECORD_DECISION=REQUIRED
REQUIRED_DURABLE_PUBLICATION=PASS_AFTER_THIS_RECORD_IS_PUBLISHED
~~~

## Final result

~~~text
I064_PRODUCT_COMPLETE=YES
I064_CLOSURE_AUTHORIZED=YES
NEXT_TECHNICAL_SLICE=NONE
NEXT_IMPLEMENTATION_SLICE=NONE
BLOCKER_CLASS=NONE
~~~

No I064-C product slice is warranted. Any future request for a public immutable
tree-capture facility would require new demand and new design authority rather
than reopening I064 by implication.

AI assistance: this final closure record was drafted with ChatGPT from the exact
published Protos revision, current normative/runtime source, live GitHub Issue
hierarchy, exact-sha GitHub Actions results and maintainer-reported local
validation.
