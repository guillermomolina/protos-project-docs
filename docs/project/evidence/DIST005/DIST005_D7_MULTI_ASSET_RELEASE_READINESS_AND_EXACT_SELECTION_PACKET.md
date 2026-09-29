# DIST005-D7 — multi-asset release readiness and exact selection packet

Status: OWNER APPROVED
Date: 2026-09-29
Owning Issue: `guillermomolina/protos#549`
Slice: `DIST005-D7 — multi-asset release readiness and exact selection packet`
Work type: INVESTIGATION

## Investigation boundary

DIST005-D7 was a read-only release-readiness investigation. It executed no
commands, builds, tests, project programs, validation scripts, candidate
materialization, tag creation, GitHub Release publication, asset upload, Issue
mutation, or repository-content mutation in `guillermomolina/protos`.

The investigation reconciled current `guillermomolina/protos/main`, the live
DIST005/DIST007 Issues and public release state, the current release machinery,
and retained DIST005-D4/D5/D6 evidence.

## Current authoritative Protos state

At the D7 decision point:

```text
CURRENT_PROTOS_HEAD=1637fe514ee9327241e49964031d981423b16198
CURRENT_PROTOS_VERSION=0.3.116-SNAPSHOT
CURRENT_SPECIFICATION_REVISION=0.1.434

CANONICAL_GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1
CANONICAL_JDK_VERSION=25.0.4.1.1
JAVA_BYTECODE_RELEASE=21
```

The exact current HEAD is the published DIST005-D6 implementation revision and
contains the explicit fail-closed multi-asset publication primitive.

## Retained product and Native policy

No prior product-model or Native-target decision was reopened.

```text
DIST005_SELECTED_MODEL=JVM_PLUS_NATIVE
RECOMMENDED_FIRST_RUN_ARTIFACT=NATIVE
PORTABLE_JVM_ARTIFACT=RETAIN
PORTABLE_JVM_ROLE=COMPATIBILITY_FALLBACK_AND_EXTERNAL_CONSUMER_ARTIFACT

DIST001_RELEASE_IDENTITY_MODEL=RETAIN
SECOND_RELEASE_IDENTITY_MODEL=NO

PUBLIC_NATIVE_PLATFORM_MATRIX=LINUX_X86_64_GLIBC_DYNAMIC
PUBLIC_NATIVE_PLATFORM_BUILD_BASE=ORACLE_LINUX_10
PUBLIC_NATIVE_CPU_ISA_POLICY=COMPATIBILITY
PUBLIC_NATIVE_BUILD_MARCH=-march=compatibility
PUBLIC_NATIVE_ABI_POLICY=GLIBC_2.39
PUBLIC_NATIVE_GLIBC_MIN=2.39
```

The one advertised Native target plus the portable JVM fallback is sufficient
for DIST005 closure. No macOS, Windows, AArch64, musl, generic-Linux, or older
glibc support is inferred.

## Reusable historical proof versus selected-candidate proof

DIST005-D4 proved a complete clean detached `0.3.113` Native + portable JVM
candidate under the selected policy, including no-Java/no-Maven first-run,
`--version`, direct `-e`, unrelated-CWD source execution, Package Tool,
workspace execution, Test Tool, REPL, LSP, DAP, forced guest JIT/Tier-2,
archive integrity, provenance, checksums, license/notices, portable admission
and exact multi-asset-envelope identity.

DIST005-D5 proved deterministic exact-candidate preparation.

DIST005-D6 proved the explicit fail-closed resumable publication transaction and
completed full repository validation at the current HEAD.

Those results prove the machinery and policy, but they do not substitute for
proof over the exact candidate selected for the real public release.

The following must therefore be repeated for the selected candidate:

```text
NATIVE_CLEAN_HOST_PROOF=REQUIRES_NEW_CANDIDATE
NATIVE_COMPLETE_ADMISSION=REQUIRES_NEW_CANDIDATE
PORTABLE_JVM_ADMISSION=REQUIRES_NEW_CANDIDATE
MULTI_ASSET_ENVELOPE=REQUIRES_NEW_CANDIDATE
```

## Remaining parent acceptance criteria

The D7 classification of materially open DIST005 criteria is:

```text
Native one-archive/no-Java/no-Maven first-run=REQUIRES_NEW_SELECTED_CANDIDATE_PROOF
protos --version=REQUIRES_NEW_SELECTED_CANDIDATE_PROOF
direct -e execution=REQUIRES_NEW_SELECTED_CANDIDATE_PROOF
external source from unrelated CWD=REQUIRES_NEW_SELECTED_CANDIDATE_PROOF
absence of JAVA_HOME/PATH Java requirement=REQUIRES_NEW_SELECTED_CANDIDATE_PROOF
optimizing Truffle/guest runtime compilation=REQUIRES_NEW_SELECTED_CANDIDATE_PROOF
resource/stdlib/package/test/repl/lsp/dap closure=REQUIRES_NEW_SELECTED_CANDIDATE_PROOF
runtime/dependency identity and provenance=REQUIRES_NEW_SELECTED_CANDIDATE_PROOF
internal/external checksums=REQUIRES_NEW_SELECTED_CANDIDATE_PROOF
license/notices=REQUIRES_NEW_SELECTED_CANDIDATE_PROOF
Native platform/architecture/libc/CPU policy=PROVEN_AND_REUSABLE
clean supported-host evidence=REQUIRES_NEW_SELECTED_CANDIDATE_PROOF
artifact-size measurement=MISSING_BEFORE_FINAL_PUBLICATION
startup-cost measurement=MISSING_BEFORE_FINAL_PUBLICATION
capability/limitation claims=READY_FOR_SELECTED_CANDIDATE
Try Protos/distribution documentation=DOCUMENTATION_AFTER_PUBLICATION
root README shortest first-run path=DOCUMENTATION_AFTER_PUBLICATION
real tag/GitHub Release/assets=POST_PUBLICATION_ONLY
post-publication verification=POST_PUBLICATION_ONLY
```

Artifact-size and startup-cost evidence was not found in the retained DIST005
release evidence. This absence does not block exact baseline selection, but the
measurements must be produced and retained for the exact selected candidate
before final publication authorization because parent #549 explicitly requires
them to be measured and recorded.

## Release claims packet

Proposed capabilities for exact candidate preparation:

- Native is the recommended first-run artifact.
- The validated Native artifact is self-contained with respect to external
  Java/GraalVM, Maven and a Protos source checkout.
- The Native artifact must validate `--version`, direct `-e`, source-file
  execution from an unrelated CWD, Package/Workspace execution, Test Tool,
  REPL, LSP and DAP.
- The Native artifact must retain the optimizing Truffle/guest-compilation
  contract.
- The portable POSIX/JVM artifact is retained from the same candidate as the
  compatibility fallback and external-consumer artifact.
- Both assets must share exact release version/tag/source/baseline/specification
  identity and be covered by one verified multi-asset envelope.

Proposed limitations:

- The release remains a GitHub prerelease of an experimental implementation;
  the Protos/Core v0.1 specification remains draft.
- Native support is exactly Linux x86_64, dynamic glibc, Oracle Linux 10 build
  baseline, declared glibc minimum 2.39 and `-march=compatibility`.
- No Native support is claimed for macOS, Windows, AArch64, musl or earlier
  glibc.
- The portable JVM artifact is not self-contained and requires the declared
  supported external GraalVM/JDK runtime contract.
- Java bytecode release 21 is not a generic Java 21+ compatibility claim.
- No startup/performance improvement may be claimed until selected-candidate
  measurements exist.

## Public identity collision audit

At the D7 investigation point:

```text
PROPOSED_RELEASE_TAG=v0.3.116
TAG_COLLISION=NO
GITHUB_RELEASE_COLLISION=NO
REAL_RELEASE_PUBLISHED=NO
```

The public GitHub state contained no `v0.3.116` tag and no GitHub Release with
that identity.

## DIST007 interaction

`guillermomolina/protos#737` remains a distinct portable-JVM-only release work
item. DIST005-D6 does not replace its publication path.

At D7, DIST007 had not selected an exact baseline/version/tag, so there was no
existing selected-identity collision.

Approval of DIST005 `0.3.116 / v0.3.116` reserves that identity for the
multi-asset DIST005 release. DIST007 must not later select `v0.3.116` as a
separate portable-only release. If DIST007 remains independently useful, its
future exact selection must use a different available `V-SNAPSHOT -> V / vV`
identity.

This sequencing constraint does not merge or close DIST007.

## Exact owner selection

D7 proposed:

```text
PROPOSED_RELEASE_BASELINE_REVISION=1637fe514ee9327241e49964031d981423b16198
PROPOSED_RELEASE_BASELINE_VERSION=0.3.116-SNAPSHOT
PROPOSED_RELEASE_VERSION=0.3.116
PROPOSED_RELEASE_TAG=v0.3.116
PROPOSED_SPECIFICATION_REVISION=0.1.434
```

The project owner explicitly approved that exact packet in the active
interaction on 2026-09-29:

```text
aprobado
```

Therefore the selected release identity is:

```text
DIST005_EXACT_BASELINE_SELECTED=YES
RELEASE_BASELINE_REVISION=1637fe514ee9327241e49964031d981423b16198
RELEASE_BASELINE_VERSION=0.3.116-SNAPSHOT
RELEASE_VERSION=0.3.116
RELEASE_TAG=v0.3.116
SPECIFICATION_REVISION=0.1.434

OWNER_SELECTION_REQUIRED=NO
RELEASE_PUBLICATION_AUTHORIZED=NO
PUBLICATION_AUTHORIZATION_READY=NO
REAL_RELEASE_PUBLISHED=NO
```

Approval authorizes the next bounded candidate-preparation/proof slice. It does
not authorize public tag creation, GitHub Release creation, or asset upload.

## Result

```text
DIST005_D7_STATUS=OWNER_APPROVED

CURRENT_PROTOS_HEAD=1637fe514ee9327241e49964031d981423b16198
CURRENT_PROTOS_VERSION=0.3.116-SNAPSHOT
CURRENT_SPECIFICATION_REVISION=0.1.434

DIST005_SELECTED_MODEL=JVM_PLUS_NATIVE
RECOMMENDED_FIRST_RUN_ARTIFACT=NATIVE
PORTABLE_JVM_ROLE=COMPATIBILITY_FALLBACK_AND_EXTERNAL_CONSUMER_ARTIFACT

PUBLIC_NATIVE_PLATFORM_MATRIX=LINUX_X86_64_GLIBC_DYNAMIC
PUBLIC_NATIVE_BUILD_BASE=ORACLE_LINUX_10
PUBLIC_NATIVE_GLIBC_MIN=2.39
PUBLIC_NATIVE_CPU_ISA_POLICY=COMPATIBILITY

RELEASE_READINESS=YES
RELEASE_BASELINE_REVISION=1637fe514ee9327241e49964031d981423b16198
RELEASE_BASELINE_VERSION=0.3.116-SNAPSHOT
RELEASE_VERSION=0.3.116
RELEASE_TAG=v0.3.116
SPECIFICATION_REVISION=0.1.434

TAG_COLLISION=NO
GITHUB_RELEASE_COLLISION=NO

ARTIFACT_SIZE_EVIDENCE=MISSING_BEFORE_FINAL_PUBLICATION
STARTUP_COST_EVIDENCE=MISSING_BEFORE_FINAL_PUBLICATION
RELEASE_CLAIMS_READY=YES
USER_DOCUMENTATION_STATE=POST_PUBLICATION

DIST007_SELECTED_IDENTITY_CONFLICT=NO
DIST007_V0_3_116_RESERVED_BY_DIST005=YES

PUBLICATION_AUTHORIZATION_READY=NO
REAL_RELEASE_PUBLISHED=NO
```

## Next bounded slice

```text
NEXT_SLICE=DIST005-D8
NEXT_SLICE_NAME=exact selected JVM_PLUS_NATIVE candidate preparation, measurements and pre-publication proof
NEXT_SLICE_WORK_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
```

D8 must consume the exact owner-selected identity above, materialize and prepare
the exact detached candidate using the existing release machinery, repeat the
candidate-dependent Native/portable/envelope proof, produce and retain artifact
size and startup-cost measurements, and stop before public publication
authorization/tag/Release/assets.
