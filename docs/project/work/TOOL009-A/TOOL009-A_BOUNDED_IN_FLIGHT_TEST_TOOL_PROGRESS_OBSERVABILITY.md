# TOOL009-A — Bounded in-flight Test Tool progress observability

## Identity

FORMAL_WORK_ITEM=TOOL009-A
GITHUB_ISSUE=#684
PARENT=TOOL009/#600
PRODUCT_REVISION=7632b9b7511123308ca6b742e3cdc8bc35ca8459
PRODUCT_VERSION=0.3.61-SNAPSHOT
PRODUCT_COMMIT=TOOL009-A/D174/D176: bounded in-flight Test Tool progress observability

## Purpose

Implement the already-ratified Test Tool observability boundary and stalled-Case presentation policy needed for directory-scoped test execution.

This slice is intentionally independent of the broader TOOL009 migration. It establishes bounded progress and diagnostic state so a maintainer can identify an in-flight or hung logical Case without requiring per-Case normal terminal output or a full-suite run.

## Governing authority

- D174 / #673 — Candidate C′: presentation-neutral internal CaseStarted/CaseTerminal lifecycle observations feeding separate reporting/progress machinery.
- D176 / #674 — Candidate G: one-shot stalled in-flight diagnostic after 30 seconds without a CaseTerminal while work remains in flight; at most 8 Case display references plus omitted count; re-arm only after later terminal progress.
- D120 / #455 — compact deterministic progress remains authoritative.
- TOOL009 / #600 — parent implementation work and broader migration owner.
- TOOL008 / #591 — existing --file FILE focal-selection surface remains unchanged.

No normative language or Standard Library semantics were changed by this slice.

## Implementation summary

Progress.protos now maintains an invocation-local lifecycle tracker containing in-flight Case references, invocation-local numeric tokens that are never reused, and the optional Test-Tool-private observation sink.

The tracker forwards presentation-neutral lifecycle facts only. It does not format diagnostics, schedule watchdog work, write terminal output, or infer Case identity itself.

Main.protos attaches one invocation-wide lifecycle tracker and supplies separate display renderers for the incumbent file-backed path and the suite-native logical-Case path. Both schedulers report through the same tracker while remaining presentation-neutral.

Two Test-Tool-private host facilities provide D176 reporting and watchdog behavior: ProtosTestToolStalledCaseReporter owns the in-flight state machine and bounded snapshot policy; ProtosTestToolStalledCaseDiagnosticFacility provides the host-side elapsed-time watchdog integration.

The watchdog triggers once after 30 seconds without any CaseTerminal while work remains in flight. It emits at most 8 Case display references plus a collapsed +N more, then re-arms only after later terminal progress. When no Cases remain in flight, the diagnostic is disarmed.

The diagnostic is advisory only. It never fails, cancels, times out, classifies, retries, or reschedules a Case and does not change Test Tool result or exit-code semantics.

D176 keeps elapsed-time measurement host-side and Test-Tool-private. No Clock or Timer capability is exposed to Protos guest code.

## Compatibility and invariants

- Normal compact D120 aggregate progress remains unchanged.
- No mandatory per-Case terminal write is introduced.
- Existing legacy and suite-native execution paths remain supported.
- The Core native-provider boundary is unchanged.
- The new Test-Tool-native closure is recorded in the audited non-Core native-provider inventory with an exact count of one.
- Core native-boundary growth tripwires remain unchanged.
- No public event protocol, JSON stream, JUnit report, telemetry/history, timeout policy, retry policy, or terminal UI framework is introduced.

## Validation

FOCUSED_TESTS=23
FOCUSED_TESTS_PASS=23
FOCUSED_TESTS_FAIL=0
FULL_VALIDATION=make test
FULL_VALIDATION_RESULT=PASS
DIFF_CHECK=PASS
SOURCE_STYLE_GUARD=PASS
LEGACY_EXECUTION_GUARD=PASS
CHANGED_PATHS=13
SPECIFICATION_FILES_CHANGED=0
PRODUCT_VERSION=0.3.61-SNAPSHOT

The final candidate was fully validated after the native-boundary inventory correction, including the complete Java/JUnit suite and native Protos corpus.

## Publication

PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=7632b9b7511123308ca6b742e3cdc8bc35ca8459
PRODUCT_REVISION_PUBLISHED=YES

The product commit contains the complete bounded implementation, tests, version/changelog metadata, and the corresponding non-Core native-provider inventory correction.

## Closure

TOOL009-A / #684 was closed after publication and full validation.
The parent TOOL009 / #600 remains open for the broader local-first Test Tool migration and test-corpus conversion.

## Known follow-ups

- RepositoryCorpusPlans.protos and RepositorySuite.protos retain shortened license notices from before this slice; those files were outside the owned path set and were intentionally not changed here.
- D176 rendering wording remains replaceable presentation detail; it is not a new public protocol.
- The broader TOOL009 migration remains the next owner for consuming this observability machinery during directory-scoped test migration.