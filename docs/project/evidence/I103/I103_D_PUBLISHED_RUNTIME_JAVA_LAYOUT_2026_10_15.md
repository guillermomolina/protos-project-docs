# I103-D — Published runtime Java layout (2026-10-15)

**Record type:** exact-revision intermediate implementation publication and maintainer-reported validation; not closure of I103.
**Live work:** [I103 / guillermomolina/protos#897](https://github.com/guillermomolina/protos/issues/897).
**Published product revision:** [`d12d890db64a0a4f7b9c512521774c2c4ecaa283`](https://github.com/guillermomolina/protos/commit/d12d890db64a0a4f7b9c512521774c2c4ecaa283).
**Product version:** `0.3.364-SNAPSHOT` in published `pom.xml` and root `CHANGELOG.md`.
**Parent implementation:** I103, 30 direct tracked regular files per first-party directory.
**Prior slice:** [I103-C publication evidence](I103_C_PUBLISHED_CORE_NETWORK_LAYOUT_2026_10_15.md).

## Published scope and exact root counts

GitHub inspection at the published product commit confirms:

| Java source owner | Direct `.java` before I103-D | Direct `.java` after I103-D | Current state |
| --- | ---: | ---: | --- |
| `src/main/java/com/guillermomolina/protos/runtime/` | 104 | **30** | Meets 30-file root threshold |
| `src/test/java/com/guillermomolina/protos/runtime/` | 79 | **30** | Meets 30-file root threshold |
| `src/main/java/com/guillermomolina/protos/execution/` | 240 | 240 | Still exceeds threshold |
| `src/test/java/com/guillermomolina/protos/execution/` | 447 | 447 | Still exceeds threshold |

The runtime Java trees retain **104 production** and **79 test** source identities in total, now organized under responsibility-specific directories. The root decrease corresponds to 74 production and 49 test files no longer directly in the root, not to deleted tests or features.

The product tree now includes production runtime directories `actors/`, `callables/`, `control/`, `diagnostics/`, `io/`, `lexical/`, `lookup/`, `modules/`, `numeric/`, `process/`, `transfer/` and `values/`; runtime test directories `contexts/`, `control/`, `internal/actors/`, `internal/numeric/`, `io/`, `lookup/`, `process/`, `transfer/` and `values/`. Directory labels describe physical responsibility; **not every directory declares a corresponding Java package**.

## Encapsulation and migration decisions

- Preserve the existing `com.guillermomolina.protos.runtime` Java package for strongly coupled implementation units, including inherited Truffle `@ExportMessage` visibility, package-private methods/constructors and JUnit access to runtime internals.
- Use actual Java subpackages for suitable public APIs and self-contained components; reconcile imports, qualified references and generated Truffle processing as needed.
- `ProtosBytesValue` was restored to the original Java package because inherited `ProtosObjectValue` exported messages were otherwise invisible to the Truffle annotation processor.
- `ProtosSemanticTransferValue`, `ProtosSemanticTransferFamily`, `ProtosSemanticTransferDestination` and `ProtosSemanticTransferRecord` share package-private construction/state contracts with numeric and transfer runtime code; they were restored to the original Java package. The independent `ProtosSemanticTransferPayload` lives in `runtime/transfer/`.
- Fix physical-source paths read by structural tests, including semantic payload, Array, IdentityMap and Future source readers; preserve the underlying checks rather than relaxing tests.
- The intermediate moves ran under local human-executor control and the final source commit was made and pushed by the maintainer. No normative Protos specification, guest semantics or public API change was approved or intended.
- A byte-identical comparison to the *pre-slice* HEAD is **not** a valid criterion for every physical relocation: earlier steps in the same slice legitimately reconciled imports/references. Any strict byte-identity claim must be scoped to the exact individual physical-only move and its pre-move contents.

## Human-reported validation

| Gate | Evidence |
| --- | --- |
| Iterative Java/Truffle compile | `mvn -DskipTests test-compile` succeeded after reconciliation of Truffle export and transfer-package boundaries |
| Runtime test discovery and behavior | A focused run reported **20 tests, 0 failures, 0 errors, 0 skipped** |
| Structural and runtime focal tests | Later focused run reported **24 tests, 0 failures, 0 errors, 0 skipped** |
| Full-suite intermediate failure | Full Java phase initially reported **3,637 tests, 0 failures, 3 errors, 2 skipped**; the three errors were missing old source-file paths referenced by structural tests |
| Full-suite final validation | After updating those three paths, maintainer reported **`make test` PASS** |
| Publication | Maintainer reported clean worktree and successful push `5948ee7c..d12d890d`; exact product commit and Maven version independently confirmed through GitHub |

No agent-run test is claimed. The exact publication commit, runtime direct-file counts and version were verified on GitHub. This evidence does not certify the **whole-repository** 30-file guard or substitute for the I103-H Git-index census.

## Remaining I103 work

Proceed with **I103-E**: execution module/package/workspace and foreign-provider/interop responsibilities, including matching JUnit placement. **I103-F/G** retain further execution responsibilities and strongly coupled Bytecode/lowering/invocation machinery; **I103-H** must prove zero first-party offending directories and enable strict enforcement in `make check` **and** the actual CI pathway. The I103 issue stays **OPEN**; final global acceptance boxes remain unchecked.

[Durable implementation plan](../../work/I103/I103_DIRECTORY_FILE_LIMIT_IMPLEMENTATION_PLAN.md).
