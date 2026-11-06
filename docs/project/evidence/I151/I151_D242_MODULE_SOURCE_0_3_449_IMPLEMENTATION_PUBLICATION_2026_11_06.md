# I151 — D242 MODULE/SOURCE implementation publication (0.3.449)

**Date:** 2026-11-06. **Role:** revision-bound implementation/publication and human-validation evidence (non-normative).
**Implementation owner:** [I151 #1066](https://github.com/guillermomolina/protos/issues/1066).
**Approved design:** [D242 #1060](https://github.com/guillermomolina/protos/issues/1060), [ratified language decision](../../decisions/language/D242_MODULE_SOURCE_PUBLIC_EXPORTS_AND_PACKAGE_MANIFEST_RATIFIED.md), and [GITHUB010/GITHUB021 review](../D242/D242_MODULE_SOURCE_EXPORTS_BREAKING_REPLACEMENT_REVIEW_2026_11_06.md).

## Exact product publication

- **Published revision:** [`9088e87d339558af168676a4dd3f31774e016b65`](https://github.com/guillermomolina/protos/commit/9088e87d339558af168676a4dd3f31774e016b65).
- **Previous revision:** [`07406c7e4e51c627035256c0cba3c4bd2b7f433f`](https://github.com/guillermomolina/protos/commit/07406c7e4e51c627035256c0cba3c4bd2b7f433f).
- **Verified commit message:** `I151: implement D242 MODULE/SOURCE separation (0.3.449)`.
- **Published Maven and implementation changelog version:** `0.3.449-SNAPSHOT`; see [`pom.xml`](https://github.com/guillermomolina/protos/blob/9088e87d339558af168676a4dd3f31774e016b65/pom.xml) and [`CHANGELOG.md`](https://github.com/guillermomolina/protos/blob/9088e87d339558af168676a4dd3f31774e016b65/CHANGELOG.md).
- GitHub's commit summary reports **7,220 additions and 4,624 deletions**. This is a single published implementation commit; it is not a benchmark or an independent runtime test report.

## Delivered MODULE/SOURCE implementation

- [`spec/semantics/MODULES.md`](https://github.com/guillermomolina/protos/blob/9088e87d339558af168676a4dd3f31774e016b65/spec/semantics/MODULES.md) records the replacement contract. A public Protos source-backed MODULE is identified by a logical directory containing `module.protos`; execution uses a private lexical initializer context, while explicit `this.Name: value` creation publishes own slots on the separate Actor-cached public module object. No automatic exports from arbitrary source filenames are introduced.
- A direct authorized SOURCE import of a `.protos` file yields its **complete source execution context**, including its top-level bindings. MODULE and SOURCE have noncolliding canonical identities, with cache-before-execute, partial cycles and failure/retry behavior. The implementation includes lexical SOURCE origin retention across Closure creation and later invocation; see [`ProtosModuleRuntime.java`](https://github.com/guillermomolina/protos/blob/9088e87d339558af168676a4dd3f31774e016b65/src/main/java/com/guillermomolina/protos/execution/modules/ProtosModuleRuntime.java), [`ProtosBytecodeRootNode.java`](https://github.com/guillermomolina/protos/blob/9088e87d339558af168676a4dd3f31774e016b65/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java), and [MODULE/SOURCE conformance](https://github.com/guillermomolina/protos/blob/9088e87d339558af168676a4dd3f31774e016b65/protos/tests/conformance/modules/module-source-separation.protos).
- Standard Library public modules were migrated to lowercase logical paths and explicit `module.protos` exports, with private SOURCE composition and corresponding tests, guides, examples and tutorials. Representative public entry: [`std:collections/array`](https://github.com/guillermomolina/protos/blob/9088e87d339558af168676a4dd3f31774e016b65/protos/lib/collections/array/module.protos). Old uppercase/file-backed `std:` imports are deliberately rejected; negative import tests remain negative tests.
- Tooling adapted to this boundary: Package Tool SOURCE dependencies, explicit Test Tool Package Case entries, and the Java Package Tool integration harness. The final local repair sequence reconciled the exact Package SOURCE entrances used by the `protos test` corpus and by Java fixtures, without restoring legacy public aliases. Relevant product files: [`ProtosCli.java`](https://github.com/guillermomolina/protos/blob/9088e87d339558af168676a4dd3f31774e016b65/src/main/java/com/guillermomolina/protos/cli/ProtosCli.java), [`ProtosPackageToolProtosTestSupport.java`](https://github.com/guillermomolina/protos/blob/9088e87d339558af168676a4dd3f31774e016b65/src/test/java/com/guillermomolina/protos/execution/packages/ProtosPackageToolProtosTestSupport.java), and [Package Tool cases](https://github.com/guillermomolina/protos/tree/9088e87d339558af168676a4dd3f31774e016b65/protos/tests/package-tool).
- Specification and documentation changed with the implementation, including `spec/semantics/MODULES.md`, related execution/callable rules and `docs/guide/06-modules-and-imports.md`. This is the selected D242 breaking change, **not** an additive compatibility migration.

## Validation provenance and boundaries

**Owner-reported from the local implementation checkout, before the version bump:**

- `make test-protos` — **PASS**.
- `make test` — **PASS**, after the Package Tool Java harness repair.
- `make check` — **PASS**.
- `git diff --check` — **clean**; product commit **pushed**.
- During remediation, 292 Package Tool SOURCE imports in 74 corpus files were migrated; the reported targeted Package Tool check was **19 passed, 0 failed**. The final negative-import and Network correction was **16 passed, 0 failed**, and the TLS targeted set was **33 passed, 0 failed**.
- The maintainer's workflow intentionally did **not** rerun tests after the `0.3.449-SNAPSHOT` bump.

**Independently checked here:** the exact product GitHub commit exists with the message and revision above, and its published `pom.xml` and changelog identify `0.3.449-SNAPSHOT`. Local `make` commands and `git diff --check` were executed/reported by the maintainer, **not** rerun by the documentation agent; full local test logs were not independently verified as CI evidence. No benchmark improvement or Native Image parity is claimed.

## Scope and disposition

I151 is the **MODULE/SOURCE runtime, Standard Library and import-call-site implementation**, published at the exact Protos revision above. [I152 #1067](https://github.com/guillermomolina/protos/issues/1067) separately owns replacing `protos.toml` with `package.toml` and related package identity/tooling; its completion is **not claimed** by this record. [D242 #1060](https://github.com/guillermomolina/protos/issues/1060) is already owner-approved and durably ratified; this implementation evidence does not revise that decision.

No scheduling priority, release milestone, issue-parent relation, independent CI result or product release is inferred from this publication.
