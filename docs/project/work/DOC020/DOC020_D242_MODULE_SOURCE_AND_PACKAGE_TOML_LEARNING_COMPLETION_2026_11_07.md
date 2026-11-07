# DOC020 — D242 MODULE/SOURCE and package.toml learning completion

**Date:** 2026-11-07. **Owner:** [DOC020 #1077](https://github.com/guillermomolina/protos/issues/1077). **Role:** completed non-normative documentation, tutorial and cookbook work. **Exact product publication:** [Protos `3791394c46aff8ad7508f64e56cde4594e502a56`](https://github.com/guillermomolina/protos/commit/3791394c46aff8ad7508f64e56cde4594e502a56).

## Scope and authority

DOC020 teaches the already implemented [D242 #1060](https://github.com/guillermomolina/protos/issues/1060) breaking contract; it neither defines new language semantics nor implements the runtime. [I151 #1066](https://github.com/guillermomolina/protos/issues/1066) separately completed the MODULE/SOURCE runtime and explicit export mechanism at Protos `9088e87d339558af168676a4dd3f31774e016b65`; [I152 #1067](https://github.com/guillermomolina/protos/issues/1067) completed `package.toml` and package ContentIdentity v2 at `8a6366e486d0b58b58cce17ce627734d9cadcc2d`. The authoritative [MODULES specification](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/spec/semantics/MODULES.md) and those implementation owners are unchanged by DOC020.

## Published learning material

- **Programming guide:** [chapter 06](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/docs/guide/06-modules-and-imports.md) replaces the old all-imports-as-one-context/no-exports explanation with complete SOURCE contexts versus separately cached MODULE public objects, private initializer `context` versus public `this`, explicit own-slot publication, closure capture, cycles, failure/retry, Actor-local identity and package authority. The [guide index](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/docs/guide/README.md), [CLI guide](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/docs/guide/tools/cli.md), and [Package Tool guide](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/docs/guide/tools/package-tool.md) provide entry-mode and package links.
- **Progressive lessons:** [SOURCE full-context lesson](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/protos/tutorials/08-language-interactions/04-source-full-context.protos), [public MODULE/prototype identity lesson](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/protos/tutorials/08-language-interactions/05-public-module-object.protos), and [package MODULE lesson](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/protos/tutorials/08-language-interactions/06-package-exports/README.md) with `Main/module.protos`, `Prices/module.protos`, internal SOURCE, `package.toml`, `protos.lock` and negative access case. The [tutorial index](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/protos/tutorials/README.md) documents the progression.
- **Task-oriented examples:** [public-module pricing](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/protos/examples/modules/public-module-pricing/README.md) shows private SOURCE rules and explicitly published pricing Closure; [workspace invoice](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/protos/examples/modules/package-invoice/README.md) demonstrates `self:` and authorized `dep:` against a workspace member with its own `package.toml`, plus private, SOURCE and missing-initializer rejection examples. Both contain real lockfiles; [cookbook index](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/protos/examples/README.md) connects these examples to the existing direct-file import.
- **Executable protection:** [`ProtosDoc020LearningMaterialsTest`](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/src/test/java/com/guillermomolina/protos/cli/ProtosDoc020LearningMaterialsTest.java) exercises three public package runs, negative private/dependency/source import cases and legacy-manifest coexistence rejection. The negative-manifest fixture first proves that the otherwise identical project runs successfully.

## Publication and validation

GitHub comparison [I152 baseline to DOC020](https://github.com/guillermomolina/protos/compare/8a6366e486d0b58b58cce17ce627734d9cadcc2d...3791394c46aff8ad7508f64e56cde4594e502a56) confirms **one commit**, **38 changed paths**, **980 additions**, **259 deletions**. These comprise 9 Markdown files (6 updated, 3 created), 21 Protos lesson/example SOURCE files, 4 `package.toml` manifests, 3 `protos.lock` files, and 1 JUnit test class. No `src/main/**`, `spec/**`, `pom.xml`, or `CHANGELOG.md` change; implementation version remains `0.3.450-SNAPSHOT`, with no documentation-only version bump.

**Maintainer-run and maintainer-reported:** all 21 new Protos files pass `bin/protos lint`; previously added direct-file/standard-MODULE lessons print their expected results; the three `protos run Main` examples print `package-total=44`, `price=42` and `invoice-total=49`; both selected Maven/JUnit classes pass after correction of a test-only escaped-newline assertion; `git diff --check` and `git diff --cached --check` clean; `make test` and `make check` PASS. The maintainer committed and pushed the exact revision. **Independent verification here** is confined to GitHub commit/compare and source records, not replay of local tests or independent CI certification.

See [DOC020 revision-bound evidence](../../evidence/DOC020/DOC020_D242_MODULE_SOURCE_AND_PACKAGE_TOML_PUBLICATION_VALIDATION_2026_11_07.md). Live closure/status remains in [DOC020 #1077](https://github.com/guillermomolina/protos/issues/1077); broader [INC004 #1010](https://github.com/guillermomolina/protos/issues/1010) stays open for unrelated incubated ideas while its module-export candidate is resolved.

```text
WORK_ITEM=DOC020
PROTOS_REVISION=3791394c46aff8ad7508f64e56cde4594e502a56
PRODUCT_BASE_REVISION=8a6366e486d0b58b58cce17ce627734d9cadcc2d
PRODUCT_CHANGED_PATHS=38
PRODUCT_ADDITIONS=980
PRODUCT_DELETIONS=259
LINT_PROTOS=21_OF_21_PASS_REPORTED
PACKAGE_RUNS=3_OF_3_PASS_REPORTED
FOCAL_JUNIT=PASS_REPORTED
MAKE_TEST=PASS_REPORTED
MAKE_CHECK=PASS_REPORTED
GIT_DIFF_CHECK=CLEAN_REPORTED
PRODUCT_PUBLICATION=INDEPENDENTLY_VERIFIED
NORMATIVE_OR_RUNTIME_CHANGE=NO
PRODUCT_VERSION_BUMP=NO
```
