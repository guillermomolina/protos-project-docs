# I115-C — Human Executor (local product repo)

**Scope:** grouped I115-C first: points 1–3 under [PLAT066 wave-2 plan](../PLAT066/PLAT066_WAVE2_FIVE_POINT_CONVERGENCE_2026_10_19.md). **ONE execution mode** is assumed for this task; do not coordinate with mode-migration work. Product repository is `guillermomolina/protos` (two “mo”); agent prepares edits, human runs commands/commits/pushes.

**Baseline:** public source `27cb31d` establishes why we resume, NOT a request to checkout or reset; use current local HEAD and preserve concurrent modifications. Preexisting published I115-A/B are historical; this is C. Do not rerun tests without source changes.

## First editing batch

The [self-contained Python edit block](I115_C_FIRST_EDIT_BATCH_2026_10_19.md) updates `ProtosObjectValue.java` to allow lazily registered `call` selection dependencies specifically for final Closure objects and move existing registered-assumption invalidation before create/assign/remove/compose publication. It is **not the full PIC**. The agent then completes runtime fact admission and DSL PIC integration + regression tests without running them. If any anchor differs from actual HEAD, inspect and adapt; don't blindly apply partial edits.

## Human validation AFTER the complete agent C patch

First inspect the patch and check whitespace (human-executed; no `set`, `pipefail`, `exit`):

```bash
git status --short
git diff --check
git diff --stat
```

Only once C source/tests have been fully edited, use the project test harness and focused tests identified by the agent. Existing `src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/directcall/ProtosI115ARuntimeAdmissionTest.java` and `.../ProtosI115BDeferredCallerTest.java` are regression anchors. A suggested, subject-to-project-conventions focused command:

```bash
mvn -q -Dtest=ProtosI115ARuntimeAdmissionTest,ProtosI115BDeferredCallerTest test
```

Then, because the slice changes shared slot authority/call and Context facts:

```bash
make test
```

If tests fail, human supplies only relevant failure and agent edits the source/tests; do not update release metadata during fixes. When all relevant tests are green and `git diff --check` is clean, update `pom.xml` and `CHANGELOG.md` **once, immediately before commit/push** under project versioning rules. Absolutely NO tests after touching those files. Human alone stages/commits/pushes product repository and reports exact `PROTOS_REVISION` and test outcome.

## Next handoff

After C publication, I115-D compares B5 caller and guest numeric conversion against the included GraalPy facts, keeping exact public values, current caller, Context and root-wide physical cost. Human runs matched PERF042 BGV when D is complete. Parent PLAT066 stays OPEN even when a particular child implementation is green. No automatic issue closure or claim of parity.
