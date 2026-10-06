# TEST009-V3 — prebuilt Graal 25.4 headless BGV analyzer

## Published implementation

```text
WORK_ITEM=TEST009/#795
SLICE=TEST009-V3
REPOSITORY=guillermomolina/protos-benchmarks
PROTOS_BENCHMARKS_REVISION=4aae2a211e376ed965239b87854fe0bb2758cdb0
COMMIT_SUBJECT=TEST009-V3: publish Graal 25.4 headless BGV analyzer

MAINTAINER_REPORTED_ALL_LOCAL_TESTS=PASS
MAINTAINER_REPORTED_GIT_DIFF_CHECK=PASS
PUBLICATION=PUSHED
```

TEST009-V3 implements the external side of the BGV repository boundary selected
after TEST009-V/V2. Protos remains responsible only for capture; this companion
repository owns reusable headless BGV analysis.

## Version-aligned analyzer

The default/current analyzer is now aligned with the Graal runtime used by the
current Protos diagnostic work:

```text
ANALYZER_GRAALVM_VERSION=25.4.4.1.1
GRAAL_REV=95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b
MX_REV=22381992c7322f661498cd6101144f0f49c72ae1

OFFICIAL_IGVUTIL_CLASS=org.graalvm.igvutil.IgvUtility
LOW_LEVEL_COMMANDS=list,filter,flatten
```

The builder image is pinned by tag and digest. The final runtime image contains
only the collected analyzer jars on a neutral Java 21 runtime; the Graal and mx
source checkouts and build cache are not shipped.

Historical Graal 24 tooling remains isolated and unchanged.

## Build/runtime separation

The normal-use contract is:

```text
BUILD_ANALYZER=RARE_PUBLICATION_OPERATION
NORMAL_USE_BUILDS_GRAAL=NO
NORMAL_USE_RUNS_MX=NO
NORMAL_USE_BUILDS_ANALYZER=NO
NORMAL_USE_REQUIRES_PROTOS=NO
NORMAL_USE_REQUIRES_GUI=NO
NORMAL_USE_NETWORK=NO
```

The published wrapper defaults to:

```text
ghcr.io/guillermomolina/protos-benchmarks/igv-analyzer:graal-25.4.4.1.1
```

and exposes explicit `pull`, `inspect`, `list`, `filter`, `flatten`,
`smoke`, and maintainer-only `build` operations.

Normal `inspect/list/filter/flatten/smoke` execution requires an already-present
image and never falls back to cloning Graal, running mx, or building the image.

## Real parser build self-test

The image build includes a real BGV parser self-test using the exact pinned Graal
checkout fixture:

```text
IGV_ANALYZER_CLASS_SMOKE=PASS
IGV_ANALYZER_BUILD_SELFTEST=PASS
FIXTURE=compiler/src/jdk.graal.compiler.test/src/jdk/graal/compiler/graph/test/graphio/parsing/bigv-3.0.bgv
FIXTURE_SHA256=518f0709eb4a7ce430ccb3c0e0ff19d8ae37dffba7ea1e6c27d9364a10016f54
IGV_ANALYZER_SMOKE=PASS
```

The self-test proves the jars shipped in the final runtime can:

- list a real upstream BGV;
- filter a real upstream BGV to non-empty valid JSON.

The fixture is used only during image construction and is not shipped.

## Cheap TEST009 inspection surface

The new `inspect` command accepts:

```text
inspect <file.bgv> [protos-root:<16 lowercase hex>]
```

and intentionally uses only the cheap `IgvUtility list` hierarchy view.

Its machine-readable acceptance fields are:

```text
BGV_READABLE=PASS|FAIL
SELECTED_ROOT_PRESENT=PASS|FAIL|NOT_REQUESTED
AFTER_TRUFFLE_TIER_PRESENT=PASS|FAIL
IGV_INSPECT=PASS|FAIL
```

This is structural acceptance/triage only. It does not perform causal graph
interpretation and does not export a potentially huge CodeTooLarge graph to
JSON merely to answer root/phase presence.

## GHCR publication status

The pushed revision added:

```text
.github/workflows/igv-analyzer-image.yml
```

which is configured to publish:

```text
ghcr.io/guillermomolina/protos-benchmarks/igv-analyzer:graal-25.4.4.1.1
ghcr.io/guillermomolina/protos-benchmarks/igv-analyzer:graal-25.4.4.1.1-<source-sha>
```

At evidence-record time the push-triggered workflow is:

```text
WORKFLOW_RUN_ID=37449321843
WORKFLOW=igv-analyzer-image
HEAD_SHA=4aae2a211e376ed965239b87854fe0bb2758cdb0
STATUS=in_progress
CONCLUSION=<none yet>

PREBUILT_IMAGE_PUBLISHED=NOT_YET_CONFIRMED
```

This is not classified as a V3 implementation defect while the publication run
is still active, but TEST009 must not consume the prebuilt image until the run
completes successfully and the versioned image can be pulled.

No additional implementation slice is authorized merely to wait for this
publication result. If the workflow fails because of an implementation defect,
that defect is repaired inside V3 rather than creating V4.

## Repository boundary

After V2 + V3 the intended architecture is:

```text
guillermomolina/protos
  -> stable single-root CompileOnly capture
  -> fresh non-empty BGV
  -> STOP

BGV
  -> repository interface

guillermomolina/protos-benchmarks
  -> prebuilt headless igvutil
  -> cheap inspect / low-level list/filter/flatten
```

There is no product dependency from Protos back to the benchmark repository.

## Coordination

```text
TEST009_STATE=OPEN_IN_PROGRESS
TEST009_V3_IMPLEMENTATION=PUBLISHED
TEST009_V3_LOCAL_VALIDATION=PASS
TEST009_V3_REAL_UPSTREAM_BGV_BUILD_SMOKE=PASS
TEST009_V3_GHCR_PUBLICATION=PENDING

NEW_FORMAL_ISSUE_REQUIRED=NO
NEW_SUB_ISSUE_REQUIRED=NO
NEW_IMPLEMENTATION_SLICE_REQUIRED_FOR_PUBLICATION_WAIT=NO

NEXT_DIAGNOSTIC=TEST009-W
TEST009_W_START_GATE=PREBUILT_IMAGE_PUBLISHED_AND_PULLABLE
TEST009_W_SCOPE=ONE_CURRENT_CODE_TOO_LARGE_ROOT
```
