# I140 — Human executor: one atomic D236-B1 Protos-first implementation

**Type: IMPLEMENTATION, not research.** Product repository: `guillermomolina/protos` (two “mo” sequences), local checkout already at its intended HEAD. Source authority: ratified [D236 #1028](https://github.com/guillermomolina/protos/issues/1028), exact owner approval **“ok apruebo B1”** (2026-11-01), project-docs decision/evidence at SHA `55783cc92ebec20ba63c3b1c0ee027ce0178f325`. **Single one-slice I140 #1029**; no I140-A/B, no external repo reads/web required by the editor.

Paired autonomous editing prompt: [I140_AGENT_ONE_SLICE_PROMPT.md](I140_AGENT_ONE_SLICE_PROMPT.md). Pass that document's **complete contents** to the editing agent. Merely passing a link would violate its no-other-repo/no-web independence; do not ask the agent to open project-docs. The agent reads only local Protos HEAD and edits; this human executor owns all command execution and product publication.

## A. Human preflight (before editor)

Run in the existing intended product checkout; do not switch branches, checkout historical SHA or discard user changes.

```bash
pwd
git status --short
git log -1 --format='%H %s'
git diff --name-only
git diff --cached --name-only
```

Document existing tracked/untracked work and notify the editor of any overlap with planned paths. Ensure no other concurrent patch owns a file. The editor must not execute checkout/pull/fetch, tests, scripts, make/mvn, validation, Git mutations or repository publication; read-only targeted file inspection is okay. All external decisions/reference comparisons are already in the editing prompt.

## B. Inspect exact one-slice delta after agent edits

```bash
git status --short
git diff --check
git diff --stat
git diff -- protos/lib/core/Range.protos spec/semantics/VALUES_AND_COLLECTIONS.md spec/PROTOS_SPEC_CHANGELOG.md docs/guide/05-values-identity-equality-and-collections.md
git diff -- protos/tests/conformance/library/collections/range-core.protos protos/tests/conformance/library/collections/range-prelude-and-module.protos protos/tests/conformance/library/collections/range-callbacks-and-control.protos protos/tests/conformance/actor/range-identity-and-transfer.protos
git diff -- src/test/java/com/guillermomolina/protos/execution/integration/modules/ProtosCoreBootstrapTest.java
```

Check **all other** changed/untracked paths separately; do not infer the patch from the expected list. Any untracked new Protos test/manifest requires explicit inspection and staging. No unrelated files should change.

Specific review gates:

- **Two source-backed own `call` Closures** (exact six local names) on the frozen canonical Core prototypes, no Java host branch; `super.call` correct on normal/equal bounds.
- Each `call` checks original receiver identity **before canonical arity/domain tests**; noncanonical derived objects retain `Object.call` original-receiver/custom `init` behavior and ordinary overrides. Constructor canonical counterpart lookup is independent of Actor-local module rebinding and caller lexical shadow.
- Direction-normalized original bounds and immediate parents on all four nonempty canonical cases; equal bounds retain called orientation. Exactly two semantic Integers; errors and left-to-right effects; new fresh frozen instances with only `start`, `stop` own slots.
- `init` remains strict, `each`/`eachStep` signed semantics, `invert` vs `reverse`, `^`/`ensure`/Error/Future/Actor/P unchanged. Module still has EXACTLY TWO Actor-local exports, Prelude prototypes still frozen/direct Object parents. No D231 `for`, no extra syntax/Iterator/host intrinsics, no undocumented grammar changes.
- Normative `VALUES_AND_COLLECTIONS.md` and documentation agree; `CALLABLES.md` standard `Object.call` unchanged. One new global spec revision derived from current HEAD, no archived spec changelog edits. Existing APL license preserved.

## C. Human-run high-signal validation before expensive suites

Run the narrowest applicable commands selected from **actual local HEAD** `AGENTS.work/IMPLEMENTATION.md`, root `Makefile`, `pom.xml`, corpus registry and existing test-tool entrypoints. Do not invent selectors or turn this into a redesign. A plausible JVM focal test, if the local Maven reactor/test configuration supports it, is:

```bash
mvn -q -Dtest=ProtosCoreBootstrapTest test
```

Then run the current registered Protos conformance invocations for:

- `protos/tests/conformance/library/collections/range-core.protos`
- `protos/tests/conformance/library/collections/range-prelude-and-module.protos`
- `protos/tests/conformance/library/collections/range-callbacks-and-control.protos`
- `protos/tests/conformance/actor/range-identity-and-transfer.protos`
- all directly affected Actor/P and module/bootstrap fixture cases

Use actual local test-tool syntax/corpus listing for these files. First run cheap static/source-style/identity/doc extraction/checks applicable to changed files as directed by the local policy. If the local launcher lacks a focal selection, use the narrowest supported affected suite; report unavailable checks rather than fabricating success.

**Focus especially on** direct/extracted `Range.call` and `InvertedRange.call`, custom derived receiver with custom `init`, wrong-arity/type calls, side effects evaluated once, huge Integers/cross zero, wrong signed steps after normalization, equal-empty, fresh identities, `slotNames` exactly six/instances exactly two, frozen source closures and Core capture, module rebinding, callback unwinds/cancellation, Actor echo and P workers.

Only after focal checks succeed, run the integrated gates as required by the repo's impact policy (these may be expensive; do not launch repeatedly after every minor edit):

```bash
make test
make check
git diff --check
```

If applicable by the changed-file impact matrix, perform Native bootstrap/CLI smoke with existing local instructions and binaries; report `NOT_RUN` rather than assuming parity. If a gate fails, pass the precise failure excerpt back to the **same** editing agent for in-slice repair; rerun impacted focal checks and the final integrated gates on final bytes. No push before reported success.

## D. Version, spec revision and product commit (human-only)

After PASS, derive exact next `pom.xml` version and `CHANGELOG.md` entry from actual current HEAD, never from 2026-11-01 allocation/version guesses; settle the next global `spec/PROTOS_SPEC_CHANGELOG.md` revision from current top entry and `spec/AGENTS.md`. Include version/changelog with source/spec/tests/guide in **the same I140 product commit**. If version changes require regenerating files, use only the prescribed repo workflow and rerun affected validations. Ensure source files have the proper APL Part 5 header.

Before publication, the **human** synchronizes/rechecks current `origin/main` under repository rules and resolves any overlap; do not let the editor `git pull`, fetch or checkout. Check the exact staged paths by **explicit `git add <path>` only**, not `git add -A` or `git add .`. Example planned paths (adjust based on actual reviewed delta):

```bash
git add protos/lib/core/Range.protos
git add spec/semantics/VALUES_AND_COLLECTIONS.md spec/PROTOS_SPEC_CHANGELOG.md
git add docs/guide/05-values-identity-equality-and-collections.md
git add protos/tests/conformance/library/collections/range-core.protos
git add protos/tests/conformance/library/collections/range-prelude-and-module.protos
git add protos/tests/conformance/library/collections/range-callbacks-and-control.protos
git add protos/tests/conformance/actor/range-identity-and-transfer.protos
git add src/test/java/com/guillermomolina/protos/execution/integration/modules/ProtosCoreBootstrapTest.java
git add pom.xml CHANGELOG.md
# Add any other individually reviewed patch-owned files explicitly.
git diff --cached --check
git diff --cached --stat
git diff --cached --name-only
```

Proceed only after staged path audit and final PASS:

```bash
git commit -m "I140: implement guarded bidirectional Range factories"
git push origin main
git status --short
git log -1 --format='%H %s'
```

One line message, no AI trailer/body, no force push, no extra commits/slices. Never claim publication until the exact SHA is observed remotely. Do not close I140 because editor files exist or tests were planned.

## E. Final human result to coordinating assistant

Return final exact Protos product commit SHA; source version and spec revision; patch-owned paths (including any non-obvious new tests); focal Java and registered Protos conformance PASS/FAIL; `make test`, `make check`, `git diff --check` statuses; Native PASS/NOT_RUN; source style/license status; any unrun required checks or remaining blockers. A compact PASS report is sufficient where allowed by `AGENTS.md`.

After **verified human product publication** the coordinator separately creates immutable I140 delivery evidence in `guillermomolina/protos-project-docs`, publishes/verifies that record (authorized direct docs publication), adds closure evidence to GitHub Issue I140 and closes it. The D236 design stays ratified independently; D231 remains open/unselected.
