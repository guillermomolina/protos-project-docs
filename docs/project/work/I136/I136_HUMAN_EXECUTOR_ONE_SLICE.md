# I136 — Human executor prompt and one-slice publication contract

**Execution repository:** an existing local checkout of **`guillermomolina/protos`** only, **already at HEAD**. No checkout, pull, fetch, worktree, web browsing, alternate repo, or change to `guillermomolina/protos-project-docs` is needed to run the implementation. **One complete implementation slice**, no staged interim feature publications.

## D232 ratified — implementation gate open

**I136 is READY.** On 2026-11-01 the owner expressly approved both the dual inversions (“ok apruebo esto último”) and the final construction/export, ordered-bound, exactly-two-Integer and frozen-reflectable-instance contract (“Si lo apruebo”). Durable [D232 decision at commit 064485be](https://github.com/guillermomolina/protos-project-docs/blob/064485be40fde8eef0db93f6ce6b8116b360b9a7/docs/project/decisions/language/D232_DIRECTIONAL_RANGE_DUAL_INVERSION_SELECTED.md). The [agent prompt at commit b5fa6e2](https://github.com/guillermomolina/protos-project-docs/blob/b5fa6e2c7d732942df3c8450911682b00832c8f5/docs/project/work/I136/I136_AGENT_ONE_SLICE_PROMPT.md) reproduces the entire exact contract; the editing agent must not consult other repositories or the web.

### Required approved constructor shape

```protos
Ranges: import("std:collections/Range")
Range: Ranges.Range
InvertedRange: Ranges.InvertedRange

a: Range(0, 3)
b: InvertedRange(3, 0)
```

Both are ordinary prototype-created first-class objects. `a.parent() === Range` and `b.parent() === InvertedRange`. Exactly two genuine semantic Integer arguments, no missing or extra values; wrong directional bound order signals `Error`, equal bounds mean empty range. Constructor calls evaluate their receiver and args once left-to-right. Each successful instance is frozen and owns reflectable local `start` and `stop` slots; `reverse()` and `invert()` produce new frozen instances. There is **no additional owner design gate** for this surface.

**Approved semantic quick-check, to use during review:**

```text
Range(0,3)                          0,1,2
InvertedRange(3,0)                  3,2,1
Range(0,3).invert()                 3,2,1
Range(0,3).reverse()                2,1,0
InvertedRange(3,0).reverse()        1,2,3
Range(0,11).eachStep(3, block)      0,3,6,9
Range(0,11).invert().eachStep(-3)  11,8,5,2
Range(0,11).reverse().eachStep(-3) 10,7,4,1
Range.eachStep(negative, block)     Error
InvertedRange.eachStep(positive)    Error
any eachStep(0)                     Error
```

The old `Ranges.each(start,stop,block)` and `Ranges.reverseEach(start,stop,block)` module helpers must disappear, not survive as aliases.

## Run sequence after the editing agent has prepared one cohesive diff

Human action 1: check which working tree, changed files and patch are in front of you. This is read-only and safe for a checkout already at HEAD:

```bash
pwd
git status --short
git diff --check
git diff --stat
git diff -- spec/semantics/VALUES_AND_COLLECTIONS.md protos/lib/collections/Range.protos
```

Human action 2: ask the agent for **the precise current Makefile/test-tool command** to run focused conformance tests. The agent must derive the target from existing scripts/test registry on local HEAD; neither side should guess an unverified `CASE`/CLI flag. Run that focal command, return exact failures and then use the same I136 slice to fix them. Repeat until focal tests pass.

Human action 3: full product regression and consistency checks. These are human-executed and must really PASS before publishing. Depending on current repository Makefile impact policy, include required Native-Image smoke test reported by the agent:

```bash
make test
make check
git diff --check
git status --short
git diff --stat
```

Human action 4: agent updates `pom.xml`/version, `spec/PROTOS_SPEC_CHANGELOG.md` and `CHANGELOG.md` only per current HEAD and project policy. No guessed next version; baseline was `0.3.429-SNAPSHOT`, but another product commit may have changed it. Re-run checks requested by the agent for these final edits.

Human action 5: independently inspect the changed-path inventory and any unrelated working-tree changes. **Explicitly stage only the owned exact paths**, including deleted old helpers/migrated examples, tests, docs, spec, version, changelog. Do not run `git add -A` blindly or implicitly stage user changes. An illustrative restricted-path staging pattern (adapt to the verified diff) is:

```bash
git add -- protos/lib/collections/Range.protos \
  protos/tests/conformance/library/collections \
  protos/examples/algorithms/fizzbuzz.protos \
  protos/examples/algorithms/100-doors.protos \
  protos/examples/closures/value-capture.protos \
  protos/examples/closures/first-class-functions.protos \
  spec/semantics/VALUES_AND_COLLECTIONS.md \
  spec/PROTOS_SPEC_CHANGELOG.md \
  CHANGELOG.md pom.xml
# Also stage only OTHER OWNED, ACTUALLY CHANGED paths shown by reviewed diff.
git diff --cached --check
git diff --cached --stat
git status --short
```

If there are additional changed normative cross-references or docs, stage only those explicit approved paths. The agent must account for test registration and new file paths. If the working tree has concurrent/unrelated edits or origin moves, stop to reconcile rather than cherry-pick/rebase/recheckout automatically.

Human action 6: after checks and review, perform the ONE final product publication, not separate A/B stages:

```bash
git commit -m "I136: replace Range helpers with first-class directional ranges"
git push origin main
git status --short
git log -1 --format='%H %s'
```

This script assumes you are on the correct existing main checkout and that its push is fast-forward. Do not force-push. If Git rejects a push, stop for conflict review; report exact evidence. Confirm the actual immutable SHA, current product version, specification revision and human-executed PASS, not an inferred result.

## Handoff back to coordination

Send the agent the **actual** conformance/Maven/Make/Native results and immutable product commit. The agent may then prepare GitHub issue updates and durable revision-bound evidence, but must not claim publication merely from local diff or the planned command sequence. Ensure:
- D232 public contract is fully owner-approved and durably published; implementation proceeds within the one ready I136 issue.
- The single I136 product commit contains all source-backed prototypes, both transforms, both signed iterators, old-selector removal, migrated callers, tests, spec, docs and version/changelog changes.
- Any resulting performance observation is not advertised as measured unless an actual benchmark is reported; PERF045 is independently responsible for that investigation.
- All updates use **`guillermomolina`** (two `mo`), never any other namespace.

## Required external/reference packet (already provided; no browsing)

The editing-agent prompt includes all details: Python `range` and Swift `stride` derive direction from signed step, Kotlin uses `downTo` and a positive step magnitude, Rust distinguishes `rev` and `step_by`, Ruby has a first-class Range; GraalPy/TruffleRuby demonstrate that language range semantics need not dictate Protos host representations. Current Protos standard `Object.call` / `init`, source-backed modules, ordinary prototype delegation and callback dispatch are authoritative. No external exploration or Java Truffle specialization is necessary for I136.

**Human-executor-only commands, no agent code execution. One atomic published slice.**
