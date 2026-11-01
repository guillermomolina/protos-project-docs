# I136 — Final publication, local validation and closure evidence (0.3.430)

**Date:** 2026-11-01  
**Work item:** [I136 #1015](https://github.com/guillermomolina/protos/issues/1015), one atomic implementation slice in **`guillermomolina/protos`**.  
**Design authority:** [D232 #1014](https://github.com/guillermomolina/protos/issues/1014), already owner-approved and closed; [exact approved language decision](../../decisions/language/D232_DIRECTIONAL_RANGE_DUAL_INVERSION_SELECTED.md).  
**Evidence class:** revision-bound GitHub publication verification plus separately attributed local maintainer validation report. This is not an independent test log and is not a new normative decision.

## Immutable publication identity

- **Product commit:** [`guillermomolina/protos@dd4db2e9722df293aac0f3674aa2d88b46841aff`](https://github.com/guillermomolina/protos/commit/dd4db2e9722df293aac0f3674aa2d88b46841aff).
- **Commit subject (verified from GitHub):** `I136: replace Range helpers with Protos-first Range and InvertedRange prototypes (0.3.430)`.
- **Parent commit:** `fba47c40c4b79b6e580df1c1f78a3fcecbd067e2` (I135, previous implementation snapshot).
- **Published implementation version:** `0.3.430-SNAPSHOT`, verified in the product `pom.xml` and root `CHANGELOG.md` at the product commit.
- **Published global specification revision:** `0.1.485`, verified in `spec/PROTOS_SPEC_CHANGELOG.md` at the product commit.
- **GitHub commit tree summary:** **14 modified files; 764 insertions, 242 deletions**, a single product commit. No new production Java class, parser change, new Core runtime family, or native Truffle implementation.

GitHub was queried for the actual `main` commit and the exact `Range.protos`, test, `pom.xml`, `CHANGELOG.md`, and spec-changelog blobs at the immutable commit. That verifies publication and content, **not that any test command ran on the evidence agent's machine**.

## Source change and conformance trace

| Published source | Evidence of delivered contract |
| --- | --- |
| [`protos/lib/collections/Range.protos`](https://github.com/guillermomolina/protos/blob/dd4db2e9722df293aac0f3674aa2d88b46841aff/protos/lib/collections/Range.protos) | Replaces old top-level helpers with exactly two ordinary exported prototype slots `Range`/`InvertedRange`; each defines `init`, `each`, `eachStep`, `invert`, `reverse`. |
| Same module | `init(...args)` enforces exactly two genuine Integers and the correct parent and bound order **before** adding reflectable own `start`/`stop` slots and calling `freeze()`. Ordinary inherited `Object.call` allocates the child. |
| Same module | `Range.eachStep`: positive Integer step, `current < stop`; `InvertedRange.eachStep`: negative Integer step, `current > stop`; `each` delegates to ±1. No proportional array materialization. |
| Same module | Practical `invert` swaps limits and direction, generally changing visited members; set-preserving `reverse` adjusts bounds by ±1. Both return new ordinary objects without mutating the source. |
| [`range-core.protos`](https://github.com/guillermomolina/protos/blob/dd4db2e9722df293aac0f3674aa2d88b46841aff/protos/tests/conformance/library/collections/range-core.protos) | Constructor/reflection/frozen and failure behavior; exact argument counts; Integer admission; large-Integer boundaries; both directional sequences, strides, involutions and empty ranges. Includes the final explicit post-inversion step distinction: `Range(0,11).invert().eachStep(-3)` visits `11,8,5,2`, while `reverse().eachStep(-3)` visits `10,7,4,1`. |
| [`range-callbacks-and-control.protos`](https://github.com/guillermomolina/protos/blob/dd4db2e9722df293aac0f3674aa2d88b46841aff/protos/tests/conformance/library/collections/range-callbacks-and-control.protos) | Exact one-Integer callback argument, ignored result and `null` completion, Error identity, `^` nonlocal return, `ensure` cleanup, empty noninvokable callbacks and polymorphic `call`. |
| [`module-surface-and-imports.protos`](https://github.com/guillermomolina/protos/blob/dd4db2e9722df293aac0f3674aa2d88b46841aff/protos/tests/conformance/library/collections/module-surface-and-imports.protos) | Checks two module exports, correct parent identities, module import identity and absence of old module helper selectors. |
| `protos/examples/algorithms/{fizzbuzz,100-doors}.protos` and `protos/examples/closures/{value-capture,first-class-functions}.protos` | All four published examples migrated from old module helper sends to `Ranges.Range` extraction and `Range(a,b).each(block)`. |
| [Normative `VALUES_AND_COLLECTIONS.md`](https://github.com/guillermomolina/protos/blob/dd4db2e9722df293aac0f3674aa2d88b46841aff/spec/semantics/VALUES_AND_COLLECTIONS.md) | New D232 Standard Library section is the single normative owner, with explicit LIB017 incompatibility. |
| `docs/guide/05-values-identity-equality-and-collections.md`, `docs/guide/library/README.md`, `spec/PROTOS_SPEC_CHANGELOG.md`, `pom.xml`, `CHANGELOG.md` | Practical usage and index, global spec revision `0.1.485`, implementation version `0.3.430-SNAPSHOT`, and atomic product release notes. |

**Deliberate public-API break:** the former `Range.each(start,stop,block)` and `Range.reverseEach(start,stop,block)` module helper selectors are removed **without compatibility aliases**. This is intentional, recorded under D232 and carried by I136; no claim is made that independently maintained external consumers have been migrated. Historical LIB017 approval remains unmodified.

## Validation provenance and limits

The **project maintainer** explicitly reported in the current conversation, after publishing the immutable I136 commit:

> I136: replace Range helpers with Protos-first Range and InvertedRange prototypes (0.3.430) pushed.  
> el git diff check esta limpio.  
> make test y make check pasan en local.

The maintainer had earlier reported that **all focused tests pass** and supplied the pre-publication diff, which was reviewed; the later product commit was read independently through GitHub.

| Checkpoint | Outcome | Evidence and scope |
| --- | --- | --- |
| Publication at product commit `dd4db2e9722df293aac0f3674aa2d88b46841aff` | **VERIFIED via GitHub** | Exact commit and file list, product source/changelogs |
| I136 focused conformance tests | **PASS — maintainer previously reported** | User statement in the same I136 workflow; individual command logs not provided |
| `git diff --check` | **PASS — maintainer reports clean** | Local maintainer assertion; no independent command execution |
| `make test` | **PASS — maintainer reports local** | Maintainer attestation, not CI or locally rerun by evidence writer |
| `make check` | **PASS — maintainer reports local** | Maintainer attestation, not CI or locally rerun by evidence writer |
| Java/Native compatibility as part of reported gates | **Scope limited to make targets** | No separate Native Image build/run explicitly claimed |
| Benchmark/performance improvement | **NOT CLAIMED** | No before/after run or measured speedup supplied |
| Published repository working tree after push | **NOT INDEPENDENTLY INSPECTED** | GitHub commit verified; local status not asserted |
| External consumers under deliberate breaking change | **NOT VALIDATED** | No compatibility wrapper promised |

**Attribution is intentional:** PASS statements are the maintainer's local human-executor report. Repository inspection proves the 14-file commit and public contract, not independent test execution or external deploy status.

## GITHUB021 and closeout reconciliation

The published implementation preserves the owner-ratified behavioral distinctions from D232:
- Both range values are ordinary prototype-backed frozen instances, strict constructor Integer values and matching directional bounds. They preserve the explicitly selected `Range(0,3)` -> `0,1,2` and `InvertedRange(3,0)` -> `3,2,1`.
- `invert()` swaps endpoints and changes direction; `reverse()` retains the original unit-range members. Both are involutions on descriptor values. The concrete `eachStep(-3)` counterexamples above explicitly distinguish them.
- Direction-specific signed steps are strictly validated even for empty intervals; callbacks use ordinary Protos dispatch/error/nonlocal control and exact arbitrary-precision Integers.
- LIB017's prior module helper surface and wrong-order semantics are intentionally superseded by the approved new API; no Java, grammar or Core additions are inferred.

**Decision approval provenance:** owner-approved D232.  
**One-slice product publication:** confirmed immutable `dd4db2e9722df293aac0f3674aa2d88b46841aff`.  
**Full human local acceptance:** `make test` + `make check` + clean `git diff --check` reported PASS.  
**Required durable project evidence:** this file.  
**Issue disposition:** I136 may be closed as **completed**, with `family:I` retained and the superseded `status:ready` label removed. D232 remains closed independently; closing I136 does not reopen its design or require another implementation slice.

```text
WORK_ITEM=I136
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=dd4db2e9722df293aac0f3674aa2d88b46841aff
COMMIT_VERIFIED=YES
IMPLEMENTATION_VERSION=0.3.430-SNAPSHOT
SPEC_REVISION=0.1.485
CHANGED_FILES=14
ONE_SLICE_IMPLEMENTATION=PASS
PROTOS_FIRST=YES
PUBLIC_HELPER_REMOVAL=YES_BREAKING_CHANGE
FOCAL_CONFORMANCE=PASS_MAINTAINER_REPORTED
GIT_DIFF_CHECK=PASS_MAINTAINER_REPORTED
MAKE_TEST=PASS_MAINTAINER_REPORTED
MAKE_CHECK=PASS_MAINTAINER_REPORTED
SEPARATE_NATIVE_BUILD=NOT_ATTESTED
PERFORMANCE_DELTA=NOT_ATTESTED
EXTERNAL_CONSUMER_MIGRATION=NOT_ATTESTED
REQUIRED_DURABLE_PUBLICATION=THIS_COMMIT
ISSUE_CLOSURE=COMPLETED
```
