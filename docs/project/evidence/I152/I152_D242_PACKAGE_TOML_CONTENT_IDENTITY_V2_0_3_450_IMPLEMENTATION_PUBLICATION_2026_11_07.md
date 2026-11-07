# I152 — D242 package.toml and ContentIdentity v2 implementation publication (0.3.450)

**Date:** 2026-11-07. **Role:** revision-bound implementation/publication and maintainer-reported validation evidence (non-normative).
**Implementation owner:** [I152 #1067](https://github.com/guillermomolina/protos/issues/1067).
**Approved design:** [D242 #1060](https://github.com/guillermomolina/protos/issues/1060), [ratified D242 decision](../../decisions/language/D242_MODULE_SOURCE_PUBLIC_EXPORTS_AND_PACKAGE_MANIFEST_RATIFIED.md). **Prerequisite:** [I151 #1066](https://github.com/guillermomolina/protos/issues/1066), [published I151 implementation record](../I151/I151_D242_MODULE_SOURCE_0_3_449_IMPLEMENTATION_PUBLICATION_2026_11_06.md).

## Exact product publication

- **Published Protos revision:** [\`8a6366e486d0b58b58cce17ce627734d9cadcc2d\`](https://github.com/guillermomolina/protos/commit/8a6366e486d0b58b58cce17ce627734d9cadcc2d) on \`main\`.
- **Previous product revision:** [\`9088e87d339558af168676a4dd3f31774e016b65\`](https://github.com/guillermomolina/protos/commit/9088e87d339558af168676a4dd3f31774e016b65) (I151).
- **Commit message:** \`I152: replace protos.toml with package.toml and rebaseline identity (0.3.450)\`.
- **Commit delta independently checked in GitHub:** 160 files changed, 809 additions, 360 deletions; one commit ahead of I151. The maintainer's local push output also confirms publication of the same revision.
- **Implementation version:** \`0.3.450-SNAPSHOT\`, independently read in the exact-revision [\`pom.xml\`](https://github.com/guillermomolina/protos/blob/8a6366e486d0b58b58cce17ce627734d9cadcc2d/pom.xml) and [\`CHANGELOG.md\`](https://github.com/guillermomolina/protos/blob/8a6366e486d0b58b58cce17ce627734d9cadcc2d/CHANGELOG.md). This is an implementation revision, not a GitHub Release or a published package registry artifact.

## Delivered contract and implementation surfaces

1. **Incompatible manifest replacement:** exactly \`package.toml\` is the regular, exact-cased root manifest of a package and each workspace member. Legacy-only \`protos.toml\`, coexistence, forbidden entry kinds and symlink aliases are rejected without fallback. The TOML schema remains \`manifest-version = 1\`; package \`[exports]\` names logical module directories containing mandatory \`module.protos\`, following the separate D242/I151 MODULE/SOURCE contract. See [Package Manifest Format](https://github.com/guillermomolina/protos/blob/8a6366e486d0b58b58cce17ce627734d9cadcc2d/docs/design/PACKAGE_MANIFEST_FORMAT.md), [\`ManifestCommand.protos\`](https://github.com/guillermomolina/protos/blob/8a6366e486d0b58b58cce17ce627734d9cadcc2d/protos/tools/package/ManifestCommand.protos), [\`ResolutionRoot.protos\`](https://github.com/guillermomolina/protos/blob/8a6366e486d0b58b58cce17ce627734d9cadcc2d/protos/tools/package/ResolutionRoot.protos), and [\`ProtosCli.java\`](https://github.com/guillermomolina/protos/blob/8a6366e486d0b58b58cce17ce627734d9cadcc2d/src/main/java/com/guillermomolina/protos/cli/ProtosCli.java).
2. **New ContentIdentity domain:** the canonical package-tree domain separator and method are \`protos-package-tree-v2\` (NUL-terminated magic). The canonical stream's ordered framed path/contents records use \`package.toml\`, rather than silently reinterpreting frozen v1 identities. The exact-revision [ContentIdentity implementation](https://github.com/guillermomolina/protos/blob/8a6366e486d0b58b58cce17ce627734d9cadcc2d/protos/tools/package/ContentIdentity.protos), [v2 fixed vectors](https://github.com/guillermomolina/protos/blob/8a6366e486d0b58b58cce17ce627734d9cadcc2d/docs/design/PACKAGE_CONTENT_IDENTITY_VECTORS.md), and [independent Java conformance test](https://github.com/guillermomolina/protos/blob/8a6366e486d0b58b58cce17ce627734d9cadcc2d/src/test/java/com/guillermomolina/protos/execution/packages/ProtosPackageContentIdentityV2ConformanceTest.java) identify ten rebaselined positive SHA-256 or comparison vectors and explicit negative legacy-domain cases. Historical v1 evidence is not a supported current reader.
3. **Intentionally unchanged identifiers:** the metadata digest still uses \`protos-project-metadata-v1\`, but records \`package.toml\` and member \`<location>/package.toml\` filenames; the resolution-input method stays \`protos-resolution-input-v1\`; the lock and project metadata filenames remain \`protos.lock\` and \`protos.project\`. See [\`ProjectMetadata.protos\`](https://github.com/guillermomolina/protos/blob/8a6366e486d0b58b58cce17ce627734d9cadcc2d/protos/tools/package/ProjectMetadata.protos) and [Lockfile Format](https://github.com/guillermomolina/protos/blob/8a6366e486d0b58b58cce17ce627734d9cadcc2d/docs/design/PACKAGE_LOCKFILE_FORMAT.md).
4. **Authority, fixtures and surface integration:** the CLI, workspace project binding, Package Tool planning, external package checks, filesystem custody, Test Tool fixture entrances and native/LSP fixture script were updated. Positive manifest fixtures were physically renamed; lock content methods were rebased without changing \`protos.lock\`. The implementation commit also updates the root README and 15 package-related design and guide Markdown documents. The precise changed-path set is the [published commit diff](https://github.com/guillermomolina/protos/commit/8a6366e486d0b58b58cce17ce627734d9cadcc2d).

## Validation and regression evidence

**Maintainer-executed / maintainer-reported in the working checkout, not independently rerun here:**

- Two groups of bounded Maven/JUnit Package Tool, workspace, exact-package, custody and ContentIdentity tests completed successfully (\`mvn -q -Dtest=… test\`, with selected class lists visible in the working-session log).
- Selected Protos Package Tool test corpora: **229 passed, 0 failed** in total after repair. The first five nonempty selected subcorpora totalled 95 cases: 93 initially passed, two \`project-projection\` expected metadata hashes failed, and the targeted rebaseline subsequently passed 4/4; three further TOML/version/resolution-input subcorpora passed 134/134.
- The two \`protos-project-metadata-v1\` golden hashes were independently recalculated using exact fixture bytes and their renamed canonical record paths: \`root-only = f59e12289eceb97dd47d73088a03a3e3a673a5afb09e1747e2e9b9ee06d0b88d\`; \`metadata-members = d7e4771132fb598af01ca23b2894a4898ca795e54bd9d71ab47788eba630b024\`.
- A pre-finalization source audit reported 30 \`protos.lock\` test fixtures, 17 v2 references and **zero v1 references**, zero legacy manifest files under the inspected positive fixture directories, and zero superseded golden digests. Residual \`protos.toml\` references in the audited production files were only explicit rejection guards or diagnostic text.
- The maintainer reports the integrated \`make test\` **PASS**, \`make check\` **PASS**, and \`git diff --check\` **clean**. These full gates were run **before** the version and changelog bump. No post-bump behavior tests were requested or reported; the maintainer subsequently committed and pushed the complete candidate.

The \`content-identity\` Test Tool directory contains no selectable cases in its own \`manifest.tsv\`; invoking \`--directory\` on that empty plan is rejected by the selector. ContentIdentity's real contract is instead exercised by the Java v2 conformance/integration tests. An initial overly broad selector error therefore is not evidence of failed product tests.

**Independently checked using GitHub read-only source and commit data:** the exact product revision, commit topology, changed-file count and additions/deletions, published version/changelog, v2 method, retained metadata method, canonical vector document and Java conformance fixture. Local Java/Protos/Make execution and local diff checks are maintainer reports, not independently observed CI results.

## Disposition and boundaries

I152 completes the D242 **package manifest/tooling/identity** implementation, separately from I151's already-published MODULE/SOURCE runtime work. This is the previously approved breaking decision, not a new design approval. No alias reader, published-package SHA migration, package registry rollout, Native Image equivalence, independent CI result, or performance claim is inferred.

\`\`\`text
FORMAL_IDENTIFIER=I152
FAMILY=I
STATUS=COMPLETED_PRODUCT_PUBLISHED
PROTOS_REVISION=8a6366e486d0b58b58cce17ce627734d9cadcc2d
PREVIOUS_REVISION=9088e87d339558af168676a4dd3f31774e016b65
IMPLEMENTATION_VERSION=0.3.450-SNAPSHOT
MANIFEST=package.toml
CONTENT_IDENTITY=protos-package-tree-v2
PROJECT_METADATA=protos-project-metadata-v1
RESOLUTION_INPUT=protos-resolution-input-v1
LOCK_FILENAME=protos.lock
OWNER_REPORTED_MAKE_TEST=PASS
OWNER_REPORTED_MAKE_CHECK=PASS
OWNER_REPORTED_GIT_DIFF_CHECK=CLEAN
POST_VERSION_BUMP_TESTS=NOT_REPORTED
PUBLISHED_IMPLEMENTATION=YES
\`\`\`
