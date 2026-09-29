# DIST008 — Maven compatibility floor and lean provisioning

Status: READY
Live coordination: GitHub Issue #739
Investigation slice: DIST008-A CLOSED
Next implementation slice: DIST008-B1 READY
Protos investigation revision: `754de7a2a2d73dd4b39109bb842522ed9cc8153a`
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

### DIST008-B1 — Protos toolchain/provisioning implementation — READY

Repository: `guillermomolina/protos`.

Implement the selected Maven compatibility contract and the OL10
`maven-unbound` provisioning in both GraalVM container surfaces, update the
verifier/tests coherently, and validate the resulting product/build/distribution
surfaces.

### DIST008-B2 — benchmark consumer reconciliation — PENDING B1

Repository: `guillermomolina/protos-benchmarks`.

After B1 publishes the authoritative toolchain schema/contract, update only the
benchmark repository's live current-contract consumers. Preserve historical
Dockerfiles and retained measurements exactly where their old Maven/JDK identity
is part of reproducibility evidence.

This two-repository sequence remains slices under DIST008: B2 consumes the B1
contract and is not independently schedulable before B1.

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
