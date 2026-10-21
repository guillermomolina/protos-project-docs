# CLI010 — standalone `--inspect` publication and validation evidence

**Evidence date:** 2026-10-21.  
**Formal work:** [CLI010 / #938](https://github.com/guillermomolina/protos/issues/938).  
**Product revision:** [`guillermomolina/protos@e03e2c82ad1d2f4090e8e5baae631ad9790a09c6`](https://github.com/guillermomolina/protos/commit/e03e2c82ad1d2f4090e8e5baae631ad9790a09c6).  
**Implementation version:** `0.3.391-SNAPSHOT`.  
**Evidence class:** GitHub-verified product commit, published changelog, and maintainer-reported tests and diff check. The coordinating agent did not execute builds, test suites, or a Native executable.

## Published scope

CLI010 exposes the already-implemented D205 `std:inspect` facility from the standalone CLI through an explicit opt-in. The supported launcher forms are:

```console
protos --inspect show.protos
protos --inspect -e 'Inspect: import("std:inspect")'
protos --inspect
```

- `--inspect` sets `protos.Inspect=true` for the new invocation's Context, before guest execution/import. Without opt-in, `std:inspect` import still signals an ordinary Error.
- The single-use prefix flag can appear before `--foreign-provider-path` or after it. A duplicate flag or use before `run`, `debug`, `test`, `package`, `format`, `lint`, language-server, or help/version commands is a usage error (exit 2).
- The file/`-e` application arguments remain intact; a trailing argument spelled `--inspect` does not grant authorization.
- The host retains immutable inspection authorization while the Context builder sets `protos.Inspect` only when explicitly authorized. The default Context option is left unset, preserving independently configured embedder Engine options.
- The existing D205 API is unchanged: `representation/1`, `callableInfo/1`, `bytecode/1`, `truffleTree/1`. No changes to the guest's Standard Library contract, DAP, execution profiles, Bytecode lowering, compilation policy, or unrelated authorities were published. Inspection remains on demand (PAYG).

**Product paths published:** `src/main/java/com/guillermomolina/protos/cli/ProtosCli.java`, `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosPolyglotRuntimeHost.java`, `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosPolyglotExecutionContext.java`, `src/test/java/com/guillermomolina/protos/cli/ProtosCli010InspectTest.java`, `src/test/java/com/guillermomolina/protos/execution/foreign/api/ProtosExternalForeignProviderTest.java`, `build/native/test-native.sh`, `docs/guide/library/inspect.md`, `docs/guide/tools/cli.md`, `pom.xml`, and `CHANGELOG.md`.

## Acceptance evidence and provenance

The maintainer explicitly reported that `git diff --check` was clean and all local tests passed following publication. The published [`0.3.391-SNAPSHOT` changelog](https://github.com/guillermomolina/protos/blob/e03e2c82ad1d2f4090e8e5baae631ad9790a09c6/CHANGELOG.md) records:

| Gate | Reported outcome |
|---|---|
| Focused JUnit suite | 86/86 PASS |
| Native harness | PASS, including `NATIVE_INSPECT=PASS` |
| Integrated `make test` | PASS |
| `make check` | PASS |
| `git diff --check` | PASS, maintainer statement in coordination interaction |

The GitHub commit and changed-file set were independently fetched, but the agent did not receive raw test transcripts or execute the reported validation. Therefore the result is **MAINTAINER_REPORTED_PASS**, not an independently reproduced run.

CLI-specific regression coverage exercises authorized and denied file/`-e`/REPL imports, all four API operations, duplicate/incompatible flags, application argument preservation, reuse of a CLI instance without leaking authorization, and help text. `ProtosExternalForeignProviderTest` uses its real `inventado:` plugin JAR and checks both flag orders in one Context. The Native harness exercises allowed/denied file and `-e` execution without claiming JVM/Native diagnostic text equivalence.

## Coordination and remaining approval boundary

The product publication and maintainer-reported technical validation are complete. The CLI010 Issue [#938](https://github.com/guillermomolina/protos/issues/938) explicitly designated the new **public CLI option contract**—name, entry forms, ordering, duplicate flags, and unsupported commands—as subject to an owner design-approval gate. No separate, exact owner approval of that contract is evidenced in the interaction used for this record. Publishing the implementation and passing tests must not be represented as formal design ratification.

At this evidence checkpoint, technical publication is **complete**, but formal issue closure remains subject to confirming explicit owner acceptance of the exact CLI surface. This is a coordination gate, not a claim that another code change or test failure exists. The D205 Standard Library contract remains ratified and unaffected; no new normative `spec/` artifact or public release was required.

```text
WORK_ITEM=CLI010
PRODUCT_REVISION=e03e2c82ad1d2f4090e8e5baae631ad9790a09c6
IMPLEMENTATION_VERSION=0.3.391-SNAPSHOT
PRODUCT_PUBLICATION=VERIFIED
FOCUSED_JUNIT=86_OF_86_MAINTAINER_PASS
NATIVE_INSPECT=MAINTAINER_REPORTED_PASS
MAKE_TEST=MAINTAINER_REPORTED_PASS
MAKE_CHECK=MAINTAINER_REPORTED_PASS
PRODUCT_DIFF_CHECK=MAINTAINER_REPORTED_PASS
RAW_LOCAL_TEST_LOGS=NOT_PROVIDED
NORMATIVE_SPEC_CHANGE=NONE
PUBLIC_CLI_CONTRACT_OWNER_APPROVAL=NOT_EXPLICITLY_DOCUMENTED
FORMAL_CLOSURE=PENDING_CONTRACT_ACCEPTANCE
```
