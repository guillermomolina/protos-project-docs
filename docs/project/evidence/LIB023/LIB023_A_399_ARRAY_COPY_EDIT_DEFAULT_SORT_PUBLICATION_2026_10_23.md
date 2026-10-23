# LIB023-A — Array copy/edit operations and default sort: verified publication

**Formal implementation owner:** [LIB023 #953](https://github.com/guillermomolina/protos/issues/953), slice A of three (B and C remain pending).
**Decision authority:** [D213 #949](https://github.com/guillermomolina/protos/issues/949); [owner-ratified, non-normative decision record](../../decisions/language/D213_COLLECTIONS_LIBRARY_API.md).
**Exact product publication:** [`64b198f3b10ad050e83918eea9e8636b2a6e17cc`](https://github.com/guillermomolina/protos/commit/64b198f3b10ad050e83918eea9e8636b2a6e17cc) in `guillermomolina/protos`, verified by GitHub commit read-back; subject `LIB023-A: add Array copy/edit operations and default sort comparator (0.3.399)`.
**Publication checkpoint:** 2026-10-23 (date of the committed specification revision and maintainer report).
**Implementation version:** `0.3.399-SNAPSHOT`, verified in the product commit's `pom.xml` and `CHANGELOG.md`.
**Normative specification revision:** `0.1.469`, verified in the product commit's `spec/PROTOS_SPEC_CHANGELOG.md`.
**Record status:** slice A product published; LIB023 as a whole remains **OPEN**, awaiting B and C. This record is implementation evidence, not a new decision or an independently executed test report.

## Exact published source and test delta

The product commit contains **12 changed paths**, including one new conformance source and both implementation-publication metadata files:

| Role | Paths |
| --- | --- |
| Standard Library source | `protos/lib/collections/Array.protos` |
| New conformance suite | `protos/tests/conformance/library/collections/array-copy-edits.protos` |
| Extended conformance suites | `protos/tests/conformance/library/collections/array-sort.protos`; `array-common-contracts.protos`; `module-surface-and-imports.protos` (all under the same collections directory) |
| Test registration | `protos/tests/conformance/manifest.tsv` |
| Normative specification | `spec/semantics/VALUES_AND_COLLECTIONS.md`; `spec/PROTOS_SPEC_CHANGELOG.md` |
| User documentation | `docs/guide/05-values-identity-equality-and-collections.md`; `docs/guide/library/README.md` |
| Version and implementation history | `pom.xml`; root `CHANGELOG.md` |

The published `std:collections/Array` module adds six **ordinary library-level** operations:

- `append(array, value)`, `concat(left, right)`: fresh shallow copies through ordinary Array spread; `append` retains an Array `value` as a single element.
- `slice(array, start, end)`: strict half-open range `0 <= start <= end <= n`; shallow snapshot, local geometrically doubling `sized` allocator, sequential copy into a fresh result.
- `insertAt(array, index, value)`: strict `0 <= index <= n`; constructs a fresh result and shifts its own suffix backward, without modifying the source.
- `removeAt(array, index)`: strict `0 <= index < n`; snapshot and local `sized` allocator, copying all elements except the selected index.
- `reverse(array)`: fresh shallow copy and symmetric in-result swaps.

All six recognize actual standard Array owners (`Array.recognizes`); index parameters must be semantic Integers (`Integer.recognizes`) satisfying exact mathematical bounds, including checks for very large Integers. Results are new, **open**, independently stored Arrays even for closed/frozen sources and empty/no-op slices. Nested references are preserved; Core Array length remains fixed. Array copy/edit semantics are not provided by new Java, Truffle, parser or native builder operations.

The existing stable merge `sort` is preserved except for the signature default `less: (left, right) => left < right`. Only an **omitted** comparator selects this default; explicit `null` does not. The implementation retains the existing comparator error/stability rules and leaves `parallelSort` unchanged.

The registered new `array-copy-edits.protos` source supplies 29 logical tests per the editing handoff, with empty/full/boundary cases, enormous/invalid indices, shallow identity and self-reference, original/result independence, closed/frozen/descendant sources, lookalikes, source-slot overrides and flexible extra arguments. Existing sort, common-contract and public-import tests were expanded. The committed specification and guide document **only LIB023-A**, not the as-yet-unpublished B/C functions.

## Validation provenance and prior INFRA diagnostic

**Maintainer-reported final acceptance (2026-10-23):**
- Product `git diff --check` clean.
- Local test suite **PASS**, as explicitly reported by the maintainer after publication.
- Earlier explicit `MAKE_CHECK_RC=0`, establishing `make check` PASS prior to the final version/changelog publication.
- Focused executions independently **shown by the maintainer in the conversation**: the `array-common-contracts.protos` file passed all three cases with `--jobs 1`; it passed **3/3 in each of three runs** with `--jobs 8`; the whole `protos/tests/conformance/library/collections` directory passed **135/135** with `--jobs 8`.

An earlier full-corpus attempt reported `2811 completed, 1 failed - infrastructure-aborted` for `array-common-contracts.protos::array copy edit results from closed and frozen sources are open`. The agent revised the test, but the same `INFRA` result appeared again before the targeted PASS runs. `INFRA` describes the Test Tool's infrastructure/error classification rather than an ordinary failing assertion; the original internal phase was **not extracted**. The later successful focused runs and maintainer's final full-suite PASS establish the accepted publication checkpoint, but do not by themselves prove a specific root cause for that earlier abort.

GitHub source/commit/version checks were performed by the documenting assistant; it did **not** execute Maven, `make test`, `make check`, or the Protos test runner. No Native Image, throughput/benchmark, JIT, tagged release or independently archived machine-readable full-suite log is claimed.

## Remaining scope and issue boundary

The D213 [LIB023 #953](https://github.com/guillermomolina/protos/issues/953) implementation plan remains a **single three-slice issue**:

- **A — PUBLISHED:** six copy/edit methods and optional default `sort`.
- **B — PENDING:** `find`, `contains`, `any`, `all`, `take`, `drop`.
- **C — PENDING:** `flatMap`, `zip`, `partition`, `distinct` and cross-operation completion.

Keep the canonical public API/semantic authority in the product `spec/`, and maintain this file as historical revision-coupled evidence. Do not close #953 or infer D213's entire approved surface is implemented until slices B and C are published and their final acceptance is recorded.
