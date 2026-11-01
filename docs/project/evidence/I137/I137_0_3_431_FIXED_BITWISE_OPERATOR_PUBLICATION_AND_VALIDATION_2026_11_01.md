# I137 — Fixed bitwise operator sugar: publication and validation evidence (0.3.431)

**Date:** 2026-11-01
**Implementation owner:** [I137 #1017](https://github.com/guillermomolina/protos/issues/1017), one atomic slice in `guillermomolina/protos`.
**Previously ratified design:** [D233-B1 / D233 #1016](https://github.com/guillermomolina/protos/issues/1016), with [immutable language decision](../../decisions/language/D233_FIXED_BITWISE_OPERATOR_SUGAR_B1.md) and [comparison/owner approval evidence](../D233/D233_B1_COMPARATIVE_AND_APPROVAL_2026_11_01.md).
**Evidence type:** independently read GitHub publication facts, source-backed traceability, and separately attributed human maintainer validation report. Not a test-run log or a new normative decision.

## Exact publication identity

- **Product repository:** `guillermomolina/protos`.
- **Published commit:** [`c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5`](https://github.com/guillermomolina/protos/commit/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5).
- **Commit subject:** `I137: add fixed bitwise and shift operator syntax over Integer messages (0.3.431)`.
- **Parent:** `dd4db2e9722df293aac0f3674aa2d88b46841aff`, the I136 publication.
- **Version:** `0.3.431-SNAPSHOT` in product `pom.xml` and `CHANGELOG.md`.
- **Specification revision:** `0.1.486` in `spec/PROTOS_SPEC_CHANGELOG.md`.
- **Commit change summary:** 25 files, 937 insertions, 34 deletions; one public product commit, including source, tests, specification, guide, and final version metadata.
- **GitHub verification:** the commit object, changed-file inventory and relevant immutable source blobs were read through the GitHub repository connection. The commit is on the queried default-branch HEAD at verification time. This does not independently verify the maintainer's working tree or run any test.

## Ratified contract and published implementation

| Fixed source form | Ordinary message | Frontend owner |
| --- | --- | --- |
| `a & b` | `a.bitAnd(b)` | lexer, parser, canonicalizer |
| `a \| b` | `a.bitOr(b)` | lexer, parser, canonicalizer |
| `a ~ b` | `a.bitXor(b)` | lexer, parser, canonicalizer |
| `~a` | `a.bitNot()` | lexer, parser, canonicalizer |
| `a << n` | `a.shiftLeft(n)` | lexer, parser, canonicalizer |
| `a >> n` | `a.shiftRight(n)` | lexer, parser, canonicalizer |

- [`TokenType.java`](https://github.com/guillermomolina/protos/blob/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5/src/main/java/com/guillermomolina/protos/lexer/TokenType.java) and [`ProtosLexer.java`](https://github.com/guillermomolina/protos/blob/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5/src/main/java/com/guillermomolina/protos/lexer/ProtosLexer.java) add the five exact symbolic tokens; maximal munch remains unchanged and invalid composite spellings remain lexical errors.
- [`ProtosParser.java`](https://github.com/guillermomolina/protos/blob/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5/src/main/java/com/guillermomolina/protos/parser/ProtosParser.java) adds four left-associative levels from comparison toward additive: `|`, binary `~`, `&`, shifts `<<`/`>>`. The same `TILDE` token serves unary prefix NOT.
- [`Canonicalizer.java`](https://github.com/guillermomolina/protos/blob/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5/src/main/java/com/guillermomolina/protos/semantic/Canonicalizer.java) lowers all six operations to ordinary strict `CanonicalSend` with their original receiver and arguments, not new AST/runtime operators. `&&` and `||` still lower their RHS as a Closure, unlike the new bitwise binary sends.
- [`Structural.protos`](https://github.com/guillermomolina/protos/blob/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5/protos/tools/formatter/Structural.protos) implements lexical-safe prefix joining in Protos for unary operations and non-local return; it preserves existing grouping and formatter architecture.
- [`SHA256.protos`](https://github.com/guillermomolina/protos/blob/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5/protos/lib/crypto/SHA256.protos) now expresses private bit helpers with the approved operators and retains explicit 32-bit masking.
- The normative [grammar](https://github.com/guillermomolina/protos/blob/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5/spec/PROTOS_GRAMMAR.md) and corresponding `CALLABLES.md`, `VALUES_AND_COLLECTIONS.md`, `PROTOS_LANGUAGE_SPEC.md`, spec changelog and programmer guide record the same exact contract.

**Preserved boundaries:** D230-C1 unbounded Integer semantics and ordinary dispatch; D149 prohibition on user-defined symbolic operators; `^` as non-local return; lazy Boolean `&&`/`||`; strict maximal-munch rejection of unsupported symbolic runs. There are no new numeric Java kernels or Truffle/bytecode operation-node files in the 25-file published delta. No `^^`, `>>>` or compound assignment surface was added.

## Published regression and conformance evidence

- New [`surface-sugar/bitwise-operators.protos`](https://github.com/guillermomolina/protos/blob/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5/protos/tests/conformance/surface-sugar/bitwise-operators.protos), registered once as `suite-native` in [`manifest.tsv`](https://github.com/guillermomolina/protos/blob/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5/protos/tests/conformance/manifest.tsv): arithmetic, exact/large Integers, huge shifts, ordinary Error behavior, equivalent named messages, non-Integer overrides, strict left-to-right evaluation, Boolean laziness, precedence, and non-local return.
- Java regression changes include `ProtosLexerTest`, `ProtosParserStandardOperatorTest`, `ProtosParserNonLocalReturnTest`, `CanonicalizerFoundationTest`, `ProtosSourceLayoutViewTest`, `ProtosStaticAnalysisCoreTest`, `ProtosFormatterToolBootstrapTest` and `ProtosLanguageServerFormattingTest`.
- Lexer adversaries and formatter round-trip/idempotence are covered; there is no new parallel formatter or Java bitwise kernel.

## Human validation report and attribution

After pushing I137, the project maintainer explicitly reported:

> I137: add fixed bitwise and shift operator syntax over Integer messages (0.3.431), pushed.
>
> el git diff check esta limpio.
>
> make test y make check pasan en local.

Prior to final publication the maintainer also reported all tests green, including focal validation, in the I137 interactive workflow.

| Gate | Outcome | Attribution and limit |
| --- | --- | --- |
| Product publication, exact commit and 25-file delta | **VERIFIED** | GitHub commit and immutable contents |
| `git diff --check` | **PASS reported** | Maintainer local statement, not independently executed |
| `make test` | **PASS reported** | Maintainer local statement; canonical integrated Java/Protos suite |
| `make check` | **PASS reported** | Maintainer local statement |
| Earlier focused validations | **PASS reported** | Maintainer workflow; raw individual test logs not supplied |
| Distinct native-image build or execution | **NOT ATTESTED** | No claim beyond reported make targets |
| CI pipeline status | **NOT ATTESTED** | No independent green CI assertion |
| Benchmarked speedup | **NOT CLAIMED** | A syntax/desugaring change, not demonstrated performance work |
| Maintainer's local clean working tree after push | **NOT INDEPENDENTLY INSPECTED** | Only the diff check report is known |

The product revision and the owner's validation attestation jointly support I137 completion. This record does not infer an external release, independent test execution, performance improvement, or deployment.

## Closure reconciliation

- **Approval provenance:** the separate D233-B1 owner approval and ratified decision remain authoritative; this document does not approve new semantics.
- **Product:** one atomic commit at `c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5`, implementing exactly the selected source forms, Protos-first formatter and conformance.
- **Required durable project evidence:** this revision-coupled record, to be cited from I137's closure comment using its exact published project-docs commit.
- **Next coordination action:** close [I137 #1017](https://github.com/guillermomolina/protos/issues/1017) as `completed` after publication/verification of this record and a final issue closure comment; retain `family:I`, remove stale `status:ready`. D233 #1016 stays closed with optional post-ratification delivery cross-reference.

```text
WORK_ITEM=I137
DESIGN= D233-B1
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5
PRODUCT_COMMIT_VERIFIED=YES
IMPLEMENTATION_VERSION=0.3.431-SNAPSHOT
SPEC_REVISION=0.1.486
CHANGED_FILES=25
ONE_ATOMIC_SLICE=YES
PROTOS_FIRST_FORMATTER=YES
NEW_NUMERIC_TRUFFLE_NODES=NO
GIT_DIFF_CHECK=PASS_MAINTAINER_REPORTED
MAKE_TEST=PASS_MAINTAINER_REPORTED
MAKE_CHECK=PASS_MAINTAINER_REPORTED
CI=NOT_ATTESTED
SEPARATE_NATIVE_IMAGE=NOT_ATTESTED
BENCHMARK=NOT_CLAIMED
DURABLE_RECORD_DECISION=REQUIRED_USER_REQUESTED
ISSUE_CLOSURE_TARGET=COMPLETED
```
