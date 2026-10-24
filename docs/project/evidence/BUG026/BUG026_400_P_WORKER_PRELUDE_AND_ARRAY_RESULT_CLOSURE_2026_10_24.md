# BUG026 — Parallel worker Prelude scope and Array-result regression: 0.3.400 publication

Date: 2026-10-24. This is non-normative, revision-bound closure evidence for [BUG026 #958](https://github.com/guillermomolina/protos/issues/958), detected while preparing [I122 #956](https://github.com/guillermomolina/protos/issues/956).

## Exact product identity

```text
WORK_ITEM=BUG026
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=2174e88694d7b1c11b3c7132f827f09dc004c398
PRODUCT_COMMIT=BUG026: restore standard Prelude scope in parallel Closure projections (0.3.400)
IMPLEMENTATION_VERSION=0.3.400-SNAPSHOT
PRODUCT_CHANGED_PATH_COUNT=7
SPECIFICATION_CHANGE=NO
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

The exact [published product commit](https://github.com/guillermomolina/protos/commit/2174e88694d7b1c11b3c7132f827f09dc004c398) was inspected on GitHub and verified as `main` at the documentation checkpoint. Its seven changed paths are:

1. `src/main/java/com/guillermomolina/protos/execution/runtime/concurrency/ProtosParallelRuntime.java`
2. `src/test/java/com/guillermomolina/protos/execution/integration/process/ProtosParallelExecutionTest.java`
3. `protos/tests/conformance/regression/parallel-worker-prelude-and-results.protos` (new)
4. `protos/tests/conformance/manifest.tsv`
5. `protos/tests/conformance/library/collections/array-parallel-map.protos`
6. `pom.xml`
7. `CHANGELOG.md`

The Maven version and changelog both identify `0.3.400-SNAPSHOT`. No normative specification text or Standard Library implementation changed.

## Diagnosis and bounded correction

The originally reported symptoms were generic `Error: Object {}` from workers returning `Array(1, n)`, `[]`, or `Integer.recognizes(n)`. The initial suggestion that an empty Array uses no bare-name lookup was incorrect: canonical Array-literal construction resolves `Array`. The earlier suspicion of a generic broken outbound value-copy mechanism was also not established.

The confirmed source defect was in `ProtosParallelRuntime.Transfer.allocateAndCopy`, when projecting a `ProtosClosureValue` into the P domain. The old lexical projection included only `p.newExecutionContext()`, omitting the frozen standard Prelude bindings. A worker could resolve its parameter while bare standard names such as `Array` and `Integer` failed before normal result transfer.

The repair changes the projected context list from:

```java
List.of(p.newExecutionContext())
```

to:

```java
List.of(p.newExecutionContext(), p.bindings())
```

The ordering is deliberate: fresh isolated P execution context innermost, frozen standard Prelude outermost, as required by `spec/concurrency/PARALLEL_EXECUTION.md` §71.15. Caller-local lexical contexts remain excluded. Existing `ProtosArrayValue` graph-transfer logic and the §71.14 transfer contract were not rewritten; generic-error behavior was a symptom, not the selected intervention.

## Published regression coverage

The product commit adds four Java integration tests in `ProtosParallelExecutionTest`:

- `parallelClosureSeesStandardPrelude`: `Integer.recognizes`, `Array(1, 84)`, `[]`, Boolean and String worker results.
- `parallelArrayResultGraphsTransferWithIdentity`: empty/nested Arrays, internal aliases, object slots, copied arguments, caller isolation and self-referential cycles.
- `parallelClosureDoesNotCaptureCallerLocals`: unresolved caller-local name signals `SlotNotFound`; explicitly passed argument succeeds.
- `parallelFailuresAndNonTransferableResults`: transferable `SlotNotFound` failures, `NonParallelValue` for an untransferable Future, successful Array return, and no partially resolved result after failure.

The new manifest-registered `regression/parallel-worker-prelude-and-results.protos` suite provides twelve guest conformance cases for the same lexical/transfer/failure boundaries. The existing `array-parallel-map.protos` suite gains three cases for Array-returning callbacks, empty Arrays, alias identity, non-sharing between invocations, and unchanged caller data. Earlier BUG015/BUG017 tests were not removed.

These published tests exercise the real `Closure.parallel` / Future-observation path and the Standard Library `Array.parallelMap` composition, rather than proving transfer correctness by inspecting only a transfer helper.

## Verification provenance and limitations

**Maintainer report, 2026-10-24:** `git diff --check` clean; all local tests passed; the product commit above was pushed. No final command transcript, full-suite test count, separate Native-image run log, or original pre-fix command stdout/stderr/exit-status capture was supplied in this handoff. Do not infer exact test counts or independent Native execution from a `suite-native` manifest registration.

**Documentation-agent verification:** read the exact GitHub commit, its seven-file patch, the issue and the previously recorded I122 checkpoint, and confirmed the advertised product revision on `main`. The documentation agent did not execute Maven, guest test suites, command-line reproducers, or Native-image validation. The license notice of the new guest suite is included in the published diff; modified source files retain their existing notices.

## Downstream work remains separate

BUG026 repairs the runtime premise needed by the deferred Rosetta Code `Parallel calculations` task, but **does not implement that example**. [I122 #956](https://github.com/guillermomolina/protos/issues/956) was already accepted and closed for eleven examples at [`662dc3d9`](https://github.com/guillermomolina/protos/commit/662dc3d9c4037fb69574b6231e8f0304275816c9). Its historical closure record remains unchanged. The twelfth catalog entry remains `Planned` until its own source and correctness checks are explicitly implemented and validated in a future authorized work item.

The defect's resolution introduces no new language rule or independent architectural decision.
