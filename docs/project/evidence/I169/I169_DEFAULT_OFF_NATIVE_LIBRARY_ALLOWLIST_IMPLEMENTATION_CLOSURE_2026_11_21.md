# I169 — Default-off host-authorized native FFM library allowlist: implementation and closure evidence

**Date:** 2026-11-21  
**Evidence kind:** immutable product-implementation publication, maintainer-reported validation and formal I169 closure handoff.  
**Implementation issue:** [I169 #1140](https://github.com/guillermomolina/protos/issues/1140), family I.  
**Design authority:** [D259-B2+A #1139](https://github.com/guillermomolina/protos/issues/1139), ratified and already closed.  
**Normative authority:** [specification 0.1.506 at Protos `ee8be92c482cc2451e888d444d74b382b319f4f1`](https://github.com/guillermomolina/protos/commit/ee8be92c482cc2451e888d444d74b382b319f4f1); `FFI.md`, `ACTORS.md` §24K.1 and `PROCESS_IO.md`.  
**Exact published executable revision:** [Protos `c808de27b35137c064b8166ed7cca20e34a6db59`](https://github.com/guillermomolina/protos/commit/c808de27b35137c064b8166ed7cca20e34a6db59), on `main`, commit message `I169: implement default-off host-authorized native library allowlist for the external FFM provider`, committed 2026-11-21 06:32:40 UTC.  
**Implementation version:** `0.4.187-SNAPSHOT`, with matching `pom.xml` and root `CHANGELOG.md` in the same published product commit.

## 1. Publication integrity

The GitHub commit endpoint was read directly for `c808de27b35137c064b8166ed7cca20e34a6db59`; its single parent is the normative publication `ee8be92c482cc2451e888d444d74b382b319f4f1`. The product commit changes **46 paths**: 7 Java production files across CLI/RuntimeHost/package drivers/FFM, tests and fixtures, distribution validation, executable examples, documentation, `pom.xml` and `CHANGELOG.md`. The implementation is committed; this is no longer a normative-only checkpoint.

The new Protos-owned test sources `ffm-allowlist-denied.protos`, `ffm-library-allowlist.protos` and `native/ffi_second.c` retain the project Part 5 APL licence notice, as directly verified at the published revision.

This record **does not rewrite** the [earlier normative-publication handoff](I169_D259_B2_A_NORMATIVE_SPECIFICATION_PUBLICATION_2026_11_21.md), which correctly documented the historical pre-I169 state at `ee8be92c482cc2451e888d444d74b382b319f4f1`.

## 2. Published implementation, by authority boundary

| Boundary | Product evidence at exact revision | Established contract |
| --- | --- | --- |
| Provider-side host policy | [`FfmLibraries.java`](https://github.com/guillermomolina/protos/blob/c808de27b35137c064b8166ed7cca20e34a6db59/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmLibraries.java) | Distributable `protos-ffm` accepts syntactically valid logical aliases but grants none by default; only immutable host `library.<alias>` options supply an exact native load name or absolute path; no implicit `sqlite3`, filename derivation, suffix search or test-fixture fallback |
| Foreign provider and session | [`FfmForeignProvider.java`](https://github.com/guillermomolina/protos/blob/c808de27b35137c064b8166ed7cca20e34a6db59/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmForeignProvider.java), [`FfmSession.java`](https://github.com/guillermomolina/protos/blob/c808de27b35137c064b8166ed7cca20e34a6db59/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmSession.java) | Canonical import syntax remains independent of grants; `authorization.resolve(canonicalTarget)` is before the FFM `Linker.nativeLinker()` and `Arena.ofShared()` for that acquisition; existing session, cache, library lookup and resource-custody paths reused |
| CLI and embedding | [`ProtosCli.java`](https://github.com/guillermomolina/protos/blob/c808de27b35137c064b8166ed7cca20e34a6db59/src/main/java/com/guillermomolina/protos/cli/ProtosCli.java) | Repeatable `--allow-library alias=target` is host-only, checked before entry, routed through the existing immutable `ProtosForeignProviderConfiguration.trustedInProcess(...).withProviderOptions("protos-ffm", ...)`; independent `--foreign-provider-path` registration; no guest `process.args()` pollution |
| Package/workspace applications | [`ProtosPackageRunDriver.java`](https://github.com/guillermomolina/protos/blob/c808de27b35137c064b8166ed7cca20e34a6db59/src/main/java/com/guillermomolina/protos/execution/packages/ProtosPackageRunDriver.java), [`ProtosWorkspaceRunDriver.java`](https://github.com/guillermomolina/protos/blob/c808de27b35137c064b8166ed7cca20e34a6db59/src/main/java/com/guillermomolina/protos/execution/workspace/ProtosWorkspaceRunDriver.java) | Existing RuntimeHost-owner path gets a host factory for workspace-only, external-package and RootActor-service execution; old overloads retain default no-provider behavior; preflight, materialization, execution, termination and scope ownership retained |
| Debug hosting | [`ProtosPolyglotRuntimeHost.java`](https://github.com/guillermomolina/protos/blob/c808de27b35137c064b8166ed7cca20e34a6db59/src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosPolyglotRuntimeHost.java) | Real DAP Engine creation reuses existing external-provider plugin loading with a provider-aware `openDebug` overload, preserving readiness protocol, no implicit discovery and classloader/Engine failure cleanup |
| End-to-end product behavior | [`dist/test_external_foreign_providers.py`](https://github.com/guillermomolina/protos/blob/c808de27b35137c064b8166ed7cca20e34a6db59/dist/test_external_foreign_providers.py), [`ffm-library-allowlist.protos`](https://github.com/guillermomolina/protos/blob/c808de27b35137c064b8166ed7cca20e34a6db59/protos/tests/foreign-provider/ffm-library-allowlist.protos) | Distribution tests separately cover absent provider, default denial, unauthorized and unavailable libraries, malformed CLI, rejection by bundled Tools, two independent C libraries, and native OS-level file write without a Protos guest Filesystem grant |

The **Protos-first boundary** is unchanged: ordinary `std:ffi` descriptors, Closures and guest source implement C calls and SQLite usage; there is no per-library Java wrapper. The native library is deliberately `TRUSTED_IN_PROCESS`; a grant allows the authorized code and its loader/dependencies to exercise OS process authority, not just the independently granted guest `Filesystem`/`Network` APIs. No claim of Landlock, per-library sandboxing, C++ ABI or native-global isolation is made.

## 3. Validation evidence and provenance

Two different types of evidence are available and must not be conflated.

**Directly reviewed earlier maintainer-supplied command transcript** (pre-finalization runtime version `0.4.186-SNAPSHOT`):

- `python3 dist/build_portable.py --allow-dirty` yielded `BUILD SUCCESS`, `DIST_CLASSFILE_MAJOR: 69`, source identity/CRC/layout/runtime metadata checks `PASS` and `DIST_BUILD: PASS`.
- `python3 dist/test_external_foreign_providers.py` against `protos-0.4.186-SNAPSHOT-posix-jvm.zip` yielded **77/77 passed**, including `I169_DISTRIBUTION_BASELINE=PASS`, `I169_PROVIDER_ABSENT=PASS`, `I169_ALLOWLIST_DENIED=PASS`, `I169_UNAUTHORIZED_ALIAS=PASS`, `I169_LIBRARY_UNAVAILABLE=PASS`, `I169_CLI_INVALID_OPTIONS=PASS`, `I169_TOOLS_DENIED=PASS` and `I169_TWO_LIBRARIES=PASS`, alongside SQLite, LIB029/LIB030 and previous external-provider checks.
- These results test the substantial executable I169 candidate **before** the implementation-version/changelog finalization; the archive is **not** represented as an independently rerun `0.4.187-SNAPSHOT` release archive.

**Current final maintainer handoff (2026-11-21, after publishing `c808de27b35137c064b8166ed7cca20e34a6db59`):**

- Maintainer explicitly reports `git diff --check` clean and **all local tests passing** after the I169 implementation was pushed.
- Exact final local test command(s), test counts per suite, complete `make test` log and final-commit dist log were **not supplied with this handoff**. Do not present those command-specific details as directly observed or independently reproduced.
- The GitHub product commit, exact changed paths and implementation version/changelog were independently verified read-only. The coordinating agent ran no local test, Maven build, Git checkout or native validation.

| Closure check | Result | Provenance |
| --- | --- | --- |
| D259-B2+A owner-approved design and normative owner | PASS | Existing ratification and published spec `0.1.506` |
| Implementation content on `main` | **PASS** | GitHub read-only commit `c808de27b35137c064b8166ed7cca20e34a6db59` |
| `pom.xml` / `CHANGELOG.md` finalization | **PASS** | Both published at `0.4.187-SNAPSHOT` |
| Maintainer diff check | **PASS (reported)** | Current maintainer handoff |
| Local tests including affected code | **PASS (reported)** | Current maintainer handoff: “todos los tests pasan en local” |
| Distribution build and compatibility lane | **PASS (observed in supplied transcript)** | Pre-metadata runtime `0.4.186-SNAPSHOT`; `77/77` |
| Actual final `make test` command and raw log | **NOT INDEPENDENTLY DOCUMENTED** | Maintainer affirms all local tests pass; no per-command report |
| Native Image, arbitrary C++ or cross-platform ABI beyond tested lane | **NOT CLAIMED** | Out of approved I169 scope |
| Formal I169 closure | **READY** | Following durable-record publication and final Issue comment |

## 4. Distinct neighboring work and closure outcome

The completed outcome is the explicit library allowlist and its supported entry modes. It is **not** a retroactive proof of general SQLite confinement, the separate durable-storage design, or host-wide prevention of native effects. Previously retired Landlock proposals remain retired. Do not reopen D259 or create a second implementation Issue for the same work.

The remaining coordination transaction is to **re-read this published record by its exact docs commit**, add a compact final comment to [I169 #1140](https://github.com/guillermomolina/protos/issues/1140) identifying both the product and project-record revision plus the evidence provenance, then close I169 as `completed`, with `family:I` preserved and `status:completed`. This is a record of a published outcome, not a request for new implementation.

```text
FORMAL_IDENTIFIER=I169
ISSUE=guillermomolina/protos#1140
PRODUCT_COMMIT=c808de27b35137c064b8166ed7cca20e34a6db59
NORMATIVE_COMMIT=ee8be92c482cc2451e888d444d74b382b319f4f1
SPEC_REVISION=0.1.506
IMPLEMENTATION_VERSION=0.4.187-SNAPSHOT
PRODUCT_PUBLICATION=PASS
PRODUCT_VERSION_CHANGELOG=PASS
MAINTAINER_DIFF_CHECK=PASS_REPORTED
MAINTAINER_LOCAL_TESTS=PASS_REPORTED
DIST_VALIDATION=PASS_77_OF_77_PRE_VERSION_FINALIZATION
DIST_I169_TWO_LIBRARIES=PASS
FULL_SUITE_COMMAND_LOG=NOT_SUPPLIED
DURABLE_RECORD_DECISION=REQUIRED
ISSUE_CLOSURE_COMMENT=PENDING
ISSUE_CLOSURE=READY_AFTER_PUBLICATION
```
