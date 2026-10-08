# I086 / PLAT054-3B — Exact Java scalar argument admission in standard Polyglot embedding

**Status:** PUBLISHED — slice B complete, I086 still open  
**Date:** 2026-10-08  
**Product commit:** [`9d8fddc93594f4d73f0077a3aebe5cab8f7d8b22`](https://github.com/guillermomolina/protos/commit/9d8fddc93594f4d73f0077a3aebe5cab8f7d8b22)  
**Commit subject:** `I086 PLAT054-3B: admit exact Java Integer and String arguments in Value.execute`  
**Implementation issue:** [I086/#840](https://github.com/guillermomolina/protos/issues/840)  
**Approved decision:** [PLAT054/#838](https://github.com/guillermomolina/protos/issues/838)  
**Performance consumer:** [PERF032/#831](https://github.com/guillermomolina/protos/issues/831)  
**Spec revision:** `0.1.449` (unchanged)  
**Product version:** `0.3.284-SNAPSHOT`.

## Exact publication proof

The published product commit, verified from GitHub, modifies **four files only**:

- `src/main/java/com/guillermomolina/protos/execution/ProtosHostExecutableClosure.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardPolyglotEmbeddingTest.java`
- `pom.xml`
- `CHANGELOG.md`

The product `pom.xml` and root `CHANGELOG.md` were both updated together from `0.3.283-SNAPSHOT` to `0.3.284-SNAPSHOT`. There is no normative specification or benchmark change. No pending B-specific version finalization remains.

## Implementation evidence

In `ProtosHostExecutableClosure.admitArguments`, only a **standard-embedding adapter** admits additional host scalars. The session-prepared PERF033 adapter keeps its exact zero-argument contract.

- Java `Byte`, `Short`, `Integer`, `Long` -> `new ProtosIntegerValue(((Number)value).longValue())`, preserving the exact signed integer value.
- Java `String` -> `new ProtosStringValue(value)`, enforcing the existing Unicode scalar sequence invariant.
- Existing Protos-represented or ordinary Protos objects are passed unchanged.
- Conversion preserves positional order; the argument array is **cloned only if at least one argument requires conversion**. No conversion is done for zero arguments.
- Unpaired Unicode surrogate input is rejected; other unsupported Java values (`Float`, `Double`, `Boolean`, `Character`, Java `BigInteger`, arrays/collections, callbacks or arbitrary objects) are rejected at the interop argument boundary with `UnsupportedTypeException`, before execution of guest code. Rejection does not terminate the Process.
- Ordinary source/native guest Closure dispatch, default/rest binding, arity failure semantics and fatal unhandled guest Error rules remain the owners. No new guest-call engine or mandatory Task/RootTask is introduced.

The published tests cover Java scalar direct calls; positive/negative/zero and numeric boundaries; Long overflow into exact Protos arithmetic; ASCII/BMP/supplementary Unicode and scalar count; mixed Java and Protos arguments; positional order/default/rest semantics; unsupported types and invalid surrogates rejected before observable guest effects; fatal guest arity Error; and minimal Task/Actor carrier checks. Prior PERF033 adapter tests are retained.

## Validation provenance

The project owner reported, after pushing:

> I086 PLAT054-3B: admit exact Java Integer and String arguments in Value.execute pushed

> el git diff check esta limpio. Todos los tests han pasado en local

These statements are **human-executor reports**. The GitHub commit and test source were inspected; actual local test logs, precise counts, CI runs and durations were **not** supplied or independently re-executed.

```text
SLICE=I086_PLAT054_3B
PROTOS_REVISION=9d8fddc93594f4d73f0077a3aebe5cab8f7d8b22
PRODUCT_VERSION=0.3.284-SNAPSHOT
SPECIFICATION_REVISION=0.1.449
CHANGED_FILES=4
GIT_PUBLICATION=VERIFIED
HUMAN_LOCAL_TESTS=PASS_REPORTED
HUMAN_DIFF_CHECK=CLEAN_REPORTED
AGENT_EXECUTED_BUILDS_TESTS_GIT_IN_PROTOS=NO
JAVA_BYTE_SHORT_INTEGER_LONG=EXACT_PROTOS_INTEGER
JAVA_STRING=VALIDATED_PROTOS_STRING
UNSUPPORTED_TYPES=PRE_ENTRY_REJECTION
PERF033_SESSION_ADAPTER=ZERO_ARGUMENT_UNCHANGED
SLICE_B_COMPLETE=YES
I086_COMPLETE=NO
```

## Remaining implementation

**PLAT054-3C — packaged Core and no-option embedding** is the next bounded, independently testable implementation step. At present `ProtosEmbeddedProcess.resolveCoreRoot` only supports explicit `protos.CoreRoot` and `getLanguageHome()`; absent both it throws. Existing `ProtosCoreBootstrap`, `ProtosSourceFileLoader` and `ProtosStandardLibraryModuleResolver` use filesystem-style paths and source-loading assumptions. The next change must provide Core/standard-module resources *inside the packaged JAR* and preserve resolver identity and authority without relying on ambient working-directory or external absolute paths. Ensure the truly option-free public Java example works from the produced JAR in a fresh unrelated working directory, with default sandbox privileges and no Core installation.

**Subsequent PLAT054-3D**, if C cannot safely cover independently owned authority/lifecycle concerns within a bounded publishable patch: explicit host filesystem/network grant mapping, Actor thread permissions and failure mode, Context close with open Process resources/provider sessions, and cross-platform/runtime conformance. The exact correspondence between Truffle host authority and represented Protos capabilities is a security-relevant contract: no generic allowIO or thread policy may be guessed to supply broader privilege than the host explicitly granted. Unresolved semantic/authority choices must be surfaced to the owner before implementing them.

**PERF032** graph/timing parity is not implied by Java scalar argument support or by the product commit; benchmarks remain unchanged.

## AI assistance

The evidence record was prepared by ChatGPT using the published product commit, source and owner validation report. Product editing, local tests, commit and push were performed by the human executor, not ChatGPT.
