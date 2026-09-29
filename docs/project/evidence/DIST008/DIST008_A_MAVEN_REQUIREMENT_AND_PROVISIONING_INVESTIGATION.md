# DIST008-A — Maven requirement and provisioning investigation

Status: PASS
Date: 2026-09-29
Owning Issue: `guillermomolina/protos#739`
Investigated Protos revision:
`754de7a2a2d73dd4b39109bb842522ed9cc8153a`
Benchmark inventory revision:
`a2a8eafe74a45cce987a0918d1023be061ed17f3`

## Investigation boundary

DIST008-A was investigation-only. Repository and upstream evidence was read
without modifying the Protos or benchmark repositories. The runtime/package
proof recorded below was executed by the project owner and supplied as evidence.

## Repository findings

At the investigated Protos revision:

- `toolchain.json` declares `"maven": {"version": "3.9.9"}`;
- `tools/verify_toolchain.py` requires an exact `x.y.z` Maven coordinate;
- `.devcontainer/Dockerfile` installs plain OS `maven`;
- `build/native/Dockerfile` installs plain OS `maven`;
- the root and Native Makefiles invoke generic `mvn`;
- portable-distribution and publication-validation machinery invokes generic
  Maven lifecycle operations and does not consume exact Maven patch identity.

The explicitly configured Apache Maven plugins have a highest documented
mechanical Maven-Core prerequisite of 3.6.3. That is the theoretical plugin
floor, not the selected Protos support floor.

Apache Maven currently treats Maven 3.8.9 and earlier as EOL and the plugin
compatibility plan has moved current plugin API compatibility to Maven 3.9.
The lowest current Protos Maven version with retained relevant validation
evidence is 3.9.9.

Conclusion:

```text
MAVEN_DOCUMENTED_TECHNICAL_FLOOR=3.6.3
MAVEN_RECOMMENDED_PROTOS_FLOOR=3.9.9
EXACT_MAVEN_REQUIRED=NO
SUPPORTED_LINE=stable Maven 3.x >= 3.9.9
MAVEN_4_OR_PRERELEASE=OUTSIDE_CURRENT_CONTRACT
```

## RPM dependency finding

EL10 `maven` requires the RPM capability `maven-jdk-binding`.

An OpenJDK binding such as `maven-openjdk25` satisfies that capability but
hard-requires `java-25-openjdk-headless`, explaining the redundant JDK observed
in a GraalVM-based image.

EL10 `maven-unbound` instead:

```text
Provides:
    maven-jdk-binding

Requires:
    maven
    javapackages-tools
```

and does not require an OpenJDK RPM.

Oracle Linux 10 publishes `maven-unbound-3.9.9-3.el10_1` in
`ol10_codeready_builder`.

## Owner-executed GraalVM base-image proof

The project owner used:

```text
ghcr.io/graalvm/graalvm-community:25i4-25.0.4.1.1-ol10
```

The pull resolved to:

```text
sha256:a7b4810d7c755e9627feaa1459eb5a93338643b16d745d4f3fc86db71e5da7f5
```

Repository inventory inside that unmodified base image reported:

```text
ol10_codeready_builder Oracle Linux 10 CodeReady Builder (x86_64) - (Unsupported) disabled
```

This proves the repository definition is already present in the selected base
image; DIST008 does not need to add a new `.repo` file or external repository
URL.

The owner then executed an ephemeral container transaction equivalent to:

```text
microdnf install -y \
  --enablerepo=ol10_codeready_builder \
  maven \
  maven-unbound
```

The transaction completed successfully. Relevant installed packages included:

```text
maven-3.9.9-3.el10_1.noarch
maven-lib-3.9.9-3.el10_1.noarch
maven-resolver-1.9.18-4.el10.noarch
maven-shared-utils-3.4.2-8.el10.noarch
maven-unbound-3.9.9-3.el10_1.noarch
maven-wagon-3.5.3-9.el10.noarch
```

The explicit installed-Java RPM probe:

```text
rpm -qa | grep -Ei "openjdk|java-[0-9]+-openjdk"
```

returned no matches.

`mvn -version` reported:

```text
Apache Maven 3.9.9 (Red Hat 3.9.9-3)
Maven home: /usr/share/maven
Java version: 25.0.4.1.1
vendor: GraalVM Community
runtime: /opt/graalvm-community-java25i4
```

Therefore:

```text
MAVEN_UNBOUND_INSTALL=PASS
REDUNDANT_OPENJDK_INSTALLED=NO
MAVEN_JAVA_VENDOR=GraalVM Community
MAVEN_JAVA_RUNTIME=/opt/graalvm-community-java25i4
CANONICAL_GRAALVM_JDK_PRESERVED=YES
```

The warning about restricted native access emitted in the ad-hoc probe is not a
DIST008 blocker: the real Protos Dockerfiles already set
`MAVEN_OPTS=--enable-native-access=ALL-UNNAMED --sun-misc-unsafe-memory-access=allow`.

## Cross-repository inventory

### Protos

Revision:
`754de7a2a2d73dd4b39109bb842522ed9cc8153a`.

Current implementation surfaces:

```text
toolchain.json
.devcontainer/Dockerfile
build/native/Dockerfile
tools/verify_toolchain.py
tools/test_verify_toolchain.py
```

### Protos benchmarks

Revision:
`a2a8eafe74a45cce987a0918d1023be061ed17f3`.

Live current-contract consumers found:

```text
docker/protos-dist006d/Dockerfile
runner/toolchain.py
tests/test_toolchain.py
```

The Dockerfile currently installs plain OS `maven`, asserts exact Maven
3.9.9, and embeds an exact-version `toolchain.json` fixture. The runner also
requires `maven.version` as a pinned coordinate.

The repository contains many additional benchmark Dockerfiles, including
historical Maven builder images and manual Maven bootstrap environments. Those
are not automatically migration targets: benchmark reproducibility requires
preserving historical environment identity. DIST008-B2 must distinguish live
current-contract consumers from retained historical evidence.

### VS Code extension

Revision:
`6fff19e11246373605ec3265d7264c3b85431db0`.

`.devcontainer/Dockerfile` is based on Node and does not install Maven.
No DIST008 implementation change is required.

### Website

Revision:
`1834b97c4f2e208bb6335f90383d61ac791bb8b6`.

The website intentionally uses
`maven:3.9.16-eclipse-temurin-21` as documentation tooling and copies that
toolchain into its build base. It is not a consumer of the Protos canonical
GraalVM OL10 Maven binding and is outside DIST008.

## Owner approval

After reviewing the package/runtime proof, the project owner explicitly approved
the selected `maven-unbound` provisioning model on 2026-09-29.

## Decision packet

```text
DIST008_A_RESULT=PASS
REPOSITORY_HEAD=754de7a2a2d73dd4b39109bb842522ed9cc8153a

MAVEN_DOCUMENTED_TECHNICAL_FLOOR=3.6.3
MAVEN_RECOMMENDED_PROTOS_FLOOR=3.9.9
EXACT_MAVEN_REQUIRED=NO

SELECTED_PROVISIONING_MODEL=OL10 maven-unbound via already-defined ol10_codeready_builder
OS_PACKAGE_REDUNDANT_JDK=NO under selected unbound binding
CANONICAL_GRAALVM_JDK_PRESERVED=YES

PROPOSED_MAVEN_CONTRACT=stable Maven 3.x >=3.9.9; exact patch identity not required; Maven 4/prerelease outside current support
PROPOSED_VERIFIER_CONTRACT=verify supported Maven range, selected provisioning model, and canonical GraalVM Java authority separately

NEXT_SLICE=DIST008-B1
NEXT_REPOSITORY=guillermomolina/protos
FOLLOWUP_SLICE=DIST008-B2
FOLLOWUP_REPOSITORY=guillermomolina/protos-benchmarks

NEW_DESIGN_OR_PLATFORM_DECISION_REQUIRED=NO
BLOCKER=NONE
```

## Published external evidence

- https://maven.apache.org/developers/compatibility-plan.html
- https://maven.apache.org/docs/history.html
- https://yum.oracle.com/repo/OracleLinux/OL10/codeready/builder/x86_64/index.html
- https://fr2.rpmfind.net/linux/RPM/centos-stream/10/appstream/x86_64/maven-3.9.9-3.el10.noarch.html
- https://fr.rpmfind.net/linux/RPM/almalinux-kitten/10/devel/x86_64/maven-unbound-3.9.9-3.el10.x86_64.html
- https://rpmfind.net/linux/RPM/almalinux-kitten/10/devel/aarch64/maven-openjdk25-3.9.9-3.el10.aarch64.html
