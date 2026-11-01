# I139 — Frozen Protos-first Core Range, module identity bridge and D064 closure

**Date:** 2026-11-01. **Work item:** [I139 #1022](https://github.com/guillermomolina/protos/issues/1022), implementation successor to [approved D235-B1-F #1019](https://github.com/guillermomolina/protos/issues/1019). **Status:** one product implementation commit published; final integrated validation reported PASS by the maintainer. **Authority:** this is durable, non-normative publication and validation evidence, not an additional language decision. The product `spec/` is normative and GitHub Issues own live work status.

## Revision-coupled publication

- **Product repository/revision:** [`guillermomolina/protos@a7b1a66dc6bb73c97ec0d52bb603bd9836600737`](https://github.com/guillermomolina/protos/commit/a7b1a66dc6bb73c97ec0d52bb603bd9836600737) — commit title `I139: promote Range prototypes to frozen Core with module compatibility`, verified through the GitHub commit endpoint. The commit contains **26 changed files**.
- **Implementation:** `0.3.433-SNAPSHOT`, verified in `pom.xml` and `CHANGELOG.md` at the exact product revision.
- **Global normative specification:** `0.1.488`, verified in `spec/PROTOS_SPEC_CHANGELOG.md` at that revision; observable contract in `spec/semantics/VALUES_AND_COLLECTIONS.md`.
- **Design authority:** [D235-B1-F ratification](../../decisions/language/D235_RANGE_CORE_PRELUDE_B1F_SELECTED.md) and [contemporaneous owner-approval/evidence record](../D235/D235_B1F_OWNER_APPROVAL_AND_COMPATIBILITY_2026_11_01.md). D232's ordinary directed Integer range contract is preserved except for the expressly approved prototype ownership, availability, identity and freeze changes.
- **Separate scope:** [D231 #1013](https://github.com/guillermomolina/protos/issues/1013) remains an open, unselected design of iteration syntax. I139 adds **no** `for` grammar, new Iterator protocol or Java Range intrinsic.

## Implemented source and compatibility witnesses

1. `protos/lib/core/Range.protos` is the single Protos-source definition of canonical `Range` and `InvertedRange`, both direct children of `Object` with exactly five own methods: `init`, `each`, `eachStep`, `invert`, `reverse`. Constructor invocation remains inherited `Object.call`; D232 Integer-only bounds, signed step, exclusive stop, callback, error, `invert` and `reverse` rules are retained.
2. `ProtosCoreBootstrap.java` source-loads the pair once before `prelude.protos`, validates their direct parent and uses the existing shared standard-graph freeze, including source Closure captures. `prelude.protos` exposes unqualified `Range` and `InvertedRange` names; no new native family, protocol, runtime Range payload, specialized dispatch node or per-Actor Core clone is introduced.
3. The existing eager `ProtosPrelude.standardModuleMembers` registration installs the exact frozen Core identities as the two initial slots of the ordinary, Actor-local `std:collections/Range` module instance. `protos/lib/collections/Range.protos` retains a comment-only source body, with independent, writable module slots but no duplicate prototypes. Existing import caching, module lifecycle, failure and cycle semantics are not modified.
4. Tests added or adapted in `protos/tests/conformance/library/collections/{range-core,range-callbacks-and-control,range-prelude-and-module}.protos`, `protos/tests/conformance/actor/range-identity-and-transfer.protos`, `actor/modules/workers.protos` and `core-surface/required-core-bindings.protos` cover frozen Core prototypes, module identity and rebinding, local shadowing, P visibility and copied instances, cross-Actor canonical parents and large Integer bounds, along with D232 behavior. The manifests register the new conformance sources; Java bootstrap/source-name tests assert the public surface and source-backed frozen objects.
5. D064/D067 documentation remains accurate despite runtime-preinstalled module members. `protos/lib/collections/Range.protos` uses two non-executable `//% @standard-member NAME | DOCUMENTATION` declarations. `ProtosStandardLibraryDocumentationExtractor.java` supports this convention *generically* from lexer-recognized comments, rejects invalid/duplicate/colliding declarations, and includes the resulting symbols in ordinary documentation/coverage. `ProtosStandardLibraryDocumentationExtractorTest.java` adds affirmative and negative fixtures. No D064 artifact format change, runtime member publication change, or hard-coded Range special case was introduced.
6. User-facing guide/library documentation and four existing algorithm/Closure examples now use the unqualified Core names. The implementation and specification changelogs record the product and normative revisions.

## Final human-executor validation attestation

After the product commit was pushed, the maintainer explicitly reported:

> I139: promote Range prototypes to frozen Core with module compatibility pushed.
>
> el git diff check esta limpio.
>
> make test y make check pasan en local.

In the preceding interactive review, the maintainer also reported **all focused tests PASS** for the I139 candidate; the editor's intermediate report documented correcting a mistaken test expectation (`InvertedRange` parent in the Actor `bounds` fixture), and the final product commit includes the correction. D064 test coverage and the new extraction convention are present in the published source. The agent publishing this record did not run builds, tests, local Git commands, or independent Native binaries; no logs or CI run URLs were supplied for separate attestation.

| Gate or claim | Recorded result | Basis |
| --- | --- | --- |
| Exact product commit, changed-file inventory | **VERIFIED** | GitHub commit inspection of `a7b1a66dc6bb73c97ec0d52bb603bd9836600737` |
| `0.3.433-SNAPSHOT` / specification `0.1.488` | **VERIFIED** | Exact-revision `pom.xml`, `CHANGELOG.md`, `spec/PROTOS_SPEC_CHANGELOG.md` |
| Focused Protos/JVM conformance during review | **PASS — maintainer-reported** | Earlier in-session statement, not independently executed here |
| `git diff --check` | **PASS — maintainer-reported** | Final owner report |
| `make test` — integrated Java and Protos | **PASS — maintainer-reported** | Final owner report |
| `make check` — repository checks | **PASS — maintainer-reported** | Final owner report |
| Separate `make -C build/native test` / Native Image | **NOT ATTESTED** | No explicit final result supplied |
| `dist/build_portable.py` / `dist/validate_portable.sh` | **NOT ATTESTED** | No explicit final result supplied |
| Benchmarks/performance improvement | **NOT ATTESTED** | No results supplied; no performance claim |

## Closure reconciliation

The design decision [D235 #1019](https://github.com/guillermomolina/protos/issues/1019) was already ratified and closed; the entire approved implementation and accompanying D064 documentation correction were published together in **one** product commit. The user's final local `make test`, `make check` and whitespace validation attest to the integrated closure candidate. Separate Native/distribution gates are not represented as passing without evidence. I139's executable `FULL` validation requirement is supported by the maintainer's reported `make test`; this record supports closing [I139 #1022](https://github.com/guillermomolina/protos/issues/1022) as `completed`, without selecting or closing D231.

```text
WORK_ITEM=I139
OWNER_DECISION=D235-B1-F
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=a7b1a66dc6bb73c97ec0d52bb603bd9836600737
IMPLEMENTATION_VERSION=0.3.433-SNAPSHOT
SPEC_REVISION=0.1.488
PRODUCT_COMMIT=PUBLISHED_AND_VERIFIED
FOCAL_TESTS=PASS_MAINTAINER_REPORTED
GIT_DIFF_CHECK=PASS_MAINTAINER_REPORTED
MAKE_TEST=PASS_MAINTAINER_REPORTED
MAKE_CHECK=PASS_MAINTAINER_REPORTED
NATIVE_IMAGE=NOT_ATTESTED
PORTABLE_DIST=NOT_ATTESTED
D231_FOR_SYNTAX=OPEN_UNSELECTED
EXPECTED_ISSUE_CLOSURE=completed
```
