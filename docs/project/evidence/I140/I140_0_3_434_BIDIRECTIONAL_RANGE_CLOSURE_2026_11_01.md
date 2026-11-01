# I140 — Protos-first bidirectional Range factories, published 0.3.434 and closure evidence

**Date:** 2026-11-01. **Implementation:** [I140 #1029](https://github.com/guillermomolina/protos/issues/1029), successor to owner-approved [D236-B1 #1028](https://github.com/guillermomolina/protos/issues/1028). **Scope:** one atomic Protos-first implementation slice, with human-executed product validation and commit/push. **Authority:** durable non-normative publication and verification evidence. The product `spec/` remains the normative language authority; GitHub Issues own live status.

## Exact published product

- **PRODUCT_REPOSITORY:** `guillermomolina/protos`
- **PRODUCT_REVISION:** [`2108e5ab2eacd1ebd0f64041b49d7245a414481e`](https://github.com/guillermomolina/protos/commit/2108e5ab2eacd1ebd0f64041b49d7245a414481e), independently confirmed as the published `main` commit through GitHub, with parent `8af1a70a56ca113a94eea1a5935a74a80fecdfd2`.
- **Commit message:** `I140: implement guarded bidirectional Range factories (0.3.434)`.
- **Implementation version:** `0.3.434-SNAPSHOT`, confirmed in `pom.xml` and `CHANGELOG.md` at PRODUCT_REVISION.
- **Normative specification revision:** `0.1.489` (D236-B1), confirmed in `spec/PROTOS_SPEC_CHANGELOG.md` at PRODUCT_REVISION. The owning changed specification is `spec/semantics/VALUES_AND_COLLECTIONS.md`.
- **Committed scope:** exactly 11 changed files. No new files; no Java production source, parser, grammar, new Core family, iterator, or additional module export.

### Exact changed-file inventory

1. `protos/lib/core/Range.protos`
2. `spec/semantics/VALUES_AND_COLLECTIONS.md`
3. `spec/PROTOS_SPEC_CHANGELOG.md`
4. `protos/tests/conformance/library/collections/range-core.protos`
5. `protos/tests/conformance/library/collections/range-prelude-and-module.protos`
6. `protos/tests/conformance/library/collections/range-callbacks-and-control.protos`
7. `protos/tests/conformance/actor/range-identity-and-transfer.protos`
8. `src/test/java/com/guillermomolina/protos/execution/integration/modules/ProtosCoreBootstrapTest.java`
9. `docs/guide/05-values-identity-equality-and-collections.md`
10. `pom.xml`
11. `CHANGELOG.md`

These paths are verified against GitHub's exact published commit, not inferred from the editor handoff. The initial editor slice covered nine source/spec/test/guide files; version/changelog were finalized in the same single product commit after human validation.

## Implemented semantic invariants and Protos-first evidence

The two canonical frozen Core prototypes `Range` and `InvertedRange` each now define **exactly six ordinary source-backed own methods**: `init`, `call`, `each`, `eachStep`, `invert`, `reverse`. Both new `call` Closures live in `protos/lib/core/Range.protos`. Canonical calls require exactly two genuine Integers, retain supplied endpoints without swapping, and choose the resulting canonical parent by endpoint order:

| Source expression | Actual immediate parent | Traversal |
| --- | --- | --- |
| `Range(0, 5)` | `Range` | 0, 1, 2, 3, 4 |
| `Range(5, 0)` | `InvertedRange` | 5, 4, 3, 2, 1 |
| `InvertedRange(5, 0)` | `InvertedRange` | 5, 4, 3, 2, 1 |
| `InvertedRange(0, 5)` | `Range` | 0, 1, 2, 3, 4 |
| `Range(5, 5)` | `Range` | empty |
| `InvertedRange(5, 5)` | `InvertedRange` | empty |

For a canonical receiver and direction already matched, `super.call(start, stop)` reuses normal `Object.call` construction; for opposite bounds, the Closure invokes the other canonical prototype. For a noncanonical inheriting receiver, the `this !== Range` / `this !== InvertedRange` guard delegates **before canonical validation** by `super.call(...args)`, retaining derived `init` and arbitrary ordinary argument vectors. Own `call` overrides and extracted bound-Closure invocation retain normal dispatch and methodHome rules. Core's lexical opposite prototype is independent of a caller's shadowing and the Actor-local compatibility-module export rebinding.

The two existing `init` bodies remain strict and unchanged; wrong direct `init` sends still fail. Each resulting instance is fresh, frozen, and has exactly the original `start` and `stop` local slots. Traversal and signed `eachStep` remain governed by the **actual result parent**, and existing `invert` and `reverse` algorithms remain unchanged. The corrected normative transformation table describes preconditioned instances `r` and `ir` rather than treating constructor syntax as a type assertion. Core/Prelude sharing, exactly two `std:collections/Range` exports, Actor/P behavior, callback control and unbounded Integer arithmetic are preserved.

This is an intentional D236-B1 reopening of D232's wrong-bound `Error` contract and D235-B1-F's five-own-method/inherited-only-`call` contract, not a silent compatibility claim. Source-backed construction reuses already specified invocation and super mechanisms; there is no Range-specific Java runtime kernel or Truffle dispatch node.

## Human-executor verification provenance

In the interactive review of the exact I140 patch, the maintainer first stated **“todos los tests focales pasan”** and supplied the combined staged/unstaged patch for review. The agent's only requested normative follow-up corrected the four-row `### Transformations` table, preserving formulas and clarifying instance parents. The user then reported the final product commit was pushed and explicitly attested:

> I140: implement guarded bidirectional Range factories (0.3.434) pushed.
>
> el git diff check esta limpio.
>
> make test y make check pasan en local.

The evidence publisher verified the exact product commit, changed-file list, version and specification revision via GitHub and inspected the resulting normative transformation table. The evidence publisher **did not execute** the product tests, build, local `git diff --check`, Native Image, or benchmark. No CI job URL or full test log was supplied; do not misrepresent maintainer-reported local PASS as independent CI verification.

| Gate | Result | Authority |
| --- | --- | --- |
| One product commit published | **VERIFIED** | GitHub commit `2108e5ab2eacd1ebd0f64041b49d7245a414481e` |
| Exactly 11 expected changed files | **VERIFIED** | Exact GitHub commit inventory |
| Implementation `0.3.434-SNAPSHOT` | **VERIFIED** | Product `pom.xml` and `CHANGELOG.md` at exact revision |
| Normative `0.1.489`, transformation-table correction | **VERIFIED** | Product `spec/` at exact revision |
| Focused conformance and Java validation | **PASS — maintainer reported** | User's in-session focal-test confirmation |
| `git diff --check` | **PASS — maintainer reported** | User's final confirmation |
| `make test` | **PASS — maintainer reported** | User's final confirmation |
| `make check` | **PASS — maintainer reported** | User's final confirmation |
| Separate native-image/distribution qualification | **NOT ATTESTED** | No result supplied |
| Performance/Truffle graph improvement | **NOT ATTESTED** | No measurements supplied |

## Decision links, exclusions and closure

- [D236-B1 selected design](../../decisions/language/D236_RANGE_CROSS_DIRECTION_B1_SELECTED.md); [owner approval and GITHUB021 invariants](../D236/D236_B1_OWNER_APPROVAL_INVARIANTS_2026_11_01.md) at ratification docs revision `55783cc92ebec20ba63c3b1c0ee027ce0178f325`.
- [Self-contained single-slice editor prompt](../../work/I140/I140_AGENT_ONE_SLICE_PROMPT.md) and [human executor handoff](../../work/I140/I140_HUMAN_EXECUTOR_ONE_SLICE.md), published in docs revision `4dbce4ce550466843994302335797491130b6bac`.
- **D231 #1013** remains out of scope: no `for` syntax, step-inference design approval, or iterator implementation.
- I140 is eligible for `closed / completed` only after this durable record is published, its exact project-record revision is verified, and both repository revisions are cross-linked in the live I140 issue. No priority, assignee, milestone or native hierarchy is to be invented.

```text
WORK_ITEM=I140
OWNER_DECISION=D236-B1
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=2108e5ab2eacd1ebd0f64041b49d7245a414481e
IMPLEMENTATION_VERSION=0.3.434-SNAPSHOT
SPEC_REVISION=0.1.489
PRODUCT_COMMIT=PUBLISHED_AND_VERIFIED
CHANGED_FILES=11_VERIFIED
FOCAL_TESTS=PASS_MAINTAINER_REPORTED
GIT_DIFF_CHECK=PASS_MAINTAINER_REPORTED
MAKE_TEST=PASS_MAINTAINER_REPORTED
MAKE_CHECK=PASS_MAINTAINER_REPORTED
NATIVE_IMAGE=NOT_ATTESTED
PORTABLE_DIST=NOT_ATTESTED
PERFORMANCE=NOT_MEASURED
D231_FOR_SYNTAX=OUT_OF_SCOPE_UNSELECTED
EXPECTED_ISSUE_CLOSURE=completed
```
