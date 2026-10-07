# DIST005-D2 — Native public platform and CPU-ISA selection

Status: OWNER APPROVED
Date: 2026-09-29
Owning Issue: `guillermomolina/protos#549`
Slice: `DIST005-D2 — Native public platform and CPU-ISA selection packet`

## Selection identity

The read-only DIST005-D2 investigation inspected:

```text
PROTOS_REVISION=80fb2650399f7edce35599c2399f6e6019660468
PROTOS_VERSION=0.3.111-SNAPSHOT
PROJECT_RECORD_BASE_REVISION=5b6277d29364cb8b615b3e2222cfee23d0d98eb7
```

The project owner explicitly approved the recommended selection on 2026-09-29:

```text
PLATFORM=P2
CPU_ISA=B
ABI_POLICY=GLIBC_2.34
```

The selected public Native policy is therefore:

```text
PUBLIC_NATIVE_PLATFORM_OPTION=P2-LINUX_X86_64_OL9
PUBLIC_NATIVE_OS=linux
PUBLIC_NATIVE_ARCH=x86_64
PUBLIC_NATIVE_LIBC=glibc
PUBLIC_NATIVE_LINKAGE=dynamic

PUBLIC_NATIVE_CPU_ISA_POLICY=COMPATIBILITY
PUBLIC_NATIVE_BUILD_MARCH=-march=compatibility

PUBLIC_NATIVE_ABI_POLICY=GLIBC_2.34
PUBLIC_NATIVE_GLIBC_MIN=2.34

INITIAL_NATIVE_AARCH64_SUPPORT=NO
INITIAL_NATIVE_MACOS_SUPPORT=NO
INITIAL_NATIVE_WINDOWS_SUPPORT=NO
INITIAL_NATIVE_MUSL_SUPPORT=NO
INITIAL_NATIVE_STATIC_SUPPORT=NO
```

This approval selects build/support policy. It does not prove a clean public
Native candidate, publish an artifact, create a tag/GitHub Release, or claim
support outside the selected initial matrix.

## Existing product decisions retained

DIST005-C remains authoritative:

```text
DIST005_SELECTED_MODEL=JVM_PLUS_NATIVE
RECOMMENDED_FIRST_RUN_ARTIFACT=NATIVE
PORTABLE_JVM_ARTIFACT=RETAIN
PORTABLE_JVM_ROLE=COMPATIBILITY_FALLBACK_AND_EXTERNAL_CONSUMER_ARTIFACT
DIST001_RELEASE_IDENTITY_MODEL=RETAIN
SECOND_RELEASE_IDENTITY_MODEL=NO
TRUFFLE_RUNTIME_COMPILER_IN_NATIVE=REQUIRED
EXTERNAL_PROTOS_RESOURCE_TREE_PRESERVED=YES
```

DIST005-D1 remains the multi-asset release-envelope foundation and continues to
fail closed while public Native platform/ISA identity is unresolved.

## Current proven development target

DIST005-B proved one complete Native development distribution for:

```text
target_os=linux
target_arch=x86_64
libc_family=glibc
linkage=dynamic

NATIVE_RELOCATABLE_ARCHIVE_PROOF=PASS
NATIVE_EXTERNAL_JAVA_REQUIRED=NO
NATIVE_EXTERNAL_MAVEN_REQUIRED=NO
NATIVE_EXTERNAL_PROTOS_CHECKOUT_REQUIRED=NO

NATIVE_RUN_SURFACE=PASS
NATIVE_PACKAGE_SURFACE=PASS
NATIVE_TEST_SURFACE=PASS
NATIVE_REPL_SURFACE=PASS
NATIVE_LSP_SURFACE=PASS
NATIVE_DAP_SURFACE=PASS
NATIVE_OPTIMIZING_TRUFFLE_SURFACE=PASS

NATIVE_FULL_TEST_TOOL=1263/1263_PASS
DIST005_NATIVE_COMPLETE_ADMISSION=PASS
```

That development proof was built with the then-current Oracle Linux 10 Native
Image builder and reported an `x86-64-v3` target machine while deliberately
recording:

```text
cpu_isa_assumption=unresolved
```

It is therefore not a clean release candidate and is not, by itself, a broad
Linux/x86_64 compatibility promise.

## CPU-ISA selection

Current GraalVM JDK 25 Native Image documentation states:

- AMD64 defaults to `x86-64-v3`;
- `-march=compatibility` reduces the instruction set for best compatibility;
- `-march=native` uses CPU features detected on the build machine.

Authoritative upstream reference:

https://www.graalvm.org/jdk25/reference-manual/native-image/overview/Options/

For a generally downloadable recommended-first-run artifact, the selected
policy is:

```text
-march=compatibility
```

The public build must set this explicitly. It must not rely on the GraalVM
default and must not use `-march=native`.

The implementation must record the selected policy in artifact/runtime
metadata. The later clean-candidate proof must also capture the exact Native
Image target-machine/build evidence rather than infer support solely from the
policy token.

## Linux builder and ABI selection

The current Native builder is:

```text
ghcr.io/graalvm/native-image-community:25i4-25.0.4.1.1-ol10
```

GraalVM Community container documentation defines Oracle Linux 8, 9 and 10
variants and gives `25i4-25.0.4.1.1-ol9` as a valid specific-tag form for the
25.4.4.1.1 line.

Authoritative upstream reference:

https://www.graalvm.org/docs/getting-started/container-images/

Oracle Linux 9 uses the `ol9_codeready_builder` repository name.

Authoritative Oracle references:

https://docs.oracle.com/en/learn/ol-dnf/
https://docs.oracle.com/en/operating-systems/oracle-linux/software-management/sfw-mgmt-AvailableYumRepositories.html

Current Oracle Linux 9 package evidence identifies glibc 2.34 on x86_64.

Authoritative Oracle evidence:

https://linux.oracle.com/errata/ELSA-2026-20597.html

The selected initial public Native build baseline is therefore:

```text
builder_family=Oracle Linux 9
native_image_container=ghcr.io/graalvm/native-image-community:25i4-25.0.4.1.1-ol9
libc_family=glibc
libc_abi_min=2.34
linkage=dynamic
```

This is a build/support policy, not yet runtime proof. A later slice must prove
that the exact clean candidate satisfies the selected ABI floor and required
dynamic-library closure.

## D1 metadata consequence

DIST005-D1 currently records Native runtime/platform identity including:

```text
target_os
target_arch
libc_family
linkage
cpu_isa_assumption
```

A public Linux support promise also needs an explicit ABI-floor field.

Selected minimum addition:

```text
libc_abi_min
```

with the initial selected value:

```text
libc_family=glibc
libc_abi_min=2.34
```

Therefore:

```text
D1_SCHEMA_FOLLOWUP_REQUIRED=YES
```

The release-envelope preparation and verification paths must fail closed if a
Native artifact lacks this ABI-floor identity or disagrees with the selected
public build policy.

## Alternatives not selected

### Current OL10 / x86-64-v3 development configuration

Technically viable but not selected as the initial public recommendation because
it couples the recommended Native artifact to a newer userspace baseline and a
narrower CPU feature level than necessary.

### `-march=native`

Not viable for the general downloadable artifact. It would make public CPU
requirements depend on the build host and weaken reproducibility/support
identity.

### Linux/AArch64

GraalVM upstream capability exists, but Protos has not performed the required
AArch64 closed-world, tool-surface, optimizer, clean-environment, ABI and
release-envelope proof. It is not part of the initial public support matrix.

### macOS, Windows, musl and static Native images

No current Protos evidence establishes these as supported Native public targets.
They remain outside the selected initial matrix.

## Required implementation/proof decomposition

The selected policy must be completed in bounded slices.

### DIST005-D3 — selected Native build policy and ABI metadata implementation

Implementation in `guillermomolina/protos`.

Own only:

- switch the canonical Native build baseline from OL10 to OL9;
- update the Oracle Linux CodeReady Builder repository selector accordingly;
- make `-march=compatibility` explicit in the Native Image build;
- replace unresolved development ISA metadata with the selected explicit public
  policy where the release/build surface requires it;
- add `libc_abi_min` to Native runtime/release-envelope identity;
- set the selected initial ABI policy to glibc 2.34;
- extend preparation/verification/tests so missing or inconsistent ISA/ABI
  policy fails closed;
- preserve the portable JVM artifact and legacy single-JVM release path.

D3 must not create a public release candidate or claim that the selected policy
has been runtime-proven merely because the machinery was implemented.

### DIST005-D4 — clean Native candidate proof

After D3 publication, construct and validate the exact clean Native candidate.
This proof must cover at least:

- exact source/candidate identity;
- selected OL9 build environment;
- selected CPU-ISA build policy and observed build target evidence;
- glibc/ABI verification;
- dynamic-library closure;
- clean-environment smoke;
- full Native CLI/tool-surface admission;
- full current Native test-tool admission;
- Truffle runtime-compilation admission;
- multi-asset envelope verification;
- artifact integrity.

### Later bounded closure

Artifact-size/startup measurements, final license/notice review, public wording
and release publication remain later release boundaries.

## Result

```text
DIST005_D2_STATUS=OWNER_APPROVED

PROTOS_REVISION=80fb2650399f7edce35599c2399f6e6019660468
PROTOS_VERSION=0.3.111-SNAPSHOT

DIST005_SELECTED_MODEL=JVM_PLUS_NATIVE

PUBLIC_NATIVE_PLATFORM_MATRIX=LINUX_X86_64_GLIBC_DYNAMIC
PUBLIC_NATIVE_PLATFORM_BUILD_BASE=ORACLE_LINUX_9
PUBLIC_NATIVE_CPU_ISA_POLICY=COMPATIBILITY
PUBLIC_NATIVE_BUILD_MARCH=-march=compatibility
PUBLIC_NATIVE_ABI_POLICY=GLIBC_2.34
PUBLIC_NATIVE_GLIBC_MIN=2.34

D1_SCHEMA_FOLLOWUP_REQUIRED=YES
D1_MINIMUM_SCHEMA_ADDITION=libc_abi_min

INITIAL_NATIVE_AARCH64_SUPPORT=NO
NATIVE_CLEAN_RELEASE_CANDIDATE_PROVEN=NO
PUBLIC_NATIVE_RELEASE_ARTIFACT_CREATED=NO
PUBLIC_RELEASE_CREATED=NO

NEXT_SLICE=DIST005-D3
NEXT_SLICE_WORK_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
```
