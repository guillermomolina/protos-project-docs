# DIST005-D3 — selected Native build policy and ABI metadata implementation

Status: PASS
Date: 2026-09-29
Owning Issue: `guillermomolina/protos#549`
Slice: `DIST005-D3 — selected Native build policy and ABI metadata implementation`

## Publication identity

DIST005-D3 was published to `guillermomolina/protos` as:

```text
PROTOS_REVISION=96f066c3689359afc0f03d6760de171bdfe2311f
PROTOS_VERSION=0.3.112-SNAPSHOT
COMMIT_SUBJECT=DIST005-D3: apply Native platform and ABI policy
PUBLICATION_BASE=80fb2650399f7edce35599c2399f6e6019660468
```

Changed product surfaces:

```text
CHANGELOG.md
dist/build_native.py
dist/release_asset_envelope.py
dist/test_multi_asset_release_metadata.py
pom.xml
```

No normative Protos language or Standard Library specification file changed.

## Owner reconciliation during D3

The earlier DIST005-D2 investigation selected Oracle Linux 9 / glibc 2.34 as
the initial compatibility-oriented Native build baseline. During D3
implementation, the project owner explicitly rejected that platform divergence
because the maintained Protos development and Native toolchain generation is
Oracle Linux 10.

The owner directed D3 to stay aligned with the maintained environment rather
than introduce a dedicated OL9 exception.

Therefore the historical D2 OL9 / glibc 2.34 selection is superseded for the
active public Native policy by the D3 owner reconciliation below:

```text
PUBLIC_NATIVE_PLATFORM_OPTION=P2-LINUX_X86_64_OL10
PUBLIC_NATIVE_PLATFORM_MATRIX=LINUX_X86_64_GLIBC_DYNAMIC
PUBLIC_NATIVE_PLATFORM_BUILD_BASE=ORACLE_LINUX_10

PUBLIC_NATIVE_CPU_ISA_POLICY=COMPATIBILITY
PUBLIC_NATIVE_BUILD_MARCH=-march=compatibility

PUBLIC_NATIVE_ABI_POLICY=GLIBC_2.39
PUBLIC_NATIVE_GLIBC_MIN=2.39
```

This reconciliation does not reopen the already-approved artifact model:

```text
DIST005_SELECTED_MODEL=JVM_PLUS_NATIVE
RECOMMENDED_FIRST_RUN_ARTIFACT=NATIVE
PORTABLE_JVM_ARTIFACT=RETAIN
PORTABLE_JVM_ROLE=COMPATIBILITY_FALLBACK_AND_EXTERNAL_CONSUMER_ARTIFACT
DIST001_RELEASE_IDENTITY_MODEL=RETAIN
```

The earlier D2 record remains historical evidence of the investigation and the
initial owner choice. This D3 record is the later project-owner-authorized
selection actually implemented and published.

## Implemented Native public policy

The published Native build policy is:

```text
target_os=linux
target_arch=x86_64
libc_family=glibc
libc_abi_min=2.39
linkage=dynamic
cpu_isa_assumption=compatibility

native_build_container=ghcr.io/graalvm/native-image-community:25i4-25.0.4.1.1-ol10
native_build_authority=build/native/Dockerfile
native_build_march=-march=compatibility
```

The canonical Native builder itself remains on the existing Oracle Linux 10
generation. D3 does not introduce a separate OL9 toolchain branch.

The explicit `-march=compatibility` setting is intentionally retained even
though the build base is OL10. The public Native CPU support policy must not
silently inherit a stronger default machine type from the toolchain or build
host.

## Native distribution metadata

`dist/build_native.py` now owns the selected public Native identity:

```text
PUBLIC_NATIVE_TARGET_OS=linux
PUBLIC_NATIVE_TARGET_ARCH=x86_64
PUBLIC_NATIVE_LINKAGE=dynamic
PUBLIC_NATIVE_LIBC_FAMILY=glibc
PUBLIC_NATIVE_LIBC_ABI_MIN=2.39
PUBLIC_NATIVE_CPU_ISA_ASSUMPTION=compatibility
```

The build fails closed if the configured Native builder, observed target OS,
observed target architecture, observed linkage, or observed libc family does
not match the selected public policy.

Generated Native `RUNTIME.txt` now records:

```text
target_os
target_arch
linkage
libc_family
libc_abi_min
cpu_isa_assumption
native_build_container
```

D3 records `libc_abi_min=2.39` as the selected release/support policy. It does
not claim that writing this token proves the ELF symbol-version floor of a
produced binary.

## Multi-asset envelope enforcement

`dist/release_asset_envelope.py` now requires an exact public Native runtime
identity:

```text
linux
x86_64
glibc
2.39
dynamic
compatibility
```

The Native artifact identity includes the ABI floor:

```text
native-linux-x86_64-glibc-2.39-dynamic-compatibility
linux/x86_64/glibc/2.39/dynamic/compatibility
```

Release-manifest and release-note generation expose the libc family, libc ABI
minimum and CPU ISA policy separately.

Preparation/verification fails closed for at least:

```text
missing libc_abi_min
wrong libc_abi_min
unresolved CPU ISA
x86-64-v3 public CPU policy
native host-specific CPU policy
wrong target OS
wrong target architecture
wrong libc family
wrong linkage
```

The portable JVM compatibility/fallback artifact and the legacy single-JVM
release path remain retained.

## Validation evidence

Focal and policy validation on the final D3 bytes reported:

```text
python syntax compilation=PASS
multi-asset envelope tests=12/12 PASS

TOOLCHAIN_VERIFIER_TESTS=PASS
TOOLCHAIN_DRIFT_COUNT=0
TOOLCHAIN_BINDINGS=PASS

builder_ol10=PASS
builder_ol10_codeready=PASS
march_compatibility_exactly_once=PASS
builder_glibc_2_39=PASS
builder_cpu_compatibility=PASS
envelope_glibc_2_39=PASS
envelope_cpu_compatibility=PASS
DIST005_D3_POLICY_SANITY=PASS

git diff --check=PASS
```

The canonical integrated repository validation was then run in the Oracle Linux
10 / GraalVM 25.4.4.1.1 environment:

```text
make test=PASS
PROTOS_TEST_TOOL=1263/1263 PASS
FULL_TEST_STATUS=0
META-INF/LICENSE.TXT in packaged JAR=PASS
```

After the exact candidate commit was created, immutable publication validation
for the range
`80fb2650399f7edce35599c2399f6e6019660468..96f066c3689359afc0f03d6760de171bdfe2311f`
reported:

```text
SOURCE_STYLE_PREVENTION_GATE=PASS
LEGACY_EXECUTION_PREVENTION_GATE=PASS
VALIDATION_IMPACT=FULL
AFFECTED_TEST_SET=ALL
FULL_TEST_SUITE=PASS
PUBLICATION_VALIDATION=PASS
PUBLICATION_VALIDATION_STATUS=0
```

The candidate remained unchanged and the worktree clean through publication.
The pushed `main` identity was verified as:

```text
HEAD=96f066c3689359afc0f03d6760de171bdfe2311f
origin/main=96f066c3689359afc0f03d6760de171bdfe2311f
```

## What D3 does not prove

D3 enforces selected policy and metadata but deliberately does not promote those
declarations into empirical release-candidate proof.

Still unresolved after D3:

```text
NATIVE_CLEAN_RELEASE_CANDIDATE_PROVEN=NO
PUBLIC_NATIVE_RELEASE_ARTIFACT_CREATED=NO
PUBLIC_RELEASE_CREATED=NO

ACTUAL_ELF_GLIBC_SYMBOL_FLOOR_PROVEN=NO
ACTUAL_DYNAMIC_LIBRARY_CLOSURE_PROVEN_FOR_CLEAN_CANDIDATE=NO
ACTUAL_NATIVE_BUILD_TARGET_EVIDENCE_RETAINED_FOR_CLEAN_CANDIDATE=NO
MULTI_ASSET_CLEAN_CANDIDATE_ENVELOPE_PROVEN=NO
```

In particular, `libc_abi_min=2.39` in `RUNTIME.txt` is a policy declaration.
DIST005-D4 must inspect the exact produced ELF binary and prove that its required
GLIBC symbol versions do not exceed the selected 2.39 floor.

## Next bounded slice

```text
DIST005-D4 — clean Native candidate proof
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
```

D4 must operate on the current published D3 policy and construct/validate an
exact clean candidate without creating a tag or GitHub Release.

At minimum it must prove:

- exact source/candidate identity;
- canonical Oracle Linux 10 Native build environment;
- explicit `-march=compatibility` build policy plus retained observed target
  evidence;
- actual ELF GLIBC symbol-version requirement not greater than 2.39;
- actual dynamic-library closure;
- clean-environment execution with no external Java/GraalVM/Maven;
- complete Native CLI/tool-surface admission;
- complete current Native Test Tool admission;
- retained Truffle runtime-compilation admission;
- Native archive integrity;
- multi-asset release-envelope compatibility for the exact candidate.

Artifact-size/startup measurements, final release wording, tag/GitHub Release
creation and public asset publication remain later boundaries unless the owning
DIST005 workflow explicitly advances them.

## Result

```text
DIST005_D3_STATUS=PASS

PROTOS_REVISION=96f066c3689359afc0f03d6760de171bdfe2311f
PROTOS_VERSION=0.3.112-SNAPSHOT

PUBLIC_NATIVE_PLATFORM_MATRIX=LINUX_X86_64_GLIBC_DYNAMIC
PUBLIC_NATIVE_PLATFORM_BUILD_BASE=ORACLE_LINUX_10
PUBLIC_NATIVE_CPU_ISA_POLICY=COMPATIBILITY
PUBLIC_NATIVE_BUILD_MARCH=-march=compatibility
PUBLIC_NATIVE_ABI_POLICY=GLIBC_2.39
PUBLIC_NATIVE_GLIBC_MIN=2.39

D1_SCHEMA_FOLLOWUP_REQUIRED=SATISFIED
D1_MINIMUM_SCHEMA_ADDITION=libc_abi_min

NATIVE_CLEAN_RELEASE_CANDIDATE_PROVEN=NO
PUBLIC_NATIVE_RELEASE_ARTIFACT_CREATED=NO
PUBLIC_RELEASE_CREATED=NO

NEXT_SLICE=DIST005-D4
NEXT_SLICE_WORK_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
```
