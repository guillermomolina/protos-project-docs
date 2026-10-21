# I119 — Human Executor: atomic D211 A+B+C product validation/publication

**Date:** 2026-10-21  
**Owning implementation:** [I119/#945](https://github.com/guillermomolina/protos/issues/945).  
**Ratified decision:** [D211](../../decisions/language/D211_CONTEXTUAL_IF_BUNDLE.md).  
**Editor handoff (fully autonomous):** [I119 Agent Editor](I119_AGENT_EDITOR_2026_10_21.md).  
**Product repository (only execution target):** `guillermomolina/protos`. Work from the human's **actual** local HEAD. No fetch/checkout/branch reset is necessary, and the editing agent must not try one.

## Boundaries and publication unit

The human owns every compilation, test, validation, program run, benchmark, Git index/commit/push, version bump publication command and publication verification for the product repository. The editor only edits existing/new files in `protos`. One atomic product commit/release must contain **A+B+C simultaneously**. P1/P2/P3 are editing batches, **never product milestones**. Do not publish a partial standard `.if/`.else` removal or partial contextual syntax.

The user has reported `git diff --check` clean and all local tests passing **before I119 code editing**. Do **not** treat that report as a post-I119 test outcome. Final tests/status must be provided by the human after all substantive edits and before metadata finalization.

## Step 0: inspect the intended patch and preserve unrelated edits

```bash
cd /workspaces/protos
git status --short
git diff --no-ext-diff --check
git diff --no-ext-diff HEAD --     spec/ src/main/java/ src/test/java/ protos/tests/     protos/tools/formatter/ docs/guide/
git ls-files --others --exclude-standard
```

Review the exact I119-owned path set before running patch instructions. The checked P1 Python edit command in the Agent Editor handoff requires its three files to match the observed source anchors. If other changes exist in the same owned paths, reconcile them with the editor instead of overwriting. Avoid `rg`; use `grep` for targeted read-only follow-up.

## Step 1: editor-only mechanical P1/P2/P3 work

The editor is responsible for applying the complete ratified change; the human is **not** asked to run partial tests between P1, P2 and P3. No implementation agent may run a build, `git diff --check` or any test, and no partial product commit is allowed.

P1 removes the old standard selectors in native/structured/root bootstrap. P2 adds exact contextual one/two-arm expression parsing and ordinary `ifTrue`/`ifTrueIfFalse` canonical sends. P3 wires source facts, formatter, static analysis, LSP/DAP source spans, conformance, guide and three normative specification files. Review full patch for both select and fallback paths, including the difference between D210's deferred loop condition and D211's immediate condition.

## Step 2: human validation, after ALL substantive P1+P2+P3 edits

Start with the least costly high-signal gate:

```bash
cd /workspaces/protos
git diff --check
git status --short
git diff --stat
```

Run focal Java/parser/Boolean/structured/tool tests (the editor must adapt this list to the actual changed file dependency closure; do not silently skip new test classes):

```bash
mvn -Dtest=ProtosParserTrailingClosureTest,ProtosParserSourceFactsTest,CanonicalizerCallTest,ProtosStandardBooleanProtocolTest,ProtosI072PhaseEStructuredSendConvergenceTest,ProtosI095BooleanControlInlineTest,ProtosI095BooleanProtocolInlineTest,ProtosI101ProfileSpecializedLoweringTest,ProtosSourceLayoutViewTest,ProtosStaticDefinitionsTest,ProtosStaticReferencesTest,ProtosFormatterToolBootstrapTest test
```

If the local repository's Maven command profile differs, follow `AGENTS.work/IMPLEMENTATION.md` and the existing project Makefile rather than installing anything or inventing a new runtime. Source conformance must cover custom dispatch, grouped lexical `(if)`, same-line `else` and newline negative cases, parser source facts, formatter round-trip, NLR, Error, Future, Task/suspension/cancellation and root slot removal. Run the standard integrated validation required by the actual changed-file impact:

```bash
make test
make check
```

On failure, return **only** the relevant first failing test/diagnostic and modified-path listing to the editor; do not claim green until fixed and final substantive tests truly pass. No speculative performance comparison or Tier1/Tier2 claim is part of I119.

## Step 3: human-confirmed green → final publication metadata

**Only after** the final substantive tests and `git diff --check` pass, the editing agent derives from current local HEAD and updates all three metadata artifacts **once**:

- `pom.xml` to the actual new implementation version;
- `CHANGELOG.md` with the published I119 syntax/standard selector removal compatibility change;
- `spec/PROTOS_SPEC_CHANGELOG.md` with the actual normative revision documenting D211 A+B+C.

Do not guess numeric versions. Follow the existing nearby version/revision convention in those three files. In this project's local finalizer contract, **no further tests run after these final metadata edits**; inspect the exact delta and `git diff --check` only if the human/existing policy permits static diff inspection without reopening the substantive test gate. If metadata changes substantive behavior, rerun the appropriate human gates and reset the finalization sequence.

## Step 4: single atomic human product commit and push

Do not stage everything blindly. After inspecting `git status --short`, construct explicit `git add <I119-owned path>` entries for the complete verified owned set, including tracked and new AST/tests/spec/tools. **Never** use `git add .` or `git add -A`; preserve unrelated local edits. No force-push or history rewriting. Synchronize with current `origin/main` per root publication policy by the human if necessary, recheck relevant concurrent changes, then do:

```bash
git diff --cached --check
git diff --cached --stat
git commit -m "I119: replace standard Boolean if else with contextual syntax"
git push origin HEAD
git rev-parse HEAD
git status --short
```

The exact final product SHA is returned for I119 coordination. **No autonomous product repository commits** by the editing agent or ChatGPT connector are authorized.

## Step 5: revision-bound documentation and issue closure

Only after successful human product publication, update [I119/#945](https://github.com/guillermomolina/protos/issues/945) with:
- exact product SHA, implementation/version and specification revision;
- which specific focal/integrated commands the human ran and their PASS/FAIL outcomes;
- exact set of changed paths and integration/syntax outcomes;
- newly published evidence under `guillermomolina/protos-project-docs:docs/project/evidence/I119/`, if closure requires durable evidence;
- `ISSUE_CLOSURE_COMMENT=PASS`, `CLOSURE_EVIDENCE_IDENTIFIED=PASS`, `DURABLE_RECORD_DECISION=REQUIRED|NOT_REQUIRED`, `PROJECT_RECORD_REVISION=<exact SHA>|NOT_APPLICABLE`, `REQUIRED_DURABLE_PUBLICATION=PASS|NOT_APPLICABLE` according to coordination contract.

D211 design ratification does **not** imply I119 product publication or test success. Keep I119 open until actual product implementation and relevant final evidence are verified. The user authorizes ChatGPT agent content commit/push **only** for `guillermomolina/protos-project-docs` D211/I119 record publication; it does not change the human-only `guillermomolina/protos` boundary.
