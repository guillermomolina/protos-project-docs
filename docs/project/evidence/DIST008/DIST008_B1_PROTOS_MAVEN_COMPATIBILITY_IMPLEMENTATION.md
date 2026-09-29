# DIST008-B1 — Protos Maven compatibility/provisioning implementation

Status: PASS
Date: 2026-09-29
Owning Issue: `guillermomolina/protos#739`
Baseline Protos revision:
`a6aa7f1177f99c77e5148b8363a77d7250184793`
Published Protos revision:
`6411d39bf33014c958ba4dad60a6e3fe44760ebf`

## Implementation boundary

DIST008-B1 implemented the Maven compatibility contract selected by DIST008-A
in `guillermomolina/protos`.

The published delta is exactly one commit and exactly these five paths:

```text
.devcontainer/Dockerfile
build/native/Dockerfile
toolchain.json
tools/test_verify_toolchain.py
tools/verify_toolchain.py
```

No `src/`, `protos/lib/`, POM dependency/plugin, release-version, or
`CHANGELOG.md` change was part of B1.

## Published contract

`toolchain.json` moved from `protos-toolchain-v1` with an exact
`maven.version` coordinate to `protos-toolchain-v2` with:

```json
"maven": {
  "minimum_version": "3.9.9",
  "supported_major": 3
}
```

The resulting supported Maven line is:

```text
stable Apache Maven 3.x >= 3.9.9
exact Maven Core patch identity required: NO
Maven 4.x: unsupported by the current contract
prerelease/RC Maven coordinates: unsupported by the current contract
```

The GraalVM, JDK and Graal/Truffle coordinates remain unchanged.

## Container provisioning

Both current GraalVM OL10 Maven provisioning surfaces now use one dedicated
Maven transaction containing:

```text
--enablerepo=ol10_codeready_builder
maven
maven-unbound
```

The development container keeps ordinary OS packages in its normal repository
transaction; `ol10_codeready_builder` is enabled only for the Maven
transaction.

The Native Image builder uses the same unbound Maven binding.

## Verifier behavior

`tools/verify_toolchain.py` now separates:

- Maven compatibility policy;
- static Maven provisioning;
- CodeReady Builder repository scope;
- canonical `JAVA_HOME` precedence;
- optional runtime Maven/JDK/RPM evidence.

The static verifier fails closed on the legacy exact-Maven schema, malformed
Maven minima, plain Maven provisioning without `maven-unbound`, stale Native
builder bindings, and missing `JAVA_HOME` precedence.

The runtime mode evaluates the actual Maven version, Maven Java version/vendor,
Maven Java runtime versus `JAVA_HOME`, and redundant OpenJDK RPM presence.

`tools/test_verify_toolchain.py` covers:

```text
3.9.9                       accepted
newer stable Maven 3.x      accepted
below 3.9.9                 rejected
Maven 4.x                   rejected
prerelease/RC               rejected
malformed version           rejected
legacy exact schema         rejected
devcontainer plain Maven    rejected
Native builder plain Maven  rejected
wrong Java vendor/runtime   rejected
redundant OpenJDK RPM       rejected
```

## Validation evidence

### Focal verifier

Owner-executed validation:

```text
python3 -m py_compile tools/verify_toolchain.py tools/test_verify_toolchain.py
python3 tools/test_verify_toolchain.py
```

Result:

```text
TOOLCHAIN_VERIFIER_TESTS: PASS
```

### Static repository bindings

Both:

```text
python3 tools/verify_toolchain.py --mode check --scope development
python3 tools/verify_toolchain.py --mode check --scope all
```

completed with:

```text
TOOLCHAIN_CONTRACT: PASS
TOOLCHAIN_DRIFT_COUNT: 0
TOOLCHAIN_BINDINGS: PASS
```

The all-surface result included PASS for the Native Image container,
`maven+maven-unbound` provisioning, CodeReady Builder repository scope and
`JAVA_HOME` precedence.

### Native Image builder package/runtime proof

The rebuilt B1 Native Image builder installed:

```text
maven-3.9.9-3.el10_1.noarch
maven-unbound-3.9.9-3.el10_1.noarch
```

Its runtime evidence was:

```text
JAVA_HOME=/usr/lib64/graalvm/graalvm-community-java25i4
Apache Maven 3.9.9 (Red Hat 3.9.9-3)
Java version: 25.0.4.1.1
vendor: GraalVM Community
runtime: /usr/lib64/graalvm/graalvm-community-java25i4
REDUNDANT_OPENJDK_NATIVE=NO
```

Therefore Maven uses the Native builder's canonical GraalVM JDK and the
selected unbound binding adds no redundant OpenJDK RPM.

### Development container package/runtime proof

The development container was rebuilt from the B1 Dockerfile.

Installed packages:

```text
maven-3.9.9-3.el10_1.noarch
maven-unbound-3.9.9-3.el10_1.noarch
```

Runtime evidence:

```text
JAVA_HOME=/opt/graalvm-community-java25i4
Apache Maven 3.9.9 (Red Hat 3.9.9-3)
Java version: 25.0.4.1.1
vendor: GraalVM Community
runtime: /opt/graalvm-community-java25i4
REDUNDANT_OPENJDK_DEVCONTAINER=NO
```

The canonical Java path differs between the two GraalVM base images, but in
both cases Maven resolves exactly to that image's `JAVA_HOME`.

### Integrated repository validation

After rebuilding the development container, the owner ran:

```text
make test
```

Result:

```text
PASS
```

### Portable POSIX/JVM distribution

The dirty pre-publication candidate was built and validated with the repository's
explicit pre-commit source mode.

Build result:

```text
DIST_RUNTIME_PROJECTION_CHECK: PASS
DIST_RUNTIME_PROJECTION_JARS: 12
DIST_ARCHIVE_CRC_CHECK: PASS
DIST_LAYOUT_CHECK: PASS
DIST_SOURCE_IDENTITY_CHECK: PASS
DIST_RUNTIME_METADATA_CHECK: PASS
DIST_LAUNCHER_MODE_CHECK: PASS
DIST_BUILD: PASS
```

Cross-slice validation included:

```text
DIST001_B2_VERIFY: PASS
DIST001_B3_SMOKE: PASS
DIST001_B4A_SMOKE: PASS
TEST001_H_PORTABLE_REPOSITORY_SUITE_CHECK: PASS passed=1263 failed=0
DIST001_B4B_SMOKE: PASS
DIST001_B5_CROSS_SLICE: PASS
PORTABLE_DISTRIBUTION_STATUS=0
```

Validated portable archive SHA-256:

```text
a2bd4f9b60842a849e00ceb2f3e6d0e3ae0e3195ef0139e59a0b9efc95ee0ba2
```

### Native distribution impact

B1 changes the Native builder's Maven provisioning, not the produced Protos
Native executable semantics or distribution layout.

The B1 Native builder itself was rebuilt successfully and its Maven/JDK/RPM
runtime binding was proven as recorded above. Static all-surface toolchain
verification also covered the Native Image Dockerfile.

A full DIST005 Native artifact admission was not repeated as part of B1:
`dist/build_native.py` requires an already-built `target/native/protos`,
and B1 does not modify the Native executable, its Native Image arguments,
launcher, distribution builder, or admission logic. The failed precondition
probe therefore did not represent a B1 product failure.

## Publication audit

Immediately before commit:

```text
BASELINE_HEAD=a6aa7f1177f99c77e5148b8363a77d7250184793
CHANGED_PATHS=5
UNEXPECTED_PATHS=NONE
git diff --check=PASS
```

Published commit:

```text
6411d39bf33014c958ba4dad60a6e3fe44760ebf
DIST008-B1: make Maven provisioning compatibility-based
5 files changed, 426 insertions(+), 51 deletions(-)
```

The push advanced `main` from `a6aa7f11` to `6411d39b`.

## Result packet

```text
DIST008_B1_RESULT=PASS
PROTOS_REVISION=6411d39bf33014c958ba4dad60a6e3fe44760ebf
TOOLCHAIN_SCHEMA=protos-toolchain-v2

MAVEN_MINIMUM_VERSION=3.9.9
MAVEN_SUPPORTED_MAJOR=3
EXACT_MAVEN_REQUIRED=NO

DEVCONTAINER_MAVEN_UNBOUND=PASS
NATIVE_BUILDER_MAVEN_UNBOUND=PASS
REDUNDANT_OPENJDK_DEVCONTAINER=NO
REDUNDANT_OPENJDK_NATIVE=NO
CANONICAL_GRAALVM_JAVA_AUTHORITY=PASS

TOOLCHAIN_VERIFIER_TESTS=PASS
STATIC_DEVELOPMENT_BINDINGS=PASS
STATIC_ALL_BINDINGS=PASS
FULL_REPOSITORY_TESTS=PASS
PORTABLE_DISTRIBUTION=PASS

GRAALVM_JDK_TRUFFLE_COORDINATES_UNCHANGED=YES
UNEXPECTED_PATHS=NONE
BLOCKER=NONE

NEXT_SLICE=DIST008-B2
NEXT_REPOSITORY=guillermomolina/protos-benchmarks
```
