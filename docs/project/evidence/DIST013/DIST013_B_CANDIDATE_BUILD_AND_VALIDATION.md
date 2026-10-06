# DIST013-B — Protos 0.3.237 candidate build and validation

Status: **PASS / publication not authorized**

DIST013-B closed the exact prepublication candidate selected in DIST013-A2. No tag, GitHub Release, or public asset upload was performed.

## Exact release identity

```text
RELEASE_BASELINE_REVISION=72c0515a707b876697c5a3afb96b0672264eac61
RELEASE_BASELINE_VERSION=0.3.237-SNAPSHOT
RELEASE_VERSION=0.3.237
RELEASE_TAG=v0.3.237
SPECIFICATION_REVISION=0.1.444
CANDIDATE_SOURCE_REVISION=f627734c8feaf3c26c4a32903b6ea0e25f4a9956
```

The detached candidate is a single-parent release-only commit over the selected baseline and changes only the root `pom.xml` project version from `0.3.237-SNAPSHOT` to `0.3.237`. The candidate remained detached and clean, has no branch/tag/remote ref, and future tag `v0.3.237` was available when the final lineage gate ran.

## Native artifact

```text
NATIVE_ARCHIVE=protos-0.3.237-native-linux-x86_64.zip
NATIVE_SHA256=510537c7c47a323ed05a5ea8e4e46bc3cd63c761a58e68cb0639548feb8cc46c
NATIVE_COMPLETE_ADMISSION=PASS
NATIVE_FULL_TEST_TOOL=2241 passed, 0 failed
NATIVE_GUEST_JIT=UNSUPPORTED_UPSTREAM_ORACLE_GRAAL_14579
```

Native admission included relocation/no-Java/no-Maven execution, package/workspace execution, Test Tool focal and full-suite admission, REPL, LSP, DAP, interpreter-only PLAT045 evidence, ELF/glibc/runtime checks, outer checksum verification, and archive-unchanged verification.

Two release-validation defects were repaired on active `main` during admission without changing or rebuilding the candidate artifact:

```text
PLAT045_WARNING_FIX_REVISION=4bb7af1df13584c601d8ce178067c385a2b9d73d
TEST_TOOL_PROGRESS_FIX_REVISION=a08e89f816e75d9ca2d0448be7edf4b5c8a76aac
```

The first made the validator consume repeated copies of the exact upstream PLAT045 interpreter-only warning while continuing to reject any other stderr. The second aligned Native Test Tool progress expectations with the TOOL011-B authoritative `[library/uri]` grouping and added a regression guard.

## Portable JVM artifact

```text
PORTABLE_JVM_ARCHIVE=protos-0.3.237-posix-jvm.zip
PORTABLE_JVM_SHA256=28f200f3c1d9b218ddbfdb5c906204418c543260371a4ca58119a228d5e79565
PORTABLE_JVM_ADMISSION=PASS
PORTABLE_TEST_TOOL=2241 passed, 0 failed
DIST001_B5_CROSS_SLICE=PASS
```

The portable archive passed source identity, public-prerelease mode, clean-source, internal checksum coverage/value, outside-checkout execution, supported runtime isolation, caller-CWD and Package Tool checks, bundled Test Tool, exact GraalVM/JDK optimizer/runtime gates, and composed B5 validation.

## Multi-asset envelope

Generation and independent verification both passed for the JVM_PLUS_NATIVE model:

```text
MULTI_ASSET_ENVELOPE_GENERATION=PASS
MULTI_ASSET_ENVELOPE_VERIFICATION=PASS
COMMON_CANDIDATE_IDENTITY=PASS
UNAMBIGUOUS_RECOMMENDED_FIRST_RUN=PASS
DETERMINISTIC_ASSET_ORDERING=PASS
```

Roles:

- Native Linux x86_64 / glibc 2.39+ / dynamic / compatibility ISA: recommended first run.
- Portable JVM: compatibility fallback.

Exact envelope digests:

```text
RELEASE_MANIFEST_SHA256=cf997603d218a5fbf181287a5d52d7b35ff82eaa86e5676e39e37c6540ab0458
RELEASE_NOTES_SHA256=f57432df668387c4a961faf3995e4d92e06c20875c603760bf8061494c540b9f
NATIVE_CHECKSUM_FILE_SHA256=2321a11dae0ca9feca2d891ef0a76cc2cc6df92a2e46fc78e1d96b5fdfe75e75
PORTABLE_CHECKSUM_FILE_SHA256=5a8587ac512abd1b7d652b74876932533822c1865e347bf00db24aa2bb7d5700
```

Per-artifact kind, role, platform, runtime, checksum, license/notices, common-candidate identity, and deterministic ordering all passed independent envelope verification.

## LM011-D1 release prerequisite proof

```text
LM011_D1_REVISION=75cfed853c7eef8c9d3dfb0c6f2507fdf8864ecb
LM011_D1_CANDIDATE_ANCESTOR=YES
DOCUMENT_FORMATTING_PROVIDER=YES
FORMATTER_AUTHORITY=ProtosWholeDocumentFormatter
LM011_D1_RUNTIME_PROOF=PASS
```

The exact candidate contains LM011-D1. Its LSP server advertises standard whole-document formatting and the LSP formatting path delegates to the same `ProtosWholeDocumentFormatter` authority used by the CLI path.

This satisfies DIST013-B's release prerequisite for LM011-E. LM011-E remains blocked until the candidate is actually published and independently verified in DIST013-C.

## Publication boundary

DIST013-B stops before publication.

```text
RELEASE_PUBLICATION_AUTHORIZED=NO
GIT_TAG_CREATED=NO
GITHUB_RELEASE_CREATED=NO
RELEASE_ASSETS_PUBLISHED=NO
```

DIST013-C requires a new, separate owner authorization bound exactly to:

```text
candidate_source_revision=f627734c8feaf3c26c4a32903b6ea0e25f4a9956
release_version=0.3.237
release_tag=v0.3.237
release_manifest_sha256=cf997603d218a5fbf181287a5d52d7b35ff82eaa86e5676e39e37c6540ab0458
github_release_prerelease=true
github_release_draft=false
```
