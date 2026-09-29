# DIST008 — Maven compatibility floor and lean provisioning

Status: READY
Live coordination: GitHub Issue #739
Investigation slice: DIST008-A CLOSED
Protos implementation slice: DIST008-B1 CLOSED
Next implementation slice: DIST008-B2 READY
Protos investigation revision: `754de7a2a2d73dd4b39109bb842522ed9cc8153a`
Protos implementation revision: `6411d39bf33014c958ba4dad60a6e3fe44760ebf`
Benchmark inventory revision: `a2a8eafe74a45cce987a0918d1023be061ed17f3`

## Purpose

Reconcile Maven as ordinary compatible build tooling rather than an exact Protos
runtime coordinate, eliminate the redundant OpenJDK installed by the default
Oracle Linux Maven binding in GraalVM-based containers, and propagate the
resulting contract to current Protos companion consumers without rewriting
historical benchmark environments.

DIST008 follows DIST002 and DIST004. DIST002 centralized Maven 3.9.9 as an exact
environment-alignment coordinate. DIST004 moved the development container to
Oracle Linux 10 and replaced the historical Apache tarball bootstrap with the OS
`maven` package while deliberately retaining that exact coordinate. Neither
record established that Protos intrinsically requires Maven Core 3.9.9.

## DIST008-A conclusion

DIST008-A investigated Protos HEAD
`754de7a2a2d73dd4b39109bb842522ed9cc8153a` and closed with sufficient
evidence for implementation.

The selected contract is:

```text
GraalVM / JDK / Truffle:
    exact repository-owned coordinates

Maven:
    stable Apache Maven 3.x
    minimum supported version 3.9.9
    no exact Maven Core patch requirement
    Maven 4.x and prerelease/RC lines outside the supported contract until
    separately validated
```

The current POM/plugin set has a documented mechanical floor of Maven 3.6.3,
but that is not adopted as the Protos support floor. Maven 3.8.x and earlier are
upstream-EOL, current Maven plugin compatibility policy has moved to Maven 3.9,
and the lowest current-Protos toolchain version with retained validation evidence
is Maven 3.9.9. Therefore the supported floor remains 3.9.9 while exact identity
is removed.

No current build, test, Native Image, portable-distribution, release-identity or
CI invariant was found that consumes exact Maven Core patch identity.

## Approved provisioning model

On 2026-09-29 the project owner approved the DIST008-A recommendation:

```text
Oracle Linux 10 RPM Maven
+
maven-unbound JDK binding
+
canonical GraalVM JAVA_HOME remains Java authority
```

The pinned GraalVM Community OL10 image already defines
`ol10_codeready_builder`, but leaves it disabled. No new repository file or
external repository URL needs to be added to the image. The intended Docker
transaction enables that already-defined repository only for the Maven
installation and installs the unbound binding.

The implementation direction is conceptually:

```dockerfile
RUN microdnf install -y \
        --enablerepo=ol10_codeready_builder \
        maven-unbound \
    && microdnf clean all
```

`maven-unbound` depends on `maven` and provides the required
`maven-jdk-binding` capability without requiring an OpenJDK RPM.

## Owner-executed package/runtime proof

The owner tested the exact GraalVM base image:

```text
ghcr.io/graalvm/graalvm-community:25i4-25.0.4.1.1-ol10
digest sha256:a7b4810d7c755e9627feaa1459eb5a93338643b16d745d4f3fc86db71e5da7f5
```

Read-only repository inventory reported:

```text
ol10_codeready_builder Oracle Linux 10 CodeReady Builder (x86_64) - (Unsupported) disabled
```

The owner then installed `maven` plus `maven-unbound` with
`--enablerepo=ol10_codeready_builder`. The transaction selected:

```text
maven-1:3.9.9-3.el10_1.noarch
maven-lib-1:3.9.9-3.el10_1.noarch
maven-unbound-1:3.9.9-3.el10_1.noarch
```

and installed no OpenJDK RPM.

Runtime identity was:

```text
Apache Maven 3.9.9 (Red Hat 3.9.9-3)
Maven home: /usr/share/maven
Java version: 25.0.4.1.1
vendor: GraalVM Community
runtime: /opt/graalvm-community-java25i4
```

This proves the selected package model removes the redundant distro JDK while
preserving the canonical GraalVM JDK as Maven's Java authority.

## Cross-repository scope

DIST008 is not confined to the `guillermomolina/protos` repository.

### guillermomolina/protos

Current implementation-owned surfaces include:

- `toolchain.json` — exact Maven identity must become minimum/supported-line
  semantics;
- `.devcontainer/Dockerfile` — use the unbound Maven binding;
- `build/native/Dockerfile` — use the same unbound binding;
- `tools/verify_toolchain.py` — separate compatibility floor, provisioning and
  Java authority instead of overloading one exact Maven coordinate;
- `tools/test_verify_toolchain.py` — cover supported-newer Maven, below-floor
  rejection, unsupported-major/prerelease policy as applicable, and both
  container bindings.

The root and Native Makefiles already invoke generic `mvn` and do not require
an exact Maven patch release.

### guillermomolina/protos-benchmarks

Current benchmark revision
`a2a8eafe74a45cce987a0918d1023be061ed17f3` contains live consumers of the
Protos toolchain contract that must be reconciled after the product contract is
published:

- `runner/toolchain.py` currently requires `maven.version` and exposes
  `maven_version`;
- `tests/test_toolchain.py` encodes exact-version fixtures;
- `docker/protos-dist006d/Dockerfile` installs plain OS `maven`, asserts exact
  Maven 3.9.9, and embeds an exact-version `toolchain.json` fixture.

The benchmark repository also contains many older Dockerfiles that intentionally
record historical benchmark environments, including old Maven builder images,
OL8/manual Maven bootstrap paths and old GraalVM/JDK coordinates. Those are
reproducibility evidence and MUST NOT be bulk-rewritten merely because they
contain Maven or a Dockerfile. Only current/live consumers of the present
toolchain contract are implementation targets.

### Other Protos companion repositories

Inventory at 2026-09-29 established:

- `guillermomolina/protos-vscode-extension` devcontainer is Node-based and does
  not install Maven: no DIST008 change.
- `guillermomolina/protos-website` intentionally uses
  `maven:3.9.16-eclipse-temurin-21` for documentation tooling. It does not use
  the canonical Protos GraalVM OL10 Maven binding and is not a DIST008 target.

Unrelated repositories outside the Protos project are outside DIST008 even if
they independently contain GraalVM/Maven Dockerfiles.

## Implementation decomposition

### DIST008-B1 — Protos toolchain/provisioning implementation — CLOSED

Repository: `guillermomolina/protos`.

Published at Protos revision
`6411d39bf33014c958ba4dad60a6e3fe44760ebf`.

B1 replaced the exact Maven patch contract with
`minimum_version=3.9.9` plus `supported_major=3`, provisioned
`maven+maven-unbound` from the already-defined `ol10_codeready_builder`
repository in both GraalVM OL10 container surfaces, and extended the repository
verifier/tests to distinguish compatibility, provisioning, repository scope,
canonical Java authority and runtime Maven/JDK/RPM evidence.

### DIST008-B2 — benchmark consumer reconciliation — CLOSED

Repository: `guillermomolina/protos-benchmarks`.

Published at benchmark revision
`e8a1f1735e0c2751de99459aeeb689a8d46e5f0d`
(`DIST008-B2: reconcile benchmark Maven compatibility`).

B2 consumed the authoritative `protos-toolchain-v2` contract published by B1,
migrated the live DIST006-D benchmark consumer to the Maven compatibility
contract, retained explicit support for replaying historical
`protos-toolchain-v1` evidence in PERF009-A, and changed no retained benchmark
measurement evidence.

The published B2 delta is exactly:

```text
config/dist006d-baseline.json
docker/protos-dist006d/Dockerfile
runner/dist006d_baseline.py
runner/perf009a.py
runner/toolchain.py
tests/test_dist006d_baseline.py
tests/test_toolchain.py
```

Final owner-executed admission established:

```text
TOOLCHAIN_SCHEMA=protos-toolchain-v2
MAVEN_MINIMUM_VERSION=3.9.9
MAVEN_SUPPORTED_MAJOR=3
EXACT_MAVEN_REQUIRED=NO
DIST006D_STATIC_VALIDATION=PASS
DIST006D_SMOKE_CORRECTNESS=PASS
REDUNDANT_OPENJDK=NO
MAVEN_GRAALVM_JAVA_AUTHORITY=PASS
RETAINED_PERFORMANCE_EVIDENCE=NO
TIMING_EVIDENCE=NO
REFERENCE_EVIDENCE=NO
```

The full benchmark `make test` / `make validate` gate remains red because of
one pre-existing unrelated PERF010-A help-surface assertion already present at
the exact B2 baseline
`a2a8eafe74a45cce987a0918d1023be061ed17f3`. DIST008-B2 modifies no PERF010-A
path. Focal B2 tests, static DIST006-D validation, whitespace checks, and the
non-retained DIST006-D Docker smoke against exact Protos B1 revision
`6411d39bf33014c958ba4dad60a6e3fe44760ebf` all passed.

This two-repository sequence remains slices under DIST008: B2 consumed the B1
contract and required no independently schedulable child Issue.

## DIST008-B1 implementation result

DIST008-B1 is **CLOSED / PASS** at Protos revision
`6411d39bf33014c958ba4dad60a6e3fe44760ebf`.

The published delta is exactly:

```text
.devcontainer/Dockerfile
build/native/Dockerfile
toolchain.json
tools/test_verify_toolchain.py
tools/verify_toolchain.py
```

The resulting contract is:

```text
TOOLCHAIN_SCHEMA=protos-toolchain-v2
MAVEN_MINIMUM_VERSION=3.9.9
MAVEN_SUPPORTED_MAJOR=3
EXACT_MAVEN_REQUIRED=NO
```

Owner-executed validation established:

- verifier focal tests PASS;
- static development and all-surface bindings PASS with zero drift;
- rebuilt development container uses `maven-3.9.9` plus
  `maven-unbound-3.9.9`, runs Maven on the canonical GraalVM
  `JAVA_HOME`, and installs no redundant OpenJDK RPM;
- rebuilt Native Image builder has the same Maven/unbound result and canonical
  GraalVM Java authority, with no redundant OpenJDK RPM;
- integrated `make test` PASS after the development-container rebuild;
- portable POSIX/JVM distribution build and cross-slice validation PASS,
  including the bundled repository corpus at `1263 passed, 0 failed`.

B1 validates the Native **builder/provisioning** impact. It does not claim a
fresh DIST005 Native executable admission because B1 changes neither the Native
executable semantics nor its distribution/admission machinery.

Durable B1 evidence was first published at project-docs revision
`b741ae42cfadfc4ac9790e765a7ca5438e7090a6`:

- `docs/project/evidence/DIST008/DIST008_B1_PROTOS_MAVEN_COMPATIBILITY_IMPLEMENTATION.md`

DIST008-B2 is now closed at benchmark revision
`e8a1f1735e0c2751de99459aeeb689a8d46e5f0d`.

## DIST008 closure result

DIST008 is **CLOSED / PASS**.

Exact closure coordinates:

```text
PROTOS_REVISION=6411d39bf33014c958ba4dad60a6e3fe44760ebf
BENCHMARK_REVISION=e8a1f1735e0c2751de99459aeeb689a8d46e5f0d
TOOLCHAIN_SCHEMA=protos-toolchain-v2
MAVEN_MINIMUM_VERSION=3.9.9
MAVEN_SUPPORTED_MAJOR=3
EXACT_MAVEN_REQUIRED=NO
PROVISIONING_MODEL=OL10 maven + maven-unbound
REDUNDANT_JDK_PROVISIONING=ELIMINATED
GRAALVM_JDK_TRUFFLE_COORDINATES_UNCHANGED=YES
```

DIST008-A established and retained the compatibility/provisioning evidence.
DIST008-B1 published the canonical Protos contract and both live GraalVM OL10
provisioning surfaces. DIST008-B2 reconciled the live benchmark consumer while
preserving historical v1 benchmark evidence.

The top-level acceptance criteria are satisfied by the combined A/B1/B2
evidence. The unrelated pre-existing PERF010-A help assertion in
`protos-benchmarks` is not a DIST008 regression and is not part of this work's
published path set.

Durable B2/closure evidence:

- `docs/project/evidence/DIST008/DIST008_B2_BENCHMARK_CONSUMER_RECONCILIATION_AND_CLOSURE.md`

No new Dxxx/PLATxxx decision was required.

## Historical evidence

DIST008 changes the current/future contract. It does not rewrite historical
records that truthfully state Maven 3.9.9, manual Apache Maven provisioning,
OpenJDK-bound Maven images, or older GraalVM/JDK coordinates.

In particular preserve DIST002, UPSTREAM002, DIST004 and benchmark measurement
evidence under their exact historical environments.

## External evidence

- Apache Maven plugin compatibility plan:
  https://maven.apache.org/developers/compatibility-plan.html
- Apache Maven releases/EOL history:
  https://maven.apache.org/docs/history.html
- Oracle Linux 10 CodeReady Builder package listing containing
  `maven-unbound-3.9.9-3.el10_1`:
  https://yum.oracle.com/repo/OracleLinux/OL10/codeready/builder/x86_64/index.html
- EL10 Maven package metadata: `maven` requires the
  `maven-jdk-binding` capability:
  https://fr2.rpmfind.net/linux/RPM/centos-stream/10/appstream/x86_64/maven-3.9.9-3.el10.noarch.html
- EL10 `maven-unbound` metadata: provides `maven-jdk-binding` and requires
  `maven`/javapackages tooling without an OpenJDK dependency:
  https://fr.rpmfind.net/linux/RPM/almalinux-kitten/10/devel/x86_64/maven-unbound-3.9.9-3.el10.x86_64.html
- EL10 OpenJDK 25 binding metadata: provides `maven-jdk-binding` but requires
  `java-25-openjdk-headless`:
  https://rpmfind.net/linux/RPM/almalinux-kitten/10/devel/aarch64/maven-openjdk25-3.9.9-3.el10.aarch64.html

## Design authority

No new Dxxx/PLATxxx decision is required. This is distribution/toolchain
implementation policy within the already-established canonical GraalVM/JDK
authority.
