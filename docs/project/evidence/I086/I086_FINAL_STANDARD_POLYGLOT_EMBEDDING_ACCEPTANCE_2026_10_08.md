# I086-FINAL — Standard Polyglot embedding final acceptance

**Date:** 2026-10-08  
**Issue:** [I086 / guillermomolina/protos#840](https://github.com/guillermomolina/protos/issues/840)  
**Related ratified design:** [PLAT054 / #838](https://github.com/guillermomolina/protos/issues/838)  
**Published product revision:** [`6dab6ecc08c9a2102a388e00908c710a15cf2a4c`](https://github.com/guillermomolina/protos/commit/6dab6ecc08c9a2102a388e00908c710a15cf2a4c)  
**Product version:** `0.3.298-SNAPSHOT`  
**Normative specification:** `0.1.451`  
**Slice:** `I086-FINAL`  
**Type:** IMPLEMENTATION ACCEPTANCE — zero source modifications

## Exact acceptance provenance

The human executor reports, for the exact clean HEAD above:

- `git rev-parse HEAD` = `6dab6ecc08c9a2102a388e00908c710a15cf2a4c`; clean working tree and `git diff --check` clean.
- Previously completed full integrated local suite = **PASS**, already reported against this exact product revision. It was **not** rerun because no candidate bytes changed.
- `make -C build/native test` = **PASS** by human execution, including Native Image build, Native guest CLI smoke, Native Test Tool, Native DAP stacktrace and PLAT045 interpreter-only policy.
- `make dist-validate` = **PASS** by human execution, including portable JVM distribution build/validation; `smoke_polyglot_embedding.sh` with its `PLAT054_*` PASS markers (packaged Core, home and override precedence, invalid override, working outside checkout) and archive identity PASS.
- No defects were found in the final source/acceptance audit and **no product file was edited**. No new product commit, version bump or root `CHANGELOG.md` entry is required.

**Important provenance boundary:** Native/portable PASS values and the markers above are **human-reported**, not independently rerun or corroborated from raw logs by the coordinator. The source, commit identity, existing tests, Makefile targets and distribution smoke wiring were previously independently inspected. This evidence is sufficient for the agreed human-executor completion contract; it is not a CI attestation.

## Scope and acceptance classification

| Gate / acceptance | Result | Evidence type |
| --- | --- | --- |
| Standard `Context.newBuilder("protos").build()`, eval, bindings and executable host Value | PASS | Published product, JUnit suite and portable-JAR smoke |
| Packaged Core, language-home/override precedence, no mandatory Core path | PASS | Published tests and reported portable `PLAT054_*` smoke |
| Correct closure identity, scalar/other args, modules/bindings, Future and RootActor lifecycle | PASS | Published tests and human-reported full suite |
| Host Filesystem, provider confinement, bootstrap abort Alternative A, Network independence | PASS | Published custody/backends, conformance tests and human-reported suite |
| Context/Process resources, cancellation and no resurrection | PASS | Published lifecycle tests and human-reported suite |
| PAY AS YOU GROW: ordinary host callable and unused authority laziness | PASS | Published structural regression tests and human-reported suite |
| Native Image executable build, guest CLI/Test Tool/DAP and interpreter-only policy | PASS | Human-executed `make -C build/native test` |
| Portable JVM archive, standard embedding smoke and archive identity | PASS | Human-executed `make dist-validate` |
| I086 mandatory acceptance gaps | NONE | Source review plus supplied execution report |

## Explicit Native scope limitation

The existing Native gate builds and tests **the Protos native CLI**. It does **not** establish the separate claim that a Java host application using the standard Polyglot embedding has itself been compiled into Native Image and executed successfully. No such test exists in the current gate, and no such claim is made.

The I086 requirement is **no regression in Native execution**, not a new supported Native-hosted Java-embedding public distribution. The recorded limitation is **not a mandatory I086 closure blocker**, and the absence of this extra scenario must not silently expand the issue's scope.

## Closure and relationship to other work

- **B011 = CLOSED:** default Filesystem normative and product implementation are published, together with the approved fail-closed bootstrap decision.
- **B012 = CLOSED:** default Network normative and product implementation are published.
- **I086/#840 = TECHNICALLY CLOSEABLE:** all mandatory product and distribution acceptance gates pass according to the human executor. Its final GitHub lifecycle state still requires independently verified coordination/postconditions before the coordinator reports `CLOSED`.
- **PLAT054/#838 = OPEN / REVIEW:** architecture is ratified; this Issue's own native hierarchy/project/closure postconditions remain independent.
- **PERF033/#832 = CLOSED** already. **PERF032/#831** remains independent; no GraalJS/GraalPy timing parity, performance optimization, or benchmark-harness mutation is claimed.
- **I087/#841** application module concerns remain independent.

No new issue, no repository implementation change, no release/tag/artifact publication. The existing previously published source commit is the exact product result.

## Coordination controls

Product SHA is pinned, and links to earlier runtime evidence should resolve against the exact docs revision used at closing, not moving HEAD. The coordinator must re-read this file after publication, check live I086 labels/state, ensure top-level identifier/family/project-routing invariants and then apply `status:completed` / closed only when the remaining conditions are satisfied. This record alone is **not** a claim that the GitHub state transition has occurred.

## AI assistance and validation disclosure

This document was prepared with ChatGPT assistance from the owner's/human executor's explicit PASS reports and previously independently read GitHub code/revision. No build, tests, shell scripts or product Git operations were run by the coordinating assistant.

```text
ISSUE=I086/#840
SLICE=I086-FINAL
TYPE=IMPLEMENTATION_ACCEPTANCE
PRODUCT_REVISION=6dab6ecc08c9a2102a388e00908c710a15cf2a4c
PRODUCT_VERSION=0.3.298-SNAPSHOT
SPEC_REVISION=0.1.451
SOURCE_CHANGES=NONE
PRODUCT_GIT_PUBLICATION=NO_NEW_COMMIT_NEEDED
HUMAN_BASELINE=EXACT_HEAD_AND_CLEAN_DIFF_REPORTED
HUMAN_FULL_SUITE=PASS_EARLIER_SAME_REVISION
HUMAN_NATIVE_BUILD=PASS
HUMAN_NATIVE_TEST=PASS_CLI_GUEST_TEST_TOOL_DAP_INTERPRETER_ONLY
HUMAN_PORTABLE_VALIDATION=PASS
HUMAN_PORTABLE_SMOKE=PASS_PLAT054_MARKERS
HUMAN_ARCHIVE_IDENTITY=PASS
NATIVE_HOSTED_JAVA_EMBEDDING=NOT_TESTED_NOT_IN_MANDATORY_SCOPE
B011=CLOSED
B012=CLOSED
MANDATORY_ACCEPTANCE_GAPS=NONE
TECHNICALLY_CLOSEABLE=YES
ISSUE_CLOSURE=AWAITING_GITHUB_TRANSACTION
PLAT054_STATUS=OPEN_REVIEW
```
