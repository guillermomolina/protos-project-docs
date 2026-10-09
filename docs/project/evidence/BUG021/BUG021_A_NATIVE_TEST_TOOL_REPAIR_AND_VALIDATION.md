# BUG021-A — Native Test Tool progress and discovery-failure repair

Date: 2026-10-09

This non-normative evidence record covers the published BUG021-A product
implementation for [guillermomolina/protos#859](https://github.com/guillermomolina/protos/issues/859).
The Issue remains the live work/closure authority. This record does not create
new language semantics or assert unpublished runtime acceptance.

## Exact identities

```text
WORK_ITEM=BUG021
SLICE=BUG021-A
ISSUE=guillermomolina/protos#859
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=418837272a27f49f6617c7d8891458d773c574af
PRODUCT_COMMIT=BUG021-A: fix Native Test Tool progress and discovery failures
IMPLEMENTATION_VERSION=0.3.317-SNAPSHOT
SPECIFICATION_CHANGE=NO
```

Published revision:
[protos@418837272a27f49f6617c7d8891458d773c574af](https://github.com/guillermomolina/protos/commit/418837272a27f49f6617c7d8891458d773c574af).

## Reported defects and published repairs

**Delayed validation progress.** Before BUG021-A, `dist/validate_native.py`
used `subprocess.run(..., capture_output=True)` in `validate_full_test_tool()`,
so progress emitted by the full Test Tool run was not relayed until the child
exited. The published revision introduces `run_supervised()`, using separate
piped stdout/stderr, selector-driven incremental forwarding to an echo sink,
and retained bounded transcripts. The full Native admission invocation
does **not** impose an arbitrary time limit on a valid long-running suite.
The supervisor exposes exit, signal, timeout, and transcript-limit
termination categories, and closes/reaps the child.

**Discovery failures that could stall termination.** A host exception or
`Error` escaping logical Case discovery could leave the Test Tool root task
non-terminal and make Process shutdown wait indefinitely. The published
`ProtosTestLogicalCaseDiscoveryFacility` catches host failures at the
discovery boundary, emits an actionable stderr diagnostic including
`<corpus>:<source>` and the available cause, and converts the failure to the
existing ordinary Error path. Cancellation is preserved and Protos signals
are rethrown after an attributable diagnostic. `ProtosCli` now passes the
tool diagnostic stream through the asynchronous execution scope.

The diagnostic format is:

```text
Test tool discovery error: <corpus>:<source>: <cause>
```

## Published files

The exact product commit changes eight paths:

```text
CHANGELOG.md
dist/test_validate_native_supervision.py
dist/validate_native.py
pom.xml
src/main/java/com/guillermomolina/protos/cli/ProtosCli.java
src/main/java/com/guillermomolina/protos/cli/ProtosTestToolAsyncExecutionScope.java
src/main/java/com/guillermomolina/protos/execution/ProtosTestLogicalCaseDiscoveryFacility.java
src/test/java/com/guillermomolina/protos/execution/ProtosTestLogicalCaseDiscoveryFacilityTest.java
```

The version advances from `0.3.316-SNAPSHOT` to
`0.3.317-SNAPSHOT`, with a matching root changelog entry.
No `spec/` files or public language semantics are changed.

## Retained regression coverage

The new Python supervision tests exercise:

- live progress reaching the parent *before* the child may complete,
  using a FIFO release handshake instead of a timing-only assertion;
- successful full-Test-Tool admission;
- non-zero child exit with preserved transcript and an attributable
  simulated discovery failure;
- missing summary rejection;
- signal termination identification;
- timeout cleanup and child reaping; and
- transcript-limit cleanup and child reaping.

The new Java discovery tests exercise a simulated host `LinkageError`
with the exact corpus/source and cause in the diagnostic, and a
Protos-signalled discovery failure with source attribution. The expected
execution outcome is failure, not silent success.

These are source-level regression tests. They are not evidence that a
fresh Native Image was rebuilt or that an actual Native distribution
acceptance invocation ran after this commit.

## Validation provenance

The maintainer reports in the 2026-10-09 handoff:

```text
PRODUCT_PUSHED_MAIN=YES
PRODUCT_HEAD=418837272a27f49f6617c7d8891458d773c574af
MAINTAINER_GIT_DIFF_CHECK=CLEAN
MAINTAINER_ALL_LOCAL_TESTS=PASS
```

The coordinator independently inspected the published GitHub commit and
its exact changed paths. No raw local test output, command-level
breakdown, test count, rebuilt Native Image artifact identity, or
post-repair Native distribution validation transcript was supplied.

GitHub returned no pull-request-triggered workflow runs or combined
status checks for this exact product SHA at the time of this record;
remote CI success is not claimed.

The earlier DIST015 Native Test Tool 2581 PASS / 0 FAIL evidence
belongs to a version **before** this BUG021-A repair and must not be
misattributed to the new commit.

## Acceptance and remaining evidence

```text
PRODUCT_FIX_PUBLISHED=YES
SOURCE_REGRESSIONS_PUBLISHED=YES
MAINTAINER_REPORTED_LOCAL_TESTS=PASS
MAINTAINER_REPORTED_DIFF_CHECK=CLEAN
LIVE_PROGRESS_SUPERVISION_IMPLEMENTED=YES
DISCOVERY_ERROR_ATTRIBUTION_IMPLEMENTED=YES
LANGUAGE_SPEC_CHANGED=NO
POST_REPAIR_NATIVE_IMAGE_ADMISSION_EVIDENCE=NOT_REPORTED
NEXT_CODE_SLICE=NONE_KNOWN
```

BUG021-A is implemented and published. A formal conclusion about real
post-repair Native Image acceptance must be based on an exact artifact/run
rather than inferred from local tests or the earlier DIST015 release.
No new release is required by BUG021 itself.

AI assistance: this evidence was prepared from the exact published product
revision, live BUG021 Issue, and the maintainer's reported local validation.
It does not claim that the coordinator executed tests or Native Image builds.
