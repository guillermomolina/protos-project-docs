# I102-B — Published B5 caller-state conformance and full-suite handoff

**Evidence date:** 2026-10-15.
**Live implementation owner:** [I102/#896](https://github.com/guillermomolina/protos/issues/896).
**Ratified platform authority:** [PLAT060-B5](../../decisions/platform/PLAT060_B5_UNIVERSAL_ORDINARY_CLOSURE_CALL_ABI.md), including the [separately owner-approved published-caller clarification](../PLAT060/PLAT060_B5_PUBLISHED_CALLER_SLOT_APPROVAL_2026_10_15.md).
**Product source publication:** [`guillermomolina/protos@2710b282b3a84be8bfcfee15fbed099cb664e8e4`](https://github.com/guillermomolina/protos/commit/2710b282b3a84be8bfcfee15fbed099cb664e8e4).
**Parent/current candidate baseline:** [`0982d1149e45c68deee13623afea6890b49f166a`](https://github.com/guillermomolina/protos/commit/0982d1149e45c68deee13623afea6890b49f166a) (contains the independently published I103-A and PERF040 changes). The human confirmed local `HEAD` and `origin/main` both equaled this baseline before publication.

## Published delta — verified against GitHub commit

**Commit:** `2710b282b3a84be8bfcfee15fbed099cb664e8e4`; message `I102: validate B5 published caller contract`. GitHub's immutable commit response lists exactly **two modified files**, **157 additions / 1 deletion**:

1. `src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java` — the header's slot-3 code comment now expresses the approved conditional invariant: exact caller in unpublished Closure state; retained exact caller or null in already-published rich-entry state.
2. `src/test/java/com/guillermomolina/protos/execution/ProtosPerf025H1LazyRootActivationTest.java` — introduces 155 lines of test/import/assertion coverage: rich published Call0/Call1 and supplied arguments; exact invocation activation; normal return-home and arity-rejection paths; receiver/home of immediate methods; null slot 3 when no exact dynamic caller exists in a rich entry; retention of the original caller in slot 3 when a compact entry lazily publishes slot 0.

The product's execution implementation is **not changed** in I102-B. The previous I102-A implementation of `publishedInvocation` and compact-to-published state transitions already follows this approved interpretation. The owner explicitly approved the refined state-dependent contract with **“si apruebo”** after the earlier unconditional interpretation proved unable to reconstruct an exact dynamic caller in independently created rich activations. Approval does not imply a second ABI, eager activation/Context/Task or a synthetic caller.

## Maintainer-executed validation

The owner reported, in order:

- An initial focal `mvn -Dtest=ProtosPerf025H1LazyRootActivationTest test` compilation failure due to a missing `ProtosObjectValue` import in the new test. The import was corrected.
- Subsequently: `Tests run: 13, Failures: 0, Errors: 0, Skipped: 0` for that focal class.
- After adding the approved slot-3 state assertions and production contract comment: **`make test` PASS**, the canonical full integrated suite. This is a **maintainer report**, not an independently executed agent test; no raw transcript, full test count, optimizer graph run or test manifest was supplied.
- Prior to publishing, the human showed the actual checkout and remote were synchronized on the baseline `0982d114...`, with exactly the two I102-B Java files modified and `git diff --check` producing no errors. Following the commit/push, the human showed `HEAD=2710b282...` and an empty `git status --short`. GitHub independently confirms the commit exists in `main`.

**No test was requested or claimed after any product version/changelog metadata change.** All reported execution validation precedes this product commit.

## Publication metadata divergence — explicit outstanding item

Unlike the four-file candidate previously proposed, the **actual** `2710b282` commit contains only the two Java files. Immutable GitHub readback of this published HEAD confirms:

- `pom.xml` still declares `0.3.360-SNAPSHOT` (the independently published PERF040 version).
- `CHANGELOG.md` begins with `0.3.360-SNAPSHOT`; there is **no** I102-B entry or `0.3.361-SNAPSHOT` section in the published changelog.
- Therefore `I102_B_SOURCE_AND_TESTS_PUBLISHED=YES`, **but `I102_B_VERSION_AND_CHANGELOG_PUBLICATION=PENDING`**. Do not retroactively claim that the proposed `0.3.361-SNAPSHOT` bump was in `2710b282`, and do not modify product repository contents through the documentation workflow.

The user requested version/changelog changes **only after successful tests and immediately before the relevant commit/push**, and **no tests after touching those files**. A separate human-executed metadata-only completion commit, based on the latest verified product HEAD and preserving concurrent changes, can reconcile this without rerunning unchanged executable tests. If executable dependencies change again, reassess the existing validation against those changes before final publication.

## Acceptance boundaries

- **Approved invariant reconciled:** yes, the explicit PLAT060-B5 addendum is durable and the code comment plus new tests now protect the two publication states.
- **Ordinary ABI:** the five-slot, profile-independent implementation was published by I102-A; B adds no new ABI.
- **Existing regression matrix:** integrated `make test` PASS as reported by maintainer; the B slice adds focused rich direct/method/arity/caller-identity coverage.
- **Measured optimizer/escape or stable Tier-2 speedup:** `NOT_MEASURED`; do not invent a benchmark or a comparative compiler outcome. Historical graph evidence is not a matched benchmark on this revision.
- **I102 Issue closure:** `OPEN` until the publication metadata divergence has been reconciled and the final revision-bound record accepted. PLAT060/#895 remains `CLOSED/RATIFIED`.

## Readback identifiers

```text
I102_B_PRODUCT_COMMIT=2710b282b3a84be8bfcfee15fbed099cb664e8e4
I102_B_PARENT=0982d1149e45c68deee13623afea6890b49f166a
I102_B_PRODUCT_CHANGED_FILES=2
I102_B_MAINTAINER_FOCAL_TESTS=13/13_PASS
I102_B_MAINTAINER_INTEGRATED_SUITE=make_test_PASS
I102_B_GIT_DIFF_CHECK=CLEAN
I102_B_PRODUCT_WORKTREE_AFTER_PUSH=CLEAN
I102_B_PUBLISHED_VERSION=0.3.360-SNAPSHOT
I102_B_PROPOSED_NEXT_VERSION=0.3.361-SNAPSHOT
I102_B_METADATA_PUBLICATION=PENDING
I102_B_GRAAL_FINAL_TIER_GAIN=NOT_MEASURED
```

## Final publication reconciliation — 0.3.361-SNAPSHOT

**Updated 2026-10-15.** This section supersedes **only the pending publication status** in the earlier checkpoint above; that checkpoint correctly describes the earlier Java-only product commit and remains unchanged as historical evidence.

- **Final product revision:** [`dc69fd6269a73e0510e90eb0743514eb63e4630a`](https://github.com/guillermomolina/protos/commit/dc69fd6269a73e0510e90eb0743514eb63e4630a), parent [`2710b282b3a84be8bfcfee15fbed099cb664e8e4`](https://github.com/guillermomolina/protos/commit/2710b282b3a84be8bfcfee15fbed099cb664e8e4). The owner executed `git push origin main`, publishing the metadata-only commit `I102: record B5 conformance version`. GitHub readback verifies the commit is the current `main` head at the closure checkpoint.
- **Exact delta:** only `pom.xml` and `CHANGELOG.md` (17 additions, 1 deletion). The published Maven version is **`0.3.361-SNAPSHOT`**, and the changelog has a dedicated `I102` slice-B section covering the state-specific caller-slot invariant and tests.
- **Validation ordering:** maintainer reported `make test` PASS and `git diff --check` clean for the executable Java patch before the Java commit; the independent metadata-only commit followed, with **no test reruns claimed or requested after metadata edits**. The owner also reported 13 focal tests passing earlier. No optimizer measurement is inferred.
- **No lost concurrency:** the executable commit's parent `0982d1149e45c68deee13623afea6890b49f166a` incorporated I103-A and PERF040; the final commit is a descendant and preserves both. The previously pending metadata discrepancy is now resolved.

```text
I102_B_FINAL_PROTOS_REVISION=dc69fd6269a73e0510e90eb0743514eb63e4630a
I102_B_SOURCE_TEST_REVISION=2710b282b3a84be8bfcfee15fbed099cb664e8e4
I102_B_METADATA_REVISION=dc69fd6269a73e0510e90eb0743514eb63e4630a
I102_B_PUBLISHED_VERSION=0.3.361-SNAPSHOT
I102_B_METADATA_PUBLICATION=PASS
I102_B_FULL_VALIDATION=MAINTAINER_REPORTED_PASS
I102_B_GRAAL_FINAL_TIER_GAIN=NOT_MEASURED
PLAT060_B5_OWNER_APPROVAL=RATIFIED
```

**Closure interpretation:** I102-A's ABI migration and I102-B's approved state-specific invariant, regression coverage and integrated validation are published. This source/metadata publication is sufficient to reconcile the **implementation** closure; it is not a claim of an independently measured Graal speedup or an exhaustive new peer benchmark. [I102/#896](https://github.com/guillermomolina/protos/issues/896) remains the live owner of Issue closure, and [PLAT060/#895](https://github.com/guillermomolina/protos/issues/895) remains the closed architecture authority.
