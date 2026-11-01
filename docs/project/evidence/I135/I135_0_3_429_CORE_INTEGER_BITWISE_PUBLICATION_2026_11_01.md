# I135 — Core Integer bitwise messages, Protos 0.3.429

**Date:** 2026-11-01. **Role:** immutable, non-normative implementation/publication and validation-provenance record for [I135 #1012](https://github.com/guillermomolina/protos/issues/1012). The live Issue controls closure; normative authority belongs only to the product's `spec/`.

- **Exact product revision:** [`guillermomolina/protos@fba47c40c4b79b6e580df1c1f78a3fcecbd067e2`](https://github.com/guillermomolina/protos/commit/fba47c40c4b79b6e580df1c1f78a3fcecbd067e2), confirmed as product `main` HEAD when this record was prepared.
- **Exact parent/baseline:** [`73bd9fe913421ff493146b546869f9a1f0d2b34e`](https://github.com/guillermomolina/protos/commit/73bd9fe913421ff493146b546869f9a1f0d2b34e); one product commit ahead, zero behind.
- **Commit subject:** `I135: implement Core Integer bitwise messages (0.3.429)`.
- **Maven implementation version:** `0.3.429-SNAPSHOT`; **global normative specification revision:** `0.1.484` dated 2026-11-01.
- **Design authority:** owner-approved D230-C1, [D230 #1011](https://github.com/guillermomolina/protos/issues/1011), [durable ratification](../../decisions/language/D230_CORE_INTEGER_BITWISE_OPERATIONS_C1.md) and [comparative investigation](../D230/D230_C1_COMPARATIVE_RESEARCH_2026_11_01.md). No new semantic design approved by I135.

## Exact product delta verified from GitHub

The published baseline-to-head comparison covers **one commit and ten changed files**:

| Path | Published responsibility |
| --- | --- |
| `src/main/java/com/guillermomolina/protos/execution/protocols/numeric/ProtosStandardIntegerProtocol.java` | Installs and validates six ordinary, overridable native `Integer` slots; arity-based validation rejects non-Integer arguments and negative shift counts |
| `src/main/java/com/guillermomolina/protos/runtime/ProtosNumericValueSupport.java` | Exact bitwise kernels: signed-64 operations when safe, arbitrary-precision `BigInteger` fallbacks behind Truffle boundaries, canonical result normalization, guarded Java shift-count lanes |
| `protos/lib/crypto/SHA256.protos` | Private consumer migrated from power/lookup-table bit manipulation to native Integer bitwise messages, preserving explicit 32-bit masks and `digest(Bytes)` |
| `protos/tests/conformance/integer/bitwise-messages.protos` | 16 suite-native `Test` cases, including algebraic identities, signed/large values, shift counts, error and receiver validation, extraction, ordinary overriding, normalization and hash |
| `protos/tests/conformance/manifest.tsv` | Registers the new conformance source |
| `src/test/java/com/guillermomolina/protos/execution/integration/modules/ProtosCoreNativeBoundaryArchitectureTest.java` | Registers the six new expected native Integer selectors |
| `spec/semantics/VALUES_AND_COLLECTIONS.md` | Normative **Standard Integer bitwise messages** section and reconciliation of the former exclusion |
| `spec/PROTOS_SPEC_CHANGELOG.md` | Global specification revision `0.1.484`, D230-C1 |
| `pom.xml` | Implementation version `0.3.429-SNAPSHOT` |
| `CHANGELOG.md` | I135 implementation entry for `0.3.429-SNAPSHOT` |

The six selectors are `bitAnd(other)`, `bitOr(other)`, `bitXor(other)`, `bitNot()`, `shiftLeft(count)` and `shiftRight(count)`. They operate on genuine unlimited semantic Integers under infinite two's-complement sign extension. Right shifts use floor division; left shifts are exact subject to implementation resources. Negative/non-Integer counts and invalid arities/receivers signal the standard `Error`. Result families follow signed-64 SmallInteger/BigInteger normalization. The implementation does **not** add symbolic operators, fixed-width numeric families, new Bytecode DSL operations or a privileged guarded send fast path.

The published `shiftLeft` primitive lane checks `0 <= n < 63` before Java shift operations; larger counts follow exact arithmetic rather than Java's masked shift-count semantics. The final `executeBitwise` code uses `operation.arity == 1` to decide whether the argument must be validated, instead of testing for a non-null host value. Retaining the approximately 115-line numeric kernels inside the already-oversized `ProtosNumericValueSupport` avoids exceeding the existing 30-file `runtime/` directory guard or widening its private exact-value/normalization API; no unrelated numeric refactor is claimed.

## Verification, explicitly separated by source

| Gate | Evidence | Attribution |
| --- | --- | --- |
| Product commit exists on `main` | **VERIFIED** — exact SHA and message above | Read-only GitHub commit and live branch read |
| Published product diff | **VERIFIED** — ten paths, one commit, parent baseline | Read-only GitHub comparison |
| Maven/spec revisions | **VERIFIED** — `0.3.429-SNAPSHOT` / `0.1.484` | Published product files |
| Focused bitwise, SHA-256 and Java architecture tests | **PASS REPORTED**, before final publication | Maintainer explicitly reported “Los tests focales pasan”; the subsequently changed arity guard was reviewed, but no new post-commit log is attached here |
| `git diff --check`, `git diff --cached --check` | **NOT INDEPENDENTLY ATTESTED** for the final published commit | No final maintainer result or executable log supplied in the current completion report |
| `make test` (integrated Java and Protos suites) | **NOT ATTESTED** | No explicit final PASS report or logs supplied for I135 |
| `make check` (static guards, including directory limit) | **NOT ATTESTED** | No explicit final PASS report or logs supplied for I135 |
| Native Image execution of new selectors | **NOT ATTESTED** | No specific I135 Native Image acceptance result supplied |
| Performance comparison | **NOT MEASURED** | No before/after timings available; no SHA-256 speedup claimed |

The maintainer's publication statement was `I135: implement Core Integer bitwise messages (0.3.429) pushed`. It establishes the reported publication intent, independently confirmed by the product commit; it does **not** by itself assert `make test`, `make check` or Native Image PASS. Existing Actor/P transfer tests provide coverage of ordinary normalized Integers; no I135-specific Actor/P test was added and no new transfer test PASS is claimed.

The final executable-impact classification is **FULL** because I135 changes shared runtime code, a Core protocol and the SHA-256 library. Under `AGENTS.work/IMPLEMENTATION.md`, full integrated validation is required for final top-level closure; this evidence record must not be interpreted as proof of a missing PASS. Any later maintainer validation attestation should be added as a new, properly attributed evidence record or Issue comment, without retroactively misrepresenting what was known when this snapshot was written.

## Exact provenance

```text
WORK_ITEM=I135
PARENT_DECISION=D230-C1
ISSUE=https://github.com/guillermomolina/protos/issues/1012
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=fba47c40c4b79b6e580df1c1f78a3fcecbd067e2
PRODUCT_PARENT_REVISION=73bd9fe913421ff493146b546869f9a1f0d2b34e
PUBLISHED_PRODUCT_COMMITS=1
PUBLISHED_CHANGED_PATHS=10
IMPLEMENTATION_VERSION=0.3.429-SNAPSHOT
GLOBAL_SPEC_REVISION=0.1.484
FOCAL_TESTS=PASS_MAINTAINER_REPORTED_PRE_FINAL_CHANGES
MAKE_TEST=NOT_ATTESTED
MAKE_CHECK=NOT_ATTESTED
NATIVE_IMAGE_I135=NOT_ATTESTED
PERFORMANCE_CLAIMS=NONE
DOCUMENTATION_PUBLICATION=THIS_REVISION_BOUND_RECORD
```
