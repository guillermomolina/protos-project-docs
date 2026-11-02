# LIB023-B/C — Complete D213 Standard Library Array API: published implementation and validation

**Formal owner:** [LIB023 #953](https://github.com/guillermomolina/protos/issues/953); ratified semantic decision [D213 #949](https://github.com/guillermomolina/protos/issues/949) and [durable decision record](../../decisions/language/D213_COLLECTIONS_LIBRARY_API.md).
**Exact product publication:** [`91c40a94f4fc12844c0757f131abe436d56314a5`](https://github.com/guillermomolina/protos/commit/91c40a94f4fc12844c0757f131abe436d56314a5) in `guillermomolina/protos` (`main`); GitHub commit read-back confirms **10 changed paths**.
**Implementation version:** `0.3.438-SNAPSHOT` (`pom.xml` and `CHANGELOG.md` at the exact product revision).
**Normative revision:** `0.1.491`, recorded in `spec/PROTOS_SPEC_CHANGELOG.md` and applied to `spec/semantics/VALUES_AND_COLLECTIONS.md`.
**Evidence date:** 2026-11-02.
**Status:** LIB023-A was previously published at [`64b198f3b10ad050e83918eea9e8636b2a6e17cc`](https://github.com/guillermomolina/protos/commit/64b198f3b10ad050e83918eea9e8636b2a6e17cc); this B+C product publication completes **all three LIB023 slices**. Learning follow-up [DOC017 #1036](https://github.com/guillermomolina/protos/issues/1036) remains separately tracked, and is not a missing LIB023 implementation slice.

## Exact B+C shipped delta

| Role | Published product paths |
| --- | --- |
| Public Array module | `protos/lib/collections/Array.protos` |
| Internal ordinary Protos implementation | `protos/lib/collections/internal/ArrayBulk.protos` |
| B suite-native conformance | `protos/tests/conformance/library/collections/array-queries-prefix-suffix.protos` |
| C suite-native conformance/regressions | `protos/tests/conformance/library/collections/array-bulk-transformations.protos` |
| Authoritative conformance registration | `protos/tests/conformance/manifest.tsv` |
| Normative contract/revision | `spec/semantics/VALUES_AND_COLLECTIONS.md`; `spec/PROTOS_SPEC_CHANGELOG.md` |
| Programming guide | `docs/guide/05-values-identity-equality-and-collections.md` |
| Release metadata | `pom.xml`; `CHANGELOG.md` |

### Slice B — snapshot queries and prefix/suffix

- `find(array, predicate)`: first matching **snapshotted** original element or `null` if absent (a matched `null` needs `findIndex` to distinguish).
- `contains(array, value)`: ordered scan using ordinary `element == sought`; no Set hashing or identity shortcut.
- `any(array, predicate)`, `all(array, predicate)`: strict canonical Boolean results, short-circuiting and conventional empty outcomes `false` and `true`.
- `take(array, count)`, `drop(array, count)`: nonnegative semantic Integers; saturate counts exceeding source length; return new open base Arrays, including empty/no-op results.

### Slice C — bulk transformations

- `flatMap(array, transform)`: snapshot input, one mapper invocation per input, each mapped result must be a standard Array; shallow-snapshot each chunk **immediately** and concatenate one level.
- `zip(left, right)`: truncate to shorter input, fresh outer Array and fresh open two-element pair Arrays; no padding.
- `partition(array, predicate)`: strict Boolean once per snapshot element, retaining decisions and stable order in independent fresh accepted/rejected bucket Arrays.
- `distinct(array)`: first representative/order under existing ordinary hashed Set/Map `hash` and `==` membership; membership decisions are retained for output fill, not recalculated.
- The public `std:collections/Array` module retains its original entrypoints and adds the approved operations without new Core Array selectors, resize semantics, JVM/Truffle kernels, or an exported public builder. The C algorithms live in an ordinary private `std:collections/internal/ArrayBulk` module; the public source remains under the approximately 800-line implementation guideline. Public `///` source API documentation was added so the Standard Library documentation extractor remains complete.

This concludes the previously ratified **sixteen new functions plus optional default comparator of `sort`** across A+B+C. The existing `parallelSort` contract is unaffected.

## Maintainer-reported validation and diagnostic trace

The repository owner executed product validations in the local working tree, and after the publication reported:

- `git diff --check` **clean**.
- `make test` **PASS**, after correcting a missing `///` API documentation block for each of four public C functions; these were the cause of the prior `ProtosStandardLibraryDocumentationExtractorTest` failure.
- `make check` **PASS** (reported on the same final local change set).
- Focused `protos test --jobs 1`: **30/30 PASS** for C; **91/91 PASS** for the A+B+C selected collection regression group.
- Pre-publication static inventory: public Array module 795 lines before four public API documentation additions (final source 799 lines); B test file 22 logical cases and C test file 30; both newly registered in `manifest.tsv`; internal C module lint/import PASS.

An intermediate `INFRA` in a new C test resulted from returning `Array.each(...)` (the original mutable Array) as the test body's final expression. Appending an explicit terminal `null` eliminated the infrastructure-aborted result; the maintainer then observed 30/30 and 91/91 PASS. The original internal failure phase was not separately archived; this is a diagnostic account, not an assertion that the host/runtime was defective.

The first full-suite attempt also exposed missing D064-extractable source comments for `flatMap`, `zip`, `partition`, and `distinct`. The public module gained four `///` documentation lines and the maintainer subsequently reported full `make test` PASS. Warnings from the existing Graal/Truffle compile do not constitute failed validation.

**Provenance limit:** full-suite PASS, `make check` PASS, local diff checks and focused test counts are **human-reported**; this record does not claim independent agent execution, a Native Image build, a separately retained raw JUnit report, or a release tag. The product commit, changed paths, implementation version and normative revision are verifiable from the immutable GitHub product revision.

## Closure relationship and non-goals

- LIB023 implements precisely D213's approved Array Standard Library surface; no additional slice D is required.
- Any future revisions to missing tutorial lesson/cookbook workflows belong to [DOC017 #1036](https://github.com/guillermomolina/protos/issues/1036), not a hidden fourth implementation slice.
- Specification authority remains in `guillermomolina/protos/spec/`. This file is revision-coupled, historical project evidence rather than a normative replacement.

**PROTOS_REVISION=`91c40a94f4fc12844c0757f131abe436d56314a5`**
**CROSS_REFERENCES:** [LIB023 issue](https://github.com/guillermomolina/protos/issues/953) → [this exact product revision](https://github.com/guillermomolina/protos/commit/91c40a94f4fc12844c0757f131abe436d56314a5) → [D213 decision](../../decisions/language/D213_COLLECTIONS_LIBRARY_API.md) → this project-evidence file.
