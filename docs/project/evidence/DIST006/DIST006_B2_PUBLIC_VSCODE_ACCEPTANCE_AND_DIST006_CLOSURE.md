# DIST006-B2 — public VS Code acceptance and DIST006 closure

Status: **PASS / DIST006 COMPLETE**

This durable, non-normative record retains the final installed/public VS Code
acceptance for DIST006-B2 / `guillermomolina/protos#734` and the resulting
closure evidence for parent DIST006 / `guillermomolina/protos#733`.

## Ownership and final identities

```text
DATE=2026-10-02
PARENT_WORK_ITEM=DIST006
PARENT_ISSUE=guillermomolina/protos#733
CHILD_WORK_ITEM=DIST006-B
CHILD_ISSUE=guillermomolina/protos#734

PROTOS_RELEASE_TAG=v0.3.139
PROTOS_REVISION=3895206897ddac795dfebd49709ca97f8d0908b1
PROTOS_NATIVE_ASSET=protos-0.3.139-native-linux-x86_64.zip
PROTOS_NATIVE_SHA256=8e87dfc410c6a49f194786f205ba434f0b5b60865873e99e18ded8aba66c46da

EXTENSION_REPOSITORY=guillermomolina/protos-vscode-extension
EXTENSION_REVISION=e7b4ec8d274b5366ad02e93a27ded8595d872fea
VSCODE_VERSION=1.140.0
```

The final public-runtime acceptance uses the published BUG012-fixed Native
runtime. No development-path runtime override and no unreleased Protos candidate
is part of the final gate.

## DIST006-B2 extension publication result

The extension repository published the final B2 acceptance/test-authority
revision:

```text
EXTENSION_REVISION=e7b4ec8d274b5366ad02e93a27ded8595d872fea
TEST_AUTHORITY=make test
REPOSITORY_BASELINE=npm test
INSTALLED_VSIX_ACCEPTANCE=npm run test:acceptance
CI_EXECUTION_ENVIRONMENT=devcontainer
```

The B2 revision makes local and CI validation consume the same repository
authority. The historical `npm test` baseline remains intact; installed-VSIX
acceptance is layered after it.

The final acceptance path:

- packages and canonicalizes the extension VSIX;
- installs the exact published Protos v0.3.139 Native runtime;
- installs the actual canonical VSIX into pinned VS Code 1.140.0;
- performs clean-install, real Run and real Debug acceptance;
- never uses `--extensionDevelopmentPath`.

Original Run/Debug harnesses and product runtime/debug-adapter sources were not
changed by the final test-authority reconciliation.

## Local final admission

The human executor reported the complete repository authority green on the final
candidate before publication:

```text
make test=PASS
MAKE_TEST_RC=0
git diff --check=PASS
DIFF_CHECK_RC=0
```

The final review also retained:

```text
ORIGINAL_HARNESSES_UNCHANGED=YES
PRODUCT_FILES_UNCHANGED=YES
```

## Published CI admission

GitHub Actions run:

```text
WORKFLOW_RUN_ID=37013996144
WORKFLOW_JOB_ID=110860284166
WORKFLOW_JOB=Repository test authority
HEAD_SHA=e7b4ec8d274b5366ad02e93a27ded8595d872fea
RESULT=SUCCESS
```

The workflow built the committed devcontainer and executed exactly
`make test`.

The canonical packaged extension identity printed by that run is:

```text
VSIX_ARTIFACT_SHA256=ab2bd537b89f2e254c2abbafb26e27d67b3dbc318a3c84241e7d4751b59cfac5
CANONICAL_SHA256_A=ab2bd537b89f2e254c2abbafb26e27d67b3dbc318a3c84241e7d4751b59cfac5
CANONICAL_SHA256_B=ab2bd537b89f2e254c2abbafb26e27d67b3dbc318a3c84241e7d4751b59cfac5
```

The uploaded workflow artifact is:

```text
ARTIFACT_ID=11229805192
ARTIFACT_NAME=protos-vscode-vsix-e7b4ec8d274b5366ad02e93a27ded8595d872fea
ARTIFACT_ARCHIVE_SHA256=e6c2666ab29f9d44d78e83073925cb852a5e750e346f34c1c6b5bbbf5519b520
```

## Published runtime binding

The same CI run independently reported the exact locked public runtime:

```text
PROTOS_RELEASE_TAG=v0.3.139
PROTOS_RELEASE_ASSET_SHA256=8e87dfc410c6a49f194786f205ba434f0b5b60865873e99e18ded8aba66c46da
LOCKED_PROTOS_RELEASE_TAG=v0.3.139
LOCKED_PROTOS_ARCHIVE_SHA256=8e87dfc410c6a49f194786f205ba434f0b5b60865873e99e18ded8aba66c46da
```

DIST009 had already independently verified that `v0.3.139` points to exact
source revision
`3895206897ddac795dfebd49709ca97f8d0908b1`.

## Installed/public VS Code acceptance

The final CI acceptance reports:

```text
LM009_I3A_CLEAN_INSTALL=PASS
LM009_I3B_REAL_RUN=PASS
REAL_PROTOS_RUNTIME_INVOKED=YES
LM009_I3C_REAL_DEBUG=PASS

SOURCE_BREAKPOINT=PASS
STOP_LOCATION=PASS
STACK_FRAMES=PASS
ACTIVATION_LOCAL_SCOPE=PASS
REPRESENTATIVE_SCALAR_VALUE=PASS
STEP_NEXT=PASS
CONTINUE=PASS
CLEAN_TERMINATION=PASS

EXTENSION_DEVELOPMENT_PATH_USED=NO
PROTOS_VSCODE_ACCEPTANCE_TEST=PASS
```

This closes the two remaining DIST006-B criteria: real Debug against the
canonical public runtime contract and durable exact-revision validation
evidence.

## Final Protos ancestry closure

The final public Protos revision consumed by B2 is a descendant of every
relevant DIST006 product checkpoint and both Native/DAP follow-up repairs:

```text
DIST006_A_REVISION=e65509d38c8df65246e0b5d0cc241986541b49b9
DIST006_B1_REVISION=43f883b9f244c635311c9e2a049b1b3cf65bae93
DIST006_C1_REVISION=d18822e968a1ee6986d832731054b9b90b332b1d
BUG011_REVISION=754de7a2a2d73dd4b39109bb842522ed9cc8153a
BUG012_REVISION=417e44a8c60eaba1db7859d78bbb1e61c5dace64
FINAL_PUBLIC_PROTOS_REVISION=3895206897ddac795dfebd49709ca97f8d0908b1

DIST006_A_IN_ANCESTRY=YES
DIST006_B1_IN_ANCESTRY=YES
DIST006_C1_IN_ANCESTRY=YES
BUG011_IN_ANCESTRY=YES
BUG012_IN_ANCESTRY=YES
```

GitHub compare state for each checkpoint against the final public revision was
`ahead` with zero commits behind, establishing descendant ancestry.

## Earlier durable DIST006 evidence

The final closure composes the previously retained records rather than rewriting
them:

```text
DIST006_A_PROJECT_RECORD_REVISION=b5ecd83ba88f9edd97aae3344b32bffa889e70bb
DIST006_A_PROJECT_RECORD_PATH=docs/project/evidence/DIST006/DIST006_A_GRAALVM_25_4_TOOLCHAIN_MIGRATION.md

DIST006_B1_PROJECT_RECORD_REVISION=5d1d0d9a74109a2a4d971ead03ca7999f0f89070
DIST006_B1_PROJECT_RECORD_PATH=docs/project/evidence/DIST006/DIST006_B1_GRAALVM_25_4_CORE_COMPATIBILITY.md

DIST006_C1_PROJECT_RECORD_REVISION=b457b9893281479deab33e724e69b3b632330189
DIST006_C1_PROJECT_RECORD_PATH=docs/project/evidence/DIST006/DIST006_C1_PORTABLE_NATIVE_IMAGE_25_4_CLOSURE.md

DIST006_D_PROJECT_RECORD_REVISION=fe409892ccdb5e2c0aa0891116713c210296051a
DIST006_D_PROJECT_RECORD_PATH=docs/project/evidence/DIST006/DIST006_D_GRAALVM_25_4_POST_ADOPTION_BASELINE.md
```

The build-stage Java-path follow-up observed by DIST006-D was subsequently
reconciled by DIST008-B2 without rewriting the retained DIST006-D reference:

```text
DIST006_D2_RECONCILIATION=DIST008-B2
BENCHMARK_REVISION=e8a1f1735e0c2751de99459aeeb689a8d46e5f0d
DIST008_PROJECT_RECORD_REVISION=bee1066ea470fa691ad250eecd9e2bf5720b8062
DIST006_D2_STATUS=RESOLVED
```

## DIST006 final classification

All root DIST006 acceptance criteria are satisfied:

```text
CANONICAL_GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1
CANONICAL_OL10_IMAGE_BINDING=PASS
PLAT033_SINGLE_RUNTIME_AUTHORITY=PASS
HOTSPOT_TRUFFLE_RUNTIME=PASS
BYTECODE_DSL_25_4=PASS
SOURCE_INSTRUMENTATION_DAP=PASS
PORTABLE_DISTRIBUTION_25_4=PASS
NATIVE_IMAGE_CONTAINER_25_4=PASS
FULL_REQUIRED_VALIDATION=PASS
HISTORICAL_25_3_EVIDENCE_REWRITTEN=NO
PERFORMANCE_OPTIMIZATION_MIXED_IN=NO
POST_ADOPTION_25_4_BASELINE=RETAINED

DIST006_B2_PUBLIC_RUN=PASS
DIST006_B2_PUBLIC_DEBUG=PASS
DIST006_B_STATUS=COMPLETE
DIST006_STATUS=COMPLETE
BLOCKER=NONE
```

No Protos language or Standard Library semantics changed as part of B2 closure.
No new platform or language decision was required.

## Closure transaction

```text
FORMAL_IDENTIFIER_UNIQUE=PASS
FAMILY=PASS
NATIVE_PARENT=PASS
ASSIGNEE_INVARIANT=PASS
EFFECTIVE_PRIORITY=RESOLVED
DECISION_APPROVAL_PROVENANCE=NOT_APPLICABLE
DECISION_INVARIANT_CONSISTENCY=NOT_APPLICABLE
REQUIRED_DURABLE_PUBLICATION=PASS
CROSS_REFERENCES=PASS
```

After this record is published and re-read at its exact project-record revision,
DIST006-B / #734 and parent DIST006 / #733 are ready to close as completed.
