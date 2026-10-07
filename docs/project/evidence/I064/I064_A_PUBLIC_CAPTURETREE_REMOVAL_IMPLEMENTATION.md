# I064-A — public Filesystem.captureTree removal implementation

Date: 2026-10-05

Owning work item: `I064 / guillermomolina/protos#667`

Decision authority: `D171 / guillermomolina/protos#642`, Candidate B.

## Exact published product state

~~~text
PROTOS_REVISION=812b3f0b29dcba59562e3d03930c163db8529410
COMMIT_SUBJECT=I064-A: remove public Filesystem.captureTree while preserving runtime custody
IMPLEMENTATION_VERSION=0.3.214-SNAPSHOT
SPECIFICATION_REVISION=0.1.443
PRODUCT_PUBLICATION=PUSHED
~~~

The exact revision is current Protos HEAD at this checkpoint.

## Implemented outcome

D171 Candidate B is implemented.

~~~text
D171_PUBLIC_CAPTURETREE_REMOVAL=IMPLEMENTED
FILESYSTEM_ENTRIES_PRESERVED=IMPLEMENTED
PUBLIC_ARBITRARY_SUBTREE_CAPTURE_REMOVED=IMPLEMENTED
PLAT012_CUSTODY_PRESERVED=IMPLEMENTED
SECURE_CAPTURE_ENGINE_PRESERVED=IMPLEMENTED
IMMUTABLE_CAPTURED_BACKING_PRESERVED=IMPLEMENTED
READ_ONLY_CAPTURED_FILESYSTEM_MATERIALIZATION=IMPLEMENTED
VERIFY_AND_USE_SAME_CUSTODY=IMPLEMENTED
NO_SOURCE_REOPEN=IMPLEMENTED
PACKAGE_TOCTOU_GUARANTEE_PRESERVED=IMPLEMENTED
NORMATIVE_SPEC_RECONCILIATION=IMPLEMENTED
~~~

The standard public Filesystem surface no longer installs a `captureTree`
selector. The former public capture operation, backend seam, Future/cancellation
capture flow and captured-subtree minting paths were removed. `entries` remains
the public tree-observation operation.

The internal package-custody architecture remains distinct from the removed
guest surface. `ProtosCapturedFilesystemCustody.captureSelectedRoot(...)`
continues to capture an already-selected root through
`ProtosNioReadOnlyTreeFilesystemBackend.captureRootForHostCustody()` and the
secure recursive no-follow traversal into `ProtosNioCapturedTreeFilesystemBackend`.
The immutable captured backing remains host/runtime custody.

Captured Filesystem views continue to be ordinary read-only Filesystem
capabilities for retained operations such as `open` and `entries`. They do not
expose `captureTree` and cannot mint another captured Filesystem.

Package ContentIdentity and execution-plan coverage were moved to Java-hosted
fixtures over real host-captured custody. Verification still observes the same
immutable custody later used for package reads, without reopening the original
mutable source/store Path. The fixed ContentIdentity vectors and digests are
unchanged.

## Normative reconciliation

Specification revision `0.1.443` removes the public
`Filesystem.captureTree(path)` contract and capture-only guest
Future/cancellation/result semantics while retaining Filesystem authority and
confinement, `Filesystem.entries`, and host-provisioned read-only captured
Filesystem behavior.

Historical D046/I024 design material is retained with explicit D171 supersession
context rather than rewritten as if the old public selector remained current.

## Validation and publication evidence

The maintainer reported after the exact candidate was finalized and pushed:

~~~text
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
PRODUCT_PUBLICATION=PUSHED
~~~

GitHub Actions exact-sha publication validation was inspected at this checkpoint:

~~~text
CI_RUN_ID=37322294900
CI_RUN_NUMBER=2160
CI_HEAD=812b3f0b29dcba59562e3d03930c163db8529410
CI_WORKFLOW=CI
CI_EVENT=push
CI_STATUS=in_progress
CI_CONCLUSION=NONE_YET
~~~

Therefore the implementation is published and locally green, but the formal
I064 closure gate `PUBLICATION_VALIDATION=PASS` is not yet claimed.

## Coordination state

The native GitHub hierarchy is established:

~~~text
I064_ISSUE=667
NATIVE_PARENT=642
NATIVE_PARENT_STATUS=PASS
~~~

At this checkpoint:

~~~text
I064_A_IMPLEMENTATION=COMPLETE
NEXT_IMPLEMENTATION_SLICE=NONE
I064_CLOSURE_CANDIDATE=YES
I064_CLOSURE_AUTHORIZED=NO
PENDING_GATE=EXACT_SHA_CI_COMPLETION
NEXT_ACTIVITY=FINAL_CLOSURE_REVIEW
~~~

The final closure review must re-read current Protos HEAD, confirm that I064-A is
still present without a relevant post-I064 regression, confirm exact-sha CI
success, publish final closure evidence if warranted, post the required final
Issue closure comment, and close #667 only when every closure gate is PASS.

AI assistance: this durable checkpoint was drafted with ChatGPT from the exact
published Protos revision, the I064/D171 repository authority, GitHub Issue and
Actions state, and maintainer-reported local validation.
