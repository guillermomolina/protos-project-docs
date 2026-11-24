# I171 + I172 — Published primitive value storage and closure evidence (2026-11-24)

**Owner of this joint implementation slice:** [I171 #1160](https://github.com/guillermomolina/protos/issues/1160) (`Bytes`) together with [I172 #1161](https://github.com/guillermomolina/protos/issues/1161) (`Vector` and `Matrix`). This is **one shared revision-bound record** rather than two duplicative evidence files. Both implementation issues are independently closable; the patch was intentionally integrated into one product commit.

**Product revision (verified published):** [`guillermomolina/protos@c98be4618fa5e8dc7a89f5092d3712616803e501`](https://github.com/guillermomolina/protos/commit/c98be4618fa5e8dc7a89f5092d3712616803e501), on `main` when inspected 2026-11-24. Git commit message: `I171: Store Bytes as byte[] and Vector/Matrix as primitive long[]/double[] storage`. The commit's direct parent is [`89028494587d4d22309a5c048e37da3518170b18`](https://github.com/guillermomolina/protos/commit/89028494587d4d22309a5c048e37da3518170b18), the [AUD024](../AUD024/AUD024_RUNTIME_REPRESENTATION_SOURCE_TRACE_2026_11_23.md) audited baseline. The commit API reports **31 changed paths, +1,248 / −204 lines**, including source, tests, `pom.xml`, and `CHANGELOG.md`.

**Implementation version verified in the published product:** `0.4.193-SNAPSHOT`, from [`pom.xml`](https://github.com/guillermomolina/protos/blob/c98be4618fa5e8dc7a89f5092d3712616803e501/pom.xml) and the matching [`CHANGELOG.md` entry](https://github.com/guillermomolina/protos/blob/c98be4618fa5e8dc7a89f5092d3712616803e501/CHANGELOG.md). This is a development snapshot implementation commit, **not** a GitHub Release, Native Image publication or benchmark result.

## 1. I171 — Bytes storage, contracts, and producer boundaries

- [`ProtosBytesValue.java`](https://github.com/guillermomolina/protos/blob/c98be4618fa5e8dc7a89f5092d3712616803e501/src/main/java/com/guillermomolina/protos/runtime/io/ProtosBytesValue.java) owns a growable `byte[] values` with a logical length, no `List<Object>` backing; a constructor copying the supplied `byte[]`; `octetSnapshot` returns a defensive copy; `indexedSnapshot` and interop reify unsigned semantic Integer octets 0..255.
- The runtime mutation boundary rejects invalid host/guest values before changing content. `indexedAdd` and `indexedPut` return their exact supplied argument. Indexed removal preserves logical order; ordinary open/closed/frozen behavior and read-only interop remain governed by the existing specification.
- [`ProtosStructuredEachCalls.PreparedBytesEachCall`](https://github.com/guillermomolina/protos/blob/c98be4618fa5e8dc7a89f5092d3712616803e501/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/structured/ProtosStructuredEachCalls.java) captures a `byte[]` snapshot and reifies one semantic Integer per callback, keeping `ProtosCallRequest.call` and the already-established Bytecode DSL callback/continuation path. The Standard Bytes protocol's ordinary `each` also uses a stable snapshot.
- Published call-site changes replace per-octet host reconstruction with defensive `byte[]` construction in Encoding, byte/file/stream I/O, TextWriter, Entropy, TLS, HMAC, Argon2, embedding and catalog acquisition. Actor, isolated P and detached snapshot transfers append primitive octets **into the already memoized destination Bytes shell**, without replacing guest object identity or sharing the source buffer.
- TextReader regression coverage now distinguishes **invalid octet insertion rejected at the Bytes boundary** from **invalid lower `source.read` results** (non-Bytes or empty Bytes), which continue to produce `IOError` without an extra read.

## 2. I172 — Vector/Matrix numeric storage, R2, and graph transfer

- [`ProtosVectorValue.java`](https://github.com/guillermomolina/protos/blob/c98be4618fa5e8dc7a89f5092d3712616803e501/src/main/java/com/guillermomolina/protos/runtime/values/ProtosVectorValue.java) selects private `long[]` for homogeneous SmallInteger, `double[]` for homogeneous Float, or `Object[]` for rich/mixed Numbers and other valid internal transfer states. A mismatched indexed Number write widens to references while preserving logical values and the exact returned argument.
- [`ProtosMatrixValue.java`](https://github.com/guillermomolina/protos/blob/c98be4618fa5e8dc7a89f5092d3712616803e501/src/main/java/com/guillermomolina/protos/runtime/values/ProtosMatrixValue.java) adds nonempty `long[]` storage. Existing `double[]` Float R2 remains lazily packed/persistent; all other Numbers remain in `Object[]`. `readonlyPackedFloatsForRuntime` rejects incompatible backing without casting `long[]` to a reference or Float array. The approved D214-F1 reduction/kernel contract is unchanged.
- [`completeTransferForRuntime` in Vector/Matrix](https://github.com/guillermomolina/protos/blob/c98be4618fa5e8dc7a89f5092d3712616803e501/src/main/java/com/guillermomolina/protos/runtime/values/ProtosVectorValue.java) specializes the final contents **in the same object** after each shell's population. Actor and P continue to memoize the definitive `Object[]` transfer shell first and handle Fraction/Complex descriptors, cycles, aliases, parent, slots and state before exposing a completed destination. Matrix Float remains R1 until the previously approved lazy Float packing path.
- [`ProtosArrayValue`](https://github.com/guillermomolina/protos/blob/c98be4618fa5e8dc7a89f5092d3712616803e501/src/main/java/com/guillermomolina/protos/runtime/values/ProtosArrayValue.java) is unchanged; its existing `long[]`, `double[]`, `boolean[]` and `Object[]` representation was already coherent. Vector and Matrix continue to admit Numbers only, not Boolean; no public dtype, selector, coercion, numeric algorithm, generic Java storage infrastructure or old dispatch path was introduced by this slice.
- Authorizing decision: [owner-ratified PLAT068-R3](https://github.com/guillermomolina/protos-project-docs/blob/50eb8ff73b00006281c915396a8bf86d59d09631/docs/project/decisions/platform/PLAT068_R2_FLOAT_ADAPTIVE_MATRIX.md). [PLAT068 #959](https://github.com/guillermomolina/protos/issues/959) remains **OPEN/READY** for independent SIMD/CPU matrix ISA, GPU and other hardware questions; this implementation does not settle those.

## 3. Tests and validation provenance

The project owner supplied two local focused Maven results in the conversation:

1. `mvn -q test -Dtest='ProtosStandardBytesProtocolTest,ProtosAdditionalIndexedInteropTest,ProtosStandardEncodingProtocolTest,ProtosStandardLinearAlgebraProtocolTest,ProtosActorValueTransferTest,ProtosParallelExecutionTest,ProtosPerf006Plat028BytesEachCallbackTest,ProtosI091CompactIntegerSupportTest,ProtosTextReaderLineProtocolTest'` — process returned to shell without Maven failure; the `Bytes.each` printed guards show `SNAPSHOT=PASS`, `CALLBACK_RESULT_IGNORED=PASS`, `RESULT_RECEIVER_IDENTITY=PASS`, `SUSPEND_RESUME=PASS` and `COMPLETED_PREFIX_REPLAY=NO`. This was a **pre-final-correction** checkpoint.
2. `mvn -q test -Dtest='ProtosStandardLinearAlgebraProtocolTest,ProtosActorValueTransferTest,ProtosParallelExecutionTest,ProtosTextReaderLineProtocolTest'` — command returned to shell without Maven failure, after the focused final correction.

**Final product-publication validation:** the owner explicitly states **`git diff --check` clean** and **all local tests PASS**, then reports the combined product change **pushed**. This is **maintainer-reported** validation of the published candidate, not independently executed by the coordinating agent. The transcript does not independently specify the final invocation of the canonical integrated `make test`, Native Image or benchmarks. No independent complete CI run or performance improvement is asserted.

**Changed-source/acceptance checks evidenced by the published commit:** tests in `ProtosStandardBytesProtocolTest`, `ProtosAdditionalIndexedInteropTest`, `ProtosStandardEncodingProtocolTest`, `ProtosPerf006Plat028BytesEachCallbackTest`, `ProtosTextReaderLineProtocolTest`, `ProtosI091CompactIntegerSupportTest`, `ProtosStandardLinearAlgebraProtocolTest`, `ProtosActorValueTransferTest`, and `ProtosParallelExecutionTest` cover octets/invalid writes, unsigned interop, callback snapshots, numeric backing type transitions, Float special cases, transfer identity and descriptors, and the restored TextReader invalid-source `IOError`. These paths were verified in the exact published commit; no new tests were executed by the agent.

## 4. Closure routing

- **I171 #1160:** primitive Bytes representation and host runtime integrity implemented and published; owner-reported tests and diff check successful. **Implementation closure is warranted**, separate from future release packaging or unrelated FFI changes.
- **I172 #1161:** semantically applicable numeric backing and in-place Actor/P specialization implemented and published under the PLAT068-R3 bounded decision; owner-reported tests and diff check successful. **Implementation closure is warranted**, independent of hardware acceleration.
- **AUD024 #1159:** owner-approved audit already closed; this is its published corrective implementation, not a reopening of the audit.
- **PLAT068 #959:** remains open for outstanding hardware/backend work, with R3 storage extension consumed and completed.

```text
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=c98be4618fa5e8dc7a89f5092d3712616803e501
PRODUCT_IMPLEMENTATION_VERSION=0.4.193-SNAPSHOT
PRODUCT_COMMIT_VERIFIED=PASS
CHANGELOG_AND_VERSION_IN_SAME_PRODUCT_COMMIT=PASS
OWNER_REPORTED_GIT_DIFF_CHECK=PASS
OWNER_REPORTED_ALL_LOCAL_TESTS=PASS
FOCUSED_MAVEN_EVIDENCE=PASS_REPORTED_IN_CONVERSATION
CANONICAL_MAKE_TEST_COMMAND=NOT_INDEPENDENTLY_DOCUMENTED
NATIVE_IMAGE_GATE=NOT_SEPARATELY_DOCUMENTED
BENCHMARK_GATE=NOT_REQUIRED
INDEPENDENT_EXECUTABLE_VALIDATION_BY_COORDINATING_AGENT=NOT_RUN
I171_IMPLEMENTATION=COMPLETE
I172_IMPLEMENTATION=COMPLETE
PLAT068_HARDWARE_WORK=STILL_OPEN
```
