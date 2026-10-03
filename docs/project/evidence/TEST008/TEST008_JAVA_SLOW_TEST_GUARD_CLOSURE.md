# TEST008 — Java slow-test regression guard closure evidence

Date: 2026-10-03

Status: **CLOSED**

Owning Issue: `guillermomolina/protos#761`

Exact Protos publication:

```text
PROTOS_REVISION=07bf82194556fe0a318b6b751e1100458f768a25
PROTOS_PARENT=383ffc025e82918336e3bc2ab350f1275886a576
COMMIT_SUBJECT=TEST008: preserve Java test phase failure status
```

This record is durable non-normative project evidence. It does not define
Protos language or Standard Library semantics and does not replace live GitHub
coordination.

## Purpose

PERF022 reduced the known pathological Java/Surefire hotspots. TEST008 owns the
durable admission guard that prevents a newly pathological ordinary Java test
from remaining semantically green while silently exceeding the practical
slow-test threshold.

The maintained invariant is:

```text
elapsed <= 10 s
    -> allowed

elapsed > 10 s AND exact test identifier is explicitly allowlisted
    -> allowed, within its explicit allowlist budget

elapsed > 10 s AND exact test identifier is not allowlisted
    -> FAIL
```

The allowlist remains an explicit reviewed exception mechanism. It is not an
automatically refreshed timing baseline.

## Published guard

At the exact product revision above, `make test-java` owns one current-run
Java validation transaction:

1. `tools/java_slow_test_guard.py reset` removes stale
   `target/surefire-reports` evidence and truncates/creates
   `target/test-java.log`.
2. The parallel Java phase runs through `java-test-phase`.
3. The serial Java phase runs through the same `java-test-phase`.
4. `tools/java_slow_test_guard.py check` reads the current Surefire XML
   reports and enforces the strict `> 10 s` policy against
   `tools/java_slow_tests_allowlist.txt`.

The retained log is a generated build artifact, not tracked source.

The checker uses exact fully-qualified Surefire test-class identities. Glob,
prefix and substring allowlist entries are rejected. Missing, empty, malformed
or otherwise unusable timing evidence fails closed.

Allowlisted slow tests additionally carry explicit budgets above the 10-second
threshold, so an existing exception that becomes materially slower still fails
the guard.

## Final TEST008 reconciliation

Before the final product commit, the production `java-test-phase` captured
output with an ad-hoc POSIX shell plus `tee` status file. It avoided a
false-success pipeline result, but its final:

```text
test "$status" -eq 0
```

normalized every non-zero child status to shell status 1. That did not satisfy
TEST008's requirement to preserve the real child Make/Maven failure status
unchanged.

Revision `07bf82194556fe0a318b6b751e1100458f768a25` removes that duplicate
capture path and routes the production phase through the already-existing,
focused-regression-tested guard runner:

```text
java_slow_test_guard.py run --log target/test-java.log -- make ...
```

That runner mirrors combined child output live, appends the same output to the
retained log, and returns `process.wait()` unchanged.

The same publication also clarifies that allowlist entries accepted by one
project-owner baseline decision may share one explanatory group comment. It
does not add or silently regenerate exceptions.

## Focused regression ownership

`tools/java_slow_test_guard.py --self-test` contains focused synthetic evidence
for the guard contract without inserting a genuinely slow ordinary test.

Covered cases include:

- all timings at or below 10 seconds -> PASS;
- exact allowlisted slow class -> PASS;
- unallowlisted class above 10 seconds -> FAIL;
- allowlisted class above its explicit budget -> FAIL;
- exact-identifier enforcement and malformed allowlist rejection;
- missing/empty/malformed Surefire evidence -> fail closed;
- stale report removal before the current run;
- retained stdout/stderr across child execution; and
- propagation of a non-zero child exit status unchanged.

Because the final Makefile now uses the same `run` implementation exercised by
that focused regression, the production capture/status path and the focused
status-propagation evidence no longer diverge.

## Validation provenance

After publishing the exact product revision to `main`, the maintainer reported:

```text
ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

No exact local test counts or complete command transcript were supplied, so this
record does not invent them.

The push also started GitHub Actions CI run `2110`
(`37140303155`) for the exact same product revision. At the time this durable
record was prepared, that remote run was still `in_progress`; its eventual
conclusion is therefore not used as closure evidence here.

## Acceptance reconciliation

At the exact product revision:

```text
LIVE_JAVA_OUTPUT=PASS
RETAINED_PARALLEL_AND_SERIAL_LOG=PASS
CHILD_FAILURE_STATUS_PRESERVED_UNCHANGED=PASS
CURRENT_RUN_TIMING_SCOPE=PASS
THRESHOLD_STRICTLY_GREATER_THAN_10_SECONDS=PASS
VERSION_CONTROLLED_EXACT_ALLOWLIST=PASS
NEW_UNALLOWLISTED_SLOW_TEST_FAILS=PASS
OFFENDER_IDENTIFIER_AND_DURATION_DIAGNOSTICS=PASS
NO_TIMEOUT_SKIP_OR_ALLOW_FAILURE_POLICY=PASS
STALE_REPORT_EXCLUSION=PASS
NO_PROTOS_SEMANTIC_CHANGE=PASS
FOCUSED_GUARD_REGRESSION_COVERAGE=PASS
MAINTAINER_REPORTED_FULL_VALIDATION=PASS
SECOND_TEST008_PRODUCT_SLICE_IDENTIFIED=NO
```

TEST008 is therefore complete at the product revision above. Future performance
work may reduce or remove explicit slow-test exceptions without changing the
TEST008 guard semantics.
