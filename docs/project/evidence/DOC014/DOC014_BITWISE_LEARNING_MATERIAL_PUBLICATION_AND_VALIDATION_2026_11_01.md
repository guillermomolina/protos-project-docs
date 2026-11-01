# DOC014 — Fixed bitwise operators: runnable tutorial, cookbook and guide publication

**Work item:** [DOC014 #1020](https://github.com/guillermomolina/protos/issues/1020), a single documentation/learning-material implementation slice in `guillermomolina/protos`.
**Date:** 2026-11-01.
**Language approval/implementation provenance:** [D233-B1 #1016](https://github.com/guillermomolina/protos/issues/1016) ratified/closed; [I137 #1017](https://github.com/guillermomolina/protos/issues/1017) shipped the operator grammar and ordinary-message lowering in `0.3.431-SNAPSHOT`, spec `0.1.486`.
**Evidence type:** exact GitHub publication/source verification plus separately attributed maintainer-provided local PASS. This record is not an independently executed test log or a normative language decision.

## Published product identity

- **Product commit:** [`guillermomolina/protos@5b488fc11a3bacb73ee58df8435843d34f610b0e`](https://github.com/guillermomolina/protos/commit/5b488fc11a3bacb73ee58df8435843d34f610b0e).
- **Subject:** `DOC014: add Protos-first bitwise operator tutorial, example and guide links`.
- **Product parent:** `ab637aa264ddbb2b99fdb03cb19207cc19651f45` (another implementation, I138, intervened after I137; DOC014 follows the actual product HEAD, rather than falsely claiming I137 was its direct parent).
- **Exact delta:** 6 files, 196 insertions, 0 deletions; 2 new `.protos` learning sources and 4 maintained guide/index files; one commit.
- **Implementation version:** no new bump for DOC014, which changes no `src/` or `protos/lib/` source. The commit was published with the already-existing `0.3.432-SNAPSHOT` Maven baseline (I138). Bitwise syntax itself was delivered by I137 in `0.3.431-SNAPSHOT`.
- **Specification:** no changes; D233-B1 remains the normative basis at global spec revision `0.1.486` for bitwise operators. DOC014 claims no new syntax, Integer protocol, Truffle node, distribution or release.

## Published file-by-file trace

| Product source at immutable DOC014 commit | Verified contribution |
| --- | --- |
| [`protos/tutorials/01-values-and-slots/04-bitwise-operators.protos`](https://github.com/guillermomolina/protos/blob/5b488fc11a3bacb73ee58df8435843d34f610b0e/protos/tutorials/01-values-and-slots/04-bitwise-operators.protos) | New standalone-CLI introductory Protos lesson with all six fixed forms; named `bitAnd` equivalence; unbounded signed shifts and explicit 32-bit mask; precedence/XOR-versus-NOT; both bitwise `&` and `|` strict RHS evaluation versus lazy `&&`; custom non-Integer object overriding `bitAnd`; self-checking `Error().signal()` and six predictable print lines; complete APL Part 5 notice |
| [`protos/examples/numbers/bit-flags-and-masks.protos`](https://github.com/guillermomolina/protos/blob/5b488fc11a3bacb73ee58df8435843d34f610b0e/protos/examples/numbers/bit-flags-and-masks.protos) | New standalone-CLI cookbook: permission flag test/set/clear/toggle, explicit width mask, pack and extract octets, arithmetic negative shift and large shifts; self-checking and six expected `true` lines; complete APL Part 5 notice |
| [`protos/tutorials/README.md`](https://github.com/guillermomolina/protos/blob/5b488fc11a3bacb73ee58df8435843d34f610b0e/protos/tutorials/README.md) | Entry point for the beginner bitwise lesson without restructuring the existing 14-topic learning progression |
| [`protos/examples/README.md`](https://github.com/guillermomolina/protos/blob/5b488fc11a3bacb73ee58df8435843d34f610b0e/protos/examples/README.md) | New task-oriented bit-flags/masks row and standalone CLI usage; no import or host authority required |
| [`docs/guide/05-values-identity-equality-and-collections.md`](https://github.com/guillermomolina/protos/blob/5b488fc11a3bacb73ee58df8435843d34f610b0e/docs/guide/05-values-identity-equality-and-collections.md) | Extends existing Integer bitwise subsection with actual precedence, left-associativity, maximal-munch adjacency, negative-shift error, explicit masking and ordinary message dispatch; links to tutorial/cookbook |
| [`docs/guide/README.md`](https://github.com/guillermomolina/protos/blob/5b488fc11a3bacb73ee58df8435843d34f610b0e/docs/guide/README.md) | Discovery links to guide section, tutorial and example |

**Executable learning material coverage:** existing [`ProtosCliLearningMaterialsTest.java`](https://github.com/guillermomolina/protos/blob/5b488fc11a3bacb73ee58df8435843d34f610b0e/src/test/java/com/guillermomolina/protos/cli/ProtosCliLearningMaterialsTest.java) discovers every `.protos` program under `protos/tutorials/` and `protos/examples/` that contains `print(`. Both new sources contain `print(`; no changes to Java harness or native Test Tool corpus registration were made. The programs independently signal `Error` when their expected computations fail.

## Validation provenance and scope

The project maintainer reported after the product push:

> DOC014: add Protos-first bitwise operator tutorial, example and guide links pushed.
>
> el git diff check esta limpio.
>
> make test y make check pasan en local.

The prior editing-agent handoff reported the files edited and said it had run only read-only `git diff --check` (clean) and had not run tests. The maintainer's later reported integrated results are separate and supersede that pending-validation stage.

| Gate | Report | Grounding |
| --- | --- | --- |
| Six-file product commit | **VERIFIED** | Exact GitHub commit, changed-file list, immutable product blobs |
| Tutorial/example and related guide/index content | **VERIFIED** | Exact product source at published SHA |
| APL Part 5 notice in new files | **VERIFIED** | Published source headers |
| `git diff --check` | **PASS — maintainer reported** | Local human statement; command not independently rerun |
| `make test` | **PASS — maintainer reported** | Local human statement; no independent execution/log |
| `make check` | **PASS — maintainer reported** | Local human statement; no independent execution/log |
| Explicit standalone CLI invocation for each new program | **NOT SEPARATELY ATTESTED** | `make test` covers integrated validation; no separate stdout transcript supplied |
| `ProtosCliLearningMaterialsTest` individual results | **NOT SEPARATELY ATTESTED** | Auto-discovery is verified by source; maintainer reports integrated suite PASS |
| Independent CI/native image/performance results | **NOT CLAIMED** | Not required or separately supplied |

A passing `make test` attestation is not represented as an independently observed command run; neither a CI pass nor a released binary is inferred.

## Closure reconciliation

- Documentation is accurate to the shipped D233-B1/I137 operator language contract and illustrates ordinary Protos-first uses, rather than introducing new Java kernels or a second runtime syntax family.
- Both executable learning sources and their four index/guide integrations were published in one atomic DOC014 product commit.
- The user-requested durable record is revision-bound to the exact product commit; re-read after docs publication before final issue close.
- DOC014 may be closed as `completed` after a final linked GitHub Issue closure comment; remove stale `status:ready`, preserve `family:DOC`. D233/I137 and DOC013 remain closed.

```text
WORK_ITEM=DOC014
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=5b488fc11a3bacb73ee58df8435843d34f610b0e
PRODUCT_COMMIT_VERIFIED=YES
CHANGED_FILES=6
NEW_PROTOS_LEARNING_FILES=2
GUIDE_INDEX_FILES_MODIFIED=4
DOC014_IMPLEMENTATION_VERSION_BUMP=NO
BITWISE_ORIGIN=I137_0.3.431-SNAPSHOT
LOCAL_GIT_DIFF_CHECK=PASS_MAINTAINER_REPORTED
MAKE_TEST=PASS_MAINTAINER_REPORTED
MAKE_CHECK=PASS_MAINTAINER_REPORTED
DIRECT_CLI_OUTPUT=NOT_SEPARATELY_ATTESTED
INDEPENDENT_CI=NOT_ATTESTED
PROTOS_FIRST=YES
DURABLE_RECORD_DECISION=REQUIRED_USER_REQUESTED
ISSUE_CLOSURE_TARGET=COMPLETED
```
