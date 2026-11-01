# DOC014 — Human executor: one atomic bitwise learning-material delivery

## Task identity, authority and complete scope

**Task:** IMPLEMENTATION (documentation plus executable Protos tutorial/example), **not investigation**.
**Issue:** [DOC014 #1020](https://github.com/guillermomolina/protos/issues/1020).
**Only product editing, testing and publication repository:** `guillermomolina/protos`. The paired [editing-agent prompt](DOC014_AGENT_ONE_SLICE_PROMPT.md) fully specifies all changes. The agent must assume the local checkout is already on its intended HEAD and requires no other repo, online research, Truffle implementation examples, or external decision. All required external precedent is embedded in that prompt.

**Proven published dependency:** I137 [`c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5`](https://github.com/guillermomolina/protos/commit/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5), version `0.3.431-SNAPSHOT`, spec `0.1.486`. This is the verified already-shipped language contract, **not** a forced local checkout or new revision to reserve. D233/I137 are closed. The new work is exclusively to **teach** the already implemented syntax.

One cohesive six-file DOC014 product change, one human-executed final commit/push. No extra slices, issue, parser/runtime work, new library, independent conformance registration, product version bump, or specification revision. Product documentation pages and the two runnable .protos files are non-normative learning content.

## Execution boundary

The editing agent may read/modify only files for the six specified edits. It does **not** run any commands, tests, checker, CLI, package build, Git mutation or web access. **You, the human**, are the exclusive executor of compilation, tests, CLI programs, source-style guard, commit and push in the product repository. Report concise PASS/FAIL results; when failure occurs return failing test name and relevant output only, then have the agent repair the same slice.

The separate `guillermomolina/protos-project-docs` repository owns only durable coordination and prompts; the assistant already publishes those documents directly per the owner's bounded authorization. The editing agent does not need to view or edit that repository.

## 1. Local preflight — do not move HEAD

Run in the *intended existing Protos checkout*:

```bash
pwd
git status --short
git log -1 --format='%H %s'
git diff --name-only
git diff --cached --name-only
```

Preserve uncommitted unrelated work and do not checkout, reset, discard, force-push or blindly pull. The I137 SHA is reference evidence; use actual local HEAD and coordinate any overlapping changes before publication.

## 2. The exactly six owned paths

```text
docs/guide/05-values-identity-equality-and-collections.md
docs/guide/README.md
protos/tutorials/README.md
protos/tutorials/01-values-and-slots/04-bitwise-operators.protos [NEW]
protos/examples/README.md
protos/examples/numbers/bit-flags-and-masks.protos [NEW]
```

Confirm new-file names, full APL Part 5 header, no unintentional changes to existing sources. The existing [`ProtosCliLearningMaterialsTest`](https://github.com/guillermomolina/protos/blob/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5/src/test/java/com/guillermomolina/protos/cli/ProtosCliLearningMaterialsTest.java) auto-discovers **every** `protos/tutorials/**/*.protos` and `protos/examples/**/*.protos` containing `print(` and executes them as a standalone CLI. It checks exit code zero, nonempty stdout and blank stderr. Both new programs additionally fail with `Error().signal()` if a verification is false. No new Java test/class, conforming-source manifest or library corpus plan is necessary.

First review the delta including untracked new files (ordinary `git diff` will not show their content until staged):

```bash
git status --short
git diff --check
git diff -- docs/guide/05-values-identity-equality-and-collections.md docs/guide/README.md protos/tutorials/README.md protos/examples/README.md
```

Read/inspect the complete new `.protos` source files, not just `git diff --stat`; verify the human-readable code against the approved six-message contract.

## 3. Focused validation: one complete existing CLI lane

A light compile/focal JUnit gate is sufficient to detect parse and teaching-source failures before paying for the integrated suite:

```bash
mvn -q -Dtest=ProtosCliLearningMaterialsTest test
```

This class runs all printed cookbook/tutorial sources, not only the new ones. In particular the two new files must have both been discovered as dynamic tests; check their test names when inspecting failures. A no-tests-pass or a skipped new file is **not** acceptable.

If the launcher is built and matches the actual current source tree, additionally run the two new programs directly from the product root:

```bash
bin/protos protos/tutorials/01-values-and-slots/04-bitwise-operators.protos
bin/protos protos/examples/numbers/bit-flags-and-masks.protos
```

Expected tutorial stdout has **six lines**: `8`, `14`, `6`, `-1`, `-2`, `true`. Example stdout has **six `true` lines**. Both have exit code 0 and no stderr. If the launcher is not usable or is stale, rely on the direct JUnit CLI hosting test and report the separate manual command as skipped; never claim an unrun test. Do not add a separate Java runtime/Truffle test merely to accommodate the fixture.

## 4. Full publication gate and static checks

The diff introduces executable tutorial/example sources and the project requires impact-aware validation. For **closing a top-level executable-content DOC work item**, run the complete final `make test` once, plus `make check`, after focal checks pass. Do not repeatedly run the integrated suite after tiny textual adjustments unless changed candidate bytes require it.

```bash
make test
make check
git diff --check
```

`make test` is the canonical Java + native Protos suite; `make check` is the repository's canonical check target. Do not substitute unrestricted `mvn test` for `make test`. If a validator selects a different required profile from `scripts/validation_impact.py`, record it and keep at least the stronger final closure requirement; do not invent impact mappings.

The source-style guard is a **commit-range** checker. Run it against the exact publication commit **after making the single local commit and before pushing**, not with fictitious HEADs or a half-staged diff:

```bash
python3 scripts/source_style_guard.py --base HEAD^ --head HEAD
```

If the guard fails, **do not push**. Report the mismatch for repair/revalidation in this single DOC014 slice.

## 5. Final reviewed staging and one product commit

Ensure the actual patch-owned six files are all included, no unrelated files/staged changes are included, and no extra manifests or product version metadata were changed. This is a documentation/learning content slice, not a change to `src/` production or `protos/lib/`: **do not change `pom.xml`, root `CHANGELOG.md`, `spec/PROTOS_SPEC_CHANGELOG.md` simply to publish it**.

As human executor, reconcile moving `origin/main` appropriately before publication and resolve any path conflicts without rewriting unrelated published work. Revalidate any bytes/dependency state materially changed during reconciliation.

After final PASS, stage only the exact paths:

```bash
git add docs/guide/05-values-identity-equality-and-collections.md
git add docs/guide/README.md
git add protos/tutorials/README.md
git add protos/tutorials/01-values-and-slots/04-bitwise-operators.protos
git add protos/examples/README.md
git add protos/examples/numbers/bit-flags-and-masks.protos
git diff --cached --check
git diff --cached --stat
git diff --cached --name-only
git diff --cached
git commit -m "DOC014: teach fixed bitwise operators in tutorials and examples"
python3 scripts/source_style_guard.py --base HEAD^ --head HEAD
git push origin main
git status --short
git log -1 --format='%H %s'
```

Do not use `git add .`, `git add -A`, an implicit commit over other work, or force-push. Do not push if the guard fails. Verify a final clean state **for the six owned paths**; unrelated pre-existing work may remain.

## 6. Human acceptance and publication report

Report back to the coordinating assistant:

- Exact **product commit SHA**, subject, number of changed paths; current HEAD and whether publication/push succeeded.
- `git diff --check`: PASS/FAIL.
- `ProtosCliLearningMaterialsTest`: PASS/FAIL including both newly discovered programs.
- Direct tutorial/example CLI: PASS/FAIL or honestly SKIPPED, with output check.
- `make test`, `make check`, source-style guard: PASS/FAIL.
- APL notice in both new files; accurate README/guide links; no changes to Core or numerical semantics.
- Any error, source/line mismatch or deliberately skipped check.

After verified push, the assistant (not the editor) records exact product SHA and human test provenance in `guillermomolina/protos-project-docs:docs/project/evidence/DOC014/`, updates [DOC014 #1020](https://github.com/guillermomolina/protos/issues/1020) and closes it **only when** all GITHUB020 publication/closure evidence gates pass. D233 #1016, I137 #1017 and DOC013 #1003 remain closed.
