# BUG039 — Safe Test Tool INFRA diagnostic publication (2026-11-21)

**Record type:** Revision-bound, non-normative implementation and validation evidence. The authoritative issue is [BUG039 #1143](https://github.com/guillermomolina/protos/issues/1143). This record does not change Protos semantics.

## Published source identity

- **Product revision:** [`guillermomolina/protos@3b7e9a8fe01008b310d8bc5108b5b571d5a1c970`](https://github.com/guillermomolina/protos/commit/3b7e9a8fe01008b310d8bc5108b5b571d5a1c970).
- **Version:** `0.4.189-SNAPSHOT` (`pom.xml` and `CHANGELOG.md`).
- **Commit:** `BUG039: Show safe transfer, rematerialization and cancellation reasons on Test Tool INFRA lines`.
- **Changed paths:** seven, all in `guillermomolina/protos`: `CHANGELOG.md`, `docs/guide/tools/test-tool.md`, `pom.xml`, `protos/tests/tooling/results-reporting/tool009-logical-case-result.protos`, `protos/tools/test/LogicalCaseResult.protos`, `protos/tools/test/Main.protos`, `src/test/java/com/guillermomolina/protos/execution/tools/test/runtime/ProtosTestLogicalCaseExecutionFacilityTest.java`.
- **No production Java changes** in this publication. The established logical-Case caller completion, observation/transfer machinery and Future protocols are unchanged.

## Implementation and behavioral evidence

The existing Protos-based `LogicalCaseResult` projection now exposes a presentation-only `diagnostic(completion)`. It derives safe reasons from the existing completion phase, state and Error delegation chain, without reclassifying the Case or reading arbitrary host exceptions.

- Failed transfer of an otherwise completed Case outcome with `NonTransferableValue` -> `case outcome transfer failed: NonTransferableValue`.
- `rematerialization-error` -> `case rematerialization failed`.
- Cancelled `case-execution` -> `case execution cancelled`.
- Other host failures with no safely derivable reason -> no reason is invented; the compact `INFRA` line remains.

`Main.protos` appends these reasons, when present, to the existing post-run `INFRA <exact Case reference>` line. Existing completion order, Case identity, summary/progress formatting, `FAIL` versus `INFRA` classification and exit precedence are preserved. An infrastructure error continues to have exit code 3; ordinary guest failures are `FAIL` with exit code 1 unless infrastructure errors take precedence.

The implementation deliberately **does not** print original host `Throwable` messages, stack traces or implementation-specific paths. In particular, an independent host error still becomes a plain guest `Error` in the existing completion boundary; this publication does **not** establish end-to-end retention of its original Java cause. Therefore the achieved diagnostic enhancement is **bounded**, not complete host exception provenance. Transfer admission, mutable object semantics, isolated execution and cancellation ownership remain unchanged.

## Regression evidence

- The Java logical-Case facility test exercises a successful body returning an Array that contains a mutable object from its fresh Context, then verifies the `NonTransferableValue` failed Future, single terminal transition and absence of leftover runnable completions.
- The independent host failure regression verifies plain Error typing and exactly-once Future completion.
- The Protos `tool009-logical-case-result.protos` regression verifies `tool-error` classification, safe diagnostic reasons, no invented reason for ordinary/host failures, and infrastructure exit precedence over guest failure.
- The public Test Tool guide documents the safe `INFRA` reasons and intentionally suppressed host details.

## Validation provenance and limitations

**Maintainer attestation after product push on 2026-11-21:** `git diff --check` is clean; all tests pass locally. This is **owner-reported PASS**, not an independent run or an archived per-command transcript. The exact commands, individual test counts, post-commit CI results and host-cause stack traces were not supplied; no such evidence is inferred.

The original Range test fixture that surfaced BUG039 was corrected independently at [Protos `a5e4d608`](https://github.com/guillermomolina/protos/commit/a5e4d6089a0764d4a55ebc033ac47cb2c7e0ccc2); this later BUG039 change addresses diagnostic visibility and does not alter that fixture or Range behavior.

## Closure interpretation

The publication resolves the user-visible **bare INFRA for a nontransferable logical Case result** by presenting an exact Case reference and safe transfer reason through the authoritative Protos Test Tool reporter. It also covers rematerialization and cancellation reasons where the existing completion establishes them. Independent host failure reason/Throwable provenance remains deliberately unavailable in the public contract; it must not be described as implemented.

**Evidence references:** [BUG039 #1143](https://github.com/guillermomolina/protos/issues/1143), [source commit `3b7e9a8f`](https://github.com/guillermomolina/protos/commit/3b7e9a8fe01008b310d8bc5108b5b571d5a1c970), [Test Tool public guide](https://github.com/guillermomolina/protos/blob/3b7e9a8fe01008b310d8bc5108b5b571d5a1c970/docs/guide/tools/test-tool.md).
