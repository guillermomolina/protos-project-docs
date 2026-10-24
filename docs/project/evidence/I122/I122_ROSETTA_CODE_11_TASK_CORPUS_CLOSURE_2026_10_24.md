# I122 — Rosetta Code catalog and eleven verified task examples

**Formal work item:** [I122 #956](https://github.com/guillermomolina/protos/issues/956).
**Exact product commit:** [`662dc3d9c4037fb69574b6231e8f0304275816c9`](https://github.com/guillermomolina/protos/commit/662dc3d9c4037fb69574b6231e8f0304275816c9).
**Product repository:** `guillermomolina/protos`.
**Commit subject:** `I122: add Rosetta Code catalog and eleven canonical examples`.
**Maven version at that commit:** `0.3.399-SNAPSHOT` (read from `pom.xml` at the exact commit).
**Checkpoint date:** 2026-10-24.
**Scope accepted by the maintainer:** eleven executable Rosetta Code tasks; the twelfth, Parallel calculations, remains intentionally planned pending [BUG026 #958](https://github.com/guillermomolina/protos/issues/958).
**Record authority:** non-normative implementation/verification evidence. The external Rosetta Code wiki was not modified.

## Product evidence

The verified I122 product commit changes **13 paths**:

- New task catalog: `protos/examples/ROSETTA_CODE.md` (12 task mappings: 11 source links, one planned path).
- Updated cookbook index: `protos/examples/README.md`.
- Eight new Protos source examples: `algorithms/100-doors.protos`, `algorithms/binary-search.protos`, `algorithms/bubble-sort.protos`, `algorithms/fizzbuzz.protos`, `algorithms/greatest-common-divisor.protos`, `basics/rosetta-hello-world-text.protos`, `closures/first-class-functions.protos`, `closures/value-capture.protos`, plus `objects/rosetta-delegates.protos` (nine new examples in total).
- Extended existing source: `protos/examples/algorithms/fibonacci-recursive.protos` with an iterative Fibonacci variant.
- Added CLI correctness suite: `src/test/java/com/guillermomolina/protos/cli/ProtosCliRosettaCodeExamplesTest.java`, eleven JUnit task methods with direct CLI file execution, additional expressions against the real source where applicable, exact stdout, successful exit status and blank stderr.

The eleven covered Rosetta Code tasks are:

1. Hello world/Text
2. Factorial (existing canonical source)
3. Fibonacci sequence (recursive and iterative)
4. FizzBuzz
5. Greatest common divisor
6. 100 doors
7. Binary search
8. Sorting algorithms/Bubble sort
9. First-class functions
10. Closures/Value capture
11. Delegates

The executable source remains in the existing topical `protos/examples/` tree. This checkpoint makes no Core language, runtime, Standard Library or normative specification change. Each new Protos-owned source has the project APL-1.0 header.

## Validation provenance

**Maintainer report (2026-10-24):**

- `git diff --check`: clean.
- All local tests: **PASS**.
- Product commit: **pushed**.

Earlier in the same I122 work, the maintainer explicitly showed `mvn test -Dtest='ProtosCliRosettaCodeExamplesTest,ProtosCliLearningMaterialsTest'` with `Tests run: 67, Failures: 0, Errors: 0, Skipped: 0` for the first task set. Do **not** attribute that 67-test count to the final 11-task commit; the maintainer's final confirmation was a summary that all local tests passed, not a supplied final test log.

The documentation agent checked the exact GitHub product commit and source/test file delta and read `pom.xml` at that SHA. It did not independently execute Maven, the CLI, `make test`, or Native validation.

## Intentionally deferred task and issue boundary

The twelfth indexed Rosetta Code task, `Parallel calculations`, is **Planned**, not a completed task, in the shipped `ROSETTA_CODE.md`. Its proposed `primeFactors` worker needs to return an Array over the isolated P boundary. Maintainer-observed failure to transfer such worker results is independently tracked as [BUG026 #958](https://github.com/guillermomolina/protos/issues/958), against `spec/concurrency/PARALLEL_EXECUTION.md` §71.14.

The I122 corpus does not ship a scalar-only workaround, the affected sample, or a test that depends on the broken Array-result transfer. BUG026 remains open and independently actionable. The maintainer explicitly requested closure of I122 for the **eleven shipped examples**, thereby narrowing the initial twelve-task implementation target for this checkpoint; the deferred task can be revisited after BUG026 is resolved under an appropriate future work item.

## Revision and catalog limitations

The exact verification/publication revision is bound **here** to `662dc3d9c4037fb69574b6231e8f0304275816c9` as accepted by the maintainer. At this commit the example catalog still uses `Pending validation` and `—` in the per-row verification-revision fields for the eleven tasks. Do not misrepresent those catalog cells as filled or independently validated; updating them would require a subsequent product-repository change by the human executor.

No external Rosetta Code contribution, language registration, license approval, release, or tag is claimed. The purpose of this record is the internal task corpus and its exact development evidence.
