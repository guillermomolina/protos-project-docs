# LIB022-B — `std:inspect` host-authorized diagnostics: implementation and acceptance (2026-10-18)

**Owner:** [LIB022 / #921](https://github.com/guillermomolina/protos/issues/921), second implementation slice **LIB022-B**; the first slice was [LIB022-A / #923](https://github.com/guillermomolina/protos/issues/923).
**Approved decision:** [D205 / #922](https://github.com/guillermomolina/protos/issues/922), Candidate B / exact B2+ ratified in [D205 durable decision](https://github.com/guillermomolina/protos-project-docs/blob/e76072ee54eb3de6d7d6e7818a134ca58a4caa7c/docs/project/decisions/language/D205_LIB022_RUNTIME_INTROSPECTION_B2_PLUS_PUBLIC_CONTRACT.md).
**Product revision:** [`88f7c84e54f424e3e9c5aa0be22109e85d00baf7`](https://github.com/guillermomolina/protos/commit/88f7c84e54f424e3e9c5aa0be22109e85d00baf7) on `main`.
**Published implementation version:** `0.3.379-SNAPSHOT` (commit message `0.3.379`); **normative specification revision:** `0.1.462`.
**Status:** LIB022-B published and maintainer-validated. Together with LIB022-A, the entire owner-approved LIB022/D205 B2+ scope is complete. This record is evidence, not a normative language definition or benchmark.

## Verified published change

The exact product commit has message **`LIB022-B: implement std:inspect host-authorized runtime inspection (0.3.379)`**. The published `pom.xml` specifies `0.3.379-SNAPSHOT`; the product `CHANGELOG.md` records this implementation and the local validation gates reported by the maintainer. The normative owner [`spec/semantics/MODULES.md` §73A](https://github.com/guillermomolina/protos/blob/88f7c84e54f424e3e9c5aa0be22109e85d00baf7/spec/semantics/MODULES.md) and [specification changelog](https://github.com/guillermomolina/protos/blob/88f7c84e54f424e3e9c5aa0be22109e85d00baf7/spec/PROTOS_SPEC_CHANGELOG.md) record `0.1.462`.

- [`protos/lib/inspect.protos`](https://github.com/guillermomolina/protos/blob/88f7c84e54f424e3e9c5aa0be22109e85d00baf7/protos/lib/inspect.protos) exports exactly four source-backed operations: `representation/1`, `callableInfo/1`, `bytecode/1`, and `truffleTree/1`. The bootstrap facility is privately captured and its module slot removed.
- [`ProtosInspectFacility.java`](https://github.com/guillermomolina/protos/blob/88f7c84e54f424e3e9c5aa0be22109e85d00baf7/src/main/java/com/guillermomolina/protos/execution/facilities/ProtosInspectFacility.java) implements the private frozen native operations and the `protos.Inspect` Context-level authorization check. Authorization is default-denied, independent of DEBUG, tested on import **before** an Actor-local module cache lookup and rechecked for every operation.
- Source-backed physical diagnostics use the entered Context's existing `bytecodeExecutionPlanForEnteredClosure` projection cache and admit only semantic activation roots; the inspected Closure is never invoked. Output is a detached, non-empty diagnostic String (unstable format), with Bytecode constants omitted, compact Truffle tree printing, and a 64-Ki-character bound including truncation marker. Native/unsupported physical representations and failed preparation/rematerialization use fresh ordinary Errors; diagnostic availability need not match between JVM and Native Image.
- The new module is registered as a **lazy** standard member in `ProtosCoreBootstrap`, with no eager global facility creation or compulsory inspection registry/instrumentation in programs that do not import it. This is a structural PAYG argument, **not** a measured performance claim.
- [`docs/guide/library/inspect.md`](https://github.com/guillermomolina/protos/blob/88f7c84e54f424e3e9c5aa0be22109e85d00baf7/docs/guide/library/inspect.md) documents the host option, examples, limits, and JVM/Native compatibility boundary.

## Published validation assets and provenance

- Java tests `ProtosInspectFacilityTest`, `ProtosInspectEmbeddingTest`, and `ProtosLazyStandardModuleMembersTest` cover permitted/denied Contexts, Actor-local caching, four public operations, arity/type/fresh Error behavior, non-execution, Context-specific projection reuse, failed rematerialization (including `ProtosSignalException` conversion), output bounds, and facility initialization.
- [`protos/tests/conformance/library/inspect/authorization.protos`](https://github.com/guillermomolina/protos/blob/88f7c84e54f424e3e9c5aa0be22109e85d00baf7/protos/tests/conformance/library/inspect/authorization.protos) verifies default-denied import and repeat rejection. The source is registered in the conformance manifest and `RepositorySuite.protos` TOOL011 progress grouping, so the suite includes it rather than merely storing an unexecuted file.
- The core-native-boundary architecture check accounts for four new non-Core native operations. The lazy-member test is relocated under `runtime/contexts/` in the published commit to respect directory-file limits.
- **Maintainer report, 2026-10-18:** `git diff --check` clean and all local tests PASS; the change was pushed. Published `CHANGELOG.md` records `make test`, `make check`, `make test-protos`, and `make -C build/native test` as PASS. These results are maintainer-reported; this documentation record does **not** claim independent execution, a measured runtime speedup, or separate captured Native Image transcripts.

## B2+ completion and explicit scope boundary

The complete approved public surface consists of **five operations across two libraries**:

1. **LIB022-A:** `std:runtime.info/0`, published in [`74c4d2b41102a3dfed5e382e4b62f4b2df7fa427`](https://github.com/guillermomolina/protos/commit/74c4d2b41102a3dfed5e382e4b62f4b2df7fa427), `0.3.378-SNAPSHOT`, spec `0.1.461`. See the [first-slice acceptance record](https://github.com/guillermomolina/protos-project-docs/blob/4fd2c654fae6997da5df25b57056d467e2be3830/docs/project/evidence/LIB022/LIB022_A_378_RUNTIME_METADATA_FINAL_ACCEPTANCE_2026_10_18.md).
2. **LIB022-B:** `std:inspect.representation/1`, `callableInfo/1`, `bytecode/1`, `truffleTree/1`, published in `88f7c84e54f424e3e9c5aa0be22109e85d00baf7`, `0.3.379-SNAPSHOT`, spec `0.1.462`.

There are **no further LIB022 implementation slices within approved D205/B2+**. Compiler IR/JIT state predicates, heap dumps, privileged diagnostics, structured stable inspection formats, and implicit debug/interop permissions were explicitly not authorized for this library. Any future enlargement would require separately scoped work and the applicable owner decision, not an automatic LIB022-C.

Known bounded limitations: the helper output bound is tested without constructing a genuinely enormous physical Bytecode/root tree; the non-semantic auxiliary-root rejection is guarded in production but not directly unit-tested without changing the interpreter API. These were recorded and accepted as non-blocking during LIB022-B implementation review. The compact tree renderer may construct its entire in-memory text before output truncation; explicit diagnostic requests may be expensive.

```text
WORK_ITEM=LIB022-B
FAMILY=LIB
PARENT=LIB022/#921
D205_CONTRACT=OWNER_APPROVED_B2_PLUS
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=88f7c84e54f424e3e9c5aa0be22109e85d00baf7
IMPLEMENTATION_VERSION=0.3.379-SNAPSHOT
SPEC_REVISION=0.1.462
PRODUCT_PUBLICATION=VERIFIED
GIT_DIFF_CHECK=MAINTAINER_REPORTED_PASS
MAKE_TEST=MAINTAINER_REPORTED_PASS
MAKE_CHECK=MAINTAINER_REPORTED_PASS
MAKE_TEST_PROTOS=MAINTAINER_CHANGELOG_REPORTED_PASS
NATIVE_TEST=MAINTAINER_CHANGELOG_REPORTED_PASS
IMPLEMENTATION_STATUS=COMPLETE
LIB022_A=COMPLETE
LIB022_B=COMPLETE
PARENT_LIB022=READY_TO_CLOSE
FURTHER_APPROVED_SLICES=NONE
```
