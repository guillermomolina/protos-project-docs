# DIST010-A — Canonical exact-revision artifact-set build envelope

Status: **PUBLISHED IMPLEMENTATION CHECKPOINT**

Date: 2026-10-02

## Identity

~~~text
WORK=DIST010-A
OWNING_ISSUE=guillermomolina/protos#771
PROTOS_REVISION=b91d6066eb41b420db4a2884ac4719b2468767fa
PARENT_REVISION=5b5dedd7a36b4aba0684a072a4f1a86a52ec9923
COMMIT_SUBJECT=DIST010-A: canonical exact-revision artifact-set build envelope
RATIFIED_ARCHITECTURE=D181/#770 Candidate D
NEXT_DECISION=D182/#773
~~~

This record retains the published DIST010-A producer-side construction boundary.
It does not claim that DIST010 is complete and does not select the durable remote
publication backend for D064.

## Published artifact-set entry point

The root repository now exposes:

~~~text
make artifacts
make artifacts-verify
~~~

`make artifacts` is explicitly separate from `make dist`, release candidate
materialization, tag creation, GitHub Release creation, upload, or any other
public-release publication operation.

The canonical local set is assembled under:

~~~text
target/artifact-set/
~~~

with the documented shape:

~~~text
protos-<version>-posix-jvm.zip
protos-<version>-native-<os>-<arch>.zip
protos-<version>-stdlib-documentation.json
protos-<version>-stdlib-documentation-coverage.txt
SHA256SUMS
ARTIFACT-SET.json
~~~

## Construction contract

The published orchestrator is:

~~~text
dist/build_artifact_set.py
~~~

It composes existing Protos-owned builders rather than duplicating their core
distribution logic:

~~~text
mvn clean
make -C build/native build
dist/build_native.py
dist/build_portable.py
Protos-owned D064 extractor
~~~

D064 generation executes twice from the same checkout and requires byte-identical
JSON and coverage output before the final artifact set is admitted.

The construction path is explicitly development-artifact construction:

~~~text
ARTIFACT_SET_PUBLIC_RELEASE=false
~~~

and its focused test corpus verifies that ordinary construction does not invoke
release preparation/publication markers.

## Exact-revision and provenance contract

The final `ARTIFACT-SET.json` records:

- repository identity `guillermomolina/protos`;
- one exact 40-hex source revision;
- exact project version;
- logical artifact kind;
- artifact path;
- SHA-256;
- JVM/native runtime/platform identity where applicable; and
- D064 format plus exact internal provenance.

Archive members are accepted only when their own `SOURCE.txt` agrees with the
same exact revision, implementation version, clean-source state, and
`public_release=false`.

D064 is accepted only when its internal provenance is:

~~~text
kind=repositoryRevision
repository=guillermomolina/protos
revision=<artifact-set exact revision>
~~~

## Fail-closed construction

The published implementation removes both the prior complete output and any
partial staging output before beginning.

New output is assembled under:

~~~text
target/artifact-set.partial
~~~

and is renamed to the canonical output only after the envelope verifies.

The focused tests cover at least:

- exact revision and per-artifact SHA-256 binding;
- deterministic manifest serialization;
- D064 provenance mismatch rejection;
- mixed-revision Native rejection;
- missing required artifact rejection;
- partial set without manifest rejection;
- removed member rejection;
- expected-revision mismatch rejection;
- stale member rejection;
- changed-byte digest rejection;
- tampered checksum rejection;
- unrecorded extra artifact rejection;
- ambiguous second Native archive rejection; and
- no release-publication machinery in the ordinary construction steps.

## Maintained documentation

The root `Makefile` help and `dist/README.md` now distinguish:

~~~text
make artifacts        canonical exact-revision Native + JVM + D064 set
make artifacts-verify re-verify an existing local set
make dist             portable JVM distribution only
release tooling       explicit separate release operation
~~~

The documentation explicitly states that durable exact-revision D064
publication remains later work.

## Validation evidence visible from GitHub

At the time this evidence was published, GitHub Actions run
`36958170179` for exact revision
`b91d6066eb41b420db4a2884ac4719b2468767fa` had completed with overall
conclusion **failure**.

The failure is not recorded here as a DIST010-A functional-test failure.

The job log shows:

~~~text
JUnit/Surefire serial result:
Tests run: 7, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS

JAVA_SLOW_TEST_GUARD=FAIL
THRESHOLD_SECONDS=10
REPORTED_TEST_CLASSES=472
ALLOWLISTED_SLOW_TESTS=6
UNALLOWLISTED_SLOW_TESTS=10
OVER_BUDGET_SLOW_TESTS=5
~~~

The guard then made `make test-java` exit non-zero.

Examples reported over budget included existing filesystem/package/test-tool
test classes unrelated to the four DIST010-A changed paths.

Therefore the durable statement is:

~~~text
GITHUB_CI_OVERALL=FAIL
CI_FAILURE_STAGE=JAVA_SLOW_TEST_GUARD
JUNIT_FAILURE_OBSERVED=NO
DIST010_A_FULL_CI_PASS=NOT_CLAIMED
~~~

No stronger local validation result was available through the repository/Issue
evidence inspected for this record. This file deliberately does not infer a PASS
from the implementation shape or from publication alone.

## Published changed paths

~~~text
Makefile
dist/README.md
dist/build_artifact_set.py
dist/test_build_artifact_set.py
~~~

No D064 schema, Protos language semantics, Standard Library semantics, website
source, Git tag, or GitHub Release was changed by this publication.

## Remaining architecture boundary

DIST010-A completes the **local construction** half of D181 Candidate D.

It does not answer the remaining durable cross-repository question:

~~~text
exact Protos revision X
    -> how is D064(X) durably published?
    -> what is immutable identity?
    -> how is X discovered?
    -> what is retention/deletion policy?
    -> how does the public website acquire it anonymously?
~~~

D181 intentionally did not select a concrete publication transport/backend.

That remaining architecture question has been allocated as:

~~~text
D182=guillermomolina/protos#773
D182_TITLE=Exact-revision D064 publication, retention, and discovery boundary
NEXT_SLICE=D182-A
TYPE=INVESTIGATION
SHELL_EXECUTION=NO
IMPLEMENTATION_AUTHORIZED=NO
~~~

DIST010-B remains blocked on explicit D182 selection/ratification.

## Conclusions

~~~text
DIST010_A_PUBLICATION=PASS
CANONICAL_ENTRY_POINT=make artifacts
VERIFY_ENTRY_POINT=make artifacts-verify
LOCAL_ARTIFACT_SET_ENVELOPE=PUBLISHED
EXACT_REVISION_BINDING=PUBLISHED
D064_DOUBLE_GENERATION_DETERMINISM_GATE=PUBLISHED
FAIL_CLOSED_STAGING=PUBLISHED
BUILD_RELEASE_SEPARATION=PUBLISHED

PUBLIC_RELEASE_CREATED=NO
DURABLE_REMOTE_D064_PUBLICATION=NOT_IMPLEMENTED
D064_SCHEMA_CHANGE=NO
PROTOS_SEMANTIC_CHANGE=NO

GITHUB_CI_OVERALL=FAIL
GITHUB_CI_FAILURE=JAVA_SLOW_TEST_GUARD
DIST010_A_FULL_CI_PASS=NOT_CLAIMED

NEXT_DECISION=D182/#773
~~~
