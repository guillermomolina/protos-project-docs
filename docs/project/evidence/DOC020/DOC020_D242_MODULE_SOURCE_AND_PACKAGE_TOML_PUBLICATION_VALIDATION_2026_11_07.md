# DOC020 — D242 MODULE/SOURCE and package.toml publication/validation evidence

**Date:** 2026-11-07. **Evidence owner:** [DOC020 #1077](https://github.com/guillermomolina/protos/issues/1077). **Evidence type:** exact GitHub product publication plus attributed human validation. This is non-normative and does not amend [D242](https://github.com/guillermomolina/protos/issues/1060).

## Immutable product identity

- **PROTOS_REVISION:** [`3791394c46aff8ad7508f64e56cde4594e502a56`](https://github.com/guillermomolina/protos/commit/3791394c46aff8ad7508f64e56cde4594e502a56).
- **PARENT_REVISION:** [`8a6366e486d0b58b58cce17ce627734d9cadcc2d`](https://github.com/guillermomolina/protos/commit/8a6366e486d0b58b58cce17ce627734d9cadcc2d), the completed I152 baseline.
- **COMMIT_SUBJECT:** `DOC020: document MODULE/SOURCE imports and package.toml`.
- **VERIFIED_GITHUB_COMPARE:** `ahead_by=1`, 38 paths, 980 additions, 259 deletions. [Revision-to-revision comparison](https://github.com/guillermomolina/protos/compare/8a6366e486d0b58b58cce17ce627734d9cadcc2d...3791394c46aff8ad7508f64e56cde4594e502a56).
- **CHANGED_CATEGORIES:** 9 Markdown (6 existing updated, 3 new project READMEs); 21 new Protos sources; 4 package TOML manifests; 3 lockfiles; 1 Java test class.
- **UNCHANGED:** production runtime/tools source, normative `spec/`, `pom.xml`, `CHANGELOG.md`. DOC020 follows its agreed documentation/learning-only version rule and retains `0.3.450-SNAPSHOT`.

## Direct publication links and acceptance

| Acceptance surface | Exact product artifact |
| --- | --- |
| Correct SOURCE vs MODULE guide | [Chapter 06](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/docs/guide/06-modules-and-imports.md) |
| Standalone SOURCE complete-context tutorial | [Lesson 04](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/protos/tutorials/08-language-interactions/04-source-full-context.protos) |
| Standard MODULE public identity tutorial | [Lesson 05](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/protos/tutorials/08-language-interactions/05-public-module-object.protos) |
| Package MODULE tutorial with manifest, lock and private access | [Lesson 06 README](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/protos/tutorials/08-language-interactions/06-package-exports/README.md) |
| Practical SOURCE/encapsulated MODULE example | [Public pricing example](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/protos/examples/modules/public-module-pricing/README.md) |
| Public workspace dependency and forbidden imports | [Invoice example](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/protos/examples/modules/package-invoice/README.md) |
| Testable public-run contracts and negative checks | [JUnit integration class](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/src/test/java/com/guillermomolina/protos/cli/ProtosDoc020LearningMaterialsTest.java) |
| Tool/execution guidance | [Package Tool guide](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/docs/guide/tools/package-tool.md), [CLI guide](https://github.com/guillermomolina/protos/blob/3791394c46aff8ad7508f64e56cde4594e502a56/docs/guide/tools/cli.md) |

The required package manifests map names to MODULE directories containing `module.protos`, never source filenames. Three example trees include `protos.lock`; the workspace tree also contains a member `package.toml`. The chapter explains current `protos run` versus direct `bin/protos <file>` and `protos package manifest` without inventing lock generation or registry/install commands.

## Maintainer-executed test and validation provenance

1. Standalone SOURCE tutorial (`04-source-full-context.protos`) printed `true`, `42`, `flat discount`, `40`; standard MODULE tutorial (`05-public-module-object.protos`) printed `true`, `true`, `true`, `false`.
2. `bin/protos lint` over exactly **21 new Protos sources** returned `DOC020_LINT=PASS (21 sources)`.
3. `bin/protos package manifest` and `bin/protos run Main` succeeded from each of the three package project directories; printed `package-total=44`, `price=42`, `invoice-total=49`.
4. `mvn -q -Dtest=ProtosDoc020LearningMaterialsTest,ProtosWorkspaceRunCliTest test` initially failed one DOC020 baseline string assertion because the test expected literal `\\n` rather than a newline; this was corrected **only in the Java test** (including the negative-manifest fixture string). The maintainer reran that same focal command successfully with no Maven error output. The successful result is attributed to the maintainer, not separately rerun here.
5. Final maintainer report: **`make test` PASS, `make check` PASS**, `git diff --check` clean; prior staging check `git diff --cached --check` clean. Product push output reports `8a6366e4..3791394c` on `main`.

**Independently checked:** GitHub commit `3791394c46aff8ad7508f64e56cde4594e502a56` is retrievable with all 38 expected filenames; a compare against the exact I152 revision returns `ahead_by=1`, the 38 path names and +980/-259 counts. No local shell tests, document build, CI, Native Image, benchmark, registry publication or product release were executed or inferred by this evidence record.

## Closure and related-work boundaries

- [Maintained DOC020 work record](../../work/DOC020/DOC020_D242_MODULE_SOURCE_AND_PACKAGE_TOML_LEARNING_COMPLETION_2026_11_07.md).
- [DOC020 live Issue](https://github.com/guillermomolina/protos/issues/1077).
- [D242 ratified language decision](../../decisions/language/D242_MODULE_SOURCE_PUBLIC_EXPORTS_AND_PACKAGE_MANIFEST_RATIFIED.md).
- [I151 completed implementation](https://github.com/guillermomolina/protos/issues/1066); [I152 completed implementation](https://github.com/guillermomolina/protos/issues/1067).
- [INC004 broader incubator](https://github.com/guillermomolina/protos/issues/1010) remains open but its Module access declarations/export candidate is implemented and documented.
- Live Issue closure should cite both the immutable product revision and the separately committed project-record revision. This record itself does not claim that GitHub's Issue state has already transitioned.

```text
FORMAL_IDENTIFIER=DOC020
PROTOS_REVISION=3791394c46aff8ad7508f64e56cde4594e502a56
PRODUCT_BASE_REVISION=8a6366e486d0b58b58cce17ce627734d9cadcc2d
GITHUB_COMPARE=1_COMMIT_38_PATHS_980_ADDITIONS_259_DELETIONS
NEW_PROTOS_SOURCES=21
MARKDOWN_PATHS=9
PACKAGE_MANIFESTS=4
PACKAGE_LOCKFILES=3
NEW_JAVA_TEST_CLASSES=1
HUMAN_LINT=21_PASS
HUMAN_PACKAGE_RUN=3_PASS
HUMAN_JUNIT_FOCAL=PASS_AFTER_TEST_ONLY_CORRECTION
HUMAN_MAKE_TEST=PASS
HUMAN_MAKE_CHECK=PASS
HUMAN_GIT_DIFF_CHECK=CLEAN
PRODUCT_PUBLICATION=VERIFIED
INDEPENDENT_CI_OR_TEST_REPLAY=NOT_PERFORMED
SPEC_OR_PRODUCTION_CODE_CHANGED=NO
VERSION_BUMP=NO
```
