# TEST011 — Suite-native non-local return oracle integrity: historical false greens and correction

- **Work item:** [TEST011 #1023](https://github.com/guillermomolina/protos/issues/1023).
- **Evidence date:** 2026-11-02.
- **Product revision:** [`2e85e8dc0b85548add6962ef6556f24513bc8d05`](https://github.com/guillermomolina/protos/commit/2e85e8dc0b85548add6962ef6556f24513bc8d05), published on `main` with message `TEST011: repair vacuous non-local return conformance assertions`.
- **Provenance:** GitHub read-back verifies the exact product commit and changed paths; all execution results below come from the maintainer's terminal output in the interactive TEST011 session. The evidence recorder has not rerun the executable or build gates.
- **Normative authority:** Existing return-home/Task/ensure semantics in `spec/semantics/CALLABLES.md` and `spec/semantics/EXECUTION_AND_CONTROL.md`; no semantic decision or specification change is claimed.

## Historical defect and falsification

Six pre-correction `suite-native` conformance Tests built closures within a `Test("...", () => { ... })` body, allowing an inner `^` to unwind that Test's captured return home before the intended `Assertions.require(...)` ran. This caused vacuous PASS results without establishing the asserted semantic property. I138 #1021 first identified the risk; TEST011 independently reproduced and corrected it.

The historical test sources were read from the maintainer's local Git `HEAD`, corresponding to the immediately preceding published product baseline `9a7234e4bd0f96293fb864b7249fc2dead770c6c`. The maintainer temporarily replaced six expected conditions while preserving the non-local-return-producing calls (the escaped-closure case used a direct impossible assertion), ran the real Test Tool with only the three affected files, and then restored the corrected working-tree bytes.

**Historical injected run:** `TEST011_HISTORICAL_INJECTIONS=6`, `TEST011_CALLS_PRESERVED=YES`, `12 passed, 0 failed`, process return `0`, `TEST011_HISTORICAL_FALSE_GREEN=CONFIRMED`, `TEST011_RESTORATION=PASS`, and `TEST011_HISTORICAL_REPRO=PASS`. Thus all six historical cases reported PASS under deliberately false expectations. An earlier experimental attempt replaced entire assertions with `Assertions.require(false)` and yielded `7 passed, 5 failed`: it accidentally removed five calls that caused the return and was correctly rejected as an invalid proof for those five. The preserved-call rerun is the accepted falsification.

## Corrected cases and independently observable effects

| Source | Case | Corrected oracle |
| --- | --- | --- |
| `regression/nonlocal-and-escaped-return.protos` | `escaped closure fresh invalid return` | Module-level factory establishes an expired return home; both invocations signal fresh `InvalidReturn` errors whose identities differ. |
| Same source | `non local return from default` | Module-level helper returns `42` through the default operand's `^`; flags prove the function body and post-call continuation are skipped. |
| `call/closure-call-and-return.protos` | `non local return crosses nested call` | Module-level helper returns `42`, counts one inner invocation, and confirms both trailing continuations are skipped. |
| `control/ensure-control-transfer.protos` | `ensure body nonlocal return runs cleanup` | Result `42`, ordered trace `12` (body, cleanup), post-ensure flag remains false. |
| Same source | `ensure cleanup nonlocal return supersedes normal` | Result `43`, ordered trace `12`, post-ensure flag false. |
| Same source | `ensure cleanup nonlocal return supersedes pending return` | Cleanup return `43` supersedes pending body return `42`, ordered trace `12`, post-ensure flag false. |

All six helpers now own a fresh return home at module level rather than inside the Test body. Ordinary execution returns control to the Test's assertions. The untouched foreign-Task/ensure `InvalidReturn` case was separately exercised as a negative control, demonstrating that it was not an additional false-green defect. The remaining non-`^` cases in these sources were retained. Existing `suite-native` manifest registration and `std:test/Assertions` usage remain unchanged.

## Exact published product delta

The [`2e85e8dc0b85548add6962ef6556f24513bc8d05` commit](https://github.com/guillermomolina/protos/commit/2e85e8dc0b85548add6962ef6556f24513bc8d05) changes **only** these three Protos test sources (110 insertions, 64 deletions):

1. [`protos/tests/conformance/regression/nonlocal-and-escaped-return.protos`](https://github.com/guillermomolina/protos/blob/2e85e8dc0b85548add6962ef6556f24513bc8d05/protos/tests/conformance/regression/nonlocal-and-escaped-return.protos).
2. [`protos/tests/conformance/call/closure-call-and-return.protos`](https://github.com/guillermomolina/protos/blob/2e85e8dc0b85548add6962ef6556f24513bc8d05/protos/tests/conformance/call/closure-call-and-return.protos).
3. [`protos/tests/conformance/control/ensure-control-transfer.protos`](https://github.com/guillermomolina/protos/blob/2e85e8dc0b85548add6962ef6556f24513bc8d05/protos/tests/conformance/control/ensure-control-transfer.protos).

**Version/changelog identity:** this product commit does **not** change `pom.xml` or `CHANGELOG.md`. The published Maven implementation version at this revision remains `0.3.439-SNAPSHOT`, inherited from BUG032; it is **not** a new TEST011-specific implementation version. No version increment or TEST011 changelog entry is claimed.

## Human-executed validation

| Gate | Observed result |
| --- | --- |
| Real Test Tool focused corrected cases plus I138 contextual return conformance, `--jobs 1` | `29 passed, 0 failed`; `TEST011_FOCAL_EXIT=0` |
| Seven temporary negative controls against corrected sources (six fixed cases plus foreign-Task case) | Seven expected named `FAIL`, five remaining PASS; `TEST011_NEGATIVE_TOOL_RETURN=1`, `TEST011_NEGATIVE_VERDICT=PASS`, `TEST011_NEGATIVE_CONTROL=PASS` |
| Negative-control restoration | Three corrected source files restored byte for byte; `TEST011_RESTORATION=PASS` |
| Preserved-call historical falsification | `12 passed, 0 failed`; six impossible expectations bypassed; `TEST011_HISTORICAL_REPRO=PASS` |
| Historical-restoration verification | `TEST011_RESTORATION=PASS`; original corrected diff reproduced |
| Integrated `make test` | PASS, maintainer terminal report; `TEST011_FINAL_GATE=PASS` |
| Integrated `make check` | PASS, maintainer terminal report; `TEST011_FINAL_GATE=PASS` |
| `git diff --check` | Clean; only three scoped test files modified before commit |
| Product publication | Exact SHA and three-path diff independently verified through GitHub |

The final `make check` output reported 14 completed synchronous Truffle compilations and one permanent tier-2 bailout (`Code installation failed: code is too large`), classified as `OTHER_PERMANENT_FAILURES=1` rather than generated-BCI failure; `GENERATED_BCI_COMPILATION_FAILURES=0` and the gate reported `PASS`. This diagnostic is retained transparently but is **not** claimed as a TEST011 change or evidence of repaired Truffle compilation. The directory-file-limit guard also reported zero violations.

**Limits of evidence:** complete stdout/stderr for the full integrated suites and an independent rerun by the recorder are unavailable. No additional Native Image, CI workflow, performance, or tagged-release result is asserted. The exact GitHub product commit and its three changed paths were independently inspected.

## Closure cross-reference

```text
WORK_ITEM=TEST011
ISSUE=guillermomolina/protos#1023
PROTOS_REVISION=2e85e8dc0b85548add6962ef6556f24513bc8d05
IMPLEMENTATION_VERSION_AT_REVISION=0.3.439-SNAPSHOT
IMPLEMENTATION_VERSION_BUMP_IN_COMMIT=NO
CHANGELOG_ENTRY_IN_COMMIT=NO
HISTORICAL_FALSE_GREENS=6_CONFIRMED
CORRECTED_FOCAL=29_PASS_0_FAIL
NEGATIVE_CONTROLS=7_EXPECTED_FAILS
INTEGRATED_MAKE_TEST=PASS_HUMAN_REPORTED
INTEGRATED_MAKE_CHECK=PASS_HUMAN_REPORTED
DIFF_CHECK=PASS_HUMAN_REPORTED
SPEC_OR_RUNTIME_CHANGE=NONE
```

Live completion status is controlled by the [GitHub Issue #1023](https://github.com/guillermomolina/protos/issues/1023). This non-normative snapshot must be linked from that Issue after documentation publication with the exact `PROJECT_RECORD_REVISION` for cross-repository verification.
