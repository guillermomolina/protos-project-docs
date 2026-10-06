# TEST009-V — single-root Graal diagnostic publication and capture boundary

## Status

```text
WORK_ITEM=TEST009/#795
SLICE=TEST009-V
PROTOS_REVISION=22e3aa9466d172ac816b3a74b4e3ca9db155a92e
PROTOS_VERSION=0.3.236-SNAPSHOT
COMMIT_SUBJECT=TEST009-V: add single-root Graal diagnostic workflow

TEST009_V=PUBLISHED
TEST009_COMPLETE=NO
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
COMPILER_POLICY_CHANGE=NO
PRODUCT_CODE_TOO_LARGE_REPAIR=NO
```

The maintainer published TEST009-V in `guillermomolina/protos`. V replaces the
superseded TEST009-T global correlation model with the upstream-style targeted
workflow: give one semantic root a stable diagnostic identity, isolate it with
`engine.CompileOnly`, request one per-compilation expansion view, and emit a
fresh Graal BGV under `target/`.

## Published capture capability

The published Protos side provides:

```text
STABLE_DIAGNOSTIC_ROOT_SELECTOR=YES
ROOT_DISCOVERY_SURFACE=YES
COMPILE_ONLY_SINGLE_ROOT=YES
METHOD_EXPANSION_SINGLE_VIEW=YES
NODE_EXPANSION_SINGLE_VIEW=YES
GRAAL_DUMP_TRUFFLE=YES
BGV_UNDER_TARGET=YES
T_GLOBAL_CORRELATOR_CURRENT_AUTHORITY=NO
STRICT_SYNC_CHANGED=NO
STRICT_BACKGROUND_CHANGED=NO
```

The diagnostic identity is private/diagnostic-only. Normal root naming remains
unchanged when the diagnostic property is absent. The selected identity is based
on stable root/source-span metadata rather than a run-local object hash.

The root catalog is a lowering inventory. Actual execution/compilation evidence
comes from the single-root compilation trace. A `CodeTooLarge` terminal result
is valid diagnostic evidence when the requested root was isolated and a fresh BGV
was produced; it is not itself a tool-acquisition failure.

## Post-publication architectural correction

Review of the acceptance flow exposed an important repository-boundary issue.
The initial V documentation/tool output still phrases BGV inspection as a manual
IGV step. That is too broad for the Protos repository contract and would couple
product diagnostics to an external analyzer.

The durable architecture is now:

```text
PROTOS_OWNS=CAPTURE
PROTOS_BENCHMARKS_OWNS=COMPILER_ARTIFACT_ANALYSIS
INTERFACE=BGV

PROTOS_MUST_NOT_DEPEND_ON_PROTOS_BENCHMARKS=YES
PROTOS_MUST_NOT_REQUIRE_IGVUTIL=YES
PROTOS_MUST_NOT_REQUIRE_IGV_GUI=YES
```

### Protos responsibility

`guillermomolina/protos` owns only the producer side:

1. catalog stable semantic-root selectors cheaply;
2. isolate exactly one requested root with `engine.CompileOnly`;
3. run the requested Case with deterministic diagnostic compiler settings;
4. retain the compilation trace and at most one expansion-tree view;
5. emit fresh, non-empty `.bgv` files under `target/`;
6. fail closed when the selector is missing/ambiguous, the Case fails, or the
   requested capture evidence is missing.

No BGV structural interpretation is required to validate this capture boundary.

### External analyzer responsibility

`guillermomolina/protos-benchmarks` already owns the reusable headless Graal
artifact-analysis pattern through its pinned `igv-analyzer` container and
Oracle Graal `org.graalvm.igvutil.IgvUtility` surface
(`list` / `filter` / `flatten`).

That analyzer may consume BGV files emitted by Protos, but Protos must not invoke,
vendor, build, or depend on it.

The analyzer image should be aligned with the Graal version being diagnosed and,
for normal use, should be prebuilt/pinned: analyzing an existing BGV must not
require building Protos, Maven compilation, an `mx build`, network access, or a
graphical desktop.

## Workflow correction

A broad "probe all compilations, then discover which root succeeded" run was
observed during V acceptance work. That is explicitly not the normal workflow.

The intended path remains:

```text
cheap root catalog
  -> choose one source-attributed selector
  -> compile only that selector
  -> retain trace + expansion + BGV
  -> STOP at the Protos repository boundary
  -> external analyzer consumes BGV later
```

The expensive no-`CompileOnly` probe is not a required smoke/gate and must not
become part of ordinary TEST009 operation.

## Required Protos follow-up before external analyzer work

A bounded Protos implementation correction is required to make the capture
boundary explicit and self-contained:

- remove user-facing requirements/instructions that make opening/parsing IGV a
  Protos acceptance condition;
- make successful Protos capture end at fresh BGV production plus local trace /
  expansion evidence;
- keep BGV structural interpretation outside `make test`, `make check`, and
  the Protos diagnostic result classification;
- ensure the normal documented workflow does not use the expensive no-CompileOnly
  probe;
- preserve all V stable-selector / single-root functionality and the strict
  SYNC/BACKGROUND gate unchanged.

This is a focused correction to the published V diagnostic boundary, not a
CodeTooLarge repair and not a new compiler-policy experiment.

## Coordination

```text
NEW_FORMAL_ISSUE_REQUIRED=NO
NEW_SUB_ISSUE_REQUIRED=NO
TEST009_STATE=OPEN_IN_PROGRESS

NEXT_STEP=PROTOS_CAPTURE_BOUNDARY_CLEANUP
NEXT_STEP_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos
SOURCE_REPAIR_AUTHORIZED=NO
CODE_TOO_LARGE_REPAIR_AUTHORIZED=NO
TRIAL_AND_ERROR_ALLOWED=NO

FOLLOWING_REPOSITORY=guillermomolina/protos-benchmarks
FOLLOWING_SCOPE=PREBUILT_VERSION_ALIGNED_HEADLESS_IGVUTIL_ANALYZER
```

Only after the Protos capture boundary is clean should the companion
`protos-benchmarks` analyzer be updated/aligned. Subsequent TEST009 diagnostic
work should then require two conceptual operations only: capture one BGV in
Protos, analyze that existing BGV in the benchmark/tooling repository.
