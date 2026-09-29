# DIST005-D8 — exact selected JVM_PLUS_NATIVE candidate preparation, measurements and pre-publication proof

Status: READY FOR PUBLICATION AUTHORIZATION
Date: 2026-09-29
Owning Issue: `guillermomolina/protos#549`
Slice: `DIST005-D8 — exact selected JVM_PLUS_NATIVE candidate preparation, measurements and pre-publication proof`
Work type: IMPLEMENTATION / RELEASE PREPARATION
Implementation repository: `guillermomolina/protos`

## Boundary

DIST005-D8 prepared and fully admitted the exact owner-selected
`JVM_PLUS_NATIVE` release candidate for `0.3.116`.

D8 did not authorize or perform publication. No Git tag, GitHub Release, public
asset upload, release deletion/replacement, force-push, Try Protos documentation
change, or root README first-run change occurred.

The release selection was inherited unchanged from owner-approved DIST005-D7.

## Exact selected release identity

```text
RELEASE_BASELINE_REVISION=1637fe514ee9327241e49964031d981423b16198
RELEASE_BASELINE_VERSION=0.3.116-SNAPSHOT
RELEASE_CANDIDATE_SOURCE_REVISION=0336ae20216bf2eec17854bea0f6435e4e1e9b19
RELEASE_VERSION=0.3.116
RELEASE_TAG=v0.3.116
SPECIFICATION_REVISION=0.1.434

DIST005_SELECTED_MODEL=JVM_PLUS_NATIVE
RECOMMENDED_FIRST_RUN_ARTIFACT=NATIVE
PORTABLE_JVM_ROLE=COMPATIBILITY_FALLBACK_AND_EXTERNAL_CONSUMER_ARTIFACT
```

The exact D8 selection record used the existing
`protos-dist001-e4-selection-v1` format with
`release_publication_authorized=false`.

## Retained Native and runtime policy

```text
PUBLIC_NATIVE_PLATFORM_MATRIX=LINUX_X86_64_GLIBC_DYNAMIC
PUBLIC_NATIVE_PLATFORM_BUILD_BASE=ORACLE_LINUX_10
PUBLIC_NATIVE_GLIBC_MIN=2.39
PUBLIC_NATIVE_CPU_ISA_POLICY=COMPATIBILITY
PUBLIC_NATIVE_BUILD_MARCH=-march=compatibility

GRAALVM_DISTRIBUTION=graalvm-community
GRAALVM_GRAAL_TRUFFLE_VERSION=25.4.4.1.1
JDK_FEATURE=25
JDK_VERSION=25.0.4.1.1
JAVA_BYTECODE_RELEASE=21
MAVEN_MINIMUM_VERSION=3.9.9
MAVEN_SUPPORTED_MAJOR=3
```

No Native support is inferred for macOS, Windows, AArch64, musl, earlier glibc,
or a broader generic-Linux target.

The portable JVM artifact remains dependent on the declared supported external
GraalVM/JDK runtime. Java bytecode release 21 is not a generic Java 21+ support
claim.

## Exact candidate preparation

D8 used the existing authoritative preparation path:

```text
dist/prepare_release.py
```

The path composed the already-proven deterministic flow:

```text
explicit selection
    ->
detached release-only candidate
    ->
Native Image build
    ->
Native archive build
    ->
complete Native admission
    ->
portable JVM build
    ->
complete portable JVM admission
    ->
multi-asset envelope preparation
    ->
independent multi-asset envelope verification
```

The exact preparation completed successfully:

```text
DIST005_D8_PREPARE_STATUS=0
NATIVE_CLEAN_HOST_PROOF=PASS
NATIVE_COMPLETE_ADMISSION=PASS
PORTABLE_JVM_ADMISSION=PASS
MULTI_ASSET_ENVELOPE=PASS
COMMON_CANDIDATE_IDENTITY=PASS
CANDIDATE_FINAL_STATUS=CLEAN
```

This is selected-candidate proof. It does not merely reuse the historical
DIST005-D4/D5 proof candidates.

## Exact artifacts and release envelope

Prepared archives:

```text
NATIVE_ARCHIVE=target/distributions/protos-0.3.116-native-linux-x86_64.zip
NATIVE_ARCHIVE_SHA256=61fd90b39a43c574900b3c61d4fe2e65e100166ced66e495281b336f934883e8

PORTABLE_JVM_ARCHIVE=target/distributions/protos-0.3.116-posix-jvm.zip
PORTABLE_JVM_ARCHIVE_SHA256=ac5848ce3e4f371e7424ca5628899cc53a4d0be23d09381ca48db68218cd2add

RELEASE_MANIFEST_SHA256=7704d1a10072c1d906ae529b1824b40938dbf0a827f0645904ed9a1a149c52f1
RELEASE_NOTES_SHA256=a888884fedc373547bd15cee29a4625cd792327d7b5113fd9e03242b153f8b12
```

The release envelope contains the two external checksum files,
`RELEASE_MANIFEST.txt`, and `RELEASE_NOTES.md`.

The selected public downloadable asset model remains exactly five assets:

```text
1. protos-0.3.116-native-linux-x86_64.zip
2. protos-0.3.116-native-linux-x86_64.zip.sha256
3. protos-0.3.116-posix-jvm.zip
4. protos-0.3.116-posix-jvm.zip.sha256
5. RELEASE_MANIFEST.txt
```

`RELEASE_NOTES.md` is the GitHub Release body, not a sixth downloadable asset.

## Artifact-size evidence

Exact selected-candidate measurements:

```text
NATIVE_ARCHIVE_BYTES=26331374
PORTABLE_JVM_ARCHIVE_BYTES=25295300

NATIVE_EXTRACTED_BYTES=70615068
PORTABLE_JVM_EXTRACTED_BYTES=44174131

NATIVE_EXECUTABLE_BYTES=68029432
ARTIFACT_SIZE_EVIDENCE=PASS
```

The Native launcher `bin/protos` is a 4,823-byte POSIX shell launcher. The
`NATIVE_EXECUTABLE_BYTES` value above intentionally measures the real stripped
ELF executable at:

```text
libexec/protos-native
```

The executable was observed as an x86-64 dynamically linked PIE using
`/lib64/ld-linux-x86-64.so.2`.

## Startup-cost evidence

Method:

```text
same-host independent process launches of freshly extracted bin/protos -e 'null';
Unix archive modes restored from ZIP metadata;
OS page cache not forcibly flushed
```

Each artifact used one separately recorded first process launch followed by ten
additional independent process launches. The same host and trivial executable
workload were used for both artifacts.

Results:

```text
NATIVE_STARTUP_FIRST_MS=15.736
NATIVE_STARTUP_MIN_MS=12.782
NATIVE_STARTUP_MEDIAN_MS=13.177
NATIVE_STARTUP_MAX_MS=14.246

PORTABLE_JVM_STARTUP_FIRST_MS=615.753
PORTABLE_JVM_STARTUP_MIN_MS=594.304
PORTABLE_JVM_STARTUP_MEDIAN_MS=620.953
PORTABLE_JVM_STARTUP_MAX_MS=677.197

STARTUP_COST_EVIDENCE=PASS
```

These numbers are release-engineering same-host startup observations for
`protos -e 'null'`. They are not a cold-boot/cold-cache benchmark and do not
establish or advertise a generic startup, throughput, or overall performance
improvement.

## Release claims

The selected candidate has admitted the release claims prepared in D7:

- Native is the recommended first-run artifact.
- On the declared supported Native target, the extracted Native artifact needs
  no external Java/GraalVM runtime, Maven, or Protos source checkout.
- The Native artifact admits version reporting, direct `-e`, source execution
  from an unrelated caller CWD, Package/Workspace execution, Test Tool, REPL,
  LSP, DAP, and the validated optimizing Truffle guest/runtime-compilation path.
- The portable POSIX/JVM artifact is retained from the exact same candidate as
  the compatibility fallback and external-consumer artifact under the declared
  supported GraalVM/JDK contract.

Retained limitations:

- This remains an experimental GitHub prerelease and Protos/Core v0.1 remains
  draft.
- Native support is exactly Linux x86_64 with dynamic glibc linkage, Oracle
  Linux 10 build baseline, declared glibc minimum 2.39, and
  `-march=compatibility`.
- No Native support is claimed for macOS, Windows, AArch64, musl, or earlier
  glibc.
- The portable JVM artifact is not self-contained.
- No generic performance improvement is claimed merely because the Native Image
  artifact exists.

```text
RELEASE_CLAIMS_READY=YES
```

## Publication boundary and final state

The final public-state check preserved the D8 boundary:

```text
RELEASE_PUBLICATION_AUTHORIZED=NO
PUBLICATION_AUTHORIZATION_CREATED=NO
TAG_CREATED=NO
PUBLIC_RELEASE_CREATED=NO
PUBLIC_ASSETS_UPLOADED=NO

DIST007_V0_3_116_AVAILABLE=NO
BLOCKERS=NONE
OWNER_PUBLICATION_DECISION_REQUIRED=YES
```

The exact candidate is therefore prepared and frozen for the next explicit
publication decision without having changed public release state.

## Result

```text
DIST005_D8_STATUS=READY_FOR_PUBLICATION_AUTHORIZATION

RELEASE_BASELINE_REVISION=1637fe514ee9327241e49964031d981423b16198
RELEASE_BASELINE_VERSION=0.3.116-SNAPSHOT
RELEASE_CANDIDATE_SOURCE_REVISION=0336ae20216bf2eec17854bea0f6435e4e1e9b19
RELEASE_VERSION=0.3.116
RELEASE_TAG=v0.3.116
SPECIFICATION_REVISION=0.1.434

NATIVE_COMPLETE_ADMISSION=PASS
PORTABLE_JVM_ADMISSION=PASS
MULTI_ASSET_ENVELOPE=PASS
COMMON_CANDIDATE_IDENTITY=PASS
CANDIDATE_FINAL_STATUS=CLEAN

ARTIFACT_SIZE_EVIDENCE=PASS
STARTUP_COST_EVIDENCE=PASS
RELEASE_CLAIMS_READY=YES

BLOCKERS=NONE
OWNER_PUBLICATION_DECISION_REQUIRED=YES
```

## Materially governing and inspected repository surfaces

The D8 orchestration was constrained by the following repository surfaces:

- `AGENTS.md` — Human-executor mode, exact project coordinates, release and
  repository-publication safety.
- `AGENTS.work/IMPLEMENTATION.md` — shared implementation/validation
  discipline for the DIST implementation slice.
- `AGENTS.work/RELEASE.md` — release selection/publication separation and
  explicit authorization boundary.
- `dist/prepare_release.py` — authoritative selected-candidate preparation,
  build, admission, envelope verification, exact identity, and clean-state
  flow.
- `dist/publish_release.py` — separate D6 publication primitive; deliberately
  not invoked during D8.
- the existing DIST005 D4-D7 durable records — historical machinery/policy
  evidence and the exact D7 owner-selected identity.

No normative Protos language or Standard Library semantics were changed by D8.

## Next bounded slice

```text
NEXT_SLICE=DIST005-D9
NEXT_SLICE_NAME=explicit publication authorization and real multi-asset publication
NEXT_SLICE_WORK_TYPE=IMPLEMENTATION / RELEASE PUBLICATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
```

D9 may proceed only after explicit project-owner authorization for the exact
candidate and manifest above. It must use the existing fail-closed publication
primitive and must independently verify the public tag, GitHub prerelease,
five-asset set, asset digests, release body, and manifest identity after
publication.
